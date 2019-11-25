# Laravel Notes - Models & Collections


## Notes from the Laracast series about Laravel
* ### Models
    * #### Dealing with Default Data
        * There may be times when you have a relationship declared in a model that may or may not always be there. Using the Lego Project as an example. There is a `UserSet` model that has a `inventory` relationship in it. Not all sets may have an inventory, so if you are trying to return any properties that are in the Inventory model and that set does not have an inventory, then you will get an error. This is because you are calling a property or method on `null`. To get around this, you can do a couple of different things.
            * Add the `withDefault` method to the relationship. Here, you can specify the default values to return when the relationship is null. We use the `getInventoryIdAttribute` method to return the `id` from the `inventory` relationship.
                ```php
                /**
                 * A UserSet has one inventory
                 *
                 * @return belongsTo
                */
                public function inventory()
                {
                    return $this->hasOne(Inventory::class, 'set_num', 'set_num')->withDefault(['id' => null]);
                }

                /**
                 * Attribute getter for inventory->id
                 *
                 * @return string
                */
                public function getInventoryIdAttribute()
                {
                    return $this->inventory->id;
                }
                ```
            * You can also use the optional helper method on the attribute getter instead of using the withDefault method on the relationship.
                ```php
                /**
                 * A UserSet has one inventory
                 *
                 * @return belongsTo
                */
                public function inventory()
                {
                    return $this->hasOne(Inventory::class, 'set_num', 'set_num');
                }

                /**
                 * Attribute getter for inventory->id
                 *
                 * @return string
                */
                public function getInventoryIdAttribute()
                {
                    return optional($this->inventory)->id;
                }
                ```
    * #### Miscellaneous
        * Different ways to handle Policy authorization within a vue component when you want the authorization details is available in the model's json response. This example is based off of the video called [Whatcha Working On:Frontend Authorization Brainstorming](https://laracasts.com/series/whatcha-working-on/episodes/36). JW shows ways to pass authorization details about a user working with a post. He only wants to show the edit button, if the user is an admin or the user who created the post and show a delete button if the user is an admin.

            *app/Policies/PostPolicy.php*
            ```php
            ...
            class PostPolicy
            {
                use handlesAuthorization;

                public function before(User $user)
                {
                    if ($user->id == 1) { //hard coding the admin user id to 1 for the example
                        return true;
                    }
                }

                public function destroy(User $user, Post $post)
                {
                    return false; //all other users are denied since the before method handles checking if the user is an admin
                }

                public function update(User $user, Post $post)
                {
                    return $user->id == $post->user_id;
                }
            }
            ```
            *app/Post*
            ```php
            ...

            class Post extends Model
            {
                public function authorizations($abilities = [])
                {
                    return tt(array_flip($abilities))->map(function ($index, $ability)) {
                        return \Gate::allows($ability,$this);
                    });
                }

                //override the toArray method so the authorization details can be added to model
                public function toArray()
                {
                    //make a call to the parent toArray method so you dont have to replicate all the fields here.
                    //just append the new authorizations property
                    return parent::toArray() + [
                        'authorizations' => $this->authorizations(['update', 'destroy']);
                    ];
                }
            }
            ```
* ### Model Events
    * The following events are fired at different points in the model lifecycle.
        * `retrieved` - fired when the model is retrieved from the database
        * `creating` and `created` - fire when the model is saved to the database for the first time and the `create` method is used.
        * `updating` and `updated` - fired when the model already exists in the database and the `save` method is used.
        * `saving` and `saved` - fired when either of the `update` or `create` events fired. 
        * `deleting` and `deleted` - fire when the model is deleted from the database.
        * `restoring` and `restored` - 
    * **IMPORTANT TO NOTE:** If a mass update is done via Eloquent, the `saved` and `updated` events are not fired since the models are not actually retrieved.
    * To catch events on a model. 
        * Use the `$dispathedEvents` protected property in the model.
            ```php
            /**
             * The event map for the model.
             *
             * @var array
            */
            protected $dispatchesEvents = [
                'saved' => UserSaved::class,
                'deleted' => UserDeleted::class,
            ];
            ```
        * Listeners then need to be used to actually catch the events. See docs on Listeners <https://laravel.com/docs/master/events#defining-listeners>
    * **Observers**
        * These can be used to listen for fired events on a model. All the events are group together in an `Observer` class. `php artisan make:observer UserObserver --model=User`
        * The observer needs to be registered in the `AppServiceProvider@boot` (JW says to use the EventServiceProvider. Not sure if it is now better to use AppServiceProvider because the [video](https://laracasts.com/series/code-reflections/episodes/1) is from 3/2018) using the `observe` method. `User::observe(UserObserver::class);`
            ```php
            public function boot()
            {
                parent::boot();

                \App\User::observe(UserObserver::class);
            }
            ```
            or if you have a lot of observers you can use the `$observers` property to register all observers for a certain model and then loop through them in the `boot` method. Note: all observer class have been imported in the example below
            ```php
            protected $observers = [
                \App\Thread::class => [
                    ThreadObserver::class,
                    ThreadReputaionObserver::class,
                    GenerateThreadSlugObserver::class,
                    DeleteThreadRepliesObserver::class
                ],
                \App\Reply::class => [
                    ReplyObserver::class
                ]
            ];

            public function boot()
            {
                parent::boot();

                foreach ($this->observers as $model => $observers) {
                    foreach($observers as $observer) {
                        $model::observe($observer);
                    }
                }
            }
            ```
* ### Collections
    * #### Misc Tips
        * You can return custom keys from the `groupBy` method, instead of using the numeric values returned by default.

            We want to group a collection of videos by ones created today, this week, last week & older.
            ```php
            $videos = Video::latest()->get()->groupBy(function($video) {
                if ($video->created_at->isToday()) {
                    return 'today';
                }

                if ($video->created_at->isCurrentWeek()) {
                    return 'this week';
                }

                if ($video->created_at->isLastWeek()) {
                    return 'last week';
                }

                return 'older';
            });
            ```
        * ***Dot Notation***
            * Some collection methods allow you to use a dot notation to access nested values.
            * Example: Say you have a collection and you want to access the the `city` value within the `location` array
                ```php
                $collection = collect([
                    [
                        'name' => 'John',
                        'location' => [
                            'city' => 'Scottsdale', 
                            'state' => 'Arizona'
                        ]
                    ]
                ]);
                ```
                You can the pluck the city by using:
                ```php
                $collection->pluck('location.city'); //Scottsdale
                ```
            * However, if you trying to get a value from a nested collection or array then you need to use a different syntax
                ```php
                $collection = collect([
                    [
                        'name' => 'John',
                        'locations' => [
                            [
                                'city' => 'Scottsdale', 
                                'state' => 'Arizona'
                            ],
                            [
                                'city' => 'Phoenix', 
                                'state' => 'Arizona'
                            ],
                        ]
                    ]
                ]);
                ```
                If you use the notation as in the previous example, you will get `null`

                You can the pluck the city out of a nested collection by using a wildcard (`.*`):
                ```php
                $collection->pluck('locations.*.city'); //[Scottsdale, Phoenix]
                ```
                The wildcard tells `pluck` to look in each item of `locations` and grab the `city` from it.
    * #### Custom Collections
        * Example covers a Laracasts episode from the [Laravel Explained](https://laracasts.com/series/laravel-explained) series called [Explain How to Group Records By Relative](https://laracasts.com/series/laravel-explained/episodes/6).
            * We want to group a collection of videos by ones created today, this week, last week & older. This can be done by adding a new `VideoCollection` collections class to handle it.
                ```php
                <?php

                namespace App;

                use Illuminate\Database\Eloquent\Collection;

                class VideoCollection extends Collection {
                    public function groupByDate() {
                        return $this->groupBy(function($video) {
                            if ($video->created_at->isToday()) {
                                return 'today';
                            }

                            if ($video->created_at->isCurrentWeek()) {
                                return 'this week';
                            }

                            if ($video->created_at->isLastWeek()) {
                                return 'last week';
                            }

                            return 'older';
                        });
                    }
                }
                ```
                Place where you need the query executed, which in this case is the `routes/web.php` file
                ```php
                Route::get('/', function() {
                    return Video::latest()->get()->groupByDate();
                });
                ```
                You need to override the default `newCollection` method from the `vendor/laravel/framework/src/DataBase/Eloquent/Model.php` class and return our new `VideoCollection` collection class instead of the normal `Collection` class. This is done in the model. In `App\Video.php`, add the following method.
                ```php
                /**
                 * Create a new Eloquent Collection instance.
                 *
                 * @param  array  $models
                 * @return \Illuminate\Database\Eloquent\Collection
                 */
                public function newCollection(array $models = [])
                {
                    return new VideoCollection($models);
                }
                ```
