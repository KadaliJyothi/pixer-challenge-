WITH rank_awards AS (
    SELECT 
        film, 
        award_type, 
        ROW_NUMBER() OVER (PARTITION BY film ORDER BY award_type) AS rn
    FROM academy
    WHERE status IN ('won', 'Won Special Achievement')
),
award_summary AS (
    SELECT 
        film, 
        MAX(CASE WHEN rn = 1 THEN award_type END) AS award1,
        MAX(CASE WHEN rn = 2 THEN award_type END) AS award2,
        COUNT(*) AS award_count
    FROM rank_awards
    GROUP BY film
),
genre_summary AS (
    SELECT 
        film,
        STRING_AGG(CASE WHEN category = 'Genre' THEN value END, ', ') AS genres,
        STRING_AGG(CASE WHEN category = 'Subgenre' THEN value END, ', ') AS subgenres,
        COUNT(CASE WHEN category = 'Genre' THEN 1 END) AS gen_count,
        COUNT(CASE WHEN category = 'Subgenre' THEN 1 END) AS subgen_count
    FROM genres
    GROUP BY film
),
people_summary AS (
SELECT 
    film,
    STRING_AGG(CASE WHEN role_type = 'Co-director' THEN name END, ', ') AS co_directors,
	STRING_AGG(CASE WHEN role_type = 'Director' THEN name END, ', ') AS Director,
	STRING_AGG(CASE WHEN role_type = 'Musician' THEN name END, ', ') AS Musician,
	STRING_AGG(CASE WHEN role_type = 'Producer' THEN name END, ', ') AS Producer,
	STRING_AGG(CASE WHEN role_type = 'Screenwriter' THEN name END, ', ') AS Screenwriter,
	STRING_AGG(CASE WHEN role_type = 'Storywriter' THEN name END, ', ') AS Storywriter,
    COUNT(CASE WHEN role_type IN (
			'Co-director',
			'Director',
			'Musician',
			'Producer',
			'Screenwriter',
			'Storywriter' )THEN 1 END) AS people_count
FROM people
GROUP BY film
)

SELECT 
    f.number AS film_id, 
    f.film AS film_name, 
    f.release_date, 
    f.run_time, 
    f.film_rating, 
    f.plot,

    bo.budget,
    bo.box_office_us_canada,
    bo.box_office_other,
    bo.box_office_worldwide,

    pr.rotten_tomatoes_score,
    pr.rotten_tomatoes_counts,
    pr.metacritic_score,
    pr.metacritic_counts,
    pr.cinema_score,
    pr.imdb_score,
    pr.imdb_counts,

    aw.award1,
    aw.award2,
    aw.award_count,

    g.genres,
    g.subgenres,
    g.gen_count,
    g.subgen_count,
	
	P.co_directors,
	P.Director,
	P.Musician,
	P.Producer,
	P.Screenwriter,
	P.Storywriter


FROM films AS f
JOIN box_office AS bo ON f.film = bo.film
JOIN public_response AS pr ON bo.film = pr.film
LEFT JOIN award_summary AS aw ON f.film = aw.film
LEFT JOIN genre_summary AS g ON f.film = g.film
lEFT jOIN people_summary AS P ON f.film = p.film
