# Bigtable Learning

Google Cloud Bigtable is a compressed, high-performance, proprietary data storage system built on Google File System, Chubby Lock Service, SSTable (log-structured storage like LevelDB) and a few other Google technologies. On May 6, 2015, a public version of Bigtable was made available as a service. Bigtable also underlies Google Cloud Datastore, which is available as a part of the Google Cloud Platform.

## Wide-column store vs. Columnar database
Wide-column stores such as Bigtable and Apache Cassandra are not column stores in the original sense of the term, since their two-level structures do not use a columnar data layout. In genuine column stores, a columnar data layout is adopted such that each column is stored separately on disk. Wide-column stores do often support the notion of column families that are stored separately. However, each such column family typically contains multiple columns that are used together, similar to traditional relational database tables. Within a given column family, all data is stored in a row-by-row fashion, such that the columns for a given row are stored together, rather than each column being stored separately.

Wide-column stores that support column families are also known as column family databases.

typical wide-column stores:
- Apache Accumulo
- Apache Cassandra
- Apache HBase
- Bigtable
- Azure Tables

## References
- wiki of bigtable: https://en.wikipedia.org/wiki/Bigtable
- wiki of wide-column store: https://en.wikipedia.org/wiki/Wide-column_store
- 行存储，列存储，宽列存储: https://www.51cto.com/article/618846.html
