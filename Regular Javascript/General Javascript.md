# General Javascript Notes & Examples

## Notes
* ### First

## Examples
* ### Using the `IntersectionObserver` Class
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

