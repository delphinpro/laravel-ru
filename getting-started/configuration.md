# Конфигурация

## Вступление

Все файлы конфигурации для фреймворка Laravel хранятся в каталоге `config`. Каждая опция документирована, так что не стесняйтесь просматривать файлы и знакомиться с доступными вам опциями.

## Конфигурация среды

Часто бывает полезно иметь различные значения конфигурации в зависимости от среды, в которой запущено приложение. Например, вы можете захотеть использовать другой драйвер кэша локально, нежели на вашем рабочем сервере.

Чтобы сделать это возможным, Laravel использует PHP-библиотеку [DotEnv](https://github.com/vlucas/phpdotenv) Вэнса Лукаса \(Vance Lucas\). В новой установке Laravel корневая директория вашего приложения будет содержать файл `.env.example`. Если вы установите Laravel через Composer, этот файл будет автоматически скопирован в `.env`. В противном случае, вы должны скопировать файл вручную.

Ваш файл `.env` не должен быть включен в систему контроля версий, так как для каждого разработчика/сервера, использующего ваше приложение, может потребоваться своя конфигурация среды. Более того, это будет представлять собой риск для безопасности в случае, если злоумышленник получит доступ к вашему репозиторию, так как любые конфиденциальные учетные данные будут раскрыты.

Если вы ведете разработку в команде, то можете включать файл с примером `.env.example` в ваше приложение. Поместив значения переменных окружения в пример конфигурационного файла, другие разработчики в вашей команде смогут четко увидеть, какие переменные окружения необходимы для запуска вашего приложения. Вы также можете создать файл `.env.testing`. Этот файл переопределит .env-файл при запуске тестов PHPUnit или выполнении команд Artisan с опцией `--env=testing`.

{% hint style="info" %}
Любая переменная в файле `.env` может быть переопределена внешними переменными окружения, такими как переменные окружения на уровне сервера или системы.
{% endhint %}

### Типы переменных среды

Все переменные в ваших .env-файлах анализируются как строки, поэтому были созданы некоторые зарезервированные значения, чтобы позволить вам вернуть более широкий диапазон типов из функции `env()`:

| `.env` Value | `env()` Value |
| :--- | :--- |
| `true` | `(bool) true` |
| `(true)` | `(bool) true` |
| `false` | `(bool) false` |
| `(false)` | `(bool) false` |
| `empty` | `(string) ''` |
| `(empty)` | `(string) ''` |
| `null` | `(null) null` |
| `(null)` | `(null) null` |

Если вам нужно определить переменную окружения со значением, содержащим пробелы, вы можете сделать это, заключив значение в двойные кавычки.

```text
APP_NAME="My Application"
```

### Получение конфигурации среды

Все переменные, перечисленные в этом файле, будут загружены в супер глобальную PHP-переменную `$_ENV` , когда ваше приложение получит запрос. Тем не менее, вы можете использовать функцию-помощник `env()` для получения значений из этих переменных в ваших конфигурационных файлах. Фактически, если вы просмотрите конфигурационные файлы Laravel, вы заметите несколько вариантов, уже использующих этот помощник:

```php
'debug' => env('APP_DEBUG', false),
```

Второе значение, переданное в функцию `env`, является "значением по умолчанию". Это значение будет использовано, если для данного ключа не существует переменной окружения.

### Определение текущей среды

Текущее окружение приложения определяется с помощью переменной `APP_ENV` из вашего `.env` файла. Доступ к этому значению можно получить методом `environment` фасада `App`:

```php
$environment = App::environment();
```

Также Вы можете передать аргументы в метод `environment`, чтобы проверить, соответствует ли окружение заданному значению. Метод вернет `true`, если окружение соответствует любому из заданных значений:

```php
if (App::environment('local')) {
    // The environment is local
}

if (App::environment(['local', 'staging'])) {
    // The environment is either local OR staging...
}
```

{% hint style="info" %}
Определение текущей среды приложений может быть переопределено переменной окружения `APP_ENV` на уровне сервера. Это может быть полезно, когда вам нужно совместно использовать одно и то же приложение для различных конфигураций среды, поэтому вы можете настроить заданный хост для соответствия заданной среды в конфигурации вашего сервера.
{% endhint %}

### Скрытие переменных окружения из отладочной информации

Когда исключение не поймано и переменная окружения `APP_DEBUG` имеет значение `true`, на отладочной странице будут показаны все переменные окружения и их содержимое. В некоторых случаях вам может понадобиться скрыть некоторые переменные. Это можно сделать, обновив опцию `debug_hide` в конфигурационном файле `config/app.php`.

Некоторые переменные доступны как в переменных окружения, так и в данных сервера / запроса. Поэтому, возможно, вам понадобится занести их в черный список как для `$_ENV`, так и для `$_SERVER`:

```php
return [

    // ...

    'debug_hide' => [
        '_ENV' => [
            'APP_KEY',
            'DB_PASSWORD',
        ],

        '_SERVER' => [
            'APP_KEY',
            'DB_PASSWORD',
        ],

        '_POST' => [
            'password',
        ],
    ],
];
```

## Доступ к значениям настроек

Вы можете легко получить доступ к своим значениям конфигурации с помощью глобальной функции-помощника `config()` из любой точки вашего приложения. Доступ к значениям конфигурации можно получить, используя синтаксис "точка", который включает в себя имя файла и опцию, к которой вы хотите получить доступ. Значение по умолчанию также может быть указано и будет возвращено, если опция не существует:

```php
$value = config('app.timezone');

// Retrieve a default value if the configuration value does not exist...
$value = config('app.timezone', 'Asia/Seoul');
```

Чтобы установить значения опции во время выполнения, передайте массив в функцию `config`:

```php
config(['app.timezone' => 'America/Chicago']);
```

## Кэширование настроек

Для ускорения работы приложения необходимо кэшировать все конфигурационные файлы в один файл с помощью Artisan-команды `config:cache`. Это объединит все параметры конфигурации вашего приложения в один файл, который будет быстро загружен фреймворком.

Обычно вы должны запустить команду `php artisan config:cache` как часть вашего развертывания в production-окружении. Команду не следует запускать во время локальной разработки, так как опции конфигурации будет нужно часто менять в процессе.

{% hint style="warning" %}
Если вы выполняете команду `config:cache` во время процесса развертывания, то должны быть уверены, что используете вызов функции `env` только в своих конфигах. После того, как конфигурация будет кэширована, `.env` файл не будет загружаться, и все вызовы функции `env` вернут `null`.
{% endhint %}

## Режим обслуживания

Когда ваше приложение находится в режиме обслуживания, для всех запросов в приложение будет отображаться пользовательский вид. Это позволяет легко "отключить" ваше приложение во время его обновления или при выполнении обслуживания. Проверка режима обслуживания включена в стандартный стек посредников приложения. Если приложение находится в режиме обслуживания, будет брошено исключение `MaintenanceModeException` с кодом состояния 503.

Чтобы включить режим обслуживания, выполните Artisan-команду `down`:

```text
php artisan down
```

Вы также можете передать команде опции `message` и `retry`. Значение `message` может быть использовано для отображения или записи в журнал, а значение `retry` будет установлено в качестве значения HTTP заголовка "Retry-After":

```text
php artisan down --message="Upgrading Database" --retry=60
```

Даже в режиме обслуживания определенным IP-адресам или сетям может быть разрешен доступ к приложению с помощью опции `allow`:

```text
php artisan down --allow=127.0.0.1 --allow=192.168.0.0/16
```

Чтобы отключить режим обслуживания, используйте команду `up`:

```text
php artisan up
```

{% hint style="info" %}
Вы можете изменить шаблон режима обслуживания по умолчанию, определив свой собственный шаблон в файле `resources/views/errors/503.blade.php`.
{% endhint %}

**Режим обслуживания и очереди**

Пока ваше приложение находится в режиме обслуживания, не будут обрабатываться [задания очередей](../digging-deeper/queues.md). Задания будут обрабатываться в обычном режиме, как только приложение выйдет из режима обслуживания.

**Альтернативы режиму обслуживания**

Поскольку режим обслуживания требует от Вашего приложения нескольких секунд простоя, рассмотрите альтернативные варианты, такие как [Envoyer](https://envoyer.io/), для достижения нулевого простоя развертывания с Laravel.

