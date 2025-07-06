# RTMP Server in Go with Dynamic Transcoding to HLS

A high-performance RTMP ingest server written in Go that receives video streams, pipes them in real-time to FFmpeg to generate HLS segments, and exposes those segments over HTTP for compatible players.

## Features

- **RTMP Ingestion**: Accepts RTMP streams from OBS, FFmpeg, or any RTMP-compatible client
- **Real-time Transcoding**: Uses FFmpeg to convert RTMP streams to HLS format
- **HTTP Delivery**: Serves HLS playlists and segments with proper CORS headers
- **REST API**: Control streams via HTTP API endpoints
- **Metrics**: Prometheus metrics for monitoring
- **Concurrent Streams**: Supports multiple simultaneous streams
- **Graceful Shutdown**: Proper cleanup of resources and FFmpeg processes

## Prerequisites

- Go 1.21 or later
- FFmpeg installed and available in PATH

## Installation

1. Install dependencies:
```bash
go mod download
```

3. Build the server:
```bash
go build -o rtmp-server cmd/server/main.go
```

## Usage

### Starting the Server

```bash
./rtmp-server -config config.yaml
```

The server will start:
- RTMP server on port 1935 (default)
- HTTP server on port 8080 (default)
- Metrics endpoint on port 9090 (if enabled)

### Streaming with OBS

1. Open OBS Studio
2. Go to Settings > Stream
3. Set Service to "Custom"
4. Set Server to `rtmp://localhost:1935/live`
5. Set Stream Key to your desired stream name (e.g., `mystream`)
6. Click "Start Streaming"

### Playing the Stream

The HLS stream will be available at:
- Playlist: `http://localhost:8080/hls/live/mystream/playlist.m3u8`
- Segments: `http://localhost:8080/hls/live/mystream/segment_000.ts`

You can play this in:
- Web browsers with HLS.js or Video.js
- VLC media player
- Any HLS-compatible player

## API Endpoints

### Stream Management

- `GET /api/v1/streams` - List all streams
- `GET /api/v1/streams/{streamID}` - Get stream details
- `POST /api/v1/streams/{streamID}/start` - Start a stream
- `POST /api/v1/streams/{streamID}/stop` - Stop a stream
- `DELETE /api/v1/streams/{streamID}` - Delete a stream

### Health and Metrics

- `GET /health` - Health check endpoint
- `GET /metrics` - Prometheus metrics

### HLS Delivery

- `GET /hls/{app}/{stream}/playlist.m3u8` - HLS playlist
- `GET /hls/{app}/{stream}/{segment}` - HLS segment files

## Testing

### Test with FFmpeg

1. Start the server:
```bash
./rtmp-server
```

2. Stream a test video:
```bash
ffmpeg -re -i test-video.mp4 -c copy -f flv rtmp://localhost:1935/live/test
```

3. Play the HLS stream:
```bash
ffmpeg -i http://localhost:8080/hls/live/test/playlist.m3u8 -c copy output.mp4
```

### Test with OBS

1. Configure OBS to stream to `rtmp://localhost:1935/live/obs-test`
2. Start streaming in OBS
3. Open the playlist URL in a web browser or VLC

## Docker

### Building the Docker Image

```bash
docker build -t rtmp-server .
```

### Running with Docker

```bash
docker run -p 1935:1935 -p 8080:8080 -p 9090:9090 \
  -v $(pwd)/hls:/app/hls \
  rtmp-server
```

## Architecture

The server consists of several components:

1. **RTMP Server**: Handles RTMP connections using the joy4 library
2. **Stream Manager**: Manages stream lifecycle and FFmpeg processes
3. **HTTP Server**: Serves HLS content and provides REST API
4. **Configuration**: YAML-based configuration management
5. **Metrics**: Prometheus metrics collection

## Performance

The server is designed to handle:
- Multiple concurrent RTMP streams
- Real-time transcoding with configurable quality
- Efficient HLS segment delivery
- Proper resource cleanup

## Troubleshooting

### Common Issues

1. **FFmpeg not found**: Ensure FFmpeg is installed and in PATH
2. **Port conflicts**: Check if ports 1935, 8080, or 9090 are already in use
3. **Permission errors**: Ensure the HLS output directory is writable
4. **Stream not appearing**: Check RTMP URL format and stream key