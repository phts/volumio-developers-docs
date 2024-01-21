---
sidebar_position: 2
---

# WebSocket APIs

### Introduction

The most used API transport in Volumio2 is its Websockets API as it allows almost real time communication with multiple clients. Volumio's WebUI gets and sends data (almost) exclusively via WS. Volumio's WS layer is powered by [Socket.io](http://socket.io/).
The WebSocket API interface is located at: https://github.com/volumio/Volumio2/blob/master/app/plugins/user_interfaces/websocket/index.js

### Scenarios

Websocket communication in Volumio is identifiable in the most basic server/client architecture. The Server is Volumio itself (aka the host where Volumio is running), the client can be one or more WebUIs or other consumers (Apps and so on). In some cases, Volumio hosts can also act as client, to communicate with other hosts on the same network.

### Events
Socket.io allows events triggered by other events, emit and receive communications (on its most basic implementation). As an example, defining which event should be invoked on a client connection  looks like:

```js
self.libSocketIO.on('connection', function (connWebSocket) {
  // use connWebSocket here
});
```

This way, we can define what event should be triggered when a particular message is received:

```js
connWebSocket.on('bringmepizza', function () {
  givehimpizza();
});
```

Typically, every message we send or receive to Volumio's Backend will have this structure:

```js
io.emit('message','data');
```

Where message can be for example "play" and data can be the song number.
A good policy for sending data on emits is to configure them as objects: they're easier to parse and easily extendable.
So our message can be:

```js
io.emit('addToPlaylist', {"name": "Music", "service": "mpd", "uri": "music-library/..."});
```
### Events Documentation

### Basic Playback Commands
```
**Play:** play
**Pause:** pause
**Stop:** stop
**Previous:** prev
**Next:** next
**Seek** seek N (N is the time in seconds that the playback will keep)
**Random** setRandom({"value":true|false})
**repeat** setRepeat({"value":true|false})
```

### Get Player State

```
getState
```

Reply:

```
pushState
```

```json
{
  "status": "stop",
  "position": 0,
  "title": "Matilda Mother",
  "artist": "Pink Floyd",
  "album": "The Piper At The Gates Of Dawn",
  "albumart": "/albumart?web=Pink%20Floyd/The%20Piper%20At%20The%20Gates%20Of%20Dawn/extralarge&path=%2FNAS%2FHi_Res_Music%2FPINK%20FLOYD%20Discovery%20Studio%20Album%20Box%20Set%20(2011)%20FLAC%2F1967%20The%20Piper%20At%20The%20Gates%20Of%20Dawn",
  "uri": "mnt/NAS/Hi_Res_Music/PINK FLOYD Discovery Studio Album Box Set (2011) FLAC/1967 The Piper At The Gates Of Dawn/03 - Matilda Mother.flac",
  "trackType": "flac",
  "seek": 0,
  "duration": 189,
  "random": false,
  "repeat": false,
  "repeatSingle": false,
  "volume": 39,
  "mute": false,
  "stream": false,
  "updatedb": false,
  "volatile": false,
  "service": "mpd"
}

```

Where

* **status** is the status of the player
* **position** is the position in the play queue of current playing track (if any)
* **title** is the item's title
* **artist** is the item's artist
* **album** is the item's album
* **albumart** the URL of AlbumArt (via last.fm APIs)
* **uri** is the track's unique uri
* **trackType** is the track's type: e.g. mp3, flac, spotify etc
* **seek** is the item's current elapsed time
* **duration** is the item's duration, if any
* **random** if true, random mode is enabled
* **repeat** if true, repeat mode is enabled
* **repeatSingle** if true, repeat single mode is enabled (song is replayed in a cycle)
* **volume** current Volume
* **mute** if true, Volumio is muted
* **stream** if true, Volumio is playing a stream (webradio)
* **updatedb** if true, Volumio is updating its internal music database
* **volatile** if true, Volumio is in Volatile mode (analog input)
* **samplerate** current samplerate
* **bitdepth** bitdepth
* **channels** mono or stereo
* **service** current playback service (mpd, spop...)

### Search

```
search {value:'query'}
```

Where query is my search query. (note that for using live search, DO NOT send queries with less than 3 characters, they will dramatically slow search operations).

### Volume
Set to percentage, raise or lower, mute or unmute.

Message: *volume*

Data:

* numeric value between 0 and 100
* *mute*
* *umute*
* *+*
* *-*

**Example**
```js
io.emit('volume', 90);
io.emit('volume', '+');
```

### Mute
Message: *mute*

**Example**
```js
io.emit('mute', '');
```

### Unmute
Message: *unmute*

**Example**
```js
io.emit('unmute', '');
```

### Multiroom
```
getMultiRoomDevices
```

Retrieves all devices connected to the same network.
Input: None

Output (through pushMultiRoomDevices socket.io event):

```json
   {
      "misc": {"debug": true},
      "list": [
         {
				"id":"uuid",
				"host":"",
				"name":"",
				"isSelf":true|false,
				"state": {
					"status": "",
					"volume": 0,
					"mute": true|false,
					"artist": "",
					"track": ""
				},
                                {
				"id":"uuid",
				"host":"",
				"name":"",
				"isSelf":true|false,
				"state": {
					"status": "",
					"volume": 0,
					"mute": true|false,
					"artist": "",
					"track": ""
				}
      ]
   }
```



### Browse Music Library

```
browseLibrary objBrowseParameters
```

Where `objBrowseParameters` are the parameters we want to dig into. This returns the desired level in the music library along with navigation and pagination informations.

```js
{
  navigation: {
    prev: {
      uri: '',
    },
    lists: [
      {
        title: 'any list title, e.g Albums',
        availableListViews: ['grid', 'list'],
        items: [
          { service: 'mpd', type: 'song', title: 'track a', artist: 'artist a', album: 'album', icon: 'music', uri: 'uri' },
          { type: 'folder', title: 'folder a', icon: 'folder-open-o', uri: 'uri' },
          { type: 'folder', title: 'folder b', albumart: '//ip/image', uri: 'uri2' },
          { type: 'playlist', title: 'playlist', icon: 'bars', uri: 'uri4' },
        ],
      },
      /* ...more lists... */
    ],
  },
}
```

The browsable items can be;

- Track
- Folder (can also be a category)
- Playlist

Their parameters are:

- Type: track, folder, category
- Title: If this is a song: title, if folder or category is their name.
- Artist and Album: used only if the type is song
- Icon or image: Select the icon to display (naming of [Font-Awesome](https://fortawesome.github.io/Font-Awesome/icons/) ) , or image (URL served by Volumio Backend or external service)
- Uri: Uri

#### Sorting

##### Modern

There is a list property `availableSortings` which contains info about available sorting methods.
Each sorting method has a label, URIs for ascending and descending order and a flag `active` if the
corresponding method is used for the current response.

```js
{
  navigation: {
    lists: [
      {
        availableSortings: [
          {
            label: 'Method 1, e.g. A-Z',
            asc: {
              uri: 'uri-to-sort-by-az-asc',
              active: true, // determines if this sorting is currently used for this response
            },
            desc: {
              uri: 'uri-to-sort-by-az-desc',
              active: false,
            },
          },
          {
            label: 'Method 2, e.g. Date Added',
            asc: {
              uri: 'uri-to-sort-by-dateadded-asc',
              active: false,
            },
            desc: {
              uri: 'uri-to-sort-by-dateadded-desc',
              active: false,
            },
          },
          {
            label: 'Method 3, e.g. Artist',
            asc: {
              uri: 'uri-to-sort-by-artist-asc',
              active: false,
            },
            desc: {
              uri: 'uri-to-sort-by-artist-desc',
              active: false,
            },
          },
          /* ...any other sorting methods... */
        ],
        // ...other list props...
        title: 'any list title, e.g Albums',
        items: [ /* sorted items */ ],
      },
    ],
  },
}
```

###### Persistent sorting

Server returns navigation items with an uri with a special placeholder, e.g. on root page:

```json
{
  "navigation": {
    "lists": [
      {
        //...
        "items": [
          {
            "title": "My albums",
            "uri": "qobuz://myalbums/{{qobuz-sort-myalbums}}"
            //...
          }
        ]
      }
    ]
  }
}
```

If the client does not support persistent sorting, then it navigates to this uri without any modification as is, and the server treats it as a view with default sorting.

If the client supports persistent sorting, then before navigating into this uri, it replaces this placeholder with the stored value (e.g. in `localStorage`) which received from the server on "sort" action.

Sort action returns a response with the following properties:

```json
{
  "navigation": {
    //...
  },
  "uriPlaceholder": "any value which required by the server to replicate the sort",
  "uriPlaceholderKey": "any unique key, e.g. qobuz-sort-myalbums"
}
```

Those properties should be stored by client to be able to send them back inside corresponding uri's.

##### Legacy

Legacy sorting uses a workaround which returned 2 lists in the `browseLibrary` response.
First list contains sorting methods as list items, which have their own uri, albumart, etc.
Second list contains library items which were requested for.

```js
{
  navigation: {
    lists: [
      {
        items: [
          {
            type: 'item-no-menu',
            title: 'Method 1, e.g. A-Z',
            albumart: 'icon-for-az-button.png',
            uri: 'uri-to-sort-by-az',
          },
          {
            type: 'item-no-menu',
            title: 'Method 2, e.g. Date Added',
            albumart: 'icon-for-dateadded-button.png',
            uri: 'uri-to-sort-by-dateadded',
          },
          {
            type: 'item-no-menu',
            title: 'Method 3, e.g. Artist',
            albumart: 'icon-for-artist-button.png',
            uri: 'uri-to-sort-by-artist',
          },
          /* ...any other sorting methods... */
        ],
        /* ...other list props... */
      },
      {
        type: '',
        title: 'any list title', // legacy sorting used this field to show current sorting method name
        pageTitle: 'any list title, e.g Albums', // used for backward compatibility only when legacy and modern sortings are used in the same response
        items: [ /* sorted items */ ],
        /* ...other list props... */
      },
    ],
  },
}
```

### Get Music Library Available filters

```
getBrowseFilters
```

This returns available filters (browse by)

```js
{name:'Genres by Name', index: 'index:Genres by Name'},
{name:'Artists by Name', index: 'index:Artists by Name'},
{name:'Albums by Name', index: 'index:Albums by Name'},
{name:'Albums by Artist', index: 'index:Albums by Artist'},
{name:'Tracks by Name', index: 'index:Tracks by Name'}
```

### Get Music Sources
```
getBrowseSources
```

This returns a list of available Music Sources

```js
{name:'USB', uri: 'usb'},
{name:'NAS', uri: 'nas'},
{name:'Web Radio', uri: 'web-radio'},
{name:'Spotify', uri: 'spotify'}
```

### Custom Browse Source

This can be useful when creating a new plugin, to inject custom views in the browse sources panel, along with top-level custom actions.
```js
{
  "name": "Custom Source",
  "pluginName": "streaming_controller",
  "pluginType": "music_service",
  "uri": "stream",
  "info": "Additional info",
  "menuItems": [
    {
      "name": "play",
      "socketCall": {
        "emit": "callMethod",
        "payload": {
          "endpoint": "music_service/streaming_controller",
          "method": "launchStream",
          "data": ""
        }
      }
    },
    {
      "name": "rip",
      "socketCall": {
        "emit": "callMethod",
        "payload": {
          "endpoint": "music_service/streaming_controller",
          "method": "updateStream",
          "data": ""
        }
      }
    },
    {
      "name": "eject",
      "socketCall": {
        "emit": "callMethod",
        "payload": {
          "endpoint": "music_service/streaming_controller",
          "method": "refreshStream",
          "data": ""
        }
      }
    }
  ]}
```

## Play Queue Controls


### Get Current Play Queue

```
getQueue
```

Response:
```
pushQueue
```

```js
[ { uri: 'http://yp.shoutcast.com/sbin/tunein-station.m3u?id=830692',
    title: 'ANTENA1 - 94 7 FM',
    service: 'webradio',
    name: 'ANTENA1 - 94 7 FM',
    albumart: '/albumart',
    samplerate: '',
    bitdepth: '',
    channels: 0,
    trackType: 'webradio' },
  { uri: 'mnt/NAS/FLAC/Muse - Black Holes And Revelations - FLAC - HellraiserRG/02 - Starlight.flac',
    service: 'mpd',
    name: 'Starlight',
    artist: 'Muse',
    album: 'Black Holes And Revelations',
    type: 'track',
    tracknumber: 0,
    albumart: '/albumart?web=Muse/Black%20Holes%20And%20Revelations/extralarge&path=%2FNAS%2FFLAC%2FMuse%20-%20Black%20Holes%20And%20Revelations%20-%20FLAC%20-%20HellraiserRG',
    duration: 240,
    samplerate: '44.1 KHz',
    bitdepth: '16 bit',
    trackType: 'flac',
    channels: 2 }]
```


### Remove Item from queue

```
removeFromQueue {value: N}
```

where `N` is the track number in the queue, 0 for the first, 9 for the tenth and so on

Response:
```
pushQueue
```

### Add Item to Queue

```
addToQueue {uri:'uri'}
```

where `uri` is the uri of the item we want to add

### Move a queue item

```
moveQueue {from:N,to:N2}
```

Where N is the track number we want to move, and N2 is its new position

If we want to add an individual track from a .cue file:

```
addPlayCue {uri:'uriofsong',number:3}
```

### Favourites handling

Available functionality:

- addToFavourites
- removeFromFavourites

#### addToFavourites

This method adds a song to the favourites

Input:

```json
   {
    "uri":"my_uri/...",
    "title":"my song",
    "service":"my_service"
   }
```

Output:

```json
   {
    "success":true|false
    "reason":"failure details"
   }
```

The reason field is set only if success is false

#### removeFromFavourites

This method removes all occurrences of a song from the favourites

Input:

```json
   {
    "uri":"my_uri/...",
    "service":"my_service"
   }
```

Output:

```json
   {
    "success":true|false
    "reason":"failure details"
   }
```

The reason field is set only if success is false

### Playlist handling

Available functionality:

- createPlaylist
- deletePlaylist
- listPlaylist
- addToPlaylist
- removeFromPlaylist
- playPlaylist
- enqueue

#### createPlaylist

This method creates a new playlist

Input:

```json
   {
    "name":"myplaylist"
   }
```

Output:

```json
   {
    "success":true|false
    "reason":"failure details"
   }
```

The reason field is set only if success is false

#### deletePlaylist

This method deletes a playlist

Input:

```json
   {
    "name":"myplaylist"
   }
```

Output:

```json
   {
    "success":true|false
    "reason":"failure details"
   }
```

The reason field is set only if success is false

#### listPlaylist

This method lists all playlists in the system

Input: None

Output (through event pushListPlaylist):

```json
   [
     "playlistA",
     "playlistB",
     ...
   ]
```

The reason field is set only if success is false

#### addToPlaylist

This method adds a song to an existing playlist

Input:

```json
   {
    "name":"my playlist",
    "service":"mpd",
    "uri":"USB/..."
   }
```

Output:

```json
   {
    "success":true|false
    "reason":"failure details"
   }
```

The reason field is set only if success is false

#### removeFromPlaylist

This method removes all occurrences of a song from an existing playlist

Input:

```json
   {
    "name":"my playlist",
    "uri":"USB/..."
   }
```

Output:

```json
   {
    "success":true|false
    "reason":"failure details"
   }
```

The reason field is set only if success is false

#### playPlaylist

This method clears the queue, adds the playlist and play

Input:

```json
   {
    "name":"my playlist"
   }
```

Output:

```json
   {
    "success":true|false
    "reason":"failure details"
   }
```

The reason field is set only if success is false

#### enqueue

This method enqueue all songs of a playlist

Input:

```json
   {
    "name":"my playlist"
   }
```

Output:

```json
   {
    "success":true|false
    "reason":"failure details"
   }
```

The reason field is set only if success is false



### CallMethod on Plugin

Each method of a plugin can be execute through a websocket call. As of now there's no ACL or any security feature but thi s will change in the future. To execute a method the following socket.io command shall be issued:

```
callMethod
```

The payload shall be a json with the following structure:
```json
   {
    "endpoint":"category/name",
    "method":"methodName",
    "data": {}
   }
```

where:

* **endpoint** is a string used to target the plugin. Its structure is a linux path-like string containing the plugin category, a slash and the plugin name. An example: endpoint:'music_service/spop'.
* **method** is a string containing the name of the method to be executed.
* **data** is a complex value (can be a string or a  Json) and is passed as is to the method.

**IMPORTANT:** There should be no "-" in this call, due to FE parsing method (it converts / to -). So plugins and functions should not contain "-".

Once the method returns, the result is pushed back to the client with the event 'pushMethod'.

## Miscellaneous

### Sleep & Alarm Clock

```
getSleep
```

Triggers :

```
pushSleep {enabled:true|false, time:hh:mm:}
```

To set sleep mode:

```
setSleep {enabled:true|false, time:hh:mm:}
```

```
getAlarms
```

Triggers:

```
pushAlarms {[{id:1,enabled:true, time:hh:mm, playlist:uriplaylist},{id:2,enabled:true, time:hh:mm, playlist:uriplaylist}]}
```

To add a new alarm:

```
addAlarm {time:hh:mm, playlist:uriplaylist}
```
When a new Playlist gets added, the Values enabled:true and id (as progressive numbering) are added by default by the Backend.

To edit an alarm:

```
setAlarm {id:1,enabled:true, time:hh:mm, playlist:uriplaylist}
```
Those values will replace the values of the correspondent playlist id.

To remove an alarm:

```
removeAlarm {id:3}
```
