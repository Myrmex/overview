---
layout: page
title: Graph Classes
subtitle: "Cadmus Development"
---

🛠️ Technical Note

In the editor database, graph nodes representing classes are marked as such by setting their `node.is_class` field to true. Also, to allow for a very basic inference mechanism useful for in-place searches, the special `node_class` table provides a list of all the classes a node descends from, either directly or indirectly. This table is automatically updated using an SQL function named `populate_node_class`.

This function receives the following arguments:

- `instance_id`: the numeric ID of the node to update ascendant classes for.
- `a_id`: the numeric ID of the `rdf:type` ("a") node.
- `sub_id`: the numeric ID of the `rdfs:subClassOf` node.

The function is like this (this is the MySql version of it):

```sql
DELIMITER $$
CREATE PROCEDURE `populate_node_class`(IN instance_id INT, IN a_id INT, IN sub_id INT)
BEGIN
INSERT INTO node_class(node_id, class_id, level)
WITH RECURSIVE cn AS (
    -- ANCHOR
    -- get object class node
    SELECT t.o_id AS id, 1 AS level FROM node n
    -- of a triple having S=start node P=a O=class node
    INNER JOIN triple t ON t.s_id=n.id AND t.p_id=a_id
    LEFT JOIN node n2 ON t.o_id=n2.id AND n2.is_class=true
    WHERE n.id=instance_id
    UNION DISTINCT
    -- RECURSIVE
    SELECT t.o_id AS id, level+1 AS level FROM cn
    -- sub_id is the ID of rdfs:subClassOf
    INNER JOIN triple t ON t.s_id=cn.id AND t.p_id=sub_id
    LEFT JOIN node n2 ON t.o_id=n2.id AND n2.is_class=true
)
SELECT instance_id, cn.id, cn.level FROM cn;
END$$
DELIMITER ;
```

This is a recursive CTE function, having an anchor query UNIONed (via `UNION ALL` or `UNION DISTINCT`) with a recursive query. The anchor query provides the start value, and the recursive query is repeated for each match.

Say you have these data:

```sql
INSERT INTO uri_lookup (uri) VALUES
     ('x:animal'),
     ('x:dog'),
     ('x:mammal'),
     ('x:snoopy');

-- nodes
INSERT INTO node (id, is_class, tag, label, source_type, sid) VALUES
     (16, true, NULL, 'x:animal', 0, NULL),
     (17, true, NULL, 'x:mammal', 0, NULL),
     (18, true, NULL, 'x:dog', 0, ''),
     (19, false, NULL, 'x:snoopy', 0, NULL);

-- triples:
-- snoopy a dog
-- dog subclass-of mammal
-- mammal subclass-of animal
INSERT INTO triple (s_id, p_id, o_id, o_lit, o_lit_type, o_lit_lang, o_lit_ix, o_lit_n, sid, tag) VALUES
     (19,7,18,NULL,NULL,NULL,NULL,NULL,NULL,NULL),
     (18,8,17,NULL,NULL,NULL,NULL,NULL,NULL,NULL),
     (17,8,16,NULL,NULL,NULL,NULL,NULL,NULL,NULL);
```

Here you have the node `snoopy`, and 3 class nodes: `dog`, `mammal`, and `animal`. In the above SQL code, 7 is the ID of `rdf:type` and 8 that of `rdfs:subClassOf`. If now you call the function with:

```sql
CALL populate_node_class(19, 7, 8);
```

where 19=snoopy, 7=a, 8=subclass, the function will populate table `node_class` with these rows:

node_id   | class_id  | level
----------|-----------|------
19=snoopy | 18=dog    | 1
19=snoopy | 17=mammal | 2
19=snoopy | 16=animal | 3

This way, starting from the triple telling that snoopy is a dog, we infer other triples telling that snoopy is a mammal and also an animal.

>Note that the result is the union of the anchor query, which produced level-1 row, with the recursive query, which produced the other rows.
