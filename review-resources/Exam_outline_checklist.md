## 1、Installation and Configuration
#### 1.1 Deploy and start an Elasticsearch cluster that satisfies a given set of requirements
https://www.elastic.co/guide/en/elasticsearch/reference/7.2/setup.html

#### 1.2 Configure the nodes of a cluster to satisfy a given set of requirements
https://www.elastic.co/guide/en/elasticsearch/reference/7.2/important-settings.html
https://www.elastic.co/guide/en/elasticsearch/reference/7.2/modules-node.html
#### 1.3 Secure a cluster using Elasticsearch Security
https://www.elastic.co/guide/en/elasticsearch/reference/7.2/security-settings.html
https://www.elastic.co/guide/en/elasticsearch/reference/7.2/configuring-security.html
https://www.elastic.co/guide/en/elasticsearch/reference/7.2/setup-passwords.html

#### 1.4Define role-based access control using Elasticsearch Security
https://www.elastic.co/guide/en/elasticsearch/reference/7.2/security-api-put-role.html
https://www.elastic.co/guide/en/elasticsearch/reference/7.2/security-api-put-user.html

## 2、Indexing Data
#### 2.1 Define an index that satisfies a given set of requirements
https://www.elastic.co/guide/en/elasticsearch/reference/7.2/indices-create-index.html

#### 2.2 Perform index, create, read, update, and delete operations on the documents of an index
https://www.elastic.co/guide/en/elasticsearch/reference/7.2/docs.html

#### 2.3 Define and use index aliases
https://www.elastic.co/guide/en/elasticsearch/reference/7.2/indices-aliases.html

#### 2.4 Define and use an index template for a given pattern that satisfies a given set of requirements
https://www.elastic.co/guide/en/elasticsearch/reference/7.2/indices-templates.html

#### 2.5 Define and use a dynamic template that satisfies a given set of requirements
https://www.elastic.co/guide/en/elasticsearch/reference/7.2/dynamic-templates.html

#### 2.6 Use the Reindex API and Update By Query API to reindex and/or update documents
https://www.elastic.co/guide/en/elasticsearch/reference/7.2/docs-reindex.html
https://www.elastic.co/guide/en/elasticsearch/reference/7.2/docs-update-by-query.html

#### 2.7 Define and use an ingest pipeline that satisfies a given set of requirements, including the use of Painless to modify documents
https://www.elastic.co/guide/en/elasticsearch/reference/7.2/ingest.html
https://www.elastic.co/guide/en/elasticsearch/reference/7.2/modules-scripting.html
https://www.elastic.co/guide/en/elasticsearch/painless/7.2/painless-lang-spec.html


## 3、Queries
#### 3.1 Write and execute a search query for terms and/or phrases in one or more fields of an index
https://www.elastic.co/guide/en/elasticsearch/reference/7.2/query-dsl.html

#### 3.2 Write and execute a search query that is a Boolean combination of multiple queries and filters
https://www.elastic.co/guide/en/elasticsearch/reference/7.2/query-dsl-bool-query.html

#### 3.3 Highlight the search terms in the response of a query
https://www.elastic.co/guide/en/elasticsearch/reference/7.2/search-request-highlighting.html

#### 3.4 Sort the results of a query by a given set of requirements
https://www.elastic.co/guide/en/elasticsearch/reference/7.2/search-request-sort.html

#### 3.5Implement pagination of the results of a search query
https://www.elastic.co/guide/en/elasticsearch/reference/7.2/search-request-from-size.html

#### 3.6 Use the scroll API to retrieve large numbers of results
https://www.elastic.co/guide/en/elasticsearch/reference/7.2/search-request-scroll.html

#### 3.7 Apply fuzzy matching to a query
https://www.elastic.co/guide/en/elasticsearch/reference/7.2/query-dsl-fuzzy-query.html

#### 3.8 Define and use a search template
https://www.elastic.co/guide/en/elasticsearch/reference/7.2/search-template.html

#### 3.9 Write and execute a query that searches across multiple clusters
https://www.elastic.co/guide/en/elasticsearch/reference/7.2/modules-cross-cluster-search.html


## 4、Aggregations
#### 4.1 Write and execute metric and bucket aggregations
https://www.elastic.co/guide/en/elasticsearch/reference/7.2/search-aggregations-metrics.html
https://www.elastic.co/guide/en/elasticsearch/reference/7.2/search-aggregations-bucket.html

#### 4.2 Write and execute aggregations that contain sub-aggregations
https://www.elastic.co/guide/en/elasticsearch/reference/7.2/search-aggregations-pipeline.html

## 5、 Mappings and Text Analysis
#### 5.1 Define a mapping that satisfies a given set of requirements
https://www.elastic.co/guide/en/elasticsearch/reference/7.2/mapping.html

#### 5.2 Define and use a custom analyzer that satisfies a given set of requirements
https://www.elastic.co/guide/en/elasticsearch/reference/7.2/analysis.html
https://www.elastic.co/guide/en/elasticsearch/reference/7.2/analysis-custom-analyzer.html

#### 5.3 Define and use multi-fields with different data types and/or analyzers
https://www.elastic.co/guide/en/elasticsearch/reference/7.2/multi-fields.html

#### 5.4 Configure an index so that it properly maintains the relationships of nested arrays of objects
https://www.elastic.co/guide/en/elasticsearch/reference/7.2/nested.html
https://www.elastic.co/guide/en/elasticsearch/reference/7.2/parent-join.html

## 6、Cluster Administration
#### 6.1 Allocate the shards of an index to specific nodes based on a given set of requirements
https://www.elastic.co/guide/en/elasticsearch/reference/7.2/index-modules-allocation.html
https://www.elastic.co/guide/en/elasticsearch/reference/7.2/modules-cluster.html

#### 6.2 Configure shard allocation awareness and forced awareness for an index
https://www.elastic.co/guide/en/elasticsearch/reference/7.2/allocation-awareness.html

#### 6.3 Diagnose shard issues and repair a cluster’s health
https://www.elastic.co/guide/en/elasticsearch/reference/7.2/cluster-health.html
https://www.elastic.co/guide/en/elasticsearch/reference/7.2/cluster-allocation-explain.html
https://www.elastic.co/guide/en/elasticsearch/reference/7.2/cluster-reroute.html

#### 6.4 Backup and restore a cluster and/or specific indices
https://www.elastic.co/guide/en/elasticsearch/reference/7.2/backup-cluster.html
https://www.elastic.co/guide/en/elasticsearch/reference/7.2/modules-snapshots.html

#### 6.5 Configure a cluster for use with a hot/warm architecture
https://www.elastic.co/guide/en/elasticsearch/reference/7.2/shard-allocation-filtering.html

#### 6.6 Configure a cluster for cross cluster search
https://www.elastic.co/guide/en/elasticsearch/reference/7.2/modules-remote-clusters.html
https://www.elastic.co/guide/en/elasticsearch/reference/7.2/modules-cross-cluster-search.html
