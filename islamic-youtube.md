# Meridian — Feature Roadmap & Architecture

**A self-hosted, privacy-first video platform that makes YouTube feel like a fax machine.**

Meridian is a video sharing and streaming platform designed to be deployed by individuals, communities, schools, or organizations. Zero tracking. Full control. Beautiful by default. Runs on a $5/month VPS or a Raspberry Pi.

---

## Philosophy

```
1. Own your content     — Your server, your videos, your data
2. Privacy by default   — No tracking, no ads, no algorithmic manipulation
3. Works everywhere     — Phone, tablet, desktop, smart TV, terminal
4. Federation optional  — Connect to other Meridian instances (ActivityPub)
5. AI enhances, never controls — AI tools assist creators, never decide what viewers see
6. Offline-first        — Download, sync, watch anywhere
7. One-command deploy   — docker compose up, done
```

---

## Project Structure

```
meridian/
├── docker-compose.yml            # One-command deployment
├── Dockerfile                    # Multi-stage build
├── meridian.sh                   # Bare-metal installer/launcher
│
├── backend/
│   ├── main.py                   # FastAPI entry point
│   ├── config.py                 # Settings & env management
│   ├── db.py                     # SQLAlchemy / SQLite+PostgreSQL
│   ├── models/
│   │   ├── __init__.py
│   │   ├── user.py               # User, Channel, Subscription
│   │   ├── video.py              # Video, Metadata, Thumbnail
│   │   ├── comment.py            # Comment, Reply, Reaction
│   │   ├── playlist.py           # Playlist, PlaylistItem
│   │   ├── live.py               # LiveStream, ChatMessage
│   │   ├── notification.py       # Notification queue
│   │   └── federation.py         # ActivityPub objects
│   │
│   ├── api/
│   │   ├── __init__.py
│   │   ├── auth.py               # JWT + OAuth2 + passkeys
│   │   ├── videos.py             # Upload, CRUD, search
│   │   ├── channels.py           # Channel management
│   │   ├── comments.py           # Threaded comments
│   │   ├── playlists.py          # Playlist CRUD
│   │   ├── live.py               # Live streaming endpoints
│   │   ├── search.py             # Full-text + semantic search
│   │   ├── feed.py               # Subscription feed, trending
│   │   ├── notifications.py      # Push + in-app notifications
│   │   ├── analytics.py          # Creator analytics
│   │   ├── admin.py              # Instance admin panel
│   │   ├── federation.py         # ActivityPub inbox/outbox
│   │   ├── ai.py                 # AI endpoints (captions, chapters)
│   │   ├── embed.py              # oEmbed + iframe embed API
│   │   └── export.py             # GDPR data export
│   │
│   ├── services/
│   │   ├── __init__.py
│   │   ├── transcoder.py         # FFmpeg pipeline manager
│   │   ├── thumbnail.py          # Auto-thumbnail generation
│   │   ├── storage.py            # Local / S3 / B2 abstraction
│   │   ├── search_engine.py      # Meilisearch / SQLite FTS5
│   │   ├── email.py              # SMTP notifications
│   │   ├── scheduler.py          # Cron-like task runner
│   │   ├── abuse.py              # Content moderation pipeline
│   │   ├── federation_worker.py  # ActivityPub delivery
│   │   └── ai_worker.py          # Background AI processing
│   │
│   ├── workers/
│   │   ├── transcode_worker.py   # Video processing queue
│   │   ├── thumbnail_worker.py   # Thumbnail extraction
│   │   ├── caption_worker.py     # AI caption generation
│   │   └── federation_worker.py  # Federated delivery
│   │
│   └── migrations/               # Alembic DB migrations
│       └── ...
│
├── frontend/
│   ├── package.json
│   ├── src/
│   │   ├── App.jsx               # Root component + router
│   │   ├── main.jsx              # Entry point
│   │   │
│   │   ├── pages/
│   │   │   ├── Home.jsx          # Feed / trending / subscriptions
│   │   │   ├── Watch.jsx         # Video player page
│   │   │   ├── Channel.jsx       # Channel profile
│   │   │   ├── Upload.jsx        # Upload wizard
│   │   │   ├── Studio.jsx        # Creator dashboard
│   │   │   ├── Search.jsx        # Search results
│   │   │   ├── Library.jsx       # History, Watch Later, Playlists
│   │   │   ├── Live.jsx          # Live stream page
│   │   │   ├── Shorts.jsx        # Vertical short-form videos
│   │   │   ├── Explore.jsx       # Categories, trending, discover
│   │   │   ├── Settings.jsx      # User preferences
│   │   │   ├── Admin.jsx         # Instance admin panel
│   │   │   └── Theater.jsx       # Watch party / co-viewing
│   │   │
│   │   ├── components/
│   │   │   ├── player/
│   │   │   │   ├── VideoPlayer.jsx       # Custom player shell
│   │   │   │   ├── Controls.jsx          # Play, seek, volume, CC
│   │   │   │   ├── ChapterBar.jsx        # Clickable chapter markers
│   │   │   │   ├── SpeedControl.jsx      # 0.25x — 4x
│   │   │   │   ├── QualitySelector.jsx   # Adaptive + manual
│   │   │   │   ├── SubtitleOverlay.jsx   # Multi-language subs
│   │   │   │   ├── SponsorBlock.jsx      # Community skip segments
│   │   │   │   ├── Annotations.jsx       # Creator annotations
│   │   │   │   ├── MiniPlayer.jsx        # PiP / floating player
│   │   │   │   ├── TheaterMode.jsx       # Expanded view
│   │   │   │   ├── AmbientMode.jsx       # Color bleed background
│   │   │   │   ├── Gestures.jsx          # Mobile swipe controls
│   │   │   │   └── KeyboardShorts.jsx    # Keyboard shortcuts
│   │   │   │
│   │   │   ├── feed/
│   │   │   │   ├── VideoCard.jsx         # Thumbnail + metadata card
│   │   │   │   ├── VideoGrid.jsx         # Responsive grid layout
│   │   │   │   ├── InfiniteScroll.jsx    # Lazy loading
│   │   │   │   ├── FilterBar.jsx         # Sort & filter controls
│   │   │   │   └── CategoryPills.jsx     # Horizontal category chips
│   │   │   │
│   │   │   ├── channel/
│   │   │   │   ├── ChannelHeader.jsx     # Banner, avatar, stats
│   │   │   │   ├── ChannelTabs.jsx       # Videos, Shorts, Playlists, About
│   │   │   │   └── SubscribeButton.jsx   # Sub/unsub + notification bell
│   │   │   │
│   │   │   ├── comments/
│   │   │   │   ├── CommentThread.jsx     # Threaded comment tree
│   │   │   │   ├── CommentInput.jsx      # Rich text + emoji
│   │   │   │   └── Reactions.jsx         # Emoji reactions
│   │   │   │
│   │   │   ├── studio/
│   │   │   │   ├── UploadWizard.jsx      # Step-by-step upload
│   │   │   │   ├── VideoEditor.jsx       # Trim, cut, thumbnail pick
│   │   │   │   ├── AnalyticsDash.jsx     # Views, watch time, subs
│   │   │   │   ├── CommentMod.jsx        # Comment moderation
│   │   │   │   ├── MonetizeDash.jsx      # Tips, memberships, revenue
│   │   │   │   └── SchedulePublish.jsx   # Scheduled publishing
│   │   │   │
│   │   │   ├── live/
│   │   │   │   ├── LivePlayer.jsx        # Low-latency live player
│   │   │   │   ├── LiveChat.jsx          # Real-time chat
│   │   │   │   ├── StreamSetup.jsx       # OBS/RTMP config helper
│   │   │   │   └── GoLiveButton.jsx      # Browser-based streaming
│   │   │   │
│   │   │   ├── shared/
│   │   │   │   ├── Sidebar.jsx           # Navigation sidebar
│   │   │   │   ├── Header.jsx            # Search bar, avatar, upload
│   │   │   │   ├── Modal.jsx             # Reusable modal
│   │   │   │   ├── Toast.jsx             # Notification toasts
│   │   │   │   ├── Skeleton.jsx          # Loading skeletons
│   │   │   │   ├── ThemeProvider.jsx     # Theme context
│   │   │   │   └── AccessibleFocus.jsx   # Keyboard focus ring
│   │   │   │
│   │   │   └── admin/
│   │   │       ├── InstanceStats.jsx     # Server health, storage
│   │   │       ├── UserManagement.jsx    # Ban, roles, invites
│   │   │       ├── ContentModeration.jsx # Flagged content queue
│   │   │       ├── FederationPanel.jsx   # Connected instances
│   │   │       └── StoragePanel.jsx      # Disk usage, cleanup
│   │   │
│   │   ├── hooks/
│   │   │   ├── usePlayer.js              # Player state management
│   │   │   ├── useInfiniteScroll.js      # Scroll-based pagination
│   │   │   ├── useKeyboard.js            # Global keyboard shortcuts
│   │   │   ├── useOffline.js             # Offline detection + queue
│   │   │   └── useTheme.js               # Theme switching
│   │   │
│   │   ├── stores/
│   │   │   ├── authStore.js              # Auth state (Zustand)
│   │   │   ├── playerStore.js            # Player state
│   │   │   ├── queueStore.js             # Watch queue
│   │   │   └── notificationStore.js      # Notification state
│   │   │
│   │   └── utils/
│   │       ├── api.js                    # Fetch wrapper + auth
│   │       ├── time.js                   # Relative time formatting
│   │       ├── format.js                 # View counts, durations
│   │       └── accessibility.js          # ARIA helpers
│   │
│   └── public/
│       ├── index.html
│       ├── manifest.json                 # PWA manifest
│       ├── sw.js                         # Service worker (offline)
│       └── icons/                        # App icons
│
├── mobile/                               # React Native (Phase 12+)
│   └── ...
│
├── storage/                              # Default local storage
│   ├── videos/                           # Original uploads
│   ├── transcoded/                       # HLS/DASH segments
│   ├── thumbnails/                       # Generated thumbnails
│   ├── avatars/                          # User/channel avatars
│   └── captions/                         # Subtitle files (.vtt)
│
├── nginx/
│   ├── nginx.conf                        # Reverse proxy + HLS serving
│   └── ssl/                              # Let's Encrypt certs
│
├── docs/
│   ├── INSTALL.md                        # Installation guide
│   ├── API.md                            # REST API reference
│   ├── FEDERATION.md                     # ActivityPub spec
│   ├── CONTRIBUTING.md                   # Contributor guide
│   └── SELF_HOST.md                      # Deployment recipes
│
├── scripts/
│   ├── setup.sh                          # First-run setup wizard
│   ├── backup.sh                         # Automated backup
│   ├── migrate.sh                        # YouTube import tool
│   └── benchmark.sh                      # Performance testing
│
├── .env.example                          # Environment template
├── README.md
└── LICENSE                               # AGPLv3
```

---

## Menu Structure (Admin + Creator + Viewer)

```
VIEWER NAVIGATION
  🏠  Home                    Feed, trending, subscriptions
  🔍  Search                  Full-text + semantic + filters
  📺  Explore                 Categories, trending, discover
  🎬  Shorts                  Vertical short-form videos
  📚  Library                 History, Watch Later, Playlists, Downloads
  🔔  Notifications           Subscription updates, replies, mentions
  ⚙   Settings               Theme, language, playback, privacy

CREATOR STUDIO (/studio)
  📊  Dashboard               Overview: views, subs, revenue, recent
  📤  Upload                  Upload wizard + bulk upload
  🎞   Content                All videos, drafts, scheduled, live
  📈  Analytics               Deep analytics per video + channel
  💬  Comments                Moderation queue + bulk actions
  💰  Monetization            Tips, memberships, payouts
  ✂   Editor                  In-browser trim, chapters, thumbnails
  📡  Go Live                 Stream setup + browser streaming
  🔗  Embed Generator         Get embed codes + oEmbed
  📋  Subtitles               Upload / AI-generate / edit captions
  📦  Import                  YouTube takeout import wizard

ADMIN PANEL (/admin)
  📊  Instance Stats          CPU, RAM, disk, bandwidth, users
  👥  User Management         Roles, bans, invites, quotas
  🚩  Content Moderation      Flagged queue, auto-mod rules
  🌐  Federation              Connected instances, block list
  💾  Storage                 Usage breakdown, cleanup, S3 config
  🔒  Security                Rate limits, brute force, API keys
  🎨  Customization           Instance name, logo, theme, landing
  📧  Email                   SMTP config, templates, test
  📜  Audit Log               Every admin action, searchable
  🔧  Transcoding             Queue, presets, hardware accel config
  🤖  AI Settings             Enable/disable, model config, costs
  📊  Federation Analytics    Cross-instance video stats
```

---

## Feature Specifications

---

### 1. VIDEO PLAYER (components/player/)

**What it does:**
- Custom HTML5 video player built from scratch
- Adaptive bitrate streaming (HLS with fallback to progressive)
- Keyboard-driven, touch-friendly, screen-reader accessible
- Zero external player dependencies

**Player UI:**
```
+------------------------------------------------------------------+
|                                                                    |
|                                                                    |
|                         ▶  VIDEO                                   |
|                                                                    |
|                                                                    |
|  ┌──────────────────────────────────────────────────────────────┐  |
|  │ ▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░ │  |
|  │ Ch.1: Intro    │ Ch.2: Setup    │ Ch.3: Demo    │ Ch.4      │  |
|  └──────────────────────────────────────────────────────────────┘  |
|  ▶/❚❚  ◀◀ ▶▶  🔊━━━━━  2:34 / 12:07  CC  ⚙  🖥  📐  ⛶        |
+------------------------------------------------------------------+
|  Controls: Play, Skip ±10s, Volume, Time, CC, Settings,           |
|  Theater, PiP, Fullscreen                                          |
+------------------------------------------------------------------+
```

**Keyboard shortcuts:**
```
Space / K     Play/Pause
J / L         Seek -10s / +10s
← / →         Seek -5s / +5s
, / .         Frame step (paused)
F             Fullscreen
T             Theater mode
I             Mini-player (PiP)
M             Mute
↑ / ↓         Volume ±5%
C             Toggle captions
< / >         Speed -0.25x / +0.25x
0-9           Jump to 0%-90%
Shift+N       Next in playlist
Shift+P       Previous in playlist
```

**Adaptive quality:**
```
AUTO (recommended)
──────────────
  2160p  4K
  1440p  2K
  1080p  HD
  720p   HD
  480p
  360p
  Audio only
```

**Chapter bar:**
- Auto-generated from AI or manual timestamps in description
- Hovering a chapter shows preview thumbnail + title
- Clicking jumps to chapter start
- Chapter markers visible on seek bar as dividers

**Ambient mode:**
```
┌──────────────────────────────────────────────┐
│ ╭─────────────── soft color glow ──────────╮ │
│ │                                           │ │
│ │             VIDEO PLAYER                  │ │
│ │                                           │ │
│ ╰───────────────────────────────────────────╯ │
│  The page background subtly matches the       │
│  dominant colors of the current video frame.   │
│  CSS filter: blur(80px) + opacity(0.4)        │
└──────────────────────────────────────────────┘
```

**Implementation:**
- HTML `<video>` element with hls.js for adaptive streaming
- Custom controls overlay (no native controls)
- Chapter data from video metadata or AI-generated
- Ambient mode: hidden `<canvas>` samples video frames, applies CSS blur to page background
- PiP: uses browser Picture-in-Picture API
- Gestures: Hammer.js for mobile swipe/tap

---

### 2. TRANSCODING ENGINE (services/transcoder.py)

**What it does:**
- Accepts any video format (mp4, mkv, mov, avi, webm, etc.)
- Outputs HLS segments for adaptive streaming
- Generates multiple quality renditions automatically
- Hardware acceleration (NVENC, VAAPI, VideoToolbox)
- Thumbnail sprite sheets for seek preview

**Transcoding pipeline:**
```
UPLOAD
  │
  ▼
PROBE (ffprobe)
  │ Extract: resolution, codec, duration, bitrate, audio tracks
  │
  ▼
PLAN
  │ Based on source resolution, generate rendition ladder:
  │   Source 4K → 2160p, 1440p, 1080p, 720p, 480p, 360p
  │   Source 1080p → 1080p, 720p, 480p, 360p
  │   Source 720p → 720p, 480p, 360p
  │   Source vertical → same ladder, portrait aspect ratio
  │
  ▼
TRANSCODE (ffmpeg, parallel per rendition)
  │ Video: H.264 (compatibility) + H.265/AV1 (optional, smaller)
  │ Audio: AAC 128k stereo + opus (optional)
  │ Segment: HLS .ts segments, 6 second chunks
  │ Playlist: master.m3u8 + per-rendition playlist
  │
  ▼
THUMBNAILS
  │ Extract 1 frame per second → sprite sheet (10x10 grid JPEGs)
  │ Extract "best" frame via scene detection → poster thumbnail
  │ Generate 3 thumbnail candidates for creator to choose
  │
  ▼
METADATA
  │ Duration, file sizes per rendition, dominant colors
  │ Store in DB, mark video as "ready"
  │
  ▼
NOTIFY
  │ WebSocket push to creator: "Your video is ready!"
  │ If scheduled: queue for publish at set time
```

**Rendition ladder:**
```
┌────────────┬────────────┬──────────┬──────────┐
│ Quality    │ Resolution │ Bitrate  │ Codec    │
├────────────┼────────────┼──────────┼──────────┤
│ 2160p (4K) │ 3840×2160  │ 15 Mbps  │ H.264    │
│ 1440p (2K) │ 2560×1440  │ 8 Mbps   │ H.264    │
│ 1080p (HD) │ 1920×1080  │ 4.5 Mbps │ H.264    │
│ 720p  (HD) │ 1280×720   │ 2.5 Mbps │ H.264    │
│ 480p       │ 854×480    │ 1 Mbps   │ H.264    │
│ 360p       │ 640×360    │ 0.5 Mbps │ H.264    │
│ Audio only │ —          │ 128 Kbps │ AAC      │
└────────────┴────────────┴──────────┴──────────┘
```

**HLS output structure:**
```
storage/transcoded/{video_id}/
├── master.m3u8           # Adaptive manifest
├── 2160p/
│   ├── playlist.m3u8
│   ├── segment_000.ts
│   ├── segment_001.ts
│   └── ...
├── 1080p/
│   ├── playlist.m3u8
│   └── ...
├── 720p/
│   └── ...
├── 480p/
│   └── ...
├── 360p/
│   └── ...
├── thumbnails/
│   ├── poster.jpg        # Main thumbnail
│   ├── sprite_0.jpg      # Seek preview sprite sheet
│   ├── sprite_1.jpg
│   └── candidates/       # AI-picked thumbnail options
│       ├── thumb_01.jpg
│       ├── thumb_02.jpg
│       └── thumb_03.jpg
└── captions/
    ├── en.vtt            # Auto-generated English
    └── ...
```

**Config:**
```json
{
  "transcoding": {
    "enabled": true,
    "threads": 2,
    "hardware_accel": "auto",
    "max_resolution": 2160,
    "codec": "h264",
    "hls_segment_duration": 6,
    "generate_thumbnails": true,
    "thumbnail_candidates": 3,
    "audio_normalize": true,
    "remove_original_after": false,
    "queue_max_concurrent": 2
  }
}
```

**Admin transcoding dashboard:**
```
TRANSCODING QUEUE

  #   Video                  Status      Progress   ETA
  1   "How to Build a..."    ⚙ encoding  ████░░ 62% 4m
  2   "Morning Routine"      ⏳ queued    ░░░░░░     ~12m
  3   "Podcast Ep. 47"       ⏳ queued    ░░░░░░     ~20m

  RECENT
  4   "Rust Tutorial #5"     ✓ done      ██████ 100% —
  5   "Cat Compilation"      ✓ done      ██████ 100% —

  SETTINGS
  Threads: [2]  HW Accel: [VAAPI ▼]  Max concurrent: [2]
```

---

### 3. UPLOAD WIZARD (pages/Upload.jsx)

**What it does:**
- Drag-and-drop or file picker
- Multi-file upload with queue
- Real-time upload progress + transcoding progress
- Fill in metadata while video transcodes
- Schedule publish for future date/time
- Bulk upload mode for content migration

**Upload flow UI:**
```
+------------------------------------------------------------------+
| UPLOAD VIDEO                                                       |
|                                                                    |
|  ┌──────────────────────────────────────────────────────────────┐  |
|  │                                                              │  |
|  │         ⬆  Drag video files here or click to browse          │  |
|  │                                                              │  |
|  │         Supports: mp4, mkv, mov, avi, webm (up to 10 GB)    │  |
|  │                                                              │  |
|  └──────────────────────────────────────────────────────────────┘  |
|                                                                    |
|  ── AFTER FILE SELECTED ──────────────────────────────────────     |
|                                                                    |
|  Upload:  ████████████████████░░░░ 78%  (156 MB / 200 MB)        |
|  Transcode: ░░░░░░░░░░░░░░░░░░░░ waiting for upload...           |
|                                                                    |
|  DETAILS (fill while uploading)                                    |
|  ┌────────────────────────────────┬─────────────────────────────┐  |
|  │ Title:                         │  THUMBNAIL                  │  |
|  │ [How to Build a Weather Sta ]  │  ┌───────────────────────┐  │  |
|  │                                │  │                       │  │  |
|  │ Description:                   │  │   📷 (auto-generated) │  │  |
|  │ [In this video, I'll show   ]  │  │                       │  │  |
|  │ [you how to build a DIY     ]  │  └───────────────────────┘  │  |
|  │ [weather station with an    ]  │  [Upload custom] [AI pick]  │  |
|  │ [ESP32 and BME280 sensor... ]  │                             │  |
|  │                                │  VISIBILITY                 │  |
|  │ Tags:                          │  ○ Public                   │  |
|  │ [esp32, weather, arduino, DIY] │  ○ Unlisted                 │  |
|  │                                │  ○ Private                  │  |
|  │ Category:                      │                             │  |
|  │ [Science & Tech         ▼]     │  SCHEDULE                   │  |
|  │                                │  ○ Publish now              │  |
|  │ Language:                      │  ○ Schedule: 📅 [date/time] │  |
|  │ [English              ▼]       │  ○ Save as draft            │  |
|  │                                │                             │  |
|  │ Playlist:                      │  AI FEATURES                │  |
|  │ [+ Add to playlist      ▼]    │  ☑ Auto-generate captions   │  |
|  │                                │  ☑ Auto-detect chapters     │  |
|  │ Comments:                      │  ☑ Auto-generate tags       │  |
|  │ [Enabled ▼]                    │  ☐ AI description assist    │  |
|  └────────────────────────────────┴─────────────────────────────┘  |
|                                                                    |
|  [Publish]  [Save Draft]  [Cancel]                                |
+------------------------------------------------------------------+
```

**Bulk upload mode:**
```
BULK UPLOAD (Content Migration)

  Drag a folder or select multiple files:

  ┌──┬────────────────────────────┬──────┬────────┬──────────┐
  │# │ File                       │ Size │ Status │ Title    │
  ├──┼────────────────────────────┼──────┼────────┼──────────┤
  │1 │ weather-station.mp4        │ 200M │ ✓ done │ [edit]   │
  │2 │ morning-routine.mov        │ 1.2G │ ⚙ 34% │ [edit]   │
  │3 │ podcast-ep47.mkv           │ 890M │ ⏳ que │ [edit]   │
  │4 │ cat-compilation.mp4        │ 450M │ ⏳ que │ [edit]   │
  └──┴────────────────────────────┴──────┴────────┴──────────┘

  Apply to all:
  Category: [Science & Tech ▼]  Visibility: [Unlisted ▼]
  ☑ Auto-generate captions for all
  ☑ Auto-detect chapters for all

  [Upload All]  [Cancel]
```

**API endpoints:**
```
POST   /api/videos/upload            # Initiate upload (returns upload_id)
PUT    /api/videos/upload/{id}/chunk # Upload chunk (resumable)
PATCH  /api/videos/{id}              # Update metadata
POST   /api/videos/{id}/thumbnail    # Upload custom thumbnail
GET    /api/videos/{id}/status       # Transcoding progress (SSE)
POST   /api/videos/{id}/publish      # Publish / schedule
```

**Resumable upload:**
- Uses tus protocol for resumable uploads
- If connection drops, resumes from last successful chunk
- Chunks: 5 MB each
- Parallel chunk upload for faster speeds

---

### 4. SUBSCRIPTION FEED & DISCOVERY (pages/Home.jsx)

**What it does:**
- Chronological subscription feed (no algorithmic manipulation)
- Optional smart sorting (most relevant first, user-controlled)
- Trending: based on velocity (views/time), not total views
- Explore: curated categories, staff picks, instance highlights
- Zero tracking — no shadow profiles, no engagement optimization

**Home page layout:**
```
+------------------------------------------------------------------+
| 🔍 Search...                            [Upload] 🔔  👤          |
+--------+---------------------------------------------------------+
|        |                                                          |
| SIDE   |  [Subscriptions]  [Trending]  [Explore]  [New]          |
| BAR    |                                                          |
|        |  ┌──────────┐  ┌──────────┐  ┌──────────┐  ┌─────────┐ |
| 🏠 Home│  │ ░░░░░░░░ │  │ ░░░░░░░░ │  │ ░░░░░░░░ │  │ ░░░░░░░ │ |
| 🔥 Tren│  │ ░░thumb░░ │  │ ░░thumb░░ │  │ ░░thumb░░ │  │ ░░thum░ │ |
| 🎬 Shor│  │ ░░░░░░░░ │  │ ░░░░░░░░ │  │ ░░░░░░░░ │  │ ░░░░░░░ │ |
| 📚 Libr│  │ 12:34     │  │ 8:21      │  │ 45:02     │  │ 3:15    │ |
| ────── │  │          │  │          │  │          │  │         │ |
| 📺 Subs│  │ Title... │  │ Title... │  │ Title... │  │ Title.. │ |
|  Chan1 │  │ Channel  │  │ Channel  │  │ Channel  │  │ Channel │ |
|  Chan2 │  │ 12K · 3d │  │ 5K · 1d  │  │ 89K · 1w │  │ 200 · 2h│ |
|  Chan3 │  └──────────┘  └──────────┘  └──────────┘  └─────────┘ |
|  Chan4 │                                                         |
| ────── │  ┌──────────┐  ┌──────────┐  ┌──────────┐  ┌─────────┐ |
| ⚙ Sett │  │ ...      │  │ ...      │  │ ...      │  │ ...     │ |
|        │  └──────────┘  └──────────┘  └──────────┘  └─────────┘ |
+--------+---------------------------------------------------------+
```

**Feed algorithm (transparent, user-controlled):**
```
FEED SETTINGS (accessible in Settings)

  Feed type:
  ● Chronological         Most recent first, always
  ○ Smart (transparent)   Balances recency + your watch patterns
  ○ Channel priority      Your pinned channels first, then chrono

  Show me:
  ☑ Subscriptions
  ☑ Trending on this instance
  ☐ Federation (videos from connected instances)
  ☑ Creators I've liked before (anonymous, local-only)

  Never show:
  [+ Block channel]  [+ Block tag]  [+ Block category]
```

**Trending calculation (transparent):**
```python
# Trending score = velocity, not volume
# A video with 500 views in 2 hours ranks higher than
# a video with 50,000 views in 2 months

trending_score = (
    views_last_24h * 1.0 +
    likes_last_24h * 2.0 +
    comments_last_24h * 3.0 +
    shares_last_24h * 4.0
) / hours_since_publish

# Diversity bonus: max 2 videos per channel in trending
# New creator bonus: +20% for channels with < 1000 subs
# Penalty: none (no engagement bait detection needed when no ads)
```

---

### 5. CREATOR STUDIO (pages/Studio.jsx)

**What it does:**
- Full channel management dashboard
- Video content manager with bulk actions
- Real-time analytics
- Comment moderation tools
- Monetization overview (tips, memberships)
- Subtitle/caption management

**Studio dashboard:**
```
+------------------------------------------------------------------+
| CREATOR STUDIO                                   [View Channel]   |
+------------------------------------------------------------------+
|                                                                    |
|  OVERVIEW (last 28 days)                                          |
|  ┌────────────┐ ┌────────────┐ ┌────────────┐ ┌────────────┐    |
|  │  Views     │ │ Watch Time │ │ Subscribers│ │  Revenue   │     |
|  │  42,831    │ │ 1,247 hrs  │ │ +312       │ │  €47.20    │     |
|  │  ↑ 12%     │ │ ↑ 8%       │ │ ↑ 15%      │ │  ↑ 23%     │     |
|  └────────────┘ └────────────┘ └────────────┘ └────────────┘     |
|                                                                    |
|  [════════ Views Chart (last 28 days) ════════════════════]       |
|  |        ·  ·                                                    |
|  |  · ·  · ·  ·  ·                                               |
|  | ·   ··      ··  · · ·                                         |
|  +--+--+--+--+--+--+--+--+                                      |
|    W1   W2   W3   W4                                              |
|                                                                    |
|  LATEST VIDEOS                                                    |
|  ┌──────┬─────────────────────────┬───────┬────────┬──────────┐  |
|  │ Thumb│ Title                   │ Views │ Likes  │ Status   │  |
|  ├──────┼─────────────────────────┼───────┼────────┼──────────┤  |
|  │ ░░░░ │ Weather Station Build   │ 12.4K │ 891    │ ✓ Public │  |
|  │ ░░░░ │ ESP32 BLE Tutorial      │ 8.2K  │ 643    │ ✓ Public │  |
|  │ ░░░░ │ Podcast Episode 48      │ 2.1K  │ 187    │ 📅 Mar 3 │  |
|  │ ░░░░ │ [Draft] Soldering Tips  │ —     │ —      │ 📝 Draft │  |
|  └──────┴─────────────────────────┴───────┴────────┴──────────┘  |
|                                                                    |
|  RECENT COMMENTS (needing reply)                                  |
|  "Great tutorial! What sensor did you use?" — @maker42  (2h)     |
|  "The BLE part was confusing at 4:32" — @newbie99  (5h)          |
|  [View all comments →]                                            |
+------------------------------------------------------------------+
```

**Content manager:**
```
CONTENT                              [Upload] [Bulk Actions ▼]

  Filter: [All ▼]  Sort: [Date ▼]  Search: [_____________]

  ┌──┬──────┬──────────────────────┬───────┬──────┬────────┬──────┐
  │☐ │ Thumb│ Title                │ Vis.  │Views │ Date   │Status│
  ├──┼──────┼──────────────────────┼───────┼──────┼────────┼──────┤
  │☐ │ ░░░░ │ Weather Station...   │ 🟢pub │12.4K │ Feb 28 │ ✓    │
  │☐ │ ░░░░ │ ESP32 BLE Tutorial   │ 🟢pub │ 8.2K │ Feb 25 │ ✓    │
  │☐ │ ░░░░ │ Podcast Episode 48   │ 🟡unl │  —   │ Mar 3  │ 📅   │
  │☑ │ ░░░░ │ Soldering Tips       │ ⚫dra │  —   │ —      │ 📝   │
  │☐ │ ░░░░ │ Cat Compilation      │ 🔴pri │  45  │ Feb 20 │ ✓    │
  └──┴──────┴──────────────────────┴───────┴──────┴────────┴──────┘

  BULK ACTIONS (with selected):
  • Make public / unlisted / private
  • Add to playlist
  • Delete
  • Re-generate captions
  • Download originals
```

---

### 6. ANALYTICS ENGINE (api/analytics.py)

**What it does:**
- Per-video and per-channel analytics
- Watch time, retention curve, traffic sources
- Real-time viewer count
- Audience demographics (anonymous, aggregated)
- Revenue tracking (tips, memberships)
- All data stored locally — never sent anywhere

**Per-video analytics:**
```
VIDEO ANALYTICS: "How to Build a Weather Station"

  OVERVIEW (last 28 days)
  ┌────────────┐ ┌────────────┐ ┌────────────┐ ┌────────────┐
  │  Views     │ │ Watch Time │ │  Avg View  │ │  Likes     │
  │  12,431    │ │  847 hrs   │ │  4:06 min  │ │  891 (94%) │
  └────────────┘ └────────────┘ └────────────┘ └────────────┘

  RETENTION CURVE
  100%|████
   80%|███████
   60%|█████████████
   40%|██████████████████
   20%|██████████████████████████████
      +----+----+----+----+----+----+
      0:00  2:00  4:00  6:00  8:00  12:07

  Avg percentage viewed: 34%
  Key drop-off: 1:42 (intro too long?)
  Re-watched segment: 5:10–6:30 (wiring diagram)

  TRAFFIC SOURCES
  ┌──────────────────────┬───────┬────────┐
  │ Source               │ Views │ %      │
  ├──────────────────────┼───────┼────────┤
  │ Browse (home feed)   │ 5,432 │ 43.7%  │
  │ Search               │ 3,211 │ 25.8%  │
  │ External (reddit)    │ 1,890 │ 15.2%  │
  │ Direct link          │ 1,102 │  8.9%  │
  │ Suggested / related  │   796 │  6.4%  │
  └──────────────────────┴───────┴────────┘

  SEARCH TERMS (that led to this video)
  "esp32 weather station"          1,204 views
  "diy weather station tutorial"     892 views
  "bme280 arduino"                   645 views

  REAL-TIME
  Watching now: 23 viewers
  Views today: 342
```

**Retention curve implementation:**
- Player sends anonymous "heartbeat" events every 10 seconds: `{video_id, second, session_id}`
- Server aggregates into second-by-second view counts
- Retention = viewers_at_second / viewers_at_second_0
- No personal data: session IDs are random, not tied to users
- Data stored in time-series format for efficient querying

---

### 7. COMMENT SYSTEM (api/comments.py)

**What it does:**
- Threaded replies (unlimited depth, collapsed after 3)
- Emoji reactions (not just like/dislike)
- Pinned comments (by creator)
- Heart reaction from creator
- Auto-moderation (configurable word filters, link blocking)
- Timestamped comments (click to jump to moment)
- Edit and delete (with edit history)

**Comment UI:**
```
COMMENTS (247)                                    [Sort: Top ▼]

  ┌─────────────────────────────────────────────────────────┐
  │ Add a comment...                              [Post]    │
  └─────────────────────────────────────────────────────────┘

  📌 PINNED
  ┌─────────────────────────────────────────────────────────┐
  │ 👤 @creator · 3 days ago                                │
  │ Thanks for watching! Parts list in the description.     │
  │ Next video drops Friday — ESP32 with LoRa!              │
  │ 👍 142  💬 12 replies                                   │
  └─────────────────────────────────────────────────────────┘

  👤 @maker42 · 2 hours ago
  Great tutorial! What sensor did you use for humidity?
  The part at 4:32 was really clear.          ← [4:32 clickable]
  👍 23  ❤ (creator hearted)  💬 3 replies

    ↳ 👤 @creator · 1 hour ago
      BME280! Link in the description. It does temp,
      humidity, and pressure all in one.
      👍 8

    ↳ 👤 @maker42 · 45 min ago
      Perfect, just ordered one. Thanks!
      👍 2

    ↳ [Show 1 more reply]

  👤 @newbie99 · 5 hours ago
  The BLE part was confusing at 4:32. Could you explain
  the advertising interval parameter?
  👍 7  💬 1 reply  [Show reply]

  [Load more comments...]
```

**Moderation tools (Creator Studio):**
```
COMMENT MODERATION

  Filter: [Held for review ▼]  [All videos ▼]

  ┌──┬──────────────────────────────────┬──────────┬─────────┐
  │☐ │ Comment                          │ Video    │ Action  │
  ├──┼──────────────────────────────────┼──────────┼─────────┤
  │☐ │ "Check out my channel for..."    │ Weather  │ [✓] [✗] │
  │☐ │ "You're an idiot for using..."   │ BLE Tut  │ [✓] [✗] │
  │☐ │ "Great video! <script>..."       │ Podcast  │ [✓] [✗] │
  └──┴──────────────────────────────────┴──────────┴─────────┘

  AUTO-MOD RULES
  ☑ Hold comments with links for review
  ☑ Block comments matching word list: [edit list]
  ☐ Hold comments from new accounts (< 1 day old)
  ☑ Block comments with excessive caps (>70%)
```

---

### 8. LIVE STREAMING (api/live.py)

**What it does:**
- RTMP ingest (compatible with OBS, Streamlabs)
- Browser-based streaming (WebRTC → server → HLS)
- Low-latency HLS (~3-5 second delay)
- Live chat with moderation
- Stream health monitoring
- Auto-record and publish VOD after stream ends
- Scheduled streams with countdown

**Stream setup:**
```
GO LIVE

  STREAM SETTINGS
  Title:       [Friday Build Session — ESP32 LoRa       ]
  Description: [Live build session! Ask questions in chat]
  Category:    [Science & Tech ▼]
  Visibility:  [Public ▼]
  Thumbnail:   [Upload] or [Use camera frame]

  STREAM KEY (for OBS / Streamlabs)
  ┌──────────────────────────────────────────────────┐
  │ Server:  rtmp://your-instance.com/live           │
  │ Key:     sk_live_a3f2c1b8e9d4...  [Copy] [Show] │
  └──────────────────────────────────────────────────┘

  ── OR ──

  [📹 Stream from Browser]
  (uses your webcam + mic, no OBS needed)

  CHAT SETTINGS
  ☑ Enable live chat
  ☑ Slow mode (1 message per 5 seconds)
  ☐ Subscriber-only chat
  ☐ Disable chat entirely

  [Go Live]  [Schedule for Later: 📅]
```

**Live viewer UI:**
```
+------------------------------------------------------------------+
|                                                                    |
|                    🔴 LIVE  ·  847 watching                        |
|                                                                    |
|                         VIDEO FEED                                 |
|                                                                    |
|  ┌──────────────────────────────────────────┬───────────────────┐  |
|  │                                          │ LIVE CHAT         │  |
|  │          [stream content]                │                   │  |
|  │                                          │ @v1: Amazing!     │  |
|  │                                          │ @v2: What board?  │  |
|  │                                          │ @mod: 🔧 timeout  │  |
|  │                                          │ @v4: Show wiring? │  |
|  │                                          │ @cr: Sure!        │  |
|  │                                          │                   │  |
|  │                                          │ [Type msg] [Send] │  |
|  └──────────────────────────────────────────┴───────────────────┘  |
+------------------------------------------------------------------+
```

**Implementation:**
- RTMP ingest: nginx-rtmp module or SRS (Simple Realtime Server)
- Browser streaming: getUserMedia → WebRTC → server converts to RTMP
- Transcoding: real-time HLS with low-latency mode (LL-HLS or CMAF)
- Chat: WebSocket server (built into FastAPI)
- VOD: after stream ends, transcode recording into standard HLS
- Stream health: bitrate, FPS, dropped frames, shown to streamer

---

### 9. SEARCH ENGINE (api/search.py)

**What it does:**
- Full-text search across titles, descriptions, captions, tags
- Semantic search: "videos about building things" finds DIY content
- Filters: duration, date, category, resolution, live/VOD, channel
- Search within a channel
- Caption search: find the exact moment someone says something
- Federated search: query connected instances (optional)

**Search UI:**
```
+------------------------------------------------------------------+
| 🔍 [esp32 weather station                              ] [Search] |
+------------------------------------------------------------------+
|                                                                    |
|  Filters: [Duration ▼] [Date ▼] [Type ▼] [Category ▼] [Sort ▼]  |
|           [< 4 min] [4-20 min] [> 20 min]                        |
|           [Today] [This week] [This month] [This year]            |
|           [Video] [Short] [Live] [Playlist] [Channel]             |
|                                                                    |
|  About 47 results (0.12s)                                         |
|                                                                    |
|  ┌──────────┐  How to Build a Weather Station with ESP32          |
|  │ ░░░░░░░░ │  12.4K views · 3 days ago                          |
|  │ ░░thumb░░ │  @creator · "In this video I'll show you how..."   |
|  │ 12:07     │  Matched in: title, description, captions          |
|  └──────────┘                                                     |
|                                                                    |
|  ┌──────────┐  ESP32 Weather Station — Complete Guide             |
|  │ ░░░░░░░░ │  45.2K views · 2 months ago                        |
|  │ ░░thumb░░ │  @maker_pro · "Everything you need to know..."     |
|  │ 34:21     │  Matched in: title, tags                           |
|  └──────────┘                                                     |
|                                                                    |
|  📝 CAPTION MATCHES                                               |
|  ┌──────────┐  Advanced Sensor Calibration                        |
|  │ ░░░░░░░░ │  "...the ESP32 reads the weather station data       |
|  │ ░░[8:42]░ │  through the I2C bus at address 0x76..."           |
|  │ 22:15     │  [Jump to 8:42 →]                                  |
|  └──────────┘                                                     |
+------------------------------------------------------------------+
```

**Implementation:**
- Primary: Meilisearch (fast, typo-tolerant, lightweight)
- Fallback: SQLite FTS5 (zero-dep for small instances)
- Caption indexing: every 30-second chunk of auto-generated VTT indexed
- Semantic: optional, uses sentence-transformers for embedding similarity
- Federation: broadcasts search query to connected instances via ActivityPub

**Search config:**
```json
{
  "search": {
    "engine": "meilisearch",
    "meilisearch_url": "http://localhost:7700",
    "index_captions": true,
    "semantic_search": false,
    "semantic_model": "all-MiniLM-L6-v2",
    "federated_search": false,
    "max_results": 50
  }
}
```

---

### 10. AI FEATURES (api/ai.py + workers/)

**What it does:**
- Auto-generate subtitles/captions (Whisper)
- Auto-detect chapters from transcript
- Smart thumbnail selection
- AI tag & category suggestions
- Content summary generation
- Translation to other languages
- All processing is LOCAL — nothing sent to cloud by default
- Optional cloud API fallback (OpenAI, etc.)

**AI pipeline:**
```
VIDEO UPLOADED
  │
  ├──► WHISPER (local or API)
  │      │
  │      ├── Transcript (.txt)
  │      ├── Subtitles (.vtt with timestamps)
  │      └── Language detection
  │
  ├──► CHAPTER DETECTION
  │      │ Analyzes transcript for topic shifts
  │      │ Uses scene detection (ffmpeg) for visual breaks
  │      └── Outputs: [{time: "0:00", title: "Intro"}, ...]
  │
  ├──► TAG GENERATION
  │      │ Extracts key topics from transcript
  │      └── Suggests: ["esp32", "weather station", "arduino", "diy"]
  │
  ├──► THUMBNAIL AI
  │      │ Scores frames by: face presence, text clarity,
  │      │ color vibrancy, composition (rule of thirds)
  │      └── Presents top 3 candidates to creator
  │
  ├──► SUMMARY
  │      │ 2-3 sentence description from transcript
  │      └── Creator can accept, edit, or reject
  │
  └──► TRANSLATION (optional)
         │ Translates .vtt files to other languages
         └── Uses local model or DeepL/OpenAI API
```

**AI settings (admin):**
```
AI CONFIGURATION

  CAPTION GENERATION
  Engine: ● Whisper (local)  ○ OpenAI Whisper API  ○ Disabled
  Model:  [medium ▼]  (small=fast, large=accurate)
  Languages: [Auto-detect ▼]
  GPU acceleration: ☑ (CUDA detected)

  CHAPTER DETECTION
  ☑ Enabled
  Minimum chapter length: [30] seconds
  Method: ● Transcript analysis  ○ Scene detection  ○ Both

  TAG GENERATION
  ☑ Enabled
  Max tags: [10]

  THUMBNAIL SELECTION
  ☑ Enabled
  Candidates: [3]

  SUMMARY GENERATION
  ☑ Enabled
  Max length: [150] words

  TRANSLATION
  ☐ Enabled
  Target languages: [French, Arabic, Spanish]
  Engine: ○ Local (Argos Translate)  ○ DeepL API  ○ OpenAI

  COST ESTIMATE
  Using local Whisper (medium) + local tag generation:
  ~3 minutes processing per 10 minutes of video
  GPU memory: ~2 GB VRAM
  No API costs.
```

**Creator-facing AI UI (in upload wizard):**
```
AI FEATURES                              ⚙ Powered by local AI

  CAPTIONS                                        [✓ Auto-generated]
  Language detected: English (98% confidence)
  ┌──────────────────────────────────────────────┐
  │ 00:00:00 In this video I'll show you how to  │
  │ 00:00:03 build a weather station using an     │
  │ 00:00:05 ESP32 microcontroller and a BME280   │
  │ 00:00:08 sensor. Let's get started.           │
  │ ...                                           │
  └──────────────────────────────────────────────┘
  [Edit Captions]  [Download .vtt]  [+ Add language]

  CHAPTERS (auto-detected)                        [✓ Detected 6]
  ┌────────┬──────────────────────────────────────┐
  │ 0:00   │ Intro                                │ [edit]
  │ 0:45   │ Parts List                           │ [edit]
  │ 2:10   │ Wiring the BME280                    │ [edit]
  │ 5:30   │ Writing the Code                     │ [edit]
  │ 9:15   │ Testing & Calibration                │ [edit]
  │ 11:20  │ Final Results                        │ [edit]
  └────────┴──────────────────────────────────────┘
  [Accept All]  [Edit]  [Remove]

  SUGGESTED TAGS
  [esp32] [weather station] [bme280] [arduino] [diy] [tutorial]
  [sensor] [iot] [electronics]
  [Accept]  [Edit]

  SUMMARY
  "Learn how to build a complete weather station using an ESP32
   and BME280 sensor. Covers wiring, code, and calibration."
  [Use as Description]  [Edit]  [Discard]
```

---

### 11. FEDERATION — ACTIVITYPUB (api/federation.py)

**What it does:**
- Connect your Meridian instance to other Meridian instances
- Compatible with PeerTube, Mastodon, Lemmy (video links)
- Follow remote channels, see their videos in your feed
- Boost/share videos across instances
- Comment across instances
- Block instances (instance-level and user-level)
- Optional: fully disable federation for private instances

**How it works:**
```
INSTANCE A (your-videos.com)          INSTANCE B (cool-tubes.org)
┌────────────────────┐               ┌────────────────────┐
│                    │               │                    │
│  @creator uploads  │               │  @viewer follows   │
│  a new video       │───────────────▶  @creator@your-    │
│                    │  ActivityPub   │  videos.com        │
│                    │  "Create"      │                    │
│                    │  activity      │  Video appears in  │
│                    │               │  @viewer's feed    │
│                    │               │                    │
│  Receives comment  │◀───────────────│  @viewer comments  │
│  from @viewer@     │  ActivityPub   │  on the video      │
│  cool-tubes.org    │  "Create Note" │                    │
└────────────────────┘               └────────────────────┘
```

**Federation admin panel:**
```
FEDERATION

  STATUS: ● Active (12 connected instances)

  CONNECTED INSTANCES
  ┌──────────────────────┬──────────┬───────────┬─────────┐
  │ Instance             │ Software │ Users     │ Status  │
  ├──────────────────────┼──────────┼───────────┼─────────┤
  │ cool-tubes.org       │ Meridian │ 2,340     │ 🟢 OK   │
  │ vid.hackerspace.net  │ Meridian │ 89        │ 🟢 OK   │
  │ peertube.social      │ PeerTube │ 12,000    │ 🟢 OK   │
  │ videos.example.com   │ Meridian │ 450       │ 🔴 Down │
  └──────────────────────┴──────────┴───────────┴─────────┘

  BLOCK LIST
  ┌──────────────────────┬──────────────────────────────────┐
  │ spam-videos.xyz      │ Reason: spam, adult content      │
  │ bad-instance.com     │ Reason: harassment               │
  └──────────────────────┴──────────────────────────────────┘
  [+ Block instance]

  SETTINGS
  ☑ Auto-accept follow requests from known instances
  ☐ Auto-accept from all instances (open federation)
  ☑ Require approval for new instances
  ☑ Federate public videos
  ☐ Federate unlisted videos
  Max remote video cache: [10 GB]
```

---

### 12. MONETIZATION (api/monetization.py)

**What it does:**
- Viewer tips (one-time payments)
- Channel memberships (monthly recurring)
- Super chats (highlighted messages in live streams)
- Pay-per-view for premium content
- Instance takes configurable cut (default 0%, set by admin)
- Payment processors: Stripe, Ko-fi, Liberapay, Bitcoin/Lightning
- No algorithmic incentive — monetization never affects feed ranking

**Creator monetization dashboard:**
```
MONETIZATION

  OVERVIEW (last 30 days)
  ┌────────────┐ ┌────────────┐ ┌────────────┐ ┌────────────┐
  │  Tips      │ │ Members    │ │ Super Chat │ │  Total     │
  │  €23.50    │ │ €18.00     │ │ €5.70      │ │  €47.20    │
  │  14 tips   │ │ 6 active   │ │ 3 messages │ │  ↑ 23%     │
  └────────────┘ └────────────┘ └────────────┘ └────────────┘

  MEMBERSHIP TIERS
  ┌───────────────┬────────┬─────────┬──────────────────────────┐
  │ Tier          │ Price  │ Members │ Perks                    │
  ├───────────────┼────────┼─────────┼──────────────────────────┤
  │ Supporter     │ €2/mo  │ 4       │ Badge, early access      │
  │ Builder       │ €5/mo  │ 2       │ + Behind-the-scenes      │
  │ Partner       │ €15/mo │ 0       │ + Monthly video call     │
  └───────────────┴────────┴─────────┴──────────────────────────┘
  [Edit Tiers]

  PAYOUT
  Method: Stripe → IBAN ****4832
  Next payout: €47.20 on March 1, 2026
  Instance fee: 0% (set by admin)
  [View transaction history]
```

---

### 13. SHORTS — VERTICAL VIDEO (pages/Shorts.jsx)

**What it does:**
- Vertical videos under 60 seconds
- Full-screen swipe-to-next interface (mobile-first)
- Separate upload flow optimized for short content
- Like, comment, share overlay on video
- Sound/music attribution
- Desktop: vertical player with side comments

**Mobile UI:**
```
+------------------+
|                  |
|                  |
|    ┌──────────┐  |
|    │          │  |
|    │  VIDEO   │  |
|    │  FULL    │  |
|    │  SCREEN  │  |
|    │          │  |
|    │          │  |
|    │    ❤ 2.3K│  |
|    │    💬 89 │  |
|    │    ↗ Share│  |
|    │          │  |
|    └──────────┘  |
|                  |
| @creator · desc  |
| ♪ Original Sound |
|                  |
| ↑ swipe for next |
+------------------+
```

**Desktop UI:**
```
+------------------------------------------------------------------+
|           ┌─────────────┐                                         |
|           │             │   @creator                              |
|           │   VERTICAL  │   "Quick tip: add a capacitor..."       |
|           │   VIDEO     │                                         |
|           │   PLAYER    │   ❤ 2.3K  💬 89  ↗ Share               |
|           │             │                                         |
|           │             │   COMMENTS                              |
|           │     ❤ 💬 ↗  │   "That's so smart!" — @viewer1        |
|           │             │   "What capacitor?" — @viewer2          |
|           └─────────────┘   [Load more...]                       |
|                                                                    |
|       [← Previous]                      [Next →]                  |
+------------------------------------------------------------------+
```

---

### 14. PLAYLISTS & WATCH QUEUE (api/playlists.py)

**What it does:**
- Create, edit, reorder, share playlists
- Collaborative playlists (invite others to add videos)
- Auto-playlist: "Watch Later", "Liked Videos", "History"
- Smart playlists: auto-populate by tag, channel, or keyword
- Series ordering: auto-detect numbered episodes, order correctly
- Import playlists from YouTube (via export/URL)

**Playlist UI:**
```
PLAYLIST: "ESP32 Tutorial Series"
by @creator · 12 videos · 3h 42m total · Last updated 2 days ago
[▶ Play All]  [🔀 Shuffle]  [📤 Share]  [✏ Edit]

  ┌──┬──────┬───────────────────────────────────┬────────┬───────┐
  │# │ Thumb│ Title                             │Duration│ Views │
  ├──┼──────┼───────────────────────────────────┼────────┼───────┤
  │1 │ ░░░░ │ ESP32 Getting Started             │ 15:42  │ 45.2K │
  │2 │ ░░░░ │ ESP32 WiFi & HTTP                 │ 22:10  │ 32.1K │
  │3 │ ░░░░ │ ESP32 BLE Tutorial                │ 18:45  │  8.2K │
  │4 │ ░░░░ │ ESP32 Deep Sleep                  │ 12:33  │  5.6K │
  │  │      │ ...                               │        │       │
  └──┴──────┴───────────────────────────────────┴────────┴───────┘

  SMART PLAYLIST RULES (auto-updates)
  Channel: @creator  AND  Tag: "esp32"  AND  Sort: date ↑
```

---

### 15. OFFLINE MODE & PWA (frontend/public/sw.js)

**What it does:**
- Progressive Web App: installable on desktop & mobile
- Download videos for offline viewing (with creator permission)
- Background sync: upload comments/likes when back online
- Cached feed: browse subscriptions offline
- Storage management: auto-cleanup, quality selection for downloads

**Download UI:**
```
DOWNLOAD VIDEO

  "How to Build a Weather Station"

  Quality:
  ○ 1080p  (450 MB)
  ● 720p   (210 MB)
  ○ 480p   (95 MB)
  ○ Audio only (28 MB)

  ☑ Include subtitles (English)

  [Download]

  ── MY DOWNLOADS ────────────────────

  ┌──────┬───────────────────────────┬───────┬────────┐
  │ Thumb│ Title                     │ Size  │        │
  ├──────┼───────────────────────────┼───────┼────────┤
  │ ░░░░ │ Weather Station Build     │ 210MB │ [✗ rm] │
  │ ░░░░ │ ESP32 Getting Started     │ 180MB │ [✗ rm] │
  │ ░░░░ │ Podcast Ep 47             │  28MB │ [✗ rm] │
  └──────┴───────────────────────────┴───────┴────────┘
  Total: 418 MB used
```

---

### 16. ADMIN PANEL (pages/Admin.jsx)

**What it does:**
- Full instance management from the browser
- Server health monitoring (CPU, RAM, disk, bandwidth)
- User management (roles, bans, quotas, invites)
- Content moderation queue
- Storage management and cleanup
- Transcoding queue and settings
- Federation management
- Security settings (rate limits, API keys)
- Instance customization (name, logo, theme, landing page)
- Audit log of all admin actions

**Admin dashboard:**
```
INSTANCE ADMIN                          meridian.example.com

  SERVER HEALTH
  ┌────────────┐ ┌────────────┐ ┌────────────┐ ┌────────────┐
  │  CPU       │ │  Memory    │ │  Storage   │ │  Bandwidth │
  │  23%       │ │  4.2/8 GB  │ │  120/500GB │ │  2.1 TB/mo │
  │  ████░░░░  │ │  █████░░░  │ │  ███░░░░░  │ │  ██░░░░░░  │
  └────────────┘ └────────────┘ └────────────┘ └────────────┘

  USERS                          CONTENT
  Total: 1,247                   Videos: 3,891
  Active (30d): 342              Total storage: 118 GB
  New today: 8                   Transcoding queue: 2
  Banned: 12                     Flagged: 5

  RECENT ADMIN ACTIONS
  • @admin banned @spammer (reason: spam) — 2h ago
  • @admin approved federation: vid.hackerspace.net — 1d ago
  • @admin updated instance logo — 3d ago

  QUICK ACTIONS
  [👥 Manage Users]  [🚩 Review Flagged]  [💾 Storage]
  [🔧 Transcoding]   [🌐 Federation]     [📜 Audit Log]
```

**User management:**
```
USER MANAGEMENT

  Search: [____________]  Role: [All ▼]  Sort: [Joined ▼]

  ┌────────────┬──────────┬─────────┬──────────┬──────────┬───────┐
  │ User       │ Email    │ Role    │ Videos   │ Joined   │       │
  ├────────────┼──────────┼─────────┼──────────┼──────────┼───────┤
  │ @creator   │ c@e.com  │ Creator │ 47       │ Jan 2026 │ [···] │
  │ @viewer1   │ v@e.com  │ User    │ 0        │ Feb 2026 │ [···] │
  │ @mod_ali   │ m@e.com  │ Mod     │ 2        │ Jan 2026 │ [···] │
  └────────────┴──────────┴─────────┴──────────┴──────────┴───────┘

  ROLES
  • Viewer:  Watch, like, comment, subscribe
  • Creator: + Upload, live stream, monetize
  • Mod:     + Delete comments, flag content, mute users
  • Admin:   + Full instance control

  REGISTRATION
  ○ Open (anyone can register)
  ● Invite-only (admin sends invite codes)
  ○ Closed (no new registrations)

  QUOTAS (per user)
  Upload limit: [50 GB] per user
  Video size limit: [10 GB] per video
  Daily upload limit: [5] videos
```

---

### 17. YOUTUBE IMPORT WIZARD (scripts/migrate.sh)

**What it does:**
- Import full YouTube channel via Google Takeout
- Parses takeout archive: videos, metadata, playlists, thumbnails
- Re-uploads everything to your Meridian instance
- Preserves: titles, descriptions, tags, upload dates, thumbnails
- Maps YouTube playlists to Meridian playlists
- Optional: set all imported videos as unlisted first for review
- CLI tool + web UI wizard

**Import flow:**
```
YOUTUBE IMPORT WIZARD

  Step 1 of 4: Upload Takeout Archive

  Download your data from https://takeout.google.com
  Select "YouTube and YouTube Music" → Download

  ┌────────────────────────────────────────────────┐
  │                                                │
  │    ⬆  Drop your takeout-*.zip here             │
  │       or click to browse                       │
  │                                                │
  └────────────────────────────────────────────────┘

  ── Step 2: Review ─────────────────────────

  Found in archive:
    📹 47 videos (23.4 GB)
    📋 12 playlists
    📝 All metadata (titles, descriptions, tags)
    🖼  42 custom thumbnails

  ── Step 3: Settings ───────────────────────

  Import as:
  ● Unlisted (review before publishing)
  ○ Public (publish immediately)
  ○ Private (only you can see)

  ☑ Preserve original upload dates
  ☑ Import playlists
  ☑ Import thumbnails
  ☑ Re-generate captions (AI)

  ── Step 4: Import ─────────────────────────

  Progress:
    Videos:    ████████████░░░░░░░░ 24/47  (51%)
    Playlists: ████████████████████ 12/12  (done)
    Current:   "ESP32 WiFi Tutorial" — transcoding...

  ETA: ~45 minutes

  [Pause]  [Cancel]
```

---

### 18. WATCH PARTY (pages/Theater.jsx)

**What it does:**
- Synchronized video watching with friends
- Real-time chat alongside video
- Host controls play/pause/seek for all viewers
- No account required for guests (shareable link)
- Reactions overlay (emoji burst)
- Works across instances (federation)

**Watch party UI:**
```
+------------------------------------------------------------------+
| WATCH PARTY: "Friday Movie Night"        🔗 Share Link  👥 7     |
+------------------------------------------------------------------+
|                                              |                    |
|                                              | CHAT               |
|              VIDEO PLAYER                    |                    |
|         (synced for all viewers)             | @ali: this part!   |
|                                              | @sara: 😂😂        |
|  ┌────────────────────────────────────────┐  | @omar: wait what?  |
|  │ ▓▓▓▓▓▓▓▓▓░░░░░░░░░░░░░░░░░░░░░░░░░░ │  |                    |
|  └────────────────────────────────────────┘  |                    |
|  ▶  12:34 / 45:02   🔊━━━━━                 | [msg]  [Send]      |
|                                              |                    |
|  QUEUE                                       +────────────────────+
|  1. ▶ Currently: "Planet Earth Ep. 3"
|  2. "Planet Earth Ep. 4" (added by @ali)
|  3. "Funny Cat Compilation" (added by @sara)
|
|  VIEWERS
|  👑 @host (you)  @ali  @sara  @omar  @guest1  @guest2  @guest3
+------------------------------------------------------------------+
```

**Implementation:**
- WebSocket room: all participants join a room
- Host sends play/pause/seek events, all clients sync
- Heartbeat sync: every 5 seconds, server broadcasts current timestamp
- If a viewer drifts >2 seconds, auto-correct their position
- Chat: same WebSocket connection, separate message type
- Guest access: shareable link with token, no login needed

---

### 19. ACCESSIBILITY (built into all components)

**What it does:**
- WCAG 2.1 AA compliant across entire platform
- Full keyboard navigation (every feature usable without mouse)
- Screen reader support (ARIA labels, live regions, announcements)
- High contrast mode
- Reduced motion mode
- Customizable caption styling (font, size, color, background)
- Audio descriptions track support
- Focus indicators visible on all interactive elements

**Caption styling:**
```
CAPTION SETTINGS

  Font:     [Sans-serif ▼]
  Size:     [──●──────] 120%
  Color:    [White ▼]  ■ ■ ■ ■ ■ (presets)
  Background: [Black 75% ▼]
  Position: ○ Bottom  ○ Top  ○ Custom drag

  PREVIEW:
  ┌──────────────────────────────────────┐
  │                                      │
  │                                      │
  │  ┌──────────────────────────────┐    │
  │  │ This is how your captions    │    │
  │  │ will look during playback.   │    │
  │  └──────────────────────────────┘    │
  └──────────────────────────────────────┘
```

**Keyboard navigation map:**
```
GLOBAL SHORTCUTS
  /           Focus search bar
  G then H    Go to Home
  G then S    Go to Subscriptions
  G then L    Go to Library
  G then U    Go to Upload
  G then D    Go to Studio Dashboard
  Escape      Close modal / exit fullscreen
  ?           Show keyboard shortcut overlay

  (Player shortcuts listed in Feature #1)
```

---

### 20. INSTANCE CUSTOMIZATION (admin/Customization)

**What it does:**
- Custom instance name, logo, favicon
- Custom landing page for logged-out visitors
- Custom color theme (or pick from presets)
- Custom CSS injection (advanced)
- Terms of service / community guidelines editor
- Custom footer links
- Instance-wide announcements (banner)

**Landing page builder:**
```
CUSTOMIZE LANDING PAGE

  HERO SECTION
  Title:    [Welcome to Our Community Videos        ]
  Subtitle: [A space for learning, sharing, and creating]
  CTA:      [Join Now ▼]  Links to: [Registration ▼]
  Background: [Upload image] or [Gradient ▼]

  FEATURED SECTION
  ☑ Show trending videos
  ☑ Show latest uploads
  ☐ Show curated picks (admin-selected)
  ☐ Show instance stats (users, videos, hours)

  FOOTER
  Links: [About] [Terms] [Contact] [Source Code]
  Text:  [Powered by Meridian · Self-hosted with ❤]

  [Preview]  [Save]
```

---

### 21. EMBED SYSTEM (api/embed.py)

**What it does:**
- Embed any public video on external websites
- oEmbed endpoint for automatic embeds (WordPress, Ghost, etc.)
- Customizable embed player (autoplay, start time, captions)
- Embed analytics: track where videos are embedded
- Privacy-respecting: no cookies on embed, no tracking pixels

**Embed code:**
```html
<!-- Standard iframe -->
<iframe
  src="https://your-instance.com/embed/VIDEO_ID"
  width="560" height="315"
  frameborder="0"
  allow="autoplay; fullscreen; picture-in-picture"
  allowfullscreen>
</iframe>

<!-- With options -->
<iframe
  src="https://your-instance.com/embed/VIDEO_ID?start=120&cc=en&autoplay=0"
  ...>
</iframe>
```

**oEmbed endpoint:**
```
GET /api/oembed?url=https://instance.com/watch/VIDEO_ID

Response:
{
  "type": "video",
  "version": "1.0",
  "title": "How to Build a Weather Station",
  "author_name": "@creator",
  "author_url": "https://instance.com/@creator",
  "provider_name": "Meridian",
  "provider_url": "https://instance.com",
  "thumbnail_url": "https://instance.com/thumbs/VIDEO_ID.jpg",
  "html": "<iframe src=\"...\"></iframe>",
  "width": 560,
  "height": 315
}
```

---

### 22. NOTIFICATIONS SYSTEM (api/notifications.py)

**What it does:**
- In-app notification bell with unread count
- Push notifications via Web Push API (browser)
- Email digests (configurable: instant, daily, weekly, off)
- Per-channel notification preferences (all, highlights, none)
- Notification types: new video, reply, mention, live start, milestone

**Notification UI:**
```
🔔 (3)

  ┌─────────────────────────────────────────────────────────┐
  │ NEW                                                     │
  │                                                         │
  │ 📹 @maker_pro uploaded "ESP32 LoRa Gateway"  · 2h ago  │
  │ 💬 @viewer1 replied to your comment           · 3h ago  │
  │ 🔴 @cooking_chan is live: "Sunday Baking"     · 5h ago  │
  │                                                         │
  │ EARLIER                                                 │
  │ 🎉 Your video hit 10,000 views!               · 1d ago │
  │ 📹 @electronics_guy uploaded "PCB Design"     · 2d ago  │
  │                                                         │
  │ [Mark all read]  [Notification settings]                │
  └─────────────────────────────────────────────────────────┘
```

**Notification preferences:**
```
NOTIFICATION SETTINGS

  GLOBAL
  ☑ In-app notifications
  ☑ Browser push notifications
  ○ Email: instant  ● Email: daily digest  ○ Email: weekly  ○ Off

  PER CHANNEL
  ┌──────────────────────┬──────────────────────────────┐
  │ @maker_pro           │ ● All  ○ Highlights  ○ None │
  │ @cooking_chan         │ ○ All  ● Highlights  ○ None │
  │ @electronics_guy     │ ○ All  ○ Highlights  ● None │
  └──────────────────────┴──────────────────────────────┘

  TYPES
  ☑ New uploads from subscriptions
  ☑ Replies to my comments
  ☑ Mentions (@username)
  ☑ Live stream starts
  ☑ Milestones (views, subs)
  ☐ Weekly recommendations
```

---

### 23. CONTENT MODERATION (services/abuse.py)

**What it does:**
- User reporting system (flag videos, comments, channels)
- Admin moderation queue with bulk actions
- Auto-mod rules: keyword blocklist, link filter, caps filter
- Optional AI moderation (NSFW detection, spam detection)
- Strike system: warnings → temp ban → permanent ban
- Appeal process for creators
- Audit log of all moderation decisions

**Moderation queue:**
```
MODERATION QUEUE                           5 items pending

  ┌──┬──────────────────────┬──────────────┬────────┬───────────────┐
  │☐ │ Reported Item        │ Reason       │ Reports│ Action        │
  ├──┼──────────────────────┼──────────────┼────────┼───────────────┤
  │☐ │ 📹 "Clickbait..."    │ Misleading   │ 3      │ [✓] [✗] [⚠] │
  │☐ │ 💬 "Buy crypto at.." │ Spam         │ 7      │ [✓] [✗] [⚠] │
  │☐ │ 📹 "Dangerous DIY"   │ Dangerous    │ 2      │ [✓] [✗] [⚠] │
  │☐ │ 💬 "You're a..."     │ Harassment   │ 4      │ [✓] [✗] [⚠] │
  │☐ │ 👤 @spam_account     │ Bot/spam     │ 12     │ [✓] [✗] [⚠] │
  └──┴──────────────────────┴──────────────┴────────┴───────────────┘

  Actions: [✓ Approve] [✗ Remove] [⚠ Strike + Remove]

  STRIKE LEVELS
  1 strike  → Warning email
  2 strikes → 7-day upload ban
  3 strikes → Permanent ban (appealable)
```

---

### 24. MULTI-LANGUAGE & i18n

**What it does:**
- Full interface translation (initially EN, FR, AR, ES, DE, ZH, JA)
- RTL layout for Arabic/Hebrew
- Localized date/number formatting
- Community-contributed translations via JSON files
- Per-user language preference
- Auto-detect from browser language

**Translation file structure:**
```
frontend/src/i18n/
├── en.json          # English (default)
├── fr.json          # French
├── ar.json          # Arabic (RTL)
├── es.json          # Spanish
├── de.json          # German
├── zh.json          # Chinese (Simplified)
└── ja.json          # Japanese
```

**Sample translation:**
```json
// en.json
{
  "nav.home": "Home",
  "nav.trending": "Trending",
  "nav.subscriptions": "Subscriptions",
  "nav.library": "Library",
  "nav.upload": "Upload",
  "player.play": "Play",
  "player.pause": "Pause",
  "player.next": "Next",
  "player.captions": "Captions",
  "video.views": "{count} views",
  "video.ago": "{time} ago",
  "comments.add": "Add a comment...",
  "comments.reply": "Reply",
  "channel.subscribe": "Subscribe",
  "channel.subscribers": "{count} subscribers",
  "studio.upload": "Upload video",
  "studio.analytics": "Analytics",
  "admin.users": "Users",
  "admin.moderation": "Moderation"
}

// ar.json
{
  "nav.home": "الرئيسية",
  "nav.trending": "الأكثر رواجاً",
  "nav.subscriptions": "الاشتراكات",
  "nav.library": "المكتبة",
  "nav.upload": "رفع",
  "player.play": "تشغيل",
  "player.pause": "إيقاف مؤقت",
  "video.views": "{count} مشاهدة",
  "channel.subscribe": "اشتراك",
  "channel.subscribers": "{count} مشترك"
}
```

---

## Implementation Priority

| Phase | Features                                              | Complexity | Timeline  |
|-------|-------------------------------------------------------|------------|-----------|
| 1     | Video player, upload, transcoding, basic feed         | High       | 6-8 weeks |
| 2     | User auth, channels, subscriptions, profiles          | Medium     | 3-4 weeks |
| 3     | Comments, likes, playlists, watch history             | Medium     | 3-4 weeks |
| 4     | Search engine (Meilisearch + FTS5)                    | Medium     | 2-3 weeks |
| 5     | Creator Studio (dashboard, analytics, content mgmt)   | Medium     | 4-5 weeks |
| 6     | AI pipeline (Whisper captions, chapters, tags)        | Medium     | 3-4 weeks |
| 7     | Admin panel (users, moderation, storage, settings)    | Medium     | 4-5 weeks |
| 8     | Live streaming (RTMP ingest, chat, VOD recording)     | High       | 5-6 weeks |
| 9     | Shorts (vertical video player, mobile UI)             | Medium     | 3-4 weeks |
| 10    | Federation (ActivityPub, cross-instance)              | High       | 5-6 weeks |
| 11    | Monetization (tips, memberships, super chat)          | Medium     | 4-5 weeks |
| 12    | PWA + offline mode                                    | Medium     | 3-4 weeks |
| 13    | YouTube import wizard                                 | Medium     | 2-3 weeks |
| 14    | Watch party (synced viewing, chat)                    | Medium     | 3-4 weeks |
| 15    | Accessibility audit + polish                          | Medium     | 2-3 weeks |
| 16    | Instance customization (landing page, themes)         | Low-Med    | 2-3 weeks |
| 17    | Embed system + oEmbed + notification system           | Medium     | 3-4 weeks |
| 18    | Content moderation + strike system + audit log        | Medium     | 3-4 weeks |
| 19    | Multi-language i18n (EN, FR, AR, ES, DE, ZH, JA)     | Medium     | 2-3 weeks |
| 20    | Mobile app (React Native)                             | High       | 8-10 weeks|
| 21    | Smart TV app (LG webOS, Samsung Tizen, Roku)          | High       | 6-8 weeks |

---

## Tech Stack

**Backend:**
```
FastAPI           — async Python web framework
SQLAlchemy        — ORM (PostgreSQL production, SQLite dev)
Alembic           — database migrations
Celery + Redis    — background task queue (transcoding, AI)
FFmpeg            — video transcoding & thumbnail extraction
hls.js            — adaptive bitrate streaming
Meilisearch       — full-text search (optional, FTS5 fallback)
Whisper           — local speech-to-text (optional)
nginx             — reverse proxy, HLS serving, RTMP ingest
```

**Frontend:**
```
React 18          — UI framework
Zustand           — lightweight state management
React Router      — client-side routing
Tailwind CSS      — utility-first styling
hls.js            — HLS playback in browser
Workbox           — PWA service worker
```

**Infrastructure:**
```
Docker + Docker Compose    — one-command deployment
PostgreSQL                 — primary database (SQLite for dev/small)
Redis                      — cache + task queue + real-time pubsub
MinIO (optional)           — S3-compatible object storage
nginx-rtmp                 — live stream ingest
Let's Encrypt              — free SSL via certbot
```

---

## Dependencies

**Required:**
```
fastapi, uvicorn, sqlalchemy, alembic, python-multipart
celery, redis, httpx, python-jose[cryptography], passlib
```

**Required (system):**
```
ffmpeg, ffprobe, nginx
postgresql (or sqlite3 for small instances)
redis-server
```

**Optional:**
```
openai-whisper         — local caption generation
sentence-transformers  — semantic search
meilisearch            — advanced full-text search
stripe                 — payment processing
argos-translate        — local translation
```

**Frontend CDN (loaded in browser):**
```
hls.js                 — adaptive video playback
Chart.js               — analytics charts
```

---

## Deployment Recipes

**One-liner (Docker):**
```bash
curl -sSL https://get.meridian.video | bash
# or:
git clone https://github.com/user/meridian && cd meridian
cp .env.example .env    # edit: domain, email, secrets
docker compose up -d
# Done. Visit https://your-domain.com
```

**docker-compose.yml (simplified):**
```yaml
services:
  app:
    build: .
    ports: ["8000:8000"]
    env_file: .env
    volumes:
      - ./storage:/app/storage
    depends_on: [db, redis]

  db:
    image: postgres:16-alpine
    volumes: [pg_data:/var/lib/postgresql/data]
    environment:
      POSTGRES_DB: meridian
      POSTGRES_PASSWORD: ${DB_PASSWORD}

  redis:
    image: redis:7-alpine

  worker:
    build: .
    command: celery -A backend.workers worker -l info
    volumes: [./storage:/app/storage]
    depends_on: [db, redis]

  nginx:
    image: nginx:alpine
    ports: ["80:80", "443:443", "1935:1935"]
    volumes:
      - ./nginx/nginx.conf:/etc/nginx/nginx.conf
      - ./storage:/storage:ro
      - certs:/etc/letsencrypt

  search:
    image: getmeili/meilisearch:latest
    volumes: [meili_data:/meili_data]

volumes:
  pg_data:
  meili_data:
  certs:
```

**Raspberry Pi deployment:**
```bash
# Meridian runs on a Pi 4 (4GB+ recommended)
# Transcoding will be slower (software only, no GPU)
# Recommended: transcode to 720p max on Pi

curl -sSL https://get.meridian.video | bash -s -- --lite
# --lite: SQLite, no Meilisearch, 720p max, no AI
```

**Environment variables (.env):**
```bash
# Required
DOMAIN=your-domain.com
SECRET_KEY=change-me-to-random-string
DB_URL=postgresql://meridian:password@db/meridian
REDIS_URL=redis://redis:6379

# Storage (local by default)
STORAGE_BACKEND=local                    # local, s3, b2
# STORAGE_S3_BUCKET=my-bucket
# STORAGE_S3_ENDPOINT=https://s3.amazonaws.com

# Transcoding
TRANSCODE_THREADS=2
TRANSCODE_MAX_RESOLUTION=2160           # 720, 1080, 1440, 2160
TRANSCODE_HARDWARE=auto                  # auto, nvenc, vaapi, none

# AI (optional)
AI_CAPTIONS=true
AI_WHISPER_MODEL=medium                  # tiny, base, small, medium, large
# AI_OPENAI_KEY=sk-...                   # for cloud Whisper API

# Federation (optional)
FEDERATION_ENABLED=false

# Email (optional)
# SMTP_HOST=smtp.example.com
# SMTP_USER=noreply@example.com
# SMTP_PASS=password

# Registration
REGISTRATION_MODE=invite                 # open, invite, closed
```

---

## Database Schema (key tables)

```sql
-- Users & auth
users (id, username, email, password_hash, display_name, avatar_url,
       role, bio, created_at, last_seen, is_banned, storage_used)

-- Channels (1 user = 1+ channels)
channels (id, user_id, name, handle, description, avatar_url,
          banner_url, subscriber_count, video_count, created_at)

-- Videos
videos (id, channel_id, title, description, slug, duration,
        original_file, status, visibility, category,
        view_count, like_count, dislike_count, comment_count,
        published_at, scheduled_at, created_at, updated_at,
        language, license, allow_download, thumbnail_url)

-- Video renditions (one per quality level)
renditions (id, video_id, resolution, bitrate, codec,
            file_path, file_size, hls_playlist)

-- Captions
captions (id, video_id, language, label, vtt_path,
          is_auto_generated, is_default)

-- Chapters
chapters (id, video_id, start_seconds, title, thumbnail_url)

-- Comments (self-referencing for threads)
comments (id, video_id, user_id, parent_id, body,
          like_count, is_pinned, is_hearted, created_at, edited_at)

-- Subscriptions
subscriptions (user_id, channel_id, notify, created_at)

-- Playlists
playlists (id, user_id, title, description, visibility,
           video_count, is_smart, smart_rules, created_at)
playlist_items (playlist_id, video_id, position, added_at)

-- Watch history
watch_history (user_id, video_id, progress_seconds,
               completed, watched_at)

-- Likes
likes (user_id, video_id, value, created_at)    -- value: 1 or -1

-- Notifications
notifications (id, user_id, type, data, is_read, created_at)

-- Live streams
live_streams (id, channel_id, title, description, stream_key,
              status, started_at, ended_at, viewer_peak, vod_video_id)

-- Federation
federation_instances (id, domain, software, status, last_seen)
federation_follows (local_user_id, remote_actor_uri)
federation_inbox (id, activity_json, processed, created_at)

-- Monetization
tips (id, from_user_id, to_channel_id, amount, currency, created_at)
memberships (id, user_id, channel_id, tier_id, status, started_at)
membership_tiers (id, channel_id, name, price, currency, perks)

-- Moderation
reports (id, reporter_id, video_id, comment_id, reason, status, created_at)
audit_log (id, admin_id, action, target, details, created_at)
```

---

## API Design (key endpoints)

```
AUTH
  POST   /api/auth/register          Register new account
  POST   /api/auth/login             Login (returns JWT)
  POST   /api/auth/refresh           Refresh token
  POST   /api/auth/logout            Invalidate token
  GET    /api/auth/me                Current user profile

VIDEOS
  GET    /api/videos                 List/search videos
  POST   /api/videos                 Create video (initiate upload)
  GET    /api/videos/{id}            Video details + player data
  PATCH  /api/videos/{id}            Update metadata
  DELETE /api/videos/{id}            Delete video + files
  GET    /api/videos/{id}/stream     HLS manifest URL
  POST   /api/videos/{id}/view       Record view (anonymous)
  POST   /api/videos/{id}/like       Like/dislike
  GET    /api/videos/{id}/comments   Get comments (paginated)
  POST   /api/videos/{id}/comments   Post comment
  GET    /api/videos/{id}/captions   List caption tracks
  GET    /api/videos/{id}/chapters   Get chapters
  GET    /api/videos/{id}/analytics  Creator analytics

UPLOAD
  POST   /api/upload/init            Initiate resumable upload
  PATCH  /api/upload/{id}            Upload chunk (tus protocol)
  GET    /api/upload/{id}/status     Transcoding status (SSE)

CHANNELS
  GET    /api/channels/{handle}      Channel profile + stats
  GET    /api/channels/{handle}/videos  Channel videos
  POST   /api/channels/{handle}/subscribe   Subscribe
  DELETE /api/channels/{handle}/subscribe   Unsubscribe

FEED
  GET    /api/feed/subscriptions     Subscription feed (chrono)
  GET    /api/feed/trending          Trending videos
  GET    /api/feed/explore           Explore / categories

SEARCH
  GET    /api/search?q=...           Full-text search
  GET    /api/search/captions?q=...  Search within captions

PLAYLISTS
  GET    /api/playlists              User's playlists
  POST   /api/playlists              Create playlist
  PATCH  /api/playlists/{id}         Update playlist
  POST   /api/playlists/{id}/items   Add video to playlist

LIVE
  POST   /api/live/start             Start live stream
  GET    /api/live/{id}              Live stream info
  GET    /api/live/{id}/chat         Chat messages (WebSocket)
  POST   /api/live/{id}/end          End stream

ADMIN
  GET    /api/admin/stats            Instance statistics
  GET    /api/admin/users            User list
  PATCH  /api/admin/users/{id}       Update user (role, ban)
  GET    /api/admin/reports          Moderation queue
  GET    /api/admin/audit            Audit log

FEDERATION (ActivityPub)
  POST   /inbox                      Receive activities
  GET    /users/{handle}             Actor profile (AP format)
  GET    /users/{handle}/outbox      Published activities
  GET    /.well-known/webfinger      WebFinger discovery
  GET    /.well-known/nodeinfo       NodeInfo for instance

EMBED
  GET    /api/oembed?url=...         oEmbed discovery
  GET    /embed/{video_id}           Embeddable player page

EXPORT
  POST   /api/export/request         GDPR data export request
  GET    /api/export/{id}/download   Download export archive
```

---

## Notes

- Single `docker compose up` to run everything — zero manual config for defaults
- SQLite mode for tiny instances (< 100 users, no Redis needed)
- All AI features optional — instance works perfectly without any AI
- Federation optional — private instances work entirely standalone
- Monetization optional — many instances will run ad-free, tip-free
- Every feature degrades gracefully: no Meilisearch → SQLite FTS; no GPU → software transcode; no Whisper → no captions; no Redis → in-process queue
- Mobile-first responsive design across entire platform
- GDPR-compliant: full data export, account deletion, no tracking
- AGPLv3 license: any modifications to a hosted instance must be shared
- Embed support: any video embeddable via iframe or oEmbed
- RSS feeds: every channel has an RSS feed for podcast apps
- API-first: every feature accessible via REST API
- WebSocket for real-time: live chat, watch party sync, notification push, transcoding progress
- Rate limiting on all endpoints (configurable per role)
- Automated backups: `scripts/backup.sh` dumps DB + storage to tar.gz
- Health check endpoint: `GET /api/health` for monitoring
- Runs on Raspberry Pi 4 with --lite flag (SQLite, 720p max, no AI)
- No vendor lock-in: switch storage backends, search engines, AI models anytime
- Community translations: add a language by contributing a single JSON file
- Plugin system (Phase 22+): third-party extensions for custom features
