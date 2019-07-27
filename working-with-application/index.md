---
title: Working with Application
description: Tutorial on how to work with Fano Framework application class.
---

## IWebApplication interface

In Fano Framework, application must implements `IWebApplication` interface.
Currently, it does not provide any method except inherit `run()` method from
`IRunnable` interface which is its parent.

[View IWebApplication source code](https://github.com/fanoframework/fano/blob/master/App/Contracts/AppIntf.pas).

## CGI Application

`TFanoWebApplication` and `TSimpleWebApplication` is two built-in classes that implements CGI protocol. Its task is basically to read CGI environment variables that web server send and call appropriate request handler.

Depending on how you construct the application service provider, it may pass the request to middleware layer that will reject or allow request to pass through to actual request handler.

### Not so simple application implementation.

`TFanoWebApplication` is built-in implementation of `IWebApplication` which does not assume anything. So developers must construct its dependencies by hand, which may be quite verbose.

`TFanoWebApplication` constructor requires developer to pass instance of
`IDependencyContainer`, `ICGIEnvironment` and `IErrorHandler` instance which serves
as [service container](/dependency-container), CGI environment variable container and [error handler](/error-handler) that handle any exception that may be triggered.

Fano Framework provides some built-in implementations for dependency container, environment variable and error handler. For example,
if `TBootstrapApp` inherits from `TFanoWebApplication`, you can use

```
appInstance := TBootstrapApp.create(
    TDependencyContainer.create(TDependencyList.create()),
    TCGIEnvironment.create(),
    TErrorHandler.create()
);
```

`TFanoWebApplication` is abstract class, so developers need to implements its three abstract methods:

- `buildDependencies` method will be called to allow developer to register any services required by application, for example router instance, dispatcher, application configuration, etc. This method has one parameter which is instance
of dependency container.

- `buildRoutes` method will be called to allow developer to register any routes that application need. This method has one parameter which is instance
of dependency container which developer can use to get router instance.

- `initDispatcher` method will be called and must return dispatcher instance to use. This method also pass dependency container which developer can use to get dispatcher instance.

```
(*!-----------------------------------------------
 * Build application dependencies
 *------------------------------------------------
 * @param container dependency container
 *-----------------------------------------------*)
procedure buildDependencies(const container : IDependencyContainer); virtual; abstract;

(*!-----------------------------------------------
 * Build application routes
 *------------------------------------------------
 * @param container dependency container
 *-----------------------------------------------*)
procedure buildRoutes(const container : IDependencyContainer); virtual; abstract;

(*!-----------------------------------------------
 * initialize application route dispatcher
 *------------------------------------------------
 * @param container dependency container
 * @return dispatcher instance
 *-----------------------------------------------*)
function initDispatcher(const container : IDependencyContainer) : IDispatcher; virtual; abstract;
```

[View TFanoWebApplication source code](https://github.com/fanoframework/fano/blob/master/App/Implementations/AppImpl.pas).

### Little bit simpler application

`TSimpleWebApplication` inherits from `TFanoWebApplication`. It is provided to simplify setting up application instance by providing some default service
providers.

If you use `TSimpleWebApplication`, then some required default services are provided for you if not set. If not set, following classes is assumed:

- `TDependencyContainer` instance as dependency container.
- `TCGIEnvironment` instance as CGI environment.
- `TErrorHandler` instance as CGI environment.
- Router instance which support regular expression.
- Simple dispatcher implementation which does not use middlewares.

Also, developers only need to implements two abstract methods as
`initDispatcher()` method has already been implemented.

[View TSimpleWebApplication source code](https://github.com/fanoframework/fano/blob/master/App/Implementations/AppImpl.pas).

See [sample application](https://github.com/fanoframework/fano-app) to understand how to setup application instance.

## FastCGI Application

To create web application that support FastCGI protocol, inherit from `TBaseSimpleFastCGIWebApplication` or one of its descendant

- `TSimpleFastCGIWebApplication`. This is class that you need to inherit if you want
to create FastCGI application which run independently and listen on TCP socket. See
[fano-fastcgi](https://github.com/fanoframework/fano-fastcgi) example demo application.

- `TSimpleUnixFastCGIWebApplication`. It is similar to class above but listen on unix domain socket file. See
[fano-fcgi-unix](https://github.com/fanoframework/fano-fcgi-unix) example demo application.

- `TSimpleSockFastCGIWebApplication`. This is class that you need to inherit if you want to create FastCGI application which run and managed by web server, for example
 Apache with mod_fcgid module. See
[fano-fcgid](https://github.com/fanoframework/fano-fcgid) example demo application.

See [Deploy as FastCGI application](/deployment/fastcgi)for information how to
deploy FastCGI application on various web servers.

You may want to see *Scaffolding FastCGI project directory structure* section in
[Scaffolding with Fano CLI](/scaffolding-with-fano-cli) for information how to scaffolding FastCGI web application with [Fano CLI](https://github.com/fanoframework/fano-cli).

## SCGI Application

To create web application that support [SCGI (Simple Common Gateway Interface)](https://python.ca/scgi/protocol.txt) protocol, inherit from `TBaseSimpleScgiWebApplication` or one of its descendant

- `TSimpleScgiWebApplication`. This is class that you need to inherit if you want
to create SCGI application which run independently and listen on TCP socket. See
[fano-scgi](https://github.com/fanoframework/fano-scgi) example demo application.

See [Deploy as SCGI application](/deployment/scgi)for information how to
deploy SCGI application on various web servers.

You may want to see *Scaffolding SCGI project directory structure* section in
[Scaffolding with Fano CLI](/scaffolding-with-fano-cli) for information how to scaffolding SCGI web application easily.

## Apache modules Application

Implementation of IWebApplication that run as Apache modules is not yet implemented.

## Standalone web server application

Implementation of IWebApplication that run as standalone web server is not yet implemented.

## Application implementation which support Rack-like protocol

Implementation of IWebApplication that implements Rack-like protocol, currently, is still under development.

- [Ruby Rack](https://rack.github.io/)
- [Prack](https://github.com/piradoiv/Prack)