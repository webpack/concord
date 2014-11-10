# `rc-stylesheet`

A reference-counted stylesheet is exported. It exports two symbols: `ref` and `unref`. Both are functions.

`ref()` Increases the reference counter by one. If the reference counter is now one the css rules are added to the current document.

`unref()` Decreases the reference counter by one. If the reference counter is now zero the css rules are removed from the current document.

## example

``` javascript
import { ref, unref } from "abc";

// No css rules applied

ref();

// css rules are now applied

unref();

// css rules are no longer applied
```

## support

| {implementation} |
|------------------|
| {version}        |

## features

### `+{featureName}`

{featureDescription}

| {implementation} |
|------------------|
| {version}        |
