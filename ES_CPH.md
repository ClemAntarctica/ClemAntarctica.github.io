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

<p class="text-center text-muted">Red = covered at least once. Grey = not yet. Snapshot as of 6 August 2026.</p>

### How it works

The street network (all walkable ways, including footpaths through parks and separated bike lanes) is pulled from OpenStreetMap and resampled into a point every ~10m. A point counts as covered as soon as one of my GPS tracks passes within a small buffer of it — tracks come from my own recorded runs and hikes, plus my full Strava activity history filtered down to on-foot activities that actually took place in Denmark.

### Current coverage

**1057 km done out of 4265 km of street network (19.9%).**

| District | Done (km) | Total (km) | % done |
|---|---:|---:|---:|
| Amager Vest | 298.7 | 820.5 | 36.4% |
| Nørrebro | 97.9 | 278.6 | 35.1% |
| Indre By | 180.9 | 571.7 | 31.6% |
| Frederiksberg | 142.6 | 578.8 | 24.6% |
| Østerbro | 119.3 | 498.6 | 23.9% |
| Amager Øst | 65.8 | 445.0 | 14.8% |
| Vesterbro-Kongens Enghave | 64.0 | 539.9 | 11.9% |
| Bispebjerg | 33.6 | 404.0 | 8.3% |
| Brønshøj-Husum | 23.2 | 397.6 | 5.8% |
| Vanløse | 14.6 | 317.9 | 4.6% |
| Valby | 16.8 | 469.5 | 3.6% |

*(Copenhagen's 10 official bydele, plus Frederiksberg as its own municipality — boundaries from Københavns Kommune's open-data service, since OpenStreetMap doesn't carry them.)*

A full write-up of the pipeline is coming to the blog. This page is regenerated from a small Python/geopandas script and will be updated as coverage grows.
