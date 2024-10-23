---
id: 01JATTWHSFQKHRZTF0MVFZ7928
title: Indexing
modified: 2024-10-22T15:23:25-04:00
tags:
  - indexing
  - data-structures
  - dsa
  - db
  - programming
  - roadmaps
---
# Indexing

Indexing is a data structure technique to efficiently retrieve data from a database. It essentially creates a lookup that can be used to quickly find the location of data records on a disk. Indexes are created using a few database columns and are capable of rapidly locating data without scanning every row in a database table each time the database table is accessed. Indexes can be created using any combination of columns in a database table, reducing the amount of time it takes to find data.

Indexes can be structured in several ways: Binary Tree, B-Tree, Hash Map, etc., each having its own particular strengths and weaknesses. When creating an index, it’s crucial to understand which type of index to apply in order to achieve maximum efficiency. Indexes, like any other database feature, must be used wisely because they require disk space and need to be maintained, which can slow down insert and update operations.