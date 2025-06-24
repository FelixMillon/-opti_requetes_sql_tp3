# Exercice 1: Index B-tree

## 1.1 Analyse sans index

EXPLAIN ANALYSE
SELECT * FROM title_basics
WHERE primary_title LIKE 'The%'; 

"Gather  (cost=1000.00..288760.16 rows=527851 width=84) (actual time=2.302..579.359 rows=605129 loops=1)"
"  Workers Planned: 2"
"  Workers Launched: 2"
"  ->  Parallel Seq Scan on title_basics  (cost=0.00..234975.06 rows=219938 width=84) (actual time=3.250..537.403 rows=201710 loops=3)"
"        Filter: ((primary_title)::text ~~ 'The%'::text)"
"        Rows Removed by Filter: 3709910"
"Planning Time: 0.078 ms"
"JIT:"
"  Functions: 6"
"  Options: Inlining false, Optimization false, Expressions true, Deforming true"
"  Timing: Generation 0.703 ms (Deform 0.302 ms), Inlining 0.000 ms, Optimization 0.720 ms, Emission 8.882 ms, Total 10.304 ms"
"Execution Time: 605.696 ms"


## 1.2 Création d'un index B-tree
CREATE INDEX idx_title_basics_primary_title ON title_basics (primary_title);

## 1.3 Analyse après indexation

"Gather  (cost=1000.00..288760.16 rows=527851 width=84) (actual time=2.584..516.108 rows=605129 loops=1)"
"  Workers Planned: 2"
"  Workers Launched: 2"
"  ->  Parallel Seq Scan on title_basics  (cost=0.00..234975.06 rows=219938 width=84) (actual time=2.794..477.249 rows=201710 loops=3)"
"        Filter: ((primary_title)::text ~~ 'The%'::text)"
"        Rows Removed by Filter: 3709910"
"Planning Time: 0.067 ms"
"JIT:"
"  Functions: 6"
"  Options: Inlining false, Optimization false, Expressions true, Deforming true"
"  Timing: Generation 0.585 ms (Deform 0.212 ms), Inlining 0.000 ms, Optimization 0.570 ms, Emission 7.677 ms, Total 8.831 ms"
"Execution Time: 541.329 ms"

L'index n"est pas utilisé dans le plan, pas d'amélioration des performances



## 1.4 Test des différentes opérations
### 1. Égalité exacte (WHERE primary_Title = '...')
    EXPLAIN ANALYSE
    SELECT * FROM title_basics
    WHERE primary_title = 'The Terrible Railway Accident';

    "Bitmap Heap Scan on title_basics  (cost=5.57..516.53 rows=130 width=84) (actual time=0.111..0.113 rows=1 loops=1)"
    "  Recheck Cond: ((primary_title)::text = 'The Terrible Railway Accident'::text)"
    "  Heap Blocks: exact=1"
    "  ->  Bitmap Index Scan on idx_title_basics_primary_title  (cost=0.00..5.53 rows=130 width=0) (actual time=0.084..0.084 rows=1 loops=1)"
    "        Index Cond: ((primary_title)::text = 'The Terrible Railway Accident'::text)"
    "Planning Time: 0.117 ms"
    "Execution Time: 0.679 ms"

    sur l'égalite, l'index est bien utilisé (création d'un bitmap)

### 2. Préfixe (WHERE primary_Title LIKE 'The%')
    EXPLAIN ANALYSE
    SELECT * FROM title_basics
    WHERE primary_title LIKE 'The%'; 

    "Gather  (cost=1000.00..288760.16 rows=527851 width=84) (actual time=2.851..562.680 rows=605129 loops=1)"
    "  Workers Planned: 2"
    "  Workers Launched: 2"
    "  ->  Parallel Seq Scan on title_basics  (cost=0.00..234975.06 rows=219938 width=84) (actual time=3.370..520.060 rows=201710 loops=3)"
    "        Filter: ((primary_title)::text ~~ 'The%'::text)"
    "        Rows Removed by Filter: 3709910"
    "Planning Time: 0.079 ms"
    "JIT:"
    "  Functions: 6"
    "  Options: Inlining false, Optimization false, Expressions true, Deforming true"
    "  Timing: Generation 0.638 ms (Deform 0.258 ms), Inlining 0.000 ms, Optimization 0.623 ms, Emission 9.336 ms, Total 10.597 ms"
    "Execution Time: 588.477 ms"

    sur le préfix, l'index n'est pas utilisé


### 3. Suffixe (WHERE primary_Title LIKE '%The')
    EXPLAIN ANALYSE
    SELECT * FROM title_basics
    WHERE primary_title LIKE '%The';

    "Gather  (cost=1000.00..236079.56 rows=1045 width=84) (actual time=2.505..690.160 rows=1181 loops=1)"
    "  Workers Planned: 2"
    "  Workers Launched: 2"
    "  ->  Parallel Seq Scan on title_basics  (cost=0.00..234975.06 rows=435 width=84) (actual time=4.041..667.859 rows=394 loops=3)"
    "        Filter: ((primary_title)::text ~~ '%the'::text)"
    "        Rows Removed by Filter: 3911226"
    "Planning Time: 0.103 ms"
    "JIT:"
    "  Functions: 6"
    "  Options: Inlining false, Optimization false, Expressions true, Deforming true"
    "  Timing: Generation 0.776 ms (Deform 0.290 ms), Inlining 0.000 ms, Optimization 0.610 ms, Emission 8.364 ms, Total 9.751 ms"
    "Execution Time: 690.670 ms"

    sur le suffix, pas d'utilisation de l'index

### 4. Sous-chaîne (WHERE primary_Title LIKE '%The%')
    EXPLAIN ANALYSE
    SELECT * FROM title_basics
    WHERE primary_title LIKE '%the%';

    "Gather  (cost=1000.00..299317.26 rows=633422 width=84) (actual time=2.792..838.891 rows=525744 loops=1)"
    "  Workers Planned: 2"
    "  Workers Launched: 2"
    "  ->  Parallel Seq Scan on title_basics  (cost=0.00..234975.06 rows=263926 width=84) (actual time=3.467..794.506 rows=175248 loops=3)"
    "        Filter: ((primary_title)::text ~~ '%the%'::text)"
    "        Rows Removed by Filter: 3736372"
    "Planning Time: 0.107 ms"
    "JIT:"
    "  Functions: 6"
    "  Options: Inlining false, Optimization false, Expressions true, Deforming true"
    "  Timing: Generation 0.795 ms (Deform 0.301 ms), Inlining 0.000 ms, Optimization 0.603 ms, Emission 9.615 ms, Total 11.012 ms"
    "Execution Time: 864.088 ms"

    sur la sous chaine, pas d'utilisation de l'index

### 5. Ordre (ORDER BY primary_Title)
    EXPLAIN ANALYSE
    SELECT * FROM title_basics
    ORDER BY primary_Title; 

    "Index Scan using idx_title_basics_primary_title on title_basics  (cost=0.56..1018624.88 rows=11734860 width=84) (actual time=0.110..23095.907 rows=11734860 loops=1)"
    "Planning Time: 0.100 ms"
    "Execution Time: 23590.892 ms"

    L'index est bien utilisé

## 1.5 Analyse et réflexion
### 1. Pour quels types d'opérations l'index B-tree est-il efficace?
    l'index B-tree est efficace pour les opérations de de comparaison (=) et de tri (Order).
### 2. Pourquoi l'index n'est-il pas utilisé pour certaines opérations?
    Pour les suffix et sous chaine, postgres n'a pas de moyen de savoir ou commence la sous chaine.
    Il ne peut donc pas comparer rapidement les lignes avec la sous chaine de la requete
    Pour le prefix: Il peut l'utiliser, mais cela dépendra du cout (nombre de ligne à traiter)
### 3. Dans quels cas un index B-tree est-il le meilleur choix?
    B-tree est très polyvalent. C'est un bon choix pour les tables amenés à être triés, ou qui sur lesquelles des recherches de données completes vont être éxecutées.
    Par exemple une table : utilisateur, ou client. Dans lesquelle on tri par nom, date de naissance, ville.