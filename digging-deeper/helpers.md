# Помощники

## Introduction

Laravel includes a variety of global "helper" PHP functions. Many of these functions are used by the framework itself; however, you are free to use them in your own applications if you find them convenient.

## Available Methods

### Arrays & Objects

[Arr::accessible](helpers.md#method-array-accessible) [Arr::add](helpers.md#method-array-add) [Arr::collapse](helpers.md#method-array-collapse) [Arr::crossJoin](helpers.md#method-array-crossjoin) [Arr::divide](helpers.md#method-array-divide) [Arr::dot](helpers.md#method-array-dot)[Arr::except](helpers.md#method-array-except)[Arr::exists](helpers.md#method-array-exists)[Arr::first](helpers.md#method-array-first)[Arr::flatten](helpers.md#method-array-flatten)[Arr::forget](helpers.md#method-array-forget)[Arr::get](helpers.md#method-array-get)[Arr::has](helpers.md#method-array-has)[Arr::hasAny](helpers.md#method-array-hasany)[Arr::isAssoc](helpers.md#method-array-isassoc)[Arr::last](helpers.md#method-array-last)[Arr::only](helpers.md#method-array-only)[Arr::pluck](helpers.md#method-array-pluck)[Arr::prepend](helpers.md#method-array-prepend)[Arr::pull](helpers.md#method-array-pull)[Arr::query](helpers.md#method-array-query)[Arr::random](helpers.md#method-array-random)[Arr::set](helpers.md#method-array-set)[Arr::shuffle](helpers.md#method-array-shuffle)[Arr::sort](helpers.md#method-array-sort)[Arr::sortRecursive](helpers.md#method-array-sort-recursive)[Arr::where](helpers.md#method-array-where)[Arr::wrap](helpers.md#method-array-wrap)[data\_fill](helpers.md#method-data-fill)[data\_get](helpers.md#method-data-get)[data\_set](helpers.md#method-data-set)[head](helpers.md#method-head)[last](helpers.md#method-last)

### Paths

 \[app\\_path\]\(\#method-app-path\) \[base\\_path\]\(\#method-base-path\) \[config\\_path\]\(\#method-config-path\) \[database\\_path\]\(\#method-database-path\) \[mix\]\(\#method-mix\) \[public\\_path\]\(\#method-public-path\) \[resource\\_path\]\(\#method-resource-path\) \[storage\\_path\]\(\#method-storage-path\)

### Strings

[\_\_](helpers.md#method-__)[class\_basename](helpers.md#method-class-basename)[e](helpers.md#method-e)[preg\_replace\_array](helpers.md#method-preg-replace-array)[Str::after](helpers.md#method-str-after)[Str::afterLast](helpers.md#method-str-after-last)[Str::ascii](helpers.md#method-str-ascii)[Str::before](helpers.md#method-str-before)[Str::beforeLast](helpers.md#method-str-before-last)[Str::between](helpers.md#method-str-between)[Str::camel](helpers.md#method-camel-case)[Str::contains](helpers.md#method-str-contains)[Str::containsAll](helpers.md#method-str-contains-all)[Str::endsWith](helpers.md#method-ends-with)[Str::finish](helpers.md#method-str-finish)[Str::is](helpers.md#method-str-is)[Str::isAscii](helpers.md#method-str-is-ascii)[Str::isUuid](helpers.md#method-str-is-uuid)[Str::kebab](helpers.md#method-kebab-case)[Str::length](helpers.md#method-str-length)[Str::limit](helpers.md#method-str-limit)[Str::lower](helpers.md#method-str-lower)[Str::orderedUuid](helpers.md#method-str-ordered-uuid)[Str::plural](helpers.md#method-str-plural)[Str::random](helpers.md#method-str-random)[Str::replaceArray](helpers.md#method-str-replace-array)[Str::replaceFirst](helpers.md#method-str-replace-first)[Str::replaceLast](helpers.md#method-str-replace-last)[Str::singular](helpers.md#method-str-singular)[Str::slug](helpers.md#method-str-slug)[Str::snake](helpers.md#method-snake-case)[Str::start](helpers.md#method-str-start)[Str::startsWith](helpers.md#method-starts-with)[Str::studly](helpers.md#method-studly-case)[Str::substr](helpers.md#method-str-substr)[Str::title](helpers.md#method-title-case)[Str::ucfirst](helpers.md#method-str-ucfirst)[Str::upper](helpers.md#method-str-upper)[Str::uuid](helpers.md#method-str-uuid)[Str::words](helpers.md#method-str-words)[trans](helpers.md#method-trans)[trans\_choice](helpers.md#method-trans-choice)

### Fluent Strings

[after](helpers.md#method-fluent-str-after)[afterLast](helpers.md#method-fluent-str-after-last)[append](helpers.md#method-fluent-str-append)[ascii](helpers.md#method-fluent-str-ascii)[basename](helpers.md#method-fluent-str-basename)[before](helpers.md#method-fluent-str-before)[beforeLast](helpers.md#method-fluent-str-before-last)[camel](helpers.md#method-fluent-str-camel)[contains](helpers.md#method-fluent-str-contains)[containsAll](helpers.md#method-fluent-str-contains-all)[dirname](helpers.md#method-fluent-str-dirname)[endsWith](helpers.md#method-fluent-str-ends-with)[exactly](helpers.md#method-fluent-str-exactly)[explode](helpers.md#method-fluent-str-explode)[finish](helpers.md#method-fluent-str-finish)[is](helpers.md#method-fluent-str-is)[isAscii](helpers.md#method-fluent-str-is-ascii)[isEmpty](helpers.md#method-fluent-str-is-empty)[isNotEmpty](helpers.md#method-fluent-str-is-not-empty)[kebab](helpers.md#method-fluent-str-kebab)[length](helpers.md#method-fluent-str-length)[limit](helpers.md#method-fluent-str-limit)[lower](helpers.md#method-fluent-str-lower)[ltrim](helpers.md#method-fluent-str-ltrim)[match](helpers.md#method-fluent-str-match)[matchAll](helpers.md#method-fluent-str-match-all)[plural](helpers.md#method-fluent-str-plural)[prepend](helpers.md#method-fluent-str-prepend)[replace](helpers.md#method-fluent-str-replace)[replaceArray](helpers.md#method-fluent-str-replace-array)[replaceFirst](helpers.md#method-fluent-str-replace-first)[replaceLast](helpers.md#method-fluent-str-replace-last)[replaceMatches](helpers.md#method-fluent-str-replace-matches)[rtrim](helpers.md#method-fluent-str-rtrim)[singular](helpers.md#method-fluent-str-singular)[slug](helpers.md#method-fluent-str-slug)[snake](helpers.md#method-fluent-str-snake)[split](helpers.md#method-fluent-str-split)[start](helpers.md#method-fluent-str-start)[startsWith](helpers.md#method-fluent-str-starts-with)[studly](helpers.md#method-fluent-str-studly)[substr](helpers.md#method-fluent-str-substr)[title](helpers.md#method-fluent-str-title)[trim](helpers.md#method-fluent-str-trim)[ucfirst](helpers.md#method-fluent-str-ucfirst)[upper](helpers.md#method-fluent-str-upper)[when](helpers.md#method-fluent-str-when)[whenEmpty](helpers.md#method-fluent-str-when-empty)[words](helpers.md#method-fluent-str-words)

### URLs

[action](helpers.md#method-action)[asset](helpers.md#method-asset)[route](helpers.md#method-route)[secure\_asset](helpers.md#method-secure-asset)[secure\_url](helpers.md#method-secure-url)[url](helpers.md#method-url)

### Miscellaneous

[abort](helpers.md#method-abort)[abort\_if](helpers.md#method-abort-if)[abort\_unless](helpers.md#method-abort-unless)[app](helpers.md#method-app)[auth](helpers.md#method-auth)[back](helpers.md#method-back)[bcrypt](helpers.md#method-bcrypt)[blank](helpers.md#method-blank)[broadcast](helpers.md#method-broadcast)[cache](helpers.md#method-cache)[class\_uses\_recursive](helpers.md#method-class-uses-recursive)[collect](helpers.md#method-collect)[config](helpers.md#method-config)[cookie](helpers.md#method-cookie)[csrf\_field](helpers.md#method-csrf-field)[csrf\_token](helpers.md#method-csrf-token)[dd](helpers.md#method-dd)[dispatch](helpers.md#method-dispatch)[dispatch\_now](helpers.md#method-dispatch-now)[dump](helpers.md#method-dump)[env](helpers.md#method-env)[event](helpers.md#method-event)[factory](helpers.md#method-factory)[filled](helpers.md#method-filled)[info](helpers.md#method-info)[logger](helpers.md#method-logger)[method\_field](helpers.md#method-method-field)[now](helpers.md#method-now)[old](helpers.md#method-old)[optional](helpers.md#method-optional)[policy](helpers.md#method-policy)[redirect](helpers.md#method-redirect)[report](helpers.md#method-report)[request](helpers.md#method-request)[rescue](helpers.md#method-rescue)[resolve](helpers.md#method-resolve)[response](helpers.md#method-response)[retry](helpers.md#method-retry)[session](helpers.md#method-session)[tap](helpers.md#method-tap)[throw\_if](helpers.md#method-throw-if)[throw\_unless](helpers.md#method-throw-unless)[today](helpers.md#method-today)[trait\_uses\_recursive](helpers.md#method-trait-uses-recursive)[transform](helpers.md#method-transform)[validator](helpers.md#method-validator)[value](helpers.md#method-value)[view](helpers.md#method-view)[with](helpers.md#method-with)

## Method Listing

### Arrays & Objects <a id="arrays"></a>

#### **Arr::accessible\(\)**

The `Arr::accessible` method checks that the given value is array accessible:

```text
use Illuminate\Support\Arr;
use Illuminate\Support\Collection;

$isAccessible = Arr::accessible(['a' => 1, 'b' => 2]);

// true

$isAccessible = Arr::accessible(new Collection);

// true

$isAccessible = Arr::accessible('abc');

// false

$isAccessible = Arr::accessible(new stdClass);

// false
```

#### **Arr::add\(\)**

The `Arr::add` method adds a given key / value pair to an array if the given key doesn't already exist in the array or is set to `null`:

```text
use Illuminate\Support\Arr;

$array = Arr::add(['name' => 'Desk'], 'price', 100);

// ['name' => 'Desk', 'price' => 100]

$array = Arr::add(['name' => 'Desk', 'price' => null], 'price', 100);

// ['name' => 'Desk', 'price' => 100]
```

#### **Arr::collapse\(\)**

The `Arr::collapse` method collapses an array of arrays into a single array:

```text
use Illuminate\Support\Arr;

$array = Arr::collapse([[1, 2, 3], [4, 5, 6], [7, 8, 9]]);

// [1, 2, 3, 4, 5, 6, 7, 8, 9]
```

#### **Arr::crossJoin\(\)**

The `Arr::crossJoin` method cross joins the given arrays, returning a Cartesian product with all possible permutations:

```text
use Illuminate\Support\Arr;

$matrix = Arr::crossJoin([1, 2], ['a', 'b']);

/*
    [
        [1, 'a'],
        [1, 'b'],
        [2, 'a'],
        [2, 'b'],
    ]
*/

$matrix = Arr::crossJoin([1, 2], ['a', 'b'], ['I', 'II']);

/*
    [
        [1, 'a', 'I'],
        [1, 'a', 'II'],
        [1, 'b', 'I'],
        [1, 'b', 'II'],
        [2, 'a', 'I'],
        [2, 'a', 'II'],
        [2, 'b', 'I'],
        [2, 'b', 'II'],
    ]
*/
```

#### **Arr::divide\(\)**

The `Arr::divide` method returns two arrays, one containing the keys, and the other containing the values of the given array:

```text
use Illuminate\Support\Arr;

[$keys, $values] = Arr::divide(['name' => 'Desk']);

// $keys: ['name']

// $values: ['Desk']
```

#### **Arr::dot\(\)**

The `Arr::dot` method flattens a multi-dimensional array into a single level array that uses "dot" notation to indicate depth:

```text
use Illuminate\Support\Arr;

$array = ['products' => ['desk' => ['price' => 100]]];

$flattened = Arr::dot($array);

// ['products.desk.price' => 100]
```

#### **Arr::except\(\)**

The `Arr::except` method removes the given key / value pairs from an array:

```text
use Illuminate\Support\Arr;

$array = ['name' => 'Desk', 'price' => 100];

$filtered = Arr::except($array, ['price']);

// ['name' => 'Desk']
```

#### **Arr::exists\(\)**

The `Arr::exists` method checks that the given key exists in the provided array:

```text
use Illuminate\Support\Arr;

$array = ['name' => 'John Doe', 'age' => 17];

$exists = Arr::exists($array, 'name');

// true

$exists = Arr::exists($array, 'salary');

// false
```

#### **Arr::first\(\)**

The `Arr::first` method returns the first element of an array passing a given truth test:

```text
use Illuminate\Support\Arr;

$array = [100, 200, 300];

$first = Arr::first($array, function ($value, $key) {
    return $value >= 150;
});

// 200
```

A default value may also be passed as the third parameter to the method. This value will be returned if no value passes the truth test:

```text
use Illuminate\Support\Arr;

$first = Arr::first($array, $callback, $default);
```

#### **Arr::flatten\(\)**

The `Arr::flatten` method flattens a multi-dimensional array into a single level array:

```text
use Illuminate\Support\Arr;

$array = ['name' => 'Joe', 'languages' => ['PHP', 'Ruby']];

$flattened = Arr::flatten($array);

// ['Joe', 'PHP', 'Ruby']
```

#### **Arr::forget\(\)**

The `Arr::forget` method removes a given key / value pair from a deeply nested array using "dot" notation:

```text
use Illuminate\Support\Arr;

$array = ['products' => ['desk' => ['price' => 100]]];

Arr::forget($array, 'products.desk');

// ['products' => []]
```

#### **Arr::get\(\)**

The `Arr::get` method retrieves a value from a deeply nested array using "dot" notation:

```text
use Illuminate\Support\Arr;

$array = ['products' => ['desk' => ['price' => 100]]];

$price = Arr::get($array, 'products.desk.price');

// 100
```

The `Arr::get` method also accepts a default value, which will be returned if the specific key is not found:

```text
use Illuminate\Support\Arr;

$discount = Arr::get($array, 'products.desk.discount', 0);

// 0
```

#### **Arr::has\(\)**

The `Arr::has` method checks whether a given item or items exists in an array using "dot" notation:

```text
use Illuminate\Support\Arr;

$array = ['product' => ['name' => 'Desk', 'price' => 100]];

$contains = Arr::has($array, 'product.name');

// true

$contains = Arr::has($array, ['product.price', 'product.discount']);

// false
```

#### **Arr::hasAny\(\)**

The `Arr::hasAny` method checks whether any item in a given set exists in an array using "dot" notation:

```text
use Illuminate\Support\Arr;

$array = ['product' => ['name' => 'Desk', 'price' => 100]];

$contains = Arr::hasAny($array, 'product.name');

// true

$contains = Arr::hasAny($array, ['product.name', 'product.discount']);

// true

$contains = Arr::hasAny($array, ['category', 'product.discount']);

// false
```

#### **Arr::isAssoc\(\)**

The `Arr::isAssoc` returns `true` if the given array is an associative array. An array is considered "associative" if it doesn't have sequential numerical keys beginning with zero:

```text
use Illuminate\Support\Arr;

$isAssoc = Arr::isAssoc(['product' => ['name' => 'Desk', 'price' => 100]]);

// true

$isAssoc = Arr::isAssoc([1, 2, 3]);

// false
```

#### **Arr::last\(\)**

The `Arr::last` method returns the last element of an array passing a given truth test:

```text
use Illuminate\Support\Arr;

$array = [100, 200, 300, 110];

$last = Arr::last($array, function ($value, $key) {
    return $value >= 150;
});

// 300
```

A default value may be passed as the third argument to the method. This value will be returned if no value passes the truth test:

```text
use Illuminate\Support\Arr;

$last = Arr::last($array, $callback, $default);
```

#### **Arr::only\(\)**

The `Arr::only` method returns only the specified key / value pairs from the given array:

```text
use Illuminate\Support\Arr;

$array = ['name' => 'Desk', 'price' => 100, 'orders' => 10];

$slice = Arr::only($array, ['name', 'price']);

// ['name' => 'Desk', 'price' => 100]
```

#### **Arr::pluck\(\)**

The `Arr::pluck` method retrieves all of the values for a given key from an array:

```text
use Illuminate\Support\Arr;

$array = [
    ['developer' => ['id' => 1, 'name' => 'Taylor']],
    ['developer' => ['id' => 2, 'name' => 'Abigail']],
];

$names = Arr::pluck($array, 'developer.name');

// ['Taylor', 'Abigail']
```

You may also specify how you wish the resulting list to be keyed:

```text
use Illuminate\Support\Arr;

$names = Arr::pluck($array, 'developer.name', 'developer.id');

// [1 => 'Taylor', 2 => 'Abigail']
```

#### **Arr::prepend\(\)**

The `Arr::prepend` method will push an item onto the beginning of an array:

```text
use Illuminate\Support\Arr;

$array = ['one', 'two', 'three', 'four'];

$array = Arr::prepend($array, 'zero');

// ['zero', 'one', 'two', 'three', 'four']
```

If needed, you may specify the key that should be used for the value:

```text
use Illuminate\Support\Arr;

$array = ['price' => 100];

$array = Arr::prepend($array, 'Desk', 'name');

// ['name' => 'Desk', 'price' => 100]
```

#### **Arr::pull\(\)**

The `Arr::pull` method returns and removes a key / value pair from an array:

```text
use Illuminate\Support\Arr;

$array = ['name' => 'Desk', 'price' => 100];

$name = Arr::pull($array, 'name');

// $name: Desk

// $array: ['price' => 100]
```

A default value may be passed as the third argument to the method. This value will be returned if the key doesn't exist:

```text
use Illuminate\Support\Arr;

$value = Arr::pull($array, $key, $default);
```

**Arr::query\(\)**

The `Arr::query` method converts the array into a query string:

```text
use Illuminate\Support\Arr;

$array = ['name' => 'Taylor', 'order' => ['column' => 'created_at', 'direction' => 'desc']];

Arr::query($array);

// name=Taylor&order[column]=created_at&order[direction]=desc
```

**Arr::random\(\)**

The `Arr::random` method returns a random value from an array:

```text
use Illuminate\Support\Arr;

$array = [1, 2, 3, 4, 5];

$random = Arr::random($array);

// 4 - (retrieved randomly)
```

You may also specify the number of items to return as an optional second argument. Note that providing this argument will return an array, even if only one item is desired:

```text
use Illuminate\Support\Arr;

$items = Arr::random($array, 2);

// [2, 5] - (retrieved randomly)
```

**Arr::set\(\)**

The `Arr::set` method sets a value within a deeply nested array using "dot" notation:

```text
use Illuminate\Support\Arr;

$array = ['products' => ['desk' => ['price' => 100]]];

Arr::set($array, 'products.desk.price', 200);

// ['products' => ['desk' => ['price' => 200]]]
```

**Arr::shuffle\(\)**

The `Arr::shuffle` method randomly shuffles the items in the array:

```text
use Illuminate\Support\Arr;

$array = Arr::shuffle([1, 2, 3, 4, 5]);

// [3, 2, 5, 1, 4] - (generated randomly)
```

**Arr::sort\(\)**

The `Arr::sort` method sorts an array by its values:

```text
use Illuminate\Support\Arr;

$array = ['Desk', 'Table', 'Chair'];

$sorted = Arr::sort($array);

// ['Chair', 'Desk', 'Table']
```

You may also sort the array by the results of the given Closure:

```text
use Illuminate\Support\Arr;

$array = [
    ['name' => 'Desk'],
    ['name' => 'Table'],
    ['name' => 'Chair'],
];

$sorted = array_values(Arr::sort($array, function ($value) {
    return $value['name'];
}));

/*
    [
        ['name' => 'Chair'],
        ['name' => 'Desk'],
        ['name' => 'Table'],
    ]
*/
```

**Arr::sortRecursive\(\)**

The `Arr::sortRecursive` method recursively sorts an array using the `sort` function for numeric sub=arrays and `ksort` for associative subarrays:

```text
use Illuminate\Support\Arr;

$array = [
    ['Roman', 'Taylor', 'Li'],
    ['PHP', 'Ruby', 'JavaScript'],
    ['one' => 1, 'two' => 2, 'three' => 3],
];

$sorted = Arr::sortRecursive($array);

/*
    [
        ['JavaScript', 'PHP', 'Ruby'],
        ['one' => 1, 'three' => 3, 'two' => 2],
        ['Li', 'Roman', 'Taylor'],
    ]
*/
```

**Arr::where\(\)**

The `Arr::where` method filters an array using the given Closure:

```text
use Illuminate\Support\Arr;

$array = [100, '200', 300, '400', 500];

$filtered = Arr::where($array, function ($value, $key) {
    return is_string($value);
});

// [1 => '200', 3 => '400']
```

**Arr::wrap\(\)**

The `Arr::wrap` method wraps the given value in an array. If the given value is already an array it will not be changed:

```text
use Illuminate\Support\Arr;

$string = 'Laravel';

$array = Arr::wrap($string);

// ['Laravel']
```

If the given value is null, an empty array will be returned:

```text
use Illuminate\Support\Arr;

$nothing = null;

$array = Arr::wrap($nothing);

// []
```

**data\_fill\(\)**

The `data_fill` function sets a missing value within a nested array or object using "dot" notation:

```text
$data = ['products' => ['desk' => ['price' => 100]]];

data_fill($data, 'products.desk.price', 200);

// ['products' => ['desk' => ['price' => 100]]]

data_fill($data, 'products.desk.discount', 10);

// ['products' => ['desk' => ['price' => 100, 'discount' => 10]]]
```

This function also accepts asterisks as wildcards and will fill the target accordingly:

```text
$data = [
    'products' => [
        ['name' => 'Desk 1', 'price' => 100],
        ['name' => 'Desk 2'],
    ],
];

data_fill($data, 'products.*.price', 200);

/*
    [
        'products' => [
            ['name' => 'Desk 1', 'price' => 100],
            ['name' => 'Desk 2', 'price' => 200],
        ],
    ]
*/
```

**data\_get\(\)**

The `data_get` function retrieves a value from a nested array or object using "dot" notation:

```text
$data = ['products' => ['desk' => ['price' => 100]]];

$price = data_get($data, 'products.desk.price');

// 100
```

The `data_get` function also accepts a default value, which will be returned if the specified key is not found:

```text
$discount = data_get($data, 'products.desk.discount', 0);

// 0
```

The function also accepts wildcards using asterisks, which may target any key of the array or object:

```text
$data = [
    'product-one' => ['name' => 'Desk 1', 'price' => 100],
    'product-two' => ['name' => 'Desk 2', 'price' => 150],
];

data_get($data, '*.name');

// ['Desk 1', 'Desk 2'];
```

**data\_set\(\)**

The `data_set` function sets a value within a nested array or object using "dot" notation:

```text
$data = ['products' => ['desk' => ['price' => 100]]];

data_set($data, 'products.desk.price', 200);

// ['products' => ['desk' => ['price' => 200]]]
```

This function also accepts wildcards and will set values on the target accordingly:

```text
$data = [
    'products' => [
        ['name' => 'Desk 1', 'price' => 100],
        ['name' => 'Desk 2', 'price' => 150],
    ],
];

data_set($data, 'products.*.price', 200);

/*
    [
        'products' => [
            ['name' => 'Desk 1', 'price' => 200],
            ['name' => 'Desk 2', 'price' => 200],
        ],
    ]
*/
```

By default, any existing values are overwritten. If you wish to only set a value if it doesn't exist, you may pass `false` as the fourth argument:

```text
$data = ['products' => ['desk' => ['price' => 100]]];

data_set($data, 'products.desk.price', 200, false);

// ['products' => ['desk' => ['price' => 100]]]
```

**head\(\)**

The `head` function returns the first element in the given array:

```text
$array = [100, 200, 300];

$first = head($array);

// 100
```

**last\(\)**

The `last` function returns the last element in the given array:

```text
$array = [100, 200, 300];

$last = last($array);

// 300
```

### [Paths](helpers.md#paths) <a id="paths"></a>

**app\_path\(\)**

The `app_path` function returns the fully qualified path to the `app` directory. You may also use the `app_path` function to generate a fully qualified path to a file relative to the application directory:

```text
$path = app_path();

$path = app_path('Http/Controllers/Controller.php');
```

**base\_path\(\)**

The `base_path` function returns the fully qualified path to the project root. You may also use the `base_path` function to generate a fully qualified path to a given file relative to the project root directory:

```text
$path = base_path();

$path = base_path('vendor/bin');
```

**config\_path\(\)**

The `config_path` function returns the fully qualified path to the `config` directory. You may also use the `config_path` function to generate a fully qualified path to a given file within the application's configuration directory:

```text
$path = config_path();

$path = config_path('app.php');
```

**database\_path\(\)**

The `database_path` function returns the fully qualified path to the `database` directory. You may also use the `database_path` function to generate a fully qualified path to a given file within the database directory:

```text
$path = database_path();

$path = database_path('factories/UserFactory.php');
```

**mix\(\)**

The `mix` function returns the path to a [versioned Mix file](https://laravel.com/docs/7.x/mix):

```text
$path = mix('css/app.css');
```

**public\_path\(\)**

The `public_path` function returns the fully qualified path to the `public` directory. You may also use the `public_path` function to generate a fully qualified path to a given file within the public directory:

```text
$path = public_path();

$path = public_path('css/app.css');
```

**resource\_path\(\)**

The `resource_path` function returns the fully qualified path to the `resources` directory. You may also use the `resource_path` function to generate a fully qualified path to a given file within the resources directory:

```text
$path = resource_path();

$path = resource_path('sass/app.scss');
```

**storage\_path\(\)**

The `storage_path` function returns the fully qualified path to the `storage` directory. You may also use the `storage_path` function to generate a fully qualified path to a given file within the storage directory:

```text
$path = storage_path();

$path = storage_path('app/file.txt');
```

### [Strings](helpers.md#strings) <a id="strings"></a>

**\_\_\(\)**

The `__` function translates the given translation string or translation key using your [localization files](https://laravel.com/docs/7.x/localization):

```text
echo __('Welcome to our application');

echo __('messages.welcome');
```

If the specified translation string or key does not exist, the `__` function will return the given value. So, using the example above, the `__` function would return `messages.welcome` if that translation key does not exist.

**class\_basename\(\)**

The `class_basename` function returns the class name of the given class with the class's namespace removed:

```text
$class = class_basename('Foo\Bar\Baz');

// Baz
```

**e\(\)**

The `e` function runs PHP's `htmlspecialchars` function with the `double_encode` option set to `true` by default:

```text
echo e('<html>foo</html>');

// &lt;html&gt;foo&lt;/html&gt;
```

**preg\_replace\_array\(\)**

The `preg_replace_array` function replaces a given pattern in the string sequentially using an array:

```text
$string = 'The event will take place between :start and :end';

$replaced = preg_replace_array('/:[a-z_]+/', ['8:30', '9:00'], $string);

// The event will take place between 8:30 and 9:00
```

**Str::after\(\)**

The `Str::after` method returns everything after the given value in a string. The entire string will be returned if the value does not exist within the string:

```text
use Illuminate\Support\Str;

$slice = Str::after('This is my name', 'This is');

// ' my name'
```

**Str::afterLast\(\)**

The `Str::afterLast` method returns everything after the last occurrence of the given value in a string. The entire string will be returned if the value does not exist within the string:

```text
use Illuminate\Support\Str;

$slice = Str::afterLast('App\Http\Controllers\Controller', '\\');

// 'Controller'
```

**Str::ascii\(\)**

The `Str::ascii` method will attempt to transliterate the string into an ASCII value:

```text
use Illuminate\Support\Str;

$slice = Str::ascii('û');

// 'u'
```

**Str::before\(\)**

The `Str::before` method returns everything before the given value in a string:

```text
use Illuminate\Support\Str;

$slice = Str::before('This is my name', 'my name');

// 'This is '
```

**Str::beforeLast\(\)**

The `Str::beforeLast` method returns everything before the last occurrence of the given value in a string:

```text
use Illuminate\Support\Str;

$slice = Str::beforeLast('This is my name', 'is');

// 'This '
```

**Str::between\(\)**

The `Str::between` method returns the portion of a string between two values:

```text
use Illuminate\Support\Str;

$slice = Str::between('This is my name', 'This', 'name');

// ' is my '
```

**Str::camel\(\)**

The `Str::camel` method converts the given string to `camelCase`:

```text
use Illuminate\Support\Str;

$converted = Str::camel('foo_bar');

// fooBar
```

**Str::contains\(\)**

The `Str::contains` method determines if the given string contains the given value \(case sensitive\):

```text
use Illuminate\Support\Str;

$contains = Str::contains('This is my name', 'my');

// true
```

You may also pass an array of values to determine if the given string contains any of the values:

```text
use Illuminate\Support\Str;

$contains = Str::contains('This is my name', ['my', 'foo']);

// true
```

**Str::containsAll\(\)**

The `Str::containsAll` method determines if the given string contains all array values:

```text
use Illuminate\Support\Str;

$containsAll = Str::containsAll('This is my name', ['my', 'name']);

// true
```

**Str::endsWith\(\)**

The `Str::endsWith` method determines if the given string ends with the given value:

```text
use Illuminate\Support\Str;

$result = Str::endsWith('This is my name', 'name');

// true
```

You may also pass an array of values to determine if the given string ends with any of the given values:

```text
use Illuminate\Support\Str;

$result = Str::endsWith('This is my name', ['name', 'foo']);

// true

$result = Str::endsWith('This is my name', ['this', 'foo']);

// false
```

**Str::finish\(\)**

The `Str::finish` method adds a single instance of the given value to a string if it does not already end with the value:

```text
use Illuminate\Support\Str;

$adjusted = Str::finish('this/string', '/');

// this/string/

$adjusted = Str::finish('this/string/', '/');

// this/string/
```

**Str::is\(\)**

The `Str::is` method determines if a given string matches a given pattern. Asterisks may be used to indicate wildcards:

```text
use Illuminate\Support\Str;

$matches = Str::is('foo*', 'foobar');

// true

$matches = Str::is('baz*', 'foobar');

// false
```

**Str::isAscii\(\)**

The `Str::isAscii` method determines if a given string is 7 bit ASCII:

```text
use Illuminate\Support\Str;

$isAscii = Str::isAscii('Taylor');

// true

$isAscii = Str::isAscii('ü');

// false
```

**Str::isUuid\(\)**

The `Str::isUuid` method determines if the given string is a valid UUID:

```text
use Illuminate\Support\Str;

$isUuid = Str::isUuid('a0a2a2d2-0b87-4a18-83f2-2529882be2de');

// true

$isUuid = Str::isUuid('laravel');

// false
```

**Str::kebab\(\)**

The `Str::kebab` method converts the given string to `kebab-case`:

```text
use Illuminate\Support\Str;

$converted = Str::kebab('fooBar');

// foo-bar
```

**Str::length\(\)**

The `Str::length` method returns the length of the given string:

```text
use Illuminate\Support\Str;

$length = Str::length('Laravel');

// 7
```

**Str::limit\(\)**

The `Str::limit` method truncates the given string at the specified length:

```text
use Illuminate\Support\Str;

$truncated = Str::limit('The quick brown fox jumps over the lazy dog', 20);

// The quick brown fox...
```

You may also pass a third argument to change the string that will be appended to the end:

```text
use Illuminate\Support\Str;

$truncated = Str::limit('The quick brown fox jumps over the lazy dog', 20, ' (...)');

// The quick brown fox (...)
```

**Str::lower\(\)**

The `Str::lower` method converts the given string to lowercase:

```text
use Illuminate\Support\Str;

$converted = Str::lower('LARAVEL');

// laravel
```

**Str::orderedUuid\(\)**

The `Str::orderedUuid` method generates a "timestamp first" UUID that may be efficiently stored in an indexed database column:

```text
use Illuminate\Support\Str;

return (string) Str::orderedUuid();
```

**Str::plural\(\)**

The `Str::plural` method converts a single word string to its plural form. This function currently only supports the English language:

```text
use Illuminate\Support\Str;

$plural = Str::plural('car');

// cars

$plural = Str::plural('child');

// children
```

You may provide an integer as a second argument to the function to retrieve the singular or plural form of the string:

```text
use Illuminate\Support\Str;

$plural = Str::plural('child', 2);

// children

$plural = Str::plural('child', 1);

// child
```

**Str::random\(\)**

The `Str::random` method generates a random string of the specified length. This function uses PHP's `random_bytes` function:

```text
use Illuminate\Support\Str;

$random = Str::random(40);
```

**Str::replaceArray\(\)**

The `Str::replaceArray` method replaces a given value in the string sequentially using an array:

```text
use Illuminate\Support\Str;

$string = 'The event will take place between ? and ?';

$replaced = Str::replaceArray('?', ['8:30', '9:00'], $string);

// The event will take place between 8:30 and 9:00
```

**Str::replaceFirst\(\)**

The `Str::replaceFirst` method replaces the first occurrence of a given value in a string:

```text
use Illuminate\Support\Str;

$replaced = Str::replaceFirst('the', 'a', 'the quick brown fox jumps over the lazy dog');

// a quick brown fox jumps over the lazy dog
```

**Str::replaceLast\(\)**

The `Str::replaceLast` method replaces the last occurrence of a given value in a string:

```text
use Illuminate\Support\Str;

$replaced = Str::replaceLast('the', 'a', 'the quick brown fox jumps over the lazy dog');

// the quick brown fox jumps over a lazy dog
```

**Str::singular\(\)**

The `Str::singular` method converts a string to its singular form. This function currently only supports the English language:

```text
use Illuminate\Support\Str;

$singular = Str::singular('cars');

// car

$singular = Str::singular('children');

// child
```

**Str::slug\(\)**

The `Str::slug` method generates a URL friendly "slug" from the given string:

```text
use Illuminate\Support\Str;

$slug = Str::slug('Laravel 5 Framework', '-');

// laravel-5-framework
```

**Str::snake\(\)**

The `Str::snake` method converts the given string to `snake_case`:

```text
use Illuminate\Support\Str;

$converted = Str::snake('fooBar');

// foo_bar
```

**Str::start\(\)**

The `Str::start` method adds a single instance of the given value to a string if it does not already start with the value:

```text
use Illuminate\Support\Str;

$adjusted = Str::start('this/string', '/');

// /this/string

$adjusted = Str::start('/this/string', '/');

// /this/string
```

**Str::startsWith\(\)**

The `Str::startsWith` method determines if the given string begins with the given value:

```text
use Illuminate\Support\Str;

$result = Str::startsWith('This is my name', 'This');

// true
```

**Str::studly\(\)**

The `Str::studly` method converts the given string to `StudlyCase`:

```text
use Illuminate\Support\Str;

$converted = Str::studly('foo_bar');

// FooBar
```

**Str::substr\(\)**

The `Str::substr` method returns the portion of string specified by the start and length parameters:

```text
use Illuminate\Support\Str;

$converted = Str::substr('The Laravel Framework', 4, 7);

// Laravel
```

**Str::title\(\)**

The `Str::title` method converts the given string to `Title Case`:

```text
use Illuminate\Support\Str;

$converted = Str::title('a nice title uses the correct case');

// A Nice Title Uses The Correct Case
```

**Str::ucfirst\(\)**

The `Str::ucfirst` method returns the given string with the first character capitalized:

```text
use Illuminate\Support\Str;

$string = Str::ucfirst('foo bar');

// Foo bar
```

**Str::upper\(\)**

The `Str::upper` method converts the given string to uppercase:

```text
use Illuminate\Support\Str;

$string = Str::upper('laravel');

// LARAVEL
```

**Str::uuid\(\)**

The `Str::uuid` method generates a UUID \(version 4\):

```text
use Illuminate\Support\Str;

return (string) Str::uuid();
```

**Str::words\(\)**

The `Str::words` method limits the number of words in a string:

```text
use Illuminate\Support\Str;

return Str::words('Perfectly balanced, as all things should be.', 3, ' >>>');

// Perfectly balanced, as >>>
```

**trans\(\)**

The `trans` function translates the given translation key using your [localization files](https://laravel.com/docs/7.x/localization):

```text
echo trans('messages.welcome');
```

If the specified translation key does not exist, the `trans` function will return the given key. So, using the example above, the `trans` function would return `messages.welcome` if the translation key does not exist.

**trans\_choice\(\)**

The `trans_choice` function translates the given translation key with inflection:

```text
echo trans_choice('messages.notifications', $unreadCount);
```

If the specified translation key does not exist, the `trans_choice` function will return the given key. So, using the example above, the `trans_choice` function would return `messages.notifications` if the translation key does not exist.

### [Fluent Strings](helpers.md#fluent-strings) <a id="fluent-strings"></a>

Fluent strings provide a more fluent, object-oriented interface for working with string values, allowing you to chain multiple string operations together using a more readable syntax compared to traditional string operations.

**after**

The `after` method returns everything after the given value in a string. The entire string will be returned if the value does not exist within the string:

```text
use Illuminate\Support\Str;

$slice = Str::of('This is my name')->after('This is');

// ' my name'
```

**afterLast**

The `afterLast` method returns everything after the last occurrence of the given value in a string. The entire string will be returned if the value does not exist within the string:

```text
use Illuminate\Support\Str;

$slice = Str::of('App\Http\Controllers\Controller')->afterLast('\\');

// 'Controller'
```

**append**

The `append` method appends the given values to the string:

```text
use Illuminate\Support\Str;

$string = Str::of('Taylor')->append(' Otwell');

// 'Taylor Otwell'
```

**ascii**

The `ascii` method will attempt to transliterate the string into an ASCII value:

```text
use Illuminate\Support\Str;

$string = Str::of('ü')->ascii();

// 'u'
```

**basename**

The `basename` method will return the trailing name component of the given string:

```text
use Illuminate\Support\Str;

$string = Str::of('/foo/bar/baz')->basename();

// 'baz'
```

If needed, you may provide an "extension" that will be removed from the trailing component:

```text
use Illuminate\Support\Str;

$string = Str::of('/foo/bar/baz.jpg')->basename('.jpg');

// 'baz'
```

**before**

The `before` method returns everything before the given value in a string:

```text
use Illuminate\Support\Str;

$slice = Str::of('This is my name')->before('my name');

// 'This is '
```

**beforeLast**

The `beforeLast` method returns everything before the last occurrence of the given value in a string:

```text
use Illuminate\Support\Str;

$slice = Str::of('This is my name')->beforeLast('is');

// 'This '
```

**camel**

The `camel` method converts the given string to `camelCase`:

```text
use Illuminate\Support\Str;

$converted = Str::of('foo_bar')->camel();

// fooBar
```

**contains**

The `contains` method determines if the given string contains the given value \(case sensitive\):

```text
use Illuminate\Support\Str;

$contains = Str::of('This is my name')->contains('my');

// true
```

You may also pass an array of values to determine if the given string contains any of the values:

```text
use Illuminate\Support\Str;

$contains = Str::of('This is my name')->contains(['my', 'foo']);

// true
```

**containsAll**

The `containsAll` method determines if the given string contains all array values:

```text
use Illuminate\Support\Str;

$containsAll = Str::of('This is my name')->containsAll(['my', 'name']);

// true
```

**dirname**

The `dirname` method returns the parent directory portion of the given string:

```text
use Illuminate\Support\Str;

$string = Str::of('/foo/bar/baz')->dirname();

// '/foo/bar'
```

Optionally, You may specify how many directory levels you wish to trim from the string:

```text
use Illuminate\Support\Str;

$string = Str::of('/foo/bar/baz')->dirname(2);

// '/foo'
```

**endsWith**

The `endsWith` method determines if the given string ends with the given value:

```text
use Illuminate\Support\Str;

$result = Str::of('This is my name')->endsWith('name');

// true
```

You may also pass an array of values to determine if the given string ends with any of the given values:

```text
use Illuminate\Support\Str;

$result = Str::of('This is my name')->endsWith(['name', 'foo']);

// true

$result = Str::of('This is my name')->endsWith(['this', 'foo']);

// false
```

**exactly**

The `exactly` method determines if the given string is an exact match with another string:

```text
use Illuminate\Support\Str;

$result = Str::of('Laravel')->exactly('Laravel');

// true
```

**explode**

The `explode` method splits the string by the given delimiter and returns a collection containing each section of the split string:

```text
use Illuminate\Support\Str;

$collection = Str::of('foo bar baz')->explode(' ');

// collect(['foo', 'bar', 'baz'])
```

**finish**

The `finish` method adds a single instance of the given value to a string if it does not already end with the value:

```text
use Illuminate\Support\Str;

$adjusted = Str::of('this/string')->finish('/');

// this/string/

$adjusted = Str::of('this/string/')->finish('/');

// this/string/
```

**is**

The `is` method determines if a given string matches a given pattern. Asterisks may be used to indicate wildcards:

```text
use Illuminate\Support\Str;

$matches = Str::of('foobar')->is('foo*');

// true

$matches = Str::of('foobar')->is('baz*');

// false
```

**isAscii**

The `isAscii` method determines if a given string is an ASCII string:

```text
use Illuminate\Support\Str;

$result = Str::of('Taylor')->isAscii();

// true

$result = Str::of('ü')->isAcii();

// false
```

**isEmpty**

The `isEmpty` method determines if the given string is empty:

```text
use Illuminate\Support\Str;

$result = Str::of('  ')->trim()->isEmpty();

// true

$result = Str::of('Laravel')->trim()->isEmpty();

// false
```

**isNotEmpty**

The `isNotEmpty` method determines if the given string is not empty:

```text
use Illuminate\Support\Str;

$result = Str::of('  ')->trim()->isNotEmpty();

// false

$result = Str::of('Laravel')->trim()->isNotEmpty();

// true
```

**kebab**

The `kebab` method converts the given string to `kebab-case`:

```text
use Illuminate\Support\Str;

$converted = Str::of('fooBar')->kebab();

// foo-bar
```

**length**

The `length` method returns the length of the given string:

```text
use Illuminate\Support\Str;

$length = Str::of('Laravel')->length();

// 7
```

**limit**

The `limit` method truncates the given string at the specified length:

```text
use Illuminate\Support\Str;

$truncated = Str::of('The quick brown fox jumps over the lazy dog')->limit(20);

// The quick brown fox...
```

You may also pass a third argument to change the string that will be appended to the end:

```text
use Illuminate\Support\Str;

$truncated = Str::of('The quick brown fox jumps over the lazy dog')->limit(20, ' (...)');

// The quick brown fox (...)
```

**lower**

The `lower` method converts the given string to lowercase:

```text
use Illuminate\Support\Str;

$result = Str::of('LARAVEL')->lower();

// 'laravel'
```

**ltrim**

The `ltrim` method left trims the given string:

```text
use Illuminate\Support\Str;

$string = Str::of('  Laravel  ')->ltrim();

// 'Laravel  '

$string = Str::of('/Laravel/')->ltrim('/');

// 'Laravel/'
```

**match**

The `match` method will return the portion of a string that matches a given regular expression pattern:

```text
use Illuminate\Support\Str;

$result = Str::of('foo bar')->match('/bar/');

// 'bar'

$result = Str::of('foo bar')->match('/foo (.*)/');

// 'bar'
```

**matchAll**

The `matchAll` method will return a collection containing the portions of a string that match a given regular expression pattern:

```text
use Illuminate\Support\Str;

$result = Str::of('bar foo bar')->matchAll('/bar/');

// collect(['bar', 'bar'])
```

If you specify a matching group within the expression, Laravel will return a collection of that group's matches:

```text
use Illuminate\Support\Str;

$result = Str::of('bar fun bar fly')->matchAll('/f(\w*)/');

// collect(['un', 'ly']);
```

If no matches are found, an empty collection will be returned.

**plural**

The `plural` method converts a single word string to its plural form. This function currently only supports the English language:

```text
use Illuminate\Support\Str;

$plural = Str::of('car')->plural();

// cars

$plural = Str::of('child')->plural();

// children
```

You may provide an integer as a second argument to the function to retrieve the singular or plural form of the string:

```text
use Illuminate\Support\Str;

$plural = Str::of('child')->plural(2);

// children

$plural = Str::of('child')->plural(1);

// child
```

**prepend**

The `prepend` method prepends the given values onto the string:

```text
use Illuminate\Support\Str;

$string = Str::of('Framework')->prepend('Laravel ');

// Laravel Framework
```

**replace**

The `replace` method replaces a given string within the string:

```text
use Illuminate\Support\Str;

$replaced = Str::of('Laravel 6.x')->replace('6.x', '7.x');

// Laravel 7.x
```

**replaceArray**

The `replaceArray` method replaces a given value in the string sequentially using an array:

```text
use Illuminate\Support\Str;

$string = 'The event will take place between ? and ?';

$replaced = Str::of($string)->replaceArray('?', ['8:30', '9:00']);

// The event will take place between 8:30 and 9:00
```

**replaceFirst**

The `replaceFirst` method replaces the first occurrence of a given value in a string:

```text
use Illuminate\Support\Str;

$replaced = Str::of('the quick brown fox jumps over the lazy dog')->replaceFirst('the', 'a');

// a quick brown fox jumps over the lazy dog
```

**replaceLast**

The `replaceLast` method replaces the last occurrence of a given value in a string:

```text
use Illuminate\Support\Str;

$replaced = Str::of('the quick brown fox jumps over the lazy dog')->replaceLast('the', 'a');

// the quick brown fox jumps over a lazy dog
```

**replaceMatches**

The `replaceMatches` method replaces all portions of a string matching a given pattern with the given replacement string:

```text
use Illuminate\Support\Str;

$replaced = Str::of('(+1) 501-555-1000')->replaceMatches('/[^A-Za-z0-9]++/', '')

// '15015551000'
```

The `replaceMatches` method also accepts a Closure that will be invoked with each portion of the string matching the given party, allowing you to perform the replacement logic within the Closure and return the replaced value:

```text
use Illuminate\Support\Str;

$replaced = Str::of('123')->replaceMatches('/\d/', function ($match) {
    return '['.$match[0].']';
});

// '[1][2][3]'
```

**rtrim**

The `rtrim` method right trims the given string:

```text
use Illuminate\Support\Str;

$string = Str::of('  Laravel  ')->rtrim();

// '  Laravel'

$string = Str::of('/Laravel/')->rtrim('/');

// '/Laravel'
```

**singular**

The `singular` method converts a string to its singular form. This function currently only supports the English language:

```text
use Illuminate\Support\Str;

$singular = Str::of('cars')->singular();

// car

$singular = Str::of('children')->singular();

// child
```

**slug**

The `slug` method generates a URL friendly "slug" from the given string:

```text
use Illuminate\Support\Str;

$slug = Str::of('Laravel Framework')->slug('-');

// laravel-framework
```

**snake**

The `snake` method converts the given string to `snake_case`:

```text
use Illuminate\Support\Str;

$converted = Str::of('fooBar')->snake();

// foo_bar
```

**split**

The `split` method splits a string into a collection using a regular expression:

```text
use Illuminate\Support\Str;

$segments = Str::of('one, two, three')->split('/[\s,]+/');

// collect(["one", "two", "three"])
```

**start**

The `start` method adds a single instance of the given value to a string if it does not already start with the value:

```text
use Illuminate\Support\Str;

$adjusted = Str::of('this/string')->start('/');

// /this/string

$adjusted = Str::of('/this/string')->start('/');

// /this/string
```

**startsWith**

The `startsWith` method determines if the given string begins with the given value:

```text
use Illuminate\Support\Str;

$result = Str::of('This is my name')->startsWith('This');

// true
```

**studly**

The `studly` method converts the given string to `StudlyCase`:

```text
use Illuminate\Support\Str;

$converted = Str::of('foo_bar')->studly();

// FooBar
```

**substr**

The `substr` method returns the portion of the string specified by the given start and length parameters:

```text
use Illuminate\Support\Str;

$string = Str::of('Laravel Framework')->substr(8);

// Framework

$string = Str::of('Laravel Framework')->substr(8, 5);

// Frame
```

**title**

The `title` method converts the given string to `Title Case`:

```text
use Illuminate\Support\Str;

$converted = Str::of('a nice title uses the correct case')->title();

// A Nice Title Uses The Correct Case
```

**trim**

The `trim` method trims the given string:

```text
use Illuminate\Support\Str;

$string = Str::of('  Laravel  ')->trim();

// 'Laravel'

$string = Str::of('/Laravel/')->trim('/');

// 'Laravel'
```

**ucfirst**

The `ucfirst` method returns the given string with the first character capitalized:

```text
use Illuminate\Support\Str;

$string = Str::of('foo bar')->ucfirst();

// Foo bar
```

**upper**

The `upper` method converts the given string to uppercase:

```text
use Illuminate\Support\Str;

$adjusted = Str::of('laravel')->upper();

// LARAVEL
```

**when**

The `when` method invokes the given Closure if a given condition is true. The Closure will receive the fluent string instance:

```text
use Illuminate\Support\Str;

$string = Str::of('Taylor')
                ->when(true, function ($string) {
                    return $string->append(' Otwell');
                });

// 'Taylor Otwell'
```

If necessary, you may pass another Closure as the third parameter to the `when` method. This Closure will execute if the condition parameter evaluates to `false`.

**whenEmpty**

The `whenEmpty` method invokes the given Closure if the string is empty. If the Closure returns a value, that value will also be returned by the `whenEmpty` method. If the Closure does not return a value, the fluent string instance will be returned:

```text
use Illuminate\Support\Str;

$string = Str::of('  ')->whenEmpty(function ($string) {
    return $string->trim()->prepend('Laravel');
});

// 'Laravel'
```

**words**

The `words` method limits the number of words in a string:

```text
use Illuminate\Support\Str;

$string = Str::of('Perfectly balanced, as all things should be.')->words(3, ' >>>');

// Perfectly balanced, as >>>
```

### [URLs](helpers.md#urls) <a id="urls"></a>

**action\(\)**

The `action` function generates a URL for the given controller action. You do not need to pass the full namespace of the controller. Instead, pass the controller class name relative to the `App\Http\Controllers` namespace:

```text
$url = action('HomeController@index');

$url = action([HomeController::class, 'index']);
```

If the method accepts route parameters, you may pass them as the second argument to the method:

```text
$url = action('UserController@profile', ['id' => 1]);
```

**asset\(\)**

The `asset` function generates a URL for an asset using the current scheme of the request \(HTTP or HTTPS\):

```text
$url = asset('img/photo.jpg');
```

You can configure the asset URL host by setting the `ASSET_URL` variable in your `.env` file. This can be useful if you host your assets on an external service like Amazon S3:

```text
// ASSET_URL=http://example.com/assets

$url = asset('img/photo.jpg'); // http://example.com/assets/img/photo.jpg
```

**route\(\)**

The `route` function generates a URL for the given named route:

```text
$url = route('routeName');
```

If the route accepts parameters, you may pass them as the second argument to the method:

```text
$url = route('routeName', ['id' => 1]);
```

By default, the `route` function generates an absolute URL. If you wish to generate a relative URL, you may pass `false` as the third argument:

```text
$url = route('routeName', ['id' => 1], false);
```

**secure\_asset\(\)**

The `secure_asset` function generates a URL for an asset using HTTPS:

```text
$url = secure_asset('img/photo.jpg');
```

**secure\_url\(\)**

The `secure_url` function generates a fully qualified HTTPS URL to the given path:

```text
$url = secure_url('user/profile');

$url = secure_url('user/profile', [1]);
```

**url\(\)**

The `url` function generates a fully qualified URL to the given path:

```text
$url = url('user/profile');

$url = url('user/profile', [1]);
```

If no path is provided, a `Illuminate\Routing\UrlGenerator` instance is returned:

```text
$current = url()->current();

$full = url()->full();

$previous = url()->previous();
```

### [Miscellaneous](helpers.md#miscellaneous) <a id="miscellaneous"></a>

**abort\(\)**

The `abort` function throws [an HTTP exception](https://laravel.com/docs/7.x/errors#http-exceptions) which will be rendered by the [exception handler](https://laravel.com/docs/7.x/errors#the-exception-handler):

```text
abort(403);
```

You may also provide the exception's response text and custom response headers:

```text
abort(403, 'Unauthorized.', $headers);
```

**abort\_if\(\)**

The `abort_if` function throws an HTTP exception if a given boolean expression evaluates to `true`:

```text
abort_if(! Auth::user()->isAdmin(), 403);
```

Like the `abort` method, you may also provide the exception's response text as the third argument and an array of custom response headers as the fourth argument.

**abort\_unless\(\)**

The `abort_unless` function throws an HTTP exception if a given boolean expression evaluates to `false`:

```text
abort_unless(Auth::user()->isAdmin(), 403);
```

Like the `abort` method, you may also provide the exception's response text as the third argument and an array of custom response headers as the fourth argument.

**app\(\)**

The `app` function returns the [service container](https://laravel.com/docs/7.x/container) instance:

```text
$container = app();
```

You may pass a class or interface name to resolve it from the container:

```text
$api = app('HelpSpot\API');
```

**auth\(\)**

The `auth` function returns an [authenticator](https://laravel.com/docs/7.x/authentication) instance. You may use it instead of the `Auth` facade for convenience:

```text
$user = auth()->user();
```

If needed, you may specify which guard instance you would like to access:

```text
$user = auth('admin')->user();
```

**back\(\)**

The `back` function generates a [redirect HTTP response](https://laravel.com/docs/7.x/responses#redirects) to the user's previous location:

```text
return back($status = 302, $headers = [], $fallback = false);

return back();
```

**bcrypt\(\)**

The `bcrypt` function [hashes](https://laravel.com/docs/7.x/hashing) the given value using Bcrypt. You may use it as an alternative to the `Hash` facade:

```text
$password = bcrypt('my-secret-password');
```

**blank\(\)**

The `blank` function returns whether the given value is "blank":

```text
blank('');
blank('   ');
blank(null);
blank(collect());

// true

blank(0);
blank(true);
blank(false);

// false
```

For the inverse of `blank`, see the [`filled`](helpers.md#method-filled) method.

**broadcast\(\)**

The `broadcast` function [broadcasts](https://laravel.com/docs/7.x/broadcasting) the given [event](https://laravel.com/docs/7.x/events) to its listeners:

```text
broadcast(new UserRegistered($user));
```

**cache\(\)**

The `cache` function may be used to get values from the [cache](https://laravel.com/docs/7.x/cache). If the given key does not exist in the cache, an optional default value will be returned:

```text
$value = cache('key');

$value = cache('key', 'default');
```

You may add items to the cache by passing an array of key / value pairs to the function. You should also pass the number of seconds or duration the cached value should be considered valid:

```text
cache(['key' => 'value'], 300);

cache(['key' => 'value'], now()->addSeconds(10));
```

**class\_uses\_recursive\(\)**

The `class_uses_recursive` function returns all traits used by a class, including traits used by all of its parent classes:

```text
$traits = class_uses_recursive(App\User::class);
```

**collect\(\)**

The `collect` function creates a [collection](https://laravel.com/docs/7.x/collections) instance from the given value:

```text
$collection = collect(['taylor', 'abigail']);
```

**config\(\)**

The `config` function gets the value of a [configuration](https://laravel.com/docs/7.x/configuration) variable. The configuration values may be accessed using "dot" syntax, which includes the name of the file and the option you wish to access. A default value may be specified and is returned if the configuration option does not exist:

```text
$value = config('app.timezone');

$value = config('app.timezone', $default);
```

You may set configuration variables at runtime by passing an array of key / value pairs:

```text
config(['app.debug' => true]);
```

**cookie\(\)**

The `cookie` function creates a new [cookie](https://laravel.com/docs/7.x/requests#cookies) instance:

```text
$cookie = cookie('name', 'value', $minutes);
```

**csrf\_field\(\)**

The `csrf_field` function generates an HTML `hidden` input field containing the value of the CSRF token. For example, using [Blade syntax](https://laravel.com/docs/7.x/blade):

```text
{{ csrf_field() }}
```

**csrf\_token\(\)**

The `csrf_token` function retrieves the value of the current CSRF token:

```text
$token = csrf_token();
```

**dd\(\)**

The `dd` function dumps the given variables and ends execution of the script:

```text
dd($value);

dd($value1, $value2, $value3, ...);
```

If you do not want to halt the execution of your script, use the [`dump`](helpers.md#method-dump) function instead.

**dispatch\(\)**

The `dispatch` function pushes the given [job](https://laravel.com/docs/7.x/queues#creating-jobs) onto the Laravel [job queue](https://laravel.com/docs/7.x/queues):

```text
dispatch(new App\Jobs\SendEmails);
```

**dispatch\_now\(\)**

The `dispatch_now` function runs the given [job](https://laravel.com/docs/7.x/queues#creating-jobs) immediately and returns the value from its `handle` method:

```text
$result = dispatch_now(new App\Jobs\SendEmails);
```

**dump\(\)**

The `dump` function dumps the given variables:

```text
dump($value);

dump($value1, $value2, $value3, ...);
```

If you want to stop executing the script after dumping the variables, use the [`dd`](helpers.md#method-dd) function instead.

**env\(\)**

The `env` function retrieves the value of an [environment variable](https://laravel.com/docs/7.x/configuration#environment-configuration) or returns a default value:

```text
$env = env('APP_ENV');

// Returns 'production' if APP_ENV is not set...
$env = env('APP_ENV', 'production');
```

> ![](https://laravel.com/img/callouts/exclamation.min.svg)
>
> If you execute the `config:cache` command during your deployment process, you should be sure that you are only calling the `env` function from within your configuration files. Once the configuration has been cached, the `.env` file will not be loaded and all calls to the `env` function will return `null`.

**event\(\)**

The `event` function dispatches the given [event](https://laravel.com/docs/7.x/events) to its listeners:

```text
event(new UserRegistered($user));
```

**factory\(\)**

The `factory` function creates a model factory builder for a given class, name, and amount. It can be used while [testing](https://laravel.com/docs/7.x/database-testing#writing-factories) or [seeding](https://laravel.com/docs/7.x/seeding#using-model-factories):

```text
$user = factory(App\User::class)->make();
```

**filled\(\)**

The `filled` function returns whether the given value is not "blank":

```text
filled(0);
filled(true);
filled(false);

// true

filled('');
filled('   ');
filled(null);
filled(collect());

// false
```

For the inverse of `filled`, see the [`blank`](helpers.md#method-blank) method.

**info\(\)**

The `info` function will write information to the [log](https://laravel.com/docs/7.x/logging):

```text
info('Some helpful information!');
```

An array of contextual data may also be passed to the function:

```text
info('User login attempt failed.', ['id' => $user->id]);
```

**logger\(\)**

The `logger` function can be used to write a `debug` level message to the [log](https://laravel.com/docs/7.x/logging):

```text
logger('Debug message');
```

An array of contextual data may also be passed to the function:

```text
logger('User has logged in.', ['id' => $user->id]);
```

A [logger](https://laravel.com/docs/7.x/errors#logging) instance will be returned if no value is passed to the function:

```text
logger()->error('You are not allowed here.');
```

**method\_field\(\)**

The `method_field` function generates an HTML `hidden` input field containing the spoofed value of the form's HTTP verb. For example, using [Blade syntax](https://laravel.com/docs/7.x/blade):

```text
<form method="POST">
    {{ method_field('DELETE') }}
</form>
```

**now\(\)**

The `now` function creates a new `Illuminate\Support\Carbon` instance for the current time:

```text
$now = now();
```

**old\(\)**

The `old` function [retrieves](https://laravel.com/docs/7.x/requests#retrieving-input) an [old input](https://laravel.com/docs/7.x/requests#old-input) value flashed into the session:

```text
$value = old('value');

$value = old('value', 'default');
```

**optional\(\)**

The `optional` function accepts any argument and allows you to access properties or call methods on that object. If the given object is `null`, properties and methods will return `null` instead of causing an error:

```text
return optional($user->address)->street;

{!! old('name', optional($user)->name) !!}
```

The `optional` function also accepts a Closure as its second argument. The Closure will be invoked if the value provided as the first argument is not null:

```text
return optional(User::find($id), function ($user) {
    return new DummyUser;
});
```

**policy\(\)**

The `policy` method retrieves a [policy](https://laravel.com/docs/7.x/authorization#creating-policies) instance for a given class:

```text
$policy = policy(App\User::class);
```

**redirect\(\)**

The `redirect` function returns a [redirect HTTP response](https://laravel.com/docs/7.x/responses#redirects), or returns the redirector instance if called with no arguments:

```text
return redirect($to = null, $status = 302, $headers = [], $secure = null);

return redirect('/home');

return redirect()->route('route.name');
```

**report\(\)**

The `report` function will report an exception using your [exception handler](https://laravel.com/docs/7.x/errors#the-exception-handler)'s `report` method:

```text
report($e);
```

**request\(\)**

The `request` function returns the current [request](https://laravel.com/docs/7.x/requests) instance or obtains an input item:

```text
$request = request();

$value = request('key', $default);
```

**rescue\(\)**

The `rescue` function executes the given Closure and catches any exceptions that occur during its execution. All exceptions that are caught will be sent to your [exception handler](https://laravel.com/docs/7.x/errors#the-exception-handler)'s `report` method; however, the request will continue processing:

```text
return rescue(function () {
    return $this->method();
});
```

You may also pass a second argument to the `rescue` function. This argument will be the "default" value that should be returned if an exception occurs while executing the Closure:

```text
return rescue(function () {
    return $this->method();
}, false);

return rescue(function () {
    return $this->method();
}, function () {
    return $this->failure();
});
```

**resolve\(\)**

The `resolve` function resolves a given class or interface name to its instance using the [service container](https://laravel.com/docs/7.x/container):

```text
$api = resolve('HelpSpot\API');
```

**response\(\)**

The `response` function creates a [response](https://laravel.com/docs/7.x/responses) instance or obtains an instance of the response factory:

```text
return response('Hello World', 200, $headers);

return response()->json(['foo' => 'bar'], 200, $headers);
```

**retry\(\)**

The `retry` function attempts to execute the given callback until the given maximum attempt threshold is met. If the callback does not throw an exception, its return value will be returned. If the callback throws an exception, it will automatically be retried. If the maximum attempt count is exceeded, the exception will be thrown:

```text
return retry(5, function () {
    // Attempt 5 times while resting 100ms in between attempts...
}, 100);
```

**session\(\)**

The `session` function may be used to get or set [session](https://laravel.com/docs/7.x/session) values:

```text
$value = session('key');
```

You may set values by passing an array of key / value pairs to the function:

```text
session(['chairs' => 7, 'instruments' => 3]);
```

The session store will be returned if no value is passed to the function:

```text
$value = session()->get('key');

session()->put('key', $value);
```

**tap\(\)**

The `tap` function accepts two arguments: an arbitrary `$value` and a Closure. The `$value` will be passed to the Closure and then be returned by the `tap` function. The return value of the Closure is irrelevant:

```text
$user = tap(User::first(), function ($user) {
    $user->name = 'taylor';

    $user->save();
});
```

If no Closure is passed to the `tap` function, you may call any method on the given `$value`. The return value of the method you call will always be `$value`, regardless of what the method actually returns in its definition. For example, the Eloquent `update` method typically returns an integer. However, we can force the method to return the model itself by chaining the `update` method call through the `tap` function:

```text
$user = tap($user)->update([
    'name' => $name,
    'email' => $email,
]);
```

To add a `tap` method to a class, you may add the `Illuminate\Support\Traits\Tappable` trait to the class. The `tap` method of this trait accepts a Closure as its only argument. The object instance itself will be passed to the Closure and then be returned by the `tap` method:

```text
return $user->tap(function ($user) {
    //
});
```

**throw\_if\(\)**

The `throw_if` function throws the given exception if a given boolean expression evaluates to `true`:

```text
throw_if(! Auth::user()->isAdmin(), AuthorizationException::class);

throw_if(
    ! Auth::user()->isAdmin(),
    AuthorizationException::class,
    'You are not allowed to access this page'
);
```

**throw\_unless\(\)**

The `throw_unless` function throws the given exception if a given boolean expression evaluates to `false`:

```text
throw_unless(Auth::user()->isAdmin(), AuthorizationException::class);

throw_unless(
    Auth::user()->isAdmin(),
    AuthorizationException::class,
    'You are not allowed to access this page'
);
```

**today\(\)**

The `today` function creates a new `Illuminate\Support\Carbon` instance for the current date:

```text
$today = today();
```

**trait\_uses\_recursive\(\)**

The `trait_uses_recursive` function returns all traits used by a trait:

```text
$traits = trait_uses_recursive(\Illuminate\Notifications\Notifiable::class);
```

**transform\(\)**

The `transform` function executes a `Closure` on a given value if the value is not [blank](helpers.md#method-blank) and returns the result of the `Closure`:

```text
$callback = function ($value) {
    return $value * 2;
};

$result = transform(5, $callback);

// 10
```

A default value or `Closure` may also be passed as the third parameter to the method. This value will be returned if the given value is blank:

```text
$result = transform(null, $callback, 'The value is blank');

// The value is blank
```

**validator\(\)**

The `validator` function creates a new [validator](https://laravel.com/docs/7.x/validation) instance with the given arguments. You may use it instead of the `Validator` facade for convenience:

```text
$validator = validator($data, $rules, $messages);
```

**value\(\)**

The `value` function returns the value it is given. However, if you pass a `Closure` to the function, the `Closure` will be executed then its result will be returned:

```text
$result = value(true);

// true

$result = value(function () {
    return false;
});

// false
```

**view\(\)**

The `view` function retrieves a [view](https://laravel.com/docs/7.x/views) instance:

```text
return view('auth.login');
```

**with\(\)**

The `with` function returns the value it is given. If a `Closure` is passed as the second argument to the function, the `Closure` will be executed and its result will be returned:

```text
$callback = function ($value) {
    return (is_numeric($value)) ? $value * 2 : 0;
};

$result = with(5, $callback);

// 10

$result = with(null, $callback);

// 0

$result = with(5, null);

// 5
```

