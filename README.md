# Resolve URL gulp plugin

![status](https://secure.travis-ci.org//jkulhanek/gulp-resolve-url.svg?branch=master)

Gulp plugin that resolves relative paths in url() statements based on the original source file using sourcemaps.

Use in conjunction with the sass plugin and specify your asset `url()` relative to the `.scss` file in question.

This loader will use the source-map from the SASS compiler to locate the original `.scss` source file and fix relative url paths.


## Getting started

```bash

# via npm
npm install gulp-resolve-url --save-dev
```


## Usage

```
gulp.task('styles', function (){
    return gulp.src('./src/**/*.scss', { base: './src' })
        .pipe(sourcemaps.init())
        .pipe(sass())
        .pipe(gulpResolveUrl())
        .pipe(gulp.dest('./dist'));
});


### IMPORTANT

#### Source maps required

Note that **source maps** must be added to gulp stream. You need to use for example gulp-sourcemaps plugin.


### Options
Options are passed as the first parameter to the gulpResolveUrl function.

```
    var gulpResolveUrl = require('gulp-resolve-url');
    ...
    source.pipe(gulpResolveUrl(...));

Where `...` is aa object with the following properties.

* `attempts` Limit searching for any files not where they are expected to be. This is unlimited by default so you will want to set it `1` or some small value.

* `silent` Do not display warnings on CSS syntax or source-map error.

* `fail` Syntax or source-map errors will result in an error.

* `keepQuery` Keep query string and hash within url. I.e. `url('./MyFont.eot?#iefix')`, `url('./MyFont.svg#oldiosfix')`.

* `debug` Show verbose information on the file paths being searched.

* `root` An optional directory within which search may be performed. Relative paths are permitted. Where omitted `process.cwd()` is used and should be sufficient for most use cases.

There are some additional hacks available without support. Only do this if you know what you are doing.

* `absolute` Forces the url() to be resolved to an absolute path. This is considered 
[bad practice](http://webpack.github.io/docs/how-to-write-a-loader.html#should-not-embed-absolute-paths).

* `includeRoot` (experimental, non-performant) Include the project `root` in file search. The `root` option need not be specified but `includeRoot` is only really useful if your `root` directory is shallower than your build working directory.

Note that query parameters take precedence over programmatic parameters.

## How it works

A [rework](https://github.com/reworkcss/rework) process is run on incoming CSS.

Each `url()` statement that implies an asset triggers a file search using node `fs` operations. The asset should be relative to the original source file that was transpiled. This original source is determined by consulting the incoming source-map at the point of the `url()` statement.

Usually the asset is found relative to the original source file `O(1)`.

However in cases where there is no immediate match, we start searching both deeper and shallower from the starting directory `O(n)`. Note that `n` may be limited by the `attempts` option.

This file search "magic" is mainly for historic reasons, to work around broken packages whose assets are not where we would expect.

Shallower paths must be limited to avoid the whole file system from being considered. Progressively shallower paths within the `root` will be considered. Paths featuring a `package.json` or `bower.json` file will not be considered.

If the asset is not found then the `url()` statement will not be updated with a Webpack module-relative path. However if the `url()` statement has no source-map `source` information the loader will fail.

The loader will also fail when input source-map `sources` cannot all be resolved relative to some consistent path within `root`.

Use the `debug` option to see exactly what paths are being searched.

## Limitations / Known-issues

### File search "magic"

Failure to find an asset will trigger a file search of your project.

This feature was for historic reasons, to work around broken packages, whose assets are not where we would expect. Such problems are rare now and many users may not be aware of the search feature.

We now have the `attempts` option to limit this feature. However by default it is unlimited (`attempts=0`) which could make your build non-performant.

You should explicitly set `attempts=1` and increase the value only if needed. We will look to make this the default in the next major release.


### Mixins

Where `url()` statements are created in a mixin the source file may then be the mixin file, and not the file calling the mixin. Obviously this is **not** the desired behaviour.

The incoming source map can vary greatly with different transpilers and their mixins. Use a [source map visualiser](http://sokra.github.io/source-map-visualization/#custom-choose) to see more.  If the source-map shows the correct original file and the mixin still doesn't work then raise an issue and point to the visualisation.

Ultimately you will need to work around this. Try to avoid the mixin. Worst case you can try the `includeRoot` option to force a search of your project sources.
