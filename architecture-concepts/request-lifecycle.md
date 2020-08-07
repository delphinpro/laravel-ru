# Жизненный цикл запроса

### Вступление

Используя любой инструмент в "реальном мире", вы чувствуете себя более уверенно, если понимаете, как этот инструмент работает. Разработка приложений ничем не отличается. Когда вы понимаете, как функционируют ваши инструменты разработки, вы чувствуете себя более комфортно и уверенно, используя их.

Цель этого документа — дать вам детальный обзор того, как работает система Laravel. Хорошее знакомство с фреймворком позволит вам почувствовать меньше "магии", и вы будете более уверенно создавать свои приложения. Если вы не понимаете всех терминов сразу, не унывайте! Просто постарайтесь получить базовое представление о происходящем, и ваши знания будут расти по мере того, как вы будете изучать другие разделы документации.

### Обзор жизненного цикла

#### Первые шаги

Точкой входа для всех запросов к приложению Laravel является файл `public/index.php`. Все запросы направляются к этому файлу конфигурацией вашего веб-сервера \(Apache / Nginx\). Файл `index.php` не содержит большого количества кода. Скорее, он является отправной точкой для загрузки остальной части фреймворка.

Файл `index.php` подключает сгенерированный Composer\`ом автозагрузчик, а затем получает экземпляр приложения Laravel из скрипта `bootstrap/app.php`. Первое действие, предпринятое самим Laravel — это создание экземпляра приложения / [service container](service-container.md).

#### HTTP / Console ядра

Далее входящий запрос посылается либо на ядро HTTP, либо на консольное ядро, в зависимости от типа поступающего запроса. Эти два ядра служат центральным местом, через которое проходят все запросы. А пока давайте сосредоточимся на ядре HTTP, которое находится в `app/Http/Kernel.php`.

Ядро HTTP расширяет класс `Illuminate\Foundation\Http\Kernel`, определяющий массив `bootstrappers`, которые будут запущены до выполнения запроса. Эти загрузчики настраивают обработку ошибок, ведение логов, [обнаружение среды приложения](../getting-started/configuration.md#environment-configuration) и другие задачи, которые необходимо выполнить до того, как запрос будет обработан.

Ядро HTTP также определяет список [посредников \(middleware\)](../the-basics/middleware.md) HTTP, через которые должны пройти все запросы, прежде чем они будут обработаны приложением. Эти посредники обрабатывают чтение и запись [HTTP-сессии](../the-basics/session.md), определяют, находится ли приложение в режиме обслуживания, [проверяют CSRF-токен](../the-basics/csrf-protection.md) и многое другое.

Сигнатура метода `handle` ядра HTTP достаточно проста: получить `Request` и вернуть `Response`. Считайте, что ядро — это большой черный ящик, представляющий всё ваше приложение. Направьте ему HTTP-запросы и он вернет HTTP-ответы.

**Service Providers \(Поставщики сервисов\)**

Одно из самых важных действий Ядра при загрузке — это загрузка [service providers](service-providers.md) для приложения. Все поставщики услуг настраиваются в массиве провайдеров конфигурационного файла `config/app.php`. Сначала метод `register` вызовет всех провайдеров, затем, после регистрации всех провайдеров, будет вызван метод `boot`.

Сервис-провайдеры отвечают за загрузку различных компонентов фреймворка, таких как база данных \(database\), очередь \(queue\), валидация \(validation\) и компоненты маршрутизации \(routing\). Так как они осуществляют загрузку и конфигурирование каждой функции, предлагаемой фреймворком, поставщики услуг являются наиболее важным аспектом всего процесса загрузки Laravel.

**Dispatch Request**

Once the application has been bootstrapped and all service providers have been registered, the `Request` will be handed off to the router for dispatching. The router will dispatch the request to a route or controller, as well as run any route specific middleware.

### Focus On Service Providers <a id="focus-on-service-providers"></a>

Service providers are truly the key to bootstrapping a Laravel application. The application instance is created, the service providers are registered, and the request is handed to the bootstrapped application. It's really that simple!

Having a firm grasp of how a Laravel application is built and bootstrapped via service providers is very valuable. Your application's default service providers are stored in the `app/Providers` directory.

By default, the `AppServiceProvider` is fairly empty. This provider is a great place to add your application's own bootstrapping and service container bindings. For large applications, you may wish to create several service providers, each with a more granular type of bootstrapping.

