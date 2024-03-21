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
## Hooks - [Docs](https://pestphp.com/docs/hooks)
* These are hooks that can be used to tap into certain points of time during a test. Each hook method accepts a closure to run when that hook is encountered.
    * `beforeEach()` - Executes before every test in the current file. Can be used to initialize properties that utilized across all tests
    * `afterEach()` - Executes after every test in the current file. Can be used to perform a cleanup like resetting a userRespository.
    * `beforeAll()` - Executes before any test in the current file is run. The `$this` variable is not available since there is no instance of the test class.
    * `afterAll()` - Executes after all the tests in the current file have run. The `$this` variable is also not available since there is not longer a test instance.
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
## Test Filtering
* To execute a test, use the command `pest` or `.vendor/bin/pest` if the previous doesn’t work.
* Options that can be added to the `pest` command to filter tests or perform a certain task.
* `—-bail` - stops execution on first error
* `—-dirty` - only runs tests that have uncommitted changes accordingly to Git.
* `—-filter “description”` - runs only test where the given text or regular expression matches the file name, test description, dataset, etc
* `—-group=integration,browser` - if you have grouping set up on the tests, you can execute on the given groups. More on [Grouping Test](https://pestphp.com/docs/grouping-tests)
* `—-exclude-group=integration,browser` - opposite of `—-group`
* `—-retry` - only runs tests that previously failed.
* `—-todo` - only executes tests marked as a todo (`todo()` method added to the `it()` method)
## Skipping Tests
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
## Optimizing Tests
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