```python
import xarray as xr
import rioxarray # for the .rio functions
import matplotlib.pyplot as plt
import numpy as np

from pyproj import Transformer
from scipy.interpolate import RegularGridInterpolator, griddata
```

# Procedures and good habits when writting and reading large geo-spatio-temporal datasets

We deal with large datasets, often time in netCDF format, these datasets span space and time, with associated coordinate systems. Proper handling of those coordinates is essential to reduce error and facilitate user-friendliness, which remains a key component of our work, our science needs to be useful, in perticular to others. Knowledge of the global temperature in ten years is only useful if you can use it to draw conclusions on other processes, in other fields of sciences. This document focuses on proper encoding of the spatial Coordinate Reference System (CRS) and data formatting, based on pre-existing conventions, specifically the [NetCDF Climate and Forecast (CF) Metadata Convention](https://cfconventions.org/Data/cf-conventions/cf-conventions-1.11/cf-conventions.pdf).

**TL/DR**: when encoding a dataset, this one needs a CRS (Coordinate Reference System) encoded along with coordinates in that CRS. A CRS_WKT (Well Known Text) is an efficient way to encode key information in all files. Make sure your datasets have that CRS/proper grid/WKT encoded in, whether it is a model output or a dataset you created for your own use. If you read someone else's file with a compromised/missing CRS, here are some methods to help you.


```python
ds = xr.open_dataset('/dmidata/projects/nckf/smb_hirham/ANT11_eval_ERA5/1999/01/01/00/ts_sfx_ANT11_ANT11_eval_ERA5_1hr_199901010000-199902010000.nc')
ds
```




<div><svg style="position: absolute; width: 0; height: 0; overflow: hidden">
<defs>
<symbol id="icon-database" viewBox="0 0 32 32">
<path d="M16 0c-8.837 0-16 2.239-16 5v4c0 2.761 7.163 5 16 5s16-2.239 16-5v-4c0-2.761-7.163-5-16-5z"></path>
<path d="M16 17c-8.837 0-16-2.239-16-5v6c0 2.761 7.163 5 16 5s16-2.239 16-5v-6c0 2.761-7.163 5-16 5z"></path>
<path d="M16 26c-8.837 0-16-2.239-16-5v6c0 2.761 7.163 5 16 5s16-2.239 16-5v-6c0 2.761-7.163 5-16 5z"></path>
</symbol>
<symbol id="icon-file-text2" viewBox="0 0 32 32">
<path d="M28.681 7.159c-0.694-0.947-1.662-2.053-2.724-3.116s-2.169-2.030-3.116-2.724c-1.612-1.182-2.393-1.319-2.841-1.319h-15.5c-1.378 0-2.5 1.121-2.5 2.5v27c0 1.378 1.122 2.5 2.5 2.5h23c1.378 0 2.5-1.122 2.5-2.5v-19.5c0-0.448-0.137-1.23-1.319-2.841zM24.543 5.457c0.959 0.959 1.712 1.825 2.268 2.543h-4.811v-4.811c0.718 0.556 1.584 1.309 2.543 2.268zM28 29.5c0 0.271-0.229 0.5-0.5 0.5h-23c-0.271 0-0.5-0.229-0.5-0.5v-27c0-0.271 0.229-0.5 0.5-0.5 0 0 15.499-0 15.5 0v7c0 0.552 0.448 1 1 1h7v19.5z"></path>
<path d="M23 26h-14c-0.552 0-1-0.448-1-1s0.448-1 1-1h14c0.552 0 1 0.448 1 1s-0.448 1-1 1z"></path>
<path d="M23 22h-14c-0.552 0-1-0.448-1-1s0.448-1 1-1h14c0.552 0 1 0.448 1 1s-0.448 1-1 1z"></path>
<path d="M23 18h-14c-0.552 0-1-0.448-1-1s0.448-1 1-1h14c0.552 0 1 0.448 1 1s-0.448 1-1 1z"></path>
</symbol>
</defs>
</svg>
<style>/* CSS stylesheet for displaying xarray objects in jupyterlab.
 *
 */

:root {
  --xr-font-color0: var(--jp-content-font-color0, rgba(0, 0, 0, 1));
  --xr-font-color2: var(--jp-content-font-color2, rgba(0, 0, 0, 0.54));
  --xr-font-color3: var(--jp-content-font-color3, rgba(0, 0, 0, 0.38));
  --xr-border-color: var(--jp-border-color2, #e0e0e0);
  --xr-disabled-color: var(--jp-layout-color3, #bdbdbd);
  --xr-background-color: var(--jp-layout-color0, white);
  --xr-background-color-row-even: var(--jp-layout-color1, white);
  --xr-background-color-row-odd: var(--jp-layout-color2, #eeeeee);
}

html[theme=dark],
body[data-theme=dark],
body.vscode-dark {
  --xr-font-color0: rgba(255, 255, 255, 1);
  --xr-font-color2: rgba(255, 255, 255, 0.54);
  --xr-font-color3: rgba(255, 255, 255, 0.38);
  --xr-border-color: #1F1F1F;
  --xr-disabled-color: #515151;
  --xr-background-color: #111111;
  --xr-background-color-row-even: #111111;
  --xr-background-color-row-odd: #313131;
}

.xr-wrap {
  display: block !important;
  min-width: 300px;
  max-width: 700px;
}

.xr-text-repr-fallback {
  /* fallback to plain text repr when CSS is not injected (untrusted notebook) */
  display: none;
}

.xr-header {
  padding-top: 6px;
  padding-bottom: 6px;
  margin-bottom: 4px;
  border-bottom: solid 1px var(--xr-border-color);
}

.xr-header > div,
.xr-header > ul {
  display: inline;
  margin-top: 0;
  margin-bottom: 0;
}

.xr-obj-type,
.xr-array-name {
  margin-left: 2px;
  margin-right: 10px;
}

.xr-obj-type {
  color: var(--xr-font-color2);
}

.xr-sections {
  padding-left: 0 !important;
  display: grid;
  grid-template-columns: 150px auto auto 1fr 20px 20px;
}

.xr-section-item {
  display: contents;
}

.xr-section-item input {
  display: none;
}

.xr-section-item input + label {
  color: var(--xr-disabled-color);
}

.xr-section-item input:enabled + label {
  cursor: pointer;
  color: var(--xr-font-color2);
}

.xr-section-item input:enabled + label:hover {
  color: var(--xr-font-color0);
}

.xr-section-summary {
  grid-column: 1;
  color: var(--xr-font-color2);
  font-weight: 500;
}

.xr-section-summary > span {
  display: inline-block;
  padding-left: 0.5em;
}

.xr-section-summary-in:disabled + label {
  color: var(--xr-font-color2);
}

.xr-section-summary-in + label:before {
  display: inline-block;
  content: '►';
  font-size: 11px;
  width: 15px;
  text-align: center;
}

.xr-section-summary-in:disabled + label:before {
  color: var(--xr-disabled-color);
}

.xr-section-summary-in:checked + label:before {
  content: '▼';
}

.xr-section-summary-in:checked + label > span {
  display: none;
}

.xr-section-summary,
.xr-section-inline-details {
  padding-top: 4px;
  padding-bottom: 4px;
}

.xr-section-inline-details {
  grid-column: 2 / -1;
}

.xr-section-details {
  display: none;
  grid-column: 1 / -1;
  margin-bottom: 5px;
}

.xr-section-summary-in:checked ~ .xr-section-details {
  display: contents;
}

.xr-array-wrap {
  grid-column: 1 / -1;
  display: grid;
  grid-template-columns: 20px auto;
}

.xr-array-wrap > label {
  grid-column: 1;
  vertical-align: top;
}

.xr-preview {
  color: var(--xr-font-color3);
}

.xr-array-preview,
.xr-array-data {
  padding: 0 5px !important;
  grid-column: 2;
}

.xr-array-data,
.xr-array-in:checked ~ .xr-array-preview {
  display: none;
}

.xr-array-in:checked ~ .xr-array-data,
.xr-array-preview {
  display: inline-block;
}

.xr-dim-list {
  display: inline-block !important;
  list-style: none;
  padding: 0 !important;
  margin: 0;
}

.xr-dim-list li {
  display: inline-block;
  padding: 0;
  margin: 0;
}

.xr-dim-list:before {
  content: '(';
}

.xr-dim-list:after {
  content: ')';
}

.xr-dim-list li:not(:last-child):after {
  content: ',';
  padding-right: 5px;
}

.xr-has-index {
  font-weight: bold;
}

.xr-var-list,
.xr-var-item {
  display: contents;
}

.xr-var-item > div,
.xr-var-item label,
.xr-var-item > .xr-var-name span {
  background-color: var(--xr-background-color-row-even);
  margin-bottom: 0;
}

.xr-var-item > .xr-var-name:hover span {
  padding-right: 5px;
}

.xr-var-list > li:nth-child(odd) > div,
.xr-var-list > li:nth-child(odd) > label,
.xr-var-list > li:nth-child(odd) > .xr-var-name span {
  background-color: var(--xr-background-color-row-odd);
}

.xr-var-name {
  grid-column: 1;
}

.xr-var-dims {
  grid-column: 2;
}

.xr-var-dtype {
  grid-column: 3;
  text-align: right;
  color: var(--xr-font-color2);
}

.xr-var-preview {
  grid-column: 4;
}

.xr-index-preview {
  grid-column: 2 / 5;
  color: var(--xr-font-color2);
}

.xr-var-name,
.xr-var-dims,
.xr-var-dtype,
.xr-preview,
.xr-attrs dt {
  white-space: nowrap;
  overflow: hidden;
  text-overflow: ellipsis;
  padding-right: 10px;
}

.xr-var-name:hover,
.xr-var-dims:hover,
.xr-var-dtype:hover,
.xr-attrs dt:hover {
  overflow: visible;
  width: auto;
  z-index: 1;
}

.xr-var-attrs,
.xr-var-data,
.xr-index-data {
  display: none;
  background-color: var(--xr-background-color) !important;
  padding-bottom: 5px !important;
}

.xr-var-attrs-in:checked ~ .xr-var-attrs,
.xr-var-data-in:checked ~ .xr-var-data,
.xr-index-data-in:checked ~ .xr-index-data {
  display: block;
}

.xr-var-data > table {
  float: right;
}

.xr-var-name span,
.xr-var-data,
.xr-index-name div,
.xr-index-data,
.xr-attrs {
  padding-left: 25px !important;
}

.xr-attrs,
.xr-var-attrs,
.xr-var-data,
.xr-index-data {
  grid-column: 1 / -1;
}

dl.xr-attrs {
  padding: 0;
  margin: 0;
  display: grid;
  grid-template-columns: 125px auto;
}

.xr-attrs dt,
.xr-attrs dd {
  padding: 0;
  margin: 0;
  float: left;
  padding-right: 10px;
  width: auto;
}

.xr-attrs dt {
  font-weight: normal;
  grid-column: 1;
}

.xr-attrs dt:hover span {
  display: inline-block;
  background: var(--xr-background-color);
  padding-right: 10px;
}

.xr-attrs dd {
  grid-column: 2;
  white-space: pre-wrap;
  word-break: break-all;
}

.xr-icon-database,
.xr-icon-file-text2,
.xr-no-icon {
  display: inline-block;
  vertical-align: middle;
  width: 1em;
  height: 1.5em !important;
  stroke-width: 0;
  stroke: currentColor;
  fill: currentColor;
}
</style><pre class='xr-text-repr-fallback'>&lt;xarray.Dataset&gt;
Dimensions:              (y: 637, x: 739, time: 745)
Coordinates:
  * y                    (y) float64 0.0 1.1e+04 2.2e+04 ... 6.985e+06 6.996e+06
  * x                    (x) float64 0.0 1.1e+04 2.2e+04 ... 8.107e+06 8.118e+06
  * time                 (time) datetime64[ns] 1999-01-01 ... 1999-02-01
    lon                  (y, x) float64 ...
    lat                  (y, x) float64 ...
Data variables:
    Polar_Stereographic  |S1 ...
    ts                   (time, y, x) float32 ...
Attributes:
    Conventions:       CF-1.4
    institute_id:      HCLIMcom
    model_id:          HCLIM
    experiment_id:     ANT11_eval_ERA5
    domain:            ANT11
    frequency:         1hr
    driving_model_id:  ERA5
    creation_date:     Sun Mar 31 03:23:30 2024
    title:             Surface Temperature
    comment:           Created with gl/xtool</pre><div class='xr-wrap' style='display:none'><div class='xr-header'><div class='xr-obj-type'>xarray.Dataset</div></div><ul class='xr-sections'><li class='xr-section-item'><input id='section-93e2e899-70fb-47e9-a460-632787d14f5e' class='xr-section-summary-in' type='checkbox' disabled ><label for='section-93e2e899-70fb-47e9-a460-632787d14f5e' class='xr-section-summary'  title='Expand/collapse section'>Dimensions:</label><div class='xr-section-inline-details'><ul class='xr-dim-list'><li><span class='xr-has-index'>y</span>: 637</li><li><span class='xr-has-index'>x</span>: 739</li><li><span class='xr-has-index'>time</span>: 745</li></ul></div><div class='xr-section-details'></div></li><li class='xr-section-item'><input id='section-b5f8fcbd-5824-4c19-a819-b1cfaec9bb71' class='xr-section-summary-in' type='checkbox'  checked><label for='section-b5f8fcbd-5824-4c19-a819-b1cfaec9bb71' class='xr-section-summary' >Coordinates: <span>(5)</span></label><div class='xr-section-inline-details'></div><div class='xr-section-details'><ul class='xr-var-list'><li class='xr-var-item'><div class='xr-var-name'><span class='xr-has-index'>y</span></div><div class='xr-var-dims'>(y)</div><div class='xr-var-dtype'>float64</div><div class='xr-var-preview xr-preview'>0.0 1.1e+04 ... 6.985e+06 6.996e+06</div><input id='attrs-7a2dfc46-079e-4e11-89bf-17a470f59011' class='xr-var-attrs-in' type='checkbox' ><label for='attrs-7a2dfc46-079e-4e11-89bf-17a470f59011' title='Show/Hide attributes'><svg class='icon xr-icon-file-text2'><use xlink:href='#icon-file-text2'></use></svg></label><input id='data-ba2152c0-8601-4623-b80b-75ef7f316f6b' class='xr-var-data-in' type='checkbox'><label for='data-ba2152c0-8601-4623-b80b-75ef7f316f6b' title='Show/Hide data repr'><svg class='icon xr-icon-database'><use xlink:href='#icon-database'></use></svg></label><div class='xr-var-attrs'><dl class='xr-attrs'><dt><span>axis :</span></dt><dd>Y</dd><dt><span>standard_name :</span></dt><dd>projection_y_coordinate</dd><dt><span>long_name :</span></dt><dd>Y Coordinate Of Projection</dd><dt><span>units :</span></dt><dd>m</dd></dl></div><div class='xr-var-data'><pre>array([      0.,   11000.,   22000., ..., 6974000., 6985000., 6996000.])</pre></div></li><li class='xr-var-item'><div class='xr-var-name'><span class='xr-has-index'>x</span></div><div class='xr-var-dims'>(x)</div><div class='xr-var-dtype'>float64</div><div class='xr-var-preview xr-preview'>0.0 1.1e+04 ... 8.107e+06 8.118e+06</div><input id='attrs-f21ad9df-e270-4a10-a1e4-4d7b958c9995' class='xr-var-attrs-in' type='checkbox' ><label for='attrs-f21ad9df-e270-4a10-a1e4-4d7b958c9995' title='Show/Hide attributes'><svg class='icon xr-icon-file-text2'><use xlink:href='#icon-file-text2'></use></svg></label><input id='data-9fd138c2-e91d-4d3d-9af3-16bfd0941413' class='xr-var-data-in' type='checkbox'><label for='data-9fd138c2-e91d-4d3d-9af3-16bfd0941413' title='Show/Hide data repr'><svg class='icon xr-icon-database'><use xlink:href='#icon-database'></use></svg></label><div class='xr-var-attrs'><dl class='xr-attrs'><dt><span>axis :</span></dt><dd>X</dd><dt><span>standard_name :</span></dt><dd>projection_x_coordinate</dd><dt><span>long_name :</span></dt><dd>X Coordinate Of Projection</dd><dt><span>units :</span></dt><dd>m</dd></dl></div><div class='xr-var-data'><pre>array([      0.,   11000.,   22000., ..., 8096000., 8107000., 8118000.])</pre></div></li><li class='xr-var-item'><div class='xr-var-name'><span class='xr-has-index'>time</span></div><div class='xr-var-dims'>(time)</div><div class='xr-var-dtype'>datetime64[ns]</div><div class='xr-var-preview xr-preview'>1999-01-01 ... 1999-02-01</div><input id='attrs-cae00db7-fd66-40da-8f2c-c9f599c9fa5e' class='xr-var-attrs-in' type='checkbox' ><label for='attrs-cae00db7-fd66-40da-8f2c-c9f599c9fa5e' title='Show/Hide attributes'><svg class='icon xr-icon-file-text2'><use xlink:href='#icon-file-text2'></use></svg></label><input id='data-88182b72-0496-46ee-9279-b2496f7eaffb' class='xr-var-data-in' type='checkbox'><label for='data-88182b72-0496-46ee-9279-b2496f7eaffb' title='Show/Hide data repr'><svg class='icon xr-icon-database'><use xlink:href='#icon-database'></use></svg></label><div class='xr-var-attrs'><dl class='xr-attrs'><dt><span>standard_name :</span></dt><dd>time</dd><dt><span>long_name :</span></dt><dd>Time</dd><dt><span>_CoordinateAxisType :</span></dt><dd>Time</dd></dl></div><div class='xr-var-data'><pre>array([&#x27;1999-01-01T00:00:00.000000000&#x27;, &#x27;1999-01-01T01:00:00.000000000&#x27;,
       &#x27;1999-01-01T02:00:00.000000000&#x27;, ..., &#x27;1999-01-31T22:00:00.000000000&#x27;,
       &#x27;1999-01-31T23:00:00.000000000&#x27;, &#x27;1999-02-01T00:00:00.000000000&#x27;],
      dtype=&#x27;datetime64[ns]&#x27;)</pre></div></li><li class='xr-var-item'><div class='xr-var-name'><span>lon</span></div><div class='xr-var-dims'>(y, x)</div><div class='xr-var-dtype'>float64</div><div class='xr-var-preview xr-preview'>...</div><input id='attrs-fbed53c7-0f4d-43c3-9011-8fe4b376bc84' class='xr-var-attrs-in' type='checkbox' ><label for='attrs-fbed53c7-0f4d-43c3-9011-8fe4b376bc84' title='Show/Hide attributes'><svg class='icon xr-icon-file-text2'><use xlink:href='#icon-file-text2'></use></svg></label><input id='data-d828c54d-acde-489c-89ce-01ec41f4e34a' class='xr-var-data-in' type='checkbox'><label for='data-d828c54d-acde-489c-89ce-01ec41f4e34a' title='Show/Hide data repr'><svg class='icon xr-icon-database'><use xlink:href='#icon-database'></use></svg></label><div class='xr-var-attrs'><dl class='xr-attrs'><dt><span>standard_name :</span></dt><dd>longitude</dd><dt><span>long_name :</span></dt><dd>Longitude</dd><dt><span>units :</span></dt><dd>degrees_east</dd><dt><span>_CoordinateAxisType :</span></dt><dd>Lon</dd></dl></div><div class='xr-var-data'><pre>[470743 values with dtype=float64]</pre></div></li><li class='xr-var-item'><div class='xr-var-name'><span>lat</span></div><div class='xr-var-dims'>(y, x)</div><div class='xr-var-dtype'>float64</div><div class='xr-var-preview xr-preview'>...</div><input id='attrs-0578e7b5-225d-440a-ae42-38631501c0e3' class='xr-var-attrs-in' type='checkbox' ><label for='attrs-0578e7b5-225d-440a-ae42-38631501c0e3' title='Show/Hide attributes'><svg class='icon xr-icon-file-text2'><use xlink:href='#icon-file-text2'></use></svg></label><input id='data-a5977794-f2ea-4c6d-b088-3dccdc0cb7e4' class='xr-var-data-in' type='checkbox'><label for='data-a5977794-f2ea-4c6d-b088-3dccdc0cb7e4' title='Show/Hide data repr'><svg class='icon xr-icon-database'><use xlink:href='#icon-database'></use></svg></label><div class='xr-var-attrs'><dl class='xr-attrs'><dt><span>standard_name :</span></dt><dd>latitude</dd><dt><span>long_name :</span></dt><dd>Latitude</dd><dt><span>units :</span></dt><dd>degrees_north</dd><dt><span>_CoordinateAxisType :</span></dt><dd>Lat</dd></dl></div><div class='xr-var-data'><pre>[470743 values with dtype=float64]</pre></div></li></ul></div></li><li class='xr-section-item'><input id='section-fa66418e-d38c-459c-83d4-eba268465859' class='xr-section-summary-in' type='checkbox'  checked><label for='section-fa66418e-d38c-459c-83d4-eba268465859' class='xr-section-summary' >Data variables: <span>(2)</span></label><div class='xr-section-inline-details'></div><div class='xr-section-details'><ul class='xr-var-list'><li class='xr-var-item'><div class='xr-var-name'><span>Polar_Stereographic</span></div><div class='xr-var-dims'>()</div><div class='xr-var-dtype'>|S1</div><div class='xr-var-preview xr-preview'>...</div><input id='attrs-a771ae10-c78d-4562-97bc-ffcf5a11287f' class='xr-var-attrs-in' type='checkbox' ><label for='attrs-a771ae10-c78d-4562-97bc-ffcf5a11287f' title='Show/Hide attributes'><svg class='icon xr-icon-file-text2'><use xlink:href='#icon-file-text2'></use></svg></label><input id='data-9b1bf73b-9327-4426-b669-00f42d9359bc' class='xr-var-data-in' type='checkbox'><label for='data-9b1bf73b-9327-4426-b669-00f42d9359bc' title='Show/Hide data repr'><svg class='icon xr-icon-database'><use xlink:href='#icon-database'></use></svg></label><div class='xr-var-attrs'><dl class='xr-attrs'><dt><span>grid_mapping_name :</span></dt><dd>polar_stereographic</dd><dt><span>straight_vertical_longitude_from_pole :</span></dt><dd>20.0</dd><dt><span>latitude_of_projection_origin :</span></dt><dd>-90.0</dd><dt><span>standard_parallel :</span></dt><dd>-90.0</dd><dt><span>false_easting :</span></dt><dd>4416613.428326072</dd><dt><span>false_northing :</span></dt><dd>3071812.9120324715</dd><dt><span>earth_radius :</span></dt><dd>6371229.0</dd><dt><span>proj4 :</span></dt><dd>+proj=stere +lat_0=-90.000000 +lat_ts=-90.000000 +lon_0=20.000000 +k_0=1.0 +x_0=4416613.428326 +y_0=3071812.912032 +a=6371229.000000 +b=6371229.000000</dd></dl></div><div class='xr-var-data'><pre>[1 values with dtype=|S1]</pre></div></li><li class='xr-var-item'><div class='xr-var-name'><span>ts</span></div><div class='xr-var-dims'>(time, y, x)</div><div class='xr-var-dtype'>float32</div><div class='xr-var-preview xr-preview'>...</div><input id='attrs-10b6dd4d-040c-49be-8c51-cecf4750f21d' class='xr-var-attrs-in' type='checkbox' ><label for='attrs-10b6dd4d-040c-49be-8c51-cecf4750f21d' title='Show/Hide attributes'><svg class='icon xr-icon-file-text2'><use xlink:href='#icon-file-text2'></use></svg></label><input id='data-1c7d0d32-ef2a-4520-b601-1275bb40e041' class='xr-var-data-in' type='checkbox'><label for='data-1c7d0d32-ef2a-4520-b601-1275bb40e041' title='Show/Hide data repr'><svg class='icon xr-icon-database'><use xlink:href='#icon-database'></use></svg></label><div class='xr-var-attrs'><dl class='xr-attrs'><dt><span>standard_name :</span></dt><dd>surface_temperature</dd><dt><span>long_name :</span></dt><dd>Surface Temperature</dd><dt><span>units :</span></dt><dd>K</dd><dt><span>FA_name :</span></dt><dd>SFX.TS</dd><dt><span>par :</span></dt><dd>11</dd><dt><span>lvt :</span></dt><dd>heightAboveGround</dd><dt><span>lev :</span></dt><dd>0</dd><dt><span>tri :</span></dt><dd>0</dd><dt><span>grid_mapping :</span></dt><dd>Polar_Stereographic</dd><dt><span>cell_methods :</span></dt><dd>time: point</dd></dl></div><div class='xr-var-data'><pre>[350703535 values with dtype=float32]</pre></div></li></ul></div></li><li class='xr-section-item'><input id='section-c02ec854-2f6c-4dca-be8e-147bc4a08e22' class='xr-section-summary-in' type='checkbox'  ><label for='section-c02ec854-2f6c-4dca-be8e-147bc4a08e22' class='xr-section-summary' >Indexes: <span>(3)</span></label><div class='xr-section-inline-details'></div><div class='xr-section-details'><ul class='xr-var-list'><li class='xr-var-item'><div class='xr-index-name'><div>y</div></div><div class='xr-index-preview'>PandasIndex</div><div></div><input id='index-40484bd1-e994-4b28-9a95-06a4392baf7a' class='xr-index-data-in' type='checkbox'/><label for='index-40484bd1-e994-4b28-9a95-06a4392baf7a' title='Show/Hide index repr'><svg class='icon xr-icon-database'><use xlink:href='#icon-database'></use></svg></label><div class='xr-index-data'><pre>PandasIndex(Index([      0.0,   11000.0,   22000.0,   33000.0,   44000.0,   55000.0,
         66000.0,   77000.0,   88000.0,   99000.0,
       ...
       6897000.0, 6908000.0, 6919000.0, 6930000.0, 6941000.0, 6952000.0,
       6963000.0, 6974000.0, 6985000.0, 6996000.0],
      dtype=&#x27;float64&#x27;, name=&#x27;y&#x27;, length=637))</pre></div></li><li class='xr-var-item'><div class='xr-index-name'><div>x</div></div><div class='xr-index-preview'>PandasIndex</div><div></div><input id='index-bf30fac4-1958-4ae9-b9d2-72a2f460b88c' class='xr-index-data-in' type='checkbox'/><label for='index-bf30fac4-1958-4ae9-b9d2-72a2f460b88c' title='Show/Hide index repr'><svg class='icon xr-icon-database'><use xlink:href='#icon-database'></use></svg></label><div class='xr-index-data'><pre>PandasIndex(Index([      0.0,   11000.0,   22000.0,   33000.0,   44000.0,   55000.0,
         66000.0,   77000.0,   88000.0,   99000.0,
       ...
       8019000.0, 8030000.0, 8041000.0, 8052000.0, 8063000.0, 8074000.0,
       8085000.0, 8096000.0, 8107000.0, 8118000.0],
      dtype=&#x27;float64&#x27;, name=&#x27;x&#x27;, length=739))</pre></div></li><li class='xr-var-item'><div class='xr-index-name'><div>time</div></div><div class='xr-index-preview'>PandasIndex</div><div></div><input id='index-ec27729f-242f-43bb-9fa9-1b40abb8fa6a' class='xr-index-data-in' type='checkbox'/><label for='index-ec27729f-242f-43bb-9fa9-1b40abb8fa6a' title='Show/Hide index repr'><svg class='icon xr-icon-database'><use xlink:href='#icon-database'></use></svg></label><div class='xr-index-data'><pre>PandasIndex(DatetimeIndex([&#x27;1999-01-01 00:00:00&#x27;, &#x27;1999-01-01 01:00:00&#x27;,
               &#x27;1999-01-01 02:00:00&#x27;, &#x27;1999-01-01 03:00:00&#x27;,
               &#x27;1999-01-01 04:00:00&#x27;, &#x27;1999-01-01 05:00:00&#x27;,
               &#x27;1999-01-01 06:00:00&#x27;, &#x27;1999-01-01 07:00:00&#x27;,
               &#x27;1999-01-01 08:00:00&#x27;, &#x27;1999-01-01 09:00:00&#x27;,
               ...
               &#x27;1999-01-31 15:00:00&#x27;, &#x27;1999-01-31 16:00:00&#x27;,
               &#x27;1999-01-31 17:00:00&#x27;, &#x27;1999-01-31 18:00:00&#x27;,
               &#x27;1999-01-31 19:00:00&#x27;, &#x27;1999-01-31 20:00:00&#x27;,
               &#x27;1999-01-31 21:00:00&#x27;, &#x27;1999-01-31 22:00:00&#x27;,
               &#x27;1999-01-31 23:00:00&#x27;, &#x27;1999-02-01 00:00:00&#x27;],
              dtype=&#x27;datetime64[ns]&#x27;, name=&#x27;time&#x27;, length=745, freq=None))</pre></div></li></ul></div></li><li class='xr-section-item'><input id='section-d4d8c7ed-4e99-4e13-abd2-d38fab247ee6' class='xr-section-summary-in' type='checkbox'  ><label for='section-d4d8c7ed-4e99-4e13-abd2-d38fab247ee6' class='xr-section-summary' >Attributes: <span>(10)</span></label><div class='xr-section-inline-details'></div><div class='xr-section-details'><dl class='xr-attrs'><dt><span>Conventions :</span></dt><dd>CF-1.4</dd><dt><span>institute_id :</span></dt><dd>HCLIMcom</dd><dt><span>model_id :</span></dt><dd>HCLIM</dd><dt><span>experiment_id :</span></dt><dd>ANT11_eval_ERA5</dd><dt><span>domain :</span></dt><dd>ANT11</dd><dt><span>frequency :</span></dt><dd>1hr</dd><dt><span>driving_model_id :</span></dt><dd>ERA5</dd><dt><span>creation_date :</span></dt><dd>Sun Mar 31 03:23:30 2024</dd><dt><span>title :</span></dt><dd>Surface Temperature</dd><dt><span>comment :</span></dt><dd>Created with gl/xtool</dd></dl></div></li></ul></div></div>



This is an example dataset from an 11km reanalysis of ERA5. I would like to attract your attention on the Dimensions and Coordinates. There are two dimensions here: *x* and *y*. Coordinates are that *x* and *y*, *time* and *lat* and *lon*. We notice right away that *x* and *y* are 1D arrays, vectors in other words, of size 739 and 635 respectively. This indicates a "regular grid", that can be simply be defined by these two vectors. In other words, one row of that grid will have a single *y* value, and one column will have a single *x* value. This regular grid may look exactly like what we need, but we need to make sure that this grid is connected to reality somehow. Indeed, we need to locate this grid with respect to some standards, imagine a coordinate reference system in high school mathematics, you need the coordinates of the origin and its unit vectors. This is the information carried by the CRS, it is a standard way of encoding some coordinate system's key information, it will usually be latitudes/longitudes or in meters, both having their advantages and inconvenients. Thousands of CRS exist, you can find them on [EPSG.io](https://epsg.io/), some will be globally oriented (ex: the modern definition of latitude and longitude is a CRS), some will be more regionally oriented, peticularily the metered CRS (a CRS that uses a unit of meters), they are really accurate at small scale but start being wrong further away from the CRS's origin due to the curvature of the Earth. There are very simple ways to check one datatset's CRS:


```python
print(f"Dataset CRS: {ds.rio.crs}")
```

    Dataset CRS: None


This is problematic, there is no CRS encoded here. A common procedure is to have an Attribute called **CRS_WKT**, standing for Coordinate Reference System Well Known Text, summarizing all the information about the CRS in one spot. More information can be found in the NetCDF CF Metadata Convention document. Here, there is no such attribute because this *x*/*y* grid is actually the grid on which the model was run. **The format of this dataset is standard (Section 5.2 of the NetCDF CF Metadata Conventions), it is output this way such that no alteration of the data or grid is done and the user can choose to make their own changes.** We only have one last source of information: the lat and lon, let's look at them.


```python
fig,axs = plt.subplots(ncols=2, figsize=(15,6))
ds.lat.plot(ax=axs[0])
ds.lon.plot(ax=axs[1])
axs[0].set_title("Latitute")
axs[1].set_title("Longitude")
```




    Text(0.5, 1.0, 'Longitude')




    
![png](https://raw.githubusercontent.com/pokekrom/science/main/BLOG_figures_24-25/2024_geodata_reading_reproject_FIG1.png)
    


We are in a complex situation where the coordinates that make physical sense, that matter to us, are a 2D grid, which does not function well with xarray and netCDFs. Indeed, we cannot just erase *x* and *y*, they are Dimensions and can therefore not be removed, although they have no meaning to anyone else than the person running the model. There are several ways to resolve this.

# 1: Manually create a grid with appropriate coordinates

We will simply reproject the grid of lat/lon onto a grid that suits us better, but without altering the original dataset. For that exercice, I chose to reproject everything onto a metered CRS centered around Antarctica, the [EPSG:3031](https://epsg.io/3031). This method has the advantage of not compromising the data. Interpolation indeed would affect the data, while reprojecting the grid only affect the location of these data.


```python
transformer = Transformer.from_crs(4326, 3031, always_xy=True)
transformer
```




    <Concatenated Operation Transformer: pipeline>
    Description: axis order change (2D) + Antarctic Polar Stereographic
    Area of Use:
    - name: World
    - bounds: (-180.0, -90.0, 180.0, 90.0)




```python
x_reprojected, y_reprojected = transformer.transform(ds.lon, ds.lat)
```


```python
fig,axs = plt.subplots(ncols=2, figsize=(15, 6))
ds.ts[0].plot(ax=axs[0])
axs[1].pcolormesh(x_reprojected, y_reprojected, ds.ts[0])
```




    <matplotlib.collections.QuadMesh at 0x7bd77e9c4b00>




    
![png](https://raw.githubusercontent.com/pokekrom/science/main/BLOG_figures_24-25/2024_geodata_reading_reproject_FIG2.png)
    



```python
## Bonus: finding the x/y of the south pole on the new grid
t = np.unravel_index(np.argmin(ds.lat.values), ds.lat.values.shape)
print(r'Coordinates of the cell closest to the south pole (our grid is 11km spaced, so we are ~4km away from the south pole):', x_reprojected[t], y_reprojected[t])
```

    Coordinates of the cell closest to the south pole (our grid is 11km spaced, so we are ~4km away from the south pole): 4005.7661470373446 -4382.867657798594


I would like to bring your attention on the following:
 - Notice the location of the origin for both grids: 0 is somewhere off in the ocean for the left plot, it is at the south pole in our reprojected grid.
 - The orientation: it changed, both Antarctica has been turned but also our grid coordinates are different, shown by the white triangles on the side.
 - The scale along one axis: Antarctica has somewhat shrinked on the right plot, although this could be a plotting artifact, this is also the result of distances being appropriately represented after using the lat/lon to meters conversion.

The right plot is closer to a usable grid, we can actual start performing certain tasks using the ´x_reprojected´ and ´y_reprojected´ grids, for example extracting the data at a given location (whether it is lat/lon or meters) very easily. 

This method does not affect the data itself, only the location of it, but it takes us away from xarray (example of an extremely powerful and optimised library for this kind of problems), we would like to have a properly set dataset instead of this hybrid numpy/xarray solution, but this only comes at the cost of resampling the data.




```python
######## Bonus: most efficient way to pull data from a list of coordinates:
lat_stations = [-89, -88, -85]
lon_stations = [10,-50,150] # mock numbers

x_stations, y_stations = transformer.transform(lon_stations, lat_stations)

arg_stations = []

for x_, y_ in zip(x_stations, y_stations):
    arg_station = np.argmin(np.ravel(( (x_reprojected-x_)**2 + (y_reprojected-y_)**2)**.5) )   # find the argument of the cell closest to the desired location
    arg_stations.append(np.unravel_index(arg_station, x_reprojected.shape)) 

print('Station indices:', arg_stations)
print('Data at those indices (first time step):', [ds.ts.values[0, *s_] for s_ in arg_stations])
```

    Station indices: [(289, 400), (286, 383), (247, 440)]
    Data at those indices (first time step): [254.09996, 256.95547, 255.72516]


# 2: Resampling the data

If altering the data is not a problem for you; if you want to save the data into your own simple file; if you want to use the full power of xarray on a well organized dataset, you may need to resample the data. To do so, we use the SciPy griddata function.



```python
aspect = ds.y.size / ds.x.size
nx = 700
ny = int(aspect * nx)
shape = (1, ny, nx) # the first dimension is for time
print(f"Target Grid Shape: {shape}")
```

    Target Grid Shape: (1, 603, 700)


## A) Building the Target Grid as a Regular Grid
We want a regular grid where every pixel is the same size in x and y. The y coordinates are flipped
to follow the common convention of having a decreasing y coordinate for raster data.


```python
x_target = np.linspace(x_reprojected.min(), x_reprojected.max(), nx)
y_target = np.linspace(y_reprojected.min(), y_reprojected.max(), ny)[::-1]
X_target, Y_target = np.meshgrid(x_target, y_target)
```


```python
fig, axs = plt.subplots(ncols=2, figsize=(15, 6))
axs[0].imshow(X_target)
axs[0].set_title("X")
axs[1].imshow(Y_target)
axs[1].set_title("Y")
```




    Text(0.5, 1.0, 'Y')




    
![png](https://raw.githubusercontent.com/pokekrom/science/main/BLOG_figures_24-25/2024_geodata_reading_reproject_FIG3.png)
    


## B) Regridding the Data with `griddata`
We need to interpolate the data onto the target grid.


```python
regridded_data = griddata(
    list(zip(x_reprojected.ravel(), y_reprojected.ravel())),
    ds.ts[0].to_numpy().ravel(),
    list(zip(X_target.ravel(), Y_target.ravel())),
    method="cubic"
)
regridded_data = regridded_data.reshape(shape)
```


```python
plt.figure(figsize=(10, 10))
plt.pcolormesh(X_target, Y_target, regridded_data[0])
```




    <matplotlib.collections.QuadMesh at 0x7bd7ff4e3170>




    
![png](https://raw.githubusercontent.com/pokekrom/science/main/BLOG_figures_24-25/2024_geodata_reading_reproject_FIG4.png)
    


## C) Building a New Xarray Object


```python
xdata = xr.DataArray(
    regridded_data,
    dims=("time", "y", "x"),
    coords=(ds.time[:1], y_target, x_target)
).rio.write_crs("EPSG:3031")
xdata
```




<div><svg style="position: absolute; width: 0; height: 0; overflow: hidden">
<defs>
<symbol id="icon-database" viewBox="0 0 32 32">
<path d="M16 0c-8.837 0-16 2.239-16 5v4c0 2.761 7.163 5 16 5s16-2.239 16-5v-4c0-2.761-7.163-5-16-5z"></path>
<path d="M16 17c-8.837 0-16-2.239-16-5v6c0 2.761 7.163 5 16 5s16-2.239 16-5v-6c0 2.761-7.163 5-16 5z"></path>
<path d="M16 26c-8.837 0-16-2.239-16-5v6c0 2.761 7.163 5 16 5s16-2.239 16-5v-6c0 2.761-7.163 5-16 5z"></path>
</symbol>
<symbol id="icon-file-text2" viewBox="0 0 32 32">
<path d="M28.681 7.159c-0.694-0.947-1.662-2.053-2.724-3.116s-2.169-2.030-3.116-2.724c-1.612-1.182-2.393-1.319-2.841-1.319h-15.5c-1.378 0-2.5 1.121-2.5 2.5v27c0 1.378 1.122 2.5 2.5 2.5h23c1.378 0 2.5-1.122 2.5-2.5v-19.5c0-0.448-0.137-1.23-1.319-2.841zM24.543 5.457c0.959 0.959 1.712 1.825 2.268 2.543h-4.811v-4.811c0.718 0.556 1.584 1.309 2.543 2.268zM28 29.5c0 0.271-0.229 0.5-0.5 0.5h-23c-0.271 0-0.5-0.229-0.5-0.5v-27c0-0.271 0.229-0.5 0.5-0.5 0 0 15.499-0 15.5 0v7c0 0.552 0.448 1 1 1h7v19.5z"></path>
<path d="M23 26h-14c-0.552 0-1-0.448-1-1s0.448-1 1-1h14c0.552 0 1 0.448 1 1s-0.448 1-1 1z"></path>
<path d="M23 22h-14c-0.552 0-1-0.448-1-1s0.448-1 1-1h14c0.552 0 1 0.448 1 1s-0.448 1-1 1z"></path>
<path d="M23 18h-14c-0.552 0-1-0.448-1-1s0.448-1 1-1h14c0.552 0 1 0.448 1 1s-0.448 1-1 1z"></path>
</symbol>
</defs>
</svg>
<style>/* CSS stylesheet for displaying xarray objects in jupyterlab.
 *
 */

:root {
  --xr-font-color0: var(--jp-content-font-color0, rgba(0, 0, 0, 1));
  --xr-font-color2: var(--jp-content-font-color2, rgba(0, 0, 0, 0.54));
  --xr-font-color3: var(--jp-content-font-color3, rgba(0, 0, 0, 0.38));
  --xr-border-color: var(--jp-border-color2, #e0e0e0);
  --xr-disabled-color: var(--jp-layout-color3, #bdbdbd);
  --xr-background-color: var(--jp-layout-color0, white);
  --xr-background-color-row-even: var(--jp-layout-color1, white);
  --xr-background-color-row-odd: var(--jp-layout-color2, #eeeeee);
}

html[theme=dark],
body[data-theme=dark],
body.vscode-dark {
  --xr-font-color0: rgba(255, 255, 255, 1);
  --xr-font-color2: rgba(255, 255, 255, 0.54);
  --xr-font-color3: rgba(255, 255, 255, 0.38);
  --xr-border-color: #1F1F1F;
  --xr-disabled-color: #515151;
  --xr-background-color: #111111;
  --xr-background-color-row-even: #111111;
  --xr-background-color-row-odd: #313131;
}

.xr-wrap {
  display: block !important;
  min-width: 300px;
  max-width: 700px;
}

.xr-text-repr-fallback {
  /* fallback to plain text repr when CSS is not injected (untrusted notebook) */
  display: none;
}

.xr-header {
  padding-top: 6px;
  padding-bottom: 6px;
  margin-bottom: 4px;
  border-bottom: solid 1px var(--xr-border-color);
}

.xr-header > div,
.xr-header > ul {
  display: inline;
  margin-top: 0;
  margin-bottom: 0;
}

.xr-obj-type,
.xr-array-name {
  margin-left: 2px;
  margin-right: 10px;
}

.xr-obj-type {
  color: var(--xr-font-color2);
}

.xr-sections {
  padding-left: 0 !important;
  display: grid;
  grid-template-columns: 150px auto auto 1fr 20px 20px;
}

.xr-section-item {
  display: contents;
}

.xr-section-item input {
  display: none;
}

.xr-section-item input + label {
  color: var(--xr-disabled-color);
}

.xr-section-item input:enabled + label {
  cursor: pointer;
  color: var(--xr-font-color2);
}

.xr-section-item input:enabled + label:hover {
  color: var(--xr-font-color0);
}

.xr-section-summary {
  grid-column: 1;
  color: var(--xr-font-color2);
  font-weight: 500;
}

.xr-section-summary > span {
  display: inline-block;
  padding-left: 0.5em;
}

.xr-section-summary-in:disabled + label {
  color: var(--xr-font-color2);
}

.xr-section-summary-in + label:before {
  display: inline-block;
  content: '►';
  font-size: 11px;
  width: 15px;
  text-align: center;
}

.xr-section-summary-in:disabled + label:before {
  color: var(--xr-disabled-color);
}

.xr-section-summary-in:checked + label:before {
  content: '▼';
}

.xr-section-summary-in:checked + label > span {
  display: none;
}

.xr-section-summary,
.xr-section-inline-details {
  padding-top: 4px;
  padding-bottom: 4px;
}

.xr-section-inline-details {
  grid-column: 2 / -1;
}

.xr-section-details {
  display: none;
  grid-column: 1 / -1;
  margin-bottom: 5px;
}

.xr-section-summary-in:checked ~ .xr-section-details {
  display: contents;
}

.xr-array-wrap {
  grid-column: 1 / -1;
  display: grid;
  grid-template-columns: 20px auto;
}

.xr-array-wrap > label {
  grid-column: 1;
  vertical-align: top;
}

.xr-preview {
  color: var(--xr-font-color3);
}

.xr-array-preview,
.xr-array-data {
  padding: 0 5px !important;
  grid-column: 2;
}

.xr-array-data,
.xr-array-in:checked ~ .xr-array-preview {
  display: none;
}

.xr-array-in:checked ~ .xr-array-data,
.xr-array-preview {
  display: inline-block;
}

.xr-dim-list {
  display: inline-block !important;
  list-style: none;
  padding: 0 !important;
  margin: 0;
}

.xr-dim-list li {
  display: inline-block;
  padding: 0;
  margin: 0;
}

.xr-dim-list:before {
  content: '(';
}

.xr-dim-list:after {
  content: ')';
}

.xr-dim-list li:not(:last-child):after {
  content: ',';
  padding-right: 5px;
}

.xr-has-index {
  font-weight: bold;
}

.xr-var-list,
.xr-var-item {
  display: contents;
}

.xr-var-item > div,
.xr-var-item label,
.xr-var-item > .xr-var-name span {
  background-color: var(--xr-background-color-row-even);
  margin-bottom: 0;
}

.xr-var-item > .xr-var-name:hover span {
  padding-right: 5px;
}

.xr-var-list > li:nth-child(odd) > div,
.xr-var-list > li:nth-child(odd) > label,
.xr-var-list > li:nth-child(odd) > .xr-var-name span {
  background-color: var(--xr-background-color-row-odd);
}

.xr-var-name {
  grid-column: 1;
}

.xr-var-dims {
  grid-column: 2;
}

.xr-var-dtype {
  grid-column: 3;
  text-align: right;
  color: var(--xr-font-color2);
}

.xr-var-preview {
  grid-column: 4;
}

.xr-index-preview {
  grid-column: 2 / 5;
  color: var(--xr-font-color2);
}

.xr-var-name,
.xr-var-dims,
.xr-var-dtype,
.xr-preview,
.xr-attrs dt {
  white-space: nowrap;
  overflow: hidden;
  text-overflow: ellipsis;
  padding-right: 10px;
}

.xr-var-name:hover,
.xr-var-dims:hover,
.xr-var-dtype:hover,
.xr-attrs dt:hover {
  overflow: visible;
  width: auto;
  z-index: 1;
}

.xr-var-attrs,
.xr-var-data,
.xr-index-data {
  display: none;
  background-color: var(--xr-background-color) !important;
  padding-bottom: 5px !important;
}

.xr-var-attrs-in:checked ~ .xr-var-attrs,
.xr-var-data-in:checked ~ .xr-var-data,
.xr-index-data-in:checked ~ .xr-index-data {
  display: block;
}

.xr-var-data > table {
  float: right;
}

.xr-var-name span,
.xr-var-data,
.xr-index-name div,
.xr-index-data,
.xr-attrs {
  padding-left: 25px !important;
}

.xr-attrs,
.xr-var-attrs,
.xr-var-data,
.xr-index-data {
  grid-column: 1 / -1;
}

dl.xr-attrs {
  padding: 0;
  margin: 0;
  display: grid;
  grid-template-columns: 125px auto;
}

.xr-attrs dt,
.xr-attrs dd {
  padding: 0;
  margin: 0;
  float: left;
  padding-right: 10px;
  width: auto;
}

.xr-attrs dt {
  font-weight: normal;
  grid-column: 1;
}

.xr-attrs dt:hover span {
  display: inline-block;
  background: var(--xr-background-color);
  padding-right: 10px;
}

.xr-attrs dd {
  grid-column: 2;
  white-space: pre-wrap;
  word-break: break-all;
}

.xr-icon-database,
.xr-icon-file-text2,
.xr-no-icon {
  display: inline-block;
  vertical-align: middle;
  width: 1em;
  height: 1.5em !important;
  stroke-width: 0;
  stroke: currentColor;
  fill: currentColor;
}
</style><pre class='xr-text-repr-fallback'>&lt;xarray.DataArray (time: 1, y: 603, x: 700)&gt;
array([[[nan, nan, nan, ..., nan, nan, nan],
        [nan, nan, nan, ..., nan, nan, nan],
        [nan, nan, nan, ..., nan, nan, nan],
        ...,
        [nan, nan, nan, ..., nan, nan, nan],
        [nan, nan, nan, ..., nan, nan, nan],
        [nan, nan, nan, ..., nan, nan, nan]]])
Coordinates:
  * time         (time) datetime64[ns] 1999-01-01
  * y            (y) float64 5.067e+06 5.052e+06 ... -4.035e+06 -4.051e+06
  * x            (x) float64 -5.071e+06 -5.057e+06 ... 4.686e+06 4.7e+06
    spatial_ref  int64 0</pre><div class='xr-wrap' style='display:none'><div class='xr-header'><div class='xr-obj-type'>xarray.DataArray</div><div class='xr-array-name'></div><ul class='xr-dim-list'><li><span class='xr-has-index'>time</span>: 1</li><li><span class='xr-has-index'>y</span>: 603</li><li><span class='xr-has-index'>x</span>: 700</li></ul></div><ul class='xr-sections'><li class='xr-section-item'><div class='xr-array-wrap'><input id='section-3fc24cab-1ea6-4a33-ae9a-f4297e185ad5' class='xr-array-in' type='checkbox' checked><label for='section-3fc24cab-1ea6-4a33-ae9a-f4297e185ad5' title='Show/hide data repr'><svg class='icon xr-icon-database'><use xlink:href='#icon-database'></use></svg></label><div class='xr-array-preview xr-preview'><span>nan nan nan nan nan nan nan nan ... nan nan nan nan nan nan nan nan</span></div><div class='xr-array-data'><pre>array([[[nan, nan, nan, ..., nan, nan, nan],
        [nan, nan, nan, ..., nan, nan, nan],
        [nan, nan, nan, ..., nan, nan, nan],
        ...,
        [nan, nan, nan, ..., nan, nan, nan],
        [nan, nan, nan, ..., nan, nan, nan],
        [nan, nan, nan, ..., nan, nan, nan]]])</pre></div></div></li><li class='xr-section-item'><input id='section-1b7ad529-dfc9-4232-81e0-0df1aa05e86e' class='xr-section-summary-in' type='checkbox'  checked><label for='section-1b7ad529-dfc9-4232-81e0-0df1aa05e86e' class='xr-section-summary' >Coordinates: <span>(4)</span></label><div class='xr-section-inline-details'></div><div class='xr-section-details'><ul class='xr-var-list'><li class='xr-var-item'><div class='xr-var-name'><span class='xr-has-index'>time</span></div><div class='xr-var-dims'>(time)</div><div class='xr-var-dtype'>datetime64[ns]</div><div class='xr-var-preview xr-preview'>1999-01-01</div><input id='attrs-5bfaa5c1-7f61-4657-b856-a298fb1f39c6' class='xr-var-attrs-in' type='checkbox' ><label for='attrs-5bfaa5c1-7f61-4657-b856-a298fb1f39c6' title='Show/Hide attributes'><svg class='icon xr-icon-file-text2'><use xlink:href='#icon-file-text2'></use></svg></label><input id='data-3bc74ed5-9028-4efe-ae8b-483cf718852c' class='xr-var-data-in' type='checkbox'><label for='data-3bc74ed5-9028-4efe-ae8b-483cf718852c' title='Show/Hide data repr'><svg class='icon xr-icon-database'><use xlink:href='#icon-database'></use></svg></label><div class='xr-var-attrs'><dl class='xr-attrs'><dt><span>standard_name :</span></dt><dd>time</dd><dt><span>long_name :</span></dt><dd>Time</dd><dt><span>_CoordinateAxisType :</span></dt><dd>Time</dd></dl></div><div class='xr-var-data'><pre>array([&#x27;1999-01-01T00:00:00.000000000&#x27;], dtype=&#x27;datetime64[ns]&#x27;)</pre></div></li><li class='xr-var-item'><div class='xr-var-name'><span class='xr-has-index'>y</span></div><div class='xr-var-dims'>(y)</div><div class='xr-var-dtype'>float64</div><div class='xr-var-preview xr-preview'>5.067e+06 5.052e+06 ... -4.051e+06</div><input id='attrs-6d1742c5-e49f-47db-925a-c7fb61086b23' class='xr-var-attrs-in' type='checkbox' disabled><label for='attrs-6d1742c5-e49f-47db-925a-c7fb61086b23' title='Show/Hide attributes'><svg class='icon xr-icon-file-text2'><use xlink:href='#icon-file-text2'></use></svg></label><input id='data-b39faf78-efdb-4ce7-b9df-3a5032f4710b' class='xr-var-data-in' type='checkbox'><label for='data-b39faf78-efdb-4ce7-b9df-3a5032f4710b' title='Show/Hide data repr'><svg class='icon xr-icon-database'><use xlink:href='#icon-database'></use></svg></label><div class='xr-var-attrs'><dl class='xr-attrs'></dl></div><div class='xr-var-data'><pre>array([ 5066989.156886,  5051843.642368,  5036698.127849, ..., -4020319.554211,
       -4035465.068729, -4050610.583248])</pre></div></li><li class='xr-var-item'><div class='xr-var-name'><span class='xr-has-index'>x</span></div><div class='xr-var-dims'>(x)</div><div class='xr-var-dtype'>float64</div><div class='xr-var-preview xr-preview'>-5.071e+06 -5.057e+06 ... 4.7e+06</div><input id='attrs-e8a9a9c8-9a1b-48e4-88df-3f294f3cdef6' class='xr-var-attrs-in' type='checkbox' disabled><label for='attrs-e8a9a9c8-9a1b-48e4-88df-3f294f3cdef6' title='Show/Hide attributes'><svg class='icon xr-icon-file-text2'><use xlink:href='#icon-file-text2'></use></svg></label><input id='data-e5971868-87e9-4a3e-9503-090e1a32cf2e' class='xr-var-data-in' type='checkbox'><label for='data-e5971868-87e9-4a3e-9503-090e1a32cf2e' title='Show/Hide data repr'><svg class='icon xr-icon-database'><use xlink:href='#icon-database'></use></svg></label><div class='xr-var-attrs'><dl class='xr-attrs'></dl></div><div class='xr-var-data'><pre>array([-5071446.506883, -5057466.884673, -5043487.262462, ...,  4672350.17376 ,
        4686329.79597 ,  4700309.418181])</pre></div></li><li class='xr-var-item'><div class='xr-var-name'><span>spatial_ref</span></div><div class='xr-var-dims'>()</div><div class='xr-var-dtype'>int64</div><div class='xr-var-preview xr-preview'>0</div><input id='attrs-f9a1fc58-f0ad-4c5d-8fdd-e2418ab0a385' class='xr-var-attrs-in' type='checkbox' ><label for='attrs-f9a1fc58-f0ad-4c5d-8fdd-e2418ab0a385' title='Show/Hide attributes'><svg class='icon xr-icon-file-text2'><use xlink:href='#icon-file-text2'></use></svg></label><input id='data-fae7908c-8632-4bc9-9ec4-e9c61ed3227f' class='xr-var-data-in' type='checkbox'><label for='data-fae7908c-8632-4bc9-9ec4-e9c61ed3227f' title='Show/Hide data repr'><svg class='icon xr-icon-database'><use xlink:href='#icon-database'></use></svg></label><div class='xr-var-attrs'><dl class='xr-attrs'><dt><span>crs_wkt :</span></dt><dd>PROJCS[&quot;WGS 84 / Antarctic Polar Stereographic&quot;,GEOGCS[&quot;WGS 84&quot;,DATUM[&quot;WGS_1984&quot;,SPHEROID[&quot;WGS 84&quot;,6378137,298.257223563,AUTHORITY[&quot;EPSG&quot;,&quot;7030&quot;]],AUTHORITY[&quot;EPSG&quot;,&quot;6326&quot;]],PRIMEM[&quot;Greenwich&quot;,0,AUTHORITY[&quot;EPSG&quot;,&quot;8901&quot;]],UNIT[&quot;degree&quot;,0.0174532925199433,AUTHORITY[&quot;EPSG&quot;,&quot;9122&quot;]],AUTHORITY[&quot;EPSG&quot;,&quot;4326&quot;]],PROJECTION[&quot;Polar_Stereographic&quot;],PARAMETER[&quot;latitude_of_origin&quot;,-71],PARAMETER[&quot;central_meridian&quot;,0],PARAMETER[&quot;false_easting&quot;,0],PARAMETER[&quot;false_northing&quot;,0],UNIT[&quot;metre&quot;,1,AUTHORITY[&quot;EPSG&quot;,&quot;9001&quot;]],AXIS[&quot;Easting&quot;,NORTH],AXIS[&quot;Northing&quot;,NORTH],AUTHORITY[&quot;EPSG&quot;,&quot;3031&quot;]]</dd><dt><span>semi_major_axis :</span></dt><dd>6378137.0</dd><dt><span>semi_minor_axis :</span></dt><dd>6356752.314245179</dd><dt><span>inverse_flattening :</span></dt><dd>298.257223563</dd><dt><span>reference_ellipsoid_name :</span></dt><dd>WGS 84</dd><dt><span>longitude_of_prime_meridian :</span></dt><dd>0.0</dd><dt><span>prime_meridian_name :</span></dt><dd>Greenwich</dd><dt><span>geographic_crs_name :</span></dt><dd>WGS 84</dd><dt><span>horizontal_datum_name :</span></dt><dd>World Geodetic System 1984</dd><dt><span>projected_crs_name :</span></dt><dd>WGS 84 / Antarctic Polar Stereographic</dd><dt><span>grid_mapping_name :</span></dt><dd>polar_stereographic</dd><dt><span>standard_parallel :</span></dt><dd>-71.0</dd><dt><span>straight_vertical_longitude_from_pole :</span></dt><dd>0.0</dd><dt><span>false_easting :</span></dt><dd>0.0</dd><dt><span>false_northing :</span></dt><dd>0.0</dd><dt><span>spatial_ref :</span></dt><dd>PROJCS[&quot;WGS 84 / Antarctic Polar Stereographic&quot;,GEOGCS[&quot;WGS 84&quot;,DATUM[&quot;WGS_1984&quot;,SPHEROID[&quot;WGS 84&quot;,6378137,298.257223563,AUTHORITY[&quot;EPSG&quot;,&quot;7030&quot;]],AUTHORITY[&quot;EPSG&quot;,&quot;6326&quot;]],PRIMEM[&quot;Greenwich&quot;,0,AUTHORITY[&quot;EPSG&quot;,&quot;8901&quot;]],UNIT[&quot;degree&quot;,0.0174532925199433,AUTHORITY[&quot;EPSG&quot;,&quot;9122&quot;]],AUTHORITY[&quot;EPSG&quot;,&quot;4326&quot;]],PROJECTION[&quot;Polar_Stereographic&quot;],PARAMETER[&quot;latitude_of_origin&quot;,-71],PARAMETER[&quot;central_meridian&quot;,0],PARAMETER[&quot;false_easting&quot;,0],PARAMETER[&quot;false_northing&quot;,0],UNIT[&quot;metre&quot;,1,AUTHORITY[&quot;EPSG&quot;,&quot;9001&quot;]],AXIS[&quot;Easting&quot;,NORTH],AXIS[&quot;Northing&quot;,NORTH],AUTHORITY[&quot;EPSG&quot;,&quot;3031&quot;]]</dd></dl></div><div class='xr-var-data'><pre>array(0)</pre></div></li></ul></div></li><li class='xr-section-item'><input id='section-8c57f145-d2a0-4286-b7c7-0dc8efd78302' class='xr-section-summary-in' type='checkbox'  ><label for='section-8c57f145-d2a0-4286-b7c7-0dc8efd78302' class='xr-section-summary' >Indexes: <span>(3)</span></label><div class='xr-section-inline-details'></div><div class='xr-section-details'><ul class='xr-var-list'><li class='xr-var-item'><div class='xr-index-name'><div>time</div></div><div class='xr-index-preview'>PandasIndex</div><div></div><input id='index-50753fe7-dc30-4eac-8305-8631fbf4e530' class='xr-index-data-in' type='checkbox'/><label for='index-50753fe7-dc30-4eac-8305-8631fbf4e530' title='Show/Hide index repr'><svg class='icon xr-icon-database'><use xlink:href='#icon-database'></use></svg></label><div class='xr-index-data'><pre>PandasIndex(DatetimeIndex([&#x27;1999-01-01&#x27;], dtype=&#x27;datetime64[ns]&#x27;, name=&#x27;time&#x27;, freq=None))</pre></div></li><li class='xr-var-item'><div class='xr-index-name'><div>y</div></div><div class='xr-index-preview'>PandasIndex</div><div></div><input id='index-f4c9110f-55df-4334-9c45-1784ac132795' class='xr-index-data-in' type='checkbox'/><label for='index-f4c9110f-55df-4334-9c45-1784ac132795' title='Show/Hide index repr'><svg class='icon xr-icon-database'><use xlink:href='#icon-database'></use></svg></label><div class='xr-index-data'><pre>PandasIndex(Index([  5066989.156886292,   5051843.642367795,   5036698.127849301,
         5021552.613330806,    5006407.09881231,   4991261.584293814,
         4976116.069775319,   4960970.555256825,   4945825.040738329,
         4930679.526219834,
       ...
        -3914300.952581398, -3929446.4670998934, -3944591.9816183886,
       -3959737.4961368837,  -3974883.010655379,  -3990028.525173874,
       -4005174.0396923693,  -4020319.554210865,   -4035465.06872936,
       -4050610.5832478553],
      dtype=&#x27;float64&#x27;, name=&#x27;y&#x27;, length=603))</pre></div></li><li class='xr-var-item'><div class='xr-index-name'><div>x</div></div><div class='xr-index-preview'>PandasIndex</div><div></div><input id='index-0e3c9266-89c1-4329-a460-ce00849afb05' class='xr-index-data-in' type='checkbox'/><label for='index-0e3c9266-89c1-4329-a460-ce00849afb05' title='Show/Hide index repr'><svg class='icon xr-icon-database'><use xlink:href='#icon-database'></use></svg></label><div class='xr-index-data'><pre>PandasIndex(Index([ -5071446.506882979,  -5057466.884672587,  -5043487.262462196,
        -5029507.640251804,  -5015528.018041413, -5001548.3958310215,
        -4987568.773620631,  -4973589.151410239,  -4959609.529199847,
        -4945629.906989456,
       ...
        4574492.8182870895,   4588472.440497482,   4602452.062707873,
         4616431.684918264,   4630411.307128657,  4644390.9293390475,
         4658370.551549438,   4672350.173759829,   4686329.795970222,
         4700309.418180612],
      dtype=&#x27;float64&#x27;, name=&#x27;x&#x27;, length=700))</pre></div></li></ul></div></li><li class='xr-section-item'><input id='section-07b1a510-e7fd-4f98-bd96-cd8ba78ac589' class='xr-section-summary-in' type='checkbox' disabled ><label for='section-07b1a510-e7fd-4f98-bd96-cd8ba78ac589' class='xr-section-summary'  title='Expand/collapse section'>Attributes: <span>(0)</span></label><div class='xr-section-inline-details'></div><div class='xr-section-details'><dl class='xr-attrs'></dl></div></li></ul></div></div>




```python
plt.figure(figsize=(10, 8))
xdata.plot(robust=True, cmap="viridis")
```




    <matplotlib.collections.QuadMesh at 0x7bd775fa8710>




    
![png](https://raw.githubusercontent.com/pokekrom/science/main/BLOG_figures_24-25/2024_geodata_reading_reproject_FIG5.png)
    



```python
print(f"Regridded data CRS: {xdata.rio.crs}")
```

    Regridded data CRS: EPSG:3031


The key point here is that you are able to save this data in a new netCDF file because it has a regular grid. Regular examples from the netCDF or xarray library would suffice to do so.
