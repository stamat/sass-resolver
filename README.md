# sass-resolver

A custom [`FileImporter`](https://sass-lang.com/documentation/js-api/interfaces/fileimporter/) for Dart Sass that adds include path resolution, similar to the deprecated `includePaths` option.

## Why?

Dart Sass removed the `includePaths` option. This package provides a `FileImporter` that restores that behavior, allowing you to resolve `@use` and `@import` from directories like `node_modules`.

## Install

```bash
npm install sass-resolver
```

## Usage

```js
import { compile } from 'sass'
import { sassResolver } from 'sass-resolver'

const result = compile('src/styles/main.scss', {
  importers: [sassResolver('node_modules')]
})
```

Multiple include paths:

```js
const result = compile('src/styles/main.scss', {
  importers: [sassResolver(['node_modules', 'vendor/styles'])]
})
```

Works with `compileString` too:

```js
import { compileString } from 'sass'
import { sassResolver } from 'sass-resolver'

const result = compileString('@use "my-package";', {
  importers: [sassResolver('node_modules')]
})
```

## Resolution algorithm

Given `@use "my-pkg/core/config"` and an include path of `node_modules`, the resolver tries the following in order:

1. **Directory with index file** - `node_modules/my-pkg/core/config/index.{sass,scss,css}`
2. **Directory with `package.json`** - reads `sass`, `scss`, `style`, `css`, or `main` fields
3. **Exact file** - `node_modules/my-pkg/core/config`
4. **File with extension** - `node_modules/my-pkg/core/config.{sass,scss,css}`
5. **Underscore partial** - `node_modules/my-pkg/core/_config.{sass,scss,css}`
6. **Package-relative path** - resolves the path relative to the package entry point defined in `package.json`

The first match is returned as a `file:` URL. If nothing matches across all include paths, `null` is returned and Sass falls through to its default resolution.

## `package.json` fields

When resolving a package root (e.g. `@use "my-pkg"`), the resolver reads `package.json` and checks the following fields in order:

- `sass`
- `scss`
- `style`
- `css`
- `main`

## API

### `sassResolver(includePaths)`

Returns a Dart Sass [`FileImporter`](https://sass-lang.com/documentation/js-api/interfaces/fileimporter/) object.

- **`includePaths`** `string | string[]` - One or more directories to search for imports. Throws if not provided or not a string/array.

### `resolvePath(url, includePath)`

Lower-level function that resolves a single URL against a single include path. Returns a `URL` object or `null`.

## License

MIT
