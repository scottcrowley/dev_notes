# Laravel Notes - Core Concepts


## Notes from the Laracast series about Laravel
### Naming Conventions
* ### Miscellaneous Install Notes
    * When using `Laravel Valet`:
        * If the correct site is not being served, `Apache` needs to be stopped by using `sudo apachectl stop`. Then run `valet start` to make sure valet is running.
        * To create a directory for `Valet` to look in when serving a site, cd to the directory and then type `valet park`.
        * It is possible to serve a site in a random directory, outside of the parked directory.
            * cd to the working directory of the site and type `valet link app-name` where `app-name` is the name of the app you want to use in the url. i.e. `app-name.dev`
            * To see all linked apps, type `valet links`
            * To removed a linked app, type `valet unlink app-name` where `app-name` is the name of the app you want to remove
        * Other options are available to serve the site using a secure connection and to share a served app to someone that isn't local. See docs at <https://laravel.com/docs/master/valet>
    * To use Bootstrap 4 instead of Bootstrap 3
        * Run `npm uninstall bootstrap-sass --save-dev` and then `npm install bootstrap@4.0.0-alpha.6 --save-dev`
        * Remove `resources/assets/sass/_variables.scss` and create a `resources/assets/sass/_custom.scss` file
        * Amend `resources/assets/sass/app.scss`:
        * @import "custom";
        * @import "node_modules/bootstrap/scss/bootstrap";
        * Amend resources/assets/js/bootstrap.js:
            ```js
            try {
                window.$ = window.jQuery = require('jquery');
                window.Tether = require('tether');
                require('bootstrap');
            } catch (e) {}
            ```
* ### Names of Controllers and Models
    * All words should star with caps. i.e. `TasksController` or `Task` model
* ### PHP Artisan
    * when creating a model with a migration, the model name should be singular, since Laravel will make the model name plural when creating the migration.
        * `php artisan make:model Task -m`  will create a Task model file along with a migration file for a `Tasks` table with references to the correct `Tasks` table within the migration file. The migration file is placed in `database/migrations/`
    * a controller can also be done at the same time by adding a `-c` or just `c` if other options are being used. Note that the normal method of having a model be the singular form of the word (`Task`), artisan will also make the controller singular (`TaskController`). You need to manually change this file name to keep the naming convention consistent.
        * `php artisan make:model Task -mc` will create a `Task` model file along with a migration file for a `Tasks` table with references to the correct `Tasks` table within the migration file. A controller named `TaskController` will also be added to `app/Http/Controllers`, but this name should be manually updated to `TasksController` to remain consistent.
* ### Route Model Binding
    * If a route is referencing a wildcard in the URI and a controller is being used, it is important that the wildcard slug is the same as the parameter being used in the function call within the controller file

        Routes File `(routes/web.php)`
        ```php
        Route::get('/tasks/{task}', 'TasksController@show'); //This will pass the {task} slug to the show method within the TasksController
        ```
        TaskController File `(app\Http\Controllers\TaskController.php)`
        ```php
        namespace App\Http\Controllers;

        use App\Task;
                    
        class TasksController extends Controller 
        {
            public function show(Task $task)
            {
                return view('tasks.show', compact('task'));
            }
        } 
        ```
        The `show` method is using an instance of the `Task` model and is being passed the wildcard from the route called `task`. This variable name must be the same as the one being referenced as the wildcard slug in the `Routes` file. So now the `show` function is using the `Task` model to query the `Tasks` table in the database and look for the primary key with the value being passed in the `$task` variable. This is the default behavior.`
* ### Layouts using Blade Template Engine
    * All blade  commands start with `@` and do not need to be wrapped in php tags
    * A master layout file could be created in the r`esources/views/layouts/` directory called `master.blade.php`. This file contains all the redundant html being used in the views and also contains several designated sections that are used to add in specific content for other pages.
        * Sections are designated by using `@yield('content')` where the word content can be anything to label the section being yielded.
        * `@yield` can also accept a default value as the second argument. i.e. `@yield('title', 'My Website')`
    * Partials can also be added to the master layout file to make things more organized. For example: the nav section can be in a separate file called `nav.blade.php` within the layouts directory noted above. This file then needs to be imported into the master file where ever it is needed using `@include ('layouts.nav')`. Notice that the directory structure is using a `.` instead of `/` and the `.blade.php` is left off since it isn't needed.
        * Now other views can be created in the `resources/views/` directory and will be referenced directly by the routes file.
        * These files need to start with `@extends ('layouts.master')` so the file knows which master layout file to use. Notice the `.blade.php` was not needed. Laravel assumes this already so it doesn't need to be added. Also the directory structure uses `.` instead of `/`.
            * To place content within a yielded section you use `@section ('content')` followed by a `@endsection`, after all the content for that section.
    * `@section` can also receive a second argument if you want to pass something small like a title. In this case you don't need the closing `@endsection`. i.e. `@section('title', 'Contact Page')`
    * Variables are referenced using `{{  }}` around the variable name.
		```php
        Routes File
		Routes::get('/', function () {
			$hello = 'Hello World';
			return view('hello', compact('hello')); //This will send the hello variable to the hello view
		});
		The hello.blade.php File
		…html
		{{ hello }}, my name is Scott!
		…html
        ```
    * Regular php functions can be used in blade format without having to have code wrapped in php tags. A `@` just needs to be appended and a `@end{phpfunctionname}` needs to be at the end of the code. Below is an example of a foreach.
		```php
        Routes::get('/', function () {
			$tasks = ['Go to the store', 'Make lunch', 'Take a nap'];
			return view('tasks', compact('tasks')); //This will send the tasks variable to the tasks view
		});
		The tasks.blade.php File
		…html
		@foreach ($tasks as $task)
			<ul>
				<li>{{ $task }}</li>
			</ul>
		@endforeach
		…html
        ```
    * Blade Components
        * A component can be used if there are several different partials that a template is using and each of those partials has a lot of duplicate code. See manual for examples and more detail <https://laravel.com/docs/master/blade#components-and-slots>
    * Blade statements are automatically sent through php's `htmlspecialcharacters` method to escape the data and prevent XSS attacks. To disable this use `{!! $var !!}`. Be very careful if the unescaped data is going to be sent to the server again because it can be used to make an attack.
    * There is a helper function to retrieve messages from language files. `{{ __('messages.welcome') }}` fetches the welcome message from the `resources/lang/messages.php` file. <https://laravel.com/docs/master/localization#retrieving-translation-strings>
* ### Sessions
    * **Using the session helper function**
        * `session()` can be used anywhere without importing anything.
        * setting a value is done by passing an array. `session(['name' => 'JohnDoe']);`
        * Getting a value is done by passing the name of the variable you want. It is also possible to pass a default value in case something isn't found in the session
            ```php
            session('name');
            session('name', 'default value');
            ```
        * Other methods available
            ```php
            session()->forget('name');
            session()->push('user.teams', 'developers'); //pushes to the [user][teams] array
            session()->put('name', 'JohnDoe'); //stores a value to the session
            ```
            * session getter
                ```php
                session()->get('name');
                session()->get('name', 'default');
                session()->get(function() { return 'calculated value'; });
                ```
                * `session()->get('errors')->getBag($errorBagName);` Can be used to get all the errors associtated with a certain error bag name. See the section, under the `Form Requests` section, called `Using multiple error bags`, for more info on multiple error bags
            * `session()->all();` Retrieve all session values.
            * `session()->has('name');` check if the session has a value. returns true if the item is present or null if not
            * `session()->exists('name');` checks if the item is in the session and returns true even if the value is null
            * `$value = session()->pull('name', 'default');` returns the value of an item and deletes it from the session
    * **Using the session method on the Request object**
        * Make sure to type hint the request class into any method you are using this in. `function store(Request $request) {  }`
        * `$request->session()`
            * mostly contains the same functionality as using the helper function.
            * `$value = $request->session()->pull('name', 'default');` returns the value of an item and deletes it from the session
            * `$request->flash();` flashes the entire request to the session where it will be deleted after the next http request
            * `$request->flash('name', 'JohnDoe');` flashes the a value to the session where it will be deleted after the next http request
            * `$request->reflash();` saves the data for an additional http request
            * `$request->keep(['name', 'email']);` only flashes certain values
            * `$request->flush();` deletes all session values
            * `$request->regenerate();` manual generates a new session id. Can be used to guard against certain session attacks. Laravel sets a new session id when a user logs in via the provided login controller.
    * **A flash message can also be attached to a redirect by using the with method**
        * `return redirect('/')->with('message', 'This is your message');`
* ### Vardump & Die
    * To use a `vardump` in your app you can use `dump($var);` instead.
    * To die and dump a variable you can use `dd($var);`
    * **NEW IN 5.7** There is now a dump server that will allow you to display any `dd()` or `dump()` in the console instead of on the page.
        * To start the dump server run: `php artisan dump-server`
        * Now when you execute your code, the dumps will no longer be visible in the browser but will appear in the terminal where you ran dump-server
        * You can use tools like `HTTPie` or `Postman` (see General Notes section below for details on these tools) to make various http requests when you want to test routes in the console.
* ### URL Helper Function
    * The `url` helper function is used to generate or retrieve a url for the application
        ```php
        echo url("/posts/{$post->id}"); //http://example.com/posts/1
        echo url()->current(); // Get the current URL without the query string...
        echo url()->full(); // Get the current URL including the query string...
        echo url()->previous(); // Get the full URL for the previous request...
        ```
* ### The Global Errors Variable
    * This variable is available to all views and can be populated with different alerts
    * There is a @error blade directive that can be used in a view, to check if an error exists for a certain input. Note: using Bulma CSS for this example.
        ```html
        <input class="input @error('title') is-danger @enderror" type="text" name="title" id="title">

        @error('title')
            <p class="help is-danger">{{ $errors->first('title') }}</p>
        @enderror
        ```
    * You can set up a simple partial to include in a page view that will display errors if they are present. The below example is using Twitter Bootstrap css for the classes.
	    ```php
        @if (count($errors)) // OR @if ($errors->any())
		    <div class=“form-group”>
                <div class=“alert alert-danger”>
                    <ul>
                        @foreach ($errors->all() as $error)
                            <li>{{ $error }}</li>
                        @endforeach
                    </ul>
                </div>
		    </div>
		@endif
        ```
    * See the section above called **Using multiple error bags** under the **Form Request** section for info on how to handle pages that have multiple forms and you need to isolate where the errors are being displayed
    * Other methods available on the `$errors` object
        * `->any()` Returns boolean for whether or not there are any errors
        * `->has('field_name')` Returns a boolean for whether or not there are any errors for the specified field
        * `->first('field_name')` Returns the first error message for the given field
        * `->get('field_name')` Returns the all error message for the given field
            * `->get('field_name.*')` When working with array form fields, this will return the error message for each field. Should be run in a foreach
* ### Flash Messaging
    * You can store a message that will be displayed only once on the next page load.
        * Use the `flash` method on the global helper function `session`
            * `session()->flash('message', 'Thanks for signing up!');` This will store the assigned message into the `message` session variable for the next page load only
* ### Routing Workflow Example For Resourceful Controller
    * If you are using a resourceful controller as outlined below, you don't have to set up individual routes to each endpoint. all you need to do is use `Route::resource('endpoint', 'endpointController');` i.e. If you have all the endpoints setup for `threads`, you can use `Route::resource('threads', 'threadsController');` Then you can get rid of all the other routes for each endpoint.
    * This example is using the blog post example from the series

		posts
        ```
		GET /posts displays all posts using @index()
		GET /posts/create displays a form to add a new post using @create()
		POST /posts store a new post in the database using @store()
		GET /posts/{id} displays a given post using @show()
		GET /posts/{id}/edit edit a given post using @edit()
		PATCH /posts/{id} updates the edited post using @update()
		DELETE /posts/{id} deletes the given post using @destroy()
        ```
* ### View Composers
    * A view composer is a way to hook into any view that is loaded to perform a certain action. Sort of like a callback function.
    * Using the `Archives Sidebar` example from the series, we need to pass, to any view that is using the side bar, the `archives` variable that contains an array of months w/ years that there are archived posts for
        * This view composer is going to be attached to the boot method of the `AppServiceProvider` Service Provider file.
            * inside the `boot` method, the view is passed to the `composer` method when `layouts.sidebar` is loaded. Then the view is loaded with the `archives` variable populated with the results of an `archives` method within the `post` model: 
                ```php
                \View::composer('layouts.sidebar', function($view) {
                    $view->with('archives', \App\Post::archives());
                });
                ```
            * If you want the `archives` variable on multiple pages, you can pass an array of view files in place of `layouts.sidebar`. i.e. `['layouts.sidebar', 'someotherview']`
                * If you want to share the variable on all views you can:
                    ```php
                    \View::share('archives', \App\Post::archives()); 
                    ```
                    instead of using the entire statement above. Using this will make the variable available even before the view is loaded, which may cause some problems with tests that run before the view.
                    * Or pass '`*`' instead of an array to have the variable available in all views.
* ### Service Providers
    * Service Providers are like the building blocks of Laravel. Once they are registered and the framework is completely loaded, Laravel will go back through each provider and run the `boot` method in each.
    * The `boot` method is where you can perform any logic or action with the assumption that framework is completely loaded
    * You can bind directly into the service container by using either the `App` facade `App:bind()` or the `App` helper function `auth()->bind()`
        * This is helpful if you need to reference a class and pass something to it when ever the class is called. i.e

            In the `register` method of the `AppServiceProvider.php` file `app\Providers` 
            ```php
            \App:bind('App\Billing\Stripe', function() { 
                return new \App\Billing\Stripe(config('services.stripe.secret));
            });
            ```
            This will pass the secret key, which is located in the `services` config file, to an instance of `Stripe` and return it whenever the `Stripe` class is called.
            
            instead of using `\App::bind()` you can use the `$app` property, which is always part of the service container. `$this->app->`
            
            You can then create the instance or resolve it out of the service container by using the following:
            
            `$stripe = App:make('App\Billing\Stripe');`
            
            or

            `$stripe = resolve('App\Billing\Stripe');`
* ### Eventing
    * Eventing is used to trigger an event on something that is being listened for.
    * Events and their listeners are registered in the `app\Providers\EventServiceProvider.php` file within the `$listen` property. **See further down in this section for automatic event listener discovery if you are using Laravel 5.8.9**
    ```php
    protected $listen = [
        'App\Events\SomeEvent'  => [ 'App\Listeners\SendNotification']
    ];
    ```
    * An event class can be created using php artisan: `php artisan make:event SomeEvent` 
    * A listener class can be created using php artisan: `php artisan make:listener SendNotification --event=SomeEvent` 
    * It is also possible to make both the event and listener classes automatically by running `php artisan event:generate`. This command will read the `$listen` property from the `EventServiceProvider` and auto generate both classes for each registered event
    * An event can be triggered using the `event` helper method. `event(new App\Events\SomeEvent());`
    * **Automatic Event Listener Discovery in Laravel 5.8.9**
        * To make this feature work, it needs to be turned on by adding the following to `App\Providers\EventServiceProvider.php`

            ```php
            public function shouldDiscoverEvents() {
                return true;
            }
            ```
        * The following command should be run before deploying to production
            ```php
            php artisan event:cache
            ```
* ### Laravels Pipeline API
    * Taken from a [lesson](https://laracasts.com/series/queue-it-up/episodes/9) within the [Queue it Up](https://laracasts.com/series/queue-it-up) series.
    * JW talks about the concept of pipelines and how Laravel uses them to modify data. Middleware is a good example of something being sent through a series of pipes.
    * In `app/Http/Kernel.php` there is the `$middleware` property. Consider this a list of pipes that Laravel will send the request through. Each middleware class has a `handle` method, which is required to be used as a pipe. A string, object, class or array can be sent through a pipe. Below is a basic example of how the Pipeline system can be used to modify some sort of input. In the example, we are using closures but a class can be used as long as it has a `handle` method.
        
        `routes/web.php`
        ```php
        Route::get('/', function() {
            $pipeline = app(Pipeline::class);

            $pipeline->send('hello freaking world')
                ->through([
                    function ($string, $next) {
                        $string = ucwords($string); //Hello Freaking World

                        return $next($string);
                    },
                    function ($string, $next) {
                        $string = str_ireplace('freaking', '', $string); //Hello  World

                        return $next($string);
                    }
                ])
                ->then(function ($string) {
                    dump($string); //Hello  World
                });
            
            return 'Done!';
        });
        ```
        In this example, a string is being sent (`send`) into the pipeline and then it is run `through` an array of pipes. Each pipe can modify the string or throw an exception if needed and then it passes the string off to the next pipe in the array. Once the string has been sent through all the pipes, `then` it dumped.
* ### Debugging with Laravel Telescope (Requires Laravel 5.7.7+)
    * <https://github.com/laravel/telescope>
    * **Install & Setup**
        ```zsh
        composer require laravel/telescope --dev
        php artisan telescope:install
        php artisan migrate
        ```
    * **Updating Telescope**
        * Need to republish the Telescope vendor assets
            ```zsh
            php artisan vendor:publish --tag=telescope-assets --force
            ```
    * **Usage**
        * Access the Telescope dashboard at `{project-url}/telescope` in a browser. By default this is only available in the local dev environment. See documentation for details on adding access in a production environment.
    * **Miscellaneous**
        * The telescope_entries table in the database can accumulate data very quickly.
            ```zsh
            php artisan telescope:prune
            ```
        * It is recommended to prune the database tables and schedule this action. Not sure how to implement this
            ```php
            $schedule->command('telescope:prune')->daily();
            ```
* ### Webpack & Mix
    * Use the `extract` method to make webpack create a separate js file for all your vendor js and keep your js separate.
        ```js
        mix.js('resources/js/app.js', 'public/js').extract();
        ```
        * In your template file, make sure to include all three created js files.
            ```html
            <script src="/js/manifest.js"></script>
            <script src="/js/vendor.js"></script>
            <script src="/js/app.js"></script>
            ```
    * You can also `version` the assets so the cache can be force updated when needed.
        ```js
        mix.js('resources/js/app.js', 'public/js')
            .sass('resources/sass/app.scss', 'public/css')
            .extract()
            .version();
        ```
        * You now need to reference the version files using the `mix` helper function. Both the css and js files need to use the helper function.
            ```html
            <script src="{{ mix('js/app.js') }}"></script>
            <link rel="stylesheet" type="text/css" href="{{ mix('css/app.css') }}">
            ```
