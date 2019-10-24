# General Javascript Notes & Examples

## Notes
* ### First

## Examples
* ### <a id="insersection-observer">Using the `IntersectionObserver` Class</a>
    * This class can be used to observe an element on a page and then fire an event when that element comes into view in the viewport
    * A good example of this working in a real world scenario is with the [Stick Nav Bar](../Vue.js/Vue%202%20Examples.md#sticky-nav-bar) example in [Vue 2 Examples](../Vue.js/Vue%202%20Examples.md) document
    * Usage Example
        * Assign a `IntersectionObserver` object to the `observer` variable that will accept the entries when the observer is fired.
            * `InsersectionObserver` can accept an options object as the second argument. In this case we only want the observer to fire when the `intersectionRation` threshold of the entry crosses .25 or 25% visible.
        * Register the element you want to observe by calling the `observe` method and pass in the element.
            ```js
            let nav = document.querySelector('nav');
            let observer = new IntersectionObserver(entries => {
                if (entries[0].isIntersecting) {
                    nav.classList.add('is-floating');
                } else {
                    nav.classList.remove('is-floating');
                }
            }, {
                threshold: .25
            });
            observer.observe(document.querySelector('#featured'));
            ```
            You can group the process together so it is cached by wrapping everything in a generic function that accepts the element to observer and then call it and pass in the element.
            ```js
            (function (nav) {
                let observer = new IntersectionObserver(entries => {
                    if (entries[0].isIntersecting) {
                        nav.classList.add('is-floating');
                    } else {
                        nav.classList.remove('is-floating');
                    }
                }, {
                    threshold: .25
                });
                observer.observe(document.querySelector('#featured'));
            })(document.querySelector('nav'));
            ```
        * When the element is observed, here is the entries value being sent to the function as displayed by `console.log(entries);`.
            ```
            [IntersectionObserverEntry]
                0: IntersectionObserverEntry
                    boundingClientRect: DOMRectReadOnly
                        bottom: 1942
                        height: 923
                        left: 0
                        right: 1342
                        top: 1019
                        width: 1342
                        x: 0
                        y: 1019
                        __proto__: DOMRectReadOnly
                    intersectionRatio: 0.20368364453315735
                    intersectionRect: DOMRectReadOnly
                        bottom: 1207
                        height: 188
                        left: 0
                        right: 1342
                        top: 1019
                        width: 1342
                        x: 0
                        y: 1019
                        __proto__: DOMRectReadOnly
                    isIntersecting: false
                    isVisible: false
                    rootBounds: DOMRectReadOnly
                        bottom: 1207
                        height: 1207
                        left: 0
                        right: 1342
                        top: 0
                        width: 1342
                        x: 0
                        y: 0
                        __proto__: DOMRectReadOnly
                    target: section#featured.px-10
                    time: 227.97499998705462
            ```
* ### Lazy load images on a page using the new IntersectionObserver class
    * This example uses a package called [Lozad.js](https://www.npmjs.com/package/lozad) or the [Github repo](https://github.com/ApoorvSaxena/lozad.js)
    * Lozad offers a much more efficient way to lazy load images on a page while the user scrolls.
    * Once it is installed using `npm i lozad` then you need to do is add a `lozad` class and a `data-src` attribute to the img tag.
        ```html
        <img class="lozad" data-src="image.png" />
        ```
    * After that instatiate Lozad by using the following:
        ```js
        const el = document.querySelector('img');
        const observer = lozad(el); // passing a `NodeList` (e.g. `document.querySelectorAll()`) is also valid
        observer.observe();
        ```
    * You can also add custom options (like `threshold` as seen in the [IntersectionObserver](#intersection-observer) example above)
        ```js
        const observer = lozad('.lozad', {
            rootMargin: '10px 0px', // syntax similar to that of CSS Margin
            threshold: 0.1 // ratio of element convergence
        });
        ```
    * If you need to lazy load dynamically added images, you can use the following:
        ```js
        const observer = lozad();
        observer.observe();

        // ... code to dynamically add elements
        observer.observe(); // observes newly added elements as well
        ```
    * Other ways to implement Lozad are availabe in the [documentation](https://github.com/ApoorvSaxena/lozad.js).

