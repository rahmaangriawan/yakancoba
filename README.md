# 🎬 Clipbro — Advanced YouTube Comment Scraper & Smart Editor

> AI-powered YouTube clipping & smart timeline editor built with Next.js + FFmpeg.

---

# ✨ Overview

Clipbro adalah aplikasi video editor berbasis web yang memungkinkan pengguna melakukan:

- Import video YouTube
- Auto generate viral clips dari komentar
- Smart timeline editing
- Export single clip maupun full timeline
- Super-fast rendering menggunakan FFmpeg direct streaming

Dibangun menggunakan:

- Next.js 16
- Tailwind CSS v4
- Flowbite React
- FFmpeg
- yt-dlp
- TypeScript

---

# 🚀 Premium Features

---

# 🧠 AI Viral Comment Scraper

## 🔥 Top Comments Extraction

Backend melakukan scraping hingga 150 komentar teratas YouTube menggunakan:

```bash
yt-dlp
```

Lalu sistem akan mendeteksi timestamp viral yang disebutkan pengguna.

---

## ⏱️ Smart Timestamp Detection

Support format:

- `MM:SS`
- `H:MM:SS`

Contoh:

```txt
12:35
1:22:10
```

---

## 📈 Viral Hotspot Clustering

Timestamp yang berdekatan akan digabung menjadi:

```txt
Viral Hotspot
```

Menggunakan:

- Mention count
- Like engagement
- Temporal clustering

---

## 🏷️ Auto Clip Naming

Komentar terbaik otomatis dijadikan nama clip:

```txt
[Viral #1] windah kaget seru...
```

---

# 🎞️ Smart 7-Clip Timeline Generator

## ✨ Auto Clip Engine

Fitur Auto Clip akan:

- Membuat tepat 7 clip
- Memprioritaskan hotspot viral
- Menggunakan fallback mathematical segmentation jika hotspot kurang

Timeline otomatis terisi pada:

- Video V1
- Audio A1

---

# 🎛️ Interactive Timeline Editor

---

## 🔍 Smooth Timeline Zoom

Support:

- Ctrl + Mouse Wheel
- Slider zoom 1x–25x

---

## 🎯 Draggable Playhead

Playhead dapat:

- Di-drag realtime
- Sinkron dengan preview video
- Scrubbing akurat

---

## 🕒 Precise Time Labels

Setiap clip menampilkan range waktu:

```txt
[12:30 - 12:50]
```

---

# 🖱️ Context Menu System

Klik kanan clip untuk membuka menu:

- Export Single Clip
- Delete Clip

---

# ⚡ Super Fast Export Engine

---

# 🚀 Direct Stream Export Optimization

## ❌ Old Method

Sebelumnya backend melakukan:

```txt
Download full video → Slice → Merge
```

Masalah:

- Lambat
- Boros storage
- Boros CPU

---

## ✅ New Optimized Method

Sekarang backend menggunakan:

```txt
YouTube Direct Stream URL → FFmpeg Slice
```

Tanpa download full video.

---

# ⚡ Zero Re-Encoding

Menggunakan:

```bash
-c copy
```

Benefit:

- Export hitungan detik
- Tidak encode ulang
- Kualitas tetap original

---

# ⚡ Concurrent Clip Processing

Menggunakan:

```ts
Promise.all()
```

Semua clip diproses paralel.

---

# ⚡ Smart URL Validation

Frontend mengirim:

```ts
previewUrl
```

Backend akan:

- Validasi expiry URL
- Refresh otomatis jika expired
- Cache stream URL

---

# ⚡ Fallback Re-Encode

Jika stream copy gagal:

```bash
-c:v libx264 -preset ultrafast -c:a aac
```

---

# 🧹 Cascading Media Deletion

Menghapus media dari Media Bin akan otomatis:

- Menghapus clip terkait di timeline
- Mencegah export error

---

# 🎚️ Premium Zoom Slider

Features:

- 1x → 25x zoom
- Bullet ticks
- Dynamic active state
- Smooth synchronization

---

# 🎨 Premium Timeline UI

## Features

- Custom glowing scrollbar
- Sticky track headers
- Horizontal scrolling
- Scroll-aware scrubbing
- Floating playhead visibility

---

# 📂 File Structure

---

## `page.tsx`

Frontend timeline editor:

- Timeline state
- Zoom handling
- Playhead drag
- Context menu
- Media management
- Export handling

---

## `route.ts`

Backend processing:

- yt-dlp metadata crawler
- Viral comment parser
- FFmpeg export engine
- Direct stream slicing
- Segment concatenation

---

# 🧪 Verification Checklist

---

## ✅ Speed Benchmark

Expected:

- 7 clips export
- Video 1–2 jam
- Render < 10 detik

---

## ✅ Audio Video Sync

Verify:

- Video playable
- Audio normal
- No desync

---

## ✅ Single Clip Export

Test:

- Right click clip
- Export single clip
- Auto download works

---

## ✅ Expired URL Recovery

Backend otomatis:

- Detect expired stream URL
- Refresh menggunakan yt-dlp

---

# 🚀 Installation

## Install dependencies

```bash
npm install
```

---

# ▶️ Running Development Server

```bash
npm run dev
```

---

# 🌐 Open Browser

```txt
http://localhost:3000
```

---

# 🎬 How To Use

---

## 1. Import Video

Paste URL YouTube:

```txt
https://www.youtube.com/live/ctIAw1mW7nU
```

Klik:

```txt
Import Video
```

---

## 2. Auto Generate Clips

Klik:

```txt
Auto Clip
```

Timeline otomatis akan berisi 7 clip.

---

## 3. Edit Timeline

Features:

- Drag playhead
- Zoom timeline
- Delete clips
- Rearrange segments

---

## 4. Right Click Clip

Actions:

- Export Single Clip
- Delete Clip

---

## 5. Export Video

Klik:

```txt
Export Video
```

Server akan:

- Slice semua clip
- Merge otomatis
- Download final MP4

---

# 🧱 Tech Stack

| Technology | Usage |
|---|---|
| Next.js 16 | Frontend Framework |
| TypeScript | Type Safety |
| Tailwind CSS v4 | Styling |
| Flowbite React | UI Components |
| FFmpeg | Video Processing |
| yt-dlp | YouTube Metadata & Streams |

---

# ⚡ Performance Improvements

| Before | After |
|---|---|
| Full video download | Direct stream slicing |
| Slow rendering | Instant slicing |
| High CPU usage | Minimal CPU load |
| Sequential processing | Parallel processing |
| Storage heavy | Stream-based |

---

# 📌 Status

```txt
Production Ready ✅
```

---

# 📄 License

MIT License

---

# 👨‍💻 Author

Built with ❤️ using Next.js + FFmpeg
