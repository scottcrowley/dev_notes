# Laravel Notes - Mail


## Notes from the Laracast series [Laravel 6 from Scratch](https://laracasts.com/series/laravel-6-from-scratch/episodes/44) Section 10: Mail
* ### General Notes
    * [mailtrap.io](https://mailtrap.io/) is a good mailing sandbox
    * All mail server config details are updated in the .env file
    * Also a good idea to add MAIL_FROM_ADDRESS to the .env file to provide a default from email address
    * The mail facade needs to be imported whenever using the `Mail::class`. `Illuminate\Support\Facades\Mail`
* ### Sending Generic Text Emails
    * The Mail::raw method is used to send raw text emails
        * This method accepts the message as the first argument (string) and a closure can be passed as the second argument to define any other message options
            ```php
            Mail::raw('It works!', function ($message) {
                $message->to(request('email'))
                    ->subject('Hello There!');
            });
            ```
* ### Sending HTML Emails Using Blade Templates
    * There is a Artisan command to add a new mailable class
        ```zsh
        php artisan make:mail ContactMe
        ```
        This will create a new `ContactMe` class within `\app\Mail`
    * There will be a `build` method in the `ContactMe` class, which is returning a view. You need to create a new blade template containing all the formatting for the mail you want to use. Create a new file at `\resources\views\emails\contact-me.blade.php` and then update the view name within the `build` method.
        ```php
        return $this->view('emails.contact-me');
        ```
        * The template should contain all the html just as if it were a normal html 5 page
    * You can then use the Mail facade to assign on the necessary options. The `send` method is used to declare which mailable class you want to use.
        ```php
        Mail::to(request('email'))
            ->send(new ContactMe());
        ```
    * Any ***public properties*** that in the Mailable class (ContactMe), will be available in the view specified in the `build` method. ***private*** and ***protected*** properties will not be available.
        * For example: you want to pass in a topic to the mailable.
            ```php
            //ContactMe __construct method
            public $topic;

            public function __construct(string $topic)
            {
                $this->topic = $topic;
            }
            ```
            Now `$topic` will be available in the `contact-me` view automatically. Now when to instantiate the mailable, just pass in the topic.

            ```php
            Mail::to(request('email'))
                ->send(new ContactMe('shirts'));
            ```
    * When returning the view in the build method of the Mailable class, there are other options you can chain. For example: the subject can be declared here along with other things like attachments, bcc, cc, queuing the delivery or deferring it.
        ```php
        return $this->view('emails.contact-me')
            ->subject('More information about the ' . $this->topic );
        ```
* ### Sending Emails using Markdown
    * There is a Artisan command to add a new mailable class using a markdown template
        ```zsh
        php artisan make:mail Contact --markdown=emails.contact
        ```
        This will create a new `Contact` class within `\app\Mail` and a `contact.blade.php` file in `\resources\views\emails\`
    * In the `build` method within the `Contact` mailable, you would use the `markdown` method instead of the `view` method. When you create the markdown template using the above artisan command, the markdown method will be used automatically.
        ```php
        return $this->markdown('emails.contact');
        ```
    * From here, the usage is the same as sending an html email. See section below for details on the markdown template syntax
* ### Customizing the styling used in template components.
    * If you want to apply your own styling to a component used in a markdown template, you first need to publish the assets being used by the mailable class.
        ```zsh
        php artisan vendor:publish --tag=laravel-mail
        ```
        This will place all the assets in your `resources/views/vendor/mail` directory
    * If you want to create a new theme to use, it is suggested that you copy the `default.css` file located in `resources/views/vendor/mail/html/themes/` and call it whatever you want an save it to that directory. 
        * You will also need to tell Laravel you want to use this new file instead of the default one. You can update the `theme` value in the `markdown` key within the `config/mail.php` file to do this.
        * Also note that since you published the assets, the this config file should reflect the new path being used for the components in the `path` value of the `markdown` key. It should be `views/vendor/mail`.
* ### Markdown Template Syntax
    * See the [documentation](https://laravel.com/docs/master/mail#writing-markdown-messages) for full details.
    * It is very import that you DO NOT INDENT with the markdown files. Doing so will screw up the parser at runtime.
    * The markdown template consists of components that are declared using the `@component` blade directive. Everything should reside within the mail::message component
        ```
        @component('mail::message')

        @endcomponent
        ```
    * Everything within a component should be standard md format
    * Available components are `button`, `panel` & `table`
    