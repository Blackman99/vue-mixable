# `vue-comixable`

> Turn Mixins into composables to reuse them in Composition API

## Quick Intro

```js
// given an existing mixin such as this:
export const messageMixin  = {
    data() {
        return {
            msg: 'Hello World'
        }
    },
    computed: {
        loudMsg() {
            return this.capitalize(this.msg) + '!!!1eleven!'
        }
    },
    methods: {
        capitalize(value) { return value.toUpperCase() }
    } 
}

// we can create a composable from it with a single function call
import { createComposableFromMixin } from 'vue-comixable'
export const useMessage = createComposableFromMixin(messageMixin)
```

This composable can then be used in  `setup()`/ `<script setup>`:

```html
<script setup>
const {
    msg, // ref
    loudMsg, // computed() ref
    capitalize // function
} = useLoudMessage()
</script>
```

## Usecase

This library is primarily useful for developers trying to migrate a Options-API codebase using Mixins for code sharing to Composition API using composables for code sharing.

One of the challenges in such a migration is that one often cannot rewrite a mixin into a composable and replace all of that mixin's usage instances in the app at once, epsecially when mixins depend on one another, which is often the case in larger code-bases.

This is where `vue-comixable` can help: The team can keep all their mixins for the time of the migration, but convert each of them into composables with one line of code. You get have your cake, and eat it to, in a way. 

Then they can migrate individual components from the mixin to the composable at their own pace, and once the migration is done, they can rewrite the mixin into a proper standalone composable and finally remove the mixin from your codebase.


## Installation

```bash
npm install vue-comixable
```

## Usage Notes

### Supported APIs

`vue-comixable` provides full support for mixins that use the following Options APIs

* `data`
* `computed`
* `methods`
* `watch`
* `provide`
* `inject`
* `props` (see Note in the next section)
* `emits` (see Note in the next section)

Options with no direct support (currently):

* `inheritAttrs`
* `components`
* `directives`
* `name`

If you use any of the above options, you would have to set them manually in any component that uses the generated composable instead of the original mixin.

### `props` & `emits`  options

Mixins can contain props definitions. Composables cannot, as they are functions invoked during component initialization (in `setup()`, at which point props must have been defined already.

`vue-comixable` solves with in the following way:

```js
const mixin = {
    props: ['modelValue', 'age', 'street', 'city'],
    emits: ['modelValue:update', 'certified']
    // ...and other mixin content, i.e.:
    data: () => ({
        //...
    })
}

export const usePerson = createComposableFromMixin(mixin)
// props and emits options will be available 
// as properties on the composable function(!)
usePerson.props // => ['modelValue', 'age', 'street', 'city']
usePerson.emits // => ['modelValue:update', 'certified']
```
Usage
```js
import { usePerson } from '...'

export default defineComponent({
    props: ['firstname', 'lastname', ...usePerson.props],
    emits: usePerson.emits,
    setup(props, { emit }) {
        const person = usePerson()

        return {

        }
    }
})

```
### Shape of the composable's return value

The shape of the return value is essentially a flattened version of the mixins `data`, `computed` and `methods` properties, with `data` and `computed` being `ref()`'s. All other supported properties (lifecylces, `watch`) have nothing to expose externally.

```js
const mixin = {
    data: () =>({
        a: 'A',
        b: 'B',
    }),
    computed: {
        c() { return this.A },
        d() { return this.B }
    },
    methods: {
        e() {
            return callSomething(this.a, this.c)
        }
    }
}

const useComposable = createComposableFromMixin(mixin)
```
would be turned into:
```js

const {
    a, // ref('A')
    b, // ref('B')
    c, // computed ref 
    d, // computed ref
    e, // normal function
} = useComposable()
```

### Feature Roadmap

- [ ] Nested Mixins
- [x] Mixins that implicitly depend on APIs from other mixins
- [x] Exclude specific properties from composables return value (essentially making some mixin properties private in the composable)
- [ ] 

Out of scope / not planned

- [ ] mixins with circular dependencies on one another.

## Caveats

### `this.$watch()` in created

creating a watcher imperatively in `created` will likely not work as expected, because in the created composable, that hooks is run before `setup()` returns, so any data properties declared in the mixin/composable will be missing on `this`.

Possible workarounds:

- use the normal `watch:`option
- create the watcher in `beforeMount`.

## Typescript Support

> Typescript support is still considered unstable as we plan on improving the types, possibly introduction breaking changes to the types.

## Developer Instructions


### Compile and Minify for Production, create Type Declarations

```sh
pnpm build
```

### Run Unit Tests with [Vitest](https://vitest.dev/)

```sh
pnpm test:unit
```

### Lint with [ESLint](https://eslint.org/)

```sh
pnpm lint
```