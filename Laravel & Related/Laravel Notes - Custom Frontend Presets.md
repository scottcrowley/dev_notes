# Laravel Notes - Custom Frontend Presets


## Notes from the Laracast series about Laravel
* ### Creating Custom Frontend Presets
    * A good package that can be used to install `Bulma` css framework and update the necessary views with the correct classes. <https://github.com/laravel-frontend-presets/bulma>
    * There is a Tailwind Laravel Preset available at <https://github.com/laravel-frontend-presets/tailwindcss> that will automatically install Tailwind and change the default views to use Tailwind classes.
    * Taken from the [How to Create Custom Laravel Presets](https://laracasts.com/series/how-to-create-custom-presets)
    * Out of the box, Laravel comes with 4 presets that can be used to scaffold the frontend. [`none`, `bootstrap`, `vue`, `react`]
        * These are accessed by running the command `php artisan preset --presetName`
    * We are going to make a Laracasts preset in the example
        * Create a `Laracast` directory in the root and add a `src` directory within it.
        * Create `Laracast/src/stubs` directory
            * Create `Laracasts/src/stubs/webpack.mix.js`

                ```js
                let mix = require('laravel-mix');
                require('laravel-mix-tailwind');
                mix
                    .js('resources/assets/js/app.js', 'js')
                    .sass('resources/assets/css/sass/app.scss', 'css')
                    .tailwind();
                ```
                * Create `Laracasts/src/stubs/app.js`
                    ```js
                    require('./bootstrap');
                    ```
                * Create `Laracasts/src/stubs/bootstrap.js`
                    ```js
                    window.axios = require('axios');
                    window.axios.defaults.header.common['X-Requested-With'] = 'XMLHttpRequest';
                    let token = document.head.querySelector('meta[name=“csrf-token”]');
                    if (token) { 
                        window.axios.defaults.header.common['X-CSRF-TOKEN'] = token.content; 
                    }
                    else { 
                        console.error('CSRF Token not found: https://laravel.com/docs/csrf#csrf-x-csrf-token'); 
                    }
                    ```
        * Run `php artisan make:provider LaracastsServiceProvider`
            * This will make a new provider within `app/providers`. Move the file to `Laracasts/src`
            * Update the `namespace` in the file to `Laracasts` instead of `App\Providers`
            * Open `config/app.php` 
                * in the providers array under the `Package Service Providers` comment block add `Laracasts\LaracastsServiceProvider::class,`
            * Open `composer.json`
                * go to `“autoload”: “psr-4”` and add a new root namespace `“Laracasts\\”: “Laracasts/src/“`
                    * Now when you namespace a file with `Laracasts`, the app will know to look in `Laracasts/src` for the file.
        * Open the `LaracastsServiceProvider` service provider
            * The new command for the preset can be added to the `boot` method.
                add 
                ```php
                PresetCommand::macro('Laracasts', function($command) { 
                    Preset::install(); 
                    $command->info('Laracast Preset applied successfully! Please recompile your assets.'); 
                });
                ```
                make sure to import `PresetCommand` from `Illuminate/Foundation/Console/PresetCommand`
        * run `composer dump-autoload`
            * Now you should be able to use the command `php artisan preset Laracasts`
        * Create a `Laracasts/src/Presets.php` file
            
            ```php
            namespace Laracasts;
            use Illuminate\Support\Arr;
            use Illuminate\Foundation\Console\Presets\Preset as LaravelPreset;
            use Illuminate\Support\Facades\File;
            
            class Preset extends LaravelPreset
            {
                public static function install() {
                    static::cleanSassDirectory();

                    //call method on LaravelPreset, which will call updatePackageArray, 
                    //which was overridden in this file
                    static::updatePackages(); 

                    static::updateMix();
                    static::updateScripts();
                    static::updateStyles();
                }
                public static function cleanSassDirectory() {
                    File::cleanDirectory(resource_path('assets/sass'));
                }
                //is passed all the base packages from updatePackages
                public static function updatePackageArray($packages) { 
                    return array_merge([
                        'laravel-mix-tailwind' => '^0.1.0'], 
                        Arr::except($packages, [ //method allows you to remove certain keys from an array
                        'popper.js', 'lodash', 'jquery'
                    ]));
                }
                public static function updateMix() {
                    //copies the stub to the root path of the project
                    copy(__DIR__.'stubs/webpack.mix.js', base_path('webpack.mix.js'));
                }
                public static function updateScripts() {
                    copy(__DIR__.'stubs/app.js', resource_path('assets/js/app.js'));
                    copy(__DIR__.'stubs/bootstrap.js', resource_path('assets/js/bootstrap.js'));
                }
                public static function updateScripts() {
                    //creates a blank sass file
                    File::put(resource_path('assets/css/sass/app.scss'), '');
                }
            }
            ```
        * To extract the above to an independent project that can be pulled in by anyone, see the Laracast video, from the series, called [Distribute a Composer Package](https://laracasts.com/series/how-to-create-custom-presets/episodes/4)
