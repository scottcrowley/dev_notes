# Laravel - Practical Usage Examples

#### Notes from miscellaneous tutorials on how to use Laravel in different scenarios

## Server-Fetched Partials
#### Background:
* Tutorial taken from [Laracast - JavaScript Techniques For Server-Side Applications Ep 1](https://laracasts.com/series/javascript-techniques-for-server-side-developers/episodes/1)
* The idea behind server side partials is to have a page shell and html code that renders the data both in a seperate file. That way the data partial file can then be retrieved using Javascript's Fetch API, while leaving most of the heavy lifting in php. The retrieved partial is not JSON as it normally would be with a regular AJAX request, but instead it is the rendered view html containing all the pertinent data.
* Using Server-Fetched partials also allows for easier testing since you don't need to test the javascript, just the rendered html from the partial. PHPUnit will also generate errors if there is something wrong with your blade template, which is something you don't get if you are using Vue.

#### Code Segments
##### /resources/views/sponsorships.blade.php
```html
<ul id="js-developers-partial-target">
    <!--  -->
</ul>
<script>
    function fetchDevelopers() {
        fetch('/partials/developers')
            .then(response => response.text())
            .then(html => {
                document.querySelector('#js-developers-partial-target').innerHtml = html
            });
    }
</script>
```
* The above snippet just contains the `ul` that the fetched partial will be inserted into whenever the `fetchDevelopers` method is called.
* Notice that the id on the `ul` is very explicit in its description. It is recommended that you prefix these types of id's with "`js-`" so it is very clear that this element is attached to something in javascript. It makes things easier to read.

##### /tests/features/SponsorshipsTest.php
```php
function can_view_sponsorship_page() 
{
    $this->get('/')->assertSuccessful();
}
function can_fetch_developers_partial() 
{
    factory(User::class, 5)->create();

    $this->get('/partials/developers')
        ->assertViewHas('users', function($users) {
            return count($users) === 5;
        })
        ->assertSuccessful();
}
```
* Having these tests, allows us to be confident that we are able to get to all the routes and the proper number of users is being sent to the partial view as well as confirmation that there is nothing wrong with the actual view template.

## Deferred Loading and Caching Server-Fetched Partials
#### Background:
* Tutorial taken from [Laracast - JavaScript Techniques For Server-Side Applications Ep 2](https://laracasts.com/series/javascript-techniques-for-server-side-developers/episodes/2)
* This tutorial uses the [Include Fragment Element](https://github.com/github/include-fragment-element) library by GitHub, to make it easier to inject fetched content using Javascript from a backend php partial file.
* Unpkg.com is also being used to add the *Include Fragment Element* package to the page using a cdn version instead of installing the full package into your project. The following script tag, located before the closing body tag, is being used to import the cdn version. Note that `@github/include-fragment-element` is the package name that you would normally install using `npm install --save @github/include-fragment-element`.
    ```html
    <script src="https://unpkg.com/@github/include-fragment-element"></script>
    ```

#### Setup: 
Mimic the loading of a html table containing invoices and their status. Assume the invoices are being fetched from a third party api or something, where it may take a long time to retrieve. The regular Fetch API was used in the beginning to show how you'd do this using normal Javascript, but then the *Include Fragment Element* library was used to make everything much cleaner.
```html
<div class="relative rounded overflow-hidden" id="js-invoices-partials-target">
    <script>
        fetch('/partials/invoices')
            .then(response => response.text())
            .then(html => {
                document.querySelector('#js-invoices-partials-target').innerHtml = html
            });
    </script>
</div>
```
Retrieves the rendered html from a partial file that contains php to get the data and then spit out a rendered blade view.
```html
<div class="relative rounded overflow-hidden">
    <include-fragment src="/partials/invoices">
        Loading....
    </include-fragment>
</div>
```
The above code is how it would look when using the *Include Fragment Element* library from GitHub. Note that there is no Javascript being used.

#### Code Segments
##### /routes/web.php
```php
use Illuminate\Support\Carbon;

Route::get('/', function () {
    return view('invoices', [
        'cachedInvoicesPartial' => Cache::get('partials.invoices'),
    ]);
});

Route::get('/partials/invoices', function () {
    return Cache::remember('partials.invoices', Carbon::parse('10 seconds'), function () {
        return view('_invoices', [
            'invoices' => App\Invoice::all(),
        ])->render();
    });
});
```
* The root route loads the invoices.blade.php file and passes in the cached invoices if they are available. If none are available then it will load another partial that contains placeholder html to display while the first fetch is performed.
* The route for the partial itself, returns either the cached table or creates a new cached table using the rendered _invoices partial blade view.
    * This route is being called using the *Include Fragment Element* `src` attribute.
    * Notice that the `view()` method is calling for the partial file and passing in all the invoice models and then the `render()` method is being called, which will spit out all the rendered html from the compiled blade view file.

##### /resources/views/invoices.blade.php
```html
<div class="relative rounded overflow-hidden">
    @if ($cachedInvoicesPartial)
        {!! $cachedInvoicesPartial !!}
    @else
        <include-fragment src="/partials/invoices">
            @include('_invoices-placeholder')
        </include-fragment>
    @endif
</div>
```
* You need to make sure that the rendered html table, retrieved from the cache, is not escaped by using the `{{!!  !!}}` syntax
* If there are no cached invoices when this page is loaded then the `_invoices-placeholder.blade.php` placeholder partial is loaded while the invoice content is retrieved from `_invoices.blade.php`

##### /resources/views/_invoices.blade.php
```html
<table class="bg-gray-100 w-full">
    @foreach ($invoices as $invoice)
        <tr class="border-b">
            <td class="p-4">
                <div>
                    <span class="font-medium">{{ $invoice->title }}</span>
                    @if ($invoice->paid_at)
                        <span 
                            class="ml-2 bg-gray-300 font-semibold p-1 rounded text-gray-600 text-sm"
                        >
                            Paid
                        </span>
                    @endif
                </div>
            </td>
            <td class="p-4 text-right font-mono font-semibold">
                <span>{{ money($invoice->amount_in_cents) }}</span>
            </td>
            <td class="p-4 text-right">
                <span class="text-blue-600">Download</span>
            </td>
        </tr>
    @endforeach
</table>
```
* Renders the table containing all the invoices sent to the view from the Invoice::all() method in the routes file.

##### /resources/views/_invoices-placeholder.blade.php
```html
<div class="absolute inset-0" style="background-image: linear-gradient(0deg, #cbd5e0, transparent);"></div>

<table class="bg-gray-100 w-full">
    @foreach (range(1, 10) as  $invoice)
        <tr class="border-b">
            <td class="p-4">
                <span class="bg-gray-300 text-transparent rounded">
                    {{ str_repeat('_', rand(10, 30)) }}
                </span>
            </td>
            <td class="p-4">
                <div class="bg-gray-300 text-transparent rounded">$123.45</div>
            </td>
            <td class="p-4">
                <div class="bg-gray-300 text-transparent rounded">Download</div>
            </td>
        </tr>
    @endforeach
</table>
```
* This place holder table is just eye candy and is used to make it look like how Facebook does when it is loading data into a page. Gives the illusion that there is data there but is grayed out while it loads.

## Make Your Apps Faster With Turbolinks
#### Background:
* Tutorial taken from [Laracast - JavaScript Techniques For Server-Side Applications Ep 3](https://laracasts.com/series/javascript-techniques-for-server-side-developers/episodes/3)
* This tutorial uses the [Turbolinks](https://github.com/turbolinks/turbolinks), which can make your app feel like a SPA with very little setup. Turbolinks will hijack all links on the page but instead of doing a new page load, it will swap out the `<body>` content for the content found by following the link. It will also do a merge of the content found in the `<head>` tag, so if there are additional page requirements for the new page, they will also be added.
* Unpkg.com is also being used to add the *Turbolinks* package to the page using a cdn version instead of installing the full package into your project. The following script tag, located before the closing head tag, is being used to import the cdn version. Note that `turbolinks` is the package name that you would normally install using `npm install --save turbolinks`.
    ```html
    <script src="https://unpkg.com/turbolinks"></script>
    ```

#### Setup: 
There really isn't any other setup required. It may just work straight away after adding the above script tag. There are some gotchas to consider though. See below.

#### Gotchas:
1. If you have any javascript, that resides in the `<head>` tag and is listening for the page to be completely loaded.
    ```html
    <script>
        document.addEventListener('DOMContentLoaded', () => {
            console.log('foo');
        });
    </script>
    ```
    Turbolinks will not refire the DOMContentLoaded event when a new link is click. To add this behavior back in use the following turbolinks event:
    ```html
    <script>
        document.addEventListener('turbolinks:load', () => {
            console.log('foo');
        });
    </script>
    ```
    This event will fire everytime Turbolinks has finished loading a new page.
1. If you have some javascript in the `<body>` tag and you only want it to be fired one time, then you need to add a special `data` attribute to the `<script>` tag. There may be times where you have the `<script>` tag within the `<body>` tag, within a layout file that loads the content via partials (as outlined in other previous examples) but you only want the javascript to be fired once. Since Turbolinks swaps out the entire `<body>` tag, the javascript will be fired with every link clicked. To stop this, add `data-turbolinks-eval="false"` to the `<script>` tag within the `<body>` tag.
    ```html
    <script data-turbolinks-eval="false">
        console.log('foo');
    </script>
    ```
    This tells Turbolinks to ignore this tag if it has already encountered it.
1. There maybe times when you don't want Turbolinks to hijack a link and you want the link to act normally with a regular page load. You can add the data attribute `data-turbolinks="false"` to the link you don't want Turbolinks to hijack.
    ```html
    <a href="/account" data-turbolinks="false">Account</a>
    ```
    Turbolinks will now ignore this link.
1. If you are using a redirect within a Route, then you need to add some middleware to tell Turbolinks that a redirect has occurred. For example: You have a link that goes to `/profile` but you want it redirected to `/account` by using `Route::redirect('/profile', '/account');` in the `routes/web.php` file. So when you goto `/profile`, you'll be redirected to `/account` and the url in the browser will reflect this. However, when using Turbolinks in this scenario, the redirect will be performed but the url won't update and will still show `/profile`. To get around this, you need to add `Turbolinks-Location` to the header of the final redirect. This can be done using middleware in Laravel. More info about [redirects](https://github.com/turbolinks/turbolinks#following-redirects) on the Turbolinks site.
    ```zsh
    php artisan make:middleware SetTurbolinksHeader
    ```
    *app/Http/Middleware/SetTurbolinksHeader.php*
    ```php
    public function handle($request, Closure $next)
    {
        $response = $next($request);

        $response->header('Turbolinks-Location', $request->url());

        return $response;
    }
    ```
    *app/Http/Kernel.php* add the following to `'web'` `$middlewareGroups` property
    ```php
    protected $middlewareGroups = [
        'web' => [
            //....
            SetTurbolinksHeader::class, //make sure to import the class
        ], 
        //....
    ];
    ```
    Now with every redirect, Laravel will add `Turbolinks-Location` to the header.
