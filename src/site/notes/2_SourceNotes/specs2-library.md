---
{"dg-publish":true,"permalink":"/2-source-notes/specs2-library/","tags":["n/source","gardenEntry"]}
---




# specs2 library

- We need to be careful when using triple equals with `None` in specs2 `(None)`: https://github.com/soundcloud/gobbly/pull/657#discussion_r955916517

```
// the test returns error, as expected, with any of these options
trackMerger.mergedOrNewTrackToPersist(any[TrackWaitingForData], ===(None.asInstanceOf[Option[UpdateableExistingTrack]])).returns(Some(translatedTrack))
org.mockito.Mockito.when(trackMerger.mergedOrNewTrackToPersist(any[TrackWaitingForData], argThat(===(None.asInstanceOf[Option[UpdateableExistingTrack]])))).thenReturn(Some(translatedTrack))
org.mockito.Mockito.when(trackMerger.mergedOrNewTrackToPersist(any[TrackWaitingForData], org.mockito.ArgumentMatchers.eq(None))).thenReturn(Some(translatedTrack))

// the test was green - not what we expect
trackMerger.mergedOrNewTrackToPersist(any[TrackWaitingForData], ===(None)).returns(Some(translatedTrack))
org.mockito.Mockito.when(trackMerger.mergedOrNewTrackToPersist(any[TrackWaitingForData], ===(None))).thenReturn(Some(translatedTrack))
org.mockito.Mockito.when(trackMerger.mergedOrNewTrackToPersist(any[TrackWaitingForData], argThat(===(None)))).thenReturn(Some(translatedTrack))
```

## Mockito
- https://etorreborre.github.io/specs2/guide/SPECS2-4.0.0/org.specs2.guide.UseMockito.html

View the invocations

```scala

```
