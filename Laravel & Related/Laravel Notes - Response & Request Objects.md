# Laravel Notes - Response & Request Objects


## Notes from the Laracast series about Laravel
* ### The Response Object
    * A list of HTTP status codes can be viewed in the `symfony/http-foundation/Response.php`
    * The following methods are a few that can be used on a response object
        * `->getContent()` - retrieves the content of the response
        * `->headers()` - retrieves the http header of the response
        * `->getStatusCode()` - retrieves the http status code of the response
    * **Creating Responses**
        * A response can be view
            * Using the view helper function: `return view('threads.index');`
            * The view method can also accept data and http status codes as the second and third parameter
                * Using the view method on the response object: `return response()->view('hello', $data, 200)->header('Content-Type', $type);`
        * You can return text or an object along with a http status code with a regular response
            * `return response('Hello', 200);` or `return response($thread, 201);`
            * When returning an object or array, it will automatically be cast to `JSON`
        * You can modify the header of the response
            * `return response('Hello', 200)->header('Content-Type', 'text/plain');`
            * If you are changing several headers then you can either use multiple calls to the `->header` method or do this:
                ```php
                return response($content)
                    ->withHeaders([
                        'Content-Type' => $type, 
                        'X-Header-One' => 'Header Value', 
                        'X-Header-One' => 'Header Value'
                    ]);
                ```
        * You can attach cookies to the response as well:
            ```php
            return response($content)->header('Content-Type', $type)->cookie('name', 'value', $minutes);
            ```
            The above `->cookie` method has a few more available parameters and is basically the same as PHP's `setcookie` method.
    * **Redirect Responses**
        * Using the redirect helper function: `return redirect('home/dashboard');`
        * Using the back helper function: `return back()->withInput();`
        * Redirect to a named route: `return redirect()->route('threads.index');`
            * Route parameters can be passed in the second argument of the route method: `return redirect()->route('profile', ['id' => 1]);`
            * Populating parameters using Eloquent Models
                * Pass the entire model to the route and Eloquent will use the `id` value of the model
                    * `return redirect()->route('profile', [$user]);` `$user->id` will be used as the identifier when passed to the profile route
                    * To use a different identifier you need to change the `getRouteKey` method in the model class
                        * `public function getRouteKey() { return $this->slug; }`
        * Redirect to a controller action
            * `return redirect()->action('HomeController@index');`
            * If the controller action requires parameters, they can be sent as the second argument in the action method
                * `return redirect()->action('UserController@profile', ['id' => 1]);`
        * Redirect to an external domain
            * `return redirect()->away('https://www.google.com');`
        * Redirect with a flash session message
            * `return redirect('dashboard')->with('status', 'Profile updated!');`
            * `status` will then be available using the session helper function. `session('status')`
    * **JSON Responses**
        * The json method will automatically set the appropriate content-type header and convert the given array or object to `JSON` using the `json_encode` PHP function.
            * `return response()->json(['name' => 'Abigail', 'state' => 'CA']);`
        * `JSONP` responses can be created by using the `withCallBack` method after the `json` method.
            * `return response()->json(['name' => 'Abigail', 'state' => 'CA'])->withCallback($request->input('callback'));`
    * **File Download Responses**
        * The `download` method can be used to force the users browser to download a file at the given path.
            * The first argument in the `download` method is the path to the file
            * The second optional argument in the `download` method is the name of the file. This name will be used to name the file being downloaded on the users computer as well. The file being downloaded needs to have an ASCII file name.
            * The third optional argument in the `download` method is the header information, if it needs to be customized.
            * `return response()->download($pathToFile);`
            * `return response()->download($pathToFile, $name, $headers);`
            * `return response()->download($pathToFile)->deleteFileAfterSend();`
    * **File Streaming Responses**
        * The `streamDownload` method can be used to turn a string response into a download response without having to write the contents of the operation to the users disk. The method accepts a callback as the first parameter, the file name as the second and an optional array of headers
            ```php
            return response()->streamDownload(function () { 
                echo GitHub::api('repo')
                    ->contents()
                    ->readme('laravel', 'laravel')['contents']; 
            }, 'laravel-readme.md');
            ```
    * **File Responses**
        * A `file` method can be used to display a file in the users browser instead of initiating a download. This would be useful for PDF's or certain images. The file path is required as the first argument and an array of headers as an option second.
            * `return response()->file($pathToFile);`
            * `return response()->file($pathToFile, $headers);`
    * Response Macros are also an option. See the documentation for more info. <https://laravel.com/docs/master/responses#response-macros>
* ### Request()
    * To obtain an instance of the current HTTP request you can type-hint the `Illuminate\Http\Request` class on your controller method. `public function store(Request $request) {  }`
    * The request object can contain `POST` & `GET` data
    * **Request `Path` & `Method`:**
        * The `path` method returns the request's path information. So, if the incoming request is targeted at `http://domain.com/foo/bar`, the `path` method will return `foo/bar`:
            * `$uri = $request->path();`
        * The `is` method allows you to verify that the incoming request path matches a given pattern. You may use the `*` character as a wildcard when utilizing this method:
            * `if ($request->is('admin/*')) {  }`
        * The `url` & `fullUrl` methods can be used to retrieve the request url with or without the query string
            * `$url = $request->url();`
            * `$url = $request->fullUrl();`
        * The `method` method can be used to retrieve the type of request
            * `$method = $request->method();  //post or get`
        * The `isMethod` method allows to check if the request type is a certain one.
            * `if ($request->isMethod('post')) {  }`
    * By default, all incoming string fields are trimmed and converted to `null`, if they are empty, because the `TrimStrings` & `ConvertEmptyStringsToNull` middleware are being used. To disable these, remove them from the `App\Http\Kernel` class.
    * **Helpers**
        * `->all()` - returns the value of all the fields contained in the object.
        * `->has('field')` - checks the request and returns boolean if the given field exists
        * `->filled('field')` - does the same thing as `has()` except it checks to see if the field is empty or not.
        * `->only()` - receives an array as a parameter and only returns the values for the keys provided. Can be used to make sure nothing is being injected maliciously.
        * `->except()` - opposite of `only()`.
        * `->intersect()` - receives an array as a parameter and only returns the values for the keys that exist in both the request and the provided array. This can be used to filter out null values, like what can be given by using the `only()` method. **DEPRECATED**
        * `->input('field')` - Retrieves the users input for a given field. This will return values for the entire payload including the query string
            * A default value can be passed if the retrieved value is null. `$request->input('field', 'default');`
            * Use dot notation to access array values.
                * `$request->input('products.0.name'); //['products' => [ ['name = 'Fred'] ]`
                * The `*` can be used as a wild card. `$request->input('products.*.name');`
            * To retrieve all input values as an associative array, don't pass any arguments to the input method. `$request->input();`
        * `->query('field')` - Retrieves the users input for a given field only from the query string. `$request->query('field');`
    * **Retrieve Inputs via Dynamic Properties:**
        * For example, if there is a form with a name field, you can access the value by using `$request->name`;
    * **Retrieving JSON values:**
        * The request must have the correct Content-Type header of `application/json`
        * Dot notation can be used to access specific values.
            * `$name = $request->input('user.name');`
    * **Flashing Input to the Session:**
        * The `flash` method will flash the current request input values to the session, so they will be available during the next request to the application. `$request->flash();`
        * To flash on certain input values to the session, the `flashOnly` and `flashExcept` methods can be used.
            * `$request->flashOnly(['username', 'email']);`
            * `$request->flashExcept('password');`
        * To flash input then redirect you can use the `withInput` method
            * `return redirect('form')->withInput();`
            * `return redirect('form')->withInput($request->except('password'));`
    * **Retrieving old input values from the request:**
        * This gets the values from the previous request. `$username = $request->old('username');`
    * **Retrieving cookies from the request:**
        * `$value = $request->cookie('name');`
        * or `$value = $value = Cookie::get('name'); //uses the cookie facade`
    * **Retrieving files from the request:**
        * The `file` method can be used to gain access to an uploaded file. The `file` method returns an instance of `Illuminate\Http\UploadedFile` class, which extends the PHP `SplFileInfo` class and provides a variety of methods for interacting with the file:
            * `$file = $request->file('photo');`
            * `$file = $request->photo;`
        * To determine if the file is present, use the `hasFile` method.
            * `if ($request->hasFile('photo')) {  }`
        * To check that there were no problems uploading the file, you can use the `isValid` method.
            * `if ($request->file('photo')->isValid()) {  }`
        * The `UploadedFile` class contains many methods that can be used to gather info about a file.
            * `$path = $request->photo->path(); //the path to the file`
            * `$extension = $request->photo->extension(); //tries to determine the extension of the file`
            * Other methods are available. See documentation. <http://api.symfony.com/3.0/Symfony/Component/HttpFoundation/File/UploadedFile.html>
    * **Storing uploaded files:**
        * The `UploadedFile` class contain a `store` method, used to save the uploaded file. A filename should not be given in the path since the system will generate a unique name automatically. If you want to use a specific name, then use the `storeAs` method instead.
            * The `store` method receives the path, relative to the filesystems configured root directory, as the first argument and the name of the disk as an optional second argument.
                * `$path = $request->photo->store('images');`
                * `$path = $request->photo->store('images', 's3');`
        * The `storeAs` method can be used if you want the uploaded file to be stored with a specific name.
            * `$path = $request->photo->storeAs('images', 'filename.jpg');`
            * `$path = $request->photo->storeAs('images', 'filename.jpg', 's3');`
    * **Retrieving details from a route**
        * It is possible to get details like a model from a route by using `request()->route('routeParameter');`
        * For example:

            You have a route...
            ```php
            Route::get('/{client}/projects/create', 'ProjectsController@create')->name('projects.create');
            ```
            ... and you are using a Policy for Projects and need to access the {client} param in the policy. You'd use `request()->route('client');` within the policy to access the `Client` model.
            
            ProjectPolicy.php
            ```php
            public function create(User $user)
            {
                $client = request()->route('client');
                return ($client->user_id == $user->id);
            }
            ```
            *be careful not to use a strict comparison (`===`) since the value for `user_id` may be returned as a string (even thought it was cast as an integer within the `Client` model) and `$user->id` is an integer, as in the case above.*
