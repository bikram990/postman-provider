# Vapor Postman Provider

[![Swift](http://img.shields.io/badge/swift-4.2-brightgreen.svg)](https://swift.org)
[![Vapor](http://img.shields.io/badge/vapor-3.0-brightgreen.svg)](https://vapor.codes)
[![CircleCI](https://circleci.com/gh/vapor-community/postman-provider.svg?style=shield)](https://circleci.com/gh/vapor-community/postman-provider)
[![MIT License](http://img.shields.io/badge/license-MIT-brightgreen.svg)](LICENSE)

[Postman](https://www.getpostman.com/docs/v6/postman/postman_api/intro_api) is a developer tool for making network requests and testing APIs.

Included in `Postman` is a `PostmanClient` which you can use to get and update your Postman [environment](https://www.getpostman.com/docs/v6/postman/environments_and_globals/manage_environments), especially useful for updating environment variables. 

## Getting Started

In your `Package.swift` file, add the following:

```swift
.package(url: "https://github.com/vapor-community/postman-provider.git", from: "1.0.0")
```

Register the configuration and the provider.

```swift
let config = PostmanConfig(apiKey: "your-api-key", environmentUID: "your-environment-uid")

services.register(config)

try services.register(PostmanProvider())

app = try Application(services: services)

postmanClient = try app.make(PostmanClient.self)
```
*Note: `environmentUID` is your environment's `"uid"` and **not** your environment's `"id"`.*

## Using the API

`Postman`'s current functionality revolves around environments. `Postman`'s environment model is `PostmanEnvironment`:

```swift
public struct PostmanEnvironment: Content {
    public var name: String
    public var values: [String: String]
}
```

Use the `PostmanClient` to get your environment:

```swift
postmanClient.getEnvironment().map { environment in
    environment.name
    environment.values
}
```

You can also update your environment:

```swift
let updatedEnvironment = PostmanEnvironment(
    name: "Updated Name",
    values: ["token": updatedToken, "testUserID": newTestUserID]
)

postmanClient.update(updatedEnvironment) // Future<Void>
```

It is importnat to note that when you update your environment, all of the environment variables will be replaced by the values you provide. So if an existing environment variable is not included in `values` it will be deleted.

## Error Handling

If requests to the API fail, a `PostmanError` is thrown which includes a `name` and `message` to help you understand what went wrong. 

```swift
postmanClient.update(updatedEnvironment).catch { error in
    if let postmanError = error as? PostmanError {
        postmanError.name
        postmanError.message
    }
}
```
