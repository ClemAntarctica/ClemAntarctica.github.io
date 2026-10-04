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

<p class="text-center text-muted">Purple = covered at least once. Grey = not yet. Snapshot as of 4 October 2026.</p>

<p class="text-center"><small><a href="{{ "/static/assets/img/landing/es_cph_coverage_HR.png" | prepend: site.baseurl }}">High-res version</a> &middot; <a href="{{ "/static/assets/img/landing/es_cph_coverage_negative.png" | prepend: site.baseurl }}">Negative (what's left)</a></small></p>

### How it works

The street network (all walkable ways, including footpaths through parks and separated bike lanes) is pulled from OpenStreetMap and resampled into a point every ~10m. A point counts as covered as soon as one of my GPS tracks passes within a small buffer of it — tracks come from my own recorded runs and hikes, plus my full Strava activity history filtered down to on-foot activities that actually took place in Denmark.

### Current coverage

**1454 km done out of 5371 km of street network (27.1%).**

| District | Done (km) | Total (km) | % done |
|---|---:|---:|---:|
| Østerbro | 260.7 | 498.6 | 51.4% |
| Amager Vest | 382.3 | 820.5 | 46.3% |
| Indre By | 236.2 | 571.7 | 40.3% |
| Nørrebro | 98.9 | 278.6 | 35.3% |
| Amager Øst | 135.9 | 445.0 | 30.4% |
| Frederiksberg | 144.2 | 578.8 | 24.8% |
| Vesterbro-Kongens Enghave | 91.4 | 539.9 | 16.7% |
| Bispebjerg | 33.8 | 404.0 | 8.4% |
| Valby | 32.9 | 469.5 | 6.9% |
| Brønshøj-Husum | 23.2 | 397.6 | 5.8% |
| Vanløse | 14.6 | 317.9 | 4.6% |


*(Copenhagen's 10 official bydele, plus Frederiksberg as its own municipality — boundaries from Københavns Kommune's open-data service, since OpenStreetMap doesn't carry them.)*

A full write-up of the pipeline is coming to the blog. This page is regenerated from a small Python/geopandas script and will be updated as coverage grows.
