<!--

@license Apache-2.0

Copyright (c) 2018 The Stdlib Authors.

Licensed under the Apache License, Version 2.0 (the "License");
you may not use this file except in compliance with the License.
You may obtain a copy of the License at

   http://www.apache.org/licenses/LICENSE-2.0

Unless required by applicable law or agreed to in writing, software
distributed under the License is distributed on an "AS IS" BASIS,
WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.
See the License for the specific language governing permissions and
limitations under the License.

-->

# Parallel

[![NPM version][npm-image]][npm-url] [![Build Status][test-image]][test-url] [![Coverage Status][coverage-image]][coverage-url] <!-- [![dependencies][dependencies-image]][dependencies-url] -->

> Execute scripts in parallel.

<section class="installation">

## Installation

```bash
npm install @stdlib/utils-parallel
```

Alternatively,

-   To load the package in a website via a `script` tag without installation and bundlers, use the [ES Module][es-module] available on the [`esm` branch][esm-url].
-   If you are using Deno, visit the [`deno` branch][deno-url].
-   For use in Observable, or in browser/node environments, use the [Universal Module Definition (UMD)][umd] build available on the [`umd` branch][umd-url].

</section>

<section class="usage">

## Usage

```javascript
var parallel = require( '@stdlib/utils-parallel' );
```

#### parallel( files, \[options,] clbk )

Executes scripts in parallel.

<!-- run-disable -->

```javascript
var files = [
    './a.js',
    './b.js'
];

function done( error ) {
    if ( error ) {
        console.error( 'Exit code: '+error.code );
        console.error( 'Signal: '+error.signal );
        throw error;
    }
    console.log( 'Done!' );
}

parallel( files, done );
```

The function accepts the following `options`:

-   **cmd**: executable file/command. Default: `'node'`.
-   **workers**: number of workers. Default: number of CPUs minus `1`.
-   **concurrency**: number of scripts to execute concurrently. Default: `options.workers`.
-   **ordered**: `boolean` indicating whether to preserve the order of script output. Default: `false`.
-   **maxBuffer**: maximum child process `stdio` buffer size. This option is **only** applied when `options.ordered = true`. Default: `200*1024*1024` bytes.
-   **uid**: child process user identity.
-   **gid**: child process group identity.

By default, the number of workers running scripts is equal to the number of CPUs minus `1` (master process). To adjust the number of workers, set the `workers` option.

<!-- run-disable -->

```javascript
var files = [
    './a.js',
    './b.js'
];

function done( error ) {
    if ( error ) {
        console.error( 'Exit code: '+error.code );
        console.error( 'Signal: '+error.signal );
        throw error;
    }
    console.log( 'Done!' );
}

var opts = {
    'workers': 8
};

parallel( files, opts, done );
```

By default, the number of scripts running concurrently is equal to the number of workers. To adjust the concurrency, set the `concurrency` option.

<!-- run-disable -->

```javascript
var files = [
    './a.js',
    './b.js'
];

function done( error ) {
    if ( error ) {
        console.error( 'Exit code: '+error.code );
        console.error( 'Signal: '+error.signal );
        throw error;
    }
    console.log( 'Done!' );
}

var opts = {
    'concurrency': 6
};

parallel( files, opts, done );
```

By specifying a concurrency greater than the number of workers, a worker may be executing more than `1` script at any one time. While not likely to be advantageous for synchronous scripts, setting a higher concurrency may be advantageous for scripts performing asynchronous tasks.

By default, each script is executed as a [Node.js][node-js] script.

```text
$ node <script_path>
```

To run scripts via an alternative executable or none at all, set the `cmd` option.

<!-- run-disable -->

```javascript
var files = [
    './a.js',
    './b.js'
];

function done( error ) {
    if ( error ) {
        console.error( 'Exit code: '+error.code );
        console.error( 'Signal: '+error.signal );
        throw error;
    }
    console.log( 'Done!' );
}

var opts = {
    'cmd': '' // e.g., if scripts contain a shebang
};

parallel( files, opts, done );
```

By default, the `stdio` output for each script is interleaved; i.e., the `stdio` output from one script **may** be interleaved with the `stdio` output from one or more other scripts. To preserve the `stdio` output order for each script, set the `ordered` option to `true`.

<!-- run-disable -->

```javascript
var files = [
    './a.js',
    './b.js'
];

function done( error ) {
    if ( error ) {
        console.error( 'Exit code: '+error.code );
        console.error( 'Signal: '+error.signal );
        throw error;
    }
    console.log( 'Done!' );
}

var opts = {
    'ordered': true
};

parallel( files, opts, done );
```

</section>

<!-- /.usage -->

<section class="notes">

## Notes

-   Relative file paths are resolved relative to the current working directory.
-   Ordered script output does **not** imply that scripts are executed in order. To preserve script order, execute the scripts sequentially via some other means.
-   Script concurrency cannot exceed the number of scripts.
-   If the script concurrency is less than the number of workers, the number of workers is reduced to match the specified concurrency.

</section>

<!-- /.notes -->

<section class="examples">

## Examples

<!-- run-disable -->

<!-- FIXME: re-enable running of code block once able to set `maxBuffer` configuration -->

<!-- eslint no-undef: "error" -->

```javascript
var fs = require( 'fs' );
var path = require( 'path' );
var writeFileSync = require( '@stdlib/fs-write-file' ).sync;
var unlinkSync = require( '@stdlib/fs-unlink' ).sync;
var parallel = require( '@stdlib/utils-parallel' );

var nFiles = 100;
var files;
var opts;
var dir;

function template( id ) {
    var file = '';

    file += '\'use strict\';';

    file += 'var id;';
    file += 'var N;';
    file += 'var i;';

    file += 'id = '+id+';';
    file += 'N = 1e5;';
    file += 'console.log( \'File: %s. id: %s. N: %d.\', __filename, id, N );';

    file += 'for ( i = 0; i < N; i++ ) {';
    file += 'console.log( \'id: %s. v: %d.\', id, i );';
    file += '}';

    return file;
}

function createDir() {
    var dir = path.join( __dirname, 'examples', 'tmp' );
    fs.mkdirSync( dir );
    return dir;
}

function createScripts( dir, nFiles ) {
    var content;
    var fpath;
    var files;
    var i;

    files = new Array( nFiles );
    for ( i = 0; i < nFiles; i++ ) {
        content = template( i.toString() );
        fpath = path.join( dir, i+'.js' );
        writeFileSync( fpath, content, {
            'encoding': 'utf8'
        });
        files[ i ] = fpath;
    }
    return files;
}

function cleanup() {
    var i;

    // Delete the temporary files...
    for ( i = 0; i < files.length; i++ ) {
        unlinkSync( files[ i ] );
    }
    // Remove temporary directory:
    fs.rmdirSync( dir );
}

function done( error ) {
    if ( error ) {
        throw error;
    }
    cleanup();
    console.log( 'Done!' );
}

// Make a temporary directory to store files...
dir = createDir();

// Create temporary files...
files = createScripts( dir, nFiles );

// Set the runner options:
opts = {
    'concurrency': 3,
    'workers': 3,
    'ordered': false
};

// Run all temporary scripts:
parallel( files, opts, done );
```

</section>

<!-- /.examples -->

* * *

<section class="cli">

## CLI

<section class="installation">

## Installation

To use the module as a general utility, install the module globally

```bash
npm install -g @stdlib/utils-parallel
```

</section>

<!-- CLI usage documentation. -->

<section class="usage">

### Usage

```text
Usage: parallel [options] <script1> <script2> ...

Options:

  -h,    --help                Print this message.
  -V,    --version             Print the package version.
         --cmd cmd             Executable file/command.
         --workers num         Number of workers.
         --concurrency num     Number of scripts to run concurrently.
         --ordered             Preserve order of script output.
         --uid uid             Process user identity.
         --gid gid             Process group identity.
         --maxbuffer size      Max buffer size for stdout and stderr.
```

</section>

<!-- /.usage -->

<section class="examples">

### Examples

```bash
$ parallel --cmd 'node' --workers 4 --concurrency 8 ./1.js ./2.js ./3.js ./4.js ./5.js ./6.js ./7.js ./8.js ./9.js ./10.js
```

</section>

<!-- /.examples -->

</section>

<!-- /.cli -->

<!-- Section for related `stdlib` packages. Do not manually edit this section, as it is automatically populated. -->

<section class="related">

</section>

<!-- /.related -->

<!-- Section for all links. Make sure to keep an empty line after the `section` element and another before the `/section` close. -->


<section class="main-repo" >

* * *

## Notice

This package is part of [stdlib][stdlib], a standard library for JavaScript and Node.js, with an emphasis on numerical and scientific computing. The library provides a collection of robust, high performance libraries for mathematics, statistics, streams, utilities, and more.

For more information on the project, filing bug reports and feature requests, and guidance on how to develop [stdlib][stdlib], see the main project [repository][stdlib].

#### Community

[![Chat][chat-image]][chat-url]

---

## License

See [LICENSE][stdlib-license].


## Copyright

Copyright &copy; 2016-2022. The Stdlib [Authors][stdlib-authors].

</section>

<!-- /.stdlib -->

<!-- Section for all links. Make sure to keep an empty line after the `section` element and another before the `/section` close. -->

<section class="links">

[npm-image]: http://img.shields.io/npm/v/@stdlib/utils-parallel.svg
[npm-url]: https://npmjs.org/package/@stdlib/utils-parallel

[test-image]: https://github.com/stdlib-js/utils-parallel/actions/workflows/test.yml/badge.svg?branch=main
[test-url]: https://github.com/stdlib-js/utils-parallel/actions/workflows/test.yml?query=branch:main

[coverage-image]: https://img.shields.io/codecov/c/github/stdlib-js/utils-parallel/main.svg
[coverage-url]: https://codecov.io/github/stdlib-js/utils-parallel?branch=main

<!--

[dependencies-image]: https://img.shields.io/david/stdlib-js/utils-parallel.svg
[dependencies-url]: https://david-dm.org/stdlib-js/utils-parallel/main

-->

[chat-image]: https://img.shields.io/gitter/room/stdlib-js/stdlib.svg
[chat-url]: https://gitter.im/stdlib-js/stdlib/

[stdlib]: https://github.com/stdlib-js/stdlib

[stdlib-authors]: https://github.com/stdlib-js/stdlib/graphs/contributors

[umd]: https://github.com/umdjs/umd
[es-module]: https://developer.mozilla.org/en-US/docs/Web/JavaScript/Guide/Modules

[deno-url]: https://github.com/stdlib-js/utils-parallel/tree/deno
[umd-url]: https://github.com/stdlib-js/utils-parallel/tree/umd
[esm-url]: https://github.com/stdlib-js/utils-parallel/tree/esm

[stdlib-license]: https://raw.githubusercontent.com/stdlib-js/utils-parallel/main/LICENSE

[node-js]: http://nodejs.org/

</section>

<!-- /.links -->
