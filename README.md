# Use case: Media Edge API timelines

This guide provides two use-case examples of media sessions tracked with the Media Edge API service, as follows: 

1. Two chapters separated by an ad break 
2. A buffer state and a pause

Media Edge APIs are built on the Adobe Experience Platform to provide media event tracking data within the framework of [XDM schemas](https://experienceleague.adobe.com/docs/experience-platform/xdm/home.html#:~:text=Experience%20Data%20Model%20(XDM)%2C,the%20power%20of%20digital%20experiences). For more information, see the [Media Edge API overview](https://experienceleague.adobe.com/docs/experience-platform/edge-network-server-api/media-edge-apis/overview.html?).


## Use case 1: two chapters separated by an ad break

This session is an example of a tracked user session that contains:

* Two chapters: `Chapter 1` and `Chapter 2`.
* An ad break inserted at the middle of the content that contains two ads: `Ad 1` and `Ad 2`.


### Table

The following table shows the user session broken down by actions. The `Seconds From Beginning (Real-Time)` and `Playhead Position` columns are both represented in seconds.

| # | Action | Seconds From Beginning (Real-Time) | Playhead Position | Client request |
| --- | --- | --- | --- | --- |
| 1 | Auto-play or Play button is pressed and the video starts loading | 0 | 0 | `/sessionStart?configId=<datastreamID>` |
| 2 | App starts the ping event timer | 0 | 0 | `/ping?configId=<datastreamID>` |
| 3 | Tracks `play` event | 0 | 0 | `/play?configId=<datastreamID>` |
| 4 | Tracks the start of `Chapter 1` | 1 | 1 | `/chapterStart?configId=<datastreamID>` |
| 5 | Sends the ping event | 10 | 10 | `/ping?configId=<datastreamID>` |
| 6 | Tracks the completion of `Chapter 2` | 15 | 15 | `/chapterComplete?configId=<datastreamID>` |
| 7 | Tracks the ad break start | 16 | 16 | `/adBreakStart?configId=<datastreamID>` |
| 8 | Tracks the `Ad 1` start | 16 | 16 | `/adStart?configId=<datastreamID>` |
| 9 | Sends the ping events | 20,30 | 20,30 | `/ping?configId=<datastreamID>` |
| 10 | Tracks the `Ad 1` complete | 31 | 31 | `/adComplete?configId=<datastreamID>` |
| 11 | Tracks the `Ad 2` start | 31 | 31 | `/adStart?configId=<datastreamID>` |
| 12 | Send the ping event | 40 | 40 | `/ping?configId=<datastreamID>` |
| 13 | Tracks the `Ad 2` complete | 43 | 43 | `/adComplete?configId=<datastreamID>` |
| 14 | Tracks the ad break as complete | 43 | 43 | `/adBreakComplete?configId=<datastreamID>` |
| 15 | Tracks the start of `Chapter 2` | 44 | 44 | `/chapterStart?configId=<datastreamID>` |
| 16 | Sends the ping event | 50 | 50 | `/ping?configId=<datastreamID>` |
| 17 | Tracks the completion of `Chapter 2` | 54 | 54 | `/chapterComplete?configId=<datastreamID>` |
| 18 | The user finishes watching the content to the end | 55 | 55 | `/sessionComplete?configId=<datastreamID>` |

#### Description

The description of each action, together with the payload sent to Media Edge API are presented below.

| # | Action | Seconds From Beginning (Real-Time) | Playhead Position | Client request |
| --- | --- | --- | --- | --- |
| 1 | Auto-play or Play button is pressed and the video starts loading | 0 | 0 | `/sessionStart?configId=<datastreamID>` |

This call signals the intention of the user to play a video. It returns a Session ID which is referenced in the following examples with `{SID}`. This `{SID}` is returned to the client, and is used to identify all subsequent tracking calls within the session. The player state is not yet `playing`, but is instead `starting`. This call also generates an reporting event that is pushed to AEP and/or Analytics, depending on datastream configuration. Mandatory parameters must be included.

```json
{
  "eventType": "media.sessionStart",
  "timestamp": "YYYY-08-01T02:00:00.000Z",
  "mediaCollection": {
    "playhead": 0,
    "sessionDetails": {
      "name": "VA API Sample Player",
      "friendlyName": "ClickMe",
      "length": 60,
      "contentType": "VOD",
      "playerName": "sample-html5-api-player",
      "channel": "sample-channel",
      "appVersion": "va-api-0.0.0"
    }
  }
}
```

| # | Action | Seconds From Beginning (Real-Time) | Playhead Position | Client request |
| --- | --- | --- | --- | --- |
| 2 | App starts the ping event timer | 0 | 0 | `/ping?configId=<datastreamID>` |

The app starts the ping timer. The first ping event should fire 10 seconds later.

| # | Action | Seconds From Beginning (Real-Time) | Playhead Position | Client request |
| --- | --- | --- | --- | --- |
| 3 | Tracks `play` event | 0 | 0 | `/play?configId=<datastreamID>` |

Enters the `playing` state using the `play` event.

```json
{
  "eventType": "media.play",
  "timestamp": "YYYY-08-01T02:00:00Z",
  "mediaCollection": {
    "sessionID": "{SID}",
    "playhead": 0
  }
}
```

| # | Action | Seconds From Beginning (Real-Time) | Playhead Position | Client request |
| --- | --- | --- | --- | --- |
| 4 | Tracks the start of `Chapter 1` | 1 | 1 | `/chapterStart?configId=<datastreamID>` |

Tracks the start of the first chapter.

```json
{
  "eventType": "media.chapterStart",
  "timestamp": "YYYY-08-01T02:00:01Z",
  "mediaCollection": {
    "sessionID": "{SID}",
    "playhead": 1,
    "chapterDetails": {
      "index": 1,
      "offset": 0,
      "friendlyName": "Chapter one",
      "length": 5
    }
  }
}
```

| # | Action | Seconds From Beginning (Real-Time) | Playhead Position | Client request |
| --- | --- | --- | --- | --- |
| 5 | Sends the ping event | 10 | 10 | `/ping?configId=<datastreamID>` |

Ping the backend every 10 seconds.

```json
{
  "eventType": "media.ping",
  "timestamp": "YYYY-08-01T02:00:10Z",
  "mediaCollection": {
    "sessionID": "{SID}",
    "playhead": 10
  }
}
```

| # | Action | Seconds From Beginning (Real-Time) | Playhead Position | Client request |
| --- | --- | --- | --- | --- |
| 6 | Tracks the completion of `Chapter 2` | 15 | 15 | `/chapterComplete?configId=<datastreamID>` |

The first chapter ends right before the ad break.

```json
{
  "eventType": "media.chapterComplete",
  "timestamp": "YYYY-08-01T02:00:15Z",
  "mediaCollection": {
    "sessionID": "{SID}",
    "playhead": 15
  }
}
```

| # | Action | Seconds From Beginning (Real-Time) | Playhead Position | Client request |
| --- | --- | --- | --- | --- |
| 7 | Tracks the ad break start | 16 | 16 | `/adBreakStart?configId=<datastreamID>` |

Ad break starts. It will contain two ads.

```json
{
  "eventType": "media.adBreakStart",
  "timestamp": "YYYY-08-01T02:00:16Z",
  "mediaCollection": {
    "sessionID": "{SID}",
    "playhead": 16,
    "advertisingPodDetails": {
      "index": 0,
      "offset": 0,
      "friendlyName": "Mid-roll"
    }
  }
}
```

| # | Action | Seconds From Beginning (Real-Time) | Playhead Position | Client request |
| --- | --- | --- | --- | --- |
| 8 | Tracks the `Ad 1` start | 16 | 16 | `/adStart?configId=<datastreamID>` |

Ad break starts. It will contain two ads. `Ad 1` begins.

```json
{
  "eventType": "media.adStart",
  "timestamp": "YYYY-08-01T02:00:16Z",
  "mediaCollection": {
    "sessionID": "{SID}",
    "playhead": 0,
    "advertisingDetails": {
      "name": "001",
      "advertiser": "Ad Guys",
      "campaignID": "1",
      "creativeID": "42",
      "creativeURL": "https://example.com",
      "length": 15,
      "friendlyName": "Ad 1",
      "placementID": "sample_placement",
      "playerName": "Sample Player",
      "podPosition": 1,
      "siteID": "XYZ"
    },
    "customMetadata": [
      {
        "name": "myCustomData1",
        "value": "CustomData1"
      },
      {
        "name": "myCustomData2",
        "value": "CustomData2"
      }
    ]
  }
}
```

| # | Action | Seconds From Beginning (Real-Time) | Playhead Position | Client request |
| --- | --- | --- | --- | --- |
| 9 | Sends the ping events | 20,30 | 20,30 | `/ping?configId=<datastreamID>` |

Ping the backend every 10 seconds (in this particular case there are two separate events sent at timestamps 20 and 30 respectively).

```json
{
  "eventType": "media.ping",
  "timestamp": "YYYY-08-01T02:00:20Z",
  "mediaCollection": {
    "sessionID": "{SID}",
    "playhead": 20
  }
}
```

```json
{
  "eventType": "media.ping",
  "timestamp": "YYYY-08-01T02:00:30Z",
  "mediaCollection": {
    "sessionID": "{SID}",
    "playhead": 30
  }
}
```

| # | Action | Seconds From Beginning (Real-Time) | Playhead Position | Client request |
| --- | --- | --- | --- | --- |
| 10 | Tracks the `Ad 1` complete | 31 | 31 | `/adComplete?configId=<datastreamID>` |

Track the end of the first ad.

```json
{
  "eventType": "media.adComplete",
  "timestamp": "YYYY-08-01T02:00:31Z",
  "mediaCollection": {
    "sessionID": "{SID}",
    "playhead": 31
  }
}
```

| # | Action | Seconds From Beginning (Real-Time) | Playhead Position | Client request |
| --- | --- | --- | --- | --- |
| 11 | Tracks the `Ad 2` start | 31 | 31 | `/adStart?configId=<datastreamID>` |

`Ad 2` begins.

```json
{
  "eventType": "media.adStart",
  "timestamp": "YYYY-08-01T02:00:31Z",
  "mediaCollection": {
    "playhead": 0,
    "sessionID": "{SID}",
    "advertisingDetails": {
      "name": "002",
      "advertiser": "Ad Guys",
      "campaignID": "2",
      "creativeID": "44",
      "creativeURL": "https://example.com",
      "length": 12,
      "friendlyName": "Ad 2",
      "placementID": "sample_placement2",
      "playerName": "Sample Player",
      "podPosition": 1,
      "siteID": "XYZ"
    }
  }
}
```

| # | Action | Seconds From Beginning (Real-Time) | Playhead Position | Client request |
| --- | --- | --- | --- | --- |
| 12 | Send the ping event | 40 | 40 | `/ping?configId=<datastreamID>` |

Pings the backend every 10 seconds.

```json
{
  "eventType": "media.ping",
  "timestamp": "YYYY-08-01T02:00:40Z",
  "mediaCollection": {
    "sessionID": "{SID}",
    "playhead": 40
  }
}
```

| # | Action | Seconds From Beginning (Real-Time) | Playhead Position | Client request |
| --- | --- | --- | --- | --- |
| 13 | Tracks the `Ad 2` complete | 43 | 43 | `/adComplete?configId=<datastreamID>` |

Tracks the end of the second ad.

```json
{
  "eventType": "media.adComplete",
  "timestamp": "YYYY-08-01T02:00:43Z",
  "mediaCollection": {
    "sessionID": "{SID}",
    "playhead": 43
  }
}
```

| # | Action | Seconds From Beginning (Real-Time) | Playhead Position | Client request |
| --- | --- | --- | --- | --- |
| 14 | Tracks the ad break as complete | 43 | 43 | `/adBreakComplete?configId=<datastreamID>` |

Tracks the end of the ad break.

```json
{
  "eventType": "media.adBreakComplete",
  "timestamp": "YYYY-08-01T02:00:43Z",
  "mediaCollection": {
    "sessionID": "{SID}",
    "playhead": 43
  }
}
```

| # | Action | Seconds From Beginning (Real-Time) | Playhead Position | Client request |
| --- | --- | --- | --- | --- |
| 15 | Tracks the start of `Chapter 2` | 44 | 44 | `/chapterStart?configId=<datastreamID>` |

Tracks the start of the second chapter right after the ad break completion.

```json
{
  "eventType": "media.chapterStart",
  "timestamp": "YYYY-08-01T02:00:44Z",
  "mediaCollection": {
    "sessionID": "{SID}",
    "playhead": 1,
    "chapterDetails": {
      "index": 2,
      "offset": 44,
      "friendlyName": "Chapter two",
      "length": 10
    }
  }
}
```

| # | Action | Seconds From Beginning (Real-Time) | Playhead Position | Client request |
| --- | --- | --- | --- | --- |
| 16 | Sends the ping event | 50 | 50 | `/ping?configId=<datastreamID>` |

Pings the backend every 10 seconds.

```json
{
  "eventType": "media.ping",
  "timestamp": "YYYY-08-01T02:00:50Z",
  "mediaCollection": {
    "sessionID": "{SID}",
    "playhead": 50
  }
}
```

| # | Action | Seconds From Beginning (Real-Time) | Playhead Position | Client request |
| --- | --- | --- | --- | --- |
| 17 | Tracks the completion of `Chapter 2` | 54 | 54 | `/chapterComplete?configId=<datastreamID>` |

Track the end of the second and final chapter.

```json
{
  "eventType": "media.chapterComplete",
  "timestamp": "YYYY-08-01T02:00:54Z",
  "mediaCollection": {
    "sessionID": "{SID}",
    "playhead": 54
  }
}
```

| # | Action | Seconds From Beginning (Real-Time) | Playhead Position | Client request |
| --- | --- | --- | --- | --- |
| 18 | The user finishes watching the content to the end | 55 | 55 | `/sessionComplete?configId=<datastreamID>` |

Sends `sessionComplete` to the backend to indicate that the user finished watching the entire content.

```json
{
  "eventType": "media.sessionComplete",
  "timestamp": "YYYY-08-01T02:00:55Z",
  "mediaCollection": {
    "sessionID": "{SID}",
    "playhead": 55
  }
}
```

## Session with a buffer state and a pause

This is an example of a tracked session that contains both a `buffering` state and content that is paused. Note how as opposed to the `Session with 2 chapters separated by an ad break` example, in this example the `Seconds From Beginning (Real-Time)` time differs from the `Playhead Position`.

### Table

The following table shows the user session broken down by actions. The `Seconds From Beginning (Real-Time)` and `Playhead Position` columns are both represented in seconds.

| # | Action | Seconds From Beginning (Real-Time) | Playhead Position | Client Request |
| --- | --- | --- | --- | --- |
| 1 | Auto-play or Play button is pressed and the video starts loading | 0 | 0 | `/sessionStart?configId=<datastreamID>` |
| 2 | App starts the ping event timer | 0 | 0 | `/ping?configId=<datastreamID>` |
| 3 | Tracks the buffer start  | 1 | 1 | `/bufferStart?configId=<datastreamID>` |
| 4 | Tracks the buffer end and sends play event | 4 | 1 | `/play?configId=<datastreamID>` |
| 5 | Sends the ping event | 10 | 7 | `/ping?configId=<datastreamID>` |
| 6 | The user presses `pause` | 15 | 12 | `/pauseStart?configId=<datastreamID>` |
| 7 | Sends the ping event | 20 | 12 | `/ping?configId=<datastreamID>` |
| 8 | The user presses `play` to resume the main content | 24 | 12 | `/play?configId=<datastreamID>` |
| 9 | The user closes the app without watching the content to the end | 29 | 17 | `/sessionEnd?configId=<datastreamID>` |

#### Description

The description of each action, together with the payload sent to Media Edge API are presented below.

| # | Action | Seconds From Beginning (Real-Time) | Playhead Position | Client Request |
| --- | --- | --- | --- | --- |
| 1 | Auto-play or Play button is pressed and the video starts loading | 0 | 0 | `/sessionStart?configId=<datastreamID>` |

This call signals the intention of the user to play a video. The player state is not yet `playing` but is instead `starting`. It returns a Session ID referenced in the following examples with `{SID}` to the client. This `{SID}` is used to identify all subsequent tracking calls within the session. This call generates an reporting event that is pushed to AEP and/or Analytics, depending on datastream configuration. Mandatory params must be included.

```json
{
 "eventType": "media.sessionStart",
  "timestamp": "YYYY-08-01T02:00:00.000Z",
  "mediaCollection": {
    "playhead": 0,
    "sessionDetails": {
      "name": "VA API Sample Player",
      "friendlyName": "ClickMe",
      "length": 60,
      "contentType": "VOD",
      "playerName": "sample-html5-api-player",
      "channel": "sample-channel",
      "appVersion": "va-api-0.0.0"
    }
  }
}
```

| # | Action | Seconds From Beginning (Real-Time) | Playhead Position | Client Request |
| --- | --- | --- | --- | --- |
| 2 | App starts the ping event timer | 0 | 0 | `/ping?configId=<datastreamID>` |

Starts your ping timer. The first ping event should fire 10 seconds later.

| # | Action | Seconds From Beginning (Real-Time) | Playhead Position | Client Request |
| --- | --- | --- | --- | --- |
| 3 | Tracks the buffer start  | 1 | 1 | `/bufferStart?configId=<datastreamID>` |

Player enters the `buffering` state. This is a state in which content is not being played therefore the playhead is not advancing.

```json
{
  "eventType": "media.bufferStart",
  "timestamp": "YYYY-08-01T02:00:01Z",
  "mediaCollection": {
    "sessionID": "{SID}",
    "playhead": 0
  }
}
```

| # | Action | Seconds From Beginning (Real-Time) | Playhead Position | Client Request |
| --- | --- | --- | --- | --- |
| 4 | Tracks the buffer end and sends play event | 4 | 1 | `/play?configId=<datastreamID>` |

Buffering ends after 3 seconds so the player is sent into the `playing` state. This is necessary to mark exiting the buffering state.
Sending a `play` call after the `bufferStart` call has been sent infers a `bufferEnd` state. This makes it so there is no need for a separate `bufferEnd` event.

```json
{
  "eventType": "media.play",
  "timestamp": "YYYY-08-01T02:00:04Z",
  "mediaCollection": {
    "sessionID": "{SID}",
    "playhead": 1
  }
}
```

| # | Action | Seconds From Beginning (Real-Time) | Playhead Position | Client Request |
| --- | --- | --- | --- | --- |
| 5 | Sends the ping event | 10 | 7 | `/ping?configId=<datastreamID>` |

Pings the backend every 10 seconds.

```json
{
  "eventType": "media.ping",
  "timestamp": "YYYY-08-01T02:00:10Z",
  "mediaCollection": {
    "sessionID": "{SID}",
    "playhead": 7
  }
}
```

| # | Action | Seconds From Beginning (Real-Time) | Playhead Position | Client Request |
| --- | --- | --- | --- | --- |
| 6 | The user presses `pause` | 15 | 12 | `/pauseStart?configId=<datastreamID>` |

The user pauses the video. This moves the play state to `paused`.

```json

{
  "eventType": "media.pauseStart",
  "timestamp": "YYYY-08-01T02:00:15Z",
  "mediaCollection": {
    "sessionID": "{SID}",
    "playhead": 12
  }
}
```

| # | Action | Seconds From Beginning (Real-Time) | Playhead Position | Client Request |
| --- | --- | --- | --- | --- |
| 7 | Sends the ping event | 20 | 12 | `/ping?configId=<datastreamID>` |

Pings the backend every 10 seconds. The player remains in a `paused` state.

```json
{
  "eventType": "media.ping",
  "timestamp": "YYYY-08-01T02:00:20Z",
  "mediaCollection": {
    "sessionID": "{SID}",
    "playhead": 12
  }
}
```

| # | Action | Seconds From Beginning (Real-Time) | Playhead Position | Client Request |
| --- | --- | --- | --- | --- |
| 8 | The user presses `play` to resume the main content | 24 | 12 | `/play?configId=<datastreamID>` |

The user plays the media. This moves the play state to `playing`. The `play` call after a `pauseStart` call infers a `resume` call to the back end. This makes it so there is no need for a separate `resume` event.

```json
{
  "eventType": "media.play",
  "timestamp": "YYYY-08-01T02:00:24Z",
  "mediaCollection": {
    "sessionID": "{SID}",
    "playhead": 12
  }
}
```

| # | Action | Seconds From Beginning (Real-Time) | Playhead Position | Client Request |
| --- | --- | --- | --- | --- |
| 9 | The user closes the app without watching the content to the end | 29 | 17 | `/sessionEnd?configId=<datastreamID>` |

The user closes the app. Send `sessionEnd` to the Media Edge API to signal that the session should be closed immediately, with no further processing.

```json
{
  "eventType": "media.sessionEnd",
  "timestamp": "YYYY-08-01T02:00:29Z",
  "mediaCollection": {
    "sessionID": "{SID}",
    "playhead": 17
  }
}
```
