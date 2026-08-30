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

# kernel

[![NPM version][npm-image]][npm-url] [![Build Status][test-image]][test-url] [![Coverage Status][coverage-image]][coverage-url] <!-- [![dependencies][dependencies-image]][dependencies-url] -->

> Return a kernel for applying a one-dimensional strided array function to an input ndarray and assigning results to an output ndarray.

<section class="intro">

</section>

<!-- /.intro -->



<section class="usage">

## Usage

To use in Observable,

```javascript
kernel = require( 'https://cdn.jsdelivr.net/gh/stdlib-js/ndarray-base-kernels-generic-unary-strided1d-unblocked@umd/browser.js' )
```

To vendor stdlib functionality and avoid installing dependency trees for Node.js, you can use the UMD server build:

```javascript
var kernel = require( 'path/to/vendor/umd/ndarray-base-kernels-generic-unary-strided1d-unblocked/index.js' )
```

To include the bundle in a webpage,

```html
<script type="text/javascript" src="https://cdn.jsdelivr.net/gh/stdlib-js/ndarray-base-kernels-generic-unary-strided1d-unblocked@umd/browser.js"></script>
```

If no recognized module system is present, access bundle contents via the global scope:

```html
<script type="text/javascript">
(function () {
    window.kernel;
})();
</script>
```

#### kernel( ndims )

Returns a kernel for applying a one-dimensional strided array function to an input ndarray and assigning results to an output ndarray.

<!-- eslint-disable max-len -->

```javascript
var Float64Array = require( '@stdlib/array-float64' );
var ndarray2array = require( '@stdlib/ndarray-base-to-array' );
var gcusum = require( '@stdlib/blas-ext-base-ndarray-gcusum' );

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
var f = kernel( 2 );

// Apply strided function:
f( gcusum, [ x, y, initial ], views, [ 1, 3 ], [ 12, 4 ], [ 12, 4 ], strategy, strategy, {} );

var arr = ndarray2array( y.data, y.shape, y.strides, y.offset, y.order );
// returns [ [ [ [ 1.0, 3.0 ], [ 6.0, 10.0 ] ], [ [ 5.0, 11.0 ], [ 18.0, 26.0 ] ], [ [ 9.0, 19.0 ], [ 30.0, 42.0 ] ] ] ]
```

The function accepts the following arguments:

-   **ndims**: number of loop dimensions.

If the function is provided an `ndims` value greater than the maximum number of supported loop dimensions, the function returns `null`.

```javascript
var f = kernel( 100000 );
// returns null
```

The returned function accepts the following arguments:

-   **fcn**: function which will be applied to a one-dimensional input subarray and should update a one-dimensional output subarray with results.
-   **arrays**: array containing one input ndarray [descriptor][@stdlib/ndarray/base/descriptor] and one output ndarray [descriptor][@stdlib/ndarray/base/descriptor], followed by any additional ndarray arguments.
-   **views**: initialized ndarray [descriptors][@stdlib/ndarray/base/descriptor] representing subarray views.
-   **shape**: loop dimensions.
-   **stridesX**: loop dimension strides for the input ndarray.
-   **stridesY**: loop dimension strides for the output ndarray.
-   **strategyX**: strategy for marshaling data to and from an input ndarray view.
-   **strategyY**: strategy for marshaling data to and from an output ndarray view.
-   **options**: function options which are passed through to `fcn`.

The returned function iterates over ndarray elements according to the memory layout of the input ndarray.

<!-- lint disable maximum-heading-length -->

#### kernel.kernel0d( fcn, arrays, views, shape, stridesX, strideY, strategyX, strategyY, options )

<!-- lint enable maximum-heading-length -->

Applies a one-dimensional strided array function to a list of specified dimensions in an input ndarray and assigns results to a provided output ndarray.

<!-- eslint-disable max-len -->

```javascript
var Float64Array = require( '@stdlib/array-float64' );
var ndarray2array = require( '@stdlib/ndarray-base-to-array' );
var gcusum = require( '@stdlib/blas-ext-base-ndarray-gcusum' );

// Create data buffers:
var xbuf = new Float64Array( [ 1.0, 2.0, 3.0, 4.0 ] );
var ybuf = new Float64Array( [ 0.0, 0.0, 0.0, 0.0 ] );

// Define the array shapes:
var xsh = [ 2, 2 ];
var ysh = [ 2, 2 ];

// Define the array strides:
var sx = [ 2, 1 ];
var sy = [ 2, 1 ];

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
    'shape': [ 3 ],
    'strides': [ 0 ],
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

// Apply strided function:
kernel.kernel0d( gcusum, [ x, y, initial ], [], [], [], [], strategy, strategy, {} );

var v = y.data;
// returns <Float64Array>[ 1.0, 3.0, 6.0, 10.0 ]
```

The function has the following parameters:

-   **fcn**: function which will be applied to a one-dimensional input subarray and should update a one-dimensional output subarray with results.
-   **arrays**: array containing one input ndarray [descriptor][@stdlib/ndarray/base/descriptor] and one output ndarray [descriptor][@stdlib/ndarray/base/descriptor], followed by any additional ndarray arguments.
-   **views**: initialized ndarray [descriptors][@stdlib/ndarray/base/descriptor] representing subarray views.
-   **shape**: loop dimensions. Should have zero elements.
-   **stridesX**: loop dimension strides for the input ndarray.
-   **stridesY**: loop dimension strides for the output ndarray.
-   **strategyX**: strategy for marshaling data to and from an input ndarray view.
-   **strategyY**: strategy for marshaling data to and from an output ndarray view.
-   **options**: function options which are passed through to `fcn`.

The `views`, `shape`, `stridesX`, and `stridesY` parameters are unused. Providing empty arrays for these parameters is recommended in order to ensure a monomorphic API.

<!-- lint disable maximum-heading-length -->

#### kernel.kernel1d( fcn, arrays, views, shape, stridesX, strideY, strategyX, strategyY, options )

<!-- lint enable maximum-heading-length -->

Applies a one-dimensional strided array function to a list of specified dimensions in an input ndarray and assigns results to a provided output ndarray.

<!-- eslint-disable max-len -->

```javascript
var Float64Array = require( '@stdlib/array-float64' );
var ndarray2array = require( '@stdlib/ndarray-base-to-array' );
var gcusum = require( '@stdlib/blas-ext-base-ndarray-gcusum' );

// Create data buffers:
var xbuf = new Float64Array( [ 1.0, 2.0, 3.0, 4.0, 5.0, 6.0, 7.0, 8.0, 9.0, 10.0, 11.0, 12.0 ] );
var ybuf = new Float64Array( [ 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0 ] );

// Define the array shapes:
var xsh = [ 3, 2, 2 ];
var ysh = [ 3, 2, 2 ];

// Define the array strides:
var sx = [ 4, 2, 1 ];
var sy = [ 4, 2, 1 ];

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
    'shape': [ 3 ],
    'strides': [ 0 ],
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

// Apply strided function:
kernel.kernel1d( gcusum, [ x, y, initial ], views, [ 3 ], [ 4 ], [ 4 ], strategy, strategy, {} );

var arr = ndarray2array( y.data, y.shape, y.strides, y.offset, y.order );
// returns [ [ [ 1.0, 3.0 ], [ 6.0, 10.0 ] ], [ [ 5.0, 11.0 ], [ 18.0, 26.0 ] ], [ [ 9.0, 19.0 ], [ 30.0, 42.0 ] ] ]
```

The function has the following parameters:

-   **fcn**: function which will be applied to a one-dimensional input subarray and should update a one-dimensional output subarray with results.
-   **arrays**: array containing one input ndarray [descriptor][@stdlib/ndarray/base/descriptor] and one output ndarray [descriptor][@stdlib/ndarray/base/descriptor], followed by any additional ndarray arguments.
-   **views**: initialized ndarray [descriptors][@stdlib/ndarray/base/descriptor] representing subarray views.
-   **shape**: loop dimensions. Should have one element.
-   **stridesX**: loop dimension strides for the input ndarray.
-   **stridesY**: loop dimension strides for the output ndarray.
-   **strategyX**: strategy for marshaling data to and from an input ndarray view.
-   **strategyY**: strategy for marshaling data to and from an output ndarray view.
-   **options**: function options which are passed through to `fcn`.

<!-- lint disable maximum-heading-length -->

#### kernel.kernel2d( fcn, arrays, views, shape, stridesX, strideY, strategyX, strategyY, options )

<!-- lint enable maximum-heading-length -->

Applies a one-dimensional strided array function to a list of specified dimensions in an input ndarray and assigns results to a provided output ndarray.

<!-- eslint-disable max-len -->

```javascript
var Float64Array = require( '@stdlib/array-float64' );
var ndarray2array = require( '@stdlib/ndarray-base-to-array' );
var gcusum = require( '@stdlib/blas-ext-base-ndarray-gcusum' );

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

// Apply strided function:
kernel.kernel2d( gcusum, [ x, y, initial ], views, [ 1, 3 ], [ 12, 4 ], [ 12, 4 ], strategy, strategy, {} );

var arr = ndarray2array( y.data, y.shape, y.strides, y.offset, y.order );
// returns [ [ [ [ 1.0, 3.0 ], [ 6.0, 10.0 ] ], [ [ 5.0, 11.0 ], [ 18.0, 26.0 ] ], [ [ 9.0, 19.0 ], [ 30.0, 42.0 ] ] ] ]
```

The function has the following parameters:

-   **fcn**: function which will be applied to a one-dimensional input subarray and should update a one-dimensional output subarray with results.
-   **arrays**: array containing one input ndarray [descriptor][@stdlib/ndarray/base/descriptor] and one output ndarray [descriptor][@stdlib/ndarray/base/descriptor], followed by any additional ndarray arguments.
-   **views**: initialized ndarray [descriptors][@stdlib/ndarray/base/descriptor] representing subarray views.
-   **shape**: loop dimensions. Should have two elements.
-   **stridesX**: loop dimension strides for the input ndarray.
-   **stridesY**: loop dimension strides for the output ndarray.
-   **strategyX**: strategy for marshaling data to and from an input ndarray view.
-   **strategyY**: strategy for marshaling data to and from an output ndarray view.
-   **options**: function options which are passed through to `fcn`.

<!-- lint disable maximum-heading-length -->

#### kernel.kernel3d( fcn, arrays, views, shape, stridesX, strideY, strategyX, strategyY, options )

<!-- lint enable maximum-heading-length -->

Applies a one-dimensional strided array function to a list of specified dimensions in an input ndarray and assigns results to a provided output ndarray.

<!-- eslint-disable max-len -->

```javascript
var Float64Array = require( '@stdlib/array-float64' );
var ndarray2array = require( '@stdlib/ndarray-base-to-array' );
var gcusum = require( '@stdlib/blas-ext-base-ndarray-gcusum' );

// Create data buffers:
var xbuf = new Float64Array( [ 1.0, 2.0, 3.0, 4.0, 5.0, 6.0, 7.0, 8.0, 9.0, 10.0, 11.0, 12.0 ] );
var ybuf = new Float64Array( [ 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0 ] );

// Define the array shapes:
var xsh = [ 1, 1, 3, 2, 2 ];
var ysh = [ 1, 1, 3, 2, 2 ];

// Define the array strides:
var sx = [ 12, 12, 4, 2, 1 ];
var sy = [ 12, 12, 4, 2, 1 ];

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
    'shape': [ 1, 1, 3 ],
    'strides': [ 0, 0, 0 ],
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

// Apply strided function:
kernel.kernel3d( gcusum, [ x, y, initial ], views, [ 1, 1, 3 ], [ 12, 12, 4 ], [ 12, 12, 4 ], strategy, strategy, {} );

var arr = ndarray2array( y.data, y.shape, y.strides, y.offset, y.order );
// returns [ [ [ [ [ 1.0, 3.0 ], [ 6.0, 10.0 ] ], [ [ 5.0, 11.0 ], [ 18.0, 26.0 ] ], [ [ 9.0, 19.0 ], [ 30.0, 42.0 ] ] ] ] ]
```

The function has the following parameters:

-   **fcn**: function which will be applied to a one-dimensional input subarray and should update a one-dimensional output subarray with results.
-   **arrays**: array containing one input ndarray [descriptor][@stdlib/ndarray/base/descriptor] and one output ndarray [descriptor][@stdlib/ndarray/base/descriptor], followed by any additional ndarray arguments.
-   **views**: initialized ndarray [descriptors][@stdlib/ndarray/base/descriptor] representing subarray views.
-   **shape**: loop dimensions. Should have three elements.
-   **stridesX**: loop dimension strides for the input ndarray.
-   **stridesY**: loop dimension strides for the output ndarray.
-   **strategyX**: strategy for marshaling data to and from an input ndarray view.
-   **strategyY**: strategy for marshaling data to and from an output ndarray view.
-   **options**: function options which are passed through to `fcn`.

<!-- lint disable maximum-heading-length -->

#### kernel.kernel4d( fcn, arrays, views, shape, stridesX, strideY, strategyX, strategyY, options )

<!-- lint enable maximum-heading-length -->

Applies a one-dimensional strided array function to a list of specified dimensions in an input ndarray and assigns results to a provided output ndarray.

<!-- eslint-disable max-len -->

```javascript
var Float64Array = require( '@stdlib/array-float64' );
var ndarray2array = require( '@stdlib/ndarray-base-to-array' );
var gcusum = require( '@stdlib/blas-ext-base-ndarray-gcusum' );

// Create data buffers:
var xbuf = new Float64Array( [ 1.0, 2.0, 3.0, 4.0, 5.0, 6.0, 7.0, 8.0, 9.0, 10.0, 11.0, 12.0 ] );
var ybuf = new Float64Array( [ 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0 ] );

// Define the array shapes:
var xsh = [ 1, 1, 1, 3, 2, 2 ];
var ysh = [ 1, 1, 1, 3, 2, 2 ];

// Define the array strides:
var sx = [ 12, 12, 12, 4, 2, 1 ];
var sy = [ 12, 12, 12, 4, 2, 1 ];

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
    'shape': [ 1, 1, 1, 3 ],
    'strides': [ 0, 0, 0, 0 ],
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

// Apply strided function:
kernel.kernel4d( gcusum, [ x, y, initial ], views, [ 1, 1, 1, 3 ], [ 12, 12, 12, 4 ], [ 12, 12, 12, 4 ], strategy, strategy, {} );

var arr = ndarray2array( y.data, y.shape, y.strides, y.offset, y.order );
// returns [ [ [ [ [ [ 1.0, 3.0 ], [ 6.0, 10.0 ] ], [ [ 5.0, 11.0 ], [ 18.0, 26.0 ] ], [ [ 9.0, 19.0 ], [ 30.0, 42.0 ] ] ] ] ] ]
```

The function has the following parameters:

-   **fcn**: function which will be applied to a one-dimensional input subarray and should update a one-dimensional output subarray with results.
-   **arrays**: array containing one input ndarray [descriptor][@stdlib/ndarray/base/descriptor] and one output ndarray [descriptor][@stdlib/ndarray/base/descriptor], followed by any additional ndarray arguments.
-   **views**: initialized ndarray [descriptors][@stdlib/ndarray/base/descriptor] representing subarray views.
-   **shape**: loop dimensions. Should have four elements.
-   **stridesX**: loop dimension strides for the input ndarray.
-   **stridesY**: loop dimension strides for the output ndarray.
-   **strategyX**: strategy for marshaling data to and from an input ndarray view.
-   **strategyY**: strategy for marshaling data to and from an output ndarray view.
-   **options**: function options which are passed through to `fcn`.

<!-- lint disable maximum-heading-length -->

#### kernel.kernel5d( fcn, arrays, views, shape, stridesX, strideY, strategyX, strategyY, options )

<!-- lint enable maximum-heading-length -->

Applies a one-dimensional strided array function to a list of specified dimensions in an input ndarray and assigns results to a provided output ndarray.

<!-- eslint-disable max-len -->

```javascript
var Float64Array = require( '@stdlib/array-float64' );
var ndarray2array = require( '@stdlib/ndarray-base-to-array' );
var gcusum = require( '@stdlib/blas-ext-base-ndarray-gcusum' );

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

// Apply strided function:
kernel.kernel5d( gcusum, [ x, y, initial ], views, [ 1, 1, 1, 1, 3 ], [ 12, 12, 12, 12, 4 ], [ 12, 12, 12, 12, 4 ], strategy, strategy, {} );

var arr = ndarray2array( y.data, y.shape, y.strides, y.offset, y.order );
// returns [ [ [ [ [ [ [ 1.0, 3.0 ], [ 6.0, 10.0 ] ], [ [ 5.0, 11.0 ], [ 18.0, 26.0 ] ], [ [ 9.0, 19.0 ], [ 30.0, 42.0 ] ] ] ] ] ] ]
```

The function has the following parameters:

-   **fcn**: function which will be applied to a one-dimensional input subarray and should update a one-dimensional output subarray with results.
-   **arrays**: array containing one input ndarray [descriptor][@stdlib/ndarray/base/descriptor] and one output ndarray [descriptor][@stdlib/ndarray/base/descriptor], followed by any additional ndarray arguments.
-   **views**: initialized ndarray [descriptors][@stdlib/ndarray/base/descriptor] representing subarray views.
-   **shape**: loop dimensions. Should have five elements.
-   **stridesX**: loop dimension strides for the input ndarray.
-   **stridesY**: loop dimension strides for the output ndarray.
-   **strategyX**: strategy for marshaling data to and from an input ndarray view.
-   **strategyY**: strategy for marshaling data to and from an output ndarray view.
-   **options**: function options which are passed through to `fcn`.

<!-- lint disable maximum-heading-length -->

#### kernel.kernel6d( fcn, arrays, views, shape, stridesX, strideY, strategyX, strategyY, options )

<!-- lint enable maximum-heading-length -->

Applies a one-dimensional strided array function to a list of specified dimensions in an input ndarray and assigns results to a provided output ndarray.

<!-- eslint-disable max-len -->

```javascript
var Float64Array = require( '@stdlib/array-float64' );
var ndarray2array = require( '@stdlib/ndarray-base-to-array' );
var gcusum = require( '@stdlib/blas-ext-base-ndarray-gcusum' );

// Create data buffers:
var xbuf = new Float64Array( [ 1.0, 2.0, 3.0, 4.0, 5.0, 6.0, 7.0, 8.0, 9.0, 10.0, 11.0, 12.0 ] );
var ybuf = new Float64Array( [ 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0 ] );

// Define the array shapes:
var xsh = [ 1, 1, 1, 1, 1, 3, 2, 2 ];
var ysh = [ 1, 1, 1, 1, 1, 3, 2, 2 ];

// Define the array strides:
var sx = [ 12, 12, 12, 12, 12, 4, 2, 1 ];
var sy = [ 12, 12, 12, 12, 12, 4, 2, 1 ];

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
    'shape': [ 1, 1, 1, 1, 1, 3 ],
    'strides': [ 0, 0, 0, 0, 0, 0 ],
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

// Apply strided function:
kernel.kernel6d( gcusum, [ x, y, initial ], views, [ 1, 1, 1, 1, 1, 3 ], [ 12, 12, 12, 12, 12, 4 ], [ 12, 12, 12, 12, 12, 4 ], strategy, strategy, {} );

var arr = ndarray2array( y.data, y.shape, y.strides, y.offset, y.order );
// returns [ [ [ [ [ [ [ [ 1.0, 3.0 ], [ 6.0, 10.0 ] ], [ [ 5.0, 11.0 ], [ 18.0, 26.0 ] ], [ [ 9.0, 19.0 ], [ 30.0, 42.0 ] ] ] ] ] ] ] ]
```

The function has the following parameters:

-   **fcn**: function which will be applied to a one-dimensional input subarray and should update a one-dimensional output subarray with results.
-   **arrays**: array containing one input ndarray [descriptor][@stdlib/ndarray/base/descriptor] and one output ndarray [descriptor][@stdlib/ndarray/base/descriptor], followed by any additional ndarray arguments.
-   **views**: initialized ndarray [descriptors][@stdlib/ndarray/base/descriptor] representing subarray views.
-   **shape**: loop dimensions. Should have six elements.
-   **stridesX**: loop dimension strides for the input ndarray.
-   **stridesY**: loop dimension strides for the output ndarray.
-   **strategyX**: strategy for marshaling data to and from an input ndarray view.
-   **strategyY**: strategy for marshaling data to and from an output ndarray view.
-   **options**: function options which are passed through to `fcn`.

<!-- lint disable maximum-heading-length -->

#### kernel.kernel7d( fcn, arrays, views, shape, stridesX, strideY, strategyX, strategyY, options )

<!-- lint enable maximum-heading-length -->

Applies a one-dimensional strided array function to a list of specified dimensions in an input ndarray and assigns results to a provided output ndarray.

<!-- eslint-disable max-len -->

```javascript
var Float64Array = require( '@stdlib/array-float64' );
var ndarray2array = require( '@stdlib/ndarray-base-to-array' );
var gcusum = require( '@stdlib/blas-ext-base-ndarray-gcusum' );

// Create data buffers:
var xbuf = new Float64Array( [ 1.0, 2.0, 3.0, 4.0, 5.0, 6.0, 7.0, 8.0, 9.0, 10.0, 11.0, 12.0 ] );
var ybuf = new Float64Array( [ 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0 ] );

// Define the array shapes:
var xsh = [ 1, 1, 1, 1, 1, 1, 3, 2, 2 ];
var ysh = [ 1, 1, 1, 1, 1, 1, 3, 2, 2 ];

// Define the array strides:
var sx = [ 12, 12, 12, 12, 12, 12, 4, 2, 1 ];
var sy = [ 12, 12, 12, 12, 12, 12, 4, 2, 1 ];

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
    'shape': [ 1, 1, 1, 1, 1, 1, 3 ],
    'strides': [ 0, 0, 0, 0, 0, 0, 0 ],
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

// Apply strided function:
kernel.kernel7d( gcusum, [ x, y, initial ], views, [ 1, 1, 1, 1, 1, 1, 3 ], [ 12, 12, 12, 12, 12, 12, 4 ], [ 12, 12, 12, 12, 12, 12, 4 ], strategy, strategy, {} );

var arr = ndarray2array( y.data, y.shape, y.strides, y.offset, y.order );
// returns [ [ [ [ [ [ [ [ [ 1.0, 3.0 ], [ 6.0, 10.0 ] ], [ [ 5.0, 11.0 ], [ 18.0, 26.0 ] ], [ [ 9.0, 19.0 ], [ 30.0, 42.0 ] ] ] ] ] ] ] ] ]
```

The function has the following parameters:

-   **fcn**: function which will be applied to a one-dimensional input subarray and should update a one-dimensional output subarray with results.
-   **arrays**: array containing one input ndarray [descriptor][@stdlib/ndarray/base/descriptor] and one output ndarray [descriptor][@stdlib/ndarray/base/descriptor], followed by any additional ndarray arguments.
-   **views**: initialized ndarray [descriptors][@stdlib/ndarray/base/descriptor] representing subarray views.
-   **shape**: loop dimensions. Should have seven elements.
-   **stridesX**: loop dimension strides for the input ndarray.
-   **stridesY**: loop dimension strides for the output ndarray.
-   **strategyX**: strategy for marshaling data to and from an input ndarray view.
-   **strategyY**: strategy for marshaling data to and from an output ndarray view.
-   **options**: function options which are passed through to `fcn`.

<!-- lint disable maximum-heading-length -->

#### kernel.kernel8d( fcn, arrays, views, shape, stridesX, strideY, strategyX, strategyY, options )

<!-- lint enable maximum-heading-length -->

Applies a one-dimensional strided array function to a list of specified dimensions in an input ndarray and assigns results to a provided output ndarray.

<!-- eslint-disable max-len -->

```javascript
var Float64Array = require( '@stdlib/array-float64' );
var ndarray2array = require( '@stdlib/ndarray-base-to-array' );
var gcusum = require( '@stdlib/blas-ext-base-ndarray-gcusum' );

// Create data buffers:
var xbuf = new Float64Array( [ 1.0, 2.0, 3.0, 4.0, 5.0, 6.0, 7.0, 8.0, 9.0, 10.0, 11.0, 12.0 ] );
var ybuf = new Float64Array( [ 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0 ] );

// Define the array shapes:
var xsh = [ 1, 1, 1, 1, 1, 1, 1, 3, 2, 2 ];
var ysh = [ 1, 1, 1, 1, 1, 1, 1, 3, 2, 2 ];

// Define the array strides:
var sx = [ 12, 12, 12, 12, 12, 12, 12, 4, 2, 1 ];
var sy = [ 12, 12, 12, 12, 12, 12, 12, 4, 2, 1 ];

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
    'shape': [ 1, 1, 1, 1, 1, 1, 1, 3 ],
    'strides': [ 0, 0, 0, 0, 0, 0, 0, 0 ],
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

// Apply strided function:
kernel.kernel8d( gcusum, [ x, y, initial ], views, [ 1, 1, 1, 1, 1, 1, 1, 3 ], [ 12, 12, 12, 12, 12, 12, 12, 4 ], [ 12, 12, 12, 12, 12, 12, 12, 4 ], strategy, strategy, {} );

var arr = ndarray2array( y.data, y.shape, y.strides, y.offset, y.order );
// returns [ [ [ [ [ [ [ [ [ [ 1.0, 3.0 ], [ 6.0, 10.0 ] ], [ [ 5.0, 11.0 ], [ 18.0, 26.0 ] ], [ [ 9.0, 19.0 ], [ 30.0, 42.0 ] ] ] ] ] ] ] ] ] ]
```

The function has the following parameters:

-   **fcn**: function which will be applied to a one-dimensional input subarray and should update a one-dimensional output subarray with results.
-   **arrays**: array containing one input ndarray [descriptor][@stdlib/ndarray/base/descriptor] and one output ndarray [descriptor][@stdlib/ndarray/base/descriptor], followed by any additional ndarray arguments.
-   **views**: initialized ndarray [descriptors][@stdlib/ndarray/base/descriptor] representing subarray views.
-   **shape**: loop dimensions. Should have eight elements.
-   **stridesX**: loop dimension strides for the input ndarray.
-   **stridesY**: loop dimension strides for the output ndarray.
-   **strategyX**: strategy for marshaling data to and from an input ndarray view.
-   **strategyY**: strategy for marshaling data to and from an output ndarray view.
-   **options**: function options which are passed through to `fcn`.

<!-- lint disable maximum-heading-length -->

#### kernel.kernel9d( fcn, arrays, views, shape, stridesX, strideY, strategyX, strategyY, options )

<!-- lint enable maximum-heading-length -->

Applies a one-dimensional strided array function to a list of specified dimensions in an input ndarray and assigns results to a provided output ndarray.

<!-- eslint-disable max-len -->

```javascript
var Float64Array = require( '@stdlib/array-float64' );
var ndarray2array = require( '@stdlib/ndarray-base-to-array' );
var gcusum = require( '@stdlib/blas-ext-base-ndarray-gcusum' );

// Create data buffers:
var xbuf = new Float64Array( [ 1.0, 2.0, 3.0, 4.0, 5.0, 6.0, 7.0, 8.0, 9.0, 10.0, 11.0, 12.0 ] );
var ybuf = new Float64Array( [ 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0 ] );

// Define the array shapes:
var xsh = [ 1, 1, 1, 1, 1, 1, 1, 1, 3, 2, 2 ];
var ysh = [ 1, 1, 1, 1, 1, 1, 1, 1, 3, 2, 2 ];

// Define the array strides:
var sx = [ 12, 12, 12, 12, 12, 12, 12, 12, 4, 2, 1 ];
var sy = [ 12, 12, 12, 12, 12, 12, 12, 12, 4, 2, 1 ];

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
    'shape': [ 1, 1, 1, 1, 1, 1, 1, 1, 3 ],
    'strides': [ 0, 0, 0, 0, 0, 0, 0, 0, 0 ],
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

// Apply strided function:
kernel.kernel9d( gcusum, [ x, y, initial ], views, [ 1, 1, 1, 1, 1, 1, 1, 1, 3 ], [ 12, 12, 12, 12, 12, 12, 12, 12, 4 ], [ 12, 12, 12, 12, 12, 12, 12, 12, 4 ], strategy, strategy, {} );

var arr = ndarray2array( y.data, y.shape, y.strides, y.offset, y.order );
// returns [ [ [ [ [ [ [ [ [ [ [ 1.0, 3.0 ], [ 6.0, 10.0 ] ], [ [ 5.0, 11.0 ], [ 18.0, 26.0 ] ], [ [ 9.0, 19.0 ], [ 30.0, 42.0 ] ] ] ] ] ] ] ] ] ] ]
```

The function has the following parameters:

-   **fcn**: function which will be applied to a one-dimensional input subarray and should update a one-dimensional output subarray with results.
-   **arrays**: array containing one input ndarray [descriptor][@stdlib/ndarray/base/descriptor] and one output ndarray [descriptor][@stdlib/ndarray/base/descriptor], followed by any additional ndarray arguments.
-   **views**: initialized ndarray [descriptors][@stdlib/ndarray/base/descriptor] representing subarray views.
-   **shape**: loop dimensions. Should have nine elements.
-   **stridesX**: loop dimension strides for the input ndarray.
-   **stridesY**: loop dimension strides for the output ndarray.
-   **strategyX**: strategy for marshaling data to and from an input ndarray view.
-   **strategyY**: strategy for marshaling data to and from an output ndarray view.
-   **options**: function options which are passed through to `fcn`.

<!-- lint disable maximum-heading-length -->

#### kernel.kernel10d( fcn, arrays, views, shape, stridesX, strideY, strategyX, strategyY, options )

<!-- lint enable maximum-heading-length -->

Applies a one-dimensional strided array function to a list of specified dimensions in an input ndarray and assigns results to a provided output ndarray.

<!-- eslint-disable max-len -->

```javascript
var Float64Array = require( '@stdlib/array-float64' );
var ndarray2array = require( '@stdlib/ndarray-base-to-array' );
var gcusum = require( '@stdlib/blas-ext-base-ndarray-gcusum' );

// Create data buffers:
var xbuf = new Float64Array( [ 1.0, 2.0, 3.0, 4.0, 5.0, 6.0, 7.0, 8.0, 9.0, 10.0, 11.0, 12.0 ] );
var ybuf = new Float64Array( [ 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0 ] );

// Define the array shapes:
var xsh = [ 1, 1, 1, 1, 1, 1, 1, 1, 1, 3, 2, 2 ];
var ysh = [ 1, 1, 1, 1, 1, 1, 1, 1, 1, 3, 2, 2 ];

// Define the array strides:
var sx = [ 12, 12, 12, 12, 12, 12, 12, 12, 12, 4, 2, 1 ];
var sy = [ 12, 12, 12, 12, 12, 12, 12, 12, 12, 4, 2, 1 ];

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
    'shape': [ 1, 1, 1, 1, 1, 1, 1, 1, 1, 3 ],
    'strides': [ 0, 0, 0, 0, 0, 0, 0, 0, 0, 0 ],
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

// Apply strided function:
kernel.kernel10d( gcusum, [ x, y, initial ], views, [ 1, 1, 1, 1, 1, 1, 1, 1, 1, 3 ], [ 12, 12, 12, 12, 12, 12, 12, 12, 12, 4 ], [ 12, 12, 12, 12, 12, 12, 12, 12, 12, 4 ], strategy, strategy, {} );

var arr = ndarray2array( y.data, y.shape, y.strides, y.offset, y.order );
// returns [ [ [ [ [ [ [ [ [ [ [ [ 1.0, 3.0 ], [ 6.0, 10.0 ] ], [ [ 5.0, 11.0 ], [ 18.0, 26.0 ] ], [ [ 9.0, 19.0 ], [ 30.0, 42.0 ] ] ] ] ] ] ] ] ] ] ] ]
```

The function has the following parameters:

-   **fcn**: function which will be applied to a one-dimensional input subarray and should update a one-dimensional output subarray with results.
-   **arrays**: array containing one input ndarray [descriptor][@stdlib/ndarray/base/descriptor] and one output ndarray [descriptor][@stdlib/ndarray/base/descriptor], followed by any additional ndarray arguments.
-   **views**: initialized ndarray [descriptors][@stdlib/ndarray/base/descriptor] representing subarray views.
-   **shape**: loop dimensions. Should have ten elements.
-   **stridesX**: loop dimension strides for the input ndarray.
-   **stridesY**: loop dimension strides for the output ndarray.
-   **strategyX**: strategy for marshaling data to and from an input ndarray view.
-   **strategyY**: strategy for marshaling data to and from an output ndarray view.
-   **options**: function options which are passed through to `fcn`.

</section>

<!-- /.usage -->

<section class="notes">

## Notes

-   The strided array function is expected to have the following signature:

    ```text
    fcn( arrays[, options] )
    ```

    where

    -   **arrays**: array containing a one-dimensional subarray of the input ndarray, a one-dimensional subarray of the output ndarray, and any additional ndarray arguments as subarrays.
    -   **options**: function options (_optional_).

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
<script type="text/javascript" src="https://cdn.jsdelivr.net/gh/stdlib-js/array-float64@umd/browser.js"></script>
<script type="text/javascript" src="https://cdn.jsdelivr.net/gh/stdlib-js/ndarray-base-to-array@umd/browser.js"></script>
<script type="text/javascript" src="https://cdn.jsdelivr.net/gh/stdlib-js/blas-ext-base-ndarray-gcusum@umd/browser.js"></script>
<script type="text/javascript" src="https://cdn.jsdelivr.net/gh/stdlib-js/ndarray-base-kernels-generic-unary-strided1d-strategy@umd/browser.js"></script>
<script type="text/javascript" src="https://cdn.jsdelivr.net/gh/stdlib-js/ndarray-base-kernels-generic-unary-strided1d-unblocked@umd/browser.js"></script>
<script type="text/javascript">
(function () {

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
var f = kernel( 5 );

// Apply strided function:
f( gcusum, [ x, y, initial ], views, [ 1, 1, 1, 1, 3 ], [ 12, 12, 12, 12, 4 ], [ 12, 12, 12, 12, 4 ], strategyX, strategyY, {} );

console.log( ndarray2array( x.data, x.shape, x.strides, x.offset, x.order ) );
console.log( ndarray2array( y.data, y.shape, y.strides, y.offset, y.order ) );

})();
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

This package is part of [stdlib][stdlib], a standard library for JavaScript and Node.js, with an emphasis on numerical and scientific computing. The library provides a collection of robust, high performance libraries for mathematics, statistics, streams, utilities, and more.

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

[@stdlib/ndarray/base/descriptor]: https://github.com/stdlib-js/ndarray-base-descriptor/tree/umd

</section>

<!-- /.links -->
