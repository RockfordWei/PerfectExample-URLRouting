# URL Routing
This example illustrates how to set up URL routing to direct requests to your custom handlers.

To use the example with Swift Package Manager, type ```swift build``` and then run ``` .build/debug/URLRouting```.

To use the example with Xcode, run the **URL Routing** target. This will launch the Perfect HTTP Server. 

Navigate in your web browser to [http://localhost:8181/](http://localhost:8181/). Experiment with the URL routes wchich are added.

## Enabling URL Routing

The following code is taken from the example project and shows how to enable the system and add routes.

```swift
func addURLRoutes() {
    
    Routing.Routes[.get, ["/", "index.html"] ] = indexHandler
    Routing.Routes["/foo/*/baz"] = echoHandler
    Routing.Routes["/foo/bar/baz"] = echoHandler
    Routing.Routes[.get, "/user/{id}/baz"] = echo2Handler
    Routing.Routes[.get, "/user/{id}"] = echo2Handler
    Routing.Routes[.post, "/user/{id}/baz"] = echo3Handler
    
    // Test this one via command line with curl:
    // curl --data "{\"id\":123}" http://0.0.0.0:8181/raw --header "Content-Type:application/json"
    Routing.Routes[.post, "/raw"] = rawPOSTHandler
    
    // Trailing wildcard matches any path
    Routing.Routes["**"] = echo4Handler
    
    // Check the console to see the logical structure of what was installed.
    print("\(Routing.Routes.description)")
}
```
## Handling Requests

The example `EchoHandler` consists of the following.

```swift
func echoHandler(request: WebRequest, _ response: WebResponse) {
	response.appendBody(string: "Echo handler: You accessed path \(request.requestURI!) with variables \(request.urlVariables)")
	response.requestCompleted()
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
