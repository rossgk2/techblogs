# ASP.NET Core Web API notes

## Legacy ASP.NET, before "Core"

In legacy ASP.NET, there were separate frameworks, each dedicated to different tasks:

* ASP.NET Web API - defining APIs

- ASP.NET MVC - writing web applications

In modern ASP.NET, there is a single framework- ASP.NET Core, or "Core". The different tasks of API definition and web app development are both accomplished by using the same core framework.

"ASP.NET Core Web API" is a loose term; it translates to "the libraries and usage patterns in ASP.NET Core that are used to implement APIs". An ASP.NET Web API project is one that

- Uses ASP.NET Core (including Microsoft.AspNetCore.Mvc)

- Uses `ControllerBase`

- Does not include views

- Perhaps uses `[ApiController]`

"ASP.NET Core Web App" is also a loose term; it translates to "the libraries and usage patterns in ASP.NET Core that are used to implement web applications". An ASP.NET Web App project is one that

- Uses ASP.NET Core (including Microsoft.AspNetCore.Mvc)

- Uses Razor Pages *or* MVC Controllers + Views

- Renders HTML instead of JSON

- Uses UI-focused features (Tag Helpers, Layouts, Razor syntax)

Confusingly, "ASP.NET Core Web API" and "ASP.NET Core Web App" both depend heavily on Microsoft.AspNetCore.Mvc, which is can be misinterpreted as being evocative of the legacy framework "ASP.NET MVC".

## Controllers

### Controllers in ASP.NET Core Web API

In ASP.NET Core Web API, every API endpoint corresponds to a *controller class*- a class inheriting from `ControllerBase`- that defines how HTTP requests to that API endpoint are to be handled. 

Specifically, handlers for the different verbs (GET, PUT, POST, PUT, DELETE, PATCH) that can be issued to that API endpoint are defined as methods of the controller class. Such methods are called *action methods*.

We describe how a HTTP request sent to an API endpoint is matched to a controller action. There are two steps to this process:

1. Determining the controller corresponding to the API endpoint
2. Determining the action corresponding to the HTTP verb

#### Conventional routing

In the olden days, before the `[Route]` attribute existed, the following *global* configuration code associating URLs with controller actions was placed in `Program.cs`, the entry-point file:

```c#
app.UseEndpoints(endpoints =>
{
    endpoints.MapControllerRoute(
        name: "default",
        pattern: "{controller}/{action}/{id?}");
});
```

This code causes a URL such as "/Clients/Info" to be handled by the action method named `Info` in the controller called `ClientsController`:

```c#
public IActionResult Info()
{
    return Ok("Info action called");
}
```

Though, the first above block would actually typically include some defaults, so that the URL "/" is handled by the action called `Index` in the controller called `HomeController`, and would look like this:

```c#
app.UseEndpoints(endpoints =>
{
    endpoints.MapControllerRoute(
        name: "default",
        pattern: "{controller=Home}/{action=Index}/{id?}");
});
```

#### Attribute-based routing

##### 1. Determining the controller

Modern code uses C#'s decorator syntax- attributes- to decentralize the configuration code. The association of URLs with controllers no longer happens all in one place; instead, it is done on a per-controller, by annotating the controller:

```c#
[Route("[controller]")]
public class ClientsController : ControllerBase { ... }
```

The `"[controller]"`  argument to the `Route` attribute indicates that the name of the decorated controller, minus "Controller", prepended with a "/", and put into lowercase, is the URL to be associated with `ClientsController`. That is, since `[Route("[controller]")]` decorates `ClientsController`, then "/clients" is the URL handled by `ClientsController`.

##### 2. Determining the action method

In the old approach, determination of the action method relies a lot upon enforcing naming conventions on method names. That is much less so the case in the attribute-based approach. Similarly to as with the controller, the action is annotated with the HTTP verb to be matched. 

```c#
[HttpGet]
public IActionResult Get()
{
    return Ok("Info action called");
}
```

A HTTP is matched to an action with the following algorithm:

1. Filter all actions so that the actions under consideration are the ones with routes (formally called "route templates") that match the request's URL path.
2. If there exists a matching action with whose HTTP verb attribute (e.g. `[HttpGet]`) matches that of the request, use one of those actions.
3. Otherwise, use conventional routing to match the HTTP request to an action method that participates in conventional routing (i.e. that doesn't have an HTTP verb attribute).

### Controllers in ASP.NET Core Web App

Controllers in ASP.NET Core Web App inherit `Controller`. (Recall that controllers in ASP.NET Core Web API inherit `ControllerBase`.)

### More on controllers in ASP.NET Core Web API

Controllers in ASP.NET Core Web API (classes that inherit from `ControllerBase`) have basic functionality for building API controllers, like automatic model binding.

When a controller is additionally annotated with `[ApiController]`, its methods now *must* be annotated with a HTTP verb attribute (like `[HttpGet]` or `[HttpPost]`).

## Requests

As one would expect, the types of the inputs in a controller action method correspond to the kinds of the inputs can be passed in a HTTP request that's sent to the corresponding API endpoint.

### Scalar input and output

To represent scalar inputs (strings, numbers, or booleans), simply use a primitive type like `string`, `int`, `double`, or `bool`. An example of an action method corresponding to a POST endpoint that accepts a `string` input is

```c#
[HttpPost("resource")]
public async Task<ActionResult<OuterWrapper>> Post(string input)
{
    return await clientsService.GetResult();
}
```

### JSON input and output

To represent JSON input in particular, it's most common to use a complex type. For example, suppose that the API endpoint POST /api/resource receives an HTTP request containing the following JSON payload[^1]:

[^1]: "A HTTP request containing a JSON payload `json`" = "A HTTP request with `Content-Type` = `application/json` and a JSON-formatted string `json` provided as the request body"

  ```json
{
	"inner": 
	{
		"a": 1,
		"b": {"name": "bob"},
		"c": [30, "hi", false],
		"d": 4,
		"e": 5
	}
}
  ```

To properly receive this JSON payload as an input, we need to define a type that mimics its schema, like so:

  ```c#
public record OuterType
{
	public InnerType inner { get; set; } // matches up with "inner: { ... } "
}

public record InnerType
{
	public int a { get; set; } // matches up with "a": 1
	public Dictionary<string, string> b { get; set; } // matches up with "b": {"name": "bob"} 
	public IEnumerable<object> c { get; set; } // matches up with "c": [30, "hi", false]
	public int d { get; set; } // matches up with "d": 4
	public int e { get; set; } // matches up with "e": 5
}
  ```

(One strange "gotcha" is that you have to define properties (like `public int a { get; set; }`) instead of fields (like `public int a;`)! Otherwise, the type will not be properly interpreted by the middleware.

Also, notice that *nesting* in the JSON schema corresponds to *composition* of complex types in the type representation of the schema.)

Now that we can represent the JSON payload input, we can define a  `[HttpPost]` action method that receives it:

```c#
[HttpPost("resource")]
public async Task<ActionResult<OuterWrapper>> Post(Outer input)
{
    return await clientsService.GetResult();
}
```

The middleware will take care of parsing the JSON payload provided by an API caller, so that, at the beginning of `Post()`, action method, the properties of that `OuterType` variable are populated as you would expect. 

### General typing

While it is most common- and probably best practice- to represent JSON inputs as complex types, there are some more general ways to do so worth mentioning. 

The *most* general way to represent a JSON input is to use `Dictionary<string, System.Text.JsonElement>`.

Scalars can also be represented in a more general way. Since scalars can be interpreted as constituting valid JSON payloads, they can be represented as `JsonElement`. 

Here are the full details:

| Kind of input or output to API endpoint            | Types that can represent it                                  |
| -------------------------------------------------- | ------------------------------------------------------------ |
| Raw scalar value                                   | - `JsonElement` <br />- A primitive type (`int`, `float`, `string`, `bool`, etc.) |
| JSON list of scalar values                         | - A weakly derived type of `ICollection<JsonElement>` <br />- A weakly derived type of `ICollection<Entity>`, in the special case when the elements of the array are all representable by type `Entity`<br />- A type (`record`, `class`, or `struct`) wrapping any of the above |
| List of general values                             | Same as with list of scalar values, since `JsonElement` is used to represent both scalar and general values. |
| A JSON object (i.e. a dictionary) of scalar values | - A weakly derived type of `Dictionary<string, JsonElement>`<br />- A weakly derived type of `Dictionary<string, Entity>`, in the special case when the values of the dictionary are all representable by type `Entity` <br />- A POCC (a `record`, `class`, or `struct` with fields equal to the dictionary keys)<br />- A type (`record`, `class`, or `struct`) wrapping any of the above |
| Dictionary ("object") with general values          | Same as with dictionary of scalar values, since `JsonElement` is used to represent both scalar and general values. |

## Responses

Types that represent responses are handled by the middleware in the same way as types that represent requests, with only a couple of caveats:

- Response types are return types instead of input types
- It is preferable to return types of the form `ActionResult<T>` (or `Task<ActionResult<T>>` for `async` action methods), since common API server responses like `NotFound()` and `BadRequest()` are of type `ActionResult`
  - If `T` is not primitive, type erasure is used: the middleware converts `ActionResult<T>` to `ActionResult`

