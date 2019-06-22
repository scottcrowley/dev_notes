# Vue 2 Examples

* ### [Practical Vue Components](https://laracasts.com/series/practical-vue-components) examples
    * **Smooth Scrolling** - [Episode #1](https://laracasts.com/series/practical-vue-components/episodes/1)
        * This is an example of how to make a link scroll to a link on a page smoothly. This example is using a CDN version of `tailwind.js` 1.0 official release.

            ***`resources/js/app.js`***
            ```js
            import Vue from 'vue';
            import ScrollLink from './components/ScrollLink';

            window.Vue = Vue;

            Vue.component('scroll-link', ScrollLink);

            new Vue({
                el: '#app'
            });
            ```

            *`resources/js/components/ScrollLink.vue`*
            ```vue
            <template>
                <a :href="href" @click.prevent="scroll">
                    <slot></slot>
                </a>
            </template>

            <script>
                export default {
                    props: ['href'],

                    methods: {
                        scroll() {
                            // For older browsers, consider pulling in a polyfill.
                            // https://www.npmjs.com/package/smoothscroll-polyfill

                            document.querySelector(this.href)
                                    .scrollIntoView({ behavior: 'smooth' });
                        }
                    }
                }
            </script>
            ```

            ***`resources/views/smooth-scroll.php`***
            ```html
            <!doctype html>
            <html lang="en">
            <head>
                <meta charset="UTF-8">
                <meta name="viewport"
                    content="width=device-width, user-scalable=no, initial-scale=1.0, maximum-scale=1.0, minimum-scale=1.0">
                <meta http-equiv="X-UA-Compatible" content="ie=edge">
                <link href="https://unpkg.com/tailwindcss@^1.0/dist/tailwind.min.css" rel="stylesheet">

                <title>Smooth Scrolling</title>
            </head>
            <body>
                <div id="app" class="p-8">
                    <h1 class="text-2xl font-bold">Smooth Scroll</h1>

                    <scroll-link href="#categories" class="text-blue-500">Go To Testimonials</scroll-link>

                    <div style="height: 2000px"></div>

                    <div id="categories">
                        <h2 class="font-bold mb-6">Testimonials</h2>

                        <div class="flex">
                            <div class="w-1/3 h-48 bg-gray-200 p-4">
                                <scroll-link href="#app" class="text-blue-500">Go Back Up</scroll-link>
                            </div>
                            <div class="w-1/3 h-48 bg-gray-400 p-4">Item</div>
                            <div class="w-1/3 h-48 bg-gray-200 p-4">Item</div>
                        </div>
                    </div>
                </div>

                <script src="/js/app.js"></script>
            </body>
            </html>
            ```
    * **Context Menus** - [Episode #2](https://laracasts.com/series/practical-vue-components/episodes/2)
        * This is an example of how to make a contextual menu dropdown menu with mild animations. This example is using a CDN version of `tailwind.js` 1.0 official release.
            
            ***`resources/js/app.js`***
            ```js
            import Vue from 'vue';
            import Dropdown from './components/Dropdown';

            window.Vue = Vue;

            Vue.component('dropdown', Dropdown);

            new Vue({
                el: '#app'
            });
            ```

            ***`resources/js/components/Dropdown.vue`***
            ```vue
            <template>
                <div class="dropdown relative">
                    <div class="dropdown-trigger"
                        @click.prevent="isOpen = ! isOpen"
                        aria-haspopup="true"
                        :aria-expanded="isOpen"
                    >
                        <slot name="trigger"></slot>
                    </div>

                    <transition name="pop-out-quick">
                        <ul v-show="isOpen"
                            class="dropdown-menu absolute bg-black mt-2 py-2 rounded shadow text-white z-10"
                            :class="classes"
                        >
                            <slot></slot>
                        </ul>
                    </transition>
                </div>
            </template>

            <script>
                export default {
                    props: ['classes'],
                    data() {
                        return {
                            isOpen: false
                        };
                    },
                    watch: {
                        isOpen(isOpen) {
                            if (isOpen) {
                                document.addEventListener(
                                    'click',
                                    this.closeIfClickedOutside
                                );
                            }
                        }
                    },
                    methods: {
                        closeIfClickedOutside(event) {
                            if (! event.target.closest('.dropdown')) {
                                this.isOpen = false;
                                document.removeEventListener('click', this.closeIfClickedOutside);
                            }
                        }
                    }
                }
            </script>

            <style>
                .pop-out-quick-enter-active,
                .pop-out-quick-leave-active {
                    transition: all 0.4s;
                }
                .pop-out-quick-enter,
                .pop-out-quick-leave-active {
                    opacity: 0;
                    transform: translateY(-7px);
                }
            </style>
            ```
            
            ***`resources/views/context-menu.php`*** - example page to show off the menu
            ```html
            <!doctype html>
            <html lang="en">
            <head>
                <meta charset="UTF-8">
                <meta name="viewport"
                    content="width=device-width, user-scalable=no, initial-scale=1.0, maximum-scale=1.0, minimum-scale=1.0">
                <meta http-equiv="X-UA-Compatible" content="ie=edge">
                <link href="https://unpkg.com/tailwindcss@^1.0/dist/tailwind.min.css" rel="stylesheet">

                <title>Smooth Scrolling</title>
            </head>

            <body>
                <div id="app" class="flex flex-col items-center p-8">
                    <h1 class="text-2xl font-bold mb-8">Context Menu</h1>

                    <div>
                        <div class="bg-gray-400 w-64 h-64 flex items-center justify-center">
                            <!-- Example 1 -->
                            <dropdown>
                                <template v-slot:trigger>
                                    <button class="hover:text-blue-500">...</button>
                                </template>

                                <li><a href="#" class="pl-2 pr-8 leading-loose text-xs block hover:bg-gray-900">Edit</a></li>
                                <li><a href="#" class="pl-2 pr-8 leading-loose text-xs block hover:bg-gray-900">Delete</a></li>
                                <li><a href="#" class="pl-2 pr-8 leading-loose text-xs block hover:bg-gray-900">Report</a></li>
                            </dropdown>
                        </div>
                    </div>

                    <hr>

                    <!-- Example 2 -->
                    <dropdown classes="w-full">
                        <template v-slot:trigger>
                            <a class="text-blue-500">Example With Full Width Menu<button>
                        </template>

                        <li><a href="#" class="pl-2 pr-8 leading-loose text-xs block hover:bg-gray-900">Edit</a></li>
                        <li><a href="#" class="pl-2 pr-8 leading-loose text-xs block hover:bg-gray-900">Delete</a></li>
                        <li><a href="#" class="pl-2 pr-8 leading-loose text-xs block hover:bg-gray-900">Report</a></li>
                    </dropdown>
                </div>

                <script src="/js/app.js"></script>
            </body>
            </html>
            ```
    * **Show an Element When Another is Hidden** - [Episode #3](https://laracasts.com/series/practical-vue-components/episodes/3)
        * This example shows how to display a block of html only when another element has been hidden or scrolled past on the screen. This can be used to have a "Add Discussion" button when a user scrolls past the normal button to add a discussion.
            * Important to note that anytime a "scroll" listener is added to the window or document, this check needs to be throttled or there will be an insane amount of calls to the callback function processing the event. Below are options on how to avoid this.
                * Lodash debounce or some sort of throttling technique should be used
                * If you are not going to be messing with the actual scroll position then you can use the `passive` option on the listener
                    ```js
                    window.addEventListener('scroll' () => {
                        console.log('hello there');
                    }, { passive: true });
                    ```
        * This example is using a third party npm package called [in-viewport](https://github.com/vvo/in-viewport), to check for whether or not the requested element is in the viewport or not. Due to all the different browser constraints, it's better to have a external package handle this logic.
            ```
            npm install in-viewport
            ```
            To open the documentation for this package, you can open it by using the following command. It will open in your default browser
            ```
            npm repo in-viewport
            ```
            In the document that is using the logic, you need to import `in-viewport` and then pass an element to the `InViewport` function.
            ```js
            import inViewport from 'in-viewport';
            let elem = document.getElementById('myFancyDiv');
            let isInViewport = inViewport(elem); //returns bool or you can pass a second param as a callback
            ```
        * A transition is being used to display the button that has the visibility toggled on. View the full [Vue transition documentation](https://vuejs.org/v2/guide/transitions.html)
            * Wrap the template content in a transition component. The name attribute is being used to hook into the transitions' events. There are many events or phases that a transition uses which will add various classes to the transition component.
                * `enter-active`, `leave-active`, `enter`, `leave-to`. See [documentation](https://vuejs.org/v2/guide/transitions.html#Transition-Classes) for more details.
                * Whatever is assign to the `name` attribute can be added to the beginning of the class names above. In this case, `fade` is assigned to the `name` attribute so `fade-` will be added to the class names. e.g. `fade-enter-active` or `fade-leave-to`, etc

                ***Template section***
                ```vue
                <template>
                    <transition name="fade">
                        <div v-show="shouldDisplay">
                            <slot></slot>
                        </div>
                    </transition>
                </template>
                ```

                ***Style section***
                ```vue
                <style>
                    .fade-enter-active, .fade-leave-active {
                        transition: opacity .3s;
                    }
                    .fade-enter, .fade-leave-to {
                        opacity: 0;
                    }
                </style>
                ```
            Add the following to *`resources/js/app.js`*    
            ```js
            import Visible from './components/Visible';
            Vue.component('visible', Visible);
            ```

            ***`resources/js/components/Visible.vue`***
            ```vue
            <template>
                <transition name="fade">
                    <div v-show="shouldDisplay">
                        <slot></slot>
                    </div>
                </transition>
            </template>

            <script>
                import inViewport from 'in-viewport';
                export default {
                    props: ['whenHidden'],
                    data() {
                        return {
                            shouldDisplay: false
                        }
                    },
                    mounted() {
                        window.addEventListener('scroll', () => {
                            this.shouldDisplay = ! inViewport(
                                document.querySelector(this.whenHidden)
                            );
                        }, { passive: true });
                    }
                }
            </script>

            <style>
                .fade-enter-active, .fade-leave-active {
                    transition: opacity .3s;
                }
                .fade-enter, .fade-leave-to {
                    opacity: 0;
                }
            </style>
            ```

            ***`resources/views/conditional-visibility.php`***
            ```html
            <!doctype html>
            <html lang="en">
            <head>
                <meta charset="UTF-8">
                <meta name="viewport"
                    content="width=device-width, user-scalable=no, initial-scale=1.0, maximum-scale=1.0, minimum-scale=1.0">
                <meta http-equiv="X-UA-Compatible" content="ie=edge">
                <link href="https://unpkg.com/tailwindcss@^1.0/dist/tailwind.min.css" rel="stylesheet">

                <title>Conditional Visibility</title>
            </head>

            <body>
                <div id="app" class="relative flex flex-col items-center p-8">
                    <h1 class="text-2xl font-bold mb-8">Conditional Visibility</h1>

                    <div class="container w-3/4 bg-gray-200 p-4" style="height: 2000px">
                        <a
                            id="new-post-link"
                            href="#"
                            class="text-blue-500"
                        >New Post</a>

                        <visible when-hidden="#new-post-link">
                            <button
                                class="bg-blue-500 hover:bg-blue-600 rounded-full w-24 h-24 text-white text-4xl fixed z-10 right-0 bottom-0 mr-4 mb-4"
                            >+</button>
                        </visible>
                    </div>
                </div>

                <script src="/js/app.js"></script>
            </body>
            </html>
            ```
    * <a id="ep4">**Modals and Custom Vue Plugins**</a> - [Episode #4](https://laracasts.com/series/practical-vue-components/episodes/4)
        * This example expands on the [Modern CSS for Backend Developers:Modals with Zero JavaScript](https://laracasts.com/series/modern-css-for-backend-developers/episodes/17) CSS example of making a modal with no javascript. Now JW shows how to add the concept to have it reside in a Vue component.
            
            ***`resources/js/plugins/modal/Component.vue`***
            ```vue
            <template>
                <div :id="name" class="overlay text-left">
                    <a href="#" class="cancel"></a>

                    <div class="modal">
                        <slot></slot>

                        <footer class="flex mt-8">
                            <slot name="footer"></slot>
                        </footer>

                        <a href="#" class="close">&times;</a>
                    </div>
                </div>
            </template>

            <script>
                export default {
                    props: ['name']
                }
            </script>

            <style>
                .overlay {
                    visibility: hidden;
                    position: absolute;
                    top: 0;
                    right: 0;
                    bottom: 0;
                    left: 0;
                    display: flex;
                    align-items: center;
                    justify-content: center;
                    background: rgba(0, 0, 0, .4);
                    transition: opacity .3s;
                    opacity: 0;
                }

                .overlay:target {
                    visibility: visible;
                    opacity: 1;
                }

                .modal {
                    position: relative;
                    width: 500px;
                    max-width: 80%;
                    background: white;
                    border-radius: 4px;
                    padding: 2.5em;
                    box-shadow: 0 5px 11px rgba(36, 37, 38, 0.08);
                }

                .modal .close {
                    position: absolute;
                    top: 15px;
                    right: 15px;
                    color: grey;
                    text-decoration: none;
                }

                .overlay .cancel {
                    position: absolute;
                    width: 100%;
                    height: 100%;
                }
            </style>
            ```

            ***`resources/views/modal.php`***
            ```html
            <!doctype html>
            <html lang="en">
            <head>
                <meta charset="UTF-8">
                <meta name="viewport"
                    content="width=device-width, user-scalable=no, initial-scale=1.0, maximum-scale=1.0, minimum-scale=1.0">
                <meta http-equiv="X-UA-Compatible" content="ie=edge">
                <link href="https://unpkg.com/tailwindcss@^1.0/dist/tailwind.min.css" rel="stylesheet">

                <title>Modal</title>

                <style> body { font-family: sans-serif; } </style>
            </head>

            <body class="p-10">
                <div id="app" class="text-center">
                    <h1 class="text-2xl font-bold mb-8">Modal</h1>

                    <p>
                        <a href="#cancel-modal" class="text-blue-500 underline">Open Modal</a>
                    </p>

                    <modal name="cancel-modal">
                        <h1 class="font-bold text-xl mb-2">Leaving So Soon?</h1>

                        <p>
                            Lorem ipsum dolor sit amet, consectetur adipisicing elit, sed do eiusmod
                            tempor incididunt ut labore.
                        </p>

                        <template v-slot:footer>
                            <a href="#" class="bg-gray-500 py-2 px-4 rounded-lg text-white hover:bg-gray-600 mr-2">Cancel</a>
                            <a href="#confirm-cancel-modal" class="bg-blue-500 py-2 px-4 rounded-lg text-white hover:bg-blue-600" >Confirm Cancellation</a>
                        </template>
                    </modal>

                    <modal name="confirm-cancel-modal">
                        <h1 class="font-bold text-xl mb-2">You're 100% Sure?</h1>

                        <p>
                            Lorem ipsum dolor sit amet, consectetur adipisicing elit, sed do eiusmod
                            tempor incididunt ut labore.
                        </p>

                        <template v-slot:footer>
                            <a href="#" class="bg-gray-500 py-2 px-4 rounded-lg text-white hover:bg-gray-600 mr-2">Cancel</a>
                            <a href="#" class="bg-blue-500 py-2 px-4 rounded-lg text-white hover:bg-blue-600">Yes</a>
                        </template>
                    </modal>
                </div>

                <script src="/js/app.js"></script>
            </body>
            </html>
            ```

            ***`resources/js/plugins/modal/ModalPlugin.js`***
            ```js
            import Component from './Component';

            let Plugin = {
                install: function (Vue, options = {}) {
                    Vue.component('modal', Component);

                    Vue.prototype.$modal = {
                        show(name) {
                            location.hash = name;
                        },

                        hide(name) {
                            location.hash = '#';
                        }
                    }
                }
            };

            export default Plugin;
            ```

            ***`resources/js/app.js`***
            ```js
            import Modal from './components/Modal';

            Vue.component('modal', Modal);
            ```
    * **Confirmation Dialogs and Buttons** - [Episode #5](https://laracasts.com/series/practical-vue-components/episodes/5)
        * This example works with confirmation dialog boxes and buttons. This is using the modal that was created in [Episode 4 - "Modals and Custom Vue Plugins"](#ep4)

            ***`resources/js/app.js`***
            ```js
            import Vue from 'vue';

            import Modal from './plugins/modal/ModalPlugin';

            import ScrollLink from './components/ScrollLink';
            import Dropdown from './components/Dropdown';
            import Visible from './components/Visible';
            import ConfirmButton from './components/ConfirmButton';
            import ConfirmDialog from './components/ConfirmDialog';

            window.Vue = Vue;

            Vue.use(Modal);

            Vue.component('scroll-link', ScrollLink);
            Vue.component('dropdown', Dropdown);
            Vue.component('visible', Visible);
            Vue.component('confirm-button', ConfirmButton);
            Vue.component('confirm-dialog', ConfirmDialog);

            new Vue({
                el: '#app',

                methods: {
                    confirm(message) {
                        this.$modal.dialog(message)
                            .then(confirmed => {
                                if (confirmed) {
                                    // Proceed. Submit ajax request, etc.
                                    alert('Proceed');
                                } else {
                                    // Optionally override the button visibility and labels.
                                    this.$modal.dialog('Okay, canceled', {
                                        cancelButton: 'Close',
                                        confirmButton: false
                                    });
                                }
                            });
                    }
                }
            });
            ```

            ***`resources/js/components/ConfirmButton.vue`***
            ```vue
            <template>
                <button @click="confirm">
                    <slot></slot>
                </button>
            </template>

            <script>
                export default {
                    props: {
                        message: {},
                        confirmButton: { default: 'Continue' },
                        cancelButton: { default: 'Cancel' }
                    },
                    data() {
                        return { confirmed: false };
                    },
                    methods: {
                        confirm(e) {
                            if (this.confirmed) {
                                return;
                            }
                            e.preventDefault();
                            this.$modal.dialog(this._props)
                                .then(confirmed => {
                                    this.confirmed = confirmed;
                                    if (confirmed) {
                                        this.$el.click();
                                    }
                                });
                        }
                    }
                }
            </script>
            ```

            ***`resources/js/components/ConfirmDialog.vue`***
            ```vue
            <template>
                <modal name="dialog">
                    {{ params.message }}

                    <template v-slot:footer>
                        <button
                            class="bg-gray-500 hover:bg-gray-600 py-2 px-4 text-white rounded-lg mr-2"
                            @click.prevent="handleClick(false)"
                            v-if="params.cancelButton"
                            v-text="params.cancelButton"
                        >
                        </button>

                        <button
                            class="bg-blue-500 hover:bg-blue-600 py-2 px-4 text-white rounded-lg"
                            @click.prevent="handleClick(true)"
                            v-if="params.confirmButton"
                            v-text="params.confirmButton"
                        >
                        </button>
                    </template>
                </modal>
            </template>

            <script>
                import Modal from '../plugins/modal/ModalPlugin';
                export default {
                    data() {
                        return {
                            params: {
                                message: 'Are you sure?',
                                confirmButton: 'Continue',
                                cancelButton: 'Cancel'
                            }
                        };
                    },
                    beforeMount() {
                        Modal.events.$on('show', params => {
                            Object.assign(this.params, params);
                        });
                    },
                    methods: {
                        handleClick(confirmed) {
                            Modal.events.$emit('clicked', confirmed);
                            this.$modal.hide();
                        }
                    }
                }
            </script>
            ```

            ***`resources/js/plugins/modal/ModalPlugin.js`***
            ```js
            import Component from './Component';

            let Plugin = {
                install: function (Vue, options = {}) {
                    Vue.component('modal', Component);

                    Plugin.events = new Vue();

                    Vue.prototype.$modal = {
                        show(name, params = {}) {
                            location.hash = name;

                            Plugin.events.$emit('show', params);
                        },

                        hide(name) {
                            location.hash = '#';
                        },

                        dialog(message, params = {}) {
                            if (typeof message === 'string') {
                                params.message = message;
                            } else {
                                params = message;
                            }

                            return new Promise((resolve, reject) => {
                                this.show('dialog', params);

                                Plugin.events.$on(
                                    'clicked', confirmed => resolve(confirmed)
                                );
                            });
                        }
                    }
                }
            };

            export default Plugin;
            ```

            ***`resources/views/confirmation-button.php`***
            ```html
            <!doctype html>
            <html lang="en">
            <head>
                <meta charset="UTF-8">
                <meta name="viewport"
                    content="width=device-width, user-scalable=no, initial-scale=1.0, maximum-scale=1.0, minimum-scale=1.0">
                <meta http-equiv="X-UA-Compatible" content="ie=edge">
                <link href="https://unpkg.com/tailwindcss@^1.0/dist/tailwind.min.css" rel="stylesheet">

                <title>Confirmation Dialogs</title>
            </head>

            <body class="font-sans p-10">
                <div id="app" class="text-center">
                    <h1 class="text-2xl font-bold mb-8">Confirmation Dialogs</h1>

                    <div class="mb-6">
                        <form method="POST">
                            <confirm-button
                                message="Are you sure you want to cancel your account?"
                                class="bg-blue-500 hover:bg-blue-600 py-2 px-4 text-white rounded-lg"
                            >
                                Option 1
                            </confirm-button>
                        </form>
                    </div>

                    <div class="mb-6">
                        <form method="POST">
                            <confirm-button
                                message="Are you sure you want to cancel your account?"
                                cancel-button="Go Back"
                                confirm-button="Continue On"
                                class="bg-blue-500 hover:bg-blue-600 py-2 px-4 text-white rounded-lg"
                            >
                                Option 2
                            </confirm-button>
                        </form>
                    </div>

                    <div class="mb-6">
                        <form method="POST" @submit.prevent="confirm('Are you really sure about this?')">
                            <button class="bg-blue-500 hover:bg-blue-600 py-2 px-4 text-white rounded-lg">
                                Option 3
                            </button>
                        </form>
                    </div>

                    <confirm-dialog></confirm-dialog>
                </div>

                <script src="/js/app.js"></script>
            </body>
            </html>
            ```

            ***Add the following to the routes file `routes/web.php`***
            ```php
            Route::post('confirmation-button', function () {
                return 'Form submitted';
            });
            ```
* ### Custom Input Example:
    * Say you want to be able to reuse an input that has validation logic, sanitizing, etc attached to it. i.e. `<coupon></coupon>` instead of `<input type="text" v-model="coupon">`
        * On your page you can still use `v-model` on the component `<coupon v-model="coupon"></coupon>`
            * In you js file:
                ```js
                Vue.component('coupon', {
                    props: ['code'],
                    template: `<input type="text" :value="code" @input="updateCode($event.target.value)" ref="input">`,
                    data() {
                        return {
                            invalid: ['ALLFREE', 'SOMETHINGELSE']
                        }
                    },
                    methods: {
                        updateCode(code) {
                            //Validation logic here
                            if(this.invalids.includes(code)) { 
                                alert('This coupon is no longer valid. Sorry!');
                                this.$ref.input.value = code = '';
                            }
                            this.$emit('input', code);
                        }
                    }
                }
                ```
* ### Creating a Collection Vue Subclass
    * This is handy when you are dealing with adding or removing items from data collections frequently. Using the Forum Series as an example, the `Replies.vue` uses collections to add and remove data from it.
        * create a `Collections.js` file within `resources/assets/js`.
            ```js
            export default Vue.extend({
                props: [ 'data' ],
                data() {
                    return {
                    items: this.data
                }
                },
                    methods() {
                        add(item) {
                            this.items.push(item);
                            this.$emit('added');
                        },
                        remove(index) {
                            this.items.splice(index, 1);
                            this.$emit('removed');
                    }
                }
            });
            ```
        * with the `Replies.vue` file:
            * `import Collection from '../Collection';`
            * replace `export default {` with `export default Collection.extend({`
            * Add the closing parentheses following the last `}`
            * The entire `methods` object can now be removed since it is being handled by the `Collection` subclass
            * The entire `props` property can be removed as well
            * Remove items: `this.data`, from the `data` object.
* ### Adding separate form handling class to a component
    * If you are going to have many forms you can create a separate class that handles all the basic form functionality so the code doesn't need to be repeated in other components that are using forms. Taken from Laracasts [Build a Laravel App with TDD:Object-Oriented Javascript](https://laracasts.com/series/build-a-laravel-app-with-tdd/episodes/42)
        
        Create a new FormHandler class in a new file called FormHandler.js
        ```js
        class FormHandler {
            constructor(data) {
                //originalData contains all the attributes and their default values
                this.originalData = JSON.parse(JSON.stringify(data));

                Object.assign(this, data);

                this.errors = {};
                this.submitted = false;
            }
            data() {
                return Object.keys(this.originalData).reduce((data, attribute) => {
                    data[attribute] = this[attribute];
                    return data;
                }, {});
            }
            post(endpoint) {
                this.submit(endpoint, 'post');
            }
            patch(endpoint) {
                this.submit(endpoint, 'patch');
            }
            delete(endpoint) {
                this.submit(endpoint, 'delete');
            }
            submit(endpoint, requestType = 'post') {
                return axios[requestType](endpoint, this.data())
                    .catch(this.onFail.bind(this))
                    .then(this.onSuccess.bind(this));
            }
            onSuccess(response) {
                this.submitted = true;
                this.errors = {};
                return response;
            }
            onFail(error) {
                this.submitted = false;
                this.errors = error.response.data.errors;
                throw error;
            }
            reset() {
                Object.assign(this, this.originalData);
            }
        }
        
        export default FormHandler;
        ```

        The throw error; in the onFail method is needed so the errors stops propagation and the then promise, in the component isn't called. Without it, `.then(response => location = response.data.message);` in the component example below, is being executed even though an error occurred. This will stop the `promise` from executing the `then`. You could also add a `.catch` to this as well to handle the error differently on the front end.

        In the component where the form is being used:
        ```js
        import FormHandler from './FormHandler';
        export default {
            data() {
                return {
                    form: new FormHandler({
                        'title': '',
                        'description': '',
                        tasks: [
                            {body: ''},
                        ]
                    }),
                };
            },
            method: {
                async submit() {
                    if (!this.form.tasks[0].body) {
                        delete this.form.originalData.tasks;
                    }
                    this.form.submit('/projects')
                        .then(response => location = response.data.message);
                }
            }
        }
        ```
* ### How to create a "*Pin to the Top*" component that will pin or fix an element to the top of the screen once the user has scrolled past it on the page:
    * Usage: `<pinned><div>CONTENTS TO PIN</div></pinned>`
    * Have a css class called `is-fixed-to-top` that contains the following styles:
        ```css
        position: fixed;
        top: 0;
        width: 100%;
        z-index: 10;
        ```
    * Create a `pinned.vue` component file
        ```vue
        <template>
            <div><slot></slot></div>
        </template>
        <script>
        import { throttle } from 'lodash';
        export default {
            mounted() {
                let el = this.$el;

                let originalOffsetTop = this.el.offsetTop;

                window.addEventListener('scroll', throttle(function() {
                    if (window.scrollY >= originalOffsetTop) {
                        el.classList.add('is-fixed-to-top');
                    } else {
                        el.classList.remove('is-fixed-to-top');
                    }
                }, 300));
            }
        }
        </script>
        ```
* ### *Tool Tip* examples:
    * All require `Tooltip.js`. Install using `npm install tooltip.js --save`
    * Import `Tooltip.js` in the `app.js` file. Calling it `PopperTooltip` to not collide with the custom component we will be making as well.
        * `import PopperTooltip from 'tooltip.js'`
    * **Using data attributes**
        * Within the root Vue instance in `app.js` file you can add an query selector to find all `data-tooltip` attributes and grab the tooltip from there

            Add a mounted event:
            ```js
            mounted() {
                document.querySelectorAll('[data-tooltip]').forEach(elem => {
                    new PopperTooltip(elem, {
                        placement: elem.dataset.tooltipPlacement || 'top',
                        title: elem.dataset.tooltip
                    });
                });
            }
            ```
        * On the page you can then have an element with the `data-tooltip` attribute and an optional `data-tooltip-placement` attribute to specify the placement.
            * `<p data-tooltip="This is a tooltip" data-tooltip-placement="right">Hover over me</p>`
    * **Dedicated Vue Directive**
        * Register a new directive. Can be in its own file or within the `app.js`
            ```js
            Vue.directive('tooltip' , {
                bind(elem, bindings) {
                    new PopperTooltip(elem, {
                        placement: bindings.arg || 'top',
                        title: bindings.value
                    });
                }
            });
            ```
        * On the page you can then have an element with the `v-tooltip` directive to produce the tooltip
            * `<p v-tooltip="This is a tooltip">Hover over me</p>`
            * To add a different placement argument use: `<p v-tooltip:left="This is a tooltip">Hover over me</p>`
    * **Use a custom component**
        * Register the component in the `app.js` folder
            * `import Tooltip from './components/Tooltip.js';`
            * `Vue.component('tooltip', Tooltip);`
        * Create a `Tooltip.vue` file within `resources/assets/js/components/`
            ```vue
            <template>
                <div v-show="false">
                    <slot></slot>
                </div>
            </template>
            <script>
            import PopperTooltip from 'tooltip.js';
            export default {
                props: { 
                    name: {},
                    placement: { default: 'top' },
                    offset: { default: false }
                }
                mounted() {
                    document.querySelectorAll(`[data-tooltip-name=${this.name}]`).forEach(elem => {
                        new PopperTooltip(elem, {
                            placement: this.placement,
                            title: this.$el.body.innerHTML, //can be a string or dom node
                            offset: this.offset,
                            html: true //needed if you are using any html in the title
                        }
                    });
                }
            };
            </script>
            ```
        * On the page you need to specify the tooltip trigger as well as the formatting of the component itself
            ```html
            <p>Hi There, <span data-tooltip-name="our-products-tooltip">Hover over me</span></p>
            <tooltip name="our-products-tooltip" 
                placement="bottom" 
                offset="0,20"> 
                placement and offset are optional
                <h1>Our Products</h1>
                <p>Some info about our products here.</p>
            </tooltip>
            ```
* ### *Lazy Loader* Example:
    * This will be a separate component that can be used to load more results into a given container when the user has scrolled to about 70% of the way through those results. JW used `Algolia` but it should work with an ajax call to load more results as well.
        * Create a `LoadMore.js` file within `resources/assets/js/components/`. Since we are not using a traditional template, then we are just using the `.js` instead of `.vue`.
            ```js
            import { throttle } from 'lodash';
            export default {
                template: '<div></div>', //you could have a slot tag in-between the divs if you wanted to add a Loading More... message or spinner
                props: { container: {} },
                mounted() {
                    window.addEventListener('scroll', this.loadMore);
                },
                methods: {
                    loadMore: throttle(function(e) {
                        if (this.shouldLoadMore()) {
                            this.$emit('ready');
                        }
                    }, 300), shouldLoadMore() {
                        let containerHeight = $(this.container).height(); //Note that this is using jQuery
                        let distanceFromTop = $(this.container).offset().top; //Again using jQuery
                        return (
                            //calculates if the user has scrolled through 70% of the container
                            window.pageYoffset >= (distanceFromTop + containerHeight) * 0.7
                        )
                    }
                }
            }
            ```
        * Import the `LoadMore.js` component into the component you want the lazy loader in. `import LoadMore from '../components/LoadMore';`
        * Register `LoadMore` in the component. `components: { LoadMore },`
        * Place the component on the page somewhere after the results container is located.
            * `<load-more container=".results-container"></load-more>` If the container needs to be reference by an id then use `#` instead of `.`
* ### Using Vue in a SPA (Single Page Application) with Laravel
    * Basic setup the JW did:
        * Rename the `components` directory in `resources/assets/js/` to `view` since all of the route `.vue` views will be stored there. JW did say that he'd normally keep the components directory in real life.
        * Rename the `resources/assets/js/components/Example.vue` file to `Home.vue`
        * `resources/assets/js/bootstrap.js` has the following:
            ```js
            import Vue from 'vue';
            import VueRouter from 'vue-router';
            import Axios from 'axios';
            window.vue = Vue;
            Vue.use(VueRouter);
            window.axios = Axios;
            ```
        * `resources/assets/js/app.js` has the following:
            ```js
            import './bootstrap.js';
            import router from './routes';
            new Vue( { el: '#app', router } );
            ```
        * copied the content from `resources/view/welcome.blade.php` into a master layout file at `resources/views/layouts/master.blade.php`
            * make sure the `body` has a `div` with the `id` of `app`
            * add `<script src="/js/app.js">` to the bottom of the body
            * add `<link rel="stylesheet" src="/css/app.css">` to the head
        * The new `welcome.blade.php` file will then just contain `@require('layouts.master')`
    * You will still need a `router` since Laravel is only going to route the 1st request to the root of the site in the `routes/web.php` file.
        * [Vue-Router](https://router.vuejs.org/): `npm install vue-router --save-dev`
        * create a `routes.js` file within `resources/assets/js/`
            * import Vue Router using `import VueRouter from 'vue-router';`
            * declare the `routes` array, which will contain all the route objects: 
                ```js
                let routes = [
                    {path: '/', component: './views/Home.vue'}, 
                    {path: '/about', component: './views/About.vue'}
                ];
                ```
                The `.vue` in the component path is optional since laravel-mix will be able to use either.

                ```js
                export defaults new VueRouter({routes});
                ```
            * Other options to use in the `VueRouter` instance after the routes object
                * `linkActiveClass: {string}` used to change the class name that is automatically applied to a link in the `router-link` component.
        * To use the vue router within the `master.blade.php` file: 
            * To create the nav links for the application you need to use the `router-link` component included with `vue-router`. 
                ```js
                <router-link to="/" exact>
                    Home
                </router-link>
                <router-link to="/about" exact>
                    About
                </router-link>
                ```

                * The `exact` attribute tells vue that the url must match the to attribute exactly.
                * If you want the `router-link` component to be a different kind of tag, you can add the `tag` attribute and then wrap the content between the `router-link` tag with something else like an anchor tag without the href. i.e. `<router-link tag="li" to="/" exact><a>Home</a></router-link>`
            * The view then needs to be rendered on the page using the `router-view` component somewhere within the body tag. `<router-view></router-view>`
    * Make sure you have your watcher running. `npm run watch`