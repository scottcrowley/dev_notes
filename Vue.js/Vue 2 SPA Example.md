# Laravel, Vue & SPAs

## Notes from the Laracast [Laravel, Vue & SPAs](https://laracasts.com/series/laravel-vue-and-spas) series

* ### Setup
    * Set up a new Laravel project as normal and run `npm install`
    * Need to install [Vue Router](https://router.vuejs.org/)
        ```zsh
        npm install vue-router
        ```
    * In the `resources/js/app.js` file, we will start from scratch, so the file should look like:
        ```js
        import Vue from 'vue';
        import VueRouter from 'vue-router';
        import routes from './routes';

        Vue.use(VueRouter);

        let app = new Vue({
            el: '#app',

            router: new VueRouter(routes)
        });
        ```
    * As noted in the app.js file, we need a routes file that will contain all the routes for the application. Create the file at `resources/js/routes.js` and have it contain some starter code
        ```js
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
    * Since Vue will handle all the routing then we need to tell Laravel to send all routes to the app page. `routes/web.php` file:
        ```php
        Route::get('/{any?}', function () {
            return view('app');
        });
        ```
        the `{any?}` specifies a wild card of `anything` or `nothing`

    * The `Home` and `About` Vue components need to be setup in the `resources/js/components` directory and can just contain place holder details for now
        ```vue
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
    * Create a new app view in `resources/view/app.blade.php`
        ```html
        <!doctype html>
        <html lang="{{ str_replace('_', '-', app()->getLocale()) }}">
        <head>
            <meta charset="utf-8">
            <meta name="viewport" content="width=device-width, initial-scale=1">

            <title>Laracasts Assets</title>
            <link rel="stylesheet" href="/css/app.css">
        </head>

        <body class="font-sans">
            <div id="app">
                <router-view></router-view>
                <hr/>
                <router-link to="/">Home</router-link>
                <router-link to="/about">About</router-link>
            </div>

            <script src="/js/app.js"></script>
        </body>
        </html>
        ```
        The `router-view` component is where `Vue` router inserts all the content from the route components. The `router-link` component is used to specify navigation for all page loads. The `a` tag cannot be used.

    * Make sure your watcher is running
        ```zsh
        npm run watch
        ```
* ### Links
    * **Active Links**
        * *CSS Approach*
            * When you click on a `router-link` link a couple of classes are added to the link to signify that it is an active link. These classes are `router-link-exact-active` & `router-link-active`. You can then use these classes to style your active links accordingly.
        * *Attribute Approach*
            * You can also add the active-class attribute to the router-link link. Doing this will automatically apply any styling you have set the attribute to.
                
                When the `Logo` link is active, Vue will automatically add the `font-bold` class to the `router-link`
                ```html
                <ul class="list-reset">
                    <li class="text-sm leading-loose">
                        <router-link class="text-black" active-class="font-bold" to="/">Logo</router-link>
                    </li>
                    <li class="text-sm leading-loose">
                        <router-link class="text-black" active-class="font-bold" to="/logo-symbol">Logo Symbol</router-link>
                    </li>
                    <li class="text-sm leading-loose">
                        <router-link class="text-black" active-class="font-bold" to="/colors">Colors</router-link>
                    </li>
                    <li class="text-sm leading-loose">
                        <router-link class="text-black" active-class="font-bold" to="/typography">Typography</router-link>
                    </li>
                <ul>
                ```
                There is a inherant problem with the example above. Vue will match the root link as active for all the links because all links begin with `/`. So if you click on `Colors`, the `Colors` link and the `Logo` link will be marked as active. To fix this issue you need to add the `exact` attribute to the link that should always be matched exactly. In this case it is the `Logo` link.
                ```html
                <router-link class="text-black" active-class="font-bold" to="/" exact>Logo</router-link>
                ```
        * *Constructor Approach*
            * You can also specify the classes to add to active links in the constructor being passed to the VueRouter. In this case, the routes object (`resources/js/routes.js`) is being passed, so this is where we will add a property called `linkActiveClass`.
                ```js
                ...

                export default {
                    mode: 'history',

                    linkActiveClass: 'font-bold',

                    routes: [
                        ...
                };
                ```
                Note that you will still need the `exact` attribute on the Logo link as specified in the previous example under the *`Attribute Approach`* section.
* ### 404 Pages
    * Since we used a wild card (`{any?}`) in our Laravel routes file (`routes/web.php`), we need to specify a catch all for anything else that isn't covered in our `routes.js` file.
        * *`routes.js`*
            
            We need to add the following to the `routes` array in the `routes.js` file and also import the `NotFound` component at the top of the file.
            ```js
            {
                path: '*',
                component: NotFound
            }
            ```
            Here is what it should look like afterwards
            ```js
            ...
            import NotFound from './components/NotFound';
            
            export default {
                mode: 'history',

                linkActiveClass: 'font-bold',

                routes: [
                    {
                        path: '*',
                        component: NotFound
                    },

                    {
                        path: '/',
                        component: Logo
                    },

                    {
                        path: '/logo-symbol',
                        component: LogoSymbol
                    },
                ]
            };
            ```
        * Example NotFound Component

            resources/js/components/NotFound.vue
            ```vue
            <template>
                <h1>Whoops! 404 - Page not found</h1>
            </template>

            <script>
            export default {

            };
            </script>
            ```
        * `The routes/web.php` file needs to be updated to catch page requests to multilevel urls.
            * Currently, with `/{any?}` specified in web.php, this only pertains to the first level of the url. i.e. `test.test/something` would be captured but `test.test/something/somethingelse` would not. This would then be redirected to the Laravel 404 page and not our `NotFound` component.
            * To fix this the route needs to be updated.
                ```php
                Route::get('/{any}', function() {
                    return view('app');
                })->where('any', '.*');
                ```
                The `where` method specifies further restrictions on the wildcard. It receives the name of the wildcard, `any`, and a regular expression to use to determine if it should be captured. The regular expression translates to: anything (`.`) followed by anything else (`*`)
* ### Lazy Loading Routes
    * Asyncronous Vue components with Webpack dynamic imports
    * The reason this is needed is because JW has a `Loaders and Animation` page that has an animation on it that requires [AirBnb's Lottie library](https://airbnb.io/lottie/#/) to render it. This library is very huge and added more than 500kb to the size of the compiled `app.js` file. Instead of Vue always loading that library, he wants to be able to only load it when the page is requested or lazy load it.
    * In the `routes.js` file we are changing the import of the `LoadersAndAnimations` component to be asyncronous. Making this component resolve and render itself on the fly.

        Change this
        ```js
        import LoadersAndAnimations from './components/LoadersAndAnimations';
        ```
        To this
        ```js
        let LoadersAndAnimations = () => import('./components/LoadersAndAnimations');
        ```
        Using the `import` function in this way declares it as a dynamic import. This tells webpack to split this code into its own bundle that should be loaded on the fly and only when needed.
    * A babel plugin is needed for Webpack to dynamicaly import something. 
        * Pull the package in using:
            ```zsh
            npm install @babel/plugin-syntax-dynamic-import
            ```
        * Create a `.babelrc` file in the site root. This configuration file will be merged with the configuration that Webpack supplies by default. This file simply acts as an override for the plugins. The file should look like this:
            ```json
            {
                "plugins": ["@babel/plugin-syntax-dynamic-import"]
            }
            ```
        * When you run `npm run dev`, you'll notice that 2 new `.js` files are created. Both of them are for the the dynamically imported `Lottie` library.
            * By default, webpack will name the split out files `0.js` & `1.js`. You can apply a name to a bundle by adding a magic comment (`webpackChunkName`) to the `import` function call.
                ```js
                let LoadersAndAnimations = () => 
                    import(/* webpackChunkName: "loaders" */ './components/LoadersAndAnimations');
                ```
            * Now the files will be called `loaders.js` and `vendor~loaders.js` when the scripts are compiled.
        * So now when you hit the site again, this `Lottie` library is not being loaded until you click on the `Loaders and Animations` link.
* ### Cross-Origin Resource Sharing or CORS
    * JW wants to add a stats section to the site that will make an ajax request to the main Laracast site to retrieve the number of lessons and series that the site currently has.
    * The main **Laracast** project:
        * `routes/api.php`

            Everything in this file will have the api middleware applied to it, which provides throttling. All routes declared here will automatically be prefixed with `/api`. So to access the stats page, you'd go to `laracarst.test/api/stats`
            ```php
            <?php

            Route::get('/stats', function() {
                return [
                    'series' => 200,
                    'lessons' => 1300
                ];
            })
            ```
        * We need a new middleware class to handle CORS.
            ```zsh
            php artisan make:middleware Cors
            ```
        * Register the new middleware in `app/Http/Kernel.php` file and add the class globally to the `$middleware` property that already contains the default global middleware.
            ```php
            protected $middleware = [
                ...
                Cors::class
            ]
            ```
        * `app/Http/Middleware/Cors.php`
            ```php
            public function handle($request, Closure $next)
            {
                return $next($request)
                    ->header('Access-Control-Allow-Origin', 'http://assets.test')
                    ->header('Access-Control-Allow-Headers', 'X-REQUESTED-WITH');
            }
            ```
    * The **Assets** site project:
        * resources/js/app.js

            The following needs to be added at the top of the file, after the other imports, so we can use axios, since we never imported the normal Laravel bootstrap.js file.
            ```js
            import axios from 'axios';

            window.axios = axios;
            window.axios.defaults.headers.common['X-Requested-With'] = 'XMLHttpRequest';
            ```
        * resources/js/components/Stats.vue
            ```vue
            <template>
                <h1>Stats</h1>

                <ul>
                    <li><strong>Total Series:</strong> {{ series }}</li>
                    <li><strong>Total Lessons:</strong> {{ lessons }}</li>
                </ul>
            </template>

            <script>
            export default {
                data() {
                    return {
                        series: 0,
                        lessons: 0
                    }
                },
                created() {
                    axios.get('https://laracasts.test/api/stats').then(({ data }) => {
                        this.series = data.series;
                        this.lessons = data.lessons;
                    });
                }
            };
            </script>
            ```
* ### Using Tokens with API calls
    * JW added a achievements section to the assets site that will show all achievements earned on Laracasts.
    * The main **Laracast** project:
        * `config/auth.php` make sure the `guards.api` section, under the `Authentication Guards` section, is as follows. Its important to note that `guards.api.driver` = `token`
            ```php
            'guards' => [
                'web' => [
                    'driver' => 'session',
                    'provider' => 'users'
                ],

                'api' => [
                    'driver' => 'token',
                    'provider' => 'users',
                    'hash' => true // Laravel 5.8^ See note below
                ],
            ]
            ```
            In Laravel 5.8, there will also be a `hash` key in `guards.api`. If set to `true` then Laravel will hash the token from the request before quering the database. By default, Laravel will use `hash('sha256', $token)` to hash the token.
        * `routes/api.php` add the following route
            ```php
            Route::get('/achievements', function() {
                return request()->user()->achievements;
            })->middleware('auth:api');
            ```
            By using the `auth:api` middleware, this is saying that when you hit this route, Laravel must try to authorize it my checking the api token. Which is set to token in the `config/auth.php` file (see above). So it will check the `users` table for the token.
        * Make a new migration for the adding tokens to user table. This will appends the current users table with additional fields.
            ```zsh
            php artisan make:migration add_api_token_to_users_table --table=users
            ```
        * `database/migrations/...add_api_token_to_users_table.php` 

            in the `up()` method
            ```php
            Schema::table('users', function(Blueprint $table) {
                $table->string('api_token',80)
                    ->after('password')
                    ->unique()
                    ->nullable();
            });
            ```
            in the down() method
            ```php
            Schema::table('users', function(Blueprint $table) {
                $table->dropColumn('api_token');
            });
            ```
        * Run migration
            ```zsh
            php artisan migrate
            ```
        * Now you can add the api token to a user, either in the model when a new account is created or using `php artisan tinker` to add it to a specific user. Below is example with `tinker`.
            ```php
            $user =  Laracasts\User::whereUsername('JefferyWay')->first();
            $user->fill(['api_token' => str_random(60)])->save();
            ```
        
    * The **Assets** site project:
        * resources/js/components/Achievments.vue
            ```vue
            <template>
                <h1>Stats</h1>

                <input 
                    placeholder="Your Laracasts API Token" 
                    v-model="token" 
                    class="border p2 rounded w-full"
                    @keyup.enter="fetchAchievements" />
                
                <p class="text-red text-sm italic" v-if="message" v-text="message"></p>

                <ul>
                    <li
                        v-for="achievement in achievements"
                        v-text="achievement.name"
                    ></li>
                </ul>
            </template>

            <script>
            export default {
                data() {
                    return {
                        achievements: [],
                        token: '',
                        message: ''
                    }
                },
                methods() {
                    fetchAchievements() {
                        axios.get(`https://laracasts.test/api/achievements?api_token=${this.token}`)
                            .catch(error => {
                                this.message = error.response.data.error;
                                this.achievements = [];
                            })
                            .then(({ data }) => {
                                this.achievements = data;
                                this.message = null;
                            });
                    }
                }
            };
            </script>
            ```
