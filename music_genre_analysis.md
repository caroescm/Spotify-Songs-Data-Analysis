TLDR: JOINS

1. Getting the full picture: Analyzing from the six main genres, what is the average of every audio feature?

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

2. Characteristics per genre:
``` sql
WITH genre_stats AS (
    SELECT
        p.playlist_genre AS genre,
        ROUND(AVG(t.loudness)::numeric, 2) AS loudness,
        ROUND(AVG(t.tempo)::numeric, 1) AS tempo,
        ROUND(AVG(t.acousticness)::numeric, 3) AS acousticness,
        ROUND(AVG(t.danceability)::numeric, 3) AS danceability,
        ROUND(AVG(t.energy)::numeric, 3) AS energy,
        ROUND(AVG(t.valence)::numeric, 3) AS valence,
        ROUND(AVG(t.speechiness)::numeric, 3) AS speechiness,
        ROUND(AVG(t.instrumentalness)::numeric, 4) AS instrumentalness,
        ROUND(AVG(t.duration_ms) / 1000.0, 1) AS duration_sec,
        ROUND(AVG(t.track_popularity)::numeric, 1) AS popularity
    FROM tracks t
    JOIN track_playlist tp ON t.track_id = tp.track_id
    JOIN playlists p       ON tp.playlist_id = p.playlist_id
    GROUP BY p.playlist_genre
)
(SELECT 'Loudest genre' AS finding, genre, loudness AS value FROM genre_stats ORDER BY loudness DESC LIMIT 1) UNION ALL
(SELECT 'Quietest genre', genre, loudness FROM genre_stats ORDER BY loudness ASC LIMIT 1) UNION ALL
(SELECT 'Fastest genre (tempo)', genre, tempo FROM genre_stats ORDER BY tempo DESC LIMIT 1) UNION ALL
(SELECT 'Slowest genre (tempo)', genre, tempo FROM genre_stats ORDER BY tempo ASC LIMIT 1) UNION ALL
(SELECT 'Most acoustic genre', genre, acousticness FROM genre_stats ORDER BY acousticness DESC LIMIT 1) UNION ALL
(SELECT 'Least acoustic genre', genre, acousticness FROM genre_stats ORDER BY acousticness ASC  LIMIT 1) UNION ALL
(SELECT 'Most danceable genre', genre, danceability FROM genre_stats ORDER BY danceability DESC LIMIT 1) UNION ALL
(SELECT 'Least danceable genre', genre, danceability FROM genre_stats ORDER BY danceability ASC  LIMIT 1) UNION ALL
(SELECT 'Most energetic genre', genre, energy FROM genre_stats ORDER BY energy DESC LIMIT 1) UNION ALL
(SELECT 'Least energetic genre', genre, energy FROM genre_stats ORDER BY energy ASC LIMIT 1) UNION ALL
(SELECT 'Happiest genre (valence)', genre, valence FROM genre_stats ORDER BY valence DESC LIMIT 1) UNION ALL
(SELECT 'Saddest genre (valence)', genre, valence FROM genre_stats ORDER BY valence ASC LIMIT 1) UNION ALL
(SELECT 'Most lyric-heavy (speechiness)', genre, speechiness FROM genre_stats ORDER BY speechiness DESC LIMIT 1) UNION ALL
(SELECT 'Most instrumental', genre, instrumentalness FROM genre_stats ORDER BY instrumentalness DESC LIMIT 1) UNION ALL
(SELECT 'Longest songs', genre, duration_sec FROM genre_stats ORDER BY duration_sec DESC LIMIT 1) UNION ALL
(SELECT 'Shortest songs', genre, duration_sec FROM genre_stats ORDER BY duration_sec ASC LIMIT 1) UNION ALL
(SELECT 'Most popular genre', genre, popularity FROM genre_stats ORDER BY popularity DESC LIMIT 1) UNION ALL
(SELECT 'Least popular genre', genre, popularity FROM genre_stats ORDER BY popularity ASC LIMIT 1);
```
| finding                        | genre | value  |
|--------------------------------|-------|--------|
| Loudest genre                  | edm   | -5.45  |
| Quietest genre                 | r&b   | -7.91  |
| Fastest genre (tempo)          | edm   | 125.8  |
| Slowest genre (tempo)          | r&b   | 114.2  |
| Most acoustic genre            | r&b   | 0.26   |
| Least acoustic genre           | edm   | 0.083  |
| Most danceable genre           | rap   | 0.718  |
| Least danceable genre          | rock  | 0.52   |
| Most energetic genre           | edm   | 0.801  |
| Least energetic genre          | r&b   | 0.59   |
| Happiest genre (valence)       | latin | 0.608  |
| Saddest genre (valence)        | edm   | 0.402  |
| Most lyric-heavy (speechiness) | rap   | 0.198  |
| Most instrumental              | edm   | 0.2163 |
| Longest songs                  | rock  | 249    |
| Shortest songs                 | rap   | 214.1  |
| Most popular genre             | pop   | 47.7   |
| Least popular genre            | edm   | 35     |

4. 
