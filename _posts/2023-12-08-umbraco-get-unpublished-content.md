---
tags:
  - umbraco
---

# Getting unpublished Umbraco content in Controllers (Umbraco 11+)

When working with [Umbraco](https://umbraco.com/), we sometimes need to get content that may not be 'published'. To do this in a controller, we need to use the Umbraco `ContentService`. But the methods here aren't as straightforward as you might expect. To get all content with a certain document type, you can use `GetPagedOfType` - but sometimes we just want them all, and this function's parameters aren't the easiest to use!

## 1. Getting the document type ID

`GetPagedOfType` takes an integer ID for the document type, but we usually want to use the document type alias. To get the ID, we can use:

```csharp
var contentTypeId = ContentTypeService.Get("DocTypeAlias")?.Id;
```

## 2. Getting the number of items

Now we can get the content, but we need to know how many items to get. If we are implementing paging correctly, we can just get the first page - but to simplify, we can get the count of items, then just request that many - assuming it is less than `int.MaxValue` we should be OK:

```csharp
ContentService.GetPagedOfType(contentTypeId, 0, 1, out var contentCount, null);
var intCount = Convert.ToInt32(contentCount);
var contentItems = ContentService.GetPagedOfType(contentTypeId, 0, intCount, out _, null);
```

## 3. What is an IQuery<IContent>?

The reason I wanted to write this post is that I didn't know how to use the query paramter and couldn't find out from the docs. I did find a forum post explaining and wanted to record it here for future reference. It turns out the IQuery it refers to is the SQL query that Umbraco will run behind the scenes when getting the content - so we'll need to inject the SQL Scope.

Here is an example of how you'd request all forms of document type with id `contentTypeId` that are 'draft' (not published or trashed):

```csharp
var contentTypeId = ContentTypeService.Get("DocTypeAlias")?.Id;

using var scope = ScopeProvider.CreateScope(autoComplete: true);
var query = scope.SqlContext.Query<IContent>().Where(c => !c.Published && !c.Trashed);

ContentService.GetPagedOfType(contentTypeId, 0, 1, out var contentCount, query);
var intCount = Convert.ToInt32(contentCount);
var unpublishedContentItems = ContentService.GetPagedOfType(formCT.Id, 0, intCount, out _, query);
```

Remember to inject `IContentService`, `IContentTypeService` and `IScopeProvider` into your controller constructor.

## Conclusion

Now we can get all unpublished content in our controller code, and we know how to use the `IQuery<IContent>` parameter of `GetPagedOfType` to filter the results.
