# moodle

A spec for modules. Non-final name: `moodle`

## Goals

* Reuseable modules that can be used independent of the build system.
* Allow to use different module types and describe dependencies between them. i. e.
  * code (javascript, coffeescript)
  * data (json, text, yaml)
  * stylesheet (css, less, stylus)
  * image (png, jpg), shader (glsl)
  * font (ttf, woff)
  * template (html, jade)
  * WebWorker (javascript, coffeescript)
  * on-demand loaded
* Dependencies for module types other than code should be allowed too. i. e.
  * stylesheets depend on images and fonts
  * templates depend on images
* Allow modules to be used in different environments. i. e.
  * browser (es3, es5, es6)
  * WebWorker
  * server (node.js)
  * browser plugin
  * standalone GUI app (node-webkit, atom-shell)
* Allow the application using the moodles to specify/override how different module types are handled. i. e.
  * images as DataUrl or as file
  * stylesheets inlined or as separate file
  * code/template/images minimized or with SourceMap
  * svg as code or converted into png
  * load module on demand or not
  * polyfill es5/es6 features or expect modern browser
* Don't require the module author for run any build system before publishing the module, but allow him to run it for generating assets that can be used without build system.
* Don't expect from the build system to handle all module types.
* Allow the module author to conditionally use modules depending on module type and feature support of the build system.
* Allow to specify custom module types.
* Don't make to build unnecessary complex to allow fast builds.
* A moodle should be useable after installing. The user (app author or other module authors) shouldn't need to configure special preprocessing.
* Easy to configure for app and module authors
* Easy to add moodle support for existing modules
* moodle shouldn't be a package manager. Instead it should reuse an existing package manager (i. e. npm).

## Existing stuff



### [browserify](http://browserify.org/) style modules

browserifys approach is to use only node.js-compatible modules plus code and transform this code for the used environment (here browser).

Example: loading HTML file

``` javascript
// package.json
"browserify": { "transforms": ["brfs"] }
```

``` javascript
var fs = require("fs");
var content = fs.readFileSync(__dirname + "/file.html", "utf-8");
```

A transform ([brfs](https://github.com/substack/brfs)) is used and the code is transformed to something like this:

``` javascript
var content = "<html><body><img src=\"image.png\"></body></html>";
```

#### positive

* prefect node.js compatibility
* nothing need to be configured on application level to use modules

#### negative

* the exact process of preprocessing need to be specified in the code 
  (i. e. `addStylesheet(compileLess(fs.readFileSync(__dirname + "/style.less", "utf-8")))`)
* the transform need to load and parse each code file to find dependencies.
  Multiple transforms means the code is parsed multiple times. Performance!
* You need to repeat yourself when using many files of the same module type
* browserify specific transform syntax, difficult to implement a new build system

#### open questions

* How to handle module types that are not supported in a server environment? (i. e. stylesheets, WebWorker)
* How to handle dependencies in module types other than javascript? (i. e. images in HTML)
* How to alternate the behaviour from the application? (i. e. emit images in separate files instead of inlined)



### [component](https://github.com/componentjs/component)

A per-module configuration file (`component.json`) defines dependencies. A predefined set of module types is supported (code (javascript), data (json), stylesheets (css), templates (raw text), images, runtime files). It's expected from the module author to compile all other module types into these module types.

``` javascript
// component.json
templates: ["file.html"]
```

``` javascript
var content = require("./file.html");
```

The build system adds `file.html` as string module to the bundle:

``` javascript
var content = require("./file.html");
// ---
"<html><body><img src=\"image.png\"></body></html>";
```

#### positive

* modules are reusable
* It's easy to implement new a build system, because only a predefined set of module types is supported
* You can override the default behaviour of a module type in the bundler (i. e. inline images with DataUrls)

#### negative

* only browser environment supported
* all modules types must be supported by the module system. Adding new module types is difficult as all build systems must support the new module type.
  This means it's difficult to extend.
* module authors must compile the modules when using other modules types than the supported ones.
* it's a pacakge manager and has it's own registry
  * you cannot use the module in node.js as it's installed in a separate folder, but you can publish the same module to both registries


  
# [webpack 1.x](http://webpack.github.io/)

In a pre-app configuration file (webpack.config.js) you configure loaders for files. Modules that use different module types need to tell the app author to add the right loaders to the configuration file. A 

``` javascript
// webpack.config.js
{ test: /\.html$/, loader: "html-loader" },
{ test: /\.png$/, loader: "file-loader" }
```

``` javascript
var content = require("./file.html");
```

The build system preprocess the required file with the specified loader and this will result in a generated module. The generated module can have dependencies.

``` javascript
var content = __webpack_require__(2);
// ---
module.exports = "<html><body><img src=\"" + __webpack_require__(3)__ + "\"></body></html>";
// ---
module.exports = "/path/6f87a6b7e188b838e842ab23.png"
```

#### positive

* module type behaviour is specified on application-level. i. e. it's easy to choose between image as DataUrl or file
* multiple loaders can be chained to create multiple levels of transformation

#### negative

* Using an module with modules types different than pure javascript requires configuration on application-level
* webpack specific loader syntax.



## moodle

### definitions

**type plus features**: This is an often used thing for specifying types. It combines a *basetype* with a list of additional *features*: `basetype+feature1+feature2+feature3`. An addition of an feature must behave equal to the thing without this feature if the feature isn't used. So if `a+b+c` is required `a+b+c+d` could be used instead and will behave similar to `a+b+c` or `a+b+c+e`. `*` as *basetype* will match every *basetype*.

**environment**: The target of the generated thing. There can be multiple environments. It's defined with a *type plus features*. Examples: `web+es5`, `node+es5+es6`


### package configuration

``` javascript
{
	"moodle": {
		"main": "./main",
		"[server] main": "./server-main",
		"extensions": ["", ".js", ".coffee", ".less"],
		"types": {
			"./lib/*.js": "object/javascript+commonjs+es5",
			"./**/*.less": "stylesheet/less",
			"[server] ./styles/*.less": "nothing/irrelevant",
			"*.coffee": "object/coffeescript",
			"*.{png|jpg|gif}": "url/image",
			"./package.json": "data+immutable/json",
			"socket.io": "promise+lazy/*",
			"socket.io/client": "object/javascript+amd"
		},
		"modules": {
			"config": "./default-config",
			"[web] config": "./web-config",
			"./file": "./moodle-file",
			"./ignored": false
		}
	}
}
```

#### file

The moodle configuration can be in the package manager configuration file.

Examples:

* `npm`: `package.json`
* `bower`: `bower.json`

Which package manager is used depends on the build system. A build system may support multiple package managers.

#### conditional keys

`main`, `extensions` and every key in the `types` object can be conditional. The author can specify this by prefixing a expression in brackets `[...]`.

Logical operations are possible via this syntax: and `[...][...]`, or `[...|...]`, not `[!...]`.

Spacing is optionally possible and ignored by the build system. Examples: `[ ... ] key`, `[ ... | ... ] [ ... ] key`.

Depending on the style of the expression, it's matched against the environment or the supported module types.

The only supported order of different versions of the same key is this one: the unconditional version must go first (if it exists), immediate followed by the conditional versions. Only last matched version is the used value of the key.

Conditional configuration can be applied in a preprocessing step to the configuration.

Examples:

* matched against the environment
  * `[server]`
  * `[web+es6]`
  * `[*+es6+my-custom-flag]`
* matched against the supported module types (containing a `/`)
  * `[url/image]`
  * `[data+immutable/json]`
  * `[promise+lazy/*]`

Example configuration:

``` javascript
"types": {
	"./large-file": "promise/*",
	"[web] [promise+lazy/*] ./large-file": "promise+lazy/*",
	"[server] ./large-file": "promise+eager/*",
}
```

By default `promise/*` is used for `large-file`, but if lazy-loading (`promise+lazy/*`) is supported this is used for `web` environment. For a `server` environment the lazy-loading is disabled (`promise+eager/*`). If building for `web` and `server` environment lazy-loading is disabled, because it's later specified in the configuration.

#### `moodle`

Every `moodle` option in scoped with this key. This ensures that moodle doesn't conflict with other configuration.

The only allowed keys in this object are: `main`, `extensions`, `types` and `modules`. When the build system find other keys it should fail assuming there is a newer moodle version, which is not supported by this build system.

#### `main`

When the request resolve to the main directory of this package (the directory of the configuration file), the content of the `main` field is resolved relative to the package directory (Meaning `./` is required for relative paths).

The `main` field from the moodle configuration defaults package manager configuration module entry point. i. e. `main` in `package.json` or `"./index"`.

#### `extensions`

An array of strings that are appended to requests that resolve inside the package. This is also used when resolving from another package into this package i. e. `module/file`.

This also defaults to the package manager default.

#### `modules`

This key is similar to the [browser field proposed by defunctzombie](https://gist.github.com/defunctzombie/4339901).

This object configuration replacements and aliases used for requests that resolve inside the package. When the *key* matches the request the *value* is used instead.

Relative *keys* (starting with `./`): The *key* is relative to the package directory and every request which matches that file is replaced. This means a key `./dir/file` can be matched by `module/dir/file`, `module/dir/file.js`, `./file` (when issued from `module/dir/other-file.js`).

Module *keys* (starting with a module name): It's matched when the exact same string is used from a file **inside** the package. Dependencies of the package are not affected.

string *value*: This request is resolved instead, it's resolved relative to the package directory.

`false` *value*: A empty module is used instead. It exports an empty object. It's assumes that this object isn't used.

Globbing can be used for the keys except the module part. `*`: matches anything but `/`, `**` matches anything, `{a,b,c}` matches `a`, `b` or `c`.

There is a special case when the key doesn't start with `./` but with a string containing `*` before the first `/`. This cannot be a module *key* because the module *key* is not allowed to contain `*` (no globbing for the module part). So in this case it is threaded as relative *key*, but it don't need to match the complete path. Examples: `*.js` matches every javascript file, `view-*/*` matches every file in a directory starting with `view-`, `case/*.js` matches `./test/case/xyz.js` but not `./testcase/xyz.js`.

Example:

* relative
  * `./file.html` matches the file `file.html` in the package directory
  * `./dir/abc.js` matches the file `abc.js` in the directory `dir` in the package directory
  * `./

#### `types`

The *keys* of the object are threaded equally to the *keys* in `modules` above.

The *value* specifies the module type. The entries are processed in order from top to bottom. When a entry is matched the module type is set to the *value*. `*` in the *value* is replaced with the previous *value*.

For a module *key* the previous value is obtained from the modules' moodle configuration.

The value defaults to `object/javascript` for `*.js` and to `object/json` for `*.json`.

Example:

``` javascript
"types": {
	"*.less": "stylesheet/less",
	"./styles/*": "promise/*",
	"module": "promise/*"
}
```

`a.less` has the module type `stylesheet/less`. `styles/b.less` has the module type `promise/stylesheet/less`. `c.css` failed because it has no module type. `styles/c.css` fails because while applying `"promise/*"` no previous module type is defined. `module` has i. e. the module type `promise/object/javascript`.

### module type

``` text
module-type
	modifications "/" export-type "/" content-type
	export-type "/" content-type

modifications
	modifications "/" modification-type
	modification-type

export-type
	type-with-features

content-type
	type-with-features

modification-type
	type-with-features

type-with-features
	type ( "+" feature )*
```

The **content type** explains the content of the file.

The **export type** explains what is expected to be exported from the module. This also summarizes side effects from loading this module.

The **modification type** explains how the original export is modified.

#### example content types

* `css` `less` `stylus`: a stylesheet written in css/less/stylus
* `javascript`: a module written in javascript
  * `+commonjs`: CommonJs is supported (`this` is `exports`)
  * `+amd`: AMD is supported
  * `+global`: `this` is `window`
  * `+es5` `+es6`
* `json`: a stringified json object
* `image`: an image
  * `+lossless`
  * `+lossy`
  * `+descriptive`
  * `+pixels`
* `font`: a font
* `irrelevant`: The content is not relevant

#### example export types

* `object`: any javascript object (or primive value) is exported
* `data`: data (no code: no functions or modified getter/setter)
  * `+immutable`: data will never be changed (allows optimizations like inlining)
* `stylesheet`: css rules are applied to the document
* `stylesheet-rc`: stylesheet wrapped in a reference-counted container.
* `text`: exported string
* `url`: an URL

#### example modification types

* `promise`: The export is wrapped in a promise (on demand loading is preferred)
  * `+eager`: thing is not loaded on demand
  * `+lazy`: thing is loaded on demand

