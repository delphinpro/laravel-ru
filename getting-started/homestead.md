# Homestead

## Вступление

Laravel стремится сделать разработку удобной, включая вашу локальную среду разработки. [Vagrant](https://www.vagrantup.com/) предоставляет простой, элегантный способ управления и предоставления виртуальных машин.

Laravel Homestead — это официальный, предварительно упакованный Vagrant образ, который предоставляет вам прекрасную среду разработки, не требуя установки PHP, веб-сервера и любого другого серверного программного обеспечения на вашей локальной машине. Больше не нужно беспокоиться о том, чтобы испортить вашу операционную систему! Образы Vagrant полностью утилизируемые. Если что-то пойдет не так, вы можете уничтожить и воссоздать образ за считанные минуты!

Homestead работает на любой системе Windows, Mac или Linux, и включает в себя Nginx, PHP, MySQL, PostgreSQL, Redis, Memcached, Node, и все другие полезные функции, необходимые для разработки приложений Laravel.

{% hint style="info" %}
Если вы используете Windows, вам может потребоваться включить аппаратную виртуализацию \(VT-x\). Обычно она может быть включена через BIOS. Если вы используете Hyper-V на системе UEFI, вам может понадобиться дополнительно отключить Hyper-V, чтобы получить доступ к VT-x.
{% endhint %}

#### Входящее в комплект программное обеспечение

* Ubuntu 18.04
* Git
* PHP 7.4
* PHP 7.3
* PHP 7.2
* PHP 7.1
* PHP 7.0
* PHP 5.6
* Nginx
* MySQL
* lmm for MySQL or MariaDB database snapshots
* Sqlite3
* PostgreSQL \(9.6, 10, 11, 12\)
* Composer
* Node \(With Yarn, Bower, Grunt, and Gulp\)
* Redis
* Memcached
* Beanstalkd
* Mailhog
* avahi
* ngrok
* Xdebug
* XHProf / Tideways / XHGui
* wp-cli

#### Дополнительное ПО

* Apache
* Blackfire
* Cassandra
* Chronograf
* CouchDB
* Crystal & Lucky Framework
* Docker
* Elasticsearch
* Gearman
* Go
* Grafana
* InfluxDB
* MariaDB
* MinIO
* MongoDB
* MySQL 8
* Neo4j
* Oh My Zsh
* Open Resty
* PM2
* Python
* RabbitMQ
* Solr
* Webdriver & Laravel Dusk Utilities

## Установка и настройка

### Первые шаги

Перед запуском среды Homestead необходимо установить [VirtualBox 6.x](https://www.virtualbox.org/wiki/Downloads), [VMWare](https://www.vmware.com/), [Parallels](https://www.parallels.com/products/desktop/) или [Hyper-V](https://docs.microsoft.com/en-us/virtualization/hyper-v-on-windows/quick-start/enable-hyper-v), а также [Vagrant](https://www.vagrantup.com/downloads.html). Все эти программные пакеты предоставляют простые в использовании визуальные инсталляторы для всех популярных операционных систем.

Для использования VMware необходимо приобрести как VMware Fusion / Workstation, так и [VMware Vagrant plug-in](https://www.vagrantup.com/vmware). Хотя это и не бесплатно, VMware может обеспечить более быструю работу с общими папками.

Чтобы воспользоваться Parallels, вам необходимо установить [плагин Parallels Vagrant](https://github.com/Parallels/vagrant-parallels). Это бесплатно.

Из-за [ограничений Vagrant](https://www.vagrantup.com/docs/hyperv/limitations.html) Hyper-V игнорирует все сетевые настройки.

**Установка Homestead Vagrant Box**

После установки VirtualBox / VMware и Vagrant, вы должны добавить `laravel/homestead` в Vagrant, используя следующую команду в терминале. Загрузка займет несколько минут, в зависимости от скорости Вашего подключения к Интернету:

```text
vagrant box add laravel/homestead
```

Если эта команда не работает, убедитесь, что ваш Vagrant обновлен.

{% hint style='info' %}
Homestead периодически выпускает "alpha"/"beta" боксы для тестирования, которые могут помешать команде `vagrant box add`. Если у вас возникли проблемы с запуском `vagrant box add`, вы можете запустить команду `vagrant up` и правильный бокс будет загружен, когда Vagrant попытается запустить виртуальную машину.
{% endhint %}

**Установка Homestead**

Вы можете установить Homestead, клонируя репозиторий на вашу машину. Рассмотрите возможность клонирования репозитория в папку `Homestead` в вашем "домашнем" каталоге, так как бокс Homestead будет служить хостом для всех ваших проектов Laravel:

```text
git clone https://github.com/laravel/homestead.git ~/Homestead
```

Вам следует проверить версию Homestead с меткой, так как ветка `master` не всегда может быть стабильной. Вы можете найти последнюю стабильную версию на [GitHub Release Page](https://github.com/laravel/homestead/releases). Или же вы можете посмотреть ветку `release`, которая всегда содержит последний стабильный релиз:

```text
cd ~/Homestead

git checkout release
```

После клонирования репозитория Homestead запустите команду `bash init.sh` из каталога Homestead для создания конфигурационного файла `Homestead.yaml`. Файл `Homestead.yaml` будет помещен в каталог Homestead:

```text
// Mac / Linux...
bash init.sh

// Windows...
init.bat
```

#### Настройка Homestead

**Настройка провайдера**

Ключ `provider` в файле `Homestead.yaml` указывает, какой провайдер Vagrant должен быть использован: `virtualbox`, `vmware_fusion`, `vmware_workstation`, `parallels` или `hyperv`. Вы можете установить тот провайдер, который предпочитаете:

```text
provider: virtualbox
```

**Настройка общих папок**

Свойство `folder` в файле `Homestead.yaml` перечисляет все папки, которые вы хотите расшарить с вашей средой Homestead. При изменении файлов в этих папках, они будут синхронизироваться между вашей локальной машиной и средой Homestead. Вы можете настроить столько общих папок, сколько необходимо:

```text
folders:
    - map: ~/code/project1
      to: /home/vagrant/project1
```

{% hint style='info' %}
Пользователи Windows не должны использовать синтаксис пути `~/` и вместо нужно использовать полный путь к своему проекту, например `C:\Users\user\Code\project1`.
{% endhint %}

Вы всегда должны привязывать отдельные проекты к их собственной папке mapping вместо того, чтобы привязывать всю папку `~/code`. Когда вы сопоставляете папку, виртуальная машина должна следить за всем дисковым IO для _каждого_ файла в папке. Это приводит к проблемам с производительностью, если у вас большое количество файлов в папке.

```text
folders:
    - map: ~/code/project1
      to: /home/vagrant/project1

    - map: ~/code/project2
      to: /home/vagrant/project2
```

{% hint style='info' %}
Вы никогда не должны монтировать `.` \(текущий каталог\) при использовании Homestead. Это приводит к тому, что Vagrant не сопоставляет текущую папку с `/vagrant`, делает недоступными некоторые возможности и приводит к неожиданным результатам во время инициализации.
{% endhint %}

Чтобы включить [NFS](https://www.vagrantup.com/docs/synced-folders/nfs.html), достаточно добавить флаг в конфигурацию синхронизированной папки:

```text
folders:
    - map: ~/code/project1
      to: /home/vagrant/project1
      type: "nfs"
```

{% hint style='info' %}
При использовании NFS под Windows следует подумать об установке плагина [vagrant-winnfsd](https://github.com/winnfsd/vagrant-winnfsd). Этот плагин будет поддерживать правильные пользовательские/групповые разрешения для файлов и каталогов в Homestead box.
{% endhint %}

Вы также можете передать любые опции, поддерживаемые [Synced Folders](https://www.vagrantup.com/docs/synced-folders/basic_usage.html) Vagrant, перечислив их в ключе `options`:

```text
folders:
    - map: ~/code/project1
      to: /home/vagrant/project1
      type: "rsync"
      options:
          rsync__args: ["--verbose", "--archive", "--delete", "-zz"]
          rsync__exclude: ["node_modules"]
```

**Configuring Nginx Sites**

Not familiar with Nginx? No problem. The `sites` property allows you to easily map a "domain" to a folder on your Homestead environment. A sample site configuration is included in the `Homestead.yaml` file. Again, you may add as many sites to your Homestead environment as necessary. Homestead can serve as a convenient, virtualized environment for every Laravel project you are working on:

```text
sites:
    - map: homestead.test
      to: /home/vagrant/project1/public
```

If you change the `sites` property after provisioning the Homestead box, you should re-run `vagrant reload --provision` to update the Nginx configuration on the virtual machine.

{% hint style='info' %}
Homestead scripts are built to be as idempotent as possible. However, if you are experiencing issues while provisioning you should destroy and rebuild the machine via `vagrant destroy && vagrant up`.
{% endhint %}

**Enable / Disable Services**

Homestead starts several services by default; however, you may customize which services are enabled or disabled during provisioning. For example, you may enable PostgreSQL and disable MySQL:

```text
services:
    - enabled:
        - "postgresql@12-main"
    - disabled:
        - "mysql"
```

The specified services will be started or stopped based on their order in the `enabled` and `disabled` directives.

**Hostname Resolution**

Homestead publishes hostnames over `mDNS` for automatic host resolution. If you set `hostname: homestead` in your `Homestead.yaml` file, the host will be available at `homestead.local`. MacOS, iOS, and Linux desktop distributions include `mDNS` support by default. Windows requires installing [Bonjour Print Services for Windows](https://support.apple.com/kb/DL999?viewlocale=en_US&locale=en_US).

Using automatic hostnames works best for "per project" installations of Homestead. If you host multiple sites on a single Homestead instance, you may add the "domains" for your web sites to the `hosts` file on your machine. The `hosts` file will redirect requests for your Homestead sites into your Homestead machine. On Mac and Linux, this file is located at `/etc/hosts`. On Windows, it is located at `C:\Windows\System32\drivers\etc\hosts`. The lines you add to this file will look like the following:

```text
192.168.10.10  homestead.test
```

Make sure the IP address listed is the one set in your `Homestead.yaml` file. Once you have added the domain to your `hosts` file and launched the Vagrant box you will be able to access the site via your web browser:

```text
http://homestead.test
```

#### Launching The Vagrant Box

Once you have edited the `Homestead.yaml` to your liking, run the `vagrant up` command from your Homestead directory. Vagrant will boot the virtual machine and automatically configure your shared folders and Nginx sites.

To destroy the machine, you may use the `vagrant destroy --force` command.

#### Per Project Installation

Instead of installing Homestead globally and sharing the same Homestead box across all of your projects, you may instead configure a Homestead instance for each project you manage. Installing Homestead per project may be beneficial if you wish to ship a `Vagrantfile` with your project, allowing others working on the project to `vagrant up`.

To install Homestead directly into your project, require it using Composer:

```text
composer require laravel/homestead --dev
```

Once Homestead has been installed, use the `make` command to generate the `Vagrantfile` and `Homestead.yaml` file in your project root. The `make` command will automatically configure the `sites` and `folders` directives in the `Homestead.yaml` file.

Mac / Linux:

```text
php vendor/bin/homestead make
```

Windows:

```text
vendor\\bin\\homestead make
```

Next, run the `vagrant up` command in your terminal and access your project at `http://homestead.test` in your browser. Remember, you will still need to add an `/etc/hosts` file entry for `homestead.test` or the domain of your choice if you are not using automatic [hostname resolution](https://laravel.com/docs/7.x/homestead#hostname-resolution).

#### Installing Optional Features

Optional software is installed using the "features" setting in your Homestead configuration file. Most features can be enabled or disabled with a boolean value, while some features allow multiple configuration options:

```text
features:
    - blackfire:
        server_id: "server_id"
        server_token: "server_value"
        client_id: "client_id"
        client_token: "client_value"
    - cassandra: true
    - chronograf: true
    - couchdb: true
    - crystal: true
    - docker: true
    - elasticsearch:
        version: 7
    - gearman: true
    - golang: true
    - grafana: true
    - influxdb: true
    - mariadb: true
    - minio: true
    - mongodb: true
    - mysql8: true
    - neo4j: true
    - ohmyzsh: true
    - openresty: true
    - pm2: true
    - python: true
    - rabbitmq: true
    - solr: true
    - webdriver: true
```

**MariaDB**

Enabling MariaDB will remove MySQL and install MariaDB. MariaDB serves as a drop-in replacement for MySQL, so you should still use the `mysql` database driver in your application's database configuration.

**MongoDB**

The default MongoDB installation will set the database username to `homestead` and the corresponding password to `secret`.

**Elasticsearch**

You may specify a supported version of Elasticsearch, which may be a major version or an exact version number \(major.minor.patch\). The default installation will create a cluster named 'homestead'. You should never give Elasticsearch more than half of the operating system's memory, so make sure your Homestead machine has at least twice the Elasticsearch allocation.

{% hint style='tip' %}
Check out the [Elasticsearch documentation](https://www.elastic.co/guide/en/elasticsearch/reference/current) to learn how to customize your configuration.
{% endhint %}

**Neo4j**

The default Neo4j installation will set the database username to `homestead` and corresponding password to `secret`. To access the Neo4j browser, visit `http://homestead.test:7474` via your web browser. The ports `7687` \(Bolt\), `7474` \(HTTP\), and `7473` \(HTTPS\) are ready to serve requests from the Neo4j client.

#### Aliases

You may add Bash aliases to your Homestead machine by modifying the `aliases` file within your Homestead directory:

```text
alias c='clear'
alias ..='cd ..'
```

After you have updated the `aliases` file, you should re-provision the Homestead machine using the `vagrant reload --provision` command. This will ensure that your new aliases are available on the machine.

## Daily Usage

### Accessing Homestead Globally

Sometimes you may want to `vagrant up` your Homestead machine from anywhere on your filesystem. You can do this on Mac / Linux systems by adding a Bash function to your Bash profile. On Windows, you may accomplish this by adding a "batch" file to your `PATH`. These scripts will allow you to run any Vagrant command from anywhere on your system and will automatically point that command to your Homestead installation:

**Mac / Linux**

```text
function homestead() {
    ( cd ~/Homestead && vagrant $* )
}
```

Make sure to tweak the `~/Homestead` path in the function to the location of your actual Homestead installation. Once the function is installed, you may run commands like `homestead up` or `homestead ssh` from anywhere on your system.

**Windows**

Create a `homestead.bat` batch file anywhere on your machine with the following contents:

```text
@echo off

set cwd=%cd%
set homesteadVagrant=C:\Homestead

cd /d %homesteadVagrant% && vagrant %*
cd /d %cwd%

set cwd=
set homesteadVagrant=
```

Make sure to tweak the example `C:\Homestead` path in the script to the actual location of your Homestead installation. After creating the file, add the file location to your `PATH`. You may then run commands like `homestead up` or `homestead ssh` from anywhere on your system.

### Connecting Via SSH

You can SSH into your virtual machine by issuing the `vagrant ssh` terminal command from your Homestead directory.

But, since you will probably need to SSH into your Homestead machine frequently, consider adding the "function" described above to your host machine to quickly SSH into the Homestead box.

#### Connecting To Databases

A `homestead` database is configured for both MySQL and PostgreSQL out of the box. To connect to your MySQL or PostgreSQL database from your host machine's database client, you should connect to `127.0.0.1` and port `33060` \(MySQL\) or `54320` \(PostgreSQL\). The username and password for both databases is `homestead` / `secret`.

{% hint style='info' %}
You should only use these non-standard ports when connecting to the databases from your host machine. You will use the default 3306 and 5432 ports in your Laravel database configuration file since Laravel is running _within_ the virtual machine.
{% endhint %}

#### Database Backups

Homestead can automatically backup your database when your Vagrant box is destroyed. To utilize this feature, you must be using Vagrant 2.1.0 or greater. Or, if you are using an older version of Vagrant, you must install the `vagrant-triggers` plug-in. To enable automatic database backups, add the following line to your `Homestead.yaml` file:

```text
backup: true
```

Once configured, Homestead will export your databases to `mysql_backup` and `postgres_backup` directories when the `vagrant destroy` command is executed. These directories can be found in the folder where you cloned Homestead or in the root of your project if you are using the [per project installation](https://laravel.com/docs/7.x/homestead#per-project-installation) method.

#### Database Snapshots

Homestead supports freezing the state of MySQL and MariaDB databases and branching between them using [Logical MySQL Manager](https://github.com/Lullabot/lmm). For example, imagine working on a site with a multi-gigabyte database. You can import the database and take a snapshot. After doing some work and creating some test content locally, you may quickly restore back to the original state.

Under the hood, LMM uses LVM's thin snapshot functionality with copy-on-write support. In practice, this means that changing a single row in a table will only cause the changes you made to be written to disk, saving significant time and disk space during restores.

Since `lmm` interacts with LVM, it must be run as `root`. To see all available commands, run `sudo lmm` inside your Vagrant box. A common workflow looks like the following:

1. Import a database into the default `master` lmm branch.
2. Save a snapshot of the unchanged database using `sudo lmm branch prod-YYYY-MM-DD`.
3. Modify the database.
4. Run `sudo lmm merge prod-YYYY-MM-DD` to undo all changes.
5. Run `sudo lmm delete <branch>` to delete unneeded branches.

#### Adding Additional Sites

Once your Homestead environment is provisioned and running, you may want to add additional Nginx sites for your Laravel applications. You can run as many Laravel installations as you wish on a single Homestead environment. To add an additional site, add the site to your `Homestead.yaml` file:

```text
sites:
    - map: homestead.test
      to: /home/vagrant/project1/public
    - map: another.test
      to: /home/vagrant/project2/public
```

If Vagrant is not automatically managing your "hosts" file, you may need to add the new site to that file as well:

```text
192.168.10.10  homestead.test
192.168.10.10  another.test
```

Once the site has been added, run the `vagrant reload --provision` command from your Homestead directory.

**Site Types**

Homestead supports several types of sites which allow you to easily run projects that are not based on Laravel. For example, we may easily add a Symfony application to Homestead using the `symfony2` site type:

```text
sites:
    - map: symfony2.test
      to: /home/vagrant/my-symfony-project/web
      type: "symfony2"
```

The available site types are: `apache`, `apigility`, `expressive`, `laravel` \(the default\), `proxy`, `silverstripe`, `statamic`, `symfony2`, `symfony4`, and `zf`.

**Site Parameters**

You may add additional Nginx `fastcgi_param` values to your site via the `params` site directive. For example, we'll add a `FOO` parameter with a value of `BAR`:

```text
sites:
    - map: homestead.test
      to: /home/vagrant/project1/public
      params:
          - key: FOO
            value: BAR
```

#### Environment Variables

You can set global environment variables by adding them to your `Homestead.yaml` file:

```text
variables:
    - key: APP_ENV
      value: local
    - key: FOO
      value: bar
```

After updating the `Homestead.yaml`, be sure to re-provision the machine by running `vagrant reload --provision`. This will update the PHP-FPM configuration for all of the installed PHP versions and also update the environment for the `vagrant` user.

#### Wildcard SSL

Homestead configures a self-signed SSL certificate for each site defined in the `sites:` section of your `Homestead.yaml` file. If you would like to generate a wildcard SSL certificate for a site you may add a `wildcard` option to that site's configuration. By default, the site will use the wildcard certificate _instead_ of the specific domain certificate:

```text
- map: foo.domain.test
  to: /home/vagrant/domain
  wildcard: "yes"
```

If the `use_wildcard` option is set to `no`, the wildcard certificate will be generated but will not be used:

```text
- map: foo.domain.test
  to: /home/vagrant/domain
  wildcard: "yes"
  use_wildcard: "no"
```

#### Configuring Cron Schedules

Laravel provides a convenient way to [schedule Cron jobs](https://laravel.com/docs/7.x/scheduling) by scheduling a single `schedule:run` Artisan command to be run every minute. The `schedule:run` command will examine the job schedule defined in your `App\Console\Kernel` class to determine which jobs should be run.

If you would like the `schedule:run` command to be run for a Homestead site, you may set the `schedule` option to `true` when defining the site:

```text
sites:
    - map: homestead.test
      to: /home/vagrant/project1/public
      schedule: true
```

The Cron job for the site will be defined in the `/etc/cron.d` folder of the virtual machine.

#### Configuring Mailhog

Mailhog allows you to easily catch your outgoing email and examine it without actually sending the mail to its recipients. To get started, update your `.env` file to use the following mail settings:

```text
MAIL_MAILER=smtp
MAIL_HOST=localhost
MAIL_PORT=1025
MAIL_USERNAME=null
MAIL_PASSWORD=null
MAIL_ENCRYPTION=null
```

Once Mailhog has been configured, you may access the Mailhog dashboard at `http://localhost:8025`.

#### Configuring Minio

Minio is an open source object storage server with an Amazon S3 compatible API. To install Minio, update your `Homestead.yaml` file with the following configuration option in the [features](https://laravel.com/docs/7.x/homestead#installing-optional-features) section:

```text
minio: true
```

By default, Minio is available on port 9600. You may access the Minio control panel by visiting `http://localhost:9600/`. The default access key is `homestead`, while the default secret key is `secretkey`. When accessing Minio, you should always use region `us-east-1`.

In order to use Minio you will need to adjust the S3 disk configuration in your `config/filesystems.php` configuration file. You will need to add the `use_path_style_endpoint` option to the disk configuration, as well as change the `url` key to `endpoint`:

```text
's3' => [
    'driver' => 's3',
    'key' => env('AWS_ACCESS_KEY_ID'),
    'secret' => env('AWS_SECRET_ACCESS_KEY'),
    'region' => env('AWS_DEFAULT_REGION'),
    'bucket' => env('AWS_BUCKET'),
    'endpoint' => env('AWS_URL'),
    'use_path_style_endpoint' => true,
]
```

Finally, ensure your `.env` file has the following options:

```text
AWS_ACCESS_KEY_ID=homestead
AWS_SECRET_ACCESS_KEY=secretkey
AWS_DEFAULT_REGION=us-east-1
AWS_URL=http://localhost:9600
```

To provision buckets, add a `buckets` directive to your Homestead configuration file:

```text
buckets:
    - name: your-bucket
      policy: public
    - name: your-private-bucket
      policy: none
```

Supported `policy` values include: `none`, `download`, `upload`, and `public`.

#### Ports

By default, the following ports are forwarded to your Homestead environment:

* **SSH:** 2222 → Forwards To 22
* **ngrok UI:** 4040 → Forwards To 4040
* **HTTP:** 8000 → Forwards To 80
* **HTTPS:** 44300 → Forwards To 443
* **MySQL:** 33060 → Forwards To 3306
* **PostgreSQL:** 54320 → Forwards To 5432
* **MongoDB:** 27017 → Forwards To 27017
* **Mailhog:** 8025 → Forwards To 8025
* **Minio:** 9600 → Forwards To 9600

**Forwarding Additional Ports**

If you wish, you may forward additional ports to the Vagrant box, as well as specify their protocol:

```text
ports:
    - send: 50000
      to: 5000
    - send: 7777
      to: 777
      protocol: udp
```

#### Sharing Your Environment

Sometimes you may wish to share what you're currently working on with coworkers or a client. Vagrant has a built-in way to support this via `vagrant share`; however, this will not work if you have multiple sites configured in your `Homestead.yaml` file.

To solve this problem, Homestead includes its own `share` command. To get started, SSH into your Homestead machine via `vagrant ssh` and run `share homestead.test`. This will share the `homestead.test` site from your `Homestead.yaml` configuration file. You may substitute any of your other configured sites for `homestead.test`:

```text
share homestead.test
```

After running the command, you will see an Ngrok screen appear which contains the activity log and the publicly accessible URLs for the shared site. If you would like to specify a custom region, subdomain, or other Ngrok runtime option, you may add them to your `share` command:

```text
share homestead.test -region=eu -subdomain=laravel
```

{% hint style='info' %}
Remember, Vagrant is inherently insecure and you are exposing your virtual machine to the Internet when running the `share` command.
{% endhint %}

#### Multiple PHP Versions

Homestead 6 introduced support for multiple versions of PHP on the same virtual machine. You may specify which version of PHP to use for a given site within your `Homestead.yaml` file. The available PHP versions are: "5.6", "7.0", "7.1", "7.2", "7.3", and "7.4" \(the default\):

```text
sites:
    - map: homestead.test
      to: /home/vagrant/project1/public
      php: "7.1"
```

In addition, you may use any of the supported PHP versions via the CLI:

```text
php5.6 artisan list
php7.0 artisan list
php7.1 artisan list
php7.2 artisan list
php7.3 artisan list
php7.4 artisan list
```

You may also update the default CLI version by issuing the following commands from within your Homestead virtual machine:

```text
php56
php70
php71
php72
php73
php74
```

#### Web Servers

Homestead uses the Nginx web server by default. However, it can install Apache if `apache` is specified as a site type. While both web servers can be installed at the same time, they cannot both be _running_ at the same time. The `flip` shell command is available to ease the process of switching between web servers. The `flip` command automatically determines which web server is running, shuts it off, and then starts the other server. To use this command, SSH into your Homestead machine and run the command in your terminal:

```text
flip
```

#### Mail

Homestead includes the Postfix mail transfer agent, which is listening on port `1025` by default. So, you may instruct your application to use the `smtp` mail driver on `localhost` port `1025`. Then, all sent mail will be handled by Postfix and caught by Mailhog. To view your sent emails, open [http://localhost:8025](http://localhost:8025/) in your web browser.

## Debugging & Profiling

#### Debugging Web Requests With Xdebug

Homestead includes support for step debugging using [Xdebug](https://xdebug.org/). For example, you can load a web page from a browser, and PHP will connect to your IDE to allow inspection and modification of the running code.

By default Xdebug is already running and ready to accept connections. If you need to enable Xdebug on the CLI run the `sudo phpenmod xdebug` command within your Vagrant box. Next, follow your IDE's instructions to enable debugging. Finally, configure your browser to trigger Xdebug with an extension or [bookmarklet](https://www.jetbrains.com/phpstorm/marklets/).

{% hint style='info' %}
Xdebug causes PHP to run significantly slower. To disable Xdebug, run `sudo phpdismod xdebug` within your Vagrant box and restart the FPM service.
{% endhint %}

#### Debugging CLI Applications

To debug a PHP CLI application, use the `xphp` shell alias inside your Vagrant box:

```text
xphp path/to/script
```

**Autostarting Xdebug**

When debugging functional tests that make requests to the web server, it is easier to autostart debugging rather than modifying tests to pass through a custom header or cookie to trigger debugging. To force Xdebug to start automatically, modify `/etc/php/7.x/fpm/conf.d/20-xdebug.ini` inside your Vagrant box and add the following configuration:

```text
; If Homestead.yaml contains a different subnet for the IP address, this address may be different...
xdebug.remote_host = 192.168.10.1
xdebug.remote_autostart = 1
```

#### Profiling Applications with Blackfire

[Blackfire](https://blackfire.io/docs/introduction) is a SaaS service for profiling web requests and CLI applications and writing performance assertions. It offers an interactive user interface which displays profile data in call-graphs and timelines. It is built for use in development, staging, and production, with no overhead for end users. It provides performance, quality, and security checks on code and `php.ini` configuration settings.

The [Blackfire Player](https://blackfire.io/docs/player/index) is an open-source Web Crawling, Web Testing and Web Scraping application which can work jointly with Blackfire in order to script profiling scenarios.

To enable Blackfire, use the "features" setting in your Homestead configuration file:

```text
features:
    - blackfire:
        server_id: "server_id"
        server_token: "server_value"
        client_id: "client_id"
        client_token: "client_value"
```

Blackfire server credentials and client credentials [require a user account](https://blackfire.io/signup). Blackfire offers various options to profile an application, including a CLI tool and browser extension. Please [review the Blackfire documentation for more details](https://blackfire.io/docs/cookbooks/index).

#### Profiling PHP Performance Using XHGui

[XHGui](https://www.github.com/perftools/xhgui) is a user interface for exploring the performance of your PHP applications. To enable XHGui, add `xhgui: 'true'` to your site configuration:

```text
sites:
    -
        map: your-site.test
        to: /home/vagrant/your-site/public
        type: "apache"
        xhgui: 'true'
```

If the site already exists, make sure to run `vagrant provision` after updating your configuration.

To profile a web request, add `xhgui=on` as a query parameter to a request. XHGui will automatically attach a cookie to the response so that subsequent requests do not need the query string value. You may view your application profile results by browsing to `http://your-site.test/xhgui`.

To profile a CLI request using XHGui, prefix the command with `XHGUI=on`:

```text
XHGUI=on path/to/script
```

CLI profile results may be viewed in the same way as web profile results.

Note that the act of profiling slows down script execution, and absolute times may be as much as twice as real-world requests. Therefore, always compare percentage improvements and not absolute numbers. Also, be aware the execution time includes any time spent paused in a debugger.

Since performance profiles take up significant disk space, they are deleted automatically after a few days.

## Network Interfaces

The `networks` property of the `Homestead.yaml` configures network interfaces for your Homestead environment. You may configure as many interfaces as necessary:

```text
networks:
    - type: "private_network"
      ip: "192.168.10.20"
```

To enable a [bridged](https://www.vagrantup.com/docs/networking/public_network.html) interface, configure a `bridge` setting and change the network type to `public_network`:

```text
networks:
    - type: "public_network"
      ip: "192.168.10.20"
      bridge: "en1: Wi-Fi (AirPort)"
```

To enable [DHCP](https://www.vagrantup.com/docs/networking/public_network.html), just remove the `ip` option from your configuration:

```text
networks:
    - type: "public_network"
      bridge: "en1: Wi-Fi (AirPort)"
```

## Extending Homestead

You may extend Homestead using the `after.sh` script in the root of your Homestead directory. Within this file, you may add any shell commands that are necessary to properly configure and customize your virtual machine.

When customizing Homestead, Ubuntu may ask you if you would like to keep a package's original configuration or overwrite it with a new configuration file. To avoid this, you should use the following command when installing packages to avoid overwriting any configuration previously written by Homestead:

```text
sudo apt-get -y \
    -o Dpkg::Options::="--force-confdef" \
    -o Dpkg::Options::="--force-confold" \
    install your-package
```

#### User Customizations

When using Homestead in a team setting, you may want to tweak Homestead to better fit your personal development style. You may create a `user-customizations.sh` file in the root of your Homestead directory \(The same directory containing your `Homestead.yaml`\). Within this file, you may make any customization you would like; however, the `user-customizations.sh` should not be version controlled.

## Updating Homestead

Before you begin updating Homestead ensure you have removed your current virtual machine by running the following command in your Homestead directory:

```text
vagrant destroy
```

Next, you need to update the Homestead source code. If you cloned the repository you can run the following commands at the location you originally cloned the repository:

```text
git fetch

git pull origin release
```

These commands pull the latest Homestead code from the GitHub repository, fetches the latest tags, and then checks out the latest tagged release. You can find the latest stable release version on the [GitHub releases page](https://github.com/laravel/homestead/releases).

If you have installed Homestead via your project's `composer.json` file, you should ensure your `composer.json` file contains `"laravel/homestead": "^11"` and update your dependencies:

```text
composer update
```

Then, you should update the Vagrant box using the `vagrant box update` command:

```text
vagrant box update
```

Next, you should run the `bash init.sh` command from the Homestead directory in order to update some additional configuration files. You will be asked whether you wish to overwrite your existing `Homestead.yaml`, `after.sh`, and `aliases` files:

```text
// Mac / Linux...
bash init.sh

// Windows...
init.bat
```

Finally, you will need to regenerate your Homestead box to utilize the latest Vagrant installation:

```text
vagrant up
```

## Provider Specific Settings

### VirtualBox

**natdnshostresolver**

By default, Homestead configures the `natdnshostresolver` setting to `on`. This allows Homestead to use your host operating system's DNS settings. If you would like to override this behavior, add the following lines to your `Homestead.yaml` file:

```text
provider: virtualbox
natdnshostresolver: 'off'
```

**Symbolic Links On Windows**

If symbolic links are not working properly on your Windows machine, you may need to add the following block to your `Vagrantfile`:

```text
config.vm.provider "virtualbox" do |v|
    v.customize ["setextradata", :id, "VBoxInternal2/SharedFoldersEnableSymlinksCreate/v-root", "1"]
end
```

