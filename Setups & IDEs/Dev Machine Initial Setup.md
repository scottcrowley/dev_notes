# Dev Machine Initial Setup

## Notes from the Setup a Mac Dev Machine from Scratch

* ### Suggested Tools
    * [Alfred](https://www.alfredapp.com/) Used for searching, shortcuts and hotkeys
    * [iTerm 2](https://www.iterm2.com/) Terminal replacement
        * **In the Preferences**
            * General: Uncheck Native full screen windows so that it uses full size windows without going into Full Screen mode which is slower when switching between different apps.
            * Keys: Show/hide iTerm2 with a system-wide hotkey. JW changed to a \` but I left it at `opt+space`
            * Profiles
                * Text:
                    * Check `Use Ligatures` (makes => or -> appear as one character). Doesn’t work with all fonts
                    * Click Change Font
                        * You may want to change the font to `Fira Code` (see font section below) or something similar
                        * Add a little vertical spacing
                        * Increase font size to about 13 or so
                * General:
                    * Under Working Directory, click on Reuse previous session’s directory
        * To remove the "`Last Login`" message that appears at the top of every new tab
            * `touch .hushlogin` in the terminal and in your home directory
    * [Oh My Zsh](https://github.com/robbyrussell/oh-my-zsh) to be used with `zsh` instead of `bash`
        * Make sure `zsh` is installed (should be by default). `zsh --version` should be at least 5.1.1
        * Verify that `zsh` is listed in `/etc/shells` by typing `nano /etc/shells` from the your home directory
        * Switch to the zsh shell by typing `chsh -s $(which zsh)`
        * You will need to log out and then back in for the new shell to take effect
        * Verify that zsh is being used:
            * `echo $SHELL` should show something like `/bin/zsh`
            * `$SHELL --version` should display the same result as `zsh --version` above
        * Download using curl: `sh -c "$(curl -fsSL https://raw.githubusercontent.com/robbyrussell/oh-my-zsh/master/tools/install.sh)"`
        * Edit the `.zshrc` file by using `nano ~/.zshrc` in the terminal
            * You can edit the default theme and which plugins to use from here.
            * Towards to the bottom of the doc there is a line that is commented out `# alias zshconfig="mate ~/.zshrc"`
                * Un-comment this line and change `mate` to a terminal text editor like `nano`
                    * `alias zshconfig="nano ~/.zshrc"`
                * After that you can access the config file by typing `zshconfig` in the terminal.
            * You will want to uncomment the `export PATH` line in the config
                * `export PATH=$HOME/bin:/usr/local/bin:$HOME/.composer/vendor/bin:$PATH`
                * After composer is installed, make sure that `$HOME/.composer/vendor/bin:` is added to the path as well, as it is above.
        * If you want to use aliases in the terminal:
            * In the `.zshrc` file
                * At the very bottom type `source ~/.aliases`
            * In your home directory type `nano .aliases` You’ll then be able to add a line for all new aliases.
                * `alias gs="git status"`
                * `alias gl="git log"`
    * Install new fonts
        * [Fira Code](https://github.com/tonsky/FiraCode)
            * Download the .zip and then navigate to the `distr/ttf` folder
            * Select all the fonts in this folder then right-click and select `open`.
            * Click on `Install Fonts`
        * [Source Code Pro from Adobe](https://github.com/adobe-fonts/source-code-pro)
            * Can install with `Brew` by using:
                * `brew tap caskroom/fonts && brew cask install font-source-code-pro`
    * Git
        * After first installing git, you’ll need to configure the global config
            * `git config --global --edit`
                * uncomment the `name` & `email` and provide the correct info
    * [Homebrew](https://brew.sh)
        * `/usr/bin/ruby -e "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/master/install)"`
        * You can use `brew search {package name}` to search for a specific package that can be installed via home-brew
    * To install PHP 7.2 using Homebrew
        ```
        brew update
        brew upgrade
        brew install php@7.2
        brew link --force php@7.2
        ```
        If you have `Composer` and `Oh My Zsh` installed, replace the `export PATH` line of of the `.zshrc` file with the following:
        ```
        # export PATH=$HOME/bin:/usr/local/bin:$PATH
        export PATH="$(brew --prefix homebrew/core/php@7.2)/bin:$HOME/.composer/vendor/bin:$PATH"
        ```
        You may need to resource the `.zshrc` file to have it reload the config
        ```
        source ~/.zshrc
        ```
        To actually have the service started and stay started after restarts, run the following
        ```
        brew services start php@7.2
        ```
    * [Node](https://nodejs.org/en/) Also will install `npm` at the same time
    * `Yarn` is sometimes faster than `npm` <https://yarnpkg.com/en/>
        * `brew install yarn`
        * or s`udo npm install -g yarn`
    * `MySQL` or `MariaDB` (drop in replacement for MySQL)
        * Both can be installed via Homebrew
            ```
            brew install mysql@5.7
            brew link --force mysql@5.7
            ```
            To actually have the service started and stay started after restarts, run the following
            ```
            brew services start mysql@5.7
            ```
    * [Composer](https://getcomposer.org/)
        * Once you’ve installed composer you need to move the `composer.phar` file to your bin directory
            * `mv /Users/scott/composer.phar /usr/local/bin/composer`
            * Now you can access composer with `composer` instead of `php composer.phar`
        * [Packagist.org](https://packagist.org/) is the place to go to find packages to install with composer
            * Key Packages:
                * `php-cs-fixer` A tool to automatically fix PHP code styles
                    * `composer global require friendsofphp/php-cs-fixer`
    * Laravel Valet
        * `composer global require laravel/valet`
        * `valet install`
        * Run `valet park` in the root of the code directory
    * Laravel Installer
        * `composer global require "laravel/installer"`
        * Now you can create a new Laravel project by typing `laravel new {project name}` within the code directory
    * SSH Keys & GitHub
        * You should generate new ssh keys in your root user directory to allow for more seamless workflow with GitHub
            * In GitHub go to Settings then click on `SSH & GPG Keys`
            * `ssh-keygen -t rsa -b 4096 -C "scott@scottcrowley.com"`
                * It will ask for the file to save the key. Use the default file so press `enter`
                * Enter a passphrase to secure the key
                * Make sure the SSH Agent is enabled by typing: `eval "$(ssh-agent -s)"`
                * Modify the `~/.ssh/config` file so keys can be automatically loaded and the passphrase can be stored in the keychain.
                    * `nano ~/.ssh/config`
                    * In the file
                        ```
                        Host *
                        AddKeysToAgent yes
                        UseKeychain yes
                        IdentityFile ~/.ssh/id_rsa
                        ```
                * Add the new key to the agent
                    * `ssh-add -K ~/.ssh/id_rsa`
                * Now you need to add the new key to GitHub
                    * Copy the key to the clipboard by typing
                    * `cat ~/.ssh/id_rsa.pub | pbcopy`
                    * Go to GitHub/Settings/SSH & GPG Keys and click on add New SSH Key
                    * Name it whatever the computer is you’re installing the key on.
                    * Paste the key in and save it.
    * Mac OS Quicklook Plugins:
        * [Quick look JSON](http://www.sagtau.com/quicklookjson.html)
            * `brew cask install quicklook-json`
        * [QLMarkdown](https://github.com/toland/qlmarkdown)
            * `brew cask install qlmarkdown`
            * If you want to be able to select text within the Quicklook window, you'll need to run the following.