# Laravel Notes - User Authentication


## Notes from the Laracast series about Laravel
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
    * You can disable user registration functionality by passing `['register' => false]` to the `Auth::routes` method.
        ```php
        Auth::routes(['register' => false]);
        ```
