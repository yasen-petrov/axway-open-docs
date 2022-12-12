---
title: Update Operational Insights in Helm
linkTitle: Update Helm
weight: 10
date: 2022-08-04
description: Instructions on how to update Operational Insights between versions in a Helm chart.
---

If a new Elastic version is installed as a result of an upgrade, you must ensure that you have at least three Elasticsearch nodes running in the cluster. This is to ensure that a new master node can be selected during the upgrade and that updated nodes can join the cluster.

Example how to upgrade an existing release:

```bash
helm upgrade -n apim-elk -f myvalues.yaml axway-elk ($PLACEHOLDER))
```
