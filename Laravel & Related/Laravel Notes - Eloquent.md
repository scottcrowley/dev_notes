# Laravel Notes - Eloquent


## Notes from the Laracast series about Laravel
* ### Eloquent
    * **Database Migrations:**
        * The `enum` field type can be used to specify that the only certain values are accepted. i.e. for a skill level field you only want the following values: `basic, intermediate, advanced`.
            * `$table->enum('level', ['basic', 'intermediate', 'advanced'])->default('intermediate');`
        * If for whatever reason you do not want a primary key given the unsigned attribute (which it does by default) do the following in the migration
            
            ***Before***
            ```php
            $table->bigIncrements('id');
            ```
            ***After***
            ```php
            $table->bigIncrements('id')->unsigned(false);
            ```
    * **Using more complex queries:**
        * `Raw` - The word `raw` can be added to several different SQL actions (select, order by, group by, etc), so you are able to use more specific queries or SQL functions
            * Consider the following query: `select year(created_at) year, monthname(created_at), count(*) published from posts group by year, month order by min(created_at)`
                ```php
                App\Post::selectRaw('year(created_at) year, monthname(created_at), count(*) published')
                    ->groupBy('year', 'month')
                    ->orderByRaw('min(created_at) desc')
                    ->get();
                ```
    * **What is returned:**
        * `find` or `first` - Returns a single model if a result if found.
            * `App\Post::find(4); //returns the post with an id of 4`
            * `App\Post::first(); //returns the first found post`
        * `findOrFail` or `firstOrFail` - tries to find a record with an id of the provided argument or throws an exception if nothing is found. The exception is `Illuminate\Database\Eloquent\ModelNotFoundException`. If the exception isn't caught then a `404 HTTP response` is returned to the user.
            * `$model = App\Flight::findOrFail(1);`
            * `$model = App\Flight::where('legs', '>', 100)->firstOrFail();`
        * Aggregate methods like `count`, `sum`, `max`, `min`, `avg`, - Returns the appropriate value
            * `$count = App\Flight::where('active', 1)->count();`
            * `$max = App\Flight::where('active', 1)->max('price');`
        * `exists` and `doesntExist` - Returns a boolean
            * `DB::table('orders')->where('finalized', 1)->exists();`
            * `DB::table('orders')->where('finalized', 1)->doesntExist();`
        * `get` - returns a collection of models found.
            * `App\Post::where('user_id', 4)->get();`
        * `all` - returns all records from the database, as a collection of models, but cannot be used with any other methods like `where` or `sortBy`.
            * `App\Post::all();`
        * `latest` - returns all found records and orders them by the latest record first. This is based on '`created_at`' by default but can be changed by passing a different field name.
            * `App\Post::latest(); //latest can be used with or without a following ->get()`
        * `oldest` - returns all found records and orders them by the oldest record first. This is based on '`created_at`' by default but can be changed by passing a different field name.
            * `App\Post::oldest(); //latest can be used with or without a following ->get()`
    * **Pivot Tables**
        * The table using follows the naming convention of alphabetical order and singular form of `table1_table2`. i.e. To create a `tags` to `posts` relationship table it would be called `post_tag`.
        * The table usually doesn't have an auto incrementing id field.
        * The table would have to following fields in the migration
            * `$table->integer('post_id')`
            * `$table->integer('tag_id')`
            * `$table->primary([ 'post_id', 'tag_id' ])` This makes it so there can never be a duplicate entry where the post id and tag id exist in the same record more than once.
        * The relationship between tags and post is a belongsToMany. This relationship type almost always should be associated with a pivot table
            * In the `Post` model, you would add a `tags` method that references the `Tag` model: 
                ```php
                public function tags() { 
                    return $this->belongsToMany(Tag::class); 
                }
                ```
            * In the `Tag` model, you would add a `posts` method that references the `Post` model: 
                ```php
                public function posts() { 
                    return $this->belongsToMany(Post::class); 
                }
                ```
            * As stated above, Laravel will normally expect a pivot table to be named the singular name of each table in alphabetical order. If the name of the pivot table differs from this naming convention, then you need to provide the name of the table in the relationship method.
                ```php
                public function tags() { 
                    return $this->belongsToMany(Tag::class, 'different_pivot_table');
                }
                ```
        * Laravel has the ability to load all the tags associated with a post without making the normal additional database query.
            * So instead of running through each post in the results of (`App\Post::all()` gets all posts) to then perform a database query to get the tags associated with the each post (`$post->tags;`) you can just use `$posts = App\Post::with('tags')->get();` This will execute the tags relationship when each post is retrieved using the tags method.
        * To add a new tag to a post you don't use the `create()` or `save()` methods, as before. Now you can retrieve the post (`$post = App\Post::first();`), then the tag (`$tag = App\Tag::where('name', 'personal')->first();`) then call the `tags` method from the `$post` object and attach the `$tag` object. `$post->tags()->attach($tag);`
        * To remove a tag to post relationship you use the `detach()` method. Using the example above, you'd use `$post->tags()->detach($tag);`
        * There is a way to sync the data that should be in the pivot table for a specific object. i.e. You want to sync all the tags that should be associated with a specific post.
            * Say that a post should only be associated with tag that have the following id's `[1,6,8]`. You can sync the posts associations with the tags by providing the array of id's to the `sync` method on the model.
                * `$post->tags()->sync([1,6,8]);` Any id that is in the pivot table and not in the given array, will be detached from the pivot table. The end result will be that the pivot table will only contain the given id's
                * You can also sync without detaching any id's by running the `$post->tags()->syncWithoutDetaching([1,6,8]);` This will leave any id's that are in the pivot table that are not in the array. It basically just adds the given id's to the pivot table that are not already there.
        * You can also toggle the values in a pivot table. If the given id exists in the pivot table then it will be detached. Otherwise it will be attached to the pivot table.
            * `$post->tags()->toggle([1,6,8]);`
    * **Special Properties within a Model**
        * `protected $with = [];` An array of relationship method names that you want to always include when a model is retrieved. See the section on Eager Loading below.
        * `protected $appends = [];` An array of attribute names that you want to always include when a model is retrieved. Helpful when using a JSON request to get the data. See the section on Mutators below.
        * `protected $fillable = [];` An array of database field names that you only want to use. The opposite of $guarded. Protects against Mass Assignment issues. See section on Mass Assignment above.
        * `protected $guarded = [];` An array of database field names that you do not want to be used. The opposite of `$fillable`. Protects against Mass Assignment issues. An empty array will bypass the Mass Assignment protection entirely. See section on Mass Assignment above.
    * Eloquent always assumes that when a record is retrieved, the query is based off of the primary key.
        * This behavior can be changed by adding a `getRouteKeyName()` method to the model in question
            * In the `Tag` model: `public function getRouteKeyName() { return 'name'; }` which tells Eloquent to use the `name` field when querying the table instead of the primary key.
    * **`hasOneThrough` Relationships**
        * Using an example from the Lego project: An inventory (with fields `id`, `version`, `set_num`) has a `hasManyThrough` relationship with `themes` through the `sets` table. Below is an explanation of each parameter being used with the method:
            ```php
            hasOneThrough(
                $related, //= the table or model you want to reach (Theme::class)
                $through, //= the table or model you need to go through to reach $related (Set::class)
                $firstKey, //= the field to use in the where clause and the select for the through table ('set_num')
                $secondKey, //= the field in the $related table that is joined to the $through table ('id' on themes)
                $localKey, //= the field in the current model to get the value for in the where clause ('set_num' on inventories)
                $secondLocalKey //= the field in the $through table that is joined to the $related table ('theme_id' on sets)
            ) {

            }
            ```
            Laravel relationship in the `Inventory` model
            ```php
            public function theme()
            {
                return $this->hasOneThrough(
                    Theme::class, 
                    Set::class, 
                    'set_num', 
                    'id', 
                    'set_num', 
                    'theme_id', 
                    'set_num'
                );
            }
            ```
            SQL being executed by the about relationship
            ```sql
            select 
                `themes`.*, `sets`.`set_num` as `laravel_through_key` 
            from `themes` 
            inner join 
                `sets` on `sets`.`theme_id` = `themes`.`id` 
            where `sets`.`set_num` = '7922-1' 
            limit 1
            ```
            SQL with placeholders
            ```sql
            select 
                $related.*, $through.$firstKey as `laravel_through_key` 
            from $related 
            inner join 
                $through on $through.$secondLocalKey = $related.$secondKey 
            where $through.$firstKey = $localKey 
            limit 1
            ```
    * **`hasManyThrough` Relationships**
        * Using an example where there is a users, posts and affiliations (contains either 'Liberal' or 'Conservative' and the id is stored on the users table) table. If you wanted to see all posts from Liberals then you could setup a `hasManyThrough` relationship on the affiliations model.
            * `public function posts() { return $this->hasManyThrough(Posts::class, Users::class); }` Think of it like: `affiliations` have many `posts` through `users` on the `users` table.
    * **`hasManyThrough` Relationship when using a Pivot Intermediate Table**
        * Lego DB Example: We want to grap all parts that are associated with a storage location. `StorageLocation`->belongsToMany->`PartCategory` & uses pivot part_category_storage_location to create a relationship. `PartCategory`->belongsTo->`Part`. If the pivot table wasn't being used and StorageLocation_id was stored in PartCategory, then you could use a simple `hasManyThrough` relationship. At the time this note is being made, this isn't possible using simple relationships.
        * Found a package on GitHub to accomplish this. [eloquent-has-many-deep](https://github.com/staudenmeir/eloquent-has-many-deep)
            * Install using
                ```zsh
                composer require staudenmeir/eloquent-has-many-deep:"^1.7"
                ```
            * Add the following use statement to the class where you want to use this functionality
                ```php
                class StorageLocation extends Model
                {
                    use \Staudenmeir\EloquentHasManyDeep\HasRelationships;
                }
                ```
            * Using example above, the following relation was created in the StorageLocation model
                ```php
                public function parts()
                {
                    return $this->hasManyDeep(
                        Part::class, 
                        ['part_category_storage_location', PartCategory::class]
                    );
                }
                ```
            * Check [documentation](https://github.com/staudenmeir/eloquent-has-many-deep) for examples on how to use in different scenarios.
    * **`Polymorphic` Relationships**
        * Using an example where there is a `users` table, a `posts` table, a `comments` table and a `likes` table. The `likes` table will be used for many different types of likes (`comments`, `posts`, etc) and consist of the following fields: `user_id` = user id of the user who liked the item, `likeable_id` = the id of the item being liked (a post, comment, etc) , `likeable_type`  = the class name for the item being liked (`App\Post`, `App\Comment`, etc). 
            * The `Post` and `Comment` model will have a `likes()` method, which will retrieve all the likes for that type.
                * This method will return a `morphMany` relationship. `return $this->morphMany(Like::class, 'likeable');` The 2nd param is stating the naming convention used in the like pivot table (`likeable_id` & `likeable_type`)
            * As a side note: in the migration for the likes table you can now add `$table->morphs('likeable');` This will add the `likeable_type` and `likeable_id` fields to the table automatically when it is migrated.
                * You can also use the `nullableMorphs` method if you want to allow the `_type` & `_id` fields to be `nullable`.
        * Another example would be there is a `videos` table, a `series` table and a `collections` table. A video can be part of a `series` (stored in the `series` table) or any other miscellaneous type of grouping which is stored in the `collections` table.
            * The `videos` table migration contains `$table->morphs('watchable');` which will create `watchable_type` and `watchable_id` in the table during migration
            * Each of the `series` and `collection` models will have a `videos` method
                * `public function videos() { return $this->morphMany(Video::Class, 'watchable'); }` Returns all videos with the appropriate relationship
            * The `video` model will have a `watchable` method to retrieve the relationship
                * `public function watchable() { return $this->morphTo(); }` This will return either return a collection of a series model or collection model for the video in question. The method name needs to be the same as the name chosen in the migration, which is `watchable` in this case. 
                    * Or if you want the method name to be something else, then you can pass the morphing name in the `morphTo` method.
                        * `public function viewable() { return $this->morphTo('watchable'); }`
        * If you don't want to use the default naming convention for the type (i.e. `App\Series` or `App\Collection`). Using the video example above. Instead of having `App\Series` stored in the database, you wanted just `series`. And instead of having `App\Collection` stored in the database, you wanted just `collection`.
            * In `app\providers\AppServiceProvider.php`, with in the `boot` method.
                * add `use Illuminate\Database\Eloquent\Relations\Relation;` to the top of the document.
                ```php
                public function boot() { Relation::morphMap( [ 'series' => 'App\Series', 'collection' => 'App\Collection' ] ); }
                ```
    * **Mutators** - A way to retrieve and modify attributes within a given model. This is used when you'd like to either retrieve a column value and modify it or modify a specific column to have a newly formatted value.
        * You can add a getter for an existing attribute or a new attribute by wording the models method name the following way `getColumnNameAttribute()`, where the `ColumnName` is the `“studly”` case version of the attribute you want. (i.e. `column_name`). If a param is included with the method `getColumnNameAttribute($value)`, then a value will first be passed to the it then it can be returned.
            
            Within the model:
            ```php
            public function getFirstNameAttribute($value) {
                return ucfirst($value);
            }
            ```
            You can then access the attribute:
            ```php
            $user = App\User::find(1);
            $firstName = $user->first_name; //should return a Capitalized version of the first_name column from the model.
            ```
        * You can also set an attributes value by using the same approach as above but use `set` instead of `get`.
        * You can create new attributes this same way but you would not use a param. For example: You want to add `example_attribute` attribute to the model (where the `example_attribute` column doesn't exist), so you can access it like `$user->example_attribute`. You would add a `getExampleAttributeAttribute()` method to your model, which would handle your required processing for the value.
        * It is also possible to hook into a model event and then apply a mutator then:

            Within the model:
            ```php
            protected static function boot() {
                parent::boot();
                static::created(function ($thread) {
                    $thread->update(['slug' => $thread->title]);
                }
            }
            ```
            This would then trigger the `setSlugAttribute` method when the thread was created.
    * **Miscellaneous Methods**
    * `whereMonth` and `whereYear` can be used to filter a query using a special where clause. Usage: `whereMonth('field_name', value);`
    * the `pluck()` method can be used to pluck out just the passed in field name with the results instead of all fields. `$tags = App\Tag::pluck('name');` will just included all the `name` values from the `Tags` table.
    * the `has()` method is used when you want to check a method within a model for results before returning all results. i.e. You want to display all the tags that have posts attached to them. `$tags = App\Tag::has('posts')->pluck('name');` This will use the `posts` method in the `Tag` model to see if there are related posts and then only return the tag names that have posts.
    * `take()` will limit the record set to the number provided.
    * `chunk()` will allow you to pull sets of the given number from the database at a time. This saves on memory usage. Usage: `\App\Users::chunk(100, function() { //do something with the chunk of users here. });` will pull all users from the database in chunks of 100 and perform the callback function provided.
    * `is()` & `isNot()` can be used to check if 2 models are the same or not.
        ```php
        // checks to see if the current logged in user is the same as the owner of a project.
        auth()->user()->is($project->owner);
        ```
    * **Eager Loading:** Defining relationships between different models or tables in the database
        * Using Blog Post/Comments example from series
            * Using the logic that a post `hasMany` comments and a comment `belongsTo` one post
                * The `Comments` table would contain a field reference to the `Posts` table. In this case it would be call '`post_id`'. Using the convention of the singular version of the table being referenced.
                * In the `Post` model, a `comments()` method would be added to gather all the comments associated with a post. This method would return a `hasMany` instance of the `Comment` model class. `return $this->hasMany(Comment::class);` Doing it this way, the `Post` model will auto fill the `post_id` with the current post id and return that instance of the `Comment` class.
                    * The `comments` method should not be called as a method but as a property instead. When Laravel sees this, it will load the relationship. `$comments = $post->comments;`
                * To perform the inverse and get the `post` associated with a `comment`, you would add a `post()` method to the `Comment` model. This method would return a `belongsTo` instance of the `Post` model class. `return $this->belongsTo(Post::class);`
                    * Just like the `comments` method in the `Post` model, you would call the `post` method in the `Comment` model as a property and not a method. `$post = $comment->post;`
                * Having the `comments()` & `post()` methods in the different model classes also allows us to call other methods on those classes using the relationships already setup.
                    * For example: someone is adding a comment to a post.
                        * Set up a route to accept the data from the form being submitted. This would call a `store` method in the `CommentsController`, which would accept an instance of the `Post` model
                        
                            ```php
                            public function store(Post $post) { 
                                $post->addComment(request('body')); 
                            }
                            ```
                        * Now the `Post` model will actually handle adding the comment to the database. The `store` method, in the `CommentsController` will call an `addComment` method in the `Post` model and pass it any of the request data needed. In this case it would be just the `body` field from the form. The `addComment` method now just needs to call it's own `comments` method followed by the `create` method with the body being passed as a parameter. All the relationship info is handled by Laravel.
                            ```php
                            public function addComment($body) {
                                $this->comments()->create([
                                    'body' => $body
                                ]);
                            }
                            ```
            * Always eager load certain relationships within a model.
                * If there is a time when you need to always eager load a relationship (using a method within the model), then you can add a `protected $with` property to the model. i.e. Within the `Post` model: `protected $with = ['comments'];` will load the comments relationship via the `comments` method within the `Post` model. This way, whenever a post is retrieved then the comments will be available without having to run another sql query.
        * When referencing a relationship in a model, there are 2 ways to do it. 
            * Dynamic property `$comments = $post->comments`
                * When you use a dynamic property, a collection is performed and the sql statement is executed.
            * As a method `$comments = $post->comments()->latest()-first()` 
                * When using the method, you are given an instance of the query builder (sql statement not executed yet), where you can continue to add other conditions on to the query.
    * **Relationship Notes:**
        * When using a relationship within a model, you can pass a second argument to the relationship method that tells Eloquent to use a different primary key to use instead of the default one.
            * By default, Eloquent will calculate the primary key name based off of the model name and the method name being used to call the relationship.
                * `post_id` when dealing with the `Post` model, `user_id` when dealing with the `User` model, etc
                * If you wanted to call the `post` to `user` relationship, `creator` instead of `user`, you'd need to tell Eloquent to use `user_id` instead of `creator_id`
                    * `public function creator() { return $this->belongsTo(User::class, 'user_id'); }`
            * If you pass a the second argument to the relationship method, then you can change the default behavior. For example: In the `User` model you add a method called `posts` to gather all the `posts` for that user. However, in the `Posts` table, you use `owner_id` instead of `user_id`. You'd need to create the relationship in the following way.
                * User model: `public function posts() { return $this->hasMany(Post::class, 'owner_id'); }`
        * It is possible to update the `updated_at` timestamp for a parent model of a `belongsTo` or `belongsToMany` relationship.
            * On the child model you need to add a `touches` property, which is an array of the relationships you want to update timestamps for whenever the child model is updated.
                * For example: You have a Task Manager app, and you want to update the parent `Project` model whenever the `Task` model is updated
                    * In the `Task` model, add `protected $touches = ['project'];`
                    * This will reference the `project` method also in the `Task` model
                        * `public function project() { return $this->belongsTo(Project::class); }`
                    * Now, whenever the `Task` is updated, the parent model (`Project`) will also have the `updated_at` timestamp updated.
        * You can add a sorting method to a relationship method. Useful if you always want a relationship gathered in a certain order.
            ```php
            public function project() { 
                return $this->belongsTo(Project::class)->latest('updated_at'); 
            }
            public function project() { 
                return $this->belongsTo(Project::class)->orderBy('updated_at', 'desc'); 
            }
            public function project() { 
                return $this->belongsTo(Project::class)->orderByDesc('updated_at'); 
            }
            ```
    * **DB query logging:** Useful when using Tinker. **NOTE: SHOULD ONLY BE DONE FOR TESTING PURPOSES AND NOT IN PRODUCTION**
        * `DB::enableQueryLog();` Enables logging
        * `DB::getQueryLog();` Shows the current query log
        * `DB::listen(function ($sql) { var_dump($sql->sql, $sql->bindings); });` Will dump out the sql statement along with any bindings with each query.
        * You can also use Laravel [Telescope](https://github.com/laravel/telescope). See section on Laravel Telescope.
    * **Other Creation Methods:**
        * `firstOrCreate` - Tries to find a record in the database matching the provided arguments and if not found then it will create it using the arguments passed. You can also pass a second argument that can be used for other attributes you would want used in the create but not in there where clause for the find. An instance of the model will be returned using this method.
            ```php
            // Retrieve flight by name, or create it if it doesn't exist...
            $flight = App\Flight::firstOrCreate(['name' => 'Flight 10']);

            // Retrieve flight by name, or create it with the name and delayed attributes...
            $flight = App\Flight::firstOrCreate(['name' => 'Flight 10'], ['delayed' => 1]); 
            ```
        * `firstOrNew` - The exact same as `firstOrCreate` except the model is not persisted to the database if a record isn't found.  An instance of the model will be returned using this method. You will still need to run `->save()` on the model to persist it to the database.
        * `updateOrCreate` - Can be used to update a record in the database or create a new record if the provided arguments returns no results. This method will persist the model to the database if it needs to create a new model.
            ```php
            // If there's a flight from Oakland to San Diego, set the price to $99. 
            // If no matching model exists, create one.
            $flight = App\Flight::updateOrCreate(
                ['departure' => 'Oakland', 'destination' => 'San Diego'], ['price' => 99]
            ); 
            ```
    * **Deleting Models:**
        * Delete the model directly on an instance of the model.
            ```php
            $flight = App\Flight::find(1); $flight->delete();
            ```
        * Delete an existing model using the primary key, a collection of keys or an array of keys.
            ```php
            App\Flight::destroy(1);
            App\Flight::destroy(1, 2, 3);
            App\Flight::destroy([1, 2, 3]);
            App\Flight::destroy(collect([1, 2, 3]));
            ```
        * Delete a model by query
            ```php
            $deletedRows = App\Flight::where('active', 0)->delete();
            ```
        * When issuing mass delete statements, it's important to note that the `deleting` and `deleted` events will not file since the model isn't created first.
    * **Miscellaneous Notes:**
        * Within a model you can use the `protected $casts` property to tell laravel how certain fields should be handled. i.e. `'confirmed' => 'boolean'`
            * Allowed cast types: `integer, real, float, double, string, boolean, object, array,  collection, date, datetime, and timestamp`
        * By default, pivot tables do not record timestamps when the record is created through a relationship. If you want timestamps you need to use the `withTimestamps()` method on the relationship.
            ```php
            public function tags() { return $this->belongsToMany(Tag::class)->withTimestamps(); }
            ```