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

The street network (all walkable ways, including footpaths through parks) is pulled from OpenStreetMap and resampled into a point every ~20m. A point counts as covered as soon as one of my GPS tracks passes within a small buffer of it — tracks come from my own recorded runs and hikes, plus my full Strava activity history filtered down to on-foot activities that actually took place in Denmark.

### Current coverage

**1,110 km done out of 5,178 km of street network (21.4%).**

| District | Done (km) | Total (km) | % done |
|---|---:|---:|---:|
| Nørrebro | 120.8 | 275.6 | 43.8% |
| Indre By | 237.6 | 602.0 | 39.5% |
| Østerbro | 159.1 | 506.3 | 31.4% |
| Amager Vest | 195.3 | 671.6 | 29.1% |
| Frederiksberg | 127.1 | 606.2 | 21.0% |
| Vesterbro-Kongens Enghave | 99.5 | 529.4 | 18.8% |
| Amager Øst | 62.1 | 435.9 | 14.2% |
| Bispebjerg | 39.0 | 405.0 | 9.6% |
| Valby | 31.6 | 457.9 | 6.9% |
| Brønshøj-Husum | 20.7 | 368.7 | 5.6% |
| Vanløse | 17.1 | 319.1 | 5.4% |

*(Copenhagen's 10 official bydele, plus Frederiksberg as its own municipality — boundaries from Københavns Kommune's open-data service, since OpenStreetMap doesn't carry them.)*

A full write-up of the pipeline is coming to the blog. This page is regenerated from a small Python/geopandas script and will be updated as coverage grows.
