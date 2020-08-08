# Очереди



### [Introduction](https://laravel.com/docs/7.x/queues#introduction) <a id="introduction"></a>

> ![](https://laravel.com/img/callouts/lightbulb.min.svg)
>
> Laravel now offers Horizon, a beautiful dashboard and configuration system for your Redis powered queues. Check out the full [Horizon documentation](https://laravel.com/docs/7.x/horizon) for more information.

Laravel queues provide a unified API across a variety of different queue backends, such as Beanstalk, Amazon SQS, Redis, or even a relational database. Queues allow you to defer the processing of a time consuming task, such as sending an email, until a later time. Deferring these time consuming tasks drastically speeds up web requests to your application.

The queue configuration file is stored in `config/queue.php`. In this file you will find connection configurations for each of the queue drivers that are included with the framework, which includes a database, [Beanstalkd](https://beanstalkd.github.io/), [Amazon SQS](https://aws.amazon.com/sqs/), [Redis](https://redis.io/), and a synchronous driver that will execute jobs immediately \(for local use\). A `null` queue driver is also included which discards queued jobs.

#### [Connections Vs. Queues](https://laravel.com/docs/7.x/queues#connections-vs-queues) <a id="connections-vs-queues"></a>

Before getting started with Laravel queues, it is important to understand the distinction between "connections" and "queues". In your `config/queue.php` configuration file, there is a `connections` configuration option. This option defines a particular connection to a backend service such as Amazon SQS, Beanstalk, or Redis. However, any given queue connection may have multiple "queues" which may be thought of as different stacks or piles of queued jobs.

Note that each connection configuration example in the `queue` configuration file contains a `queue` attribute. This is the default queue that jobs will be dispatched to when they are sent to a given connection. In other words, if you dispatch a job without explicitly defining which queue it should be dispatched to, the job will be placed on the queue that is defined in the `queue` attribute of the connection configuration:

```text
// This job is sent to the default queue...
Job::dispatch();

// This job is sent to the "emails" queue...
Job::dispatch()->onQueue('emails');
```

Some applications may not need to ever push jobs onto multiple queues, instead preferring to have one simple queue. However, pushing jobs to multiple queues can be especially useful for applications that wish to prioritize or segment how jobs are processed, since the Laravel queue worker allows you to specify which queues it should process by priority. For example, if you push jobs to a `high` queue, you may run a worker that gives them higher processing priority:

```text
php artisan queue:work --queue=high,default
```

#### [Driver Notes & Prerequisites](https://laravel.com/docs/7.x/queues#driver-prerequisites) <a id="driver-prerequisites"></a>

**Database**

In order to use the `database` queue driver, you will need a database table to hold the jobs. To generate a migration that creates this table, run the `queue:table` Artisan command. Once the migration has been created, you may migrate your database using the `migrate` command:

```text
php artisan queue:table

php artisan migrate
```

**Redis**

In order to use the `redis` queue driver, you should configure a Redis database connection in your `config/database.php` configuration file.

**Redis Cluster**

If your Redis queue connection uses a Redis Cluster, your queue names must contain a [key hash tag](https://redis.io/topics/cluster-spec#keys-hash-tags). This is required in order to ensure all of the Redis keys for a given queue are placed into the same hash slot:

```text
'redis' => [
    'driver' => 'redis',
    'connection' => 'default',
    'queue' => '{default}',
    'retry_after' => 90,
],
```

**Blocking**

When using the Redis queue, you may use the `block_for` configuration option to specify how long the driver should wait for a job to become available before iterating through the worker loop and re-polling the Redis database.

Adjusting this value based on your queue load can be more efficient than continually polling the Redis database for new jobs. For instance, you may set the value to `5` to indicate that the driver should block for five seconds while waiting for a job to become available:

```text
'redis' => [
    'driver' => 'redis',
    'connection' => 'default',
    'queue' => 'default',
    'retry_after' => 90,
    'block_for' => 5,
],
```

> ![](https://laravel.com/img/callouts/exclamation.min.svg)
>
> Setting `block_for` to `0` will cause queue workers to block indefinitely until a job is available. This will also prevent signals such as `SIGTERM` from being handled until the next job has been processed.

**Other Driver Prerequisites**

The following dependencies are needed for the listed queue drivers:

* Amazon SQS: `aws/aws-sdk-php ~3.0`
* Beanstalkd: `pda/pheanstalk ~4.0`
* Redis: `predis/predis ~1.0` or phpredis PHP extension

### [Creating Jobs](https://laravel.com/docs/7.x/queues#creating-jobs) <a id="creating-jobs"></a>

#### [Generating Job Classes](https://laravel.com/docs/7.x/queues#generating-job-classes) <a id="generating-job-classes"></a>

By default, all of the queueable jobs for your application are stored in the `app/Jobs` directory. If the `app/Jobs` directory doesn't exist, it will be created when you run the `make:job` Artisan command. You may generate a new queued job using the Artisan CLI:

```text
php artisan make:job ProcessPodcast
```

The generated class will implement the `Illuminate\Contracts\Queue\ShouldQueue` interface, indicating to Laravel that the job should be pushed onto the queue to run asynchronously.

> ![](https://laravel.com/img/callouts/lightbulb.min.svg)
>
> Job stubs may be customized using [stub publishing](https://laravel.com/docs/7.x/artisan#stub-customization)

#### [Class Structure](https://laravel.com/docs/7.x/queues#class-structure) <a id="class-structure"></a>

Job classes are very simple, normally containing only a `handle` method which is called when the job is processed by the queue. To get started, let's take a look at an example job class. In this example, we'll pretend we manage a podcast publishing service and need to process the uploaded podcast files before they are published:

```text
<?php

namespace App\Jobs;

use App\AudioProcessor;
use App\Podcast;
use Illuminate\Bus\Queueable;
use Illuminate\Contracts\Queue\ShouldQueue;
use Illuminate\Foundation\Bus\Dispatchable;
use Illuminate\Queue\InteractsWithQueue;
use Illuminate\Queue\SerializesModels;

class ProcessPodcast implements ShouldQueue
{
    use Dispatchable, InteractsWithQueue, Queueable, SerializesModels;

    protected $podcast;

    /**
     * Create a new job instance.
     *
     * @param  Podcast  $podcast
     * @return void
     */
    public function __construct(Podcast $podcast)
    {
        $this->podcast = $podcast;
    }

    /**
     * Execute the job.
     *
     * @param  AudioProcessor  $processor
     * @return void
     */
    public function handle(AudioProcessor $processor)
    {
        // Process uploaded podcast...
    }
}
```

In this example, note that we were able to pass an [Eloquent model](https://laravel.com/docs/7.x/eloquent) directly into the queued job's constructor. Because of the `SerializesModels` trait that the job is using, Eloquent models and their loaded relationships will be gracefully serialized and unserialized when the job is processing. If your queued job accepts an Eloquent model in its constructor, only the identifier for the model will be serialized onto the queue. When the job is actually handled, the queue system will automatically re-retrieve the full model instance and its loaded relationships from the database. It's all totally transparent to your application and prevents issues that can arise from serializing full Eloquent model instances.

The `handle` method is called when the job is processed by the queue. Note that we are able to type-hint dependencies on the `handle` method of the job. The Laravel [service container](https://laravel.com/docs/7.x/container) automatically injects these dependencies.

If you would like to take total control over how the container injects dependencies into the `handle` method, you may use the container's `bindMethod` method. The `bindMethod` method accepts a callback which receives the job and the container. Within the callback, you are free to invoke the `handle` method however you wish. Typically, you should call this method from a [service provider](https://laravel.com/docs/7.x/providers):

```text
use App\Jobs\ProcessPodcast;

$this->app->bindMethod(ProcessPodcast::class.'@handle', function ($job, $app) {
    return $job->handle($app->make(AudioProcessor::class));
});
```

> ![](https://laravel.com/img/callouts/exclamation.min.svg)
>
> Binary data, such as raw image contents, should be passed through the `base64_encode` function before being passed to a queued job. Otherwise, the job may not properly serialize to JSON when being placed on the queue.

**Handling Relationships**

Because loaded relationships also get serialized, the serialized job string can become quite large. To prevent relations from being serialized, you can call the `withoutRelations` method on the model when setting a property value. This method will return an instance of the model with no loaded relationships:

```text
/**
 * Create a new job instance.
 *
 * @param  \App\Podcast  $podcast
 * @return void
 */
public function __construct(Podcast $podcast)
{
    $this->podcast = $podcast->withoutRelations();
}
```

#### [Job Middleware](https://laravel.com/docs/7.x/queues#job-middleware) <a id="job-middleware"></a>

Job middleware allow you to wrap custom logic around the execution of queued jobs, reducing boilerplate in the jobs themselves. For example, consider the following `handle` method which leverages Laravel's Redis rate limiting features to allow only one job to process every five seconds:

```text
/**
 * Execute the job.
 *
 * @return void
 */
public function handle()
{
    Redis::throttle('key')->block(0)->allow(1)->every(5)->then(function () {
        info('Lock obtained...');

        // Handle job...
    }, function () {
        // Could not obtain lock...

        return $this->release(5);
    });
}
```

While this code is valid, the structure of the `handle` method becomes noisy since it is cluttered with Redis rate limiting logic. In addition, this rate limiting logic must be duplicated for any other jobs that we want to rate limit.

Instead of rate limiting in the handle method, we could define a job middleware that handles rate limiting. Laravel does not have a default location for job middleware, so you are welcome to place job middleware anywhere in your application. In this example, we will place the middleware in a `app/Jobs/Middleware` directory:

```text
<?php

namespace App\Jobs\Middleware;

use Illuminate\Support\Facades\Redis;

class RateLimited
{
    /**
     * Process the queued job.
     *
     * @param  mixed  $job
     * @param  callable  $next
     * @return mixed
     */
    public function handle($job, $next)
    {
        Redis::throttle('key')
                ->block(0)->allow(1)->every(5)
                ->then(function () use ($job, $next) {
                    // Lock obtained...

                    $next($job);
                }, function () use ($job) {
                    // Could not obtain lock...

                    $job->release(5);
                });
    }
}
```

As you can see, like [route middleware](https://laravel.com/docs/7.x/middleware), job middleware receive the job being processed and a callback that should be invoked to continue processing the job.

After creating job middleware, they may be attached to a job by returning them from the job's `middleware` method. This method does not exist on jobs scaffolded by the `make:job` Artisan command, so you will need to add it to your own job class definition:

```text
use App\Jobs\Middleware\RateLimited;

/**
 * Get the middleware the job should pass through.
 *
 * @return array
 */
public function middleware()
{
    return [new RateLimited];
}
```

### [Dispatching Jobs](https://laravel.com/docs/7.x/queues#dispatching-jobs) <a id="dispatching-jobs"></a>

Once you have written your job class, you may dispatch it using the `dispatch` method on the job itself. The arguments passed to the `dispatch` method will be given to the job's constructor:

```text
<?php

namespace App\Http\Controllers;

use App\Http\Controllers\Controller;
use App\Jobs\ProcessPodcast;
use Illuminate\Http\Request;

class PodcastController extends Controller
{
    /**
     * Store a new podcast.
     *
     * @param  Request  $request
     * @return Response
     */
    public function store(Request $request)
    {
        // Create podcast...

        ProcessPodcast::dispatch($podcast);
    }
}
```

If you would like to conditionally dispatch a job, you may use the `dispatchIf` and `dispatchUnless` methods:

```text
ProcessPodcast::dispatchIf($accountActive === true, $podcast);

ProcessPodcast::dispatchUnless($accountSuspended === false, $podcast);
```

#### [Delayed Dispatching](https://laravel.com/docs/7.x/queues#delayed-dispatching) <a id="delayed-dispatching"></a>

If you would like to delay the execution of a queued job, you may use the `delay` method when dispatching a job. For example, let's specify that a job should not be available for processing until 10 minutes after it has been dispatched:

```text
<?php

namespace App\Http\Controllers;

use App\Http\Controllers\Controller;
use App\Jobs\ProcessPodcast;
use Illuminate\Http\Request;

class PodcastController extends Controller
{
    /**
     * Store a new podcast.
     *
     * @param  Request  $request
     * @return Response
     */
    public function store(Request $request)
    {
        // Create podcast...

        ProcessPodcast::dispatch($podcast)
                ->delay(now()->addMinutes(10));
    }
}
```

> ![](https://laravel.com/img/callouts/exclamation.min.svg)
>
> The Amazon SQS queue service has a maximum delay time of 15 minutes.

**Dispatching After The Response Is Sent To Browser**

Alternatively, the `dispatchAfterResponse` method delays dispatching a job until after the response is sent to the user's browser. This will still allow the user to begin using the application even though a queued job is still executing. This should typically only be used for jobs that take about a second, such as sending an email:

```text
use App\Jobs\SendNotification;

SendNotification::dispatchAfterResponse();
```

You may `dispatch` a Closure and chain the `afterResponse` method onto the helper to execute a Closure after the response has been sent to the browser:

```text
use App\Mail\WelcomeMessage;
use Illuminate\Support\Facades\Mail;

dispatch(function () {
    Mail::to('taylor@laravel.com')->send(new WelcomeMessage);
})->afterResponse();
```

#### [Synchronous Dispatching](https://laravel.com/docs/7.x/queues#synchronous-dispatching) <a id="synchronous-dispatching"></a>

If you would like to dispatch a job immediately \(synchronously\), you may use the `dispatchNow` method. When using this method, the job will not be queued and will be run immediately within the current process:

```text
<?php

namespace App\Http\Controllers;

use App\Http\Controllers\Controller;
use App\Jobs\ProcessPodcast;
use Illuminate\Http\Request;

class PodcastController extends Controller
{
    /**
     * Store a new podcast.
     *
     * @param  Request  $request
     * @return Response
     */
    public function store(Request $request)
    {
        // Create podcast...

        ProcessPodcast::dispatchNow($podcast);
    }
}
```

#### [Job Chaining](https://laravel.com/docs/7.x/queues#job-chaining) <a id="job-chaining"></a>

Job chaining allows you to specify a list of queued jobs that should be run in sequence after the primary job has executed successfully. If one job in the sequence fails, the rest of the jobs will not be run. To execute a queued job chain, you may use the `withChain` method on any of your dispatchable jobs:

```text
ProcessPodcast::withChain([
    new OptimizePodcast,
    new ReleasePodcast
])->dispatch();
```

In addition to chaining job class instances, you may also chain Closures:

```text
ProcessPodcast::withChain([
    new OptimizePodcast,
    new ReleasePodcast,
    function () {
        Podcast::update(...);
    },
])->dispatch();
```

> ![](https://laravel.com/img/callouts/exclamation.min.svg)
>
> Deleting jobs using the `$this->delete()` method will not prevent chained jobs from being processed. The chain will only stop executing if a job in the chain fails.

**Chain Connection & Queue**

If you would like to specify the default connection and queue that should be used for the chained jobs, you may use the `allOnConnection` and `allOnQueue` methods. These methods specify the queue connection and queue name that should be used unless the queued job is explicitly assigned a different connection / queue:

```text
ProcessPodcast::withChain([
    new OptimizePodcast,
    new ReleasePodcast
])->dispatch()->allOnConnection('redis')->allOnQueue('podcasts');
```

#### [Customizing The Queue & Connection](https://laravel.com/docs/7.x/queues#customizing-the-queue-and-connection) <a id="customizing-the-queue-and-connection"></a>

**Dispatching To A Particular Queue**

By pushing jobs to different queues, you may "categorize" your queued jobs and even prioritize how many workers you assign to various queues. Keep in mind, this does not push jobs to different queue "connections" as defined by your queue configuration file, but only to specific queues within a single connection. To specify the queue, use the `onQueue` method when dispatching the job:

```text
<?php

namespace App\Http\Controllers;

use App\Http\Controllers\Controller;
use App\Jobs\ProcessPodcast;
use Illuminate\Http\Request;

class PodcastController extends Controller
{
    /**
     * Store a new podcast.
     *
     * @param  Request  $request
     * @return Response
     */
    public function store(Request $request)
    {
        // Create podcast...

        ProcessPodcast::dispatch($podcast)->onQueue('processing');
    }
}
```

**Dispatching To A Particular Connection**

If you are working with multiple queue connections, you may specify which connection to push a job to. To specify the connection, use the `onConnection` method when dispatching the job:

```text
<?php

namespace App\Http\Controllers;

use App\Http\Controllers\Controller;
use App\Jobs\ProcessPodcast;
use Illuminate\Http\Request;

class PodcastController extends Controller
{
    /**
     * Store a new podcast.
     *
     * @param  Request  $request
     * @return Response
     */
    public function store(Request $request)
    {
        // Create podcast...

        ProcessPodcast::dispatch($podcast)->onConnection('sqs');
    }
}
```

You may chain the `onConnection` and `onQueue` methods to specify the connection and the queue for a job:

```text
ProcessPodcast::dispatch($podcast)
              ->onConnection('sqs')
              ->onQueue('processing');
```

#### [Specifying Max Job Attempts / Timeout Values](https://laravel.com/docs/7.x/queues#max-job-attempts-and-timeout) <a id="max-job-attempts-and-timeout"></a>

**Max Attempts**

One approach to specifying the maximum number of times a job may be attempted is via the `--tries` switch on the Artisan command line:

```text
php artisan queue:work --tries=3
```

However, you may take a more granular approach by defining the maximum number of attempts on the job class itself. If the maximum number of attempts is specified on the job, it will take precedence over the value provided on the command line:

```text
<?php

namespace App\Jobs;

class ProcessPodcast implements ShouldQueue
{
    /**
     * The number of times the job may be attempted.
     *
     * @var int
     */
    public $tries = 5;
}
```

**Time Based Attempts**

As an alternative to defining how many times a job may be attempted before it fails, you may define a time at which the job should timeout. This allows a job to be attempted any number of times within a given time frame. To define the time at which a job should timeout, add a `retryUntil` method to your job class:

```text
/**
 * Determine the time at which the job should timeout.
 *
 * @return \DateTime
 */
public function retryUntil()
{
    return now()->addSeconds(5);
}
```

> ![](https://laravel.com/img/callouts/lightbulb.min.svg)
>
> You may also define a `retryUntil` method on your queued event listeners.

**Max Exceptions**

Sometimes you may wish to specify that a job may be attempted many times, but should fail if the retries are triggered by a given number of exceptions. To accomplish this, you may define a `maxExceptions` property on your job class:

```text
<?php

namespace App\Jobs;

class ProcessPodcast implements ShouldQueue
{
    /**
     * The number of times the job may be attempted.
     *
     * @var int
     */
    public $tries = 25;

    /**
     * The maximum number of exceptions to allow before failing.
     *
     * @var int
     */
    public $maxExceptions = 3;

    /**
     * Execute the job.
     *
     * @return void
     */
    public function handle()
    {
        Redis::throttle('key')->allow(10)->every(60)->then(function () {
            // Lock obtained, process the podcast...
        }, function () {
            // Unable to obtain lock...
            return $this->release(10);
        });
    }
}
```

In this example, the job is released for ten seconds if the application is unable to obtain a Redis lock and will continue to be retried up to 25 times. However, the job will fail if three unhandled exceptions are thrown by the job.

**Timeout**

> ![](https://laravel.com/img/callouts/exclamation.min.svg)
>
> The `timeout` feature is optimized for PHP 7.1+ and the `pcntl` PHP extension.

Likewise, the maximum number of seconds that jobs can run may be specified using the `--timeout` switch on the Artisan command line:

```text
php artisan queue:work --timeout=30
```

However, you may also define the maximum number of seconds a job should be allowed to run on the job class itself. If the timeout is specified on the job, it will take precedence over any timeout specified on the command line:

```text
<?php

namespace App\Jobs;

class ProcessPodcast implements ShouldQueue
{
    /**
     * The number of seconds the job can run before timing out.
     *
     * @var int
     */
    public $timeout = 120;
}
```

#### [Rate Limiting](https://laravel.com/docs/7.x/queues#rate-limiting) <a id="rate-limiting"></a>

> ![](https://laravel.com/img/callouts/exclamation.min.svg)
>
> This feature requires that your application can interact with a [Redis server](https://laravel.com/docs/7.x/redis).

If your application interacts with Redis, you may throttle your queued jobs by time or concurrency. This feature can be of assistance when your queued jobs are interacting with APIs that are also rate limited.

For example, using the `throttle` method, you may throttle a given type of job to only run 10 times every 60 seconds. If a lock can not be obtained, you should typically release the job back onto the queue so it can be retried later:

```text
Redis::throttle('key')->allow(10)->every(60)->then(function () {
    // Job logic...
}, function () {
    // Could not obtain lock...

    return $this->release(10);
});
```

> ![](https://laravel.com/img/callouts/lightbulb.min.svg)
>
> In the example above, the `key` may be any string that uniquely identifies the type of job you would like to rate limit. For example, you may wish to construct the key based on the class name of the job and the IDs of the Eloquent models it operates on.![](https://laravel.com/img/callouts/exclamation.min.svg)
>
> Releasing a throttled job back onto the queue will still increment the job's total number of `attempts`.

Alternatively, you may specify the maximum number of workers that may simultaneously process a given job. This can be helpful when a queued job is modifying a resource that should only be modified by one job at a time. For example, using the `funnel` method, you may limit jobs of a given type to only be processed by one worker at a time:

```text
Redis::funnel('key')->limit(1)->then(function () {
    // Job logic...
}, function () {
    // Could not obtain lock...

    return $this->release(10);
});
```

> ![](https://laravel.com/img/callouts/lightbulb.min.svg)
>
> When using rate limiting, the number of attempts your job will need to run successfully can be hard to determine. Therefore, it is useful to combine rate limiting with [time based attempts](https://laravel.com/docs/7.x/queues#time-based-attempts).

#### [Error Handling](https://laravel.com/docs/7.x/queues#error-handling) <a id="error-handling"></a>

If an exception is thrown while the job is being processed, the job will automatically be released back onto the queue so it may be attempted again. The job will continue to be released until it has been attempted the maximum number of times allowed by your application. The maximum number of attempts is defined by the `--tries` switch used on the `queue:work` Artisan command. Alternatively, the maximum number of attempts may be defined on the job class itself. More information on running the queue worker [can be found below](https://laravel.com/docs/7.x/queues#running-the-queue-worker).

### [Queueing Closures](https://laravel.com/docs/7.x/queues#queueing-closures) <a id="queueing-closures"></a>

Instead of dispatching a job class to the queue, you may also dispatch a Closure. This is great for quick, simple tasks that need to be executed outside of the current request cycle:

```text
$podcast = App\Podcast::find(1);

dispatch(function () use ($podcast) {
    $podcast->publish();
});
```

When dispatching Closures to the queue, the Closure's code contents is cryptographically signed so it can not be modified in transit.

### [Running The Queue Worker](https://laravel.com/docs/7.x/queues#running-the-queue-worker) <a id="running-the-queue-worker"></a>

Laravel includes a queue worker that will process new jobs as they are pushed onto the queue. You may run the worker using the `queue:work` Artisan command. Note that once the `queue:work` command has started, it will continue to run until it is manually stopped or you close your terminal:

```text
php artisan queue:work
```

> ![](https://laravel.com/img/callouts/lightbulb.min.svg)
>
> To keep the `queue:work` process running permanently in the background, you should use a process monitor such as [Supervisor](https://laravel.com/docs/7.x/queues#supervisor-configuration) to ensure that the queue worker does not stop running.

Remember, queue workers are long-lived processes and store the booted application state in memory. As a result, they will not notice changes in your code base after they have been started. So, during your deployment process, be sure to [restart your queue workers](https://laravel.com/docs/7.x/queues#queue-workers-and-deployment). In addition, remember that any static state created or modified by your application will not be automatically reset between jobs.

Alternatively, you may run the `queue:listen` command. When using the `queue:listen` command, you don't have to manually restart the worker when you want to reload your updated code or reset the application state; however, this command is not as efficient as `queue:work`:

```text
php artisan queue:listen
```

**Specifying The Connection & Queue**

You may also specify which queue connection the worker should utilize. The connection name passed to the `work` command should correspond to one of the connections defined in your `config/queue.php` configuration file:

```text
php artisan queue:work redis
```

You may customize your queue worker even further by only processing particular queues for a given connection. For example, if all of your emails are processed in an `emails` queue on your `redis` queue connection, you may issue the following command to start a worker that only processes that queue:

```text
php artisan queue:work redis --queue=emails
```

**Processing A Single Job**

The `--once` option may be used to instruct the worker to only process a single job from the queue:

```text
php artisan queue:work --once
```

**Processing All Queued Jobs & Then Exiting**

The `--stop-when-empty` option may be used to instruct the worker to process all jobs and then exit gracefully. This option can be useful when working Laravel queues within a Docker container if you wish to shutdown the container after the queue is empty:

```text
php artisan queue:work --stop-when-empty
```

**Resource Considerations**

Daemon queue workers do not "reboot" the framework before processing each job. Therefore, you should free any heavy resources after each job completes. For example, if you are doing image manipulation with the GD library, you should free the memory with `imagedestroy` when you are done.

#### [Queue Priorities](https://laravel.com/docs/7.x/queues#queue-priorities) <a id="queue-priorities"></a>

Sometimes you may wish to prioritize how your queues are processed. For example, in your `config/queue.php` you may set the default `queue` for your `redis` connection to `low`. However, occasionally you may wish to push a job to a `high` priority queue like so:

```text
dispatch((new Job)->onQueue('high'));
```

To start a worker that verifies that all of the `high` queue jobs are processed before continuing to any jobs on the `low` queue, pass a comma-delimited list of queue names to the `work` command:

```text
php artisan queue:work --queue=high,low
```

#### [Queue Workers & Deployment](https://laravel.com/docs/7.x/queues#queue-workers-and-deployment) <a id="queue-workers-and-deployment"></a>

Since queue workers are long-lived processes, they will not pick up changes to your code without being restarted. So, the simplest way to deploy an application using queue workers is to restart the workers during your deployment process. You may gracefully restart all of the workers by issuing the `queue:restart` command:

```text
php artisan queue:restart
```

This command will instruct all queue workers to gracefully "die" after they finish processing their current job so that no existing jobs are lost. Since the queue workers will die when the `queue:restart` command is executed, you should be running a process manager such as [Supervisor](https://laravel.com/docs/7.x/queues#supervisor-configuration) to automatically restart the queue workers.

> ![](https://laravel.com/img/callouts/lightbulb.min.svg)
>
> The queue uses the [cache](https://laravel.com/docs/7.x/cache) to store restart signals, so you should verify a cache driver is properly configured for your application before using this feature.

#### [Job Expirations & Timeouts](https://laravel.com/docs/7.x/queues#job-expirations-and-timeouts) <a id="job-expirations-and-timeouts"></a>

**Job Expiration**

In your `config/queue.php` configuration file, each queue connection defines a `retry_after` option. This option specifies how many seconds the queue connection should wait before retrying a job that is being processed. For example, if the value of `retry_after` is set to `90`, the job will be released back onto the queue if it has been processing for 90 seconds without being deleted. Typically, you should set the `retry_after` value to the maximum number of seconds your jobs should reasonably take to complete processing.

> ![](https://laravel.com/img/callouts/exclamation.min.svg)
>
> The only queue connection which does not contain a `retry_after` value is Amazon SQS. SQS will retry the job based on the [Default Visibility Timeout](https://docs.aws.amazon.com/AWSSimpleQueueService/latest/SQSDeveloperGuide/AboutVT.html) which is managed within the AWS console.

**Worker Timeouts**

The `queue:work` Artisan command exposes a `--timeout` option. The `--timeout` option specifies how long the Laravel queue master process will wait before killing off a child queue worker that is processing a job. Sometimes a child queue process can become "frozen" for various reasons. The `--timeout` option removes frozen processes that have exceeded that specified time limit:

```text
php artisan queue:work --timeout=60
```

The `retry_after` configuration option and the `--timeout` CLI option are different, but work together to ensure that jobs are not lost and that jobs are only successfully processed once.

> ![](https://laravel.com/img/callouts/exclamation.min.svg)
>
> The `--timeout` value should always be at least several seconds shorter than your `retry_after` configuration value. This will ensure that a worker processing a given job is always killed before the job is retried. If your `--timeout` option is longer than your `retry_after` configuration value, your jobs may be processed twice.

**Worker Sleep Duration**

When jobs are available on the queue, the worker will keep processing jobs with no delay in between them. However, the `sleep` option determines how long \(in seconds\) the worker will "sleep" if there are no new jobs available. While sleeping, the worker will not process any new jobs - the jobs will be processed after the worker wakes up again.

```text
php artisan queue:work --sleep=3
```

### [Supervisor Configuration](https://laravel.com/docs/7.x/queues#supervisor-configuration) <a id="supervisor-configuration"></a>

**Installing Supervisor**

Supervisor is a process monitor for the Linux operating system, and will automatically restart your `queue:work` process if it fails. To install Supervisor on Ubuntu, you may use the following command:

```text
sudo apt-get install supervisor
```

> ![](https://laravel.com/img/callouts/lightbulb.min.svg)
>
> If configuring Supervisor yourself sounds overwhelming, consider using [Laravel Forge](https://forge.laravel.com/), which will automatically install and configure Supervisor for your Laravel projects.

**Configuring Supervisor**

Supervisor configuration files are typically stored in the `/etc/supervisor/conf.d` directory. Within this directory, you may create any number of configuration files that instruct supervisor how your processes should be monitored. For example, let's create a `laravel-worker.conf` file that starts and monitors a `queue:work` process:

```text
[program:laravel-worker]
process_name=%(program_name)s_%(process_num)02d
command=php /home/forge/app.com/artisan queue:work sqs --sleep=3 --tries=3
autostart=true
autorestart=true
user=forge
numprocs=8
redirect_stderr=true
stdout_logfile=/home/forge/app.com/worker.log
stopwaitsecs=3600
```

In this example, the `numprocs` directive will instruct Supervisor to run 8 `queue:work` processes and monitor all of them, automatically restarting them if they fail. You should change the `queue:work sqs` portion of the `command` directive to reflect your desired queue connection.

> ![](https://laravel.com/img/callouts/exclamation.min.svg)
>
> You should ensure that the value of `stopwaitsecs` is greater than the number of seconds consumed by your longest running job. Otherwise, Supervisor may kill the job before it is finished processing.

**Starting Supervisor**

Once the configuration file has been created, you may update the Supervisor configuration and start the processes using the following commands:

```text
sudo supervisorctl reread

sudo supervisorctl update

sudo supervisorctl start laravel-worker:*
```

For more information on Supervisor, consult the [Supervisor documentation](http://supervisord.org/index.html).

### [Dealing With Failed Jobs](https://laravel.com/docs/7.x/queues#dealing-with-failed-jobs) <a id="dealing-with-failed-jobs"></a>

Sometimes your queued jobs will fail. Don't worry, things don't always go as planned! Laravel includes a convenient way to specify the maximum number of times a job should be attempted. After a job has exceeded this amount of attempts, it will be inserted into the `failed_jobs` database table. To create a migration for the `failed_jobs` table, you may use the `queue:failed-table` command:

```text
php artisan queue:failed-table

php artisan migrate
```

Then, when running your [queue worker](https://laravel.com/docs/7.x/queues#running-the-queue-worker), you can specify the maximum number of times a job should be attempted using the `--tries` switch on the `queue:work` command. If you do not specify a value for the `--tries` option, jobs will only be attempted once:

```text
php artisan queue:work redis --tries=3
```

In addition, you may specify how many seconds Laravel should wait before retrying a job that has failed using the `--delay` option. By default, a job is retried immediately:

```text
php artisan queue:work redis --tries=3 --delay=3
```

If you would like to configure the failed job retry delay on a per-job basis, you may do so by defining a `retryAfter` property on your queued job class:

```text
/**
 * The number of seconds to wait before retrying the job.
 *
 * @var int
 */
public $retryAfter = 3;
```

If you require more complex logic for determining the retry delay, you may define a `retryAfter` method on your queued job class:

```text
/**
* Calculate the number of seconds to wait before retrying the job.
*
* @return int
*/
public function retryAfter()
{
    return 3;
}
```

#### [Cleaning Up After Failed Jobs](https://laravel.com/docs/7.x/queues#cleaning-up-after-failed-jobs) <a id="cleaning-up-after-failed-jobs"></a>

You may define a `failed` method directly on your job class, allowing you to perform job specific clean-up when a failure occurs. This is the perfect location to send an alert to your users or revert any actions performed by the job. The `Exception` that caused the job to fail will be passed to the `failed` method:

```text
<?php

namespace App\Jobs;

use App\AudioProcessor;
use App\Podcast;
use Exception;
use Illuminate\Bus\Queueable;
use Illuminate\Contracts\Queue\ShouldQueue;
use Illuminate\Queue\InteractsWithQueue;
use Illuminate\Queue\SerializesModels;

class ProcessPodcast implements ShouldQueue
{
    use InteractsWithQueue, Queueable, SerializesModels;

    protected $podcast;

    /**
     * Create a new job instance.
     *
     * @param  \App\Podcast  $podcast
     * @return void
     */
    public function __construct(Podcast $podcast)
    {
        $this->podcast = $podcast;
    }

    /**
     * Execute the job.
     *
     * @param  \App\AudioProcessor  $processor
     * @return void
     */
    public function handle(AudioProcessor $processor)
    {
        // Process uploaded podcast...
    }

    /**
     * Handle a job failure.
     *
     * @param  \Exception  $exception
     * @return void
     */
    public function failed(Exception $exception)
    {
        // Send user notification of failure, etc...
    }
}
```

> ![](https://laravel.com/img/callouts/exclamation.min.svg)
>
> The `failed` method will not be called if the job was dispatched using the `dispatchNow` method.

#### [Failed Job Events](https://laravel.com/docs/7.x/queues#failed-job-events) <a id="failed-job-events"></a>

If you would like to register an event that will be called when a job fails, you may use the `Queue::failing` method. This event is a great opportunity to notify your team via email or [Slack](https://www.slack.com/). For example, we may attach a callback to this event from the `AppServiceProvider` that is included with Laravel:

```text
<?php

namespace App\Providers;

use Illuminate\Support\Facades\Queue;
use Illuminate\Support\ServiceProvider;
use Illuminate\Queue\Events\JobFailed;

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
        Queue::failing(function (JobFailed $event) {
            // $event->connectionName
            // $event->job
            // $event->exception
        });
    }
}
```

#### [Retrying Failed Jobs](https://laravel.com/docs/7.x/queues#retrying-failed-jobs) <a id="retrying-failed-jobs"></a>

To view all of your failed jobs that have been inserted into your `failed_jobs` database table, you may use the `queue:failed` Artisan command:

```text
php artisan queue:failed
```

The `queue:failed` command will list the job ID, connection, queue, failure time, and other information about the job. The job ID may be used to retry the failed job. For instance, to retry a failed job that has an ID of `5`, issue the following command:

```text
php artisan queue:retry 5
```

If necessary, you may pass multiple IDs or an ID range \(when using numeric IDs\) to the command:

```text
php artisan queue:retry 5 6 7 8 9 10

php artisan queue:retry --range=5-10
```

To retry all of your failed jobs, execute the `queue:retry` command and pass `all` as the ID:

```text
php artisan queue:retry all
```

If you would like to delete a failed job, you may use the `queue:forget` command:

```text
php artisan queue:forget 5
```

To delete all of your failed jobs, you may use the `queue:flush` command:

```text
php artisan queue:flush
```

#### [Ignoring Missing Models](https://laravel.com/docs/7.x/queues#ignoring-missing-models) <a id="ignoring-missing-models"></a>

When injecting an Eloquent model into a job, it is automatically serialized before being placed on the queue and restored when the job is processed. However, if the model has been deleted while the job was waiting to be processed by a worker, your job may fail with a `ModelNotFoundException`.

For convenience, you may choose to automatically delete jobs with missing models by setting your job's `deleteWhenMissingModels` property to `true`:

```text
/**
 * Delete the job if its models no longer exist.
 *
 * @var bool
 */
public $deleteWhenMissingModels = true;
```

### [Job Events](https://laravel.com/docs/7.x/queues#job-events) <a id="job-events"></a>

Using the `before` and `after` methods on the `Queue` [facade](https://laravel.com/docs/7.x/facades), you may specify callbacks to be executed before or after a queued job is processed. These callbacks are a great opportunity to perform additional logging or increment statistics for a dashboard. Typically, you should call these methods from a [service provider](https://laravel.com/docs/7.x/providers). For example, we may use the `AppServiceProvider` that is included with Laravel:

```text
<?php

namespace App\Providers;

use Illuminate\Support\Facades\Queue;
use Illuminate\Support\ServiceProvider;
use Illuminate\Queue\Events\JobProcessed;
use Illuminate\Queue\Events\JobProcessing;

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
        Queue::before(function (JobProcessing $event) {
            // $event->connectionName
            // $event->job
            // $event->job->payload()
        });

        Queue::after(function (JobProcessed $event) {
            // $event->connectionName
            // $event->job
            // $event->job->payload()
        });
    }
}
```

Using the `looping` method on the `Queue` [facade](https://laravel.com/docs/7.x/facades), you may specify callbacks that execute before the worker attempts to fetch a job from a queue. For example, you might register a Closure to rollback any transactions that were left open by a previously failed job:

```text
Queue::looping(function () {
    while (DB::transactionLevel() > 0) {
        DB::rollBack();
    }
});
```

