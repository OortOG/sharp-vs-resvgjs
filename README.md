# sharp 0.33.5 vs resvg-js 2.6.2

A comparison between [sharp](https://github.com/lovell/sharp) vs [resvg-js](https://github.com/yisibl/resvg-js) performance to bulk convert SVGs to PNGs.

Sharp configuration: [`scripts/benchmark/sharp.js`](/scripts/benchmark/sharp.js)

rsvg-js configuration: [`scripts/benchmark/resvg.js`](/scripts/benchmark/resvg.js)


## Reproduction

1. Install dependencies:

	```sh
	pnnpm i
	```

2. Benchmark:

	```sh
	pnpm benchmark
	```

## Results

### sharp is _much_ faster

When converting 400 SVG icons from [simple-icons](https://www.npmjs.com/package/simple-icons) to 2500 DPI 800px width PNGs, sharp is 3x faster than resvg-js.

```
resvg: { duration: '5217ms', icons: 400 }
sharp: { duration: '768ms', icons: 400 }
sharp is faster by 6.79x
```

### resvg-js crashes on too many icons [UPDATED]

Previously it was reported that the number of icons was [limited to 400](/scripts/benchmark/index.js#L13). This was  because when processing too many icons at once, resvg-js crashes the whole Node.js process with the following error:

```
$ pnpm benchmark

> node scripts/benchmark

thread '<unnamed>' panicked at 'the previous segment must be M/L/C', /Users/runner/.cargo/git/checkouts/resvg-4b7e4ee32ad6d954/cab0b15/usvg/src/pathdata.rs:160:17
note: run with `RUST_BACKTRACE=1` environment variable to display a backtrace
fatal runtime error: failed to initiate panic, error 5
```

However, as of 20241125 with resvg-js 2.6.2 on Apple silicon, we have not been able to reproduce this issue. 

### rsvg-js doesn't yield expected DPI
After benchmarking, the resulting PNGs are saved to `pngs/`. When comparing icons between sharp and rsvg, the DPI on sharp is 2400 (as expected), but the DPI on resvg-js is 72.

Zoom in to compare difference


| resvg-js - 11.1 KB - 72 DPI | sharp - 16.6 KB - 2400 DPI |
| - | - |
| <img src="./pngs/resvg/500px.svg.png"> | <img src="./pngs/sharp/500px.svg.png"> |

NOTE: __While the DPI issue demonstrated remains in these test results, this tester has not explored any improvements to resvg-js since the initial tests using v 2.1.0 that would improve output quality and DPI specifically.__
