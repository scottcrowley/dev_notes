# Vue 2 Notes

## Notes from the Learn Vue 2: Step By Step

* ### When using Vue with blade
    * If you are echoing out something in a blade file, you need to preface it with a `@` symbol. i.e. `<p>@{{ message }}</p>`
    * You can also use `v-text` in this case to bypass any conflict. i.e. `<p v-text="message"></p>`
    * You can also wrap a lot of code in the Laravel `verbatim` tag. `@verbatim <p>{{ message }}</p> @endverbatim`
* ### Vue Instance:
    * When declaring the root instance for Vue, you need to make sure that all components are registered before the root instance. The root instance should be at the bottom of your .js file.
    * `new Vue({  });`
        * `el: '#rootElement"` Element the Vue instance is bound to
        * `data: {  }` Object containing various data values
        * `mounted() {  }` This method is called when the Vue instance is mounted and ready to be used. Like `ready` in `jQuery`
        * `methods: {  }` Object containing any custom methods that the instance can use.
        * `computed: {  }` An object containing methods where properties are computed before they are available.
        * `created() {  }` used with components and is sort of like mounted except this method is run when the component is created.
        * `filters: {  }` An object containing methods where specific filters can be applied. Consider then as functions that can be stacked. See section on filters
        * `components: { componentName }` Should be used to declare the use of a component in the local scope of an instance or another component. Make sure that the component is imported into this instance as well. `"componentName" from "../components/componentName.vue"`
        * `watch: {  }` Can be used to watch for changes to certain properties.
* ### Event tips:
    * When using event listeners (`@click` or `v-on`) then you can get info about the event by using the `$event` property.
        * For example you are wanting to get the value of an input as someone is typing into it and update a data variable declared in the instance (`coupon`). `<input type="text" @input="coupon = $event.target.value">`
            * `target` is used to specify itself.
* ### Event Modifiers:
    * Sometimes when an event is fired on an element, the event needs to be modified before execution
    * Can be formatted as `<form method="post" @submit.prevent="onSubmit">...` When a `submit` event is received and before the `onSubmit` method is called, the modifier is applied.
    * Modifiers can be chained together by just adding another `.modifier`.
    * Modifiers can also be used by themselves without a method call. Just leave off the ="methodName"
    * Modifiers:
        * `.prevent` - same as `event.preventDefautl()`.
        * `.stop` - stops the event propagation
        * `.capture` - use capture mode when adding the event listener
        * `.self` - only trigger handler if ` ` is the element itself
        * `.once` - the click event will be triggered at most once
* ### Filters:
    * Filters are a way to have functions that can alter specific data that is sent to them. Filters can be stacked so several methods can be applied at the same time.
    * Filters can be declared globally or in a component
        * Globaly:
        ```
        Vue.filter('filtername', function(value) {
            //code to modify the value and return it
        });
        ```
        * Component: use the `filters` object within the component.
    * Example:
        * You want to send a created_at value (from a database collection) to a date formatting function that will use `moment.js`.
        * You access the filter with the following syntax.
            * `{{ status.created_at | ago }}`
            * `status = data` object bound to the instance.
            * `.created_at` = the field you want to pipe to the filter function
            * `| ago` = the pipe symbol followed by the filter function name declared in the filters property of either the instance or the component.
            * Multiple filters can be applied by adding another `|` followed by the next filter function name.
        * The filters object in the instance or component may looks something like this:
            ```
            filters: { 
                ago(date) { 
                    return moment(date).fromNow();
                }
            }
            ```
            The ago filter will accept a date and then send it to `moment` and have it converted to a readable date, which is using the `fromNow` formatting from `moment.js`

        * Additional filters example:
            ```
            filters: { 
                ago(date) { 
                    return moment(date).fromNow(); 
                }, 
                capitalize(value) { 
                    return value.toUpperCase(); 
                }
            }
            ```
            to access both filters the syntax would look like this: `{{ status.created_at | ago | capitalize }}`
* ### Mixins
    * A mixin is a file that can be imported into a component that contains code that is reusable in different components. The mixin file is a standard .js file and contains all the properties you want to reuse from the component.
    * For example: JW added a collection mixin that contained methods used to add a reply to the replies array collection.
        * `collection.js` (within `/js/mixins/`)
            ```
            export default {
                data() {
                    return {
                        items: []
                    };
                },

                methods: {
                    add(item) {
                        this.items.push(item);

                        this.$emit('added');
                    },

                    remove(index) {
                        this.items.splice(index, 1);

                        this.$emit('removed');
                    }
                }
            }
            ```
            Notice that the `data` object and the `methods` property are going to be mixed into the component when imported. It won't overwrite anything.
        * Within the component where you want to use the mixin:

            the collection.js file needs to be imported
            ```
            import collection from "../mixins/collection";
            ```
            the mixin needs to be added to the components mixin property
            ```
            mixins: [collection],
            ```
        * Now, any of the methods in collection.js can be used within the component where it is imported.
* ### Vue Directives
    * All directives begin with "`v-`"
    * `v-model` - Used to sync data with the root Vue instance and usually pertains to form inputs. Can be used to create custom inputs (see below). `v-model="some-data-value"` where `some-data-value` is a value declared within the Vue instance. `v-model` can only be used when the data has been declared in the data property of the instance. `v-model` is the same as doing something like this:
        ```
        <input type="text" :value="someVar" @input="someVar=$event.target.value">
        ```
    * `v-for` - used to cycle through a collection of items, like a standard for loop. `v-for="name in names"` where `names` is an array declared within the Vue instance.
    * `v-text` - sets the text content of an element to a variable value.
    * `v-on` - registering an event listener. `v-on:click="onClickMethod"` registers an on click event listener and will execute the `onClickMethod` when a click event happens.
        * The argument following the `:` can be any of the standard event names. i.e. `click`, `keyUp`, `keyDown`, `mouseOver`, etc.
        * The '`@`' symbol can be used as shorthand to replace the `v-on` directive. `@click="onClickMethod"`
        * Custom events can be used as well.
    * `v-bind` - binds to an attribute of the element. `v-bind:title="title"` binds the `title` data value to the elements' `title` attribute.
        * The '`:`' symbol can be used as shorthand to replace the `v-bind` directive. `:title="title"`
        * This can be done to the class attribute as well. Conditional statements can be added to make the class change more dynamically. i.e. :class="{ 'isLoading' : isLoading }" The isLoading class will be applied if the value of the isLoading variable is true.
    * `v-if` - conditional visibility for an element. Must test for a boolean. This directive will remove the element from the `dom` if it `false`.
    * `v-else` - conditional visibility for an element. Must test for a boolean. Needs to be next to the `v-if` directive element and must be the same kind of element. i.e. (`<p>` or `<div>`)
    * `v-show` - conditional visibility for an element. Must test for a boolean. Similar to `v-if` except if it is `false`, then `display: none` is used instead of removing the element from the dom.
    * `:key` - This needs to be used if you need to track each node's identity and thus reuse or reorder existing elements. A unique Id needs to be used. If you are rendering several elements from the database in a list, you could use the primary key value and bind it to that elements key attribute. i.e. `:key="reply.id"`
* ### Custom Vue Directives
    * Taken from [The Net Ninja](https://www.youtube.com/channel/UCW5YeuERMmlnqo4oq8vwUpg) Youtube video called [Vue JS 2 Tutorial #34 - Custom Directives](https://www.youtube.com/watch?v=3-fLYMEKOU0&list=PL4cUxeGkcC9gQcYgjhBoeQH7wiAyZNrYa&index=36&t=0s)
    * It's possible to add custom `v-` directives that do special things to an element.
        * Example: Make an element have a random text color

            In the `main.js` file
            ```
            Vue.directive('rainbow', {
                bind(el, binding, vnode) {
                    el.style.color = '#' + Math.random.toString().splice(2,8);
                }
            });
            ```
            Where `el` is the element, `binding` is any passed in value or argument and `vnode` is the virtual node.
            
            To use this directive it doesn't require any parameter so simply use `v-raindow` on the element.
            ```
            <h2 v-rainbow>Some Text Here!</h2>
            ```
        * Example: To allow for a theme value to be applied to a root element that changes the max width of the element. Options include "`wide`" or "`narrow`". A `column` argument can also be added, which creates some padding and applies a grey background.

            In the `main.js` file
            ```
            Vue.directive('theme', {
                bind(el, binding, vnode) {
                    if (binding.value == 'wide') {
                        el.style.maxWidth = '1200px';
                    } else if (binding.value = 'narrow') {
                        el.style.maxWidth = '560px';
                    }
                    if (binding.arg == 'column') {
                        el.style.background = '#ddd';
                        el.style.padding = '20px';
                    }
                }
            });
            ```
            To use the directive with either `wide` or `narrow`, you need to pass a string to it. NOTE: Since `objects` and `variables` can also be passed, if a `string` is being used, you need to provide single quotes around it.
            ```
            //apply the wide theme to the div. max-width: 1200px;
            <div v-theme="'wide'"></div>

            //or

            //apply the narrow theme to the div. max-width: 560px;
            <div v-theme="'narrow'"></div>
            
            //or

            //apply the wide theme and the column argument to the div.
            ///max-width: 1200px;, background: #ddd; padding: 20px;
            <div v-theme:column="'wide'"></div>
            ```
            Notice the single quotes around `'wide'` and `'narrow'`.
* ### Axios:
    * Axios uses javascript promises. See section on promise
    * Allows you to use the `then` and `catch` methods after making an Axios request to a given endpoint
* ### Promise (Javascript object)
    * The Promise object represents the eventual completion (or failure) of an asynchronous operation, and its resulting value.
    * Using an example that there is a separate form class that is being used to submit form values using Axios
        * The form class could have a `submit` method that handles the ajax request along with return a new promise to any classes, using the form class, to further process the response.
            ```
            submit(requestType, url) { 
                return new Promise((resolve, reject) => {
                    axios[requestType](url, this.data())
                    .then(response => { 
                        this.onSuccess(response.data);
                        resolve(response.data);
                    }).catch(error => { 
                        this.onFail(error.response.data);
                        reject(error.response.data);
                    });
                }); 
            }
            ```

            * A Vue instance can have a method called `onSubmit` that listens for the form to be submitted then calls the `submit` method on the form class. Then Vue can further process the results of the submitted form by using the returned promise object.
                ```
                onSubmit() { 
                    this.form.submit(
                        'post', 
                        '/someEndPoint'
                    ).then(data => console.log(data))
                    .catch(error => console.log(error)); 
                }
                ```
            * When the Promise object returns a success using the `resolve` method, the promise can be used by the class using the form class by using the `then` method. Just like how Axios handles the `then` method.
            * When the Promise object returns an error using the `reject` method, the promise can be used by the class using the form class by using the `catch` method. Just like how Axios handles the `catch` method.
* ### Vue Components
    * Creates a way to use special rendering on a page using custom elements
    * Important to remember that if there is more than one child element within the component template, then you should wrap all the children in a root tag like a `div`. You only want a single root element
    * **Global component declaration:** 
        * `Vue.component('component-name', {  });`
        * `template: '<li><slot></slot></li>'` whenever `<component-name>Test Component</component-name>` is used, `<li>Test Component</li>` will be used in its place. The `slot` tag is replace with whatever is in between the `component-name` tag. It is important to know that if a template is going to be rendering multiple items (i.e. using a `v-for` directive), then the looped components must be within their own root element. i.e. `template: '<div><task v-for="task in tasks">{{ task.description }}</task></div>'`
        * `data() { return { test: 'testing', isTesting: true } }` `Data` cannot be an object like it is in a normal Vue instance because it isn't bound to anything, therefore it needs to be a method that can return an object of data values. This allows you to bind data to an individual component. It is important to always specify any data that a component uses.
        * `props: [ 'title', 'body' ]` An array containing all the property names that this component accepts. i.e. `<component-name title="test component" body="testing this new component"></component-name>`
            * props can also be an object where you can assign each property of the object certain criteria
                * i.e. `props: { name: { required: true } }` says that we must receive a `name` attribute when this component is used.
                * i.e. `props: { isSelected: { default: false } }`
            * `props` are immutable, meaning they can not be changed once they are set. If you need the values to change you would need to add a different variable to either the `computed` properties or the `data` object.
        * Global components can only communicate with the root instance by using events
            * For example: the component contains a button that changes the visibility of a root element. The button would emit an event and the root element would listen for it. Component: `<button @click="$emit('close')">Close</button> Root: <message v-if="showMessage" @close="showMessage = false"></message>`
    * **Slots:**
        * **Named slots:**
            * `Named slots` and regular `slots` (without a name), can be used together in a template.
            * The `slot` tag, within the component, can be given a `name` attribute. i.e. `<slot name="header"></slot>`
            * To use the named slot within the component tag, you can do 1 of the following:
                * You can just use the `slot` attribute and provide the `name` of the `slot` you want. i.e. 
                ```
                <component-name>
                    <div slot="header">Content Here</div>
                </component-name>
                ```
                This will inject the `div` tag within the component template where the `slot` tag is located within the template. This may not be what you want. For example: you have the slot tag located within a `p` tag in your template. You wouldn't want to inject the entire div tag into the `p` tag. You would want to just inject the the content by itself. See next option below.
                
                * You can use a `template` tag with the `slot` attribute to just insert the content into the `slot` without using the enclosing tag itself. i.e. `<template slot="header">Content here</template>`
        * **Default values in slots.**
            * If you put something within the `slot` tag in your template and then don't provide any content for the `slot`, when calling the component, then the content in the `slot` tag will be used when rendering the component
                * This also works for named `slots` as well.
        * **Scoped slots:**
            * These can be used when you want to override the default formatting between the component tags. Its a good way to provide default formatting then allow for overrides if needed.
                * The front end file where you want the component rendered
                    ```
                    <menu-list :items="['one','two','three']">
                        <template scope="props"> or you can use ES6 destructuring <template scope="{ item }">
                            <h2 v-text="props.item"></h2> or you can use ES6 if used above <h2 v-text="item"></h2>
                        </template>
                    </menu-list>
                    ```

                * The MenuList.vue file
                    ```
                    <template>
                        <ul>
                            <li v-for="item in items">
                                <slot :item="item">{{ item }}</slot>
                            </li>
                        </ul>
                    </template>
                    <script>
                        export default {
                            props: [ 'items' ]
                        }
                    </script>
                    ```
                * If named slots are being used make sure to add the following attributes:
                    * `name="slot-name"` to the `slot` tag in the MenuList.vue file
                    * `slot="slot-name"` to the `template` tag within your display file.
    * **Inline Templates:**
        * When a component is created, you can leave out the `template` property. Then when you call the component you can add the `inline-template` attribute to the tag. i.e. `<component-name inline-template><div>Content Here</div></component-name>`
    * **Event Broadcasting:**
        * When a child component emits an event `this.$emit('someEvent');` The parent element is the only component that can be listening. You can't use this method to emit events to a sibling component. NOTE: You can pass a second argument when emitting an event. `$emit('someEvent', { someOtherDataHere });`
        * To pass an event to a sibling component, you can do the following:
            * Use `window.Event = new Vue();` before the component is declared. i.e. at the top of the .js file.
            * Have a sibling fire off an event using `event.$emit('someEvent'); `
            * Add a listener to a different component (can be the parent or a sibling or another child of the component firing the event) `event.$on('someEvent', () => alert('Event Fired') );`
    * **Component files:**
        * A file can be created that contains all the template details as well as the script and style tags for the component
            * File needs a `.vue` extension. i.e. `componentName.vue`
            * The file needs to contain at least the `template` & `script` tags
                ```
                <template></template>
                <script> 
                export default {  

                }
                </script>
                ```
            * This file needs to be imported into the main file that is using the component using import `"componentName" from "../components/componentName.vue"` This is assuming the main file resides in the following directory `/resources/assets/js/views/` and the component resides in `/resources/assets/js/components/`
            * Also, whenever you use components, locally within the scope of another instance or component, you must let the root instance know about the component by adding the component name to the `components` object in the root instance.
        * If you need to import a component into another component you can do the following:
            * use `import Component from './components/Component.vue';`
            * Then add a components property. `components: { Component },`
            * Now the Component component will be available in the template section of the main component.
** ### Shared State: Sharing data between multiple Vue instances or components
    * There should be a single source of truth
        * Outside of the instances for components you could have a single declaration that will be used in all other required instances.
            * `let store = { user: { name: 'John Doe' } };`
* ### Removing Reactivity from a property
    * When you want to clone a property in a component but want to disconnect the reactivity
        * Doesn't remove reactivity
            ```
            data() {
                return {
                    myData: { somedata: somevalue, somemoredata: somemorevalue }
                }
            },
            methods: {
                updateMe() {
                    let myNewData = this.myData;
                    myNewData.someData = somenewvalue;
                }
            }
            ```
            The new `myNewData` property will still be linked to the `myData` property and any changes made to `myNewData` will be reflected in `myData`. `myNewData.someData == myData.someData`
        * One way to handle this issue
            ```
            data() {
                return {
                    myData: { somedata: somevalue, somemoredata: somemorevalue }
                }
            },
            methods: {
                updateMe() {
                    let myNewData = JSON.parse(JSON.stringify(this.myData));
                    myNewData.someData = somenewvalue;
                }
            }
            ```
            By stringifying `myData` and then parsing it back to an object then the reactivity is removed. `myData`. `myNewData.someData != myData.someData`
        * lodash supposedly has function to do things similar to this but I don't have any example.
* ### Compound Words within Components
    * If you start to see that you are using a lot of compound words within your components, then it may be an indicator that you should refactor that logic to its own component.
        * Using the Forum series, the Reply component has some favoriting elements within it. There are several method and properties that start with favorites. i.e. `favoritesCount`, `toggleFavorite`, etc. A new `Favorite` component would be a good use case for this.
            * Create a **Favorite.vue** file within the components directory
            * **Reply.vue**
                * before the export default section add `import Favorite from './Favorite.vue';`
                * add a new `components` property. `components: { Favorite },`
                * move the `favoritesCount` data property over to `data()` in `Favorite.vue`
                * move the `isFavorited`  data property over to `data()` in `Favorite.vue`
                * move the `favoriteClasses` computed property over to `computed` property in `Favorite.vue`
                * move the `toggleFavorite`, `favorite` & `unfavorite`  methods over to `methods` section in `Favorite.vue`
            * **Favorite.vue**
                * `props: [ 'reply' ];`
                    * change any reference to `this.attributes` to `this.reply`
                * for the template section of the component, copy in the entire `button` block from `reply.blade.php` and replace it with `<favorite :reply="{{ $reply }}"></favorite>`
                * change all references of `favoriteClasses` to `classes`
                * change all references of `toggleFavorite` to `toggle`
                * change all references of `favoritesCount` to `count` (make sure to leave `this.reply.favoritesCount` alone in the `data` object, since this references the Laravel model.)
                * change all references of `isFavorited` to `active` (make sure to leave `this.reply.isFavorited` alone in the `data` object, since this references the Laravel model.)
* ### Using Async and Await
    * When you add the word async in front of a method, you are saying that this is an asyncronous method and it should return a promise.
    * Below is an [example](https://laracasts.com/lessons/await-please) where JW had a method in a view component that allowed a user to report spam. It is using a SweetAlert dialog box that waits for the user to click yes before an axios call is used to process the report for spam.
        
        Vue component before using `async`/`await`
        ```
        methods: {
            notifyAdminstrator() {
                swal({
                    title: 'Spam?',
                    text: this.message,
                    icon: 'warning',
                    buttons: ['Cancel', 'Yes']
                }).then(confirmed => {
                    if (! confirmed) return;

                    axios.post(this.url).then(() => {
                        flash('Okay, thanks for the help!')
                    });
                });
            }
        }
        ```
        Same Vue component with using `async`/`await`. Using the `await` keyword within a `async` function, the execution is stopped until a promise is received. SweetAlert returns a promise as well as axios, so the `.then` can be removed entirely.
        ```
        methods: {
            async notifyAdminstrator() {
                let confirmed = await swal({
                    title: 'Spam?',
                    text: this.message,
                    icon: 'warning',
                    buttons: ['Cancel', 'Yes']
                });
                
                if (! confirmed) return;

                await axios.post(this.url);

                flash('Okay, thanks for the help!')
            }
        }
        ```
        The await keyword cannot be used without async. It will throw an error if you try. `await` is the same as saying, "wait here until a promise is received." Also, if you try to return a value (i.e. `return 'Hi There!';`) at the end of the function that is using `async`, `"Hi There!"` will not be returned but a promise that resolve the value will be.
* ### Miscellaneous:
    * **IMPORTANT UPDATE WITH VUE v2.5.21^ & WEBPACK/MIX 4**
        * When registering a component you need to add `.default` after the `require` method.
            * OLD: `Vue.component('flash', require('./components/Flash.vue'));`
            * NEW: `Vue.component('flash', require('./components/Flash.vue').default);`
    * If you wan to share a Vue method amongst all Vue components, then you can put the following in the `/resources/assets/js/bootstrap.js` file after the `window.Vue = require('vue');`
        * `window.Vue.prototype.someFunctionName = function (attributes) { // function logic here }`
    * To access the `key` value in a `v-for` loop (`v-for="(item, index) in items" :key="index"`) you need to use `$node`. i.e. `this.$node.key`.
    * If you are simply creating a Vue project and not using Laravel, you can use `cue-loader` to help create the project and do all the `ecmascript` conversions. It also allows you to use the `.vue` component files.
        * <https://vue-loader.vuejs.org/>
    * To register a Vue plugin, you need to add the following to you `app.js` file or the `bootstrap.js` file. (Using `Portal Vue` for this example. See below for details on the plugin.
        * `import PortalVue from 'portal-vue';`
        * `Vue.use(PortalVue);`
    * There is a plugin called [Portal Vue](https://github.com/LinusBorg/portal-vue) that allows you to render a template anywhere in the dom
        * This is handy to if you want to keep all parts of component in the same `.vue` file but need part of the component to render somewhere else on the page.
        * The example from the series has a mega menu being rendered later after a nav section but the section template is stored with the template that displays the link to activate the menu.
    * Another useful plugin for using `modals` is [Vue js Modal](https://github.com/euvl/vue-js-modal)
    * A very handy `alert plugin` for javascript (not Vue dependent) is called [SweetAlert](https://sweetalert.js.org/) or on [Github](https://github.com/t4t5/sweetalert)
        * This can also be used to handle confirmations as well since a promise is returned when using it.
    * When using Vue and you need access to a `doc node`, then you can use the `ref` attribute on that `node` and give it a name (i.e. `<div ref="this-is-my-node">`)
        * You can then access that `ref` by using `this.$refs['this-is-my-node']`
    * To reference the root element in a component you can use `this.$el`
    * `Tooltip.js` is a good tool to use for tool tips. It is part of the `popper.js` library but can be installed on it's own. <https://popper.js.org/tooltip-examples.html>
        * `npm install tooltip.js --save`
    * Different binding options: 
        * using a `.` after a directive is considered a modifier. `v-{directive}.someModifier` will add `someModifier` to the `modifier` object of the `bindings` object
        * using a `:` after a directive is considered an `arg`. `v-{directive}:someArg` will add `someArg` to the `arg` object of the `bindings` object
    * Handy npm utility that can be used to perform an action only when the target element is visible in the viewport: [inViewport](https://www.npmjs.com/package/in-viewport)
    * Vue Transitions can be used by wrapping the element you want to apply the transition to within a `<transition>` tag. <https://vuejs.org/v2/guide/transitions.html>
        * Vue will only apply the transition when the containing elements are visible.
        * You can also pass a `name` variable in the tag, which is used to specify the type of transition to use. `<transition name="fade"><div v-show="isVisible><p>show me</p></div></transition>`
    * Hide a Vue component while the page loads
        * In your css file add `[v-cloak] { display: none }`
        * Add `v-cloak` to the root Vue element.
            * `<div id="app" v-cloak>`
    * Within a style tag in a component you can add the "`scoped`" attribute that will scope all the styles listed within the tag just to be available to the component and not outside of it.
        * `<style type="text/css" scoped></style>`
    * Turbolinks is a package that intercepts a page click via an a tag and injects the linked page into the current page without doing a redirect. [Turbolinks on Github](https://github.com/turbolinks/turbolinks)
        * Install the package
            ```
            npm install --save turbolinks
            ```
        * Add the following to your bootstrap.js file to have Turbolinks start up
            ```
            import Turbolinks from 'turbolinks';

            Turbolinks.start();
            ```
        * If you import all your scripts at the bottom of the `body` tag then the script tag should be moved to the `head` tag instead.
            ```
            // remove from the end of the body tag and place before the end of the head tag
            <script src="{{ asset('js/app.js') }}"></script>
            ```
        * There is a problem with Turbolinks losing reactivity with some Vue components. See [video](https://laracasts.com/series/whatcha-working-on/episodes/15) on Laracasts or specific example. Below are the ways to fix
            * Install [vue-turbolinks](https://github.com/jeffreyguenther/vue-turbolinks), which is a Turbolinks adapter for Vue components.

                In your app.js file after the import for bootstrap.js, import the package
                ```
                import TurbolinksAdapter from 'vue-turbolinks';
                ```
                Now where the new Vue instance is assigned to the app constant, you need to wrap the entire block in an event listener. Below is the before and after
                ```
                // before
                const app = new Vue({
                    el: '#app'
                });

                // after
                document.addEventListener('turbolinks:load', () => {
                    let app = new Vue({
                        el: "#app",
                        mixins: [TurbolinksAdapter],
                    });
                });
                ```
* ### Miscellaneous Javascript Tips
    * Add event listener
        * `window.addEventListener('event name', function() {  });` or if using `ES6` `window.addEventListener('event name', () => {  });`
            * one event would be '`scroll`' to detect when a user is scrolling
        * You can pass a third argument to `addEventListener` `window.addEventListener('event name', this.someFunction, { passive: true });`
            * `passive` tells the browser that you want a `passive` event listener. This will greatly improve the scrolling performance.
    * When using **Chrome Dev Tools**:
        * When you select a `node` (i.e. a `div`) and then go to the console, you can reference that selected `div` by using `$0`
        * Then you can do things like `$0.offsetTop` to get the location, relative to the top of the screen for that selected element.
    * `window.scrollY` or `window.scrollX` will give you the location of that you are current scrolling at.
    * To modify class names on a `node` you can use `classList`: `{node}.classList`
        * then you can `add` more by doing `.add('class1', 'class2', 'class3');`
        * or to `remove` a list of classes use: `.remove('class2', 'class3');`
    * To import a specific method or subclass of a library you can use: `import { method } from 'libraryName';`
        * `import { throttle } from 'lodash';` will import the `throttle` function from `LoDash`.
    * To grab all elements on the page that have an attribute called `data-tooltip` you can use:
        * `document.querySelectorAll('[data-tooltip]')`
    * To get all the `data` attributes of an element you can use `.dataset`. This will return all the `data-` attributes that are set. i.e. `data-tooltip`, etc.
        * `$el.dataset.tooltip` will return the value of the `data-tooltip` attribute.
        * If you have a kabob cased attribute you can reference it with camelcase.
            * `$el.dataset.tooltipPlacement` will return the value for `data-tooltip-placement`
    * Say there is a time when you want to call a method within a component or class but you want the method to refer to the current instance instead of the passed in object when using `this`.
        ```
        submit(endpoint) {
            return axios.post(endpoint, this.data())
                .catch(this.onFail.bind(this));
        }
        onFail(error) {
            this.errors = error.response.data.errors;
        }
        ```
        Without adding `.bind(this)` when call `this.onFail`, this is referencing the passed in axios object instead of the local instance when using `this`. Using `.bind(this)` will bind the call to the local instance.
* ### Vue Testing:
    * **SETUP:**
        * `ava` is a package that is recommended. Import `ava` using `npm install ava --save-dev`
        * To use `ES2017` syntax you'll need `Babel-Register`. Allows us to render the `ES2017` syntax on the fly. None of the following code will work without adding `babel-register` first.
            * `npm install babel-register --save-dev`
            * In the `package.json` file you'll need to add:
                * `"babel": { "presets": [ "es2015" ] }` or `run npm install babel-preset-es2015 --save-dev`
                * `"ava": { "require": [ "babel-register", "./test/helpers/setup-browser-env.js" ] }` The helper file needs to be created. See below
        * Since this test example is using a node module and not a browser we need `browser-env` to mimic a browser
            * `npm install browser-env --save-dev`
        * Set up `setup-browser-env.js` file.
            ```
            import browser-env from 'browser-env';
            browserEnv();
            ```
        * To execute a test the `ava` executable is located in `node_modules/.bin/ava`
    * Consider the following example:
        * File located in `/src` contains module called `Notification.js`
            * `export default { data() { return { message: 'Hello World" } } }`
        * Test file located in `/test` contains `notification.js`
            ```
            import Vue from 'vue'; 
            //This version is for the runtime only version. 
            //It may be more preferred to use 'vue/dist/vue.js'.
            //This is needed if you are going to be using templates in your modules.

            import test from "ava';
            import Notification from '../src/Notification.js';
            test('that it renders a notification', t => {
                t.is(Notification.data().message, 'Hello World');
            });
            ```
        * To test if a prop is sent to the module correctly.
            * in `Notification.js`, get rid of the `data` method all together and add:
                ```
                template: '<div><h1>{{ message }}</h1><div>',
                props: [ 'message' ]
                ```
            * inside the `t` object of the `notification.js` test file, you should have the following:
                ```
                let N = Vue.extend(Notification);

                let vm = new N({ propsData: {
                    message: 'Foobar'
                }).$mount(); 
                //$mount is needed to manually mount the Vue instance.

                t.is(vm.$el.textContent, 'Foobar');
                ```
            * If you were to want to reuse the extend code in multiple different tests you could use the `beforeEach` method to have this code run before each test. The following would be before any test are defined.
                ```
                let vm;
                test.beforeEach(t => {
                    let N = Vue.extend(Notification);
                });
                ```
                
                Now you can get rid of this line from each of your test but also pass different values to the message prop.
