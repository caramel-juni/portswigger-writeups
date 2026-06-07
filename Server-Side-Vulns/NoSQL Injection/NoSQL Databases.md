---
sticker: emoji//2705
---
#nosql

## What are NoSQL Databases?
**NoSQL** (Not Only SQL) databases are any databases that **store/retrieve data WITHOUT the use of relational (row/column-based) tables**, like those in traditional SQL databases. Instead of using the universal Structured Query Language (SQL), they **can receive data via a wide range of query languages**, including `JSON`, `XML`, or even **custom query languages.**
**Strengths:**
- Can handle large amounts of un/semi-structured data
- Scalable, flexible & performant (less consistency checks, more permissive)... *but also fewer constraints = wider attack surface*
![](attachments/NoSQL%20Injection%20&%20Databases-1.png)

### Types of NoSQL Databases:
- **Document Stores:** data stored in **flexible, semi-structured documents**, and queried in an API or query language. The documents themselves can contain all sorts of values: numbers, arrays, objects, strings, Booleans, key0-value pairs, nested documents or other structured data.
	- **Formats:** `JSON`, `BSON`, `XML`
	- **Examples:** `MongoDB`, `Couchbase`
- **Key-value stores:** associates, stores & retrieves data as **groupings of unique key-value pairs**, with each data record represented by a unique key string & associated value.
	- **Examples:** `Redis`, `Amazon DynamoDB`, `ScyllaDB`
- **Wide-column stores:** data is organised into **flexible column "families"**, rather than traditional rows.
	- **Examples:** `Apache Cassandra`, `Apache HBase`
- **Graph databases:** data entries **stored in graph "nodes"**, and **relationships between entities stored in "edges"**. Good for social networks/hierarchies.
	- **Examples:** `Neo4j`, `Amazon Neptune`, `Kibana`

##### SQL Database examples:
- OracleDB
- MySQL
- PostgreSQL
- MSSQL
- SQLite

---

