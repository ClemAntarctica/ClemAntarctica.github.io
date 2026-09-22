---
layout: post
title:  "ES_CPH — Every Street, Copenhagen"
permalink: /ES_CPH/
date:   2026-07-31
desc: "Tracking which streets of Copenhagen and Frederiksberg I've actually walked, run, or hiked"
keywords: "Clement,Cherblanc,Copenhagen,Frederiksberg,every street,running,hiking,GIS,python"
categories: [Python]
tags: [python,gis,running,]
icon: icon-html
---

Inspired by the "every street" mapping challenge, this project tracks exactly which streets and paths of Copenhagen and Frederiksberg I've covered on foot — running, hiking, or walking — and by how much.

<img src="{{ "/static/assets/img/landing/es_cph_coverage.png" | prepend: site.baseurl }}" width="100%">

<p class="text-center text-muted">Purple = covered at least once. Grey = not yet. Snapshot as of 22 September 2026.</p>

<p class="text-center"><small><a href="{{ "/static/assets/img/landing/es_cph_coverage_HR.png" | prepend: site.baseurl }}">High-res version</a> &middot; <a href="{{ "/static/assets/img/landing/es_cph_coverage_negative.png" | prepend: site.baseurl }}">Negative (what's left)</a></small></p>

### How it works

The street network (all walkable ways, including footpaths through parks and separated bike lanes) is pulled from OpenStreetMap and resampled into a point every ~10m. A point counts as covered as soon as one of my GPS tracks passes within a small buffer of it — tracks come from my own recorded runs and hikes, plus my full Strava activity history filtered down to on-foot activities that actually took place in Denmark.

### Current coverage

**1367 km done out of 5422 km of street network (25.7%).**

| District | Done (km) | Total (km) | % done |
|---|---:|---:|---:|
| Amager Vest | 380.0 | 820.5 | 46.3% |
| Østerbro | 205.0 | 498.6 | 41.1% |
| Indre By | 230.9 | 571.7 | 40.4% |
| Nørrebro | 98.0 | 278.6 | 35.2% |
| Amager Øst | 114.0 | 445.0 | 25.6% |
| Frederiksberg | 143.7 | 578.8 | 24.8% |
| Vesterbro-Kongens Enghave | 91.0 | 539.9 | 16.9% |
| Bispebjerg | 33.6 | 404.0 | 8.3% |
| Valby | 32.9 | 469.5 | 7.0% |
| Brønshøj-Husum | 23.2 | 397.6 | 5.8% |
| Vanløse | 14.6 | 317.9 | 4.6% |


*(Copenhagen's 10 official bydele, plus Frederiksberg as its own municipality — boundaries from Københavns Kommune's open-data service, since OpenStreetMap doesn't carry them.)*

A full write-up of the pipeline is coming to the blog. This page is regenerated from a small Python/geopandas script and will be updated as coverage grows.
