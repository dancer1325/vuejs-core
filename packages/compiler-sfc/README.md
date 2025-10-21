# @vue/compiler-sfc

* == low level utilities
* allows
  * compile Vue Single File Components (SFC)
* | 3.2.13+,
  * 👀MAIN `vue` package's dependency👀
    * == `vue/compiler-sfc`
    * == use -- through --- MAIN `vue`
* uses as standalone |
  * write a bundler OR module system's plugin / transform /
    * compiles Vue Single File Components (SFCs) -- into -- JavaScript
  * [vue-loader](https://github.com/vuejs/vue-loader)
  * [@vitejs/plugin-vue](https://github.com/vitejs/vite-plugin-vue/tree/main/packages/plugin-vue)

## API

* low-level
  * Reason: 🧠integrate Vue SFCs | build system🧠
- Separate hot-module replacement (HMR) -- for -- script, template and styles
  - ==
    - template updates should NOT reset component state
    - style updates should be performed WITHOUT component re-render
- tool's plugin system can be used -- for -- pre-processor handling
  * _Example:_ `<style lang="scss">` should be processed -- by the -- corresponding webpack loader
- | SOME cases,
  - ❌transformers of each block do NOT share the same execution context❌
    * _Example:_ + `thread-loader` or other parallelized configurations -> `vue-loader`'s template sub-loader may NOT have access to the full SFC and its descriptor

* 's goal
  * facade module / imports the component's individual blocks  (-- via -- different query strings)

```
                                  +--------------------+
                                  |                    |
                                  |  script transform  |
                           +----->+                    |
                           |      +--------------------+
                           |
+--------------------+     |      +--------------------+
|                    |     |      |                    |
|  facade transform  +----------->+ template transform |
|                    |     |      |                    |
+--------------------+     |      +--------------------+
                           |
                           |      +--------------------+
                           +----->+                    |
                                  |  style transform   |
                                  |                    |
                                  +--------------------+
```

Where the facade module looks like this:

```js
// main script
import script from '/project/foo.vue?vue&type=script'
// template compiled to render function
import { render } from '/project/foo.vue?vue&type=template&id=xxxxxx'
// css
import '/project/foo.vue?vue&type=style&index=0&id=xxxxxx'

// attach render function to script
script.render = render

// attach additional metadata
// some of these should be dev only
script.__file = 'example.vue'
script.__scopeId = 'xxxxxx'

// additional tooling-specific HMR handling code
// using __VUE_HMR_API__ global

export default script
```

### High Level Workflow

1. In facade transform, parse the source into descriptor with the `parse` API and generate the above facade module code based on the descriptor;

2. In script transform, use `compileScript` to process the script. This handles features like `<script setup>` and CSS variable injection. Alternatively, this can be done directly in the facade module (with the code inlined instead of imported), but it will require rewriting `export default` to a temp variable (a `rewriteDefault` convenience API is provided for this purpose) so additional options can be attached to the exported object.

3. In template transform, use `compileTemplate` to compile the raw template into render function code.

4. In style transform, use `compileStyle` to compile raw CSS to handle `<style scoped>`, `<style module>` and CSS variable injection.

Options needed for these APIs can be passed via the query string.

For detailed API references and options, check out the source type definitions. For actual usage of these APIs, check out [@vitejs/plugin-vue](https://github.com/vitejs/vite-plugin-vue/tree/main/packages/plugin-vue) or [vue-loader](https://github.com/vuejs/vue-loader/tree/next).
