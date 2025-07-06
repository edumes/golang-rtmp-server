# HLS Stream Generator

This module provides functionality to convert video files into HLS (HTTP Live Streaming) format with configurable segment duration and playlist window size.

## Features

- **Video Segmentation**: Split videos into `.ts` segments of configurable length
- **Sliding Window Playlist**: Maintain a configurable number of segments in the playlist
- **Real-time Playlist Updates**: Playlist updates automatically as segments are created
- **Robust Error Handling**: Validates input files and cleans up on failure
- **Signal Handling**: Gracefully handles SIGINT/SIGTERM signals
- **Cross-platform**: Works on Windows, macOS, and Linux
- **Flexible Configuration**: Support for custom codecs, bitrates, resolutions, and extra FFmpeg flags

## Requirements

- **FFmpeg**: Must be installed and available in your system PATH
- **Go 1.21+**: For the Go implementation

### Installing FFmpeg

**Windows:**
```bash
# Using Chocolatey
choco install ffmpeg

# Using Scoop
scoop install ffmpeg

# Or download from https://ffmpeg.org/download.html
```

**macOS:**
```bash
# Using Homebrew
brew install ffmpeg
```

**Linux (Ubuntu/Debian):**
```bash
sudo apt update
sudo apt install ffmpeg
```

## Usage

### Go Function

The main function for creating HLS streams:

```go
func CreateHLS(inputPath, outputDir string, segmentDuration, playlistWindow int) error
```

#### Basic Usage

```go
package main

import (
    "log"
    "golang-rtmp/internal/hls"
)

func main() {
    err := hls.CreateHLS("input.mp4", "output/", 4, 3)
    if err != nil {
        log.Fatalf("Failed to create HLS stream: %v", err)
    }
}
```

#### Advanced Usage with Custom Options

```go
package main

import (
    "log"
    "golang-rtmp/internal/hls"
)

func main() {
    opts := hls.DefaultHLSOptions()
    opts.SegmentDuration = 6
    opts.PlaylistWindow = 5
    opts.VideoBitrate = "2000k"
    opts.Resolution = "1920x1080"
    opts.ExtraFlags = []string{"-preset", "fast", "-crf", "23"}

    err := hls.CreateHLSWithOptions("input.mp4", "output/", opts)
    if err != nil {
        log.Fatalf("Failed to create HLS stream: %v", err)
    }
}
```

### Command Line Tools

#### Shell Script (Linux/macOS)

```bash
# Make executable
chmod +x scripts/create-hls.sh

# Basic usage
./scripts/create-hls.sh -i input.mp4 -o output/

# Advanced usage
./scripts/create-hls.sh -i input.mp4 -o output/ \
    -d 6 -w 5 -vb 2000k -r 1920x1080 \
    -e "-preset fast -crf 23"
```

#### Batch File (Windows)

```cmd
# Basic usage
scripts\create-hls.bat -i input.mp4 -o output\

# Advanced usage
scripts\create-hls.bat -i input.mp4 -o output\ -d 6 -w 5 -vb 2000k -r 1920x1080 -e "-preset fast -crf 23"
```

#### Go Example Program

```bash
# Build the example
go build -o hls-example cmd/hls-example/main.go

# Basic usage
./hls-example -input input.mp4 -output output/

# Advanced usage
./hls-example -input input.mp4 -output output/ \
    -duration 6 -window 5 -vb 2000k -resolution 1920x1080 \
    -flags "-preset fast -crf 23"
```

## FFmpeg Commands

The module generates FFmpeg commands similar to this:

### Basic Command
```bash
ffmpeg -i input.mp4 \
    -c:v libx264 -c:a aac \
    -b:v 1000k -b:a 128k \
    -s 1280x720 -r 30 \
    -f hls \
    -hls_time 4 \
    -hls_list_size 3 \
    -hls_flags delete_segments \
    -hls_segment_filename output/segment_%03d.ts \
    output/stream.m3u8
```

### Advanced Command with Custom Settings
```bash
ffmpeg -i input.mp4 \
    -c:v libx264 -c:a aac \
    -b:v 2000k -b:a 192k \
    -s 1920x1080 -r 60 \
    -preset fast -crf 23 \
    -f hls \
    -hls_time 6 \
    -hls_list_size 5 \
    -hls_flags delete_segments \
    -hls_segment_filename output/segment_%03d.ts \
    output/stream.m3u8
```

## Configuration Options

| Option | Default | Description |
|--------|---------|-------------|
| `SegmentDuration` | 4 | Segment duration in seconds |
| `PlaylistWindow` | 3 | Number of segments to keep in playlist |
| `VideoCodec` | libx264 | Video codec |
| `AudioCodec` | aac | Audio codec |
| `VideoBitrate` | 1000k | Video bitrate |
| `AudioBitrate` | 128k | Audio bitrate |
| `Resolution` | 1280x720 | Video resolution |
| `FPS` | 30 | Frame rate |
| `ExtraFlags` | [] | Additional FFmpeg flags |

## Output Structure

After successful conversion, your output directory will contain:

```
output/
├── stream.m3u8          # Main playlist file
├── segment_000.ts       # Video segment 1
├── segment_001.ts       # Video segment 2
├── segment_002.ts       # Video segment 3
└── ...                  # Additional segments
```

### Playlist File (stream.m3u8)

The generated playlist file will look like:

```m3u8
#EXTM3U
#EXT-X-VERSION:3
#EXT-X-TARGETDURATION:4
#EXT-X-MEDIA-SEQUENCE:0
#EXTINF:4.000000,
segment_000.ts
#EXTINF:4.000000,
segment_001.ts
#EXTINF:4.000000,
segment_002.ts
#EXT-X-ENDLIST
```

## Testing

### Running Unit Tests

```bash
# Run all HLS tests
go test ./internal/hls/...

# Run with verbose output
go test -v ./internal/hls/...

# Run with coverage
go test -cover ./internal/hls/...
```

### Manual Testing

1. **Create a test video file** (or use an existing one)
2. **Run the conversion**:
   ```bash
   go run cmd/hls-example/main.go -input test.mp4 -output test-output/
   ```
3. **Verify the output**:
   ```bash
   ls -la test-output/
   cat test-output/stream.m3u8
   ```
4. **Serve and test**:
   ```bash
   cd test-output/
   python3 -m http.server 8080
   # Open http://localhost:8080/stream.m3u8 in a video player
   ```

## Error Handling

The module includes comprehensive error handling:

- **Input Validation**: Checks file existence, readability, and non-empty status
- **Directory Creation**: Automatically creates output directories if missing
- **Process Management**: Properly handles FFmpeg process lifecycle
- **Signal Handling**: Gracefully responds to SIGINT/SIGTERM
- **Cleanup**: Removes partial output files on failure
- **Logging**: Detailed logging of FFmpeg stdout/stderr

## Common Issues and Solutions

### FFmpeg Not Found
```
Error: failed to start FFmpeg: exec: "ffmpeg": executable file not found in $PATH
```
**Solution**: Install FFmpeg and ensure it's in your system PATH.

### Permission Denied
```
Error: failed to create output directory: permission denied
```
**Solution**: Check write permissions for the output directory.

### Invalid Input File
```
Error: input validation failed: failed to stat input file: stat /path/to/file: no such file or directory
```
**Solution**: Verify the input file path is correct and the file exists.

### Insufficient Disk Space
```
Error: FFmpeg process failed: exit status 1
```
**Solution**: Ensure sufficient disk space for the output files.

## Performance Considerations

- **Segment Duration**: Shorter segments (2-4s) provide faster start times but more files
- **Playlist Window**: Larger windows use more disk space but provide better buffering
- **Video Quality**: Higher bitrates and resolutions increase file sizes and processing time
- **Codec Selection**: Hardware acceleration (e.g., `-c:v h264_nvenc`) can significantly improve performance

## Integration with Existing Codebase

The HLS module integrates seamlessly with the existing RTMP server:

```go
// Example: Convert RTMP stream to HLS
stream := streamManager.CreateStream("live", "mystream", "hls-output/")
err := stream.StartFFmpeg("ffmpeg", params, 4, 3)
```

## License

This module is part of the golang-rtmp project and follows the same license terms. 