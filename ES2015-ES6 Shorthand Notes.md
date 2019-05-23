# ES2015 (ES6) Shorthand Notes

### When using this shorthand, you'll need to also use a compiler, like [Babel](https://babeljs.io/), to convert the shorthand to conventional formatting.

* ### Use `let` or `const` for declaring variable names instead of `var`
* ### ARROW SYNTAX:
    * i.e. `this.tasks.forEach(function(task) { console.log(task) });` now can be one of the following:
        * `this.tasks.forEach((task) => { console.log(task) });` Optional `()` around `task` if only `1 param`, must be used with more than `1 param`
        * `this.tasks.forEach(task => { console.log(task) });` Only if there is one param
        * `this.tasks.forEach(() => { console.log('done') });` Only if there are no param
        * Single line method: If there is only one line of logic inside the `{}` then you can get rid of the `{}` all together to be `this.tasks.forEach(task => console.log(task));` when using this method, `return` is implied in the function call, therefore it isn't needed. i.e `return console.log(task);` should be just `console.log(task);` without the `return`
    * NOTE: when using `this` inside the function call logic, `this` will be referenced in the `parent function` or `class` and not in the `local scope` as it did when using the function logic
* ### DEFAULT PARAMETERS
    * when making a function call the parameters being passed to the function can now have default values.
        * `function test(param1, param2) {  }`  now can be `function test(param1, param2 = 'default') {  }`
        * a `function` can also be reference as a `default parameter value`. `function test(param1, param2 = newTestFunction()) {  }`
* ### REST PARAMETERS
    * when you want an unknown number of params you can now use: `function sum(…fncParams) {  }`, which will then receive an array of params for `fncParams`
    * the `rest operator` should be the only param or you can have a param before the param.
        * `function sum(foo, …fncParams) {  }`
* ### SPREAD PARAMETERS
    * used when you want to send an array of values to a function call that is expecting a certain amount of parameters to be passed
        ```
        function sum(x, y) { 
            return x + y; 
        } 
        let nums = [1, 2]; 
        console.log(sum(…nums)); //returns 3
        ```
* ### TEMPLATE STRINGS
    * you can now declare a string in a formatted way with `returns`, `spaces` or `tabs`. You need to use ` instead of ` or `
    * to reference variables in a declared formatted string you can do so using `${varName}:`
        * ``let name = 'john'; let template = `Hello, ${name}`;`` 
* ### OBJECT ENHANCEMENTS
    * **Object Shorthand:** when referencing a variable or member within an object you now can do this
        * instead of using `let var = { name: name, age: age }` you can use `let var = { name, age }`
    * **Method Shorthand:** when referencing a method with an object you can do this
        * instead of using `let var = {getName: function () { blah } }` you can now use `let var = {getName() { blah }}`
    * **Object Destructuring:** 
        ```
        let data = {
            name: 'karen', 
            age: 32, 
            results: ['foo', 'bar'], 
            count: 30
        }; 
        let {results, count} = data;
        ```
        This will now assign the values from the `data` object to new variables called `results` & `count`.
        ```
        let data = { name: 'luke', age: 24 }; 
        
        function greet({ name, age }) {
            console.log(`Hello, ${name}. You are ${age}`);
        }
        
        greet(data);
        ```
        ```
        axios.get('status')
            ->then(({data}) => this.status = data ); 
        ```
        instead of
        ```
        axios.get('status')
            ->then(response => this.status = response.data);
        ```

* ### CLASSES
    * Classes now work the same way as they do in PHP. You can create a class with a constructor and reference methods when the new class is called.
* ### STRING ADDITIONS
    * Various methods that can be called on strings or arrays.
        * `includes`, `startsWith` , `endsWith`: `let title = 'Red Rising'; Params: { 'string test', offset int, }`
            * **includes** - `if (title.includes('Red')) { alert('yes') } //yes`
            * **startsWith** - `if (title.startsWith('Red')) { alert('yes') } //yes`
            * **endsWith** - `if (title.endsWith('Red')) { alert('yes') } else { alert('no'); } //no`
        * `repeat`: 
            * `'str'.repeat(50); //repeats str 50 times`
* ### ARRAY ADDITIONS
    * various methods that can be called on arrays
        * **includes** - returns boolean `[6, 8, 10, 12, 20].includes(20); //true`
        * **find** - returns first result. `[6, 8, 10, 12, 20].find(item => item > 8); //returns 10`
        * **findIndex** - returns the index of the result `[6, 8, 10, 12, 20].find(item => item > 8); //returns 2`
        * **fill**
        * **keys**
        * **values**
        * **entries**

		Above methods work recursively as well:
        ```
		class User {
			constructor(name, isAdmin) {
				this.name = name;
				this.isAdmin = isAdmin;
			}
		}
		let users = [
			new User('Vanessa', false),
			new User('Aydan', false),
			new User('River', false),
			new User('Scott', true)
		]
		console.log(
			users.find(user => user.isAdmin).name
		);
        //returns Scott since he is the first occurrence where isAdmin is set to true
        ```

* ### GENERATORS or ITERATORS
    * `Generators` are functions that allow you to pause or exit the function then return back to it later.
        * generator functions must contain an `*` before the name. i.e. `function *range() {  }`, `function* range() { }`, `function * range() { }`
        * `yield` is used within the function and allows the function to be paused but also returns the value specified
        * `next()` method is used outside the function on an object assigned to the function, which will make the generator proceed to the next yield
        ```
		function *range(start, end) {
			while (start <= end) {
				yield start;
				start++;
			}
		}

		let iterator = range(1, 5);

		console.log(iterator.next()); //returns an object { value: 1, done: false }
		console.log(iterator.next()); //returns an object { value: 2, done: false }
		console.log(iterator.next()); //returns an object { value: 3, done: false }
		console.log(iterator.next()); //returns an object { value: 4, done: false }
		console.log(iterator.next()); //returns an object { value: 5, done: false }
		console.log(iterator.next()); //returns an object { value: undefined, done: true }
		
		//use the for-of operator to have it proceed to the end of the generator and only return the value not the entire object
		for (let i of iterator) console.log(i); // returns 1 2 3 4 5

		//use the spread operator to pass the params to the generator and have it return an array of values only, not the object
		console.log( […range(1, 5)] ); //returns [1, 2, 3, 4, 5]
        ```
* ### SETS
    * `Sets` are collections of values where each one must be unique. Duplicate values will be ignored
		```
		let items = new Set([ 'one', 'two', 'three' ]);
		console.log(items); //returns a Set object {'one', 'two', 'three')
		
		//API is now available for Set object
		items.size; //returns 3
		items.add('four'); //returns a Set object {'one', 'two', 'three', 'four')
		items.delete('two'); //returns a Set object {'one', 'three', 'four')
		items.has('four'); //returns true
		items.has('five'); //returns false
		items.forEach(item => console.log(item)); returns one three four undefined
		items.clear(); //clears out the entire object
		items.values(); //returns a SetIterator object containing the values of the Sets object
        ```