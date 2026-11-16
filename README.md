# wasm-module-shims

Shims for the JS API of the [WebAssembly ESM integration](https://github.com/WebAssembly/esm-integration), for module loaders implementing Wasm modules on engines without native support, as used by [es-module-shims](https://github.com/guybedford/es-module-shims).

A single self-contained ES module with no dependencies, written to be bundled directly from source:

```js
import { applyWasmPolyfills, compileStreaming, moduleImports } from 'wasm-module-shims';
```

## Global polyfills

`applyWasmPolyfills()` patches the `WebAssembly` global where the engine does not support the source phase natively, and is otherwise a no-op:

* `WebAssembly.Module` extends `AbstractModuleSource` per the [source phase imports](https://github.com/tc39/proposal-source-phase-imports) proposal, with modules from `new WebAssembly.Module`, `compile`, `compileStreaming`, `instantiate` and `instantiateStreaming` all branded as module sources.
* Modules compiled with the [`builtins: ['js-string']`](https://github.com/WebAssembly/js-string-builtins) or `importedStringConstants` compile options are provided with a JS implementation of those namespaces on instantiation, and reflect without those imports in `WebAssembly.Module.imports`. Modules compiled without these options are not affected. The implementation requires WasmGC support (Chrome 119+, Firefox 120+), below which imports resolve from the import object as normal.
* `WebAssembly.namespaceInstance(ns)` returns the `WebAssembly.Instance` registered for a Wasm module namespace.

`applyNamespaceInstance()` applies only the `WebAssembly.namespaceInstance` polyfill.

## Loader API

For loaders compiling and instantiating Wasm modules themselves, without relying on the global polyfills:

* `compileStreaming(response)`: compiles a module source with the ESM integration compile options (`jsStringCompileOptions`).
* `moduleImports(module)`, `moduleExports(module)`: import and export reflection, with the builtin namespaces removed from the imports.
* `hasBuiltinImports(module)`: whether the module still reflects builtin namespace imports, in which case `jsStringNamespaces` should be provided to its instantiation when `supportsJsStringBuiltins()` holds.
* `wasmInstances`: the `WeakMap` registry backing `WebAssembly.namespaceInstance`, to be populated by the loader with the instance for each Wasm module namespace.

## Testing

Tests run in the browser through [Chomp](https://chompbuild.com):

```
chomp test
```

## License

MIT
