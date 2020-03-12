# Visual Studio Code Shortcuts


## Notes on shortcuts for Microsoft Visual Studio Code:

* ### Useful Keyboard Shortcuts
    * #### File Related
        * `cmd+alt+n` - uses `Advanced New File` extension. Allows you to add a new file at a specified location
        * `shift+cmd+F` - find in all files
        * `cmd+R` - go to a symbol in a file ***CUSTOM BINDING***
        * `shift+cmd+R` - go to a symbol in the workspace ***CUSTOM BINDING***
        * `ctrl+s` - triggers suggestions for a class. Used to help import classes. ***CUSTOM BINDING***
        * `shift+alt+F` - Runs `php-cs-fixer` on the current file. Requires `php cs fixer` ***CUSTOM BINDING***
    * #### Editor:
        * `ctrl+tab` & `shift+ctrl+tab` - Move to next or previous open editor tab
        * `shift+cmd+T` - reopen closed editor
        * `cmd+/` - toggle line comment
        * `shift+alt+A` - toggle block comment
        * `cmd+;` - clears the terminal ***CUSTOM BINDING***
        * `shift+alt+D` - Creates a doc block for a method or property when then are selected. Requires `Php DocBlock Generator` ***CUSTOM BINDING***
        * `alt+z` - toggles word wrap
        * `ctrl+space` - trigger suggestion
        * `shift+cmd+space` - trigger parameter hints
        * `F12` - go to definition
        * `alt+F12` - peek definition
    * #### U/I:
        * `cmd+shift+P` - `Control Palette` - search for things that have to do with VSC like shortcuts, snippets, other installed extensions
        * `cmd+P` - search for things within a site. Uses fuzzy logic so `r/we` will show results in `routes/web.php`
        * `fn+shift+f10` - Opens context menu instead of having to right-click on something to bring it up.
        * `shift+cmd+E` - toggles file tree side bar
        * `ctrl+shift+G` - toggles source control panel
        * ctrl+` - show/hide terminal
        * shift+ctrl+` - create new terminal ***CUSTOM BINDING***
        * alt+` - focus terminal ***CUSTOM BINDING***
        * cmd+ctrl+` - focus next terminal ***CUSTOM BINDING***
        * alt+ctrl+` - focus previous terminal ***CUSTOM BINDING***
        * `cmd+B` - show/hide sidebar
        * `shift+cmd+X` - show/hide extensions
        * `cmd+.` - brings up Quick Fix context menu when diagnosing an error in a file.
    * #### Testing:
        * `cmd+T` - runs a selected test if you have clicked within a test or test class and have ``Better PHPUnit`` installed ***CUSTOM BINDING***
        * `shift+cmd+T` - runs a previously run test. Requires `Better PHPUnit` installed ***CUSTOM BINDING***
        * `alt+T` - runs the entire PHPUnit suite. Requires `Better PHPUnit` ***CUSTOM BINDING***
    * #### Cursor & Selections:
        * ***`ctrl+-`*** - goes back to where the cursor previously was in the current file or previous edit point
        * `shift+alt+down` duplicates the current line down.
        * `alt+down` - moves a selection down
        * `alt+up` - moves a selection up
        * `ctrl+cmd+g` - selects current word or variable name ***CUSTOM BINDING***
        * `cmd+ctrl+shift+->` - Smart Select Grow `smartSelect.grow` This will select an entire variable, hit it again and it will grow the selection to include different results depending on what is on the line. It may just select a line or what is between brackets or quotes.
        * `cmd+ctrl+shift+<-` - Smart Select Shrink `smartSelect.shrink` This will shrink the selection. The inverse to Smart Select Grow
        * `ctrl+m` - select the enclosing bracket or parenthesis ***CUSTOM BINDING***
        * `cmd+l` - selects current line
        * `cmd+alt+up` - insert cursor above
        * `cmd+alt+down` - insert cursor below
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
        * `alt+\` - deletes the entire word right of the cursor. ***CUSTOM BINDING***
        * `ctrl+alt+backspace` - deletes the partial word left of the cursor. If the word is camel case then it will delete each hump separately
        * `ctrl+alt+\` - deletes the partial word right of the cursor. If the word is camel case then it will delete each hump separately ***CUSTOM BINDING***
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
        * **Setup**
            * Disable VSCodes PHP Language Features
                * Go to `Extensions`
                * Search for `@builtin php`
                * Disable the `PHP Language Features` extension and reload VSCode
            * add `"files.associations": { "*.module": "php"},` to the `settings.json` file
    * `Vim` - *vscodevim*
    * `snippet-creator` - *nikitaKunevich* DEPRICATED - Creates snippets from a selection
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
    * #### Preview Mode:
        * When you normally open a file by clicking on it in the sidebar or by selecting it in the command palatte, it is being opened in Preview Mode. This means that when you open a different file, the new file will close the current file in Preview Mode then display the new file in Preview Mode. You can tell a file is in Preview Mode by looking at the file name in the editors tab. It will be italic when its in Preview Mode. There are several ways to have a file stay open and not be in Preview Mode:
            * Double click the file from the sidebar
            * Double click the editors tab
            * Right click the editors tab and select `Keep Open`
            * Use `cmd+k enter` and the current editor in Preview Mode will be kept open
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
        "workbench.statusBar.visible": true,
        "workbench.iconTheme": "vscode-icons",
        "workbench.colorTheme": "Slime",
        "terminal.integrated.scrollback": 10000,
        "blade.format.enable": true,
        "search.useIgnoreFiles": false,
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
        "php-cs-fixer.lastDownload": 1583260276435,
        "window.newWindowDimensions": "maximized",
        "editor.formatOnSave": true,
        "editor.defaultFormatter": "junstyle.php-cs-fixer",
        "explorer.confirmDragAndDrop": false,
        "editor.formatOnSaveTimeout": 5000,
        "window.nativeFullScreen": false,
        "intelephense.diagnostics.undefinedTypes": false,
        "intelephense.diagnostics.undefinedFunctions": false,
        "intelephense.diagnostics.undefinedConstants": false,
        "intelephense.diagnostics.undefinedClassConstants": false,
        "intelephense.diagnostics.undefinedMethods": false,
        "intelephense.diagnostics.undefinedVariables": false,
        "intelephense.diagnostics.undefinedProperties": false,
        "files.defaultLanguage": "{activeEditorLanguage}",
        "files.associations": {"*.module": "php"},
        "editor.fontFamily": "'JetBrains Mono', 'Fira Code', Menlo, Monaco, 'Courier New', monospace",
        "terminal.integrated.fontFamily": "'JetBrains Mono'",
        "terminal.integrated.fontSize": 13,
        "diffEditor.ignoreTrimWhitespace": false,
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
        {
            "key": "shift+cmd+r",
            "command": "workbench.action.showAllSymbols"
        },
        {
            "key": "cmd+t",
            "command": "-workbench.action.showAllSymbols"
        },
        {
            "key": "alt+`",
            "command": "workbench.action.terminal.focus"
        },
        {
            "key": "ctrl+cmd+`",
            "command": "workbench.action.terminal.focusNext"
        },
        {
            "key": "ctrl+alt+`",
            "command": "workbench.action.terminal.focusPrevious"
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
                " * ${TM_FILENAME_BASE} constructor",
                " */",
                "public function __construct()",
                "{",
                "\t$1",
                "}"
            ],
            "description": "Blank Class Constructor"
        },
        "Model Property - Timestamps": {
            "prefix": "mptimestamps",
            "body": [
                "/**",
                " * no timestamps needed",
                " * ",
                " * @var boolean",
                " */",
                "public \\$timestamps = false;"
            ],
            "description": "Model Property - Timestamps"
        },
        "Model Property - Guarded": {
            "prefix": "mpguarded",
            "body": [
                "/**",
                " * The attributes that aren't mass assignable.",
                " * ",
                " * @var array",
                " */",
                "protected \\$guarded = [$1];"
            ],
            "description": "Model Property - Guarded"
        },
        "Model Property - With": {
            "prefix": "mpwith",
            "body": [
                "/**",
                " * The attributes that are mass assignable.",
                " * ",
                " * @var array",
                " */",
                "protected \\$with = [$1];"
            ],
            "description": "Model Property - With"
        },
        "Model Property - Appends": {
            "prefix": "mpappends",
            "body": [
                "/**",
                " * The accessors to append to the model's array form.",
                " * ",
                " * @var array",
                " */",
                "protected \\$appends = [$1];"
            ],
            "description": "Model Property - Appends"
        },
        "Model Property - Fillable": {
            "prefix": "mpfillable",
            "body": [
                "/**",
                " * The attributes that are mass assignable.",
                " * ",
                " * @var array",
                " */",
                "protected \\$fillable = [$1];"
            ],
            "description": "Model Property - Fillable"
        },
        "Model Property - Hidden": {
            "prefix": "mphidden",
            "body": [
                "/**",
                " * The attributes that should be hidden for arrays.",
                " * ",
                " * @var array",
                " */",
                "protected \\$hidden = [$1];"
            ],
            "description": "Model Property - Hidden"
        },
        "Model Property - Visible": {
            "prefix": "mpvisible",
            "body": [
                "/**",
                " * The attributes that should be visible in arrays.",
                " * ",
                " * @var array",
                " */",
                "protected \\$visible = [$1];"
            ],
            "description": "Model Property - Visible"
        },
        "Model Property - Table": {
            "prefix": "mptable",
            "body": [
                "/**",
                " * The table associated with the model.",
                " * ",
                " * @var string",
                " */",
                "protected \\$table = '$1';"
            ],
            "description": "Model Property - Table"
        },
        "Model Property - Primary Key": {
            "prefix": "mpprimary",
            "body": [
                "/**",
                " * The primary key associated with the table.",
                " * ",
                " * @var string",
                " */",
                "protected \\$primaryKey = '$1';"
            ],
            "description": "Model Property - Primary Key"
        },
        "Model Property - Primary Key Type": {
            "prefix": "mpkeytype",
            "body": [
                "/**",
                " * The \"type\" of the auto-incrementing ID.",
                " * ",
                " * @var string",
                " */",
                "protected \\$keyType = 'string';"
            ],
            "description": "Model Property - Primary Key Type"
        },
        "Model Property - Incrementing Key": {
            "prefix": "mpincrementing",
            "body": [
                "/**",
                " * Indicates if the IDs are auto-incrementing.",
                " * ",
                " * @var bool",
                " */",
                "public \\$incrementing = false;"
            ],
            "description": "Model Property - Incrementing Key"
        },
        "Model Property - Dates": {
            "prefix": "mpdates",
            "body": [
                "/**",
                " * The attributes that should be mutated to dates.",
                " * ",
                " * @var array",
                " */",
                "protected \\$dates = ['$1'];"
            ],
            "description": "Model Property - Dates"
        },
        "Model Property - Casts": {
            "prefix": "mpcasts",
            "body": [
                "/**",
                " * The attributes that should be cast to native types.",
                " * ",
                " * @var array",
                " */",
                "protected \\$casts = ['$1' => '$2'];"
            ],
            "description": "Model Property - Casts"
        },
        "Model Property - Attributes": {
            "prefix": "mpattributes",
            "body": [
                "/**",
                " * The model's default values for attributes.",
                " * ",
                " * @var array",
                " */",
                "protected \\$attributes = ['$1' => $2];"
            ],
            "description": "Model Property - Attributes"
        },
        "Model Property - Dispatches Events": {
            "prefix": "mpattributes",
            "body": [
                "/**",
                " * The event map for the model.",
                " * ",
                " * @var array",
                " */",
                "protected \\$dispatchesEvents = [",
                "\t'$1' => $2::class",
                "];"
            ],
            "description": "Model Property - Dispatches Events"
        },
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
    ~/Library/Application Support/Code/User/snippets/html.json
    ```json
    {
        "Form Textarea": {
            "prefix": "bstextarea",
            "body": [
                "<div class=\"form-group\">",
                "\t<label for=\"$1\">$2</label>",
                "\t<textarea name=\"$3\" id=\"$4\" class=\"form-control\"></textarea>",
                "</div>"
            ],
            "description": "Bootstrap textarea"
        },
        "Bootstrap Text Field": {
            "prefix": "bstext",
            "body": [
                "<div class=\"form-group\">",
                "\t<label for=\"$1\">$2</label>",
                "\t<input type=\"text\" class=\"form-control\" id=\"$3\" name=\"$4\" placeholder=\"$5\">",
                "</div>"
            ],
            "description": "Bootstrap Text Field"
        },
        "Bootstrap Submit Button": {
            "prefix": "bssubmit",
            "body": [
                "<div class=\"form-group\">",
                "\t<button type=\"submit\" class=\"btn btn-default\">$1</button>",
                "</div>"
            ],
            "description": "Bootstrap Submit Button"
        }
    }
    ```
    ~/Library/Application Support/Code/User/snippets/javascript.json
    ```json
    {
        // Place your snippets for javascript here. Each snippet is defined under a snippet name and has a prefix, body and 
        // description. The prefix is what is used to trigger the snippet and the body will be expanded and inserted. Possible variables are:
        // $1, $2 for tab stops, $0 for the final cursor position, and ${1:label}, ${2:another} for placeholders. Placeholders with the 
        // same ids are connected.
        // Example:
        // "Print to console": {
        // 	"prefix": "log",
        // 	"body": [
        // 		"console.log('$1');",
        // 		"$2"
        // 	],
        // 	"description": "Log output to console"
        // }

        "Cypress describe Command": {
            "prefix": "cyde",
            "body": [
                "describe('$1', () => {",
                "\tit('$2', () => {",
                "\t\tcy.$3",
                "\t});",
                "});"
            ],
            "description": "Cypress describe Command"
        },
        "Cypress it Command": {
            "prefix": "cyit",
            "body": [
                "it('$1', () => {",
                "\tcy.$2",
                "});"
            ],
            "description": "Cypress it Command"
        },
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
* ### Example `.php_cs` config file
    * Make sure that `"php-cs-fixer.config": "/Users/scott/.vscode/.php_cs",` is added to the `settings.json` file and points to the correct directory

    ~/.vscode/.php_cs
    ```php
    <?php

    return PhpCsFixer\Config::create()
        ->setRules(array(
            '@PSR2' => true,
            'array_indentation' => true,
            'array_syntax' => array('syntax' => 'short'),
            'combine_consecutive_unsets' => true,
            'method_separation' => true,
            'no_multiline_whitespace_before_semicolons' => true,
            'single_quote' => true,
            'binary_operator_spaces' => array(
                'align_double_arrow' => false,
                'align_equals' => false,
            ),
            // 'blank_line_after_opening_tag' => true,
            // 'blank_line_before_return' => true,
            'braces' => array(
                'allow_single_line_closure' => true,
            ),
            // 'cast_spaces' => true,
            // 'class_definition' => array('singleLine' => true),
            'concat_space' => array('spacing' => 'none'),
            'declare_equal_normalize' => true,
            'function_typehint_space' => true,
            'hash_to_slash_comment' => true,
            'include' => true,
            'lowercase_cast' => true,
            // 'native_function_casing' => true,
            // 'new_with_braces' => true,
            // 'no_blank_lines_after_class_opening' => true,
            // 'no_blank_lines_after_phpdoc' => true,
            // 'no_empty_comment' => true,
            // 'no_empty_phpdoc' => true,
            // 'no_empty_statement' => true,
            'no_extra_consecutive_blank_lines' => array(
                'curly_brace_block',
                'extra',
                'parenthesis_brace_block',
                'square_brace_block',
                'throw',
                'use',
            ),
            // 'no_leading_import_slash' => true,
            // 'no_leading_namespace_whitespace' => true,
            // 'no_mixed_echo_print' => array('use' => 'echo'),
            'no_multiline_whitespace_around_double_arrow' => true,
            // 'no_short_bool_cast' => true,
            // 'no_singleline_whitespace_before_semicolons' => true,
            'no_spaces_around_offset' => true,
            // 'no_trailing_comma_in_list_call' => true,
            // 'no_trailing_comma_in_singleline_array' => true,
            // 'no_unneeded_control_parentheses' => true,
            'no_unused_imports' => true,
            'no_whitespace_before_comma_in_array' => true,
            'no_whitespace_in_blank_line' => true,
            // 'normalize_index_brace' => true,
            'not_operator_with_successor_space' => true,
            'object_operator_without_whitespace' => true,
            'ordered_imports' => array(
                'sort_algorithm' => 'length'
            ),
            // 'php_unit_fqcn_annotation' => true,
            // 'phpdoc_align' => true,
            // 'phpdoc_annotation_without_dot' => true,
            // 'phpdoc_indent' => true,
            // 'phpdoc_inline_tag' => true,
            // 'phpdoc_no_access' => true,
            // 'phpdoc_no_alias_tag' => true,
            // 'phpdoc_no_empty_return' => true,
            // 'phpdoc_no_package' => true,
            // 'phpdoc_no_useless_inheritdoc' => true,
            // 'phpdoc_return_self_reference' => true,
            // 'phpdoc_scalar' => true,
            // 'phpdoc_separation' => true,
            // 'phpdoc_single_line_var_spacing' => true,
            // 'phpdoc_summary' => true,
            // 'phpdoc_to_comment' => true,
            // 'phpdoc_trim' => true,
            // 'phpdoc_types' => true,
            // 'phpdoc_var_without_name' => true,
            // 'pre_increment' => true,
            // 'return_type_declaration' => true,
            // 'self_accessor' => true,
            // 'short_scalar_cast' => true,
            'single_blank_line_before_namespace' => true,
            // 'single_class_element_per_statement' => true,
            // 'space_after_semicolon' => true,
            // 'standardize_not_equals' => true,
            'ternary_operator_spaces' => true,
            // 'trailing_comma_in_multiline_array' => true,
            'trim_array_spaces' => true,
            'unary_operator_spaces' => true,
            'whitespace_after_comma_in_array' => true,
        ))
        //->setIndent("\t")
        ->setLineEnding("\n")
    ;
    ```