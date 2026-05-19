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

## Controllers and action methods

In ASP.NET Core Web API, groupings of related API endpoints corresponds to a *controller class*- a class inheriting from `ControllerBase`- that defines how HTTP requests to those API endpoints are to be handled. Make sure not to confuse `ControllerBase` with the closely-related `Controller` class, which adds view support and is most commonly in ASP.NET Core Web App.

Specifically, handlers for the different verbs (GET, PUT, POST, DELETE, PATCH) that can be issued to that API endpoint are defined as methods of the controller class. Such methods are called *action methods*.

We describe how a HTTP request sent to an API endpoint is matched to a controller action. There are two steps to this process:

1. Determining the controller corresponding to the API endpoint
2. Determining the action method corresponding to the HTTP verb

### Conventional routing

In the olden days, before the `[Route]` attribute existed, the following *global* configuration code associating URLs with controller actions was placed in `Program.cs`, the entry-point file:

```c#
app.UseEndpoints(endpoints =>
{
    endpoints.MapControllerRoute(
        name: "default",
        pattern: "{controller}/{action}/{id?}");
});
```

This code causes a URL such as "/resource/info" to be handled by the action method named `Info` in the controller called `ResourceController`:

```c#
public ActionResult Info()
{
    return Ok("Info action called"); // Ok is of type ActionResult
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

### Attribute-based routing

#### 1. Determining the controller

Modern code uses C#'s decorator syntax- attributes- to decentralize the configuration code. The association of URLs with controllers no longer happens all in one place; instead, it is done on a per-controller, by annotating the controller with the `[Route]` attribute:

```c#
[Route("[controller]")]
public class ResourceController : ControllerBase { ... }
```

The `"[controller]"`  argument to the `Route` attribute indicates that the case-insensitive name of the decorated controller, minus "Controller" and prepended with a "/", is the URL to be associated with `ResourceController`. That is, since `[Route("[controller]")]` decorates `ResourceController`, then "/resource" is the URL handled by `ResourceController`.

#### 2. Determining the action method

In the old approach, determination of the action method relies a lot upon enforcing naming conventions on method names. That is much less so the case in the attribute-based approach. Similarly to as with the controller, the action is annotated with the HTTP verb to be matched. 

```c#
[HttpGet("info")]
public ActionResult Get()
{
    return Ok("Info action called");
}
```

A HTTP request is matched to an action with the following algorithm:

1. Filter all actions so that the actions under consideration are the ones with routes (formally called "route templates") that match the request's URL path.
2. If there exists a matching action with whose HTTP verb attribute (e.g. `[HttpGet]`) matches that of the request, use one of those actions.
3. Of those actions, filter further, so that only the actions having inputs compatible with the inputs sent in the HTTP request are considered.
4. Choose one of these actions[^1].

[^1]: I'm intentionally not specifying how to uniquely specify one of "those actions" because things should never be this ambiguous in best practice.

## Requests

As one would expect, the kinds of the inputs can be passed in a HTTP request that's sent to an API endpoint correspond to the the types of the inputs in the corresponding controller action method. Here we describe exactly how to define those types.

### Representing dictionary inputs

There are two ways for an API caller to pass a dictionary in an API request. They can represent the dictionary by sending...

* a HTTP request with a JSON payload representing the dictionary[^2]
* a HTTP request with a `urlencoded` payload representing the dictionary[^3]

[^2]: A HTTP request with a `Content-Type` header of `application/json` and a request body containing a JSON-formatted string that represents the dictionary.
[^3]: A HTTP request with a `Content-Type` header of `application/x-www-form-urlencoded` and a request body containing the result of specifying the key-value pairs from the dictionary in a string that looks like `"key1=value1&key2=value2&key3=value3"`.

Fortunately, the middleware is sophisticated enough that we don't even need tell it which of the two kinds of dictionaries to expect. Whether the dictionary is passed as JSON payload or in `urlencoded` format, it will deserialize it into an instance of a type, so long as we define that type correctly.  

For an example, suppose that the POST /api/resource endpoint receives an HTTP request of `Content-Type` = `application/json`, which contains the following JSON payload:

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

To properly receive the payload as an input, we need to define a type that mimics its schema, like so[^4]:

[^4]: One strange "gotcha" is that you have to define properties (like `public int a { get; set; }`) instead of fields (like `public int a;`) if you want to have things work as they should.

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

(Notice that *nesting* in the JSON schema corresponds to *composition* of complex types in the type representation of the schema.)

Now that we can represent the JSON payload input, we can define a  `[HttpGet]` action method that receives it:

```c#
[HttpPost("resource")]
public async Task<ActionResult> Post(OuterType input)
{
    return await service.Create(input);
}
```

The middleware will take care of parsing the JSON payload provided by an API caller, so that, at the beginning of `Post()`, action method, the properties of that `OuterType` variable are populated as you would expect. 

### Representing scalar inputs

Scalar inputs (strings, numbers, or booleans) can only ever be specified as route parameters or query parameters.

To represent a scalar input, simply use a primitive type like `string`, `int`, `double`, or `bool`. An example of an action method corresponding to a POST endpoint that accepts a `string` input is

```c#
[HttpPost("info")]
public async Task<ActionResult> Post(string input)
{
    return await service.Create(input);
}
```

(Don't worry about the return type too much for now. Just know that  `ActionResult` is the typical return type for action methods that return no output other than a success message or error message.)

### General typing

While it is most common- and probably best practice- to represent scalar and JSON inputs in the standard ways mentioned above, there are some more general ways to do so worth mentioning. 

The *most* general way to represent a JSON input is to use `Dictionary<string, System.Text.JsonElement>`.

Since scalars can be interpreted as constituting valid JSON payloads, they can also be represented in a more general way, as a `JsonElement`. 

Here are the full details:

| Kind of input or output to API endpoint            | C# types that can represent it                               |
| -------------------------------------------------- | ------------------------------------------------------------ |
| Raw scalar value                                   | - `JsonElement` <br />- A primitive type (`int`, `float`, `string`, `bool`, etc.) |
| JSON list of scalar values                         | - A weakly derived type of `ICollection<JsonElement>` <br />- A weakly derived type of `ICollection<Entity>`, in the special case when the elements of the array are all representable by type `Entity`<br />- A type (`record`, `class`, or `struct`) wrapping any of the above |
| List of general values                             | Same as with list of scalar values, since `JsonElement` is used to represent both scalar and general values. |
| A JSON object (i.e. a dictionary) of scalar values | - A weakly derived type of `Dictionary<string, JsonElement>`<br />- A weakly derived type of `Dictionary<string, Entity>`, in the special case when the values of the dictionary are all representable by type `Entity` <br />- A POCC (a `record`, `class`, or `struct` with fields equal to the dictionary keys)<br />- A type (`record`, `class`, or `struct`) wrapping any of the above |
| Dictionary ("object") with general values          | Same as with dictionary of scalar values, since `JsonElement` is used to represent both scalar and general values. |

## Responses

Similarly to as was the case with requests, the kind of the output that will be be passed in the HTTP response sent by an API endpoint corresponds to the type of the output of the corresponding controller action method.

Types that represent outputs of responses are essentially[^5] handled by the middleware in the same way as types that represent inputs for requests, with only one main caveat: while it is *possible* to simply return the type `T` representing the output (or `Task<T>` if the method is `async`), it is *preferable* to return `ActionResult<T>` (or `Task<ActionResult<T>>`), since common API server responses like `NotFound()` and `BadRequest()` are of type `ActionResult`.

[^5]: Only essentially. Inputs to requests are handled by the "model binding" step of the middleware, while outputs of responses are handled by the "output formatting" step of the middleware.

## `[ApiController]`

Controllers in ASP.NET Core Web API (classes that inherit from `ControllerBase`) have basic functionality for building API controllers, like automatic model binding.

When a controller is additionally annotated with `[ApiController]`, its methods now *must* be annotated with a HTTP verb attribute (like `[HttpGet]` or `[HttpPost]`).

