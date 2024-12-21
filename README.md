# Optimisation-des-Requ-tes-SQL
### Optimisation des Requêtes

Voici plusieurs exemples de réécriture de requêtes SQL pour améliorer les performances dans PostgreSQL, avec des ajustements comme l'utilisation de `JOIN` à la place de `IN`, l'agrégation avant le `JOIN`, la sélection de colonnes spécifiques, la création d'index, et l'amélioration des requêtes avec des caractères génériques.

---

### **Exemple 1 : Remplacer `IN` par `JOIN`**

#### Requête originale (utilisation de `IN`) :
```sql
EXPLAIN ANALYZE
SELECT * 
FROM emp 
WHERE deptid IN (SELECT deptid FROM dept WHERE salary > 800);
```

#### Requête optimisée (utilisation de `JOIN`) :
```sql
EXPLAIN ANALYZE
SELECT emp.*
FROM emp
JOIN dept ON emp.deptid = dept.deptid
WHERE dept.salary > 800;
```

*Explication :* Remplacer l’utilisation de `IN` par un `JOIN` permet souvent d'améliorer les performances, surtout si la sous-requête dans le `IN` est complexe ou renvoie beaucoup de lignes.

---

### **Exemple 2 : Agrégation et Jointure**

#### Requête originale :
```sql
CREATE TABLE bill(id INT, status VARCHAR);
INSERT INTO bill VALUES (1, 'billed'), (2, 'non-billed');

CREATE TABLE order_item(orderid SERIAL, order_status INT);
INSERT INTO order_item(order_status)
SELECT x % 2 + 1
FROM generate_series(1, 1000000) AS x;

EXPLAIN ANALYZE
SELECT status, COUNT(*)
FROM bill AS a, order_item AS b
WHERE a.id = b.order_status
GROUP BY 1;
```

#### Requête optimisée avec `WITH` (Agrégation avant la jointure) :
```sql
EXPLAIN ANALYZE
WITH x AS (
  SELECT order_status, COUNT(*) AS res
  FROM order_item AS a
  GROUP BY 1
)
SELECT status, res
FROM x
JOIN bill AS y ON x.order_status = y.id;
```

*Explication :* En réalisant l'agrégation dans une sous-requête `WITH` (Common Table Expression ou CTE), nous réduisons la quantité de données à joindre, ce qui peut améliorer les performances, surtout lorsqu'il y a un grand nombre de lignes.

---

### **Exemple 3 : Sélection de colonnes spécifiques au lieu de `*`**

#### Requête originale :
```sql
EXPLAIN ANALYZE
SELECT * FROM pg_stats;
```

#### Requête optimisée (sélectionner uniquement les colonnes nécessaires) :
```sql
EXPLAIN ANALYZE
SELECT schemaname, tablename, attname
FROM pg_stats;
```

*Explication :* Sélectionner uniquement les colonnes nécessaires améliore les performances, car PostgreSQL évite de renvoyer toutes les données et les ressources nécessaires au traitement sont réduites.

---

### **Exemple 4 : Créer un index pour éviter le tri récurrent**

#### Requête originale (requête avec tri) :
```sql
EXPLAIN ANALYZE
SELECT * 
FROM emp 
ORDER BY empid;
```

#### Requête optimisée (création d'un index) :
```sql
CREATE INDEX idx_emp1 ON emp(empid);
EXPLAIN ANALYZE
SELECT * 
FROM emp 
ORDER BY empid;
```

*Explication :* Créer un index sur la colonne utilisée dans la clause `ORDER BY` permet à PostgreSQL de récupérer les données dans le bon ordre directement depuis l'index, évitant ainsi un tri coûteux à chaque exécution de la requête.

---

### **Exemple 5 : Utilisation de `LIKE` avec des caractères génériques**

#### Requête originale (wildcard à la fin) :
```sql
SELECT City 
FROM Customers 
WHERE City LIKE '%Char%';
```
*Résultat :* Charleston, Charleston, Charlton, Cape Charles, Crab Orchard, et Richardson.

#### Requête optimisée (wildcard au début) :
```sql
SELECT City 
FROM Customers 
WHERE City LIKE 'Char%';
```
*Résultat :* Charleston, Charlotte, et Charlton.

*Explication :* En modifiant le `LIKE` pour que le wildcard (`%`) ne soit qu'au début de la chaîne, PostgreSQL peut utiliser un index (s'il existe) pour accélérer la recherche, ce qui améliore les performances. Un `LIKE '%...%'` ne peut pas utiliser un index de manière optimale car il nécessite un scan complet de la table.

---

### Conclusion

L'optimisation des requêtes SQL dans PostgreSQL passe souvent par des ajustements simples, comme l'utilisation de `JOIN` au lieu de `IN`, l'agrégation préalable, la sélection des colonnes pertinentes, la création d'index pour éviter les tris coûteux, et l'amélioration de l'utilisation des caractères génériques dans les requêtes `LIKE`.
