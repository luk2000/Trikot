# Gaussian Splat Viewer (PlayCanvas)

A single-file Gaussian Splatting viewer built on PlayCanvas, designed to be embedded via iframe into Wix (or any website).

## Quick Start

1. Place your `.ply` splat file in the same directory as `index.html`
2. Open `index.html` and set `const SPLAT_URL` to your file path or URL
3. Open in a browser – done!

---

## Deploying to GitHub Pages

### Step-by-step

1. **Create a GitHub repository**
   - Go to [github.com/new](https://github.com/new)
   - Name it e.g. `splat-viewer`, set it to **Public**, and create

2. **Upload your files**
   - Click **"Add file" → "Upload files"**
   - Upload `index.html` and your `scene.ply` file
   - Commit to `main`

3. **Enable GitHub Pages**
   - Go to **Settings → Pages**
   - Under "Source", select **Deploy from a branch**
   - Choose `main` branch, `/ (root)` folder
   - Click **Save**

4. **Wait ~1 minute**, then visit:
   ```
   https://<your-username>.github.io/<repo-name>/
   ```

> **Note:** GitHub has a 100 MB file size limit. If your `.ply` is larger, see [Hosting large .ply files](#hosting-large-ply-files) below.

---

## Embedding in Wix

1. In the Wix Editor, click **Add (+) → Embed Code → HTML iframe**
2. Drag the element to the desired location and size
3. Click **"Enter Code"** and paste:

```html
<iframe
  src="https://<your-username>.github.io/<repo-name>/"
  width="100%"
  height="100%"
  style="border: none; display: block;"
  allow="autoplay; fullscreen"
  loading="lazy"
></iframe>
```

4. Replace `<your-username>` and `<repo-name>` with your actual values
5. Resize the iframe element to fit your page layout

### Tips for Wix
- Set the iframe element to a fixed height (e.g. 600px) or use "Stretch to full width"
- If `TRANSPARENT_BG = true`, the Wix section background will show through the viewer
- Test on both desktop and mobile previews

---

## CORS Warning

The `.ply` file **must** be served from one of:

- **Same origin** as the `index.html` (simplest – just put them in the same repo)
- A **CORS-enabled host** that sends `Access-Control-Allow-Origin: *`

If you host the `.ply` on a different domain without proper CORS headers, the browser will block the request and loading will fail.

---

## Hosting Large .ply Files

If your splat file is too large for GitHub (>100 MB), use one of these free options:

| Service | Free tier | CORS | Notes |
|---------|-----------|------|-------|
| **Cloudflare R2** | 10 GB storage, 10 GB/month egress | ✅ Configurable | Best free option. Set CORS in bucket settings. |
| **GitHub LFS** | 1 GB storage, 1 GB/month bandwidth | ✅ | Works with GitHub Pages; bandwidth limits are low. |
| **Backblaze B2** | 10 GB storage, 1 GB/day egress | ✅ Configurable | Pair with Cloudflare CDN for free egress. |
| **Hugging Face Spaces** | Generous limits | ✅ | Upload as a dataset file; use the raw URL. |

After uploading, update `SPLAT_URL` in `index.html` to the full URL of your hosted file.

---

## Configuration

All settings are at the top of the `<script>` block in `index.html`:

| Variable | Default | Description |
|----------|---------|-------------|
| `SPLAT_URL` | `'./scene.ply'` | Path or URL to your .ply or .sog splat file |
| `CAMERA_START_POS` | `[0, 1, 5]` | Initial camera position |
| `CAMERA_LOOK_AT` | `[0, 0, 0]` | Point the camera orbits around |
| `TRANSPARENT_BG` | `true` | Transparent canvas background |
| `BG_COLOR` | `[0.06, 0.06, 0.09]` | Background color when transparency is off |
| `DAMPING` | `0.08` | Orbit smoothing (lower = smoother) |
| `ZOOM_SPEED` | `0.002` | Scroll zoom sensitivity |
| `MIN_DISTANCE` | `0.5` | Closest zoom distance |
| `MAX_DISTANCE` | `50` | Farthest zoom distance |

---

## Controls

| Input | Action |
|-------|--------|
| Left-click drag | Rotate |
| Right-click drag | Pan |
| Scroll wheel | Zoom |
| Touch drag (1 finger) | Rotate |
| Pinch (2 fingers) | Zoom |
| ⟲ button (bottom-right) | Reset camera |
