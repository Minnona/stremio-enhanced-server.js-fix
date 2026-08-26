# stremio-enhanced-server.js-fix
A fixed version for server.js engine for stremio-enhanced

Current baseline: **Stremio Server 4.21.0**

# Stremio Enhanced `server.js` Changes

## Torrent / Peer Performance

- Increased peer-search minimum from **40 → 80**
- Increased peer-search maximum from **150 → 500**
- Added explicit BitTorrent listener on **port 6881**
- Fixed `engine.listen(callback)` so it uses the proper engine object and port `6881`
- Added swarm error forwarding into the torrent engine error handler
- Added error handlers for TCP/uTP listener failures
- Increased outstanding block requests per peer:
  - Old: roughly **5–50**
  - New: roughly **16–112**
- Changed request scaling to use available peers more aggressively
- Increased choked-peer timeout from **5s → 15s**
  - Reduces unnecessary peer disconnect/reconnect churn

## Playback / Streaming

- Made `file.select()` idempotent
  - Previously repeated calls could create duplicate/overlapping whole-file selections
  - Now an existing selection is reused
- Fixed `file.deselect()` so the stored selection reference is cleared properly
- Changed `prewarmStream()` so it does not blindly add another full-file selection when the file is already being handled by the streaming/cache path
- Increased critical playback/read-ahead window:
  - Old: roughly **1 MiB**, maximum **4 pieces**
  - New: roughly **4 MiB**, maximum **8 pieces**
- Ensured the critical playback window can never become **0 pieces**
  - The previous calculation could truncate to `0` on torrents with large piece sizes
- Kept the existing HTTP Range implementation unchanged
  - It already correctly supports:
    - `206 Partial Content`
    - `Content-Range`
    - `Content-Length`
    - `Accept-Ranges`
    - ranged torrent `createReadStream()` calls

## HLS Playback Stability

- Queued concurrent HLS segment requests instead of canceling the active FFmpeg conversion
- Stopped the existing FFmpeg converter before starting a genuine seek replacement
  - Prevents abandoned FFmpeg processes from accumulating
- Kept a rolling cache of the 12 most recent audio/video HLS segments
  - Avoids aggressively clearing every generated A/V segment buffer

## Unchanged

- Existing Stremio cache behavior
- Download speed soft/hard limits
- Maximum-connections setting
- Handshake/request timeout settings
- Torrent HTTP Range implementation
- Torrent verification/storage logic
- Codec selection and transcoding parameters

## Summary

The patch improves:

- Peer discovery
- Peer utilization
- Incoming BitTorrent connectivity
- Download request concurrency
- Network error handling
- Peer stability
- Playback-piece prioritization
- Prevention of duplicate torrent selections
- HLS converter stability
- Reuse of recently generated HLS segments

The goal is to preserve improved torrent download speeds while fixing playback startup failures, FFmpeg converter thrashing, and short HLS buffer stalls.

## Installation

Replace the existing Stremio Enhanced `server.js` with the patched version.

### Windows

```text
%APPDATA%\stremio-enhanced\streamingserver\server.js
```

Usually resolves to:

```text
C:\Users\<USERNAME>\AppData\Roaming\stremio-enhanced\streamingserver\server.js
```

### Linux

```text
~/.config/stremio-enhanced/streamingserver/server.js
```

Usually resolves to:

```text
/home/<USERNAME>/.config/stremio-enhanced/streamingserver/server.js
```
