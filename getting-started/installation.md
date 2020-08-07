# Установка

### Требования к серверу

Фреймворк Laravel имеет несколько системных требований. Все эти требования выполняются в виртуальной машине [Laravel Homestead](homestead.md), поэтому настоятельно рекомендуется использовать Homestead в качестве локальной среды разработки Laravel.

Если вы не используете Homestead, вам необходимо убедиться, что ваш сервер соответствует следующим требованиям:

* PHP &gt;= 7.2.5
* BCMath PHP Extension
* Ctype PHP Extension
* Fileinfo PHP extension
* JSON PHP Extension
* Mbstring PHP Extension
* OpenSSL PHP Extension
* PDO PHP Extension
* Tokenizer PHP Extension
* XML PHP Extension

### Установка Laravel

Laravel использует Composer для управления своими зависимостями. Поэтому, прежде чем использовать Laravel, убедитесь, что на вашем компьютере установлен Composer.

**С помощью Laravel Installer**

Сначала скачайте Laravel installer, используя Composer:

```text
composer global require laravel/installer
```

Убедитесь, что вы добавили директорию vendor bin Composer\`a в системную переменную среды $PATH, чтобы исполняемый файл laravel мог быть найден вашей системой. Эта директория различается в зависимости от вашей операционной системы; вот некоторые из возможных расположений:

* macOS: `$HOME/.composer/vendor/bin`
* Windows: `%USERPROFILE%\AppData\Roaming\Composer\vendor\bin`
* GNU / Linux Distributions: `$HOME/.config/composer/vendor/bin` или `$HOME/.composer/vendor/bin`

Вы также можете найти путь глобальной инсталляции Composer, выполнив команду `composer global about` и посмотрев первую строку.

После установки команда `laravel new` создаст новую установку Laravel в указанном вами каталоге. Например, `laravel new blog` создаст каталог под названием blog, содержащий свежую установку Laravel со всеми уже установленными зависимостями Laravel:

```text
laravel new blog
```

**С помощью Composer Create-Project**

Также, Вы можете установить Laravel, выполнив команду Composer `create-project` в Вашем терминале:

```text
composer create-project --prefer-dist laravel/laravel blog
```

**Локальный сервер разработки**

Если у вас PHP установлен локально и вы хотите использовать встроенный сервер разработки PHP для разработки приложения, то можете использовать команду Artisan `serve`. Эта команда запустит сервер разработки по адресу `http://localhost:8000`:

```text
php artisan serve
```

Более надежные варианты локальной разработки доступны через [Homestead](homestead.md) и [Valet](valet.md).

### Конфигурация

#### **Публичный каталог**

После установки Laravel, вы должны настроить веб-корень \(DOCUMENT\_ROOT\) вашего веб-сервера, чтобы он указывал на каталог `public`. Файл `index.php` в этой директории служит в качестве фронт-контроллера для всех HTTP-запросов, поступающих к вашему приложению.

#### Файлы конфигурации

Все файлы конфигурации для фреймворка Laravel хранятся в каталоге `config`. Каждая опция документирована, так что не стесняйтесь просматривать файлы и знакомиться с доступными опциями.

#### **Directory Permissions**

After installing Laravel, you may need to configure some permissions. Directories within the `storage` and the `bootstrap/cache` directories should be writable by your web server or Laravel will not run. If you are using the [Homestead](https://laravel.com/docs/7.x/homestead) virtual machine, these permissions should already be set.

After installing Laravel, you may need to configure some permissions. Directories within the `storage` and the `bootstrap/cache` directories should be writable by your web server or Laravel will not run. If you are using the [Homestead](https://laravel.com/docs/7.x/homestead) virtual machine, these permissions should already be set.

#### **Application Key**

The next thing you should do after installing Laravel is set your application key to a random string. If you installed Laravel via Composer or the Laravel installer, this key has already been set for you by the `php artisan key:generate` command.

The next thing you should do after installing Laravel is set your application key to a random string. If you installed Laravel via Composer or the Laravel installer, this key has already been set for you by the `php artisan key:generate` command.

Typically, this string should be 32 characters long. The key can be set in the `.env` environment file. If you have not copied the `.env.example` file to a new file named `.env`, you should do that now. **If the application key is not set, your user sessions and other encrypted data will not be secure!**

Typically, this string should be 32 characters long. The key can be set in the `.env` environment file. If you have not copied the `.env.example` file to a new file named `.env`, you should do that now. **If the application key is not set, your user sessions and other encrypted data will not be secure!**

#### **Additional Configuration**

Laravel needs almost no other configuration out of the box. You are free to get started developing! However, you may wish to review the `config/app.php` file and its documentation. It contains several options such as `timezone` and `locale` that you may wish to change according to your application.

Laravel needs almost no other configuration out of the box. You are free to get started developing! However, you may wish to review the `config/app.php` file and its documentation. It contains several options such as `timezone` and `locale` that you may wish to change according to your application.

* 
You may also want to configure a few additional components of Laravel, such as:

You may also want to configure a few additional components of Laravel, such as:

* [Cache](https://laravel.com/docs/7.x/cache#configuration)
* [Database](https://laravel.com/docs/7.x/database#configuration)
* [Session](https://laravel.com/docs/7.x/session#configuration)

### Конфигурация Web-сервера

#### Directory Configuration

#### Laravel should always be served out of the root of the "web directory" configured for your web server. You should not attempt to serve a Laravel application out of a subdirectory of the "web directory". Attempting to do so could expose sensitive files present within your application. <a id="pretty-urls"></a>

#### Laravel should always be served out of the root of the "web directory" configured for your web server. You should not attempt to serve a Laravel application out of a subdirectory of the "web directory". Attempting to do so could expose sensitive files present within your application. <a id="pretty-urls"></a>

#### Pretty URLs

**Apache**

Laravel includes a `public/.htaccess` file that is used to provide URLs without the `index.php` front controller in the path. Before serving Laravel with Apache, be sure to enable the `mod_rewrite` module so the `.htaccess` file will be honored by the server.

Laravel includes a `public/.htaccess` file that is used to provide URLs without the `index.php` front controller in the path. Before serving Laravel with Apache, be sure to enable the `mod_rewrite` module so the `.htaccess` file will be honored by the server.

If the `.htaccess` file that ships with Laravel does not work with your Apache installation, try this alternative: 

If the `.htaccess` file that ships with Laravel does not work with your Apache installation, try this alternative:

```text
Options +FollowSymLinks -Indexes
RewriteEngine On

RewriteCond %{HTTP:Authorization} .
RewriteRule .* - [E=HTTP_AUTHORIZATION:%{HTTP:Authorization}]

RewriteCond %{REQUEST_FILENAME} !-d
RewriteCond %{REQUEST_FILENAME} !-f
RewriteRule ^ index.php [L]
```

**Nginx**

If you are using Nginx, the following directive in your site configuration will direct all requests to the `index.php` front controller:

If you are using Nginx, the following directive in your site configuration will direct all requests to the `index.php` front controller:

```text
location / {
    try_files $uri $uri/ /index.php?$query_string;
}
```

When using [Homestead](https://laravel.com/docs/7.x/homestead) or [Valet](https://laravel.com/docs/7.x/valet), pretty URLs will be automatically configured.

