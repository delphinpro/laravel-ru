# Laravel Passport

## Вступление

Laravel уже упрощает выполнение аутентификации с помощью традиционных форм входа, но как насчет API? API обычно используют токены для аутентификации пользователей и не поддерживают состояние сеанса между запросами. Laravel делает аутентификацию API легкой задачей, используя Laravel Passport, который обеспечивает полную реализацию сервера OAuth2 для вашего приложения в течение нескольких минут. Passport построен на [OAuth2 сервере] (https://github.com/thephpleague/oauth2-server), который поддерживается Энди Миллингтоном (Andy Millington) и Саймоном Хэмпом (Simon Hamp).

{% hint style="warning" %}
Эта документация предполагает, что вы уже знакомы с OAuth2. Если вы ничего не знаете об OAuth2, то прежде чем продолжить, ознакомьтесь с общей [терминологией](https://oauth2.thephpleague.com/terminology/) и особенностями OAuth2.
{% endhint %}

## Обновление Passport

При обновлении до новой основной версии Passport'а важно внимательно ознакомиться с [руководством по обновлению](https://github.com/laravel/passport/blob/master/UPGRADE.md).

## Установка

Для начала установите Passport через менеджер пакетов Composer:

```bash
composer require laravel/passport
```

Сервис-провайдер Passport регистрирует в фреймворке собственный каталог миграции базы данных, поэтому после установки пакета необходимо выполнить миграцию базы данных. Миграция Passport создаст таблицы, необходимые приложению для хранения клиентов и токенов доступа:

```bash
php artisan migrate
```

Далее следует запустить команду `passport:install`. Эта команда создаст ключи шифрования, необходимые для генерации маркеров безопасного доступа. Кроме того, команда создаст клиентов "персонального доступа (personal access)" и "выдачи пароля (password grant)", которые будут использоваться для генерации токенов доступа:

```bash
php artisan passport:install
```

{% hint style="info" %}
Если вы хотите использовать UUID в качестве значения первичного ключа (primary key) модели `Client` Passport'a вместо автоинкрементируемых целых чисел, пожалуйста, установите Passport, используя [опцию `uuids`] (passport.md#client-uuids).
{% endhint %}

После выполнения команды `passport:install` добавьте трейт `Laravel\Passport\HasApiTokens` к вашей модели `App\User`. Этот трейт предоставит несколько вспомогательных методов для модели, которые позволят проверять токен аутентифицированного пользователя и области видимости (scopes):

```php
<?php

namespace App;

use Illuminate\Foundation\Auth\User as Authenticatable;
use Illuminate\Notifications\Notifiable;
use Laravel\Passport\HasApiTokens;

class User extends Authenticatable
{
    use HasApiTokens, Notifiable;
}
```

Далее следует вызвать метод `Passport::routes` в рамках метода `boot` вашего `AuthServiceProvider`. В этом методе будут зарегистрированы маршруты, необходимые для выдачи и отзыва токенов доступа, клиентов и персональных токенов доступа:

```php
<?php

namespace App\Providers;

use Illuminate\Foundation\Support\Providers\AuthServiceProvider as ServiceProvider;
use Illuminate\Support\Facades\Gate;
use Laravel\Passport\Passport;

class AuthServiceProvider extends ServiceProvider
{
    /**
     * The policy mappings for the application.
     *
     * @var array
     */
    protected $policies = [
        'App\Model' => 'App\Policies\ModelPolicy',
    ];

    /**
     * Register any authentication / authorization services.
     *
     * @return void
     */
    public function boot()
    {
        $this->registerPolicies();

        Passport::routes();
    }
}
```

Наконец, в конфигурационном файле `config/auth.php` вы должны установить опцию `driver` для защитника `api` аутентификации в значение `passprot`. Это даст указание приложению использовать `TokenGuard` Passport'а при аутентификации входящих API запросов:

```php
'guards' => [
    'web' => [
        'driver' => 'session',
        'provider' => 'users',
    ],

    'api' => [
        'driver' => 'passport',
        'provider' => 'users',
    ],
],
```

**Client UUIDs**

Вы можете запустить команду `passport:install` с опцией `--uuids`. Этот флаг укажет Passport'у, что вы хотите использовать UUID вместо автоинкрементирумого целого в качестве значения Primary Key модели `Client`. После выполнения команды `passport:install` с опцией `--uuids`, вам будут даны дополнительные инструкции относительно отключения миграции Passport по умолчанию:

```bash
php artisan passport:install --uuids
```

### Быстрый старт на фронтэнде

{% hint style="warning" %}
Для того, чтобы использовать Vue-компоненты Passport'а, необходим javascript-фреймворк [Vue](https://vuejs.org/). Эти компоненты также используют фреймворк Bootstrap CSS. Однако, даже если вы не используете данные инструменты, эти компоненты служат ценным справочником для вашей собственной реализации фронтенда.
{% endhint %}

Для публикации Vue-компонентов Passport'а используйте Artisan-команду `vendor:publish`:

```bash
php artisan vendor:publish --tag=passport-components
```

Опубликованные компоненты будут размещены в каталоге `resources/js/components`. После того, как компоненты будут опубликованы, вы должны зарегистрировать их в файле `resources/js/app.js`:

```javascript
Vue.component(
    'passport-clients',
    require('./components/passport/Clients.vue').default
);

Vue.component(
    'passport-authorized-clients',
    require('./components/passport/AuthorizedClients.vue').default
);

Vue.component(
    'passport-personal-access-tokens',
    require('./components/passport/PersonalAccessTokens.vue').default
);
```

{% hint style="warning" %}
До версии Laravel v5.7.19 добавление `по умолчанию` при регистрации компонентов приводит к консольной ошибке. Объяснение этого изменения можно найти в [Laravel Mix v4.0.0 release notes](https://github.com/JeffreyWay/laravel-mix/releases/tag/v4.0.0).
{% endhint %}

После регистрации компонентов, обязательно запустите `npm run dev` для перекомпиляции ассетов. После перекомпиляции, Вы можете бросить компоненты в один из шаблонов приложения, чтобы начать создавать клиентов и токены персонального доступа:

```markup
<passport-clients></passport-clients>
<passport-authorized-clients></passport-authorized-clients>
<passport-personal-access-tokens></passport-personal-access-tokens>
```

### Развертывание Passport

При первом развертывании Passport'а на производственных серверах вам, скорее всего, придется выполнить команду `passport:keys`. Эта команда генерирует ключи шифрования, необходимые Паспорту для генерации токена доступа. Генерируемые ключи обычно исключаются из системы контроля версий:

```bash
php artisan passport:keys
```

При необходимости можно определить путь, откуда будут загружаться ключи паспорта. Для этого можно использовать метод `Passport::loadKeysFrom`:

```php
/**
 * Register any authentication / authorization services.
 *
 * @return void
 */
public function boot()
{
    $this->registerPolicies();

    Passport::routes();

    Passport::loadKeysFrom('/secret-keys/oauth');
}
```

Кроме того, вы можете опубликовать конфигурационный файл паспорта, используя `php artisan vendor:published --tag=passport-config`, который затем предоставит возможность загрузить ключи шифрования из переменных окружения:

```text
PASSPORT_PRIVATE_KEY="-----BEGIN RSA PRIVATE KEY-----
<private key here>
-----END RSA PRIVATE KEY-----"

PASSPORT_PUBLIC_KEY="-----BEGIN PUBLIC KEY-----
<public key here>
-----END PUBLIC KEY-----"
```

### Настройка миграций

Если вы не собираетесь использовать миграции Паспорта по умолчанию, вам следует вызвать метод `Passport::ignoreMigrations` в методе `register` вашего `AppServiceProvider`. Вы можете экспортировать миграции по умолчанию, используя `php artisan vendor:published --tag=passport-migrations`.

## Настройка

### Хеширование токенов

Если вы хотите, чтобы при хранении базе данных токены клиента были хэшированы, вам следует вызвать метод `Passport::hashClientSecrets` в методе `boot` вашего `AppServiceProvider`:

```php
Passport::hashClientSecrets();
```

После включения, все ваши клиентские токены будут показаны только один раз, при создании клиента. Так как текстовое значение токена клиента никогда не хранится в базе данных, восстановить его в случае потери невозможно.

### Время жизни токенов

По умолчанию паспорт выпускает долговечные токены доступа, срок действия которых истекает через год. Если вы хотите настроить более длительный/короткий срок службы токенов, вы можете использовать методы `tokensExpireIn`, `refreshTokensExpireIn` и `personalAccessTokensExpireIn`. Эти методы должны вызываться из метода `boot` вашего `AuthServiceProvider`:

```php
/**
 * Register any authentication / authorization services.
 *
 * @return void
 */
public function boot()
{
    $this->registerPolicies();

    Passport::routes();

    Passport::tokensExpireIn(now()->addDays(15));

    Passport::refreshTokensExpireIn(now()->addDays(30));

    Passport::personalAccessTokensExpireIn(now()->addMonths(6));
}
```

{% hint style="warning" %}
Столбцы `expires_at` в таблицах базы данных Паспорта предназначены только для чтения и отображения. При выдаче токенов Паспорт хранит информацию об истечении срока действия в подписанных и зашифрованных токенах. Если Вам необходимо сделать токен недействительным, Вы должны его отозвать.
{% endhint %}

### Переопределение моделей по умолчанию

Вы можете свободно расширять модели, используемые внутри паспорта:

```php
use Laravel\Passport\Client as PassportClient;

class Client extends PassportClient
{
    // ...
}
```

Затем, вы можете указать Паспорту, что нужно использовать ваши пользовательские модели через класс `Passport`:

```php
use App\Models\Passport\AuthCode;
use App\Models\Passport\Client;
use App\Models\Passport\PersonalAccessClient;
use App\Models\Passport\Token;

/**
 * Register any authentication / authorization services.
 *
 * @return void
 */
public function boot()
{
    $this->registerPolicies();

    Passport::routes();

    Passport::useTokenModel(Token::class);
    Passport::useClientModel(Client::class);
    Passport::useAuthCodeModel(AuthCode::class);
    Passport::usePersonalAccessClientModel(PersonalAccessClient::class);
}
```

## Выдача токенов доступа

Использование OAuth2 с кодами авторизации такое же, как большинство разработчиков знакомы с OAuth2. При использовании кодов авторизации клиентское приложение перенаправит пользователя на ваш сервер, где он либо одобрит, либо отклонит запрос на выдачу токена доступа клиенту.

### Управление клиентами

Сначала разработчикам, создающим приложения, которые должны взаимодействовать с API вашего приложения, нужно будет зарегистрировать свое приложение в вашем, создав "клиент". Обычно это заключается в предоставлении имени своего приложения и URL, на который ваше приложение может быть перенаправлено после того, как пользователи одобрят его запрос на авторизацию.

**Команда passport:client**

Самый простой способ создания клиента — использование команды `passport:client` Artisan. Эта команда может быть использована для создания собственных клиентов для тестирования функциональности OAuth2. При запуске команды `client` паспорт запросит у вас дополнительную информацию о вашем клиенте и предоставит вам идентификатор и секрет клиента:

```bash
php artisan passport:client
```

**Перенаправление адресов**

Если вы хотите разрешить несколько URL-адресов редиректа для вашего клиента, то можете указать их, используя разделенный запятыми список при запросе командой `passport:client` URL-адреса:

```text
http://example.com/callback,http://examplefoo.com/callback
```

{% hint style="warning" %}
Любой URL, содержащий запятые, должен быть закодирован.
{% endhint %}

**JSON API**

Так как ваши пользователи не смогут использовать команду `client`, паспорт предоставляет JSON API, который вы можете использовать для создания клиентов. Это избавляет вас от необходимости вручную писать контроллеры для создания, обновления и удаления клиентов.

Тем не менее, вам нужно будет сопоставить JSON API паспорта с вашим собственным фронтэндом, чтобы обеспечить панель инструментов для ваших пользователей для управления своими клиентами. Ниже мы рассмотрим все конечные точки API для управления клиентами. Для удобства мы используем [Axios](https://github.com/axios/axios), чтобы продемонстрировать выполнение HTTP-запросов к конечным точкам.

JSON API защищен посредниками `web` и `auth`, поэтому он может быть вызван только из вашего собственного приложения. Он не может быть вызван из внешнего источника.

{% hint style="info" %}
Если Вы не хотите самостоятельно реализовывать весь фронтенд управления клиентами, Вы можете использовать [frontend quickstart](passport.md#frontend-quickstart), чтобы иметь полнофункциональный фронтенд в считанные минуты.
{% endhint %}

**GET /oauth/clients**

This route returns all of the clients for the authenticated user. This is primarily useful for listing all of the user's clients so that they may edit or delete them:

```javascript
axios.get('/oauth/clients')
    .then(response => {
        console.log(response.data);
    });
```

**POST /oauth/clients**

This route is used to create new clients. It requires two pieces of data: the client's `name` and a `redirect` URL. The `redirect` URL is where the user will be redirected after approving or denying a request for authorization.

When a client is created, it will be issued a client ID and client secret. These values will be used when requesting access tokens from your application. The client creation route will return the new client instance:

```javascript
const data = {
    name: 'Client Name',
    redirect: 'http://example.com/callback'
};

axios.post('/oauth/clients', data)
    .then(response => {
        console.log(response.data);
    })
    .catch (response => {
        // List errors on response...
    });
```

**PUT /oauth/clients/{client-id}**

This route is used to update clients. It requires two pieces of data: the client's `name` and a `redirect` URL. The `redirect` URL is where the user will be redirected after approving or denying a request for authorization. The route will return the updated client instance:

```javascript
const data = {
    name: 'New Client Name',
    redirect: 'http://example.com/callback'
};

axios.put('/oauth/clients/' + clientId, data)
    .then(response => {
        console.log(response.data);
    })
    .catch (response => {
        // List errors on response...
    });
```

**DELETE /oauth/clients/{client-id}**

This route is used to delete clients:

```javascript
axios.delete('/oauth/clients/' + clientId)
    .then(response => {
        //
    });
```

### Requesting Tokens

**Redirecting For Authorization**

Once a client has been created, developers may use their client ID and secret to request an authorization code and access token from your application. First, the consuming application should make a redirect request to your application's `/oauth/authorize` route like so:

```php
Route::get('/redirect', function (Request $request) {
    $request->session()->put('state', $state = Str::random(40));

    $query = http_build_query([
        'client_id' => 'client-id',
        'redirect_uri' => 'http://example.com/callback',
        'response_type' => 'code',
        'scope' => '',
        'state' => $state,
    ]);

    return redirect('http://your-app.com/oauth/authorize?'.$query);
});
```

> ![](https://laravel.com/img/callouts/lightbulb.min.svg)
>
> Remember, the `/oauth/authorize` route is already defined by the `Passport::routes` method. You do not need to manually define this route.

{% hint style="info" %}

{% endhint %}

**Approving The Request**

When receiving authorization requests, Passport will automatically display a template to the user allowing them to approve or deny the authorization request. If they approve the request, they will be redirected back to the `redirect_uri` that was specified by the consuming application. The `redirect_uri` must match the `redirect` URL that was specified when the client was created.

If you would like to customize the authorization approval screen, you may publish Passport's views using the `vendor:publish` Artisan command. The published views will be placed in `resources/views/vendor/passport`:

```javascript
php artisan vendor:publish --tag=passport-views
```

Sometimes you may wish to skip the authorization prompt, such as when authorizing a first-party client. You may accomplish this by [extending the `Client` model](passport.md#overriding-default-models) and defining a `skipsAuthorization` method. If `skipsAuthorization` returns `true` the client will be approved and the user will be redirected back to the `redirect_uri` immediately:

```php
<?php

namespace App\Models\Passport;

use Laravel\Passport\Client as BaseClient;

class Client extends BaseClient
{
    /**
     * Determine if the client should skip the authorization prompt.
     *
     * @return bool
     */
    public function skipsAuthorization()
    {
        return $this->firstParty();
    }
}
```

**Converting Authorization Codes To Access Tokens**

If the user approves the authorization request, they will be redirected back to the consuming application. The consumer should first verify the `state` parameter against the value that was stored prior to the redirect. If the state parameter matches the consumer should issue a `POST` request to your application to request an access token. The request should include the authorization code that was issued by your application when the user approved the authorization request. In this example, we'll use the Guzzle HTTP library to make the `POST` request:

```php
Route::get('/callback', function (Request $request) {
    $state = $request->session()->pull('state');

    throw_unless(
        strlen($state) > 0 && $state === $request->state,
        InvalidArgumentException::class
    );

    $http = new GuzzleHttp\Client;

    $response = $http->post('http://your-app.com/oauth/token', [
        'form_params' => [
            'grant_type' => 'authorization_code',
            'client_id' => 'client-id',
            'client_secret' => 'client-secret',
            'redirect_uri' => 'http://example.com/callback',
            'code' => $request->code,
        ],
    ]);

    return json_decode((string) $response->getBody(), true);
});
```

This `/oauth/token` route will return a JSON response containing `access_token`, `refresh_token`, and `expires_in` attributes. The `expires_in` attribute contains the number of seconds until the access token expires.

> ![](https://laravel.com/img/callouts/lightbulb.min.svg)
>
> Like the `/oauth/authorize` route, the `/oauth/token` route is defined for you by the `Passport::routes` method. There is no need to manually define this route. By default, this route is throttled using the settings of the `ThrottleRequests` middleware.

{% hint style="info" %}

{% endhint %}

**JSON API**

Passport also includes a JSON API for managing authorized access tokens. You may pair this with your own frontend to offer your users a dashboard for managing access tokens. For convenience, we'll use [Axios](https://github.com/mzabriskie/axios) to demonstrate making HTTP requests to the endpoints. The JSON API is guarded by the `web` and `auth` middleware; therefore, it may only be called from your own application.

**GET /oauth/tokens**

This route returns all of the authorized access tokens that the authenticated user has created. This is primarily useful for listing all of the user's tokens so that they can revoke them:

```javascript
axios.get('/oauth/tokens')
    .then(response => {
        console.log(response.data);
    });
```

**DELETE /oauth/tokens/{token-id}**

This route may be used to revoke authorized access tokens and their related refresh tokens:

```javascript
axios.delete('/oauth/tokens/' + tokenId);
```

### Refreshing Tokens

If your application issues short-lived access tokens, users will need to refresh their access tokens via the refresh token that was provided to them when the access token was issued. In this example, we'll use the Guzzle HTTP library to refresh the token:

```php
$http = new GuzzleHttp\Client;

$response = $http->post('http://your-app.com/oauth/token', [
    'form_params' => [
        'grant_type' => 'refresh_token',
        'refresh_token' => 'the-refresh-token',
        'client_id' => 'client-id',
        'client_secret' => 'client-secret',
        'scope' => '',
    ],
]);

return json_decode((string) $response->getBody(), true);
```

This `/oauth/token` route will return a JSON response containing `access_token`, `refresh_token`, and `expires_in` attributes. The `expires_in` attribute contains the number of seconds until the access token expires.

### Revoking Tokens

You may revoke a token by using the `revokeAccessToken` method on the `TokenRepository`. You may revoke a token's refresh tokens using the `revokeRefreshTokensByAccessTokenId` method on the `RefreshTokenRepository`:

```php
$tokenRepository = app('Laravel\Passport\TokenRepository');
$refreshTokenRepository = app('Laravel\Passport\RefreshTokenRepository');

// Revoke an access token...
$tokenRepository->revokeAccessToken($tokenId);

// Revoke all of the token's refresh tokens...
$refreshTokenRepository->revokeRefreshTokensByAccessTokenId($tokenId);
```

### Purging Tokens

When tokens have been revoked or expired, you might want to purge them from the database. Passport ships with a command that can do this for you:

```bash
# Purge revoked and expired tokens and auth codes...
php artisan passport:purge

# Only purge revoked tokens and auth codes...
php artisan passport:purge --revoked

# Only purge expired tokens and auth codes...
php artisan passport:purge --expired
```

You may also configure a [scheduled job](scheduling) in your console `Kernel` class to automatically prune your tokens on a schedule:

```php
/**
 * Define the application's command schedule.
 *
 * @param  \Illuminate\Console\Scheduling\Schedule  $schedule
 * @return void
 */
protected function schedule(Schedule $schedule)
{
    $schedule->command('passport:purge')->hourly();
}
```

## Authorization Code Grant with PKCE

The Authorization Code grant with "Proof Key for Code Exchange" \(PKCE\) is a secure way to authenticate single page applications or native applications to access your API. This grant should be used when you can't guarantee that the client secret will be stored confidentially or in order to mitigate the threat of having the authorization code intercepted by an attacker. A combination of a "code verifier" and a "code challenge" replaces the client secret when exchanging the authorization code for an access token.

### Creating The Client

Before your application can issue tokens via the authorization code grant with PKCE, you will need to create a PKCE-enabled client. You may do this using the `passport:client` command with the `--public` option:

```bash
php artisan passport:client --public
```

### Requesting Tokens

**Code Verifier & Code Challenge**

As this authorization grant does not provide a client secret, developers will need to generate a combination of a code verifier and a code challenge in order to request a token.

The code verifier should be a random string of between 43 and 128 characters containing letters, numbers and `"-"`, `"."`, `"_"`, `"~"`, as defined in the [RFC 7636 specification](https://tools.ietf.org/html/rfc7636).

The code challenge should be a Base64 encoded string with URL and filename-safe characters. The trailing `'='` characters should be removed and no line breaks, whitespace, or other additional characters should be present.

```php
$encoded = base64_encode(hash('sha256', $code_verifier, true));

$codeChallenge = strtr(rtrim($encoded, '='), '+/', '-_');
```

**Redirecting For Authorization**

Once a client has been created, you may use the client ID and the generated code verifier and code challenge to request an authorization code and access token from your application. First, the consuming application should make a redirect request to your application's `/oauth/authorize` route:

```php
Route::get('/redirect', function (Request $request) {
    $request->session()->put('state', $state = Str::random(40));

    $request->session()->put('code_verifier', $code_verifier = Str::random(128));

    $codeChallenge = strtr(rtrim(
        base64_encode(hash('sha256', $code_verifier, true))
    , '='), '+/', '-_');

    $query = http_build_query([
        'client_id' => 'client-id',
        'redirect_uri' => 'http://example.com/callback',
        'response_type' => 'code',
        'scope' => '',
        'state' => $state,
        'code_challenge' => $codeChallenge,
        'code_challenge_method' => 'S256',
    ]);

    return redirect('http://your-app.com/oauth/authorize?'.$query);
});
```

**Converting Authorization Codes To Access Tokens**

If the user approves the authorization request, they will be redirected back to the consuming application. The consumer should verify the `state` parameter against the value that was stored prior to the redirect, as in the standard Authorization Code Grant.

If the state parameter matches, the consumer should issue a `POST` request to your application to request an access token. The request should include the authorization code that was issued by your application when the user approved the authorization request along with the originally generated code verifier:

```php
Route::get('/callback', function (Request $request) {
    $state = $request->session()->pull('state');

    $codeVerifier = $request->session()->pull('code_verifier');

    throw_unless(
        strlen($state) > 0 && $state === $request->state,
        InvalidArgumentException::class
    );

    $response = (new GuzzleHttp\Client)->post('http://your-app.com/oauth/token', [
        'form_params' => [
            'grant_type' => 'authorization_code',
            'client_id' => 'client-id',
            'redirect_uri' => 'http://example.com/callback',
            'code_verifier' => $codeVerifier,
            'code' => $request->code,
        ],
    ]);

    return json_decode((string) $response->getBody(), true);
});
```

## Password Grant Tokens

The OAuth2 password grant allows your other first-party clients, such as a mobile application, to obtain an access token using an e-mail address / username and password. This allows you to issue access tokens securely to your first-party clients without requiring your users to go through the entire OAuth2 authorization code redirect flow.

### Creating A Password Grant Client

Before your application can issue tokens via the password grant, you will need to create a password grant client. You may do this using the `passport:client` command with the `--password` option. If you have already run the `passport:install` command, you do not need to run this command:

```bash
php artisan passport:client --password
```

### Requesting Tokens

Once you have created a password grant client, you may request an access token by issuing a `POST` request to the `/oauth/token` route with the user's email address and password. Remember, this route is already registered by the `Passport::routes` method so there is no need to define it manually. If the request is successful, you will receive an `access_token` and `refresh_token` in the JSON response from the server:

```php
$http = new GuzzleHttp\Client;

$response = $http->post('http://your-app.com/oauth/token', [
    'form_params' => [
        'grant_type' => 'password',
        'client_id' => 'client-id',
        'client_secret' => 'client-secret',
        'username' => 'taylor@laravel.com',
        'password' => 'my-password',
        'scope' => '',
    ],
]);

return json_decode((string) $response->getBody(), true);
```

> ![](https://laravel.com/img/callouts/lightbulb.min.svg)
>
> Remember, access tokens are long-lived by default. However, you are free to [configure your maximum access token lifetime](passport.md#configuration) if needed.

{% hint style="info" %}

{% endhint %}

### Requesting All Scopes

When using the password grant or client credentials grant, you may wish to authorize the token for all of the scopes supported by your application. You can do this by requesting the `*` scope. If you request the `*` scope, the `can` method on the token instance will always return `true`. This scope may only be assigned to a token that is issued using the `password` or `client_credentials` grant:

```php
$response = $http->post('http://your-app.com/oauth/token', [
    'form_params' => [
        'grant_type' => 'password',
        'client_id' => 'client-id',
        'client_secret' => 'client-secret',
        'username' => 'taylor@laravel.com',
        'password' => 'my-password',
        'scope' => '*',
    ],
]);
```

### Customizing The User Provider

If your application uses more than one [authentication user provider](authentication#introduction), you may specify which user provider the password grant client uses by providing a `--provider` option when creating the client via the `artisan passport:client --password` command. The given provider name should match a valid provider defined in your `config/auth.php` configuration file. You can then [protect your route using middleware](passport.md#via-middleware) to ensure that only users from the guard's specified provider are authorized.

### Customizing The Username Field

When authenticating using the password grant, Passport will use the `email` attribute of your model as the "username". However, you may customize this behavior by defining a `findForPassport` method on your model:

```php
<?php

namespace App;

use Illuminate\Foundation\Auth\User as Authenticatable;
use Illuminate\Notifications\Notifiable;
use Laravel\Passport\HasApiTokens;

class User extends Authenticatable
{
    use HasApiTokens, Notifiable;

    /**
     * Find the user instance for the given username.
     *
     * @param  string  $username
     * @return \App\User
     */
    public function findForPassport($username)
    {
        return $this->where('username', $username)->first();
    }
}
```

### Customizing The Password Validation

When authenticating using the password grant, Passport will use the `password` attribute of your model to validate the given password. If your model does not have a `password` attribute or you wish to customize the password validation logic, you can define a `validateForPassportPasswordGrant` method on your model:

```php
<?php

namespace App;

use Illuminate\Foundation\Auth\User as Authenticatable;
use Illuminate\Notifications\Notifiable;
use Illuminate\Support\Facades\Hash;
use Laravel\Passport\HasApiTokens;

class User extends Authenticatable
{
    use HasApiTokens, Notifiable;

    /**
     * Validate the password of the user for the Passport password grant.
     *
     * @param  string  $password
     * @return bool
     */
    public function validateForPassportPasswordGrant($password)
    {
        return Hash::check($password, $this->password);
    }
}
```

## Implicit Grant Tokens

The implicit grant is similar to the authorization code grant; however, the token is returned to the client without exchanging an authorization code. This grant is most commonly used for JavaScript or mobile applications where the client credentials can't be securely stored. To enable the grant, call the `enableImplicitGrant` method in your `AuthServiceProvider`:

```php
/**
 * Register any authentication / authorization services.
 *
 * @return void
 */
public function boot()
{
    $this->registerPolicies();

    Passport::routes();

    Passport::enableImplicitGrant();
}
```

Once a grant has been enabled, developers may use their client ID to request an access token from your application. The consuming application should make a redirect request to your application's `/oauth/authorize` route like so:

```php
Route::get('/redirect', function (Request $request) {
    $request->session()->put('state', $state = Str::random(40));

    $query = http_build_query([
        'client_id' => 'client-id',
        'redirect_uri' => 'http://example.com/callback',
        'response_type' => 'token',
        'scope' => '',
        'state' => $state,
    ]);

    return redirect('http://your-app.com/oauth/authorize?'.$query);
});
```

> ![](https://laravel.com/img/callouts/lightbulb.min.svg)
>
> Remember, the `/oauth/authorize` route is already defined by the `Passport::routes` method. You do not need to manually define this route.

{% hint style="info" %}

{% endhint %}

## Client Credentials Grant Tokens

The client credentials grant is suitable for machine-to-machine authentication. For example, you might use this grant in a scheduled job which is performing maintenance tasks over an API.

Before your application can issue tokens via the client credentials grant, you will need to create a client credentials grant client. You may do this using the `--client` option of the `passport:client` command:

```bash
php artisan passport:client --client
```

Next, to use this grant type, you need to add the `CheckClientCredentials` middleware to the `$routeMiddleware` property of your `app/Http/Kernel.php` file:

```php
use Laravel\Passport\Http\Middleware\CheckClientCredentials;

protected $routeMiddleware = [
    'client' => CheckClientCredentials::class,
];
```

Then, attach the middleware to a route:

```php
Route::get('/orders', function (Request $request) {
    ...
})->middleware('client');
```

To restrict access to the route to specific scopes you may provide a comma-delimited list of the required scopes when attaching the `client` middleware to the route:

```php
Route::get('/orders', function (Request $request) {
    ...
})->middleware('client:check-status,your-scope');
```

#### Retrieving Tokens

To retrieve a token using this grant type, make a request to the `oauth/token` endpoint:

```php
$guzzle = new GuzzleHttp\Client;

$response = $guzzle->post('http://your-app.com/oauth/token', [
    'form_params' => [
        'grant_type' => 'client_credentials',
        'client_id' => 'client-id',
        'client_secret' => 'client-secret',
        'scope' => 'your-scope',
    ],
]);

return json_decode((string) $response->getBody(), true)['access_token'];
```

## Personal Access Tokens

Sometimes, your users may want to issue access tokens to themselves without going through the typical authorization code redirect flow. Allowing users to issue tokens to themselves via your application's UI can be useful for allowing users to experiment with your API or may serve as a simpler approach to issuing access tokens in general.

### Creating A Personal Access Client

Before your application can issue personal access tokens, you will need to create a personal access client. You may do this using the `passport:client` command with the `--personal` option. If you have already run the `passport:install` command, you do not need to run this command:

```bash
php artisan passport:client --personal
```

After creating your personal access client, place the client's ID and plain-text secret value in your application's `.env` file:

```bash
PASSPORT_PERSONAL_ACCESS_CLIENT_ID=client-id-value
PASSPORT_PERSONAL_ACCESS_CLIENT_SECRET=unhashed-client-secret-value
```

Next, you should register these values by placing the following calls to `Passport::personalAccessClientId` and `Passport::personalAccessClientSecret` within the `boot` method of your `AuthServiceProvider`:

```php
/**
 * Register any authentication / authorization services.
 *
 * @return void
 */
public function boot()
{
    $this->registerPolicies();

    Passport::routes();

    Passport::personalAccessClientId(
        config('passport.personal_access_client.id')
    );

    Passport::personalAccessClientSecret(
        config('passport.personal_access_client.secret')
    );
}
```

### Managing Personal Access Tokens

Once you have created a personal access client, you may issue tokens for a given user using the `createToken` method on the `User` model instance. The `createToken` method accepts the name of the token as its first argument and an optional array of [scopes](passport.md#token-scopes) as its second argument:

```php
$user = App\User::find(1);

// Creating a token without scopes...
$token = $user->createToken('Token Name')->accessToken;

// Creating a token with scopes...
$token = $user->createToken('My Token', ['place-orders'])->accessToken;
```

**JSON API**

Passport also includes a JSON API for managing personal access tokens. You may pair this with your own frontend to offer your users a dashboard for managing personal access tokens. Below, we'll review all of the API endpoints for managing personal access tokens. For convenience, we'll use [Axios](https://github.com/mzabriskie/axios) to demonstrate making HTTP requests to the endpoints.

The JSON API is guarded by the `web` and `auth` middleware; therefore, it may only be called from your own application. It is not able to be called from an external source.

> ![](https://laravel.com/img/callouts/lightbulb.min.svg)
>
> If you don't want to implement the personal access token frontend yourself, you can use the [frontend quickstart](passport.md#frontend-quickstart) to have a fully functional frontend in a matter of minutes.

{% hint style="info" %}

{% endhint %}

**GET /oauth/scopes**

This route returns all of the [scopes](passport.md#token-scopes) defined for your application. You may use this route to list the scopes a user may assign to a personal access token:

```javascript
axios.get('/oauth/scopes')
    .then(response => {
        console.log(response.data);
    });
```

**GET /oauth/personal-access-tokens**

This route returns all of the personal access tokens that the authenticated user has created. This is primarily useful for listing all of the user's tokens so that they may edit or revoke them:

```javascript
axios.get('/oauth/personal-access-tokens')
    .then(response => {
        console.log(response.data);
    });
```

**POST /oauth/personal-access-tokens**

This route creates new personal access tokens. It requires two pieces of data: the token's `name` and the `scopes` that should be assigned to the token:

```javascript
const data = {
    name: 'Token Name',
    scopes: []
};

axios.post('/oauth/personal-access-tokens', data)
    .then(response => {
        console.log(response.data.accessToken);
    })
    .catch (response => {
        // List errors on response...
    });
```

**DELETE /oauth/personal-access-tokens/{token-id}**

This route may be used to revoke personal access tokens:

```javascript
axios.delete('/oauth/personal-access-tokens/' + tokenId);
```

## Protecting Routes

### Via Middleware

Passport includes an [authentication guard](authentication#adding-custom-guards) that will validate access tokens on incoming requests. Once you have configured the `api` guard to use the `passport` driver, you only need to specify the `auth:api` middleware on any routes that require a valid access token:

```php
Route::get('/user', function () {
    //
})->middleware('auth:api');
```

**Multiple Authentication Guards**

If your application authenticates different types of users that perhaps use entirely different Eloquent models, you will likely need to define a guard configuration for each user provider type in your application. This allows you to protect requests intended for specific user providers. For example, given the following guard configuration the `config/auth.php` configuration file:

```php
'api' => [
    'driver' => 'passport',
    'provider' => 'users',
],

'api-customers' => [
    'driver' => 'passport',
    'provider' => 'customers',
],
```

The following route will utilize the `api-customers` guard, which uses the `customers` user provider, to authenticate incoming requests:

```php
Route::get('/customer', function () {
    //
})->middleware('auth:api-customers');
```

> ![](https://laravel.com/img/callouts/lightbulb.min.svg)
>
> For more information on using multiple user providers with Passport, please consult the [password grant documentation](passport.md#customizing-the-user-provider).

{% hint style="info" %}

{% endhint %}

### Passing The Access Token

When calling routes that are protected by Passport, your application's API consumers should specify their access token as a `Bearer` token in the `Authorization` header of their request. For example, when using the Guzzle HTTP library:

```php
$response = $client->request('GET', '/api/user', [
    'headers' => [
        'Accept' => 'application/json',
        'Authorization' => 'Bearer '.$accessToken,
    ],
]);
```

## Token Scopes

Scopes allow your API clients to request a specific set of permissions when requesting authorization to access an account. For example, if you are building an e-commerce application, not all API consumers will need the ability to place orders. Instead, you may allow the consumers to only request authorization to access order shipment statuses. In other words, scopes allow your application's users to limit the actions a third-party application can perform on their behalf.

### Defining Scopes

You may define your API's scopes using the `Passport::tokensCan` method in the `boot` method of your `AuthServiceProvider`. The `tokensCan` method accepts an array of scope names and scope descriptions. The scope description may be anything you wish and will be displayed to users on the authorization approval screen:

```php
use Laravel\Passport\Passport;

Passport::tokensCan([
    'place-orders' => 'Place orders',
    'check-status' => 'Check order status',
]);
```

### Default Scope

If a client does not request any specific scopes, you may configure your Passport server to attach a default scope to the token using the `setDefaultScope` method. Typically, you should call this method from the `boot` method of your `AuthServiceProvider`:

```php
use Laravel\Passport\Passport;

Passport::setDefaultScope([
    'check-status',
    'place-orders',
]);
```

### Assigning Scopes To Tokens

**When Requesting Authorization Codes**

When requesting an access token using the authorization code grant, consumers should specify their desired scopes as the `scope` query string parameter. The `scope` parameter should be a space-delimited list of scopes:

```php
Route::get('/redirect', function () {
    $query = http_build_query([
        'client_id' => 'client-id',
        'redirect_uri' => 'http://example.com/callback',
        'response_type' => 'code',
        'scope' => 'place-orders check-status',
    ]);

    return redirect('http://your-app.com/oauth/authorize?'.$query);
});
```

**When Issuing Personal Access Tokens**

If you are issuing personal access tokens using the `User` model's `createToken` method, you may pass the array of desired scopes as the second argument to the method:

```php
$token = $user->createToken('My Token', ['place-orders'])->accessToken;
```

### Checking Scopes

Passport includes two middleware that may be used to verify that an incoming request is authenticated with a token that has been granted a given scope. To get started, add the following middleware to the `$routeMiddleware` property of your `app/Http/Kernel.php` file:

```php
'scopes' => \Laravel\Passport\Http\Middleware\CheckScopes::class,
'scope' => \Laravel\Passport\Http\Middleware\CheckForAnyScope::class,
```

**Check For All Scopes**

The `scopes` middleware may be assigned to a route to verify that the incoming request's access token has _all_ of the listed scopes:

```php
Route::get('/orders', function () {
    // Access token has both "check-status" and "place-orders" scopes...
})->middleware(['auth:api', 'scopes:check-status,place-orders']);
```

**Check For Any Scopes**

The `scope` middleware may be assigned to a route to verify that the incoming request's access token has _at least one_ of the listed scopes:

```php
Route::get('/orders', function () {
    // Access token has either "check-status" or "place-orders" scope...
})->middleware(['auth:api', 'scope:check-status,place-orders']);
```

**Checking Scopes On A Token Instance**

Once an access token authenticated request has entered your application, you may still check if the token has a given scope using the `tokenCan` method on the authenticated `User` instance:

```php
use Illuminate\Http\Request;

Route::get('/orders', function (Request $request) {
    if ($request->user()->tokenCan('place-orders')) {
        //
    }
});
```

**Additional Scope Methods**

The `scopeIds` method will return an array of all defined IDs / names:

```php
Laravel\Passport\Passport::scopeIds();
```

The `scopes` method will return an array of all defined scopes as instances of `Laravel\Passport\Scope`:

```php
Laravel\Passport\Passport::scopes();
```

The `scopesFor` method will return an array of `Laravel\Passport\Scope` instances matching the given IDs / names:

```php
Laravel\Passport\Passport::scopesFor(['place-orders', 'check-status']);
```

You may determine if a given scope has been defined using the `hasScope` method:

```php
Laravel\Passport\Passport::hasScope('place-orders');
```

## Consuming Your API With JavaScript

When building an API, it can be extremely useful to be able to consume your own API from your JavaScript application. This approach to API development allows your own application to consume the same API that you are sharing with the world. The same API may be consumed by your web application, mobile applications, third-party applications, and any SDKs that you may publish on various package managers.

Typically, if you want to consume your API from your JavaScript application, you would need to manually send an access token to the application and pass it with each request to your application. However, Passport includes a middleware that can handle this for you. All you need to do is add the `CreateFreshApiToken` middleware to your `web` middleware group in your `app/Http/Kernel.php` file:

```php
'web' => [
    // Other middleware...
    \Laravel\Passport\Http\Middleware\CreateFreshApiToken::class,
],
```

> ![](https://laravel.com/img/callouts/exclamation.min.svg)
>
> You should ensure that the `CreateFreshApiToken` middleware is the last middleware listed in your middleware stack.

{% hint style="warning" %}

{% endhint %}

This Passport middleware will attach a `laravel_token` cookie to your outgoing responses. This cookie contains an encrypted JWT that Passport will use to authenticate API requests from your JavaScript application. The JWT has a lifetime equal to your `session.lifetime` configuration value. Now, you may make requests to your application's API without explicitly passing an access token:

```javascript
axios.get('/api/user')
    .then(response => {
        console.log(response.data);
    });
```

**Customizing The Cookie Name**

If needed, you can customize the `laravel_token` cookie's name using the `Passport::cookie` method. Typically, this method should be called from the `boot` method of your `AuthServiceProvider`:

```php
/**
 * Register any authentication / authorization services.
 *
 * @return void
 */
public function boot()
{
    $this->registerPolicies();

    Passport::routes();

    Passport::cookie('custom_name');
}
```

**CSRF Protection**

When using this method of authentication, you will need to ensure a valid CSRF token header is included in your requests. The default Laravel JavaScript scaffolding includes an Axios instance, which will automatically use the encrypted `XSRF-TOKEN` cookie value to send a `X-XSRF-TOKEN` header on same-origin requests.

> ![](https://laravel.com/img/callouts/lightbulb.min.svg)
>
> If you choose to send the `X-CSRF-TOKEN` header instead of `X-XSRF-TOKEN`, you will need to use the unencrypted token provided by `csrf_token()`.

{% hint style="info" %}

{% endhint %}

## Events

Passport raises events when issuing access tokens and refresh tokens. You may use these events to prune or revoke other access tokens in your database. You may attach listeners to these events in your application's `EventServiceProvider`:

```php
/**
 * The event listener mappings for the application.
 *
 * @var array
 */
protected $listen = [
    'Laravel\Passport\Events\AccessTokenCreated' => [
        'App\Listeners\RevokeOldTokens',
    ],

    'Laravel\Passport\Events\RefreshTokenCreated' => [
        'App\Listeners\PruneOldTokens',
    ],
];
```

## Testing

Passport's `actingAs` method may be used to specify the currently authenticated user as well as its scopes. The first argument given to the `actingAs` method is the user instance and the second is an array of scopes that should be granted to the user's token:

```php
use App\User;
use Laravel\Passport\Passport;

public function testServerCreation()
{
    Passport::actingAs(
        factory(User::class)->create(),
        ['create-servers']
    );

    $response = $this->post('/api/create-server');

    $response->assertStatus(201);
}
```

Passport's `actingAsClient` method may be used to specify the currently authenticated client as well as its scopes. The first argument given to the `actingAsClient` method is the client instance and the second is an array of scopes that should be granted to the client's token:

```php
use Laravel\Passport\Client;
use Laravel\Passport\Passport;

public function testGetOrders()
{
    Passport::actingAsClient(
        factory(Client::class)->create(),
        ['check-status']
    );

    $response = $this->get('/api/orders');

    $response->assertStatus(200);
}
```

