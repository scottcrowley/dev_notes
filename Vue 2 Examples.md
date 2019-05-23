# Vue 2 Examples

* ### Custom Input Example:
    * Say you want to be able to reuse an input that has validation logic, sanitizing, etc attached to it. i.e. `<coupon></coupon>` instead of `<input type="text" v-model="coupon">`
        * On your page you can still use `v-model` on the component `<coupon v-model="coupon"></coupon>`
            * In you js file:
                ```
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
            ```
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
        ```
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
        ```
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
        ```
        position: fixed;
        top: 0;
        width: 100%;
        z-index: 10;
        ```
    * Create a `pinned.vue` component file
        ```
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
            ```
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
            ```
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
            ```
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
            ```
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
            ```
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
            ```
            import Vue from 'vue';
            import VueRouter from 'vue-router';
            import Axios from 'axios';
            window.vue = Vue;
            Vue.use(VueRouter);
            window.axios = Axios;
            ```
        * `resources/assets/js/app.js` has the following:
            ```
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
                ```
                let routes = [
                    {path: '/', component: './views/Home.vue'}, 
                    {path: '/about', component: './views/About.vue'}
                ];
                ```
                The `.vue` in the component path is optional since laravel-mix will be able to use either.

                ```
                export defaults new VueRouter({routes});
                ```
            * Other options to use in the `VueRouter` instance after the routes object
                * `linkActiveClass: {string}` used to change the class name that is automatically applied to a link in the `router-link` component.
        * To use the vue router within the `master.blade.php` file: 
            * To create the nav links for the application you need to use the `router-link` component included with `vue-router`. 
                ```
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