# Icecast Metadata Player

Icecast Metadata Player is a simple to use Javascript class that plays an Icecast stream with real-time metadata updates.

  * Plays an Icecast stream using the Media Source Extensions API and HTML5 audio.
  * Pushes synchronized metadata updates taken from ICY metadata, or OGG metadata.
  * Available as an [NPM Package](https://www.npmjs.com/package/icecast-metadata-player) and as a file to include in a `<script>` tag.
    * See [Installing](#installing)

## Supported codecs:

* **MP3** `audio/mpeg`
* **AAC, AAC+, AAC-HE** `audio/aac`
* **FLAC, OPUS** `application/ogg`

## Checkout the demos [here](https://eshaz.github.io/icecast-metadata-js/)!

* [Installing](#installing)
* [Usage](#usage)
  * [ICY Metadata](#icy-metadata)
  * [OGG Metadata](#ogg-metadata)
  * [ICY and OGG Metadata](#icy-and-ogg-metadata)
  * [Playing a Stream](#playing-a-stream)
    * [Metadata](#metadata)
* [API](#api)
  * [Instantiating](#instantiating)
  * [Options](#options)
  * [Callbacks](#callbacks)
  * [Getters](#getters)
  * [Methods](#methods)
* [Troubleshooting](#troubleshooting)

See the main page of this repo for other Icecast JS tools:
https://github.com/eshaz/icecast-metadata-js

---

## Installing

### [NPM](https://www.npmjs.com/package/icecast-metadata-player)

* Run `npm i icecast-metadata-player` in the same directory as your `package.json` file to install it.
* Once icecast-metadata-js is installed, you can import each module listed above.
  * ES6 import (browser): `import { IcecastMetadataReader } from ("icecast-metadata-js");`
  * CommonJS require (NodeJS): `const { IcecastMetadataReader } = require("icecast-metadata-js");`

### `<script>` tag

* Download the Javascript file [here](https://github.com/eshaz/icecast-metadata-js/tree/master/src/icecast-metadata-player/build).
* Include the file in a `<script>` tag in your html.
  * i.e. `<script src="icecast-metadata-player-0.0.1.min.js"></script>`
* `IcecastMetadataReader` is made available as a global variable in your webpage to use wherever.

---

## Usage

* To use `IcecastMetadataPlayer`, create a new instance by passing in the stream endpoint, and the options object (optional). See the [Methods](#methods) section below for additional options.

   ```
   const player = new IcecastMetadataPlayer("https://stream.example.com", {
     onMetadata: (metadata) => {console.log(metadata)},
     ...options
   })
   ```
  IcecastMetadataPlayer supports reading ICY metadata, Ogg (Vorbis Comment) metadata, or both. Each section below describes how to instantiate `IcecastMetadataPlayer` to use these different metadata types.

  ### ICY Metadata

  * When reading ICY metadata, it is preferable, but not required, to pass in the `Icy-MetaInt` into the constructor of `IcecastMetadataReaader`. If `icyMetaInt` is falsy, for example if the CORS policy does not allow clients to read the `Icy-MetaInt` header, then `IcecastMetadataReader` will attempt to detect the metadata interval based on the incoming request data.

    <pre>
    const player = new IcecastMetadataPlayer("https://stream.example.com/stream.mp3", {
      onMetadata: (metadata) => {console.log(metadata)},
      metadataTypes: ["icy"]
      ...options
    })
    </pre>

  ### OGG Metadata

  * OGG (Vorbis Comment) metadata, if available, usually offers more detail than ICY metadata.

    <pre>
    const player = new IcecastMetadataPlayer("https://stream.example.com/stream.opus", {
      onMetadata: (metadata) => {console.log(metadata)},
      metadataTypes: ["ogg"]
      ...options
    })
    </pre>

  ### ICY and OGG Metadata

  * ICY and OGG metadata can both be read from the stream. Usually a stream will only have one or the other, but this option is possible if needed.

    <pre>
    const player = new IcecastMetadataPlayer("https://stream.example.com/stream.flac", {
      onMetadata: (metadata) => {console.log(metadata)},
      metadataTypes: ["icy", "ogg"]
      ...options
    })
    </pre>

### Playing a Stream

1. To begin playing a stream, call the `.play()` method on the instance.

    *Note:* IcecastMetadataPlayer will attempt to "fallback" on any CORS issues or Media Source API issues. See the [Troubleshooting](#troubleshooting) section for more details.

    <pre>
    const player = new IcecastMetadataPlayer("https://stream.example.com/stream.flac", {
      onMetadata: (metadata) => {console.log(metadata)},
      metadataTypes: ["icy"]
      ...options
    })

    player.play();
    </pre>

1. Metadata will be sent as soon as it is discovered via the `onMetadataEnqueue` and when the metadata is synchronized with the audio via the `onMetadata` callback. See the [Methods](#methods) section below for additional callbacks.
    
    #### `metadata`
    <pre>
    {
      metadata: {
        StreamTitle: "The stream's title", // ICY
        TITLE: "The stream's title", // OGG
        ... // key value pairs of metadata
      },
      timestampOffset: 10, // audio time when the metadata should be shown
      timestamp: 5, // audio player time when the metadata was discovered
    }
    </pre>

1. To stop playing the stream, call the `stop()` method on the instance.

    ```
    player.stop();
    ```

See the [HTML demos](https://github.com/eshaz/icecast-metadata-js/tree/master/src/demo/public/html-demos/) for examples.

---

## API

### Instantiating

```
const player = new IcecastMetadataPlayer(endpoint, {
  audioElement,
  icyMetaInt,
  icyDetectionTimeout,
  metadataTypes,
  onStream,
  onMetadata,
  onMetadataEnqueue,
  onError
})
```
### Options
* `endpoint` (required)
  * HTTP(s) endpoint for the Icecast compatible stream.
* `audioElement` (optional) - **Default** `new Audio()`
  * HTML5 Audio Element to use to play the Icecast stream.
* `metadataTypes` (optional) - **Default** `["icy"]`
  * Array containing zero, one, or both metadata types to parse
  * Values:
    * `[]` - Will not parse metadata
    * `["icy"]` - **Default** Parse ICY metadata only 
    * `["ogg"]` - Parse OGG (vorbis comment) metadata only
    * `["icy", "ogg"]` - Parse both ICY and OGG metadata
* `icyMetaInt` (optional)
  * ICY Metadata interval read from `Icy-MetaInt` header in the response
* `icyDetectionTimeout` (optional)
  * Duration in milliseconds to search for ICY metadata if icyMetaInt isn't passed in
  * Set to `0` to disable metadata detection
  * default: `2000`

### Callbacks
* `onStream(streamData)` (optional)
  * Called when stream audio data is sent to the audio element.
* `onMetadata(metadata, timestampOffset, timestamp)` (optional)
  * Called when metadata is synchronized with the audio.
  * `metadata` ICY or Ogg metadata in an object of key value pairs
    * ICY: `{ "StreamTitle: "The Stream Title" }`
    * Ogg: `{ "TITLE: "The Stream Title", "ARTIST": "Artist 1; Artist 2"... }`
  * `timestampOffset` time when is scheduled to be updated.
  * `timestamp` time when metadata was discovered on the stream.
* `onMetadata(metadata, timestampOffset, timestamp)` (optional)
  * Called when metadata is discovered on the stream.
    * ICY: `{ "StreamTitle: "The Stream Title" }`
    * Ogg: `{ "TITLE: "The Stream Title", "ARTIST": "Artist 1; Artist 2"... }`
  * `timestampOffset` time when is scheduled to be updated.
  * `timestamp` time when metadata was discovered on the stream.
* `onError(message)` (optional)
  * Called when a fallback condition or error condition is met.

### Getters
* `player.audioElement`
  * Returns the HTML5 Audio element.
* `player.icyMetaInt`
  * Returns the ICY Metadata Interval of this instance.
* `player.metadataQueue`
  * Returns the array of `metadata` objects in FILO order.
    ```
    [
      {
        metadata: { StreamTitle: "Title 1" },
        timestampOffset: 2.5,
        timestamp: 1
      },
      {
        metadata: { StreamTitle: "Title 2" },
        timestampOffset: 5,
        timestamp: 2
      }
    ]
    ```
* `player.playing`
  * Returns `true` if the IcecastMetadataPlayer is playing and `false` if it is not.

### Methods
* `player.play()`
  * Plays the Icecast Stream

* `player.stop()`
  * Stops playing the Icecast Stream

---

## Troubleshooting

### Error messages

> Passed in Icy-MetaInt is invalid. Attempting to detect ICY Metadata.

* The stream has been requested with ICY metadata, but the server did not respond with the `Icy-MetaInt` header. `IcecastMetadataReader` will attempt to detect the ICY metadata interval, and will timeout after a default of 2 seconds, or the value in milliseconds passed into the `icyDetectionTimeout` option.
* This warning could also show if the stream was requested with ICY metadata, but it does not contain ICY metadata. In this case, the ICY detection should timeout and the stream should play without ICY metadata. Please update your code to no longer request ICY metadata.

> This stream is not an OGG stream. No OGG metadata will be returned.

* IcecastMetadataReader has `"ogg"` passed into the `metadataTypes` options, but the stream response is not an ogg stream. ICY metadata and the stream will work without issues. Please remove the `"ogg"` option to remove this warning.

> Network request failed, possibly due to a CORS issue. Trying again without ICY Metadata.

* A network error occurred while requesting the stream with the `Icy-MetaData: 1` header. If you want ICY metadata, your CORS policy must allow this header to be requested. See [CORS Troubleshooting](https://github.com/eshaz/icecast-metadata-js#cors) for more information.

> Media Source Extensions API in your browser does not support `codec`, `audio/mp4; codec="codec"`

* The Media Source API in your browser does not support the audio codec of the Icecast stream. Metadata playback is currently not possible with this stream endpoint. This message should be followed up with the below message.

> Falling back to HTML5 audio with no metadata updates. See the console for details on the error.

* A general error occurred when playing the stream. IcecastMetadataPlayer should continue to play the stream, but there will be no metadata updates.
The Media Source API is a relatively new browser feature, and there can be varying support across platforms.