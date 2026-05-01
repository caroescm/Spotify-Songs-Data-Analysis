TLDR: JOINS

1. Analyzing from the six main genres, what is the average of every audio feature?

```sql
WITH unique_track_genre AS (
    SELECT DISTINCT
        t.track_id,
        p.playlist_genre,
        t.danceability, t.energy, t.valence, t.speechiness,
        t.acousticness, t.instrumentalness, t.liveness,
        t.loudness, t.tempo, t.duration_ms, t.track_popularity
    FROM tracks t
    JOIN track_playlist tp ON t.track_id = tp.track_id
    JOIN playlists p       ON tp.playlist_id = p.playlist_id
)
SELECT
    playlist_genre,
    COUNT(*) AS unique_tracks_in_genre,
    ROUND(AVG(danceability)::numeric, 3)     AS avg_danceability,
    ROUND(AVG(energy)::numeric, 3)           AS avg_energy,
    ROUND(AVG(valence)::numeric, 3)          AS avg_valence,
    ROUND(AVG(speechiness)::numeric, 3)      AS avg_speechiness,
    ROUND(AVG(acousticness)::numeric, 3)     AS avg_acousticness,
    ROUND(AVG(instrumentalness)::numeric, 4) AS avg_instrumentalness,
    ROUND(AVG(liveness)::numeric, 3)         AS avg_liveness,
    ROUND(AVG(loudness)::numeric, 2)         AS avg_loudness_db,
    ROUND(AVG(tempo)::numeric, 1)            AS avg_tempo_bpm,
    ROUND(AVG(duration_ms) / 1000.0, 1)      AS avg_duration_sec,
    ROUND(AVG(track_popularity)::numeric, 1) AS avg_popularity
FROM unique_track_genre
GROUP BY playlist_genre
ORDER BY avg_popularity DESC;
```
Results:
| playlist_genre | unique_tracks_in_genre | avg_danceability | avg_energy | avg_valence | avg_speechiness | avg_acousticness | avg_instrumentalness | avg_liveness | avg_loudness_db | avg_tempo_bpm | avg_duration_sec | avg_popularity |
|----------------|------------------------|------------------|------------|-------------|-----------------|------------------|----------------------|--------------|-----------------|---------------|------------------|----------------|
| pop            | 5132                   | 0.638            | 0.701      | 0.502       | 0.074           | 0.172            | 0.0634               | 0.177        | -6.38           | 121           | 218.1            | 45.9           |
| latin          | 4641                   | 0.711            | 0.708      | 0.6         | 0.101           | 0.211            | 0.0489               | 0.181        | -6.4            | 118.6         | 217              | 44.3           |
| rap            | 5449                   | 0.716            | 0.65       | 0.504       | 0.197           | 0.196            | 0.0799               | 0.191        | -7.08           | 120.6         | 213.4            | 42.1           |
| rock           | 4451                   | 0.52             | 0.733      | 0.535       | 0.058           | 0.147            | 0.0656               | 0.205        | -7.54           | 125.1         | 247.8            | 40.4           |
| r&b            | 4948                   | 0.672            | 0.592      | 0.536       | 0.118           | 0.259            | 0.0279               | 0.176        | -7.91           | 114.2         | 238.4            | 39             |
| edm            | 5537                   | 0.658            | 0.799      | 0.406       | 0.088           | 0.085            | 0.2202               | 0.211        | -5.5            | 125.6         | 223.5            | 34.1           |
