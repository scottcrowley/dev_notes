# PHP Bits

## Notes on [PHP Bits](https://laracasts.com/series/php-bits) from Laracasts and other Miscellaneous PHP notes

* ### Array Destructuring
    * Available in PHP 7^
    * Examples:

        Assign variables names to each element of an array. Similar to the PHP `list` function
        ```php
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
        ```php
        $people = [
            ['John', 'Doe'],
            ['Jane', 'Doe']
        ];

        foreach($people as [$first, $last]) {
            var_dump("{$first} {$last}");
        }
        ```
        If you are only interested in a certain position of the array you can leave the other positions blank
        ```php
        [, $last] = ['John', 'Doe']; 
        var_dump($last); //Doe

        preg_match('/\d{3}-\d{3}-(\d{4})/', '555-555-5555', $matches);
        var_dump($matches); // [0] => 555-555-5555 [1] => '5555'

        [,$lastFour] = $matches;
        var_dump($lastFour); // 5555
         ```
        If you have an object that implements the [IteratorAggregate](https://www.php.net/manual/en/class.iteratoraggregate.php) interface then you can destructure that by
        ```php
        // Laravel uses this on the factory class
        [$user1, $user2, $user3] = factory('App\User', 3)->create();
        ```
* ### Invokable Classes
    * A magic method (`__invoke`) within a class that is executed any time the class is interacted with as function.
        ```php
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
        ```php
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
        ```php
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
        ```php
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
        ```php
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
        ```php
        public function foo(string $name, callable $handler)
        {

        }
        ```
        using return types
        ```php
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
* ### PHP Generators and dealing with memory intensive processes
    * A PHP Generator is used when you need to iterate over a very large set of data and you don't want it to do all of it in memory. The Generator class implements the Iterator contract, so the generator can be used in a foreach loop.
        * For example
            ```php
            Route::get('/', function() {
                $items = range(1, 10000000);

                return 'done';
            });
            ```
            This will cause PHP to die with a `FatalErrorException` because it is trying to load 10 million things into the `$items` array. To solve this issue you could do something like:
            ```php
            function customRange($begin, $end) {
                for ($i = $begin; $i <= $end, $i++) {
                    yield $i;
                }
            }

            Route::get('/', function() {
                foreach (customRange(1, 10000000) as $i) {
                    if ($i < 10000) {
                        dump($i);
                    }
                }
                return 'done';
            });
            ```
            So, instead of returning something from the function or adding the items back to an array, which would load everything into memory, you will yield the value instead. 

        * Another example showing that you can yield multiple times
            ```php
            function test() {
                yield 1;
                yield 2;
                yield 3;
            }

            Route::get('/', function() {
                foreach (test() as $item) {
                    dump($item);
                }
            });
            ```
            This will display:
            ```
            1
            2
            3
            ```
        * Another example showing you can yield anything you want. Say you want to create 1 million Project models using the Factory class.
            ```php
            $projects = factory(App\Project::class, 1000000)->create();
            ```
            This will die with a memory exhausted error. Instead you could use a generator:
            ```php
            function make($times) {
                for ($i=0; $i <= $times; $i++) {
                    yield factory(App\Project::class)->create();
                }
            }

            Route::get('/', function() {
                foreach (make(100) as $result) {
                    dump($result);
                }
            });
            ```
            Using the `foreach` will execute the `factory` method, within the `make` function, 100 times but won't load any of it into memory. Note that if the `foreach` wasn't used and the `make` method was still called then the `factory` method would not be called. Instead you would have a generator with 100 items in it. The actual code in the `yield` statement isn't executed until you either iterate over the generator or access an item directly.
            ```
            $projects = make(100);
            $Projects->current();
            $projects->next();
            $projects->current();
            ...etc
            ```
    * Generators can not be interacted with like an array. So you can't access a generator value by specifying a key
        ```php
            function test() {
                yield 1;
                yield 2;
                yield 3;
            }

            Route::get('/', function() {
                $generator = test();
            });
        ```
        So you can't access the second value by using `$generator[1]`. This will give you a "`Cannot use object of type Generator as an array`" error. Instead, since the Generator implements the iterator class, you can use things like
        ```php
        $generator->current(); //displays the item at the current pointer
        $generator->next(); //moves the pointer to the next item
        ```
    * You can cast the generator back to an array by using the following command. Becare, since this might create the same memory error as before using the generator.
        ```php
        interator_to_array($generator);
        ```