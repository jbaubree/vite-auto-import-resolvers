# vite-auto-import-resolvers

The vite resolvers of [unplugin-auto-import]((https://github.com/antfu/unplugin-auto-import)) mainly deals with the `API` of the `vite` project itself, which is automatically imported on demand.


<br />


## README 🦉

[简体中文](./README.md) | English

<br />
<br />

## Motation 🐇

In order to automatically import the `API` of modules in the specified directory on demand.

<br />
<br />

## Basic Usage 🦖

1. install

```shell
npm i vite-auto-import-resolvers unplugin-auto-import -D

# pnpm 👇
# pnpm i vite-auto-import-resolvers unplugin-auto-import -D

# yarn 👇
# yarn add vite-auto-import-resolvers unplugin-auto-import -D
```

2. Configure plugins

```ts
// vite.config.js
// OR vite.config.ts

import { defineConfig } from 'vite'
import Vue from '@vitejs/plugin-vue'
import AutoImports from 'unplugin-auto-import/vite'
import { dirResolver, DirResolverHelper } from 'vite-auto-import-resolvers'

export default defineConfig({
    plugins: [
        Vue(),
        // This helper is required 👇
        DirResolverHelper(),
        AutoImports({
            // Set auto-imports.d.ts in your src directory
            dts: 'src/auto-imports.d.ts',
            imports: ['vue'],
            resolvers: [
                dirResolver()
            ]
        })
    ]
})
```
3. After that, the default export of modules under `src/composables` will be automatically imported as needed in the project

for example 👇

```ts
// src/composables/foo.ts

export default 100
```

```html
// src/App.vue
<script setup>
    console.log(foo) // log 100，And it is automatically introduced on demand
</script>

<template>
    Hello World!!
</template>
```

4. Type configuration (Abandoned, not required)

If your project is `ts`, your `tsconfig.json` should have the following configuration 👇

```json
{
    "compilerOptions": {
        // other configs
        "baseUrl": ".",
        "paths": {
            "/src/*": ["src/*"]
        }
    },
    // other configs
}
```

<br />

## Advanced 🦕

### Mandatory prefix or mandatory suffix

```ts
import { defineConfig } from 'vite'
import Vue from '@vitejs/plugin-vue'
import AutoImports from 'unplugin-auto-import/vite'
import { dirResolver, DirResolverHelper } from 'vite-auto-import-resolvers'

export default defineConfig({
    plugins: [
        Vue(),
        DirResolverHelper(),
        AutoImports({
            imports: ['vue'],
            resolvers: [
                dirResolver({ prefix: 'use' }), // prefix use
                dirResolver({
                    target: 'stores', // Target directory, The default is composables
                    suffix: 'Store' // suffix Store
                })
            ]
        })
    ]
})
```

So

-  `src/composables`, only modules starting with `use` will be loaded on demand
- `src/stores`, only modules ending in `Store` will be loaded on demand

for example 👇

```ts
// src/stores/counterStore.ts
const counter = ref(100)

export default () => {
    const inc = (v: number = 1) => (counter.value += v)
    return {
        inc,
        counter
    }
}
```

```html
<script setup lang="ts">
    // This is on demand
    const n = counterStore()
</script>

<template>
    <div @click="n.inc()">{{n.counter}}</div>
</template>
```

<br />
<br />

### include or exclude

```ts
import { defineConfig } from 'vite'
import Vue from '@vitejs/plugin-vue'
import AutoImports from 'unplugin-auto-import/vite'
import { dirResolver, DirResolverHelper } from 'vite-auto-import-resolvers'

export default defineConfig({
    plugins: [
        Vue(),
        DirResolverHelper(),
        AutoImports({
            imports: ['vue'],
            resolvers: [
                dirResolver({ 
                    prefix: 'use',
                    include: ['foo'], // foo modules are included even if they do not start with use
                    exclude: ['useBar'] // The useBar module will always be excluded
                }) 
            ]
        })
    ]
})
```

<br />
<br />

### root

```ts
import { resolve } from 'path'
import { defineConfig } from 'vite'
import Vue from '@vitejs/plugin-vue'
import AutoImports from 'unplugin-auto-import/vite'
import { dirResolver, DirResolverHelper } from 'vite-auto-import-resolvers'

export default defineConfig({
    plugins: [
        Vue(),
        DirResolverHelper(),
        AutoImports({
            imports: ['vue'],
            resolvers: [
                dirResolver({ 
                    root: '.' // The default is 'src'
                }) 
            ]
        })
    ]
})
```

<br />
<br />

### Other style path aliases (Abandoned, not required)

You may use other styles of path aliases in your project，for example `@`

Then you can configure it like this 👇

```ts
import { resolve } from 'path'
import { defineConfig } from 'vite'
import Vue from '@vitejs/plugin-vue'
import AutoImports from 'unplugin-auto-import/vite'
import { dirResolver, DirResolverHelper } from 'vite-auto-import-resolvers'

export default defineConfig({
    resolve: {
        alias: {
            // Change alias
           '@/': `${resolve(__dirname, 'src')}/`
        }
    },
    plugins: [
        Vue(),
        DirResolverHelper(),
        AutoImports({
            imports: ['vue'],
            resolvers: [
                dirResolver({ srcAlias: '@' }) // Set alias, default to /src/
            ]
        })
    ]
})
```

If you are a project of `ts`, `tsconfig.json` should be changed 👇

```json
{
    "compilerOptions": {
        // other configs
        "baseUrl": ".",
        "paths": {
            "@/*": ["src/*"]
        }
    },
    // other configs
}
```

<br />
<br />

### generate on-demand `API` presets

When using `unplugin auto imports`, you need to manage `imports` manually 👇

```ts
import { defineConfig } from 'vite'
import Vue from '@vitejs/plugin-vue'
import AutoImports from 'unplugin-auto-import/vite'

export default defineConfig({
    plugins: [
        Vue(),
        AutoImports({
            imports: ['vue', 'vue-router', 'pinia'] // 手动管理
        })
    ]
})
```

But sometimes you may need to change some dependencies, such as changing `pinia` to `vuex`. At this time, if the configuration is not changed, an error will occur. At the same time, if you set an uninstalled package, it will cause unnecessary performance consumption.

So you can 👇

```ts
import { defineConfig } from 'vite'
import Vue from '@vitejs/plugin-vue'
import AutoImports from 'unplugin-auto-import/vite'
import { AutoGenerateImports } from "vite-auto-import-resolvers"

export default defineConfig({
    plugins: [
        Vue(),
        AutoImports({
          imports: AutoGenerateImports() // Automatic management. Only when the corresponding package is loaded, the preset will be set automatically on demand
        })
    ]
})
```

<br />

#### Default support list

`include`

- vue
- pinia
- vuex
- vitest
- vue-i18n
- vue-router
- @vueuse/core
- @vueuse/head
- @nuxtjs/composition-api
- preact
- quasar
- react
- react-router
- react-router-dom
- svelte
- svelte/animate
- svelte/easing
- svelte/motion
- svelte/store
- svelte/transition
- vitepress
- vee-validate

<br />

#### exclude

```ts
import { defineConfig } from 'vite'
import Vue from '@vitejs/plugin-vue'
import AutoImports from 'unplugin-auto-import/vite'
import { AutoGenerateImports } from "vite-auto-import-resolvers"

export default defineConfig({
    plugins: [
        Vue(),
        AutoImports({
          imports: AutoGenerateImports({
              exclude: ['pinia'] // Pinia will always be excluded
          }) 
        })
    ]
})
```

<br />
<br />

## Inspire 🐳

The `resolvers` comes from the `issue` discussion of `unplugin-auto-import` 👉 [How should I auto import composition functions](https://github.com/antfu/unplugin-auto-import/issues/76)。


<br />
<br />

## More 🐃

More project engineering practices，you can be see 👉 [tov-template](https://github.com/dishait/tov-template)

<br />
<br />

## License

Made with markthree

Published under [MIT License](./LICENSE).

<br />
