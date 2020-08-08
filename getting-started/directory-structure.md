# Структура директорий

## Вступление

Структура приложения Laravel по умолчанию предназначена для обеспечения отличной отправной точки как для больших, так и для маленьких приложений. Но вы можете организовать свое приложение так, как вам нравится. Laravel практически не накладывает никаких ограничений на то, где находится тот или иной класс —до тех пор, пока Composer может автоматически загрузить класс.

**Где находится каталог моделей?**

Приступая к работе с Laravel, многие разработчики путаются из-за отсутствия каталога `models`. Однако отсутствие такого каталога является преднамеренным. Слово "модели" мы находим неоднозначным, так как для многих людей оно означает много разных вещей. Одни разработчики называют "модель" приложения как совокупность всей его бизнес-логики, другие - "модели" как классы, взаимодействующие с реляционной базой данных.

По этой причине по умолчанию мы решили поместить модели Eloquent в каталог `app` и позволить разработчику разместить их где-нибудь в другом месте, если он захочет.

## Корневой каталог

### **Каталог `App`**

Каталог `app` содержит основной код вашего приложения. Ниже мы рассмотрим этот каталог более подробно. Почти все классы вашего приложения будут находиться в этом каталоге.

### **Каталог `Bootstrap`**

Каталог `bootstrap` содержит файл `app.php`, который загружает фреймворк. Здесь также  присутствует каталог `cache`, который содержит файлы, сгенерированные фреймворком для оптимизации производительности.

### **Каталог `Config`**

Каталог `config`, как следует из названия, содержит все конфигурационные файлы приложения. Это отличная идея, чтобы прочитать все эти файлы и ознакомиться со всеми доступными опциями.

### **Каталог `Database`**

В каталоге `database` содержатся миграции базы данных, фабрики моделей и сиды. При желании вы также можете использовать этот каталог для хранения БД SQLite.

### **Каталог `Public`**

Каталог `public` содержит файл `index.php`, который является точкой входа для всех запросов, поступающих в ваше приложение, и настраивает автозагрузку. Этот каталог также содержит статичные файлы, такие как изображения, JavaScript и CSS.

### **Каталог `Resources`**

Каталог `resources` содержит ваши представления \(views\), а также ваши необработанные, некомпилированные ассеты, такие как LESS, SASS или JavaScript. Здесь также располагаются все языковые файлы.

### **Каталог `Routes`**

Каталог `routes` содержит все определения маршрутов для приложения. По умолчанию в Laravel включено несколько файлов с маршрутами: `web.php`, `api.php`, `console.php` и `channels.php`.

Файл `web.php` содержит маршруты, которые `RouteServiceProvider` помещает в группу веб-посредников, обеспечивающую состояние сессии, защиту от CSRF и шифрование cookie-файлов. Если ваше приложение не предлагает RESTful API, все ваши маршруты, скорее всего, будут определены в файле `web.php`.

Файл `api.php` содержит маршруты, которые `RouteServiceProvider` помещает в группу api-посредников, что обеспечивает ограничение скорости. Эти маршруты предназначены для апатридов, поэтому запросы, поступающие в приложение по этим маршрутам, предназначены для аутентификации с помощью токенов и не будут иметь доступа к состоянию сессии.

В файле `console.php` вы можете определить консольные команды, основанные на Closure. Каждая Closure привязана к экземпляру команды, что позволяет использовать простой подход к взаимодействию с IO-методами каждой команды. Несмотря на то, что этот файл не определяет HTTP-маршруты, он определяет консольные точки входа \(маршруты\) в ваше приложение.

В файле `channels.php` вы можете зарегистрировать все каналы трансляции событий, которые поддерживает ваше приложение.

### **Каталог `Storage`**

Каталог `storage` содержит скомпилированные шаблоны Blade, файловые сессии, файловые кэши и другие файлы, сгенерированные фреймворком. Этот каталог разделен на каталоги `app`, `framework` и `logs`. Каталог `app` может использоваться для хранения любых файлов, сгенерированных вашим приложением. Каталог `framework` используется для хранения файлов и кэшей, сгенерированных фреймворком. Наконец, каталог `logs` содержит файлы журналов вашего приложения.

Каталог `storage/app/public` может использоваться для хранения пользовательских файлов, таких как профильные аватары, которые должны быть общедоступны. Вы должны создать на `public/storage` символическую ссылку, указывающую на этот каталог. Вы можете создать ссылку, используя команду `php artisan storage:link`.

### **Каталог `Tests`**

Каталог `tests` содержит автоматизированные тесты. Пример теста PHPUnit предоставляется из коробки. Каждый класс теста должен быть дополнен словом Test. Вы можете запускать свои тесты, используя команды `phpunit` или `php vendor/bin/phpunit`.

### **Каталог `Vendor`**

Каталог `vendor` содержит [Composer](https://getcomposer.org/) зависимости.

## Каталог приложения

Большая часть вашего приложения находится в каталоге `app`. По умолчанию этот каталог находится в пространстве имен `App` и автоматически загружается Composer по [стандарту автозагрузки PSR-4](https://www.php-fig.org/psr/psr-4/).

Каталог `app` содержит множество дополнительных каталогов, таких как `Console`, `Http` и `Providers`. Считайте, что каталоги `Console` и `Http` предоставляют API в ядро вашего приложения. Протокол HTTP и CLI являются механизмами взаимодействия с приложением, но на самом деле не содержат логики приложения. Другими словами, они являются двумя способами выдачи команд приложению. Каталог `Console` содержит команды Artisan, в то время как каталог `Http` содержит контроллеры, посредники и запросы.

Внутри каталога приложения будет сгенерирован ряд других каталогов, так как для генерации классов вы используете Artisan команды `make`. Так, например, каталог `app/Jobs` не будет существовать до тех пор, пока вы не выполните Artisan команду `make:job` для генерации класса job.

{% hint style="info" %}
Многие из классов в каталоге приложений могут быть сгенерированы Artisan с помощью команд. Чтобы просмотреть доступные команды, выполните `php artisan list make` в терминале.
{% endhint %}

### **Каталог `Broadcasting`**

The `Broadcasting` directory contains all of the broadcast channel classes for your application. These classes are generated using the `make:channel` command. This directory does not exist by default, but will be created for you when you create your first channel. To learn more about channels, check out the documentation on [event broadcasting](https://laravel.com/docs/7.x/broadcasting).

### **Каталог `Console`**

The `Console` directory contains all of the custom Artisan commands for your application. These commands may be generated using the `make:command` command. This directory also houses your console kernel, which is where your custom Artisan commands are registered and your [scheduled tasks](https://laravel.com/docs/7.x/scheduling) are defined.

### **Каталог `Events`**

This directory does not exist by default, but will be created for you by the `event:generate` and `make:event` Artisan commands. The `Events` directory houses [event classes](https://laravel.com/docs/7.x/events). Events may be used to alert other parts of your application that a given action has occurred, providing a great deal of flexibility and decoupling.

### **Каталог `Exceptions`**

The `Exceptions` directory contains your application's exception handler and is also a good place to place any exceptions thrown by your application. If you would like to customize how your exceptions are logged or rendered, you should modify the `Handler` class in this directory.

### **Каталог `Http`**

The `Http` directory contains your controllers, middleware, and form requests. Almost all of the logic to handle requests entering your application will be placed in this directory.

### **Каталог `Jobs`**

This directory does not exist by default, but will be created for you if you execute the `make:job` Artisan command. The `Jobs` directory houses the [queueable jobs](https://laravel.com/docs/7.x/queues) for your application. Jobs may be queued by your application or run synchronously within the current request lifecycle. Jobs that run synchronously during the current request are sometimes referred to as "commands" since they are an implementation of the [command pattern](https://en.wikipedia.org/wiki/Command_pattern).

### **Каталог `Listeners`**

This directory does not exist by default, but will be created for you if you execute the `event:generate` or `make:listener` Artisan commands. The `Listeners` directory contains the classes that handle your [events](https://laravel.com/docs/7.x/events). Event listeners receive an event instance and perform logic in response to the event being fired. For example, a `UserRegistered` event might be handled by a `SendWelcomeEmail` listener.

### **Каталог `Mail`**

This directory does not exist by default, but will be created for you if you execute the `make:mail` Artisan command. The `Mail` directory contains all of your classes that represent emails sent by your application. Mail objects allow you to encapsulate all of the logic of building an email in a single, simple class that may be sent using the `Mail::send` method.

### **Каталог `Notifications`**

This directory does not exist by default, but will be created for you if you execute the `make:notification` Artisan command. The `Notifications` directory contains all of the "transactional" notifications that are sent by your application, such as simple notifications about events that happen within your application. Laravel's notification features abstracts sending notifications over a variety of drivers such as email, Slack, SMS, or stored in a database.

### **Каталог `Policies`**

This directory does not exist by default, but will be created for you if you execute the `make:policy` Artisan command. The `Policies` directory contains the authorization policy classes for your application. Policies are used to determine if a user can perform a given action against a resource. For more information, check out the [authorization documentation](https://laravel.com/docs/7.x/authorization).

### **Каталог `Providers`**

The `Providers` directory contains all of the [service providers](https://laravel.com/docs/7.x/providers) for your application. Service providers bootstrap your application by binding services in the service container, registering events, or performing any other tasks to prepare your application for incoming requests.

In a fresh Laravel application, this directory will already contain several providers. You are free to add your own providers to this directory as needed.

### **Каталог `Rules`**

This directory does not exist by default, but will be created for you if you execute the `make:rule` Artisan command. The `Rules` directory contains the custom validation rule objects for your application. Rules are used to encapsulate complicated validation logic in a simple object. For more information, check out the [validation documentation](https://laravel.com/docs/7.x/validation).

