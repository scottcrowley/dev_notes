# Vim Mastery


## Notes on [Vim Mastery](https://laracasts.com/series/vim-mastery):

* ### Setup & Basics
    * [MacVim](https://macvim-dev.github.io/macvim/) Mac wrapper for Vim. Simplifies certain functionality.
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
                * `/` - search within the current file
                    * Type `n` to go to the next selection
                * `y` - yank or copy the current selection
                    * To copy an entire line, where the cursor is, type `Vy{esc}` then move to the area you want to paste and then press `p`
                    * Another shortcut to copy the entire line is `yy`, which does the same thing as above
                * `p` - paste
                * `.` - repeats the last command
                * `zz` - centers to screen at the current cursor line
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
                    let mapleader = ','                     "The default leader is a \ but a comma is much better.
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
        

