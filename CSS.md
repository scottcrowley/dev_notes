# CSS


## Notes from Laracasts: Modern CSS for Backend Developers

* ### Flex box:
    * `display: flex;` will align all children as inline or horizontally. Same as using flex-direction: row;
    * `flex-direction: { column, column-reverse, inherit, initial, row, row-reverse, unset }` changes how the children are aligned horizontally
        * `column-reverse` can be used when having a mobile layout where you want the 2nd element to display before the 1st.
    * You can apply the `order` directive to a child to have the layout order specifically set. By default all child elements have an order of 0. To reorder all the children then the order directive needs to be on every element starting with `order: 1`; 
        * If one element needs to come first then you can do `order: -1;` to place it before all other children.
    * `justify-content: { space-around, space-between, etc }` how the children are spaced out.
        * `space-around` - same spacing around every child
        * `space-between` - same spacing between all children
    * `align-items:` how the children are aligned vertically unless parent has flex-direction: column; In this case the horizontal and vertical are reversed.
    * `flex: {n};` Changes how much space a child takes up.
        * `1` = element is at 100% or if applied to all children then they'd all take the same amount of space. Also makes an element take up all available space.
        * `2` = element is at 200%. All other children will adjust their size to allow for this element to be twice the size as the others.
        * ...n
    * `flex-wrap:` when placed on the flex container, the children that have a set width will keep that width and the remaining elements will wrap to the next line as needed.
        * i.e. you have a flex container with 8 children elements, each with a width set to 25%. Without `flex-wrap`, all 8 elements will have their widths reduced to keep them on the same line. With `flex-wrap`, the children keep their specified widths of 25% and 4 elements will be on the first row and the remaining 4 on the next row.
* ### Font Size
    * a default value of `16px` is usually applied to the html element by most browsers. This can be used to adjust the size of fonts when using mobile layouts as long as all the other elements are using `rem`'s instead of `px`'s
        * i.e. There is a formula for figuring out the `rem` value. `current px size / current root element px size = rem`  `h1: 40px` when the html element is `16px` then the `h1` should be `2.5em`
        * Using this method allows for relative changes to be made to the various font-sizes without hard coding them. You can simply add a media query for a specific screen size, then change the font-size on the html element based on that size. This will then change all the other font-sizes based on the root element size.
* ### Color
    * To change the color to a variant of the background
        * `color: rgba(255,255,255,.5);` changes the color to `white` and uses `50% alpha`.
    * For links you can have the color inherited from the parent by using:
        * `color: inherit;`
* ### Border Radius
    * if set to something like `9999px;` then the corners are at max roundness.
* ### Background Images
    * You can use more than 1 background image for an element by having each property value separated by a comma
        * `background-image: url('url/to/the/1st/bottom-left-image.svg'), url('url/to/the/2nd/bottom-right-image.svg');`
        * `background-repeat: no-repeat;`
        * `background-position: 0, 100%, 100% 100%;`
* ### Typical Utility Classes
    * `container` - usually sets a max-width: 1200px; and margin: 0 auto;
    * `row` or `flex` - `display: flex;` if a col is a direct child element and it has padding, it is usually required to add a negative margin to compensate for the padding. i.e. col has `padding: 10px;` then row would have `margin-left: -10px;` and `margin-right: -10px;`
    * `col` - `flex: 1;` and some default padding ~ `padding: 10px;`
    * `box` - sets a background color & maybe some height to a content section
* ### Tailwind CSS Framework
    * There is a `Tailwind Laravel Preset` available at <https://github.com/laravel-frontend-presets/tailwindcss> that will automatically install `Tailwind` and change the default views to use `Tailwind` classes.
    * Can be pulled in via a CDN or imported into a project using NPM. <https://tailwind.css>
    * Install
        ```zsh
        npm install tailwindcss --save-dev
        npm install laravel-mix-tailwind --save-dev
        node_modules/.bin/tailwind init tailwind.js //Creates the tailwind.js config file in the root of the project
        ```
        make sure that `require('laravel-mix-tailwind');` is added to the top of `webpack.mix.js` file.
        
        Also add the `tailwind()` method to the end of the `mix` command.
            ```
            mix.js('resources/assets/js/app.js', 'public/js')
                .sass('resources/assets/sass/app.scss', 'public/css')
                .tailwind();
            ```
    * Custom Directives:
        * within your `.sass` file you can use `@screen` followed by either `sm`, `md`, `lg` or `xl`. This allows you to specify a breakpoint without hard coding a screen size value.
* ### Miscellaneous CSS add-ons & plugins
    * `Custom Margin & Padding` helper utilities. James Furey <https://github.com/furey>
* ### How-to's
    * #### Having an element be hidden if there is not content within it.
        * This can be useful in a Vue component slot that may or may not have content in it. e.g. the component has a slot named footer within a footer tag. You can assign the :empty psuedo class to the footer element, which will be hidden if there is no content within it.
            ```css
            footer:empty {
                display: none;
            }
            ```
    * #### Modal with zero Javascript
        * How to create a modal without using any javascript at all and using `Blade Components` to make it reuseable. This works by using anchor tags and the `:target` psuedo class to toggle visibility.

            **`modal.blade.php`**
            ```html
            <div id="{{ $name }}" class="overlay">
                <a href="#" class="cancel"></a>

                <div class="modal">
                    {{ $slot }}

                    <a href="#" class="close">&times;</a>
                </div>
            </div>
            ```
            **`modal.css`**
            ```css
            .overlay {
                visibility: hidden;
                position: absolute;
                top: 0;
                right: 0;
                bottom: 0;
                left: 0;
                display: flex;
                align-items: center;
                justify-content: center;
                background: rgba(0, 0, 0, .7);
            }

            .overlay:target {
                visibility: visible;
            }

            .modal {
                position: relative;
                width: 600px;
                max-width: 80%;
                background: white;
                border-radius: 8px;
                padding: 1em 2em;
            }

            .modal .close {
                position: absolute;
                top: 15px;
                right: 15px;
                color: grey;
                text-decoration: none;
            }

            .overlay .cancel {
                position: absolute;
                width: 100%;
                height: 100%;
            }
            ```
            **`usage.blade.php`**
            ```html
            <a href="#join-modal">Join</a>

            @component('modal', ['name' => 'join-modal'])
                <h1>Pick a Plan</h1>

                <p>
                Lorem ipsum...
                </p>
            @endcomponent
            ```
