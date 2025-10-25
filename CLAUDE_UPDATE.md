# Additional Section for Canvas Transparency

### Canvas Transparency

- Canvas context created with `{ alpha: true }` to support transparency
- `ctx.clearRect()` clears to transparent instead of white
- Circular clipping creates transparent areas outside the circle
- PNG exports preserve alpha channel for downloads and 3D preview
- Critical for background removal feature to work properly
