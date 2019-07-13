# Vim Mastery


## Notes on [Vim Mastery](https://laracasts.com/series/vim-mastery):

* ### Setup & Basics
    * [MacVim](https://macvim-dev.github.io/macvim/) Mac wrapper for Vim. Simplifies certain functionality.
        * Use Brew to install.
            ```
            brew install macvim
            ```
        * You can use the `mvim` command in the terminal to open a file in MacVim
    * #### <a id="modes"></a>Modes
        * `Normal` mode
            * Vim is in `normal` mode when it is opened.
            * This mode is used for navigation
            * **Key usage:**
                * `h` - moves the cursor left
                * `l` - moves the cursor right
                * `k` - moves the cursor up
                * `j` - moves the cursor down
                * `u` - undo
                * `o` - creates a new line
                * `gg` - go to the top of the file
                * `G` - go to the bottom of the file
                    * In Visual Mode, typing `G` will select the entire page.
                * `/` - search within the current file
                    * Type `n` to go to the next selection
                * `y` - yank or copy the current selection
                    * To copy an entire line, where the cursor is, type `Vy{esc}` then move to the area you want to paste and then press `p`
                    * Another shortcut to copy the entire line is `yy`, which does the same thing as above
                * `p` - paste
                * `.` - repeats the last command
                * `zz` - centers to screen at the current cursor line
                * `{ctrl}+^` - goes to previous edit point. Can be used to toggle between 2 files.
                * `gt` - Switch between tabs when using a GUI vim editor like MacVim
                * `d{action}{symbol}` - Deletes selection. action = `i` (inside symbol) or `a` (including symbol), followed by the symbol. i.e. `(`, `[`, `{`, `, `, etc
                    * `di(` - Deletes everything between the enclosed `()` where ever the cursor is at
                    * `di[` - Deletes everything between the enclosed `[]` where ever the cursor is at
                    * `di'` - Deletes everything between the enclosed `''` where ever the cursor is at
                * `c{action}{symbol}` - Delete and Insert Mode selection. action = `i` (inside symbol) or `a` (including symbol), followed by the symbol. i.e. `(`, `[`, `{`, `, `, etc 
                    * `ci'` - Deletes everything between the enclosed `''` where ever the cursor is at and then enters `Insert Mode`
                * `v{action}{symbol}` - Select. action = `i` (inside symbol) or `a` (including symbol), followed by the symbol. i.e. `(`, `[`, `{`, `, `, etc
                    * `vi'` - Selects everything between the enclosed `''` where ever the cursor is at
                    * `va'` - Selects everything between and including the enclosed `''` where ever the cursor is at
        * Pressing `esc` returns you to `Normal Mode` from any other mode
        * <a id="command"></a>`:` - `Command Mode`. Allows you to then enter commands
            * Some unix commands also work here. i.e. `:pwd` - present working directory
            * `:e` - edit. Used to edit a file
                * Once in edit mode, there is auto-complete available by starting to type the file name then press `tab`.
            * `:w` - write. Used to save file
            * `:q` - quit
            * `:wq` - saves the file then quits
            * `:colorscheme ` - followed by a tab, allows you to select one of the available color schemes. See the section below on [Vim Config File](#config)
            * `:so %` - source (reload) followed by `%`, which means the current file.
            * `:help` - access the Vim help file
            * `:bp` - opens previous file instead of current one.
                * `:ls` - lists all the current buffer files. It will display a numbered list of previously opened files. To open the file, use it's number `:b1` or `:b2` or `:b3`, etc
            * `:bufdo bd!` - Closes out of everything
            * `:bd` - Closes current buffer or tab
            * `:!{shell command}` - Execute a shell command.
                * `:!ls` - lists the current directory contents
            * `copen` - Open an error/other information list
            * `:!rm %` - Deletes the currently opened file from the project
        * <a id="insert"></a>`i` - `Insert Mode`. Allows you to type normally
        * <a id="visual"></a>`v` - `Visual Mode`. Used to make selections
            * use `h` & `l` to make the selection on the same line
            * you can then type `d` to delete the selection
        * <a id="visualline"></a>`V` - Line `Visual Mode`.
            * use `j` & `k` to make the line selections
            * you can then type `d` to delete the selection
    * #### Tabs
        * Tabs are considered buffers
        * A new tab can be added by typing `:tabedit {filename or path}`
        * To close all tabs or delete the buffer use the `:bd` command
        * `:tabclose` or `:tabc` - To close the current tab
    * #### Variables
        * Vim has variables for different parts of Vim.
        * Variables are prefixed with a `$`
        * In Normal Mode, use the echo command to see the value of a variable.
            ```
            echo $MYVIMRC
            /Users/Scott/.vimrc
            ```
        * Variables
            * `$MYVIMRC` - the path to the .vimrc file
    * #### Splits
        * You can split the window vertically or horizontally or a mixture of both
        * `sp` - splits the window horizontally
        * `vsp` - splits the window vertically
        * To exit out of a split use `:q`
        * To switch between splits the default keybindings are `{ctrl}w` followed by either `h`, `j`, `k` or `l`
            * If you only have 2 splits, use `{ctrl}ww` to toggle between them
        * You can maximize the current split by using `{ctrl}w|` it will increase the screen size of the current split. Then if you switched to the other split, the new split will increase a little bit in size. To maximize the size use `{ctrl}w|` again.
            * To normalize the splits back to the default use `{ctrl}w=`
    * #### <a id="config"></a>Vim config file
        * `.vimrc` is located in the users root directory. `~/.vimrc`
            * The file is empty by default
        *  Add the following commands on their own lines to update the config
            * You should add comments to the file to document what each line does. Comments are enclosed within double quotes.
                ```
                "------------------Mappings------------------"
                ```
                If the comment is just text and not symbols then the end quote isn't necessary
            * `syntax enable` - enables syntax highlighting
            * `colorscheme {scheme-name}` - changes the color theme. You can cycle through the available color themes by enter `command` mode and typing `:colorscheme ` followed by a tab.
            * `set backspace=indent,eol,start` - enables `backspace` or `delete` key to be used in `Insert Mode`. Normally the delete key does not work in `Insert Mode`.
                ```
                set backspace=indent,eol,start                      "Make backspace behave like every other editor.
                ```
            * #### <a id="mappings"></a>Mappings
                * `map` is used for global mappings.
                * `imap` is used for mapping only in Insert Mode
                * `nmap` is used for mapping only in Normal Mode
                * The `map` command has the following format
                    ```
                    {nmap|map|imap} {keyboard keybindings} {command to execute}
                    ```
                * To perform a carriage return in the command use `<cr>`
                * Special operator keys
                    * `ctrl` followed by a key use `<C-{key}>`. i.e. `ctrl+t` = `<C-t>`
                    * `command` followed by a key use `<D-{key}>`. i.e. `cmd+1` = `<D-1>`
                * Config file
                    ```
                    "------------------Mappings------------------"

                    "Make it easy to edit the .vimrc file.
                    nmap <Leader>ev :tabedit $MYVIMRC<cr>

                    "Add simple highlight removal
                    namp <Leader><space> :nohlsearch<cr>
                    ```
                * There is a common convention used for the keybindings. The first stroke is usually your own designator or namespace, so to speak. In the above mapping, JW used a `,`. You can also access the default leader key by using `<Leader>` before the remaining keybindings. The default leader is a `\`
                    ```
                    nmap <Leader>ev :tabedit $MYVIMRC<cr>
                    ```

                    The Leader can be changed by adding the following line before the mappings. 
                    ```
                    let mapleader=','                     "The default leader is a \ but a comma is much better.
                    ```
            * #### Auto-Commands
                * `autocmd` is used to specify an auto-command
                * Auto-command format
                    ```
                    autocmd {event to listen for} {file type or extension} {command to perform}
                    ```
                * Config file
                    ```
                    "------------------Auto-Commands------------------"

                    "Automatically source the .vimrc file on save.
                    augroup autosourcing
                        autocmd!
                        autocmd BufWritePost .vimrc source %
                    augroup END
                    ```
                    Wrapping the `BufWritePost` in an `augroup`, groups the commands together so they are run at the same time. The `autocmd!` command clears out the previous group and improves the `source` command performance.
            * #### Search
                * Use `/` to search the current file.
                * Config file
                    ```
                    "------------------Search------------------"
                    set hlsearch                        "Highlights the search term.
                    set incsearch                       "Starts highlighting as you type.
                    ```
                * See [Mapping](#mappings) section for info about shutting off the highlighting when you are done with the search.
                    * Custom mapping for this action is `/{space}`
            * #### Visuals
                * Config file
                    ```
                    "------------------Visuals------------------"
                    set t_CO=256                        "Terminal specific, force 256 colors.
                    colorscheme atom-dark               "Use the vim-atom-dark color theme.
                    set linespace=15                    "MacVim specific line-height
                    set guifont=Fira_Code:h14           "MacVim specific font & font size.
                    set guioptions-=l                   "Removes left scroll bar.
                    set guioptions-=L                   "Removes left scroll bar when in split mode.
                    set guioptions-=r                   "Removes right scroll bar.
                    set guioptions-=R                   "Removes right scroll bar when in split mode.
                    ```
            * #### Split Management
                * Config file
                    ```
                    "------------------Split Management------------------"
                    set splitbelow                            "Sets split below instead of above default.
                    set splitright                            "Sets split right instead of left default.
                    nmap <C-J> <C-W><C-J>                     "Switch to below split ctrl+j.
                    nmap <C-K> <C-W><C-K>                     "Switch to above split ctrl+k.
                    nmap <C-H> <C-W><C-H>                     "Switch to left split ctrl+h.
                    nmap <C-L> <C-W><C-L>                     "Switch to right split ctrl+l.
                    ```
            * #### Other Settings
                ```
                set number                              "Let's activate line numbers.
                set linespace=15                        "MacVim specific line-height
                ```
        * Any changes to the `.vimrc` file need to be save and sourced before they become available. See the `:so` command under the [Command Mode](#command) section.
    * #### Searching
        * To help with project wide searches, install `ag.vim`, which is a front end to `The Silver Searcher`, which is a very fast searching tool.
            ```
            brew install the_silver_searcher
            ```
            in the `~/.vim/plugins.vim` file, add the following to the plugin section
            ```
            Plugin 'rking/ag.vim'
            ```
            Save the `~/.vim/plugins.vim` file and then open the `.vimrc` file and save that. This will resource the plugins.vim file. Now install plugins within Vim.
            ```
            :PluginInstall
            ```
            * The `Ag` command is now available in the shell and within Vim.
                ```
                :Ag 'John Doe'
                ```
                This will search all files and open the quickfix window (`:copen`), with the search results. Use `j` and `k` to switch between the found files. Press `return` to open the selected file.
        * To perform the replace side of a search, use the [greplace.vim](https://github.com/skwp/greplace.vim) plugin.

            * in the `~/.vim/plugins.vim` file, add the following to the plugin section
                ```
                Plugin 'skwp/greplace.vim'
                ```
            * Save the `~/.vim/plugins.vim` file and then open the `.vimrc` file and add the following configuration for the plugin.
                ```
                "/
                "/ Greplace.vim
                "/
                set grepprg=ag                                          "We want to use Ag for the search.
                let g:grep_cmd_opts = '--line-numbers --noheading'
                ```
            * Now install plugins within Vim.
                ```
                :PluginInstall
                ```
            * The command `Gsearch` is now available
                * Type `:Gsearch` followed by return and it will ask you for a Search Pattern
                * After entering a search pattern it will ask you for where to search. Use `*` to search then entire project or you can use a path like `app/*`, to search only in the `app` directory.
                * It should bring up a list with all the results. If you select all the files in the list you can type the following to perform the actual find/replace. `:s{search pattern}/{replace pattern}`

                    MAKE SURE YOU HAVE ALL THE FILES SELECTED YOU WANT TO CHANGE
                    ```
                    :s/John Doe/Jane Doe<enter>
                    ```
                    After that
                    ```
                    Greplace<enter>
                    ```
                    It will now go through every change you made and ask if you want to commit the change. You can use `y` for yes, `n` for no or `a` to perform all the changes without asking. Now you need to save all changed files by typing `:wa`. This should exit you out of Greplace 
    * #### Adding theme colors
        * If you don't already have a `.vim` directory in your root, add one by typing `mkdir ~/.vim`
            ```
            cd ~/.vim
            mkdir colors
            cd colors
            ```
        * A good starting theme is the [vim-atom-dark](https://github.com/gosukiwi/vim-atom-dark) theme
            * Navigate to the colors directory where there are 2 files. The 256 colors version is good if you are using just the terminal to use Vim. Use the other file if you are using MacVim
            * Type the following to download the file
                ```
                wget https://raw.githubusercontent.com/gosukiwi/vim-atom-dark/master/colors/atom-dark.vim
                ```
            * After it is downloaded, open the `.vimrc` file and change the colorscheme setting to `colorscheme atom-dark`, then save the file, which should auto source the file.
    * #### The .gvimrc File
        * This is a configuration file that is used for GUI options only, like when using MacVim
        * Create a .gvimrc file in your root user directory.

            In MacVim
            ```
            :e ~/.gvimrc
            :w
            ```
        * Example `.gvimrc` file. JW uses this to disable the `cmd+p` key from using the system default for printing, so it can be used for `CtrlP` to be mapped to `cmd+p` instead.
            ```
            "Disable the print key for MacVim
            if has("gui_macvim")
                macmenu &File.Print key=<nop>
            endif
            ```
    * #### Viewing File System
        * To browse the current working directory use `:e.`
        * Good idea to use a package management system for Vim. [Vundle](https://github.com/VundleVim/Vundle.vim) is what JW uses.
            * Install & Setup
                * Install
                    ```
                    git clone https://github.com/VundleVim/Vundle.vim.git ~/.vim/bundle/Vundle.vim
                    ```
                * Create a plugins.vim file
                    ```
                    touch ~/.vim/plugins.vim
                    ```
                * Add the following to the top of your `.vimrc` file
                    ```
                    set nocompatible                                    "We want the latest Vim settings/options.
                    
                    so ~/.vim/plugins.vim
                    ```
                * Add the following to `~/.vim/plugins.vim`.
                    ```
                    filetype off                  " required

                    " set the runtime path to include Vundle and initialize
                    set rtp+=~/.vim/bundle/Vundle.vim
                    call vundle#begin()

                    Plugin 'VundleVim/Vundle.vim'
                    Plugin 'tpope/vim-vinegar'

                    " All of your Plugins must be added before the following line
                    call vundle#end()            " required
                    filetype plugin indent on    " required
                    ```
                * Within `Vim` or `MacVim`, use the command `PluginInstall` to install all the plugins registered in the file.
                    ```
                    :PluginInstall
                    ```
        * Plugins to help with file browsing processes
            * [vinegar.vim](https://github.com/tpope/vim-vinegar) - Allows for better directory browsing than what is included with Vim by default. 
                * add the following to the `~/.vim/plugins.vim` file under the line with `Plugin 'VundleVim/Vundle.vim'` and save it

                    ```
                    Plugin 'tpope/vim-vinegar'
                    ```
                * Within Vim type the following command in `Normal Mode`
                    ```
                    :PluginInstall
                    ```
                * ##### Usage
                    * `-` - Browses the directory that the currently opened file resides in
                        * Hitting `-` again goes up a directory level
                    * When you are browsing a directory the following commands can be used
                        * `d` - Allows you to add a new directory at the current level.
                        * `D` - Allows you to delete the currently selected directory or the currently selected file
                        * `%` - Creates a new file at the current level
            * [nerdtree](https://github.com/scrooloose/nerdtree) - Provides a more typical file system explorer that you'd normally see in a regular IDE.
                * add the following to the `~/.vim/plugins.vim` file along with any other plugins already registered and save it

                    ```
                    Plugin 'scrooloose/nerdtree'
                    ```
                * Within Vim type the following command in `Normal Mode`
                    ```
                    :PluginInstall
                    ```
                * ##### Usage
                    * `cmd+1` or `:NERDTreeToggle` - Within a file use the command `:NERDTreeToggle` to open the sidebar file browser. Type `:NERDTreeToggle` again to hide the sidebar. Added a new keybinding for this, which is `cmd+1`
                    * `?` - Shows all available opens when you are in the sidebar
            * [ctrlp.vim](https://github.com/ctrlpvim/ctrlp.vim) - Provides the same functionality as ctrl+p does in a regular IDE, where you can find files in your project quickly.
                * Some of the functionality requires ctags to be installed. This allows you to search for method names or properties.
                    ```
                    brew install ctags
                    ```
                    * Setup a tags file within a project. Should also add this file to the .gitignore file after it's created. This command needs to be run whenever a new method is added. Best to bind the `:!ctag -R` command to a keybinding.
                        ```
                        ctag -R
                        ```
                        or within Vim
                        ```
                        :!ctag -R
                        ```
                    * Usage:
                        * `:tag {methodname}` - searches files for a method with the methodname. Add a mapping for this - `<Leader>f`. This command supports tab completion.
                            * `:ts` - select from a list of files found from above search
                            * `:tn` - goes to next file found from above search
                            * `:tp` - goes to previous file found from above search
                        * `ctrl+]` - **IMPORTANT** Goes to the method when the cursor is over a method name. This will even bring you to the method when it's in a different file.
                            * `ctrl+^` - Can then be used to go back to previous edit point. i.e. the place you were before typing `ctrl+]`.
                * add the following to the `~/.vim/plugins.vim` file along with any other plugins already registered and save it

                    ```
                    Plugin 'ctrlpvim/ctrlp.vim'
                    ```
                * Within Vim type the following command in `Normal Mode`
                    ```
                    :PluginInstall
                    ```
                * ##### Usage
                    * `cmd+p`, `ctrl+p` or `:CtrlP` - Opens up the current working directory where you can start typing the file you are looking for and it will display the narrowed results as you type. Note: The first time you hit `ctrl+p`, it will index the site, which may take a moment to complete.
                        * If you see that a recently created file isn't showing up type `fn+f5` to refresh the results.
                    * `cmd+r` or `:CtrlPBufTag` - Allows you to search for methods or tags within the currently opened file.
                    * `cmd+e` or `:CtrlPMRUFiles` - Allows you to see the most recently used files
                    * `cmd+d` - If used within the results window, will toggle between matching the path or just the file name
    * #### Example .vimrc file
        ```
        set nocompatible                            		"We want the latest Vim settings/options.
        so ~/.vim/plugins.vim

        syntax enable
        set backspace=indent,eol,start              		"Make backspace behave like every other editor.
        let mapleader = ',' 					"The default is \, but a comma is much better.
        set number						    "Let's activate line numbers. JW uses nonumber.




        "-------------Visuals--------------"
        colorscheme atom-dark

        set t_CO=256						"Use 256 colors. This is useful for Terminal Vim.
        set macligatures					"We want pretty symbols, when available
        set guifont=Fira_Code:h17			"Set the default font family and size.
        set guioptions-=e					"We don't want Gui tabs.
        set linespace=15   					"Macvim-specific line-height.

        set guioptions-=l                   "Disable Gui scrollbars.
        set guioptions-=L
        set guioptions-=r
        set guioptions-=R

        hi LineNr guibg=bg                  "Set the line number background color.
        set foldcolumn=2                    "Establish a fold column width on the left.
        hi foldcolumn guibg=bg              "Set the background color for the fold column

        hi vertsplit guifg=bg guibg=grey    "Set the background color to the vertical split bar




        "-------------Search--------------"
        set hlsearch						"Highlight all matched terms.
        set incsearch						"Incrementally highlight, as we type.




        "-------------Split Management--------------"
        set splitbelow 						"Make splits default to below...
        set splitright						"And to the right. This feels more natural.

        "We'll set simpler mappings to switch between splits.
        nmap <C-J> <C-W><C-J>
        nmap <C-K> <C-W><C-K>
        nmap <C-H> <C-W><C-H>
        nmap <C-L> <C-W><C-L>




        "-------------Mappings--------------"

        "Make it easy to edit the Vimrc file.
        nmap <Leader>ev :tabedit $MYVIMRC<cr>

        "Add simple highlight removal.
        nmap <Leader><space> :nohlsearch<cr>

        nmap <Leader>f :tag<space>




        "-------------Plugins--------------"

        "/
        "/ CtrlP
        "/
        set wildignore+=*/tmp/*,*.so,*.swp,*.zip     " MacOSX/Linux

        let g:ctrlp_custom_ignore = 'node_modules\DS_Store\|git'
        let g:ctrlp_match_window = 'top,order:ttb,min:1,max:30,results:30'
        let g:ctrlp_user_command = ['.git', 'cd %s && git ls-files -co --exclude-standard']	"Ignore files in .gitignore.

        nmap <D-p> :CtrlP<cr>
        nmap <D-r> :CtrlPBufTag<cr>         "Requires ctags. brew install ctags.
        nmap <D-e> :CtrlPMRUFiles<cr>

        "/
        "/ NERDTree
        "/
        let NERDTreeShowHidden = 1			"Always show hidden files.
        let NERDTreeHijackNetrw = 0         "Make using - work as expected for vinegar.vim

        "Make NERDTree easier to toggle.
        nmap <D-1> :NERDTreeToggle<cr>

        "/
        "/ Greplace.vim
        "/
        set grepprg=ag                                          "We want to use Ag for the search.
        let g:grep_cmd_opts = '--line-numbers --noheading'




        "-------------Laravel Specific--------------"
        nmap <Leader>lr :e app/Http/routes.php<cr>			"Opens routes.php file.
        nmap <Leader>lm :!php artisan make:				    "Perform artisan make command
        nmap <Leader><Leader>c :e app/Http/Controllers/<cr>
        nmap <Leader><Leader>m :e app/<cr>
        nmap <Leader><Leader>v :e resources/views/<cr>




        "-------------Auto-Commands--------------"

        "Automatically source the Vimrc file on save.

        augroup autosourcing
            autocmd!
            autocmd BufWritePost .vimrc source %
        augroup END




        "-------------Tips and Reminders--------------"
        " - Press 'zz' to instantly center the line where the cursor is located.
        ```

