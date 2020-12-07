# Валидация

## Вступление

Laravel предлагает несколько различных подходов к проверке входящих данных вашего приложения. По умолчанию, базовый класс контроллера Laravel использует трейт `ValidatesRequests`, который обеспечивает удобный метод для проверки входящих HTTP-запросов с различными мощными правилами проверки.

## Быстрый старт

Чтобы узнать о мощных функциях валидации Laravel, давайте рассмотрим полный пример валидации формы и отображения сообщений об ошибках обратно к пользователю.

### Определение маршрутов

Во-первых, предположим, что в нашем файле `routes/web.php` определены следующие маршруты:

```php
Route::get('post/create', 'PostController@create');

Route::post('post', 'PostController@store');
```

Маршрут `GET` отобразит форму для создания пользователем нового поста блога, а `POST` маршрут сохранит новый пост блога в базе данных.

### Создание контроллера

Далее, давайте посмотрим на простой контроллер, который обрабатывает эти маршруты. Пока оставим метод `store` пустым:

```php
<?php

namespace App\Http\Controllers;

use App\Http\Controllers\Controller;
use Illuminate\Http\Request;

class PostController extends Controller
{
    /**
     * Show the form to create a new blog post.
     *
     * @return Response
     */
    public function create()
    {
        return view('post.create');
    }

    /**
     * Store a new blog post.
     *
     * @param  Request  $request
     * @return Response
     */
    public function store(Request $request)
    {
        // Validate and store the blog post...
    }
}
```

### Пишем логику проверки

Теперь мы готовы заполнить наш метод `store` логикой проверки новой записи в блоге. Для этого мы будем использовать метод `validate`, предоставляемый объектом `Illuminate\Http\Request`. Если правила валидации пройдут, то ваш код будет продолжать выполняться нормально, но если валидация не пройдет, то будет выброшено исключение, и пользователю будет автоматически отправлен правильный ответ об ошибке. В случае традиционного HTTP-запроса будет сгенерирован редиректный ответ, а для AJAX-запросов будет отправлен JSON-ответ.

Чтобы лучше понять метод `validate`, давайте вернемся к методу `store`:

```php
/**
 * Store a new blog post.
 *
 * @param  Request  $request
 * @return Response
 */
public function store(Request $request)
{
    $validatedData = $request->validate([
        'title' => 'required|unique:posts|max:255',
        'body' => 'required',
    ]);

    // The blog post is valid...
}
```

Как видите, мы передаем желаемые правила валидации в метод `validate`. Опять же, если валидация не пройдет, правильный ответ будет сгенерирован автоматически. Если валидация пройдет, наш контроллер продолжит работать в нормальном режиме.

Кроме того, правила проверки могут быть указаны в виде массивов правил, а не в виде отдельной строки `|`:

```php
$validatedData = $request->validate([
    'title' => ['required', 'unique:posts', 'max:255'],
    'body' => ['required'],
]);
```

Вы можете использовать метод `validateWithBag` для проверки запроса и хранения любых сообщений об ошибках в [именованной коллекции ошибок](validation.md#named-error-bags):

```php
$validatedData = $request->validateWithBag('post', [
    'title' => ['required', 'unique:posts', 'max:255'],
    'body' => ['required'],
]);
```

**Остановка при первой неудаче проверки**

Иногда вам может понадобиться прекратить выполнение правил проверки атрибута после первого сбоя проверки. Для этого, назначьте правило `bail` для атрибута:

```php
$request->validate([
    'title' => 'bail|required|unique:posts|max:255',
    'body' => 'required',
]);
```

В этом примере, если правило `unique` по атрибуту `title` не срабатывает, правило `max` не будет проверено. Правила будут проверяться в порядке их назначения.

**Замечание о вложенных атрибутах**

Если ваш HTTP-запрос содержит "вложенные" параметры, вы можете указать их в правилах проверки, используя синтаксис "точка":

```php
$request->validate([
    'title' => 'required|unique:posts|max:255',
    'author.name' => 'required',
    'author.description' => 'required',
]);
```

### Отображение ошибок проверки

Что если параметры входящего запроса не проходят заданные правила проверки? Как уже упоминалось ранее, Laravel автоматически перенаправит пользователя обратно на его предыдущее местоположение. Кроме того, все ошибки валидации будут автоматически [сохранены в сессию](https://github.com/delphinpro/laravel-ru/tree/2d2ea3adc5e79ae67432bf74106531d53e55c7ab/the-basics/session/README.md#flash-data).

Опять же, обратите внимание, что нам не пришлось явно привязывать сообщения об ошибках к представлению в нашем `GET` маршруте. Это связано с тем, что Laravel будет проверять данные сессии на наличие ошибок и автоматически привязывать к представлению, если они доступны. Переменная `$errors` будет экземпляром `Illuminate\Support\MessageBag`. Подробнее о работе с этим объектом можно прочитать в его документации \[\#working-with-error-messages\].

{% hint style="info" %}
Переменная `$errors` привязана к представлению с помощью посредника `Illuminate\View\Middleware\ShareErrorsFromSession`, который предоставляется группой `web`. **Когда этот посредник применяется, переменная `$errors` всегда будет доступна в вашем представлении**, что позволяет удобно предположить, что переменная `$errors` всегда определена и может быть безопасно использована.
{% endhint %}

Таким образом, в нашем примере, пользователь будет перенаправлен в метод `create` контроллера, когда проверка не удастся, что позволит нам отобразить сообщения об ошибках в представлении:

```markup
<!-- /resources/views/post/create.blade.php -->

<h1>Create Post</h1>

@if ($errors->any())
    <div class="alert alert-danger">
        <ul>
            @foreach ($errors->all() as $error)
                <li>{{ $error }}</li>
            @endforeach
        </ul>
    </div>
@endif

<!-- Create Post Form -->
```

**Директива @error**

Вы также можете использовать директиву `@error` [Blade](https://laravel.com/docs/7.x/blade) для быстрой проверки наличия сообщений об ошибках проверки для данного атрибута. В директиве `@error` для отображения сообщения об ошибке можно использовать переменную `$message`:

```markup
<!-- /resources/views/post/create.blade.php -->

<label for="title">Post Title</label>

<input id="title" type="text" class="@error('title') is-invalid @enderror">

@error('title')
    <div class="alert alert-danger">{{ $message }}</div>
@enderror
```

### Замечание о дополнительных полях

По умолчанию Laravel включает посредники `TrimStrings` и `ConvertEmptyStringsToNull` в глобальный стек посредников приложения. Эти посредники перечислены в классе `App\Http\Kernel`. Из-за этого вам часто придется отмечать свои "необязательные" поля запроса как `nullable`, если вы не хотите, чтобы валидатор считал `nullable` значения недействительными. Например:

```php
$request->validate([
    'title' => 'required|unique:posts|max:255',
    'body' => 'required',
    'publish_at' => 'nullable|date',
]);
```

В данном примере мы указываем, что поле `publish_at` может быть как `null`, так и действительным представлением даты. Если модификатор `nullable` не будет добавлен в определение правила, валидатор будет считать `null` недействительной датой.

**AJAX запросы и валидация**

В данном примере мы использовали традиционную форму для отправки данных в приложение. Однако многие приложения используют AJAX-запросы. При использовании метода `validate` во время AJAX-запроса, Laravel не будет генерировать редиректный ответ. Вместо этого, Laravel генерирует JSON ответ, содержащий все ошибки проверки. Этот JSON-ответ будет отправлен с кодом статуса 422 HTTP.

## Проверка формы запроса

### Создание формы запроса

Для более сложных сценариев валидации вы можете создать "форму запроса". Форма запроса — это пользовательские классы запросов, которые содержат логику валидации. Для создания класса формы запроса используйте команду Artisan CLI `make:request`:

```bash
php artisan make:request StoreBlogPost
```

Сгенерированный класс будет помещен в каталог `app/Http/Requests`. Если этот каталог не существует, то он будет создан при выполнении команды `make:request`. Добавим несколько правил проверки в метод `rules`:

```php
/**
 * Get the validation rules that apply to the request.
 *
 * @return array
 */
public function rules()
{
    return [
        'title' => 'required|unique:posts|max:255',
        'body' => 'required',
    ];
}
```

{% hint style="info" %}
В сигнатуре метода `rules` вы можете указать любые необходимые вам зависимости. Они будут автоматически разрешены через Laravel [сервис-контейнер](https://laravel.com/docs/7.x/container).
{% endhint %}

Так как же оцениваются правила валидации? Все, что вам нужно сделать, это указать класс запроса в методе контроллера. Запрос входящей формы проверяется перед вызовом метода контроллера, что означает, что вам не нужно загромождать контроллер какой-либо логикой валидации:

```php
/**
 * Store the incoming blog post.
 *
 * @param  StoreBlogPost  $request
 * @return Response
 */
public function store(StoreBlogPost $request)
{
    // The incoming request is valid...

    // Retrieve the validated input data...
    $validated = $request->validated();
}
```

Если проверка провалилась, будет сгенерирован ответ с перенаправлением для отправки пользователя обратно в его предыдущее местоположение. Ошибки также будут сохранены в в сессию, чтобы они были доступны для отображения. Если был AJAX запрос, HTTP ответ с кодом состояния 422 будет возвращен пользователю, включая JSON представление ошибок проверки.

**Добавление хука перед вызовом формы запроса**

Если Вы хотите добавить к запросу формы хук "after", то можете использовать метод `withValidator`. Этот метод получает полностью построенный валидатор, позволяющий вызывать любой из его методов до того, как правила валидации будут реально оценены:

```php
/**
 * Configure the validator instance.
 *
 * @param  \Illuminate\Validation\Validator  $validator
 * @return void
 */
public function withValidator($validator)
{
    $validator->after(function ($validator) {
        if ($this->somethingElseIsInvalid()) {
            $validator->errors()->add('field', 'Something is wrong with this field!');
        }
    });
}
```

### Авторизованные формы запроса

Класс запроса формы также содержит метод `authorize`. В рамках этого метода можно проверить, действительно ли аутентифицированный пользователь имеет право обновлять данный ресурс. Например, вы можете определить, действительно ли пользователь владеет комментарием к блогу, который он пытается обновить:

```php
/**
 * Determine if the user is authorized to make this request.
 *
 * @return bool
 */
public function authorize()
{
    $comment = Comment::find($this->route('comment'));

    return $comment && $this->user()->can('update', $comment);
}
```

Поскольку все запросы формы расширяют базовый класс запроса Laravel, мы можем использовать метод `user` для доступа к текущему аутентифицированному пользователю. Также обратите внимание на вызов метода `route` в примере выше. Этот метод предоставляет доступ к параметрам URI, определенным на вызываемом маршруте, таким как `{comment}`, в примере ниже:

```php
Route::post('comment/{comment}');
```

If the `authorize` method returns `false`, a HTTP response with a 403 status code will automatically be returned and your controller method will not execute.

If you plan to have authorization logic in another part of your application, return `true` from the `authorize` method:

```php
/**
 * Determine if the user is authorized to make this request.
 *
 * @return bool
 */
public function authorize()
{
    return true;
}
```

{% hint style="info" %}
Вы можете указать любые зависимости, которые вам нужны в методе `authorize`. Они будут автоматически разрешены через Laravel [сервис-контейнер](https://laravel.com/docs/7.x/container).
{% endhint %}

### Настройка сообщений об ошибках

Вы можете настроить сообщения об ошибках, используемые запросом формы, переопределив метод `messages`. Этот метод должен возвращать массив пар атрибут/правило и соответствующие им сообщения об ошибках:

```php
/**
 * Get the error messages for the defined validation rules.
 *
 * @return array
 */
public function messages()
{
    return [
        'title.required' => 'A title is required',
        'body.required' => 'A message is required',
    ];
}
```

### Настройка атрибутов проверки

Если Вы хотите, чтобы часть `:attribute` сообщения о проверке была заменена именем пользовательского атрибута, Вы можете указать пользовательские имена, переопределив метод `attributes`. Этот метод должен возвращать массив пар атрибут/имя:

```php
/**
 * Get custom attributes for validator errors.
 *
 * @return array
 */
public function attributes()
{
    return [
        'email' => 'email address',
    ];
}
```

### Подготовка входных данных для проверки

Если вам необходимо очистить какие-либо данные из запроса перед применением правил валидации, вы можете воспользоваться методом `prepareForValidation`:

```php
use Illuminate\Support\Str;

/**
 * Prepare the data for validation.
 *
 * @return void
 */
protected function prepareForValidation()
{
    $this->merge([
        'slug' => Str::slug($this->slug),
    ]);
}
```

## Ручное создание валидаторов

Если вы не хотите использовать метод `validate` в запросе, то можете создать экземпляр валидатора вручную, используя [фасад](https://laravel.com/docs/7.x/facades) `Validator`. Метод `make` фасада генерирует новый экземпляр валидатора:

```php
<?php

namespace App\Http\Controllers;

use App\Http\Controllers\Controller;
use Illuminate\Http\Request;
use Illuminate\Support\Facades\Validator;

class PostController extends Controller
{
    /**
     * Store a new blog post.
     *
     * @param  Request  $request
     * @return Response
     */
    public function store(Request $request)
    {
        $validator = Validator::make($request->all(), [
            'title' => 'required|unique:posts|max:255',
            'body' => 'required',
        ]);

        if ($validator->fails()) {
            return redirect('post/create')
                        ->withErrors($validator)
                        ->withInput();
        }

        // Store the blog post...
    }
}
```

Первый аргумент, передаваемый в метод `make` — данные для проверки. Второй аргумент — правила валидации, которые должны применяться к данным.

После проверки, если проверка запроса не прошла успешно, вы можете использовать метод `withErrors` для сохранения сообщений об ошибках в сессии. При использовании этого метода, переменная `$errors` после переадресации будет автоматически передана в представления, что позволит вам легко отобразить их пользователю. Метод `withErrors` принимает валидатор, `MessageBag` или PHP-массив.

### Автоматическая переадресация

Если вы хотите создать экземпляр валидатора вручную, но все же пользуетесь автоматической переадресацией, предлагаемой методом `validate` запроса, вы можете вызвать метод `validate` на существующем экземпляре валидатора. В случае неудачи валидации, пользователь будет автоматически перенаправлен или, в случае запроса AJAX, будет возвращен JSON ответ:

```php
Validator::make($request->all(), [
    'title' => 'required|unique:posts|max:255',
    'body' => 'required',
])->validate();
```

Вы можете использовать метод `validateWithBag` для хранения сообщений об ошибках в [именованной коллекции ошибок](validation.md#named-error-bags), если проверка не прошла успешно:

```php
Validator::make($request->all(), [
    'title' => 'required|unique:posts|max:255',
    'body' => 'required',
])->validateWithBag('post');
```

### Именованные коллекции ошибок

Если у вас несколько форм на одной странице, вы можете дать имя коллекции `MessageBag` ошибок, что позволит вам получить сообщения об ошибках для конкретной формы. Передайте имя в качестве второго аргумента в `withErrors`:

```php
return redirect('register')
       ->withErrors($validator, 'login');
```

Затем вы можете получить доступ к именованному экземпляру `MessageBag` из переменной `$errors`:

```markup
{{ $errors->login->first('email') }}
```

### Хук после валидации

Валидатор также позволяет вам прикреплять обратные вызовы, которые будут выполняться после завершения валидации. Это позволяет вам легко выполнять дальнейшую проверку и даже добавлять больше сообщений об ошибках в коллекцию сообщений. Чтобы начать, используйте метод `after` на экземпляре валидатора:

```php
$validator = Validator::make(...);

$validator->after(function ($validator) {
    if ($this->somethingElseIsInvalid()) {
        $validator->errors()->add('field', 'Something is wrong with this field!');
    }
});

if ($validator->fails()) {
    //
}
```

## Работа с сообщениями об ошибках

После вызова метода `errors` экземпляра `Validator` Вы получите экземпляр `Illuminate\Support\MessageBag`, который имеет множество удобных методов работы с сообщениями об ошибках. Переменная `$errors`, которая автоматически становится доступной для всех представлений, также является экземпляром класса `MessageBag`.

**Получение первого сообщения об ошибке для поля**

Для получения первого сообщения об ошибке для данного поля используйте метод `first`:

```php
$errors = $validator->errors();

echo $errors->first('email');
```

**Получение всех сообщений об ошибках для поля**

Если Вам необходимо получить массив всех сообщений для данного поля, используйте метод `get`:

```php
foreach ($errors->get('email') as $message) {
    //
}
```

Если вы проверяете массив полей формы, вы можете получить все сообщения для каждого из элементов массива, используя символ `*`:

```php
foreach ($errors->get('attachments.*') as $message) {
    //
}
```

**Получение всех сообщений об ошибках для всех полей**

Чтобы получить массив всех сообщений для всех полей, используйте метод `all`:

```php
foreach ($errors->all() as $message) {
    //
}
```

**Определение наличия сообщений для поля**

Метод `has` может быть использован для определения наличия сообщений об ошибках для данного поля:

```php
if ($errors->has('email')) {
    //
}
```

### Пользовательские сообщения об ошибках

При необходимости вы можете использовать пользовательские сообщения об ошибках для проверки вместо сообщений по умолчанию. Существует несколько способов указать пользовательские сообщения. Во-первых, вы можете передать пользовательские сообщения в качестве третьего аргумента в метод `Validator::make`:

```php
$messages = [
    'required' => 'The :attribute field is required.',
];

$validator = Validator::make($input, $rules, $messages);
```

В этом примере плейсхолдер `:attribute` будет заменен на фактическое имя проверяемого поля. Вы также можете использовать другие плейсхолдеры в сообщениях с проверкой. Например:

```php
$messages = [
    'same' => 'The :attribute and :other must match.',
    'size' => 'The :attribute must be exactly :size.',
    'between' => 'The :attribute value :input is not between :min - :max.',
    'in' => 'The :attribute must be one of the following types: :values',
];
```

**Указание пользовательского сообщения для заданного атрибута**

Иногда может понадобиться указать пользовательское сообщение об ошибке только для конкретного поля. Это можно сделать с помощью нотации "точка". Сначала укажите имя атрибута, а затем правило:

```php
$messages = [
    'email.required' => 'We need to know your e-mail address!',
];
```

**Указание пользовательских сообщений в языковых файлах**

В большинстве случаев, вы, вероятно, будете указывать свои пользовательские сообщения в языковом файле вместо того, чтобы передавать их непосредственно в `Validator`. Для этого добавьте свои сообщения в массив `custom` в языковом файле `resources/lang/xx/validation.php`.

```php
'custom' => [
    'email' => [
        'required' => 'We need to know your e-mail address!',
    ],
],
```

**Указание пользовательских значений атрибута**

Если вы хотите, чтобы часть `:attribute` вашего сообщения о проверке достоверности была заменена именем пользовательского атрибута, вы можете указать пользовательское имя в массиве `attributes` языкового файла `resources/lang/xx/validation.php`:

```php
'attributes' => [
    'email' => 'email address',
],
```

Вы также можете передать пользовательские атрибуты в качестве четвертого аргумента в метод `Validator::make`:

```php
$customAttributes = [
    'email' => 'email address',
];

$validator = Validator::make($input, $rules, $messages, $customAttributes);
```

**Указание пользовательских значений в языковых файлах**

Иногда вам может понадобиться заменить часть `:value` вашего сообщения о подтверждении на пользовательское представление значения. Например, рассмотрим следующее правило, которое указывает, что номер кредитной карты необходим, если `payment_type` имеет значение `cc`:

```php
$request->validate([
    'credit_card_number' => 'required_if:payment_type,cc'
]);
```

Если это правило проверки дает сбой, оно выдает следующее сообщение об ошибке:

```text
The credit card number field is required when payment type is cc.
```

Вместо того, чтобы отображать `cc` в качестве значения типа платежа, вы можете указать пользовательское представление значения в вашем `validation` языковом файле, определив массив `values`:

```php
'values' => [
    'payment_type' => [
        'cc' => 'credit card'
    ],
],
```

Теперь, если правило проверки не работает, оно выдаст следующее сообщение:

```text
The credit card number field is required when payment type is credit card.
```

## Доступные правила проверки

Ниже приведен список всех доступных правил проверки и их функции:

* [Accepted](validation.md#rule-accepted)
* [Active URL](validation.md#rule-active-url)
* [After \(Date\)](validation.md#rule-after)
* [After Or Equal \(Date\)](validation.md#rule-after-or-equal)
* [Alpha](validation.md#rule-alpha)
* [Alpha Dash](validation.md#rule-alpha-dash)
* [Alpha Numeric](validation.md#rule-alpha-num)
* [Array](validation.md#rule-array)
* [Bail](validation.md#rule-bail)
* [Before \(Date\)](validation.md#rule-before)
* [Before Or Equal \(Date\)](validation.md#rule-before-or-equal)
* [Between](validation.md#rule-between)
* [Boolean](validation.md#rule-boolean)
* [Confirmed](validation.md#rule-confirmed)
* [Date](validation.md#rule-date)
* [Date Equals](validation.md#rule-date-equals)
* [Date Format](validation.md#rule-date-format)
* [Different](validation.md#rule-different)
* [Digits](validation.md#rule-digits)
* [Digits Between](validation.md#rule-digits-between)
* [Dimensions \(Image Files\)](validation.md#rule-dimensions)
* [Distinct](validation.md#rule-distinct)
* [Email](validation.md#rule-email)
* [Ends With](validation.md#rule-ends-with)
* [Exclude If](validation.md#rule-exclude-if)
* [Exclude Unless](validation.md#rule-exclude-unless)
* [Exists \(Database\)](validation.md#rule-exists)
* [File](validation.md#rule-file)
* [Filled](validation.md#rule-filled)
* [Greater Than](validation.md#rule-gt)
* [Greater Than Or Equal](validation.md#rule-gte)
* [Image \(File\)](validation.md#rule-image)
* [In](validation.md#rule-in)
* [In Array](validation.md#rule-in-array)
* [Integer](validation.md#rule-integer)
* [IP Address](validation.md#rule-ip)
* [JSON](validation.md#rule-json)
* [Less Than](validation.md#rule-lt)
* [Less Than Or Equal](validation.md#rule-lte)
* [Max](validation.md#rule-max)
* [MIME Types](validation.md#rule-mimetypes)
* [MIME Type By File Extension](validation.md#rule-mimes)
* [Min](validation.md#rule-min)
* [Not In](validation.md#rule-not-in)
* [Not Regex](validation.md#rule-not-regex)
* [Nullable](validation.md#rule-nullable)
* [Numeric](validation.md#rule-numeric)
* [Password](validation.md#rule-password)
* [Present](validation.md#rule-present)
* [Regular Expression](validation.md#rule-regex)
* [Required](validation.md#rule-required)
* [Required If](validation.md#rule-required-if)
* [Required Unless](validation.md#rule-required-unless)
* [Required With](validation.md#rule-required-with)
* [Required With All](validation.md#rule-required-with-all)
* [Required Without](validation.md#rule-required-without)
* [Required Without All](validation.md#rule-required-without-all)
* [Same](validation.md#rule-same)
* [Size](validation.md#rule-size)
* [Sometimes](validation.md#conditionally-adding-rules)
* [Starts With](validation.md#rule-starts-with)
* [String](validation.md#rule-string)
* [Timezone](validation.md#rule-timezone)
* [Unique \(Database\)](validation.md#rule-unique)
* [URL](validation.md#rule-url)[UUID](validation.md#rule-uuid)

**accepted**

Поле под проверкой должно быть _yes_, _on_, _1_, или _true_. Это полезно для подтверждения принятия "Terms of Service".

**active\_url**

Поле под валидацией должно иметь действительную A или AAAA-запись в соответствии с PHP-функцией `dns_get_record`. Имя хоста предоставленного URL извлекается с помощью PHP-функции `parse_url` перед передачей в `dns_get_record`.

**after:date**

Поле, подлежащее проверке, должно быть значением после заданной даты. Даты будут переданы в PHP-функцию `strtotime`:

```php
'start_date' => 'required|date|after:tomorrow'
```

Вместо передачи строки даты, которая будет оцениваться `strtotime`, вы можете указать другое поле для сравнения с датой:

```php
'finish_date' => 'required|date|after:start_date'
```

**after\_or\_equal:date**

Поле, подлежащее проверке, должно быть значением после или равным данной дате. Для получения дополнительной информации см. правило [after](validation.md#rule-after).

**alpha**

Поле, подлежащее проверке, должно содержать только буквенные символы.

**alpha\_dash**

Поле, подлежащее проверке, может содержать буквенно-цифровые символы, а также тире и подчеркивания.

**alpha\_num**

Поле, подлежащее проверке, должно содержать только буквенно-цифровые символы.

**array**

Проверяемое поле должно быть PHP-массивом.

**bail**

Прекрать выполнение правил проверки после первой неудачи проверки.

**before:date**

Поле, подлежащее проверке, должно быть значением, предшествующим данной дате. Даты будут переданы в функцию PHP `strtotime`. Кроме того, как и в правиле [`after`](validation.md#rule-after), в качестве значения `date` может быть введено имя другого поля, находящегося на проверке.

**before\_or\_equal:date**

Проверяемое поле должно быть значением, предшествующим или равным данной дате. Даты будут переданы в функцию PHP `strtotime`. Кроме того, как и в правиле [`after`](validation.md#rule-after), в качестве значения `date` может быть введено имя другого поля, находящегося на проверке.

**between:min,max**

Проверяемое поле должно иметь размер между _min_ и _max_. Строки, цифры, массивы и файлы оцениваются так же, как и правило [`size`](validation.md#rule-size).

**boolean**

Проверяемое поле должно способно быть представлено как булевое. Принимаются знаечния `true`, `false`, `1`, `0`, `"1"` и `"0"`.

**confirmed**

Проверяемое поле должно иметь соответствующее поле `foo_confirmation`. Например, если проверяемое поле `password`, то во входных данных должно присутствовать поле `password_confirmation`, совпадающее с полем `password`.

**date**

Проверяемое поле должно быть достоверной, non-relative датой в соответствии с PHP-функцией `strtotime`.

**date\_equals:date**

Проверяемое поле должно быть равно заданной дате. Даты будут переданы в функцию PHP `strtotime`.

**date\_format:format**

Проверяемое поле должно совпадать с заданным _форматом_. При проверке поля следует использовать `date` **или** `date_format`, а не оба. Это правило проверки поддерживает все форматы, поддерживаемые PHP классом [DateTime](https://www.php.net/manual/en/class.datetime.php).

**different:field**

Проверяемое поле должно иметь значение, отличное от _field_.

**digits:value**

Проверяемое поле должно быть _numeric_ и должно иметь точную длину _value_.

**digits\_between:min,max**

Проверяемое поле должно быть _numeric_ и должно иметь длину между заданными _min_ и _max_.

**dimensions**

Файл, находящийся на проверке, должен быть изображением, удовлетворяющим ограничениям по размеру, указанным в параметрах правила:

```php
'avatar' => 'dimensions:min_width=100,min_height=200'
```

Доступные ограничения: _min\_width_, _max\_width_, _min\_height_, _max\_height_, _width_, _height_, _ratio_.

Ограничение _ratio_ должно быть представлено как ширина, деленная на высоту. Это может быть задано либо оператором типа `3/2`, либо плавающей точкой типа `1.5`:

```php
'avatar' => 'dimensions:ratio=3/2'
```

Поскольку это правило требует наличия нескольких аргументов, для его построения можно использовать метод `Rule::dimensions`:

```php
use Illuminate\Validation\Rule;

Validator::make($data, [
    'avatar' => [
        'required',
        Rule::dimensions()->maxWidth(1000)->maxHeight(500)->ratio(3 / 2),
    ],
]);
```

**distinct**

При работе с массивами поле, подлежащее проверке, не должно иметь дублирующих значений.

```php
'foo.*.id' => 'distinct'
```

**email**

Поле, подлежащее проверке, должно быть отформатировано в виде адреса электронной почты. Под капотом, это правило проверки использует пакет [`egulias/email-validator`](https://github.com/egulias/EmailValidator) для проверки электронного адреса. По умолчанию применяется валидатор `RFCValidation`, но вы можете применять и другие стили валидации:

```php
'email' => 'email:rfc,dns'
```

В примере выше будут применены проверки `RFCValidation` и `DNSCheckValidation`. Вот полный список стилей проверки, которые вы можете применить:

* `rfc`: `RFCValidation`
* `strict`: `NoRFCWarningsValidation`
* `dns`: `DNSCheckValidation`
* `spoof`: `SpoofCheckValidation`
* `filter`: `FilterEmailValidation`

Валидатор `filter`, использующий под капотом функцию PHP `filter_var`, поставляется с Laravel и является поведением до версии 5.8. Валидаторы `dns` и `spoof` требуют расширения PHP `intl`.

**ends\_with:foo,bar,...**

Проверяемое поле должно заканчиваться одним из заданных значений.

**exclude\_if:anotherfield,value**

Проверяемое поле будет исключено из данных запроса, возвращаемых методами `validate` и `validated`, если поле _another field_ равно _value_.

**exclude\_unless:anotherfield,value**

Проверяемое поле будет исключено из данных запроса, возвращаемых методами `validate` и `validated`, если поле _anotherfield_ не будет равно _value_.

**exists:table,column**

Проверяемое поле должно существовать в заданной таблице базы данных.

**Основное использование существующих правил**

```php
'state' => 'exists:states'
```

Если опция `column` не указана, то будет использовано название поля.

**Указание пользовательского названия столбца**

```php
'state' => 'exists:states,abbreviation'
```

Иногда может понадобиться указать конкретное подключение к БД, которое будет использоваться для существующего \(`exists`\) запроса. Вы можете сделать это, предопределив имя подключения к имени таблицы с помощью синтаксиса "точка":

```php
'email' => 'exists:connection.staff,email'
```

Вместо непосредственного указания имени таблицы можно указать модель Eloquent, которую следует использовать для определения имени таблицы:

```php
'user_id' => 'exists:App\User,id'
```

Если вы хотите настроить запрос, выполняемый по правилу проверки, вы можете использовать класс `Rule` для беглого определения правила. В этом примере мы также укажем правила валидации в виде массива вместо использования символа `|` для их разделения:

```php
use Illuminate\Validation\Rule;

Validator::make($data, [
    'email' => [
        'required',
        Rule::exists('staff')->where(function ($query) {
            $query->where('account_id', 1);
        }),
    ],
]);
```

**file**

Поле, находящееся на проверке, должно быть успешно загруженным файлом.

**filled**

Поле, находящееся на проверке, не должно быть пустым, когда оно присутствует.

**gt:field**

Проверяемое поле должно быть больше заданного _field_. Эти два поля должны быть одного типа. Строки, цифры, массивы и файлы оцениваются по тем же конвенциям, что и правило [`size`](validation.md#rule-size).

**gte:field**

Проверяемое поле должно быть больше или равно заданному _field_. Эти два поля должны быть одного типа. Строки, цифры, массивы и файлы оцениваются по тем же конвенциям, что и правило [`size`](validation.md#rule-size).

**image**

Файл, находящийся на проверке, должен представлять собой изображение \(jpeg, png, bmp, gif, svg или webp\).

**in:foo,bar,...**

Проверяемое поле должно быть включено в данный список значений. Так как это правило часто требует "имплодить" \(`implode`\) массив, то для построения правила можно использовать метод `Rule::in`:

```php
use Illuminate\Validation\Rule;

Validator::make($data, [
    'zones' => [
        'required',
        Rule::in(['first-zone', 'second-zone']),
    ],
]);
```

**in\_array:anotherfield.\***

Проверяемое поле должно существовать в значениях других полей _anotherfield_.

**integer**

Проверяемое поле должно быть целым числом.

{% hint style="warning" %}
Данное правило проверки не проверяет, что входные данные являются типом "целой" переменной, а только то, что они являются строкой или числовым значением, содержащим целое число.
{% endhint %}

**ip**

Поле, подлежащее проверке, должно быть IP-адресом.

**ipv4**

Проверяемое поле должно быть адресом IPv4.

**ipv6**

Проверяемое поле должно быть адресом IPv6.

**json**

Проверяемое поле должно быть правильной JSON строкой.

**lt:field**

Проверяемое поле должно быть меньше заданного _field_. Эти два поля должны быть одного типа. Строки, цифры, массивы и файлы оцениваются по тем же конвенциям, что и правило [`size`](validation.md#rule-size).

**lte:field**

Проверяемое поле должно быть меньше или равно заданному _field_. Эти два поля должны быть одного типа. Строки, цифры, массивы и файлы оцениваются по тем же конвенциям, что и правило [`size`](validation.md#rule-size).

**max:value**

Поле, подлежащее проверке, должно быть меньше или равно максимальному значению _value_. Строки, цифры, массивы и файлы оцениваются таким же образом, как и правило [`size`](validation.md#rule-size).

**mimetypes:text/plain,...**

Проверяемый файл должен соответствовать одному из MIME-типов:

```php
'video' => 'mimetypes:video/avi,video/mpeg,video/quicktime'
```

Для определения MIME-типа загружаемого файла, содержимое файла будет прочитано, а фреймворк попытается угадать MIME-тип, который может отличаться от MIME-типа, предоставляемого клиентом.

**mimes:foo,bar,...**

Проверяемый файл должен иметь MIME тип, соответствующий одному из перечисленных расширений.

**Основное использование правил MIME**

```php
'photo' => 'mimes:jpeg,bmp,png'
```

Несмотря на то, что вам нужно лишь указать расширения, это правило на самом деле проверяет MIME тип файла, читая его содержимое и угадывая его MIME тип.

Полный список типов MIME и их соответствующих расширений можно найти в следующем месте: [https://svn.apache.org/repos/asf/httpd/httpd/trunk/docs/conf/mime.types](https://svn.apache.org/repos/asf/httpd/httpd/trunk/docs/conf/mime.types)

**min:value**

Проверяемое поле должно иметь минимальное _value_. Строки, цифры, массивы и файлы оцениваются так же, как и правило [`size`](validation.md#rule-size).

**not\_in:foo,bar,...**

Проверяемое поле не должно быть включено в данный список значений. Для построения правила может быть использован метод `Rule::notIn`:

```php
use Illuminate\Validation\Rule;

Validator::make($data, [
    'toppings' => [
        'required',
        Rule::notIn(['sprinkles', 'cherries']),
    ],
]);
```

**not\_regex:pattern**

Поле под валидацией не должно совпадать с заданным регулярным выражением.

Внутренне это правило использует функцию PHP `preg_match`. Указанный шаблон должен подчиняться тому же форматированию, которое требуется в `preg_match` и, таким образом, также включать в себя допустимые разделители. Например: `'email' => 'not_regex:/^.+$/i'`.

**Замечание:** При использовании шаблонов `regex`/`not_regex` может потребоваться указать правила в массиве вместо использования разделителей "\|", особенно если регулярное выражение содержит символ "\|".

**nullable**

Проверяемое поле может быть "null". Это особенно полезно при валидации примитивов, таких как строки и целые числа, которые могут содержать `null` значения.

**numeric**

Проверяемое поле должно быть числовым.

**password**

Проверяемое поле должно совпадать с паролем аутентифицированного пользователя. Вы можете указать защиту от аутентификации, используя первый параметр правила:

```php
'password' => 'password:api'
```

**present**

Проверяемое поле должно присутствовать во входных данных, но может быть пустым.

**regex:pattern**

Проверяемое поле должно соответствовать заданному регулярному выражению.

Внутренне это правило использует функцию PHP `preg_match`. Указанный шаблон должен подчиняться тому же форматированию, которое требуется в `preg_match` и, таким образом, также включать в себя допустимые разделители. Например: `'email' => 'regex:/^.+@.+$/i'`.

**Замечание:** При использовании шаблонов `regex`/`not_regex` может потребоваться указать правила в массиве вместо использования разделителей "\|", особенно если регулярное выражение содержит символ "\|".

**required**

Ппроверяемое поле должно присутствовать во входных данных и не должно быть пустым. Поле считается "пустым", если одно из следующих условий является верным:

* Значение равно `null`.
* Значение является пустой строкой.
* Значение является пустым массивом и пустым объектом `Countable`.
* Значение представляет собой загруженный файл без пути.

**required\_if:anotherfield,value,...**

Проверяемое поле должно присутствовать и не должно быть пустым, если поле _anotherfield_ равно любому значению _value_.

Если вы хотите построить более сложное условие для правила `required_if`, вы можете использовать метод `Rule::requiredIf`. Этот метод принимает логическое значение или Closure. После прохождения Closure, Closure должен возвращать `true` или `false`, чтобы указать, требуется ли проверяемое поле:

```php
use Illuminate\Validation\Rule;

Validator::make($request->all(), [
    'role_id' => Rule::requiredIf($request->user()->is_admin),
]);

Validator::make($request->all(), [
    'role_id' => Rule::requiredIf(function () use ($request) {
        return $request->user()->is_admin;
    }),
]);
```

**required\_unless:anotherfield,value,...**

Проверяемое поле должно присутствовать и не должно быть пустым, если только поле _anotherfield_ не равно любому значению _value_.

**required\_with:foo,bar,...**

Поле, находящееся на проверке, должно присутствовать и не должно быть пустым _только если_ любое из остальных указанных полей присутствует.

**required\_with\_all:foo,bar,...**

Поле, находящееся на проверке, должно присутствовать и не должно быть пустым _только если_ присутствуют все остальные указанные поля.

**required\_without:foo,bar,...**

Поле, находящееся на проверке, должно присутствовать и не должно быть пустым _только тогда_, когда нет ни одного из остальных указанных полей.

**required\_without\_all:foo,bar,...**

Проверяемое поле должно присутствовать и не должно быть пустым _только когда_ все остальные указанные поля отсутствуют.

**same:field**

Данное _поле_ должно соответствовать проверяемому полю.

**size:value**

Проверяемое поле должно иметь размер, соответствующий заданному значению _value_. Для строковых данных _value_ соответствует количеству символов. Для числовых данных _value_ соответствует заданному целому значению \(атрибут также должен иметь `numeric` или `integer` правило\). Для массива _size_ соответствует `value` массива. Для файлов _size_ соответствует размеру файла в килобайтах. Рассмотрим некоторые примеры:

```php
// Validate that a string is exactly 12 characters long...
'title' => 'size:12';

// Validate that a provided integer equals 10...
'seats' => 'integer|size:10';

// Validate that an array has exactly 5 elements...
'tags' => 'array|size:5';

// Validate that an uploaded file is exactly 512 kilobytes...
'image' => 'file|size:512';
```

**starts\_with:foo,bar,...**

Проверяемое поле должно начинаться с одного из заданных значений.

**string**

Проверяемое поле должно быть строкой. Если Вы хотите, чтобы поле также было `null`, Вам следует присвоить этому полю правило `nullable`.

**timezone**

Проверяемое поле должно быть валидным идентификатором часового пояса в соответствии с PHP-функцией `timezone_identifiers_list`.

**unique:table,column,except,idColumn**

Проверяемое поле не должно существовать в данной таблице БД.

**Указание пользовательского названия таблицы/столбца:**

Вместо непосредственного указания имени таблицы можно указать модель Eloquent, которую следует использовать для определения имени таблицы:

```php
'email' => 'unique:App\User,email_address'
```

Опция `column` может быть использована для указания соответствующего столбца БД поля. Если опция `column` не указана, то будет использовано название поля.

```php
'email' => 'unique:users,email_address'
```

**Пользовательское подключение к базе данных**

Иногда может понадобиться установить пользовательское соединение для запросов к БД, выполняемых валидатором. Как видно выше, установка `unique:users` в качестве правила проверки будет использовать подключение по умолчанию для запросов к БД. Чтобы переопределить это, укажите соединение и имя таблицы, используя синтаксис "точка":

```php
'email' => 'unique:connection.users,email_address'
```

**Заставить уникальное правило игнорировать выданный ID:**

Иногда может возникнуть желание проигнорировать заданный ID во время уникальной проверки. Например, рассмотрим окно "Обновить профиль", в котором указывается имя пользователя, адрес электронной почты и местоположение. Скорее всего, вы захотите проверить, что адрес электронной почты уникален. Однако, если пользователь изменяет только поле имени, а не поле электронной почты, вы не хотите, чтобы ошибка проверки была вызвана тем, что пользователь уже является владельцем адреса электронной почты.

Чтобы указать валидатору игнорировать идентификатор пользователя, мы воспользуемся классом `Rule` для определения правила. В этом примере мы также укажем правила проверки в виде массива вместо использования символа `|` для разделения правил:

```php
use Illuminate\Validation\Rule;

Validator::make($data, [
    'email' => [
        'required',
        Rule::unique('users')->ignore($user->id),
    ],
]);
```

{% hint style="warning" %}
Вы никогда не должны передавать в метод `ignore` какие-либо управляемые пользователем входные данные запроса. Вместо этого, вы должны передавать только сгенерированный системой уникальный ID, такой как автоинкрементирующий ID или UUID от экземпляра модели Eloquent. В противном случае ваше приложение будет уязвимо для атаки SQL инъекции.
{% endhint %}

Вместо того, чтобы передавать значение ключа модели методу `ignore`, можно передать весь экземпляр модели. Laravel автоматически извлечет ключ из модели:

```php
Rule::unique('users')->ignore($user)
```

Если ваша таблица использует имя столбца первичного ключа, отличное от `id`, вы можете указать имя столбца при вызове метода `ignore`:

```php
Rule::unique('users')->ignore($user->id, 'user_id')
```

По умолчанию правило `unique` проверяет уникальность колонки, соответствующей имени проверяемого атрибута. Однако, Вы можете передать другому аргументу имя колонки в качестве второго аргумента в метод `unique`:

```php
Rule::unique('users', 'email_address')->ignore($user->id),
```

**Добавление дополнительных условий:**

Вы также можете указать дополнительные ограничения на запрос, настроив его с помощью метода `where`. Например, добавим ограничение, которое проверяет, что `account_id` равен `1`:

```php
'email' => Rule::unique('users')->where(function ($query) {
    return $query->where('account_id', 1);
})
```

**url**

Проверяемое поле должно быть действительным URL-адресом.

**uuid**

Проверяемое поле должно быть действительным RFC 4122 \(версии 1, 3, 4 или 5\) универсальным уникальным идентификатором \(UUID\).

## Правила условного добавления

**Пропуск проверки, когда поля имеют определенные значения**

Время от времени Вы можете захотеть не проверять данное поле, если другое поле имеет заданное значение. Вы можете сделать это, используя правило проверки `exclude_if`. В данном примере поля `appointment_date` и `doctor_name` не будут проверяться, если поле `has_appointment` имеет значение `false`:

```php
$v = Validator::make($data, [
    'has_appointment' => 'required|bool',
    'appointment_date' => 'exclude_if:has_appointment,false|required|date',
    'doctor_name' => 'exclude_if:has_appointment,false|required|string',
]);
```

В качестве альтернативы, вы можете использовать правило `exclude_unless`, чтобы не проверять данное поле, если только другое поле не имеет заданного значения:

```php
$v = Validator::make($data, [
    'has_appointment' => 'required|bool',
    'appointment_date' => 'exclude_unless:has_appointment,true|required|date',
    'doctor_name' => 'exclude_unless:has_appointment,true|required|string',
]);
```

**Проверка при наличии**

В некоторых ситуациях, если это поле присутствует во входном массиве, можно запустить проверку валидности с полем **only**. Чтобы быстро это сделать, добавьте правило `sometimes` в список правил:

```php
$v = Validator::make($data, [
    'email' => 'sometimes|required|email',
]);
```

В приведенном выше примере поле `email` будет проверено только в том случае, если оно присутствует в массиве `$data`.

{% hint style="info" %}
Если вы пытаетесь проверить поле, которое всегда должно присутствовать, но может быть пустым, обратите внимание [на эту заметку о необязательных полях](validation.md#a-note-on-optional-fields)
{% endhint %}

**Комплексная условная проверка**

Иногда может возникнуть желание добавить правила проверки, основанные на более сложной условной логике. Например, вы можете захотеть потребовать заданное поле только в том случае, если другое поле имеет большее значение, чем 100. Или же вам могут понадобиться два поля, чтобы иметь заданное значение только в том случае, если другое поле присутствует. Во-первых, создайте экземпляр `Validator` с вашими _статическими правилами_, которые никогда не меняются:

```php
$v = Validator::make($data, [
    'email' => 'required|email',
    'games' => 'required|numeric',
]);
```

Предположим, что наше веб-приложение предназначено для коллекционеров игр. Если игровой коллектор регистрируется в нашем приложении и ему принадлежит более 100 игр, мы хотим, чтобы они объяснили, почему им принадлежит так много игр. Например, возможно, они управляют магазином по перепродаже игр, или, может быть, им просто нравится собирать. Чтобы условно добавить это требование, мы можем использовать метод `sometimes` в `Validator`.

```php
$v->sometimes('reason', 'required|max:500', function ($input) {
    return $input->games >= 100;
});
```

Первый аргумент, переданный в метод `sometimes` - это имя поля, которое мы условно проверяем. Второй аргумент - правила, которые мы хотим добавить. Если `Closure`, переданный в качестве третьего аргумента, возвращает `true`, то правила будут добавлены. Этот метод делает построение сложных условных валидаций легким делом. Можно даже добавлять условные валидации сразу для нескольких полей:

```php
$v->sometimes(['reason', 'cost'], 'required', function ($input) {
    return $input->games >= 100;
});
```

{% hint style="info" %}
Параметр `$input`, переданный вашему `Closure`, будет экземпляром `Illuminate\Support\Fluent` и может быть использован для доступа к вашим входным данным и файлам.
{% endhint %}

## Проверка массивов

Проверка полей ввода форм, основанных на массивах, не должна быть нудной. Для проверки атрибутов внутри массива можно использовать "точечную нотацию". Например, если входящий HTTP-запрос содержит поле `photos[profile]`, вы можете проверить его таким образом:

```php
$validator = Validator::make($request->all(), [
    'photos.profile' => 'required|image',
]);
```

Вы также можете проверить каждый элемент массива. Например, чтобы проверить, что каждое письмо в данном поле ввода массива уникально, можно сделать следующее:

```php
$validator = Validator::make($request->all(), [
    'person.*.email' => 'email|unique:users',
    'person.*.first_name' => 'required_with:person.*.last_name',
]);
```

Аналогичным образом, вы можете использовать символ `*` при указании ваших валидационных сообщений в языковых файлах, что сделает использование одного валидационного сообщения для полей на базе массива легким делом:

```php
'custom' => [
    'person.*.email' => [
        'unique' => 'Each person must have a unique e-mail address',
    ]
],
```

## Пользовательские правила проверки

### Использование объектов правил

Laravel предоставляет множество полезных правил проверки; однако, вы можете указать некоторые из них. Одним из способов регистрации пользовательских правил валидации является использование объектов правил. Чтобы сгенерировать новый объект правила, вы можете использовать команду `make:rule` Artisan. Давайте используем эту команду для генерации правила, которое проверяет, что строка заглавная. Ларавел поместит новое правило в директорию `app/Rules`:

```bash
php artisan make:rule Uppercase
```

После создания правила мы готовы определить его поведение. Объект правила содержит два метода: `passes` и `message`. Метод `passes` получает значение и имя атрибута и должен возвращать `true` или `false` в зависимости от того, является ли значение атрибута действительным или нет. Метод `message` должен возвращать сообщение об ошибке проверки, которое должно использоваться в случае неудачи проверки:

```php
<?php

namespace App\Rules;

use Illuminate\Contracts\Validation\Rule;

class Uppercase implements Rule
{
    /**
     * Determine if the validation rule passes.
     *
     * @param  string  $attribute
     * @param  mixed  $value
     * @return bool
     */
    public function passes($attribute, $value)
    {
        return strtoupper($value) === $value;
    }

    /**
     * Get the validation error message.
     *
     * @return string
     */
    public function message()
    {
        return 'The :attribute must be uppercase.';
    }
}
```

Вы можете вызвать помощника `trans` из вашего метода `message`, если хотите вернуть сообщение об ошибке из ваших файлов перевода:

```php
/**
 * Get the validation error message.
 *
 * @return string
 */
public function message()
{
    return trans('validation.uppercase');
}
```

После того, как правило определено, вы можете прикрепить его к валидатору, передав экземпляр объекта правила вместе с другими валидационными правилами:

```php
use App\Rules\Uppercase;

$request->validate([
    'name' => ['required', 'string', new Uppercase],
]);
```

### Использование функций обратного вызова

Если вам нужна функциональность пользовательского правила только один раз на протяжении всего приложения, вы можете использовать Closure вместо объекта правила. Closure получает имя атрибута, значение атрибута и обратный вызов `$fail`, который должен быть вызван, если проверка не прошла успешно:

```php
$validator = Validator::make($request->all(), [
    'title' => [
        'required',
        'max:255',
        function ($attribute, $value, $fail) {
            if ($value === 'foo') {
                $fail($attribute.' is invalid.');
            }
        },
    ],
]);
```

### Использование расширений

Другой способ регистрации пользовательских правил валидации - использование метода `extend` [фасада](https://laravel.com/docs/7.x/facades) `Validator`. Используем этот метод внутри [сервис-провайдера](https://laravel.com/docs/7.x/providers) для регистрации пользовательского правила проверки:

```php
<?php

namespace App\Providers;

use Illuminate\Support\ServiceProvider;
use Illuminate\Support\Facades\Validator;

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
        Validator::extend('foo', function ($attribute, $value, $parameters, $validator) {
            return $value == 'foo';
        });
    }
}
```

Пользовательский валидатор Closure получает четыре аргумента: имя проверяемого атрибута `$attribute`, значение атрибута `$value`, массив параметров, переданных в правило `$parameters`, и экземпляр `Validator`.

Вы также можете передать класс и метод в метод `extend` вместо Closure:

```php
Validator::extend('foo', 'FooValidator@validate');
```

**Определение сообщения об ошибке**

Вам также нужно будет определить сообщение об ошибке для вашего пользовательского правила. Вы можете сделать это либо с помощью встроенного пользовательского массива сообщений, либо добавив запись в файл языка проверки. Это сообщение должно быть размещено на первом уровне массива, а не в массиве `custom`, который предназначен только для сообщений об ошибках, специфичных для атрибутов:

```php
"foo" => "Your input was invalid!",

"accepted" => "The :attribute must be accepted.",

// The rest of the validation error messages...
```

При создании пользовательского правила проверки, иногда может понадобиться определить пользовательские замены для сообщений об ошибках. Вы можете сделать это, создав пользовательский валидатор, как описано выше, а затем вызвать метод `Replacer` фасада `Validator`. Вы можете сделать это в рамках метода `boot` [сервис-провайдера](https://laravel.com/docs/7.x/providers):

```php
/**
 * Bootstrap any application services.
 *
 * @return void
 */
public function boot()
{
    Validator::extend(...);

    Validator::replacer('foo', function ($message, $attribute, $rule, $parameters) {
        return str_replace(...);
    });
}
```

### Неявные расширения

По умолчанию, когда проверяемый атрибут отсутствует или содержит пустую строку, обычные правила проверки, включая пользовательские расширения, не выполняются. Например, правило [`unique`](validation.md#rule-unique) не будет запущено против пустой строки:

```php
$rules = ['name' => 'unique:users,name'];

$input = ['name' => ''];

Validator::make($input, $rules)->passes(); // true
```

Для того, чтобы правило выполнялось, даже когда атрибут пуст, правило должно подразумевать, что атрибут является обязательным. Для создания такого "неявного" расширения используйте метод `Validator::extensionImplicit()`:

```php
Validator::extendImplicit('foo', function ($attribute, $value, $parameters, $validator) {
    return $value == 'foo';
});
```

{% hint style="warning" %}
Неявное расширение только _implies_, что атрибут является обязательным. Действительно ли оно делает недействительным отсутствующий или пустой атрибут, зависит от Вас.
{% endhint %}

**Неявные объекты правила**

Если вы хотите, чтобы объект правила запускался, когда атрибут пуст, то вы должны реализовать интерфейс `Illuminate\Contracts\Validation\ImplicitRule`. Этот интерфейс служит "marker interface" для валидатора, поэтому он не содержит методов, которые необходимо реализовать.

