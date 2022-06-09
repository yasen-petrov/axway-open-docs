---
title: Troubleshooting Operational Insights
linkTitle: Troubleshoot
weight: 200
date: 2022-06-09
description: Troubleshoot problems you might encounter when configuring and running Operational Insights.
---

## Solution does not show data in real time

An important aspect of the solution is to store the API requests as fast as possible in Elasticsearch, so that they are available in real-time in the API gateway traffic monitor or in the Kibana dashboard.
If your setup cannot process the log information fast enough, then the solution may never be real-time and eventually even log information may be lost. The following video provides information on why the document ingest rate may be too low, what information is available for analysis in Kibana, and how to correct this if necessary.

## placeholder