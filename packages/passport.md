# Laravel Passport

## Вступление

Laravel уже упрощает выполнение аутентификации с помощью традиционных форм входа, но как насчет API? API обычно используют токены для аутентификации пользователей и не поддерживают состояние сеанса между запросами. Laravel делает аутентификацию API легкой задачей, используя Laravel Passport, который обеспечивает полную реализацию сервера OAuth2 для вашего приложения в течение нескольких минут. Passport построен на [OAuth2 сервере](https://github.com/thephpleague/oauth2-server), который поддерживается Энди Миллингтоном \(Andy Millington\) и Саймоном Хэмпом \(Simon Hamp\).

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

Далее следует запустить команду `passport:install`. Эта команда создаст ключи шифрования, необходимые для генерации маркеров безопасного доступа. Кроме того, команда создаст клиентов "персонального доступа \(personal access\)" и "выдачи пароля \(password grant\)", которые будут использоваться для генерации токенов доступа:

```bash
php artisan passport:install
```

{% hint style="info" %}
Если вы хотите использовать UUID в качестве значения первичного ключа \(primary key\) модели `Client` Passport'a вместо автоинкрементируемых целых чисел, пожалуйста, установите Passport, используя \[опцию `uuids`\] \(passport.md\#client-uuids\).
{% endhint %}

После выполнения команды `passport:install` добавьте трейт `Laravel\Passport\HasApiTokens` к вашей модели `App\User`. Этот трейт предоставит несколько вспомогательных методов для модели, которые позволят проверять токен аутентифицированного пользователя и области видимости \(scopes\):

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

Этот маршрут возвращает всех клиентов аутентифицированному пользователю. Это в первую очередь полезно для перечисления всех клиентов пользователя, чтобы он мог их редактировать или удалять:

```javascript
axios.get('/oauth/clients')
    .then(response => {
        console.log(response.data);
    });
```

**POST /oauth/clients**

Этот маршрут используется для создания новых клиентов. Для него требуются две части данных: имя клиента `name` и `redirect` — URL для перенаправления. URL-адрес `redirect` — это то место, куда пользователь будет перенаправлен после одобрения или отклонения запроса на авторизацию.

При создании клиента ему будет выдан идентификатор и секрет клиента. Эти значения будут использоваться при запросе токенов доступа из вашего приложения. Маршрут создания клиента вернет новый экземпляр клиента:

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

Этот маршрут используется для обновления клиентов. Для него требуются две части данных: имя клиента `name` и `redirect` — URL для редиректа. URL-адрес `redirect` — это то место, куда пользователь будет перенаправлен после одобрения или отклонения запроса на авторизацию. Маршрут вернет обновленный экземпляр клиента:

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

Этот маршрут используется для удаления клиентов:

```javascript
axios.delete('/oauth/clients/' + clientId)
    .then(response => {
        //
    });
```

### Запрос токенов

**Переадресация для авторизации**

После создания клиента разработчики могут использовать свой клиентский ID и секрет для запроса кода авторизации и токена доступа из вашего приложения. Сначала запрашивающее приложение должно переадресовать запрос на маршрут `/oauth/authorize` вашего приложения таким образом:

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

{% hint style="info" %}
Помните, что маршрут `/oauth/authorize` уже определен методом `Passport::routes`. Вам не нужно вручную определять этот маршрут.
{% endhint %}

**Одобрение запроса**

При получении запроса на авторизацию Паспорт автоматически выводит пользователю шаблон, позволяющий ему одобрить или отклонить запрос на авторизацию. Если запрос на авторизацию будет одобрен, то он будет перенаправлен обратно на `redirect_uri`, который был указан запрашивающим приложением. URL-адрес `redirect_uri` должен совпадать с URL-адресом `redirect`, который был указан при создании клиента.

Если вы хотите настроить экран одобрения авторизации, то можете опубликовать представления Паспорта, используя команду Artisan `vendor:publish`. Опубликованные представления будут размещены в `resources/views/vendor/passport`:

```bash
php artisan vendor:publish --tag=passport-views
```

Иногда вам может понадобиться пропустить запрос авторизации, например, при авторизации клиента первой стороны. Вы можете сделать это, [расширив модель `Clients`](passport.md#overriding-default-models) и определив метод `skipsAuthorization`. Если `skipsAuthorization` вернёт `true`, то клиент будет одобрен и пользователь будет немедленно перенаправлен обратно в `redirect_uri`:

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

**Преобразование кодов авторизации в токены доступа**

Если пользователь одобрит запрос на авторизацию, то он будет перенаправлен обратно в приложение-потребитель. Потребитель должен сначала проверить параметр `state` в сравнении со значением, которое было сохранено до перенаправления. Если параметр "state" соответствует потребителю, то он должен отправить в Ваше приложение запрос "POST" на получение токена доступа. В запросе должен быть указан код авторизации, который был выдан вашим приложением при одобрении запроса на авторизацию пользователем. В данном примере мы будем использовать HTTP-библиотеку Guzzle для создания `POST` запроса:

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

Этот маршрут `/oauth/token` вернет JSON-ответ, содержащий атрибуты `access_token`, `refresh_token` и `expires_in`. Атрибут `expires_in` содержит количество секунд до истечения срока действия токена доступа.

{% hint style="info" %}
Как и маршрут `/auth/authorize`, маршрут `/oauth/token` определяется для вас методом `Passport::routes`. Нет необходимости вручную определять этот маршрут. По умолчанию этот маршрут дросселируется с помощью настроек посредника `ThrottleRequests`.
{% endhint %}

**JSON API**

Паспорт также включает JSON API для управления токенами авторизованного доступа. Вы можете связать его с вашим собственным фронтендом, чтобы предложить вашим пользователям панель инструментов для управления токенами доступа. Для удобства мы используем [Axios](https://github.com/mzabriskie/axios), чтобы продемонстрировать выполнение HTTP-запросов к конечным точкам. JSON API защищен посредниками `web` и `auth`, поэтому он может быть вызван только из вашего собственного приложения.

**GET /oauth/tokens**

Этот маршрут возвращает все токены авторизованного доступа, созданные аутентифицированным пользователем. Это в первую очередь полезно для перечисления всех токенов пользователя, чтобы он мог их отозвать:

```javascript
axios.get('/oauth/tokens')
    .then(response => {
        console.log(response.data);
    });
```

**DELETE /oauth/tokens/{token-id}**

Этот маршрут может использоваться для отзыва токенов авторизованного доступа и связанных с ними токенов обновления:

```javascript
axios.delete('/oauth/tokens/' + tokenId);
```

### Токены обновления

Если ваше приложение выпускает кратковременные токены доступа, пользователям необходимо будет обновлять свои токены доступа с помощью токена обновления, который был предоставлен им при выпуске токена доступа. В этом примере мы будем использовать HTTP-библиотеку Guzzle для обновления токена:

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

Этот маршрут `/oauth/token` вернет JSON-ответ, содержащий атрибуты `access_token`, `refresh_token` и `expires_in`. Атрибут `expires_in` содержит количество секунд до истечения срока действия токена доступа.

### Отзыв токенов

Вы можете отозвать токен, используя метод `revokeAccessToken` в `TokenRepository`. Отзыв токена осуществляется методом `revokeRefreshTokensByAccessTokenId` в `RefreshTokenRepository`:

```php
$tokenRepository = app('Laravel\Passport\TokenRepository');
$refreshTokenRepository = app('Laravel\Passport\RefreshTokenRepository');

// Отозвать токен доступа...
$tokenRepository->revokeAccessToken($tokenId);

// Revoke all of the token's refresh tokens...
// Отозвать все обновляющие токены токена доступа...
$refreshTokenRepository->revokeRefreshTokensByAccessTokenId($tokenId);
```

### Очистка токенов

Когда маркеры отозваны или просрочены, возможно, вы захотите удалить их из базы данных. Паспорт поставляется с командой, которая может сделать это за вас:

```bash
# Очистка отозванных и истекших токенов и кодов авторизации...
php artisan passport:purge

# Очистка только отозванных токенов и кодов авторизации...
php artisan passport:purge --revoked

# Очистка только истекших токенов и кодов авторизации...
php artisan passport:purge --expired
```

Вы также можете настроить [задание по расписанию](https://github.com/delphinpro/laravel-ru/tree/bdac4480dcc41bf18a88ab5a393bc58a445f7814/packages/scheduling/README.md) в классе `Kernel` вашей консоли, чтобы автоматически удалять токены по расписанию:

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

## Код авторизации Грант с PKCE

Код авторизации с помощью "Proof Key for Code Exchange" \(PKCE\) является безопасным способом аутентификации одностраничных приложений или обычных приложений для доступа к вашему API. Этот грант должен использоваться, когда вы не можете гарантировать, что клиентская тайна будет храниться конфиденциально, или для уменьшения угрозы того, что код авторизации будет перехвачен злоумышленником. Комбинация "верификатор кода" и "вызов кода" заменяет клиентский секрет при обмене кода авторизации на токен доступа.

### Создание клиента

Прежде чем ваше приложение сможет выпускать токены с помощью кода авторизации, выданного с помощью PKCE, вам нужно будет создать клиент с поддержкой PKCE. Вы можете сделать это, используя команду `passport:client` с опцией `-public`:

```bash
php artisan passport:client --public
```

### Запрос токенов

**Верификатор кода и код вызова**

Поскольку данное разрешение на авторизацию не предоставляет секрета клиента, разработчикам необходимо будет сгенерировать комбинацию верификатора кода и вызова кода, чтобы запросить токен.

Верификатор кода должен представлять собой случайную строку от 43 до 128 символов, содержащую буквы, цифры и `"-"`, `"."`, `"_"`, `"~"`, как определено в [спецификации RFC 7636](https://tools.ietf.org/html/rfc7636).

Кодом вызова должна быть Base64-кодированная строка с URL и символами, безопасными для файлов. Оконечные символы `'='` должны быть удалены, и не должно быть ни разрывов строк, ни пробелов, ни других дополнительных символов.

```php
$encoded = base64_encode(hash('sha256', $code_verifier, true));

$codeChallenge = strtr(rtrim($encoded, '='), '+/', '-_');
```

**Переадресация для авторизации**

После создания клиента вы можете использовать идентификатор клиента и сгенерированный верификатор кода и код вызова для запроса кода авторизации и токена доступа из вашего приложения. Сначала клиентское приложение должно сделать запрос перенаправления на маршрут `/oauth/authorize` вашего приложения:

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

**Преобразование кодов авторизации в токены доступа**

Если пользователь одобрит запрос на авторизацию, то он будет перенаправлен обратно в клиентское приложение. Оно должено сравнить параметр `state` со значением, которое было сохранено до перенаправления, как в стандартном разрешении на код авторизации.

Если параметр состояния совпадает, клиент должен отправить в Ваше приложение `POST` запрос на запрос токена доступа. В запросе должен быть указан код авторизации, который был выдан вашим приложением при одобрении запроса на авторизацию пользователем, а также изначально сгенерированный верификатор кода:

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

If your application uses more than one [authentication user provider](https://github.com/delphinpro/laravel-ru/tree/310ae4ba9e9192a52b44ffbd0d02380355505025/packages/authentication/README.md#introduction), you may specify which user provider the password grant client uses by providing a `--provider` option when creating the client via the `artisan passport:client --password` command. The given provider name should match a valid provider defined in your `config/auth.php` configuration file. You can then [protect your route using middleware](passport.md#via-middleware) to ensure that only users from the guard's specified provider are authorized.

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

Passport includes an [authentication guard](https://github.com/delphinpro/laravel-ru/tree/310ae4ba9e9192a52b44ffbd0d02380355505025/packages/authentication/README.md#adding-custom-guards) that will validate access tokens on incoming requests. Once you have configured the `api` guard to use the `passport` driver, you only need to specify the `auth:api` middleware on any routes that require a valid access token:

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

