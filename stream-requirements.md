
# Stream Requirements

NexPlayer Multiview feature works both for HLS and DASH live streams. It should have the following specifications for an optimised video playback. 

- Multiple video rendition should be provided to optimise network usage and the performance. PiP players will be playing the lower renditions while the main player is playing higher ones. An example ABR ladder might look like: 240p 25fps, 360p 30fps, 720p 30fps, 1080p 30fps.

- Video encoders should be syncronised and should be embedding correct timestamp for the  video streams

- DASH is recommended over HLS as it has the overhead of manifest relaod.

- For HLS, EXT-X-PROGRAM-DATE-TIME should exist.

- In case of Widevine DRM is enabled, Widevine Level 3 should be allowed on the DRM Server.