# Pest Driven Laravel

### Notes on [Pest Driven Laravel](https://laracasts.com/series/pest-driven-laravel) from Laracasts and Notes directly from the Pest Docs on their site.

## Installation & Setup
* Docs and info about Pest is at (https://pestphp.com)
* Require Pest 
    ```bash
    Composer require pestphp/pest —dev —with-all-dependencies
    ```
* Require Laravel Plugin
    ```bash
    Composer require pestphp/pest-plugin-laravel —dev
    ```
* Install Pest
    ```bash
    php artisan pest:install
    ```
    This will install a new `Pest.php` file within the `tests` directory. This file is like a config file for Pest.
* `Pest.php` config file within the `tests` directory
    * Mainly used to specify the base test class to be used
    * Can also define any [Global Hooks](https://pestphp.com/docs/global-hooks) or [Custom Expectations](https://pestphp.com/docs/custom-expectations).
    * The `$this` variable is typically bound to `PHPUnit\Framework\TestCase` class. This can be changed if needed.
    * Any `public` or `protected` method that are within the `TestCase` class will be available in all of your tests
* phpunit.xml is still used since Pest runs on top of PHPUnit. Uncomment the 2 lines for the database. `DB_CONNECTION` and `DB_DATABASE`. These can be setup using a MySQL database or a SQLite database, which is the default.
* Install **Pest Extension** for VSCode called [Better Pest](https://marketplace.visualstudio.com/items?itemName=m1guelpf.better-pest) by _Miguel Piedrafita_
* Install **Snippets Extension** for Pest called [Pest Snippets](https://marketplace.visualstudio.com/items?itemName=dansysanalyst.pest-snippets) by _dansysanalyst_
## General Format and Usage
* A new test file doesn’t require any imports, namespace or classes to work. It simply uses a function called `it`. Assertion functions need to be imported as with the `get` function used below.
    ```php
    <?php 
    
    use function Pest\Laravel\get
    
    it(‘gives back a successful response from home page’, function() {
        get(route(‘home’))
            ->assertOk();
    });
    
    ?>
    ```
* To run a test
    ```bash 
    php artisan test
    ```
* If you are using the `RefreshDatabase` trait then it will also need to be added to each test. The `uses` method tells Pest that you want to use a trait.
    ```php 
    use Illuminate\Foundation\Testing\RefreshDatabase; 
    
    uses(RefreshDatabase::class);
    ```
## Assertions
* `assertOk()` - same as `assertStatus(200)`
* `assertSeeText(‘text’)` - checks to see text on page
* `assertDontSeeText(‘text’)` - checks to see if text is missing from page
* `assertSeeTextInOrder([‘text1’,’text2’]) - checks if text on page appears in the given order
## Expectation API - [Docs](https://pestphp.com/docs/expectations#match)
* Expectations are used to check if something is an expected value. `expect($value)` is used then any of the following can be tacked on to it the verify your expectation.
```php
toBe()
toBeArray()
toBeBetween()
toBeEmpty()
toBeTrue()
toBeTruthy()
toBeFalse()
toBeFalsy()
toBeGreaterThan()
toBeGreaterThanOrEqual()
toBeLessThan()
toBeLessThanOrEqual()
toContain($needles)
toContainEqual($needles)
toContainOnlyInstancesOf($class)
toHaveCount(int $count)
toHaveMethod(string $name)
toHaveMethods(iterable $names)
toHaveProperty(string $name, $value=null)
toHaveProperties(iterable $name)
toMatchArray($array)
toMatchObject($object)
toEqual()
toEqualCanonicalizing()
toEqualWithDelta()
toBeIn(array)
toBeInfinite()
toBeInstanceOf($class)
toBeBool()
toBeCallable()
toBeFile()
toBeFloat()
toBeInt()
toBeIterable()
toBeNumeric()
toBeDigits()
toBeObject()
toBeResource()
toBeScalar()
toBeString()
toBeJson()
toBeNan()
toBeNull()
toHaveKey(string $key)
toHaveKeys(array $keys)
toHaveLength(int $number)
toBeReadableDirectory()
toBeReadableFile()
toBeWritableDirectory()
toBeWritableFile()
toStartWith(string)
toThrow()
toEndWith()
toMatch(string)
toMatchConstraint()
toBeUppercase()
toBeLowercase()
toBeAlpha()
toBeAlphaNumeric()
toBeSnakeCase()
toBeKebabCase()
toBeCamelCase()
toBeStudlyCase()
toHaveSnakeCaseKeys()
toHaveKebabCaseKeys()
toHaveCamelCaseKeys()
toHaveStudlyCaseKeys()
toHaveSameSize()
toBeUrl()
toBeUuid()
```
* The following modifiers can also be used with the above expectations:
```php
and($value)
dd()
ddWhen() i.e. 
    expect([1, 2])->each(
        fn ($number) => $number->ddWhen(fn (int $number) =>     $number === 2) // 2
    );
ddUnless()
each()
json()
match()
not
ray()
sequence()
unless()
when()
```
## Hooks
### Local - [Docs](https://pestphp.com/docs/hooks)
* These are hooks that can be used to tap into certain points of time during a test. Each hook method accepts a closure to run when that hook is encountered.
    * `beforeEach()` - Executes before every test in the current file. Can be used to initialize properties that utilized across all tests
    * `afterEach()` - Executes after every test in the current file. Can be used to perform a cleanup like resetting a userRespository.
    * `beforeAll()` - Executes before any test in the current file is run. The `$this` variable is not available since there is no instance of the test class.
    * `afterAll()` - Executes after all the tests in the current file have run. The `$this` variable is also not available since there is not longer a test instance.
### Global - [Docs](https://pestphp.com/docs/global-hooks)
* Global hooks are the same as the local hooks except they are configured in the `Pest.php` config file.
    ```php
    uses(TestCase::class)->beforeAll(function () {
        // Runs before each file...
    })->beforeEach(function () {
        // Runs before each test...
    })->afterEach(function () {
        // Runs after each test...
    })->afterAll(function () {
        // Runs after each file...
    })->group('integration')->in('Feature');
    ```
* Any before* hooks defined in the Pest.php configuration file will be executed prior to hooks defined in individual test files. Similarly, any after* hooks specified in the Pest.php configuration file will be executed after any hooks defined in individual test files.
## Datasets - [Docs](https://pestphp.com/docs/datasets)
### General
* Used to define an array of test data where the same test will be run for each item in the set. Pest will add test descriptions to each test run for the dataset so you will have a better understanding of where an issue may be.
* After your `it` method you can add `->with` to add your dataset.
    ```php
    it('has emails', function (string $name, string $email) {
        expect($email)->not->toBeEmpty();
    })->with([
        ['Nuno', 'enunomaduro@gmail.com'],
        ['Other', 'other@example.com']
    ]);
    ```
* If a key is provided in the array then Pest will add the key as a label in the test results
    ```php
    it('has emails', function (string $email) {
        expect($email)->not->toBeEmpty();
    })->with([
        'james' => 'james@laravel.com',
        'taylor' => 'taylor@laravel.com',
    ]);
    ```
### Bound Datasets
* This can be used when you want to obtain a dataset after the `beforeEach()` method of your tests.
    ```php
    it('can generate the full name of a user', function (User $user) {
        expect($user->full_name)->toBe("{$user->first_name} {$user->last_name}");
    })->with([
        fn() => User::factory()->create(['first_name' => 'Nuno', 'last_name' => 'Maduro']),
        fn() => User::factory()->create(['first_name' => 'Luke', 'last_name' => 'Downing']),
        fn() => User::factory()->create(['first_name' => 'Freek', 'last_name' => 'Van Der Herten']),
    ]);
    ```
* If you want to bind a single argument to the test case then it must be fully typed in the `it|test` function arguments.
    ```php
    it('can generate the full name of a user', function (User $user, $fullName) {
        expect($user->full_name)->toBe($fullName);
    })->with([
        [fn() => User::factory()->create(['first_name' => 'Nuno', 'last_name' => 'Maduro']), 'Nuno Maduro'],
        [fn() => User::factory()->create(['first_name' => 'Luke', 'last_name' => 'Downing']), 'Luke Downing'],
        [fn() => User::factory()->create(['first_name' => 'Freek', 'last_name' => 'Van Der Herten']), 'Freek Van Der Herten'],
    ]);
    ```
### Sharing Datasets
* You can store a dataset in a separate file so your tests don’t become too cluttered. The dataset needs to be stored in the `tests/Datasets` directory and the capitalized name of the set needs to be used for the file name. I.e. `Emails.php` when the dataset is called `emails`
    ```php
    // tests/Unit/ExampleTest.php...
    it('has emails', function (string $email) {
        expect($email)->not->toBeEmpty();
    })->with('emails');
 
    // tests/Datasets/Emails.php...
    dataset('emails', [
        'enunomaduro@gmail.com',
        'other@example.com'
    ]);
    ```
### Scoped Datasets
* These can be used when a dataset only pertains to a specific feature or set of folders. Instead of adding the dataset within the `Datasets` folder, you can use `Datasets.php file within the relevant folder requiring the dataset.
    ```php
    // tests/Feature/Products/ExampleTest.php
    it('has products', function (string $product) {
        expect($product)->not->toBeEmpty();
    })->with('products');
 
    // tests/Feature/Products/Datasets.php
    dataset('products', [
        'egg',
        'milk'
    ]);
    ```
### Combining Datasets
* You can create complex datasets by combining both inline and shared datasets. In the following example, we verify that all of the specified businesses are closed on each of the provided weekdays.
    ```php
    dataset('days_of_the_week', [
        'Saturday',
        'Sunday',
    ]);
 
    test('business is closed on day', function(string $business, string $day) {
        expect(new $business)->isClosed($day)->toBeTrue();
    })->with([
        Office::class,
        Bank::class,
        School::class
    ])->with('days_of_the_week');
    ```
### Repeating Test
* After the `it` method, you can add on a `->repeat(100)` to have a test repeat the given number of times.
## Exceptions - [Docs](https://pestphp.com/docs/exceptions)
* If you want to test for a certain exception message you can use the `throws()` method. This method can receive the expected exception class along with an expected message.
    ```php
    it('throws exception', function () {
        throw new Exception('Something happened.');
    })->throws(Exception::class, 'Something happened.');
    ```
* If the exception class is irrelevant then you can leave that out and just provide the expected message to the `throws()` method.
* You can use the `throwsIf()` method to use a conditional to verify the exception.
    ```php
    it('throws exception', function () {
        //
    })->throwsIf(fn() => DB::getDriverName() === 'mysql', Exception::class, 'MySQL is not supported.');
    ```
* The `throwsUnless()` method can be used to return false unless the given condition is met. It’s the opposite of `throwsIf()`.
* If you want to check for an exception on a given expectation you can use the `toThrow()` method.
    ```php
    it('throws exception', function () {
        expect(fn() => throw new Exception('Something happened.'))->toThrow(Exception::class);
    });
    ```
* To verify that no exception is thrown you can use the `throwsNoException()` method added to the `it` method.
    ```php
    it('throws no exceptions', function () {
        $result = 1 + 1;
    })->throwsNoExceptions();
    ```
* The `fail()` method can be used inside the closure of the `it()` method to make a test fail. A custom message can also be passed to the `fail()` method.
* The `fails()` method can be added after the `it()` method when you expect the test to fail.
    ```php
    it('fails', function () {
        throw new Exception('Something happened.');
    })->fails('Something went wrong.');
    ```
## Test Filtering - [Docs](https://pestphp.com/docs/filtering-tests)
* To execute a test, use the command `pest` or `.vendor/bin/pest` if the previous doesn’t work.
* Options that can be added to the `pest` command to filter tests or perform a certain task.
* `—-bail` - stops execution on first error
* `—-dirty` - only runs tests that have uncommitted changes accordingly to Git.
* `—-filter “description”` - runs only test where the given text or regular expression matches the file name, test description, dataset, etc
* `—-group=integration,browser` - if you have grouping set up on the tests, you can execute on the given groups. More on [Grouping Test](https://pestphp.com/docs/grouping-tests)
* `—-exclude-group=integration,browser` - opposite of `—-group`
* `—-retry` - only runs tests that previously failed.
* `—-todo` - only executes tests marked as a todo (`todo()` method added to the `it()` method)
## Skipping Tests - [Docs](https://pestphp.com/docs/skipping-tests)
* You can use the `skip()` method, added on to the `it()` method, to skip a certain test. This will create a yellow `WARN` message in the test results.
* The `skip()` method can receive text, condition followed by text or a closure.
    ```php
    it('has home', function () {
        //
    })->skip('temporarily unavailable');
    
    it('has home', function () {
        //
    })->skip($condition == true, 'temporarily unavailable');
    
    it('has home', function () {
        //
    })->skip(fn () => DB::getDriverName() !== 'mysql', 'db driver not supported');
    ```
* You can skip a test based on operating system by using the `skipOnWindows()`, `skipOnMac()` and `skipOnLinux()` methods.
* You can also do the opposite by using the `onlyOnWindows()`, `onlyOnMac()` and `onlyOnLinux()` methods.
* You can skip a test based on certain versions of PHP by using the `skipOnPhp(‘>=8.0.0’)` method. Valid operators are `=`,`>=`, `>`, `<`, `<=`
## Creating Todos
* You can mark a test as a todo by adding the `todo()` method onto the end of the `it()` method. This will display a blue `TODO` in the test results.
* You can then execute only the test marked as todo by using the `—-todo` option flag when you execute the tests.
## Optimizing Tests - [Docs](https://pestphp.com/docs/optimizing-tests)
### Parallel Testing
* By default all tests are executed in a linear fashion in a single process. By using the `—-parallel` option flag, all the tests are executed in parallel mode using multiple processes and is up to 80% faster.
* You can limit the number of processes parallel mode uses by passing the `—-processes=10` flag along with `—-parallel`. 
    ```bash
    .vendor/bin/pest —-parallel —-processes=10
    ```
* The following points need to be considered when using parallel mode:
    1. Database resources may not be shared between tests: Each test should be isolated and independent from other tests.
    2. Test order may not be guaranteed: Tests should not rely on any specific order of execution.
    3. Tests may be affected by race conditions: Race conditions can occur when multiple processes or threads are accessing shared resources. Make sure to design your tests to handle potential race conditions and avoid them whenever possible.    
### Profiling
* You can use this mode to find slow running tests. The execution time is included in the test results. Use the `—-profile` options flag to run in profile mode.
### Compact Printer
* To make the test results appear in a more compact fashion you can use the `—-compact` options flag. Each passing test will appear as a `.` and each failing test will appear with a red `x`. It will still tell you the details about any failing tests.
## Grouping Tests - [Docs](https://pestphp.com/docs/grouping-tests)
* You can assign the tests located in a certain directory to a group name so they can all be executed together. This is good when you may have a group of slow running tests.
* One use of this is to add the group to the `Pest.php` config file.
    ```php
    uses(TestCase::class)
        ->group('feature')
        ->in('Feature');
    ```
* You can then execute the group by using the `—-group` option flag on the `pest` command.
    ```bash
    ./vendor/bin/pest --group=feature
    ```
* You can also add the `group()` method to the end of the `it()` test function.
    ```php
    // single group
    it('has home', function () {
        //
    })->group('feature');
    
    // multiple groups
    it('has home', function () {
        //
    })->group('feature', 'browser');
    ```
* If you want the entire test file to be part of a group you can use the `uses()` method along with `group()`.
    ```php
    uses()->group('feature');
 
    it('has home', function () {
        //
    });
    ```
## Mocking - [Docs](https://pestphp.com/docs/mocking)
* Pest recommends using the [Mockery](https://github.com/mockery/mockery/) library but I’m not sure what comes installed with Laravel.
    * Install `Mockery`
        ```bash
        composer require mockery/mockery --dev
        ```
* To create a mock of a class, use the `Mockery::mock()` method and then specify the method you expect to receive by using the `shouldReceive()` method.
    ```php
    use App\Repositories\BookRepository;
    use Mockery;
 
    it('may buy a book', function () {
        $client = Mockery::mock(PaymentClient::class);
        $client->shouldReceive('post');
 
        $books = new BookRepository($client);
        $books->buy(); // The API is not actually invoked since `$client->post()` has been mocked...
    });
    ```
* You can also specify specific arguments that should be used the the `shouldReceive()` method by using the `with()` method.
    ```php
    $client->shouldReceive('post')
        ->with($firstArgument, $secondArgument);
    ```
* To match any argument, you can use the `Mockery::all()` method in place of any of the provided arguments.
    ```php
    $client->shouldReceive('post')
        ->with($firstArgument, Mockery::any());
    ```
* To tell Mockery what should be returned from the mocked method, you can use the `andReturn()` method
    ```php
    $client->shouldReceive('post')->andReturn('post response');
    ```
* Multiple values can be returned as well
    ```php
    $client->shouldReceive('post')->andReturn(1, 2);
 
    $client->post(); // int(1)
    $client->post(); // int(2)
    ```
* If you need to calculate what is returned you can use the `andReturnUsing()` method
    ```php
    $mock->shouldReceive('post')
        ->andReturnUsing(
            fn () => 1,
            fn () => 2,
        );
    ```
* If you want to return an exception you can use the `andThrow()` method
    ```php
    $client->shouldReceive('post')->andThrow(new Exception);
    ```
* You can also specify how many times a method should be invoked:
    ```php
    $mock->shouldReceive('post')->once();
    $mock->shouldReceive('put')->twice();
    $mock->shouldReceive('delete')->times(3);
    // ...
    ```
* The `atLeast()` and `atMost()` methods can be used to set limits on how many times a method should be invoked.
    ```php
    $mock->shouldReceive('delete')->atLeast()->times(3);
    $mock->shouldReceive('delete')->atMost()->times(3);
    ```
## Plugins - [Docs](https://pestphp.com/docs/plugins)
Some official and community plugins used to extend Pest:
* Faker - [Source](https://github.com/pestphp/pest-plugin-faker) [Docs](https://fakerphp.github.io/)
    * Install
        ```php
        composer require pestphp/pest-plugin-faker --dev
        ```
    * Used to generate fake data
        ```php
        use function Pest\Faker\fake;
 
        it('generates a name', function () {
            $name = fake()->name; // random name...
 
            //
        });
        ```
* Laravel - [Source](https://github.com/pestphp/pest-plugin-laravel) [Docs](https://laravel.com/docs/10.x/testing)
    * Install
        ```bash
        composer require pestphp/pest-plugin-laravel --dev
        ```
    * Artisan commands now available
        ```php
        // Create a new test in the tests/Feature directory
        php artisan pest:test UsersTest
        
        // Create a new test in the tests/Unit directory
        php artisan pest:test UsersTest --unit
        
        // Create a new Dataset in the tests/Datasets directory 
        php artisan pest:dataset Emails
        ```
    * You can bypass using the $this variable in the test by importing the corresponding function being used. In this case it is `get()`
        ```php
        // Using the $this variable as you normally would
        it('has a welcome page', function () {
            $this->get('/')->assertStatus(200);
        });
        
        // Now the function is imported and $this is no longer needed
        use function Pest\Laravel\{get};
 
        it('has a welcome page', function () {
            get('/')->assertStatus(200);
            // same as $this->get('/')...
        });
        ```
    * Another example of not using $this with the actingAs method
        ```php
        use App\Models\User;
        use function Pest\Laravel\{actingAs};
 
        test('authenticated user can access the dashboard', function () {
            $user = User::factory()->create();
 
            actingAs($user)->get('/dashboard')
            ->assertStatus(200);
        });
        ```
* Livewire - [Source](https://github.com/pestphp/pest-plugin-livewire)
    * Install
        ```bash
        composer require pestphp/pest-plugin-livewire --dev
        ```
    * Now you can use the Livewire namespaced functions to access your Livewire components
        ```php
        use function Pest\Livewire\livewire;
 
        it('can be incremented', function () {
            livewire(Counter::class)
                ->call('increment')
                ->assertSee(1);
        });
 
        it('can be decremented', function () {
            livewire(Counter::class)
                ->call('decrement')
                ->assertSee(-1);
        });
        ```
* Watch - [Source](https://github.com/pestphp/pest-plugin-watch)
    * Install
        ```bash
        composer require pestphp/pest-plugin-watch --dev
        ```
    * [fswatch](https://github.com/emcrisostomo/fswatch#getting-fswatch) also need to be installed so Pest can monitor changes to files
    * When you run a test with `pest —-watch`, Pest can keep track of and automatically rerun a test when the file changes.
    * By default the plugin monitors the following directories: `tests/`, `app/` and `src/`.
    * To specify which directories to watch you can supply a list of comma separated directories (relative to the site root)
        ```bash
        pest --watch=app,routes,tests
        ```
## Architecture Testing - [Docs](https://pestphp.com/docs/arch-testing#expect-toBeClasses)
If you want to test whether or not your code adheres to a certain set of architectural rules, you can use Architecture Testing to specify your expectations. I.e. Make sure that your `Traits` or `Concerns` directory contains only files that are traits.
### Expectations
* The following are some expectations that can be run after the `arch()` method:
    ```php
    toBeAbstract()
    toBeClasses()
    toBeEnums()
    toBeIntBackedEnums()
    toBeInterfaces()
    toBeInvokable()
    toBeFinal()
    toBeReadonly()
    toBeStringBackedEnums()
    toBeTraits()
    toBeUsed()
    toBeUsedIn()
    toExtend()
    toExtendNothing()
    toImplement()
    toImplementNothing()
    toHaveAttribute()
    toHaveMethod()
    toHavePrefix()
    toHaveSuffix()
    toHaveConstructor()
    toHaveDestructor()
    toOnlyImplement()
    toOnlyUse()
    toOnlyBeUsedIn()
    toUse()
    toUseNothing()
    toUseStrictTypes()
    ```
* Check the docs are further explanation on how these expectations work.
### Modifiers
* If you want to exclude certain namespaces or types of files, modifiers can be used:
    ```php
    classes()
    enums()
    ignoring()
    interfaces()
    traits()
    ```
## Test Dependency - [Docs](https://pestphp.com/docs/test-dependencies)
* If a test depends on another test to complete before it should run then you can use the `depends()` method to specify that dependency.
    ```php
    test('parent', function () {
        expect(true)->toBeTrue();
     
        $this->assertTrue(true);
    });
     
    test('child', function () {
        expect(false)->toBeFalse();
    })->depends('parent');
    ```
* It is important to remember that the `it()` function adds the word "it" onto the beginning of the test name. You need to make sure to have the correct name specified in the `depends()` method.
    ```php
    it('is the parent', function () {
        expect(true)->toBeTrue();
    });
     
    test('child', function () {
        expect(false)->toBeFalse();
    })->depends('it is the parent');
    ```
* The parent test can also pass along an argument to the child class
    ```php
    test('parent', function () {
        expect(true)->toBeTrue();
     
        return 'from parent';
    });
     
    test('child', function ($parentValue) {
        var_dump($parentValue); // from parent
     
        expect($parentValue)->toBe('from parent');
    })->depends('parent');
    ```
* A test could also have multiple dependencies.
    ```php
    test('a', function () {
        expect(true)->toBeTrue();
     
        return 'a';
    });
     
    test('b', function () {
        expect(true)->toBeTrue();
     
        return 'b';
    });
     
    test('c', function () {
        expect(true)->toBeTrue();
     
        return 'c';
    });
     
    test('d', function ($testA, $testC, $testB) {
        var_dump($testA); // a
        var_dump($testB); // b
        var_dump($testC); // c
    })->depends('a', 'b', 'c');
    ```
## Migrating from PHPUnit - [Docs](https://pestphp.com/docs/migrating-from-phpunit-guide)
The `pestphp/pest-plugin-drift` package can be used to migrate PHPUnit tests in your existing project over to Pest.
* Install
    ```bash
    composer require pestphp/pest-plugin-drift --dev
    ```
* Use the `--drift` option flag on the `pest` command to execute the migration
    ```bash
    ./vendor/bin/pest --drift
    ```
* Before and after migration
    ```php   
    // PHPUnit test 
    namespace Tests\Unit;
     
    use PHPUnit\Framework\TestCase;
     
    class ExampleTest extends TestCase
    {
        public function test_that_true_is_true(): void
        {
            $this->assertTrue(true);
        }
    }
    
    // After migration
    test('true is true', function () {
        expect(true)->toBeTrue();
    });
    ```
* The output will contain a summary of the conversion process, as well as a list of the files that were converted.
## Custom Helpers - [Docs](https://pestphp.com/docs/custom-helpers)
* Helpers should be simple functions and can defined in the test file itself, the `tests/Pest.php` file, the `tests/Helpers.php` file or a separate file with the `tests/Helpers` directory. Helpers in any of these locations will automatically be added to Pest.
* In situations where your helper needs access to the underlying test class, instead of using the `$test` variable, you should use the `test()` method.
    ```php
    use App\Models\User;
    use Tests\TestCase;
     
    function asAdmin(): TestCase
    {
        $user = User::factory()->create([
            'admin' => true,
        ]);
     
        return test()->actingAs($user);
    }
     
    it('can manage users', function () {
        asAdmin()->get('/users')->assertOk();
    })
    ```
* If you do want to use the `$this` variable in your tests then the helper should be defined within the base test class.
    ```php
    use App\Clients\PaymentClient;
    use PHPUnit\Framework\TestCase as BaseTestCase;
    use Mockery;
     
    // tests/TestCase.php
    class TestCase extends BaseTestCase
    {
        public function mockPayments(): void
        {
            $client = Mockery::mock(PaymentClient::class);
    
            return $client;
        }
    }
     
    // tests/Pest.php
    uses(TestCase::class)->in('Features');
     
    // tests/Features/PaymentsTest.php
    it('may buy a book', function () {
        $client = $this->mockPayments();
    })
    ```
## Higher Order Testing - [Docs](https://pestphp.com/docs/higher-order-testing)
* The idea with Higher Order Testing is that you want to make the tests as simple as possible by eliminating unneeded closures.
    ```php
    // Old way
    it('works', function () {
        $this->get('/')
            ->assertStatus(200);
    });
	
    // Higher Order way
    it('works')
        ->get('/')
        ->assertStatus(200);
	
    // Old way
    it('has a name', function () {
        $user = User::create([
            'name' => 'Nuno Maduro',
        ]);
     
        expect($user->name)->toBe('Nuno Maduro');
    });

    // Higher Order way
    it('has a name')
        ->expect(fn () => User::create(['name' => 'Nuno Maduro'])->name)
        ->toBe('Nuno Maduro');
    ```
* It is crucial to use lazy evaluation for the expectation value by passing a closure to the `expect()` method. This ensures that the expected value is created only when the test runs and not before. If you need to make assertions on an object that requires lazy evaluation at runtime, you can use the `defer()` method.
    ```php
    // assertion will be called on the result of the closesure passed to defer()
    it('creates admins')
        ->defer(fn () => $this->artisan('user:create --admin'))
        ->assertDatabaseHas('users', ['id' => 1]);
    ```
### Higher Order Expectations
* With Higher Order Expectations, you can perform expectations directly on the `properties` or `methods` of the expectation `$value`.
    ```php
    // Old way
    expect($user->name)->toBe('Nuno');
    expect($user->surname)->toBe('Maduro');
    expect($user->addTitle('Mr.'))->toBe('Mr. Nuno Maduro');
    
    // Higher Order Expectations
    expect($user)
        ->name->toBe(‘Nuno’)
        ->surname->toBe(‘Maduro’)
        ->addTitle(‘Mr.’)->toBe(‘Mr. Nuno Maduro’);
    ```
* When working with arrays, you may also access the $value array keys and perform expectations on them.
    ```php
    expect(['name' => 'Nuno', 'projects' => ['Pest', 'OpenAI', 'Laravel Zero']])
        ->name->toBe('Nuno’)
        ->projects->toHaveCount(3)
        ->each->toBeString();
     
    expect([‘Dan’, ‘Luke’, ‘Nuno’])
        ->{0}->toBe(‘Dan’);
    ```
* You can even create Higher Order Expectations within closures
    ```php
    expect(['name' => 'Nuno', 'projects' => ['Pest', 'OpenAI', 'Laravel Zero']])
        ->name->toBe(‘Nuno’)
        ->projects->toHaveCount(3)
        ->sequence(
            fn ($project) => $project->toBe(‘Pest’),
            fn ($project) => $project->toBe(‘OpenAI’),
            fn ($project) => $project->toBe(‘Laravel Zero’),
        );
    ```
* With Scoped Higher Order Expectations, you may use the method scoped() and a closure to gain access and lock an expectation in to a certain level in the chain. This is very useful for Laravel Eloquent models, where you want to check properties of a child relation.
    ```php
    expect($user)
    ->name->toBe(‘Nuno’)
    ->email->toBe(‘enunomaduro@gmail.com’)
    ->address()->scoped(fn ($address) => $address
        ->line1->toBe(‘1 Pest Street’)
        ->city->toBe(‘Lisbon’)
        ->country->toBe(‘Portugal’)
    );
    ```
	




