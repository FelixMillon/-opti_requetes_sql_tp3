# Exercice 2: Index Hash

### 2.1 Requête d'égalité exacte

``` sql
EXPLAIN ANALYZE
SELECT * FROM title_basics WHERE tconst = 'tt0000001';
```

* Output

| QUERY PLAN |
| :--- |
| Index Scan using idx\_title\_basics\_tconst\_hash on title\_basics  \(cost=0.00..8.02 rows=1 width=84\) \(actual time=0.034..0.036 rows=1 loops=1\) |
|   Index Cond: \(\(tconst\)::text = 'tt0000001'::text\) |
| Planning Time: 0.153 ms |
| Execution Time: 0.064 ms |


### 2.2 Création d'un index Hash

``` sql
CREATE INDEX idx_title_basics_tconst_hash ON title_basics USING hash (tconst);
```

### 2.3 Comparaison avec B-tree

* Création d'index B-tree sur la meme colone: 

**_1. Mesurez le temps d'exécution avec chaque index_**

**_2. Comparez la taille des deux index (utilisez pg_indexes_size)_**

``` sql
SELECT
  pg_size_pretty(pg_indexes_size('title_basics')) AS total_index_size;
```

* Output

| total\_index\_size |
| :--- |
| 1472 MB |

**_3. Testez une recherche par égalité et une recherche par plage_**

* Recherche par égalité: 

``` sql
EXPLAIN ANALYSE
SELECT * FROM title_basics WHERE tconst = 'tt0000001';
```

Output

| QUERY PLAN |
| :--- |
| Index Scan using idx\_title\_basics\_tconst\_hash on title\_basics  \(cost=0.00..8.02 rows=1 width=84\) \(actual time=0.034..0.036 rows=1 loops=1\) |
|   Index Cond: \(\(tconst\)::text = 'tt0000001'::text\) |
| Planning Time: 0.203 ms |
| Execution Time: 0.063 ms |

* Recherche par plage

``` sql
EXPLAIN ANALYSE
SELECT * FROM title_basics WHERE tconst >= 'tt0000001' AND tconst < 'tt0001000';
```
Output

| QUERY PLAN |
| :--- |
| Index Scan using title\_basics\_pkey on title\_basics  \(cost=0.43..8.48 rows=2 width=84\) \(actual time=0.119..0.661 rows=988 loops=1\) |
|   Index Cond: \(\(\(tconst\)::text &gt;= 'tt0000001'::text\) AND \(\(tconst\)::text &lt; 'tt0001000'::text\)\) |
| Planning Time: 0.261 ms |
| Execution Time: 0.750 ms |

### 2.4 Analyse et réflexion

_**1. Quelles sont les différences de performance entre Hash et B-tree pour l'égalité exacte?**_


**_2. Pourquoi l'index Hash ne fonctionne-t-il pas pour les recherches par plage?_**

Le Hash ne préserve pas l’ordre des valeurs. Il est inutile pour des opérations comme BETWEEN, >=, etc


**_3. Dans quel contexte précis privilégier un index Hash à un B-tree?_**



