# [gulp](https://github.com/gulpjs/gulp/)-sri
[![npm version](http://img.shields.io/npm/v/gulp-sri.svg)](https://npmjs.org/package/gulp-sri)
[![Build Status](https://travis-ci.org/mathisonian/gulp-sri.svg?branch=master)](https://travis-ci.org/mathisonian/gulp-sri)

[SubResource Integrity](http://www.w3.org/TR/SRI/) hashes generator for gulp. Code heavily borrowed from [gulp-buster](https://github.com/UltCombo/gulp-buster).

## What is subresource integrity?

A way to verify the contents of files after they have been delivered to the browser. There is a good blog post here: https://blog.cloudflare.com/an-introduction-to-javascript-based-ddos/


## Install

First off, [install gulp](https://github.com/gulpjs/gulp/blob/master/docs/getting-started.md).

Then install gulp-sri as a development dependency:

```
npm install --save-dev gulp-sri
```

## How to use

gulp-sri can be used standalone as part of a build task, or in conjunction with [gulp-watch](https://npmjs.org/package/gulp-watch) to update the hashes as the files are modified.

Example with gulp-watch `^1.0.5` and gulp-ruby-sass `^0.7.1` (compile, bust and watch for changes):

```js
var gulp = require('gulp'),
	watch = require('gulp-watch'),
	sass = require('gulp-ruby-sass'),
	sri = require('gulp-sri');

gulp.task('default', function() {
	var srcGlob = 'scss/*.scss';
	return gulp.src(srcGlob)
		.pipe(watch(srcGlob, function(files) {
			return files
				.pipe(sass())
				.pipe(gulp.dest('css'))
				.pipe(sri())           // pipe generated files into gulp-sri
				.pipe(gulp.dest('.'));  // output sri.json to project root
	}));
});
```

## Syntax

```none
<through stream> sri([options])
```

### Parameters

- `options` (object|string, optional): the configuration options object. Passing `options` as a string is treated as `{ fileName: options }`.

- `options.fileName` (string, optional): the output filename. Defaults to `'sri.json'`.

- `options.algorithms` (string|function, optional): the hashing algorithms to be used. Defaults to `['sha256']`. This option is passed directly to the [sri-toolbox](https://github.com/neftaly/npm-sri-toolbox).

- `options.type` (string, optional): Content type string to be included in the output hash. This option is passed directly to the [sri-toolbox](https://github.com/neftaly/npm-sri-toolbox).

- `options.transform` (function, optional): allows mutating the hashes object, or even creating a completely new data structure, before passing it to the `formatter`. It takes a copy of the hashes object (a plain object in the `filePath: hash` format) as the first argument and must return a value compatible with the `formatter` option. Defaults to passing through the hashes object.

- `options.formatter` (function, optional): the function responsible for serializing the hashes data structure into the string content of the output file. It takes the value returned from the `transform` function as the first argument and must return a string. Defaults to `JSON.stringify`.

**Note:** all of the options which accept a function can be run asynchronously by returning a promise (or *thenable*). If the given option has a return value constraint, the constraint will still be applied to the promise's fulfillment value.

## Integrating with Web applications

gulp-sri is language-agnostic, thus this part relies on you and your specific use case. By default, gulp-sri generates a JSON file in the following format:

```js
{
	"path/to/file/relative/to/project/root/filename.ext": "srihash",
	//other entries
}
```

Integration can be easily achieved on any language which supports JSON parsing, in either back-end or front-end. See the [Implementations page](https://github.com/UltCombo/gulp-buster/blob/master/IMPLEMENTATIONS.md) for examples and existing solutions for your language of choice.

**Note:** The output file contents' data structure and format can be customized through the [configuration options](#parameters). This enables gulp-sri to output the cache buster hashes file in a suitable native data structure for your target language, as well as allowing to mutate the paths and hashes to suit your specific needs.

## Architecture

When gulp-sri is initialized, it creates an empty object which serves as a cache for the generated hashes. Generated hashes are then grouped by the `options.fileName` parameter. That means, piping two different streams into `sri('foo.json')` will merge both of those streams' files' hashes into the same output file. This approach's main pros are:

- Allows piping only modified files into gulp-sri, the other hashes are retrieved from the cache when generating the output file;
- Deleted files' hashes are automatically cleaned on startup, as the hashes cache object starts empty on every startup;
- Although this feature is very similar to [gulp-remember](https://github.com/ahaurw01/gulp-remember), gulp-sri's hashes cache is much more efficient. Using gulp-remember would cause all of the files that have ever went through the stream to be re-hashed whenever piping any new file, unlike gulp-sri's hashes cache which allows hashing only the files that are piped into gulp-buster.

There are close to no cons, the only notable drawback is that all the to-be-hashed files must be piped into gulp-sri (preferably at startup) before it generates the output file with all hashes that you'd expect. A feature to allow inputting an already-generated hashes file was considered in order to avoid having to pipe all the to-be-hashed files at startup, but that seems to bring more cons than pros -- the auto-cleanup of deleted files' hashes would no longer happen, outdated hashes could stay in the output hashes file if the to-be-hashed files were edited while gulp was not running, and finally it'd also be incompatible with the `transform` and `formatter` features.
