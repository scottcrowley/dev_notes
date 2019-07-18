# Visual Studio Code Shortcuts


## Notes on shortcuts for Microsoft Visual Studio Code:

* ### Useful Keyboard Shortcuts
    * #### File Related
        * `cmd+alt+n` - uses `Advanced New File` extension. Allows you to add a new file at a specified location
        * `shift+cmd+F` - find in all files
        * `cmd+R` - go to a symbol in a file ``CUSTOM BINDING``
        * `shift+cmd+R` - go to a symbol in the workspace and shows all symbols in the current file `CUSTOM BINDING`
        * `cmd+;` - clears the terminal `CUSTOM BINDING`
        * `ctrl+s` - triggers suggestions for a class. Used to help import classes. `CUSTOM BINDING`
        * `cmd+/` - comment out selection
        * `shift+alt+D` - Creates a doc block for a method or property when then are selected. Requires `Php DocBlock Generator` `CUSTOM BINDING`
        * `shift+alt+F` - Runs `php-cs-fixer` on the current file. Requires `php cs fixer` `CUSTOM BINDING`
    * #### U/I:
        * `cmd+shift+P` - `Control Palette` - search for things that have to do with VSC like shortcuts, snippets, other installed extensions
        * `cmd+P` - search for things within a site. Uses fuzzy logic so `r/we` will show results in `routes/web.php`
        * `fn+shift+f10` - Opens context menu instead of having to right-click on something to bring it up.
        * `shift+cmd+E` - toggles file tree side bar
        * `ctrl+shift+G` - toggles source control panel
        * ctrl+` - show/hide terminal
        * `cmd+B` - show/hide sidebar
        * `shift+cmd+X` - show/hide extensions
        * `cmd+.` - brings up Quick Fix context menu when diagnosing an error in a file.
    * #### Testing:
        * `cmd+T` - runs a selected test if you have clicked within a test or test class and have ``Better PHPUnit`` installed `CUSTOM BINDING`
        * `shift+cmd+T` - runs a previously run test. Requires `Better PHPUnit` installed `CUSTOM BINDING`
        * `alt+T` - runs the entire PHPUnit suite. Requires `Better PHPUnit` `CUSTOM BINDING`
    * #### Cursor & Selections:
        * ***`ctrl+-`*** - goes back to where the cursor previously was in the current file or previous edit point
        * `shift+alt+down` duplicates the current line down.
        * `alt+down` - moves a selection down
        * `alt+up` - moves a selection up
        * `ctrl+cmd+g` - selects current word or variable name `CUSTOM BINDING`
        * `cmd+ctrl+shift+->` - Smart Select Grow `smartSelect.grow` This will select an entire variable, hit it again and it will grow the selection to include different results depending on what is on the line. It may just select a line or what is between brackets or quotes.
        * `cmd+ctrl+shift+<-` - Smart Select Shrink `smartSelect.shrink` This will shrink the selection. The inverse to Smart Select Grow
        * `ctrl+m` - select the enclosing bracket or parenthesis `CUSTOM BINDING`
    * #### Quick and Simple Text Selection Extension:
        * `cmd+k+"` - selects between "
        * `cmd+k+’` - selects between '
        * `cmd+k+;` - smart selects between " or '
        * `cmd+k+:` - toggles word quotes. e.g. "word" to 'word’
        * `cmd+k+`` - selects between `
        * `cmd+k+(` - inner selects between ()
        * `cmd+k+)` - outer selects between ()
        * `cmd+k+[` - inner selects between []
        * `cmd+k+]` - outer selects between []
        * `cmd+k+{` - inner selects between {}
        * `cmd+k+}` - outer selects between {}
        * `cmd+k+<` - selects matching tag
        * `cmd+k+>` - selects tag value
    * #### Deleting:
        * `cmd+shift+k` - deletes the entire line of code
        * `ctrl+k` - deletes everything right of the cursor.
        * `ctrl+shift+k` - deletes everything left of the cursor.
        * `alt+backspace` - deletes the entire word left of the cursor.
        * `ctrl+alt+backspace` - deletes the partial word left of the cursor. If the word is camel case then it will delete each hump separately
        * `alt+\` - deletes the entire word right of the cursor. `CUSTOM BINDING`
        * `ctrl+alt+\` - deletes the partial word right of the cursor. If the word is camel case then it will delete each hump separately `CUSTOM BINDING`
* ### Cosmetic Alterations:
    * Hide the status bar: `cmd+,` then find `workbench.statusBar.visible` and set to `false`
    * Hide the Open Editors on the left side bar: `cmd+,` then find `explorer.openEditors.visible` and set to `0`
    * Hide mini map: set `editor.minimap.enabled` to `false` in `User Settings`
    * `editor.tabCompletion` should be set to `true` if you want to be able to use the tab key to auto complete snippets and other things
* ### Miscellaneous Settings:
    * By default, whenever a new file is opened, within a project, Code will replace the file that is currently open, unless it has been previously edited. To have it open all files in a new window change the `Open Files in New Window` setting. 
    
        ```
        "window.openFilesInNewWindow": "on",
        ```
    * IMPORTANT! By default, Code will not search the `vendor` directory when using the `command palette` unless you set `search.useIgnoreFiles` to `false`
    * If you are going to use a `PostCSS` framework like `Tailwind CSS`,
        * Install the `sylelint` extension by `Shinnosuke Watanabe`
        * Add to `user settings`
            ```json
            "css.validate": false,
            "less.validate": false,
            "scss.validate": false
            ```
    * To change how many lines are shown in the terminal scroll history, set the following in the `settings.json` file: `"terminal.integrated.scrollback": 5000,` The `5000` is a suggestion only.
    * To make it so VSC ignore the native full screen mode used in Mac OS, use: `"window.nativeFullScreen": false,`
* ### Useful Extensions:
    * `advanced-new-file` - *patbenatar*
    * `File Utils` - *Leistner* Use command palette cmd+shift+p to access file utils like rename, delete, duplicate, etc
    * `PHP Intelephense` - *Ben Mewburn*
    * `Vim` - *vscodevim*
    * `snippet-creator` - *nikitaKunevich*
    * `Laravel Artisan` - *Ryan Naddy*
    * `Better PHPUnit` - *calebporzio*
    * `php cs fixer` - *junstyle*
    * `Vetur` - *Pine Wu*
    * `Import Cost` - *Wix*
    * `ESLint` - *Dirk Baeumer*
    * `Laravel Blade Snippets` - *Winnie Lin*
    * `Php DocBlock Generator` - *vincentkos* ***FOUND ON MY OWN***
    * `Quick and Simple Text Selection` - *David Bankier* ***FOUND ON MY OWN***. CURRENTLY HAVING TROUBLE MAKING THIS RESPOND TO THE DEFAULT SHORTCUTS
    * `stylelint` - *Shinnosuke Watanabe* ***FOUND ON MY OWN***. Use this if using tailwind.css. See note in the Miscellaneous Settings
    * `Tailwind CSS IntelliSense` - *Brad Cornes* ***FOUND ON MY OWN***. Use this if using tailwind.css. Autocomplete for tailwind classes
    * `Tailwind sass syntax` - *Maciej Ładoś* ***FOUND ON MY OWN***. Use this if using tailwind.css. Syntax highlighting for tailwind directives and classes.
    * `Vue VSCode Snippets` - *sarah.drasner* ***FOUND ON MY OWN***. Helpful snippets when using vue.
        * `vmethod` - sets up the methods object within the vue instance
        * `vbase` - sets up the document from scratch
        * `vcreated` - sets up the created method
        * many more available
    * `PHP Extension Pack` - *Felix Becker* Must have PHP extension pack
    * `Auto Rename Tag` - *Jun Han* ***FOUND ON MY OWN*** Renames paired HTML tags
    * `Auto-Open Markdown Preview` - *hnw* ***FOUND ON MY OWN*** Automatically opens side panel with Markdown preview
    * `IntelliSense for CSS class names in HTML` - *Zignd* ***FOUND ON MY OWN*** CSS classname completion
    * `Sass` - *Robin Bentley* ***FOUND ON MY OWN*** Indented Sass syntax highlighting, autocomplete & snippets
    * `Markdown Preview Enhanced` - *Yiyi Wang* ***FOUND ON MY OWN*** Added markdown functionality
    * `Markdown Preview Github Styling` - *Matt Bierner* ***FOUND ON MY OWN*** Makes markdown previews look like Github
    * `Markdown Vue` - *donaldshen* ***FOUND ON MY OWN*** Formats vue code blocks in markdown files. Does not style the preview though
    * `GitLens` - *Eric Amodio* Adds additional Github functionality
* ### UI Extensions:
    * `Slime Theme` - *smlombardi*
    * `vscode-icons` - *VSCode Icons Team*
* ### Useful Tips:
    * To allow files to be opened in Code via the terminal: This will add Code to your Path
        * Open the Command Palette (`shift+cmd+p`) then type `shell command`
        * Restart the terminal to make the changes take effect
        * Now you can open files by using the `code` command
            ```zsh
            # opens the zshrc file in VSCode
            code ~/.zshrc

            # opens the current directory in VSCode
            code .
            ```
    * Hitting `cmd+P` will display all the recently opened files. Keep hitting `p` while holding `cmd` to cycle through the files
    * `ctrl+-` takes you to your previous edit point
    * Check out <https://code.visualstudio.com/docs/editor/userdefinedsnippets> for useful ideas on snippets
    * Select a type hinted class and type `ctrl+space` to bring up the import menu to change the current imported class path.
    * `Right-click` on a method will give the option to view all references of that method if `Intelephense` extension is installed
    * `Right-click` on a method/class/use level will give the option to `Peek` at a definition if `Intelephense` is installed. Allows to see the method/class/use file without opening the file containing that method
    * `Right-click` on a dependency being used in a method injection, select `Add Use Definition` will add the '`use`' declaration at the top of the file and condense the injection down to the class name. `Intelephense` is required
    * `cmd+click` on a classname or method will open that class or method if you have `Intelephense` extension installed.
    * With `Better PHPUnit` installed, you can click anywhere in a test method or class and then open `command palette`, type `unit` and several test options are available
    * Make sure that the code command is installed into `PATH`. Open `command palette` then type `shell` then `select install 'code’ command in PATH`. This allows you to use `code` to open something in VSC from the terminal
    * You can add `// @ts-check` to the top of a `.js` file to apply some `typescript` checking
    * If you are using `// @ts-check` and you want to ignore a certain error you can add `// @ts-ignore` before the error location
    * `// @ts-check` will also check doc blocks for information about a method. i.e. the doc block show that a method should receive a param but a method call doesn’t contain one, an error will be generated on the call. Within the doc block a param declaration looks like `@param {number} paramname`. If you want to param to be optional use an '`=`' after the param type. `{number=}`
    * Set up `XDebug`. See <https://tighten.co/blog/configure-vscode-to-debug-phpunit-tests-with-xdebug>
    * To have `Import Cost` extension see `.vue` files directly, you need to add `"importCost.typescriptExtensions": [ "\\.vue?$" ]` to the `User Settings` file
    * When using Laravel Blade Snippets, you can type '`b:`’, followed by the command (i.e. `extend`, `yield`, `content`, etc), to bring up a contextual window with blade snippets to use.
* ### Code Specific User File Locations
    * In the event that you need to install VSCode from scratch, below is where some of the common setup files are located.
        * `settings.json` - `Users/scott/Library/Application Support/Code/User/`
        * `keybinding.json` - `Users/scott/Library/Application Support/Code/User/`
        * `php.json` - `Users/scott/Library/Application Support/Code/User/snippets/`
        * `html.json` - `Users/scott/Library/Application Support/Code/User/snippets/`
        * `vue.json` - `Users/scott/Library/Application Support/Code/User/snippets/`
* ### Multiple Cursors
    * Using the mouse you can click some where and then hold `alt` while click elsewhere to create multiple cursors
    * `cmd+D` will select the next occurrence of what is selected.
    * `ctrl+cmd+G` will select all occurrences. 
* ### Configure ESLint
    * With `ESLint` installed make sure that `.vue` files are added to the `eslint.validate` property. Go into `User Settings` and search for `eslint.validate` and copy settings to `User Settings`. Then add `"vue", "vue-html"` to the existing array that has `"javascript", "javascriptreact"`
    * You will also need an .`eslintrc.json` file.
        * Add one by opening `command palette` and typing `eslint` and selecting `Create 'eslintrc.json’ File`. This file is added to your project root. So this will need to be done for every project being used.
        * or type `eslint --init` in the terminal and answer the questions asked.
        * See below for an example `.eslintrc.json` file.
    * GETTING AN ERROR TRYING TO RUN THE ABOVE COMMAND. MAY NEED TO INSTALL A DIFFERENT WAY Not an issue anymore for some reason
    * You will also need an `eslint-plugin-vue` plugin. Type `npm install -g eslint-plugin-vue` into the terminal to install it globally.
        * `"plugin:vue/recommended"` needs to be added to the `extends` key array in the `.eslintrc.json` file
        * Make sure if you have the `Vetur` extension installed that you set the `"vetur.validation.template":` setting to `false` in your `User Settings`.
* ### Configuring PHP CS FIXER
    * Make sure extension is installed
    * If a config file doesn’t already exist in `vscode` directory do so by:
        * `cd ~/.vscode`
        * `touch .php_cs`
        * `code .php_cs` (make sure that code is installed in PATH. see Useful Tips section)
        * Paste the example config file from the description on the `Extensions Panel` for `php cs fixer`
        * You may have to add a full path to the `user settings` for `php-cs-fixer.config` and set it to `/Users/scott/.vscode/.php_cs`
    * Modifications to the `config` file:
        * `'no_unused_imports' => true` may need to un-comment
        * `'array_syntax' => array('syntax' => 'short')`
        * `'single_quote' => true`
        * `'not_operator_with_successor_space’ => true`
        * `'ordered_imports' => array('sort_algorithm' => 'length')`
* ### Example `settings.json` file:
    ~/Library/Application Support/Code/User/settings.json
    ```json
    {
        "editor.tabCompletion": "on",
        "editor.minimap.enabled": false,
        "editor.fontFamily": "'Fira Code', Menlo, Monaco, 'Courier New', monospace",
        "editor.fontSize": 13,
        "editor.fontLigatures": true,
        "editor.tokenColorCustomizations": {
            "textMateRules": [
                {
                    "scope": "comment, comment.block.html",
                    "settings": {
                        "fontStyle": "italic"
                    }
                }
            ]
        },
        "explorer.openEditors.visible": 0,
        "explorer.confirmDelete": false,
        "window.openFilesInNewWindow": "on",
        "window.zoomLevel": 0,
        "workbench.statusBar.visible": false,
        "workbench.iconTheme": "vscode-icons",
        "workbench.colorTheme": "Slime",
        "terminal.integrated.scrollback": 5000,
        "blade.format.enable": true,
        "search.useIgnoreFiles": true,
        "search.exclude": {
            "**/public/[abcdefghjklmnopqrstuvwxyz]*": true,
            "**/public/i[abcdefghijklmopqrstuvwxyz]*": true,
            "storage/framework/views": true
        },
        "css.validate": false,
        "less.validate": false,
        "scss.validate": false,
        "php-cs-fixer.executablePath": "${extensionPath}/php-cs-fixer.phar",
        "php-cs-fixer.formatHtml": true,
        "php-cs-fixer.onsave": true,
        "php-cs-fixer.config": "/Users/scott/.vscode/.php_cs",
        "[php]": {
            "editor.defaultFormatter": "junstyle.php-cs-fixer"
        },
        // File extensions to be parsed by the Typescript parser
        "importCost.typescriptExtensions": [
            "\\.vue?$"
        ],
        "eslint.validate": [
            "javascript",
            "javascriptreact",
            "vue",
            "vue-html"
        ],
        "vetur.validation.template": false,
        "php-cs-fixer.lastDownload": 1563489562217,
    }
    ```
* ### Example `keybindings.json` file:
    ~/Library/Application Support/Code/User/keybindings.json
    ```json
    // Place your key bindings in this file to overwrite the defaults
    [
        {
            "key": "cmd+r",
            "command": "workbench.action.gotoSymbol"
        },
        {
            "key": "shift+cmd+o",
            "command": "-workbench.action.gotoSymbol"
        },
        {
            "key": "cmd+t",
            "command": "better-phpunit.run"
        },
        {
            "key": "shift+cmd+t",
            "command": "better-phpunit.run-previous"
        },
        {
            "key": "alt+t",
            "command": "better-phpunit.run-suite"
        },
        {
            "key": "shift+alt+f",
            "command": "php-cs-fixer.fix2"
        },
        {
            "key": "shift+alt+d",
            "command": "php-docblock-generator.createDocBlock"
        },
        {
            "key": "ctrl+s",
            "command": "editor.action.triggerSuggest",
            "when": "editorHasCompletionItemProvider && textInputFocus && !editorReadonly"
        },
        {
            "key": "ctrl+shift+k",
            "command": "deleteAllLeft",
            "when": "textInputFocus && !editorReadonly"
        },
        {
            "key": "cmd+backspace",
            "command": "-deleteAllLeft",
            "when": "textInputFocus && !editorReadonly"
        },
        {
            "key": "alt+\\",
            "command": "deleteWordRight",
            "when": "textInputFocus && !editorReadonly"
        },
        {
            "key": "alt+delete",
            "command": "-deleteWordRight",
            "when": "textInputFocus && !editorReadonly"
        },
        {
            "key": "ctrl+alt+\\",
            "command": "deleteWordPartRight",
            "when": "textInputFocus && !editorReadonly"
        },
        {
            "key": "ctrl+alt+delete",
            "command": "-deleteWordPartRight",
            "when": "textInputFocus && !editorReadonly"
        },
        {
            "key": "ctrl+cmd+g",
            "command": "editor.action.selectHighlights",
            "when": "editorFocus"
        },
        {
            "key": "shift+cmd+l",
            "command": "-editor.action.selectHighlights",
            "when": "editorFocus"
        },
        {
            "key": "ctrl+m",
            "command": "editor.action.selectToBracket"
        },
        {
            "key": "cmd+;",
            "command": "workbench.action.terminal.clear",
            "when": "terminalFocus"
        },
        {
            "key": "cmd+k",
            "command": "-workbench.action.terminal.clear",
            "when": "terminalFocus"
        },
    ]
    ```
* ### Example Snippets:
    ~/Library/Application Support/Code/User/snippets/php.json
    ```json
    {
        "Public Method": {
            "prefix": "met",
            "body": [
                "public function $1() ",
                "{",
                "\t$2",
                "}"
            ],
            "description": "New Public method"
        },
        "Protected Method": {
            "prefix": "pmet",
            "body": [
                "protected function $1() ",
                "{",
                "\t$2",
                "}"
            ],
            "description": "New Protected method"
        },
        "Class": {
            "prefix": "class",
            "body": [
                "namespace $1;",
                "",
                "class ${TM_FILENAME_BASE} ",
                "{",
                "\t$2",
                "}"
            ],
            "description": "New Class"
        },
        "Test Method": {
            "prefix": "tmet",
            "body": [
                "/** @test */",
                "public function $1()",
                "{",
                "\t$2",
                "}"
            ],
            "description": "Public Test Method"
        },
        "Class Constructor": {
            "prefix": "constr",
            "body": [
                "/**",
                "* ${TM_FILENAME_BASE} constructor",
                "*/",
                "public function __construct()",
                "{",
                "\t$1",
                "}"
            ],
            "description": "Blank Class Constructor"
        }
    }
    ```
    ~/Library/Application Support/Code/User/snippets/vue.json
    ```json
    {
        "Blank Component": {
            "prefix": "vcomp",
            "body": [
                "<template>",
                "    ",
                "</template>",
                "",
                "<script>",
                "    export default {",
                "        props: [],",
                "",
                "        components: {},",
                "",
                "        data() {",
                "            return {",
                "",
                "            }",
                "        },",
                "",
                "        computed: {",
                "",
                "        },",
                "",
                "        methods: {",
                "            ",
                "        }",
                "    }",
                "</script>"
            ],
            "description": "New Vue Component Scafolding"
        }
    }
    ```
* ### Example `ESLint` config file (`.eslintrc.json`):
    ```json
    {
        "env": {
            "browser": true,
            "commonjs": true,
            "es6": true,
            "node": true
        },
        "extends": [
            "eslint:recommended",
            "plugin:vue/recommended"
        ],
        "parserOptions": {
            "ecmaFeatures": {
                "jsx": true
            },
            "ecmaVersion": 2016,
            "sourceType": "module"
        },
        "rules": {
            "indent": [
                "error",
                4
            ],
            "linebreak-style": [
                "error",
                "unix"
            ],
            "quotes": [
                "error",
                "single"
            ],
            "semi": [
                "error",
                "always"
            ],
            "eqeqeq": [
                "error", 
                "always", 
                {
                    "null": "ignore"
                }
            ]
        }
    }
    ```