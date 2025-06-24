# Exercice 5: Index d'expressions

5.1 Recherche insensible à la casse

Écrivez une requête qui recherche des titres indépendamment de la casse (par exemple, trouver "Star Wars" même si saisi comme "star wars").

``` sql
SELECT * FROM title_basics
WHERE LOWER(primary_title) = LOWER('Star Wars');
```

5.2 Mesure des performances sans index adapté
Analysez les performances avec l'index B-tree standard sur primary_Title.

| QUERY PLAN |
| :--- |
| Gather  \(cost=1000.00..254066.27 rows=58674 width=84\) \(actual time=266.154..3257.027 rows=76 loops=1\) |
|   Workers Planned: 2 |
|   Workers Launched: 2 |
|   -&gt;  Parallel Seq Scan on title\_basics  \(cost=0.00..247198.88 rows=24448 width=84\) \(actual time=250.590..3168.646 rows=25 loops=3\) |
|         Filter: \(lower\(\(primary\_title\)::text\) = 'star wars'::text\) |
|         Rows Removed by Filter: 3911595 |
| Planning Time: 2.717 ms |
| JIT: |
|   Functions: 6 |
|   Options: Inlining false, Optimization false, Expressions true, Deforming true |
|   Timing: Generation 2.837 ms \(Deform 0.668 ms\), Inlining 0.000 ms, Optimization 2.311 ms, Emission 34.462 ms, Total 39.610 ms |
| Execution Time: 3258.931 ms |


5.3 Création d'un index d'expression
Créez un index sur l'expression LOWER(primary_Title).

``` sql
CREATE INDEX idx_title_basics_lower_title ON title_basics (LOWER(primary_title));
```

5.4 Mise à jour de la requête et test
Modifiez votre requête pour utiliser LOWER() et mesurez l'amélioration des performances.

``` sql
EXPLAIN ANALYZE
SELECT * FROM title_basics
WHERE LOWER(primary_title) = 'star wars';
```

* Output

| QUERY PLAN |
| :--- |
| Bitmap Heap Scan on title\_basics  \(cost=1191.28..121951.27 rows=58674 width=84\) \(actual time=0.216..0.375 rows=76 loops=1\) |
|   Recheck Cond: \(lower\(\(primary\_title\)::text\) = 'star wars'::text\) |
|   Heap Blocks: exact=76 |
|   -&gt;  Bitmap Index Scan on idx\_title\_basics\_lower\_title  \(cost=0.00..1176.62 rows=58674 width=0\) \(actual time=0.177..0.178 rows=76 loops=1\) |
|         Index Cond: \(lower\(\(primary\_title\)::text\) = 'star wars'::text\) |
| Planning Time: 0.244 ms |
| JIT: |
|   Functions: 2 |
|   Options: Inlining false, Optimization false, Expressions true, Deforming true |
|   Timing: Generation 0.449 ms \(Deform 0.194 ms\), Inlining 0.000 ms, Optimization 0.000 ms, Emission 0.000 ms, Total 0.449 ms |
| Execution Time: 1.013 ms |



5.5 Autre exemple d'expression
Créez un index sur une autre expression (par exemple, sur l'extraction de l'année à partir d'une date) et testez son efficacité.

5.6 Analyse et réflexion
Répondez aux questions:
1. Pourquoi l'expression dans la requête doit-elle correspondre exactement à celle de l'index?
2. Quel est l'impact des index d'expressions sur les performances d'écriture?
3. Quels types de transformations sont souvent utilisés dans les index d'expressions?