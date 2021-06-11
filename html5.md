## Basic Setup

To integrate [NexPlayer™](https://nexplayersdk.com/) multiview into your project you must complete the following steps:


We will declare the containers where the videos will be stored.

```HTML
<body>
  <div id="player_container">
     <div id="player" width="530" height="315"></div>
  </div>
  <div id="global_container">
     <div id="player_container2">
         <div id="player2" width="530" height="315"></div>
     </div>
     <div id="player_container3">
         <div id="player3" width="530" height="315"></div>
     </div>
     <div id="player_container4">
         <div id="player4" width="530" height="315"></div>
     </div>
  </div>
</body>
```

To create the players we will have to call ```multiview.additionalVideo()```. It is important to call ```multiView.Initialize()```, this will allow us to start creating the players:

```javascript
var multiView = new nexplayer.MultipleView();

  var callBackWithPlayers = function (nexplayerInstance, videoElement) {
    
      };

      multiView.additionalVideo({
          key: "Your licence key",
          allowScreenPlayPause: false,
          callbacksForPlayer: callBackWithPlayers,
          src: 'Your stream URL',
      });

      multiView.additionalVideo({
          key: "Your licence key",
          allowScreenPlayPause: false,
          callbacksForPlayer: callBackWithPlayers,
          src: 'Your stream URL',
      });

      multiView.additionalVideo({
          key: "Your licence key",
          div: document.getElementById('player3'),
          callbacksForPlayer: callBackWithPlayers,
          src: 'Your stream URL',
      });

      multiView.additionalVideo({
          key: "Your licence key",
          div: document.getElementById('player4'),
          callbacksForPlayer: callBackWithPlayers,
          src: 'Your stream URL',
      });

   multiView.Initialize();
```
Please note that the library is required, you can consult the releases [here](https://nexplayer.github.io/NexPlayer_HTML5_Documentation/#/releases?id=releases-top).

You can see a more complete sample here and check our latest [demo](https://nex360.s3.amazonaws.com/MultiView/index.html). Also, in the following link you can access the [API](https://nexplayer.github.io/NexPlayer_HTML5_Documentation/#/API?id=multiview) available for MultiView.

## Advanced configuration

To be able to use the synchronization we have to configure in the setup the ```lowLatency``` and ```dashSettings```.

```javascript
  
   multiView.additionalVideo({
          key: "Your licence key",
          div: document.getElementById('player4'),
          callbacksForPlayer: callBackWithPlayers,
          lowLatency: true,
          dashSettings: {  // Optional: Allow modifying some dash properties like the following
            "liveDelay": 20,    // Allow adjusting the live delay
            "liveCatchUpPlaybackRate": 0.5, // The speed that the player gets in order to keep the live delay
            "liveCatchUpMaxDrift": 3,   // The maximun delay before to make a seek live
            "liveCatchupLatencyThreshold": 30,  // The threshold where the synchronization properties works
          }
          src: 'Your stream URL',
      });
      
```
Currently, live synchronization is only available for Dash.
