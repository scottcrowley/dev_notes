# Laravel Notes - Forms


## Notes from the Laracast series about Laravel
* ### Forms
    * All forms must have a `CSRF` field in it to protect against hacking attempts. The following needs to be with the form tags in a view: `{{ csrf_field() }}` This will create a hidden field in the form with a token that is used to verify the form.
        * There is a new blade directive that can be used instead: `@csrf`
    * If a form is being used for either a `PATCH` or `DELETE` request, then a `{{ method_field('PATCH') }}` is needed. This is because currently html doesn't support anything besides `GET` and `POST` as a value in the method parameter of the form tag.
        * There is a new blade directive that can be used instead: `@method('PATCH')`
    * #### Validation
        * In the processing method of your controller you can call the `validate()` method before you call your create method.
            `This example is using a store method within a Posts controller (blog example).`
            ```php
            public function store() {
                $this->validate(request(), [
                    'title' => 'required|min:10' //a pipe separated list that pertains to a field name in the submitted values (required and a min of 10 characters)
                ]); 
                //This method receives the form values and an 
                //array of validation rules.
                //request() helper function contains all 
                //the submitted form values in this example
                //make sure you have a section to display 
                //the global errors variable or no 
                //validation alerts will be visible
            }
            ```
        * Validation rules can either be a string separated by a `|` or an array
            ```php
            request()->validate([ 'field_name' => 'required|min:2' ]);
            request()->validate([ 'field_name' => ['required', 'min:2'] ]);
            ```
        * When you call `validate()` on the request, all the validated attributes are returned. You can then assign that to a var and pass it to the `create` method instead of getting the values from the `request` object
        * There may be a time when you only want to validate a field if there is data present. To do this you add the "sometimes" validation rule.
            ```php
            'email' => 'sometimes|required|email'
            ```
        * There may be sometimes where you want to allow a field to be empty. You need to add the nullable validation rule to allow this.
            ```php
            'published_at' => 'nullable|date'
            ``` 
        * **Comparing dates:**
            * If you are trying to compare to dates (start & end), the validation rules could look like this:
                ```php
                'start' => 'required|date',
                'end' => 'required|date|after:start',
                ```
    * #### Form Requests - A form request class can be used to process more complex validation logic. Following notes come from [Build A Laravel App with TDD: Improve Test Arrangements with Factory Classes](https://laracasts.com/series/build-a-laravel-app-with-tdd/episodes/17)
        * **Make the form request class**
            * `php artisan make:request UpdateProjectRequest`
            * `UpdateProjectRequest` class
                * rules method contains all the validation rules exactly the same way as it used to appear in the controller: `return request()->validate(['title'=>'sometimes|required', 'description'=>'sometimes|required', 'notes'=>'nullable']);`
                    ```php
                    public function rules() {
                        return [
                            'title'=>'sometimes|required', 
                            'description'=>'sometimes|required', 
                            'notes'=>'nullable'
                        ];
                    }
                    ```
                * `authorize` method contains all the authorization logic just like how it appeared in the controller `update` method: `$this->authorize('update', $project);`
                    ```php
                    public function authorize() {
                        return Gate::allows('update', $this->route('project')); 
                        //need to import Illuminate\Support\Facades\Gate
                    * }
                    ```
                    * The `authorize` method used in the controller can't be used since it was imported in the controller and isn't available in the form request
                    * To access the type hinted `$project` variable from the controller, you can use `$this->route` method and retrieve the `project` from the route.
                    * This is possible because the route has a variable called `project` specified in the `routes/web.php` file.
                        * `Route::patch('/projects/{project}', 'ProjectsController@update');`
            * In the `ProjectsController:update` method, you need to type hint the form request along with the project.
                * All validation will now occur before the update method is hit when the form request is type hinted.
                    ```php
                    public function update(UpdateProjectRequest $request, Project $project) { 
                        //both UpdateProjectRequest and Project need to be imported at the top of the class
                        $project->update($request->validated()); 
                            //the validated data from the Form Request is 
                            //now available in the $request object by 
                            //using the validated() method.
                        return redirect($project->path());
                    }
                    ```
            * If you want to change the validation error messages, you can add a `messages` method to the form request class.
                ```php
                public function messages() {
                    return [
                        'title.required'=>'A title is required', 
                        'description.required'=>'A description is required'
                    ];
                }
                ```
        * **Using multiple error bags:** when there are more than one form on the page and you want to isolate the errors for a specific form
            * Within your form request class, add the following protected property `protected $errorBag = 'formIdentifierName';`
            * To display the errors, just add `formIdentifierName` before any method call on the error bag.
                ```php
                @if ($errors->formIdentifierName->any()) //checks to see if any errors exist for the error bag formIdentifierName
                @foreach ($errors->formIdentifierName->all() as $error)
                ```
            * If you are using a partial to display errors and it can be used for multiple forms, you can make it dynamic by doing:
                ```php
                @if ($errors->{ $bag ?? 'default' }->any()) //default is the default name of the error bag.
                @foreach ($errors->{ $bag ?? 'default' }->all() as $error)
                ```
                * Then when you are calling the partial in your blade template, add the `$bag` variable if you want to specify something other than default
                    ```php
                    @include('errors', ['bag' => 'formIdentifierName'])
                    ```
            * If you use named error bags and are performing a test that check if the `sessionHasErrors`, you need to add a third argument to the method, which is the name of the error bag. The second argument of this method is for formatting and is null by default.
                ```php
                ->assertSessionHasErrors('name of field', null, 'formIdentifierName');
                ```
                * The same formatting must be used for the `assertSessionDoesntHaveErrors` method as well.
            * There is another method you can use when dealing with named error bags and `assertSessionHasErrors`. It's called `assertSessionHasErrorsIn` and receives the name of the error bag as the first argument, the keys as the second and formatting as the third.
                ```php
                ->assertSessionHasErrorsIn('formIdentifierName', 'name of field');
                ```
    * #### User Registration Form Password Validation
        * It is important that the form field name and id for the password confirmation field be called `password_confirmation` and the regular password field have the name and id of `password`
        * In the `store` method of the `RegistrationController`, the validation rules must specify that you want to use password confirmation. 
            ```php
            $this->validate([ 'password' => 'required|confirmed' ]);
            ```
            Now when the form is validated it will make sure that password and password_confirmation both match
    * #### Mass Assignment
        * This is when you are not explicitly assigning allowed form fields.
        * When the `create()` method is called in a model, it is important to send only the fields you are wanting and not all submitted form fields using `request()->all()`
            ```php
            Post::create([
                'title' => request('title'),
                'body' => request('body')
            ]);
            ```
        * A `protected $fillable` property array should be assigned in the model class that pertains to the form being used.
            * the array contains only the names of the fields that are ok to update
        * The `protected $guarded` property array is like the `$fillable` property except the opposite. Allow all fields except…
            * If set to an empty array, then that is bypassing the Mass Assignment protection feature
            * If this is being used it may be more efficient to create your own Model class that extends the Eloquent model class and add this property to that. Then all other model classes will extend the new model class instead of the Eloquent model class and all form fields will be allowed in all models.

                Create a new model
                ```php
                namespace App;
                use Illuminate\Database\Eloquent\Model as Eloquent;

                class Model extends Eloquent {
                    protected $guarded = [];
                }
                ```
                All other model classes would now extend this model instead of `Illuminate\Database\Eloquent\Model`
                ```php
                namespace App;

                class Post extends Model {

                }
                ```
    * #### Useful Tips
        * You can use `{{ old{'filename') }}` to populate a form field with previous data. Useful when there is an user error in the form and you want to retain the info they have already used.
        * If you are using `Recaptcha` or need any other `AJAX` functionality within your php scripts, you can use a tool called [Zttp](https://github.com/kitetail/zttp) which is a wrapper for `Guzzle`.
            * Install using `composer require kitetail/zttp`
            * `Zttp` sends `AJAX` requests as `JSON`, so if you need to send as standard form params then you can use the `asFormParams()` method. This is needed when using `Recaptcha`
                ```php
                $response = Zttp::asFormParams()
                    ->post(
                        'https://www.google.com/recaptcha/api/siteverify', 
                        [
                            'secret' => config('services.recaptcha.secret', 
                            'response' => $request->input('g-recaptcha-response'), 
                            'remoteip' => $_SERVER['REMOTE_ADDR']
                        ]
                    );
                ```