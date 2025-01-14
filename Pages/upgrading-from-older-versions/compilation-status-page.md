# Compilation status page

**Compilation status page** is diagnostic tool for DotVVM which allows to easily check all [DotHTML page](~/pages/concepts/dothtml-markup/overview), [master page](~/pages/concepts/layout/master-pages), or [markup control](~/pages/concepts/control-development/markup-controls), and detect compilation errors in them. 

It is a very useful tool which can help while you upgrade DotVVM packages in your app. It can quickly ensure that all markup files are valid.

> Prior to DotVVM 4.0, the Compilation status page was distributed as a separate NuGet package. From DotVVM 4.0, it is included in the main `DotVVM` NuGet package. **The package `DotVVM.Diagnostics.StatusPage` should be uninstalled.**

## How it works

DotVVM views are compiled on demand when the page requests a DotHTML file. This package adds you one diagnostics page to you dotvvm application. When you access this status page by default on route `_dotvvm/diagnostics/status`, all DotHTML files registered in `DotvvmStartup.cs` will be compiled, and all errors will be reported.

![Compilation status page](https://raw.githubusercontent.com/riganti/dotvvm-samples-compilation-status-page/42184142d7905be3d2e23661dbb1905c3ed4ba80/docs/sample.PNG)


## Enable the Compilation status page

1. To enable the status page, register it in your `DotvvmStartup.cs` file:

```CSHARP
public void ConfigureServices(IDotvvmServiceCollection services)
{
    services.AddStatusPage();
    ...
}
```

2. Run the app and visit `_dotvvm/diagnostics/status` to see the status.


## Security

Having status page publicly available is not recommended because that would allow potential attackers to trigger page recompilation which could slow down the entire application.  
Due to this concern, there is an option to specify an authorization function which decides whether user will be allowed to access status page or not.  

The authorization function can be specified during status page registration. Examples of such, the authorization function can check the source IP address, or check whether the user identity contains some specific claim.

```CSHARP
options.AddStatusPage(new StatusPageOptions()
{
    Authorize = context => Task.FromResult(context.HttpContext.User.HasClaim(
            claim => claim.Type == ClaimTypes.Name && !string.IsNullOrWhiteSpace(claim.Value)
        ))
});
```


## Accessing compilation results via API

If manual checking using status page is not an option, then the status page API can be used. This can be especially useful for automated checking of the compilation status before/after deployment to the production.

The API can be found (in default configuration) at After registration using `_dotvvm/diagnostics/status/api`.  

The API is enabled by calling `AddStatusPageApi`.

```CSHARP
public void ConfigureServices(IDotvvmServiceCollection services)
{
    services.AddStatusPageApi();
    ...
}
```

### API Security

The API can be secured in the same way as the status page is. API can trigger only one compilation of any given page, so there should not be possibility of DDoS attack. Leaking of sensitive information is still possible, so some security measures should be put in place.

Additionally, the behavior for unauthorized access can be configured via `NonAuthorizedApiAccessMode` property on `StatusPageApiOptions`.

Possible values are:
-   `Deny`: Default behavior where unauthorized clients will receive `401` error code.
-   `BasicResponse`: Unauthorized clients will receive 200 for successful compilation and 500 for failed one.  
-   `DetailedResponse`: Even Unauthorized clients will receive complete enumeration of discovered errors.

```CSHARP
public void ConfigureServices(IDotvvmServiceCollection services)
{
    var statusPageApiOptions = StatusPageApiOptions.CreateDefaultOptions();
            
    statusPageApiOptions.NonAuthorizedApiAccessMode = NonAuthorizedApiAccessMode.BasicResponse;

    options.AddStatusPageApi(statusPageApiOptions);
    ...
}
```

## See also

* [Routing](~/pages/concepts/routing/overview)
* [Markup controls](~/pages/concepts/control-development/markup-controls)

