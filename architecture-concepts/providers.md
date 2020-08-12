# Service Providers

## Введение

Сервис-провайдеры являются центральным местом загрузки всех приложений Laravel. Ваше собственное приложение, как и все основные сервисы Laravel, загружаются через сервис-провайдеров.

Но, что мы имеем в виду под "загрузкой"? В общем, мы имеем в виду **регистрацию** чего угодно, в том числе регистрацию сервисных контейнеров, слушателей событий, посредников и даже маршрутов. Сервис-провайдеры являются центральным местом для настройки вашего приложения.

Если вы откроете файл `config/app.php`, входящий в состав Laravel, вы увидите массив `providers`. Это все классы провайдеров, которые будут загружены для вашего приложения. Обратите внимание, что многие из них являются "отложенными" провайдерами, то есть они будут загружаться не при каждом запросе, а только тогда, когда сервисы, которые они предоставляют, действительно необходимы.

В этом обзоре вы узнаете, как написать своих собственных поставщиков услуг и зарегистрировать их в своем приложении Laravel.

## Написание сервис-провайдеров

Все сервис-провайдеры расширяют класс `Illuminate\Support\ServiceProvider`. Большинство сервис-провайдеров содержат метод `register` и `boot`. В методе `register` вы должны **только привязывать к ** [**сервис-контейнеру**](container.md). Вы никогда не должны пытаться зарегистрировать слушателей событий, маршрутов или любой другой элемент функциональности в методе `register`.

Artisan CLI может сгенерировать нового провайдера с помощью команды `make:provider`:

```bash
php artisan make:provider RiakServiceProvider
```

### Метод register

Как уже упоминалось ранее, в рамках метода `register` следует только связывать вещи c [сервисным контейнером](container.md). Вы никогда не должны пытаться зарегистрировать слушателей событий, маршрутов или любой другой элемент функциональности в методе `register`. В противном случае Вы можете случайно воспользоваться сервисом, предоставляемым сервис-провайдером, который еще не загружен.

Давайте посмотрим на основного поставщика услуг. В рамках любого из методов Вашего сервис-провайдера Вы всегда имеете доступ к свойству `$app`, которое обеспечивает доступ к сервис-контейнеру:

```php
<?php

namespace App\Providers;

use Illuminate\Support\ServiceProvider;
use Riak\Connection;

class RiakServiceProvider extends ServiceProvider
{
    /**
     * Register any application services.
     *
     * @return void
     */
    public function register()
    {
        $this->app->singleton(Connection::class, function ($app) {
            return new Connection(config('riak'));
        });
    }
}
```

Этот провайдер услуг определяет только метод `купшыеук` и использует его для определения реализации `Riak\Connection` в сервис-контейнере. Если вы не понимаете, как работает сервис-контейнер, ознакомьтесь с [его документацией](container.md).

**Свойства bindings и singletons**

Если Ваш сервис-провайдер регистрирует много простых привязок, Вы можете использовать свойства `bindings` и `singletons` вместо того, чтобы вручную регистрировать каждую привязку контейнера. Когда провайдер загружается фреймворком, он автоматически проверяет эти свойства и регистрирует их привязки:

```php
<?php

namespace App\Providers;

use App\Contracts\DowntimeNotifier;
use App\Contracts\ServerProvider;
use App\Services\DigitalOceanServerProvider;
use App\Services\PingdomDowntimeNotifier;
use App\Services\ServerToolsProvider;
use Illuminate\Support\ServiceProvider;

class AppServiceProvider extends ServiceProvider
{
    /**
     * All of the container bindings that should be registered.
     *
     * @var array
     */
    public $bindings = [
        ServerProvider::class => DigitalOceanServerProvider::class,
    ];

    /**
     * All of the container singletons that should be registered.
     *
     * @var array
     */
    public $singletons = [
        DowntimeNotifier::class => PingdomDowntimeNotifier::class,
        ServerToolsProvider::class => ServerToolsProvider::class,
    ];
}
```

### Метод Boot

Что если нам нужно зарегистрировать [представление composer](https://laravel.com/docs/7.x/views#view-composers) в нашем сервис-провайдере? Это должно быть сделано в рамках метода `boot`. **Этот метод вызывается после того, как все другие провайдеры были зарегистрированы**, что означает, что у вас есть доступ ко всем другим сервисам, которые были зарегистрированы фреймворком:

```php
<?php

namespace App\Providers;

use Illuminate\Support\ServiceProvider;

class ComposerServiceProvider extends ServiceProvider
{
    /**
     * Bootstrap any application services.
     *
     * @return void
     */
    public function boot()
    {
        view()->composer('view', function () {
            //
        });
    }
}
```

**Boot Method Dependency Injection**

Вы можете указывать тайп-хинт зависимостей в методе `boot` вашего сервис-провайдера. [Сервис-контейнер](container.md) автоматически внедрит любые необходимые Вам зависимости:

```php
use Illuminate\Contracts\Routing\ResponseFactory;

public function boot(ResponseFactory $response)
{
    $response->macro('caps', function ($value) {
        //
    });
}
```

## Регистрация провайдеров

Все провайдеры услуг зарегистрированы в конфигурационном файле `config/app.php`. Этот файл содержит массив `providers`, в котором можно перечислить имена классов ваших провайдеров. По умолчанию в этом массиве перечислен набор провайдеров ядра Laravel. Эти провайдеры загружают основные компоненты Laravel, такие как mailer, очередь, кэш и другие.

Чтобы зарегистрировать сервис-провайдера, добавьте его в массив:

```php
'providers' => [
    // Other Service Providers

    App\Providers\ComposerServiceProvider::class,
],
```

## Deferred Providers

Если Ваш сервис-провайдер **только** регистрирует привязки в [контейнере услуг](container.md), Вы можете отложить его регистрацию до тех пор, пока одна из зарегистрированных связок не понадобится на самом деле. Отсрочка загрузки такого провайдера улучшит производительность Вашего приложения, так как оно не загружается из файловой системы при каждом запросе.

Laravel составляет и хранит список всех услуг, предоставляемых поставщиками отложенных услуг, вместе с названием класса своего поставщика услуг. Затем, только когда вы пытаетесь разрешить одну из этих услуг, Laravel загружает поставщика услуг.

Чтобы отложить загрузку провайдера, реализуем интерфейс `\Illuminate\Contracts\Support\DeferrableProvider` и определим метод `provides`. Метод `provides` должен возвращать привязки контейнеров услуг, зарегистрированных провайдером:

```php
<?php

namespace App\Providers;

use Illuminate\Contracts\Support\DeferrableProvider;
use Illuminate\Support\ServiceProvider;
use Riak\Connection;

class RiakServiceProvider extends ServiceProvider implements DeferrableProvider
{
    /**
     * Register any application services.
     *
     * @return void
     */
    public function register()
    {
        $this->app->singleton(Connection::class, function ($app) {
            return new Connection($app['config']['riak']);
        });
    }

    /**
     * Get the services provided by the provider.
     *
     * @return array
     */
    public function provides()
    {
        return [Connection::class];
    }
}
```

