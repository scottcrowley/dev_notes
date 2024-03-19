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
            @routes
            <script src="{{ mix('/js/app.js') }}" defer></script>
        </head>

        <body>
            @inertia
        </body>

        </html>
    ```
    * Take note of the `@routes` Blade Directive before the scripts are added. This allows for using the routes declared in the `web.php` file within Vue components. See the section below on Ziggy
    * Install the middleware for Inertia
    ```zsh
    php artisan inertia:middleware
    ```
    * Register the middleware by adding the following to the `web` middleware group in the `App\Http\Kernel.php` file
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
## [Ziggy](https://github.com/tighten/ziggy)
Not covered in the video series.

Ziggy allows you to use the routes declared in the `web.php` file within Vue components. By adding the `@routes` Blade directive in the `head` tag of your main layout file, before the app's Javascript files are imported, you can gain access to this information. Now you can use the `route()` helper function in all your Javascript files.
### Usage
* Getting Routes
    * Say you have a named route, in `web.php`, called `posts.index`. You can access the route uri in Javascript by using `route('posts.index')`. 
    * If your route has wild cards in the definition, you can pass a second argument to the `route()` function. 
        ```php
        // routes\web.php
        Route::get('posts/{post}', [PostController::class, 'show'])->name('posts.show');
        ```
        ```js
        // Vue component or Javascript
        route('posts.show', 1);
        route('posts.show', [1]);
        route('posts.show', { post: 1 });
        // All return https://thesite.test/posts/1
        ```
    * If your route uses multiple wild cards you can pass them all through the second argument.
        ```php
        // routes\web.php
        Route::get('events/{event}/venues/{venue}', [EventController::class, 'show'])->name('events.venues.show');
        ```
        ```js
        // Vue component or Javascript
        route('events.venues.show', [1, 2]);
        route('events.venues.show', { event: 1, venue: 2 });
        // returns https://thesite.test/events/1/venues/2
        ```
    * You can also pass query parameters when getting a route as well
        ```php
        // routes\web.php
        Route::get('events/{event}/venues/{venue}', [EventController::class, 'show'])->name('events.venues.show');
        ```
        ```js
        // Vue component or Javascript
        route('events.venues.show', { 
            event: 1, 
            venue: 2,
            page: 5,
            count: 10
        });
        // returns https://thesite.test/events/1/venues/2?page=5&count=10
        ```
        If you have query parameters that have the same name as a wild card, then you can pass them in using the `_query` key.
        ```js
        // Vue component or Javascript
        route('events.venues.show', { 
            event: 1, 
            venue: 2,
            _query: {
                event: 5,
                page: 10
            }
        });
        // returns https://thesite.test/events/1/venues/2?event=5&page=10
        ```
* Using the `Router` Class
    * Calling the `route()` helper with no arguments returns an instance of the Javascript `Router` class, which has some useful methods that can be used.
        ```js
        // Route called 'events.index', with URI '/events'
        // Current window URL is https://ziggy.test/events
        route().current();               // 'events.index'
        route().current('events.index'); // true
        route().current('events.*');     // true
        route().current('events.show');  // false

        // Route called 'events.venues.show', with URI '/events/{event}/venues/{venue}'
        // Current window URL is https://myapp.com/events/1/venues/2?authors=all
        route().current('events.venues.show', { event: 1, venue: 2 }); // true
        route().current('events.venues.show', { authors: 'all' });     // true
        route().current('events.venues.show', { venue: 6 });           // false

        // App has only one named route, 'home'
        route().has('home');   // true
        route().has('orders'); // false

        // Route called 'events.venues.show', with URI '/events/{event}/venues/{venue}'
        // Current window URL is https://myapp.com/events/1/venues/2?authors=all

        route().params; // { event: '1', venue: '2', authors: 'all' }
        // NOTE: values returned using .params will always be a string
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
* You can also send data in a `POST` or `PATCH` request by using the `:data` directive
```js
<Link href="/logout" method="post" :data="{name: 'foobar'}">Logout</Link>
```
* A link tag can also be rendered as a button by adding the `as` directive. This prevents you from `command` clicking on a link to open it in a new tab.
```js
<Link href="/logout" method="post" as="button">Logout</Link>
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
* ***Flash Messages*** - This is a good use-case for adding flashed messages to a page. You can grab the value assign to a message flashed to the session.
    ```php
    'flash' => [
        'message' => fn () => $request->session()->get('message')
    ],
    ```
    ```html
    <div v-if="$page.props.flash.message" class="alert">
        {{ $page.props.flash.message }}
    </div>
    ```
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

### Dynamic Components
There may be times where you need to generate a dynamic component based on whether something is true or not. In the episode on [Pagination](https://laracasts.com/series/build-modern-laravel-apps-using-inertia-js/episodes/17), a dynamic component is being used to generate the pagination links of the `users` object being sent to the page. The `users` object contains a `link` property containing an array of all the links for the page. When you are on the first page, the "Previous" link has no value for the `url` property. Same goes for when you are on the last page, the "Next" link has no value for the `url` property. You can use the code below to dynamically generate either a `Link` component or a `<span>` tag based on whether or not there is a `url`.
```js
<div class="mt-6">
    <Component
        :is="link.url ? 'Link' : 'span'"
        v-for="link in users.links"
        :href="link.url"
        v-html="link.label"
        class="px-1"
        :class="{ 'text-gray-500': ! link.url, 'font-bold': link.active }"
    /> 
</div>
```
In the controller action, the `users` object was being sent to the page using a `map` function.
```php
    return Inertia::render('Users', [
        'users' => User:all()->map(fn($user) => [
            'id' => $user.id,
            'name' => $user.name,
        ])
    ]);
```
When using a paginator this code will break because the paginator contains several other properties and puts all the data in the `data` property. The code below will break the page:
```php
    return Inertia::render('Users', [
            'users' => User:paginate(10)->map(fn($user) => [
            'id' => $user.id,
            'name' => $user.name,
        ])
    ]);
```
Instead of using the `map` function, you can use the `through` function, which will leave the object intact while it maps over it. I was unable to find any details in the Laravel docs on how to use `through`, but there was a comment regarding it. The `through` method is only available on a paginator instance.
```php
    return Inertia::render('Users', [
        'users' => User:paginate(10)->through(fn($user) => [
            'id' => $user.id,
            'name' => $user.name,
        ])
    ]);
```

### Forms
You can use the form helper provided by Inertia to help manage all aspects of a form. There are two ways to use the helper depending on if you are using the **Options API** or **Composition API**.
```js
//Options API
<script>
export default {
  data() {
    return {
      form: this.$inertia.form({
        email: null,
        password: null,
        remember: false,
      }),
    }
  },
}
</script>

//Composition API
<script setup>
import { useForm } from '@inertiajs/inertia-vue3'

let form = useForm({
    name: '',  
    email: '',
    password: '',
});

let submit = () => {
    form.post('/users/');
};
</script>
```
Accessing the helper's functions is performed the same way regardless of which ever API is being used.

#### Submitting the Form
You can use the following submit methods: `get`, `post`, `put`, `patch` and `delete`. The second argument can be used to specify options for the submit or to tap into events that occur during the processing.
```js
form.post(url,options); //replace post with any method

//options:
preserveScroll: true //boolean or callback
preserverState: true //boolean or callback

//events:
onBefore: (visit) => {},
onStart: (visit) => {},
onProgress: (progress) => {},
onSuccess: (page) => {},
onError: (errors) => {},
onCancel: () => {},
onFinish: visit => {},
```
While the form is being processed you can use the `form.processing` property to display something or disable the submit button.
```html
<button type="submit" :disabled="form.processing">Submit</button>
```
You can also access `form.progress` or `form.progress.percentage` to display upload or processing status.
```html
<progress v-if="form.progress" :value="form.progress.percentage" max="100">
  {{ form.progress.percentage }}%
</progress>
```

#### Other Form Methods and Properties
***transform()*** - If you need to modify data in the form before it is sent to the server, you can use `form.transform` before the submit method is called.
```js
form
  .transform((data) => ({
    ...data,
    remember: data.remember ? 'on' : '',
  }))
  .post('/login')
```
***errors*** - If you need to modify data in the form before it is sent to the server, you can use `form.transform` before the submit method is called.
```html
<div v-if="form.errors.email">{{ form.errors.email }}</div>
```
You can also check if the form has *any* errors at all by using the `form.hasErrors` property.
If you need to clear the errors from the form you can use the `form.clearErrors()` to clear all form errors or `form.clearErrors('field', 'anotherField')` to clear individual field errors.

***reset()*** - This method can be used to clear all fields on the form `form.reset()` or clear individual fields by using `form.reset('field', 'anotherField')`.

***wasSuccessful*** - This boolean property will be set to `true` if a form was submitted successfully. `form.wasSuccessful`

***recentlySuccessful*** - This boolean property will be set to `true` for 2 seconds after the form was submitted successfully. This can be used to display a success flash message or some other type of user feedback. `form.recentlySuccessful`

***isDirty*** - This boolean property will be set to `true` when a form has changed. This can be used to display a message that the form has changed and needs to be saved.
```html
<div v-if="form.isDirty">There are unsaved form changes.</div>
```

## Authentication with Inertia
The `web.php` should have the following routes set up
```php
Route::get('login'. [LoginController::class, 'create'])->name('login');
Route::post('login', [LoginController::class, 'store']);
Route::post('logout', [LoginContoller::class, 'destroy'])->middleware('auth');
```

The `LoginController` should have a `create` method that renders an `Auth/Login` vue page and a `store` method that actually logs the user in.
```php
namespace App/Http/Controllers/Auth;

use App\Http\Contollers\Controller;
use Illuminate\Http\Request;
use Illuminate\Support\Facades\Auth;
use Inertia\Inertia;

class LoginController extends Controller
{
    public function create()
    {
    	return Inertia::render('Auth/Login');
    }
    
    public function store(Request $request) 
    {
    	$credentials = $request->validate([
            'email' => ['required', 'email'],
            'password' => ['required']
        ]);
        
        if (Auth::attempt($credentials)) {
            $request->session()->regenerate();
            
            return redirec()->intended();
        }
        
        return back()->withErrors([
            'email' => 'The provided credentials do not match our records.'
        ]);
    }
    
    public function destroy()
    {
    	Auth::logout();
        
        return redirect()->route('login');
    }
}
```

Create a new page in `resources/js/Pages/Auth` called `Login.vue`. This page will have the login form that will use Inertia's `form` object to submit the login form to the login endpoint.


## Examples
### Search Filtering on a List
In this example, we will use a page that displays a list of users from the `users` table. The route is a `get` request to the `/users` route endpoint. The controller will send a `$users` variable to the page containing a paginated list of all the users in the database. We start by adding an input field to the page and assign a `v-model` to it.
```html
<input v-model="search" type="text" placeholder="Search..." class="border px-2 rounded-lg" />
```
Next we adjust the code that handles the search functionality. I included the new **Composition API** technique along with the traditional **Options API**.
```js
// Options API

import Pagination from '../Shared/Pagination';

export default {
  components: { Pagination },

  props: {
    users: Object,
    filters: Object
  },

  data() {
    return {
      search: this.filters.search
    }
  },

  watch: {
    search(value) {
      this.$inertia.get('/users', { search: value }, {
        preserveState: true,
        replace: true
      });
    }
  }
}

//Composition API

<script setup>
import Pagination from '../Shared/Pagination';
import { ref, watch } from "vue";
import { Inertia } from "@inertiajs/inertia";

let props = defineProps({
  users: Object,
  filters: Object
});

let search = ref(props.filters.search);

watch(search, value => {
  Inertia.get('/users', { search: value }, {
    preserveState: true,
    replace: true
  });
});
</script>
```
It's important to note that the use of `preserveState: true` makes sure that the search input field is not cleared when a letter is typed into the field. It also makes the focus remain on the field as you type. Since each time you type into the field, it is making a `GET` request to the server that would normally act as if you went to a new page.
Also, the use of `replace: true` is good to use because each new `GET` request will replace the previous one. Without this option, there will be a new history entry made for each letter typed.

Finally, the controller action, on the Laravel side, needs to be updated to handle the new search.
```php
return Inertia::render('Users', [
    'users' => User::query()
        ->when(Request::input('search'), function ($query, $search) {
            $query->where('name', 'like', "%{$search}%");
        })
        ->paginate(10)
        ->withQueryString()
        ->through(fn($user) => [
            'id' => $user->id,
            'name' => $user->name
        ]),

    'filters' => Request::only(['search'])
]);
```
Since the `Users` page component already uses the `$users` variable, the current data displayed on the page will be replaced with whatever is returned from the search query. 
The `->when()` method allows us to just apply the filtering when there is a value in the `search` query string. 
It is important to make sure to use `->withQueryString()` so that the pagination links are updated with the query string included. Without it, going to a new page in the data set will reset the search.
An addition variable called `filters` is now being sent to the `Users` page. This is used to populate the search field with whatever is in the `search` query string. In the component, the default value for the search input field will be set to the `filters.search` value.
```js
data() {
    return {
        search: this.filters.search
    }
}
```
#### Throttling and Debounce the Ajax Request
There are too many requests generated when the search is used and therefore some throttling should be used on the `watch` listener. This is done using `lodash`. Difference between `throttle` and `debounce` is that `debounce` will wait the specified time after you stop typing. So if you keep typing, no request is sent to the server. The request is then sent `x` amount of ms after you stop. 
```js
//add the import to where the other imports are used.
import throttle from "lodash/throttle";
//import debounce from "lodash/debounce";

//change the previous watch method to the following. Adds 500ms delay
watch(search, throttle(function (value) {
  Inertia.get('/users', { search: value }, {
    preserveState: true,
    replace: true
  });
}, 500));

//change the previous watch method to the following. Adds 300ms delay after user stops typing
//watch(search, debounce(function (value) {
//  Inertia.get('/users', { search: value }, {
//    preserveState: true,
//    replace: true
//  });
//}, 300));
```

### Flash Messages
See the section above called ***Shared Data on All Pages*** on how to add a flash message to all pages.