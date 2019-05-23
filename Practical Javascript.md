# Practical Javascript Notes

## Notes from the Watch and Code Practical Javascript course

* ### Empty objects are not equal since they have different memory addresses. `{} === {} //false`
* ### When you set an object to equal another object they become linked.
    ```
    var h1 = { color: ‘blue’ };
    var h2 = h1;
    h2.color = ‘red’; 
    h1.color == ‘red’; //true
    ```
* ### Make sure to load your `scripts` at the end of the `body` tag.
* ### Difference between `window` and `document`
    * the `window object` is the first thing loaded and contains info about the `length`, `innerHeight`, `innerWidth`, `name`, etc
    * the `document object` is loaded into the `window object` is more `dom` related elements. You can access the document properties by using just `document.{property}`. `window.document.{property}` is not necessary.
* ### Arrays
    * `arrayName.push(‘newArrayItem’);` tacks on the `newArrayItem` on to the end of the array
    * `arrayName.splice(0,1);` removes an item from the array at `{index, numberToSlice}`. In this case at index 0 and only one item is being removed.
* ### Methods:
    * `getElementByID`
    * `querySelectorAll`
    * `addEventListener` - adds an event listener to an element. Instead you could also add an event to the element itself. i.e. `onclick`
    * `document.createElement(‘li’)` - creates and a new `li` element. Can then be used to place that new element in the `dom`.
    * `el.appendChild(newElement)` - appends the `newElement` element as a child to the chosen element.
    * `document.querySelector(‘ul’)` - searches the document for a match the the element provided.
    * `setTimeout(function () {  }, 1000);` - runs the provided function after a give amount of milliseconds. In this case 1000 or 1 sec.
    * `arr.forEach(function(val) { console.log(val); });` - Iterates over the given array and performs the given function.
        * If you pass a second argument to the callback function, this will be in the `index` of the array that the pointer is currently at. `function(val, index)`
        * If you need access to the current object (`this`) that exists outside of the `forEach` callback, then it can be passed as an argument along with the callback function. i.e. 
            ```
            arr.forEach(function(val) { 
                console.log(val);
            }, this);
            ```
* ### Functions:
    * `parseInt(value)` - Type cast the passed value as an `integer`
    * `bind()` - returns a copy of the function where this is set to the first argument passed.
    * `apply()` & `call()` - works the same as `bind()` except the attached function is called immediately where `bind()` does not execute the function. `apply()` & `call()` do not return a copy of the function.
* ### Accessing Element Properties
    * `element.innerHTML` - gets or sets the inner html for the chosen element
    * `element.textContent` - gets or sets the text content for the chosen element
    * `element.className` - gets or sets the class name for the chosen element
* ### Debugging in Chrome
    * To start the debugger at a certain point in your code just add `debugger;` This will pause the execution at that point
    * In dev tools, hovering over `variables`, `object` or `arrays` will show you the value at that point in the execution.
    * When the debugger gets to a function call, you can debug that function as well by clicking the `down arrow` (Step into next function call) `button` in the dev tools debugger
* ### Understanding `’this’`
    * When `this` is used in a regular function (not inside another function or object) then `this = window`
        * In strict mode, `this` would be undefined.
    * When `this` is used in a method within an object, then `this` is equal to the `object`
    * When `this` is used within a function that is called as a constructor, then `this` is the `new object` that is created.
        ```
        function Person(name) { 
            this.name = name; 
        }
        var myName = new Person(‘Scott’);
        ```
        When a function is called like the example above, `this` is returned by the constructor. `myName` will now be equal to the object returned from the constructor. `{ name: ‘Scott’ }` 
    * You can explicitly declare the value of `this` by using `bind`, `apply` or `call`.
        ```
        function logThis() { 
            console.log(this); 
        }
        var explicitlySetLogThis = logThis.bind({ name: ‘scott’ }); 
        // returns logThis as a new function with name bound to the object
        ```
        You can execute the function directly instead of assigning it to a variable by adding the function call `()` at the end of the statement.
        
            ```
            logThis.bind({ name: 'scott' })(); // object { name: 'scott' }

            explicitlySetLogThis(); //this is now an object { name: 'scott' }

            function logThisWithArguments(greeting, name) { 
                console.log(greeting, name); 
                console.log(this); 
            }

            logThisWithArguments
                .apply({ name: 'Scott' }, ['hi,', 'scott']); 
            //this = {name: 'Scott'}

            logThisWithArguments
                .call({ name: 'Scott' }, 'hi,', 'scott'); 
            //call can send arguments to the function without 
            //it being in an array just apply requires. this = {name: 'Scott'}
            ```
        
* ### Terms:
    * `Higher order functions` - functions that accept other functions. They enhance the functionality of the other function
    * `Callback functions` - the functions that are passed into `higher order functions`
    * `Event Delegation` - `DOM event delegation` is a mechanism of responding to `ui-events` via a single common parent rather than each child, through the magic of `event "bubbling"` (aka `event propagation`). <https://stackoverflow.com/questions/1687296/what-is-dom-event-delegation>
    * `Prototype` - The mechanism by which JavaScript objects inherit features from one another.
* ### Notes:
    * when getting the value of an input element from the `dom` you can use
        * `.value` to get the contents of the input element
        * `.valueAsNumber` to get the contents as a number
