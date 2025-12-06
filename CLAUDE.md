# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

This is a Streamlit custom component that provides a sketching canvas using Fabric.js. It consists of two parts:
- **Python package** (`streamlit_drawable_canvas/`): Provides the `st_canvas` function that Streamlit apps call
- **React/TypeScript frontend** (`streamlit_drawable_canvas/frontend/`): The actual canvas component built with Fabric.js

## Development Commands

### Frontend (JavaScript/TypeScript)
```bash
cd streamlit_drawable_canvas/frontend
npm install          # Install dependencies
npm run start        # Start webpack dev server (port 3001)
npm run build        # Production build
npm run test         # Run tests
```

### Python
```bash
pip install -e .                    # Install package in editable mode
streamlit run app.py                # Run the demo app (requires frontend running in dev)
```

### Cypress E2E Tests
```bash
cd e2e && npm install               # Install Cypress
cd streamlit_drawable_canvas/frontend && npm run start  # Start frontend
streamlit run e2e/app_to_test.py    # Start test app
cd e2e && npm run cypress:open      # Open Cypress
```

**Dev mode toggle**: Set `_RELEASE = False` in `streamlit_drawable_canvas/__init__.py` to use the local dev server instead of the built frontend.

## Architecture

### Python Side
- `streamlit_drawable_canvas/__init__.py`: Main entry point containing `st_canvas()` function
- `CanvasResult` dataclass: Returns `image_data` (numpy array) and `json_data` (Fabric.js JSON)
- Background images are resized to canvas dimensions and converted to URLs via Streamlit's image manager

### React/TypeScript Side
- `DrawableCanvas.tsx`: Main component, receives props from Python, manages Fabric.js canvas
- `DrawableCanvasState.tsx`: Canvas state management with undo/redo history
- `lib/` directory: Drawing tool implementations
  - `fabrictool.ts`: Abstract base class `FabricTool` all tools extend
  - Each tool (`freedraw.ts`, `line.ts`, `rect.ts`, `circle.ts`, `point.ts`, `polygon.ts`, `transform.ts`) implements `configureCanvas()` to set up canvas event listeners
- `components/CanvasToolbar.tsx`: Undo/redo/reset toolbar
- `components/UpdateStreamlit.tsx`: Handles sending canvas data back to Python

### Drawing Mode System
Tools are registered in `lib/index.ts` and selected via the `drawingMode` prop. Each tool:
1. Extends `FabricTool`
2. Implements `configureCanvas(props)` which sets up mouse event handlers
3. Returns a cleanup function to remove event listeners

### Data Flow
1. Python `st_canvas()` passes args to React component
2. User draws on Fabric.js canvas
3. On mouse up (or realtime), canvas state is serialized
4. `UpdateStreamlit` component sends image data URL + JSON back to Python
5. Python returns `CanvasResult` with numpy array and JSON data
