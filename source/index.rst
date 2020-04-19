.. cool documentation master file, created by
   sphinx-quickstart on Sun Apr 19 11:21:06 2020.
   You can adapt this file completely to your liking, but it should at least
   contain the root `toctree` directive.

Welcome to cool's documentation!
================================
Cool is a cohort online analytical processing system, which support both cohort query and OLAP query.


Storage design
==============
NEED STORAGE DIAGRAM HERE

Datasources
-----------
Cool data is stored in datasources, which are familier to tables in a tranditional RDBMS. A datasource is consist of metachunk(store meta data and data schema) and cublets(store structed data), compressed as a dz format file. It should be noted that numbers of dz files could belong to the same datasources, these dz files may have same metadata or not, it can be easily understood as sharding.

Metadata
--------
The metadata storage holds available data. There are some field types in cool: AppKey, UserKey, ActionTime, Day, Action, Segment, Property, Metric, Week, Month, Birth (**Field type should be clean later**). They can be simply divided into two types: range metadata and hash metadata(**May be inaccuracy**). Range metadata holds left bound and right bound of this field, apply to numeric data and date data. Hash metadata holds available values of this field, apply to string data. The system can easily filter whether this cubler is active or not. From anther point, metadata is essential for compress data, for more details in [compress](TODO) page.

Cublets
-------
Cublets is consist of data chunks. Data chunk is designed as column-oriented storage, which is consist of field. Field(Column) can also be invided to two types: range field and hash field. In range field, we get left bound and right bound of this data chunk first, which are helpful to judge should scan this chunk or not. Then we get records to process. In hash field, according to compress strategy we get keys first. Key is the index of available values in meta chunk(which called global id), and we get index of keys as local id consis of records. Use key the system can simply filter processable data chunk. Cool also provid optimize option for enumeration(like order status: unpaid, paid, shipping, completed), called pre-calculate. Benefit by compress algorithm, we get significant promotion by paying less storage space. Noted that data should order by user(for cohort query) and time.

.. compress TODO

Query processing
================

cohort query
------------
Cohort analysis is a method of analyzing a mertic by comparing its behaviour between groups of users. (**NEED COHORT QUERY DEMO HERE**)The execution of cohort query can be divided to three steps:

1. First initialize cohort selection and aggregation. In this stage we get birth filter and age filter.
2. Process metachunk next, system judge the cublet is active or not by birth filter and age filter. Add get global ids of birth action if the cublet is active.
3. If cublet is active, the system would traverse data chunks. Selector would filter data chunk. If data chunk is acrtive, then aggregation would process cohort results. System would merge results after all cublets have been processed.

