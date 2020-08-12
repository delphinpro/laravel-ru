# Service Container

## Вступление

Сервисный контейнер Laravel является мощным инструментом для управления классовыми зависимостями и выполнения инъекций зависимостей. Инъекция зависимостей — это причудливая фраза, которая, по сути, означает следующее: классовые зависимости "инжектируются" в класс через конструктор или, в некоторых случаях, через методы "setter".

Давайте рассмотрим простой пример:

```php
<?php

namespace App\Http\Controllers;

use App\Http\Controllers\Controller;
use App\Repositories\UserRepository;
use App\User;

class UserController extends Controller
{
    /**
     * The user repository implementation.
     *
     * @var UserRepository
     */
    protected $users;

    /**
     * Create a new controller instance.
     *
     * @param  UserRepository  $users
     * @return void
     */
    public function __construct(UserRepository $users)
    {
        $this->users = $users;
    }

    /**
     * Show the profile for the given user.
     *
     * @param  int  $id
     * @return Response
     */
    public function show($id)
    {
        $user = $this->users->find($id);

        return view('user.profile', ['user' => $user]);
    }
}
```

В этом примере `UserController` должен извлекать пользователей из источника данных. Таким образом, мы **внедрим** службу, которая может извлекать пользователей. В этом контексте наш `UserRepository` скорее всего использует [Eloquent](../eloquent/eloquent.md) для получения информации о пользователе из базы данных. Однако, поскольку репозиторий инжектируется, мы можем легко поменять его на другую реализацию. Также мы можем легко "мокнуть", или создать фиктивную реализацию `UserRepository` при тестировании нашего приложения.

Глубокое понимание сервисного контейнера Laravel необходимо для создания мощного, крупного приложения, а также для внесения вклада в само ядро Laravel.

## Биндинг

### Основы биндинга

Почти все биндинги к сервис-контейнерам будут зарегистрированы в [сервис-провайдере](providers.md), поэтому большинство из этих примеров продемонстрируют использование контейнера в данном контексте.

{% hint style="info" %}
Нет необходимости привязывать классы к контейнеру, если они не зависят от каких-либо интерфейсов. Контейнер не нуждается в инструктаже о том, как строить эти объекты, так как он может автоматически разрешать эти объекты с помощью рефлексии.
{% endhint %}

**Simple Bindings**

Внутри поставщика услуг вы всегда имеете доступ к контейнеру через свойство `$this->app`. Мы можем зарегистрировать привязку, используя метод `bind`, передав имя класса или интерфейса, который мы хотим зарегистрировать вместе с `Closure`, который возвращает экземпляр класса:

```php
$this->app->bind('HelpSpot\API', function ($app) {
    return new \HelpSpot\API($app->make('HttpClient'));
});
```

Обратите внимание, что мы получаем сам контейнер в качестве аргумента в резолвер. Затем мы можем использовать контейнер для разрешения суб-зависимостей строящегося объекта.

**Биндинг синглтона**

Метод `singleton` привязывает к контейнеру класс или интерфейс, который должен быть разрешен только один раз. Как только singleton-привязка будет разрешена, тот же самый экземпляр объекта будет возвращен при последующих вызовах в контейнер:

```php
$this->app->singleton('HelpSpot\API', function ($app) {
    return new \HelpSpot\API($app->make('HttpClient'));
});
```

**Биндинг экземпляра**

Вы также можете привязать существующий экземпляр объекта к контейнеру с помощью метода `instance`. Данный экземпляр всегда будет возвращаться при последующих обращениях в контейнер:
You may also bind an existing object instance into the container using the `instance` method. The given instance will always be returned on subsequent calls into the container:

```php
$api = new \HelpSpot\API(new HttpClient);

$this->app->instance('HelpSpot\API', $api);
```

### Биндинг интерфейса к реализации

Очень мощной особенностью сервисного контейнера является его способность привязывать интерфейс к заданной реализации. Например, предположим, что у нас есть интерфейс `EventPusher` и реализация `RedisEventPusher`. После того, как мы закодировали нашу реализацию `RedisEventPusher` этого интерфейса, мы можем зарегистрировать его в сервисном контейнере таким образом: 

```php
$this->app->bind(
    'App\Contracts\EventPusher',
    'App\Services\RedisEventPusher'
);
```

Это выражение говорит контейнеру, что он должен внедрять `RedisEventPusher`, когда классу нужна реализация `EventPusher`. Теперь мы можем писать тайп-хинт на интерфейс `EventPusher` в конструкторе, или в любом другом месте, где зависимости инжектируются сервисным контейнером:

```php
use App\Contracts\EventPusher;

/**
 * Create a new class instance.
 *
 * @param  EventPusher  $pusher
 * @return void
 */
public function __construct(EventPusher $pusher)
{
    $this->pusher = $pusher;
}
```

### Контекстный биндинг

Иногда у вас может быть два класса, которые используют один и тот же интерфейс, но вы хотите внедрять различные реализации в каждый класс. Например, два контроллера могут зависеть от различных реализаций [контракта](contracts.md) `Illuminate\Contracts\Filesystem\Filesystem`. Ларавел предоставляет простой интерфейс для определения такого поведения:

```php
use App\Http\Controllers\PhotoController;
use App\Http\Controllers\UploadController;
use App\Http\Controllers\VideoController;
use Illuminate\Contracts\Filesystem\Filesystem;
use Illuminate\Support\Facades\Storage;

$this->app->when(PhotoController::class)
          ->needs(Filesystem::class)
          ->give(function () {
              return Storage::disk('local');
          });

$this->app->when([VideoController::class, UploadController::class])
          ->needs(Filesystem::class)
          ->give(function () {
              return Storage::disk('s3');
          });
```

### Биндинг примитивов

Иногда вы можете иметь класс, который получает некоторые вводимые классы, но также нуждается в внедрении примитивного значения, такого как целое число. Вы можете легко использовать контекстную привязку, чтобы ввести любое значение, которое может понадобиться вашему классу:

```php
$this->app->when('App\Http\Controllers\UserController')
          ->needs('$variableName')
          ->give($value);
```

Иногда класс может зависеть от массива тегированных экземпляров. Используя метод `giveTagged`, можно легко ввести все привязки контейнеров с помощью этого тега:

```php
$this->app->when(ReportAggregator::class)
    ->needs('$reports')
    ->giveTagged('reports');
```

### Binding Typed Variadics

Иногда можно иметь класс, который получает массив типизированных объектов с помощью конструктора variadic:

```php
class Firewall
{
    protected $logger;
    protected $filters;

    public function __construct(Logger $logger, Filter ...$filters)
    {
        $this->logger = $logger;
        $this->filters = $filters;
    }
}
```

Используя контекстную привязку, вы можете разрешить эту зависимость, предоставив метод `give` с Closure, который возвращает массив разрешенных экземпляров `Filter`:

```php
$this->app->when(Firewall::class)
          ->needs(Filter::class)
          ->give(function ($app) {
                return [
                    $app->make(NullFilter::class),
                    $app->make(ProfanityFilter::class),
                    $app->make(TooLongFilter::class),
                ];
          });
```

Для удобства, вы также можете просто предоставить массив имен классов, которые будут разрешаться контейнером всякий раз, когда `Firewall` нуждается в `Filter` экземплярах:

```php
$this->app->when(Firewall::class)
          ->needs(Filter::class)
          ->give([
              NullFilter::class,
              ProfanityFilter::class,
              TooLongFilter::class,
          ]);
```

**Variadic Tag Dependencies**

Иногда класс может иметь вариадическую зависимость, которая имеет тайп-хинт \(`Report ...$reports`\). Используя методы `needs` и `giveTagged`, можно легко ввести все привязки контейнеров с этим тегом для данной зависимости:

```php
$this->app->when(ReportAggregator::class)
    ->needs(Report::class)
    ->giveTagged('reports');
```

### Тегирование

Иногда может понадобиться решить все проблемы определенной "категории" привязки. Например, возможно, Вы строите агрегатор отчетов, который получает массив из множества различных реализаций интерфейса `Report`. После регистрации реализаций `Report`, Вы можете присвоить им тег, используя метод `tag`:

```php
$this->app->bind('SpeedReport', function () {
    //
});

$this->app->bind('MemoryReport', function () {
    //
});

$this->app->tag(['SpeedReport', 'MemoryReport'], 'reports');
```

После того, как сервисы будут тегированы, вы можете легко разрешить их все с помощью метода `tagged`:

```php
$this->app->bind('ReportAggregator', function ($app) {
    return new ReportAggregator($app->tagged('reports'));
});
```

### Расширенные биндинги

Метод `extend` позволяет модифицировать разрешенные сервисы. Например, когда сервис разрешен, вы можете запустить дополнительный код для украшения или настройки сервиса. Метод `extend` в качестве единственного аргумента принимает Closure, который должен возвращать модифицированный сервис. Closure получает разрешаемый сервис и экземпляр контейнера:

```php
$this->app->extend(Service::class, function ($service, $app) {
    return new DecoratedService($service);
});
```

## Разрешение

### **Метод `make`**

Вы можете использовать метод `make` для разрешения экземпляра класса из контейнера. Метод `make` принимает имя класса или интерфейса, который вы хотите разрешить:

```php
$api = $this->app->make('HelpSpot\API');
```

Если Вы находитесь в месте, где находится Ваш код, который не имеет доступа к переменной `$app`, Вы можете воспользоваться глобальным помощником `resolve`:

```php
$api = resolve('HelpSpot\API');
```

Если некоторые из зависимостей вашего класса не разрешимы через контейнер, вы можете внедрить их, передав в виде ассоциативного массива в метод `makeWith`:

```php
$api = $this->app->makeWith('HelpSpot\API', ['id' => 1]);
```

### **Автоматическое внедрение**

Или же, что немаловажно, можно "type-hint" зависимость в конструкторе класса, которая разрешается контейнером, в том числе в [контроллерах](.../the-basics/controllers.md), [слушателях событий](.../digging-deeper/events.md), [посредниках](.../the-basics/middleware.md), и многих других. Кроме того, вы можете type-hint зависимости в методе `handle` [очереди заданий](../digging-deeper/queues.md). На практике, это то, как большинство ваших объектов должны быть решены контейнером.

Например, вы можете type-hint в конструкторе контроллера подсказку к репозиторию, определённому вашим приложением. Хранилище будет автоматически разрешено и внедрено в класс:

```php
<?php

namespace App\Http\Controllers;

use App\Users\Repository as UserRepository;

class UserController extends Controller
{
    /**
     * The user repository instance.
     */
    protected $users;

    /**
     * Create a new controller instance.
     *
     * @param  UserRepository  $users
     * @return void
     */
    public function __construct(UserRepository $users)
    {
        $this->users = $users;
    }

    /**
     * Show the user with the given ID.
     *
     * @param  int  $id
     * @return Response
     */
    public function show($id)
    {
        //
    }
}
```

## События контейнера

Сервисный контейнер запускает событие каждый раз при разрешении объекта. Вы можете прослушать это событие, используя метод `resolving`:

```php
$this->app->resolving(function ($object, $app) {
    // Called when container resolves object of any type...
});

$this->app->resolving(\HelpSpot\API::class, function ($api, $app) {
    // Called when container resolves objects of type "HelpSpot\API"...
});
```

Как видите, разрешаемый объект будет передан функции обратного вызова, что позволит вам установить любые дополнительные свойства на объекте до того, как он будет передан его потребителю.

## PSR-11

Сервисный контейнер Laravel реализует интерфейс [PSR-11](https://github.com/php-fig/fig-standards/blob/master/accepted/PSR-11-container.md). Поэтому, чтобы получить экземпляр контейнера Laravel, Вы можете указать type-hint на интерфейс контейнера PSR-11:

```php
use Psr\Container\ContainerInterface;

Route::get('/', function (ContainerInterface $container) {
    $service = $container->get('Service');

    //
});
```

Исключение бросается, если заданный идентификатор не может быть разрешен. Исключением будет экземпляр `Psr\Container\NotFoundExceptionInterface`, если идентификатор никогда не был связан. Если идентификатор был связан, но не смог быть разрешен, то будет выброшен экземпляр `Psr\Container\ContainerExceptionInterface`.

# Service Container

## Вступление

Сервисный контейнер Laravel является мощным инструментом для управления классовыми зависимостями и выполнения инъекций зависимостей. Инъекция зависимостей — это причудливая фраза, которая, по сути, означает следующее: классовые зависимости "инжектируются" в класс через конструктор или, в некоторых случаях, через методы "setter".

Давайте рассмотрим простой пример:

```php
<?php

namespace App\Http\Controllers;

use App\Http\Controllers\Controller;
use App\Repositories\UserRepository;
use App\User;

class UserController extends Controller
{
    /**
     * The user repository implementation.
     *
     * @var UserRepository
     */
    protected $users;

    /**
     * Create a new controller instance.
     *
     * @param  UserRepository  $users
     * @return void
     */
    public function __construct(UserRepository $users)
    {
        $this->users = $users;
    }

    /**
     * Show the profile for the given user.
     *
     * @param  int  $id
     * @return Response
     */
    public function show($id)
    {
        $user = $this->users->find($id);

        return view('user.profile', ['user' => $user]);
    }
}
```

В этом примере `UserController` должен извлекать пользователей из источника данных. Таким образом, мы **внедрим** службу, которая может извлекать пользователей. В этом контексте наш `UserRepository` скорее всего использует [Eloquent](../eloquent/eloquent.md) для получения информации о пользователе из базы данных. Однако, поскольку репозиторий инжектируется, мы можем легко поменять его на другую реализацию. Также мы можем легко "мокнуть", или создать фиктивную реализацию `UserRepository` при тестировании нашего приложения.

Глубокое понимание сервисного контейнера Laravel необходимо для создания мощного, крупного приложения, а также для внесения вклада в само ядро Laravel.

## Биндинг

### Основы биндинга

Почти все биндинги к сервис-контейнерам будут зарегистрированы в [сервис-провайдере](providers.md), поэтому большинство из этих примеров продемонстрируют использование контейнера в данном контексте.

{% hint style="info" %}
Нет необходимости привязывать классы к контейнеру, если они не зависят от каких-либо интерфейсов. Контейнер не нуждается в инструктаже о том, как строить эти объекты, так как он может автоматически разрешать эти объекты с помощью рефлексии.
{% endhint %}

**Simple Bindings**

Внутри поставщика услуг вы всегда имеете доступ к контейнеру через свойство `$this->app`. Мы можем зарегистрировать привязку, используя метод `bind`, передав имя класса или интерфейса, который мы хотим зарегистрировать вместе с `Closure`, который возвращает экземпляр класса:

```php
$this->app->bind('HelpSpot\API', function ($app) {
    return new \HelpSpot\API($app->make('HttpClient'));
});
```

Обратите внимание, что мы получаем сам контейнер в качестве аргумента в резолвер. Затем мы можем использовать контейнер для разрешения суб-зависимостей строящегося объекта.

**Биндинг синглтона**

Метод `singleton` привязывает к контейнеру класс или интерфейс, который должен быть разрешен только один раз. Как только singleton-привязка будет разрешена, тот же самый экземпляр объекта будет возвращен при последующих вызовах в контейнер:

```php
$this->app->singleton('HelpSpot\API', function ($app) {
    return new \HelpSpot\API($app->make('HttpClient'));
});
```

**Биндинг экземпляра**

Вы также можете привязать существующий экземпляр объекта к контейнеру с помощью метода `instance`. Данный экземпляр всегда будет возвращаться при последующих обращениях в контейнер:
You may also bind an existing object instance into the container using the `instance` method. The given instance will always be returned on subsequent calls into the container:

```php
$api = new \HelpSpot\API(new HttpClient);

$this->app->instance('HelpSpot\API', $api);
```

### Биндинг интерфейса к реализации

Очень мощной особенностью сервисного контейнера является его способность привязывать интерфейс к заданной реализации. Например, предположим, что у нас есть интерфейс `EventPusher` и реализация `RedisEventPusher`. После того, как мы закодировали нашу реализацию `RedisEventPusher` этого интерфейса, мы можем зарегистрировать его в сервисном контейнере таким образом: 

```php
$this->app->bind(
    'App\Contracts\EventPusher',
    'App\Services\RedisEventPusher'
);
```

Это выражение говорит контейнеру, что он должен внедрять `RedisEventPusher`, когда классу нужна реализация `EventPusher`. Теперь мы можем писать тайп-хинт на интерфейс `EventPusher` в конструкторе, или в любом другом месте, где зависимости инжектируются сервисным контейнером:

```php
use App\Contracts\EventPusher;

/**
 * Create a new class instance.
 *
 * @param  EventPusher  $pusher
 * @return void
 */
public function __construct(EventPusher $pusher)
{
    $this->pusher = $pusher;
}
```

### Контекстный биндинг

Иногда у вас может быть два класса, которые используют один и тот же интерфейс, но вы хотите внедрять различные реализации в каждый класс. Например, два контроллера могут зависеть от различных реализаций [контракта](contracts.md) `Illuminate\Contracts\Filesystem\Filesystem`. Ларавел предоставляет простой интерфейс для определения такого поведения:

```php
use App\Http\Controllers\PhotoController;
use App\Http\Controllers\UploadController;
use App\Http\Controllers\VideoController;
use Illuminate\Contracts\Filesystem\Filesystem;
use Illuminate\Support\Facades\Storage;

$this->app->when(PhotoController::class)
          ->needs(Filesystem::class)
          ->give(function () {
              return Storage::disk('local');
          });

$this->app->when([VideoController::class, UploadController::class])
          ->needs(Filesystem::class)
          ->give(function () {
              return Storage::disk('s3');
          });
```

### Биндинг примитивов

Иногда вы можете иметь класс, который получает некоторые вводимые классы, но также нуждается в внедрении примитивного значения, такого как целое число. Вы можете легко использовать контекстную привязку, чтобы ввести любое значение, которое может понадобиться вашему классу:

```php
$this->app->when('App\Http\Controllers\UserController')
          ->needs('$variableName')
          ->give($value);
```

Иногда класс может зависеть от массива тегированных экземпляров. Используя метод `giveTagged`, можно легко ввести все привязки контейнеров с помощью этого тега:

```php
$this->app->when(ReportAggregator::class)
    ->needs('$reports')
    ->giveTagged('reports');
```

### Binding Typed Variadics

Иногда можно иметь класс, который получает массив типизированных объектов с помощью конструктора variadic:

```php
class Firewall
{
    protected $logger;
    protected $filters;

    public function __construct(Logger $logger, Filter ...$filters)
    {
        $this->logger = $logger;
        $this->filters = $filters;
    }
}
```

Используя контекстную привязку, вы можете разрешить эту зависимость, предоставив метод `give` с Closure, который возвращает массив разрешенных экземпляров `Filter`:

```php
$this->app->when(Firewall::class)
          ->needs(Filter::class)
          ->give(function ($app) {
                return [
                    $app->make(NullFilter::class),
                    $app->make(ProfanityFilter::class),
                    $app->make(TooLongFilter::class),
                ];
          });
```

Для удобства, вы также можете просто предоставить массив имен классов, которые будут разрешаться контейнером всякий раз, когда `Firewall` нуждается в `Filter` экземплярах:

```php
$this->app->when(Firewall::class)
          ->needs(Filter::class)
          ->give([
              NullFilter::class,
              ProfanityFilter::class,
              TooLongFilter::class,
          ]);
```

**Variadic Tag Dependencies**

Иногда класс может иметь вариадическую зависимость, которая имеет тайп-хинт \(`Report ...$reports`\). Используя методы `needs` и `giveTagged`, можно легко ввести все привязки контейнеров с этим тегом для данной зависимости:

```php
$this->app->when(ReportAggregator::class)
    ->needs(Report::class)
    ->giveTagged('reports');
```

### Тегирование

Иногда может понадобиться решить все проблемы определенной "категории" привязки. Например, возможно, Вы строите агрегатор отчетов, который получает массив из множества различных реализаций интерфейса `Report`. После регистрации реализаций `Report`, Вы можете присвоить им тег, используя метод `tag`:

```php
$this->app->bind('SpeedReport', function () {
    //
});

$this->app->bind('MemoryReport', function () {
    //
});

$this->app->tag(['SpeedReport', 'MemoryReport'], 'reports');
```

После того, как сервисы будут тегированы, вы можете легко разрешить их все с помощью метода `tagged`:

```php
$this->app->bind('ReportAggregator', function ($app) {
    return new ReportAggregator($app->tagged('reports'));
});
```

### Расширенные биндинги

Метод `extend` позволяет модифицировать разрешенные сервисы. Например, когда сервис разрешен, вы можете запустить дополнительный код для украшения или настройки сервиса. Метод `extend` в качестве единственного аргумента принимает Closure, который должен возвращать модифицированный сервис. Closure получает разрешаемый сервис и экземпляр контейнера:

```php
$this->app->extend(Service::class, function ($service, $app) {
    return new DecoratedService($service);
});
```

## Разрешение

### **Метод `make`**

Вы можете использовать метод `make` для разрешения экземпляра класса из контейнера. Метод `make` принимает имя класса или интерфейса, который вы хотите разрешить:

```php
$api = $this->app->make('HelpSpot\API');
```

Если Вы находитесь в месте, где находится Ваш код, который не имеет доступа к переменной `$app`, Вы можете воспользоваться глобальным помощником `resolve`:

```php
$api = resolve('HelpSpot\API');
```

Если некоторые из зависимостей вашего класса не разрешимы через контейнер, вы можете внедрить их, передав в виде ассоциативного массива в метод `makeWith`:

```php
$api = $this->app->makeWith('HelpSpot\API', ['id' => 1]);
```

### **Автоматическое внедрение**

Или же, что немаловажно, можно "type-hint" зависимость в конструкторе класса, которая разрешается контейнером, в том числе в [контроллерах](.../the-basics/controllers.md), [слушателях событий](.../digging-deeper/events.md), [посредниках](.../the-basics/middleware.md), и многих других. Кроме того, вы можете type-hint зависимости в методе `handle` [очереди заданий](../digging-deeper/queues.md). На практике, это то, как большинство ваших объектов должны быть решены контейнером.

Например, вы можете type-hint в конструкторе контроллера подсказку к репозиторию, определённому вашим приложением. Хранилище будет автоматически разрешено и внедрено в класс:

```php
<?php

namespace App\Http\Controllers;

use App\Users\Repository as UserRepository;

class UserController extends Controller
{
    /**
     * The user repository instance.
     */
    protected $users;

    /**
     * Create a new controller instance.
     *
     * @param  UserRepository  $users
     * @return void
     */
    public function __construct(UserRepository $users)
    {
        $this->users = $users;
    }

    /**
     * Show the user with the given ID.
     *
     * @param  int  $id
     * @return Response
     */
    public function show($id)
    {
        //
    }
}
```

## События контейнера

Сервисный контейнер запускает событие каждый раз при разрешении объекта. Вы можете прослушать это событие, используя метод `resolving`:

```php
$this->app->resolving(function ($object, $app) {
    // Called when container resolves object of any type...
});

$this->app->resolving(\HelpSpot\API::class, function ($api, $app) {
    // Called when container resolves objects of type "HelpSpot\API"...
});
```

Как видите, разрешаемый объект будет передан функции обратного вызова, что позволит вам установить любые дополнительные свойства на объекте до того, как он будет передан его потребителю.

## PSR-11

Сервисный контейнер Laravel реализует интерфейс [PSR-11](https://github.com/php-fig/fig-standards/blob/master/accepted/PSR-11-container.md). Поэтому, чтобы получить экземпляр контейнера Laravel, Вы можете указать type-hint на интерфейс контейнера PSR-11:

```php
use Psr\Container\ContainerInterface;

Route::get('/', function (ContainerInterface $container) {
    $service = $container->get('Service');

    //
});
```

Исключение бросается, если заданный идентификатор не может быть разрешен. Исключением будет экземпляр `Psr\Container\NotFoundExceptionInterface`, если идентификатор никогда не был связан. Если идентификатор был связан, но не смог быть разрешен, то будет выброшен экземпляр `Psr\Container\ContainerExceptionInterface`.

