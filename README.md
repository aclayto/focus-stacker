# Focus Stacker

A browser-based focus stacking tool designed for microscopy imaging workflows. Combine multiple images taken at different focal planes into a single, fully-focused composite image.

## What is Focus Stacking?

When imaging specimens under a microscope, depth of field is often very shallow—only a thin slice of the sample appears sharp at any given focus setting. **Focus stacking** solves this by:

1. Capturing multiple images at different focal distances (a "focus stack")
2. Analyzing which regions of each image are sharpest
3. Combining the in-focus regions into a single composite

The result is an image with extended depth of field where the entire specimen appears in focus.

---

## How to Use

### Input Options

**Multiple Images**: Drop or select 2+ images taken at different focal planes. The tool will analyze and merge them.

**Single Video**: Drop a video where you've focused through the specimen. The tool will extract frames at regular intervals, then stack them.

> ⚠️ You cannot mix images and video. Choose one input type per session.

### Workflow

1. **Load your images** (drag & drop or click to browse)
2. **Adjust settings** if needed (see below)
3. **Click "Stack Images"** to process
4. **Download** or **Compare** your result

---

## Algorithm Settings

### Focus Detection Algorithms

The algorithm determines how the tool identifies which pixels are "in focus." Each method calculates a **focus measure** for every pixel—higher values indicate sharper/more in-focus regions.

#### Laplacian (Recommended)

```
Focus = |∇²I| = |∂²I/∂x² + ∂²I/∂y²|
```

The **Laplacian** operator measures the second derivative of image intensity. In-focus regions have rapid intensity changes (sharp edges), producing high Laplacian values. Out-of-focus regions are blurry with gradual transitions, producing low values.

**Best for**: General use, most microscopy applications  
**Characteristics**: Fast, robust, good edge detection

#### Sobel Gradient

```
Focus = √(Gx² + Gy²)
```

The **Sobel** operator calculates the gradient magnitude using directional filters. It measures the rate of intensity change (first derivative) in both horizontal and vertical directions.

**Best for**: Images with strong directional features  
**Characteristics**: Emphasizes edges, slightly less sensitive to noise than Laplacian

#### Local Variance

```
Focus = σ² = E[(I - μ)²]
```

**Local Variance** measures the statistical spread of pixel intensities within a neighborhood. Sharp, in-focus regions have high local contrast (high variance), while blurry regions have uniform intensity (low variance).

**Best for**: Specimens with fine texture details, low-contrast samples  
**Characteristics**: More computationally intensive, excellent for textured surfaces

---

### Kernel Size

**Range**: 3–15 (odd numbers only)  
**Default**: 5

The kernel size defines the **neighborhood window** used when calculating focus measures. It determines how many surrounding pixels are considered when evaluating sharpness at each point.

| Kernel Size | Effect |
|-------------|--------|
| **Small (3–5)** | Fine detail detection, sensitive to noise, faster processing |
| **Medium (7–9)** | Balanced detail and noise resistance |
| **Large (11–15)** | Smoother focus detection, better noise resistance, may miss fine details |

**When to adjust**:
- **Increase** if your images are noisy or have very fine, uniform texture
- **Decrease** if you need to preserve very fine structural details

---

### Blend Radius

**Range**: 5–50 pixels  
**Default**: 15px

The blend radius controls how **smoothly** the tool transitions between in-focus regions from different source images. It applies Gaussian smoothing to the focus maps before blending.

| Blend Radius | Effect |
|--------------|--------|
| **Small (5–10px)** | Sharp transitions, may show visible seams between regions |
| **Medium (15–25px)** | Natural blending, good balance |
| **Large (30–50px)** | Very smooth transitions, may reduce local sharpness |

**When to adjust**:
- **Increase** if you see harsh boundaries or "patchwork" artifacts in the result
- **Decrease** if the result looks too soft or loses fine detail

---

## Technical Details

### How Blending Works

1. **Focus Map Calculation**: For each input image, a focus map is computed where each pixel's value represents its local sharpness (using the selected algorithm).

2. **Focus Map Smoothing**: Gaussian blur is applied to focus maps (controlled by Blend Radius) to prevent harsh transitions.

3. **Weighted Blending**: The final image is computed as a weighted average:
   ```
   Result(x,y) = Σ(Image_i(x,y) × Weight_i(x,y)) / Σ(Weight_i(x,y))
   ```
   Where weights are derived from squared focus values (emphasizing the sharpest regions).

### Supported Formats

**Images**: PNG, JPG, JPEG, TIFF, WebP, BMP, GIF  
**Video**: MP4, MOV, WebM, AVI (browser-dependent)  
**Output**: PNG (lossless)

### Performance Notes

- Processing happens entirely in your browser—no data is uploaded anywhere
- Large images (>4000px) may take longer to process
- Video frame extraction quality depends on video codec and browser support
- For best results with video, use high-quality recordings with minimal compression

---

## Tips for Best Results

### Capturing Your Focus Stack

1. **Use a tripod or stable mount** — alignment issues reduce quality
2. **Consistent lighting** — avoid changing exposure between shots
3. **Even focal steps** — space your focus positions evenly through the specimen
4. **Sufficient overlap** — ensure some regions are sharp in multiple images
5. **5–15 images** is typically sufficient for most specimens

### Troubleshooting

| Problem | Solution |
|---------|----------|
| Result looks blurry overall | Try a smaller kernel size or blend radius |
| Visible seams/patches | Increase blend radius |
| Noise in result | Increase kernel size, or pre-process images with denoising |
| Color shifts | Ensure consistent white balance across source images |
| Missing details | Some regions may not be in focus in any source image—capture more focal planes |

---

## Browser Compatibility

Works in all modern browsers:
- Chrome/Edge 80+
- Firefox 75+
- Safari 13+
- iOS Safari (when hosted - see below)
- Android Chrome

---

## Hosting for iOS/Mobile Use

HTML files don't run properly when opened directly on iOS. You need to host the files on a web server. Here are your options:

### Option 1: GitHub Pages (Free, Recommended)

1. Create a GitHub account (if you don't have one)
2. Create a new repository
3. Upload these files: `focus-stacker.html`, `manifest.json`
4. Go to Settings → Pages → Enable GitHub Pages
5. Your app will be at: `https://yourusername.github.io/reponame/focus-stacker.html`

### Option 2: Local Server (for testing)

On your Mac, run this in Terminal from the Focus Stacker folder:
```bash
python3 -m http.server 8080
```

Then on your iPhone (same WiFi network):
- Open Safari and go to `http://YOUR-MAC-IP:8080/focus-stacker.html`
- Find your Mac's IP in System Preferences → Network

### Option 3: Any Web Host

Upload the files to any web hosting service (Netlify, Vercel, your own server, etc.)

---

## Installing as an App (iOS/Android)

Once hosted on a web server, you can "install" Focus Stacker as an app:

### iOS (Safari)
1. Open the hosted URL in Safari
2. Tap the Share button (square with arrow)
3. Scroll down and tap "Add to Home Screen"
4. Tap "Add"

### Android (Chrome)
1. Open the hosted URL in Chrome
2. Tap the menu (three dots)
3. Tap "Add to Home screen" or "Install app"

The app will then appear on your home screen and run in full-screen mode like a native app.
