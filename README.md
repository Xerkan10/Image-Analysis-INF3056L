# Image-Analysis-INF3056L

📄 You can find the full report in french [rapport_analyse.pdf](rapport_analyse.pdf).


This README summarizes the steps of a computer vision notebook that detects the contours of puzzle pieces using OpenCV and Python libraries.

---

## 📦 Library Imports and Image Loading

The notebook begins by importing key libraries:
- `cv2` (OpenCV) for image processing
- `numpy` for matrix operations
- `matplotlib.pyplot` for visualization

The image containing puzzle pieces is loaded with `cv2.imread`, converted from BGR to RGB using `cv2.cvtColor`, and further converted to HSV for easier color-based operations.

---

## 🧭 Region of Interest (ROI) and Histogram

A **Region of Interest (ROI)** is manually selected from the background (often orange). From this area, a **2D histogram** of Hue and Saturation components is computed with `cv2.calcHist` and normalized using `cv2.normalize`.

This histogram represents the background color distribution and is used to identify similar areas across the image.

---

## 🎯 Backprojection and Binary Mask Generation

The histogram is backprojected on the entire HSV image using `cv2.calcBackProject`, producing a grayscale image where background-like pixels are lighter. The image is blurred (`cv2.filter2D` or `cv2.GaussianBlur`) and then thresholded with `cv2.threshold` to generate a **binary mask**.

In some cases, an inverted threshold (`cv2.THRESH_BINARY_INV`) is used.

---

## 🧼 Mask Cleaning with Morphology

To clean the binary mask, **morphological operations** are used:
- `cv2.getStructuringElement` creates an elliptical kernel.
- `cv2.morphologyEx` with `MORPH_OPEN` removes noise.
- `MORPH_CLOSE` fills small holes in white areas.

This results in a cleaner separation between background and objects.

---

## 🖼️ Mask Application and Background Segmentation

The binary mask is applied to the original image using `cv2.bitwise_and` to isolate only the background. This shows which parts of the image were considered background.

Multiple images are displayed side-by-side:
- Original image
- Backprojection
- Binary mask
- Segmented background (black except for background)

---

## 🪞 Mask Refinement Using Intensity (Grayscale)

An alternative approach refines the binary mask using **grayscale intensity**:
- Converts the image to grayscale or extracts the V channel (brightness) from HSV.
- Applies intensity thresholds (`threshold_min = 100`, `threshold_max = 140`).
- Creates a second binary mask using these thresholds.

Morphological operations (`cv2.erode`, `cv2.dilate`) are applied to remove small artifacts and close holes.

---

## 🔁 Mask Inversion and Connected Component Detection

To prepare for object identification:
- The refined mask is inverted with `cv2.bitwise_not`.
- `cv2.connectedComponentsWithStats` is used to label each connected white object (puzzle piece).

Each label comes with statistics like bounding box and area. Components below a minimum area (`min_area = 2000`) are discarded.

---

## 🔍 Visualization of Identified Puzzle Pieces

Two visualizations are generated:
1. A color-coded image where each puzzle piece is given a unique random color using `np.random.randint`.
2. The original image with red **bounding boxes** (`patches.Rectangle`) and labels (e.g., “ID 3”) added using Matplotlib.

---

## 💾 Individual Piece Extraction and Saving

For each valid component:
- Extracts its **bounding box** `(x, y, w, h)`
- Builds a **binary mask** for that piece
- Extracts the original color patch
- Applies bitwise masking with `cv2.bitwise_and` to isolate the piece

These are stored in a dictionary `puzzle_pieces`, indexed by piece ID, with their mask, bounding box, cropped image, and area.

---

## ✏️ Contour Detection of Each Piece

Contours of each isolated piece are found using:
```python
cv2.findContours(piece_mask, cv2.RETR_EXTERNAL, cv2.CHAIN_APPROX_SIMPLE)
```
- `RETR_EXTERNAL` extracts only outer contours.
- `CHAIN_APPROX_SIMPLE` simplifies contour storage.

Contours are drawn onto the masked piece using `cv2.drawContours`.

---

This process successfully isolates and outlines individual puzzle pieces for analysis.

## 🚀 Future Improvements

A next step would be to implement a classification module to categorize puzzle pieces based on their shapes. For example:

- **Corner pieces**: pieces with two straight edges
- **Edge pieces**: pieces with one straight edge
- **Middle pieces**: pieces with no straight edges

This classification could be based on the contour geometry or edge detection results, improving both visualization and puzzle-solving automation.


