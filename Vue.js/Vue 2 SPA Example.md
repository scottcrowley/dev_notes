# Laravel, Vue & SPAs

## Notes from the Laracast [Laravel, Vue & SPAs](https://laracasts.com/series/laravel-vue-and-spas) series

* ### Setup
    * Set up a new Laravel project as normal and run `npm install`
    * Need to install [Vue Router](https://router.vuejs.org/)
        ```
        npm install vue-router
        ```
    * In the `resources/js/app.js` file, we will start from scratch, so the file should look like:
        ```
        import Vue from 'vue';
        import VueRouter from 'vue-router';
        import routes from './routes';

        Vue.use(VueRouter);

        let app = new Vue({
            el: '#app',

            router: new VueRouter(routes)
        });
        ```
    * As noted in the app.js file, we need a routes file that will contain all the routes for the application. Create the file at resources/js/routes.js and have it contain some starter code
        ```
        import Home from './components/Home';
        import About from './components/About';

        export default {
            mode: 'history',

            routes: [
                {
                    path: '/',
                    component: Home
                },

                {
                    path: '/about',
                    component: About
                },
            ]
        };
        ```
    * Since Vue will handle all the routing then we need to tell Laravel to send all routes to the home page. `routes/web.php` file:
        ```
        Route::get('/{any?}', function () {
            return view('welcome');
        });
        ```
        the `{any?}` specifies a wild card of `anything` or `nothing`

    * The `Home` and `About` Vue components need to be setup in the `resources/js/components` directory and can just contain place holder details for now
        ```
        <template>
            <p>
                Home
            </p>
        </template>

        <script>
        export default {

        };
        </script>
        ```
    * The welcome view in `resources/view/welcome.blade.php`, strip out everything in the `<head>` except the `title` and `meta` tags. Also, the only thing in the `<body>` tag should be a `div` with the `id` of `app`. Make sure to import the `app.js` file at the bottom of the `body` tag. `<script src="/js/app.js"></script>` Below are the other components we'll need in the body of the doc.
        ```
        <div id="app">
            <router-view></router-view>
            <hr/>
            <router-link to="/">Home</router-link>
            <router-link to="/about">About</router-link>
        </div>
        ```
        The `router-view` component is where `Vue` router inserts all the content from the route components. The `router-link` component is used to specify navigation for all page loads. The `a` tag cannot be used.

    * Make sure your watcher is running
        ```
        npm run watch
        ```
    
