# Exercice 4: Index partiels

## 4.1 Identifier un sous-ensemble fréquent
SELECT (startYear / 10) * 10 AS decenie,
       COUNT(*) AS total_titres
FROM title_basics
WHERE startYear IS NOT NULL
GROUP BY decenie
ORDER BY total_titres DESC
LIMIT 10;

résultat :
2010	3856234
2020	2500598
2000	1629925
1990	754161
1980	465351
1970	400174
1960	325856
1950	164612
1910	72582
1920	36953
1930	32804

## 4.2 Requête sur ce sous-ensemble
SELECT COUNT(*) AS total_movies
FROM title_basics
WHERE startYear BETWEEN 2010 AND 2019;

## 4.3 Création d'un index partiel
CREATE INDEX idx_title_basics_startYear_partial_2010_2020 ON title_basics(startYear) where startYear BETWEEN 2010 AND 2019;

## 4.4 Comparaison avec index complet
CREATE INDEX idx_title_basics_startYear_complete ON title_basics(startYear);

### 1. Les performances pour les requêtes dans la période ciblée
EXPLAIN ANALYSE
SELECT COUNT(*) AS total_movies
FROM title_basics
WHERE startYear BETWEEN 2010 AND 2019;

"Finalize Aggregate  (cost=53777.26..53777.27 rows=1 width=8) (actual time=178.980..182.130 rows=1 loops=1)"
"  ->  Gather  (cost=53777.05..53777.26 rows=2 width=8) (actual time=178.931..182.105 rows=3 loops=1)"
"        Workers Planned: 2"
"        Workers Launched: 2"
"        ->  Partial Aggregate  (cost=52777.05..52777.06 rows=1 width=8) (actual time=175.437..175.439 rows=1 loops=3)"
"              ->  Parallel Index Only Scan using idx_title_basics_startyear_partial_2010_2020 on title_basics  (cost=0.43..48720.02 rows=1622812 width=0) (actual time=0.027..109.923 rows=1285411 loops=3)"
"                    Heap Fetches: 0"
"Planning Time: 0.191 ms"
"Execution Time: 182.159 ms"

### 2. Les performances pour les requêtes hors de cette période
EXPLAIN ANALYSE
SELECT COUNT(*) AS total_movies
FROM title_basics
WHERE startYear BETWEEN 2020 AND 2029;

"Finalize Aggregate  (cost=47156.94..47156.95 rows=1 width=8) (actual time=113.082..115.922 rows=1 loops=1)"
"  ->  Gather  (cost=47156.73..47156.94 rows=2 width=8) (actual time=112.876..115.915 rows=3 loops=1)"
"        Workers Planned: 2"
"        Workers Launched: 2"
"        ->  Partial Aggregate  (cost=46156.73..46156.74 rows=1 width=8) (actual time=109.450..109.451 rows=1 loops=3)"
"              ->  Parallel Index Only Scan using idx_title_basics_startyear_complete on title_basics  (cost=0.43..43579.59 rows=1030854 width=0) (actual time=0.051..69.091 rows=833533 loops=3)"
"                    Index Cond: ((startyear >= 2020) AND (startyear <= 2029))"
"                    Heap Fetches: 0"
"Planning Time: 0.141 ms"
"Execution Time: 115.950 ms"

### 3. La taille des deux index
