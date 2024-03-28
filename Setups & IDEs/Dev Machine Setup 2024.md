# Dev Machine Initial Setup 2024
### Notes on [Setting Up a Mac for Laravel in 2024](https://laracasts.com/series/lukes-larabits/episodes/14) from Laracasts

## Laravel Herd
* PHP, Laravel & Composer along with web serving tools can be installed using a Laravel Mac app called [Herd](https://herd.laravel.com).
#### Installation
* Download the image from https://herd.laravel.com and drag the application to the `Applications` folder on your drive.
* Open `Herd` and follow the on-screen prompts. It will install a default version of `PHP`
* Make sure to `Automatically launch Herd on system startup`

## Node & NPM
* Go to the [Node](https://nodejs.org) homepage and download the either recommended stable version or the most recent version listed on their site. 
* Install the package and leave all the setting at default.

## Terminal App - Warp.dev
* Luke recommends an app called [Warp.dev](https://warp.dev) to use as his default terminal app. I've been pretty happy with `iTerm` but may be worth checking out.
* One downside with the app is that you need to sign up for an account. It doesn't cost anything but it is required.

## Verify all installations
* Use the `-v` flag to check to make sure everything is installed in the terminal.
    ```bash
    php -v
    composer -v
    laravel -v
    node -v
    npm -v
    ```
## Databases
* Should use an app called `DBngin`, which is available at https://dbngin.com. It supports most of the popular databases like MySQL, redis, sqllite, etc
* Download the app and drag it to the `Applications` folder
* Open `DBngin` then select `Launch at Login` at the top of the window shown along with `Open in menu bar`.
* Click the `+` button in the toolbar to add a new service.
* In the `Services` pull down select `MySQL` and type in `MySQL` in the `Name` field.
* Make sure to check the box for `Automatically start service on login`.
* **NOTE** By default `DBngin` uses `root` as the user with `no password`, which is the default configuration for a new `Laravel` project.
* After you click on `Create` you will see a button to `Start` the service.
#### MySQL Terminal plugin
* If you need to use `MySQL` within the terminal then you will need to install the CLI version manually
* You will need to install the package manager called [Homebrew](https://brew.sh).
* You can either install it via the terminal or download the package from [GitHub](https://github.com/Homebrew/brew/releases/tag/4.2.15), which is what Luke does. At the bottom of the page in the `Assets` section you should see `Homebrew-{version}.pkg`. Click that to download it.
* Install the downloaded package and follow the screen promopts
* To make it so you can use the `brew` command instead of `/opt/homebrew/bin/brew` you will need to add the path to your `~/.zshrc` file.
* When you open this file in your IDE, you will see that `Herd` has already added some path info to it. Underneath the `Herd` path, add a new line with `export PATH="/opt/homebrew/bin/":$PATH` so your system will see the binaries in this directory.
* Install the CLI MySQL tool using this command: `brew install mysql-client`
* When it is installed you need to add a new path to your `.zshrc` file. `export PATH="/opt/homebrew/opt/mysql-client/bin/":$PATH`

## IDE Install
* Luke uses `PHPStorm` but this requires a subscription model and tends to be pretty expensive. There is a free version available but its like a beta version and may be unstable. It's called their `EAP` or `Early Access Program` and is available at https://www.jetbrains.com/phpstorm/nextversion/

## Aliases in .zshrc file
* Luke uses the following aliases that are declared in the `~/.zshrc` file.
    ```bash
    # Allows you to open the .zshrc file in VSCode from anywhere
    alias aliases="code ~/.zshrc"
    # Clears the terminal window
    alias c="clear"
    # Opens the host file
    alias hosts="sudo /etc/hosts"
    # Laravel php artisan shortcut
    alias a="php artisan"
    # Reverts all work done in a Git respository losing all current progress if something isn't working
    nope() {
        git reset --hard
        git clean -f -d
        git checkout HEAD
    }
    ```
