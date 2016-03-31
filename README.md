# gulp-rev-urlhash

Static asset revisioning by create manifest.json, you can get assets uri like xxx.js?v=hs27skvi from manifest.json

It comes from [gulp-rev](https://www.npmjs.com/package/gulp-rev), and **this plugin will not change your dist filename, but add url query v=XXXXXXXX in the manifest.json instead**.

You can use manifest.json to get files' latest revisioned name.

Make sure to set the files to [never expire](http://developer.yahoo.com/performance/rules.html#expires) for this to have an effect.


## Install

```
$ npm install --save-dev gulp-rev-urlhash
```


## Usage

```js
var gulp = require('gulp');
var revUrlhash = require('gulp-rev-urlhash');

gulp.task('default', function () {
	return gulp.src('src/*.css')
		.pipe(revUrlhash())
		.pipe(gulp.dest('dist'));
});
```


## API

### revUrlhash()

### revUrlhash.manifest([path], [options])

#### path

Type: `string`
Default: `"rev-manifest.json"`

Manifest file path.

#### options

##### base

Type: `string`
Default: `process.cwd()`

Override the `base` of the manifest file.

##### cwd

Type: `string`
Default: `process.cwd()`

Override the `cwd` (current working directory) of the manifest file.

##### merge

Type: `boolean`
Default: `false`

Merge existing manifest file.


### Original path

Original file paths are stored at `file.revOrigPath`. This could come in handy for things like rewriting references to the assets.


### Asset hash

The hash of each rev'd file is stored at `file.revHash`. You can use this for customizing the file renaming, or for building different manifest formats.


### Asset manifest

```js
var gulp = require('gulp');
var revUrlhash = require('gulp-rev-urlhash');

gulp.task('default', function () {
	// by default, gulp would pick `assets/css` as the base,
	// so we need to set it explicitly:
	return gulp.src(['assets/css/*.css', 'assets/js/*.js'], {base: 'assets'})
		.pipe(gulp.dest('build/assets'))  // copy original assets to build dir
		.pipe(revUrlhash())
		.pipe(gulp.dest('build/assets'))  // write rev'd assets to build dir
		.pipe(revUrlhash.manifest())
		.pipe(gulp.dest('build/assets')); // write manifest to build dir
});
```

An asset manifest, mapping the original paths to the revisioned paths, will be written to `build/assets/rev-manifest.json`:

```json
{
	"css/unicorn.css": "css/unicorn.css?v=d41d8cd98f",
	"js/unicorn.js": "js/unicorn.js?v=273c2cin3f"
}
```

By default, `rev-manifest.json` will be replaced as a whole. To merge with an existing manifest, pass `merge: true` and the output destination (as `base`) to `revUrlhash.manifest()`:

```js
var gulp = require('gulp');
var revUrlhash = require('gulp-rev-urlhash');

gulp.task('default', function () {
	// by default, gulp would pick `assets/css` as the base,
	// so we need to set it explicitly:
	return gulp.src(['assets/css/*.css', 'assets/js/*.js'], {base: 'assets'})
		.pipe(gulp.dest('build/assets'))
		.pipe(revUrlhash())
		.pipe(gulp.dest('build/assets'))
		.pipe(revUrlhash.manifest({
			base: 'build/assets',
			merge: true // merge with the existing manifest (if one exists)
		}))
		.pipe(gulp.dest('build/assets'));
});
```

You can optionally call `revUrlhash.manifest('manifest.json')` to give it a different path or filename.


## Sourcemaps and `gulp-concat`

Because of the way `gulp-concat` handles file paths, you may need to set `cwd` and `path` manually on your `gulp-concat` instance to get everything to work correctly:

```js
var gulp = require('gulp');
var revUrlhash = require('gulp-rev-urlhash');
var sourcemaps = require('gulp-sourcemaps');
var concat = require('gulp-concat');

gulp.task('default', function () {
	return gulp.src('src/*.js')
		.pipe(sourcemaps.init())
		.pipe(concat({path: 'bundle.js', cwd: ''}))
		.pipe(revUrlhash())
		.pipe(sourcemaps.write('.'))
		.pipe(gulp.dest('dist'));
```


## Streaming

This plugin does not support streaming. If you have files from a streaming source, such as browserify, you should use [gulp-buffer](https://github.com/jeromew/gulp-buffer) before `gulp-rev-urlhash` in your pipeline:

```js
var gulp = require('gulp');
var browserify = require('browserify');
var source = require('vinyl-source-stream');
var buffer = require('gulp-buffer');
var revUrlhash = require('gulp-rev-urlhash');

gulp.task('default', function () {
	return browserify('src/index.js')
		.bundle({debug: true})
		.pipe(source('index.min.js'))
		.pipe(buffer())
		.pipe(revUrlhash())
		.pipe(gulp.dest('dist'))
});
```
