---
tags:
  - umbraco
  - examine
  - lucene
  - index
  - search
---

# Adding data to an Umbraco index (Umbraco 11+)

[Umbraco](https://umbraco.com/) uses the Lucene search index - with a wrapper called [Examine](https://shazwazza.github.io/Examine/). If you want to customize the search index, you can attach an event handler to an event that fires when the index is updated and change/add to the data that is being stored.

## Adding the application started event handler

Unfortunately, there's no Umbraco notification handler for when the index is updated. But there is an event that fires when the application starts, and we can use that to attach our event handler to the index update event.

This is done by calling `AddEventHandler` to the `IServiceCollection` object in `Startup`:

```csharp
public void ConfigureServices(IServiceCollection services)
{
  services
    .AddUmbraco(_env, _config)
    .AddBackOffice()
    .AddWebsite()
    .AddNotificationHandler<UmbracoApplicationStartingNotification, ExtraIndexDataHandler>()
}
```

## Creating the event handler

The event handler needs to implement `INotificationHandler<UmbracoApplicationStartingNotification>` and will get the index we want to use (Umbraco has `ExternalIndex`, `InternalIndex` and `MemberIndex` built in) then add the event handler to the `TransformingIndexValues` event:

```csharp
public class ExtraIndexDataHandler : INotificationHandler<UmbracoApplicationStartingNotification>
{
    private readonly IExamineManager _examineManager;
    private readonly IExtraIndexDataHandler _handler;

    public ExtraIndexDataHandler(
        IExamineManager examineManager,
        IExtraIndexDataHandler handler)
    {
        _examineManager = examineManager;
        _handler = handler;
    }

    public void Handle(UmbracoApplicationStartingNotification notification)
    {
        if (_examineManager.TryGetIndex("ExternalIndex", out var index))
        {
            ((BaseIndexProvider)index).TransformingIndexValues += _handler.Handle;
        }
    }
}
```

## Creating the index data handler

Now we can do the work of adding the data to the index in our `IExtraIndexDataHandler` implementation. We need to wrap the code in a `try`/`catch` block otherwise an error when indexing one document can affect the rest of the index.

This handler will add the value of the `Interesting` property to the index for any document type with the alias `InterestingDocType`:

```csharp
public class ExtraIndexDataHandler : IExtraIndexDataHandler
{
    private readonly IUmbracoContextFactory _umbracoContextFactory;
    private readonly ILogger<ExtraIndexDataHandler> _logger;

    public ExtraIndexDataHandler(
        IUmbracoContextFactory umbracoContextFactory,
        ILogger<ExtraIndexDataHandler> logger)
    {
        _umbracoContextFactory = umbracoContextFactory;
        _logger = logger;
    }

    public void Handle(object? sender, IndexingItemEventArgs e)
    {
        try
        {
            HandleUnsafely(e);
        }
        catch (Exception ex)
        {
            _logger.LogError(ex, "Error attempting to add extra data to index while processing nodeId {nodeId}", e.ValueSet.Id);
        }
    }

    private void HandleUnsafely(IndexingItemEventArgs e)
    {
        if (!e.ValueSet.ItemType.InvariantEquals("InterestingDocType"))
        {
            return;
        }

        var values = e.ValueSet.Values.ToDictionary(x => x.Key, x => (IEnumerable<object>)x.Value);

        using var ctx = _umbracoContextFactory.EnsureUmbracoContext();
        var node = ctx.UmbracoContext?.Content?.GetById(int.Parse(e.ValueSet.Id));

        if (node == null)
        {
            return;
        }

        var interestingPropertyValue = node.GetValue("Interesting")?.GetValue();
        values.Add("Interesting", new [] { interestingPropertyValue });
        e.SetValues(values);
    }
}
```

## Adding implementation to DI

Of course we will also need to register the `IExtraIndexDataHandler` implementation in DI. This is done in `Startup` - also in the `ConfigureServices` method:

```csharp
public void ConfigureServices(IServiceCollection services)
{
  services.
    AddUmbraco()
  ...
  services.AddTransient<IExtraIndexDataHandler, ExtraIndexDataHandler>();
}

```

## Conclusion

Now we can add extra data to the index for any document type we want, and search for that data or include it in search results.
