# ts_library_invalid_import

To reproduce the issue, move to the `webpack` directory and run `yarn build`.

Webpack should fail to build with
```
BREAKING CHANGE: The request './src/foo' failed to resolve only because it was resolved as fully specified
(probably because the origin is strict EcmaScript Module, e. g. a module with javascript mimetype, a '*.mjs' file, or a '*.js' file where the package.json contains '"type": "module"').
The extension in the request is mandatory for it to be fully specified.
Add the extension to the request.
```

This is expected since `.mjs` files [need to have a fully specified import for relative imports, including the file extension](https://nodejs.org/api/esm.html#mandatory-file-extensions). If you look in `node_modules/example-package/index.mjs`, you'll see `import { foo } from './src/foo';`, which does not include the file extension.

I've tried adding the `.mts` file extension in the `.ts` file in `monorepo/index.ts`, but that makes compilation fail with
```
index.ts:1:21 - error TS2691: An import path cannot end with a '.mts' extension. Consider importing './src/foo' instead.

1 import { foo } from './src/foo.mts';
                      ~~~~~~~~~~~~~~~
```

This is expected since typescript does not rewrite import statements, however, what is unexpected is that it still fails when the extension is `.mjs`:
```
index.ts:1:21 - error TS2307: Cannot find module './src/foo.mjs' or its corresponding type declarations.

1 import { foo } from './src/foo.mjs';
                      ~~~~~~~~~~~~~~~
```


Perhaps ts_library could automatically add the `.mjs` extension for these imports, although that probably ranges between really difficult to impossible.
