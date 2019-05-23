# PHP Bits

## Notes on [PHP Bits](https://laracasts.com/series/php-bits) from Laracasts and other Miscellaneous PHP notes

* ### Array Destructuring
    * Available in PHP 7^
    * Examples:

        Assign variables names to each element of an array. Similar to the PHP `list` function
        ```
        $people = [
            ['John', 'Doe'],
            ['Jane', 'Doe']
        ];

        foreach($people as $person) {
            [$first, $last] = $person;
            var_dump("{$first} {$last}");
        }
        ```
        Another way to do the above example
        ```
        $people = [
            ['John', 'Doe'],
            ['Jane', 'Doe']
        ];

        foreach($people as [$first, $last]) {
            var_dump("{$first} {$last}");
        }
        ```
        If you are only interested in a certain position of the array you can leave the other positions blank
        ```
        [, $last] = ['John', 'Doe']; 
        var_dump($last); //Doe

        preg_match('/\d{3}-\d{3}-(\d{4})/', '555-555-5555', $matches);
        var_dump($matches); // [0] => 555-555-5555 [1] => '5555'

        [,$lastFour] = $matches;
        var_dump($lastFour); // 5555
         ```
        If you have an object that implements the [IteratorAggregate](https://www.php.net/manual/en/class.iteratoraggregate.php) interface then you can destructure that by
        ```
        // Laravel uses this on the factory class
        [$user1, $user2, $user3] = factory('App\User', 3)->create();
        ```
* ### Invokable Classes
    * A magic method (`__invoke`) within a class that is executed any time the class is interacted with as function.
        ```
        class Example
        {
            public function __invoke()
            {
                return 'Hello World';
            }
        }

        $example = new Example;

        // interacting with an object as a function or method
        echo $example(); // Hello World
        ```
        Sometimes you only need a single action for something or a single use contorller. In this case you can add a controller with an `__invoke` method to accomplish this. This can also be an example of where you don't want to pass a closure to the `Route::get` method and instead use a invokable class to perform the logic.
        ```
        // routes\web.php file
        Route::get('/', 'HomeController');

        // App\Http\Controllers\HomeController.php
        namespace App\Http\Controllers;

        class HomeController extends Controller
        {
            public function __invoke() {
                return 'You've reached the Home Controller';
            }
        }

        // So any time you hit the home route then 
        // the controller will automatically return "You've reached the Home Controller"
        ```
        Another use case is when you have a complex db query and you really don't want it in the controller, then you could use a `query object`.
        ```
        // App\Queries\FeaturedCollectionsQuery.php
        namespace App\Queries;

        class FeaturedCollectionsQuery()
        {
            public function __invoke() {
                return 'running the complex query here';
            }
        }

        // App\Http\Controllers\HomeController.php
        namespace App\Http\Controllers;

        class HomeController extends Controller
        {
            public function __invoke(FeaturedCollectionsQuery $featuredCollections) {
                $featureCollections();
            }
        }

        // So any time you hit the home route then 
        // the controller will automatically return "running the complex query here"
        ```
* ### Refactor repeated suffixes and prefixes
    * If you find your class containing several of the same suffixes or prefixes then it may be a sign that you need to take those methods an promote them to a first class citizen role.
        ```
        // A User class before refactor
        class User
        {
            public function favoritesCount() {
            }
            public function watchLatersCount() {
            }
            public function completionsCount() {
            }
            public function experienceCount() {
            }
        }
        ```
        All these methods pertain to a Users stats and should be refactored to their own `Stats` class. Then you want to be able to access these stats like this: `$user->stats()->favorites();`
        ```
        class Stats()
        {
            private $user;

            public function __construct($user)
            {
                $this->user = $user;
            }
            public function favorites() {
            }
            public function watchLaters() {
            }
            public function completions() {
            }
            public function experience() {
            }
        }

        class User() {
            public function stats() {
                return new Stats($this);
            }
        }
        ```
        It is important to note that the User class is passing an instance of itself to the newed up Stats class. This way the Stats class can then use that designated user to perform all the calculations internally. Also you aren't using Stats as a trait, it is its own first class citizen.
* ### Type Hints or Scalar Type Hinting or Duck Typing
    * This is the process by where you asign a certain type to a parameter or argument. This is useful for clearer documentation but it also may add unnecessary noise to the code.
    * Some type hints: `array`, `string`, `callable`, `bool`, `int`, `float`
    * Examples:
        type hinting arguments in a method
        ```
        public function foo(string $name, callable $handler)
        {

        }
        ```
        using return types
        ```
        public function foo(string $name, callable $handler) : void
        {
        }
        public function checkThis(string $name, callable $handler) : bool
        {
        }
        public function createName(string $first, string $last) : string
        {
        }
        ```
