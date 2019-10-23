# Tailwind CSS


## Notes taken from "[Designing with Tailwind CSS](https://tailwindcss.com/screencasts/)" screencast @ tailwindcss.com
### Covers v 1.1.2

* ### Tailwind Config File (tailwind.config.js)
    * `Active` variants are not enabled by default
        * To enable the `active` variant for background color:
            ```js
            variants: {
                backgroundColor: ['responsive', 'hover', 'focus', 'active']
            }
            ```
            It is important to note that when specifying the variant you want, that you include all the variants. For example: if you just added `'active'`, all the other variants for `backgroundColor` would not be enabled since the default values are overridden by what you have in the config file.
            
            Also, order matters. Make sure to order the array properly, where the later values supercede the earlier ones.
* ### Usage Tips
    * **How to have an image take up the entire space of it's container**
        * Use [Object Fit](https://tailwindcss.com/docs/object-fit) and [Object Position](https://tailwindcss.com/docs/object-position) utilities on the `img` element.
            * `object-cover` - Makes the image cover the entire container
                * Other Object Fit classes
                    * `object-contain` - Makes the image resize to be fully contained in the container
                    * `object-fill` - Makes the image stretch to fill the entire container
                    * `object-none` - Makes the image display at full size regardless of the container size. This does not resize the container though.
                    * `object-scale-down` - Displays the image at its original size by scales it down to fit within the container.
            * `object-center` - Makes the image centered within the container.
                * Other Object Position classes
                    * `object-bottom`, `object-left`, `object-left-bottom`, `object-left-top`, `object-right`, `object-right-bottom`, `object-right-top`, `object-top`
        * IE 11 does not support Object Fit, so you can use a background image along with the bg-cover and bg-center utilities to accomplish the same thing.
            ```html
            <div class="h-48 bg-cover bg-center" style="background-image: url('/img/someimage.jpg');"></div>
            ```
    * **How to constrain an image to a certain ratio**
        * By using percentage based padding you can constrain a container to a certain ratio. In the example we want to constrain the image to 2/3 ratio by using padding-bottom
        * By making the `img` tag within the container have a position of `absolute`, you are removing it from the normal page flow. If you then add the `top-0` utility to the image it will be positioned at the top of the screen since that is the only designated relative position. Now if we assign `relative` to the container, we are changing the `relative` position that the `absolute` `img` tag aligns to. The `img` tag also needs to have `w-full` and `h-full` as well as `object-cover` added to it.
            ```html
            <div class="relative pb-2/3">
                <img src="/img/someimage.jpg" class="absolute h-full w-full object-cover">
            </div>
            ```
            Now when the container is resized, the image will resize correctly but at the aspect ratio we set using `pb-2/3`
    * **Custom CSS placement**
        * It is important to order the component imports in your main css file properly.
            * `@tailwind 'base';` or `@import "tailwindcss/base";` should always come first followed by `@tailwind 'components';` or `@import "tailwindcss/components";`. After the tailwind components is where you should have any of your custom components at, followed by `@tailwind 'utilities'` or `@import "tailwindcss/utilities";`. The reason for this is because if for example you have a custom button component that includes padding and you want to override that padding by using the tailwind utility `p-?`, this will only work if the tailwind utilities are imported after your custom button component. If your component comes after the utilities then the `p-?` will be ignored since the custom declaration overrides it.
    * **Truncate the text in an element**
        * If you use the `truncate` utility class, any text that overflows or wraps will be replaced with `...`
    * **A way to add depth without using only a shadow**
        * In this example, we are working on a card that has an image with a info content div on top of it to give the illusion of 2 different layers.
        * The use of relative positioning along with negative margins is how this can be accomplished.
            ```html
            <div class="relative pb-5/6">
                <img src="/img/someimage.jpg" class="absolute h-full w-full object-cover rounded-lg shadow-md">
            </div>
            <div class="relative px-4 -mt-16">
                <div class="bg-white p-6 rounded-lg shadow-lg">
                    //content
                </div>
            </div>
            ```
            Notice that the image has a smaller shadow applied to it since it is underneath the content div. This gives the illusion of more depth. Also, the content div container has a `relative` position to it. If it didn't, it would be rendered under the image because css renders positioned elements on top of non positioned elements. Since the image div is set to `relative` and the content div (without `relative`) is set to the default position of `static`, then the image will be rendered on top on the content. Not what we want. Both containers need to have `relative` for the flow to continue.
* ### Working with SVGs
    * Go to https://tailwindcss.com/resources to see various sites that have free images & svg icons for you to use or download.
    * When you receive an svg from a designer, it may be sent as an export from a sketch app like [Sketch](https://sketchapp.com). This file may contain a lot of information that isn't needed when using an inline svg. You can copy the content from the file and go to https://jakearchibald.github.io/svgomg/ then click on `Paste Markup` and then paste in the code you copied. Then you can click on the copy button to copy the new reduced size svg.
    * Square svgs are much easier to work with.
    * If the `svg` tag has `width` and `height` hard coded, remove them so you can control these attributes with the width & height utilities.
    * If the `path` element within the svg contains a `fill` attribute, you should remove it as well so the svg color can be controlled using utilities
        * Use `fill-current` to fill the svg with the current text color, then you can use normal text color utilities to change the color to what you want (i.e. `text-blue-500`)
* ### Using PurgeCSS with Tailwind
    * Install
        ```zsh
        npm install @fullhuman/postcss-purgecss
        ```
    * Update config
        * When using `postcss.config.js` file, add the following to the plugins section
            ```js
            process.env.NODE_ENV === 'production' && require('@fullhuman/postcss-purgecss')({
                content: [
                    //strings of the directories where you have your template files located
                    //i.e. './resources/views/**/*.blade.php' & './resources/js/**/*.vue'
                ],
                defaultExtractor: content => content.match(/[A-Za-z0-9-_:/]+/g) || []
            })
            ```
            Using `process.env.NODE_ENV === 'production' && ` means that it should only require PurgeCss only when in production mode.
        * When using Laravel and Webpack, the `webpack.mix.js` file should look something like the following:
            ```js
            mix.js('resources/js/app.js', 'public/js')
                .postCss('resources/css/app.css', 'public/css', [
                    require('postcss-import'),
                    require('tailwindcss'),
                    require('postcss-nesting'),
                    process.env.NODE_ENV === 'production' && require('@fullhuman/postcss-purgecss')({
                        content: [
                            //strings of the directories where you have your template files located
                            //i.e. './resources/views/**/*.blade.php' & './resources/js/**/*.vue'
                        ],
                        defaultExtractor: content => content.match(/[A-Za-z0-9-_:/]+/g) || []
                    })
                ]);
            ```
* ### Component Examples
    * **Responsive NavBar**
        Uses Tailwind along with Vue.js to make a collapsable navbar. Features a "hamburger" svg which turns into an "X" svg when the menu is open.
        ```js
        <template>
            <header class="bg-gray-900 sm:flex sm:justify-between sm:items-center sm:px-4 sm:py-3">
                <div class="flex items-center justify-between px-4 py-3 sm:p-0">
                <div>
                    <img class="h-8" src="/img/logo-inverted.svg" alt="Workcation">
                </div>
                <div class="sm:hidden">
                    <button @click="isOpen = !isOpen" type="button" class="block text-gray-500 hover:text-white focus:text-white focus:outline-none">
                    <svg class="h-6 w-6 fill-current" viewBox="0 0 24 24">
                        <path v-if="isOpen" fill-rule="evenodd" d="M18.278 16.864a1 1 0 0 1-1.414 1.414l-4.829-4.828-4.828 4.828a1 1 0 0 1-1.414-1.414l4.828-4.829-4.828-4.828a1 1 0 0 1 1.414-1.414l4.829 4.828 4.828-4.828a1 1 0 1 1 1.414 1.414l-4.828 4.829 4.828 4.828z"/>
                        <path v-if="!isOpen" fill-rule="evenodd" d="M4 5h16a1 1 0 0 1 0 2H4a1 1 0 1 1 0-2zm0 6h16a1 1 0 0 1 0 2H4a1 1 0 0 1 0-2zm0 6h16a1 1 0 0 1 0 2H4a1 1 0 0 1 0-2z"/>
                    </svg>
                    </button>
                </div>
                </div>
                <nav :class="isOpen ? 'block' : 'hidden'" class="px-2 pt-2 pb-4 sm:flex sm:p-0">
                    <a href="#" class="block px-2 py-1 text-white font-semibold rounded hover:bg-gray-800">List your property</a>
                    <a href="#" class="mt-1 block px-2 py-1 text-white font-semibold rounded hover:bg-gray-800 sm:mt-0 sm:ml-2">Trips</a>
                    <a href="#" class="mt-1 block px-2 py-1 text-white font-semibold rounded hover:bg-gray-800 sm:mt-0 sm:ml-2">Messages</a>
                </nav>
            </header>
        </template>

        <script>
        export default {
            data() {
                return {
                    isOpen: false,
                }
            },
        }
        </script>
        ```

    * **Clickable event behind a dropdown and escape listener**
        * This is one part of a dropdown component. When a dropdown is displayed, we need a way to accept a click event any place behind the dropdown so it closes the dropdown. This is accomplished by adding a invisible button that takes up the entire viewport.
            By adding the button with a fixed position, full width & height, we can use a simple click event listener to close the dropdown. Also, by assigning a -1 to the tabindex attribute you can disable the keyboard from accessing the button using the tab key.
            ```html
            <div class="relative">
                <!-- Trigger button for dropdown -->
                <button 
                    @click="isOpen = !isOpen" 
                    class="relative z-10 block h-8 w-8 rounded-full overflow-hidden border-2 border-gray-600 focus:outline-none focus:border-white"
                >
                    <img 
                        class="h-full w-full object-cover" 
                        src="https://images.unsplash.com/photo-1487412720507-e7ab37603c6f?ixlib=rb-1.2.1&ixid=eyJhcHBfaWQiOjEyMDd9&auto=format&fit=crop&w=256&q=80" 
                        alt="Your avatar">
                </button>
                <!-- Overlay button that is behind dropdown -->
                <button 
                    v-if="isOpen" 
                    @click="isOpen = false" 
                    tabindex="-1" 
                    class="fixed inset-0 h-full w-full bg-black opacity-50 cursor-default">
                </button>
                <!-- The actual dropdown menu -->
                <div v-if="isOpen" class="absolute right-0 mt-2 py-2 w-48 bg-white rounded-lg shadow-xl">
                    <a href="#" class="block px-4 py-2 text-gray-800 hover:bg-indigo-500 hover:text-white">Account settings</a>
                    <a href="#" class="block px-4 py-2 text-gray-800 hover:bg-indigo-500 hover:text-white">Support</a>
                    <a href="#" class="block px-4 py-2 text-gray-800 hover:bg-indigo-500 hover:text-white">Sign out</a>
                </div>
            </div>
            ```
            Something to also note is the the trigger is being moved in front of the overlay so that the mouse will be a pointer when hovering over the it. This is done by adding the `relative` and `z-10` class to the trigger button. Without these classes, the trigger is rendered behind the overlay and the mouse will not change to a pointer when hovering over it.
        * To allow the user to press escape on the keyboard the following should be added to the create method on your Vue component.
            ```js
            created() {
                // Esc key event closure
                const handleEscape = (e) => {
                    if (e.key === 'Esc' || e.key === 'Escape') {
                        this.isOpen = false
                    }
                }

                // Add the key event to the document
                document.addEventListener('keydown', handleEscape);

                // Before the component is destroyed, remove the listener
                this.$once('hook:beforeDestroy', () => {
                    document.removeEventListener('keydown', handleEscape)
                });
            }
            ```
    * **Full Dropdown Component**
        ```js
        <template>
        <div class="relative">
            <button @click="isOpen = !isOpen" class="relative z-10 block h-8 w-8 rounded-full overflow-hidden border-2 border-gray-600 focus:outline-none focus:border-white">
                <img class="h-full w-full object-cover" src="https://images.unsplash.com/photo-1487412720507-e7ab37603c6f?ixlib=rb-1.2.1&ixid=eyJhcHBfaWQiOjEyMDd9&auto=format&fit=crop&w=256&q=80" alt="Your avatar">
            </button>
            <button v-if="isOpen" @click="isOpen = false" tabindex="-1" class="fixed inset-0 h-full w-full bg-black opacity-50 cursor-default"></button>
            <div v-if="isOpen" class="absolute right-0 mt-2 py-2 w-48 bg-white rounded-lg shadow-xl">
                <a href="#" class="block px-4 py-2 text-gray-800 hover:bg-indigo-500 hover:text-white">Account settings</a>
                <a href="#" class="block px-4 py-2 text-gray-800 hover:bg-indigo-500 hover:text-white">Support</a>
                <a href="#" class="block px-4 py-2 text-gray-800 hover:bg-indigo-500 hover:text-white">Sign out</a>
            </div>
        </div>
        </template>

        <script>
        export default {
            data() {
                return {
                isOpen: false
                }
            },
            created() {
                const handleEscape = (e) => {
                    if (e.key === 'Esc' || e.key === 'Escape') {
                        this.isOpen = false;
                    }
                }

                document.addEventListener('keydown', handleEscape);

                this.$once('hook:beforeDestroy', () => {
                    document.removeEventListener('keydown', handleEscape)
                });
            }
        }
        </script>
        ```
    * **Same Dropdown Component as Above Except For Mobile and In a Navbar**
        
        Navbar.vue
        ```js
        <template>
            <header class="bg-gray-900 sm:flex sm:justify-between sm:items-center sm:px-4 sm:py-3">
                <div class="flex items-center justify-between px-4 py-3 sm:p-0">
                    <div>
                        <img class="h-8" src="/img/logo-inverted.svg" alt="Workcation">
                    </div>
                    <div class="sm:hidden">
                        <button @click="isOpen = !isOpen" type="button" class="block text-gray-500 hover:text-white focus:text-white focus:outline-none">
                            <svg class="h-6 w-6 fill-current" viewBox="0 0 24 24">
                                <path v-if="isOpen" fill-rule="evenodd" d="M18.278 16.864a1 1 0 0 1-1.414 1.414l-4.829-4.828-4.828 4.828a1 1 0 0 1-1.414-1.414l4.828-4.829-4.828-4.828a1 1 0 0 1 1.414-1.414l4.829 4.828 4.828-4.828a1 1 0 1 1 1.414 1.414l-4.828 4.829 4.828 4.828z"/>
                                <path v-if="!isOpen" fill-rule="evenodd" d="M4 5h16a1 1 0 0 1 0 2H4a1 1 0 1 1 0-2zm0 6h16a1 1 0 0 1 0 2H4a1 1 0 0 1 0-2zm0 6h16a1 1 0 0 1 0 2H4a1 1 0 0 1 0-2z"/>
                            </svg>
                        </button>
                    </div>
                </div>
                <nav :class="isOpen ? 'block' : 'hidden'" class="sm:block">
                    <div class="px-2 pt-2 pb-4 sm:flex sm:p-0">
                        <a href="#" class="block px-2 py-1 text-white font-semibold rounded hover:bg-gray-800">List your property</a>
                        <a href="#" class="mt-1 block px-2 py-1 text-white font-semibold rounded hover:bg-gray-800 sm:mt-0 sm:ml-2">Trips</a>
                        <a href="#" class="mt-1 block px-2 py-1 text-white font-semibold rounded hover:bg-gray-800 sm:mt-0 sm:ml-2">Messages</a>
                        <AccountDropdown class="hidden sm:block sm:ml-6"/>
                    </div>
                    <div class="px-4 py-5 border-t border-gray-800 sm:hidden">
                        <div class="flex items-center">
                            <img class="h-8 w-8 border-2 border-gray-600 rounded-full object-cover" src="https://images.unsplash.com/photo-1487412720507-e7ab37603c6f?ixlib=rb-1.2.1&ixid=eyJhcHBfaWQiOjEyMDd9&auto=format&fit=crop&w=256&q=80" alt="Your avatar">
                            <span class="ml-3 font-semibold text-white">Jane Doe</span>
                        </div>
                        <div class="mt-4">
                            <a href="#" class="block text-gray-400 hover:text-white">Account settings</a>
                            <a href="#" class="mt-2 block text-gray-400 hover:text-white">Support</a>
                            <a href="#" class="mt-2 block text-gray-400 hover:text-white">Sign out</a>
                        </div>
                    </div>
                </nav>
            </header>
        </template>

        <script>
        import AccountDropdown from './AccountDropdown'

        export default {
            components: {
                AccountDropdown,
            },
            data() {
                return {
                    isOpen: false,
                }
            },
        }
        </script>
        ```

        AccountDropdown.vue
        ```js
        <template>
            <div class="relative">
                <button @click="isOpen = !isOpen" class="relative z-10 block h-8 w-8 rounded-full overflow-hidden border-2 border-gray-600 focus:outline-none focus:border-white">
                    <img class="h-full w-full object-cover" src="https://images.unsplash.com/photo-1487412720507-e7ab37603c6f?ixlib=rb-1.2.1&ixid=eyJhcHBfaWQiOjEyMDd9&auto=format&fit=crop&w=256&q=80" alt="Your avatar">
                </button>
                <button v-if="isOpen" @click="isOpen = false" tabindex="-1" class="fixed inset-0 h-full w-full bg-black opacity-50 cursor-default"></button>
                <div v-if="isOpen" class="absolute right-0 mt-2 py-2 w-48 bg-white rounded-lg shadow-xl">
                    <a href="#" class="block px-4 py-2 text-gray-800 hover:bg-indigo-500 hover:text-white">Account settings</a>
                    <a href="#" class="block px-4 py-2 text-gray-800 hover:bg-indigo-500 hover:text-white">Support</a>
                    <a href="#" class="block px-4 py-2 text-gray-800 hover:bg-indigo-500 hover:text-white">Sign out</a>
                </div>
            </div>
        </template>

        <script>
        export default {
            data() {
                return {
                    isOpen: false
                }
            },
            created() {
                const handleEscape = (e) => {
                    if (e.key === 'Esc' || e.key === 'Escape') {
                        this.isOpen = false
                    }
                }
                document.addEventListener('keydown', handleEscape);

                this.$once('hook:beforeDestroy', () => {
                    document.removeEventListener('keydown', handleEscape)
                });
            }
        }
        </script>
        ```