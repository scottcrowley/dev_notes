# Laravel Notes


## Notes from the Laracast series about Laravel
### Naming Conventions
* ### Names of Controllers and Models
    * All words should star with caps. i.e. `TasksController` or `Task` model
* ### PHP Artisan
    * when creating a model with a migration, the model name should be singular, since Laravel will make the model name plural when creating the migration.
        * `php artisan make:model Task -m`  will create a Task model file along with a migration file for a `Tasks` table with references to the correct `Tasks` table within the migration file. The migration file is placed in `database/migrations/`
    * a controller can also be done at the same time by adding a `-c` or just `c` if other options are being used. Note that the normal method of having a model be the singular form of the word (`Task`), artisan will also make the controller singular (`TaskController`). You need to manually change this file name to keep the naming convention consistent.
        * `php artisan make:model Task -mc` will create a `Task` model file along with a migration file for a `Tasks` table with references to the correct `Tasks` table within the migration file. A controller named `TaskController` will also be added to `app/Http/Controllers`, but this name should be manually updated to `TasksController` to remain consistent.
* ### Route Model Binding
    * If a route is referencing a wildcard in the URI and a controller is being used, it is important that the wildcard slug is the same as the parameter being used in the function call within the controller file

        `Routes File (routes/web.php)`

        ```php
        Route::get('/tasks/{task}', 'TasksController@show'); //This will pass the {task} slug to the show method within the TasksController
        ```
        `TaskController File (app\Http\Controllers\TaskController.php)`

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
* ### Forms
    * All forms must have a `CSRF` field in it to protect against hacking attempts. The following needs to be with the form tags in a view: `{{ csrf_field() }}` This will create a hidden field in the form with a token that is used to verify the form.
        * There is a new blade directive that can be used instead: `@csrf`
    * If a form is being used for either a `PATCH` or `DELETE` request, then a `{{ method_field('PATCH') }}` is needed. This is because currently html doesn't support anything besides `GET` and `POST` as a value in the method parameter of the form tag.
        * There is a new blade directive that can be used instead: `@method('PATCH')`
    * #### Validation
        * In the processing method of your controller you can call the `validate()` method before you call your create method.
            `This example is using a store method within a Posts controller (blog example).`
            ```php
            public function store() {
                $this->validate(request(), [
                    'title' => 'required|min:10' //a pipe separated list that pertains to a field name in the submitted values (required and a min of 10 characters)
                ]); 
                //This method receives the form values and an 
                //array of validation rules.
                //request() helper function contains all 
                //the submitted form values in this example
                //make sure you have a section to display 
                //the global errors variable or no 
                //validation alerts will be visible
            }
            ```
        * Validation rules can either be a string separated by a `|` or an array
            ```php
            request()->validate([ 'field_name' => 'required|min:2' ]);
            request()->validate([ 'field_name' => ['required', 'min:2'] ]);
            ```
        * When you call `validate()` on the request, all the validated attributes are returned. You can then assign that to a var and pass it to the `create` method instead of getting the values from the `request` object
        * There may be a time when you only want to validate a field if there is data present. To do this you add the "sometimes" validation rule.
            ```php
            'email' => 'sometimes|required|email'
            ```
        * There may be sometimes where you want to allow a field to be empty. You need to add the nullable validation rule to allow this.
            ```php
            'published_at' => 'nullable|date'
            ``` 
        * **Comparing dates:**
            * If you are trying to compare to dates (start & end), the validation rules could look like this:
                ```php
                'start' => 'required|date',
                'end' => 'required|date|after:start',
                ```
    * #### Form Requests - A form request class can be used to process more complex validation logic. Following notes come from [Build A Laravel App with TDD: Improve Test Arrangements with Factory Classes](https://laracasts.com/series/build-a-laravel-app-with-tdd/episodes/17)
        * **Make the form request class**
            * `php artisan make:request UpdateProjectRequest`
            * `UpdateProjectRequest` class
                * rules method contains all the validation rules exactly the same way as it used to appear in the controller: `return request()->validate(['title'=>'sometimes|required', 'description'=>'sometimes|required', 'notes'=>'nullable']);`
                    ```php
                    public function rules() {
                        return [
                            'title'=>'sometimes|required', 
                            'description'=>'sometimes|required', 
                            'notes'=>'nullable'
                        ];
                    }
                    ```
                * `authorize` method contains all the authorization logic just like how it appeared in the controller `update` method: `$this->authorize('update', $project);`
                    ```php
                    public function authorize() {
                        return Gate::allows('update', $this->route('project')); 
                        //need to import Illuminate\Support\Facades\Gate
                    * }
                    ```
                    * The `authorize` method used in the controller can't be used since it was imported in the controller and isn't available in the form request
                    * To access the type hinted `$project` variable from the controller, you can use `$this->route` method and retrieve the `project` from the route.
                    * This is possible because the route has a variable called `project` specified in the `routes/web.php` file.
                        * `Route::patch('/projects/{project}', 'ProjectsController@update');`
            * In the `ProjectsController:update` method, you need to type hint the form request along with the project.
                * All validation will now occur before the update method is hit when the form request is type hinted.
                    ```php
                    public function update(UpdateProjectRequest $request, Project $project) { 
                        //both UpdateProjectRequest and Project need to be imported at the top of the class
                        $project->update($request->validated()); 
                            //the validated data from the Form Request is 
                            //now available in the $request object by 
                            //using the validated() method.
                        return redirect($project->path());
                    }
                    ```
            * If you want to change the validation error messages, you can add a `messages` method to the form request class.
                ```php
                public function messages() {
                    return [
                        'title.required'=>'A title is required', 
                        'description.required'=>'A description is required'
                    ];
                }
                ```
        * **Using multiple error bags:** when there are more than one form on the page and you want to isolate the errors for a specific form
            * Within your form request class, add the following protected property `protected $errorBag = 'formIdentifierName';`
            * To display the errors, just add `formIdentifierName` before any method call on the error bag.
                ```php
                @if ($errors->formIdentifierName->any()) //checks to see if any errors exist for the error bag formIdentifierName
                @foreach ($errors->formIdentifierName->all() as $error)
                ```
            * If you are using a partial to display errors and it can be used for multiple forms, you can make it dynamic by doing:
                ```php
                @if ($errors->{ $bag ?? 'default' }->any()) //default is the default name of the error bag.
                @foreach ($errors->{ $bag ?? 'default' }->all() as $error)
                ```
                * Then when you are calling the partial in your blade template, add the `$bag` variable if you want to specify something other than default
                    ```php
                    @include('errors', ['bag' => 'formIdentifierName'])
                    ```
            * If you use named error bags and are performing a test that check if the `sessionHasErrors`, you need to add a third argument to the method, which is the name of the error bag. The second argument of this method is for formatting and is null by default.
                ```php
                ->assertSessionHasErrors('name of field', null, 'formIdentifierName');
                ```
                * The same formatting must be used for the `assertSessionDoesntHaveErrors` method as well.
            * There is another method you can use when dealing with named error bags and `assertSessionHasErrors`. It's called `assertSessionHasErrorsIn` and receives the name of the error bag as the first argument, the keys as the second and formatting as the third.
                ```php
                ->assertSessionHasErrorsIn('formIdentifierName', 'name of field');
                ```
    * #### User Registration Form Password Validation
        * It is important that the form field name and id for the password confirmation field be called `password_confirmation` and the regular password field have the name and id of `password`
        * In the `store` method of the `RegistrationController`, the validation rules must specify that you want to use password confirmation. 
            ```php
            $this->validate([ 'password' => 'required|confirmed' ]);
            ```
            Now when the form is validated it will make sure that password and password_confirmation both match
    * #### Mass Assignment
        * This is when you are not explicitly assigning allowed form fields.
        * When the `create()` method is called in a model, it is important to send only the fields you are wanting and not all submitted form fields using `request()->all()`
            ```php
            Post::create([
                'title' => request('title'),
                'body' => request('body')
            ]);
            ```
        * A `protected $fillable` property array should be assigned in the model class that pertains to the form being used.
            * the array contains only the names of the fields that are ok to update
        * The `protected $guarded` property array is like the `$fillable` property except the opposite. Allow all fields except…
            * If set to an empty array, then that is bypassing the Mass Assignment protection feature
            * If this is being used it may be more efficient to create your own Model class that extends the Eloquent model class and add this property to that. Then all other model classes will extend the new model class instead of the Eloquent model class and all form fields will be allowed in all models.

                Create a new model
                ```php
                namespace App;
                use Illuminate\Database\Eloquent\Model as Eloquent;

                class Model extends Eloquent {
                    protected $guarded = [];
                }
                ```
                All other model classes would now extend this model instead of `Illuminate\Database\Eloquent\Model`
                ```php
                namespace App;

                class Post extends Model {

                }
                ```
    * #### Useful Tips
        * You can use `{{ old{'filename') }}` to populate a form field with previous data. Useful when there is an user error in the form and you want to retain the info they have already used.
        * If you are using `Recaptcha` or need any other `AJAX` functionality within your php scripts, you can use a tool called [Zttp](https://github.com/kitetail/zttp) which is a wrapper for `Guzzle`.
            * Install using `composer require kitetail/zttp`
            * `Zttp` sends `AJAX` requests as `JSON`, so if you need to send as standard form params then you can use the `asFormParams()` method. This is needed when using `Recaptcha`
                ```php
                $response = Zttp::asFormParams()
                    ->post(
                        'https://www.google.com/recaptcha/api/siteverify', 
                        [
                            'secret' => config('services.recaptcha.secret', 
                            'response' => $request->input('g-recaptcha-response'), 
                            'remoteip' => $_SERVER['REMOTE_ADDR']
                        ]
                    );
                ```
* ### User Authentication
    * The `Auth::` facade or the `auth()` helper function can be used in all user authentication functions
    * `Auth::check()` checks to see if a user is logged in
    * `Auth::user()->name` would return the user name that is logged in. An error will occur if they are not logged in, so it is best to check for this before using this function
    * `auth()->id` would return the logged in user id
    * `auth()->logout()` logs a user out that is already logged in
    * `auth()->user()->publish(new Post(request(['title','body'])));` would add a new post using the user id and the form fields being passed. Doing this would be in leu of doing a `Post::create( [ 'title' => request('title'), 'body' => request('body'), 'user_id' => auth()->id ] );`
        * There would need to be an added `publish` method in the `User` model class that would receive an instance of `Post`.
            ```php
            public function publish(Post $post) { 
                $this->posts()->save($post);
            }
            ```
            * The `save` method was used instead of `create` since the publish method receives an “newed” up instance of `Post`. If it received just simple values then the `create` method could be used and pass the array of values to the method.
        * This would then call the `posts` method that should already be in the `User` model, which creates the relationship between a user `hasMany` posts. i.e. `return $this->hasMany(Post::class);`
    * To make a page only accessible when someone is logged in, you must use middleware
        * A constructor function must be added to the controller responsible for the page in question. `public function __construct() { $this->middleware('auth'); }`. Doing this, locks down all method in the controller
        * To allow certain methods guest access, you can do something like `$this->middleware('auth')->except(['index','show']);` instead. This will lock down all method except `index` and `show`.
        * If you want a controller to have only guest methods, then a constructor function must be added to the controller responsible for the page in question. `public function __construct() { $this->middleware('guest', [ 'except' => 'destroy' ]); }`. Doing this, will make it only allow access to people that are not logged in. i.e. a login page. However you do not want to remove authentication from the destroy method since this logs a user out.
    * A `SessionsController` can be used to handle logging a user in and out.
        * The controller should have a `store` method that will receive the form request data from a login form. This method can be responsible for redirecting the user back to a login page if the credentials do not match or redirecting to a different page once they've logged in successfully.
            ```php
			public function store() {
				if (! auth()->attempt(request(['email', 'password']))) { 
                    return back(); //login page
                }
				return redirect()->home();
			}
            ```
    * If you create your own middleware file `php artisan make:middleware MiddlewareName`, then you need to register it in the `app/http/Kernal.php` file.
        * add the following to the `protected $routeMiddleware` property array
            ```php
            'middle-ware-key-name' => MiddlewareName::class
            ```
            * This is then referenced by using `->middleware('middle-ware-key-name');`
        * Parameters can be passed as well by using a colon after the key name and each param separated by a comma. `->middleware('middle-ware-key-name:param1,param2');`
    * **NEW IN 5.7** There is now a email verification contract that can be used when a user registers
        * In the User.php model, you can add email verification to the process by adding implements MustVerifyEmail to the class declaration.
            ```php
            class User extends Authenticable implements MustVerifyEmail
            ```
        * In the `routes/web.php` there should be an `Auth::routes();` that is created when you run `php artisan make:auth`
            * This route needs to be change to `Auth::routes(['verify' => true]);` to enable verification after registration
        * There is now a new route middleware called `verified` that can be used to specify the user must be verified before the route will load.
            * Available by adding this to a route: `->middleware('verified');`
* ### Vardump & Die
    * To use a `vardump` in your app you can use `dump($var);` instead.
    * To die and dump a variable you can use `dd($var);`
    * **NEW IN 5.7** There is now a dump server that will allow you to display any `dd()` or `dump()` in the console instead of on the page.
        * To start the dump server run: `php artisan dump-server`
        * Now when you execute your code, the dumps will no longer be visible in the browser but will appear in the terminal where you ran dump-server
        * You can use tools like `HTTPie` or `Postman` (see General Notes section below for details on these tools) to make various http requests when you want to test routes in the console.
* ### The Response Object
    * A list of HTTP status codes can be viewed in the `symfony/http-foundation/Response.php`
    * The following methods are a few that can be used on a response object
        * `->getContent()` - retrieves the content of the response
        * `->headers()` - retrieves the http header of the response
        * `->getStatusCode()` - retrieves the http status code of the response
    * **Creating Responses**
        * A response can be view
            * Using the view helper function: `return view('threads.index');`
            * The view method can also accept data and http status codes as the second and third parameter
                * Using the view method on the response object: `return response()->view('hello', $data, 200)->header('Content-Type', $type);`
        * You can return text or an object along with a http status code with a regular response
            * `return response('Hello', 200);` or `return response($thread, 201);`
            * When returning an object or array, it will automatically be cast to `JSON`
        * You can modify the header of the response
            * `return response('Hello', 200)->header('Content-Type', 'text/plain');`
            * If you are changing several headers then you can either use multiple calls to the `->header` method or do this:
                ```php
                return response($content)
                    ->withHeaders([
                        'Content-Type' => $type, 
                        'X-Header-One' => 'Header Value', 
                        'X-Header-One' => 'Header Value'
                    ]);
                ```
        * You can attach cookies to the response as well:
            ```php
            return response($content)->header('Content-Type', $type)->cookie('name', 'value', $minutes);
            ```
            The above `->cookie` method has a few more available parameters and is basically the same as PHP's `setcookie` method.
    * **Redirect Responses**
        * Using the redirect helper function: `return redirect('home/dashboard');`
        * Using the back helper function: `return back()->withInput();`
        * Redirect to a named route: `return redirect()->route('threads.index');`
            * Route parameters can be passed in the second argument of the route method: `return redirect()->route('profile', ['id' => 1]);`
            * Populating parameters using Eloquent Models
                * Pass the entire model to the route and Eloquent will use the `id` value of the model
                    * `return redirect()->route('profile', [$user]);` `$user->id` will be used as the identifier when passed to the profile route
                    * To use a different identifier you need to change the `getRouteKey` method in the model class
                        * `public function getRouteKey() { return $this->slug; }`
        * Redirect to a controller action
            * `return redirect()->action('HomeController@index');`
            * If the controller action requires parameters, they can be sent as the second argument in the action method
                * `return redirect()->action('UserController@profile', ['id' => 1]);`
        * Redirect to an external domain
            * `return redirect()->away('https://www.google.com');`
        * Redirect with a flash session message
            * `return redirect('dashboard')->with('status', 'Profile updated!');`
            * `status` will then be available using the session helper function. `session('status')`
    * **JSON Responses**
        * The json method will automatically set the appropriate content-type header and convert the given array or object to `JSON` using the `json_encode` PHP function.
            * `return response()->json(['name' => 'Abigail', 'state' => 'CA']);`
        * `JSONP` responses can be created by using the `withCallBack` method after the `json` method.
            * `return response()->json(['name' => 'Abigail', 'state' => 'CA'])->withCallback($request->input('callback'));`
    * **File Download Responses**
        * The `download` method can be used to force the users browser to download a file at the given path.
            * The first argument in the `download` method is the path to the file
            * The second optional argument in the `download` method is the name of the file. This name will be used to name the file being downloaded on the users computer as well. The file being downloaded needs to have an ASCII file name.
            * The third optional argument in the `download` method is the header information, if it needs to be customized.
            * `return response()->download($pathToFile);`
            * `return response()->download($pathToFile, $name, $headers);`
            * `return response()->download($pathToFile)->deleteFileAfterSend();`
    * **File Streaming Responses**
        * The `streamDownload` method can be used to turn a string response into a download response without having to write the contents of the operation to the users disk. The method accepts a callback as the first parameter, the file name as the second and an optional array of headers
            ```php
            return response()->streamDownload(function () { 
                echo GitHub::api('repo')
                    ->contents()
                    ->readme('laravel', 'laravel')['contents']; 
            }, 'laravel-readme.md');
            ```
    * **File Responses**
        * A `file` method can be used to display a file in the users browser instead of initiating a download. This would be useful for PDF's or certain images. The file path is required as the first argument and an array of headers as an option second.
            * `return response()->file($pathToFile);`
            * `return response()->file($pathToFile, $headers);`
    * Response Macros are also an option. See the documentation for more info. <https://laravel.com/docs/master/responses#response-macros>
* ### URL Helper Function
    * The `url` helper function is used to generate or retrieve a url for the application
    * `echo url("/posts/{$post->id}"); //http://example.com/posts/1`
    * `echo url()->current(); // Get the current URL without the query string...`
    * `echo url()->full(); // Get the current URL including the query string...`
    * `echo url()->previous(); // Get the full URL for the previous request...`
* ### The Global Errors Variable
    * This variable is available to all views and can be populated with different alerts
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
* ### Model Events
    * The following events are fired at different points in the model lifecycle.
        * `retrieved` - fired when the model is retrieved from the database
        * `creating` and `created` - fire when the model is saved to the database for the first time and the `create` method is used.
        * `updating` and `updated` - fired when the model already exists in the database and the `save` method is used.
        * `saving` and `saved` - fired when either of the `update` or `create` events fired. 
        * `deleting` and `deleted` - fire when the model is deleted from the database.
        * `restoring` and `restored` - 
    * **IMPORTANT TO NOTE:** If a mass update is done via Eloquent, the `saved` and `updated` events are not fired since the models are not actually retrieved.
    * To catch events on a model. 
        * Use the `$dispathedEvents` protected property in the model.
            ```php
            /**
             * The event map for the model.
             *
             * @var array
            */
            protected $dispatchesEvents = [
                'saved' => UserSaved::class,
                'deleted' => UserDeleted::class,
            ];
            ```
        * Listeners then need to be used to actually catch the events. See docs on Listeners <https://laravel.com/docs/master/events#defining-listeners>
    * **Observers**
        * These can be used to listen for fired events on a model. All the events are group together in an `Observer` class. `php artisan make:observer UserObserver --model=User`
        * The observer needs to be registered in the `AppServiceProvider@boot` (JW says to use the EventServiceProvider. Not sure if it is now better to use AppServiceProvider because the [video](https://laracasts.com/series/code-reflections/episodes/1) is from 3/2018) using the `observe` method. `User::observe(UserObserver::class);`
            ```php
            public function boot()
            {
                parent::boot();

                \App\User::observe(UserObserver::class);
            }
            ```
            or if you have a lot of observers you can use the `$observers` property to register all observers for a certain model and then loop through them in the `boot` method. Note: all observer class have been imported in the example below
            ```php
            protected $observers = [
                \App\Thread::class => [
                    ThreadObserver::class,
                    ThreadReputaionObserver::class,
                    GenerateThreadSlugObserver::class,
                    DeleteThreadRepliesObserver::class
                ],
                \App\Reply::class => [
                    ReplyObserver::class
                ]
            ];

            public function boot()
            {
                parent::boot();

                foreach ($this->observers as $model => $observers) {
                    foreach($observers as $observer) {
                        $model::observe($observer);
                    }
                }
            }
            ```
* ### Request()
    * To obtain an instance of the current HTTP request you can type-hint the `Illuminate\Http\Request` class on your controller method. `public function store(Request $request) {  }`
    * The request object can contain `POST` & `GET` data
    * **Request `Path` & `Method`:**
        * The `path` method returns the request's path information. So, if the incoming request is targeted at `http://domain.com/foo/bar`, the `path` method will return `foo/bar`:
            * `$uri = $request->path();`
        * The `is` method allows you to verify that the incoming request path matches a given pattern. You may use the `*` character as a wildcard when utilizing this method:
            * `if ($request->is('admin/*')) {  }`
        * The `url` & `fullUrl` methods can be used to retrieve the request url with or without the query string
            * `$url = $request->url();`
            * `$url = $request->fullUrl();`
        * The `method` method can be used to retrieve the type of request
            * `$method = $request->method();  //post or get`
        * The `isMethod` method allows to check if the request type is a certain one.
            * `if ($request->isMethod('post')) {  }`
    * By default, all incoming string fields are trimmed and converted to `null`, if they are empty, because the `TrimStrings` & `ConvertEmptyStringsToNull` middleware are being used. To disable these, remove them from the `App\Http\Kernel` class.
    * **Helpers**
        * `->all()` - returns the value of all the fields contained in the object.
        * `->has('field')` - checks the request and returns boolean if the given field exists
        * `->filled('field')` - does the same thing as `has()` except it checks to see if the field is empty or not.
        * `->only()` - receives an array as a parameter and only returns the values for the keys provided. Can be used to make sure nothing is being injected maliciously.
        * `->except()` - opposite of `only()`.
        * `->intersect()` - receives an array as a parameter and only returns the values for the keys that exist in both the request and the provided array. This can be used to filter out null values, like what can be given by using the `only()` method. **DEPRECATED**
        * `->input('field')` - Retrieves the users input for a given field. This will return values for the entire payload including the query string
            * A default value can be passed if the retrieved value is null. `$request->input('field', 'default');`
            * Use dot notation to access array values.
                * `$request->input('products.0.name'); //['products' => [ ['name = 'Fred'] ]`
                * The `*` can be used as a wild card. `$request->input('products.*.name');`
            * To retrieve all input values as an associative array, don't pass any arguments to the input method. `$request->input();`
        * `->query('field')` - Retrieves the users input for a given field only from the query string. `$request->query('field');`
    * **Retrieve Inputs via Dynamic Properties:**
        * For example, if there is a form with a name field, you can access the value by using `$request->name`;
    * **Retrieving JSON values:**
        * The request must have the correct Content-Type header of `application/json`
        * Dot notation can be used to access specific values.
            * `$name = $request->input('user.name');`
    * **Flashing Input to the Session:**
        * The `flash` method will flash the current request input values to the session, so they will be available during the next request to the application. `$request->flash();`
        * To flash on certain input values to the session, the `flashOnly` and `flashExcept` methods can be used.
            * `$request->flashOnly(['username', 'email']);`
            * `$request->flashExcept('password');`
        * To flash input then redirect you can use the `withInput` method
            * `return redirect('form')->withInput();`
            * `return redirect('form')->withInput($request->except('password'));`
    * **Retrieving old input values from the request:**
        * This gets the values from the previous request. `$username = $request->old('username');`
    * **Retrieving cookies from the request:**
        * `$value = $request->cookie('name');`
        * or `$value = $value = Cookie::get('name'); //uses the cookie facade`
    * **Retrieving files from the request:**
        * The `file` method can be used to gain access to an uploaded file. The `file` method returns an instance of `Illuminate\Http\UploadedFile` class, which extends the PHP `SplFileInfo` class and provides a variety of methods for interacting with the file:
            * `$file = $request->file('photo');`
            * `$file = $request->photo;`
        * To determine if the file is present, use the `hasFile` method.
            * `if ($request->hasFile('photo')) {  }`
        * To check that there were no problems uploading the file, you can use the `isValid` method.
            * `if ($request->file('photo')->isValid()) {  }`
        * The `UploadedFile` class contains many methods that can be used to gather info about a file.
            * `$path = $request->photo->path(); //the path to the file`
            * `$extension = $request->photo->extension(); //tries to determine the extension of the file`
            * Other methods are available. See documentation. <http://api.symfony.com/3.0/Symfony/Component/HttpFoundation/File/UploadedFile.html>
    * **Storing uploaded files:**
        * The `UploadedFile` class contain a `store` method, used to save the uploaded file. A filename should not be given in the path since the system will generate a unique name automatically. If you want to use a specific name, then use the `storeAs` method instead.
            * The `store` method receives the path, relative to the filesystems configured root directory, as the first argument and the name of the disk as an optional second argument.
                * `$path = $request->photo->store('images');`
                * `$path = $request->photo->store('images', 's3');`
        * The `storeAs` method can be used if you want the uploaded file to be stored with a specific name.
            * `$path = $request->photo->storeAs('images', 'filename.jpg');`
            * `$path = $request->photo->storeAs('images', 'filename.jpg', 's3');`
    * **Retrieving details from a route**
        * It is possible to get details like a model from a route by using `request()->route('routeParameter');`
        * For example:

            You have a route...
            ```php
            Route::get('/{client}/projects/create', 'ProjectsController@create')->name('projects.create');
            ```
            ... and you are using a Policy for Projects and need to access the {client} param in the policy. You'd use `request()->route('client');` within the policy to access the `Client` model.
            
            ProjectPolicy.php
            ```php
            public function create(User $user)
            {
                $client = request()->route('client');
                return ($client->user_id == $user->id);
            }
            ```
            *be careful not to use a strict comparison (`===`) since the value for `user_id` may be returned as a string (even thought it was cast as an integer within the `Client` model) and `$user->id` is an integer, as in the case above.*
* ### Models
    * #### Miscellaneous
        * Different ways to handle Policy authorization within a vue component when you want the authorization details is available in the model's json response. This example is based off of the video called [Whatcha Working On:Frontend Authorization Brainstorming](https://laracasts.com/series/whatcha-working-on/episodes/36). JW shows ways to pass authorization details about a user working with a post. He only wants to show the edit button, if the user is an admin or the user who created the post and show a delete button if the user is an admin.

            *app/Policies/PostPolicy.php*
            ```
            ...
            class PostPolicy
            {
                use handlesAuthorization;

                public function before(User $user)
                {
                    if ($user->id == 1) { //hard coding the admin user id to 1 for the example
                        return true;
                    }
                }

                public function destroy(User $user, Post $post)
                {
                    return false; //all other users are denied since the before method handles checking if the user is an admin
                }

                public function update(User $user, Post $post)
                {
                    return $user->id == $post->user_id;
                }
            }
            ```
            *app/Post*
            ```
            ...

            class Post extends Model
            {
                public function authorizations($abilities = [])
                {
                    return collect(array_flip($abilities))->map(function ($index, $ability)) {
                        return \Gate::allows($ability,$this);
                    });
                }

                //override the toArray method so the authorization details can be added to model
                public function toArray()
                {
                    //make a call to the parent toArray method so you dont have to replicate all the fields here.
                    //just append the new authorizations property
                    return parent::toArray() + [
                        'authorizations' => $this->authorizations(['update', 'destroy']);
                    ];
                }
            }
            ```
* ### Eloquent
    * **Database Migrations:**
        * The `enum` field type can be used to specify that the only certain values are accepted. i.e. for a skill level field you only want the following values: `basic, intermediate, advanced`.
            * `$table->enum('level', ['basic', 'intermediate', 'advanced'])->default('intermediate');`
        * If for whatever reason you do not want a primary key given the unsigned attribute (which it does by default) do the following in the migration
            
            ***Before***
            ```php
            $table->bigIncrements('id');
            ```
            ***After***
            ```php
            $table->bigIncrements('id')->unsigned(false);
            ```
    * **Using more complex queries:**
        * `Raw` - The word `raw` can be added to several different SQL actions (select, order by, group by, etc), so you are able to use more specific queries or SQL functions
            * Consider the following query: `select year(created_at) year, monthname(created_at), count(*) published from posts group by year, month order by min(created_at)`
                * `App\Post::selectRaw('year(created_at) year, monthname(created_at), count(*) published)->groupBy('year', 'month')->orderByRaw('min(created_at) desc')->get();`
    * **What is returned:**
        * `find` or `first` - Returns a single model if a result if found.
            * `App\Post::find(4); //returns the post with an id of 4`
            * `App\Post::first(); //returns the first found post`
        * `findOrFail` or `firstOrFail` - tries to find a record with an id of the provided argument or throws an exception if nothing is found. The exception is `Illuminate\Database\Eloquent\ModelNotFoundException`. If the exception isn't caught then a `404 HTTP response` is returned to the user.
            * `$model = App\Flight::findOrFail(1);`
            * `$model = App\Flight::where('legs', '>', 100)->firstOrFail();`
        * Aggregate methods like `count`, `sum`, `max`, `min`, `avg`, - Returns the appropriate value
            * `$count = App\Flight::where('active', 1)->count();`
            * `$max = App\Flight::where('active', 1)->max('price');`
        * `exists` and `doesntExist` - Returns a boolean
            * `DB::table('orders')->where('finalized', 1)->exists();`
            * `DB::table('orders')->where('finalized', 1)->doesntExist();`
        * `get` - returns a collection of models found.
            * `App\Post::where('user_id', 4)->get();`
        * `all` - returns all records from the database, as a collection of models, but cannot be used with any other methods like `where` or `sortBy`.
            * `App\Post::all();`
        * `latest` - returns all found records and orders them by the latest record first. This is based on '`created_at`' by default but can be changed by passing a different field name.
            * `App\Post::latest(); //latest can be used with or without a following ->get()`
        * `oldest` - returns all found records and orders them by the oldest record first. This is based on '`created_at`' by default but can be changed by passing a different field name.
            * `App\Post::oldest(); //latest can be used with or without a following ->get()`
    * **Pivot Tables**
        * The table using follows the naming convention of alphabetical order and singular form of `table1_table2`. i.e. To create a `tags` to `posts` relationship table it would be called `post_tag`.
        * The table usually doesn't have an auto incrementing id field.
        * The table would have to following fields in the migration
            * `$table->integer('post_id')`
            * `$table->integer('tag_id')`
            * `$table->primary([ 'post_id', 'tag_id' ])` This makes it so there can never be a duplicate entry where the post id and tag id exist in the same record more than once.
        * The relationship between tags and post is a belongsToMany. This relationship type almost always should be associated with a pivot table
            * In the `Post` model, you would add a `tags` method that references the `Tag` model: 
                ```php
                public function tags() { 
                    return $this->belongsToMany(Tag::class); 
                }
                ```
            * In the `Tag` model, you would add a `posts` method that references the `Post` model: 
                ```php
                public function posts() { 
                    return $this->belongsToMany(Post::class); 
                }
                ```
            * As stated above, Laravel will normally expect a pivot table to be named the singular name of each table in alphabetical order. If the name of the pivot table differs from this naming convention, then you need to provide the name of the table in the relationship method.
                ```php
                public function tags() { 
                    return $this->belongsToMany(Tag::class, 'different_pivot_table');
                }
                ```
        * Laravel has the ability to load all the tags associated with a post without making the normal additional database query.
            * So instead of running through each post in the results of (`App\Post::all()` gets all posts) to then perform a database query to get the tags associated with the each post (`$post->tags;`) you can just use `$posts = App\Post::with('tags')->get();` This will execute the tags relationship when each post is retrieved using the tags method.
        * To add a new tag to a post you don't use the `create()` or `save()` methods, as before. Now you can retrieve the post (`$post = App\Post::first();`), then the tag (`$tag = App\Tag::where('name', 'personal')->first();`) then call the `tags` method from the `$post` object and attach the `$tag` object. `$post->tags()->attach($tag);`
        * To remove a tag to post relationship you use the `detach()` method. Using the example above, you'd use `$post->tags()->detach($tag);`
        * There is a way to sync the data that should be in the pivot table for a specific object. i.e. You want to sync all the tags that should be associated with a specific post.
            * Say that a post should only be associated with tag that have the following id's `[1,6,8]`. You can sync the posts associations with the tags by providing the array of id's to the `sync` method on the model.
                * `$post->tags()->sync([1,6,8]);` Any id that is in the pivot table and not in the given array, will be detached from the pivot table. The end result will be that the pivot table will only contain the given id's
                * You can also sync without detaching any id's by running the `$post->tags()->syncWithoutDetaching([1,6,8]);` This will leave any id's that are in the pivot table that are not in the array. It basically just adds the given id's to the pivot table that are not already there.
        * You can also toggle the values in a pivot table. If the given id exists in the pivot table then it will be detached. Otherwise it will be attached to the pivot table.
            * `$post->tags()->toggle([1,6,8]);`
    * **Special Properties within a Model**
        * `protected $with = [];` An array of relationship method names that you want to always include when a model is retrieved. See the section on Eager Loading below.
        * `protected $appends = [];` An array of attribute names that you want to always include when a model is retrieved. Helpful when using a JSON request to get the data. See the section on Mutators below.
        * `protected $fillable = [];` An array of database field names that you only want to use. The opposite of $guarded. Protects against Mass Assignment issues. See section on Mass Assignment above.
        * `protected $guarded = [];` An array of database field names that you do not want to be used. The opposite of `$fillable`. Protects against Mass Assignment issues. An empty array will bypass the Mass Assignment protection entirely. See section on Mass Assignment above.
    * Eloquent always assumes that when a record is retrieved, the query is based off of the primary key.
        * This behavior can be changed by adding a `getRouteKeyName()` method to the model in question
            * In the `Tag` model: `public function getRouteKeyName() { return 'name'; }` which tells Eloquent to use the `name` field when querying the table instead of the primary key.
    * **`hasManyThrough` Relationships**
        * Using an example where there is a users, posts and affiliations (contains either 'Liberal' or 'Conservative' and the id is stored on the users table) table. If you wanted to see all posts from Liberals then you could setup a `hasManyThrough` relationship on the affiliations model.
            * `public function posts() { return $this->hasManyThrough(Posts::class, Users::class); }` Think of it like: `affiliations` have many `posts` through `users` on the `users` table.
    * **`hasManyThrough` Relationship when using a Pivot Intermediate Table**
        * Lego DB Example: We want to grap all parts that are associated with a storage location. `StorageLocation`->belongsToMany->`PartCategory` & uses pivot part_category_storage_location to create a relationship. `PartCategory`->belongsTo->`Part`. If the pivot table wasn't being used and StorageLocation_id was stored in PartCategory, then you could use a simple `hasManyThrough` relationship. At the time this note is being made, this isn't possible using simple relationships.
        * Found a package on GitHub to accomplish this. [eloquent-has-many-deep](https://github.com/staudenmeir/eloquent-has-many-deep)
            * Install using
                ```zsh
                composer require staudenmeir/eloquent-has-many-deep:"^1.7"
                ```
            * Add the following use statement to the class where you want to use this functionality
                ```php
                class StorageLocation extends Model
                {
                    use \Staudenmeir\EloquentHasManyDeep\HasRelationships;
                }
                ```
            * Using example above, the following relation was created in the StorageLocation model
                ```php
                public function parts()
                {
                    return $this->hasManyDeep(
                        Part::class, 
                        ['part_category_storage_location', PartCategory::class]
                    );
                }
                ```
            * Check [documentation](https://github.com/staudenmeir/eloquent-has-many-deep) for examples on how to use in different scenarios.
    * **`Polymorphic` Relationships**
        * Using an example where there is a `users` table, a `posts` table, a `comments` table and a `likes` table. The `likes` table will be used for many different types of likes (`comments`, `posts`, etc) and consist of the following fields: `user_id` = user id of the user who liked the item, `likeable_id` = the id of the item being liked (a post, comment, etc) , `likeable_type`  = the class name for the item being liked (`App\Post`, `App\Comment`, etc). 
            * The `Post` and `Comment` model will have a `likes()` method, which will retrieve all the likes for that type.
                * This method will return a `morphMany` relationship. `return $this->morphMany(Like::class, 'likeable');` The 2nd param is stating the naming convention used in the like pivot table (`likeable_id` & `likeable_type`)
            * As a side note: in the migration for the likes table you can now add `$table->morphs('likeable');` This will add the `likeable_type` and `likeable_id` fields to the table automatically when it is migrated.
                * You can also use the `nullableMorphs` method if you want to allow the `_type` & `_id` fields to be `nullable`.
        * Another example would be there is a `videos` table, a `series` table and a `collections` table. A video can be part of a `series` (stored in the `series` table) or any other miscellaneous type of grouping which is stored in the `collections` table.
            * The `videos` table migration contains `$table->morphs('watchable');` which will create `watchable_type` and `watchable_id` in the table during migration
            * Each of the `series` and `collection` models will have a `videos` method
                * `public function videos() { return $this->morphMany(Video::Class, 'watchable'); }` Returns all videos with the appropriate relationship
            * The `video` model will have a `watchable` method to retrieve the relationship
                * `public function watchable() { return $this->morphTo(); }` This will return either return a collection of a series model or collection model for the video in question. The method name needs to be the same as the name chosen in the migration, which is `watchable` in this case. 
                    * Or if you want the method name to be something else, then you can pass the morphing name in the `morphTo` method.
                        * `public function viewable() { return $this->morphTo('watchable'); }`
        * If you don't want to use the default naming convention for the type (i.e. `App\Series` or `App\Collection`). Using the video example above. Instead of having `App\Series` stored in the database, you wanted just `series`. And instead of having `App\Collection` stored in the database, you wanted just `collection`.
            * In `app\providers\AppServiceProvider.php`, with in the `boot` method.
                * add `use Illuminate\Database\Eloquent\Relations\Relation;` to the top of the document.
                ```php
                public function boot() { Relation::morphMap( [ 'series' => 'App\Series', 'collection' => 'App\Collection' ] ); }
                ```
    * **Mutators** - A way to retrieve and modify attributes within a given model. This is used when you'd like to either retrieve a column value and modify it or modify a specific column to have a newly formatted value.
        * You can add a getter for an existing attribute or a new attribute by wording the models method name the following way `getColumnNameAttribute()`, where the `ColumnName` is the `“studly”` case version of the attribute you want. (i.e. `column_name`). If a param is included with the method `getColumnNameAttribute($value)`, then a value will first be passed to the it then it can be returned.
            
            Within the model:
            ```php
            public function getFirstNameAttribute($value) {
                return ucfirst($value);
            }
            ```
            You can then access the attribute:
            ```php
            $user = App\User::find(1);
            $firstName = $user->first_name; //should return a Capitalized version of the first_name column from the model.
            ```
        * You can also set an attributes value by using the same approach as above but use `set` instead of `get`.
        * You can create new attributes this same way but you would not use a param. For example: You want to add `example_attribute` attribute to the model (where the `example_attribute` column doesn't exist), so you can access it like `$user->example_attribute`. You would add a `getExampleAttributeAttribute()` method to your model, which would handle your required processing for the value.
        * It is also possible to hook into a model event and then apply a mutator then:

            Within the model:
            ```php
            protected static function boot() {
                parent::boot();
                static::created(function ($thread) {
                    $thread->update(['slug' => $thread->title]);
                }
            }
            ```
            This would then trigger the `setSlugAttribute` method when the thread was created.
    * **Miscellaneous Methods**
    * `whereMonth` and `whereYear` can be used to filter a query using a special where clause. Usage: `whereMonth('field_name', value);`
    * the `pluck()` method can be used to pluck out just the passed in field name with the results instead of all fields. `$tags = App\Tag::pluck('name');` will just included all the `name` values from the `Tags` table.
    * the `has()` method is used when you want to check a method within a model for results before returning all results. i.e. You want to display all the tags that have posts attached to them. `$tags = App\Tag::has('posts')->pluck('name');` This will use the `posts` method in the `Tag` model to see if there are related posts and then only return the tag names that have posts.
    * `take()` will limit the record set to the number provided.
    * `chunk()` will allow you to pull sets of the given number from the database at a time. This saves on memory usage. Usage: `\App\Users::chunk(100, function() { //do something with the chunk of users here. });` will pull all users from the database in chunks of 100 and perform the callback function provided.
    * `is()` & `isNot()` can be used to check if 2 models are the same or not.
        * `auth()->user()->is($project->owner); // checks to see if the current logged in user is the same as the owner of a project.`
    * **Eager Loading:** Defining relationships between different models or tables in the database
        * Using Blog Post/Comments example from series
            * Using the logic that a post `hasMany` comments and a comment `belongsTo` one post
                * The `Comments` table would contain a field reference to the `Posts` table. In this case it would be call '`post_id`'. Using the convention of the singular version of the table being referenced.
                * In the `Post` model, a `comments()` method would be added to gather all the comments associated with a post. This method would return a `hasMany` instance of the `Comment` model class. `return $this->hasMany(Comment::class);` Doing it this way, the `Post` model will auto fill the `post_id` with the current post id and return that instance of the `Comment` class.
                    * The `comments` method should not be called as a method but as a property instead. When Laravel sees this, it will load the relationship. `$comments = $post->comments;`
                * To perform the inverse and get the `post` associated with a `comment`, you would add a `post()` method to the `Comment` model. This method would return a `belongsTo` instance of the `Post` model class. `return $this->belongsTo(Post::class);`
                    * Just like the `comments` method in the `Post` model, you would call the `post` method in the `Comment` model as a property and not a method. `$post = $comment->post;`
                * Having the `comments()` & `post()` methods in the different model classes also allows us to call other methods on those classes using the relationships already setup.
                    * For example: someone is adding a comment to a post.
                        * Set up a route to accept the data from the form being submitted. This would call a `store` method in the `CommentsController`, which would accept an instance of the `Post` model
                        
                            ```php
                            public function store(Post $post) { 
                                $post->addComment(request('body')); 
                            }
                            ```
                        * Now the `Post` model will actually handle adding the comment to the database. The `store` method, in the `CommentsController` will call an `addComment` method in the `Post` model and pass it any of the request data needed. In this case it would be just the `body` field from the form. The `addComment` method now just needs to call it's own `comments` method followed by the `create` method with the body being passed as a parameter. All the relationship info is handled by Laravel.
                            ```php
                            public function addComment($body) {
                                $this->comments()->create([
                                    'body' => $body
                                ]);
                            }
                            ```
            * Always eager load certain relationships within a model.
                * If there is a time when you need to always eager load a relationship (using a method within the model), then you can add a `protected $with` property to the model. i.e. Within the `Post` model: `protected $with = ['comments'];` will load the comments relationship via the `comments` method within the `Post` model. This way, whenever a post is retrieved then the comments will be available without having to run another sql query.
        * When referencing a relationship in a model, there are 2 ways to do it. 
            * Dynamic property `$comments = $post->comments`
                * When you use a dynamic property, a collection is performed and the sql statement is executed.
            * As a method `$comments = $post->comments()->latest()-first()` 
                * When using the method, you are given an instance of the query builder (sql statement not executed yet), where you can continue to add other conditions on to the query.
    * **Relationship Notes:**
        * When using a relationship within a model, you can pass a second argument to the relationship method that tells Eloquent to use a different primary key to use instead of the default one.
            * By default, Eloquent will calculate the primary key name based off of the model name and the method name being used to call the relationship.
                * `post_id` when dealing with the `Post` model, `user_id` when dealing with the `User` model, etc
                * If you wanted to call the `post` to `user` relationship, `creator` instead of `user`, you'd need to tell Eloquent to use `user_id` instead of `creator_id`
                    * `public function creator() { return $this->belongsTo(User::class, 'user_id'); }`
            * If you pass a the second argument to the relationship method, then you can change the default behavior. For example: In the `User` model you add a method called `posts` to gather all the `posts` for that user. However, in the `Posts` table, you use `owner_id` instead of `user_id`. You'd need to create the relationship in the following way.
                * User model: `public function posts() { return $this->hasMany(Post::class, 'owner_id'); }`
        * It is possible to update the `updated_at` timestamp for a parent model of a `belongsTo` or `belongsToMany` relationship.
            * On the child model you need to add a `touches` property, which is an array of the relationships you want to update timestamps for whenever the child model is updated.
                * For example: You have a Task Manager app, and you want to update the parent `Project` model whenever the `Task` model is updated
                    * In the `Task` model, add `protected $touches = ['project'];`
                    * This will reference the `project` method also in the `Task` model
                        * `public function project() { return $this->belongsTo(Project::class); }`
                    * Now, whenever the `Task` is updated, the parent model (`Project`) will also have the `updated_at` timestamp updated.
        * You can add a sorting method to a relationship method. Useful if you always want a relationship gathered in a certain order.
            * `public function project() { return $this->belongsTo(Project::class)->latest('updated_at'); }`
            * `public function project() { return $this->belongsTo(Project::class)->orderBy('updated_at', 'desc'); }`
            * `public function project() { return $this->belongsTo(Project::class)->orderByDesc('updated_at'); }`
    * **DB query logging:** Useful when using Tinker. **NOTE: SHOULD ONLY BE DONE FOR TESTING PURPOSES AND NOT IN PRODUCTION**
        * `DB:enableQueryLog();` Enables logging
        * `DB::getQueryLog();` Shows the current query log
    * **Other Creation Methods:**
        * `firstOrCreate` - Tries to find a record in the database matching the provided arguments and if not found then it will create it using the arguments passed. You can also pass a second argument that can be used for other attributes you would want used in the create but not in there where clause for the find. An instance of the model will be returned using this method.
            * `$flight = App\Flight::firstOrCreate(['name' => 'Flight 10']); // Retrieve flight by name, or create it if it doesn't exist...`
            * `$flight = App\Flight::firstOrCreate(['name' => 'Flight 10'], ['delayed' => 1]); // Retrieve flight by name, or create it with the name and delayed attributes...`
        * `firstOrNew` - The exact same as `firstOrCreate` except the model is not persisted to the database if a record isn't found.  An instance of the model will be returned using this method. You will still need to run `->save()` on the model to persist it to the database.
        * `updateOrCreate` - Can be used to update a record in the database or create a new record if the provided arguments returns no results. This method will persist the model to the database if it needs to create a new model.
            * `$flight = App\Flight::updateOrCreate(['departure' => 'Oakland', 'destination' => 'San Diego'], ['price' => 99]); // If there's a flight from Oakland to San Diego, set the price to $99. If no matching model exists, create one.`
    * **Deleting Models:**
        * Delete the model directly on an instance of the model.
            * `$flight = App\Flight::find(1); $flight->delete();`
        * Delete an existing model using the primary key, a collection of keys or an array of keys.
            * `App\Flight::destroy(1);`
            * `App\Flight::destroy(1, 2, 3);`
            * `App\Flight::destroy([1, 2, 3]);`
            * `App\Flight::destroy(collect([1, 2, 3]));`
        * Delete a model by query
            * `$deletedRows = App\Flight::where('active', 0)->delete();`
        * When issuing mass delete statements, it's important to note that the `deleting` and `deleted` events will not file since the model isn't created first.
    * **Miscellaneous Notes:**
        * Within a model you can use the `protected $casts` property to tell laravel how certain fields should be handled. i.e. `'confirmed' => 'boolean'`
            * Allowed cast types: `integer, real, float, double, string, boolean, object, array,  collection, date, datetime, and timestamp`
        * By default, pivot tables do not record timestamps when the record is created through a relationship. If you want timestamps you need to use the `withTimestamps()` method on the relationship.
            ```php
            public function tags() { return $this->belongsToMany(Tag::class)->withTimestamps(); }
            ```
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
* ### Sessions
    * **Using the session helper function**
        * `session()` can be used anywhere without importing anything.
        * setting a value is done by passing an array. `session(['name' => 'JohnDoe']);`
        * Getting a value is done by passing the name of the variable you want. It is also possible to pass a default value in case something isn't found in the session
            * `session('name');`
            * `session('name', 'default value');`
        * Other methods available
            * `session()->forget('name');`
            * `session()->push('user.teams', 'developers');` pushes to the `[user][teams]` array
            * `session()->put('name', 'JohnDoe');` stores a value to the session
            * session getter
                * `session()->get('name');`
                * `session()->get('name', 'default');`
                * `session()->get(function() { return 'calculated value'; });`
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
* ### Creating Custom Frontend Presets
    * A good package that can be used to install `Bulma` css framework and update the necessary views with the correct classes. <https://github.com/laravel-frontend-presets/bulma>
    * There is a Tailwind Laravel Preset available at <https://github.com/laravel-frontend-presets/tailwindcss> that will automatically install Tailwind and change the default views to use Tailwind classes.
    * Taken from the [How to Create Custom Laravel Presets](https://laracasts.com/series/how-to-create-custom-presets)
    * Out of the box, Laravel comes with 4 presets that can be used to scaffold the frontend. [`none`, `bootstrap`, `vue`, `react`]
        * These are accessed by running the command `php artisan preset --presetName`
    * We are going to make a Laracasts preset in the example
        * Create a `Laracast` directory in the root and add a `src` directory within it.
        * Create `Laracast/src/stubs` directory
            * Create `Laracasts/src/stubs/webpack.mix.js`

                ```js
                let mix = require('laravel-mix');
                require('laravel-mix-tailwind');
                mix
                    .js('resources/assets/js/app.js', 'js')
                    .sass('resources/assets/css/sass/app.scss', 'css')
                    .tailwind();
                ```
                * Create `Laracasts/src/stubs/app.js`
                    * `require('./bootstrap');`
                * Create `Laracasts/src/stubs/bootstrap.js`
                    
                    ```js
                    window.axios = require('axios');
                    window.axios.defaults.header.common['X-Requested-With'] = 'XMLHttpRequest';
                    let token = document.head.querySelector('meta[name=“csrf-token”]');
                    if (token) { 
                        window.axios.defaults.header.common['X-CSRF-TOKEN'] = token.content; 
                    }
                    else { 
                        console.error('CSRF Token not found: https://laravel.com/docs/csrf#csrf-x-csrf-token'); 
                    }
                    ```
        * Run `php artisan make:provider LaracastsServiceProvider`
            * This will make a new provider within `app/providers`. Move the file to `Laracasts/src`
            * Update the `namespace` in the file to `Laracasts` instead of `App\Providers`
            * Open `config/app.php` 
                * in the providers array under the `Package Service Providers` comment block add `Laracasts\LaracastsServiceProvider::class,`
            * Open `composer.json`
                * go to `“autoload”: “psr-4”` and add a new root namespace `“Laracasts\\”: “Laracasts/src/“`
                    * Now when you namespace a file with `Laracasts`, the app will know to look in `Laracasts/src` for the file.
        * Open the `LaracastsServiceProvider` service provider
            * The new command for the preset can be added to the `boot` method.
                
                add 
                ```php
                PresetCommand::macro('Laracasts', function($command) { 
                    Preset::install(); 
                    $command->info('Laracast Preset applied successfully! Please recompile your assets.'); 
                });
                ```
                make sure to import `PresetCommand` from `Illuminate/Foundation/Console/PresetCommand`
        * run `composer dump-autoload`
            * Now you should be able to use the command `php artisan preset Laracasts`
        * Create a `Laracasts/src/Presets.php` file
            
            ```php
            namespace Laracasts;
            use Illuminate\Support\Arr;
            use Illuminate\Foundation\Console\Presets\Preset as LaravelPreset;
            use Illuminate\Support\Facades\File;
            
            class Preset extends LaravelPreset
            {
                public static function install() {
                    static::cleanSassDirectory();

                    //call method on LaravelPreset, which will call updatePackageArray, 
                    //which was overridden in this file
                    static::updatePackages(); 

                    static::updateMix();
                    static::updateScripts();
                    static::updateStyles();
                }
                public static function cleanSassDirectory() {
                    File::cleanDirectory(resource_path('assets/sass'));
                }
                //is passed all the base packages from updatePackages
                public static function updatePackageArray($packages) { 
                    return array_merge([
                        'laravel-mix-tailwind' => '^0.1.0'], 
                        Arr::except($packages, [ //method allows you to remove certain keys from an array
                        'popper.js', 'lodash', 'jquery'
                    ]));
                }
                public static function updateMix() {
                    //copies the stub to the root path of the project
                    copy(__DIR__.'stubs/webpack.mix.js', base_path('webpack.mix.js'));
                }
                public static function updateScripts() {
                    copy(__DIR__.'stubs/app.js', resource_path('assets/js/app.js'));
                    copy(__DIR__.'stubs/bootstrap.js', resource_path('assets/js/bootstrap.js'));
                }
                public static function updateScripts() {
                    //creates a blank sass file
                    File::put(resource_path('assets/css/sass/app.scss'), '');
                }
            }
            ```
        * To extract the above to an independent project that can be pulled in by anyone, see the Laracast video, from the series, called [Distribute a Composer Package](https://laracasts.com/series/how-to-create-custom-presets/episodes/4)
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
* ### General notes
    * **Carbon:**
        * Full docs are available at <https://carbon.nesbot.com/docs/>
        * The default timezone can be changed in the `config/app.php` file under the `'timezone'` key. This timezone is used for all timestamps and Carbon dates & time
            * For Phoenix use `'America/Phoenix'`. A full list is available at <https://www.php.net/manual/en/timezones.php>
        * When you want to use `carbon` in a controller or model, you need to have `use Carbon\Carbon;` at the top of the file. This is not needed when using `carbon` with blade.
            * When using blade and you are referencing variables that are models, then the are already instances of Carbon. Access the formatting methods directly off the variable. e.g. `$user->created_at->diffForHumans()`
        * **Laravel Helpers**
            * `now()` - Returns a Carbon Instance. Same as doing `Carbon\Carbon::now()`
            * The following methods are available on the Carbon Instance
                * `->subHours(3)` - Subtracts 3 hours from now. This methodology can be used for other modifiers like: `subYear()`, `subMinutes()`, `subCentury()`, etc. Substitute `add` for `sub` to change the method type.
                * To modify the timestamp you can also use `->add(3, 'hours')` or any other modify type listed above.
                * `->toDateTimeString()` - Converts the timestamp to a string
                * `->format('Y-M-D H:i:s')` - Returns a string equivalent for the timestamp in the given format.
    * `Scopes` do not need to return a value. Processing can just take place within the `scope` function
    * When using a `trait` with a model and you need to have something triggered in the `boot` method of the model and the logic should exist in the trait itself, you can add a `protected static` method to the trait using the following naming convention: If the traits' name is `traitName`, then the method should be called `bootTraitName`. i.e. `protected static function bootTraitName() { //boot logic here }`
    * If you are using model events and you are performing an sql action in a method, then the event may not fire unless you create a collection first.
        * For example: You have an `unfavorite()` method that performs a sql query to delete all the records for an action. `$this->favorites()->where( [ 'user_id' => auth()->id() ])->delete().` This won't fire any events since a model collection is not being made first.
        * Instead use 
            ```php
            $this->favorites()
                ->where(['user_id' => auth()->id()])
                ->get()->each(function ($favorite) { 
                    $favorite->delete(); 
                });
            ```
            This will fire a deleting event for every favorite in the collection because `get()` was used to create the model collection.
            
            You can also use a `higher order collection` (or higher order messaging <https://laravel.com/docs/master/collections#higher-order-messages>) by calling each as a property and not a method. 
            ```php
            $this->favorites()
                ->where([ 'user_id' => auth()->id()])
                ->get()
                ->each
                ->delete();
            ```
    * A useful tool to sanitize data on your site is `stevebauman/purify` on <https://packagist.org>. Install using composer
        * <https://github.com/stevebauman/purify>
    * It can be useful to have certain Laravel attributes available to javascript. For example: the csrf token value, or the authenticated user model or if the user is signed in or not, etc....
        * Add the following to a script tag within the head of `resources/views/layouts/app.blade.php`
            ```js
            window.App = {
                !! json_encode([
                    'csrfToken' => csrf_token(), 
                    'user' => Auth::user(), 
                    'signedIn' => Auth::check()
                ]) !!
            };
            ```
    * A useful tool for adding searchable user names within a thread post or reply. When the user types '`@`' followed by the username, this plugin will populate a menu with suggested names
        * <https://github.com/ichord/At.js>
    * `Travis CI` can be used to automatically check your `GitHub` commits and PRs for testing errors. <https://travis-ci.com/>
    * `Style CI` can be used to automatically check your `GitHub` commits and PRs for formatting inconsistencies. <https://styleci.io/>
    * Apps that are handy in making http requests without setting up a page in your site:
        * `HTTPie` is a command line tool that can be used to make http requests using CLI. Handy to make post requests to an endpoint without setting up a page to send the post request. <https://httpie.org/>
        * `Postman` is an App available in the App Store or <https://www.getpostman.com>
    * `PHP 7.1` added `nullable` types
        * If you place a '`?`' in front of an argument in a function, you can pass a normal value or null. Normally null would throw an error. `function test(?User $user) { return $user; }`
    * Setting up new db with mySql in the terminal after the project has been created
        * `mysql -uroot -p`
            * use the user & password that is setup with mysql
        * Once you've connected type `create database {dbname};` Make sure to include the `;`
            * The dbname should be in snake case
                ```zsh
                create database my_new_database;
                ```
        * If you need to use the db right now you can type `use {dbname};`
        * `Ctrl-z` to exit out of mysql
    * Create a cache file for all the config files located in the `config/` directory (PRODUCTION ONLY. NEEDS TO BE RUN FOR PRODUCTION)
        * This command will take all the config files and cache them into one file. Note that if you change any values in the .env file, then you would need to clear the config cache.
            * `php artisan config:cache` Creates the config cache. Only do this in production.
            * `php artisan config:clear` Clears the config cache created with config:cache.
    * To log something to the Laravel log file you can use the `\Log` class
        * `\Log::info('This happen at this time');`
    * To exit a script with specific error code you can use the `abort` function
        * `if ($projects->owner_id !== auth()->id()) abort(403); //unauthorized`
        * You can also shorten this by using the `abort_if` helper function
            * `abort_if($projects->owner_id !== auth()->id(), 403);`
        * Inverse of `abort_if` is `abort_unless` helper function
    * To auto load a certain file or class with every request, you can use the `composer.json` file
        * The are 2 sections for autoloading. `autoload` & `autoload-dev`
        * Each section can have files loaded to them using the following keys
            * `psr-4`: you can specify mappings for namespaces to paths relative to the package root.
                * Laravel sets one in the autoload section by default for the `App\` namespace. `"autoload": { "psr-4": { "App\\": "app/" } }`
                    * This tells the app to look in the `root/app` directory whenever `App\` is used.
            * `files`: an array containing specific files to load with every request
                * `"autoload-dev": { "files": [ "tests/utilities/functions.php" ] }`
                    * This will load the `functions.php` file during every request only in the dev environment.
    * Custom 404 or other status code pages
        * If you add an `errors` directory within the `resources/views` directory then Laravel will look here before rendering the default status code error pages
            * `resources/views/errors/404.blade.php`
            * The `HttpException` instance will be passed to this view when using the abort function helper (`abort(403, 'Unauthorized action.');`) and will be available in the view as `$exception`.
                * `<h2>{{ $exception->getMessage() }}</h2>`
        * You can publish the Laravel error pages and then edit them to fit your needs.
            * `php artisan vendor:publish --tag=laravel-errors`
    * When dealing with a `JSON` strings you can convert them to an array by using the PHP function `json_decode()`
        * `$meta = json_decode($post->meta, true);`
            * The second argument is needed to actually convert it to an associative array. If you don't provide it, then `json_decode` will return the data in an instance of `StdClass` object.
    * Custom Facade Classes
        * There may be times when you need to use a custom class but you want to access it like a Laravel Facade, where the methods in the class are static. e.g. In [Build A Laravel App with TDD: Improve Test Arrangements with Factory Classes](https://laracasts.com/series/build-a-laravel-app-with-tdd/episodes/17), Jeff uses a `ProjectFactory` classes to handle all the setup for his `ProjectTasksTest` tests. You can access the class by pulling it out of the service container or by accessing it as a facade.
            * `app(ProjectFactory::class)->withTasks(1)->create(); //Pulls from service container and then the withTasks and create methods can be accessed as normal.`
                * `ProjectFactory` needs to be imported to do this. `use Tests\Setup\ProjectFactory;`
            * `ProjectFactory::withTasks(1)->create();`
                * Since the methods with `ProjectFactory` are not static, this won't work unless you import the class like below
                    * `use Facades\Tests\Setup\ProjectFactory;`
                    * If you include `Facades\` before a class then Laravel will automatically import it as if it were a static class or normal facade.
    * When running `PHPUnit` tests, you may encounter an apostrophy is created while using `$faker->name`, `$fake->sentence`, etc. Since blade templates escape strings, the apostrophy will be converted to `&#039;`. Use the special e() helper function to escape the string in your tests as well.
        ```php
        $this->get(route('clients.index'))
            ->assertSee(
                e($client->name)
            );
        ```
* ### Webpack & Mix
    * Use the `extract` method to make webpack create a separate js file for all your vendor js and keep your js separate.
        * `mix.js('resources/js/app.js', 'public/js').extract();`
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