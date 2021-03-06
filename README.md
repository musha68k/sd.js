sd.js - System Dynamics for the web
===================================

A modern, high performance, open source system dynamics engine for
today's web and tomorrow's.  `sd.js` runs in all major browsers
(IE11+, Chrome, Firefox, Safari), and on the server under
[node.js](https://nodejs.org) and [io.js](https://iojs.org).

Clean, Simple API
-----------------

Running a model, displaying a stock and flow diagram, and visualizing
results on that diagram are all straightfoward.

```Javascript
var drawing, sim;

sd.load('/predator_prey.xmile', function(model) {

    // create a drawing in an existing SVG on the current page.  We
    // can have multiple diagrams for the same model, with different
    // views of the same underlying data.
    drawing = model.drawing('#diagram');

    // create a new simulation.  There can be multiple independent
    // simulations of the same model, running in parallel.
    sim = model.sim();

    // change the size of the initial predator population
    sim.setValue('lynx', 10000);

    sim.runToEnd().then(function() {
        // after completing a full run of the model, visualize the
        // results of this simulation in our stock and flow diagram.
        drawing.visualize(sim);
    }).done();
});
```

High Performance
----------------

Javascript code is dynamically generated and evaluated using modern
web browser's native
[just-in-time (JIT)](https://en.wikipedia.org/wiki/Just-in-time_compilation)
compilers.  This means simulation calculations run as native machine
code, rather than in an interpreter, with industry-leading speed as a
result.

Open Source
-----------

`sd.js` code is offered under the MIT license.  See LICENSE for
detials.  `sd.js` is built on
[q.js](http://documentup.com/kriskowal/q/) (MIT licensed),
[Snap.svg](http://snapsvg.io/) (Apache licensed), and
[mustache.js](https://github.com/janl/mustache.js) (MIT licensed).
These permissive licences mean you can use and build upon `sd.js`
without concern for royalties.

Open Standards
--------------

`sd.js` is built on
[XMILE](https://www.oasis-open.org/committees/tc_home.php?wg_abbrev=xmile),
the emerging open standard for representing system dynamics models.
When the standard is complete, `sd.js` will aim for full conformance,
making it possible to use models created in desktop software platforms
on the web, and models created or modified in `sd.js` on the desktop.

Building
--------

GNU Make, node.js and npm are required to build `sd.js`, as are
some standard unix utilities.  Windows is not supported as a
development platform (patches welcome).  Once those are installed on
your system, you can simply run `make` to build the library for node
and the browser:

```
[bpowers@vyse sd.js]$ make
  TS    build-rt
  RT    src/runtime.ts
  TS    lib
  TS    build
  R.JS  sd.js
  R.JS  sd.min.js
```

Run `make test` to run unit tests, and `make rtest` to run regression
tests against the XMILE models in the
[SDXOrg/test-models](https://github.com/SDXorg/test-models) repository:

```
[bpowers@vyse sd.js]$ make test rtest
  TS    test
  TEST


  lex
    ✓ should lex a
[...]
    ✓ should lex "hares" * "birth fraction"


  32 passing (16ms)

  RTEST test/test-models/tests/number_handling/test_number_handling.xmile
  RTEST test/test-models/tests/lookups/test_lookups_no-indirect.xmile
  RTEST test/test-models/tests/logicals/test_logicals.xmile
  RTEST test/test-models/tests/if_stmt/if_stmt.xmile
  RTEST test/test-models/tests/exponentiation/exponentiation.xmile
time 0.000 mismatch in output (0.0 != 2.0)
time 1.000 mismatch in output (1.0 != 3.0)
time 2.000 mismatch in output (4.0 != 0.0)
time 3.000 mismatch in output (9.0 != 1.0)
time 4.000 mismatch in output (16.0 != 6.0)
  RTEST test/test-models/tests/eval_order/eval_order.xmile
  RTEST test/test-models/tests/comparisons/comparisons.xmile
  RTEST test/test-models/tests/builtin_min/builtin_min.xmile
  RTEST test/test-models/tests/builtin_max/builtin_max.xmile
  RTEST test/test-models/samples/teacup/teacup.xmile
  RTEST test/test-models/samples/teacup/teacup_w_diagram.xmile
  RTEST test/test-models/samples/bpowers-hares_and_lynxes_modules/model.xmile
  RTEST test/test-models/samples/SIR/SIR.xmile
Makefile:121: recipe for target 'rtest' failed
make: *** [rtest] Error 1
```

The standalone sd.js library for use in the browser is available at `sd.js`
and minified at `sd.min.js`, and includes all the dependencies (Snap.svg,
Mustache and Q).  For use under node, `require('sd')` should simply use the
CommonJS modules in the `lib/` directory.

TODO
----

- implement Smooth1/3 and friends
- ability to save XMILE docs
- ignore dt 'reciprocal' on v10 and < v1.1b2 STELLA models
- add-back (basic) isee compat support
- intersection of arc w/ rectangle for takeoff from stock
- intersection of arc w/ rounded-rect for takeoff from module
- logging framework

Vensim TODO
-----------

- parse equations - should be pretty similar to XMILE, except logical
  ops are `:NOT:`, `:OR:`, etc.
- determine types (stocks and flows aren't explicitly defined as
  such. Stocks can be determined by use of the `INTEG` function, and
  flows are variables that are referenced inside of `INTEG` functions.
- read display section
  - read style
  - convert elements to XMILE display concepts.


Arrays TODO
-----------

- diagram changes (minimal)
- figure out if it makes sense to do single-dimensional first, or multi-
  dimensional from the start
- parser:
  - array reference/slicing
  - transpose operator
- semantic analysis/validation:
  - validate indexing
  - validate slicing
  - transpose using dimension names
  - transpose using positions
  - array slicing (`A[1, *]`)
- optimization?
  - the simple thing to do is have a nest of for loops for each individual
    variable.  This is simple, I worry it will be slow for large models (which
    are very common users of arrays).  If we're doing operations on multiple
    variables with the same dimensions in a row, we can merge them into a
    single loop.  This is straightforward logically, but there is no
    optimization framework in place yet, so that would need to be added.
- codegen:
  - apply-to-all equations
    - nested for loop
  - non-A2A equations
  - non-A2A graphical functions
  - array slicing (`A[1, *]`)
  - transpose using dimension names
  - transpose using positions
- runtime:
  - know about defined dimensions + their subscripts
  - allocate correct amount of space for arrayed variables
  - array builtins: `MIN`, `MEAN`, `MAX`, `RANK`, `SIZE`, `STDDEV`, `SUM`.
  - be able to enumerate all subscripted values for CSV output
  - be able to return results for `arrayed_variable[int_or_named_dimension]`
  - right now, the runtime is pretty dead-simple.  Every builtin
    function expects one or more numbers as input.  With the array
    builtins, this is no longer the case.
    - index into arrays with non-constant offsets:
      `constants[INT(RANDOM(1, SIZE(foods)))]`
    - create slices of arrays: `SUM(array[chosen_dim, *])`, where
      `chosen_dim` is an auxiliary variable.
    - this means runtime type checking, and I think runtime memory
      allocation (right now memory is allocated once, in one chunk, when a
      simulation is created, which is fast and optimial).
