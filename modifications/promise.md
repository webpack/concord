# `promise`

Exports an ES6-like promise, which eventually resolve (or has already resolved) to the exports.

The module system can decide whether to load the module on demand or not. It'll prefer on-demand loading.

## example

```
{example}
```

## support

| {implementation} |
|------------------|
| {version}        |

## features

### `+eager`

On demand loading is not allowed. This fail to compile if the module system only supports on-demand loading.

| {implementation} |
|------------------|
| {version}        |

### `+lazy`

On demand loading is enforced. This fail to compile if the module system doesn't support on-demand loading.

| {implementation} |
|------------------|
| {version}        |
