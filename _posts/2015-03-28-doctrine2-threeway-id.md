---
title: Doctrine2 Three-way many-to-many table result offset
tags: [doctrine2, dql]
---

There seems to be a bug with doctrine 2 when using a table looking like this:

| a\_id | b\_id | c\_id |
|    1 |    1 |    1 |
|    2 |    2 |    2 |
|    3 |    3 |    3 |

Here we have three foreign keys to other tables.
Together they are also the primary key of the table.

Now lets say we want to fetch all entries joined with the entries of table ```C``` using DQL:

~~~ php
$query = $this->_em->createQuery(
  'SELECT connector, c
  FROM Connector connector
  INNER JOIN connector.c c
');
~~~

I would expect something like this result with ```Query::HYDRATE_ARRAY```:

| connector.a\_id | connector.b\_id | connector.c\_id | c.id |
|    1 |    1 |    1 |    1 |
|    2 |    2 |    2 |    2 |
|    3 |    3 |    3 |    3 |

Instead, I get this:

| connector.a\_id | connector.b\_id | connector.c\_id | c.id |
|    1 |    1 |    1 |    2 |
|    2 |    2 |    2 |    3 |
|    3 |    3 |    3 | null |

The data of table ```C``` are all displaced by 1 row, with the last row not having any data.
This also applies to the object hydration.

It seems to be a problem related to the composite primary key.
Luckily I have access to the database itself, so I could circumvent the problem by adding an additional ```id```-column to the table, and making that one the primary key.