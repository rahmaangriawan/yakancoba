# IMPLEMENTASI

This plan addresses the performance bottleneck in the export/download functionality. Currently, exporting takes a long time because the backend downloads the entire source video file using yt-dlp before slicing. We will optimize this by streaming and slicing the video segments directly from YouTube's direct stream URL using FFmpeg's fast network seeking and stream copying.

---

# User Review Required

## IMPORTANT

### Super-Fast Slicing via Direct Streaming

Instead of downloading the full multi-gigabyte video file to the server's disk, we will pass the direct YouTube stream URL directly into FFmpeg. FFmpeg will seek directly to the clip's timestamp and extract it on the fly.

### Zero Re-encoding Codec Copying (`-c copy`)

We will use FFmpeg's stream copy mode to slice video segments in milliseconds without re-encoding, preserving full video quality and saving massive CPU cycles.

### Concurrently Process Segments

If multiple clips are exported at once, the backend will process them in parallel using `Promise.all` instead of one-by-one sequentially.

### Frontend-Backend Integration

The frontend will send the active `previewUrl` (direct streaming URL) for each clip in the export payload. If the URL is missing or has expired, the backend will fetch a fresh stream URL using a quick, cached `yt-dlp` query.

---

# Proposed Changes

---

## 1. Frontend Updates

### `[MODIFY] page.tsx`

Modify the `handleExport` and `handleExportSingleClip` functions to:

- Extract the `previewUrl` for each clip from `mediaBin`
- Include it in the `/api/export` request payload

---

## 2. Backend Export API Updates

### `[MODIFY] route.ts`

---

### Eliminate Full Downloads

Remove the step that downloads the entire `.mp4` file using `yt-dlp`.

---

### Implement Direct Streaming and Cache

Implement a helper to:

- Validate and resolve the direct streaming URL
- Reuse the `previewUrl` passed from the frontend
- Verify the `expire` query parameter
- Query a new one from `yt-dlp -g` if expired
- Cache the result so we only call it once per unique video

---

### Concurrent Slicing

Use:

```ts
Promise.all
```

to process segment slices concurrently.

---

### Robust Slicing with Fast Stream Copy & Re-encode Fallback

Implement `cutSegment` with:

```bash
-c copy
```

(stream copy) first.

Fall back to standard fast re-encoding:

```bash
-c:v libx264 -preset ultrafast -c:a aac
```

if stream copy fails for any clip segment.

---

# Verification Plan

## Automated & Manual Verification

---

### Speed Benchmarking

Export a timeline of 7 clips from a long YouTube video (e.g. 1–2 hours) and verify it takes under 10 seconds (vs. minutes).

---

### Validation

Ensure the final merged MP4 video:

- Downloads correctly
- Plays perfectly
- Maintains audio-video sync

---

### Single Clip Export

Right-click on a clip, select export, and verify it downloads a single optimized video segment instantly.

---

### URL Expiration Resiliency

Test export with an expired or missing `previewUrl` and verify the backend fetches a fresh URL using `yt-dlp` automatically.

---

# Clipbro - Advanced YouTube Comment Scraper & Smart Editor Walkthrough

Clipbro is a complete, full-stack video editor web application built using:

- Next.js 16
- Tailwind CSS v4
- Flowbite-React

Utilizing a server-side FFmpeg engine for precise video clip cutting and merging.

---

# 🌟 Premium Features Implemented

---

# 1. AI-Driven "Viral Comments" Scraper & Clustering

---

## Top Comments Scraper

The backend:

- Accepts a YouTube link
- Scrapes the top 150 comments using direct `yt-dlp` execution
- Inspects viewer comments for specific timestamp reactions

---

## Regex Temporal Extractor

Scans comment texts for timestamps of formats:

- `MM:SS`
- `H:MM:SS`

---

## Temporal Hotspot Clustering

Groups nearby timestamps (within a 25-second window) into unified:

```txt
Viral Hotspots
```

It scores each hotspot based on:

- Mentions
- Total like engagement

---

## Context-Aware Clip Labels

Automatically cleans and formats the text of the top comment inside that cluster to serve as the literal name of the clip.

Example:

```txt
[Viral #1] windah kaget seru...
```

---

# 2. Smart 7-Clip Timeline Generation

---

## Targeted 7 Clips

The "Auto Clip" engine generates exactly 7 high-potential segments on the timeline tracks:

- Video V1
- Audio A1

---

## Intelligent Padding

Prioritizes viral comment hotspots first.

If fewer than 7 hotspots are parsed, the system automatically supplements the remaining tracks with evenly-spaced mathematical segments across the video's duration, ensuring exactly 7 clips are populated instantly.

---

# 3. Highly Interactive Timeline Editing

---

## Scroll Wheel Timeline Zooming

Hovering over the timeline container and holding:

```txt
Ctrl + Scroll Wheel
```

zooms the timeline scale smoothly.

---

## Draggable Playhead Handle

We designed a rounded white and red playhead indicator handle at the top of the scrubbing line.

Clicking and dragging this handle:

- Scrubs playback position in real-time
- Instantly updates the synchronized preview video canvas

---

## Precise Clip Timestamps

Every timeline block displays highly visible formatted start and end times in an elegant translucent black badge.

Example:

```txt
Range: [12:30 - 12:50]
```

giving the user precise timing cues.

---

# 4. Dynamic Path-Safe Video Rendering & Cascading Deletion (Phase 6 Update)

---

## Runtime-Based Binary Resolution

Moved FFmpeg and FFprobe binary path resolution inside the request handler (`POST` endpoint) instead of evaluation at module-load time.

This completely bypasses Turbopack/Next.js bundle path replacement bugs where:

```txt
process.cwd()
```

gets statically replaced by a compile-time placeholder:

```txt
\ROOT\node_modules\ffmpeg-static\ffmpeg.exe
```

restoring robust spawning of binaries at request runtime.

---

## Cascading Media Bin Deletion

Added a sleek, red trash icon button to every Media Bin card.

Clicking this:

- Deletes the video from the Media Bin
- Automatically cascades to scrub and delete all corresponding timeline clips on the tracks

This prevents downstream rendering errors when exporting.

---

# 5. Context Menu & Single Clip Export & Premium Zoom Slider Ticks (Phase 7 Update)

---

## Interactive Clip Context Menu

Right-clicking any timeline clip on either the Video V1 or Audio A1 tracks opens a customized desktop-grade context menu next to the mouse coordinates containing:

### Export Single Clip

Packages only that specific clip:

- Sends it to `/api/export`
- Uses a single-item array
- Timeline offset reset to `0`

to instantly render and download only that single clip.

---

### Delete Clip

Easily removes the segment from the timeline tracks.

---

## Synchronized Premium Zoom Slider with Bullet-Ticks

The Zoom range input is now fully synchronized with the scroll wheel scaling bounds, seamlessly scaling between:

- `1` (minimum zoom)
- `25` (maximum zoom)

---

## Visual Bullet Ticks

Renders 6 elegant visual bullet-ticks underneath the slider:

- 1x
- 5x
- 10x
- 15x
- 20x
- 25x

Clicking any bullet instantly snaps the timeline zoom factor to that value.

The bullets:

- Light up blue dynamically
- Scale visually when matched

---

## Zoom Factor Display

Displays the current zoom factor text.

Example:

```txt
12x
```

for a professional UI experience.

---

# 6. Premium Scrollbars, Sticky Headers & Perfect Playhead Visibility (Phase 8 Update)

---

## Custom-Styled Webkit Scrollbar

Developed a sleek, dark glowing blue scrollbar:

```txt
.custom-scrollbar
```

inside `globals.css`.

Features:

- Translucent blue handles
- Vibrant glow on hover
- Modern editing-software appearance

---

## Dynamic Horizontal Scroll

Resolved the issue where absolutely positioned clips were hidden and didn't expand the timeline.

The timeline now dynamically calculates the maximum clip width:

```txt
totalDuration * zoom + 150px
```

and enables a premium styled horizontal scrollbar when clips overflow the screen.

---

## Sticky Track Headers

Applied:

```txt
sticky left-0 z-20
```

on the Video V1 and Audio A1 track title columns.

As you scroll horizontally:

- Labels remain pinned cleanly to the left
- Easy track identification remains intact

---

## Unclipped Draggable Playhead

Added larger breathing space:

```txt
pt-8 pb-4
```

to the tracks section and positioned the playhead at:

```txt
top-8
```

The draggable red circle playhead handle now:

- Hovers beautifully above the V1 track
- Remains fully visible
- Feels tactile
- Is never clipped by toolbar boundaries

---

## Scroll-Aware Timeline Click-Scrubbing

Enhanced click-to-scrub functionality inside the timeline tracks container by factoring in dynamic:

```txt
.scrollLeft
```

offset.

Clicking anywhere on a horizontally scrolled track now moves the playhead precisely to the correct timestamp.

---

# 🛠️ File Changes Trace

---

## route.ts

- Direct yt-dlp crawler fetching and parsing YouTube metadata
- Scraping comments
- Clustering viral comment hotspots

---

## route.ts

- Dynamic request-time FFmpeg & FFprobe setup
- Precise clipping execution
- Fast multi-segment concatenations

---

## page.tsx

Implements:

- Interactive dashboard state
- Timeline wheel zoom
- Playhead dragging
- Exact timing ranges
- Cascading media deletion
- Right-click custom context menu
- Synchronized range slider bullets

---

# 🚀 How to Test the Completed Product

---

## 1. Start the Next.js Dev Server

Execute:

```bash
npm run dev
```

in the root folder.

---

## 2. Open Browser

Visit:

```txt
http://localhost:3000
```

---

## 3. Import a Video

Paste a YouTube URL:

```txt
https://www.youtube.com/live/ctIAw1mW7nU?si=9OGDVJ5LfmwBaiId
```

Click:

```txt
Import Video
```

Expected notification:

```txt
Successfully imported! (Scraped X viral highlights from comments!)
```

---

## 4. Click "Auto Clip"

Exactly 7 distinct clips will instantly populate the timeline across:

- Video V1
- Audio A1

---

## 5. Verify Single Clip Right-Click Menu

Right-click any colorful clip block.

Expected menu:

- Export Single Clip
- Delete Clip

---

## 6. Test Single Clip Export

Click:

```txt
Export Single Clip
```

Expected behavior:

- Success toast appears
- FFmpeg slices only that segment
- Automatic download triggers

Example output:

```txt
clipbro_title.mp4
```

---

## 7. Test Delete Clip

Click:

```txt
Delete Clip
```

Expected behavior:

- Clip removed instantly from timeline

---

## 8. Verify Synchronized Zoom Slider

Inside timeline toolbar:

- Hold Ctrl + Scroll Wheel
- Observe smooth zoom updates
- Slider and zoom factor stay synchronized

---

## 9. Test Bullet Tick Zoom

Click any bullet tick:

- Timeline instantly snaps to target zoom
- Active bullet glows blue

---

## 10. Export Full Video

Click:

```txt
Export Video
```

Expected behavior:

- Timeline clips merged
- Server-side concatenation executes
- Final MP4 downloads automatically
