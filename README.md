# EZ Cam

A Camtasia-style screen recorder and video editor for Windows, macOS and Linux, built with Electron.

Record your screen, a window, or a region, with microphone narration, system audio (Windows) and a webcam. Then edit it on a multi-track timeline with callouts, arrows, blur, spotlight, zoom-and-pan, keyframe animation and cursor highlighting, clean up the voice, **edit the video by editing its transcript**, add captions, and export to MP4, WebM or GIF.

---

## Quick start

Requirements: **Node.js 18+** (20 LTS or newer recommended).

```bash
cd framecast-studio
npm install        # installs Electron, FFmpeg and the ASIO component
npm run setup      # finishes downloads that newer npm versions skip; safe to re-run
npm start          # launches the app
```

`npm run setup` prints an OK/FAIL line for Electron, FFmpeg and ASIO (Windows). Only Electron is required. Without FFmpeg, export is WebM-only. Without the ASIO component, ASIO inputs aren't listed.

### Build a Windows installer

```bash
npm run dist:win
```

This produces **`dist\EZ Cam Setup 0.4.0.exe`**, a normal installer with a choice of install folder, Start-menu and desktop shortcuts, and an uninstaller. It reuses the Electron already in `node_modules\electron\dist`, so Electron isn't downloaded again. FFmpeg and the ASIO component are bundled. Nothing native is rebuilt, so no C++ build tools are needed. The first build downloads electron-builder's small helper tools (NSIS and signing tools) from GitHub.

- The installer is **unsigned**, so Windows SmartScreen shows "Windows protected your PC" the first time: click **More info → Run anyway**. Removing that warning requires a code-signing certificate.
- If the build stops with **"Cannot create symbolic link: A required privilege is not held by the client"**, run the command once from a Command Prompt opened with **Run as administrator** (or turn on Windows **Developer Mode**). After that, normal builds work.
- To change the app's name later, edit `productName` / `shortcutName` in `package.json` and `APP_NAME` in `src/main/main.js`.

Other platforms: `npm run dist:mac` / `npm run dist:linux` (build on that OS).

> FFmpeg comes from `ffmpeg-static` and is used for MP4/GIF export and to fix up recording metadata. If it's missing, the app still runs and exports WebM. To use a different binary, set the `FFMPEG_PATH` environment variable.

---

## User guide

A full user guide is built into the app: press **F1**, choose **Help → User Guide**, or click the **?** button in the top bar. It opens in its own window (searchable, works offline). The source is `src/renderer/help.html`.

## Features

### Recorder
- **Record cursor separately (Windows, on by default for screens):** the screen is captured with FFmpeg *without* the mouse cursor. A small helper (Windows PowerShell + built-in .NET, nothing to install) records the cursor position and type (arrow, text I-beam, hand, resize, busy) about 60 times a second, on the same clock as the video. The editor draws the only cursor, so you can **resize it (0.5–5×), recolor it or hide it** with no double cursor and no lag. Pausing works: the capture keeps running and the app places one clip per recorded stretch, lined up with your audio tracks. Window recordings, and recordings on macOS/Linux, use the normal method with the cursor in the video.
- Pick any **screen** or **window** from a thumbnail picker.
- **Custom region**: drag a rectangle on the screen. The region becomes a crop on the clip, so you can widen it later without re-recording.
- **Up to 4 audio sources**, each with its own level meter and name. Each one is recorded to **its own file and timeline track**, so you can EQ and mix them separately. Sources can be any mix of Windows audio devices (USB mics, webcams, virtual cables) and ASIO inputs. **System audio** (Windows) stays with the screen clip.
- **ASIO inputs (Windows)**: interfaces with ASIO drivers appear under an **ASIO** group. Each source picks an input (mono) or input pair (stereo), for example Voice = Input 1 and Guitar = Input 2. ASIO allows one interface per app, so all ASIO sources must be on the **same interface**. They're captured in one stream and split into sample-locked **32-bit float WAV** files. Pausing and stopping stay in sync with the video.
- Browser audio processing (auto gain, echo cancellation) is turned off for Windows-audio mics, and noise suppression is optional, so voiceovers stay natural.
- **Webcam** recorded as its *own* clip, so it lands on a separate track as a movable picture-in-picture (rounded corners and shadow by default).
- **Cursor data**: the cursor position is sampled 30×/s and saved with the clip, which powers the cursor highlight effect.
- 30 or 60 fps, an optional **3-2-1 countdown**, and a floating **control bar** (timer, pause/resume, stop, discard). The control bar is hidden from the capture on Windows and macOS.
- Global hotkeys: **Ctrl/Cmd+Shift+F9** pauses or resumes, **Ctrl/Cmd+Shift+F10** stops.
- Recordings are streamed to disk while you record (crash-safe) under `Videos/EZ Cam Recordings`.

### Editor
- **Multi-track timeline**: drag clips between tracks, trim either edge, split at the playhead (**S**), duplicate, copy/paste, delete. Snapping to clip edges and the playhead, ruler scrubbing, Ctrl+wheel zoom, fit-to-window.
- **Waveforms** on audio clips, and on video clips with sound, for cutting precisely between words. They're computed in the background with FFmpeg and cached. The drawing follows trims, volume and fades.
- **Resizable tracks**: drag the bottom edge of a track header (double-click resets it), or use the track-height slider in the timeline toolbar. **Ctrl+Shift+wheel** resizes all tracks, and **Shift+wheel over a track name** resizes just that track.
- **Volume envelopes**: turn on **Envelopes** (or press **E**), click an audio clip to add a point, drag to adjust (hold **Shift** to change only the level), and double-click or right-click a point to remove it. A tooltip shows the level in dB. Points stay with the audio through trims, splits and ripple edits, play back smoothly, are included in exports, and are reflected in the waveform.
- **In/out range** (like Camtasia's selection): **Shift+drag** on the ruler or an empty part of a track, or press **I** / **O** at the playhead. Drag the green and red markers to adjust (they snap to clip edges and the playhead). Range tools: **Play range**, **Silence** (sets the *selected* audio clip to −∞ inside the range, with 20 ms ramps), **Ripple delete** (Shift+Delete), **Delete** leaving a gap (Delete), **Insert gap**, **Split at edges**, **Clear** (X or Esc). Locked tracks are never changed.
- **Duck a range**: with Envelopes on and a range set, drag a clip's envelope *inside* the range up or down to set that section's level (only that clip changes).
- **Fades**: drag the **Fade** tile from the Effects tab onto a clip. Drop it near the start for a fade-in only, near the end for a fade-out only, or in the middle for both (the drop zone highlights first). Or drag the small round handles at a clip's top corners to set or resize either fade. Fades apply to both picture and sound.
- **Extend frame** (freeze frame): select a video clip and press **F** (or click **Extend frame**). Hold the frame **at the playhead** (the rest of the clip moves later) or **at the end of the clip**, for a set length or with **Fill the gap to the next clip** (the next clip doesn't move). You can also push other tracks so narration stays in sync. Or **Alt+drag** a video clip's right edge past the end of the footage: the extra length becomes a held last frame, stopping at the next clip. Freezes are ordinary silent clips (striped), so you can trim, move, fade or delete them, and their frame time can be changed in the properties panel.
- **Clip speed** (0.25×–8×, exact values up to 16×): set it in the properties panel (slider, number, or 0.5/1/1.5/2/4× presets), or drag **Clip speed** from the Effects tab onto a clip. Audio keeps its pitch. The clip gets shorter or longer and later clips on the track move to make room. Trimming, splitting, ripple edits, envelopes, waveforms, freeze frames and cursor effects all follow the speed. Sped-up clips are labelled "2× · name".
- **Text to speech** (top bar, or **Ctrl+T**): type or paste a script, pick a voice, **Preview**, then **Insert at playhead**. Clips land on a **Voiceover** track, one per paragraph if you like, with an adjustable gap, ready for EQ, envelopes and trimming.
  - **Windows voices**: built in, offline and free (the "desktop" voices such as David and Zira, with an adjustable rate).
  - **ElevenLabs**: paste your API key once (stored encrypted with Windows' own data protection). Your voices, including cloned ones, and the available models load from your account. Adjust stability, similarity and speed. Previews are reused when you insert, so you don't pay twice for the same text.
- **Markers**: press **M** (or the **Marker** button) to add one at the playhead. On the ruler, click a marker to jump to it, drag to move it (snaps to clips), double-click to rename, right-click to delete. **Ctrl+← / Ctrl+→** jumps between markers. Clips, ranges and markers snap to each other, and ripple delete and insert gap move markers along. The **Project** panel (nothing selected) lists all markers to rename, recolor, jump to or delete, and has **Copy chapters for YouTube**. MP4 exports can embed the markers as **chapters**.
- **Ripple delete a clip**: select it and press **Shift+Delete** (or Ctrl+Backspace) to remove it and pull everything after it on that track to the left.
- **Track controls**: rename, mute, hide, lock, delete, add. **Drag a track by its grip** to reorder it, and drag the right edge of the track-name column to make room for long names. **Fit timeline to tracks** grows and shrinks the timeline with the track heights so every track stays in view. Projects open zoomed to fit with the smallest track height.
- **Mute clips** (**Shift+M**): a muted clip stays in place but isn't shown or heard (greyed, with a MUTED label).
- **Multi-select**: Ctrl/Shift+click or drag a box; move, trim, nudge, change and delete several clips at once, across tracks. **Groups** (Ctrl+G) move and trim together on the timeline, and act as one object on the preview (move, scale, animate). Alt+click picks one member.
- **Follow edit position** (Follow button): while you move or trim a clip, the playhead stays on its start (or end), so the preview shows exactly where it lands.
- **Live preview canvas**: click to select; drag to move; drag the corner handles to resize clips, annotations and zoom areas.
- **Clip properties**: position, scale, opacity, corner radius, drop shadow, crop, volume (up to 200%), fade in/out, media in-point, Fit / Fill / Center.
- **Annotations**: Callout (text box), Title, Arrow, Shape, Keystroke, **Spotlight** (dims everything else) and **Blur / Pixelate** (hides sensitive info). Text can **mix fonts, sizes, weights, italics and colours** in one block, and text boxes scale from their corners.
- **Stretch**: drag the middle of an edge of a video or image to stretch it on that side only (for example to push the Windows taskbar out of the frame).
- **Zoom-n-Pan**: a timeline item that eases into any part of the frame and back out. Drag the box on the preview to aim it. Zooms can be **keyframed** to pan and zoom while they play. **Auto-zoom** makes zooms from a screen recording's clicks and typing.
- **Animation**: keyframes for position, scale, rotation and opacity on any visual clip (Animate mode, **K**: drag on the preview to make a keyframe at the playhead), with easing per keyframe, a motion path, presets and **behaviors** for annotations (how they arrive, move and leave). **Copy and paste keyframes** (pick diamonds on the timeline clip, Ctrl+C / Ctrl+V), and **Hold until playhead** to keep a clip still for a while.
- **EQ** on any clip with audio: a 5-band EQ (high-pass, low shelf, low-mid, presence, high shelf). Drag points on the curve, scroll to change Q, double-click to reset, or type exact values. Presets include Voice: clarity, warm and broadcast; Podcast; Remove rumble; Reduce muddiness, harshness and boxiness. EQ is applied live during playback and included in exports.
- **Compressor** per clip (threshold, ratio, attack, release, makeup gain), plus one-click **copy EQ to all clips** of that recording or track. EQ, Compressor and De-esser each have their own **presets** (the + button saves yours).
- **Voice clean-up**, live (heard while you edit, nothing to process first): **Noise removal** (learns the recording's own background noise), **De-click** (mouth clicks and lip smacks, repaired by interpolation), **De-esser**, **De-plosive** and **De-breath**. "Hear only what's removed" checks that only noise is taken away.
- **Normalize** to a peak or loudness target (−16 / −14 / −23 LUFS), measured on the part of the clip you use and re-measured after edits. **Emphasize** ducks every other clip under narration, following the voice, with a smooth return. A master **limiter** and a level/LUFS meter sit under the preview.
- **Volume envelopes** can be moved up or down as a whole with **Alt+drag**.
- **Cursor highlight** with editable style (glow, solid circle, ring, spotlight that dims the rest), size, color and opacity. **Enlarged cursor** draws a bigger white or black arrow (1–5×) over the recorded cursor at its recorded position. Recordings now store the display scale so the arrow matches the real cursor size. **Cursor sync** lines the effects up with the recorded cursor (screen frames arrive slightly after the mouse-position samples). New recordings measure the delay automatically, and a slider on each clip fine-tunes it. Chromium can't capture the screen without the system cursor, so the recorded cursor stays in the video; once synced, the enlarged arrow covers it.
- **Visual effects** per video or image clip: **remove a color (chroma key)** on the GPU, with an eyedropper (*Pick from preview*), tolerance, softness and spill removal; brightness, contrast, saturation, hue, black & white, sepia and blur; border (width and color); rotation; flip horizontal and vertical. Drag presets from Effects → Visual effects onto a clip, then fine-tune in the properties panel. Clips with effects show an **FX** badge.
- **Transitions** (Transitions tab): Fade, Wipe left/right/up, Slide left/right/up, Zoom, Circle, Blur, and **3D** (cube, flip, doors, swing, drawn with WebGL). Drop one on the cut between two touching clips for a transition between them; drag its edges on the timeline to change its length; right-click a clip to remove one. Drag onto a clip's start (in), end (out) or middle (both), or set type and length per side in the properties panel. They work on video, images and annotations and appear as purple markers on the clip. For a cross-dissolve, overlap two clips on different tracks and give the top one a Fade in.
- **Zoom-n-Pan transitions**: separate **Zoom in** and **Zoom out** times (default 1 s) with Smooth or Linear motion, shown as ramps on the timeline clip and adjustable by dragging its corner handles. Back-to-back zoom clips pan from one area to the next instead of zooming out in between. The zoom now also plays while the zoom clip is selected.
- **Text borders**: callouts and titles get a **text outline** (width and color), and boxes and shapes get a **border style** (solid, dashed, dotted).
- **Media bin**: import video, images and audio (or drag files in from your OS), thumbnails, double-click to add at the playhead, drag onto a specific track. Each file shows *in use* or *unused*; **Remove unused** takes unused files out of the project and can also send EZ Cam's own copies to the Recycle Bin to free disk space. **Folder** opens the project's Media folder.
- **Library**: save clips, annotations and groups to reuse in any project. Comes with lower thirds, title designs and original **music beds** (free to use, also on monetized videos), previewable in place.
- **Projects** (`.ezcam`): a new project is saved **in its own folder with a Media folder**: recordings are moved there and imports copied (optional), with paths stored relative to the folder so it can be moved or copied to another PC. **Save Project with Media in One Folder…** collects an older project. **Recent projects**, full **undo/redo**, and autosave with crash recovery.
- **Settings** (Ctrl+,): recordings and projects folders, playback device, and more.

### Text-based editing and captions
- **Transcript tab**: speech is turned into text with **Whisper (whisper.cpp) running on your PC**. The engine and a model (Base, Small or Medium; English or any language) download once, then it works offline and your recordings never leave the PC. Word times are aligned to the sound (DTW).
- Click a word to jump there; the spoken word is highlighted while playing; find words; fix misheard words.
- **Select words and press Delete** to cut them out of the video. The gap closes up on every track, so overlays stay in step; tracks marked *Keep whole when cutting from the transcript* (music) and locked tracks are left alone. Cuts land at the quietest point between the words with a short fade. Cut words stay visible, struck through, and can be brought back with a click.
- **Clean up…** finds fillers (um, uh…), repeated words and phrases, sounds Whisper didn't write down, and pauses over a length you choose, each with a play button and a checkbox. Pauses can be **shortened** or **silenced** (sound off, length kept, with an adjustable amount kept around the words).
- **Captions**: one Captions clip shows what's being said, made from the transcript as the video plays, so edits show up straight away. Looks (Classic box, Clean outline, Bold social, Light box, Yellow), word-by-word highlight, 1–3 lines, smart line breaks and steady timing. Burned into exports, or **saved as .srt** for YouTube.
- Project canvas presets (1080p, 720p, 1440p, 4K, vertical, square) or a custom size. The first clip on an empty timeline sets the size, like Camtasia.

### Narration
- **Narration tab**: record voice only (Windows audio or an ASIO input) from the playhead while the timeline plays (optionally muted), with level meter, clip level, optional countdown and a script box saved with the project. Takes land on a "Narration" track; Space stops.

### Export
- **MP4** (H.264/AAC, faststart), **WebM** (VP9/Opus) or **animated GIF** (palette-optimized).
- Quality presets and frame-rate choice, with progress and cancel.
- **YouTube**: chapters from markers (copied for the description), thumbnail export, and upload from the app (see the user guide).
- **Fast export** (default): renders frame by frame faster than real time, using hardware video decode/encode when available, with a speed and time-left readout. **Real-time export** remains as an option.

---

## Keyboard shortcuts

| Key | Action |
| --- | --- |
| Space | Play / pause |
| S | Split at playhead (selected clip, or everything under the playhead) |
| Delete / Backspace | Delete selected (or the range, leaving a gap) |
| Shift+Delete, Ctrl+Backspace | Ripple delete the range, or the selected clip |
| I / O / X | Set In / set Out at the playhead / clear the range |
| E | Toggle volume-envelope editing |
| F | Extend frame (freeze) on the selected video clip |
| K | Animate mode (keyframes) |
| Shift+M | Mute / unmute the selected clips |
| Ctrl/Cmd+G, Ctrl/Cmd+Shift+G | Group / ungroup |
| Ctrl+T | Text to speech |
| M | Add marker at the playhead |
| Ctrl+← / Ctrl+→ | Previous / next marker |
| Ctrl+Shift+wheel, Shift+wheel on a track name | Resize all tracks / one track |
| ← / → | Step one frame (hold Shift for 1 second) |
| Home / End | Jump to start / end |
| Ctrl/Cmd+Z, Ctrl+Y / Cmd+Shift+Z | Undo / redo |
| Ctrl/Cmd+C / X / V / D | Copy / cut / paste / duplicate clip |
| Ctrl/Cmd+R | New recording |
| Ctrl/Cmd+I | Import media |
| Ctrl/Cmd+E | Export |
| Ctrl/Cmd+S, Ctrl/Cmd+Shift+S | Save / save as |
| Ctrl/Cmd+= / − / 0, Ctrl+wheel | Zoom timeline / fit |
| Ctrl/Cmd+Shift+F9 / F10 | Pause / stop recording (works globally) |

---

## Platform notes

- **macOS**: the first recording asks for **Screen Recording** permission (System Settings → Privacy & Security → Screen Recording). After you grant it, restart the app. Camera and microphone prompts appear automatically. System audio capture isn't enabled on macOS in this version.
- **Windows**: system audio uses loopback capture of the default output device.
- **ASIO**: uses the `audify` library (RtAudio), which runs in the main process. ASIO drivers allow one app at a time, so close your DAW (or anything else using the interface) before recording. The buffer size comes from your interface's ASIO control panel.
- **Linux**: on Wayland, screen capture goes through the desktop portal (PipeWire), which shows its own picker. The region-selection overlay needs a compositor that supports transparent windows.

---

## How it works

```
src/
  main/
    main.js        Electron main: windows, app:// protocol (with HTTP range for seeking),
                   capture source handler, IPC, streaming file writes, menus, hotkeys
    preload.js     Safe, contextIsolated bridge (window.api)
    ffmpeg.js      Remux (fix WebM duration) + MP4/GIF encoding
    peaks.js       Waveform peaks via FFmpeg (200/s, cached in the app's data folder)
    screencap.js   Cursor-free screen capture (FFmpeg gdigrab) + cursor position/type tracker
    tts.js         Text to speech: Windows voices (PowerShell/System.Speech), ElevenLabs API, encrypted key
    asio.js        ASIO capture via audify/RtAudio -> 32-bit float WAV, level meter
    transcribe.js  Speech to text: downloads whisper.cpp + a model once, runs it, word times (DTW)
    noiseprofile.js  Background-noise profile for live Noise removal
    library.js     Library folder, starter designs and music beds
    youtube.js     YouTube upload (OAuth, resumable upload)
scripts/
  setup.js         Finishes Electron / FFmpeg / ASIO downloads (npm run setup)
  renderer/
    index.html     Editor UI          styles.css   Dark theme
    recbar.html    Floating recording control bar
    region.html    Full-screen region selector
    js/
      util.js         Event bus, DOM helper, icons, toasts, modals
      state.js        Project model, tracks/items, split/duplicate, undo history, save format
      visualfx.js     Visual effects, WebGL chroma key, transitions, enlarged cursor
      audiofx.js      Voice EQ + compressor (Web Audio graph, presets, interactive curve)
      engine.js       Playback clock + compositor (scene canvas -> zoom-n-pan -> output canvas),
                      Web Audio mixing, fades, cursor highlight
      annotations.js  Callout/title/arrow/shape/spotlight/blur/keys/captions drawing, rich text
      motion.js       Keyframes, motion paths, behaviors, zoom keyframes
      joins.js        Transitions between touching clips    trans3d.js  3D transitions (WebGL)
      voice.js        Live voice clean-up (noise removal, de-click, de-ess, de-plosive, de-breath)
      limiter-core.js DSP shared by the page and the audio worklet (limiter, EzVoice)
      transcript.js   Transcript tab: words on the timeline, cutting, Clean up, silencing, captions, .srt
      library.js      Library tab (clips, designs, music)
      waveform.js     Waveform loading + drawing on clips
      extendframe.js  Extend-frame (freeze) dialog
      tts.js          Text-to-speech dialog + voiceover placement
      timeline.js     Timeline UI: ruler, scrub, move/trim, snapping, drop targets
      preview.js      On-canvas selection, move/resize handles, zoom-area editing
      properties.js   Inspector panel per item type + project settings
      media.js        Media probing, thumbnails, bin
      recorder.js     Source picker, devices, capture, MediaRecorder, cursor sampling
      exporter.js     Export dialog + real-time render pipeline
      fastexport.js   Fast (offline) export: WebCodecs decode/encode + offline audio mix
      app.js          Wiring: toolbar, tabs, shortcuts, menu, drag-drop, close guard
```

- The UI and media are served from one custom origin (`app://local`). That keeps the preview canvas untainted, so it can be captured for export.
- **Recording**: `getDisplayMedia` (answered in the main process with the source you picked), with a fallback to Chromium's desktop-capture constraints. `MediaRecorder` writes 1-second chunks straight to disk. FFmpeg then remuxes the WebM so it has a duration and seek cues.
- **Playback**: one `<video>` element per clip, kept in sync with a wall-clock timeline (drift correction, pre-roll for upcoming clips). Every frame, each visible item is drawn bottom track to top onto a scene canvas. Zoom-n-Pan then crops and scales that onto the output canvas.
- **Fast export** (`fastexport.js` + `src/main/fastexport.js`):
  - FFmpeg copies each source video's packets (no decoding) into a temp file and lists each packet's size, timestamp and keyframe flag (`-f data` + `-f framecrc` from one pass).
  - The renderer feeds them to WebCodecs `VideoDecoder`, so each clip is decoded once, front to back. Clips whose codec isn't supported fall back to seeking their `<video>` element.
  - Each output frame is drawn by the normal compositor, so it matches the preview. Frames go to `VideoEncoder` (H.264 Annex B for MP4/GIF, VP9/VP8 in IVF for WebM).
  - Colour spaces follow Chromium's `<video>` rules, so colours match the preview.
  - Audio is mixed with `OfflineAudioContext` through the same EQ/compressor nodes. It's done in 5-minute pieces with 1 s pre-roll; each clip's audio is extracted once by FFmpeg (with `atempo` for clip speed).
  - FFmpeg muxes the result.
- **Real-time export**: the output canvas plus the Web Audio mix go into a `MediaRecorder` while the timeline plays from start to end. FFmpeg then transcodes to MP4 or GIF.

### Project file format
`.ezcam` is plain JSON: `settings` (canvas size, fps, background), `media` (absolute file paths, plus cursor samples and crop), and `tracks[].items[]`. Item types are `video | image | audio | annotation | zoom`, each with `start`, `duration` and type-specific fields (annotation kinds include `captions`). Managed projects set `managed: true` and store media paths relative to the project folder (`rel`). Transcripts are stored on their media file (`media[].transcript.words = [{ w, s, e }]`, times in the file).

---

## Troubleshooting

- **"Electron failed to install correctly"**: run `npm run setup`. If Electron still says FAIL, do the manual install from the project folder (Windows, Command Prompt):
  ```
  powershell -Command "Invoke-WebRequest https://github.com/electron/electron/releases/download/v38.8.6/electron-v38.8.6-win32-x64.zip -OutFile electron.zip"
  powershell -Command "Expand-Archive electron.zip -DestinationPath node_modules\electron\dist -Force"
  echo|set /p="electron.exe" > node_modules\electron\path.txt
  ```
  Replace `38.8.6` with the version shown in `node_modules\electron\package.json` if yours is different.
- **No ASIO group in the microphone list**: run `npm run setup` and check the ASIO line. Also make sure the interface's ASIO driver is installed and no other app is using it.
- **ASIO "device unavailable"**: another program has the driver open, or the sample rate is locked. Close the other program, or set 48 kHz in the interface's control panel.

## Known limitations / roadmap

- **Fast export speed** depends on the PC: hardware video decoding/encoding and the GPU do most of the work. On a 2-core test machine with no GPU (all software), a 1080p project with picture-in-picture, zoom, callouts and EQ ran at about 1.4× real time.
- **Transcription** runs on the CPU and is Windows-only for now; Small takes about a minute or two per 10 minutes of speech on a typical PC. Singing over music is much less reliable than speech.
- Imported media is limited to what Chromium can decode (MP4/H.264, WebM, MP3, WAV, etc.). Many `.mov`/`.mkv` files won't play; convert them first.
- Ideas for later: caption .srt saved with each export and an "include captions" export option, word-by-word caption pop-in, importing .srt files.

## License
MIT
