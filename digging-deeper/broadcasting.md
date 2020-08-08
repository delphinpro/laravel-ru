# Broadcasting

### [Introduction](https://laravel.com/docs/7.x/broadcasting#introduction) <a id="introduction"></a>

In many modern web applications, WebSockets are used to implement realtime, live-updating user interfaces. When some data is updated on the server, a message is typically sent over a WebSocket connection to be handled by the client. This provides a more robust, efficient alternative to continually polling your application for changes.

To assist you in building these types of applications, Laravel makes it easy to "broadcast" your [events](https://laravel.com/docs/7.x/events) over a WebSocket connection. Broadcasting your Laravel events allows you to share the same event names between your server-side code and your client-side JavaScript application.

> ![](https://laravel.com/img/callouts/lightbulb.min.svg)
>
> Before diving into event broadcasting, make sure you have read all of the documentation regarding Laravel [events and listeners](https://laravel.com/docs/7.x/events).

#### [Configuration](https://laravel.com/docs/7.x/broadcasting#configuration) <a id="configuration"></a>

All of your application's event broadcasting configuration is stored in the `config/broadcasting.php` configuration file. Laravel supports several broadcast drivers out of the box: [Pusher Channels](https://pusher.com/channels), [Redis](https://laravel.com/docs/7.x/redis), and a `log` driver for local development and debugging. Additionally, a `null` driver is included which allows you to totally disable broadcasting. A configuration example is included for each of these drivers in the `config/broadcasting.php` configuration file.

**Broadcast Service Provider**

Before broadcasting any events, you will first need to register the `App\Providers\BroadcastServiceProvider`. In fresh Laravel applications, you only need to uncomment this provider in the `providers` array of your `config/app.php` configuration file. This provider will allow you to register the broadcast authorization routes and callbacks.

**CSRF Token**

[Laravel Echo](https://laravel.com/docs/7.x/broadcasting#installing-laravel-echo) will need access to the current session's CSRF token. You should verify that your application's `head` HTML element defines a `meta` tag containing the CSRF token:

```text
<meta name="csrf-token" content="{{ csrf_token() }}">
```

#### [Driver Prerequisites](https://laravel.com/docs/7.x/broadcasting#driver-prerequisites) <a id="driver-prerequisites"></a>

**Pusher Channels**

If you are broadcasting your events over [Pusher Channels](https://pusher.com/channels), you should install the Pusher Channels PHP SDK using the Composer package manager:

```text
composer require pusher/pusher-php-server "~4.0"
```

Next, you should configure your Channels credentials in the `config/broadcasting.php` configuration file. An example Channels configuration is already included in this file, allowing you to quickly specify your Channels key, secret, and application ID. The `config/broadcasting.php` file's `pusher` configuration also allows you to specify additional `options` that are supported by Channels, such as the cluster:

```text
'options' => [
    'cluster' => 'eu',
    'useTLS' => true
],
```

When using Channels and [Laravel Echo](https://laravel.com/docs/7.x/broadcasting#installing-laravel-echo), you should specify `pusher` as your desired broadcaster when instantiating the Echo instance in your `resources/js/bootstrap.js` file:

```text
import Echo from "laravel-echo";

window.Pusher = require('pusher-js');

window.Echo = new Echo({
    broadcaster: 'pusher',
    key: 'your-pusher-channels-key'
});
```

Finally, you will need to change your broadcast driver to `pusher` in your `.env` file:

```text
BROADCAST_DRIVER=pusher
```

**Pusher Compatible Laravel Websockets**

The [laravel-websockets](https://github.com/beyondcode/laravel-websockets) is a pure PHP, Pusher compatible websocket package for Laravel. This package allows you to leverage the full power of Laravel broadcasting without an external websocket provider or Node. For more information on installing and using this package, please consult its [official documentation](https://beyondco.de/docs/laravel-websockets).

**Redis**

If you are using the Redis broadcaster, you should either install the phpredis PHP extension via PECL or install the Predis library via Composer:

```text
composer require predis/predis
```

Next, you should update your broadcast driver to `redis` in your `.env` file:

```text
BROADCAST_DRIVER=redis
```

The Redis broadcaster will broadcast messages using Redis' pub / sub feature; however, you will need to pair this with a WebSocket server that can receive the messages from Redis and broadcast them to your WebSocket channels.

When the Redis broadcaster publishes an event, it will be published on the event's specified channel names and the payload will be a JSON encoded string containing the event name, a `data` payload, and the user that generated the event's socket ID \(if applicable\).

**Socket.IO**

If you are going to pair the Redis broadcaster with a Socket.IO server, you will need to include the Socket.IO JavaScript client library in your application. You may install it via the NPM package manager:

```text
npm install --save socket.io-client
```

Next, you will need to instantiate Echo with the `socket.io` connector and a `host`.

```text
import Echo from "laravel-echo"

window.io = require('socket.io-client');

window.Echo = new Echo({
    broadcaster: 'socket.io',
    host: window.location.hostname + ':6001'
});
```

Finally, you will need to run a compatible Socket.IO server. Laravel does not include a Socket.IO server implementation; however, a community driven Socket.IO server is currently maintained at the [tlaverdure/laravel-echo-server](https://github.com/tlaverdure/laravel-echo-server) GitHub repository.

**Queue Prerequisites**

Before broadcasting events, you will also need to configure and run a [queue listener](https://laravel.com/docs/7.x/queues). All event broadcasting is done via queued jobs so that the response time of your application is not seriously affected.

### [Concept Overview](https://laravel.com/docs/7.x/broadcasting#concept-overview) <a id="concept-overview"></a>

Laravel's event broadcasting allows you to broadcast your server-side Laravel events to your client-side JavaScript application using a driver-based approach to WebSockets. Currently, Laravel ships with [Pusher Channels](https://pusher.com/channels) and Redis drivers. The events may be easily consumed on the client-side using the [Laravel Echo](https://laravel.com/docs/7.x/broadcasting#installing-laravel-echo) Javascript package.

Events are broadcast over "channels", which may be specified as public or private. Any visitor to your application may subscribe to a public channel without any authentication or authorization; however, in order to subscribe to a private channel, a user must be authenticated and authorized to listen on that channel.

> ![](https://laravel.com/img/callouts/lightbulb.min.svg)
>
> If you would like to use an open source, PHP driven alternative to Pusher, check out the [laravel-websockets](https://github.com/beyondcode/laravel-websockets) package.

#### [Using An Example Application](https://laravel.com/docs/7.x/broadcasting#using-example-application) <a id="using-example-application"></a>

Before diving into each component of event broadcasting, let's take a high level overview using an e-commerce store as an example. We won't discuss the details of configuring [Pusher Channels](https://pusher.com/channels) or [Laravel Echo](https://laravel.com/docs/7.x/broadcasting#installing-laravel-echo) since that will be discussed in detail in other sections of this documentation.

In our application, let's assume we have a page that allows users to view the shipping status for their orders. Let's also assume that a `ShippingStatusUpdated` event is fired when a shipping status update is processed by the application:

```text
event(new ShippingStatusUpdated($update));
```

**The ShouldBroadcast Interface**

When a user is viewing one of their orders, we don't want them to have to refresh the page to view status updates. Instead, we want to broadcast the updates to the application as they are created. So, we need to mark the `ShippingStatusUpdated` event with the `ShouldBroadcast` interface. This will instruct Laravel to broadcast the event when it is fired:

```text
<?php

namespace App\Events;

use Illuminate\Broadcasting\Channel;
use Illuminate\Broadcasting\InteractsWithSockets;
use Illuminate\Broadcasting\PresenceChannel;
use Illuminate\Broadcasting\PrivateChannel;
use Illuminate\Contracts\Broadcasting\ShouldBroadcast;
use Illuminate\Queue\SerializesModels;

class ShippingStatusUpdated implements ShouldBroadcast
{
    /**
     * Information about the shipping status update.
     *
     * @var string
     */
    public $update;
}
```

The `ShouldBroadcast` interface requires our event to define a `broadcastOn` method. This method is responsible for returning the channels that the event should broadcast on. An empty stub of this method is already defined on generated event classes, so we only need to fill in its details. We only want the creator of the order to be able to view status updates, so we will broadcast the event on a private channel that is tied to the order:

```text
/**
 * Get the channels the event should broadcast on.
 *
 * @return \Illuminate\Broadcasting\PrivateChannel
 */
public function broadcastOn()
{
    return new PrivateChannel('order.'.$this->update->order_id);
}
```

**Authorizing Channels**

Remember, users must be authorized to listen on private channels. We may define our channel authorization rules in the `routes/channels.php` file. In this example, we need to verify that any user attempting to listen on the private `order.1` channel is actually the creator of the order:

```text
Broadcast::channel('order.{orderId}', function ($user, $orderId) {
    return $user->id === Order::findOrNew($orderId)->user_id;
});
```

The `channel` method accepts two arguments: the name of the channel and a callback which returns `true` or `false` indicating whether the user is authorized to listen on the channel.

All authorization callbacks receive the currently authenticated user as their first argument and any additional wildcard parameters as their subsequent arguments. In this example, we are using the `{orderId}` placeholder to indicate that the "ID" portion of the channel name is a wildcard.

**Listening For Event Broadcasts**

Next, all that remains is to listen for the event in our JavaScript application. We can do this using Laravel Echo. First, we'll use the `private` method to subscribe to the private channel. Then, we may use the `listen` method to listen for the `ShippingStatusUpdated` event. By default, all of the event's public properties will be included on the broadcast event:

```text
Echo.private(`order.${orderId}`)
    .listen('ShippingStatusUpdated', (e) => {
        console.log(e.update);
    });
```

### [Defining Broadcast Events](https://laravel.com/docs/7.x/broadcasting#defining-broadcast-events) <a id="defining-broadcast-events"></a>

To inform Laravel that a given event should be broadcast, implement the `Illuminate\Contracts\Broadcasting\ShouldBroadcast` interface on the event class. This interface is already imported into all event classes generated by the framework so you may easily add it to any of your events.

The `ShouldBroadcast` interface requires you to implement a single method: `broadcastOn`. The `broadcastOn` method should return a channel or array of channels that the event should broadcast on. The channels should be instances of `Channel`, `PrivateChannel`, or `PresenceChannel`. Instances of `Channel` represent public channels that any user may subscribe to, while `PrivateChannels` and `PresenceChannels` represent private channels that require [channel authorization](https://laravel.com/docs/7.x/broadcasting#authorizing-channels):

```text
<?php

namespace App\Events;

use App\User;
use Illuminate\Broadcasting\Channel;
use Illuminate\Broadcasting\InteractsWithSockets;
use Illuminate\Broadcasting\PresenceChannel;
use Illuminate\Broadcasting\PrivateChannel;
use Illuminate\Contracts\Broadcasting\ShouldBroadcast;
use Illuminate\Queue\SerializesModels;

class ServerCreated implements ShouldBroadcast
{
    use SerializesModels;

    public $user;

    /**
     * Create a new event instance.
     *
     * @return void
     */
    public function __construct(User $user)
    {
        $this->user = $user;
    }

    /**
     * Get the channels the event should broadcast on.
     *
     * @return Channel|array
     */
    public function broadcastOn()
    {
        return new PrivateChannel('user.'.$this->user->id);
    }
}
```

Then, you only need to [fire the event](https://laravel.com/docs/7.x/events) as you normally would. Once the event has been fired, a [queued job](https://laravel.com/docs/7.x/queues) will automatically broadcast the event over your specified broadcast driver.

#### [Broadcast Name](https://laravel.com/docs/7.x/broadcasting#broadcast-name) <a id="broadcast-name"></a>

By default, Laravel will broadcast the event using the event's class name. However, you may customize the broadcast name by defining a `broadcastAs` method on the event:

```text
/**
 * The event's broadcast name.
 *
 * @return string
 */
public function broadcastAs()
{
    return 'server.created';
}
```

If you customize the broadcast name using the `broadcastAs` method, you should make sure to register your listener with a leading `.` character. This will instruct Echo to not prepend the application's namespace to the event:

```text
.listen('.server.created', function (e) {
    ....
});
```

#### [Broadcast Data](https://laravel.com/docs/7.x/broadcasting#broadcast-data) <a id="broadcast-data"></a>

When an event is broadcast, all of its `public` properties are automatically serialized and broadcast as the event's payload, allowing you to access any of its public data from your JavaScript application. So, for example, if your event has a single public `$user` property that contains an Eloquent model, the event's broadcast payload would be:

```text
{
    "user": {
        "id": 1,
        "name": "Patrick Stewart"
        ...
    }
}
```

However, if you wish to have more fine-grained control over your broadcast payload, you may add a `broadcastWith` method to your event. This method should return the array of data that you wish to broadcast as the event payload:

```text
/**
 * Get the data to broadcast.
 *
 * @return array
 */
public function broadcastWith()
{
    return ['id' => $this->user->id];
}
```

#### [Broadcast Queue](https://laravel.com/docs/7.x/broadcasting#broadcast-queue) <a id="broadcast-queue"></a>

By default, each broadcast event is placed on the default queue for the default queue connection specified in your `queue.php` configuration file. You may customize the queue used by the broadcaster by defining a `broadcastQueue` property on your event class. This property should specify the name of the queue you wish to use when broadcasting:

```text
/**
 * The name of the queue on which to place the event.
 *
 * @var string
 */
public $broadcastQueue = 'your-queue-name';
```

If you want to broadcast your event using the `sync` queue instead of the default queue driver, you can implement the `ShouldBroadcastNow` interface instead of `ShouldBroadcast`:

```text
<?php

use Illuminate\Contracts\Broadcasting\ShouldBroadcastNow;

class ShippingStatusUpdated implements ShouldBroadcastNow
{
    //
}
```

#### [Broadcast Conditions](https://laravel.com/docs/7.x/broadcasting#broadcast-conditions) <a id="broadcast-conditions"></a>

Sometimes you want to broadcast your event only if a given condition is true. You may define these conditions by adding a `broadcastWhen` method to your event class:

```text
/**
 * Determine if this event should broadcast.
 *
 * @return bool
 */
public function broadcastWhen()
{
    return $this->value > 100;
}
```

### [Authorizing Channels](https://laravel.com/docs/7.x/broadcasting#authorizing-channels) <a id="authorizing-channels"></a>

Private channels require you to authorize that the currently authenticated user can actually listen on the channel. This is accomplished by making an HTTP request to your Laravel application with the channel name and allowing your application to determine if the user can listen on that channel. When using [Laravel Echo](https://laravel.com/docs/7.x/broadcasting#installing-laravel-echo), the HTTP request to authorize subscriptions to private channels will be made automatically; however, you do need to define the proper routes to respond to these requests.

#### [Defining Authorization Routes](https://laravel.com/docs/7.x/broadcasting#defining-authorization-routes) <a id="defining-authorization-routes"></a>

Thankfully, Laravel makes it easy to define the routes to respond to channel authorization requests. In the `BroadcastServiceProvider` included with your Laravel application, you will see a call to the `Broadcast::routes` method. This method will register the `/broadcasting/auth` route to handle authorization requests:

```text
Broadcast::routes();
```

The `Broadcast::routes` method will automatically place its routes within the `web` middleware group; however, you may pass an array of route attributes to the method if you would like to customize the assigned attributes:

```text
Broadcast::routes($attributes);
```

**Customizing The Authorization Endpoint**

By default, Echo will use the `/broadcasting/auth` endpoint to authorize channel access. However, you may specify your own authorization endpoint by passing the `authEndpoint` configuration option to your Echo instance:

```text
window.Echo = new Echo({
    broadcaster: 'pusher',
    key: 'your-pusher-channels-key',
    authEndpoint: '/custom/endpoint/auth'
});
```

#### [Defining Authorization Callbacks](https://laravel.com/docs/7.x/broadcasting#defining-authorization-callbacks) <a id="defining-authorization-callbacks"></a>

Next, we need to define the logic that will actually perform the channel authorization. This is done in the `routes/channels.php` file that is included with your application. In this file, you may use the `Broadcast::channel` method to register channel authorization callbacks:

```text
Broadcast::channel('order.{orderId}', function ($user, $orderId) {
    return $user->id === Order::findOrNew($orderId)->user_id;
});
```

The `channel` method accepts two arguments: the name of the channel and a callback which returns `true` or `false` indicating whether the user is authorized to listen on the channel.

All authorization callbacks receive the currently authenticated user as their first argument and any additional wildcard parameters as their subsequent arguments. In this example, we are using the `{orderId}` placeholder to indicate that the "ID" portion of the channel name is a wildcard.

**Authorization Callback Model Binding**

Just like HTTP routes, channel routes may also take advantage of implicit and explicit [route model binding](https://laravel.com/docs/7.x/routing#route-model-binding). For example, instead of receiving the string or numeric order ID, you may request an actual `Order` model instance:

```text
use App\Order;

Broadcast::channel('order.{order}', function ($user, Order $order) {
    return $user->id === $order->user_id;
});
```

**Authorization Callback Authentication**

Private and presence broadcast channels authenticate the current user via your application's default authentication guard. If the user is not authenticated, channel authorization is automatically denied and the authorization callback is never executed. However, you may assign multiple, custom guards that should authenticate the incoming request if necessary:

```text
Broadcast::channel('channel', function () {
    // ...
}, ['guards' => ['web', 'admin']]);
```

#### [Defining Channel Classes](https://laravel.com/docs/7.x/broadcasting#defining-channel-classes) <a id="defining-channel-classes"></a>

If your application is consuming many different channels, your `routes/channels.php` file could become bulky. So, instead of using Closures to authorize channels, you may use channel classes. To generate a channel class, use the `make:channel` Artisan command. This command will place a new channel class in the `App/Broadcasting` directory.

```text
php artisan make:channel OrderChannel
```

Next, register your channel in your `routes/channels.php` file:

```text
use App\Broadcasting\OrderChannel;

Broadcast::channel('order.{order}', OrderChannel::class);
```

Finally, you may place the authorization logic for your channel in the channel class' `join` method. This `join` method will house the same logic you would have typically placed in your channel authorization Closure. You may also take advantage of channel model binding:

```text
<?php

namespace App\Broadcasting;

use App\Order;
use App\User;

class OrderChannel
{
    /**
     * Create a new channel instance.
     *
     * @return void
     */
    public function __construct()
    {
        //
    }

    /**
     * Authenticate the user's access to the channel.
     *
     * @param  \App\User  $user
     * @param  \App\Order  $order
     * @return array|bool
     */
    public function join(User $user, Order $order)
    {
        return $user->id === $order->user_id;
    }
}
```

> ![](https://laravel.com/img/callouts/lightbulb.min.svg)
>
> Like many other classes in Laravel, channel classes will automatically be resolved by the [service container](https://laravel.com/docs/7.x/container). So, you may type-hint any dependencies required by your channel in its constructor.

### [Broadcasting Events](https://laravel.com/docs/7.x/broadcasting#broadcasting-events) <a id="broadcasting-events"></a>

Once you have defined an event and marked it with the `ShouldBroadcast` interface, you only need to fire the event using the `event` function. The event dispatcher will notice that the event is marked with the `ShouldBroadcast` interface and will queue the event for broadcasting:

```text
event(new ShippingStatusUpdated($update));
```

#### [Only To Others](https://laravel.com/docs/7.x/broadcasting#only-to-others) <a id="only-to-others"></a>

When building an application that utilizes event broadcasting, you may substitute the `event` function with the `broadcast` function. Like the `event` function, the `broadcast` function dispatches the event to your server-side listeners:

```text
broadcast(new ShippingStatusUpdated($update));
```

However, the `broadcast` function also exposes the `toOthers` method which allows you to exclude the current user from the broadcast's recipients:

```text
broadcast(new ShippingStatusUpdated($update))->toOthers();
```

To better understand when you may want to use the `toOthers` method, let's imagine a task list application where a user may create a new task by entering a task name. To create a task, your application might make a request to a `/task` end-point which broadcasts the task's creation and returns a JSON representation of the new task. When your JavaScript application receives the response from the end-point, it might directly insert the new task into its task list like so:

```text
axios.post('/task', task)
    .then((response) => {
        this.tasks.push(response.data);
    });
```

However, remember that we also broadcast the task's creation. If your JavaScript application is listening for this event in order to add tasks to the task list, you will have duplicate tasks in your list: one from the end-point and one from the broadcast. You may solve this by using the `toOthers` method to instruct the broadcaster to not broadcast the event to the current user.

> ![](https://laravel.com/img/callouts/exclamation.min.svg)
>
> Your event must use the `Illuminate\Broadcasting\InteractsWithSockets` trait in order to call the `toOthers` method.

**Configuration**

When you initialize a Laravel Echo instance, a socket ID is assigned to the connection. If you are using [Vue](https://vuejs.org/) and [Axios](https://github.com/mzabriskie/axios), the socket ID will automatically be attached to every outgoing request as a `X-Socket-ID` header. Then, when you call the `toOthers` method, Laravel will extract the socket ID from the header and instruct the broadcaster to not broadcast to any connections with that socket ID.

If you are not using Vue and Axios, you will need to manually configure your JavaScript application to send the `X-Socket-ID` header. You may retrieve the socket ID using the `Echo.socketId` method:

```text
var socketId = Echo.socketId();
```

### [Receiving Broadcasts](https://laravel.com/docs/7.x/broadcasting#receiving-broadcasts) <a id="receiving-broadcasts"></a>

#### [Installing Laravel Echo](https://laravel.com/docs/7.x/broadcasting#installing-laravel-echo) <a id="installing-laravel-echo"></a>

Laravel Echo is a JavaScript library that makes it painless to subscribe to channels and listen for events broadcast by Laravel. You may install Echo via the NPM package manager. In this example, we will also install the `pusher-js` package since we will be using the Pusher Channels broadcaster:

```text
npm install --save laravel-echo pusher-js
```

Once Echo is installed, you are ready to create a fresh Echo instance in your application's JavaScript. A great place to do this is at the bottom of the `resources/js/bootstrap.js` file that is included with the Laravel framework:

```text
import Echo from "laravel-echo"

window.Echo = new Echo({
    broadcaster: 'pusher',
    key: 'your-pusher-channels-key'
});
```

When creating an Echo instance that uses the `pusher` connector, you may also specify a `cluster` as well as whether the connection must be made over TLS \(by default, when `forceTLS` is `false`, a non-TLS connection will be made if the page was loaded over HTTP, or as a fallback if a TLS connection fails\):

```text
window.Echo = new Echo({
    broadcaster: 'pusher',
    key: 'your-pusher-channels-key',
    cluster: 'eu',
    forceTLS: true
});
```

**Using An Existing Client Instance**

If you already have a Pusher Channels or Socket.io client instance that you would like Echo to utilize, you may pass it to Echo via the `client` configuration option:

```text
const client = require('pusher-js');

window.Echo = new Echo({
    broadcaster: 'pusher',
    key: 'your-pusher-channels-key',
    client: client
});
```

#### [Listening For Events](https://laravel.com/docs/7.x/broadcasting#listening-for-events) <a id="listening-for-events"></a>

Once you have installed and instantiated Echo, you are ready to start listening for event broadcasts. First, use the `channel` method to retrieve an instance of a channel, then call the `listen` method to listen for a specified event:

```text
Echo.channel('orders')
    .listen('OrderShipped', (e) => {
        console.log(e.order.name);
    });
```

If you would like to listen for events on a private channel, use the `private` method instead. You may continue to chain calls to the `listen` method to listen for multiple events on a single channel:

```text
Echo.private('orders')
    .listen(...)
    .listen(...)
    .listen(...);
```

#### [Leaving A Channel](https://laravel.com/docs/7.x/broadcasting#leaving-a-channel) <a id="leaving-a-channel"></a>

To leave a channel, you may call the `leaveChannel` method on your Echo instance:

```text
Echo.leaveChannel('orders');
```

If you would like to leave a channel and also its associated private and presence channels, you may call the `leave` method:

```text
Echo.leave('orders');
```

#### [Namespaces](https://laravel.com/docs/7.x/broadcasting#namespaces) <a id="namespaces"></a>

You may have noticed in the examples above that we did not specify the full namespace for the event classes. This is because Echo will automatically assume the events are located in the `App\Events` namespace. However, you may configure the root namespace when you instantiate Echo by passing a `namespace` configuration option:

```text
window.Echo = new Echo({
    broadcaster: 'pusher',
    key: 'your-pusher-channels-key',
    namespace: 'App.Other.Namespace'
});
```

Alternatively, you may prefix event classes with a `.` when subscribing to them using Echo. This will allow you to always specify the fully-qualified class name:

```text
Echo.channel('orders')
    .listen('.Namespace\\Event\\Class', (e) => {
        //
    });
```

### [Presence Channels](https://laravel.com/docs/7.x/broadcasting#presence-channels) <a id="presence-channels"></a>

Presence channels build on the security of private channels while exposing the additional feature of awareness of who is subscribed to the channel. This makes it easy to build powerful, collaborative application features such as notifying users when another user is viewing the same page.

#### [Authorizing Presence Channels](https://laravel.com/docs/7.x/broadcasting#authorizing-presence-channels) <a id="authorizing-presence-channels"></a>

All presence channels are also private channels; therefore, users must be [authorized to access them](https://laravel.com/docs/7.x/broadcasting#authorizing-channels). However, when defining authorization callbacks for presence channels, you will not return `true` if the user is authorized to join the channel. Instead, you should return an array of data about the user.

The data returned by the authorization callback will be made available to the presence channel event listeners in your JavaScript application. If the user is not authorized to join the presence channel, you should return `false` or `null`:

```text
Broadcast::channel('chat.{roomId}', function ($user, $roomId) {
    if ($user->canJoinRoom($roomId)) {
        return ['id' => $user->id, 'name' => $user->name];
    }
});
```

#### [Joining Presence Channels](https://laravel.com/docs/7.x/broadcasting#joining-presence-channels) <a id="joining-presence-channels"></a>

To join a presence channel, you may use Echo's `join` method. The `join` method will return a `PresenceChannel` implementation which, along with exposing the `listen` method, allows you to subscribe to the `here`, `joining`, and `leaving` events.

```text
Echo.join(`chat.${roomId}`)
    .here((users) => {
        //
    })
    .joining((user) => {
        console.log(user.name);
    })
    .leaving((user) => {
        console.log(user.name);
    });
```

The `here` callback will be executed immediately once the channel is joined successfully, and will receive an array containing the user information for all of the other users currently subscribed to the channel. The `joining` method will be executed when a new user joins a channel, while the `leaving` method will be executed when a user leaves the channel.

#### [Broadcasting To Presence Channels](https://laravel.com/docs/7.x/broadcasting#broadcasting-to-presence-channels) <a id="broadcasting-to-presence-channels"></a>

Presence channels may receive events just like public or private channels. Using the example of a chatroom, we may want to broadcast `NewMessage` events to the room's presence channel. To do so, we'll return an instance of `PresenceChannel` from the event's `broadcastOn` method:

```text
/**
 * Get the channels the event should broadcast on.
 *
 * @return Channel|array
 */
public function broadcastOn()
{
    return new PresenceChannel('room.'.$this->message->room_id);
}
```

Like public or private events, presence channel events may be broadcast using the `broadcast` function. As with other events, you may use the `toOthers` method to exclude the current user from receiving the broadcast:

```text
broadcast(new NewMessage($message));

broadcast(new NewMessage($message))->toOthers();
```

You may listen for the join event via Echo's `listen` method:

```text
Echo.join(`chat.${roomId}`)
    .here(...)
    .joining(...)
    .leaving(...)
    .listen('NewMessage', (e) => {
        //
    });
```

### [Client Events](https://laravel.com/docs/7.x/broadcasting#client-events) <a id="client-events"></a>

> ![](https://laravel.com/img/callouts/lightbulb.min.svg)
>
> When using [Pusher Channels](https://pusher.com/channels), you must enable the "Client Events" option in the "App Settings" section of your [application dashboard](https://dashboard.pusher.com/) in order to send client events.

Sometimes you may wish to broadcast an event to other connected clients without hitting your Laravel application at all. This can be particularly useful for things like "typing" notifications, where you want to alert users of your application that another user is typing a message on a given screen.

To broadcast client events, you may use Echo's `whisper` method:

```text
Echo.private('chat')
    .whisper('typing', {
        name: this.user.name
    });
```

To listen for client events, you may use the `listenForWhisper` method:

```text
Echo.private('chat')
    .listenForWhisper('typing', (e) => {
        console.log(e.name);
    });
```

### [Notifications](https://laravel.com/docs/7.x/broadcasting#notifications) <a id="notifications"></a>

By pairing event broadcasting with [notifications](https://laravel.com/docs/7.x/notifications), your JavaScript application may receive new notifications as they occur without needing to refresh the page. First, be sure to read over the documentation on using [the broadcast notification channel](https://laravel.com/docs/7.x/notifications#broadcast-notifications).

Once you have configured a notification to use the broadcast channel, you may listen for the broadcast events using Echo's `notification` method. Remember, the channel name should match the class name of the entity receiving the notifications:

```text
Echo.private(`App.User.${userId}`)
    .notification((notification) => {
        console.log(notification.type);
    });
```

In this example, all notifications sent to `App\User` instances via the `broadcast` channel would be received by the callback. A channel authorization callback for the `App.User.{id}` channel is included in the default `BroadcastServiceProvider` that ships with the Laravel framework.

