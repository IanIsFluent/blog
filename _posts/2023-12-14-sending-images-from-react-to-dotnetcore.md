---
tags:
  - react
  - nextjs
  - C#
  - dotnet-core
  - react-hook-form
---

# Sending images from React to .NET Core

Normally we would send data from a React (NextJS in this case) front end to a ASP Dotnet Core API controller by JSON encoding the field data, then using the fetch API to post it. But this won't work if we send an image. We'd need to encode/decode the image somehow.

## 1. FormData to the rescue

Although it isn't simple to send an image as JSON, we can use the `FormData` object to send it as part of a multipart form. In order to do this, we need to create a `FormData` object and append the image to it. We can then send this object using the fetch API, as long as we remember to not set the `Content-Type` header to `application/json`!

## 2. The form (react-hook-form)

We are using the `react-hook-form` library to make dealing with form inputs easier. Here's an example NextJS page that has a form with a file input and a text input:

```tsx
export default function FilePage() {
  const { handleSubmit, register } = useForm<{
    text: string;
    file: FileList | null;
  }>({
    values: {
      text: '',
      file: null,
    },
  });

  return (
    <form onSubmit={handleSubmit(onSubmit)}>
      <div>
        <input type="file" {...register('file')} />
      </div>
      <div>
        <input type="text" placeholder="text here" {...register('text')} />
      </div>
      <div>
        <button type="submit">Submit</button>
      </div>
    </form>
  );
}
```

## 3. onSubmit

The `onSubmit` function will be called when the form is submitted - which I omitted from the above to keep it simple. `onSubmit` will be passed the form data, which we can use to create a `FormData` object and send to the API.

```tsx
  const onSubmit = async (data: { text: string; file: FileList | null }) => {
    const body = createFormData(data);

    const response = await fetch('api/file', {
      method: 'POST',
      body,
    });
```

## 4. createFormData

The `createFormData` function will create a `FormData` object and append the file and text to it. We can then send this to the API. Note that we need to check that the file is not null, and that if it has a length, we only append the first one - because file inputs allow multiple files to be selected - so the API uses the `FileList` type.

```tsx
const createFormData = (data: { text: string; file: FileList | null }) => {
  const formData = new FormData();
  // add all non-null values to the form data - and if they have a length, add the first one
  Object.entries(data).forEach(([key, value]) => {
    if (value && value.length == 1) {
      formData.append(key, value[0]);
    } else if (value) {
      formData.append(key, value as any);
    }
  });
  return formData;
};
```

## 5. The C# ASP Dotnet Core API

Putting all the above code together we can now receive the image in our API and write it to disk. In C# the type we need to use is IFormFile. So our API will receive this model type:

```csharp
public class FileFormModel
{
    public string Text { get; set; }
    public IFormFile File { get; set; }
}
```

And the controller will need to use the `[FromForm]` attribute to bind to the form data:

```csharp
[HttpPost]
[Route("file")]
public async Task<IActionResult> File([FromForm] FileFormModel model)
{
    var filePath = Path.Combine(Directory.GetCurrentDirectory(), "wwwroot", model.File.FileName);
    using (var stream = new FileStream(filePath, FileMode.Create))
    {
        await model.File.CopyToAsync(stream);
    }
    return Ok();
}
```

## Conclusion

By using FormData we can easily send file data from a React form to a .NET Core API.
