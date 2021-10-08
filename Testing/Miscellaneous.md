# Miscellaneous Testing

## Laravel Factory Tips (October 2021)
#### The following notes are from the Laracast episode [Laravel Factory Tips](https://laracasts.com/series/andres-larabits/episodes/4)

* #### The `for` and `has` Methods
    * #### `for` Method
        This can be used when creating another model to use in a different model when there is a relationship being used. For example, you have a User model that has a `hasMany` relationship with a Post model. You can use the `for` method when creating the Post model to have Laravel automatically fill the required field when provided a User model.
        ```php
        $user = User::factory(['name' => 'Bob'])->create();

        Post::factory()->for($user)->create();
        ```
    * #### `has` Method
        This is the opposite of the `for` method and when using the example above, can be used to create posts when creating the User model.
        ```php
        $user = User::factory(['name' => 'Bob'])->has(Post::factory(3))->create();
        ```
* #### Using a Seeder
    Used to arrange the creating of certain models to be used in a test. If you have several models being created to be used in a test, you can create a seeder class to handle all of it for you. The seeder can then be accessed using the `seed` method within the test.
    ```php
    $this->seed(SomeSeederName::class);
    ```
    Don't forget to import the seeder class in the test, it is being used in.
* #### Using a Sequence When Creating a Model
    If you want to have the factory create a model with different values for a given property you can use sequencing to do this.
    ```php
    Post::factory(10)->state(new Sequence(
        ['pinned' => true],
        ['pinned' => false],
        ['pinned' => false],
    ))->create();
    ```
    This will create a post with the `pinned` property equal to `true` with every third model created.
    You can also use a `sequence` method and access the iteration of the factory.
    ```php
    Post::factory(10)
        ->sequence(fn ($sequence) => ['name' => 'Post ' . $sequence->index])
        ->create();
    ```
    This will create a post with the incremental sequence index being added to the post name on each iteration. i.e. `Post 1`, `Post 2`, `Post 3`, etc.
* #### Useful Faker Methods
    * optional : Can be used to say a nullable field can sometimes be null when creating the model. The method is passed a `$weight` value that states how often the field will have a value. The values of `$weight` are to be between `0.0` - `1.0` 
        ```php
        'bio' => $this->faker->optional($weight = 0.3)->sentence(12);
        ```
    * valid : Can be used to specify the valid values of a field. The method is passed a callback that evaluates something and returns a boolean to whether or not the value in the field is valid.
        ```php
        'experience' => $this->faker->valid(fn ($exp) => $exp < 100)->randomNumber(),
        ```
        This will only create a random number less than 100 for each model created.
## PHP Testing Jargon
#### The following notes are from the Laracast series [PHP Testing Jargon](https://laracasts.com/series/php-testing-jargon)

### Unit Testing
#### Using the setUp method
##### The following notes are from the Laracast episode [Setup the World](https://laracasts.com/series/php-testing-jargon/episodes/3)
When using the setUp method with in the test class you can removed duplicated code and assign something to a property on the class.
```php
protected TagParser $parser;

protected function setUp(): void
{
    $this->parser = new TagParser();
}
```
***IMPORTANT:*** You have to be explicit with the "Return Type" being used. This is done by using `: void` after the method name.

#### Data Providers
##### The following notes are from the Laracast episode [When to Reach for Data Providers](https://laracasts.com/series/php-testing-jargon/episodes/4)

Data Providers can be used to simplify code where the same assertion is being made but there are different data sets to test against.

In the episode the same assertion is being made in several test that performs the same function call but are using different data sets and expected results. 
```php
public function test_it_parses_a_single_tag()
{
    $result = $this->parser->parse("personal");
    $expected = ["personal"];

    $this->assertSame($expected, $result);
}

public function test_it_parses_a_comma_separated_list_of_tags()
{
    $result = $this->parser->parse("personal, money, family");
    $expected = ["personal", "money", "family"];

    $this->assertSame($expected, $result);
}
```
The function call `$result = $this->parser->parse(` is being used in the each test as well as the same assertion `$this->assertSame($expected, $result);`

This can be simplified by using one test that utilizes Data Providers.
```php
/**
 * @dataProvider tagsProvider
 */
public function test_it_parses_tags($input, $expected)
{
    $parser = new TagParser();

    $result = $parser->parse($input);

    $this->assertSame($expected, $result);
}
```
The new test `test_it_parses_tags` replaces all other tests that are using the previous duplicated assertions. ie. `test_it_parses_a_single_tag` and `test_it_parses_a_comma_separated_list_of_tags`
The actual Data Provider is being referenced in the comment tags of the test itself. `@dataProvider tagsProvider` is used to tell the test to use a function called `tagesProvider` to get the input data as well as the expected result when run against the `$parser->parse` function.
```php
public function tagsProvider()
{
    return [
        ["personal", ["personal"]],
        ["personal, money, family", ["personal", "money", "family"]],
        ["personal,money,family", ["personal", "money", "family"]],
        ["personal | money | family", ["personal", "money", "family"]],
        ["personal|money|family", ["personal", "money", "family"]],
        ["personal!money!family", ["personal", "money", "family"]],
    ];
}
```
Each element of the root array is a different assertion to be made. In the assertion array is the value to be used for the `$input` value and the second array is the expected result (`$expected`) to test against.

The only potential downside to using Data Providers is that you lose the description of what each dataset is testing for.

### Test Doubles
**Terminology - the following are all considered to be test doubles**

`dummy` - a generated object that contains no assertions and is only a placeholder used to instantiate something else.
`stub` - like a mock but has some generated data within it but no expectations or assertions are placed on it.
`mock` - like a stub but has expectations and assertions placed on it.

#### Dummy Classes
##### The following notes are from the Laracast episode [A Dummy is a Stand-in for the Real Thing](https://laracasts.com/series/php-testing-jargon/episodes/8)

If you are using an external api or gateway, you can use a dummy class instead of actually hitting the api for your tests. You would use this because making an api call to an external source can be taxing and take a long time to get a reply. You should still have seperate tests that just verify the functionality of the api.

One way to handle this is making sure that your gateway class is implementing an interface so you can jack in a fake or dummy gateway that also implements the same interface.

You would first have a real Gateway class that is the interface for any other gateway.
```php
\App\Gateway.php

interface Gateway
{
    public function create();
}

\App\StripeGateway.php

class StripeGateway implements Gateway
{
    public function create()
    {
        // performs the real Stripe HTTP request.
        var_dump('Slow HTTP request in progress.');
    }
}

\App\Subscription.php

class Subscription
{
    protected Gateway $gateway;

    public function __construct(Gateway $gateway)
    {
        $this->gateway = $gateway;
    }

    public function create(User $user)
    {
        // create the subscription through Stripe.
        $receipt = $this->gateway->create();
    }
}
\Tests\DummyGateway.php

use App\Gateway;

class DummyGateway implements Gateway
{
    public function create()
    {
        //fills with fake data like what would be returned from the api gateway call
    }
}

\Tests\SubscriptionTest.php

use App\Subscription;
use App\User;
use PHPUnit\Framework\TestCase;

class SubscriptionTest extends TestCase
{
    /** @test */
    function creating_a_subscription_marks_the_user_as_subscribed()
    {
        $gateway = new DummyGateway();

        $subscription = new Subscription($gateway);

        $user = new User("JohnDoe");

        $this->assertFalse($user->isSubscribed());

        $subscription->create($user);

        $this->assertTrue($user->isSubscribed());
    }
```

#### Create Mocks with PHPUnit
##### The following notes are from the Laracast episode [Let PHPUnit Create Your Test Doubles](https://laracasts.com/series/php-testing-jargon/episodes/9)

PHPUnit can accomplish the same thing as what we did with the dummy class example by using a mock. Instead of creating a dummy class like DummyGateway, you can simply just use `$gateway = $this->createMock(Gateway::class);` in the `SubscriptionTest` file to do the same thing `$gateway = new DummyGateway();`. This would remove the need for the DummyGateway class entirely.

The way the mock works in this example is that any method call to the gateway class would just return `null`. The only time that it is benefitial to use `createMock` is when it's just a placeholder for something else you are trying to instantiate. Like the `Subscription` class requires a `Gateway` when instantiated. As long as you are not expecting any data to be returned from the mocked gateway, then this is a simple and light way to do it. If you need the functionality where you expect data from the mock then a "stub" may be the best way to do it.

#### Stubs
##### The following notes are from the Laracast episode [Stubs Versus Mocks](https://laracasts.com/series/php-testing-jargon/episodes/10)

Sometimes when working with a mock (see previous heading on Mocks), you need it to actually return specific data when a method is called on it. Currently, when you use `$gateway = $this->createMock(Gateway::class);` any method call on `$gateway` will return `null`. However, you can create what you expect to be returned from a method call by using the `method` and `willReturn` methods on the mocked class. 
```php
$gateway = $this->createMock(Gateway::class);
$gateway->method('create')->willReturn('receipt-stub');
```

You can also use a stub but use a dummy class like in the [Dummy Classes](#) section above. You do this by creating a new class that also implements the same `Gateway` class, just like in the example but you could call the class `GatewayStub` instead of `DummyGateway`. Then inside the `create` method, on that class, you could return any data you want to test like `return 'receipt-stub';`. Make sure to update the test class with the correct stubbed gateway class. ie. in the `SubscriptionTest` class use `$subscription = new Subscription(new GatewayStub());` when instantiating the `$subscription` variable.

You can also just use the mock as outlined earlier, which is usually preferred.

Using the previous subscription example, we want to create a test that checks to make sure that an email is sent to the user when the `create` method is called on the `Gateway` class. To set this up, another class will be created to handle all the mailing functionality called `Mailer`. The `Mailer` class would have a `deliver` method on it. You also need to update the `Subscription` class since it will also be receiving the mailer class in the constructor.
```php
    protected Gateway $gateway;
    protected Mailer $mailer;

    public function __construct(Gateway $gateway, Mailer $mailer)
    {
        $this->gateway = $gateway;
        $this->mailer = $mailer;
    }
```
Now in the test you would create a `mailer` variable that would contain a mocked version of the `Mailer` class.
```php
$gateway = $this->createMock(Gateway::class);
$mailer = $this->createMock(Mailer::class);
$subscription = new Subscription($gateway, $mailer);
```
Now you can write out you expectation on the mocked `Mailer` class and on the `Gateway` class.
```php
$gateway->method('create')->willReturn('receipt-stub');

$mailer
    ->expects(once())
    ->method('deliver')
    ->with('Your receipt number is: receipt-stub');
```
All of the above code needs to be before any call to the `create` method in the test.

### Laravel Dusk - Browser Testing

Install with `composer require laravel/dusk --dev` and then run the installer composer artisan dusk:install`

To create tests run `php artisan dusk:make TestName`. This will create a new test in `/tests/Browser/`

To run the tests run `php artisan dusk`. This will execute all the tests in the `/tests/Browser` directory.

Just like regular PHPUnit tests, make sure to use the `use DatabaseMigrations;` to import this trait.

Within a test, you can start a browser session with 
```php
$this->browse(function(Browser $browser) {
    //assertion and actions here use the $browser object
});
```
***Available Commands*** Visit [Laravel Dusk](https://laravel.com/docs/8.x/dusk) form more info
`->visit('/')` visit a specific page
`->visitRoute('namedRoute')` visit a named route
`->type('fieldName', 'test')` type `test` in a form field named `fieldName`
`->press('Button Text')` press a button with the text `Button Text`
`->screenshot('filename')` saves a screenshot with the name `filename`. Screenshots are saved in `/tests/Browser/screenshots/`

### Continuous Integration Testing (CI)
* #### CI Services
    * [Travis CI](https://travis-ci.com)
    * [Team City by JetBrains](https://www.jetbrains.com/teamcity/)
    * Github Actions are available on each of your repositories.

