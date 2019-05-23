# PHPUnit Testing Notes

## Notes from the Laracast series Testing Laravel

* ### If the `phpunit` command is not working
    * type the entire path to the binary: `vendor/bin/phpunit`
    * try updating composer dependancies.
        * `composer global update`
        * then reinstall phpunit with `composer global require phpunit/phpunit`
* ### A well written test should read like documentation.
* ### Terms:
    * `TDD` - Test Driven Development. Where tests are written before the actual functionality.
    * `BDD` - Behavior Driven Development. It is TDD but done right. It focuses more on the conversational aspects of a feature as opposed to actual assertion. Below is an example based on a conversation with a client that currently has a blog but would like to add a new feature where they can schedule the time a post goes live. This is a valid Feature Request listed in the Gherkin Syntax.
        ```
		Feature: Post Scheduling
			In order to build up the schedule (WHY)
			As an administrator (WHO)
			I need to schedule posts for future dates (WHAT)
			
			Scenario: scheduling a post for tomorrow
				Given i have a post "Old Post"
				And I have a scheduled post "Scheduled Post"
				When I view all posts
				Then the "Scheduled Post" post should not be in the list
        ```

    * `Unit testing` is a test on a very zoomed in level. It only tests a certain piece of functionality.
    * `Acceptance` or `Function` testing is a more macro style test. They test multiple parts of an application and is usually much slower than a unit test. Tests that go through a browser.
    * `Integration` testing is when you are testing database functionality within your app and tests multiple parts of an application.
        * `High Level Integration Test` is a `Functional` or `Acceptance Test` where you test from the outside in. i.e. You go to a page and perform a search, you would expect to see certain results.
        * `Low Level Integration Test` is more of a macro test on a certain part of the functionality.
    * `A Regression test` is where you have discovered a bug, then you write a test that recreates the bug, fixes it, then reruns the test to make sure it is working properly.
    * `Mocks` are objects pre-programmed with expectations which form a specification of the calls they are expected to receive. When doing a unit test, mock objects can simulate the behavior of complex, real objects and are therefore useful when a real object is impractical or impossible to incorporate into a unit test. The method is not actually being executed, a mock just says that a certain method will be called with any given data. A mock enforces that a message or data is sent to certain collaborators.
    * A stub is used in unit testing. `Stubs` provide canned answers to calls made during the test, usually not responding at all to anything outside what's programmed in for the test. 
    * `Dummy` objects are passed around but never actually used. Usually they are just used to fill parameter lists.
    * `Fake` objects actually have working implementations, but usually take some shortcut which makes them not suitable for production (an in memory database is a good example).
    * `Spies` are stubs that also record some information based on how they were called. One form of this might be an email service that records how many messages it was sent.
    * `Test Double` or "`Doubles`", are the generic terms for any kind of pretend object used in place of a real object for testing purposes. i.e. `Mocks`, `Stubs`, `Fakes`, `Dummy`
    * A `Factory` is an object or tool that gives us an easy way to populate objects or models with valid data for the purpose of testing.
* ### Directory Structure & Naming Convention Suggestions:
    * Add directories within the testing directory. One called `acceptance`, one called `unit` and one called `integration`. Each directory houses the specific test type for the application. Within the `integration` directory you should also have a `models` directory to handle all the model specific testing.
    * `Filename` could be the class name followed by Test. i.e. For a Product class the file would be called `ProductTest.php`
        * The test classes will either extend the `PHPUnit_Framework_TestCase` class (for when you are doing primarily unit testing and Laravel isn't really involved) or the Laravel `TestCase` class (for when you are doing integration style testing and Laravel is more involved).
    * For testing methods within the testing class:
        * the function name should begin with the word "`test`" so PHPUnit will know it is a test method that should be run
        * or you can use `/** @test */` before the method name. This allows for much more readable method names like `a_product_has_a_name()`
    * A `setUp` method can be added to a test class to perform certain actions before any testing takes place. For example, you can new up a new product and assign the object to a protected variable within the class and then can be reference in other methods. That way you won't have to new up a new product for every test method that requires a new product.
        * Important note: When using `TestCase`, you need to include `parent::setUp();` within your `setUp` method.
    * When setting up the test logic, use the following syntax
        * `Given` - Given this set of data
        * `When` -  When this action is executed
        * `Then` - Then I expect this result
* ### Miscellaneous console commands:
    * To create a new test class you can use `php artisan make:test ProductTest`. This will create a new test class within the testing directory
    * `phpunit --filter method_name` will just test the specified method within all the tests
    * `phpunit tests/unit` to run just your unit tests
    * `phpunit tests/unit/ProductTest.php` just to run the tests within the ProductTest class
* ### Miscellaneous testing methods. `$this` followed by the method call
    * `->visit('/')` checks to see if page exists
    * `->click('Button Text')` actually clicks a button or link with the id, name or body with requested text
    * `->see('test text')` checks to see if the requested text exists somewhere on the page
    * `->seePageIs('/testPage')` checks to see if the current uri is the requested page
    * `->assertEquals('testing name', $method_on_object->getName())` Checks to see if 1st param equals 2nd
    * `->assertCount(2, $productsArray)` checks to see if the 1st param equals the count of the 2nd
    * `->setExpectedException('Exception')` used to assert whether or not an exception was thrown by the app.
        * This assertion goes before the actual action that will throw the exception.
    * `->actingAs($user)` This requires the `TestCase` class. This tells the test that they are currently acting as a specific user. Can be used when you want to check that a logged in user wants to create a post.
    * `->seeInDatabase()` Checks to see if the array of values (2nd param) is in the provided database table (1st param)
		```
        $this->seeInDatabase('likes', [
			'user_id' => $user->id,
			'likeable_id' => $post->id //polymorphic relationship where many different things can be liked in the app and is based off of the likeable_type
			'likeable_type' => get_class($post) //returns App\\Post so you'd know that the thing being liked is a post
		]);
        ```
    * `->notSeeInDatabase()` opposite of seeInDatabase
    * `->assertDatabaseHas()` First argument is the table name and the second is an array of expected values. i.e. `'users', ['name' => 'JohnDoe']`
    * `->assertTrue()` or `->assertFalse` Checks to see the correct boolean response is received.
    * `->post('endpoint', dataArray)` Sends a post request to the "endpoint" with the provided dataArray
    * `->assertSessionHasErrors()` Checks to see if the laravel errors variable is set in the session
    * `->get()` Gets a URI
    * `->getJson()` Requests a URI to be in a JSON format
    * `->assertStatus()` Looks for a specific response code that is passed to the function
* ### Using Mock & Spy Example
    * We have a `Newsletter.php` class that has a `subscribeTo` method that makes an API call and is accessed by hitting the following route in the `routes/web.php` file
        ```
        Route::get('/subscribe', function(Newsletter $newsletter) { 
            $newsletter->subscribeTo('members', auth()->user()); 
            return 'done'; 
        }
        ```

        Notice that Laravel will automatically resolve the `Newsletter` class out of the `Service Container` when this route is hit
    * You can now write a test to check all the functionality of Newsletter without making the API call
        ```
        function it_subscribes_a_user_to_a_mailing_list() {
            $user = factory(User::class)->create();
            $mock = Mockery::mock(Newsletter::class, function($mock) use ($user) {
                //equivalent to saying: expect to have subscribeTo('members', $user) called
                $mock->shouldReceive('subscribeTo')->with('members', $user)->once(); 

                //the actual method can be called directly off of the Mockery class
                //or use $mock->shouldReceive()->subscribeTo('members', $user)->once(); 
            });

            //this will replace the normal Newsletter class 
            //that will be retrieved from the Service Container 
            //with the newly mocked version of the class
            $this->instance(Newsletter::class, $mock); 

            //you can use the app facade or $this since app is already part of TestCase
            // or app()->instance(Newsletter::class, $mock);

            $this->actingAs($user)->get('/subscribe');
        }
        ```

        As of Laravel 5.8 there is now a new `mock` method, so the above test can be simplified. The `mock` method handles declaring the `mock` as well as replacing the real class with the newly mocked class automatically
        ```
        function it_subscribes_a_user_to_a_mailing_list() {
            $user = factory(User::class)->create();
            $this->mock(Newsletter::class, function($mock) use ($user) {
                $mock->shouldReceive()->subscribeTo('members', $user)->once();
            });
            $this->actingAs($user)->get('/subscribe');
        }
        ```

        Also as of Laravel 5.8 you can write the above test using a Spy instead. A spy will evaluate after everything has run.
        ```
        function it_subscribes_a_user_to_a_mailing_list() {
            $user = factory(User::class)->create();
            $spy = $this->spy(Newsletter::class);
            $this->actingAs($user)->get('/subscribe');
            $spy->shouldReceive()->subscribeTo('members', $user);
        }
        ```
* ### The use of Model Factories when doing Integration Testing
    * Default factory file is located within the `database/factories/ModelFactory.php` file
        * In this file you would set up the various classes that will be using factories. See the example method for the `User` model within this file
    * Within your test method you can call a factory method on a class that has been setup with the `ModelFactory.php` file
        * `factory(Article::class, 2)->create()` will create 2 records using the faker config used in the `ModelFactory.php` file
        * `factory(Article::class)->create([ 'reads' => 10])` overrides the reads value setup in the factory
    * Miscellaneous Factory Methods. `factory('class')` followed by the method call:
        * `->create()` persists the record in the database
        * `->make()` creates an instance of the class provided but does not persist it to the database
        * `->raw()` create an array of the all the attributes for the class but does not create an instance or persist it to the database.
    * When doing Integration Testing it is a good idea to use database transactions so the database isn't filled with a bunch of faker data. This will remove any fake data after the test has completed.
        * The `RefreshDatabase` trait is now used instead of `DatabaseTransactions`;
        * import the `RefreshDatabase` class at the top of your document. `use Illuminate\Foundation\Testing\RefreshDatabase;`
        * use the `RefreshDatabase` class within your test class. `use RefreshDatabase;`
    * The format of a factory:
        ```
        $factory->define(App\Model::class, function(Faker $faker) {
            return [
                'field_name' => $faker->sentence, //or any other faker method. 
                'user_id' => function() { //see below for a different way to do this.
                    return factory(App\User::class)->create()->id;
                }
            ]
        });
        ```
        See <https://github.com/fzaninotto/Faker> for a full list of faker methods

        Instead of passing a closure for the '`user_id`' you can pass an instance of the factory class.
        ```
        'user_id' => factory(App\User::class) //The id will be taken automatically from the created model.
        ```
* ### Set up a testing database for use with all of your tests
    * Method 1
        * In the root level, open the `phpunit.xml` file
            * Find the `<php>` section that overrides various env variable values. (`<env name="">`)
            * Add a new env value to override the database connection to use the testing database connection instead.
                ```
                <env name="DB_CONNECTION" value="sqlite"/>
                <env name="DB_DATABASE" value=":memory:"/>
                ```
                If you are not going to use the sqlite db in memory, you'll need to create the db file by typing `touch database/testing.sqlite` in to the console.

        * *IMPORTANT!* Sqlite doesn't support foreign key restraints by default. So if your migrations contain foreign key restraints any test referencing that table may fail. To solve the issue, put the following statement at the start of your test method.
            * `DB::statement('PRAGMA foreign_keys=on');` Make sure to input the DB facade. `use Illuminate\Support\Facades\DB;`
            * or Put the statement in the `\Tests\TestCase.php` file. It needs to go in the `setUp` method. If `setUp` doesn't exists it should look like this:
                ```
                protected function setUp() { 
                    parent::setUp(); 
                    DB::statement('PRAGMA foreign_keys=on'); 
                }
                ```
                Make sure to import the facade as stated above.

    * Method 2 - **Not Used**
        * You can now create a `.env.testing` file
            * Copy the contents of the `.env` file here and change the database details to point to the testing database.
        * Within the `config/database.php` file 
            * Add a new database connection using which ever driver your production db is using.
                * In the connections array, there are all the connection drivers. Copy which ever one you are using and add `_testing` after the driver name.
                * If you are using the sqlite driver, then make sure the testing database exists in that directory (in console: `touch database/testing.sqlite`)
                * If you are using MySQL then makes sure that the `.env` file contains the db information for your testing database. As a side note, you will have separate `.env` files for your testing and production environments.
        * You will need to run all your migrations against the testing database. `run php artisan migrate --database sqlite_testing` or `mysql_testing` depending on the connection being used.
* ### Testing Artisan Console Commands
    * `$this->artisan('artisan:command')`
    * You can then tack on `->expectsQuestion` to test that console command is doing what you want.
        * `->expectsQuestion('You being asked something here.', 'A given answer here')` The first argument is the question and the second is the answer
        * `->expectsOutput('The message when the command is completed here.')` can be used to check that the command generated a specific output.
* ### Miscellaneous Info:
    * Exception handling is always on, which means that laravel will handle any thrown exception. Same as running `$this->withExceptionHandling();`. That call is no longer needed. If you want to handle an exception more directly, then you need to make sure that Laravel isn't using exception handling. `$this->withoutExceptionHandling();`
    * It is handy to add a utility function file, which could contain custom versions of `factory()->make()` and `factory()->create()`
        * Create a php file within `test/utilities/functions.php`
        * Add the file to the `autoloader` in the `composer.json` file
            * `"autoload-dev": { "files": [ "tests/utilities/functions.php" ] }`
        * `functions.php`
            ```
            function create($class, $attributes = [], $times = null)
            {
                return factory($class, $times)->create($attributes);
            }

            function make($class, $attributes = [], $times = null)
            {
                return factory($class, $times)->make($attributes);
            }

            function makeRaw($class, $attributes = [], $times = null)
            {
                return factory($class, $times)->raw($attributes);
            }

            function createRaw($class, $attributes = [], $times = null)
            {
                $f = factory($class, $times)->create($attributes);
                return $f->toArray();
            }
            ```
            You may need to run `composer dump-autoload` to re-register everything
    * Another handy utility function to use
        * In the tests/TestCase.php file
            ```
            protected function signIn($user = null)
            {
                $user = $user ?: factory('App\User')->create();

                $this->actingAs($user);

                return $this;
            }
            ```
            This can be used to log in a new user or can be passed an existing user instance to log in.
    * When running `PHPUnit` tests, you may encounter an apostrophy is created while using `$faker->name`, `$fake->sentence`, etc. Since blade templates escape strings, the apostrophy will be converted to `&#039;`. Use the special e() helper function to escape the string in your tests as well.
        ```
        $this->get(route('clients.index'))
            ->assertSee(
                e($client->name)
            );
        ```
* ### Laravel Testing Debugger
    * [Laravel Debug Bar](https://github.com/barryvdh/laravel-debugbar)
        * For Laravel <5.5
            * Type `composer require barryvdh/laravel-debugbar`
            * You can either add the following to the '`providers`' array in '`config/app.php`'
                * `Barryvdh\Debugbar\ServiceProvider::class,`
            * Or since it is only used during local development, you can add the following to '`app/Providers/appServiceProvider.php`' within the '`register`' method
                ```
                if ($this->app->isLocal()) { 
                    $this->app->register(Barryvdh\Debugbar\ServiceProvider::class);
                }
                ```
        * For Laravel >=5.5
            * type `composer require barryvdh/laravel-debugbar --dev`
            * make sure that `APP_DEBUG=true` in the `.env` file or the debug bar will not be visible.