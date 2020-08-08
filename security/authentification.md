# Аутентификация

## Вступление

{% hint style="info" %}
**Хотите быстро начать?**  
Установите с помощью Composer пакет laravel/ui и выполните команду `php artisan ui vue --auth` в свежем приложении Laravel. После миграции базы данных перейдите в браузер на `http://your-app.test/register`. Эти команды позаботятся о скелете всей системы аутентификации!
{% endhint %}

Ларавел делает внедрение аутентификации очень простым. На самом деле, практически все настроено для вас из коробки. Файл конфигурации аутентификации расположен по адресу `config/auth.php`. Он содержит несколько хорошо документированных опций для настройки поведения служб аутентификации.

В основе "Ларавел" аутентификации лежат "охранники \(guards\)" и "провайдеры \(providers\)". Охранники определяют способ аутентификации пользователей для каждого запроса. Например, Laravel поставляется с охранником `session`, который поддерживает состояние, используя сессии и cookies.

Провайдеры определяют, как пользователи извлекаются из вашего постоянного хранилища. Laravel поставляется с поддержкой извлечения пользователей с помощью Eloquent и конструктора запросов к базе данных. Тем не менее, вы можете определить дополнительных провайдеров, необходимых для вашего приложения.

Не беспокойтесь, если сейчас все это звучит запутанно! Многим приложениям никогда не придется изменять конфигурацию аутентификации по умолчанию.

### О базе данных

По умолчанию Laravel размещает [Модель Eloquent](../eloquent/getting-started.md) `App\User` в каталоге `app`. Эта модель может быть использована с драйвером аутентификации Eloquent по умолчанию. Если ваше приложение не использует Eloquent, вы можете использовать драйвер аутентификации `database`, который использует конструктор запросов Laravel.

При построении схемы базы данных для модели `App\User` убедитесь, что столбец пароля имеет длину не менее 60 символов. Хорошим выбором будет сохранение длины столбца строки по умолчанию в 255 символов.

Также вы должны убедиться, что ваша таблица `users` \(или аналогичная\) содержит строковое поле строку `remember_token`, состоящую из 100 символов. Это поле должно быть nullable. Оно будет использоваться для хранения токена для пользователей, которые выбирают опцию "запомнить меня" при входе в приложение.

## Аутентификация: быстрый старт

### Маршрутизация

Пакет `laravel/ui` предоставляет быстрый способ задания всех маршрутов и представлений, необходимых для аутентификации с помощью нескольких простых команд:

```bash
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

```bash
laravel new blog --auth
```

### Представления

Как упоминалось в предыдущем разделе, команда `php artisan ui vue --auth` из пакета `laravel/ui` создаст все представления, необходимые для аутентификации, и поместит их в каталог `resources/views/auth`.

Команда `ui` также создаст каталог `resources/views/layouts`, содержащий базовый макет для вашего приложения. Все эти представления используют фреймворк Bootstrap CSS, но вы можете настроить их, как пожелаете.

### Аутентификация

Теперь, когда вы настроили маршруты и представления для входящих в комплект контроллеров аутентификации, вы готовы регистрировать и аутентифицировать новых пользователей! Вы можете получить доступ к приложению в браузере, так как контроллеры аутентификации уже содержат логику \(через их трейты\) для аутентификации существующих пользователей и хранения новых пользователей в базе данных.

**Настройка пути**

Когда пользователь успешно аутентифицируется, он будет перенаправлен в URI `/home`. Вы можете настроить путь редиректа после аутентификации, используя константу `HOME`, определенную в вашем `RouteServiceProvider`:

```php
public conpst HOME = '/home';
```

Если вам нужна более тонкая настройка ответа, возвращаемого при аутентификации пользователя, Laravel предоставляет пустой метод `authenticated(Request $request, $user)` в трейте `AuthenticatesUsers`. Этот трейт используется классом `LoginController`, который устанавливается в ваше приложение при использовании пакета `laravel/ui`. Таким образом, Вы можете определить свой собственный метод `authenticated` в классе `LoginController`:

```php
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

```php
public function username()
{
    return 'username';
}
```

**Настройка охранника**

Вы также можете изменить охранника \("guard"\), который используется для аутентификации и регистрации пользователей. Для начала определите метод `duard` в контроллерах `LoginController`, `RegisterController` и `ResetPasswordController`. Метод должен возвращать экземпляр охраннника:

```php
use Illuminate\Support\Facades\Auth;

protected function guard()
{
    return Auth::guard('guard-name');
}
```

**Настройка валидации и хранилища**

Для изменения полей формы, которые требуются при регистрации нового пользователя в вашем приложении, или для настройки способа хранения новых пользователей в вашей базе данных, вы можете изменить класс `RegisterController`. Этот класс отвечает за проверку и создание новых пользователей Вашего приложения.

Метод `validator` класса `RegisterController` содержит правила валидации для новых пользователей приложения. Вы можете изменять этот метод по своему усмотрению.

Метод `create` класса `RegisterController` отвечает за создание новых записей `App\User` в вашей базе данных с помощью [Eloquent ORM](https://github.com/delphinpro/laravel-ru/tree/64b8c8bfab1f58e5d4b59e9575dbebc0b0462765/security/.../eloquent/getting-started.md). Вы можете свободно модифицировать этот метод в соответствии с потребностями вашей базы данных.

### Получение аутентифицированного пользователя

Вы можете получить доступ к аутентифицированному пользователю через фасад `Auth`:

```php
use Illuminate\Support\Facades\Auth;

// Get the currently authenticated user...
$user = Auth::user();

// Get the currently authenticated user's ID...
$id = Auth::id();
```

В качестве альтернативы, после того, как пользователь аутентифицирован, вы можете получить доступ к нему через экземпляр `Illuminate\Http\Request`. Помните, что классы с type-hint будут автоматически вставляться в методы вашего контроллера:

```php
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

**Определение, является ли текущий пользователь аутентифицированным.**

Чтобы определить, залогинен ли пользователь в приложении, вы можете использовать метод `check` фасада `Auth`, который вернет `true`, если пользователь аутентифицирован:

```php
use Illuminate\Support\Facades\Auth;

if (Auth::check()) {
    // Пользователь залогинен...
}
```

{% hint style="info" %}
Несмотря на то, что можно определить, является ли пользователь аутентифицированным с помощью метода `check`, вы обычно будете использовать посредика для проверки подлинности пользователя, прежде чем разрешить ему доступ к определенным маршрутам / контроллерам. Чтобы узнать больше об этом, ознакомьтесь с документацией по \[защите маршрутов\] \(authentictification.md\#zashishennye-marshruty\).
{% endhint %}

### Защищенные маршруты

[Посредник в маршрутах](../the-basics/middleware.md) может использоваться только для того, чтобы разрешить авторизованным пользователям доступ к заданному маршруту. Laravel поставляется с посредником `auth`, который определяется в `Illuminate\Auth\Middleware\Authenticate`. Так как этот посредник уже зарегистрирован в HTTP ядре, все, что вам нужно сделать, это указать его в определении маршрута:

```php
Route::get('profile', function () {
    // Only authenticated users may enter...
})->middleware('auth');
```

Если Вы используете [контроллеры](../the-basics/controllers.md), то можете вызвать метод `middleware` напрямую из конструктора контроллера вместо того, чтобы указывать его в определении маршрута:

```php
public function __construct()
{
    $this->middleware('auth');
}
```

**Перенаправление неавторизованных пользователей**

Когда посредник `auth` обнаруживает неавторизованного пользователя, оно перенаправляет пользователя на [именованный маршрут](https://github.com/delphinpro/laravel-ru/tree/23145faa5557cb82ae9c1b23ff97487d9737edca/security/.../the-basics/routing.md#imenovannye-marshruty) `login`. Вы можете изменить это поведение, обновив функцию `redirectTo` в файле `app/Http/Middleware/Authenticate.php`:

```php
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

**Определение охранника \(guard\)**

При подключении посредника `auth` к маршруту, вы также можете указать, какой защитник должен использоваться для аутентификации пользователя. Указанный защитник должен соответствовать одному из ключей в массиве `guards` конфигурационного файла `auth.php`:

```php
public function __construct()
{
    $this->middleware('auth:api');
}
```

### Подтверждение пароля

Иногда может возникнуть необходимость потребовать от пользователя подтвердить свой пароль перед тем, как получить доступ к определенной области применения. Например, это может потребоваться до того, как пользователь изменит какие-либо параметры выставления счетов в приложении.

Для этого Laravel предоставляет посредника `password.confirm`. Включение посредника `password.confirm` в маршрут перенаправит пользователей на экран, где они должны будут подтвердить свой пароль, прежде чем смогут продолжить работу:

```php
Route::get('/settings/security', function () {
    // Users must confirm their password before continuing...
})->middleware(['auth', 'password.confirm']);
```

После того, как пользователь успешно подтвердил свой пароль, он перенаправляется на маршрут, к которому изначально пытался получить доступ. По умолчанию, после подтверждения пароля, пользователю не придется подтверждать свой пароль еще раз в течение трех часов. Вы можете настроить время, в течение которого пользователь должен будет повторно подтвердить пароль с помощью опции `auth.password_timeout`.

### Ограничение входа

Если вы используете встроенный в Laravel класс `LoginController`, то трейт `Illuminate\Foundation\Auth\ThrottlesLogins` уже будет включен в контроллер. По умолчанию пользователь не сможет войти в систему в течение одной минуты, если после нескольких попыток не сможет предоставить правильные учетные данные. Ограничение уникально для имени пользователя / адреса электронной почты и его IP-адреса.

## Ручная аутентификация пользователей

Обратите внимание, что вы не обязаны использовать контроллеры аутентификации, входящие в комплект Laravel. Если вы решите удалить эти контроллеры, вам нужно будет управлять аутентификацией пользователей, используя классы аутентификации Laravel напрямую. Не волнуйтесь, это cinch!

Мы получаем доступ к сервисам аутентификации Ларавела через [фасад](../architecture-concepts/facades.md) `Auth`, поэтому нужно убедиться, что фасад `Auth` импортирован в файле. Далее посмотрим на метод `attempt`:

```php
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

Метод `attempt` принимает массив пар ключ/значение в качестве первого аргумента. Значения в массиве будут использованы для поиска пользователя в таблице базы данных. Таким образом, в приведенном выше примере пользователь будет найден по значению поля `email`. Если пользователь найден, то хэшированный пароль, хранящийся в БД, будет сравнен со значением `password`, переданным методу через массив. Не следует хэшировать пароль, указанный в качестве значения `password`, так как фреймворк автоматически хэширует значение перед сравнением с хэшированным паролем в БД. Если два хэшированных пароля совпадут, то для пользователя будет запущена аутентифицированная сессия.

Метод `attempt` вернет `true`, если аутентификация прошла успешно. В противном случае будет возвращено `false`.

Метод `intended` редиректора будет перенаправлять пользователя на URL, к которому он пытался получить доступ, прежде чем он будет перехвачен посредником для аутентификации. В случае недоступности предполагаемого адресата этому методу может быть предоставлен резервный URI.

**Указание дополнительных условий**

При желании вы также можете добавить дополнительные условия к запросу на аутентификацию в дополнение к электронной почте и паролю пользователя. Например, мы можем проверить, что пользователь отмечен как "active":

```php
if (Auth::attempt(['email' => $email, 'password' => $password, 'active' => 1])) {
    // The user is active, not suspended, and exists.
}
```

{% hint style="warning" %}
В этих примерах `email` не является обязательной опцией, он просто используется в качестве примера. Вы должны использовать любое имя столбца, соответствующее "имени пользователя" в вашей базе данных.
{% endhint %}

**Доступ к конкретным экземплярам защитника**

Вы можете указать, какой экземпляр защитника хотите использовать, используя метод `guard` фасада `Auth`. Это позволяет вам управлять аутентификацией для отдельных частей вашего приложения, используя полностью отдельные аутентифицируемые модели или пользовательские таблицы.

Имя охранника, переданное в метод `guard` должно соответствовать одному из охранников, настроенных в конфигурационном файле `auth.php`:

```php
if (Auth::guard('admin')->attempt($credentials)) {
    //
}
```

**Выход из системы**

Для выхода пользователей из приложения вы можете использовать метод `logout` фасада `Auth`. Это очистит информацию об аутентификации в сессии пользователя:

```php
Auth::logout();
```

### Запоминание пользователя

Если вы хотите обеспечить функциональность "запомнить меня", можете передать булевое значение в качестве второго аргумента методу `attempt`, который будет держать пользователя аутентифицированным бесконечно долго, или до тех пор, пока он вручную не выйдет из системы. Ваша таблица `users` должна содержать поле `remember_token`, которая будет использоваться для хранения токена "помнить меня".

```php
if (Auth::attempt(['email' => $email, 'password' => $password], $remember)) {
    // The user is being remembered...
}
```

{% hint style="info" %}
Если Вы используете встроенный `LoginController`, который поставляется вместе с Laravel, то правильная логика для "запоминания" пользователей уже реализована трейтами, используемыми контроллером.
{% endhint %}

Если вы "вспоминаете" пользователей, вы можете использовать метод `viaRemember` для определения того, был ли пользователь аутентифицирован с помощью cookie "помните меня":

```php
if (Auth::viaRemember()) {
    //
}
```

### Другие методы аутентификации

**Аутентифицировать экземпляр пользователя**

Если Вам необходимо зарегистрировать существующий экземпляр пользователя, Вы можете вызвать метод `login` этого экземпляра пользователя. Данный объект должен реализовывать [контракт](../architecture-concepts/contracts.md) `Illuminate\Contracts\Auth\Authenticatable`. Модель `App\User`, входящая в состав Laravel, уже реализует этот интерфейс:

```php
Auth::login($user);

// Login and "remember" the given user...
Auth::login($user, true);
```

Вы можете указать экземпляр защитника, который хотите использовать:

```php
Auth::guard('admin')->login($user);
```

**Аутентифицировать пользователя по ID**

Для входа пользователя в приложение по его ID можно воспользоваться методом `loginUsingId`. Этот метод принимает первичный ключ пользователя, которого вы хотите аутентифицировать:

```php
Auth::loginUsingId(1);

// Login and "remember" the given user...
Auth::loginUsingId(1, true);
```

**Аутентифицировать пользователя единожды**

Вы можете использовать метод `once` для входа пользователя в приложение для одного запроса. Сессии и куки-файлы использоваться не будут, что означает, что этот метод может быть полезен при построении API без состояния:

```php
if (Auth::once($credentials)) {
    //
}
```

## HTTP-Basic аутентификация

[HTTP-Basic аутентификация](https://en.wikipedia.org/wiki/Basic_access_authentication) обеспечивает быстрый способ аутентификации пользователей без настройки специальной страницы входа. Для начала добавьте [посредника](../the-basics/middleware.md) `auth.basic` к вашему маршруту. Посредник `auth.basic` включен в фреймворк Laravel, так что вам не нужно его определять:

```php
Route::get('profile', function () {
    // Only authenticated users may enter...
})->middleware('auth.basic');
```

Когда посредник подключен к маршруту, при доступе к маршруту в браузере вам автоматически будет предложено ввести учетные данные. По умолчанию в посреднике `auth.basic` в качестве "имени пользователя" будет использоваться поле `email`.

**Замечание о FastCGI**

Если вы используете PHP FastCGI, то HTTP-Basic аутентификация может работать некорректно. Добавьте следующие строки в файл `.htaccess`:

```php
RewriteCond %{HTTP:Authorization} ^(.+)$
RewriteRule .* - [E=HTTP_AUTHORIZATION:%{HTTP:Authorization}]
```

### HTTP Basic аутентификация без состояния

Вы также можете использовать базовую HTTP-аутентификацию без установки куки-файла идентификатора пользователя в сессии, что особенно полезно для API-аутентификации. Для этого [определите посредника](../the-basics/middleware.md), который вызывает метод `onceBasic`. Если метод `onceBasic` не возвращает ответ, то запрос может быть передан дальше в приложение:

```php
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

Далее [зарегистрируйте посредника](../the-basics/middleware.md#registering-middleware) и добавьте его к маршруту:

```php
Route::get('api/user', function () {
    // Only authenticated users may enter...
})->middleware('auth.basic.once');
```

## Выход из системы

Для ручного выхода пользователей вы можете использовать метод `logout` фасада `Auth`. Это очистит информацию об аутентификации в сессии пользователя:

```php
use Illuminate\Support\Facades\Auth;

Auth::logout();
```

### Аннулирование сеансов на других устройствах

Laravel также обеспечивает механизм для аннулирования и "выхода из системы" пользовательских сессий, которые активны на других устройствах без аннулирования сессии на их текущем устройстве. Эта функция обычно используется, когда пользователь меняет или обновляет свой пароль, и вы хотите сделать недействительными сеансы на других устройствах, при этом сохраняя текущее устройство аутентифицированным.

Прежде чем начать, вы должны убедиться, что посредник `Illuminate\Session\Middleware\AuthenticateSession` присутствует и не закомментирован в классе `app/Http/Kernel.php` в группе `web` посредников:

```php
'web' => [
    // ...
    \Illuminate\Session\Middleware\AuthenticateSession::class,
    // ...
],
```

Затем, вы можете использовать метод `logoutOtherDevices` фасада `Auth`. Этот метод требует от пользователя указать свой текущий пароль, который ваше приложение должно принять через форму ввода:

```php
use Illuminate\Support\Facades\Auth;

Auth::logoutOtherDevices($password);
```

При вызове метода `logoutOtherDevices` другие сеансы пользователя будут полностью признаны недействительными, что означает, что они будут "разлогинены" из всех охранников, которыми они были ранее аутентифицированы.

{% hint style="warning" %}
При использовании посредника `AuthenticateSession` в сочетании с пользовательским именем маршрута для маршрута `login`, вы должны переопределить метод `unuthenticated` в обработчике исключений приложения, чтобы правильно перенаправить пользователей на вашу страницу входа.
{% endhint %}

## Добавление пользовательских охранников

Вы можете определить своих собственных охранников аутентификации, используя метод `extend` фасада `Auth`. Этот вызов следует поместить в `extend` внутри [сервис-провайдера](../architecture-concepts/providers.md). Поскольку Laravel уже поставляется с `AuthServiceProvider`, мы можем разместить код в этом провайдере:

```php
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

Как видно в примере выше, функция обратного вызова, переданная в метод `extend` должна возвращать реализацию `Illuminate\Contracts\Auth\Guard`. Этот интерфейс содержит несколько методов, которые потребуется реализовать для определения пользовательского охранника. После того, как ваш пользовательский защитник будет определен, вы можете использовать этот защитник в конфигурации `guards` конфигурационного файла `auth.php`:

```php
'guards' => [
    'api' => [
        'driver' => 'jwt',
        'provider' => 'users',
    ],
],
```

### Closure Request Guards

Самый простой способ реализации пользовательской системы аутентификации, основанной на HTTP запросах, — это использование метода `Auth::viaRequest`. Этот метод позволяет быстро определить процесс аутентификации с помощью одного Closure.

Для начала вызовите метод `Auth::viaRequest` внутри метода `boot` вашего `AuthServiceProvider`. Метод `viaRequest` принимает в качестве первого аргумента имя драйвера аутентификации. Это имя может быть любой строкой, описывающей вашего пользовательского защитника. Второй аргумент, передаваемый методу, должен быть Closure, принимающий входящий HTTP-запрос и возвращающий экземпляр пользователя или, в случае неудачи аутентификации, `null`:

```php
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

После того, как ваш пользовательский драйвер аутентификации определен, используйте его в качестве драйвера в конфигурации `guards` конфигурационного файла `auth.php`:

```php
'guards' => [
    'api' => [
        'driver' => 'custom-token',
    ],
],
```

## Adding Custom User Providers

Если вы не используете традиционную реляционную базу данных для хранения пользователей, вам придется расширить Laravel с помощью собственного поставщика аутентификации пользователей. Мы будем использовать метод `provider` фасада `Auth` для определения кастомного провайдера пользователей:

```php
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

После регистрации провайдера методом `provider`, Вы можете переключиться на нового провайдера в своем конфигурационном файле `auth.php`. Сначала определите `провайдера`, который использует Ваш новый драйвер:

```php
'providers' => [
    'users' => [
        'driver' => 'riak',
    ],
],
```

Наконец, Вы можете использовать этот провайдер в конфигурации `guards`:

```php
'guards' => [
    'web' => [
        'driver' => 'session',
        'provider' => 'users',
    ],
],
```

### The User Provider Contract

Реализации `Illuminate\Contracts\Auth\UserProvider` отвечают только за получение реализации `Illuminate\Contracts\Auth\Authenticatable` из системы постоянного хранения, такой как MySQL, Riak и т.д. Эти два интерфейса позволяют механизмам аутентификации Laravel продолжать работать независимо от того, как хранятся пользовательские данные или какой тип класса используется для их представления.

Посмотрим на контракт `Illuminate\Contracts\Auth\UserProvider`:

```php
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

Функция `retrieveById` обычно получает ключ, представляющий пользователя, например, автоинкрементирующий ID из базы данных MySQL. Реализация `Authenticatable`, соответствующая ID, должна быть извлечена и возвращена этим методом.

Функция `retrieveByToken` извлекает пользователя по его уникальному идентификатору `$identifier` и токену `$token` "запомнить меня", хранящемуся в поле `remember_token`. Как и в предыдущем методе, должна быть возвращена реализация `Authenticable`.

Метод `updateRememberToken` обновляет у пользователя `$user` поле `remember_token` новым токеном `$token`. Свежий токен присваивается при успешной попытке входа "запомнить меня" или при выходе пользователя из системы.

Метод `retrieveByCredentials` получает массив регистрационных данных, передаваемых в метод `Auth::attempt` при попытке входа в приложение. Затем метод должен "запросить" из базового постоянного хранилища пользователя, соответствующего этим учетным данным. Обычно этот метод выполняет запрос с условием "where" на `$credentials['username']`. Затем метод должен вернуть реализацию `Authenticatable`. **Этот метод не должен пытаться выполнить проверку пароля или аутентификацию.**.

Метод `validateCredentials` должен сравнивать данные пользователя `$user` с учетными данными `$credentials`, чтобы аутентифицировать пользователя. Например, этот метод, вероятно, должен использовать `Hash::check` для сравнения значения `$user->getAuthPassword()` со значением `$credentials['password']`. Этот метод должен возвращать `true` или `false`, указывая на то, является ли пароль действительным.

### Контракт Authenticatable

Теперь, когда мы изучили каждый из методов `UserProvider`, давайте посмотрим на контракт `Authenticable`. Помните, что провайдер должен возвращать реализации этого интерфейса из методов `retrieveById`, `retrieveByToken` и `retrieveByCredentials`:

```php
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

Этот интерфейс прост. Метод `getAuthIdentifierName` должен возвращать имя поля "первичный ключ" \(primary key\) пользователя, а метод `getAuthIdentifier` должен возвращать "первичный ключ" пользователя. В бэкэнде с MySQL, опять же, это будет автоинкрементирующий первичный ключ. Метод `getAuthPassword` должен возвращать хэшированный пароль пользователя. Этот интерфейс позволяет системе аутентификации работать с любым классом пользователя, независимо от того, какую ORM или уровень абстракции хранилища вы используете. По умолчанию, Laravel размещает в каталоге `app` класс `User`, который реализует этот интерфейс, так что вы можете обратиться к этому классу за примером реализации.

## События

Laravel генерирует различные [события](https://github.com/delphinpro/laravel-ru/tree/23145faa5557cb82ae9c1b23ff97487d9737edca/security/.../digging-deeper/events.md) во время процесса аутентификации. Вы можете повесить слушателей к этим событиям в вашем `EventServiceProvider`:

```php
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

