---
layout: post
title:  "EV_CPH — Every Street, Copenhagen"
permalink: /EV_CPH/
date:   2026-07-31
desc: "Tracking which streets of Copenhagen and Frederiksberg I've actually walked, run, or hiked"
keywords: "Clement,Cherblanc,Copenhagen,Frederiksberg,every street,running,hiking,GIS,python"
categories: [Python]
tags: [python,gis,running,]
icon: icon-html
---

Inspired by the "every street" mapping challenge, this project tracks exactly which streets and paths of Copenhagen and Frederiksberg I've covered on foot — running, hiking, or walking — and by how much.

<img src="{{ "/static/assets/img/landing/ev_cph_coverage.png" | prepend: site.baseurl }}" width="100%">

<p class="text-center text-muted">Red = covered at least once. Grey = not yet. Snapshot as of 31 July 2026.</p>

### How it works

The street network (all walkable ways, including footpaths through parks and separated bike lanes) is pulled from OpenStreetMap and resampled into a point every ~10m. A point counts as covered as soon as one of my GPS tracks passes within a small buffer of it — tracks come from my own recorded runs and hikes, plus my full Strava activity history filtered down to on-foot activities that actually took place in Denmark.

### Current coverage

**939 km done out of 5,322 km of street network (17.6%).**

| District | Done (km) | Total (km) | % done |
|---|---:|---:|---:|
| Nørrebro | 97.1 | 278.6 | 34.9% |
| Indre By | 183.0 | 571.7 | 32.0% |
| Amager Vest | 225.1 | 820.5 | 27.4% |
| Østerbro | 123.0 | 498.6 | 24.7% |
| Frederiksberg | 93.8 | 578.8 | 16.2% |
| Vesterbro-Kongens Enghave | 62.0 | 539.9 | 11.5% |
| Amager Øst | 48.5 | 445.0 | 10.9% |
| Bispebjerg | 38.4 | 404.0 | 9.5% |
| Brønshøj-Husum | 28.4 | 397.6 | 7.2% |
| Valby | 25.4 | 469.5 | 5.4% |
| Vanløse | 14.5 | 317.9 | 4.6% |

*(Copenhagen's 10 official bydele, plus Frederiksberg as its own municipality — boundaries from Københavns Kommune's open-data service, since OpenStreetMap doesn't carry them.)*

A full write-up of the pipeline is coming to the blog. This page is regenerated from a small Python/geopandas script and will be updated as coverage grows.
