<p align="center">
    <img src="https://raw.githubusercontent.com/Farama-Foundation/TinyScaler/main/tinyscaler-text.png" width="500px"/>
</p>

A small CPU image scaling library with SIMD support on x86_64 and Arm (Neon). This is planned to be used by Gymnasium and PettingZoo as part of the planned API upgrades to replace OpenCV for image resizing, due to inconveniences and compatibility issues present with OpenCV.

## Requirements

- Python 3

## Install from PyPi

> pip install tinyscaler

## Install from repository

> pip install .

## Usage

```python
import numpy as np
import tinyscaler

img = np.random.rand(64, 64, 4).astype(np.float32)

print(tinyscaler.scale(img, (32, 32)))
```

## Notes

**New since 1.2.0: Simpler Cython build**

**New since 1.1.0: Supports area filtering. It is now the default filtering method as well**

TinyScaler supports mode='area', mode='bilinear', and mode='nearest' filtering. It also allows one to pass a destination buffer in order to avoid duplicate memory allocations.

Area filtering is only really useful for downscaling, bilinear will be used even when area filtering is set if upscaling. Area filtering is also likely not worth it when downscaling less than or equal to 2x.

TinyScaler is used through a single function. The full signature is:

```python
scale(src : np.ndarray, size : tuple, mode='area', dst : np.ndarray = None)
```

TinyScaler expects a contiguous numpy array. If it is not contiguous, it will throw an error. You can make a non-contiguous numpy array contiguous by calling np.ascontiguousarray. Usually a numpy array will already be contiguous.

If the final array dimension is not 4 (RGBA), it will automatically convert to it. Further, if the array is uint8, it will be converted to float32. So the prefered array has a shape (width, height, 4) and dtype=np.float32.

Finally, downscaling is the focus of TinyScaler. It can also upscale, but it will not be as fast as a more complex separable algorithm in that case.

## Performance

In a [simple benchmark](./examples/benchmark.py), we resized the same image (4928x3279) down to (852x567) 100 times using bilinear filtering with several libraries. Here are the times (in seconds) spent (measured with Python's perf_counter) on a AMD 1950x:

```
Time elapsed for tinyscaler: 0.7968465110002398
Time elapsed for OpenCV: 0.48667862100001
Time elapsed for Pillow: 12.672875003999707
Time elapsed for skimage: 164.45401711399973
```

And with area filtering (just TinyScaler and OpenCV):

```
Time elapsed for tinyscaler: 4.34793155800071
Time elapsed for OpenCV: 8.118138265999733
```

All methods were forced to use a single thread. OpenCV is slightly faster than TinyScaler for bilinear filtering, but TinyScaler remains very fast regardless.

Interestingly, for area filtering, TinyScaler is faster (almost 2x).

## License

MIT License, see [LICENSE.md](./LICENSE.md)
