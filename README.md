# ffmpeg web service API

An web service for converting audio/video files using Nodejs, Express and FFMPEG

Based off of jrottenberg/ffmpeg container

## Endpoints

> POST /mp3 - Convert audio file in request body or by URL to mp3

> POST /mp4 - Convert video file in request body or by URL to mp4

> POST /jpg - Convert image file in request body or by URL to jpg 
 
> POST /screenshot - Create screenshot on time from m3u8 playlist

> GET /, /readme - Web Service Readme

All conversion endpoints (`/mp3`, `/m4a`, `/mp4`, `/jpg`) support two input formats:

- multipart form upload with a file (`file=@input.ext`)
- JSON body with a URL to media (`playlistUrl` or `url`)

### /mp3, /m4a

Curl Ex:

> curl -F "file=@input.wav" 127.0.0.1:3000/mp3  > output.mp3

> curl -F "file=@input.m4a" 127.0.0.1:3000/mp3  > output.mp3

> curl -F "file=@input.mov" 127.0.0.1:3000/mp4  > output.mp4

> curl -F "file=@input.mp4" 127.0.0.1:3000/mp4  > output.mp4

> curl -F "file=@input.tiff" 127.0.0.1:3000/jpg  > output.jpg

> curl -F "file=@input.png" 127.0.0.1:3000/jpg  > output.jpg

> curl -F "file=@input.png" -F 'outputOptions=-codec:v libx264;-crf 20' 127.0.0.1:3000/mp4  > output.mp4

### /mp3, /m4a, /mp4, /jpg with URL (JSON)

You can also pass a URL (for example, an HLS playlist) instead of uploading a file.  
Send a JSON body with `Content-Type: application/json` and `url` (preferred) or `playlistUrl` (kept for backward compatibility):

```json
{
  "url": "https://example.com/playlist.m3u8",
  "userAgent": "Optional User-Agent string",
  "outputOptions": "-codec:a libmp3lame;-b:a 192k"
}
```

`userAgent` is optional and will be passed to ffmpeg as `-user_agent`.  
`outputOptions` is optional: a single string with ffmpeg options separated by `;`. If provided, it **replaces** the default options for the endpoint.  
`extraOutputOptions` is optional: a string with options separated by `;` that are **appended** to the default (or to `outputOptions`). Use it to pass offset and duration for a fragment, e.g. `-ss 00:01:30;-t 60` (start at 1m30s, length 60 seconds).

Curl examples:

Convert URL to mp3:

```bash
curl -X POST http://localhost:3000/mp3 \
  -H "Content-Type: application/json" \
  -d '{
    "url": "https://example.com/playlist.m3u8"
  }' \
  --output output.mp3
```

Convert URL to m4a:

```bash
curl -X POST http://localhost:3000/m4a \
  -H "Content-Type: application/json" \
  -d '{
    "url": "https://example.com/playlist.m3u8"
  }' \
  --output output.m4a
```

Extract a 60-second audio fragment starting at 1m30s (works for both URL and file upload):

```bash
curl -X POST http://localhost:3000/mp3 \
  -H "Content-Type: application/json" \
  -d '{
    "url": "https://example.com/playlist.m3u8",
    "extraOutputOptions": "-ss 00:01:30;-t 60"
  }' \
  --output fragment.mp3
```

With file upload, pass `extraOutputOptions` as a form field:

```bash
curl -F "file=@input.wav" -F 'extraOutputOptions=-ss 00:01:30;-t 60' http://localhost:3000/mp3 -o fragment.mp3
```

> curl -X POST http://localhost:3000/screenshot \
    -H "Content-Type: application/json" \
    -d '{
        "url": "https://site.com/playlist.m3u8",
        "timestamp": "00:00:05.500",
        "userAgent": "Some User-Agent"
    }' \
    --output screenshot.jpg

## Configuration and New Endpoints
You can replace the ffmpeg conversion settings by environment variable ENDPOINTS:

    # docker-composer.yml

    ...
    environment:
      ENDPOINTS: |
        {
          "mp4": {
            "extension": "mp4",
            "outputOptions": [
              "-fflags +genpts",
              "-r 24"
            ]
          }
        }

You can also change the conversion settings on the fly using the outputOptions parameter with each option separated with ';' (see an example above)

## Installation

Requires local Node and FFMPEG installation.

1) Install FFMPEG https://ffmpeg.org/download.html

2) Install node https://nodejs.org/en/download/
Using homebrew:
> $ brew install node

## Dev - Running Local Node.js Web Service

Navigate to project directory and:

Install dependencies:
> $ npm install

Start app:
> $ node app.js

Check for errors with ESLint:
> $ ./node_modules/.bin/eslint .

## Running Local Docker Container

Build Docker Image from Dockerfile with a set image tag. ex: docker-ffpmeg
> $ docker build -t sergiusd/docker-ffpmeg .

Launch Docker Container from Docker Image, exposing port 9025 on localhost only

> docker run -d \
    --name ffmpeg-service \
    --restart=always \
    -v /storage/tmpfs:/usr/src/app/uploads \
    -p 127.0.0.1:9025:3000 \
    sergiusd/docker-ffpmeg

Launch Docker Container from Docker Image, exposing port 9026 on all IPs
> docker run -p 9025:3000 -d sergiusd/docker-ffpmeg
