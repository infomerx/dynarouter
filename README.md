# Dynarouter

Dynamic router for Node.js that maps URLs to functions in objects on a per-request basis.

## Minimal Example

Let's suppose we want to route the URL '/simple' to some function. To do so we create the following javascript object *(however I'll provide examples in coffeescript for brevity)*:

    http = require 'http'
    dynarouter = require 'dynarouter'
    
    myLogicProvider =
      simple: (req, callback) ->
        callback null, "Hello World!"

    server = http.createServer (req, res) ->
    server = http.createServer dynarouter.simpleHandler '/', myLogicProvider
    server.listen 8080

That's it. Starting node and browsing to http://localhost:8080/simple will rspond with the following JSON: "Hello World!"