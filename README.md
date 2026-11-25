# Choose a Gray Bit Depth (Group 18)

A normal grayscale image uses 8 bits per pixel (256 gray levels). This project tests how few bits we can use before the image of a cat gets too damaged. We measure the damage with the **mean absolute error (MAE)** against the 8-bit image.

## Result

| Bits | Gray levels | MAE (full image) |
|---|---|---|
| 2 | 4 | 22.86 |
| 3 | 8 | 9.21 |
| 4 | 16 | 4.24 |

**We chose 3 bits.** It is the smallest bit depth with an MAE under the threshold.

## How it works

The work is a pipeline of three stages. Each member does one stage in [main.ipynb](main.ipynb).

### Member A: grayscale conversion

The color image `cat.png` (492 × 327 pixels) is turned into grayscale in two ways:

- **Channel average:** `gray = (R + G + B) / 3`
- **Weighted:** `gray = 0.299·R + 0.587·G + 0.114·B` (OpenCV `cv2.COLOR_BGR2GRAY`)

A crop shows both versions side by side, plus a difference image (`weighted − avg`) in a blue–white–red colormap. Red means weighted is brighter, blue means average is brighter, and white means no difference. Over the full image, the two methods differ by 4.57 gray levels on average.

The **weighted** image is the reference for stages B and C.

### Member B: quantization

With `b` bits there are `2^b` evenly spaced gray levels from 0 to 255. Each pixel moves to the nearest level, then gets converted back to the 0–255 scale:

```python
step  = 255 / (2**b - 1)
recon = np.round(img / step) * step
```

This is done for 2, 3 and 4 bits.

### Member C: choose the bit depth

The MAE is computed over the **full image** for each bit depth:

```python
MAE = mean(|weighted_gray − reconstruction|)
```

A plot shows MAE against bits, with the threshold drawn as a red dashed line. A crop compares the 3-bit image with the 8-bit reference, so the loss of smooth shading (banding) is easy to see.

## How to run

Requires Python 3.11.

```bash
python3 -m venv venv
source venv/bin/activate
pip install -r requirements.txt
```

Open `main.ipynb` in Jupyter or VS Code, select the `venv` kernel, and run all cells from top to bottom. `cat.png` must be in the same folder as the notebook.

## Files

| File | What it is |
|---|---|
| `main.ipynb` | All code and plots for stages A, B and C |
| `cat.png` | Input image |
| `requirements.txt` | Python packages |

## Notes

- Images are converted to `int` or `float` before subtracting. Subtracting two `uint8` arrays wraps around (for example, 3 − 5 = 254).
- All grayscale plots use `vmin=0, vmax=255`, so the same value always shows as the same gray in every panel.
