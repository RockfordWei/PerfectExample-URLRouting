# URL Routing
This example illustrates how to set up URL routing to direct requests to your custom handlers.

To use the example with Swift Package Manager, type ```swift build``` and then run ``` .build/debug/URLRouting```.

To use the example with Xcode, run the **URL Routing** target. This will launch the Perfect HTTP Server. 

Navigate in your web browser to [http://localhost:8181/](http://localhost:8181/). Experiment with the URL routes which are added.

## Enabling URL Routing

The following code is taken from the example project and shows how to create and add routes to a server.

```swift
var routes = Routes()

routes.add(method: .get, uris: ["/", "index.html"], handler: indexHandler)
routes.add(method: .get, uri: "/foo/*/baz", handler: echoHandler)
routes.add(method: .get, uri: "/foo/bar/baz", handler: echoHandler)
routes.add(method: .get, uri: "/user/{id}/baz", handler: echo2Handler)
routes.add(method: .get, uri: "/user/{id}", handler: echo2Handler)
routes.add(method: .post, uri: "/user/{id}/baz", handler: echo3Handler)

// Test this one via command line with curl:
// curl --data "{\"id\":123}" http://0.0.0.0:8181/raw --header "Content-Type:application/json"
routes.add(method: .post, uri: "/raw", handler: rawPOSTHandler)

// Trailing wildcard matches any path
routes.add(method: .get, uri: "**", handler: echo4Handler)

// Routes with a base URI

// Create routes for version 1 API
var api = Routes()
api.add(method: .get, uri: "/call1", handler: { _, response in
	response.setBody(string: "API CALL 1")
	response.completed()
})
api.add(method: .get, uri: "/call2", handler: { _, response in
	response.setBody(string: "API CALL 2")
	response.completed()
})

// API version 1
var api1Routes = Routes(baseUri: "/v1")
// API version 2
var api2Routes = Routes(baseUri: "/v2")

// Add the main API calls to version 1
api1Routes.add(routes: api)
// Add the main API calls to version 2
api2Routes.add(routes: api)
// Update the call2 API
api2Routes.add(method: .get, uri: "/call2", handler: { _, response in
	response.setBody(string: "API v2 CALL 2")
	response.completed()
})

// Add both versions to the main server routes
routes.add(routes: api1Routes)
routes.add(routes: api2Routes)

// Check the console to see the logical structure of what was installed.
print("\(routes.navigator.description)")

// Create server object.
let server = HTTPServer()

// Listen on port 8181.
server.serverPort = 8181

// Add our routes.
server.addRoutes(routes)

do {
    // Launch the HTTP server
    try server.start()
} catch PerfectError.networkError(let err, let msg) {
    print("Network error thrown: \(err) \(msg)")
}
```
## Handling Requests

The example `EchoHandler` consists of the following.

```swift
func echoHandler(request: HTTPRequest, _ response: HTTPResponse) {
	response.appendBody(string: "Echo handler: You accessed path \(request.path) with variables \(request.urlVariables)")
	response.completed()
}
```

## Using Apache
The following Apache conf snippet can be used to pipe requests for non-existent files through to Perfect when using the URL routing system.

```apacheconf
	RewriteEngine on
	RewriteCond %{REQUEST_FILENAME} !-f
	RewriteCond %{REQUEST_FILENAME} !-d
	RewriteRule (.*) - [L,NS,H=perfect-handler]
```

## Repository Layout

We have finished refactoring Perfect to support Swift Package Manager. The Perfect project has been split up into the following repositories:

* [Perfect](https://github.com/PerfectlySoft/Perfect) - This repository contains the core PerfectLib and will continue to be the main landing point for the project.
* [PerfectTemplate](https://github.com/PerfectlySoft/PerfectTemplate) - A simple starter project which compiles with SPM into a stand-alone executable HTTP server. This repository is ideal for starting on your own Perfect based project.
* [PerfectDocs](https://github.com/PerfectlySoft/PerfectDocs) - Contains all API reference related material.
* [PerfectExamples](https://github.com/PerfectlySoft/PerfectExamples) - All the Perfect example projects and documentation.
* [PerfectEverything](https://github.com/PerfectlySoft/PerfectEverything) - This umbrella repository allows one to pull in all the related Perfect modules in one go, including the servers, examples, database connectors and documentation. This is a great place to start for people wishing to get up to speed with Perfect.
* [PerfectServer](https://github.com/PerfectlySoft/PerfectServer) - Contains the PerfectServer variants, including the stand-alone HTTP and FastCGI servers. Those wishing to do a manual deployment should clone and build from this repository.
* [Perfect-Redis](https://github.com/PerfectlySoft/Perfect-Redis) - Redis database connector.
* [Perfect-SQLite](https://github.com/PerfectlySoft/Perfect-SQLite) - SQLite3 database connector.
* [Perfect-PostgreSQL](https://github.com/PerfectlySoft/Perfect-PostgreSQL) - PostgreSQL database connector.
* [Perfect-MySQL](https://github.com/PerfectlySoft/Perfect-MySQL) - MySQL database connector.
* [Perfect-MongoDB](https://github.com/PerfectlySoft/Perfect-MongoDB) - MongoDB database connector.
* [Perfect-FastCGI-Apache2.4](https://github.com/PerfectlySoft/Perfect-FastCGI-Apache2.4) - Apache 2.4 FastCGI module; required for the Perfect FastCGI server variant.

The database connectors are all stand-alone and can be used outside of the Perfect framework and server.

## Further Information
For more information on the Perfect project, please visit [perfect.org](http://perfect.org).
