# Clipbro - Project Master Copy-Paste Prompt & Rebuilder Guide

Dokumen `.md` ini dirancang khusus agar Anda dapat menyalin dan menduplikasi seluruh proyek **Clipbro** ke asisten AI baru (seperti Claude, ChatGPT, Gemini, dll.) hanya dengan sekali salin-tempel (copy-paste).

Prompt di bawah ini berisi instruksi lengkap beserta **seluruh kode sumber fungsional dari setiap file** dalam proyek ini. Dengan ini, asisten AI baru dapat membangun kembali seluruh aplikasi web video editor "Clipbro" tanpa ada komponen yang terlewat atau rusak.

---

# 🚀 MASTER PROMPT (SALIN DAN TEMPEL SEMUA BAGIAN DI BAWAH INI)

```markdown
Anda adalah seorang AI Software Architect & Coding Specialist kelas dunia. Saya ingin Anda membantu saya membuat sebuah aplikasi Web Video Editor canggih bernama "Clipbro" dari nol menggunakan Next.js 16 (App Router), Tailwind CSS v4, Flowbite-React, dan FFmpeg untuk pemrosesan video server-side.

Aplikasi ini ditujukan untuk mengimpor video YouTube, mendeteksi klip viral secara otomatis berdasarkan komentar penonton, mengedit klip pada multi-track timeline secara interaktif, menyusun subtitle otomatis CapCut-style berkecepatan tinggi, dan melakukan export video super cepat tanpa perlu mendownload seluruh file sumber YouTube ke server.

Tolong bangun kembali aplikasi ini secara utuh. Di bawah ini adalah struktur folder proyek, detail fungsionalitas, serta seluruh isi kode dari file-file utama proyek. Tuliskan setiap file ini secara lengkap, fungsional, dan bebas dari error kompilasi!

---

## 📂 STRUKTUR DIREKTORI PROYEK

Pastikan semua file diletakkan sesuai struktur folder Next.js berikut:
1. `package.json` — Dependensi proyek dan script build.
2. `next.config.ts` — Konfigurasi Next.js dasar.
3. `src/app/layout.tsx` — Layout dasar Next.js dengan ThemeModeScript untuk Flowbite.
4. `src/app/globals.css` — Konfigurasi Tailwind CSS v4, custom premium scrollbars, dan Google Fonts.
5. `src/app/api/video-info/route.ts` — API crawler metadata YouTube, scraping komentar viral (comment hotspot clustering), & parsing + deduplicating subtitles menjadi segment 2-3 kata per cue.
6. `src/app/api/export/route.ts` — API renderer FFmpeg server-side untuk memotong klip secara paralel langsung dari video streaming CDN, penggabungan segmen, kompilasi file subtitle ASS dengan margin kustom (caption width), dan pembakaran subtitle (burning subtitles).
7. `src/app/page.tsx` — Halaman editor utama (Single Page Editor) dengan layout 3 kolom, visualisasi timeline multi-track, playhead draggable, serta caption preview overlay yang bisa di-drag langsung di layar video player.

---

## 🛠️ FILE 1: `package.json`
```json
{
  "name": "clipbro",
  "version": "0.1.0",
  "private": true,
  "scripts": {
    "dev": "next dev",
    "build": "next build",
    "start": "next start",
    "lint": "eslint"
  },
  "dependencies": {
    "@distube/ytdl-core": "^4.16.12",
    "ffmpeg-static": "^5.3.0",
    "ffprobe-static": "^3.1.0",
    "flowbite": "^4.0.2",
    "flowbite-react": "^0.12.17",
    "fluent-ffmpeg": "^2.1.3",
    "next": "16.2.6",
    "react": "19.2.4",
    "react-dom": "19.2.4",
    "react-icons": "^5.6.0",
    "youtube-dl-exec": "^3.1.7"
  },
  "devDependencies": {
    "@tailwindcss/postcss": "^4",
    "@types/fluent-ffmpeg": "^2.1.28",
    "@types/node": "^20",
    "@types/react": "^19",
    "@types/react-dom": "^19",
    "eslint": "^9",
    "eslint-config-next": "16.2.6",
    "tailwindcss": "^4",
    "typescript": "^5"
  }
}
```

---

## 🛠️ FILE 2: `next.config.ts`
```typescript
import type { NextConfig } from "next";

const nextConfig: NextConfig = {
  /* config options here */
};

export default nextConfig;
```

---

## 🛠️ FILE 3: `src/app/layout.tsx`
```typescript
import type { Metadata } from "next";
import { Geist, Geist_Mono } from "next/font/google";
import { ThemeModeScript } from "flowbite-react";
import "./globals.css";

const geistSans = Geist({
  variable: "--font-geist-sans",
  subsets: ["latin"],
});

const geistMono = Geist_Mono({
  variable: "--font-geist-mono",
  subsets: ["latin"],
});

export const metadata: Metadata = {
  title: "Clipbro - Auto YouTube Clipper",
  description: "A premium web-based video editor to automatically clip and edit YouTube videos.",
};

export default function RootLayout({
  children,
}: Readonly<{
  children: React.ReactNode;
}>) {
  return (
    <html
      lang="en"
      className={`${geistSans.variable} ${geistMono.variable} h-full antialiased dark`}
    >
      <head>
        <ThemeModeScript />
      </head>
      <body className="min-h-full flex flex-col bg-gray-50 dark:bg-gray-900 text-gray-900 dark:text-white">
        {children}
      </body>
    </html>
  );
}
```

---

## 🛠️ FILE 4: `src/app/globals.css`
```css
@import url('https://fonts.googleapis.com/css2?family=Montserrat:wght@400;700;900&family=Outfit:wght@400;700;900&family=Inter:wght@400;700;900&display=swap');
@import "tailwindcss";
@plugin "flowbite/plugin";
@source "../../node_modules/flowbite";
@source "../../node_modules/flowbite-react";

:root {
  --background: #ffffff;
  --foreground: #171717;
}

@theme inline {
  --color-background: var(--background);
  --color-foreground: var(--foreground);
  --font-sans: var(--font-geist-sans);
  --font-mono: var(--font-geist-mono);
}

@media (prefers-color-scheme: dark) {
  :root {
    --background: #0a0a0a;
    --foreground: #ededed;
  }
}

body {
  background: var(--background);
  color: var(--foreground);
  font-family: Arial, Helvetica, sans-serif;
}

/* Custom Premium Scrollbar Styling for Video Editor */
.custom-scrollbar::-webkit-scrollbar {
  height: 8px;
  width: 8px;
}

.custom-scrollbar::-webkit-scrollbar-track {
  background: rgba(31, 41, 55, 0.4);
  border-radius: 8px;
}

.custom-scrollbar::-webkit-scrollbar-thumb {
  background: rgba(59, 130, 246, 0.35);
  border-radius: 8px;
  border: 2px solid rgba(17, 24, 39, 0.9);
}

.custom-scrollbar::-webkit-scrollbar-thumb:hover {
  background: rgba(59, 130, 246, 0.75);
}

/* Premium Cross-Browser Flowbite Range Slider Styles */
input[type="range"].premium-range {
  -webkit-appearance: none;
  appearance: none;
  background: transparent;
  cursor: pointer;
}

input[type="range"].premium-range::-webkit-slider-runnable-track {
  background: rgba(55, 65, 81, 0.9); /* Sleek dark-gray track */
  height: 8px;
  border-radius: 9999px;
}

input[type="range"].premium-range::-webkit-slider-thumb {
  -webkit-appearance: none;
  appearance: none;
  margin-top: -4px; /* Center the 16px thumb on the 8px track */
  background-color: #3b82f6; /* Flowbite blue-500 */
  border: 2px solid #ffffff;
  height: 16px;
  width: 16px;
  border-radius: 9999px;
  box-shadow: 0 4px 6px -1px rgba(59, 130, 246, 0.4), 0 2px 4px -2px rgba(59, 130, 246, 0.4);
  transition: all 0.15s ease-in-out;
}

input[type="range"].premium-range::-webkit-slider-thumb:hover {
  background-color: #2563eb; /* blue-600 */
  transform: scale(1.2);
}

input[type="range"].premium-range::-moz-range-track {
  background: rgba(55, 65, 81, 0.9);
  height: 8px;
  border-radius: 9999px;
}

input[type="range"].premium-range::-moz-range-thumb {
  border: 2px solid #ffffff;
  background-color: #3b82f6;
  height: 16px;
  width: 16px;
  border-radius: 9999px;
  box-shadow: 0 4px 6px -1px rgba(59, 130, 246, 0.4), 0 2px 4px -2px rgba(59, 130, 246, 0.4);
  transition: all 0.15s ease-in-out;
}

input[type="range"].premium-range::-moz-range-thumb:hover {
  background-color: #2563eb;
  transform: scale(1.2);
}
```

---

## 🛠️ FILE 5: `src/app/api/video-info/route.ts`
```typescript
import { NextResponse } from "next/server";
import { execFile } from "child_process";
import path from "path";

// Direct cross-platform yt-dlp execution helper to bypass Next.js / Turbopack path resolution bugs
const runYtDlp = (args: string[]): Promise<string> => {
  const isWindows = process.platform === "win32";
  const binaryName = isWindows ? "yt-dlp.exe" : "yt-dlp";
  
  // Statically resolve path relative to project root (cwd)
  const binaryPath = path.join(
    process.cwd(),
    "node_modules",
    "youtube-dl-exec",
    "bin",
    binaryName
  );

  return new Promise((resolve, reject) => {
    execFile(binaryPath, args, { maxBuffer: 20 * 1024 * 1024 }, (error, stdout, stderr) => {
      if (error) {
        reject(new Error(stderr || error.message));
        return;
      }
      resolve(stdout);
    });
  });
};

// WebVTT parsing helper to convert raw VTT cues into light structured objects
function parseVtt(vttText: string): { start: number; end: number; text: string }[] {
  const lines: { start: number; end: number; text: string }[] = [];
  
  // Normalize line endings and split into blocks by blank lines
  const blocks = vttText.replace(/\r\n/g, "\n").split(/\n\n+/);
  
  const timeRegex = /(?:(\d{2}):)?(\d{2}):(\d{2})[.,](\d{3})\s*-->\s*(?:(\d{2}):)?(\d{2}):(\d{2})[.,](\d{3})/;

  for (const block of blocks) {
    const splitBlock = block.split("\n");
    const timeLine = splitBlock.find(l => timeRegex.test(l));
    if (!timeLine) continue;

    const timeIndex = splitBlock.indexOf(timeLine);
    const textLines = splitBlock.slice(timeIndex + 1).filter(l => l.trim() !== "");
    const text = textLines.join(" ");

    const parts = timeLine.split("-->").map(s => s.trim());
    const parseTimePart = (part: string): number => {
      const timeMatch = part.match(/(?:(\d{2}):)?(\d{2}):(\d{2})[.,](\d{3})/);
      if (!timeMatch) return 0;
      const h = timeMatch[1] ? parseInt(timeMatch[1], 10) : 0;
      const m = parseInt(timeMatch[2], 10);
      const s = parseInt(timeMatch[3], 10);
      const ms = parseInt(timeMatch[4], 10);
      return h * 3600 + m * 60 + s + ms / 1000;
    };

    if (parts.length >= 2) {
      const start = parseTimePart(parts[0]);
      const end = parseTimePart(parts[1]);
      
      const cleanText = text
        .replace(/<[^>]*>/g, "") // remove any styling tags like <c> or <b>
        .replace(/&nbsp;/g, " ")
        .replace(/\s+/g, " ")
        .trim();
        
      if (cleanText) {
        lines.push({
          start,
          end,
          text: cleanText
        });
      }
    }
  }
  return lines;
}

// Clean and deduplicate rolling YouTube subtitles
function cleanAndDeduplicateCaptions(captions: { start: number; end: number; text: string }[]): { start: number; end: number; text: string }[] {
  if (captions.length === 0) return [];
  
  const result: { start: number; end: number; text: string }[] = [];
  let current = { ...captions[0] };
  
  for (let i = 1; i < captions.length; i++) {
    const next = captions[i];
    
    // If the next subtitle text is identical to current, merge them by extending the end time
    if (next.text === current.text) {
      current.end = Math.max(current.end, next.end);
    } 
    // Or if the next starts within the current time window and has a rolling progression
    else if (next.start < current.end && next.text.startsWith(current.text)) {
      current.end = Math.max(current.end, next.end);
      current.text = next.text;
    }
    else {
      result.push(current);
      current = { ...next };
    }
  }
  result.push(current);
  
  // Filter out extremely short durations (under 100ms) and clean up
  return result.filter(c => (c.end - c.start) > 0.1);
}

// Split long sentences into 2-3 words chunks for CapCut/TikTok fast-paced subtitle presentation style
function splitCaptionsIntoWords(captions: { start: number; end: number; text: string }[], maxWords = 3): { start: number; end: number; text: string }[] {
  const result: { start: number; end: number; text: string }[] = [];
  
  for (const cap of captions) {
    const words = cap.text.trim().split(/\s+/).filter(w => w.length > 0);
    if (words.length <= maxWords) {
      result.push(cap);
      continue;
    }
    
    const duration = cap.end - cap.start;
    const totalWords = words.length;
    
    let wordIndex = 0;
    while (wordIndex < totalWords) {
      // Pick groups of maxWords (e.g. 3 words)
      const count = Math.min(maxWords, totalWords - wordIndex);
      const groupWords = words.slice(wordIndex, wordIndex + count);
      
      const startSecs = cap.start + (wordIndex / totalWords) * duration;
      const endSecs = cap.start + ((wordIndex + count) / totalWords) * duration;
      
      result.push({
        start: Number(startSecs.toFixed(3)),
        end: Number(endSecs.toFixed(3)),
        text: groupWords.join(" ")
      });
      
      wordIndex += count;
    }
  }
  return result;
}

export async function GET(request: Request) {
  const { searchParams } = new URL(request.url);
  const url = searchParams.get("url");

  if (!url) {
    return NextResponse.json({ error: "URL parameter is required" }, { status: 400 });
  }

  try {
    console.log(`Directly spawning yt-dlp for url: ${url}`);
    
    // Command args to fetch metadata JSON with comment scraping enabled
    const args = [
      url,
      "--dump-single-json",
      "--no-warnings",
      "--no-check-certificates",
      "--youtube-skip-dash-manifest",
      "--write-comments",
      "--extractor-args", "youtube:max-comments=150"
    ];

    const stdout = await runYtDlp(args);
    const info = JSON.parse(stdout);

    if (!info) {
      throw new Error("Failed to parse metadata from yt-dlp output");
    }

    // Extract metadata
    const title = info.title || "Unknown Title";
    const duration = parseInt(info.duration, 10) || 0;
    const author = info.uploader || info.channel || "Unknown Creator";
    const videoId = info.id;
    
    // Get thumbnail
    const thumbnail = info.thumbnail || 
      (info.thumbnails && info.thumbnails.length > 0 
         ? info.thumbnails[info.thumbnails.length - 1].url 
         : null);

    // Find the best combined format (video + audio) for browser playback
    const formats = info.formats || [];
    const combinedFormats = formats.filter((f: any) => {
      return f.vcodec && f.vcodec !== "none" && f.acodec && f.acodec !== "none";
    });

    // Sort to get the highest resolution combined format
    combinedFormats.sort((a: any, b: any) => {
      const hA = a.height || 0;
      const hB = b.height || 0;
      return hB - hA;
    });

    const previewUrl = combinedFormats.length > 0 ? combinedFormats[0].url : null;

    if (!previewUrl) {
      throw new Error("Failed to find a suitable combined video/audio stream format for preview.");
    }

    // --- Subtitles/Auto Captions Extraction ---
    let captions: { start: number; end: number; text: string }[] = [];
    try {
      const autoCaps = info.automatic_captions || {};
      const subs = info.subtitles || {};
      const availableLangs = [...Object.keys(autoCaps), ...Object.keys(subs)];
      
      let targetLang = "";
      // Prioritize Indonesian, then English, then others
      if (availableLangs.includes("id")) {
        targetLang = "id";
      } else if (availableLangs.includes("id-orig")) {
        targetLang = "id-orig";
      } else if (availableLangs.includes("id-Latn")) {
        targetLang = "id-Latn";
      } else if (availableLangs.includes("en")) {
        targetLang = "en";
      } else if (availableLangs.includes("en-orig")) {
        targetLang = "en-orig";
      } else {
        const nonChatLangs = availableLangs.filter(l => l !== "live_chat");
        if (nonChatLangs.length > 0) {
          targetLang = nonChatLangs[0];
        }
      }

      if (targetLang) {
        console.log(`Found YouTube subtitle track for language: ${targetLang}`);
        const tracks = autoCaps[targetLang] || subs[targetLang] || [];
        const vttTrack = tracks.find((t: any) => t.ext === "vtt");
        
        if (vttTrack && vttTrack.url) {
          console.log(`Fetching VTT subtitle data from YouTube timedtext URL...`);
          const vttRes = await fetch(vttTrack.url);
          if (vttRes.ok) {
            const vttText = await vttRes.text();
            const parsedLines = parseVtt(vttText);
            captions = splitCaptionsIntoWords(cleanAndDeduplicateCaptions(parsedLines), 3);
            console.log(`Successfully fetched, deduplicated, and split ${captions.length} captions into short segments.`);
          } else {
            console.warn(`Failed to fetch timedtext: status ${vttRes.status}`);
          }
        }
      }
    } catch (subErr) {
      console.error("Non-fatal error while extracting YouTube subtitles:", subErr);
    }

    // Parse comments and extract timestamps for potential viral highlights
    const viralHighlights: any[] = [];
    if (info.comments && Array.isArray(info.comments) && duration > 0) {
      const parsedHighlights: any[] = [];
      const timestampRegex = /\b((?:[0-9]{1,2}:)?[0-9]{1,2}:[0-9]{2})\b/g;

      const parseTimestampToSeconds = (ts: string): number => {
        const parts = ts.split(':').map(Number);
        if (parts.length === 3) {
          return parts[0] * 3600 + parts[1] * 60 + parts[2];
        } else if (parts.length === 2) {
          return parts[0] * 60 + parts[1];
        }
        return 0;
      };

      info.comments.forEach((c: any) => {
        const text = c.text || "";
        const matches = text.match(timestampRegex);
        if (matches) {
          matches.forEach((ts: string) => {
            const secs = parseTimestampToSeconds(ts);
            if (secs > 0 && secs < duration) {
              parsedHighlights.push({
                timestamp: ts,
                seconds: secs,
                author: c.author || "Viewer",
                likes: c.like_count || 0,
                text: text.trim()
              });
            }
          });
        }
      });

      // Group nearby timestamps (within 25 seconds of each other)
      const grouped: any[] = [];
      parsedHighlights.sort((a, b) => a.seconds - b.seconds);

      parsedHighlights.forEach(h => {
        let foundGroup = false;
        for (const g of grouped) {
          if (Math.abs(g.center - h.seconds) <= 25) {
            g.items.push(h);
            g.totalLikes += h.likes;
            g.count += 1;
            g.center = Math.round(g.items.reduce((sum: number, item: any) => sum + item.seconds, 0) / g.items.length);
            foundGroup = true;
            break;
          }
        }
        if (!foundGroup) {
          grouped.push({
            center: h.seconds,
            items: [h],
            totalLikes: h.likes,
            count: 1
          });
        }
      });

      // Sort groups by engagement: count descending, then likes descending
      grouped.sort((a, b) => {
        if (b.count !== a.count) return b.count - a.count;
        return b.totalLikes - a.totalLikes;
      });

      // Take top 7 hotspots
      grouped.slice(0, 7).forEach((g, idx) => {
        g.items.sort((a: any, b: any) => b.likes - a.likes);
        const topComment = g.items[0];
        
        let label = `Highlight #${idx + 1}`;
        if (topComment && topComment.text) {
          let cleanText = topComment.text
            .replace(/\b((?:[0-9]{1,2}:)?[0-9]{1,2}:[0-9]{2})\b/g, "") // Remove timestamps
            .replace(/[^\w\s\u00C0-\u017F\u0400-\u04FF\u0600-\u06FF\u0750-\u077F\u0900-\u097F\u3040-\u30FF\u31F0-\u31FF\u4E00-\u9FFF]/gu, "") // Keep alphanumeric, whitespace, and multi-language characters
            .trim();
          if (cleanText.length > 25) {
            cleanText = cleanText.substring(0, 22) + "...";
          }
          if (cleanText) {
            label = cleanText;
          }
        }

        viralHighlights.push({
          centerTime: g.center,
          label: label,
          mentions: g.count,
          totalLikes: g.totalLikes
        });
      });
    }

    console.log(`Successfully fetched video: "${title}". Found ${viralHighlights.length} hotspots, ${captions.length} captions.`);

    return NextResponse.json({
      title,
      duration,
      author,
      thumbnail,
      previewUrl,
      videoId,
      originalUrl: url,
      viralHighlights,
      captions
    });
  } catch (error: any) {
    console.error("Error fetching video info with direct yt-dlp execution:", error);
    return NextResponse.json({ 
      error: error.message || "Failed to fetch video metadata from YouTube." 
    }, { status: 500 });
  }
}
```

---

## 🛠️ FILE 6: `src/app/api/export/route.ts`
```typescript
import { NextResponse } from "next/server";
import { execFile } from "child_process";
import ffmpeg from "fluent-ffmpeg";
import ffmpegPath from "ffmpeg-static";
// @ts-ignore
import ffprobePath from "ffprobe-static";
import fs from "fs";
import path from "path";
import os from "os";

// Dynamically resolve ffmpeg and ffprobe paths relative to process.cwd() at request-time to bypass Next.js / Turbopack path resolution bugs
let pathsInitialized = false;

function initFfmpegPaths() {
  if (pathsInitialized) return;

  const isWin = process.platform === "win32";
  const arch = process.arch;

  const resolvedFfmpegPath = path.join(
    process.cwd(),
    "node_modules",
    "ffmpeg-static",
    isWin ? "ffmpeg.exe" : "ffmpeg"
  );
  console.log("Dynamically resolved ffmpeg path:", resolvedFfmpegPath);
  ffmpeg.setFfmpegPath(resolvedFfmpegPath);

  let resolvedFfprobePath = "";
  if (process.platform === "win32") {
    resolvedFfprobePath = path.join(
      process.cwd(),
      "node_modules",
      "ffprobe-static",
      "bin",
      "win32",
      arch === "ia32" ? "ia32" : "x64",
      "ffprobe.exe"
    );
  } else if (process.platform === "darwin") {
    resolvedFfprobePath = path.join(
      process.cwd(),
      "node_modules",
      "ffprobe-static",
      "bin",
      "darwin",
      "x64",
      "ffprobe"
    );
  } else if (process.platform === "linux") {
    resolvedFfprobePath = path.join(
      process.cwd(),
      "node_modules",
      "ffprobe-static",
      "bin",
      "linux",
      arch === "ia32" ? "ia32" : arch === "arm" ? "arm" : "x64",
      "ffprobe"
    );
  }

  if (resolvedFfprobePath && fs.existsSync(resolvedFfprobePath)) {
    console.log("Dynamically resolved ffprobe path:", resolvedFfprobePath);
    ffmpeg.setFfprobePath(resolvedFfprobePath);
  } else {
    console.warn("ffprobe path not found or not supported for platform:", process.platform);
  }

  pathsInitialized = true;
}

// Direct cross-platform yt-dlp execution helper to bypass Next.js / Turbopack path resolution bugs
const runYtDlp = (args: string[]): Promise<string> => {
  const isWindows = process.platform === "win32";
  const binaryName = isWindows ? "yt-dlp.exe" : "yt-dlp";
  
  // Statically resolve path relative to project root (cwd)
  const binaryPath = path.join(
    process.cwd(),
    "node_modules",
    "youtube-dl-exec",
    "bin",
    binaryName
  );

  return new Promise((resolve, reject) => {
    execFile(binaryPath, args, { maxBuffer: 20 * 1024 * 1024 }, (error, stdout, stderr) => {
      if (error) {
        reject(new Error(stderr || error.message));
        return;
      }
      resolve(stdout);
    });
  });
};

interface TimelineClip {
  id: string;
  videoId: string;
  startInSource: number;
  duration: number;
  trackId: string;
  startInTimeline: number;
  previewUrl?: string;
}

// Memory cache to avoid querying the same stream URL multiple times during the same request session
const streamUrlCache: Record<string, string> = {};

async function getStreamUrl(videoId: string, previewUrl?: string): Promise<string> {
  if (streamUrlCache[videoId]) {
    return streamUrlCache[videoId];
  }

  // 1. Try to validate and use the previewUrl passed from the frontend
  if (previewUrl) {
    try {
      const urlObj = new URL(previewUrl);
      const expire = parseInt(urlObj.searchParams.get("expire") || "0", 10);
      const nowInSecs = Math.floor(Date.now() / 1000);
      
      // If it has at least 60 seconds before expiration, reuse it!
      if (expire > nowInSecs + 60) {
        console.log(`[Cache Hit] Using valid direct preview URL from frontend for video ID: ${videoId}`);
        streamUrlCache[videoId] = previewUrl;
        return previewUrl;
      } else {
        console.log(`[Cache Expired] Preview URL from frontend is expired or expiring soon for video ID: ${videoId}`);
      }
    } catch (e) {
      console.warn("Failed to parse/validate frontend previewUrl:", e);
    }
  }

  // 2. Fetch direct stream URL via yt-dlp
  console.log(`[Fetch] Fetching fresh stream URL for ${videoId} using yt-dlp...`);
  const youtubeUrl = `https://www.youtube.com/watch?v=${videoId}`;
  const freshUrl = await runYtDlp([
    youtubeUrl,
    "-f", "best",
    "-g",
    "--no-warnings",
    "--no-check-certificates"
  ]);

  const cleanedUrl = freshUrl.trim();
  if (!cleanedUrl) {
    throw new Error(`Failed to resolve stream URL for video ${videoId}`);
  }

  console.log(`[Fetch Success] Successfully fetched fresh stream URL for video ID: ${videoId}`);
  streamUrlCache[videoId] = cleanedUrl;
  return cleanedUrl;
}

// Optimized segment cutting with fast stream copy and re-encoding fallback
async function cutSegment(streamUrl: string, start: number, duration: number, outputFile: string): Promise<void> {
  console.log(`Attempting fast stream-copy cut to: ${outputFile} starting at ${start}s for ${duration}s`);
  
  try {
    await new Promise<void>((resolve, reject) => {
      ffmpeg(streamUrl)
        .inputOptions([
          "-ss", start.toString(),
          "-reconnect", "1",
          "-reconnect_streamed", "1",
          "-reconnect_delay_max", "5"
        ])
        .duration(duration)
        .output(outputFile)
        .outputOptions("-c copy")
        .on("end", () => {
          console.log(`Fast stream-copy successful for: ${outputFile}`);
          resolve();
        })
        .on("error", (err) => {
          reject(err);
        })
        .run();
    });
    return;
  } catch (err: any) {
    console.warn(`Fast stream-copy failed for ${outputFile}, falling back to full re-encoding. Error:`, err.message || err);
    
    // Fallback: full re-encode (preset ultrafast to be as speedy as possible)
    await new Promise<void>((resolve, reject) => {
      ffmpeg(streamUrl)
        .inputOptions([
          "-ss", start.toString(),
          "-reconnect", "1",
          "-reconnect_streamed", "1",
          "-reconnect_delay_max", "5"
        ])
        .duration(duration)
        .output(outputFile)
        .outputOptions([
          "-c:v libx264",
          "-preset ultrafast",
          "-c:a aac"
        ])
        .on("end", () => {
          console.log(`Re-encode cut successful for: ${outputFile}`);
          resolve();
        })
        .on("error", (err) => {
          console.error(`Re-encode cut failed for: ${outputFile}`, err);
          reject(err);
        })
        .run();
    });
  }
}

interface CaptionLine {
  start: number;
  end: number;
  text: string;
}

interface CaptionStyles {
  fontFamily: string;
  fontSize: number;
  textColor: string;
  outlineColor: string;
  outlineWidth: number;
  backgroundColor: string;
  bold: boolean;
  italic: boolean;
  alignment?: "top" | "center" | "bottom";
  positionX?: number;
  positionY?: number;
  captionWidth?: number;
}

function hexToAssColor(hex: string, defaultAlpha = "00"): string {
  let clean = hex.replace("#", "").trim();
  if (clean.length === 3) {
    clean = clean[0] + clean[0] + clean[1] + clean[1] + clean[2] + clean[2];
  }
  
  let r = "FF";
  let g = "FF";
  let b = "FF";
  let a = defaultAlpha; // "00" is opaque

  if (clean.length === 6) {
    r = clean.substring(0, 2);
    g = clean.substring(2, 4);
    b = clean.substring(4, 6);
  } else if (clean.length === 8) {
    r = clean.substring(0, 2);
    g = clean.substring(2, 4);
    b = clean.substring(4, 6);
    const alphaHex = clean.substring(6, 8);
    const opacity = parseInt(alphaHex, 16);
    const transparency = 255 - opacity;
    a = transparency.toString(16).padStart(2, "0").toUpperCase();
  }
  
  return `&H${a}${b}${g}${r}`;
}

function formatSecondsToAss(seconds: number): string {
  const h = Math.floor(seconds / 3600);
  const m = Math.floor((seconds % 3600) / 60);
  const s = Math.floor(seconds % 60);
  const cs = Math.floor((seconds % 1) * 100);

  const hh = h.toString();
  const mm = m.toString().padStart(2, "0");
  const ss = s.toString().padStart(2, "0");
  const ccs = cs.toString().padStart(2, "0");

  return `${hh}:${mm}:${ss}.${ccs}`;
}

function generateAssContent(captions: CaptionLine[], styles: CaptionStyles, canvasRatio: string = "16:9"): string {
  const fontName = styles.fontFamily || "Arial";
  const fontSizeScaled = Math.round((styles.fontSize || 24) * 2.2);
  const primaryCol = hexToAssColor(styles.textColor || "#FFFFFF");
  const outlineCol = hexToAssColor(styles.outlineColor || "#000000");
  const backCol = styles.backgroundColor && styles.backgroundColor !== "transparent"
    ? hexToAssColor(styles.backgroundColor)
    : "&HFF000000";
  const borderStyle = styles.backgroundColor && styles.backgroundColor !== "transparent" ? 3 : 1;
  const outlineWidthScaled = Math.round((styles.outlineWidth || 2) * 2);
  const boldVal = styles.bold ? -1 : 0;
  const italicVal = styles.italic ? -1 : 0;
  
  // Set script resolution based on canvas ratio (default is 16:9 widescreen)
  const playResX = canvasRatio === "9:16" ? 1080 : 1920;
  const playResY = canvasRatio === "9:16" ? 1920 : 1080;

  let alignVal = 2; // Bottom Center
  if (styles.alignment === "center") alignVal = 5;
  else if (styles.alignment === "top") alignVal = 8;

  // Calculate margins based on captionWidth (default 90%)
  const widthPct = styles.captionWidth !== undefined ? styles.captionWidth : 90;
  const marginPct = (100 - widthPct) / 2;
  const marginPx = Math.max(10, Math.round((marginPct / 100) * playResX));

  const header = `[Script Info]
Title: Clipbro Styled Auto Captions
ScriptType: v4.00+
Collisions: Normal
PlayResX: ${playResX}
PlayResY: ${playResY}
Timer: 100.0000

[V4+ Styles]
Format: Name, Fontname, Fontsize, PrimaryColour, SecondaryColour, OutlineColour, BackColour, Bold, Italic, Underline, StrikeOut, ScaleX, ScaleY, Spacing, Angle, BorderStyle, Outline, Shadow, Alignment, MarginL, MarginR, MarginV, Encoding
Style: Default,${fontName},${fontSizeScaled},${primaryCol},&H000000FF,${outlineCol},${backCol},${boldVal},${italicVal},0,0,100,100,0,0,${borderStyle},${outlineWidthScaled},0,${alignVal},${marginPx},${marginPx},80,1

[Events]
Format: Layer, Start, End, Style, Name, MarginL, MarginR, MarginV, Effect, Text
`;

  let events = "";
  for (const cap of captions) {
    const startStr = formatSecondsToAss(cap.start);
    const endStr = formatSecondsToAss(cap.end);
    const cleanText = cap.text.replace(/\r?\n/g, "\\N");
    
    // Check if custom percentage coordinates are provided
    if (styles.positionX !== undefined && styles.positionY !== undefined) {
      // Calculate pixel coordinates relative to PlayResX and PlayResY
      const posX = Math.round((styles.positionX / 100) * playResX);
      const posY = Math.round((styles.positionY / 100) * playResY);
      events += `Dialogue: 0,startStr,endStr,Default,,0,0,0,,{\\pos(${posX},${posY})}${cleanText}\n`;
    } else {
      events += `Dialogue: 0,startStr,endStr,Default,,0,0,0,,${cleanText}\n`;
    }
  }

  // Replace startStr and endStr placeholders with ASS formatted variables
  let processedEvents = events;
  for (const cap of captions) {
    const startStr = formatSecondsToAss(cap.start);
    const endStr = formatSecondsToAss(cap.end);
    processedEvents = processedEvents
      .replace("startStr", startStr)
      .replace("endStr", endStr);
  }

  return header + processedEvents;
}

function escapeFfmpegSubtitlesPath(fullPath: string): string {
  let p = fullPath.replace(/\\/g, "/");
  if (process.platform === "win32") {
    p = p.replace(/^([a-zA-Z]):\//, "$1\\:/");
    p = p.replace(/'/g, "'\\\\''");
  }
  return p;
}

export async function POST(request: Request) {
  try {
    initFfmpegPaths();
    const body = await request.json();
    const clips: TimelineClip[] = body.clips;
    const captions: CaptionLine[] = body.captions || [];
    const captionStyles: CaptionStyles = body.captionStyles;
    const canvasRatio = body.canvasRatio || "16:9";

    if (!clips || clips.length === 0) {
      return NextResponse.json({ error: "No clips to export" }, { status: 400 });
    }

    // Filter and sort clips on the video track V1 by timeline starting position
    const videoClips = clips
      .filter((c) => c.trackId === "V1")
      .sort((a, b) => a.startInTimeline - b.startInTimeline);

    if (videoClips.length === 0) {
      return NextResponse.json({ error: "No video track (V1) clips found" }, { status: 400 });
    }

    const tmpDir = os.tmpdir();
    console.log(`Starting optimized concurrent export of ${videoClips.length} clips...`);

    // Slices all segments concurrently
    const segmentPromises = videoClips.map(async (clip, idx) => {
      const streamUrl = await getStreamUrl(clip.videoId, clip.previewUrl);
      const segmentFile = path.join(tmpDir, `clipbro_segment_${clip.id}_${idx}.mp4`);
      
      console.log(`Concurrent Slice Triggered: clip ID ${clip.id} [${clip.startInSource}s -> ${clip.startInSource + clip.duration}s]`);
      await cutSegment(streamUrl, clip.startInSource, clip.duration, segmentFile);
      
      return segmentFile;
    });

    const segmentPaths = await Promise.all(segmentPromises);
    console.log(`All ${segmentPaths.length} segment cuts finished successfully.`);

    // Step 3: Concatenate all cut segments
    let mergedPath = "";
    
    if (segmentPaths.length === 1) {
      mergedPath = segmentPaths[0];
    } else {
      const concatListPath = path.join(tmpDir, `clipbro_concat_${Date.now()}.txt`);
      const fileContent = segmentPaths.map((p) => `file '${p.replace(/\\/g, "/")}'`).join("\n");
      
      fs.writeFileSync(concatListPath, fileContent);
      
      mergedPath = path.join(tmpDir, `clipbro_merged_${Date.now()}.mp4`);
      console.log("Merging segments via ffmpeg concat...");

      await new Promise<void>((resolve, reject) => {
        ffmpeg()
          .input(concatListPath)
          .inputOptions(["-f", "concat", "-safe", "0"])
          .outputOptions("-c copy")
          .save(mergedPath)
          .on("end", () => {
            try {
              fs.unlinkSync(concatListPath);
            } catch (e) {}
            resolve();
          })
          .on("error", (err) => {
            console.error("Error merging segments:", err);
            reject(err);
          })
          .run();
      });
    }

    // Step 4: Process video canvas sizing and burn subtitles if required
    let finalOutputPath = mergedPath;
    let assFilePath = "";
    let processedOutputPath = "";
    const filters: string[] = [];

    // Setup subtitles filter if captions are enabled
    if (captions && captions.length > 0 && captionStyles) {
      assFilePath = path.join(tmpDir, `clipbro_subs_${Date.now()}.ass`);
      fs.writeFileSync(assFilePath, generateAssContent(captions, captionStyles, canvasRatio));
      
      const escapedSubPath = escapeFfmpegSubtitlesPath(assFilePath);
      filters.push(`subtitles=${escapedSubPath}`);
    }

    if (filters.length > 0) {
      processedOutputPath = path.join(tmpDir, `clipbro_processed_${Date.now()}.mp4`);
      const filterString = filters.join(",");
      console.log(`Applying FFmpeg filters: ${filterString} onto ${finalOutputPath}`);

      await new Promise<void>((resolve, reject) => {
        ffmpeg(finalOutputPath)
          .outputOptions([
            "-vf", filterString,
            "-c:v libx264",
            "-preset ultrafast",
            "-c:a copy"
          ])
          .save(processedOutputPath)
          .on("end", () => {
            console.log("FFmpeg filters applied successfully!");
            resolve();
          })
          .on("error", (err) => {
            console.error("Error applying FFmpeg filters:", err);
            reject(err);
          })
          .run();
      });

      finalOutputPath = processedOutputPath;
    }

    // Read the rendered final output file
    const fileBuffer = fs.readFileSync(finalOutputPath);

    // Clean up temporary files
    for (const segmentPath of segmentPaths) {
      try {
        fs.unlinkSync(segmentPath);
      } catch (e) {}
    }
    
    if (mergedPath && mergedPath !== segmentPaths[0] && mergedPath !== finalOutputPath) {
      try {
        fs.unlinkSync(mergedPath);
      } catch (e) {}
    }

    if (assFilePath) {
      try {
        fs.unlinkSync(assFilePath);
      } catch (e) {}
    }

    if (processedOutputPath) {
      try {
        fs.unlinkSync(processedOutputPath);
      } catch (e) {}
    }

    // Return the video as downloadable binary stream
    return new Response(fileBuffer, {
      headers: {
        "Content-Type": "video/mp4",
        "Content-Disposition": "attachment; filename=clipbro_export.mp4",
      },
    });
  } catch (error: any) {
    console.error("Export error:", error);
    return NextResponse.json({ error: error.message || "Failed to render video" }, { status: 500 });
  }
}
```

---

## 🛠️ FILE 7: `src/app/page.tsx`
```typescript
"use client";

import { useState, useRef, useEffect, useCallback } from "react";
import { Navbar, NavbarBrand, Button, TextInput, Spinner, Toast } from "flowbite-react";
import { 
  HiOutlineDownload, 
  HiOutlineVideoCamera, 
  HiOutlinePlay, 
  HiOutlinePause, 
  HiOutlineTrash, 
  HiOutlineSparkles,
  HiOutlinePlus,
  HiCheckCircle,
  HiOutlineAdjustments,
  HiOutlineRefresh
} from "react-icons/hi";

interface CaptionLine {
  start: number;
  end: number;
  text: string;
}

interface CaptionStyles {
  fontFamily: string;
  fontSize: number;
  textColor: string;     // e.g. "#FFFFFF"
  outlineColor: string;  // e.g. "#000000"
  outlineWidth: number;  // e.g. 2
  backgroundColor: string; // e.g. "transparent" or "#000000"
  bold: boolean;
  italic: boolean;
  alignment: "top" | "center" | "bottom";
  positionX?: number;    // e.g. 50
  positionY?: number;    // e.g. 85
  captionWidth?: number;  // percentage (20 to 100)
}

interface ViralHighlight {
  centerTime: number;
  label: string;
  mentions: number;
  totalLikes: number;
}

interface VideoMetadata {
  videoId: string;
  title: string;
  duration: number;
  author: string;
  thumbnail: string;
  previewUrl: string;
  originalUrl: string;
  viralHighlights?: ViralHighlight[];
  captions?: CaptionLine[];
}

interface TimelineClip {
  id: string;
  videoId: string;
  title: string;
  trackId: "V1" | "A1";
  startInTimeline: number; // in seconds
  duration: number; // in seconds
  startInSource: number; // in seconds
  color: string;
}

// Format seconds into highly readable HH:MM:SS or MM:SS format
const formatTime = (totalSeconds: number): string => {
  const h = Math.floor(totalSeconds / 3600);
  const m = Math.floor((totalSeconds % 3600) / 60);
  const s = Math.floor(totalSeconds % 60);
  if (h > 0) {
    return `${h.toString().padStart(2, "0")}:${m.toString().padStart(2, "0")}:${s.toString().padStart(2, "0")}`;
  }
  return `${m.toString().padStart(2, "0")}:${s.toString().padStart(2, "0")}`;
};

const STYLE_PRESETS = [
  {
    name: "Classic Meme",
    fontFamily: "Impact",
    fontSize: 28,
    textColor: "#FFFFFF",
    outlineColor: "#000000",
    outlineWidth: 4,
    backgroundColor: "transparent",
    bold: true,
    italic: false,
    alignment: "bottom" as const,
  },
  {
    name: "Karaoke Yellow",
    fontFamily: "Montserrat",
    fontSize: 24,
    textColor: "#FFCC00",
    outlineColor: "#000000",
    outlineWidth: 3,
    backgroundColor: "transparent",
    bold: true,
    italic: false,
    alignment: "bottom" as const,
  },
  {
    name: "Soft Sub",
    fontFamily: "Inter",
    fontSize: 20,
    textColor: "#FFFFFF",
    outlineColor: "#000000",
    outlineWidth: 1,
    backgroundColor: "#00000099",
    bold: false,
    italic: false,
    alignment: "bottom" as const,
  },
  {
    name: "Red Alert",
    fontFamily: "Outfit",
    fontSize: 26,
    textColor: "#FF3333",
    outlineColor: "#FFFFFF",
    outlineWidth: 3,
    backgroundColor: "transparent",
    bold: true,
    italic: false,
    alignment: "bottom" as const,
  },
  {
    name: "Neon Cyber",
    fontFamily: "Courier New",
    fontSize: 22,
    textColor: "#00FFFF",
    outlineColor: "#FF00FF",
    outlineWidth: 2,
    backgroundColor: "transparent",
    bold: true,
    italic: true,
    alignment: "bottom" as const,
  }
];

export default function Home() {

  const [captionStyles, setCaptionStyles] = useState<CaptionStyles>({
    fontFamily: "Impact",
    fontSize: 24,
    textColor: "#FFFFFF",
    outlineColor: "#000000",
    outlineWidth: 3,
    backgroundColor: "transparent",
    bold: true,
    italic: false,
    alignment: "bottom",
    positionX: 50,
    positionY: 85,
    captionWidth: 90,
  });

  // Toggle captions on/off
  const [enableCaptions, setEnableCaptions] = useState(true);

  // YouTube Fetch States
  const [youtubeUrl, setYoutubeUrl] = useState("");
  const [isFetching, setIsFetching] = useState(false);
  const [errorMessage, setErrorMessage] = useState("");
  const [successToast, setSuccessToast] = useState("");

  // Media Bin State
  const [mediaBin, setMediaBin] = useState<VideoMetadata[]>([]);
  const [selectedVideo, setSelectedVideo] = useState<VideoMetadata | null>(null);

  // Timeline & Playback States
  const [timelineClips, setTimelineClips] = useState<TimelineClip[]>([]);
  const [playhead, setPlayhead] = useState(0); // Current timeline time in seconds
  const [isPlaying, setIsPlaying] = useState(false);
  const [zoom, setZoom] = useState(5); // pixels per second factor
  const [isDragging, setIsDragging] = useState(false); // dragging the playhead marker

  // Context Menu State for Timeline Clips
  const [contextMenu, setContextMenu] = useState<{
    visible: boolean;
    x: number;
    y: number;
    clip: TimelineClip | null;
  }>({
    visible: false,
    x: 0,
    y: 0,
    clip: null
  });

  // Refs for Video Player
  const videoRef = useRef<HTMLVideoElement | null>(null);
  const timelineRef = useRef<HTMLDivElement | null>(null);

  // Determine total timeline duration based on last clip
  const getTimelineDuration = useCallback(() => {
    if (timelineClips.length === 0) return 0;
    return Math.max(...timelineClips.map(c => c.startInTimeline + c.duration));
  }, [timelineClips]);

  // Sync Video preview player with timeline playhead position
  const syncVideoWithPlayhead = useCallback((time: number) => {
    if (!videoRef.current) return;
    
    // Find if there is a clip on V1 track containing this timeline time
    const activeVideoClip = timelineClips.find(
      c => c.trackId === "V1" && time >= c.startInTimeline && time <= c.startInTimeline + c.duration
    );

    if (activeVideoClip) {
      const videoSource = mediaBin.find(v => v.videoId === activeVideoClip.videoId);
      if (videoSource) {
        // If player has different source or is paused, adjust it
        const currentSrc = videoRef.current.src;
        if (!currentSrc.includes(videoSource.previewUrl)) {
          videoRef.current.src = videoSource.previewUrl;
        }

        // Calculate time within source video
        const elapsedInClip = time - activeVideoClip.startInTimeline;
        const sourceTime = activeVideoClip.startInSource + elapsedInClip;

        // Prevent infinite loops by checking threshold
        if (Math.abs(videoRef.current.currentTime - sourceTime) > 0.5) {
          videoRef.current.currentTime = sourceTime;
        }
        
        if (isPlaying && videoRef.current.paused) {
          videoRef.current.play().catch(() => {});
        }
      }
    } else {
      // No active clip, pause video
      if (!videoRef.current.paused) {
        videoRef.current.pause();
      }
    }
  }, [timelineClips, mediaBin, isPlaying]);

  // Scroll wheel zooming hook for premium timeline scaling
  useEffect(() => {
    const timelineEl = timelineRef.current;
    if (!timelineEl) return;

    const handleWheel = (e: WheelEvent) => {
      if (e.ctrlKey) {
        e.preventDefault();
        const zoomDelta = e.deltaY < 0 ? 1 : -1;
        setZoom((prev) => Math.max(1, Math.min(25, prev + zoomDelta)));
      }
    };

    timelineEl.addEventListener("wheel", handleWheel, { passive: false });
    return () => {
      timelineEl.removeEventListener("wheel", handleWheel);
    };
  }, [timelineRef]);

  // Close context menu on global click
  useEffect(() => {
    const handleGlobalClick = () => {
      setContextMenu(prev => prev.visible ? { ...prev, visible: false } : prev);
    };
    window.addEventListener("click", handleGlobalClick);
    return () => {
      window.removeEventListener("click", handleGlobalClick);
    };
  }, []);

  // Global mousemove/mouseup listener for perfect window-wide dragging performance
  useEffect(() => {
    if (!isDragging) return;

    const handleGlobalMouseMove = (e: MouseEvent) => {
      if (!timelineRef.current) return;
      const rect = timelineRef.current.getBoundingClientRect();
      const scrollLeft = timelineRef.current.scrollLeft;
      // Offset by 112px (96px label + 16px container padding) for perfect clip alignment
      const clientX = e.clientX - rect.left - 112 + scrollLeft;
      
      const maxDuration = getTimelineDuration();
      const calculatedSeconds = clientX / zoom;
      const clampedTime = Math.max(0, Math.min(calculatedSeconds, maxDuration + 10));
      
      setPlayhead(clampedTime);
      syncVideoWithPlayhead(clampedTime);
    };

    const handleGlobalMouseUp = () => {
      setIsDragging(false);
    };

    window.addEventListener("mousemove", handleGlobalMouseMove);
    window.addEventListener("mouseup", handleGlobalMouseUp);

    return () => {
      window.removeEventListener("mousemove", handleGlobalMouseMove);
      window.removeEventListener("mouseup", handleGlobalMouseUp);
    };
  }, [isDragging, zoom, timelineClips, getTimelineDuration, syncVideoWithPlayhead]);

  // Auto-hide Toast
  useEffect(() => {
    if (successToast || errorMessage) {
      const timer = setTimeout(() => {
        setSuccessToast("");
        setErrorMessage("");
      }, 4000);
      return () => clearTimeout(timer);
    }
  }, [successToast, errorMessage]);

  // Handle Playhead progression during playback
  useEffect(() => {
    let intervalId: ReturnType<typeof setInterval> | undefined;
    if (isPlaying) {
      intervalId = setInterval(() => {
        setPlayhead((prev) => {
          const maxTimelineDuration = getTimelineDuration();
          if (prev >= maxTimelineDuration && maxTimelineDuration > 0) {
            setIsPlaying(false);
            if (videoRef.current) videoRef.current.pause();
            return 0;
          }
          const nextPlayhead = prev + 0.1;
          
          // Sync HTML5 video player with active clip under playhead
          syncVideoWithPlayhead(nextPlayhead);
          
          return nextPlayhead;
        });
      }, 100);
    } else {
      clearInterval(intervalId);
    }
    return () => clearInterval(intervalId);
  }, [isPlaying, timelineClips, getTimelineDuration, syncVideoWithPlayhead]);

  // Retrieve the active caption text under the playhead mapped to the current active timeline clip
  const getActiveCaption = (): string | null => {
    const activeClip = timelineClips.find(
      c => c.trackId === "V1" && playhead >= c.startInTimeline && playhead <= c.startInTimeline + c.duration
    );
    if (!activeClip) return null;

    const source = mediaBin.find(v => v.videoId === activeClip.videoId);
    if (!source || !source.captions) return null;

    const elapsedInClip = playhead - activeClip.startInTimeline;
    const sourceTime = activeClip.startInSource + elapsedInClip;

    const caption = source.captions.find(
      cap => sourceTime >= cap.start && sourceTime <= cap.end
    );
    return caption ? caption.text : null;
  };

  // Get all captions from active clips mapped to timeline coordinate space
  const getTimelineCaptions = (): CaptionLine[] => {
    const compiled: CaptionLine[] = [];
    const videoClips = timelineClips
      .filter(c => c.trackId === "V1")
      .sort((a, b) => a.startInTimeline - b.startInTimeline);

    for (const clip of videoClips) {
      const source = mediaBin.find(v => v.videoId === clip.videoId);
      if (!source || !source.captions) continue;

      const clipStart = clip.startInSource;
      const clipEnd = clip.startInSource + clip.duration;

      for (const cap of source.captions) {
        const activeStartInSource = Math.max(cap.start, clipStart);
        const activeEndInSource = Math.min(cap.end, clipEnd);

        if (activeStartInSource < activeEndInSource) {
          const timelineStart = clip.startInTimeline + (activeStartInSource - clipStart);
          const timelineEnd = clip.startInTimeline + (activeEndInSource - clipStart);
          
          compiled.push({
            start: timelineStart,
            end: timelineEnd,
            text: cap.text
          });
        }
      }
    }
    return compiled.sort((a, b) => a.start - b.start);
  };

  // Fetch Video Info from YouTube Link
  const handleImportVideo = async () => {
    if (!youtubeUrl) return;
    setIsFetching(true);
    setErrorMessage("");
    try {
      const res = await fetch(`/api/video-info?url=${encodeURIComponent(youtubeUrl)}`);
      const data = await res.json();

      if (!res.ok) {
        throw new Error(data.error || "Failed to fetch video");
      }

      // Add to media bin if not already present
      if (!mediaBin.some(v => v.videoId === data.videoId)) {
        setMediaBin(prev => [...prev, data]);
        setSelectedVideo(data);
        const hsCount = data.viralHighlights ? data.viralHighlights.length : 0;
        const hsMsg = hsCount > 0 ? ` (Scraped ${hsCount} viral highlights from comments!)` : "";
        setSuccessToast(`Successfully imported: "${data.title.substring(0, 22)}..."${hsMsg}`);
      } else {
        setSuccessToast("Video is already in the Media Bin!");
      }
      setYoutubeUrl("");
    } catch (err) {
      const msg = err instanceof Error ? err.message : "An error occurred";
      setErrorMessage(msg);
    } finally {
      setIsFetching(false);
    }
  };

  // Add selected video to timeline manually
  const addToTimeline = (video: VideoMetadata, track: "V1" | "A1") => {
    const timelineDuration = getTimelineDuration();
    const duration = Math.min(30, video.duration); // Default to 30s or full duration if shorter
    const newClip: TimelineClip = {
      id: `manual-${video.videoId}-${timelineClips.length}-${Math.round(timelineDuration)}`,
      videoId: video.videoId,
      title: video.title,
      trackId: track,
      startInTimeline: timelineDuration,
      duration: duration,
      startInSource: 0,
      color: track === "V1" ? "bg-blue-600/35 border-blue-500 hover:bg-blue-600/45" : "bg-green-600/35 border-green-500 hover:bg-green-600/45"
    };

    setTimelineClips(prev => [...prev, newClip]);
    setSuccessToast(`Added 30s clip to timeline (${track})`);
  };

  // Auto Clip Feature: Intelligently generates exactly 7 high-potential segments on the timeline
  const handleAutoClip = (video: VideoMetadata) => {
    if (!video) return;

    const targetClipsCount = 7;
    const clips: TimelineClip[] = [];
    const highlights = video.viralHighlights || [];
    
    let timelinePointer = 0;
    
    for (let i = 0; i < targetClipsCount; i++) {
      let startInSource = 0;
      let duration = 20; // default duration is 20s
      let title = "";

      if (i < highlights.length) {
        const hl = highlights[i];
        // Center the 20-second clip on the hotspot center, keeping it inside bounds
        startInSource = Math.max(0, Math.round(hl.centerTime - 10));
        
        // Clean title
        const cleanLabel = hl.label ? hl.label.trim() : `Highlight #${i + 1}`;
        title = `${cleanLabel}`;
      } else {
        // Fallback mathematical distribution: spacing across the video duration
        // Avoid starting too close to the end, distribute between 5% and 85% of duration
        const indexOffset = i - highlights.length;
        const totalOffsets = targetClipsCount - highlights.length;
        const pct = 0.05 + (indexOffset / Math.max(1, totalOffsets)) * 0.8;
        
        startInSource = Math.floor(video.duration * pct);
        title = `Segment ${indexOffset + 1}`;
      }

      // Ensure segment doesn't exceed video duration
      if (startInSource + duration > video.duration) {
        duration = Math.max(5, video.duration - startInSource);
      }

      // Choose a beautiful glassmorphic gradient color
      const colors = [
        "bg-purple-600/40 border-purple-500 text-purple-200 hover:bg-purple-600/50",
        "bg-indigo-600/40 border-indigo-500 text-indigo-200 hover:bg-indigo-600/50",
        "bg-pink-600/40 border-pink-500 text-pink-200 hover:bg-pink-600/50",
        "bg-blue-600/40 border-blue-500 text-blue-200 hover:bg-blue-600/50",
        "bg-cyan-600/40 border-cyan-500 text-cyan-200 hover:bg-cyan-600/50",
        "bg-fuchsia-600/40 border-fuchsia-500 text-fuchsia-200 hover:bg-fuchsia-600/50",
        "bg-violet-600/40 border-violet-500 text-violet-200 hover:bg-violet-600/50"
      ];
      const color = colors[i % colors.length];

      clips.push({
        id: `auto-${i}`,
        videoId: video.videoId,
        title: title.substring(0, 45),
        trackId: "V1",
        startInTimeline: timelinePointer,
        duration: duration,
        startInSource: startInSource,
        color: color
      });

      // Advance timeline pointer, adding a small 2-second transition gap between clips
      timelinePointer += duration + 2;
    }

    // Sync Audio track for the auto clips
    const audioClips: TimelineClip[] = clips.map(c => ({
      ...c,
      id: c.id + "-audio",
      trackId: "A1",
      color: "bg-emerald-600/40 border-emerald-500 text-emerald-200 hover:bg-emerald-600/50"
    }));

    setTimelineClips([...clips, ...audioClips]);
    setPlayhead(0);
    
    const hsCount = highlights.length;
    const msg = hsCount > 0 
      ? `Generated exactly 7 high-potential clips (prioritized ${hsCount} viral moments from comments!)`
      : "Generated exactly 7 mathematically distributed clip segments!";
    setSuccessToast(msg);
  };

  // Remove individual clip
  const deleteClip = (clipId: string) => {
    setTimelineClips(prev => prev.filter(c => c.id !== clipId));
  };

  // Remove video from Media Bin
  const handleDeleteFromMediaBin = (videoId: string) => {
    setMediaBin(prev => prev.filter(v => v.videoId !== videoId));
    if (selectedVideo?.videoId === videoId) {
      setSelectedVideo(null);
    }
    // Proactively clear timeline clips belonging to this video as well
    setTimelineClips(prev => prev.filter(c => c.videoId !== videoId));
    setSuccessToast("Removed video from Media Bin and timeline.");
  };

  // Clear all timeline
  const clearTimeline = () => {
    setTimelineClips([]);
    setPlayhead(0);
    setIsPlaying(false);
    if (videoRef.current) videoRef.current.pause();
  };

  // Handle clicking on Timeline to scrub playhead
  const handleTimelineScrub = (e: React.MouseEvent<HTMLDivElement>) => {
    if (!timelineRef.current) return;
    const rect = timelineRef.current.getBoundingClientRect();
    const scrollLeft = timelineRef.current.scrollLeft;
    const clickX = e.clientX - rect.left - 112 + scrollLeft; // Adjust for track label width (96px) + padding (16px) and scroll
    if (clickX < 0) return;

    const clickedSeconds = clickX / zoom;
    const maxDuration = getTimelineDuration();
    
    // Clamp within 0 and max duration (plus a buffer)
    const clampedTime = Math.max(0, Math.min(clickedSeconds, maxDuration + 10));
    setPlayhead(clampedTime);
    syncVideoWithPlayhead(clampedTime);
  };

  // Handle caption dragging with mouse
  const handleMouseDown = (e: React.MouseEvent) => {
    e.preventDefault();
    const container = videoRef.current?.parentElement;
    if (!container) return;

    const handleMouseMove = (moveEvent: MouseEvent) => {
      const rect = container.getBoundingClientRect();
      let xPercent = ((moveEvent.clientX - rect.left) / rect.width) * 100;
      let yPercent = ((moveEvent.clientY - rect.top) / rect.height) * 100;

      // Constrain within player viewport bounds
      xPercent = Math.max(5, Math.min(95, xPercent));
      yPercent = Math.max(5, Math.min(95, yPercent));

      setCaptionStyles((prev) => ({
        ...prev,
        positionX: Math.round(xPercent),
        positionY: Math.round(yPercent),
      }));
    };

    const handleMouseUp = () => {
      window.removeEventListener("mousemove", handleMouseMove);
      window.removeEventListener("mouseup", handleMouseUp);
    };

    window.addEventListener("mousemove", handleMouseMove);
    window.addEventListener("mouseup", handleMouseUp);
  };

  // Handle caption dragging with touch (Mobile devices)
  const handleTouchStart = (e: React.TouchEvent) => {
    const container = videoRef.current?.parentElement;
    if (!container) return;

    const handleTouchMove = (moveEvent: TouchEvent) => {
      const rect = container.getBoundingClientRect();
      const touch = moveEvent.touches[0];
      if (!touch) return;

      let xPercent = ((touch.clientX - rect.left) / rect.width) * 100;
      let yPercent = ((touch.clientY - rect.top) / rect.height) * 100;

      xPercent = Math.max(5, Math.min(95, xPercent));
      yPercent = Math.max(5, Math.min(95, yPercent));

      setCaptionStyles((prev) => ({
        ...prev,
        positionX: Math.round(xPercent),
        positionY: Math.round(yPercent),
      }));
    };

    const handleTouchEnd = () => {
      window.removeEventListener("touchmove", handleTouchMove);
      window.removeEventListener("touchend", handleTouchEnd);
    };

    window.addEventListener("touchmove", handleTouchMove);
    window.addEventListener("touchend", handleTouchEnd);
  };

  // Select manual alignment alignment and snap vertical percentage coordinates
  const handleAlignmentChange = (align: "top" | "center" | "bottom") => {
    let yPercent = 85;
    if (align === "top") yPercent = 15;
    else if (align === "center") yPercent = 50;

    setCaptionStyles((prev) => ({
      ...prev,
      alignment: align,
      positionX: 50,
      positionY: yPercent,
    }));
  };

  // Toggle playback
  const togglePlay = () => {
    if (timelineClips.length === 0) return;
    
    if (isPlaying) {
      setIsPlaying(false);
      if (videoRef.current) videoRef.current.pause();
    } else {
      setIsPlaying(true);
      // Start or resume
      syncVideoWithPlayhead(playhead);
    }
  };

  // Trigger download / export composition
  const handleExport = async () => {
    if (timelineClips.length === 0) {
      setErrorMessage("Timeline is empty! Add some clips to export.");
      return;
    }
    
    setIsFetching(true);
    setSuccessToast("Exporting... Rendering video via FFmpeg on the server (this may take a minute).");
    
    try {
      const clipsWithUrls = timelineClips.map(clip => {
        const source = mediaBin.find(v => v.videoId === clip.videoId);
        return {
          ...clip,
          previewUrl: source ? source.previewUrl : undefined
        };
      });

      const res = await fetch("/api/export", {
        method: "POST",
        headers: {
          "Content-Type": "application/json",
        },
        body: JSON.stringify({ 
          clips: clipsWithUrls,
          captions: enableCaptions ? getTimelineCaptions() : [],
          captionStyles: captionStyles,
        }),
      });

      if (!res.ok) {
        const errorData = await res.json().catch(() => ({}));
        throw new Error(errorData.error || "Failed to render video");
      }

      const blob = await res.blob();
      const url = window.URL.createObjectURL(blob);
      const a = document.createElement("a");
      a.href = url;
      
      // Use the first clip's title to name the export file
      const videoClips = timelineClips.filter(c => c.trackId === "V1").sort((x, y) => x.startInTimeline - y.startInTimeline);
      const firstClip = videoClips[0] || timelineClips[0];
      const cleanTitle = firstClip
        ? firstClip.title.replace(/[^a-z0-9]/gi, '_').replace(/_+/g, '_').replace(/^_+|_+$/g, '').toLowerCase()
        : "export";
      
      a.download = `clipbro_${cleanTitle}.mp4`;
      document.body.appendChild(a);
      a.click();
      a.remove();
      window.URL.revokeObjectURL(url);
      
      setSuccessToast("Export successful! Video downloaded.");
    } catch (err) {
      console.error(err);
      const msg = err instanceof Error ? err.message : "Failed to export video";
      setErrorMessage(msg);
    } finally {
      setIsFetching(false);
    }
  };

  // Export a single selected clip from context menu
  const handleExportSingleClip = async (clip: TimelineClip) => {
    setIsFetching(true);
    setSuccessToast(`Exporting single clip: "${clip.title}"... (Rendering on server)`);
    
    try {
      const source = mediaBin.find(v => v.videoId === clip.videoId);
      
      // Compute shifted timeline captions for this single clip
      const singleClipCaptions: CaptionLine[] = [];
      if (enableCaptions && source && source.captions) {
        const clipStart = clip.startInSource;
        const clipEnd = clip.startInSource + clip.duration;
        for (const cap of source.captions) {
          const activeStartInSource = Math.max(cap.start, clipStart);
          const activeEndInSource = Math.min(cap.end, clipEnd);
          if (activeStartInSource < activeEndInSource) {
            singleClipCaptions.push({
              start: activeStartInSource - clipStart,
              end: activeEndInSource - clipStart,
              text: cap.text
            });
          }
        }
      }

      // Create a clip structure that maps startInTimeline to 0 for a clean export
      const singleClipForExport = {
        id: clip.id,
        videoId: clip.videoId,
        startInSource: clip.startInSource,
        duration: clip.duration,
        trackId: "V1", // Ensure it's on the V1 track so the backend grabs it properly
        startInTimeline: 0,
        previewUrl: source ? source.previewUrl : undefined
      };

      const res = await fetch("/api/export", {
        method: "POST",
        headers: {
          "Content-Type": "application/json",
        },
        body: JSON.stringify({ 
          clips: [singleClipForExport],
          captions: singleClipCaptions,
          captionStyles: captionStyles,
        }),
      });

      if (!res.ok) {
        const errorData = await res.json().catch(() => ({}));
        throw new Error(errorData.error || "Failed to render video clip");
      }

      const blob = await res.blob();
      const url = window.URL.createObjectURL(blob);
      const a = document.createElement("a");
      a.href = url;
      // Sanitize the filename to be safe
      const cleanTitle = clip.title.replace(/[^a-z0-9]/gi, '_').replace(/_+/g, '_').replace(/^_+|_+$/g, '').toLowerCase();
      a.download = `clipbro_${cleanTitle}.mp4`;
      document.body.appendChild(a);
      a.click();
      a.remove();
      window.URL.revokeObjectURL(url);
      
      setSuccessToast("Single clip export successful!");
    } catch (err) {
      console.error(err);
      const msg = err instanceof Error ? err.message : "Failed to export video clip";
      setErrorMessage(msg);
    } finally {
      setIsFetching(false);
    }
  };

  return (
    <div className="flex flex-col h-screen overflow-hidden bg-gray-900 text-white">
      {/* Toast notifications */}
      {successToast && (
        <div className="absolute top-5 right-5 z-50">
          <Toast>
            <div className="inline-flex h-8 w-8 shrink-0 items-center justify-center rounded-lg bg-green-100 text-green-500 dark:bg-green-800 dark:text-green-200">
              <HiCheckCircle className="h-5 w-5" />
            </div>
            <div className="ml-3 text-sm font-normal text-gray-900 dark:text-white">{successToast}</div>
          </Toast>
        </div>
      )}

      {errorMessage && (
        <div className="absolute top-5 right-5 z-50">
          <Toast>
            <div className="inline-flex h-8 w-8 shrink-0 items-center justify-center rounded-lg bg-red-100 text-red-500 dark:bg-red-800 dark:text-red-200">
              <span className="font-bold">!</span>
            </div>
            <div className="ml-3 text-sm font-normal text-gray-900 dark:text-white">{errorMessage}</div>
          </Toast>
        </div>
      )}

      {/* Navbar */}
      <Navbar fluid className="bg-gray-800 border-b border-gray-700 py-3">
        <NavbarBrand href="#">
          <HiOutlineVideoCamera className="mr-3 h-8 w-8 text-blue-500" />
          <span className="self-center whitespace-nowrap text-2xl font-bold dark:text-white bg-gradient-to-r from-blue-500 to-indigo-400 bg-clip-text text-transparent">
            Clipbro
          </span>
        </NavbarBrand>
        <div className="flex md:order-2 space-x-3">
          <Button color="blue" onClick={handleExport} className="shadow-lg shadow-blue-500/20 hover:scale-105 transition-all cursor-pointer">
            <HiOutlineDownload className="mr-2 h-5 w-5" />
            Export Video
          </Button>
        </div>
      </Navbar>

      {/* Main Content Area */}
      <div className="flex flex-1 overflow-hidden">
        
        {/* Left Sidebar: Media Bin */}
        <aside className="w-80 flex-shrink-0 border-r border-gray-700 bg-gray-800/50 backdrop-blur-md flex flex-col">
          {/* Import YouTube Box */}
          <div className="p-4 border-b border-gray-700 bg-gray-800">
            <h2 className="text-xs font-semibold uppercase tracking-wider text-gray-400 mb-3 flex items-center">
              <HiOutlineVideoCamera className="mr-2 h-4 w-4 text-blue-400" /> Import YouTube Video
            </h2>
            <div className="flex flex-col space-y-2">
              <TextInput 
                id="youtube-url" 
                type="url" 
                placeholder="Paste YouTube link here..." 
                sizing="sm"
                value={youtubeUrl}
                onChange={(e) => setYoutubeUrl(e.target.value)}
                disabled={isFetching}
              />
              <Button 
                color="blue" 
                size="sm" 
                onClick={handleImportVideo}
                disabled={isFetching || !youtubeUrl}
                className="w-full flex items-center justify-center cursor-pointer"
              >
                {isFetching ? (
                  <>
                    <Spinner size="sm" className="mr-2" /> Fetching Details...
                  </>
                ) : (
                  "Import Video"
                )}
              </Button>
            </div>
          </div>

          {/* Media Bin List */}
          <div className="p-4 flex-1 overflow-y-auto flex flex-col custom-scrollbar">
            <h2 className="text-xs font-semibold uppercase tracking-wider text-gray-400 mb-3">Media Bin</h2>
            
            {mediaBin.length === 0 ? (
              <div className="text-center text-sm text-gray-500 mt-10 p-4 border-2 border-dashed border-gray-700 rounded-lg">
                No videos imported yet. Paste a link above to start!
              </div>
            ) : (
              <div className="space-y-3 flex-1">
                {mediaBin.map((video) => (
                  <div 
                    key={video.videoId} 
                    className={`p-3 rounded-lg border transition-all cursor-pointer ${
                      selectedVideo?.videoId === video.videoId 
                        ? "bg-blue-600/10 border-blue-500 shadow-md" 
                        : "bg-gray-800/40 border-gray-700 hover:border-gray-500"
                    }`}
                    onClick={() => setSelectedVideo(video)}
                  >
                    <img 
                      src={video.thumbnail} 
                      alt={video.title} 
                      className="w-full h-24 object-cover rounded-md mb-2 shadow"
                    />
                    <h3 className="text-xs font-semibold line-clamp-2 mb-1">{video.title}</h3>
                    <p className="text-[10px] text-gray-400 mb-1">Duration: {Math.floor(video.duration / 60)}m {video.duration % 60}s</p>
                    {video.viralHighlights && video.viralHighlights.length > 0 && (
                      <span className="inline-flex items-center gap-1 rounded bg-purple-500/20 px-2 py-0.5 text-[9px] font-semibold text-purple-300 border border-purple-500/30 mb-2">
                        🔥 {video.viralHighlights.length} Viral Hotspots Scraped
                      </span>
                    )}
                    
                    {/* Media Actions */}
                    <div className="flex space-x-1">
                      <Button 
                        color="dark" 
                        size="xs" 
                        className="flex-1 cursor-pointer"
                        onClick={(e) => {
                          e.stopPropagation();
                          addToTimeline(video, "V1");
                        }}
                      >
                        <HiOutlinePlus className="mr-1 h-3 w-3" /> Timeline
                      </Button>
                      <Button 
                        color="purple" 
                        size="xs" 
                        className="flex-1 cursor-pointer"
                        onClick={(e) => {
                          e.stopPropagation();
                          handleAutoClip(video);
                        }}
                      >
                        <HiOutlineSparkles className="mr-1 h-3 w-3" /> Auto Clip
                      </Button>
                      <Button 
                        color="red" 
                        size="xs" 
                        className="px-2 cursor-pointer"
                        title="Remove from Media Bin"
                        onClick={(e) => {
                          e.stopPropagation();
                          handleDeleteFromMediaBin(video.videoId);
                        }}
                      >
                        <HiOutlineTrash className="h-3.5 w-3.5" />
                      </Button>
                    </div>
                  </div>
                ))}
              </div>
            )}
          </div>
        </aside>

        {/* Center/Right Area: Player & Timeline */}
        <div className="flex-1 flex flex-col min-w-0">
          
          {/* Player/Preview Area */}
          <main className="flex-1 p-6 flex flex-col items-center justify-center bg-gray-950 relative">
            {selectedVideo ? (
              <div className="flex flex-col items-center justify-center max-h-[500px] w-full">
                <div className="aspect-video w-full max-w-3xl bg-black rounded-xl overflow-hidden border border-gray-800 shadow-2xl relative">
                  <video 
                    ref={videoRef}
                    className="w-full h-full object-contain"
                    controls={false}
                    src={selectedVideo.previewUrl}
                  />

                  {/* Real-time Subtitle Overlay */}
                  {enableCaptions && getActiveCaption() && (
                    <div 
                      className="absolute cursor-grab active:cursor-grabbing select-none pointer-events-auto group p-2 border border-transparent hover:border-dashed hover:border-blue-500 rounded transition-colors"
                      style={{
                        left: `${captionStyles.positionX ?? 50}%`,
                        top: `${captionStyles.positionY ?? 85}%`,
                        transform: "translate(-50%, -50%)",
                        zIndex: 30,
                        width: `${captionStyles.captionWidth ?? 90}%`,
                        maxWidth: `${captionStyles.captionWidth ?? 90}%`,
                        display: "flex",
                        justifyContent: "center",
                      }}
                      onMouseDown={handleMouseDown}
                      onTouchStart={handleTouchStart}
                    >
                      {/* Corner Anchor Dots for CapCut style resize handles */}
                      <div className="absolute -top-1 -left-1 w-2 h-2 bg-blue-500 rounded-full opacity-0 group-hover:opacity-100 transition-opacity shadow shadow-blue-500/50"></div>
                      <div className="absolute -top-1 -right-1 w-2 h-2 bg-blue-500 rounded-full opacity-0 group-hover:opacity-100 transition-opacity shadow shadow-blue-500/50"></div>
                      <div className="absolute -bottom-1 -left-1 w-2 h-2 bg-blue-500 rounded-full opacity-0 group-hover:opacity-100 transition-opacity shadow shadow-blue-500/50"></div>
                      <div className="absolute -bottom-1 -right-1 w-2 h-2 bg-blue-500 rounded-full opacity-0 group-hover:opacity-100 transition-opacity shadow shadow-blue-500/50"></div>

                      <span
                        style={{
                          fontFamily: captionStyles.fontFamily,
                          fontSize: `${captionStyles.fontSize}px`,
                          color: captionStyles.textColor,
                          fontWeight: captionStyles.bold ? "bold" : "normal",
                          fontStyle: captionStyles.italic ? "italic" : "normal",
                          textShadow: captionStyles.outlineWidth > 0 
                            ? `
                              -${captionStyles.outlineWidth}px -${captionStyles.outlineWidth}px 0 ${captionStyles.outlineColor},  
                               ${captionStyles.outlineWidth}px -${captionStyles.outlineWidth}px 0 ${captionStyles.outlineColor},
                              -${captionStyles.outlineWidth}px  ${captionStyles.outlineWidth}px 0 ${captionStyles.outlineColor},
                               ${captionStyles.outlineWidth}px  ${captionStyles.outlineWidth}px 0 ${captionStyles.outlineColor},
                              -${captionStyles.outlineWidth}px 0px 0 ${captionStyles.outlineColor},
                               ${captionStyles.outlineWidth}px 0px 0 ${captionStyles.outlineColor},
                               0px -${captionStyles.outlineWidth}px 0 ${captionStyles.outlineColor},
                               0px  ${captionStyles.outlineWidth}px 0 ${captionStyles.outlineColor}
                            `
                            : "none",
                          backgroundColor: captionStyles.backgroundColor,
                          padding: captionStyles.backgroundColor !== "transparent" ? "4px 12px" : "0",
                          borderRadius: captionStyles.backgroundColor !== "transparent" ? "6px" : "0",
                          whiteSpace: "pre-wrap",
                          display: "inline-block",
                          textAlign: "center",
                        }}
                      >
                        {getActiveCaption()}
                      </span>
                    </div>
                  )}
                </div>
                <div className="mt-3 text-center">
                  <h2 className="text-sm font-semibold truncate max-w-lg">{selectedVideo.title}</h2>
                  <p className="text-xs text-gray-500">Channel: {selectedVideo.author}</p>
                </div>
              </div>
            ) : (
              <div className="aspect-video w-full max-w-3xl bg-gray-900/50 rounded-xl border border-gray-800 shadow-2xl flex items-center justify-center">
                <div className="text-center p-6">
                  <HiOutlineVideoCamera className="mx-auto h-16 w-16 text-gray-700 mb-3 animate-pulse" />
                  <h3 className="text-lg font-medium text-gray-400">No active video</h3>
                  <p className="text-sm text-gray-600 mt-1 max-w-xs px-4">
                    Import a YouTube link or select an imported video from the Media Bin to preview it here.
                  </p>
                </div>
              </div>
            )}
          </main>

          {/* Timeline Area */}
          <section className="h-72 flex-shrink-0 border-t border-gray-700 bg-gray-800/80 backdrop-blur-md flex flex-col">
            
            {/* Timeline Toolbar Controls */}
            <div className="h-12 border-b border-gray-700 bg-gray-900 flex items-center justify-between px-6">
              <div className="flex items-center space-x-3">
                <Button 
                  color={isPlaying ? "red" : "blue"} 
                  size="xs" 
                  onClick={togglePlay}
                  disabled={timelineClips.length === 0}
                  className="w-20 cursor-pointer"
                >
                  {isPlaying ? (
                    <>
                      <HiOutlinePause className="mr-1 h-4 w-4" /> Pause
                    </>
                  ) : (
                    <>
                      <HiOutlinePlay className="mr-1 h-4 w-4" /> Play
                    </>
                  )}
                </Button>
                <Button 
                  color="dark" 
                  size="xs" 
                  onClick={clearTimeline}
                  disabled={timelineClips.length === 0}
                  className="cursor-pointer"
                >
                  <HiOutlineTrash className="mr-1 h-4 w-4" /> Clear All
                </Button>
              </div>

              {/* Time display */}
              <div className="flex items-center space-x-4">
                <div className="flex items-center space-x-2 text-xs text-gray-400">
                  <label htmlFor="zoom-range" className="font-semibold text-gray-400 cursor-pointer">Zoom:</label>
                  <input 
                    id="zoom-range"
                    type="range" 
                    min="1" 
                    max="25" 
                    value={zoom} 
                    onChange={(e) => setZoom(parseInt(e.target.value))} 
                    className="w-32 premium-range transition-all"
                  />
                </div>
                <div className="h-6 w-px bg-gray-700"></div>
                <span className="text-sm font-bold text-blue-400 font-mono">
                  {Math.floor(playhead / 60).toString().padStart(2, "0")}:
                  {(Math.floor(playhead) % 60).toString().padStart(2, "0")}.
                  {Math.floor((playhead % 1) * 10).toString()}
                </span>
                <span className="text-xs text-gray-500">
                  / {Math.floor(getTimelineDuration() / 60).toString().padStart(2, "0")}:
                  {(Math.floor(getTimelineDuration()) % 60).toString().padStart(2, "0")}
                </span>
              </div>
            </div>
            
            {/* Timeline Tracks Section */}
            <div 
              ref={timelineRef}
              className="flex-1 overflow-y-auto overflow-x-auto pt-8 pb-4 px-4 space-y-3 relative bg-gray-950/40 select-none cursor-pointer custom-scrollbar"
              onClick={handleTimelineScrub}
            >
              {/* Playhead marker line */}
              {timelineClips.length > 0 && (
                <div 
                  className="absolute top-8 bottom-4 w-px bg-red-500 z-10 pointer-events-none transition-all duration-75"
                  style={{ left: `${112 + playhead * zoom}px` }} // 96px track label + 16px container padding
                >
                  {/* Draggable Playhead Handle */}
                  <div 
                    className="w-5 h-5 bg-red-500 rounded-full -ml-2.5 -mt-2.5 shadow-lg shadow-red-500/60 cursor-ew-resize pointer-events-auto active:scale-125 hover:scale-110 transition-transform border-2 border-white flex items-center justify-center z-40"
                    onMouseDown={(e) => {
                      e.stopPropagation();
                      e.preventDefault();
                      setIsDragging(true);
                    }}
                  >
                    <div className="w-2 h-2 bg-white rounded-full"></div>
                  </div>
                </div>
              )}

              {timelineClips.length === 0 ? (
                <div className="h-full flex items-center justify-center text-xs text-gray-600 italic">
                  Timeline is empty. Use &quot;Timeline&quot; or &quot;Auto Clip&quot; on a media file to add clips.
                </div>
              ) : (
                <div 
                  style={{ 
                    width: `${96 + getTimelineDuration() * zoom + 150}px`,
                    minWidth: "100%" 
                  }}
                  className="space-y-3 relative"
                >
                  {/* Video Track V1 */}
                  <div className="flex h-16 bg-gray-900/60 rounded-lg overflow-hidden border border-gray-800 relative">
                    <div className="w-24 flex-shrink-0 bg-gray-800 border-r border-gray-700 p-2 flex items-center justify-center z-20 sticky left-0">
                      <span className="text-xs font-bold text-blue-400 uppercase tracking-wider">Video V1</span>
                    </div>
                    <div className="flex-1 relative overflow-visible">
                      {timelineClips
                        .filter(c => c.trackId === "V1")
                        .map(clip => (
                          <div 
                            key={clip.id}
                            className={`absolute h-[56px] border rounded-md m-1 p-2 text-xs flex flex-col justify-between shadow-md transition-all ${clip.color} select-none`}
                            style={{ 
                              left: `${clip.startInTimeline * zoom}px`, 
                              width: `${clip.duration * zoom}px` 
                            }}
                            onClick={(e) => {
                              e.stopPropagation(); // Avoid scrub on clip click
                            }}
                            onContextMenu={(e) => {
                              e.preventDefault();
                              e.stopPropagation();
                              setContextMenu({
                                visible: true,
                                x: e.clientX,
                                y: e.clientY,
                                clip: clip
                              });
                            }}
                          >
                            <div className="flex justify-between items-start">
                              <span className="font-semibold line-clamp-1 pr-4">{clip.title}</span>
                              <button 
                                onClick={(e) => {
                                  e.stopPropagation();
                                  deleteClip(clip.id);
                                }}
                                className="text-gray-400 hover:text-red-500 transition-colors cursor-pointer"
                              >
                                <HiOutlineTrash className="h-4 w-4" />
                              </button>
                            </div>
                            <span className="text-[9px] text-gray-300 font-mono font-semibold flex justify-between bg-black/40 px-1 py-0.5 rounded leading-none">
                              <span>Range: [{formatTime(clip.startInSource)} - {formatTime(clip.startInSource + clip.duration)}]</span>
                              <span>Dur: {Math.floor(clip.duration)}s</span>
                            </span>
                          </div>
                        ))}
                    </div>
                  </div>
 
                  {/* Audio Track A1 */}
                  <div className="flex h-16 bg-gray-900/60 rounded-lg overflow-hidden border border-gray-800 relative">
                    <div className="w-24 flex-shrink-0 bg-gray-800 border-r border-gray-700 p-2 flex items-center justify-center z-20 sticky left-0">
                      <span className="text-xs font-bold text-green-400 uppercase tracking-wider">Audio A1</span>
                    </div>
                    <div className="flex-1 relative overflow-visible">
                      {timelineClips
                        .filter(c => c.trackId === "A1")
                        .map(clip => (
                          <div 
                            key={clip.id}
                            className={`absolute h-[56px] border rounded-md m-1 p-2 text-xs flex flex-col justify-between shadow-md transition-all ${clip.color} select-none`}
                            style={{ 
                              left: `${clip.startInTimeline * zoom}px`, 
                              width: `${clip.duration * zoom}px` 
                            }}
                            onClick={(e) => {
                              e.stopPropagation();
                            }}
                            onContextMenu={(e) => {
                              e.preventDefault();
                              e.stopPropagation();
                              setContextMenu({
                                visible: true,
                                x: e.clientX,
                                y: e.clientY,
                                clip: clip
                              });
                            }}
                          >
                            <div className="flex justify-between items-start">
                              <span className="font-semibold line-clamp-1 pr-4">{clip.title}</span>
                              <button 
                                onClick={(e) => {
                                  e.stopPropagation();
                                  deleteClip(clip.id);
                                }}
                                className="text-gray-400 hover:text-red-500 transition-colors cursor-pointer"
                              >
                                <HiOutlineTrash className="h-4 w-4" />
                              </button>
                            </div>
                            <span className="text-[9px] text-gray-300 font-mono font-semibold flex justify-between bg-black/40 px-1 py-0.5 rounded leading-none">
                              <span>Range: [{formatTime(clip.startInSource)} - {formatTime(clip.startInSource + clip.duration)}]</span>
                              <span>Dur: {Math.floor(clip.duration)}s</span>
                            </span>
                          </div>
                        ))}
                    </div>
                  </div>
                </div>
              )}
            </div>
          </section>
        </div>

        {/* Right Sidebar: Inspector Panel */}
        <aside className="w-80 flex-shrink-0 border-l border-gray-700 bg-gray-800/50 backdrop-blur-md flex flex-col overflow-hidden">
          {/* Header */}
          <div className="p-4 border-b border-gray-700 bg-gray-900/50 flex items-center justify-between">
            <h2 className="text-xs font-bold uppercase tracking-wider text-gray-300 flex items-center gap-2">
              <HiOutlineAdjustments className="h-4 w-4 text-blue-400" /> Inspector Panel
            </h2>
          </div>

          <div className="flex-1 overflow-y-auto custom-scrollbar p-4 space-y-6">

            {/* Subtitle / Caption Configuration */}
            <div className="space-y-4">
              <div className="flex items-center justify-between">
                <h3 className="text-[10px] font-bold text-gray-400 uppercase tracking-wider flex items-center gap-1.5">
                  <HiOutlineSparkles className="h-3.5 w-3.5 text-blue-400" /> Subtitles
                </h3>
                <label className="relative inline-flex items-center cursor-pointer">
                  <input 
                    type="checkbox" 
                    className="sr-only peer" 
                    checked={enableCaptions} 
                    onChange={(e) => setEnableCaptions(e.target.checked)} 
                  />
                  <div className="w-9 h-5 bg-gray-700 peer-focus:outline-none rounded-full peer peer-checked:after:translate-x-full peer-checked:after:border-white after:content-[''] after:absolute after:top-[2px] after:left-[2px] after:bg-white after:border-gray-300 after:border after:rounded-full after:h-4 after:w-4 after:transition-all peer-checked:bg-blue-600"></div>
                </label>
              </div>

              {!enableCaptions ? (
                <div className="text-center p-4 border border-dashed border-gray-750 rounded-lg text-gray-500 text-xs">
                  Subtitles are disabled. Enable them above to customize styles.
                </div>
              ) : (
                <div className="space-y-5">
                  {/* Style Presets */}
                  <div className="space-y-2">
                    <h4 className="text-[9px] font-bold text-gray-400 uppercase tracking-wider">Style Presets</h4>
                    <div className="grid grid-cols-2 gap-2">
                      {STYLE_PRESETS.map((preset) => (
                        <button
                          key={preset.name}
                          onClick={() => setCaptionStyles(prev => ({
                            ...prev,
                            fontFamily: preset.fontFamily,
                            fontSize: preset.fontSize,
                            textColor: preset.textColor,
                            outlineColor: preset.outlineColor,
                            outlineWidth: preset.outlineWidth,
                            backgroundColor: preset.backgroundColor,
                            bold: preset.bold,
                            italic: preset.italic,
                          }))}
                          className={`p-2 rounded-lg border text-center transition-all cursor-pointer hover:scale-[1.02] ${
                            captionStyles.fontFamily === preset.fontFamily &&
                            captionStyles.textColor === preset.textColor &&
                            captionStyles.outlineColor === preset.outlineColor &&
                            captionStyles.backgroundColor === preset.backgroundColor
                              ? "bg-blue-600/10 border-blue-500 shadow-md"
                              : "bg-gray-850 border-gray-750 hover:border-gray-650"
                          }`}
                        >
                          <div className="text-[9px] font-semibold text-gray-400 mb-1">{preset.name}</div>
                          <div
                            style={{
                              fontFamily: preset.fontFamily,
                              color: preset.textColor,
                              textShadow: preset.outlineWidth > 0 
                                ? `
                                  -1px -1px 0 ${preset.outlineColor},  
                                   1px -1px 0 ${preset.outlineColor},
                                  -1px  1px 0 ${preset.outlineColor},
                                   1px  1px 0 ${preset.outlineColor}
                                `
                                : "none",
                              backgroundColor: preset.backgroundColor !== "transparent" ? preset.backgroundColor : "transparent",
                              borderRadius: "3px",
                              padding: preset.backgroundColor !== "transparent" ? "1px 4px" : "0",
                            }}
                            className="text-[10px] font-bold line-clamp-1 truncate"
                          >
                            AaBbCc
                          </div>
                        </button>
                      ))}
                    </div>
                  </div>

                  {/* Manual Subtitle Tuning */}
                  <div className="space-y-4 pt-1">
                    {/* Font Family */}
                    <div className="space-y-1.5">
                      <label className="text-[9px] font-bold text-gray-400 uppercase tracking-wider block">Font Family</label>
                      <select
                        value={captionStyles.fontFamily}
                        onChange={(e) => setCaptionStyles(prev => ({ ...prev, fontFamily: e.target.value }))}
                        className="w-full text-xs bg-gray-900 border border-gray-700 rounded-lg py-2 px-3 text-white focus:outline-none focus:border-blue-500 cursor-pointer"
                      >
                        <option value="Arial">Arial</option>
                        <option value="Impact">Impact (Meme)</option>
                        <option value="Montserrat">Montserrat (Modern Bold)</option>
                        <option value="Outfit">Outfit (Premium Rounded)</option>
                        <option value="Inter">Inter (Clean Sans)</option>
                        <option value="Courier New">Courier New (Typewriter)</option>
                      </select>
                    </div>

                    {/* Font Size */}
                    <div className="space-y-1.5">
                      <div className="flex justify-between items-center">
                        <label className="text-[9px] font-bold text-gray-400 uppercase tracking-wider">Font Size</label>
                        <span className="text-xs font-mono text-blue-400">{captionStyles.fontSize}px</span>
                      </div>
                      <input 
                        type="range" 
                        min="14" 
                        max="48" 
                        value={captionStyles.fontSize}
                        onChange={(e) => setCaptionStyles(prev => ({ ...prev, fontSize: parseInt(e.target.value) }))}
                        className="w-full h-2 bg-gray-700 rounded-full appearance-none cursor-pointer premium-range"
                      />
                    </div>

                    {/* Caption Width */}
                    <div className="space-y-1.5">
                      <div className="flex justify-between items-center">
                        <label className="text-[9px] font-bold text-gray-400 uppercase tracking-wider">Caption Width</label>
                        <span className="text-xs font-mono text-blue-400">{captionStyles.captionWidth ?? 90}%</span>
                      </div>
                      <input 
                        type="range" 
                        min="20" 
                        max="100" 
                        step="5"
                        value={captionStyles.captionWidth ?? 90}
                        onChange={(e) => setCaptionStyles(prev => ({ ...prev, captionWidth: parseInt(e.target.value) }))}
                        className="w-full h-2 bg-gray-700 rounded-full appearance-none cursor-pointer premium-range"
                      />
                    </div>

                    {/* Style and Position Toggles */}
                    <div className="grid grid-cols-2 gap-3">
                      <div className="space-y-1.5">
                        <label className="text-[9px] font-bold text-gray-400 uppercase tracking-wider block">Emphasis</label>
                        <div className="flex bg-gray-900 border border-gray-700 rounded-lg p-1 gap-1">
                          <button
                            onClick={() => setCaptionStyles(prev => ({ ...prev, bold: !prev.bold }))}
                            className={`flex-1 py-1 rounded text-xs font-bold transition-all cursor-pointer ${
                              captionStyles.bold 
                                ? "bg-blue-600 text-white shadow" 
                                : "text-gray-400 hover:text-white"
                            }`}
                          >
                            B
                          </button>
                          <button
                            onClick={() => setCaptionStyles(prev => ({ ...prev, italic: !prev.italic }))}
                            className={`flex-1 py-1 rounded text-xs font-bold italic transition-all cursor-pointer ${
                              captionStyles.italic 
                                ? "bg-blue-600 text-white shadow" 
                                : "text-gray-400 hover:text-white"
                            }`}
                          >
                            I
                          </button>
                        </div>
                      </div>

                      <div className="space-y-1.5">
                        <label className="text-[9px] font-bold text-gray-400 uppercase tracking-wider block">Alignment</label>
                        <div className="flex bg-gray-900 border border-gray-700 rounded-lg p-1 gap-0.5">
                          {(["top", "center", "bottom"] as const).map((align) => (
                            <button
                              key={align}
                              onClick={() => handleAlignmentChange(align)}
                              className={`flex-1 py-1 rounded text-[9px] font-semibold capitalize transition-all cursor-pointer ${
                                captionStyles.alignment === align 
                                  ? "bg-blue-600 text-white shadow" 
                                  : "text-gray-400 hover:text-white"
                              }`}
                            >
                              {align}
                            </button>
                          ))}
                        </div>
                      </div>
                    </div>

                    {/* Draggable Coordinates Reset & Info */}
                    <div className="bg-gray-900/40 border border-gray-750 rounded-lg p-2.5 space-y-2">
                      <div className="flex justify-between items-center text-[10px] text-gray-400 font-medium">
                        <span>Position (X, Y):</span>
                        <span className="font-mono text-blue-400 bg-blue-950/40 px-1.5 py-0.5 rounded">
                          {captionStyles.positionX ?? 50}%, {captionStyles.positionY ?? 85}%
                        </span>
                      </div>
                      <p className="text-[9px] text-gray-500 leading-normal">
                        💡 Click and drag the subtitles directly inside the video preview player window to adjust custom placements.
                      </p>
                      <button
                        onClick={() => {
                          setCaptionStyles(prev => ({
                            ...prev,
                            alignment: "bottom",
                            positionX: 50,
                            positionY: 85
                          }));
                        }}
                        className="w-full py-1.5 rounded bg-gray-850 hover:bg-gray-750 text-[10px] font-bold text-gray-300 flex items-center justify-center gap-1.5 transition-colors cursor-pointer border border-gray-700"
                      >
                        <HiOutlineRefresh className="h-3.5 w-3.5" /> Reset Subtitle Position
                      </button>
                    </div>

                    {/* Colors */}
                    <div className="space-y-2.5">
                      <label className="text-[9px] font-bold text-gray-400 uppercase tracking-wider block">Colors & Outline</label>
                      
                      {/* Text Color */}
                      <div className="flex items-center justify-between bg-gray-900/60 p-2 rounded-lg border border-gray-750">
                        <span className="text-xs text-gray-300">Text Color</span>
                        <div className="flex items-center gap-2">
                          <input 
                            type="color" 
                            value={captionStyles.textColor} 
                            onChange={(e) => setCaptionStyles(prev => ({ ...prev, textColor: e.target.value }))}
                            className="w-6 h-6 border-0 bg-transparent rounded cursor-pointer"
                          />
                          <span className="text-[10px] font-mono text-gray-400">{captionStyles.textColor.toUpperCase()}</span>
                        </div>
                      </div>

                      {/* Outline Style */}
                      <div className="bg-gray-900/60 p-2.5 rounded-lg border border-gray-750 space-y-2">
                        <div className="flex items-center justify-between">
                          <span className="text-xs text-gray-300">Outline Color</span>
                          <div className="flex items-center gap-2">
                            <input 
                              type="color" 
                              value={captionStyles.outlineColor} 
                              onChange={(e) => setCaptionStyles(prev => ({ ...prev, outlineColor: e.target.value }))}
                              className="w-6 h-6 border-0 bg-transparent rounded cursor-pointer"
                            />
                            <span className="text-[10px] font-mono text-gray-400">{captionStyles.outlineColor.toUpperCase()}</span>
                          </div>
                        </div>
                        
                        <div className="space-y-1">
                          <div className="flex justify-between items-center">
                            <span className="text-[9px] text-gray-400">Outline Thickness</span>
                            <span className="text-[9px] font-mono text-blue-400">{captionStyles.outlineWidth}px</span>
                          </div>
                          <input 
                            type="range" 
                            min="0" 
                            max="6" 
                            value={captionStyles.outlineWidth}
                            onChange={(e) => setCaptionStyles(prev => ({ ...prev, outlineWidth: parseInt(e.target.value) }))}
                            className="w-full h-1 bg-gray-700 rounded-full appearance-none cursor-pointer premium-range"
                          />
                        </div>
                      </div>

                      {/* Background Style */}
                      <div className="bg-gray-900/60 p-2.5 rounded-lg border border-gray-750 space-y-2.5">
                        <div className="flex items-center justify-between">
                          <span className="text-xs text-gray-300">Background Card</span>
                          <select
                            value={captionStyles.backgroundColor === "transparent" ? "none" : captionStyles.backgroundColor}
                            onChange={(e) => {
                              const val = e.target.value;
                              setCaptionStyles(prev => ({
                                ...prev,
                                backgroundColor: val === "none" ? "transparent" : val
                              }));
                            }}
                            className="text-xs bg-gray-950 border border-gray-800 rounded py-1 px-2 text-white focus:outline-none cursor-pointer"
                          >
                            <option value="none">No Background</option>
                            <option value="#00000080">Semi-Black (50%)</option>
                            <option value="#000000cc">Medium Black (80%)</option>
                            <option value="#000000">Solid Black</option>
                            <option value="#0000ff80">Semi-Blue</option>
                            <option value="#ff000080">Semi-Red</option>
                          </select>
                        </div>
                        {captionStyles.backgroundColor !== "transparent" && (
                          <div className="flex items-center justify-between">
                            <span className="text-[9px] text-gray-400">Custom Background Color</span>
                            <div className="flex items-center gap-2">
                              <input 
                                type="color" 
                                value={captionStyles.backgroundColor.substring(0, 7)} 
                                onChange={(e) => {
                                  const hex = e.target.value;
                                  const alpha = captionStyles.backgroundColor.length === 9 ? captionStyles.backgroundColor.substring(7, 9) : "CC";
                                  setCaptionStyles(prev => ({ ...prev, backgroundColor: hex + alpha }));
                                }}
                                className="w-6 h-6 border-0 bg-transparent rounded cursor-pointer"
                              />
                              <span className="text-[9px] font-mono text-gray-400">{captionStyles.backgroundColor.toUpperCase()}</span>
                            </div>
                          </div>
                        )}
                      </div>
                    </div>
                  </div>
                </div>
              )}
            </div>
          </div>
        </aside>
      </div>

      {/* Context Menu for Timeline Clips */}
      {contextMenu.visible && contextMenu.clip && (
        <div 
          className="absolute bg-gray-900 border border-gray-700 rounded-lg shadow-2xl py-1.5 z-50 text-xs text-gray-200 min-w-[175px] divide-y divide-gray-800 backdrop-blur-lg bg-opacity-95 shadow-blue-500/10"
          style={{ top: `${contextMenu.y}px`, left: `${contextMenu.x}px`, position: "fixed" }}
          onClick={(e) => e.stopPropagation()}
          onContextMenu={(e) => e.preventDefault()}
        >
          <div className="px-3 py-1.5 text-[10px] text-gray-400 font-bold uppercase tracking-wider select-none truncate max-w-[160px]">
            {contextMenu.clip.title}
          </div>
          <div className="py-1">
            <button 
              type="button"
              className="w-full text-left px-3.5 py-2 hover:bg-blue-600 hover:text-white flex items-center gap-2 transition-colors font-medium cursor-pointer"
              onClick={() => {
                handleExportSingleClip(contextMenu.clip!);
                setContextMenu(prev => ({ ...prev, visible: false }));
              }}
            >
              <HiOutlineDownload className="h-4 w-4 text-blue-400" /> Export Single Clip
            </button>
          </div>
          <div className="py-1">
            <button 
              type="button"
              className="w-full text-left px-3.5 py-2 hover:bg-red-600 hover:text-white flex items-center gap-2 transition-colors text-red-400 hover:text-white font-medium cursor-pointer"
              onClick={() => {
                deleteClip(contextMenu.clip!.id);
                setContextMenu(prev => ({ ...prev, visible: false }));
              }}
            >
              <HiOutlineTrash className="h-4 w-4" /> Delete Clip
            </button>
          </div>
        </div>
      )}
    </div>
  );
}
```

---

## 🚀 PANDUAN PENGOPERASIAN & INSTALASI DI WORKSPACE BARU

Jika Anda ingin menduplikasi proyek ini di folder baru, ikuti langkah-langkah mudah berikut:

1. **Inisialisasi Proyek Next.js Baru**:
   Buat direktori baru dan jalankan inisialisasi Next.js (pastikan menggunakan folder target kosong `./`):
   ```bash
   npx -y create-next-app@latest ./ --ts --eslint --tailwind --src-dir --app --import-alias "@/*"
   ```

2. **Instal Dependensi Pendukung**:
   Pastikan dependensi diinstal secara lengkap:
   ```bash
   npm install @distube/ytdl-core ffmpeg-static ffprobe-static flowbite flowbite-react fluent-ffmpeg react-icons youtube-dl-exec
   npm install --save-dev @tailwindcss/postcss @types/fluent-ffmpeg
   ```

3. **Salin Kode File**:
   Salin masing-masing kode fungsional penuh di atas ke file-file yang bersangkutan sesuai struktur folder.

4. **Unduh Binary Dependency (`yt-dlp`)**:
   Untuk memastikan yt-dlp berjalan lancar, jalankan skrip install post-install dari youtube-dl-exec untuk mengunduh binary yang tepat untuk platform OS Anda:
   ```bash
   node node_modules/youtube-dl-exec/bin/install_binaries.js
   ```

5. **Jalankan Aplikasi Web**:
   Jalankan server development lokal Anda:
   ```bash
   npm run dev
   ```
   Buka browser di `http://localhost:3000` dan nikmati pengalaman mengedit video YouTube viral dengan auto-subtitle kustomisasi penuh!
```
