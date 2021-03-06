![Screenshot](examples/static/karma-benchmark.gif?raw=true)

## 🌩 Installation

```
npm install --save-dev benchmark karma-benchmark
```

## 🕵🏾‍♀️ Reporters

- [karma-benchmark-json-reporter](https://github.com/etpinard/karma-benchmark-json-reporter)
  by [@etpinard](https://github.com/etpinard/)
- [karma-benchmark-plotly-reporter](https://github.com/etpinard/karma-benchmark-plotly-reporter)
  by [@etpinard](https://github.com/etpinard/)
- [karma-benchmarkjs-reporter](https://github.com/FormidableLabs/karma-benchmarkjs-reporter)
  by [@FormidableLabs](https://github.com/FormidableLabs/)
- [karma-benchmark-reporter](https://github.com/lazd/karma-benchmark-reporter)
  by [@lazd](https://github.com/lazd/)

## ⚖️ Configuration

In **karma.conf.js**, add `'benchmark'` to the list of **frameworks**:

```js
module.exports = (config) => {
  config.set({
    autoWatch: false,
    browsers: ['Chrome'],
    concurrency: 1,
    files: ['bench/**/*.bench.js'],
    frameworks: ['benchmark'],
    singleRun: true
  });
};
```

### Terminal Reporting

Now let's add
[`karma-benchmarkjs-reporter`](https://github.com/FormidableLabs/karma-benchmarkjs-reporter)
by [@FormidableLabs](https://github.com/FormidableLabs/) to report results to
the Terminal:

```
npm install --save-dev karma-benchmarkjs-reporter
```

In **karma.conf.js**, add `'benchmark'` to the list of **reporters**:

```diff
module.exports = (config) => {
  config.set({
    autoWatch: false,
    browsers: ['Chrome'],
    concurrency: 1,
    files: ['bench/**/*.bench.js'],
    frameworks: ['benchmark'],
+   reporters: ['benchmark'],
    singleRun: true
  });
};
```

### JUnit Reporting

To feed our data into Continuous Integration, we can use the
[`karma-junit-reporter`](https://github.com/karma-runner/karma-junit-reporter).

```
npm install --save-dev karma-junit-reporter
```

In **karma.conf.js**, add `junit` to the list of **reporters** and configure the
reporter accordingly:

```diff
module.exports = (config) => {
  config.set({
    autoWatch: false,
    browsers: ['Chrome'],
    concurrency: 1,
    files: ['bench/**/*.bench.js'],
    frameworks: ['benchmark'],
+   junitReporter: {
+     outputDir: 'reports',
+     outputFile: 'benchmark.xml'
+   },
-   reporters: ['benchmark'],
+   reporters: ['benchmark', 'junit'],
    singleRun: true
  });
};
```

### Data Visualisation Reporting

With a free [plot.ly](https://plot.ly) account, we can generate visualisations
using the
[karma-benchmark-plotly-reporter](https://github.com/etpinard/karma-benchmark-plotly-reporter)
by [@etpinard](https://github.com/etpinard/).

```
npm install --save-dev karma-benchmark-plotly-reporter
```

In **karma.conf.js**, add `benchmark-plotly` to the list of **reporters** and
configure the reporter accordingly:

```diff
module.exports = (config) => {
  config.set({
    autoWatch: false,
+   benchmarkPlotlyReporter: {
+     username: '<your username>',
+     apiKey: '<your api key>',
+     cloudFilename: 'plotly-example',
+     imageFilename: 'plotly-example.png'
+   },
    browsers: ['Chrome'],
    concurrency: 1,
    files: ['bench/**/*.bench.js'],
    frameworks: ['benchmark'],
    junitReporter: {
      outputDir: 'reports',
      outputFile: 'benchmark.xml'
    },
-   reporters: ['benchmark', 'junit'],
+   reporters: ['benchmark', 'benchmark-plotly', 'junit'],
    singleRun: true
  });
};
```

## 👩🏻‍🔬 Writing Benchmarks

Benchmarks can be written by using the original
[Benchmark.js](https://benchmarkjs.com) API, but a wrapper API is also provided
by karma-benchmark in the form of the `suite` and `benchmark` globals. The
karma-benchmark API aims to make the process of writing Benchmarks feel familiar
to users of [Jasmine](https://jasmine.github.io/) or [Jest](https://jestjs.io/).

In this example, a suite is defined that pits `_.each` against the native
`Array.forEach` method:

```js
suite('Array iteration', () => {
  benchmark('_.each', () => {
    _.each([1, 2, 3], (el) => {
      return el;
    });
  });

  benchmark('native forEach', () => {
    [1, 2, 3].forEach((el) => {
      return el;
    });
  });
});
```

### Suite options

Suite options are the same as in Benchmark.js with one exception: `onStart` and
`onComplete` can be set at the suite level.

See the
[Benchmark.js Suite constructor API docs](http://benchmarkjs.com/docs#Suite) for
a full list of options.

```js
suite(
  'Array iteration',
  () => {
    benchmark('_.each', () => {
      _.each(this.list, (number) => {
        return number;
      });
    });

    benchmark('native forEach', () => {
      this.list.forEach((number) => {
        return number;
      });
    });
  },
  {
    onCycle(event) {
      var suite = this;
      var benchmark = event.target;
      console.log('Cycle completed for ' + suite.name + ': ' + benchmark.name);
    },
    onStart() {
      this.list = [5, 4, 3];
    },
    onComplete() {
      this.list = null;
    }
  }
);
```

### Benchmark options

Benchmark options are the same as in Benchmark.js. If `setup` and `teardown` are
passed to `benchmark()`, they will override `setup` and `teardown` from the
suite. Pass `null` or undefined to remove them.

See the
[Benchmark.js Benchmark constructor API docs](http://benchmarkjs.com/docs#Benchmark)
for a full list of options.

```js
suite('Iteration', () => {
  benchmark(
    '_.each with array',
    () => {
      _.each(this.list, (number) => {
        return number;
      });
    },
    {
      setup() {
        this.list = ['a', 'b', 'c'];
      },
      teardown() {
        delete this.list;
      }
    }
  );

  benchmark(
    '_.each with object',
    () => {
      _.each(this.list, (number) => {
        return number;
      });
    },
    {
      setup() {
        this.list = { 0: 'a', 1: 'b', 2: 'c' };
      },
      teardown() {
        delete this.list;
      }
    }
  );
});
```

### Running only a specific benchmark or suite

To run only a specific benchmark, use `benchmark.only()` or `bbenchmark()`
instead of `benchmark()`:

```js
benchmark.only(() => {
  // Only this benchmark will run
  // bbenchmark() does the same thing
});

benchmark(() => {
  // This benchmark won't run
});
```

The same applies to suites with `suite.only()` and `ssuite()`.

### Skipping benchmarks & suites

To skip a benchmark, use `benchmark.skip()` or `xbenchmark()` instead of
`benchmark()`:

```js
benchmark.skip(() => {
  // This benchmark won't run
  // xbenchmark() does the same thing
});

benchmark(() => {
  // This and all other benchmarks will run
});
```

The same applies to suites with `suite.skip()` and `xsuite()`.
