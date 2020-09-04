# Начало работы

## Вступление

Laravel делает взаимодействие с базами данных чрезвычайно простым в различных бэкэндах баз данных, используя либо сырой SQL, либо [конструктор запросов,](queries.md) и [Eloquent ORM](.../eloquent/eloquent.md). В настоящее время Laravel поддерживает четыре базы данных:

* MySQL 5.6+ \([Version Policy](https://en.wikipedia.org/wiki/MySQL#Release_history)\)
* PostgreSQL 9.4+ \([Version Policy](https://www.postgresql.org/support/versioning/)\)
* SQLite 3.8.8+
* SQL Server 2017+ \([Version Policy](https://support.microsoft.com/en-us/lifecycle/search)\)

### Конфигурация

Конфигурация базы данных для приложения находится в файле `config/database.php`. В этом файле Вы можете определить все подключения к БД, а также указать, какое подключение должно использоваться по умолчанию. Примеры для большинства поддерживаемых систем баз данных приведены в этом файле.

По умолчанию образец [настройки среды](https://laravel.com/docs/7.x/configuration#environment-configuration) Laravel готов к использованию с [Laravel Homestead](https://laravel.com/docs/7.x/homestead), которая является удобной виртуальной машиной для разработки Laravel на вашей локальной машине. Вы можете свободно изменять эту конфигурацию по мере необходимости для вашей локальной базы данных.

#### SQLite конфигурация

После создания новой базы данных SQLite с помощью команды `touch database/database.sqlite`, вы можете легко настроить переменные окружения так, чтобы они указывали на только что созданную базу данных, используя абсолютный путь к ней:

```bash
DB_CONNECTION=sqlite
DB_DATABASE=/absolute/path/to/database.sqlite
```

Чтобы включить ограничения по внешнему ключу для соединений SQLite, необходимо установить переменную окружения `DB_FOREIGN_KEYS` в `true`:

```bash
DB_FOREIGN_KEYS=true
```

#### Конфигурация с использованием URL-адресов

Обычно соединения с базами данных настраиваются с использованием нескольких значений конфигурации, таких как `host`, `database`, `username`, `password` и др. Каждое из этих значений конфигурации имеет свою собственную соответствующую переменную окружения. Это означает, что при конфигурировании информации о подключении к базе данных на производственном сервере, необходимо управлять несколькими переменными окружения.

Некоторые провайдеры управляемых баз данных, такие как Heroku, предоставляют единый "URL" базы данных, который содержит всю информацию о соединении для БД в одной строке. Пример URL базы данных может выглядеть следующим образом:

```bash
mysql://root:password@127.0.0.1/forge?charset=UTF-8
```

Эти URL обычно следуют стандартной схеме:

```bash
driver://username:password@host:port/database?options
```

Для удобства Laravel поддерживает эти URL-адреса в качестве альтернативы настройке вашей базы данных с несколькими вариантами конфигурации. Если присутствует конфигурационная опция `url` \(или соответствующая пересменная среды `DATABASE_URL`\), то она будет использована для извлечения информации о подключении к БД и учетных данных.

### Раздельные соединения для чтения и записи

Иногда вам нужно использовать одно подключение к базе данных для чтения \(SELECT\), а другое — для записи \(INSERT, UPDATE и DELETE\). Laravel делает это простым, и правильные соединения всегда будут использоваться, независимо от того, используете ли вы необработанные запросы, конструктор запросов или Eloquent ORM.

Чтобы посмотреть, как должны быть настроены соединения чтения/записи, давайте посмотрим на этот пример:

```php
'mysql' => [
    'read' => [
        'host' => [
            '192.168.1.1',
            '196.168.1.2',
        ],
    ],
    'write' => [
        'host' => [
            '196.168.1.3',
        ],
    ],
    'sticky' => true,
    'driver' => 'mysql',
    'database' => 'database',
    'username' => 'root',
    'password' => '',
    'charset' => 'utf8mb4',
    'collation' => 'utf8mb4_unicode_ci',
    'prefix' => '',
],
```

Обратите внимание, что в массив конфигурации были добавлены три ключа: `read`, `write` и `sticky`. Ключи `read` и `write` имеют значения массива, содержащего один ключ: `host`. Остальные опции БД для соединений `read` и `write` будут объединены из основного массива `mysql`.

Размещать элементы в массивах `read` и `write` нужно только в том случае, если вы хотите переопределить значения из главного массива. Таким образом, в этом случае, `192.168.1.1` будет использоваться в качестве хоста для соединения "read", а `192.168.1.3` — для соединения "write". Учетные данные базы данных, префикс, набор символов и все остальные опции в основном массиве `mysql` будут совместно использоваться для обоих соединений.

#### Опция sticky

Опция `sticky` - это _необязательное_ значение, которое может быть использовано для немедленного считывания записей, которые были записаны в базу данных в течение текущего цикла запроса. Если опция `sticky` включена и в течение текущего цикла запроса против БД была выполнена операция "запись", то при любых дальнейших операциях "чтения" будет использоваться соединение "запись". Это гарантирует, что любые данные, записанные во время цикла запроса, могут быть немедленно считаны из БД во время того же самого запроса. Вы сами решаете, является ли это желаемым поведением вашего приложения.

### Using Multiple Database Connections

When using multiple connections, you may access each connection via the `connection` method on the `DB` facade. The `name` passed to the `connection` method should correspond to one of the connections listed in your `config/database.php` configuration file:

```php
$users = DB::connection('foo')->select(...);
```

You may also access the raw, underlying PDO instance using the `getPdo` method on a connection instance:

```php
$pdo = DB::connection()->getPdo();
```

## Running Raw SQL Queries

Once you have configured your database connection, you may run queries using the `DB` facade. The `DB` facade provides methods for each type of query: `select`, `update`, `insert`, `delete`, and `statement`.

#### **Running A Select Query**

To run a basic query, you may use the `select` method on the `DB` facade:

```php
<?php

namespace App\Http\Controllers;

use App\Http\Controllers\Controller;
use Illuminate\Support\Facades\DB;

class UserController extends Controller
{
    /**
     * Show a list of all of the application's users.
     *
     * @return Response
     */
    public function index()
    {
        $users = DB::select('select * from users where active = ?', [1]);

        return view('user.index', ['users' => $users]);
    }
}
```

The first argument passed to the `select` method is the raw SQL query, while the second argument is any parameter bindings that need to be bound to the query. Typically, these are the values of the `where` clause constraints. Parameter binding provides protection against SQL injection.

The `select` method will always return an `array` of results. Each result within the array will be a PHP `stdClass` object, allowing you to access the values of the results:

```php
foreach ($users as $user) {
    echo $user->name;
}
```

#### **Using Named Bindings**

Instead of using `?` to represent your parameter bindings, you may execute a query using named bindings:

```php
$results = DB::select('select * from users where id = :id', ['id' => 1]);
```

#### **Running An Insert Statement**

To execute an `insert` statement, you may use the `insert` method on the `DB` facade. Like `select`, this method takes the raw SQL query as its first argument and bindings as its second argument:

```php
DB::insert('insert into users (id, name) values (?, ?)', [1, 'Dayle']);
```

#### **Running An Update Statement**

The `update` method should be used to update existing records in the database. The number of rows affected by the statement will be returned:

```php
$affected = DB::update('update users set votes = 100 where name = ?', ['John']);
```

#### **Running A Delete Statement**

The `delete` method should be used to delete records from the database. Like `update`, the number of rows affected will be returned:

```php
$deleted = DB::delete('delete from users');
```

#### **Running A General Statement**

Some database statements do not return any value. For these types of operations, you may use the `statement` method on the `DB` facade:

```php
DB::statement('drop table users');
```

## Listening For Query Events

If you would like to receive each SQL query executed by your application, you may use the `listen` method. This method is useful for logging queries or debugging. You may register your query listener in a [service provider](https://laravel.com/docs/7.x/providers):

```php
<?php

namespace App\Providers;

use Illuminate\Support\Facades\DB;
use Illuminate\Support\ServiceProvider;

class AppServiceProvider extends ServiceProvider
{
    /**
     * Register any application services.
     *
     * @return void
     */
    public function register()
    {
        //
    }

    /**
     * Bootstrap any application services.
     *
     * @return void
     */
    public function boot()
    {
        DB::listen(function ($query) {
            // $query->sql
            // $query->bindings
            // $query->time
        });
    }
}
```

## Database Transactions

You may use the `transaction` method on the `DB` facade to run a set of operations within a database transaction. If an exception is thrown within the transaction `Closure`, the transaction will automatically be rolled back. If the `Closure` executes successfully, the transaction will automatically be committed. You don't need to worry about manually rolling back or committing while using the `transaction` method:

```php
DB::transaction(function () {
    DB::table('users')->update(['votes' => 1]);

    DB::table('posts')->delete();
});
```

#### **Handling Deadlocks**

The `transaction` method accepts an optional second argument which defines the number of times a transaction should be reattempted when a deadlock occurs. Once these attempts have been exhausted, an exception will be thrown:

```php
DB::transaction(function () {
    DB::table('users')->update(['votes' => 1]);

    DB::table('posts')->delete();
}, 5);
```

#### **Manually Using Transactions**

If you would like to begin a transaction manually and have complete control over rollbacks and commits, you may use the `beginTransaction` method on the `DB` facade:

```php
DB::beginTransaction();
```

You can rollback the transaction via the `rollBack` method:

```php
DB::rollBack();
```

Lastly, you can commit a transaction via the `commit` method:

```php
DB::commit();
```

> ![](https://laravel.com/img/callouts/lightbulb.min.svg)
>
> The `DB` facade's transaction methods control the transactions for both the [query builder](https://laravel.com/docs/7.x/queries) and [Eloquent ORM](https://laravel.com/docs/7.x/eloquent).

{% hint style="info" %}

{% endhint %}

