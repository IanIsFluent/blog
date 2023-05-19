---
tags:
  - azure
  - nextjs
  - static-web-apps
  - azure-pipelines
---

# Azure Static Web Apps with Next.js and Azure Pipelines

Building and deploying a Next.js site on Azure Static Web Apps with Azure Pipelines

## About Azure Static Web Apps

- Azure's answer to Vercel, Netlify, AWS Amplify etc
- Like an enhanced static file host
- Next.js deployments can run in two modes
  - Static HTML export - similar to hosting from an S3 bucket
  - Hybrid Next.js applications (preview) - similar to Vercel, Netlify etc
- Includes internal build tool [Oryx](https://github.com/microsoft/Oryx)
- Can deploy to multiple environments inside a single web app

The killer feature of static web apps is [Hybrid Next.js applications](https://learn.microsoft.com/en-us/azure/static-web-apps/deploy-nextjs-hybrid) which allows you to use Next.js SSR and API endpoints.

## About Next.js SSR vs SSG

- SSR - Server Side Rendering
  - Next.js will render the page on the server and send the HTML to the browser
  - This is the default mode for Next.js
  - The page will be rendered on every request
- SSG - Static Site Generation
  - Next.js will render the page on the server and save the HTML to a file
  - The page will be rendered once and then served from the file system

In order to use Next.js API endpoints, an Azure Static Web App needs to be deployed as a [Hybrid Next.js websites](https://learn.microsoft.com/en-us/azure/static-web-apps/nextjs#hybrid-nextjs-applications-preview) - and these run in SSR mode.

## Deploying a Next.js app with Azure Pipelines

In order to deploy as a Hybrid Next.js application, the site build has to happen inside the AzureStaticWebApp task (which uses Oryx). This is because [`skip_app_build` and `skip_api_build` features are not supported in preview](https://learn.microsoft.com/en-us/azure/static-web-apps/deploy-nextjs-hybrid#unsupported-features-in-preview) - and if you try to set `skip_app_build` then the task expects static site files, and you'll see the following error:

```
Failed to find a default file in the app artifacts folder (.next). Valid default files: index.html,Index.html.
If your application contains purely static content, please verify that the variable 'app_location' in your deployment configuration file points to the root of your application.
If your application requires build steps, please validate that a default file exists in the build output directory.
```

To set up deployment to an Azure Static Web App with Azure Pipelines, you need to:

1. Create an Azure Static Web App in the Azure portal and click `Manage deployment token` and copy the token.
1. In your Azure Pipeline, create a new variable called `static-web-apps-deploy-token`, paste the token into the value and set it to be secret.
1. Add an `AzureStaticWebApp@0` task to the pipeline yaml:

   ```yaml
   - task: AzureStaticWebApp@0
     inputs:
       azure_static_web_apps_api_token: $(static-web-apps-deploy-token)
       app_location: '/' # set this to the folder of your Next.js app
   ```

### Size Error

When we tried to deploy our Next.js app to Azure Static Web Apps, we got the following error:

```
The content server has rejected the request with: BadRequest
Reason: The size of the function content was too large. The limit for this Static Web App is 104857600 bytes.
```

This limit is only imposed on Hybrid Next.js applications - and there is a plan to raise it. In the meantime, you can work around it by removing the Next.js build cache folder before deployment.

The Next.js build cache folder very quickly gets very large. This folder isn't used at runtime, and [this GitHub issue](https://github.com/Azure/static-web-apps/issues/1034#issuecomment-1399256154) explains how to fix the error - by adding a custom build command to remove the cache folder before deployment:

```yaml
- task: AzureStaticWebApp@0
  inputs:
    ...
    api_build_command: 'rm -rf .next/cache/'
```

### Using Web App Environments

Azure Static Web Apps allows you to deploy to multiple environments inside a single web app. This is great for seeing/testing PR or staging build outputs. To enable them, just specify the production branch (usually `main` or `master`):

```yaml
- task: AzureStaticWebApp@0
  inputs:
    ...
    production_branch: 'main'
```
