<!--

@license Apache-2.0

Copyright (c) 2026 The Stdlib Authors.

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


<details>
  <summary>
    About stdlib...
  </summary>
  <p>We believe in a future in which the web is a preferred environment for numerical computation. To help realize this future, we've built stdlib. stdlib is a standard library, with an emphasis on numerical and scientific computation, written in JavaScript (and C) for execution in browsers and in Node.js.</p>
  <p>The library is fully decomposable, being architected in such a way that you can swap out and mix and match APIs and functionality to cater to your exact preferences and use cases.</p>
  <p>When you use stdlib, you can be absolutely certain that you are using the most thorough, rigorous, well-written, studied, documented, tested, measured, and high-quality code out there.</p>
  <p>To join us in bringing numerical computing to the web, get started by checking us out on <a href="https://github.com/stdlib-js/stdlib">GitHub</a>, and please consider <a href="https://opencollective.com/stdlib">financially supporting stdlib</a>. We greatly appreciate your continued support!</p>
</details>

# resolveKernel

[![NPM version][npm-image]][npm-url] [![Build Status][test-image]][test-url] [![Coverage Status][coverage-image]][coverage-url] <!-- [![dependencies][dependencies-image]][dependencies-url] -->

> Return a kernel for applying a one-dimensional strided array function to an input ndarray and assigning results to an output ndarray.

<section class="intro">

</section>

<!-- /.intro -->



<section class="usage">

## Usage

```javascript
import resolveKernel from 'https://cdn.jsdelivr.net/gh/stdlib-js/ndarray-base-kernels-generic-unary-strided1d-unblocked@esm/index.mjs';
```

#### resolveKernel( ndims )

Returns a kernel for applying a one-dimensional strided array function to an input ndarray and assigning results to an output ndarray.

<!-- eslint-disable max-len -->

```javascript
import Float64Array from 'https://cdn.jsdelivr.net/gh/stdlib-js/array-float64@esm/index.mjs';
import ndarray2array from 'https://cdn.jsdelivr.net/gh/stdlib-js/ndarray-base-to-array@esm/index.mjs';
import gcusum from 'https://cdn.jsdelivr.net/gh/stdlib-js/blas-ext-base-ndarray-gcusum@esm/index.mjs';

// Create data buffers:
var xbuf = new Float64Array( [ 1.0, 2.0, 3.0, 4.0, 5.0, 6.0, 7.0, 8.0, 9.0, 10.0, 11.0, 12.0 ] );
var ybuf = new Float64Array( [ 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0 ] );

// Define the array shapes:
var xsh = [ 1, 3, 2, 2 ];
var ysh = [ 1, 3, 2, 2 ];

// Define the array strides:
var sx = [ 12, 4, 2, 1 ];
var sy = [ 12, 4, 2, 1 ];

// Define the index offsets:
var ox = 0;
var oy = 0;

// Create an input ndarray descriptor:
var x = {
    'dtype': 'float64',
    'data': xbuf,
    'shape': xsh,
    'strides': sx,
    'offset': ox,
    'order': 'row-major'
};

// Create an ndarray descriptor for the initial sum:
var initial = {
    'dtype': 'float64',
    'data': new Float64Array( [ 0.0 ] ),
    'shape': [ 1, 3 ],
    'strides': [ 0, 0 ],
    'offset': 0,
    'order': 'row-major'
};

// Create an output ndarray descriptor:
var y = {
    'dtype': 'float64',
    'data': ybuf,
    'shape': ysh,
    'strides': sy,
    'offset': oy,
    'order': 'row-major'
};

// Initialize ndarray descriptors representing subarray views:
var views = [
    {
        'dtype': x.dtype,
        'data': x.data,
        'shape': [ 2, 2 ],
        'strides': [ 2, 1 ],
        'offset': x.offset,
        'order': x.order
    },
    {
        'dtype': y.dtype,
        'data': y.data,
        'shape': [ 2, 2 ],
        'strides': [ 2, 1 ],
        'offset': y.offset,
        'order': y.order
    },
    {
        'dtype': initial.dtype,
        'data': initial.data,
        'shape': [],
        'strides': [ 0 ],
        'offset': initial.offset,
        'order': initial.order
    }
];

// Define an input strategy:
function inputStrategy( x ) {
    return {
        'dtype': x.dtype,
        'data': x.data,
        'shape': [ 4 ],
        'strides': [ 1 ],
        'offset': x.offset,
        'order': x.order
    };
}

// Define an output strategy:
function outputStrategy( x ) {
    return x;
}

var strategy = {
    'input': inputStrategy,
    'output': outputStrategy
};

// Resolve a kernel:
var kernel = resolveKernel( 2 );

// Apply strided function:
kernel( gcusum, [ x, y, initial ], views, [ 1, 3 ], [ 12, 4 ], [ 12, 4 ], strategy, strategy, {} );

var arr = ndarray2array( y.data, y.shape, y.strides, y.offset, y.order );
// returns [ [ [ [ 1.0, 3.0 ], [ 6.0, 10.0 ] ], [ [ 5.0, 11.0 ], [ 18.0, 26.0 ] ], [ [ 9.0, 19.0 ], [ 30.0, 42.0 ] ] ] ]
```

The function accepts the following arguments:

-   **ndims**: number of loop dimensions.

If the function is provided an `ndims` value greater than the maximum number of supported loop dimensions, the function returns `null`.

```javascript
var f = resolveKernel( 100000 );
// returns null
```

</section>

<!-- /.usage -->

<section class="notes">

## Notes

-   The returned function accepts the following arguments:

    -   **fcn**: function which will be applied to a one-dimensional input subarray and should update a one-dimensional output subarray with results.
    -   **arrays**: array containing one input ndarray [descriptor][@stdlib/ndarray/base/descriptor] and one output ndarray [descriptor][@stdlib/ndarray/base/descriptor], followed by any additional ndarray arguments.
    -   **views**: initialized ndarray [descriptors][@stdlib/ndarray/base/descriptor] representing subarray views.
    -   **shape**: loop dimensions.
    -   **stridesX**: loop dimension strides for the input ndarray.
    -   **stridesY**: loop dimension strides for the output ndarray.
    -   **strategyX**: strategy for marshaling data to and from an input ndarray view.
    -   **strategyY**: strategy for marshaling data to and from an output ndarray view.
    -   **options**: function options which are passed through to `fcn`.

-   The strided array function is expected to have the following signature:

    ```text
    fcn( arrays[, options] )
    ```

    where

    -   **arrays**: array containing a one-dimensional subarray of the input ndarray, a one-dimensional subarray of the output ndarray, and any additional ndarray arguments as subarrays.
    -   **options**: function options (_optional_).

-   The returned function iterates over ndarray elements according to the memory layout of the input ndarray.

</section>

<!-- /.notes -->

<section class="examples">

## Examples

<!-- eslint-disable max-len -->

<!-- eslint no-undef: "error" -->

```html
<!DOCTYPE html>
<html lang="en">
<body>
<script type="module">

import Float64Array from 'https://cdn.jsdelivr.net/gh/stdlib-js/array-float64@esm/index.mjs';
import ndarray2array from 'https://cdn.jsdelivr.net/gh/stdlib-js/ndarray-base-to-array@esm/index.mjs';
import gcusum from 'https://cdn.jsdelivr.net/gh/stdlib-js/blas-ext-base-ndarray-gcusum@esm/index.mjs';
import strategy from 'https://cdn.jsdelivr.net/gh/stdlib-js/ndarray-base-kernels-generic-unary-strided1d-strategy@esm/index.mjs';
import resolveKernel from 'https://cdn.jsdelivr.net/gh/stdlib-js/ndarray-base-kernels-generic-unary-strided1d-unblocked@esm/index.mjs';

// Create data buffers:
var xbuf = new Float64Array( [ 1.0, 2.0, 3.0, 4.0, 5.0, 6.0, 7.0, 8.0, 9.0, 10.0, 11.0, 12.0 ] );
var ybuf = new Float64Array( [ 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0 ] );

// Define the array shapes:
var xsh = [ 1, 1, 1, 1, 3, 2, 2 ];
var ysh = [ 1, 1, 1, 1, 3, 2, 2 ];

// Define the array strides:
var sx = [ 12, 12, 12, 12, 4, 2, 1 ];
var sy = [ 12, 12, 12, 12, 4, 2, 1 ];

// Define the index offsets:
var ox = 0;
var oy = 0;

// Create an input ndarray descriptor:
var x = {
    'dtype': 'float64',
    'data': xbuf,
    'shape': xsh,
    'strides': sx,
    'offset': ox,
    'order': 'row-major'
};

// Create an ndarray descriptor for the initial sum:
var initial = {
    'dtype': 'float64',
    'data': new Float64Array( [ 0.0 ] ),
    'shape': [ 1, 1, 1, 1, 3 ],
    'strides': [ 0, 0, 0, 0, 0 ],
    'offset': 0,
    'order': 'row-major'
};

// Create an output ndarray descriptor:
var y = {
    'dtype': 'float64',
    'data': ybuf,
    'shape': ysh,
    'strides': sy,
    'offset': oy,
    'order': 'row-major'
};

// Initialize ndarray descriptors representing subarray views:
var views = [
    {
        'dtype': x.dtype,
        'data': x.data,
        'shape': [ 2, 2 ],
        'strides': [ 2, 1 ],
        'offset': x.offset,
        'order': x.order
    },
    {
        'dtype': y.dtype,
        'data': y.data,
        'shape': [ 2, 2 ],
        'strides': [ 2, 1 ],
        'offset': y.offset,
        'order': y.order
    },
    {
        'dtype': initial.dtype,
        'data': initial.data,
        'shape': [],
        'strides': [ 0 ],
        'offset': initial.offset,
        'order': initial.order
    }
];

// Resolve input/output strategies when iterating over subarray views:
var strategyX = strategy( views[ 0 ] );
var strategyY = strategy( views[ 1 ] );

// Resolve a kernel:
var kernel = resolveKernel( 5 );

// Apply strided function:
kernel( gcusum, [ x, y, initial ], views, [ 1, 1, 1, 1, 3 ], [ 12, 12, 12, 12, 4 ], [ 12, 12, 12, 12, 4 ], strategyX, strategyY, {} );

console.log( ndarray2array( x.data, x.shape, x.strides, x.offset, x.order ) );
console.log( ndarray2array( y.data, y.shape, y.strides, y.offset, y.order ) );

</script>
</body>
</html>
```

</section>

<!-- /.examples -->

<!-- Section for related `stdlib` packages. Do not manually edit this section, as it is automatically populated. -->

<section class="related">

</section>

<!-- /.related -->


<section class="main-repo" >

* * *

## Notice

This package is part of [stdlib][stdlib], a standard library with an emphasis on numerical and scientific computing. The library provides a collection of robust, high performance libraries for mathematics, statistics, streams, utilities, and more.

For more information on the project, filing bug reports and feature requests, and guidance on how to develop [stdlib][stdlib], see the main project [repository][stdlib].

#### Community

[![Chat][chat-image]][chat-url]

---

## Copyright

Copyright &copy; 2016-2026. The Stdlib [Authors][stdlib-authors].

</section>

<!-- /.stdlib -->

<!-- Section for all links. Make sure to keep an empty line after the `section` element and another before the `/section` close. -->

<section class="links">

[npm-image]: http://img.shields.io/npm/v/@stdlib/ndarray-base-kernels-generic-unary-strided1d-unblocked.svg
[npm-url]: https://npmjs.org/package/@stdlib/ndarray-base-kernels-generic-unary-strided1d-unblocked

[test-image]: https://github.com/stdlib-js/ndarray-base-kernels-generic-unary-strided1d-unblocked/actions/workflows/test.yml/badge.svg?branch=main
[test-url]: https://github.com/stdlib-js/ndarray-base-kernels-generic-unary-strided1d-unblocked/actions/workflows/test.yml?query=branch:main

[coverage-image]: https://img.shields.io/codecov/c/github/stdlib-js/ndarray-base-kernels-generic-unary-strided1d-unblocked/main.svg
[coverage-url]: https://codecov.io/github/stdlib-js/ndarray-base-kernels-generic-unary-strided1d-unblocked?branch=main

<!--

[dependencies-image]: https://img.shields.io/david/stdlib-js/ndarray-base-kernels-generic-unary-strided1d-unblocked.svg
[dependencies-url]: https://david-dm.org/stdlib-js/ndarray-base-kernels-generic-unary-strided1d-unblocked/main

-->

[chat-image]: https://img.shields.io/badge/zulip-join_chat-brightgreen.svg
[chat-url]: https://stdlib.zulipchat.com

[stdlib]: https://github.com/stdlib-js/stdlib

[stdlib-authors]: https://github.com/stdlib-js/stdlib/graphs/contributors

[umd]: https://github.com/umdjs/umd
[es-module]: https://developer.mozilla.org/en-US/docs/Web/JavaScript/Guide/Modules

[deno-url]: https://github.com/stdlib-js/ndarray-base-kernels-generic-unary-strided1d-unblocked/tree/deno
[deno-readme]: https://github.com/stdlib-js/ndarray-base-kernels-generic-unary-strided1d-unblocked/blob/deno/README.md
[umd-url]: https://github.com/stdlib-js/ndarray-base-kernels-generic-unary-strided1d-unblocked/tree/umd
[umd-readme]: https://github.com/stdlib-js/ndarray-base-kernels-generic-unary-strided1d-unblocked/blob/umd/README.md
[esm-url]: https://github.com/stdlib-js/ndarray-base-kernels-generic-unary-strided1d-unblocked/tree/esm
[esm-readme]: https://github.com/stdlib-js/ndarray-base-kernels-generic-unary-strided1d-unblocked/blob/esm/README.md
[branches-url]: https://github.com/stdlib-js/ndarray-base-kernels-generic-unary-strided1d-unblocked/blob/main/branches.md

[@stdlib/ndarray/base/descriptor]: https://github.com/stdlib-js/ndarray-base-descriptor/tree/esm

</section>

<!-- /.links -->
