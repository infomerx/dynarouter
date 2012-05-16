# Dynarouter

Dynamic router for Node.js that maps URLs to functions in objects on a per-request basis.

## Minimal Example

Let's suppose we want to route the URL '/simple' to some function. To do so we create the following javascript object *(however I'll provide examples in coffeescript for brevity)*:

```coffeescript
dynarouter = require 'dynarouter'

myLogicProvider =
  simple: (req, callback) ->
    callback null, "Hello World!"

http = require 'http'
server = http.createServer dynarouter.requestHandler myLogicProvider
server.listen 8080
```

That's it. Starting node and browsing to http://localhost:8080/simple will respond 
with the following JSON: "Hello World!"

Middleware is also provided so that the router may run in Connect/Express:

```coffeescript
express = require 'express'
app = express.createServer()
app.configure ->
  @use dynarouter.middleware myLogicProvider

app.configure 'development', () ->
  @use express.errorHandler 
    dumpExceptions:true
    showStack:true

app.listen 8081
```

## Nesting

We can build more complex objects.

```coffeescript
logicProvider =
  vehicles:
    planes: (req, callback) ->
      callback null, "This is not a plane."
    trains: (req, callback) ->
      callback null, "This is not a train."
    automobiles: (req, callback) ->
      callback null, "This just represents a car. It's not a real car."
```

Browsing to '/vehicles/planes', '/vehicles/trains' & '/vehicles/automobiles' will return 
their respective JSON responses. However, browsing to '/vehicles' will return a 404 response.

## HTTP Methods

To enable some sort of response at '/vehicles' we can add a GET property to the 'vehicles' 
property:

```coffeescript
logicProvider =
  vehicles:
    GET: (req, callback) ->
      callback null, "You may be expecting a vehicle list. Perhaps later..."
    planes: (req, callback) ->
      callback null, "This is not a plane."
    trains: (req, callback) ->
      callback null, "This is not a train."
    automobiles: (req, callback) ->
      callback null, "This just represents a car. It's not a real car."
```

There. Browsing to '/vehicles' is now enabled, through the GET function. You can similarly enable 
other HTTP methods by adding PUT, DELETE & POST properties wherever you want.

## URL parameters

We can also capture parts of the URL as parameters.

```coffeescript
logicProvider =
  users:
    $: name: 'id'
    id: (req, callback) ->
      callback null, "The user ID is: #{req.params.id}"
```

We can now browse to, say, '/users/1234' and we'll get JSON for "The user ID is: 1234".

```coffeescript
logicProvider =
  listing:
    $: name: 'category'
    category:
      $: name: 'page'
      page: (req, callback) ->
        callback null, "Listing for category #{req.params.category}, page #{req.params.page}."
```

In example above, we're capturing two parameters: 'category' and 'page' through the URL 
pattern '/listings/:category/:page'.

## Validation

URL parameters can be validated during routing, either with a regular expression, or by checking
a database, or by doing whatever you can do during a function call.

## Some Internals

Before going any further, some internals so you understand how the stuff above works.

When the router receives a path, it splits the path along '/' characters. It then digs into the 
logic provider looking for properties that match the URL. If it doesn't find one, it looks in the 
'$' property to see if a property is configured to capture the unmatched path section. If such is
the case, it stores the value in req.params. Simple, right?

The separator character '/' can be configured to be anything else, but you should use sensible
conventions. '.' and '-' comes to mind.

```coffeescript
app.configure () ->
  @use dynarouter.middleware '.', myLogicProvider     # use '.' as a path separator
```

We can build logic providers as complex as we want.

```coffeescript
logicProvider =
  articles:
    GET: (req, cb) ->
      cb null, 'A listing. But really, build your own, this is just an example.'
    $: name: 'id'
    id: 
      tag: 
        $: name: 'tag'
        GET: (req, cb) ->
          cb null, req.params
```

Here we're mapping URLs of the type '/articles/:id/tag/:tag' (I'm using express routing syntax here).

## Dynamic Routing

You might be thinking: Why should I use this cludgy thing if it's so much simpler to express 
this with express? (heh)

Well, for one simple reason. The logic providers, hence the available paths, can be assigned 
dynamically, on a per-request basis. And they can be assigned arbitrarily, according to whomever
is authenticated for that request, or by domain name, or a combination of both or whatever criteria.
It's up to you really, and your heart's content (or your bussiness requirements, whichever comes
first).

Assigning a logic provider per user allows for the most secure of security mechanisms. Each user can
only access what he should be able to access, because his logic provider (let's call them LPs from now on...)
only contains the functions he is authorized to access.

LPs, being plain Javascript objects, can be mixed and manipulated in the usual ways so it's easy to build
a library of named LPs that can be combined according to user assigned roles, allowing for a very simple yet
powerful mechanism to enable users to access different parts of a site's functionality.

## Views

LPs handle the logic (or control) part of a request, and the default behavior is to respond with JSON, which is
great for AJAX and APIs, but what about HTML?

That's where View Providers come in (VPs). Just as LPs can be assigned dynamically per request according
to arbitrary criteria, so can VPs. We can have different VPs per device type, or for AJAX, all using the 
same LPs, which only care about processing the request and not how they should respond (that's why they
don't get the response object handed to them).

VPs on the other hand are all about responses, and once the LP has done its thing and forwarded a result 
object, it's the VP's turn to respond, and usually it's already configured to do so using the appropriate
response for whatever client type is doing the request. This is all taken care of by Dynarouter.

In the end, what this means is that you can truly realize the concept of one URL per resource, for all device 
and client types. The resource is not the JSON, or the HTML, but the data contained within, and now it's all
in one place, irrespective of what is making the request.

 * More documentation coming...