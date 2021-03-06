[![NuGet](https://img.shields.io/nuget/v/AspNetCoreSubdomain.svg)](https://www.nuget.org/packages/AspNetCoreSubdomain/)

# ASP.NET Subdomain Routing
Goal of that lib is to make subdomain routing easy to use in asp net core mvc applications. Normally you would use some custom route for some special case scenario in your app. This should solve most of issues while using subdomain routing. Inspired by couple of already existing libraries around the web which handle routing in some degree this should meet requirements:

1. Register subdomain routes as you would do with normal routes.
2. Make links, forms urls etc. in views as you would do with helpers in your cshtml pages.
3. Catch all route values in controller.

## Setup
### Startup.cs

Your application have to be aware of using subdomains. Important thing is to use method ```AddSubdomains()``` before ```AddMvc()```
```csharp
public void ConfigureServices(IServiceCollection services)
{
    // Add framework services.
    //...
    services.AddSubdomains();
    services.AddMvc();
}
```
You configure your routes just like standard routes, but you cannot use standard ```MapRoute``` methods. That will be explained later in wiki. Use MapRoute method from this lib extensions method which accepts ```hostnames``` as a parameter.
```csharp
public void Configure(IApplicationBuilder app, IHostingEnvironment env, ILoggerFactory loggerFactory)
{
    var hostnames = new[] { "localhost:54575" };
    
    app.UseMvc(routes =>
    {
        routes.MapRoute(
            hostnames,
            "NormalRoute",
            "{action}",
            new { controller = "SomeController" });

        routes.MapSubdomainRoute(
            hostnames,
            "SubdomainRoute",
            "{controller}", //that's subdomain parameter, it can be anything
            "{action}",
            new { controller = "Home", action = "Action1" });
    )};
}
```
## Usage
### Your .cshtml files
Goal of that library is not only catching routes for subdomain but also generating links to actions while persisting standard razor syntax. Helper below will generate url ```<a href="http://home.localhost:54575">Hyperlink example</a>```. Route named SubdomainRoute should catch that link.
```csharp
@Html.ActionLink("Hyperlink example", "Action1", "Home")
```

### Controller
Big  advantage of library is you can catch all route values with controller.
```csharp
//HomeController.cs
public IActionResult Action1()
{
    //code
}
```

Having url ```http://home.localhost:54575/``` will invoke ```Action1``` method in ```Home``` controller.

## Samples
For samples definitely look at ```Samples``` project. For running it you have to correctly setup some files.
### C:\Windows\System32\drivers\etc\hosts
```
    127.0.0.1   subdomain1.localhost
    127.0.0.1   subdomain2.localhsot
    127.0.0.1   subdomain3.localhost
    127.0.0.1   home.localhost
    127.0.0.1   test.localhost
    127.0.0.1	staticsubdomain1.localhost
    127.0.0.1	staticsubdomain2.localhost
    127.0.0.1	subdomains.page.localhost
    127.0.0.1	subdomain.forms.page.localhost
```
### {YourPath}\AspNetCoreSubdomain\src\.vs\config\applicationhost.config
That's only needed if running with Visual Studio.
Modify section ```<bindings>``` add:
```xml
    <binding protocol="http" bindingInformation="*:54575:subdomain1.localhost" />
    <binding protocol="http" bindingInformation="*:54575:subdomain2.localhost" />
    <binding protocol="http" bindingInformation="*:54575:subdomain3.localhost" />
    <binding protocol="http" bindingInformation="*:54575:home.localhost" />
    <binding protocol="http" bindingInformation="*:54575:test.localhost" />
    <binding protocol="http" bindingInformation="*:54575:subdomains.page.localhost" />
    <binding protocol="http" bindingInformation="*:54575:subdomain.forms.page.localhost" />
```
