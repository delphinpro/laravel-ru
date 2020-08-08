# Фасады

## Вступление

Фасады обеспечивают "статический" интерфейс к классам, которые доступны в [service container](container.md) приложения. Laravel имеет множество фасадов, которые обеспечивают доступ почти ко всем возможностям фреймворка. Фасады служат "статическими прокси" к базовым классам в сервисном контейнере, обеспечивая преимущество лаконичного, экспрессивного синтаксиса при сохранении большей тестируемости и гибкости по сравнению с традиционными статическими методами.

Все фасады Laravel определены в пространстве имен `Illuminate\Support\Facades`. Таким образом, мы можем легко получить доступ к такому фасаду:

```php
use Illuminate\Support\Facades\Cache;

Route::get('/cache', function () {
    return Cache::get('key');
});
```

Во всей документации Laravel многие примеры будут использовать фасады для демонстрации различных особенностей фреймворка.

## Когда использовать фасады

Фасады имеют много преимуществ. Они обеспечивают лаконичный, запоминающийся синтаксис, который позволяет использовать функции Laravel, не запоминая длинные названия классов, которые необходимо вводить или настраивать вручную. Более того, благодаря уникальному использованию динамических методов PHP, они легко тестируются.

Тем не менее, при использовании фасадов необходимо проявлять некоторую осторожность. Основная опасность фасадов — это ползучесть по классу. Так как фасады настолько просты в использовании и не требуют внедрения, можно легко позволить Вашим классам продолжать расти и использовать много фасадов в одном классе. Используя внедрение зависимостей, этот потенциал смягчается визуальной обратной связью, которую дает вам большой конструктор, что ваш класс растет слишком большим. Поэтому, при использовании фасадов, обратите особое внимание на размер вашего класса, чтобы сфера его ответственности оставалась узкой.

{% hint style="info" %}

{% endhint %}

> ![](https://laravel.com/img/callouts/lightbulb.min.svg)
>
> При создании стороннего пакета, взаимодействующего с Laravel, лучше сделать внедрение [Laravel Contracts](contracts.md) вместо использования фасадов. Поскольку пакеты создаются вне самой Laravel, у вас не будет доступа к помощникам Laravel по тестированию фасадов.

### Фасады Против Внедрения зависимостей

Одним из основных преимуществ внедрения зависимостей является возможность обмена реализациями внедряемого класса. Это полезно при тестировании, так как можно вводить имитацию или заглушку и утверждать, что различные методы были вызваны в заглушке.

Обычно невозможно имитировать или заглушить по-настоящему статический метод класса. Однако, поскольку фасады используют динамические методы для прокси вызова методов к объектам, разрешенным из сервисного контейнера, мы фактически можем тестировать фасады точно так же, как мы тестировали бы внедряемый экземпляр класса. Например, учитывая следующий маршрут:

```php
use Illuminate\Support\Facades\Cache;

Route::get('/cache', function () {
    return Cache::get('key');
});
```

We can write the following test to verify that the `Cache::get` method was called with the argument we expected:

```php
use Illuminate\Support\Facades\Cache;

/**
 * A basic functional test example.
 *
 * @return void
 */
public function testBasicExample()
{
    Cache::shouldReceive('get')
         ->with('key')
         ->andReturn('value');

    $this->visit('/cache')
         ->see('value');
}
```

### Facades Vs. Helper Functions <a id="facades-vs-helper-functions"></a>

In addition to facades, Laravel includes a variety of "helper" functions which can perform common tasks like generating views, firing events, dispatching jobs, or sending HTTP responses. Many of these helper functions perform the same function as a corresponding facade. For example, this facade call and helper call are equivalent:

```php
return View::make('profile');

return view('profile');
```

There is absolutely no practical difference between facades and helper functions. When using helper functions, you may still test them exactly as you would the corresponding facade. For example, given the following route:

```php
Route::get('/cache', function () {
    return cache('key');
});
```

Under the hood, the `cache` helper is going to call the `get` method on the class underlying the `Cache` facade. So, even though we are using the helper function, we can write the following test to verify that the method was called with the argument we expected:

```php
use Illuminate\Support\Facades\Cache;

/**
 * A basic functional test example.
 *
 * @return void
 */
public function testBasicExample()
{
    Cache::shouldReceive('get')
         ->with('key')
         ->andReturn('value');

    $this->visit('/cache')
         ->see('value');
}
```

## How Facades Work <a id="how-facades-work"></a>

In a Laravel application, a facade is a class that provides access to an object from the container. The machinery that makes this work is in the `Facade` class. Laravel's facades, and any custom facades you create, will extend the base `Illuminate\Support\Facades\Facade` class.

The `Facade` base class makes use of the `__callStatic()` magic-method to defer calls from your facade to an object resolved from the container. In the example below, a call is made to the Laravel cache system. By glancing at this code, one might assume that the static method `get` is being called on the `Cache` class:

```php
<?php

namespace App\Http\Controllers;

use App\Http\Controllers\Controller;
use Illuminate\Support\Facades\Cache;

class UserController extends Controller
{
    /**
     * Show the profile for the given user.
     *
     * @param  int  $id
     * @return Response
     */
    public function showProfile($id)
    {
        $user = Cache::get('user:'.$id);

        return view('profile', ['user' => $user]);
    }
}
```

Notice that near the top of the file we are "importing" the `Cache` facade. This facade serves as a proxy to accessing the underlying implementation of the `Illuminate\Contracts\Cache\Factory` interface. Any calls we make using the facade will be passed to the underlying instance of Laravel's cache service.

If we look at that `Illuminate\Support\Facades\Cache` class, you'll see that there is no static method `get`:

```php
class Cache extends Facade
{
    /**
     * Get the registered name of the component.
     *
     * @return string
     */
    protected static function getFacadeAccessor() { return 'cache'; }
}
```

Instead, the `Cache` facade extends the base `Facade` class and defines the method `getFacadeAccessor()`. This method's job is to return the name of a service container binding. When a user references any static method on the `Cache` facade, Laravel resolves the `cache` binding from the [service container](https://laravel.com/docs/7.x/container) and runs the requested method \(in this case, `get`\) against that object.

## Real-Time Facades <a id="real-time-facades"></a>

Using real-time facades, you may treat any class in your application as if it were a facade. To illustrate how this can be used, let's examine an alternative. For example, let's assume our `Podcast` model has a `publish` method. However, in order to publish the podcast, we need to inject a `Publisher` instance:

```php
<?php

namespace App;

use App\Contracts\Publisher;
use Illuminate\Database\Eloquent\Model;

class Podcast extends Model
{
    /**
     * Publish the podcast.
     *
     * @param  Publisher  $publisher
     * @return void
     */
    public function publish(Publisher $publisher)
    {
        $this->update(['publishing' => now()]);

        $publisher->publish($this);
    }
}
```

Injecting a publisher implementation into the method allows us to easily test the method in isolation since we can mock the injected publisher. However, it requires us to always pass a publisher instance each time we call the `publish` method. Using real-time facades, we can maintain the same testability while not being required to explicitly pass a `Publisher` instance. To generate a real-time facade, prefix the namespace of the imported class with `Facades`:

```php
<?php

namespace App;

use Facades\App\Contracts\Publisher;
use Illuminate\Database\Eloquent\Model;

class Podcast extends Model
{
    /**
     * Publish the podcast.
     *
     * @return void
     */
    public function publish()
    {
        $this->update(['publishing' => now()]);

        Publisher::publish($this);
    }
}
```

When the real-time facade is used, the publisher implementation will be resolved out of the service container using the portion of the interface or class name that appears after the `Facades` prefix. When testing, we can use Laravel's built-in facade testing helpers to mock this method call:

```php
<?php

namespace Tests\Feature;

use App\Podcast;
use Facades\App\Contracts\Publisher;
use Illuminate\Foundation\Testing\RefreshDatabase;
use Tests\TestCase;

class PodcastTest extends TestCase
{
    use RefreshDatabase;

    /**
     * A test example.
     *
     * @return void
     */
    public function test_podcast_can_be_published()
    {
        $podcast = factory(Podcast::class)->create();

        Publisher::shouldReceive('publish')->once()->with($podcast);

        $podcast->publish();
    }
}
```

## Facade Class Reference <a id="facade-class-reference"></a>

Below you will find every facade and its underlying class. This is a useful tool for quickly digging into the API documentation for a given facade root. The [service container binding](https://laravel.com/docs/7.x/container) key is also included where applicable.

| Facade | Class | Service Container Binding |
| :--- | :--- | :--- |
| App | [Illuminate\Foundation\Application](https://laravel.com/api/7.x/Illuminate/Foundation/Application.html) | `app` |
| Artisan | [Illuminate\Contracts\Console\Kernel](https://laravel.com/api/7.x/Illuminate/Contracts/Console/Kernel.html) | `artisan` |
| Auth | [Illuminate\Auth\AuthManager](https://laravel.com/api/7.x/Illuminate/Auth/AuthManager.html) | `auth` |
| Auth \(Instance\) | [Illuminate\Contracts\Auth\Guard](https://laravel.com/api/7.x/Illuminate/Contracts/Auth/Guard.html) | `auth.driver` |
| Blade | [Illuminate\View\Compilers\BladeCompiler](https://laravel.com/api/7.x/Illuminate/View/Compilers/BladeCompiler.html) | `blade.compiler` |
| Broadcast | [Illuminate\Contracts\Broadcasting\Factory](https://laravel.com/api/7.x/Illuminate/Contracts/Broadcasting/Factory.html) |  |
| Broadcast \(Instance\) | [Illuminate\Contracts\Broadcasting\Broadcaster](https://laravel.com/api/7.x/Illuminate/Contracts/Broadcasting/Broadcaster.html) |  |
| Bus | [Illuminate\Contracts\Bus\Dispatcher](https://laravel.com/api/7.x/Illuminate/Contracts/Bus/Dispatcher.html) |  |
| Cache | [Illuminate\Cache\CacheManager](https://laravel.com/api/7.x/Illuminate/Cache/CacheManager.html) | `cache` |
| Cache \(Instance\) | [Illuminate\Cache\Repository](https://laravel.com/api/7.x/Illuminate/Cache/Repository.html) | `cache.store` |
| Config | [Illuminate\Config\Repository](https://laravel.com/api/7.x/Illuminate/Config/Repository.html) | `config` |
| Cookie | [Illuminate\Cookie\CookieJar](https://laravel.com/api/7.x/Illuminate/Cookie/CookieJar.html) | `cookie` |
| Crypt | [Illuminate\Encryption\Encrypter](https://laravel.com/api/7.x/Illuminate/Encryption/Encrypter.html) | `encrypter` |
| DB | [Illuminate\Database\DatabaseManager](https://laravel.com/api/7.x/Illuminate/Database/DatabaseManager.html) | `db` |
| DB \(Instance\) | [Illuminate\Database\Connection](https://laravel.com/api/7.x/Illuminate/Database/Connection.html) | `db.connection` |
| Event | [Illuminate\Events\Dispatcher](https://laravel.com/api/7.x/Illuminate/Events/Dispatcher.html) | `events` |
| File | [Illuminate\Filesystem\Filesystem](https://laravel.com/api/7.x/Illuminate/Filesystem/Filesystem.html) | `files` |
| Gate | [Illuminate\Contracts\Auth\Access\Gate](https://laravel.com/api/7.x/Illuminate/Contracts/Auth/Access/Gate.html) |  |
| Hash | [Illuminate\Contracts\Hashing\Hasher](https://laravel.com/api/7.x/Illuminate/Contracts/Hashing/Hasher.html) | `hash` |
| Http | [Illuminate\Http\Client\Factory](https://laravel.com/api/7.x/Illuminate/Http/Client/Factory.html) |  |
| Lang | [Illuminate\Translation\Translator](https://laravel.com/api/7.x/Illuminate/Translation/Translator.html) | `translator` |
| Log | [Illuminate\Log\LogManager](https://laravel.com/api/7.x/Illuminate/Log/LogManager.html) | `log` |
| Mail | [Illuminate\Mail\Mailer](https://laravel.com/api/7.x/Illuminate/Mail/Mailer.html) | `mailer` |
| Notification | [Illuminate\Notifications\ChannelManager](https://laravel.com/api/7.x/Illuminate/Notifications/ChannelManager.html) |  |
| Password | [Illuminate\Auth\Passwords\PasswordBrokerManager](https://laravel.com/api/7.x/Illuminate/Auth/Passwords/PasswordBrokerManager.html) | `auth.password` |
| Password \(Instance\) | [Illuminate\Auth\Passwords\PasswordBroker](https://laravel.com/api/7.x/Illuminate/Auth/Passwords/PasswordBroker.html) | `auth.password.broker` |
| Queue | [Illuminate\Queue\QueueManager](https://laravel.com/api/7.x/Illuminate/Queue/QueueManager.html) | `queue` |
| Queue \(Instance\) | [Illuminate\Contracts\Queue\Queue](https://laravel.com/api/7.x/Illuminate/Contracts/Queue/Queue.html) | `queue.connection` |
| Queue \(Base Class\) | [Illuminate\Queue\Queue](https://laravel.com/api/7.x/Illuminate/Queue/Queue.html) |  |
| Redirect | [Illuminate\Routing\Redirector](https://laravel.com/api/7.x/Illuminate/Routing/Redirector.html) | `redirect` |
| Redis | [Illuminate\Redis\RedisManager](https://laravel.com/api/7.x/Illuminate/Redis/RedisManager.html) | `redis` |
| Redis \(Instance\) | [Illuminate\Redis\Connections\Connection](https://laravel.com/api/7.x/Illuminate/Redis/Connections/Connection.html) | `redis.connection` |
| Request | [Illuminate\Http\Request](https://laravel.com/api/7.x/Illuminate/Http/Request.html) | `request` |
| Response | [Illuminate\Contracts\Routing\ResponseFactory](https://laravel.com/api/7.x/Illuminate/Contracts/Routing/ResponseFactory.html) |  |
| Response \(Instance\) | [Illuminate\Http\Response](https://laravel.com/api/7.x/Illuminate/Http/Response.html) |  |
| Route | [Illuminate\Routing\Router](https://laravel.com/api/7.x/Illuminate/Routing/Router.html) | `router` |
| Schema | [Illuminate\Database\Schema\Builder](https://laravel.com/api/7.x/Illuminate/Database/Schema/Builder.html) |  |
| Session | [Illuminate\Session\SessionManager](https://laravel.com/api/7.x/Illuminate/Session/SessionManager.html) | `session` |
| Session \(Instance\) | [Illuminate\Session\Store](https://laravel.com/api/7.x/Illuminate/Session/Store.html) | `session.store` |
| Storage | [Illuminate\Filesystem\FilesystemManager](https://laravel.com/api/7.x/Illuminate/Filesystem/FilesystemManager.html) | `filesystem` |
| Storage \(Instance\) | [Illuminate\Contracts\Filesystem\Filesystem](https://laravel.com/api/7.x/Illuminate/Contracts/Filesystem/Filesystem.html) | `filesystem.disk` |
| URL | [Illuminate\Routing\UrlGenerator](https://laravel.com/api/7.x/Illuminate/Routing/UrlGenerator.html) | `url` |
| Validator | [Illuminate\Validation\Factory](https://laravel.com/api/7.x/Illuminate/Validation/Factory.html) | `validator` |
| Validator \(Instance\) | [Illuminate\Validation\Validator](https://laravel.com/api/7.x/Illuminate/Validation/Validator.html) |  |
| View | [Illuminate\View\Factory](https://laravel.com/api/7.x/Illuminate/View/Factory.html) | `view` |
| View \(Instance\) | [Illuminate\View\View](https://laravel.com/api/7.x/Illuminate/View/View.html) |  |

