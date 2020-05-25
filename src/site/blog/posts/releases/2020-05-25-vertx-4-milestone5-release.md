---
title: Eclipse Vert.x 4 milestone 5 released!
date: 2020-05-25
template: post.html
author: vietj
draft: true
---

We are extremely pleased to announce the fifth 4.0 milestone release of Eclipse Vert.x .

Vert.x 4 is the evolution of the Vert.x 3.x series that will bring key features to Vert.x.

This release aims to provide a reliable distribution of the current development of Vert.x 4 for people that
want to try it and provide feedback.

#### The new trio: Json Schema, Web Validation and Web OpenAPI

After the lessons we learnt in Vert.x 3.x, we decided to renew completely our support for
HTTP request validation in `vertx-web` and Contract Driven development using [OpenAPI](https://github.com/OAI/OpenAPI-Specification).

In this milestone, we introduce 3 new modules:

* [Vert.x Json Schema](https://github.com/eclipse-vertx/vertx-json-schema): Extensible sync/async json-schema validator designed for Vert.x JSON types, supporting Json Schema Draft-7 and OpenAPI dialect.
* [Vert.x Web Validation](https://github.com/vert-x3/vertx-web/tree/master/vertx-web-validation): Extensible sync/async HTTP request validator, providing a DSL to describe expected HTTP requests.
* [Vert.x Web OpenAPI](https://github.com/vert-x3/vertx-web/tree/master/vertx-web-openapi): New support for Contract Driven development based on OpenAPI

Vert.x Json Schema is a powerful [Json Schema](https://json-schema.org/) validator which includes:

* Async `$ref` resolution
* Custom keywords/Custom dialects support
* DSL to build schemas directly in the code

Now combining Vert.x Json Schema and Vert.x Web Validation you can easily validate HTTP requests:

```java
ValidationHandler handler = ValidationHandler
  .builder(schemaParser)
  .pathParameter(Parameters.param("myPathParam", stringSchema()))
  .queryParameter(Parameters.optionalParam("myQueryParam", intSchema()))
  .body(Bodies.json(Schemas.ref(JsonPointer.fromURI(URI.create(
    "some_schema.json"
  )))))
  .build();

router.post("/{myPathParam}").handler(handler);
```

Check out the [Web validation examples](https://github.com/vert-x3/vertx-web/blob/master/vertx-web-validation/src/main/java/examples/WebValidationExamples.java) and provide feedback on the APIs!

Thanks to Vert.x Web Validation, now you can use Vert.x Web API Service without using Vert.x Web API Contract:
[Vert.x Web API Service example](https://github.com/vert-x3/vertx-web/tree/master/vertx-web-api-service/src/main/java/examples)

If you want to do Contract Driven development, like you used to do in Vert.x 3, you can use Vert.x Web OpenAPI:

```java
RouterFactory.create(vertx, "src/main/resources/petstore.yaml",
  routerFactoryAsyncResult -> {
  if (routerFactoryAsyncResult.succeeded()) {
    // Spec loaded with success, retrieve the router
    RouterFactory routerFactory = routerFactoryAsyncResult.result();
    // You can enable or disable different features of router factory using RouterFactoryOptions
    RouterFactoryOptions options = new RouterFactoryOptions();
    // Set the options
    routerFactory.setOptions(options);
    // Add an handler to operation listPets
    routerFactory.operation("listPets").handler(routingContext -> {
      // Handle listPets operation
      routingContext.response().setStatusMessage("Called listPets").end();
    }).failureHandler(routingContext -> { // Add a failure handler to the same operation
      // This is the failure handler
      Throwable failure = routingContext.failure();
      if (failure instanceof BadRequestException)
        // Handle Validation Exception
        routingContext
          .response()
          .setStatusCode(400)
          .putHeader("content-type", "application/json")
          .end(((BadRequestException)failure).toJson().toBuffer());
    });

    // Add a security handler
    // Handle security here
    routerFactory.securityHandler(
      "api_key",
      JWTAuthHandler.create(jwtAuth)
    );

    // Now you have to generate the router
    Router router = routerFactory.createRouter();
  } else {
    // Something went wrong during router factory initialization
  }
});
```

Vert.x Web OpenAPI is going to replace Vert.x Web API Contract but, to ease the migration, we'll continue to support
Vert.x Web API Contract for the whole lifetime of Vert.x 4.
We'll provide soon a migration guide for replacing Vert.x Web API Contract.

Check out the [Web OpenAPI examples](https://github.com/vert-x3/vertx-web/blob/master/vertx-web-openapi/src/main/java/examples/OpenAPI3Examples.java)

#### Ramping up to Vert.x 4

Instead of developing all new features exclusively in Vert.x 4, we introduce some of these features in the 3.x branch
so the community can benefit from them. The Vert.x 4 development focus on more fundamental changes that cannot be done
in the 3.x series.

<img src="{{ site_url }}assets/blog/vertx-4-milestone4-release/vertx-4-timeline.png" alt="Screenshot" class="img-responsive">

This is the fifth milestone of Vert.x 4, you can of course expect more milestones to outline the progress of the effort.

You can also read the previous milestone announces:

- https://vertx.io/blog/eclipse-vert-x-4-milestone-4-released
- https://vertx.io/blog/eclipse-vert-x-4-milestone-3-released
- https://vertx.io/blog/eclipse-vert-x-4-milestone-2-released
- https://vertx.io/blog/eclipse-vert-x-4-milestone-1-released

#### Finally

The [deprecations and breaking changes](https://github.com/vert-x3/wiki/wiki/4.0.0-Deprecations-and-breaking-changes)
 can be found on the wiki.

The release artifacts have been deployed to [Maven Central](https://search.maven.org/search?q=g:io.vertx%20AND%20v:4.0.0-milestone5) and you can get the distribution on [Maven Central](https://repo1.maven.org/maven2/io/vertx/vertx-stack-manager/4.0.0-milestone5/).

You can bootstrap a Vert.x 4.0.0-milestone5 project using https://start.vertx.io.

The documentation has been deployed on this preview web-site https://vertx-ci.github.io/vertx-4-preview/docs/

That's it! Happy coding and see you soon on our user or dev [channels](https://vertx.io/community).