#HTTP Server

Thr HTTP server for SqlStreamStore is implemented as an aspnet core middleware.

##Initialization

There is an extension method named `UseSqlStreamStoreHal` located in the namespace `SqlStreamStore.HAL`. It applies to `IApplicationBuilder`. It has two parameters, one optional:
```
public void Configure(IApplicationBuilder app) => app
    .UseSqlStreamStoreHal(IStreamStore streamStore, SqlStreamStoreMiddlewareOptions options = default);

```
where `streamStore` is an implementation of `IStreamStore`, and options (optional) is an instance of `SqlStreamStoreMiddlewareOptions`. This options class has a single configuration value: `UseCanonicalUrls`. If `true`, the middleware will sort your query string parameters and issue a redirect if the query string parameters you used are not expressed in the same order. This is to aid in HTTP caching. Set to `false` if your reverse proxy already sorts your query string parameters, e.g., AWS API Gateway. Defaults to `true`.