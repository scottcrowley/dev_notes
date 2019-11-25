# Laravel Notes - Jobs, Queues & Scheduling


## Notes from the Laracast series about Laravel
* ### Queues & Job Dispatching
    * Notes based on Laracast series called [Queue it Up](https://laracasts.com/series/queue-it-up)
    * By default Laravel will using the `sync` queue driver, which will execute all jobs in the queue as they are dispatched.
        * This has a similar behavior to not using a queue at all.
    * The `config/queue.php` file contains all the queue drivers. Redis is a good option to use for queuing.
        * To use `redis`, change the `QUEUE_CONNECTION` key, in the `env` file, to `redis`.
        * An additional package is required if using redis called `predis`.
            ```php
            composer require predis/predis
            ```
    * **Adding a job to a queue**
        * Use the `dispatch` helper function. This accepts either a closure or a dedicated class
            ```php
            dispatch(function() {
                logger('Hello There!');
            });
            ```
        * A job can also be delayed by using `->delay`
            ```php
            dispatch(function() {
                logger('Hello There!');
            })->delay(now()->addMinutes(2));
            // execute the job 2 minutes from now
            ```
    * **Starting a worker to handle executing the jobs in the queue**
        * There is a artisan command that starts a worker as a daemon (loads it into memory) that needs to continue running to process any dispatched jobs
            ```php
            php artisan queue:work
            ```
        * You can also use the `listen` command but that only listens to jobs being added to a given queue. Use `php artisan help queue:listen` for more detail.
        * See the section below on Failed Jobs, to see how to limit the worker to a certain number of tries or to use a timeout. If you don't and an exception is thrown when dispatching the job, it will keep retrying to run the job, which can fill up log files or overload the daemon running.
    * **Job Classes & Daemons**
        * `Artisan` command to create a job class
            ```php
            php artisan make:job JobClassName
            ```
            * If you want the job to run syncronously then you need to use the `--sync` switch
                ```php
                php artisan make:job JobClassName --sync
                ```
                * By default the created class will implement the `ShouldQueue` contract that tells Laravel that the job should not be syncronous unless you use the `--sync` switch. In this case the contract isn't implemented.
            * The class is created in the `app/Jobs` directory
        * The job class contains a `handle` method that should accept any requirements for the class to handle the job. i.e. the User class, etc.
            * Using the above example of the logger.

                The created job class
                ```php
                public function handle() 
                {
                    logger('Hello There!');
                }
                ```
        * To dispatch a job class you can still use the dispatch helper but instead of using a closer you would new up the created job class
            ```php
            dispatch(new JobClassName);
            ```
            * Since the JobClassName class uses a trait called `Dispatchable`, you can also use the following to add the job to a queue, instead of using the `dispatch` helper method. JW said this may be a more clean & readable approach.
                ```php
                JobClassName::dispatch($user);
                // does the same as dispatch(new JobClassName($user));
                ```
                * If you want to use the helper method instead of calling the `dispatch` method on the class, you can remove the `Dispatchable` trait from `JobClassName`.
        * The job class also imports a trait called `SerializeModels`, which is important when using models within the job class that are being passed in via the `contructor`. Laravel uses this trait to only adds the id of the model to the queued job when it is added but will pull in a fresh copy of the corresponding model from the database based on that id, when the job is dispatched. This allows you to use the model in normal ways as seen in the `handle` method below.

            Where ever the job is being dispatched
            ```php
            $user = App\User::first();
            dispatch(new JobClassName($user));
            ```
            Within the JobClassName file
            ```php
            protected $user;

            public function __construct(User $user) // import App\User at the top of the class
            {
                $this->user = $user;
            }

            public function handle(Filesystem $file) //import Illuminate\Filesystem\Filesystem at the top of the class
            {
                $file->put(public_path('testing.txt'), 'Saying Hi to a new user');
                logger('Hello There ' . $this->user->name);
            }
            ```
        * It is important to note that if you make large changes to a job class (type hinting a class into the `handle` method for example) and you already have a worker running, the worker needs to be restarted since it is already loaded into memory and won't reflect any of the changes made.
            ```php
            php artisan queue:restart
            ```
    * **Failed Jobs**
        * It's important to set limits on any running worker or listener

            Only try to run the job 3 times. The default is no limit or 0
            ```php
            php artisan queue:work --tries=3
            ```
            Only retry to run the job for 20 seconds. The default is 60.
            ```php
            php artisan queue:work --timeout=20
            ```
        * Where do failed jobs go
            * A failed job is stored in the database within a failed jobs table. Run the following artisan command to add a migration to this table up.
                ```php
                php artisan queue:failed-table

                php artisan migrate
                ```
        * Once a job fails, it won't retry it. It is up to you to troubleshoot using the failed jobs database table.
            * After you have found the problem, you can try to have it retry the job again by using this artisan command. You can have the command retry all failed jobs or just one. If you want just one, then you need the id of the job from the failed jobs table.

                Retry all failed jobs
                ```zsh
                php artisan queue:retry all
                ```
                Retry the first failed job in the database with an id of 1
                ```zsh
                php artisan queue:retry 1
                ```
                * The above command only removes the job from the failed_jobs table and adds it back to the queue. You need to make sure that you start up the worker again if it isn't running.
                    ```zsh
                    php artisan queue:work
                    ```
    * **Laravel Horizon**
        * Laravel Horizon is a Redis queue dashboard. [Main Site](https://horizon.laravel.com/) or [Github](https://github.com/laravel/horizon)
        * From the [documentation page](https://laravel.com/docs/master/horizon)
            * Installation:
                
                You need to make sure that `redis` is assigned to the `QUEUE_CONNECTION` key in your `env` file as noted above.
                ```zsh
                composer require laravel/horizon
                php artisan horizon:install
                ```
                Create the failed jobs migration
                ```zsh
                php artisan queue:failed-table
                php artisan migrate
                ```
            * Updating:
                * Make sure to read the [upgrade guide](https://github.com/laravel/horizon/blob/master/UPGRADE.md)
                * Republish all the assets
                    ```zsh
                    php artisan horizon:assets
                    ```
        * Once Horizon is installed, you can check out the `config/horizon.php` file or the `HorizonServiceProvider.php` file in the `app/Providers` directory.
            * The service provider has a `boot` method that contains all the notifications you'd like to set up
            * There is also a `gate` method that allowd you to add all of your authorization logic.
                * By default, anyone can access the dashboard when the environment is local. However, if it is in production, you should add the email address of authorized users to the gate method. This way, you will be required to login if the site is in production.
                    ```php
                    protected function gate()
                    {
                        Gate::define('viewHorizon', function ($user) {
                            return in_array($user->email, [
                                'johndoe@somedomain.com',
                            ]);
                        });
                    }
                    ```
        * To access your dashboard, simply go to the domain followed by `/horizon`. i.e. `http://mydomain.test/horizon`
        * To actually start horizon you need to run the following artisan command. Think of this command as a variant to `queue:work` among other horizon specific operations.
            ```zsh
            php artisan horizon
            ```
            This command needs to be rerun any time a code change is made to the jobs class.
            
            To pause or continue horizon, use:
            ```zsh
            php artisan horizon:pause
            php artisan horizon:continue
            ```
            You can gracefully terminate the horizon process, which will execute any jobs that are currently being processed then exit
            ```zsh
            php artisan horizon:terminate
            ```
        * **Tags**
            * Tags can be used to help track down certain running jobs. You can check for currently tracked tags by clicking the Monitoring link on the side of you Horizon dashboard. Here you can add a new tag to monitor.
            * By default horizon will tag a job with the id of a model (if one is being used). Using the example above where we were passing in a User model to the job class constructor, the tag would be App\User:{id}
            * You can manually add tags to a job class by adding a tags method to the class
                ```php
                public function tags()
                {
                    return ['tag1', 'tag2'];
                }
                ```
    * **Multiple Queues**
        * You can set the default queue name by updating the `queue` key in the `config/queue.php` file. It is set to `default` by default. To change the default queue name for `redis`, use the `REDIS_QUEUE` key in your `env` file.
        * You can also set a queue name when you are adding a job to a queue by using the `onQueue` method. This works because the job class uses the `Queueable` trait.
            ```php
            JobClassName::dispatch($user)->onQueue('high'); // adds the job to a queue named "high"
            ```
        * The queue name for a job is shown in the Horizon dashboard
        * If you are not using horizon and you want the queue worker to only execute a certain queue name, use the following artisan command. It is possible to run multiple workers that are handling different queues.
            ```zsh
            php artisan queue:work --queue="default"
            ```
            You may also specify and order in which you want the queues executed. Below, the high queue is executed before the default. This will also pertain to future jobs while the worker is running
            ```zsh
            php artisan queue:work --queue="high,default"
            ```
        * If you are using Horizon, and you want it to handle multiple queues, you need to update the queue array under the local & production sections of the config/horizon.php file.
    * **Using the Database Driver instead of the Redis driver**
        * The `QUEUE_CONNECTION` key needs to be updated to use the `database` driver. `QUEUE_CONNECTION=database`
        * The queue database table migration needs to be created.
            ```zsh
            php artisan queue:table
            php artisan migrate
            ```
        * Now all jobs are stored in the database and will be executed only when a worker is running
* ### Scheduling
    * Queued jobs or commands can be executed on a schedule.
    * The only thing that needs to be added to the server to have Laravel execute a schedule is the following cron
        ```zsh
        * * * * * cd /path-to-your-project && php artisan schedule:run >> /dev/null 2>&1
        ```
    * Schedules are defined in the `App\Console\Kernal.php` file.
        * All scheduled tasks are all defined in the `schedule` method
            
            A scheduled task using a closure with the `call` method. The `->daily()` method is executed every day at midnight.
            ```php
            protected function schedule(Schedule $schedule)
            {
                $schedule->call(function () {
                    DB::table('recent_users')->delete();
                })->daily();
            }
            ```
            An invokable object can also be used. (Invokable objects contain the magic method `__invoke`, which is called when a script tries to call an object as a function. See [Php.net](https://secure.php.net/manual/en/language.oop5.magic.php#object.invoke) for more info.)
            ```php
            $schedule->call(new DeleteRecentUsers)->daily();
            ```
            Artisan commands can also be scheduled by using the `command` method. These commands can be called with either the command name or the class
            ```php
            $schedule->command('emails:send Taylor --force')->daily();

            $schedule->command(EmailsCommand::class, ['Taylor', '--force'])->daily();
            ```
            Scheduling Queued Jobs using the `jobs` method.
            ```php
            $schedule->job(new Heartbeat)->everyFiveMinutes();

            // Dispatch the job to the "heartbeats" queue...
            $schedule->job(new Heartbeat, 'heartbeats')->everyFiveMinutes();
            ```
            Scheduling Shell Commands using the `exec` method.
            ```php
            $schedule->exec('node /home/forge/script.js')->daily();
            ```
        * Schedule frequency options. See [docs](https://laravel.com/docs/master/scheduling#schedule-frequency-options) for list
            * These options allow for more fine tuned scheduling
                ```php
                // Run once per week on Monday at 1 PM...
                $schedule->call(function () {
                    //
                })->weekly()->mondays()->at('13:00');

                // Run hourly from 8 AM to 5 PM on weekdays...
                $schedule->command('foo')
                        ->weekdays()
                        ->hourly()
                        ->timezone('America/Chicago')
                        ->between('8:00', '17:00');
                ```
                Note the `between` method above. This can be used to specify a time range. The inverse of `between` is `unlessBetween`.
                ```php
                $schedule->command('reminders:send')
                    ->hourly()
                    ->unlessBetween('23:00', '4:00');
                ```
                The `when` method can be used to execute a task based on a given truth.
                ```php
                $schedule->command('emails:send')->daily()->when(function () {
                    return true;
                });
                ```
                The inverse of `when` is `skip`.
                ```php
                $schedule->command('emails:send')->daily()->skip(function () {
                    return true;
                });
                ```
                Environment constraints can also be used, by using the `environments` method.
                ```php
                $schedule->command('emails:send')
                    ->daily()
                    ->environments(['staging', 'production']);
                ```
                The `timezone` method can be used to specify how times are evaluated. It is important to note that since some timezone utilize Daylight Savings, certain tasks may be executed twice or not at all on change over days. This method should be used with caution.
                ```php
                $schedule->command('report:generate')
                    ->timezone('America/New_York')
                    ->at('02:00');
                ```
                If all task should be executed based on the same timezone then you can use `scheduleTimezone` method to set the timezone to be used for all scheduled tasks.
                ```php
                /**
                * Get the timezone that should be used by default for scheduled events.
                *
                * @return \DateTimeZone|string|null
                */
                protected function scheduleTimezone()
                {
                    return 'America/Chicago';
                }
                ```
            * To prevent tasks from overlapping (executing a new task while another is still running), use the `withoutOverlapping` method. This method prevents a new task starting until the previously running task has been completed (unlocked). By default tasks are locked for 24 hours, which can be overridden by passing the number of minutes to the `withoutOverlapping` method.
                ```php
                $schedule->command('emails:send')->withoutOverlapping();

                $schedule->command('emails:send')->withoutOverlapping(10);
                ```
            * Tasks are run sequentially, so if you have multiple tasks running at the same time and some may take longer than others, later tasks may be executed much later than desired. To avoid this and run the task in the background asynchronous, then use the `runInBackground` method.
                ```php
                $schedule->command('analytics:report')
                    ->daily()
                    ->runInBackground();
                ```
            * By default, tasks are not executed in maintenance mode. If you need to have task run even when the project is in maintenance mode, then you need to use the `evenInMaintenanceMode` method.
                ```php
                $schedule->command('emails:send')->evenInMaintenanceMode();
                ```
        * Task Output
            * If you are using the `command` or `exec` methods then you can have the output of the task sent via different methods.

                `sendOutputTo` method sends the task output to a file.
                ```php
                $schedule->command('emails:send')
                    ->daily()
                    ->sendOutputTo($filePath);
                ```
                `appendOutputTo` method appends the task output to a file.
                ```php
                $schedule->command('emails:send')
                    ->daily()
                    ->appendOutputTo($filePath);
                ```
                `emailOutputTo` method outputs the task to a given email address. Make sure the Laravel email services are configured.
                ```php
                $schedule->command('emails:send')
                    ->daily()
                    ->emailOutputTo('foo@example.com');
                ```
                To send an email only if the task fails, use `emailOnFailure` method.
                ```php
                $schedule->command('emails:send')
                    ->daily()
                    ->emailOnFailure('foo@example.com');
                ```
        * Task Hooks
            * There are several hooks that can be used to hook into different events during the task lifecycle. See [docs](https://laravel.com/docs/master/scheduling#task-hooks) for more details.
                * Hooks: `before`, `after`, `onSuccess`, `onFailure`
            * Task can ping a server to announce the task is coming or that it has completed.
                * Ping hooks: `pingBefore`, `thenPing`, `pingBeforeIf`, `thenPingIf`, `pingOnSuccess`, `pingOnFailure`
                * All ping hooks require the Guzzle HTTP library.
                    ```zsh
                    composer require guzzlehttp/guzzle
                    ```
