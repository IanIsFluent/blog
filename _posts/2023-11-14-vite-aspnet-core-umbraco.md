---
tags:
  - umbraco
  - vite
  - aspnet-core
  - dotnet-core
---

# Vite with Umbraco for modern bundling and dev server

Umbraco has a script and asset bundling and minification solution built in, called [Smidge](https://docs.umbraco.com/umbraco-cms/reference/configuration/runtimeminificationsettings). But we have always needed more preprocessing.

We used to use Grunt or Gulp to do this processing, as well as sprite generation and image optimisation. But these tools are rather old now, and there are better options now in the form of bundlers such as [Webpack](https://webpack.js.org/) and [Vite](https://vitejs.dev/).

People have already [integrated Vite with Umbraco](https://skrift.io/issues/vite-and-umbraco-for-faster-frontend-builds/) due to its speed - and because it gives a modern development experience with hot module reloading. So we decided to try it for our next project.

## Vite.AspNetCore

The [Vite.AspNetCore](https://github.com/Eptagone/Vite.AspNetCore) nuget package provides middleware to help integrate Vite with an ASP.NET Core server (such as that used by Umbraco).

There's more work to do than just installing the nuget package, however - so after installing it, I made the following changes to my project - based on the [example ViteNET.MVC](https://github.com/Eptagone/Vite.AspNetCore/tree/main/examples/ViteNET.MVC):

- Copied `package.json`, `vite.config.js` and `tsconfig.json` to the root of the project
- Updated `appsettings.json` / `appsettings.Development.json` with the `"Vite"` section
- Added the two lines to the `Startup.cs` file (`services.AddViteServices();` and `app.UseViteDevMiddleware();`)
- Added the two sections from the `.csproj` file (after the comments `Ensure Node environment on Build` and `Build the final assets`)
- Added `@addTagHelper *, Vite.AspNetCore` to `_ViewImports.cshtml`
- Modified our `_Layout.cshtml` file to include the styles and scripts from the `main.ts` entrypoint

Once all those changes were made, the Vite dev server was running when debugging with F5 - and building the resources for deployment in the build process.

## Output paths

However, testing the Vite build locally was a little difficult because there were already site resources for our Umbraco site. in the output `wwwroot` folder - and running the build would output would leave files that we couldn't easily clean up.

It would be much better if we could give Vite its own output folder, so that it was free to clean it, and we could add it to `.gitignore`.

To do this, I made the following changes:

- Modified the `vite.config.js` file to add: `build: { outDir: '../wwwroot/vite' }`
- Added `"Vite": { "base": "/vite" }` in `appsettings.json`
- Added `/vite/` prefix to the path to `main.ts` in `_Layout.cshtml` and any other assset paths
- Added `/wwwroot/vite/` to `.gitignore`

## Spritemaps

The next feature we needed was to combine svg icons into a spritemap. For this we used [vite-plugin-svg-spritemap](https://www.npmjs.com/package/@spiriit/vite-plugin-svg-spritemap). `npm i --save-dev @spiriit/vite-plugin-svg-spritemap`.

The plugin uses a special `__spritemap` route in development, but [didn't update the manifest file correctly](https://github.com/SpiriitLabs/vite-plugin-svg-spritemap/issues/14), so I couldn't use it to change the path to the output file when running in production. But then the developer released a new version THAT DAY that fixed the issue!

So now that I could get the path to use in production, I could add this to the razor view:

```cshtml
<environment include="Development">
  <use href="__spritemap#sprite-icon-name"></use>
</environment>
<environment include="Production">
  <use href="$"{Manifest["sprite.svg"]}#sprite-icon-name"></use>
</environment>
```

But this is rather big and ugly, so I created a razor helper function to simplify it to:

```cshtml
  <use href="@viteFns.SpriteHref("icon-name")"></use>
```

To do this, I created a class that derived from `RazorPage<dynamic>` and used the IViteManifest service to get the path to the spritemap file for production:

```cs
public class ViteFunctions : RazorPage<dynamic>
{
    public readonly IViteManifest Manifest;
    public ViteFunctions(IViteManifest Manifest) { this.Manifest = Manifest; }
    public override Task ExecuteAsync() { throw new NotImplementedException(); }

    public string SpriteHref(string spriteName)
    {
        if (Environment.GetEnvironmentVariable("ASPNETCORE_ENVIRONMENT") == Environments.Development)
        {
            return $"__spritemap#sprite-{spriteName}";
        }
        else
        {
            // assume production
            return $"{Manifest["vite/spritemap.svg"]?.File}#sprite-{spriteName}";
        }
    }
}
```

I then needed to:

- Register the class in `Startup.cs`: `services.AddSingleton<ViteFunctions>();`
- Add `@inject TPI.Website.ViteFunctions viteFns` to `_ViewImports.cshtml`

## Calling JS functions inside razor views

The bundle created by Vite has a single entrypoint, and any unused exports are removed as part of minification. This means there wasn't an easy way to call a function with parameters serialised using Razor from inside the view code.

To leave exports from the entrypoint file, I needed to set: `config: { build: { preserveEntrySignatures: 'allow-extension' } }` in `vite.config.ts`.

Then I could add `import {exportedFn} from './vite/main.ts';` into a `<script type="module">` element in the view. But this won't work in production, because the `main.ts` entrypoint will end up being the processed/minified/hashed JS file. So I added another helper function to ViteFunctions to get the correct path based on the environment:

```cs
public string PathToJS(string path)
{
    if (Environment.GetEnvironmentVariable("ASPNETCORE_ENVIRONMENT") == Environments.Development)
    {
        return $"/{path}";
    }
    else
    {
        // assume production
        return $"/{Manifest["tpi/" + path]?.File}";
    }
}
```

Then the script in the page can look like:

```cshtml
<script type="module">
  import { exportedFn } from "@viteFns.PathToJS("main.ts")";
  exportedFn(@Json.Serialize(Model));
</script>
```

## Conclusion

Now we can use a fast, modern bundler and dev server for our front-end assets in our Umbraco sites, with all the features we need.
