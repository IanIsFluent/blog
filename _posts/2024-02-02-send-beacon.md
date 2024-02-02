---
tags:
  - js
  - C#
  - dotnet-core
---

# Tracking page clicks with `navigator.sendBeacon`

When you want to track that a link has been clicked on your site, the browser will not wait for your tracking request to complete - which means you may need to prevent navigation to the link, so you can wait for the request to complete before navigation.

## 1. The Beacon API

[Documented here](https://w3c.github.io/beacon/), the Beacon API allows you to send data to the server asyncronously and guarantees the request will be delivered to the server, even if the user navigates away from the page.

Here's an example of how you might use it to post a string to track a click:

```html
<a href="/some-page" onclick="sendBeacon('/some-page')">Click me</a>
```

```js
function sendBeacon(url) {
  navigator.sendBeacon('/click', url);
}
```

## 2. Problems with `sendBeacon`

The problem is that the `sendBeacon` only allows some content type headers, which are determined based on what is being sent. By default it will use text (`text/plain`). And the problem with that is its quite difficult to get to body text in a `text/plain` POST request in ASP.NET Core. You need to customize the middleware to allow the body to be read twice.

## 3. Using `FormData`

If you pass `UrlSearchParams` to `sendBeacon`, it will use `application/x-www-form-urlencoded`. If you pass a `FormData` object, it will use `multipart/form-data`. If you pass a `Blob` you can specify the content type, but it will be `application/octet-stream` by default.

So for ASP.NET Core, I found the easiest thing was to use FormData. So with this JS:

```js
function sendBeacon(url) {
  const formData = new FormData();
  formData.append('url', url);
  navigator.sendBeacon('/click', formData);
}
```

You can bind the request body with the `[FromForm]` attribute in the C# controller method:

```csharp
[HttpPost]
[Route("click")]
public async Task<IActionResult> Click([FromForm] string url)
{
    // track the click
    return Ok();
}
```
