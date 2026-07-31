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

<p class="text-center text-muted">Red = covered at least once. Grey = not yet. Snapshot as of 31 July 2026.</p>

### How it works

The street network (all walkable ways, including footpaths through parks and separated bike lanes) is pulled from OpenStreetMap and resampled into a point every ~10m. A point counts as covered as soon as one of my GPS tracks passes within a small buffer of it — tracks come from my own recorded runs and hikes, plus my full Strava activity history filtered down to on-foot activities that actually took place in Denmark.

### Current coverage

**650 km done out of 5,322 km of street network (12.2%).**

| District | Done (km) | Total (km) | % done |
|---|---:|---:|---:|
| Nørrebro | 65.7 | 278.6 | 23.6% |
| Indre By | 131.5 | 571.7 | 23.0% |
| Amager Vest | 169.0 | 820.5 | 20.6% |
| Østerbro | 83.3 | 498.6 | 16.7% |
| Frederiksberg | 67.2 | 578.8 | 11.6% |
| Vesterbro-Kongens Enghave | 38.4 | 539.9 | 7.1% |
| Amager Øst | 29.2 | 445.0 | 6.6% |
| Bispebjerg | 26.1 | 404.0 | 6.5% |
| Brønshøj-Husum | 21.2 | 397.6 | 5.3% |
| Vanløse | 8.9 | 317.9 | 2.8% |
| Valby | 9.7 | 469.5 | 2.1% |

*(Copenhagen's 10 official bydele, plus Frederiksberg as its own municipality — boundaries from Københavns Kommune's open-data service, since OpenStreetMap doesn't carry them.)*

A full write-up of the pipeline is coming to the blog. This page is regenerated from a small Python/geopandas script and will be updated as coverage grows.
