# Code Katas with PHPUnit

## Notes from the Laracast series [Code Katas with PHPUnit](https://laracasts.com/series/code-katas-with-phpunit)

* ### PHPUnit setup
    * Make a directory for the project and import PHPUnit
        ```zsh
        mkdir katas
        cd katas
        mkdir src
        mkdir tests
        composer require phpunit/phpunit
        ```
    * Edit `composer.json` file in the project root
        ```json
        {
            "require": {
                "phpunit/phpunit": "^8.5"
            },
            "autoload": {
                "psr-4": {
                    "App\\": "src/"
                }
            }
        }
        ```
    * Dump the composer autoload
        ```zsh
        composer dump-autoload
        ```
* ### Katas
    * #### Prime Factors
        * Write an algorithm that will return the prime factors for a given number. The prime factors of a number are the factors that when multiplied together equal the given number. i.e. 100 = 2 * 2 * 5 * 5
        * Add test file in `tests/PrimeFactorsTest.php`
            ```php
            <?php

            use App\PrimeFactors;

            class PrimeFactorsTest extends \PHPUnit\Framework\TestCase
            {
                
            }
            ```
        * Using a ***DataProvider*** with PHPUnit
            * A dataProvider allows you to tell php unit to run a different set of data on the same method. PHPUnit will treat each run as a separate test.
            * In this example we want to test several numbers but use the same method to make sure our algorithm works. Add the following test and method to the PrimeFactorsTest file.
                ```php
                /**
                * @test
                * @dataProvider factors
                */
                function it_generates_prime_factors($number, $expected)
                {
                    $factors = new PrimeFactors();
                    $this->assertEquals($expected, $factors->generate($number));
                }

                public function factors()
                {
                    return [
                        [1, []],
                        [2, [2]],
                        [3, [3]],
                        [4, [2,2]],
                        [5, [5]],
                        [6, [2,3]],
                        [7, [7]],
                        [8, [2, 2, 2]],
                        [9, [3, 3]],
                        [11, [11]],
                        [12, [2, 2, 3]],
                        [17, [17]],
                        [100, [2, 2, 5, 5]],
                        [999, [3, 3, 3, 37]]
                    ];
                }
                ```
                Notice the @dataProvider parameter in the comment section of the test method. This references the factors method that PHPUnit will use to run against this test. The factors method returns an array of arrays containing the number to test and the expected outcome.
        * Setup the `PrimeFactors` class in the `src/` directory
            * We want the algorithm to perform the following actions
                * Is the number divisible by 2
                * If true, then divide by 2. If false, increase the canidate and try again
                * Repeat.
            * `src/PrimeFactors.php`
                ```php
                <?php

                namespace App;

                class PrimeFactors
                {
                    public function generate($number)
                    {
                        $factors = [];

                        for ($divisor = 2; $number > 1; $divisor++) {
                            for (; $number % $divisor === 0; $number /= $divisor) {
                                $factors[] = $divisor;
                            }
                        }

                        return $factors;
                    }
                }
                ```
    * #### Roman Numerals
        * Write an algorithm to convert a given number into roman numerals. 
        * Add test file in `tests/RomanNumeralsTest.php`
            ```php
            <?php

            use App\RomanNumerals;
            use PHPUnit\Framework\TestCase;

            class RomanNumeralsTest extends TestCase
            {
                /**
                * @test
                * @dataProvider checks
                */
                function it_generates_roman_numerals($number, $numeral)
                {
                    $this->assertEquals($numeral, RomanNumerals::generate($number));
                }

                /** @test */
                function it_cannot_generate_a_roman_numeral_for_less_than_1()
                {
                    $this->assertFalse(RomanNumerals::generate(0));
                }

                /** @test */
                function it_cannot_generate_a_roman_numeral_for_more_than_3999()
                {
                    $this->assertFalse(RomanNumerals::generate(4000));
                }

                public function checks()
                {
                    return [
                        [1, 'I'],
                        [2, 'II'],
                        [3, 'III'],
                        [4, 'IV'],
                        [5, 'V'],
                        [6, 'VI'],
                        [7, 'VII'],
                        [8, 'VIII'],
                        [9, 'IX'],
                        [10, 'X'],
                        [40, 'XL'],
                        [50, 'L'],
                        [90, 'XC'],
                        [100, 'C'],
                        [400, 'CD'],
                        [500, 'D'],
                        [900, 'CM'],
                        [1000, 'M'],
                        [3999, 'MMMCMXCIX']
                    ];
                }
            }
            ```
        * Setup the `RomanNumerals` class in the `src/` directory
            * src/RomanNumerals.php
                ```php
                <?php

                namespace App;

                class RomanNumerals
                {
                    const NUMERALS = [
                        'M' => 1000,
                        'CM' => 900,
                        'D' => 500,
                        'CD' => 400,
                        'C' => 100,
                        'XC' => 90,
                        'L' => 50,
                        'XL' => 40,
                        'X' => 10,
                        'IX' => 9,
                        'V' => 5,
                        'IV' => 4,
                        'I' => 1
                    ];

                    public static function generate(int $number)
                    {
                        if ($number <= 0 || $number >= 4000) {
                            return false;
                        }

                        $result = '';

                        foreach (static::NUMERALS as $numeral => $arabic) {
                            for (; $number >= $arabic; $number -= $arabic) {
                                $result .= $numeral;
                            }
                        }

                        return $result;
                    }
                }
                ```
    * #### next kata
