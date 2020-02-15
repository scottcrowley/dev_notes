# Testing using Cypress

#### Notes from the Laracast episode [Learn Cypress with Laravel Integration](https://laracasts.com/series/whatcha-working-on/episodes/39)

## Installation
```zsh
npm install cypress --save-dev
```
Once it is installed, you need to execute the cypress binary to finish the setup.

```zsh
node_modules/.bin/cypress open
```
This should open the Cypress GUI and create a cypress directory in the root of your site along with a cypress.json config file.
## Setup
There is some setup that should be done to simplify running tests
* Copy the current `.env` file to a new file called `.env.cypress` or `.env.acceptance`
* Open the new environment file and modify the database connection details to the following
    ```
    DB_CONNECTION=sqlite
    DB_DATABASE=cypress.sqlite
    ```
    * You may need to update the `database` key in the `sqlite` driver details in the `config/database.php` config file
        ```php
        'sqlite' => [
            'driver' => 'sqlite',
            'database' => database_path(env('DB_DATABASE', 'database.sqlite')),
            'prefix' => '',
            'foreign_key_constraints' => env('DB_FOREIGN_KEYS', true),
        ],
        ```
        This just adds the `database_path` to all values.
        
        **IMPORTANT ISSUE: Changing the database key from `env('DB_DATABASE', database_path('database.sqlite')),` to `database_path(env('DB_DATABASE', 'database.sqlite')),` makes all the phpunit test, that are using the `sqlite` driver with a `:memory:` database, no longer work,**
* Update the `APP_ENV` key in the `.env.cypress` file
    ```
    APP_ENV=testing
    ```
* Create a new database file in `database/cypress.sqlite` or whatever name you used as a `DB_DATABASE` value in the `.env.cypress` file.
* Migrate the cypress database using the `.env.cypress` file
    ```zsh
    php artisan migrate --env=cypress
    ```
* Add a new link or sub domain for just testing with Cypress. This will be the domain name that Cypress will use to run all the tests. If you are using Laravel Valet then you can do the following. 
    ```zsh
    valet link sitename-cypress
    ```
* Add a check in the `bootstrap/app.php` file to see if the host is the url that was used in the last step. If it is then we need to switch to the `.env.cypress` environment file. Code should go before the `$app` is returned.
    ```php
    if (isset($_SERVER['HTTP_HOST']) && $_SERVER['HTTP_HOST'] === 'sitename-cypress.test') {
        \Dotenv\Dotenv::create(base_path(), '.env.cypress')->overload();
    }
    ```
* Add a `baseUrl` to the `cypress.json` file in the root of the site. The value need to be set to the name used when adding the Valet link or sub domain name.
    ```json
    {
        "baseUrl": "sitename-cypress",
    }
    ```
    Once you save this file, the Cypress GUI should close out. You will need to restart it again to continue using Cypress.
* Boot up the Cypress GUI
    ```zsh
    node_modules/.bin/cypress open
    ```
* Add a command to clear out the database before running any test. This command can be added to the c`ypress/support/index.js` file
    ```js
    beforeEach(() => {
        cy.exec('php artisan migrate:fresh --env=cypress');
    });
    ```
    The `--env` flag is very important or the command will be run in the `local` environment not the `testing` one.
## Helpers
* ### Add a Model Factory Endpoint
    * Add an endpoint in the `routes/web.php` file that will accept a model name along with any attributes you want updated, and return that model created using a factory.
        ```php
        if (App::environment() === 'testing') {
            Route::get('__testing__/create/{model}', function ($model) {
                return factory("App\\{$model}")->create(request()->all());
            });
        }
        ```
    * The model can be retrieved from Cypress by using the following
        ```js
        cy.request('/__testing__/create/User', { name: 'John Doe' }).its('body')
            .then(user => {
                cy.log('The created user is', user);
            });
        ```
        This will log the results of the factory to the console in the browser when Cypress runs the test.
* ### Create a Command to use the Model Factory Endpoint
    * Add a command to the cypress/support/commands.js file
        ```js
        Cypress.Commands.add('create', (model, overrides = {}) => {
            return cy.request(`/__testing__/create/${model}`, overrides).its('body')
        });
        ```
    * The command can now be used in a test like this
        ```js
        cy.create('User', { name: 'John Doe' });
        ```
* ### Add a User Login Endpoint
    * Add an endpoint in the `routes/web.php` file that will create a user and log them in. This will allow for you to provide overrides just like the Model Factory above.
        ```php
        if (App::environment() === 'testing') {
            Route::get('__testing__/login', function () {
                $user = factory('App\User')->create(request()->all());

                auth()->login($user);
            });
        }
        ```
    * A user can be logged in within a test by 
        ```js
        cy.request('/__testing__/login', { name: 'John Doe' });
        ```
* ### Create a Command to use the User Login Endpoint
    * Add a command to the cypress/support/commands.js file
        ```js
        Cypress.Commands.add('login', (overrides = {}) => {
            return cy.request('/__testing__/login', overrides);
        });
        ```
    * The command can now be used in a test like this
        ```js
        cy.login();
        cy.login({ name: 'John Doe' });
        ```
