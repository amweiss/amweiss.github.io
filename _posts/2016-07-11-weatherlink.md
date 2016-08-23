---
title: WeatherLink - An experiment with ASP.NET Core
categories: development
excerpt: ASP.NET Core hit 1.0, let's play with it a bit and see how to build a composite service.
tags: dotnetcore aspnetcore
---

{% include toc title="Contents" icon="file-text" %}

There are times when I struggle with focusing more on the management side rather than the development.
I do think it's a benefit to maintain some up-to-date technical skills to help with technical guidance and decisions.
For on-the-job learning, I've been looking at [Scala](http://www.scala-lang.org/) in relation to work and then expanding to [Lagom](https://www.lightbend.com/lagom).
However, for personal learning, I tend to stick to the .NET stack. In the .NET world lots of things have been changing recently.

## Technology

[.NET Core](https://www.microsoft.com/net/core) hit 1.0 and [ASP.NET Core](http://www.asp.net/core) hit 1.0, but best of all, they are open source!
For Windows based platforms (with some [restrictions](https://www.visualstudio.com/support/legal/mt171547)), [Visual Studio Community](https://www.visualstudio.com/en-us/products/visual-studio-community-vs.aspx) is an option, but for everything else (including OSX and Linux),
we now have [Visual Studio Code](https://code.visualstudio.com/)

So to get started, I went with Visual Studio Code so I could see how the new tool worked. Overall, I really liked it and am writing this post on it now as well.
I went with the [Insiders](https://code.visualstudio.com/insiders) build to get an in-editor terminal and tabs, but those should land in the standard version soon if not already.
It provides the familiar Intellisense like Visual Studio and much of the same functionality .NET developers are used to.

![Intellisense](/images/images/vscode-intellisense.png){: .align-center}

## Setup

I'm going to highlight some of the parts I found interesting, but the whole source is available on [GitHub](https://github.com/amweiss/WeatherLink). The `README` is a bit lacking and I apologize for that.

### Program

Those of your familiar with running ASP.NET in IIS will be somewhat surprise by [`Program.cs`](https://github.com/amweiss/WeatherLink/blob/master/src/WeatherLink/Program.cs)

```cs
public static void Main(string[] args)
{
    var config = new ConfigurationBuilder()
                    .AddCommandLine(args)
                    .AddEnvironmentVariables()
                    .Build();

    var host = new WebHostBuilder()
        .UseConfiguration(config)
        .UseKestrel()
        .UseContentRoot(Directory.GetCurrentDirectory())
        .UseIISIntegration()
        .UseStartup<Startup>()
        .Build();

    host.Run();
}
```

I have `UseKestrel()` as well as `UseIISIntegration()`. This is due to running the services on [Azure](https://azure.microsoft.com/en-us/).
Even though I was using the self-hosted [Kestrel](https://github.com/aspnet/KestrelHttpServer) server, IIS Integration has to be enabled for Azure to serve the content.
You can also see [`Startup`](https://github.com/amweiss/WeatherLink/blob/master/src/WeatherLink/Startup.cs) referenced here and that's where more of the new fun shows up.
{: .notice--info}

### Startup

The `Startup` class may looks like a bit of magic at first and heavily uses the [Builder Pattern](https://en.wikipedia.org/wiki/Builder_pattern).

#### Configuration

In `Startup` there's a few interesting parts. First, the configuration:

```cs
public Startup(IHostingEnvironment env)
{
    var builder = new ConfigurationBuilder()
        .SetBasePath(env.ContentRootPath)
        .AddJsonFile("appsettings.json", optional: true, reloadOnChange: true)
        .AddJsonFile($"appsettings.{env.EnvironmentName}.json", optional: true)
        .AddEnvironmentVariables();
    Configuration = builder.Build();
}
```

I left the `env.EnvironmentName` overrides in, but I don't personally use them currently. What I really like is that right out of the box, I can load a settings file [`appsettings.json`](https://github.com/amweiss/WeatherLink/blob/master/appsettings.json) and then use environment variables to override the values. This makes keeping secrets out of git trivial which is a huge plus for open source work. See [Working with Multiple Environments](https://docs.asp.net/en/latest/fundamentals/environments.html) for more details. To setup the overrides, on my local machine and Azure instance I have (with actual values added):

```sh
export DarkSkyApiKey=TOPSECRET
export GoogleMapsApiKey=TOPSECRET
export SlackTokens__0=TOPSECRET
export SlackTokens__1=TOPSECRET
```

This shows you a top level element and how arrays work. If you needed to have sub-elements, you would use the same `__` syntax to show the hierarchy.

#### Services

The next section shows how to configure the [Dependency Injection](https://docs.asp.net/en/latest/fundamentals/dependency-injection.html).

```cs
// Add custom services
services.Configure<WeatherLinkSettings>(Configuration);
services.AddTransient<IForecastService, DarkSkyForecastService>();
services.AddTransient<ITrafficAdviceService, WeatherBasedTrafficAdviceService>();
services.AddTransient<IGeocodeService, GoogleMapsGeocodeService>();
```

Here the setup defined in [Configuration](#configuration) is filled in with the `WeatherLinkSettings` object that represents the data. Following that is the real fun. It establishes [transient dependencies](https://docs.asp.net/en/latest/fundamentals/dependency-injection.html#service-lifetimes-and-registration-options) for each of the interfaces I created. I also configure [Swagger](http://swagger.io/) via [Swashbuckle](https://github.com/domaindrivendev/Swashbuckle) (specifically the [Ahoy](https://github.com/domaindrivendev/Ahoy) project to port it to ASP.NET Core.

```cs
// Configure swagger
services.AddSwaggerGen();
services.ConfigureSwaggerGen(options =>
{
    options.SingleApiVersion(new Info
    {
        Version = "v1",
        Title = "Weather Link",
        Description = "An API to get weather based advice.",
        TermsOfService = "None"
    });
    options.IncludeXmlComments(GetXmlCommentsPath());
    options.DescribeAllEnumsAsStrings();
});
```

#### Runtime

Finally, we get to the `Configure` method which sets up the actual HTTP request pipeline.

```cs
public void Configure(IApplicationBuilder app, IHostingEnvironment env, ILoggerFactory loggerFactory)
{
    loggerFactory.AddConsole(Configuration.GetSection("Logging"));
    loggerFactory.AddDebug();

    app.UseStaticFiles();

    app.UseMvc();

    app.UseSwagger();
    app.UseSwaggerUi();
}
```

This sets up [logging](https://docs.asp.net/en/latest/fundamentals/logging.html) to the console, serving [static files](https://docs.asp.net/en/latest/fundamentals/static-files.html), the default [MVC](https://docs.asp.net/en/latest/mvc/index.html) handling, and finally Swagger.

## Organization

Below the top level files already discussed there are a few others, [project.json](https://github.com/amweiss/WeatherLink/blob/master/src/WeatherLink/project.json), [Dockerfile](https://github.com/amweiss/WeatherLink/blob/master/src/WeatherLink/Dockerfile), etc. More interestingly is the `src` directory. Inside that is where all the rest of the program source code is.

### Models

Models are very familiar if you did ASP.NET MVC before. They are [POCOs](https://en.wikipedia.org/wiki/Plain_Old_CLR_Object) to represent your data.

### ExtensionMethods

Another friend from previous versions, [Extension Methods](https://msdn.microsoft.com/en-us/library/bb383977.aspx) are often helpful to have. I had to shamelessly copy one from [MoreLINQ](https://github.com/morelinq/MoreLINQ/blob/master/MoreLinq/MinBy.cs) as it does not support .NET Core yet.

### Controllers

Again, if you are familiar with ASP.NET MVC this will not be a surprise. [Controllers](https://docs.asp.net/en/latest/mvc/controllers/actions.html) define a set of actions the application can perform. The new elements here are the baked-in [dependency injection](#services). If you're a bit behind the times in C#, [`async` and `await`](https://msdn.microsoft.com/en-us/library/mt674882.aspx) are used here as well as [routing attributes](https://docs.asp.net/en/latest/fundamentals/routing.html) that carried over from the WebAPI days.

```cs
[Route("{latitude}/{longitude}")]
[HttpGet]
public async Task<string> GetTrafficAdvice(double latitude, double longitude)
```

Overall, the controller is fairly boring (as it should be).

### Services

Now we get to the really fun part, the services. Following [standard naming conventions](https://msdn.microsoft.com/en-us/library/ms229040(v=vs.110).aspx) the classes starting with `I` are interface declarations.

#### Geocoding

[`GoogleMapsGeocodeService`](https://github.com/amweiss/WeatherLink/blob/master/src/WeatherLink/Services/GoogleMapsGeocodeService.cs) uses the [Google Maps API](https://developers.google.com/maps/) to convert a string into a latitude and longitude. It does the processing with the ever popular [Json.NET](http://www.newtonsoft.com/json). I was lazy and just returned a `Tuple` instead of creating a geolocation model class. I also just used the `JObject` directly combined with [null-conditional operators](https://msdn.microsoft.com/en-us/library/dn986595.aspx) to follow the concept of the [Robustness principle](https://en.wikipedia.org/wiki/Robustness_principle) and be liberal in what is received. If the elements expect are present, it returns a value regardless of whatever else is returned and I barely use any of the data so I skipped creating an object for it.

```cs
var location = responseJObject?["results"]?.First()?["geometry"]?["location"];
var recievedLatitude = location?["lat"];
var recievedLongitude = location?["lng"];
```

#### Forecasts

I love [Forecast.io](https://forecast.io) and their hyper-local [API](https://developer.forecast.io/) is what powers WeatherLink's [forecast service](https://github.com/amweiss/WeatherLink/blob/master/src/WeatherLink/Services/HourlyAndMinutelyDarkSkyService.cs). Again, the actual API call is made through [`HttpClient`](https://msdn.microsoft.com/en-us/library/system.net.http.httpclient(v=vs.110).aspx) and parsed with Json.NET.

I do forcibly set the numeric format to be `N4` on the URL parameters to avoid issues, but it seemed like the DarkSky API handled it without that.
{: .notice--info}

For this web service, I do deserialize directly to an object for parts of the response, but not the entire thing for efficiency reasons. After selective deserializing, I combine the results into a `Forecast` object.

#### Traffic advice

[This](https://github.com/amweiss/WeatherLink/blob/master/src/Services/WeatherBasedTrafficAdviceService.cs) is the meat and potatoes of the service. It's also where you will find wonderful comments like:

```cs
//TODO: I hate this, fix it
```

I'm a big proponent of make it work first and make it better later. The methods are bit long for my preferences right now but they are basically string builders that operate on the results from the [forecast service](#forecasts). I think most of the code is fairly readable but I did leave a few magic numbers in like taking the 5 closest times around your target time to compare to for leaving.

```cs
var range = forecasts.Skip(afterTarget - 2)
    .Take(5)
    .Where(x => Math.Abs((DateTimeOffset.FromUnixTimeSeconds(x.time) - targetTime).Hours) <= 1)
    .ToList();
```

or a commute time of 20 minutes (I love [Buffalo](https://www.google.com/maps/place/Buffalo,+NY/@42.8962176,-78.9460568,12z/data=!4m5!3m4!1s0x89d3126152dfe5a1:0x982304a5181f8171!8m2!3d42.8864468!4d-78.8783689!5m1!1e1)):

```cs
var bestTimeToLeaveWork = forecasts.Any() ? forecasts.MinimumPrecipitation(20)?.FirstOrDefault() : null;
```

## Response times

This is fairly limited profiling, but I generally see 500ms - 800ms response times for the geocoding end point as well as the direct latitude and longitude endpoint. Based on that, it seems like making the geocoding API request is adding a negligible delay and that the majority of that time is spent in my service. I know my current math is a bit cumbersome so I might be able to reduce that a bit more.

## Summary

Overall, I like the new features in ASP.NET Core and with this basic example I didn't hit any of the current restrictions. However, what I think the best part is that it now runs anywhere (mostly) and can be developed in entirely [free tools](#technology). This is a great step forward and I can't wait to see what else Microsoft, and the community, is going to do with the platform.