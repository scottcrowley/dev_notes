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
    * #### Sticky Nav Bar
        * Nav bar disappears when scrolling but then reappears when a section is scrolled into view. Using CSS & regular JS
        * See [Stick Nav Bar](../Vue.js/Vue%202%20Examples.md#sticky-nav-bar) example in [Vue 2 Examples](../Vue.js/Vue%202%20Examples.md) document
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
    * #### Tooltips
        ```html
        <div class="tooltip">Hover over me
            <span class="tooltiptext">Tooltip text</span>
        </div>
        ```
        ```css
        /* Tooltip container */
        .tooltip {
            position: relative;
            display: inline-block;
            border-bottom: 1px dotted black; /* If you want dots under the hoverable text */
        }

        /* Tooltip text */
        .tooltip .tooltiptext {
            visibility: hidden;
            width: 120px;
            background-color: black;
            color: #fff;
            text-align: center;
            padding: 5px 0;
            border-radius: 6px;
            
            /* Position the tooltip text - see examples below! */
            position: absolute;
            z-index: 1;
        }

        /* Show the tooltip text when you mouse over the tooltip container */
        .tooltip:hover .tooltiptext {
            visibility: visible;
        }
        ```
        Positioning
        ```css
        /* Right */
        .tooltip .tooltiptext {
            top: -5px;
            left: 105%;
        }
        /* Left */
        .tooltip .tooltiptext {
            top: -5px;
            right: 105%;
        }
        /* Top */
        .tooltip .tooltiptext {
            width: 120px;
            bottom: 100%;
            left: 50%;
            margin-left: -60px; /* Use half of the width (120/2 = 60), to center the tooltip */
        }
        /* Bottom */
        .tooltip .tooltiptext {
            width: 120px;
            top: 100%;
            left: 50%;
            margin-left: -60px; /* Use half of the width (120/2 = 60), to center the tooltip */
        }
        ```
        Using with arrows
        ```css
        /* Bottom Arrow */
        .tooltip .tooltiptext::after {
            content: " ";
            position: absolute;
            top: 100%; /* At the bottom of the tooltip */
            left: 50%;
            margin-left: -5px;
            border-width: 5px;
            border-style: solid;
            border-color: black transparent transparent transparent;
        }
        /* Top Arrow */
        .tooltip .tooltiptext::after {
            content: " ";
            position: absolute;
            bottom: 100%;  /* At the top of the tooltip */
            left: 50%;
            margin-left: -5px;
            border-width: 5px;
            border-style: solid;
            border-color: transparent transparent black transparent;
        }
        /* Left Arrow */
        .tooltip .tooltiptext::after {
            content: " ";
            position: absolute;
            top: 50%;
            right: 100%; /* To the left of the tooltip */
            margin-top: -5px;
            border-width: 5px;
            border-style: solid;
            border-color: transparent black transparent transparent;
        }
        /* Right Arrow */
        .tooltip .tooltiptext::after {
            content: " ";
            position: absolute;
            top: 50%;
            left: 100%; /* To the right of the tooltip */
            margin-top: -5px;
            border-width: 5px;
            border-style: solid;
            border-color: transparent transparent transparent black;
        }
        ```
        Using Fade In Animation
        ```css
        .tooltip .tooltiptext {
            opacity: 0;
            transition: opacity 1s;
        }

        .tooltip:hover .tooltiptext {
            opacity: 1;
        }
        ```
    * #### Animations
        * When setting up an animation you need to name it using `animation-name: example;` and then use the `@keyframes example` rule to perform the animation.
            ```css
            @keyframes example {
                from {background-color: red;}
                to {background-color: yellow;}
            }

            /* The element to apply the animation to */
            div {
                width: 100px;
                height: 100px;
                background-color: red;
                animation-name: example;
                animation-duration: 4s;
            }
            ```
        * Other properties for animations:
            * `animation-duration` - defines how long time an animation should take to complete
            * `animation-delay` - specifies a delay for the start of an animation
                * A 2 second delay would be displayed as `2s`
                * A negative number can be used to say that its already been running for the time provided
            * `animation-iteration-count` - specifies the number of times an animation should run
                * Use an integer or `infinite` to run indefinitely
            * `animation-direction` - specifies whether an animation should be played forwards, backwards or in alternate cycles
                * `normal` - The animation is played as normal (forwards). This is default
                * `reverse` - The animation is played in reverse direction (backwards)
                * `alternate` - The animation is played forwards first, then backwards
                * `alternate-reverse` - The animation is played backwards first, then forwards
            * `animation-timing-function` - specifies the speed curve of the animation
                * `ease` - Specifies an animation with a slow start, then fast, then end slowly (this is default)
                * `linear` - Specifies an animation with the same speed from start to end
                * `ease-in` - Specifies an animation with a slow start
                * `ease-out` - Specifies an animation with a slow end
                * `ease-in-out` - Specifies an animation with a slow start and end
                * `cubic-bezier(n,n,n,n)` - Lets you define your own values in a cubic-bezier function
            * `animation-fill-mode` - specifies a style for the target element when the animation is not playing (before it starts, after it ends, or both)
                * `none` - Default value. Animation will not apply any styles to the element before or after it is executing
                * `forwards` - The element will retain the style values that is set by the last keyframe (depends on animation-direction and animation-iteration-count)
                * `backwards` - The element will get the style values that is set by the first keyframe (depends on animation-direction), and retain this during the animation-delay period
                * `both` - The animation will follow the rules for both forwards and backwards, extending the animation properties in both directions
        * Shorthand usage
            ```css
            div {
                animation-name: example;
                animation-duration: 5s;
                animation-timing-function: linear;
                animation-delay: 2s;
                animation-iteration-count: infinite;
                animation-direction: alternate;
            }

            /* The above can be written using shorthand */
            div {
                animation: example 5s linear 2s infinite alternate;
            }
            ```
        * @keyframe examples
            ```css
            @keyframes example {
                0%   {background-color: red;}
                25%  {background-color: yellow;}
                50%  {background-color: blue;}
                100% {background-color: green;}
            }
            @keyframes example {
                from {background-color: red;}
                to {background-color: yellow;}
            }
            @keyframes example {
                0%   {background-color:red; left:0px; top:0px;}
                25%  {background-color:yellow; left:200px; top:0px;}
                50%  {background-color:blue; left:200px; top:200px;}
                75%  {background-color:green; left:0px; top:200px;}
                100% {background-color:red; left:0px; top:0px;}
            }
            ```
        * Using the `transition` directive
            * The transition directive waits for a change on a specific property then animates the change.
                ```css
                div {
                    width: 100px;
                    height: 100px;
                    background: red;
                    transition: width 2s;
                }
                div:hover {
                    width: 300px;
                }
                ```
                This the div will have a width of 100px until you hover over it, then it will animate a change to 300px width over 2 seconds
            * Changing multiple properties
                ```css
                div {
                    transition: width 2s, height 4s;
                }
                ```
            * `transition-timing-function` & `transition-delay` can be used just like the same options available on the `animation` directive.
            * Using with `transform`
                ```css
                div {
                    transition: width 2s, height 4s, transform 2s;
                }
                ```
            * Shorthand
                ```css
                div {
                    transition-property: width;
                    transition-duration: 2s;
                    transition-timing-function: linear;
                    transition-delay: 1s;
                }

                /* The above can be written using shorthand */
                div {
                    transition: width 2s linear 1s;
                }
                ```
        * Using `transform`
            * `translate()` - moves an element from its current position (according to the parameters given for the X-axis and the Y-axis)
                ```css
                transform: translate(50px, 100px);
                ```
            * `translateX()` - moves an element from its current position according to the X-axis
                ```css
                transform: translateX(50px);
                ```
            * `translateY()` - moves an element from its current position according to the Y-axis
                ```css
                transform: translateY(100px);
                ```
            * `rotate()` - rotates an element clockwise or counter-clockwise according to a given degree
                ```css
                transform: rotate(20deg);
                ```
            * `scale()` - increases or decreases the size of an element (according to the parameters given for the width and height)
                ```css
                /* 2 times its original width and 3 times its original height */
                transform: scale(2, 3);
                ```
            * `scaleX()` - increases or decreases the width of an element
                ```css
                transform: scaleX(2);
                ```
            * `scaleY()` - increases or decreases the height of an element
                ```css
                transform: scaleY(3);
                ```
            * `skew()` - skews an element along the X and Y-axis by the given angles
                ```css
                transform: skew(20deg, 10deg);
                ```
            * `skewX()` - skews an element along the X-axis by the given angle
                ```css
                transform: skewX(20deg);
                ```
            * `skewY()` - skews an element along the Y-axis by the given angle
                ```css
                transform: skewY(10deg);
                ```
            * `matrix()` - combines all the 2D transform methods into one
                * Parameters - `matrix(scaleX(),skewY(),skewX(),scaleY(),translateX(),translateY())`
                    ```css
                    transform: matrix(1, -0.3, 0, 1, 0, 0);
                    ```