# Inertia JS Notes

## Notes from the Build Modern Laravel Apps Using Inertia.js series

* ### When using Vue with blade
    * If you are echoing out something in a blade file, you need to preface it with a `@` symbol. i.e. `<p>@{{ message }}</p>`
    * You can also use `v-text` in this case to bypass any conflict. i.e. `<p v-text="message"></p>`
    * You can also wrap a lot of code in the Laravel `verbatim` tag. `@verbatim <p>{{ message }}</p> @endverbatim`
* ### Vue Instance:
    * When declaring the root instance for Vue, you need to make sure that all components are registered before the root instance. The root instance should be at the bottom of your .js file.
    * `new Vue({  });`
        * `el: '#rootElement"` Element the Vue instance is bound to
        * `data: {  }` Object containing various data values
        * `mounted() {  }` This method is called when the Vue instance is mounted and ready to be used. Like `ready` in `jQuery`
        * `methods: {  }` Object containing any custom methods that the instance can use.
        * `computed: {  }` An object containing methods where properties are computed before they are available.
        * `created() {  }` used with components and is sort of like mounted except this method is run when the component is created.
        * `filters: {  }` An object containing methods where specific filters can be applied. Consider then as functions that can be stacked. See section on filters
        * `components: { componentName }` Should be used to declare the use of a component in the local scope of an instance or another component. Make sure that the component is imported into this instance as well. `"componentName" from "../components/componentName.vue"`
        * `watch: {  }` Can be used to watch for changes to certain properties.
* ### Event tips:
    * When using event listeners (`@click` or `v-on`) then you can get info about the event by using the `$event` property.
        * For example you are wanting to get the value of an input as someone is typing into it and update a data variable declared in the instance (`coupon`). `<input type="text" @input="coupon = $event.target.value">`
            * `target` is used to specify itself.
* ### Event Modifiers:
    * Sometimes when an event is fired on an element, the event needs to be modified before execution
    * Can be formatted as `<form method="post" @submit.prevent="onSubmit">...` When a `submit` event is received and before the `onSubmit` method is called, the modifier is applied.
    * Modifiers can be chained together by just adding another `.modifier`.
    * Modifiers can also be used by themselves without a method call. Just leave off the ="methodName"
    * Modifiers:
        * `.prevent` - same as `event.preventDefautl()`.
        * `.stop` - stops the event propagation
        * `.capture` - use capture mode when adding the event listener
        * `.self` - only trigger handler if ` ` is the element itself
        * `.once` - the click event will be triggered at most once