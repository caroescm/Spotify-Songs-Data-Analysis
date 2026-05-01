1. **Getting the full picture: Analyzing from the six main genres, what is the average of every audio feature?**
<details>
<summary>Click to see SQL code</summary>
    
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
    ROUND(AVG(danceability)::numeric, 3) AS avg_danceability,
    ROUND(AVG(energy)::numeric, 3) AS avg_energy,
    ROUND(AVG(valence)::numeric, 3) AS avg_valence,
    ROUND(AVG(speechiness)::numeric, 3) AS avg_speechiness,
    ROUND(AVG(acousticness)::numeric, 3) AS avg_acousticness,
    ROUND(AVG(instrumentalness)::numeric, 4) AS avg_instrumentalness,
    ROUND(AVG(liveness)::numeric, 3) AS avg_liveness,
    ROUND(AVG(loudness)::numeric, 2) AS avg_loudness_db,
    ROUND(AVG(tempo)::numeric, 1) AS avg_tempo_bpm,
    ROUND(AVG(duration_ms) / 1000.0, 1) AS avg_duration_sec,
    ROUND(AVG(track_popularity)::numeric, 1) AS avg_popularity
FROM unique_track_genre
GROUP BY playlist_genre
ORDER BY avg_popularity DESC;
```

</details>

Results:
| playlist_genre | unique_tracks_in_genre | avg_danceability | avg_energy | avg_valence | avg_speechiness | avg_acousticness | avg_instrumentalness | avg_liveness | avg_loudness_db | avg_tempo_bpm | avg_duration_sec | avg_popularity |
|----------------|------------------------|------------------|------------|-------------|-----------------|------------------|----------------------|--------------|-----------------|---------------|------------------|----------------|
| pop            | 5132                   | 0.638            | 0.701      | 0.502       | 0.074           | 0.172            | 0.0634               | 0.177        | -6.38           | 121           | 218.1            | 45.9           |
| latin          | 4641                   | 0.711            | 0.708      | 0.6         | 0.101           | 0.211            | 0.0489               | 0.181        | -6.4            | 118.6         | 217              | 44.3           |
| rap            | 5449                   | 0.716            | 0.65       | 0.504       | 0.197           | 0.196            | 0.0799               | 0.191        | -7.08           | 120.6         | 213.4            | 42.1           |
| rock           | 4451                   | 0.52             | 0.733      | 0.535       | 0.058           | 0.147            | 0.0656               | 0.205        | -7.54           | 125.1         | 247.8            | 40.4           |
| r&b            | 4948                   | 0.672            | 0.592      | 0.536       | 0.118           | 0.259            | 0.0279               | 0.176        | -7.91           | 114.2         | 238.4            | 39             |
| edm            | 5537                   | 0.658            | 0.799      | 0.406       | 0.088           | 0.085            | 0.2202               | 0.211        | -5.5            | 125.6         | 223.5            | 34.1           |

<ins> Some key findings:</ins>
* Pop is the most popular genre, but it's average in all the characteristics. We could say it's popular because 'it has a little bit of everything'
* The easiest way to detect a rap song (through data) is checking it's speechiness: it's roughly x2- x3 times the rate for other genres.
* Latin music tends to be the happiest (highest on valence) and in danceability (tied with rap). It's core to the sub-genres within this genre.
* Rock listeners are less likely to be listening mainstream genres. It has the lowest danceability, speechiness, longest average duration and low popularity.
* EDM has the highest energy, yet the lowest valence: energetic, but emotionally cold. It's the only genre where this occurs in such proportion.


2. Characteristics per genre:
<details>
<summary>Click to see SQL code</summary>
    
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
</details>

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

3. Trends across the decades:
<details>
<summary>Click to see SQL code</summary>

```sql
SELECT
    (LEFT(a.album_release_date, 4)::INTEGER / 10) * 10 AS decade,
    COUNT(*) AS n_tracks,
    ROUND(AVG(t.loudness)::numeric, 2)     AS avg_loudness_db,
    ROUND(AVG(t.energy)::numeric, 3)       AS avg_energy,
    ROUND(AVG(t.danceability)::numeric, 3) AS avg_danceability,
    ROUND(AVG(t.tempo)::numeric, 1)        AS avg_tempo,
    ROUND(AVG(t.duration_ms) / 1000.0, 1)  AS avg_duration_sec,
    ROUND(AVG(t.track_popularity)::numeric, 1) AS avg_popularity
FROM tracks t
JOIN albums a ON t.album_id = a.album_id
WHERE a.album_release_date IS NOT NULL
GROUP BY decade
ORDER BY decade;
```
| decade | n_tracks | avg_loudness_db | avg_energy | avg_danceability | avg_tempo | avg_duration_sec |
|--------|----------|-----------------|------------|------------------|-----------|------------------|
| 1960   | 132      | -9.92           | 0.591      | 0.514            | 122.9     | 217.6            |
| 1970   | 783      | -9.69           | 0.638      | 0.54             | 123.2     | 257.1            |
| 1980   | 1090     | -9.49           | 0.695      | 0.61             | 121.4     | 268              |
| 1990   | 2081     | -8.55           | 0.665      | 0.668            | 115.8     | 265.1            |
| 2000   | 3807     | -6.64           | 0.714      | 0.646            | 119.3     | 247.6            |
| 2010   | 19834    | -6.39           | 0.703      | 0.661            | 121.6     | 216.1            |
| 2020   | 626      | -6.89           | 0.672      | 0.668            | 122       | 194.5            |

</details>

4. Surprise Hits

<details>
<summary>Click here to see SQL code</summary>
    
```sql
WITH track_genre AS (
    SELECT DISTINCT
        t.track_id, t.track_name, t.track_artist, t.track_popularity,
        t.danceability, t.energy, t.valence, t.acousticness,
        t.speechiness, t.liveness,
        p.playlist_genre AS genre
    FROM tracks t
    JOIN track_playlist tp ON t.track_id = tp.track_id
    JOIN playlists p       ON tp.playlist_id = p.playlist_id
),
deduped AS (
    -- Keep one row per (track_name, track_artist), keeping the most popular version
    SELECT DISTINCT ON (track_name, track_artist)
        track_id, track_name, track_artist, track_popularity,
        danceability, energy, valence, acousticness, speechiness, liveness, genre
    FROM track_genre
    ORDER BY track_name, track_artist, track_popularity DESC
),
hit_profile AS (
    SELECT
        genre,
        AVG(danceability)  AS h_danceability,
        AVG(energy)        AS h_energy,
        AVG(valence)       AS h_valence,
        AVG(acousticness)  AS h_acousticness,
        AVG(speechiness)   AS h_speechiness,
        AVG(liveness)      AS h_liveness
    FROM deduped
    WHERE track_popularity >= 70
    GROUP BY genre
),
distance_to_hit AS (
    SELECT
        d.track_id, d.track_name, d.track_artist,
        d.genre, d.track_popularity,
        SQRT(
            POWER(d.danceability - hp.h_danceability, 2) +
            POWER(d.energy       - hp.h_energy,       2) +
            POWER(d.valence      - hp.h_valence,      2) +
            POWER(d.acousticness - hp.h_acousticness, 2) +
            POWER(d.speechiness  - hp.h_speechiness,  2) +
            POWER(d.liveness     - hp.h_liveness,     2)
        ) AS distance_from_hit_sound
    FROM deduped d
    JOIN hit_profile hp ON d.genre = hp.genre
)
SELECT
    track_name, track_artist, genre, track_popularity,
    ROUND(distance_from_hit_sound::numeric, 4) AS distance
FROM distance_to_hit
WHERE track_popularity >= 80          -- highly popular
ORDER BY distance_from_hit_sound DESC -- furthest from hit profile first
LIMIT 25;
``` 
</details>

| track_name                                                | track_artist    | genre | track_popularity | distance |
|-----------------------------------------------------------|-----------------|-------|------------------|----------|
| Can We Kiss Forever?                                      | Kina            | latin | 85               | 1.0233   |
| listen before i go                                        | Billie Eilish   | r&b   | 81               | 1.0001   |
| i love you                                                | Billie Eilish   | r&b   | 85               | 0.9026   |
| Bruises                                                   | Lewis Capaldi   | pop   | 86               | 0.8855   |
| July                                                      | Noah Cyrus      | latin | 88               | 0.8724   |
| A Gente Fez Amor - Ao Vivo                                | Gusttavo Lima   | edm   | 84               | 0.8676   |
| Get You The Moon (feat. Snøw)                             | Kina            | pop   | 86               | 0.8577   |
| When I Was Your Man                                       | Bruno Mars      | latin | 82               | 0.8568   |
| Good News                                                 | Mac Miller      | edm   | 87               | 0.8494   |
| lovely (with Khalid)                                      | Billie Eilish   | r&b   | 89               | 0.848    |
| Hey There Delilah                                         | Plain White T's | pop   | 80               | 0.8435   |
| you were good to me                                       | Jeremy Zucker   | r&b   | 82               | 0.8382   |
| You Are The Reason                                        | Calum Scott     | r&b   | 83               | 0.8291   |
| Invocada (Participação especial de Léo Santana) - Ao vivo | Ludmilla        | edm   | 80               | 0.8161   |
| bury a friend                                             | Billie Eilish   | pop   | 87               | 0.8073   |
| i hate u, i love u (feat. olivia o'brien)                 | gnash           | edm   | 81               | 0.8056   |
| xanny                                                     | Billie Eilish   | r&b   | 83               | 0.7867   |
| Devil Eyes                                                | Hippie Sabotage | pop   | 81               | 0.7603   |
| All of Me                                                 | John Legend     | r&b   | 85               | 0.7594   |
| Falling                                                   | Harry Styles    | r&b   | 88               | 0.7588   |
| Chanel                                                    | Frank Ocean     | pop   | 82               | 0.7585   |
| Lose You To Love Me                                       | Selena Gomez    | latin | 93               | 0.7525   |
| everything i wanted                                       | Billie Eilish   | r&b   | 97               | 0.7339   |
| Memories                                                  | Maroon 5        | latin | 98               | 0.7212   |
| If The World Was Ending (feat. Julia Michaels)            | JP Saxe         | latin | 88               | 0.7198   |

