# Laravel Notes - General


## Notes from the Laracast series about Laravel
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
