Arriba stores data in tables with strongly typed columns like a relational database. Data is stored by column, in memory, in strongly typed arrays. Arriba can infer column types automatically and create columns for new data. Columns can be sorted and word indexed to provide text and structured query support, or left as-is for fast insertion speed. Data is separated into "partitions" of 64K items which are split across processor cores and queried in parallel. Data can be saved to disk and reloaded rapidly. Queries and Aggregations are implemented via a public interface, IQuery, which has direct, strongly-typed access to the underlying arrays. This allows developers to write new custom queries which work with the data as quickly as if it was in a large array directly.