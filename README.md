# Swim

Swim is the Web for real-time apps.  Like the Web, Swim consists of a simple
network protocol, and a model for globally identifying and linking resources.
On top of these principles, Swim builds a bidirectional, low latency push
network, with always active real-time links.

Swim.it provides cloud native server frameworks and device native client SDKs
that make it easy to build, deploy, and scale real-time Swim Apps.

## Diving In

First we need to introduce three core concepts: services, lanes, and links.
To explain, we'll continue the analogy with the World Wide Web.

On the Web, URIs represent passive resources.  With Swim, URIs identify
reactive _services_.  Unlike a Web service, which can only be periodically
accessed, a Swim service can be actively linked to.  Swim is engineered to
run millions of services, distributed across massive clusters of machines.

Web resources are operated upon with HTTP methods.  Swim services are linked
to and controlled over Swim _lanes_.  A lane is a method with an event source.
Swim lanes can be specialized with different behaviors and messaging semantics.
Some lanes automatically handle data persistence, while others implement
advanced network flow control.

A Web link forms a declarative connection from one hypertext page to another.
A Swim _link_ creates a ongoing subscription to events on a lane of a
particular Swim service.  Swim efficiently multiplexes and prioritizes huge
numbers of links across a small number of network connections.

## First Swim

Let's get right to brass tacks and build an app.  A chat app.  First, we need
a cloud service to manage chat rooms.  The service will need to maintain
persistent lists of chat messages, and broadcast new messages as they're
posted.  In short, we need an event source, and a method that publishes
messages to that event source; in Swim parlance, we call such a thing a lane.
Let's create a service with a `ListLane` to manage our chat rooms.  We'll use
JavaScript for this example; put the code below in a file called `chat.js`:

```js
var service = require('swim-service-js');

var room = service.ListLane('chat/room');
```

We start off by requiring the `swim-service-js` module, which defines the API
for managing a single Swim service instance.  This code will be instantiated
many times, once for every chat room we create.  Each chat service instance
will have its own `ListLane`, named `chat/room`.

Next we need a way to instantiate chat services.  The `swimjs` container lets
us configure services declaratively.  Create a file called `swim.recon`, in
the same directory as `chat.js`, and fill it with the following:

```recon
@service { name: "chat"; main: "chat.js" }
@server {
  port: 5015
  store: "/tmp/swim.store"
  @route { prefix: "/chat/"; service: "chat" }
}
```

The `swim.recon` file defines how we want to deploy our services.  First we
declare our `chat` service, specifying the path to its `main` script.  Then
we define a `@server` container to run the services.  The server will listen
on port 5015, and store data in a local datastore.

Finally, we specify a `@route` to our `chat` services.  The simple route here
states that all URIs paths that begin with `/chat/` should map to `chat`
service instances.  In other words, every unique URI prefixed with `/chat/`
refers to a unique `chat` service instance.  This route scheme automatically
instantiates services as needed.

Swim.it provides containers that support transparent distribution and scaling
of services across clusters, high availability, and much more.  But we'll stick
to using simple containers for this tutorial.  Importantly, your code doesn't
need to change when transitioning from small containers to large clusters.

To start a container, with our chat services inside, run `swimjs` on the
command line from the same directory as the `swim.recon` file.

```bash
$ swimjs
Listening on port 5015...
```

Using our chat service from client apps is just as easy.  It takes one line of
code using the [Swim JavaScript Client SDK](https://github.com/swimit/swim-client-js):

```js
var swim = require('swim-client-js');
var room = swim.syncList('ws://localhost:5015/chat/public', 'chat/room');
```

The code above automatically synchronizes the list of chat messages with the
`/chat/public` chat service instance.  You can treat the `room` variable like
an ordinary JavaScript sequence.  So posting a chat message also takes one
line of code:

```js
room.push({body: 'Hello, world!'});
```

This will push updates in real-time to anyone who's linked to the `chat/room`
lane of the `/chat/public` service.

Using the [Swim Swift SDK](https://github.com/swimit/swim-swift) on Apple's
iOS, tvOS, watchOS, or OSX, works similarly.

```swift
import Swim
let swim = SwimClient()
var room = swim.syncList(node "ws://localhost:5015/chat/public", lane "chat/room")
room.append(...)
```

The `room` variable again behaves like an ordinary Swift collection, despite
being synchronized with the cloud in real-time.
