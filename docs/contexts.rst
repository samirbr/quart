.. _contexts:

Contexts
========

Quart, like Flask, has two contexts the *application context* and the
*request context*. Both of these contexts exist per request and allow
the global proxies ``current_app``, ``request`` etc... to be resolved.
Note that these contexts are task local, and hence will not exist if a
task is spawned by ``ensure_future`` or ``create_task``.

The design principle of these contexts is that they are likely needed
in all routes, and hence rather than pass these objects around they
are made available via global proxies. This has its downsides, notably
all the arguments relating to global variables. Therefore It is
recommended that these proxies are only used within routes so as to
isolate the scope.

Application Context
-------------------

The application context is a reference point for any information that
isn't specifically related to a request. This includes the app itself,
the g global object and a url_adapter bound only to the app. The
context is created and destroyed implicitly by the request context.

Request Context
---------------

The request context is a reference point for any information that is
related to a request. This includes the request itself, a url_adapter
bound to the request and the session. It is created and destroyed by
the :func:`~quart.Quart.handle_request` method per request.

Websocket Context
-----------------

The websocket context is analogous to the request context, but is
related only to websocket requests. It is created and destroyed by the
:func:`~quart.Quart.handle_websocket_request` method per websocket
connection.

Tasks and contexts
------------------

As a context is bound to a specific task trying to access the context
within another task will fail. This means that the following will
fail,

.. code-block:: python

    async def background_task():
        method = request.method  # Will raise an exception
        ...

    @app.route('/')
    async def index():
        asyncio.ensure_future(background_task())
        ...

as the ``background_task`` coroutine will run in a different asyncio
task than the index coroutine. To work around this Quart provides the
decorators :func:`~quart.ctx.copy_current_request_context` and
:func:`copy_current_websocket_context` which can be used as so,

.. code-block:: python

    @copy_current_request_context
    async def background_task():
        method = request.method  # Will raise an exception
        ...

    @app.route('/')
    async def index():
        asyncio.ensure_future(background_task())
        ...
