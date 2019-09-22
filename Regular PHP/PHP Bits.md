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
    * #### The code flow of a generator:
        * Taken from https://alanstorm.com/php-generators-from-scratch/
            ```php
            function myGeneratorFunction()
            {
                echo "One","\n";
                yield 'first return value';

                echo "Two","\n";
                yield 'second return value';

                echo "Three","\n";
                yield 'third return value';
            }

            $iterator = myGeneratorFunction();

            $value = $iterator->current();
            ```
            The generator is started after the above `current()` method is called, thus echoing out `One`, since it is not yielded yet. We have also assigned the value of the first `yield` statement to `$value` but won't be displayed until it is echoed out.
            ```php
            // One
            echo $value, PHP_EOL;
            //first return value
            
            $value = $iterator->next();
            // Two
            echo $value, PHP_EOL;
            // 
            ```
            `$value` is null so nothing is echoed. The `next()` method does not return anything. To get the next `yield`, you need to call the `current()` method.
            ```php
            $value = $iterator->current();
            echo $value, PHP_EOL;
            // second return value

            $value = $iterator->next();
            // Three
            $value = $iterator->current();
            echo $value, PHP_EOL;
            // third return value

            var_dump($iterator->valid());
            // bool(true) This is because the generator has paused at the third yield.

            $value = $iterator->next();
            var_dump($iterator->valid());
            // bool(false) The generator has completed and closed. 
            ```
        * Once a generator has closed, it can not be accessed anymore.
        * Another example of generator flow:
            ```php
            // 1. Define a Generator Function
            function generator_range($min, $max)
            {
                //3b. Start executing once `current`'s been called
                for ($i = $min;$i <= $max;$i++) {
                    echo 'Starting Loop',"\n";
                    yield $i;   //3c. Return execution to the main program
                    //4b. Return execution to the main program again
                    //4a. Resume exection when `next is called
                    echo 'Ending Loop',"\n";
                }
            }

            //2. Call the generator function
            $generator = generator_range(1, 5);

            //3a. Call the `current` method on the generator function
            echo $generator->current(), "\n";

            //4a. Resume execution and call `next` on the generator object
            $generator->next();

            //5 echo out value we yielded when calling `next` above
            echo $generator->current(), "\n";
            ```
            In plain english, the above program
            1. Defines a generator function
            2. Calls that generator function to get a generator object
            3. Starts executing the generator function when the program calls `current()`, which `yields` a value
            4. Returns to the generator function when we call `$generator->next()` and makes another trip through the loop until `yield` is called again
            5. echos out the value of the second `yield` when we call `current()` again
            
            The most important step is #4. When we call `next` and return execution to the generator function — the values of `$i`, `$min`, and `$max` are all the same as when we left the function in step #3. PHP held on to these values when it paused the function.
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
    * You can also send data to a generator, when it is being iterated over, by using the `send()` method on the original generator object.
        ```php 
        function getRange ($max = 10) {
            for ($i = 1; $i < $max; $i++) {
                $injected = yield $i;

                if ($injected === 'stop') return;
            }
        }

        $generator = getRange(PHP_INT_MAX);

        foreach ($generator as $range) {
            if ($range === 10000) {
                $generator->send('stop');
            }

            echo "Dataset {$range} <br>";
        }
        ```
    * It is also possible to yield a `key-value` pair from the generator.
        ```php
        function getRange ($max = 10) {
            for ($i = 1; $i < $max; $i++) {
                $value = $i * mt_rand();

                yield $i => $value;
            }
        }
        ```
    * *Generator Delegation* allows you to yield values from other generators, arrays or functions by using the `yield from` keyword.
        ```php
        function count_to_ten() {
            yield 1;
            yield 2;
            yield from [3, 4];
            yield from new ArrayIterator([5, 6]);
            yield from seven_eight();
            yield 9;
            yield 10;
        }

        function seven_eight() {
            yield 7;
            yield from eight();
        }

        function eight() {
            yield 8;
        }

        foreach (count_to_ten() as $num) {
            echo "$num ";
        }

        // This yields 1 2 3 4 5 6 7 8 9 10
        ```
    * You can return a value from a generator without yielding it first. If something is returned from a generator, the only way to access it is using the `getReturn()` method. Note that if something is returned from a generator all further execution of the generator is stopped.
        ```php
        $gen = (function() {
            yield 1;
            yield 2;
            // 3 is only returned after the generator has finished
            return 3;

        })();

        foreach ($gen as $val) {
            echo $val, PHP_EOL;
        }

        // Outputs 1 2

        echo $gen->getReturn(), PHP_EOL;

        // Outputs 3
        ```
        If you try to use `getReturn()` on a generator that has finished you will get an exception. You can use the `valid()` method to check to see if a generator has finished or not. The `valid()` method return false if the generator has closed. So the use of the `getReturn()` method could be updated with a check using the `valid()` method.
        ```php
        if ($gen->valid()) {
            echo $gen->getReturn(), PHP_EOL;
        }
        ```
    * Using an `ArrayIterator` object instead of yielding.
        ```php
        $values = [1,2,3,4,5];

        $iterator = new ArrayIterator($values);
        while($number = $iterator->current()) {
            echo $number, "\n";
            $iterator->next();
        }
        ```
    * You can cast the generator back to an array by using the following command. Be careful, since this might create the same memory error as before using the generator.
        ```php
        interator_to_array($generator);
        ```
    * #### Generator Methods
        * `current()` - gets the current yielded value
        * `return()` - gets the return value of a generator
        * `key()` - gets the yielded key
            * Used when a key-value pair is yielded in the generator
                ```php
                function Gen()
                {
                    yield 'key' => 'value';
                }

                $gen = Gen();

                echo "{$gen->key()} => {$gen->current()}";
                ```
        * `next()` - resumes execution of the generator
        * `rewind()` - rewind a generator
            * If iteration has already begun, this will throw an exception
        * `send()` - send a value to a generator
        * `throw()` - throw an exception into a generator
            * Throws an exception into the generator and resumes execution of the generator. The behavior will be the same as if the current `yield` expression was replaced with a throw `$exception` statement.
            * If the generator is already closed when this method is invoked, the exception will be thrown in the caller's context instead.
        * `valid()` - checks if the iterator has been closed
* ### PHP Reflection Classes
    * Reflection classes allow you to inspect things like a parameter list and determine what is being requested.
    * In this example, JW is using a "`load`" method that will receive a closure that is sending through any number of `User` instances. The `ReflectionFunction` class can be used to inspect what is being sent to the load method. It will reflect into the anonymous function or closure being sent to the `load` method 
        ```php
        public function test() {

            $this->load(function(User $user, User $user2, User $user3) {
                dump($user);
            });
        }

        protected function load($callable) {
            $function = new .\ReflectionFunction($callable);

            dd($function); //view what is available in the ReflectionFunction class.
            //$function->parameters will contain to instances of ReflectionParameter. 
            //One for each User instance in the closure.
            //$function->getNumberOfParameters(); will return the number of parameters in the closure.

            $users = [];

            foreach (range(1, $function->getNumberOfParameters()) as $index) {
                $users[] = new User();
            }

            //now trigger the callable anonymous closure function
            //which in this case will dump each user instance created in the above foreach
            //must destructure the array being sent to the callable.
            //In this case the destructuring will look like $callable($users[0], $users[1], $users[2])
            $callable(...$users); 
        }
        ```
        
        `load` method can be refactored to use a collection instead

        ```php
        protected function load($callable) {
            $function = new .\ReflectionFunction($callable);

            $users = collect(range(1, $function->getNumberOfParameters()))
                -map(function() {
                    return new User();
                });

            $callable(...$users); 
        }
        ```