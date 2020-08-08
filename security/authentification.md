# Аутентификация

## Вступление

{% hint style="info" %}
__Хотите быстро начать?__  
Установите с помощью Composer пакет laravel/ui и выполните команду php artisan ui vue --auth в свежем приложении Laravel. После миграции базы данных перейдите в браузер на `http://your-app.test/register`. Эти команды позаботятся о скелете всей системы аутентификации!
{% endhint %}

Ларавел делает внедрение аутентификации очень простым. На самом деле, практически все настроено для вас из коробки. Файл конфигурации аутентификации расположен по адресу `config/auth.php`. Он содержит несколько хорошо документированных опций для настройки поведения служб аутентификации.

В основе "Ларавел" аутентификации лежат "охранники (guards)" и "провайдеры (providers)". Охранники определяют способ аутентификации пользователей для каждого запроса. Например, Laravel поставляется с охранником `session`, который поддерживает состояние, используя сессии и cookies.

Провайдеры определяют, как пользователи извлекаются из вашего постоянного хранилища. Laravel поставляется с поддержкой извлечения пользователей с помощью Eloquent и конструктора запросов к базе данных. Тем не менее, вы можете определить дополнительных провайдеров, необходимых для вашего приложения.

Не беспокойтесь, если сейчас все это звучит запутанно! Многим приложениям никогда не придется изменять конфигурацию аутентификации по умолчанию.

### О базе данных

По умолчанию Laravel размещает [Модель Eloquent](../eloquent/getting-started.md) `App\User` в каталоге `app`. Эта модель может быть использована с драйвером аутентификации Eloquent по умолчанию. Если ваше приложение не использует Eloquent, вы можете использовать драйвер аутентификации `database`, который использует конструктор запросов Laravel.

При построении схемы базы данных для модели `App\User` убедитесь, что столбец пароля имеет длину не менее 60 символов. Хорошим выбором будет сохранение длины столбца строки по умолчанию в 255 символов.

Также вы должны убедиться, что ваша таблица `users` \(или аналогичная\) содержит строковое поле строку `remember_token`, состоящую из 100 символов. Это поле должно быть nullable. Оно будет использоваться для хранения токена для пользователей, которые выбирают опцию "запомнить меня" при входе в приложение.

## Аутентификация: быстрый старт

### Маршрутизация

Пакет `laravel/ui` предоставляет быстрый способ задания всех маршрутов и представлений, необходимых для аутентификации с помощью нескольких простых команд:

```text
composer require laravel/ui

php artisan ui vue --auth
```

Эта команда должна использоваться на вновь установленных приложениях и установит лейаут, представления регистрации и входа, а также маршруты для всех конечных точек аутентификации. Также будет сгенерирован `HomeController`, который будет обрабатывать запросы после входа на панель управления вашего приложения.

Пакет `laravel/ui` также генерирует несколько предустановленных контроллеров аутентификации, которые находятся в пространстве имен `App\Http\Controllers\Auth`. Пакет `RegisterController` обрабатывает регистрацию новых пользователей, `LoginController` — аутентификацию, `ForgotPasswordController` — ссылки на электронную почту для сброса паролей, а `ResetPasswordController` содержит логику сброса паролей. Каждый из этих контроллеров использует свой признак для включения необходимых методов. Для многих приложений, Вам не нужно будет изменять эти контроллеры вообще.

{% hint style="info" %}
 Если ваше приложение не нуждается в регистрации, вы можете отключить его, удалив только что созданный `RegisterController` и изменив декларацию маршрута: `Auth::routes(['register' => false]);`.
{% endhint %}

**Создание приложений, включающих аутентификацию**

Если вы запускаете совершенно новое приложение и хотите сразу включить обвязку аутентификации, то можете использовать директиву `--auth` при создании. Эта команда создаст новое приложение со всей обвязкой для аутентификации, скомпилированной и установленной:

```text
laravel new blog --auth
```

### Представления

Как упоминалось в предыдущем разделе, команда `php artisan ui vue --auth` из пакета `laravel/ui` создаст все представления, необходимые для аутентификации, и поместит их в каталог `resources/views/auth`.

Команда `ui` также создаст каталог `resources/views/layouts`, содержащий базовый макет для вашего приложения. Все эти представления используют фреймворк Bootstrap CSS, но вы можете настроить их, как пожелаете.

### Аутентификация

Теперь, когда вы настроили маршруты и представления для входящих в комплект контроллеров аутентификации, вы готовы регистрировать и аутентифицировать новых пользователей! Вы можете получить доступ к приложению в браузере, так как контроллеры аутентификации уже содержат логику \(через их трейты\) для аутентификации существующих пользователей и хранения новых пользователей в базе данных.

**Настройка пути**

Когда пользователь успешно аутентифицируется, он будет перенаправлен в URI `/home`. Вы можете настроить путь редиректа после аутентификации, используя константу `HOME`, определенную в вашем `RouteServiceProvider`:

```text
public const HOME = '/home';
```

Если вам нужна более надежная настройка ответа, возвращаемого при аутентификации пользователя, Laravel предоставляет пустой метод `authenticated(Request $request, $user)` в трейте `AuthenticatesUsers`. Этот трейт используется классом `LoginController`, который устанавливается в ваше приложение при использовании пакета `laravel/ui`. Таким образом, Вы можете определить свой собственный метод `authenticated` в классе `LoginController`: 

```text
/**
 * The user has been authenticated.
 *
 * @param  \Illuminate\Http\Request  $request
 * @param  mixed  $user
 * @return mixed
 */
protected function authenticated(Request $request, $user)
{
    return response([
        //
    ]);
}
```

**Настройка имени пользователя**

По умолчанию, Laravel использует поле `email` для аутентификации. Если вы хотите изменить это, то можете определить метод `username` в классе `LoginController`:

```text
public function username()
{
    return 'username';
}
```

**Настройка охранника**

Вы также можете изменить охранника \("guard"\), который используется для аутентификации и регистрации пользователей. Для начала определите метод `duard` в контроллерах `LoginController`, `RegisterController` и `ResetPasswordController`. Метод должен возвращать экземпляр охраннника:

```text
use Illuminate\Support\Facades\Auth;

protected function guard()
{
    return Auth::guard('guard-name');
}
```

**Настройка валидации и хранилища**

Для изменения полей формы, которые требуются при регистрации нового пользователя в вашем приложении, или для настройки способа хранения новых пользователей в вашей базе данных, вы можете изменить класс `RegisterController`. Этот класс отвечает за проверку и создание новых пользователей Вашего приложения.

Метод `validator` класса `RegisterController` содержит правила валидации для новых пользователей приложения. Вы можете изменять этот метод по своему усмотрению.

Метод `create` класса `RegisterController` отвечает за создание новых записей `App\User` в вашей базе данных с помощью [Eloquent ORM](.../eloquent/getting-started.md). Вы можете свободно модифицировать этот метод в соответствии с потребностями вашей базы данных.

### Retrieving The Authenticated User

You may access the authenticated user via the `Auth` facade:

```text
use Illuminate\Support\Facades\Auth;

// Get the currently authenticated user...
$user = Auth::user();

// Get the currently authenticated user's ID...
$id = Auth::id();
```

Alternatively, once a user is authenticated, you may access the authenticated user via an `Illuminate\Http\Request` instance. Remember, type-hinted classes will automatically be injected into your controller methods:

```text
<?php

namespace App\Http\Controllers;

use Illuminate\Http\Request;

class ProfileController extends Controller
{
    /**
     * Update the user's profile.
     *
     * @param  Request  $request
     * @return Response
     */
    public function update(Request $request)
    {
        // $request->user() returns an instance of the authenticated user...
    }
}
```

**Determining If The Current User Is Authenticated**

To determine if the user is already logged into your application, you may use the `check` method on the `Auth` facade, which will return `true` if the user is authenticated:

```text
use Illuminate\Support\Facades\Auth;

if (Auth::check()) {
    // The user is logged in...
}
```

{% hint style="info" %}


> Even though it is possible to determine if a user is authenticated using the `check` method, you will typically use a middleware to verify that the user is authenticated before allowing the user access to certain routes / controllers. To learn more about this, check out the documentation on [protecting routes](authentification.md#zashishennye-marshruty).
{% endhint %}

### Защищенные маршруты

[Route middleware](../the-basics/middleware.md) can be used to only allow authenticated users to access a given route. Laravel ships with an `auth` middleware, which is defined at `Illuminate\Auth\Middleware\Authenticate`. Since this middleware is already registered in your HTTP kernel, all you need to do is attach the middleware to a route definition:

```text
Route::get('profile', function () {
    // Only authenticated users may enter...
})->middleware('auth');
```

If you are using [controllers](../the-basics/controllers.md), you may call the `middleware` method from the controller's constructor instead of attaching it in the route definition directly:

```text
public function __construct()
{
    $this->middleware('auth');
}
```

**Redirecting Unauthenticated Users**

When the `auth` middleware detects an unauthorized user, it will redirect the user to the `login` [named route](../the-basics/routing.md#imenovannye-marshruty). You may modify this behavior by updating the `redirectTo` function in your `app/Http/Middleware/Authenticate.php` file:

```text
/**
 * Get the path the user should be redirected to.
 *
 * @param  \Illuminate\Http\Request  $request
 * @return string
 */
protected function redirectTo($request)
{
    return route('login');
}
```

**Specifying A Guard**

When attaching the `auth` middleware to a route, you may also specify which guard should be used to authenticate the user. The guard specified should correspond to one of the keys in the `guards` array of your `auth.php` configuration file:

```text
public function __construct()
{
    $this->middleware('auth:api');
}
```

### Password Confirmation

Sometimes, you may wish to require the user to confirm their password before accessing a specific area of your application. For example, you may require this before the user modifies any billing settings within the application.

To accomplish this, Laravel provides a `password.confirm` middleware. Attaching the `password.confirm` middleware to a route will redirect users to a screen where they need to confirm their password before they can continue:

```text
Route::get('/settings/security', function () {
    // Users must confirm their password before continuing...
})->middleware(['auth', 'password.confirm']);
```

After the user has successfully confirmed their password, the user is redirected to the route they originally tried to access. By default, after confirming their password, the user will not have to confirm their password again for three hours. You are free to customize the length of time before the user must re-confirm their password using the `auth.password_timeout` configuration option.

### Login Throttling

If you are using Laravel's built-in `LoginController` class, the `Illuminate\Foundation\Auth\ThrottlesLogins` trait will already be included in your controller. By default, the user will not be able to login for one minute if they fail to provide the correct credentials after several attempts. The throttling is unique to the user's username / e-mail address and their IP address.

## Manually Authenticating Users

Note that you are not required to use the authentication controllers included with Laravel. If you choose to remove these controllers, you will need to manage user authentication using the Laravel authentication classes directly. Don't worry, it's a cinch!

We will access Laravel's authentication services via the `Auth` [facade](../architecture-concepts/facades.md), so we'll need to make sure to import the `Auth` facade at the top of the class. Next, let's check out the `attempt` method:

```text
<?php

namespace App\Http\Controllers;

use Illuminate\Http\Request;
use Illuminate\Support\Facades\Auth;

class LoginController extends Controller
{
    /**
     * Handle an authentication attempt.
     *
     * @param  \Illuminate\Http\Request $request
     *
     * @return Response
     */
    public function authenticate(Request $request)
    {
        $credentials = $request->only('email', 'password');

        if (Auth::attempt($credentials)) {
            // Authentication passed...
            return redirect()->intended('dashboard');
        }
    }
}
```

The `attempt` method accepts an array of key / value pairs as its first argument. The values in the array will be used to find the user in your database table. So, in the example above, the user will be retrieved by the value of the `email` column. If the user is found, the hashed password stored in the database will be compared with the `password` value passed to the method via the array. You should not hash the password specified as the `password` value, since the framework will automatically hash the value before comparing it to the hashed password in the database. If the two hashed passwords match an authenticated session will be started for the user.

The `attempt` method will return `true` if authentication was successful. Otherwise, `false` will be returned.

The `intended` method on the redirector will redirect the user to the URL they were attempting to access before being intercepted by the authentication middleware. A fallback URI may be given to this method in case the intended destination is not available.

**Specifying Additional Conditions**

If you wish, you may also add extra conditions to the authentication query in addition to the user's e-mail and password. For example, we may verify that user is marked as "active":

```text
if (Auth::attempt(['email' => $email, 'password' => $password, 'active' => 1])) {
    // The user is active, not suspended, and exists.
}
```

{% hint style="warning" %}
1
{% endhint %}

> ![](https://laravel.com/img/callouts/exclamation.min.svg)
>
> In these examples, `email` is not a required option, it is merely used as an example. You should use whatever column name corresponds to a "username" in your database.

**Accessing Specific Guard Instances**

You may specify which guard instance you would like to utilize using the `guard` method on the `Auth` facade. This allows you to manage authentication for separate parts of your application using entirely separate authenticatable models or user tables.

The guard name passed to the `guard` method should correspond to one of the guards configured in your `auth.php` configuration file:

```text
if (Auth::guard('admin')->attempt($credentials)) {
    //
}
```

**Logging Out**

To log users out of your application, you may use the `logout` method on the `Auth` facade. This will clear the authentication information in the user's session:

```text
Auth::logout();
```

### Remembering Users

If you would like to provide "remember me" functionality in your application, you may pass a boolean value as the second argument to the `attempt` method, which will keep the user authenticated indefinitely, or until they manually logout. Your `users` table must include the string `remember_token` column, which will be used to store the "remember me" token.

{% hint style="info" %}
1
{% endhint %}

```text
if (Auth::attempt(['email' => $email, 'password' => $password], $remember)) {
    // The user is being remembered...
}
```

> ![](https://laravel.com/img/callouts/lightbulb.min.svg)
>
> If you are using the built-in `LoginController` that is shipped with Laravel, the proper logic to "remember" users is already implemented by the traits used by the controller.

If you are "remembering" users, you may use the `viaRemember` method to determine if the user was authenticated using the "remember me" cookie:

```text
if (Auth::viaRemember()) {
    //
}
```

### Other Authentication Methods

**Authenticate A User Instance**

If you need to log an existing user instance into your application, you may call the `login` method with the user instance. The given object must be an implementation of the `Illuminate\Contracts\Auth\Authenticatable` [contract](../architecture-concepts/contracts.md). The `App\User` model included with Laravel already implements this interface:

```text
Auth::login($user);

// Login and "remember" the given user...
Auth::login($user, true);
```

You may specify the guard instance you would like to use:

```text
Auth::guard('admin')->login($user);
```

**Authenticate A User By ID**

To log a user into the application by their ID, you may use the `loginUsingId` method. This method accepts the primary key of the user you wish to authenticate:

```text
Auth::loginUsingId(1);

// Login and "remember" the given user...
Auth::loginUsingId(1, true);
```

**Authenticate A User Once**

You may use the `once` method to log a user into the application for a single request. No sessions or cookies will be utilized, which means this method may be helpful when building a stateless API:

```text
if (Auth::once($credentials)) {
    //
}
```

## HTTP Basic Authentication

[HTTP Basic Authentication](https://en.wikipedia.org/wiki/Basic_access_authentication) provides a quick way to authenticate users of your application without setting up a dedicated "login" page. To get started, attach the `auth.basic` [middleware](../the-basics/middleware.md) to your route. The `auth.basic` middleware is included with the Laravel framework, so you do not need to define it:

```text
Route::get('profile', function () {
    // Only authenticated users may enter...
})->middleware('auth.basic');
```

Once the middleware has been attached to the route, you will automatically be prompted for credentials when accessing the route in your browser. By default, the `auth.basic` middleware will use the `email` column on the user record as the "username".

**A Note On FastCGI**

If you are using PHP FastCGI, HTTP Basic authentication may not work correctly out of the box. The following lines should be added to your `.htaccess` file:

```text
RewriteCond %{HTTP:Authorization} ^(.+)$
RewriteRule .* - [E=HTTP_AUTHORIZATION:%{HTTP:Authorization}]
```

### Stateless HTTP Basic Authentication

You may also use HTTP Basic Authentication without setting a user identifier cookie in the session, which is particularly useful for API authentication. To do so, [define a middleware](../the-basics/middleware.md) that calls the `onceBasic` method. If no response is returned by the `onceBasic` method, the request may be passed further into the application:

```text
<?php

namespace App\Http\Middleware;

use Illuminate\Support\Facades\Auth;

class AuthenticateOnceWithBasicAuth
{
    /**
     * Handle an incoming request.
     *
     * @param  \Illuminate\Http\Request  $request
     * @param  \Closure  $next
     * @return mixed
     */
    public function handle($request, $next)
    {
        return Auth::onceBasic() ?: $next($request);
    }

}
```

Next, [register the route middleware](../the-basics/middleware.md#registering-middleware) and attach it to a route:

```text
Route::get('api/user', function () {
    // Only authenticated users may enter...
})->middleware('auth.basic.once');
```

## Logging Out

To manually log users out of your application, you may use the `logout` method on the `Auth` facade. This will clear the authentication information in the user's session:

```text
use Illuminate\Support\Facades\Auth;

Auth::logout();
```

### Invalidating Sessions On Other Devices

Laravel also provides a mechanism for invalidating and "logging out" a user's sessions that are active on other devices without invalidating the session on their current device. This feature is typically utilized when a user is changing or updating their password and you would like to invalidate sessions on other devices while keeping the current device authenticated.

Before getting started, you should make sure that the `Illuminate\Session\Middleware\AuthenticateSession` middleware is present and un-commented in your `app/Http/Kernel.php` class' `web` middleware group:

```text
'web' => [
    // ...
    \Illuminate\Session\Middleware\AuthenticateSession::class,
    // ...
],
```

Then, you may use the `logoutOtherDevices` method on the `Auth` facade. This method requires the user to provide their current password, which your application should accept through an input form:

```text
use Illuminate\Support\Facades\Auth;

Auth::logoutOtherDevices($password);
```

When the `logoutOtherDevices` method is invoked, the user's other sessions will be invalidated entirely, meaning they will be "logged out" of all guards they were previously authenticated by.

{% hint style="warning" %}
When
{% endhint %}

> ![](https://laravel.com/img/callouts/exclamation.min.svg)
>
> When using the `AuthenticateSession` middleware in combination with a custom route name for the `login` route, you must override the `unauthenticated` method on your application's exception handler to properly redirect users to your login page.

## [Adding Custom Guards](https://laravel.com/docs/7.x/authentication#adding-custom-guards)

You may define your own authentication guards using the `extend` method on the `Auth` facade. You should place this call to `extend` within a [service provider](../architecture-concepts/providers.md). Since Laravel already ships with an `AuthServiceProvider`, we can place the code in that provider:

```text
<?php

namespace App\Providers;

use App\Services\Auth\JwtGuard;
use Illuminate\Foundation\Support\Providers\AuthServiceProvider as ServiceProvider;
use Illuminate\Support\Facades\Auth;

class AuthServiceProvider extends ServiceProvider
{
    /**
     * Register any application authentication / authorization services.
     *
     * @return void
     */
    public function boot()
    {
        $this->registerPolicies();

        Auth::extend('jwt', function ($app, $name, array $config) {
            // Return an instance of Illuminate\Contracts\Auth\Guard...

            return new JwtGuard(Auth::createUserProvider($config['provider']));
        });
    }
}
```

As you can see in the example above, the callback passed to the `extend` method should return an implementation of `Illuminate\Contracts\Auth\Guard`. This interface contains a few methods you will need to implement to define a custom guard. Once your custom guard has been defined, you may use this guard in the `guards` configuration of your `auth.php` configuration file:

```text
'guards' => [
    'api' => [
        'driver' => 'jwt',
        'provider' => 'users',
    ],
],
```

### Closure Request Guards

The simplest way to implement a custom, HTTP request based authentication system is by using the `Auth::viaRequest` method. This method allows you to quickly define your authentication process using a single Closure.

To get started, call the `Auth::viaRequest` method within the `boot` method of your `AuthServiceProvider`. The `viaRequest` method accepts an authentication driver name as its first argument. This name can be any string that describes your custom guard. The second argument passed to the method should be a Closure that receives the incoming HTTP request and returns a user instance or, if authentication fails, `null`:

```text
use App\User;
use Illuminate\Http\Request;
use Illuminate\Support\Facades\Auth;

/**
 * Register any application authentication / authorization services.
 *
 * @return void
 */
public function boot()
{
    $this->registerPolicies();

    Auth::viaRequest('custom-token', function ($request) {
        return User::where('token', $request->token)->first();
    });
}
```

Once your custom authentication driver has been defined, you use it as a driver within `guards` configuration of your `auth.php` configuration file:

```text
'guards' => [
    'api' => [
        'driver' => 'custom-token',
    ],
],
```

## Adding Custom User Providers

If you are not using a traditional relational database to store your users, you will need to extend Laravel with your own authentication user provider. We will use the `provider` method on the `Auth` facade to define a custom user provider:

```text
<?php

namespace App\Providers;

use App\Extensions\RiakUserProvider;
use Illuminate\Foundation\Support\Providers\AuthServiceProvider as ServiceProvider;
use Illuminate\Support\Facades\Auth;

class AuthServiceProvider extends ServiceProvider
{
    /**
     * Register any application authentication / authorization services.
     *
     * @return void
     */
    public function boot()
    {
        $this->registerPolicies();

        Auth::provider('riak', function ($app, array $config) {
            // Return an instance of Illuminate\Contracts\Auth\UserProvider...

            return new RiakUserProvider($app->make('riak.connection'));
        });
    }
}
```

After you have registered the provider using the `provider` method, you may switch to the new user provider in your `auth.php` configuration file. First, define a `provider` that uses your new driver:

```text
'providers' => [
    'users' => [
        'driver' => 'riak',
    ],
],
```

Finally, you may use this provider in your `guards` configuration:

```text
'guards' => [
    'web' => [
        'driver' => 'session',
        'provider' => 'users',
    ],
],
```

### The User Provider Contract

The `Illuminate\Contracts\Auth\UserProvider` implementations are only responsible for fetching a `Illuminate\Contracts\Auth\Authenticatable` implementation out of a persistent storage system, such as MySQL, Riak, etc. These two interfaces allow the Laravel authentication mechanisms to continue functioning regardless of how the user data is stored or what type of class is used to represent it.

Let's take a look at the `Illuminate\Contracts\Auth\UserProvider` contract:

```text
<?php

namespace Illuminate\Contracts\Auth;

interface UserProvider
{
    public function retrieveById($identifier);
    public function retrieveByToken($identifier, $token);
    public function updateRememberToken(Authenticatable $user, $token);
    public function retrieveByCredentials(array $credentials);
    public function validateCredentials(Authenticatable $user, array $credentials);
}
```

The `retrieveById` function typically receives a key representing the user, such as an auto-incrementing ID from a MySQL database. The `Authenticatable` implementation matching the ID should be retrieved and returned by the method.

The `retrieveByToken` function retrieves a user by their unique `$identifier` and "remember me" `$token`, stored in a field `remember_token`. As with the previous method, the `Authenticatable` implementation should be returned.

The `updateRememberToken` method updates the `$user` field `remember_token` with the new `$token`. A fresh token is assigned on a successful "remember me" login attempt or when the user is logging out.

The `retrieveByCredentials` method receives the array of credentials passed to the `Auth::attempt` method when attempting to sign into an application. The method should then "query" the underlying persistent storage for the user matching those credentials. Typically, this method will run a query with a "where" condition on `$credentials['username']`. The method should then return an implementation of `Authenticatable`. **This method should not attempt to do any password validation or authentication.**

The `validateCredentials` method should compare the given `$user` with the `$credentials` to authenticate the user. For example, this method should probably use `Hash::check` to compare the value of `$user->getAuthPassword()` to the value of `$credentials['password']`. This method should return `true` or `false` indicating on whether the password is valid.

### The Authenticatable Contract

Now that we have explored each of the methods on the `UserProvider`, let's take a look at the `Authenticatable` contract. Remember, the provider should return implementations of this interface from the `retrieveById`, `retrieveByToken`, and `retrieveByCredentials` methods:

```text
<?php

namespace Illuminate\Contracts\Auth;

interface Authenticatable
{
    public function getAuthIdentifierName();
    public function getAuthIdentifier();
    public function getAuthPassword();
    public function getRememberToken();
    public function setRememberToken($value);
    public function getRememberTokenName();
}
```

This interface is simple. The `getAuthIdentifierName` method should return the name of the "primary key" field of the user and the `getAuthIdentifier` method should return the "primary key" of the user. In a MySQL back-end, again, this would be the auto-incrementing primary key. The `getAuthPassword` should return the user's hashed password. This interface allows the authentication system to work with any User class, regardless of what ORM or storage abstraction layer you are using. By default, Laravel includes a `User` class in the `app` directory which implements this interface, so you may consult this class for an implementation example.

## Events

Laravel raises a variety of [events](../digging-deeper/events.md) during the authentication process. You may attach listeners to these events in your `EventServiceProvider`:

```text
/**
 * The event listener mappings for the application.
 *
 * @var array
 */
protected $listen = [
    'Illuminate\Auth\Events\Registered' => [
        'App\Listeners\LogRegisteredUser',
    ],

    'Illuminate\Auth\Events\Attempting' => [
        'App\Listeners\LogAuthenticationAttempt',
    ],

    'Illuminate\Auth\Events\Authenticated' => [
        'App\Listeners\LogAuthenticated',
    ],

    'Illuminate\Auth\Events\Login' => [
        'App\Listeners\LogSuccessfulLogin',
    ],

    'Illuminate\Auth\Events\Failed' => [
        'App\Listeners\LogFailedLogin',
    ],

    'Illuminate\Auth\Events\Validated' => [
        'App\Listeners\LogValidated',
    ],

    'Illuminate\Auth\Events\Verified' => [
        'App\Listeners\LogVerified',
    ],

    'Illuminate\Auth\Events\Logout' => [
        'App\Listeners\LogSuccessfulLogout',
    ],

    'Illuminate\Auth\Events\CurrentDeviceLogout' => [
        'App\Listeners\LogCurrentDeviceLogout',
    ],

    'Illuminate\Auth\Events\OtherDeviceLogout' => [
        'App\Listeners\LogOtherDeviceLogout',
    ],

    'Illuminate\Auth\Events\Lockout' => [
        'App\Listeners\LogLockout',
    ],

    'Illuminate\Auth\Events\PasswordReset' => [
        'App\Listeners\LogPasswordReset',
    ],
];
```

