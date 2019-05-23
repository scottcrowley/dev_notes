# PHP Object Oriented Programming Notes

## Notes on principles of using objects in PHP

* ### Classes
    * **Encapsulation**
        * `Public` methods and properties can be referenced anywhere, inside and out of, a class
        * `Private` methods and properties can only be referenced within the instantiated class
        * `Protected` method and properties can only be referenced with the instantiated class or any sub classes
    * An `Abstract class` can not be instantiated directly. It can only be extended by another class instance.
        * An `Abstract` method or property, within an `abstract` class, is like saying that method or property is required and must be provided in the sub class.
            * `abstract protected function getArea();` Is telling any class that extends the abstract class, that it must contain a `getArea` method or an error will  be thrown
    * `Type Hinting` is used to declare the expected data type (`array`, `object`, `interface`, etc)
        * `function functionName(array $argumentName) {  `} Expects `$argumentName` to be an `array`
        * `class ClassName { protected $objectName; public __construct(ObjectName $objectName) { $this->objectName = $objectName; } }` Expects to receive an object class called `ObjectName` as the passed argument for `$objectName`.
    * `Statics` and `Constants`
        * A `static` method or property within a class can be referenced directly without creating an instance of the object class. i.e.
            ```
			class Math {
                public static function add(…$nums) {
                    return array_sum($nums);
                }

			}

			echo Math::add(1, 2, 3); //echo 6
            ```
            * "`::`" is called a `Scope Resolution Operator`
            * To reference a static method or property within a class directly, you can no longer use `$this->`, you have to use `static::`
        * It is important to note that `static` properties remain the same no matter how many instances of the class are created. Properties are not reset when a new instance is called. Instead, they retain there current value. The values are shared among all instances.
        * A `constant` should be thought of something that can never ever change. When creating a `constant` property, the name should be all caps and the '$' is not needed. i.e. `const TAX = .09;`
* ### Interfaces
    * Think of `interfaces` as a contract. It sets the terms for all iterations of the instance.
    * An interface can only contain `public` method and properties
* ### Using Composer
    * `Composer` should be used for most projects. It handles loading dependencies as well as provides an autoloader for classes.
        * a `composer.json` file should exist in the root of the site. You can manually create it with an empty object `{  }`, or you can run `composer init` in the console. This is where the list of dependencies is given to Composer
        * After the `composer.json` file is created, you `run composer install` in the console and that will install any dependencies and set up the vendor directory in your project.
        * Now, within the `composer.json` file, you must specify how you want certain files to be auto loaded
			```
            {
				"autoload": {
					"psr-4": {
						"ProjectName\\": "src"
					}
				}
			}
            ```
			
            * "`psr-4`" is a standard convention that specifies how the files will be loaded
            * "`ProjectName\\`" refers to the `namespace` of your project. When doing this, it is required that you add `namespace ProjectName;` to the beginning of any other classes that will be autoloaded.
            * "`src`" is the location off off the root where your project classes will be loaded from. In this case it will be `root/src/`
            * IMPORTANT! Whenever the `composer.json` file is updated, you must run `composer dump-autoload` in the console.
            * The autoloader now needs to be imported into the entry point of your project. Whichever file is acting as the entry point, this line needs to be added at the top of the document. `require 'vendor/autoload.php';`
* ### Namespacing
    * See the **Using Composer** section above about referencing the `namespace` within Composer
    * At the top of all classes, there must be a line specifying the namespace. `namespace ProjectName;`
    * Any time a class, that is namespace, is referenced, there must be a `use ProjectName\TestClass;` at the top of the document. This way you can use `$test = new TestClass;` instead of `$test = new ProjectName\TestClass;` Make sure that any class being referenced has it's on use declaration.
    * Using `use ProjectName;` also holds true for any classes that have methods type hinting any autoloaded class. i.e. `class TestClass { public function __construct(FirstClass $firstClass) {  } }` The constructor references `FirstClass` as an argument, but if `use ProjectName;` is not added to the top of the class document then `ProjectName\FirstClass` needs to be used in the argument instead.
