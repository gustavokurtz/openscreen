# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

OpenScreen is an open-source screen recording and video editing desktop application, built as a simpler alternative to Screen Studio. It enables recording screens/windows and creating polished product demos with zooms, annotations, and effects.

## Development Commands

```bash
# Install dependencies
npm install

# Development mode (hot reload)
npm run dev

# Build for current platform
npm run build

# Platform-specific builds
npm run build:mac
npm run build:win
npm run build:linux

# Linting
npm run lint

# Run tests
npm test

# Run tests in watch mode
npm run test:watch
```

## Architecture

### Electron Process Model

The app uses three window types managed by the Electron main process:

1. **HUD Overlay** (`?windowType=hud-overlay`) - Frameless recording control bar
2. **Source Selector** (`?windowType=source-selector`) - Screen/window picker dialog
3. **Editor** (`?windowType=editor`) - Main video editing interface

Window routing happens in `src/App.tsx` based on URL search params.

### Directory Structure

```
electron/           # Main process
├── main.ts         # Window lifecycle, tray management
├── preload.ts      # IPC bridge (window.electronAPI)
├── windows.ts      # Window factory functions
└── ipc/handlers.ts # IPC request handlers

src/                # Renderer process (React)
├── components/
│   ├── launch/     # Recording UI (HUD, source selector)
│   ├── video-editor/
│   │   ├── timeline/       # dnd-timeline based editor
│   │   ├── videoPlayback/  # PixiJS rendering utilities
│   │   └── *.tsx           # Editor components
│   └── ui/         # Radix UI + Tailwind components
├── hooks/          # useScreenRecorder.ts
├── lib/exporter/   # MP4 & GIF export pipelines
└── utils/          # Aspect ratio, platform helpers
```

### Key Data Flow

**Recording**: Screen source → `useScreenRecorder` hook → WebM file → `currentVideoPath`

**Editing**: VideoEditor component holds all state (regions, effects) and passes props down. No Redux/Zustand - uses React useState with callback drilling.

**Export**: VideoEditor state → `VideoExporter`/`GifExporter` → VideoDecoder → FrameRenderer (PixiJS) → VideoEncoder → mp4box muxer → File

### Core Abstractions

**Timeline Regions** - All effects use time-based regions (`src/components/video-editor/types.ts`):
- `ZoomRegion`: Start/end time, depth (1.25×-5×), focus point
- `TrimRegion`: Start/end time for sections to remove
- `AnnotationRegion`: Text/image/figure overlays with position, style, duration

**PixiJS Rendering** - GPU-accelerated preview in `VideoPlayback.tsx`:
- Renders video sprite with transforms (zoom, blur, shadow)
- Composites wallpaper background layer
- Overlays annotations in real-time

**Web Codecs API** - Modern browser codec access for export:
- `VideoDecoder` for input decoding
- `VideoEncoder` for H.264 output
- Frame-by-frame processing with effects applied via PixiJS

### IPC Communication

Renderer calls `window.electronAPI.*` methods defined in `electron/preload.ts`. Main process handlers in `electron/ipc/handlers.ts` handle file I/O, source capture, and window management.

## Tech Stack

- **Electron 39** - Desktop framework
- **React 18** - UI
- **PixiJS 8** - GPU-accelerated 2D rendering
- **dnd-timeline** - Drag-drop timeline
- **Radix UI + Tailwind** - Component system
- **Vite 5** - Build tool
- **Vitest** - Testing (with fast-check for property-based tests)
- **mp4box** - MP4 muxing
- **gif.js** - GIF encoding

## Path Alias

`@/*` maps to `src/*` (configured in tsconfig.json)
