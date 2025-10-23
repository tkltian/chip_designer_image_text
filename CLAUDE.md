# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

This is a web-based chip designer application for creating custom poker chips (or similar circular tokens). The application provides both 2D editing and 3D preview capabilities, allowing users to design the front and back faces of circular chips with custom images and text.

## File Structure

The repository contains three main HTML files:

- `index.html` - Main application for GitHub Pages (full-featured with text editor)
- `chip_designer_with_text.html` - Backup copy of full-featured version
- `chip_designer_image_only.html` - Image-only editor (no text features)

Additional assets:
- `chip_3d_model/` - Contains 3D models for chip preview
  - `model2.glb` - GLB format 3D chip model (329KB) used for preview
  - Note: Files must NOT start with underscore (_) as GitHub Pages/Jekyll ignores them

All HTML files are self-contained single-page applications with inline CSS and JavaScript.

## Application Architecture

### 2D Editor (Canvas-based)

The 2D editor uses HTML5 Canvas for image manipulation:

- **Image Upload & Transform**: Users upload images which can be panned (drag) and zoomed (scroll/pinch)
- **Circular Clipping**: Images are clipped to circular boundaries using `ctx.clip()` with arc paths
- **Transform State**: Each chip side maintains its own transform state (scale, offset)
- **Canvas Export**: The `getCanvasDataURL()` method generates 300x300 PNG exports for download and 3D preview

Key implementation details:
- Transform origin is always centered (`translate(-50%, -50%)`)
- Zoom is constrained between 0.05x (5%) and 5x (500%) scale
- Minimum zoom of 5% allows large phone photos to fit inside the circle
- Both mouse and touch events are supported for mobile compatibility

### Text Editor Features

The full-featured version includes comprehensive text editing capabilities:

#### Text Input & Formatting
- **Multi-line support**: Textarea allows multiple lines of text (for straight mode)
- **13 font options**: Arial, Verdana, Times New Roman, Courier New, Georgia, Impact, Comic Sans MS, Trebuchet MS, Palatino, Garamond, Bookman, Tahoma, Lucida Console
- **Color picker**: Full RGB color selection for text
- **Size control**: Slider from 12px to 72px
- **Character spacing**: Adjustable from -0.5 to 2.0 (multiplier of font size), default 0

#### Text Modes
1. **Straight**: Horizontal centered text with multi-line support
   - Each line is independently centered
   - Line spacing is 1.2x font size
2. **Curved Upper**: Text curves along top arc of chip
   - Arc angle dynamically calculated based on character widths and spacing
   - Centered at top (-90°)
3. **Curved Lower**: Text curves along bottom arc of chip
   - Arc angle dynamically calculated (same as upper)
   - Centered at bottom (+90°)
   - Characters face upward for readability

#### Text Effects (3D Styling)
- **No Effect**: Classic white stroke with colored fill
- **Drop Shadow**: Semi-transparent black shadow offset down-right
- **Embossed**: 3D carved effect with dark shadow and light highlight
- **Bold Outline**: Thick black outline for maximum visibility
- **Glow**: Color-matched glowing halo effect

#### Text Positioning
- **Drag-and-drop**: Click and drag text to reposition
- **Hover feedback**: Visual bounding box (straight) or arc guide (curved) appears on hover
- **Cursor changes**: "move" cursor when hovering over text
- **Touch support**: Full mobile support for text dragging

### Dynamic Arc Rendering (Curved Text)

Important algorithm for curved text modes:

```javascript
// Calculate total arc length needed
let totalArcLength = sum of all character widths + spacing
// Convert to angle: arc_length = radius × angle
let totalAngle = totalArcLength / radius
// Cap at 270° to prevent wrapping too far
totalAngle = Math.min(totalAngle, Math.PI * 1.5)
// Center the arc (top or bottom)
const startAngle = centerAngle - totalAngle/2
```

This ensures:
- No character overlap regardless of font size
- Text remains centered on the chip
- Arc expands/contracts based on actual text width
- Spacing control affects arc length

### Casino Chip Value Buttons (Back Side Only)

For the back side, users can optionally load pre-generated casino chip value images:

- **Value buttons**: 1, 5, 10, 25, 100 (no dollar signs in UI)
- **Image generation**: `generateCasinoChipImage(value)` creates transparent PNG with single centered number
- **Font style**: Italic bold Georgia serif for elegant casino aesthetic
- **Colors**: Value-based colors (1=dark gray, 5=dark red, 10=dark blue, 25=dark green, 100=black)
- **Size adjustment**: 100 uses smaller font (150px vs 180px) to fit in circle
- **Styling**: Subtle drop shadow for depth
- **Integration**: Uses `loadImageFromDataURL()` method to load generated image into back side canvas
- **Text overlay**: Users can still add custom text on top of casino value images

### UI Layout

Text editor controls are organized into 4 rows:
1. **Text input**: Multi-line textarea
2. **Font & Style**: Font selector, text mode dropdown, color picker
3. **Size & Spacing**: Range sliders with live value displays
4. **Effects**: Dropdown for 3D text effects

Additionally, the back side includes:
- **Casino value buttons**: Row of buttons (1, 5, 10, 25, 100) below file upload
- **Active state styling**: Green highlight when value is selected

### 3D Preview (Three.js-based)

The 3D preview dynamically loads Three.js from CDN and creates an interactive 3D chip model:

- **Dynamic Module Loading**: Uses ES6 module imports from Skypack CDN (`three@0.129.0`)
- **GLTFLoader**: Loads `chip_3d_model/model2.glb` (GLB format 3D model)
- **Model Orientation**: Rotates model `rotation.x = Math.PI / 2` to orient z-axis as y-axis
- **Custom Textures**: Front/back custom images overlaid as circles with `innerCircleRadius = 0.57` to fit embossed area
- **Textures**: Generated from the 2D canvas exports via `toDataURL()`
- **Lighting**: Uses directional, ambient, and hemisphere lights for realistic appearance
- **Auto-rotation**: Chip rotates automatically (`rotation.z += 0.005`) when user is not interacting
- **Materials**: PBR materials with metalness and roughness for realistic rendering
- **OrbitControls**: Allows user to pan/zoom the 3D view (pan disabled, distance limited 2-6 units)

The chip group is initially tilted `rotation.x = Math.PI * 0.15` (27 degrees) to show front face to user.

## Development Workflow

### Testing Changes

To test the application:

1. Open the HTML file directly in a web browser
2. Or use a local development server:
   ```bash
   python3 -m http.server 8000
   # Then visit http://localhost:8000/index.html
   ```

### Making Edits

Since these are single-file applications:
- CSS is in `<style>` tags
- JavaScript is in `<script>` tags
- No build process or dependencies beyond CDN imports
- Edit `index.html` and copy to `chip_designer_with_text.html` to keep them synced

### Debugging

Files include extensive console logging:
- Look for console output prefixed with emoji indicators (🔹, ✅, ❌)
- Debug canvases are rendered at the bottom showing front/back exported images
- Browser DevTools can inspect canvas state and Three.js scene

## Key Technical Considerations

### Canvas Coordinate Systems

- Container coordinates: 300x300 with (0,0) at center
- Image offsets are relative to centered transform origin
- Text positioning uses same centered coordinate system
- Curved text uses polar coordinates (angle, radius)

### Text Rendering with Effects

The `drawTextWithEffect(text, x, y)` helper function:
- Applies selected visual effect before drawing character
- Uses Canvas shadow API for glow effect
- Uses multiple offset fills for emboss/shadow effects
- Maintains current transform (works in rotated contexts for curved text)

### Character-by-Character Rendering

For precise spacing control, straight and curved text both render character-by-character:
- Measures each character width with `ctx.measureText()`
- Positions each character individually
- Allows custom spacing between characters
- Enables per-character effects and transformations

### Three.js Integration

- Three.js is loaded asynchronously only when user clicks "Generate 3D Preview"
- Textures must be loaded before creating mesh materials
- OrbitControls manages camera interaction
- Animation loop uses `requestAnimationFrame` for smooth rotation

### Touch vs Mouse Events

Both input methods are handled separately:
- Mouse: `mousedown`, `mousemove`, `mouseup` with wheel zoom
- Touch: `touchstart`, `touchmove`, `touchend` with pinch-to-zoom
- Pinch-to-zoom uses `Math.hypot()` to calculate distance between touch points
- Touch events include `e.preventDefault()` to prevent default browser behaviors

## Common Patterns

### Adding New Text Controls

1. Add HTML control in text-editor section (organized by row)
2. Query the element in `setupChipEditor()` function
3. Add event listener that calls `drawCanvas()`
4. Use the value in `drawCanvas()` or `drawTextWithEffect()`

### Modifying Text Rendering

The rendering flow is:
1. `drawCanvas()` sets up canvas context and font
2. For each character position, calls `drawTextWithEffect(char, x, y)`
3. `drawTextWithEffect()` applies effects based on dropdown selection
4. Character is rendered with current transform/rotation

### Syncing Files

When making changes to `index.html`:
```bash
cp index.html chip_designer_with_text.html
```

## Publishing

The application is designed for GitHub Pages:
- `index.html` is automatically served as the homepage
- No build process required
- All dependencies loaded from CDN
- Works entirely client-side (no server needed)

### Important GitHub Pages Considerations

**Jekyll File Naming**: GitHub Pages uses Jekyll which ignores files/folders starting with underscore (_)
- ❌ `_model2.glb` - Will cause 404 errors on GitHub Pages
- ✅ `model2.glb` - Works correctly
- Always avoid underscore prefixes for assets that need to be served

**Testing**:
- Local server (e.g., `python3 -m http.server`) does NOT use Jekyll, so files with underscores work locally
- Always test on actual GitHub Pages URL to catch Jekyll-specific issues
- 404 errors on GitHub Pages but not locally often indicate underscore filename issue

## Branding

- **Title**: "LT Innovations Shop"
- **Subtitle**: "2D + 3D Chip Design Preview"
- **Copyright**: LT Innovations LLC, GitHub: https://github.com/tkltian
- **Privacy Notice**: Green banner explaining all processing is local (no server uploads)
- **Edge Color Default**: Green (#2d8e3e) to match branding
