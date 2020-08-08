# Установка

## Требования к серверу

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

## Установка Laravel

Laravel использует Composer для управления своими зависимостями. Поэтому, прежде чем использовать Laravel, убедитесь, что на вашем компьютере установлен Composer.

### **С помощью Laravel Installer**

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

### **С помощью Composer Create-Project**

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

## Конфигурация

### **Публичный каталог**

После установки Laravel, вы должны настроить веб-корень \(DOCUMENT\_ROOT\) вашего веб-сервера, чтобы он указывал на каталог `public`. Файл `index.php` в этой директории служит в качестве фронт-контроллера для всех HTTP-запросов, поступающих к вашему приложению.

### Файлы конфигурации

Все файлы конфигурации для фреймворка Laravel хранятся в каталоге `config`. Каждая опция документирована, так что не стесняйтесь просматривать файлы и знакомиться с доступными опциями.

### **Права доступа на каталоги**

После установки Laravel вам, возможно, понадобится настроить некоторые разрешения. Директории внутри `storage` и `bootstrap/cache` должны быть доступны для записи Вашим веб-сервером, иначе Laravel не будет работать. Если Вы используете виртуальную машину [Homestead](https://laravel.com/docs/7.x/homestead), эти разрешения уже должны быть установлены.

### **Ключ приложения \(Application Key\)**

Следующее, что вы должны сделать после установки Laravel, это установить ключ приложения на случайную строку. Если вы установили Laravel через Composer или установщик Laravel, этот ключ уже был установлен командой `php artisan key:generate`.

Обычно длина этой строки должна быть 32 символа. Ключ может быть установлен в файле окружения `.env`. Если вы не скопировали файл `.env.example` в новый файл с именем `.env`, вы должны сделать это сейчас. **Если ключ приложения не установлен, ваши пользовательские сессии и другие зашифрованные данные не будут безопасны!**.

### **Дополнительные настройки**

Ларавел "из коробки" практически не нуждается в других настройках. Вы можете начать разработку! Однако, вы можете просмотреть файл `config/app.php` и документацию к нему. Он содержит несколько опций, таких как `timezone` и `locale`, которые Вы, возможно, захотите изменить в соответствии с Вашим приложением.

Вы также можете настроить несколько дополнительных компонентов Laravel, например:

* [Cache](https://github.com/delphinpro/laravel-ru/tree/29222cc3b02433613d72e0c45718a6ce91069487/getting-started/cache.md#configuration)
* [Database](https://github.com/delphinpro/laravel-ru/tree/29222cc3b02433613d72e0c45718a6ce91069487/getting-started/database.md#configuration)
* [Session](https://github.com/delphinpro/laravel-ru/tree/29222cc3b02433613d72e0c45718a6ce91069487/getting-started/session.md#configuration)

## Конфигурация Web-сервера

### Directory Configuration

Laravel всегда должен располагаться вне корня вашего web-сервера. Попытка установить laravel в корень web-сервера может привести к раскрытию конфиденциальных файлов, присутствующих в вашем приложении.

### "Человеческие" URL

**Apache**

Laravel включает файл `public/.htaccess`, который используется для предоставления URL без фронтального контроллера `index.php` в пути. Перед тем, как использовать Laravel с Apache, убедитесь, что включили модуль `mod_rewrite`, чтобы файл `.htaccess` был принят сервером.

Если файл `.htaccess`, который поставляется с Laravel, не работает с вашей установкой Apache, попробуйте этот вариант:

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

Если вы используете Nginx, то следующая директива в конфигурации вашего сайта будет направлять все запросы на фронт-контроллер `index.php`:

```text
location / {
    try_files $uri $uri/ /index.php?$query_string;
}
```

При использовании [Homestead](homestead.md) или [Valet](valet.md), симпатичные URL-адреса будут настроены автоматически.

