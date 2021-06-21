

# Stream Requirements

NexPlayer Multiview feature works both for HLS and DASH live streams. It should have the following specifications for optimised video playback. 

- Multiple video rendition should be provided to optimise network usage and performance. PiP players will be playing the lower renditions while the main player is playing the higher ones. An example ABR ladder might look like: 240p 25fps, 360p 30fps, 720p 30fps, 1080p 30fps.
- Segment durations should be around 2-4 seconds for a better synchronization
- For DASH following properties should be the same for all the streams
    - MPD@suggestedPresentationDelay (for Server-Side based synchronization)
    - Period Start Time
    - Frame CTS of the segments
- Video encoders should be synchronized and should be embedding correct timestamp for the video streams
- DASH is recommended over HLS as it has the overhead of manifest reload.
- For HLS, EXT-X-PROGRAM-DATE-TIME should exist.
- In case Widevine DRM is enabled, Widevine Level 3 should be allowed on the DRM Server.

