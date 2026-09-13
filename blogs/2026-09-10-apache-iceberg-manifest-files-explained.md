---
title: "Apache Iceberg Manifest Files Explained"
url: "https://www.dremio.com/blog/apache-iceberg-manifest-files-explained/"
date: "2026-09-10"
author: "Alex Merced"
feed_url: "https://www.dremio.com/blog/feed/"
---
A manifest file is an Avro file listing data files that belong to an Apache Iceberg table, along with per-column statistics for each one. Manifests are why Iceberg can plan a query without listing directories: the engine reads partition summaries in the manifest list to skip whole manifests, then reads column bounds inside the survivors […] The post Apache Iceberg Manifest Files Explained appeared first on Dremio .
