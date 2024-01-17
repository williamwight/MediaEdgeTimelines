# Media Edge API use case: Two chapters separated by an ad break

This guide provides a use case for tracking a media session with the Media Edge API service. The example session shown for this use case contains the following elements:

* Two chapters: `Chapter 1` and `Chapter 2`.
* An ad break inserted at the middle of the content that contains two ads: `Ad 1` and `Ad 2`.

To view a use case that includes a buffer state and a pause, see [Media Edge API use case: Buffer state and a pause](https://experienceleague.adobe.com).

Media Edge APIs are built on the Adobe Experience Platform to provide media event tracking data within the framework of [XDM schemas](https://experienceleague.adobe.com/docs/experience-platform/xdm/home.html#:~:text=Experience%20Data%20Model%20(XDM)%2C,the%20power%20of%20digital%20experiences). For more information, see the [Media Edge API overview](https://experienceleague.adobe.com/docs/experience-platform/edge-network-server-api/media-edge-apis/overview.html).

## Use case introduction

For this session, you will need to make an API request for each action that you want to track. After [configuring a datastream](https://experienceleague.adobe.com/docs/experience-platform/datastreams/configure.html), you can begin tracking a session by providing the data stream ID in the following request:

POST `https://edge.adobedc.net/ee-pre-prd/va/v1/sessionStart?configId={dataStreamID}`

You can also specify session details as part of this request, including the name, length, content type, player name, channel, and app version.

### Example request to start tracking a session

The following example request shows how to start tracking a session and how to specify session details in a request:

```curl
curl -i --request POST '{uri}/ee/va/v1/sessionStart?configId={dataStreamId}' \
--header 'Content-Type: application/json' \
--data-raw '{
  "events": [
    {
      "xdm": {
        "eventType": "media.sessionStart",
        "timestamp": "YYYY-08-01T02:00:00.000Z",
        "mediaCollection": {
          "playhead": 0,
          "sessionDetails": {
            "name": "API Example Player",
            "length": 60,
            "contentType": "VOD",
            "playerName": "example-html5-api-player",
            "channel": "example-channel",
            "appVersion": "va-api-0.0.0"
          }
        }
      }
    }
  ]
}'
```

Each subsequent request is made in the same manner, but with changes to the endpoint URI and payload to match the action.

### Timeline of actions

The following table shows a timeline of actions to be tracked for this use case. Each row summarizes the action and the request endpoint. Each action is described in more detail with payloads below the table. Note that the **playhead position**, (the current position indicated in the horizontal timeline of the video) is not advanced during buffering or pausing, even though real-time has elapsed. Both of these are measured in seconds.

Note that for tracking you must fire ping events every 10 real-time seconds, beginning after 10 real-time seconds have elapsed. This must happen regardless of other API events that you have sent. 

| # | Action | Elapsed Real-Time (from beginning) | Playhead Position | Client request |
| --- | --- | --- | --- | --- |
| 1 | The auto-play function occurs, or the Play button is pressed, and the video starts loading. | 0 | 0 | `/sessionStart?configId=<datastreamID>` |
| 2 | The ping event timer starts | 0 | 0 | `/ping?configId=<datastreamID>` |
| 3 | Tracks the `play` event | 0 | 0 | `/play?configId=<datastreamID>` |
| 4 | Tracks the start of `Chapter 1` | 1 | 1 | `/chapterStart?configId=<datastreamID>` |
| 5 | Sends a ping | 10 | 10 | `/ping?configId=<datastreamID>` |
| 6 | Tracks the completion of `Chapter 1` | 15 | 15 | `/chapterComplete?configId=<datastreamID>` |
| 7 | Tracks the start of ad break | 16 | 16 | `/adBreakStart?configId=<datastreamID>` |
| 8 | Tracks the start of `Ad 1` in ad break | 16 | 16 | `/adStart?configId=<datastreamID>` |
| 9 | Sends a ping twice during `Ad 1`, each 10 seconds apart | 20,30 | 20,30 | `/ping?configId=<datastreamID>` |
| 10 | Tracks completion of `Ad 1` | 31 | 31 | `/adComplete?configId=<datastreamID>` |
| 11 | Tracks the start of `Ad 2` in ad break | 31 | 31 | `/adStart?configId=<datastreamID>` |
| 12 | Sends ping | 40 | 40 | `/ping?configId=<datastreamID>` |
| 13 | Tracks completion of `Ad 2` | 43 | 43 | `/adComplete?configId=<datastreamID>` |
| 14 | Tracks completion of ad break | 43 | 43 | `/adBreakComplete?configId=<datastreamID>` |
| 15 | Tracks the start of `Chapter 2` | 44 | 44 | `/chapterStart?configId=<datastreamID>` |
| 16 | Sends ping | 50 | 50 | `/ping?configId=<datastreamID>` |
| 17 | Tracks completion of `Chapter 2`| 54 | 54 | `/chapterComplete?configId=<datastreamID>` |
| 18 | User finishes watching the content to the end | 55 | 55 | `/sessionComplete?configId=<datastreamID>` |

#### Description

The description of each action, together with the payload sent to Media Edge API are presented below.

| # | Action | Elapsed Real-Time (from beginning) | Playhead Position | Client request |
| --- | --- | --- | --- | --- |
| 1 | The auto-play function occurs or Play button is pressed and the video starts loading | 0 | 0 | `/sessionStart?configId=<datastreamID>` |

This call signals the intention of the user to play a video. The player state is not yet `playing`, but is instead `starting`. This call returns a Session ID which is referenced in the following examples with `{SID}`. The `{SID}`, is returned to the client and is used to identify all subsequent tracking calls within the session.  This call also generates a reporting event that is pushed to AEP and/or Analytics, depending on datastream configuration. Mandatory parameters must be included.

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

| # | Action | Elapsed Real-Time (from beginning) | Playhead Position | Client request |
| --- | --- | --- | --- | --- |
| 2 | The app starts the ping event timer | 0 | 0 | `/ping?configId=<datastreamID>` |

The application starts the ping timer. A call is not sent for this event, but the first ping call should be fired 10 seconds later.

| # | Action | Elapsed Real-Time (from beginning) | Playhead Position | Client request |
| --- | --- | --- | --- | --- |
| 3 | The `play` event is tracked | 0 | 0 | `/play?configId=<datastreamID>` |

Tracking enters the `playing` state using the `play` event.

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

| # | Action | Elapsed Real-Time (from beginning) | Playhead Position | Client request |
| --- | --- | --- | --- | --- |
| 4 | The start of `Chapter 1` is tracked | 1 | 1 | `/chapterStart?configId=<datastreamID>` |

Tracks the start `Chapter 1`.

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

| # | Action | Elapsed Real-Time (from beginning) | Playhead Position | Client request |
| --- | --- | --- | --- | --- |
| 5 | The ping event is sent | 10 | 10 | `/ping?configId=<datastreamID>` |

A ping call is sent to the backend every 10 seconds.

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

| # | Action | Elapsed Real-Time (from beginning) | Playhead Position | Client request |
| --- | --- | --- | --- | --- |
| 6 | The completion of `Chapter 1` is tracked | 15 | 15 | `/chapterComplete?configId=<datastreamID>` |

`Chapter 1` ends directly before the ad break.

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

| # | Action | Elapsed Real-Time (from beginning) | Playhead Position | Client request |
| --- | --- | --- | --- | --- |
| 7 | The ad break start is tracked | 16 | 16 | `/adBreakStart?configId=<datastreamID>` |

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

| # | Action | Elapsed Real-Time (from beginning) | Playhead Position | Client request |
| --- | --- | --- | --- | --- |
| 8 | The `Ad 1` start is tracked | 16 | 16 | `/adStart?configId=<datastreamID>` |

`Ad 1` begins to play.

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

| # | Action | Elapsed Real-Time (from beginning) | Playhead Position | Client request |
| --- | --- | --- | --- | --- |
| 9 | The ping events are sent | 20,30 | 20,30 | `/ping?configId=<datastreamID>` |

A ping call is sent to the backend every 10 seconds. (in this particular case there are two separate events sent at timestamps 20 and 30 respectively).

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

| # | Action | Elapsed Real-Time (from beginning) | Playhead Position | Client request |
| --- | --- | --- | --- | --- |
| 10 | The `Ad 1` completion is tracked | 31 | 31 | `/adComplete?configId=<datastreamID>` |

The completion of `Ad 1` is tracked.

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

| # | Action | Elapsed Real-Time (from beginning) | Playhead Position | Client request |
| --- | --- | --- | --- | --- |
| 11 | The `Ad 2` start is tracked | 31 | 31 | `/adStart?configId=<datastreamID>` |

`Ad 2` begins to play.

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

| # | Action | Elapsed Real-Time (from beginning) | Playhead Position | Client request |
| --- | --- | --- | --- | --- |
| 12 | The ping event is sent | 40 | 40 | `/ping?configId=<datastreamID>` |

A ping call is sent to the backend every 10 seconds.

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

| # | Action | Elapsed Real-Time (from beginning) | Playhead Position | Client request |
| --- | --- | --- | --- | --- |
| 13 | The `Ad 2` completion is tracked | 43 | 43 | `/adComplete?configId=<datastreamID>` |

The completion of `Ad 2` is tracked.

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

| # | Action | Elapsed Real-Time (from beginning) | Playhead Position | Client request |
| --- | --- | --- | --- | --- |
| 14 | The ad break is tracked as complete | 43 | 43 | `/adBreakComplete?configId=<datastreamID>` |

The completion of the ad break is tracked.

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

| # | Action | Elapsed Real-Time (from beginning) | Playhead Position | Client request |
| --- | --- | --- | --- | --- |
| 15 | The start of `Chapter 2` is tracked | 44 | 44 | `/chapterStart?configId=<datastreamID>` |

The start of `Chapter 2` is tracked directly after the completion of the ad break.

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

| # | Action | Elapsed Real-Time (from beginning) | Playhead Position | Client request |
| --- | --- | --- | --- | --- |
| 16 | The ping event is sent | 50 | 50 | `/ping?configId=<datastreamID>` |

A ping call is sent to the backend every 10 seconds.

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

| # | Action | Elapsed Real-Time (from beginning) | Playhead Position | Client request |
| --- | --- | --- | --- | --- |
| 17 | The completion of `Chapter 2` is tracked | 54 | 54 | `/chapterComplete?configId=<datastreamID>` |

The completion of `Chapter 2` is tracked.

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

| # | Action | Elapsed Real-Time (from beginning) | Playhead Position | Client request |
| --- | --- | --- | --- | --- |
| 18 | The user finishes watching the content to the end | 55 | 55 | `/sessionComplete?configId=<datastreamID>` |

`sessionComplete` is sent to the backend to indicate that the user finished watching the entire content.

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
