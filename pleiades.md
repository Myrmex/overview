---
layout: page
title: Pleiades Wrapper
subtitle: "Pleiades Wrapper API"
---

Several projects (including [Cadmus](cadmus.md)) sometimes require a web API service to lookup geographic gazetteers data. One of the most useful gazetteers for many projects here is [Pleiades](http://www.pleiades.stoa.org/), which unfortunately does not come with its own service. So, in order to integrate it into several software solutions, I created an essential tool which:

- imports the full data graph from Pleiades (a huge JSON file) into a PostgreSQL database, remodeling data accordingly.
- provides a multiple-platform CLI tool to handle import and other related functions.
- wraps some essential query functions in a RESTful API.

The project code and documentation can be found in its [repository](https://github.com/vedph/pleiades-api). Currently, it serves two purposes in my environment:

- provides [Cadmus](cadmus.md) editors with lookup capabilities.
- provides lookup capabilites to the [epigraphic codices project](https://github.com/vedph/epicod), where massive metadata import and cross-matching is required.
