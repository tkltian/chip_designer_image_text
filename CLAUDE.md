# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

This is a web-based chip designer application for creating custom poker chips (or similar circular tokens). The application provides both 2D editing and 3D preview capabilities, allowing users to design the front and back faces of circular chips with custom images and text.

## File Structure

The repository contains two main HTML files:

- `chip_designer_image_only.html` - Image-only editor (working version)
- `chip-designer-8_text_editor_broken.html` - Extended version with text editing capabilities (has issues)

Both files are self-contained single-page applications with inline CSS and JavaScript.

## Application Architecture

### 2D Editor (Canvas-based)

The 2D editor uses HTML5 Canvas for image manipulation:

- **Image Upload & Transform**: Users upload images which can be panned (drag) and zoomed (scroll/pinch)
- **Circular Clipping**: Images are clipped to circular boundaries using `ctx.clip()` with arc paths
- **Transform State**: Each chip side maintains its own transform state (scale, offset)
- **Canvas Export**: The `getCanvasDataURL()` method generates 300x300 PNG exports for download and 3D preview

Key implementation details:
- Transform origin is always centered (`translate(-50%, -50%)`)
- Zoom is constrained between 0.2x and 5x scale
- Both mouse and touch events are supported for mobile compatibility

### 3D Preview (Three.js-based)

The 3D preview dynamically loads Three.js from CDN and creates an interactive 3D chip model:

- **Dynamic Module Loading**: Uses ES6 module imports from Skypack CDN (`three@0.129.0`)
- **Chip Geometry**: Composed of 5 meshes (cylinder, front circle, back circle, 2 beveled edges)
- **Textures**: Generated from the 2D canvas exports via `toDataURL()`
- **Lighting**: Uses directional, ambient, and hemisphere lights for realistic appearance
- **Auto-rotation**: Chip rotates automatically when user is not interacting
- **Materials**: PBR materials with metalness and roughness for realistic rendering

The chip group is oriented with `rotation.x = Math.PI / 2` to show front-facing initially.

### Text Editor (Broken Version)

The text editor version (`chip-designer-8_text_editor_broken.html`) adds:

- Text input, font selection, and text mode (straight/curved) controls
- Interactive text positioning via drag-and-drop
- Text scaling with Shift+scroll
- Curved text rendering along circular arc

**Known Issues**: The canvas rendering logic has bugs preventing proper display/interaction.

## Development Workflow

### Testing Changes

To test either version:

1. Open the HTML file directly in a web browser
2. Or use a local development server:
   ```bash
   python3 -m http.server 8000
   # Then visit http://localhost:8000/chip_designer_image_only.html
   ```

### Making Edits

Since these are single-file applications:
- CSS is in `<style>` tags
- JavaScript is in `<script>` tags
- No build process or dependencies beyond CDN imports

### Debugging

Both files include extensive console logging:
- Look for console output prefixed with emoji indicators (🔹, ✅, ❌)
- Debug canvases are rendered at the bottom showing front/back exported images
- Browser DevTools can inspect canvas state and Three.js scene

## Key Technical Considerations

### Canvas Coordinate Systems

- Container coordinates: 300x300 with (0,0) at center
- Image offsets are relative to centered transform origin
- Text positioning in broken version uses same coordinate system

### Three.js Integration

- Three.js is loaded asynchronously only when user clicks "Generate 3D Preview"
- Textures must be loaded before creating mesh materials
- OrbitControls manages camera interaction
- Animation loop uses `requestAnimationFrame` for smooth rotation

### Touch vs Mouse Events

Both input methods are handled separately:
- Mouse: `mousedown`, `mousemove`, `mouseup`
- Touch: `touchstart`, `touchmove`, `touchend`
- Pinch-to-zoom uses `Math.hypot()` to calculate distance between touch points

## Common Issues

1. **Text Editor Not Working**: The text rendering in `chip-designer-8_text_editor_broken.html` has canvas coordinate bugs
2. **3D Preview Requires Both Images**: Both front and back images must be uploaded before 3D preview works
3. **CDN Dependencies**: Requires internet connection to load Three.js from Skypack CDN
