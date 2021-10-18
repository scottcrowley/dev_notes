# Laravel with Inertia.js and Vue.js to build a SPA

### Notes on [Build Modern Laravel Apps Using Inertia.js](https://laracasts.com/series/build-modern-laravel-apps-using-inertia-js) from Laracasts

## [Inertia.js](https://inertiajs.com)
* ### Install Server-Side Requirements:
    * Install via Composer
    ```zsh
    composer require inertiajs/inertia-laravel
    ```
    * Paste the following into the main template file. Usually app.blade.php
    ```html
        <!DOCTYPE html>
        <html lang="en">

        <head>
            <meta charset="utf-8" />
            <meta name="viewport" content="width=device-width, initial-scale=1.0, maximum-scale=1.0" />
            <link href="{{ mix('/css/app.css') }}" rel="stylesheet" />
            <script src="{{ mix('/js/app.js') }}" defer></script>
        </head>

        <body>
            @inertia
        </body>

        </html>
    ```
    * Install the middleware for Inertia
    ```zsh
    php artisan inertia:middleware
    ```
    * Register the middleware by adding the following to the `web` middleware group in the App\Http\Kernel.php file
    ```php
    \App\Http\Middleware\HandleInertiaRequests::class
    ```
* ### Install Client-Side Requirements:
    * Install latest Vue using npm
    ```zsh
    npm install vue@next
    ```
    * Since we are using Single File Components with Vue we need to install `compiler-sfc` which replaces `vue-template-compiler`
    ```zsh
    npm install -D @vue/compiler-sfc
    ```
    * Use either Vue 2 or Vue 3. Make sure to specify which one you are using below.
    ```zsh
    npm install @inertiajs/inertia @inertiajs/inertia-vue3
    ```
    * Need to setup Vue and Inertia in the `resources\js\app.js` file
    ```js
    import { createApp, h } from 'vue';
    import { createInertiaApp } from '@inertiajs/inertia-vue3';

    createInertiaApp({
        resolve: name => require(`./Pages/${name}`),
        setup({ el, App, props, plugin }) {
            createApp({ render: () => h(App, props) })
            .use(plugin)
            .mount(el)
        },
    });
    ```
    Note: The pages directory is being declared in the snippet above `./Pages/${name}`. There for the app will be looking for all the pages in the `resources\js\Pages` directory.
    * Create a new directory `Pages` in the `resources\js` directory. All page templates should reside here and have be Vue files (`.vue`)
    * In the `webpack.mix.js` file make sure it looks like the snippet below:
    ```js
    mix.js('resources/js/app.js', 'public/js')
        .vue(3) //the 3 is optional but more explicit
        .postCss('resources/css/app.css', 'public/css', [
            //
        ])
        .version(); //this will add a hash to all the assets for caching purposes
    ```
    Note: Make sure that you are referencing the assets in the main template (`app.blade.php`) file using the `mix` helper
    ```html
    <link href="{{ mix('/css/app.css') }}" rel="stylesheet"/>
    <script src="{{ mix('/js/app.js') }}" defer></script>
    ```
    * Run `npm install`
    * Run `npx mix` to compile everything. Same as `npm run dev`?
    * Run `npx mix` again to make sure everything is compiled. It should show the assets that it compiled.
* ### Install Progress Bar
    * Install the default progress library used by Inertia
    ```zsh
    npm install @inertiajs/progress
    ```
    * Add the following import below the inertia-vue import in the `resources\js\app.js` file
    ```js
    import { InertiaProgress } from '@inertiajs/progress'
    ```
    Also place the following below the `createInertiaApp` method call
    ```js
    InertiaProgress.init();
    ```
    The app.js file should look something like:
    ```js
    import { createApp, h } from 'vue'
    import { createInertiaApp } from '@inertiajs/inertia-vue3'
    import { InertiaProgress } from '@inertiajs/progress'

    createInertiaApp({
        resolve: name => require(`./Pages/${name}`),
        setup({ el, App, props, plugin }) {
            createApp({ render: () => h(App, props) })
            .use(plugin)
            .mount(el)
        },
    });

    InertiaProgress.init();
    ```
    Note: The init method can accept options for the progress bar as well:
    ```js
    InertiaProgress.init({
        // The delay after which the progress bar will
        // appear during navigation, in milliseconds.
        delay: 250,

        // The color of the progress bar.
        color: '#29d',

        // Whether to include the default NProgress styles.
        includeCSS: true,

        // Whether the NProgress spinner will be shown.
        showSpinner: false,
    });
    ```
    * Recompile the assets by running
    ```zsh
    npx mix
    ```
## General Usage

### General
* Just like all template components or views are stored in `resources\js\Pages`, you can store minor components like navs, etc in a `Shared` directory. `resources\js\Shared`
For Example: A nav component called `resouces\js\Shard\Nav.vue`. This component can be imported into a template in the `Pages` directory like this:
```js
<template>
    <h1>Title</h1>
    <Nav />
</template>
<script>
import { Nav } from '../Shared/Nav'

export default {
    components: { Nav }
};
</script>
```
### Run a Watcher
Run the watcher by using:
```zsh
npx mix watch
```
or, same as ???
```zsh
npm run watch
```
### Routes (`web.php`)
* Routes are accessed just like you use the `view` method to return a view. Instead of `view` use `inertia`. Everything else is the same. You specify a route name and can pass parameters to the component by passing an array of values after the name. Make sure that the name matches the component name in the `resources\js\Pages` directory. In this example, `Home.vue` should exist.
```php
return inertia('Home', [
    'name' => 'Scott',
]);
```
Note: The Inertia documentation uses a static call instead of the helper method
```php
return Inertia::render('Home', [
    'name' => 'Scott',
]);
```
NOTE: Make sure `use Inertia\Inertia;` is at the top of the document.
* Whatever props are passed to the component using the `Inertia::render` method need to be accepted as props in the component itself
```js
<template>
    <h1>Hello, {{ name }}</h1>
</template>
<script>
export default {
    props: {
        name: String,
    }
};
</script>
```
### Inertia Links
* You can't use a standard `a` tag for links that load pages on the site. This will cause a page reload, which isn't what is wanted. Instead Inertia has a `Link` component to use
* Import the component in template where it is needed and declare it as a component within Vue
```js
import { Link } from '@inertiajs/inertia-vue3';

export default {
    components: { Link },
};
```
* Instead of an `a` tag use a `Link` tag with the same `href` that would be used normally
```js
<Link href="/users">Users</Link>
```
* By default the `Link` component uses the `GET` method but it can also use `POST`, `PATCH` and `DELETE`. These can be used without wrapping the whole tag in a `form` tag.
```js
<Link href="/logout" method="post">Logout</Link>
```
* It is also possible to preserve the scroll position on a page when using the Link component. For example, you don't want the page to refresh when clicking on a "like button" or on a toggle sort order button. In these cases you would want the page to preserve the current scroll location. This is accomplished by using the `preserve-scroll` attribute on the Link component.
```js
<Link href="/currentPage" preserve-scroll>Like This Article</Link>
```
* #### Handle Active Status Using the $page Variable
    * Every component has access to a global $page variable, which contains information about the current page or component being used. This can be used to generate "active" styling so the user knows which page they are on.
    * There are a couple of properties on the $page variable that can be used.
        * `.component` : Gives the name of the current component being used
        * `.url` : Gives the current url of the page you are on. This will also contain the query string, so it is important to remember that when trying to match to the url.
        
           * i.e. `$page.url === '/settings'` will match when you are on the `/settings` url but if the url has a query string then this check will no longer return true. In this case it would be better to use something like `$pages.url.startsWith('/settings')` will return true even if there is a query string.
    * Example Link component that will add styling to an active link. Using the `.component` property seems to be a cleaner, more precise way to handle this. However, the `.url` property can be used as well, as long as you take into account the query string as stated above.
        ```js
        <Link 
            href="/settings" 
            class="text-blue-500 hover:underline"
            :class="{'font-bold underline': $page.component === 'Settings'}"
        >
            Settings
        </Link>
        ```
### Shared Data on All Pages
There may be times where you want to pass the same data to all pages. This data can then be accessed using the `$page.props` property. By default, every page receives the `$errors` property that is normally available on all Laravel pages.
#### The HandleInertiaRequests.php Middleware File
* Located in `\app\Http\Middleware\`
* Use the `share` method to pass any additional shared data to all pages
* It is a good idea to namespace any properties that are being passed. For example: If you are passing a `username` property, you might want to have it namespaced to `auth.user.username`. This can be done by using the following format.
    ```js
    'auth' => [
        'user' => [
            'username' => auth()->user()->username,
        ]
    ]
    ```
    The `username` property then can be accessed in the page by using `{{ $page.props.auth.user.username }}`
    * You can also access it using a "computed property" within your page layout file.
        ```js
        computed: {
            username() {
                return this.$page.auth.user.username;
            }
        }
        ```
        It can then be accessed on the page simply by using `{{ username }}`
### Global Component Registration
If you want to register a component that will be used on every page, like the `Link` component outlined above, you can add the registration to the `createInertiaApp` method call in the `resources\js\app.js` file.
* First import the component by adding it to the import already being used for the createInertiaApp instance. Since both are part of `inertia-vue3`. `import { createInertiaApp, Link } from "@inertiajs/inertia-vue3";`
* Now add a `.component` method call to the `setup` method call the instantiates the instance.
    ```js
    createInertiaApp({
        resolve: name => require(`./Pages/${name}`),
        setup({ el, App, props, plugin }) {
            createApp({ render: () => h(App, props) })
            .use(plugin)
            .component("Link", Link)
            .mount(el)
        },
    });
    ```
    If you have more than one component you want to register, just add a new `.component` for each component you want to add.
    ```js
    .component("Link", Link)
    .component("Head", Head)
    ```
* After you have registered it, you can remove any old imports and registrations from the individual page files, like what was shown in previous examples. Remove the following from any page where the `Link` component was being used.
    ```js
    import { Link } from '@inertiajs/inertia-vue3';
    ```
    &
    ```js
    components: { Link },
    ```
### Persistent Layouts
There is a problem created when using layout files as outlined above. A layout file is technically a child component of the page that it is being used on. So, in essence, the layout is being destroyed whenever a new page is accessed. The example used in the video is that a Podcast is added to the layout file. When you start playing the Podcast on one page, you would want it to continue even after going to a different page that uses the layout file. In the current setup, this will not happen and the Podcast will start over and stop playing.
* To declare a persistent layout, you can use the `layout` property instead of using the `components` property in each page that is using the layout file. `components: { Layout },` changes to `layout: Layout`.
* Now you can remove the wrapping `Layout` element that is used within the `template` tag on the page using the layout specified in the `layout` property.
* After doing this, and you check the page in Vue devtools, you'll notice that the Layout instance is no longer a child of the page instance, it is the other way around.