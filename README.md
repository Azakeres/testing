```python
import pandas as pd
import geopandas as gpd
import numpy as np
from shapely.geometry import Point
import matplotlib.pyplot as plt
import mpld3
```


```python
BC_df = pd.read_csv('IN_Birthcenters.csv')
BC_df["Type"] = "Birth Center"
AC_df = pd.read_csv('IN_AbortionCenters.csv')
AC_df["Type"] = "Abortion Center"
Hosp_df = pd.read_csv('IN_Hosp.csv')
AC_df.head()


```




<div>
<style scoped>
    .dataframe tbody tr th:only-of-type {
        vertical-align: middle;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }

    .dataframe thead th {
        text-align: right;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>NAME</th>
      <th>ADDRESS</th>
      <th>CITY</th>
      <th>STATE ZIP</th>
      <th>Administrator</th>
      <th>Tell</th>
      <th>Fax</th>
      <th>License Number</th>
      <th>Lic Expire Date</th>
      <th>Lat</th>
      <th>Long</th>
      <th>Type</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>CLINIC FOR WOMEN</td>
      <td>3607 W 16TH ST STE 2B</td>
      <td>INDIANAPOLIS</td>
      <td>IN 46222</td>
      <td>LADONNA PRINCE</td>
      <td>(317)955-2641</td>
      <td>(317)955-2687</td>
      <td>19-011133-1</td>
      <td>06/30/2020</td>
      <td>39.787478</td>
      <td>-86.222633</td>
      <td>Abortion Center</td>
    </tr>
    <tr>
      <th>1</th>
      <td>PLANNED PARENTHOOD OF INDIANA AND KENTUCKY INC</td>
      <td>8645 CONNECTICUT ST</td>
      <td>MERRILLVILLE</td>
      <td>IN 46410</td>
      <td>LOLITA KINSEY-BROWN</td>
      <td>(219)769-3500</td>
      <td>(219)791-0538</td>
      <td>19-011116-1</td>
      <td>06/30/2020</td>
      <td>39.763935</td>
      <td>-86.158912</td>
      <td>Abortion Center</td>
    </tr>
    <tr>
      <th>2</th>
      <td>PLANNED PARENTHOOD OF INDIANA AND KENTUCKY INC</td>
      <td>421 S COLLEGE AVE</td>
      <td>BLOOMINGTON</td>
      <td>IN 47403</td>
      <td>MEAGAN COOK</td>
      <td>(812)336-0219</td>
      <td>(812)336-2401</td>
      <td>19-011117-1</td>
      <td>06/30/2020</td>
      <td>39.162239</td>
      <td>-86.536825</td>
      <td>Abortion Center</td>
    </tr>
    <tr>
      <th>3</th>
      <td>PLANNED PARENTHOOD OF INDIANA AND KENTUCKY INC</td>
      <td>8590 GEORGETOWN RD</td>
      <td>INDIANAPOLIS</td>
      <td>IN 46268</td>
      <td>MICHELLE STEELE</td>
      <td>(317)872-3115</td>
      <td>(317)872-3188</td>
      <td>19-011118-1</td>
      <td>06/30/2020</td>
      <td>39.910195</td>
      <td>-86.243884</td>
      <td>Abortion Center</td>
    </tr>
    <tr>
      <th>4</th>
      <td>PLANNED PARENTHOOD OF INDIANA AND KENTUCKY, INC -</td>
      <td>964 MEZZANINE DR</td>
      <td>LAFAYETTE</td>
      <td>IN 47905</td>
      <td>KALILA WEINER</td>
      <td>(765)446-8078</td>
      <td>(765)446-8160</td>
      <td>19-013765-1</td>
      <td>06/30/2020</td>
      <td>40.404664</td>
      <td>-86.835328</td>
      <td>Abortion Center</td>
    </tr>
  </tbody>
</table>
</div>




```python
Hosp_sub = Hosp_df[['NAME','CITY','county','hosp_type','Certified','Lat','Long']]
Hosp_sub.head()
```




<div>
<style scoped>
    .dataframe tbody tr th:only-of-type {
        vertical-align: middle;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }

    .dataframe thead th {
        text-align: right;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>NAME</th>
      <th>CITY</th>
      <th>county</th>
      <th>hosp_type</th>
      <th>Certified</th>
      <th>Lat</th>
      <th>Long</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>NORTHEASTERN CENTER</td>
      <td>AUBURN</td>
      <td>DeKalb</td>
      <td>PSYCHIATRIC</td>
      <td>No</td>
      <td>41.372113</td>
      <td>-85.031248</td>
    </tr>
    <tr>
      <th>1</th>
      <td>INDIANA UNIVERSITY HEALTH BEDFORD HOSPITAL</td>
      <td>BEDFORD</td>
      <td>Lawrence</td>
      <td>CRITICAL ACCESS HOSPITALS</td>
      <td>No</td>
      <td>38.859470</td>
      <td>-86.512933</td>
    </tr>
    <tr>
      <th>2</th>
      <td>ST VINCENT DUNN HOSPITAL INC</td>
      <td>BEDFORD</td>
      <td>Lawrence</td>
      <td>CRITICAL ACCESS HOSPITALS</td>
      <td>No</td>
      <td>38.853326</td>
      <td>-86.494526</td>
    </tr>
    <tr>
      <th>3</th>
      <td>ST VINCENT WARRICK</td>
      <td>BOONVILLE</td>
      <td>Warrick</td>
      <td>CRITICAL ACCESS HOSPITALS</td>
      <td>no</td>
      <td>38.041803</td>
      <td>-87.262921</td>
    </tr>
    <tr>
      <th>4</th>
      <td>DOCTORS NEUROPSYCHIATRIC HOSPITAL</td>
      <td>BREMEN</td>
      <td>Marshall</td>
      <td>PSYCHIATRIC</td>
      <td>no</td>
      <td>41.443581</td>
      <td>-86.150306</td>
    </tr>
  </tbody>
</table>
</div>




```python
baby_friendly = Hosp_sub[Hosp_df['Certified']=="Yes"]
baby_friendly.head()
```




<div>
<style scoped>
    .dataframe tbody tr th:only-of-type {
        vertical-align: middle;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }

    .dataframe thead th {
        text-align: right;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>NAME</th>
      <th>CITY</th>
      <th>county</th>
      <th>hosp_type</th>
      <th>Certified</th>
      <th>Lat</th>
      <th>Long</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>33</th>
      <td>COMMUNITY HOSPITAL OF ANDERSON AND MADISON COUNTY</td>
      <td>ANDERSON</td>
      <td>Madison</td>
      <td>SHORT TERM</td>
      <td>Yes</td>
      <td>40.131330</td>
      <td>-85.693592</td>
    </tr>
    <tr>
      <th>35</th>
      <td>IU HEALTH BLOOMINGTON HOSPITAL</td>
      <td>BLOOMINGTON</td>
      <td>Monroe</td>
      <td>SHORT TERM</td>
      <td>Yes</td>
      <td>39.160071</td>
      <td>-86.540366</td>
    </tr>
    <tr>
      <th>36</th>
      <td>PARKVIEW WHITLEY HOSPITAL</td>
      <td>COLUMBIA CITY</td>
      <td>Whitley</td>
      <td>SHORT TERM</td>
      <td>Yes</td>
      <td>41.158236</td>
      <td>-85.464892</td>
    </tr>
    <tr>
      <th>37</th>
      <td>COLUMBUS REGIONAL HOSPITAL</td>
      <td>COLUMBUS</td>
      <td>Bartholomew</td>
      <td>SHORT TERM</td>
      <td>Yes</td>
      <td>39.216528</td>
      <td>-85.895923</td>
    </tr>
    <tr>
      <th>38</th>
      <td>ELKHART GENERAL HOSPITAL</td>
      <td>ELKHART</td>
      <td>Elkhart</td>
      <td>SHORT TERM</td>
      <td>Yes</td>
      <td>41.679734</td>
      <td>-85.993245</td>
    </tr>
  </tbody>
</table>
</div>




```python
df = pd.concat([BC_df, AC_df,baby_friendly],ignore_index=True,sort=False)
df
```




<div>
<style scoped>
    .dataframe tbody tr th:only-of-type {
        vertical-align: middle;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }

    .dataframe thead th {
        text-align: right;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>NAME</th>
      <th>ADDRESS</th>
      <th>CITY</th>
      <th>STATE ZIP</th>
      <th>Administrator</th>
      <th>Tell</th>
      <th>Fax</th>
      <th>License Number</th>
      <th>Lic Expire Date</th>
      <th>Lat</th>
      <th>Long</th>
      <th>Type</th>
      <th>county</th>
      <th>hosp_type</th>
      <th>Certified</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>AUBURN BIRTHING CENTER LLC</td>
      <td>1915 WESLEY RD</td>
      <td>AUBURN</td>
      <td>IN 46706</td>
      <td>THADDEUS WEGHORST</td>
      <td>(260)927-0035</td>
      <td>(260)927-0036</td>
      <td>19-012463-1</td>
      <td>12/31/19</td>
      <td>41.371098</td>
      <td>-85.029044</td>
      <td>Birth Center</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
    </tr>
    <tr>
      <th>1</th>
      <td>BLESSED BEGINNINGS CARE CENTER INC</td>
      <td>2521 E MARKET ST</td>
      <td>NAPPANEE</td>
      <td>IN 46550</td>
      <td>JAMIE YODER</td>
      <td>(574)773-7755</td>
      <td>(574)773-7785</td>
      <td>19-013752-1</td>
      <td>6/30/20</td>
      <td>41.441417</td>
      <td>-85.974289</td>
      <td>Birth Center</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
    </tr>
    <tr>
      <th>2</th>
      <td>GOSHEN BIRTH CENTER INC</td>
      <td>1155 LIGHTHOUSE LN</td>
      <td>GOSHEN</td>
      <td>IN 46526</td>
      <td>BETSY BLACK</td>
      <td>(574)971-4840</td>
      <td>(574)533-4041</td>
      <td>19-005774-1</td>
      <td>12/31/19</td>
      <td>41.571696</td>
      <td>-85.865904</td>
      <td>Birth Center</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
    </tr>
    <tr>
      <th>3</th>
      <td>HOLY FAMILY BIRTH CENTER</td>
      <td>9835 AUBURN ROAD</td>
      <td>FORT WAYNE</td>
      <td>IN 46825</td>
      <td>CHRIS STROUD</td>
      <td>(260)450-8878</td>
      <td>(260)209-5956</td>
      <td>19-014552-1</td>
      <td>6/30/20</td>
      <td>41.173402</td>
      <td>-85.117261</td>
      <td>Birth Center</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
    </tr>
    <tr>
      <th>4</th>
      <td>NEW EDEN CARE CENTER INC</td>
      <td>7980 W 100 S</td>
      <td>TOPEKA</td>
      <td>IN 46571</td>
      <td>JAMIE YODER</td>
      <td>(260)768-4401</td>
      <td>(260)768-7655</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>41.625628</td>
      <td>-85.581676</td>
      <td>Birth Center</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
    </tr>
    <tr>
      <th>5</th>
      <td>SACRED ROOTS BIRTH CENTER INC</td>
      <td>6620 PARKDALE PLACE SUITE K</td>
      <td>INDIANAPOLIS</td>
      <td>IN 46254</td>
      <td>VICTORIA FLOYD</td>
      <td>(317)437-3681</td>
      <td>(317)552-2671</td>
      <td>19-014058-1</td>
      <td>12/31/19</td>
      <td>39.826245</td>
      <td>-86.278949</td>
      <td>Birth Center</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
    </tr>
    <tr>
      <th>6</th>
      <td>CLINIC FOR WOMEN</td>
      <td>3607 W 16TH ST STE 2B</td>
      <td>INDIANAPOLIS</td>
      <td>IN 46222</td>
      <td>LADONNA PRINCE</td>
      <td>(317)955-2641</td>
      <td>(317)955-2687</td>
      <td>19-011133-1</td>
      <td>06/30/2020</td>
      <td>39.787478</td>
      <td>-86.222633</td>
      <td>Abortion Center</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
    </tr>
    <tr>
      <th>7</th>
      <td>PLANNED PARENTHOOD OF INDIANA AND KENTUCKY INC</td>
      <td>8645 CONNECTICUT ST</td>
      <td>MERRILLVILLE</td>
      <td>IN 46410</td>
      <td>LOLITA KINSEY-BROWN</td>
      <td>(219)769-3500</td>
      <td>(219)791-0538</td>
      <td>19-011116-1</td>
      <td>06/30/2020</td>
      <td>39.763935</td>
      <td>-86.158912</td>
      <td>Abortion Center</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
    </tr>
    <tr>
      <th>8</th>
      <td>PLANNED PARENTHOOD OF INDIANA AND KENTUCKY INC</td>
      <td>421 S COLLEGE AVE</td>
      <td>BLOOMINGTON</td>
      <td>IN 47403</td>
      <td>MEAGAN COOK</td>
      <td>(812)336-0219</td>
      <td>(812)336-2401</td>
      <td>19-011117-1</td>
      <td>06/30/2020</td>
      <td>39.162239</td>
      <td>-86.536825</td>
      <td>Abortion Center</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
    </tr>
    <tr>
      <th>9</th>
      <td>PLANNED PARENTHOOD OF INDIANA AND KENTUCKY INC</td>
      <td>8590 GEORGETOWN RD</td>
      <td>INDIANAPOLIS</td>
      <td>IN 46268</td>
      <td>MICHELLE STEELE</td>
      <td>(317)872-3115</td>
      <td>(317)872-3188</td>
      <td>19-011118-1</td>
      <td>06/30/2020</td>
      <td>39.910195</td>
      <td>-86.243884</td>
      <td>Abortion Center</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
    </tr>
    <tr>
      <th>10</th>
      <td>PLANNED PARENTHOOD OF INDIANA AND KENTUCKY, INC -</td>
      <td>964 MEZZANINE DR</td>
      <td>LAFAYETTE</td>
      <td>IN 47905</td>
      <td>KALILA WEINER</td>
      <td>(765)446-8078</td>
      <td>(765)446-8160</td>
      <td>19-013765-1</td>
      <td>06/30/2020</td>
      <td>40.404664</td>
      <td>-86.835328</td>
      <td>Abortion Center</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
    </tr>
    <tr>
      <th>11</th>
      <td>WOMEN'S MED GROUP PROFESSIONAL CORPORATION</td>
      <td>1201 N ARLINGTON AVE</td>
      <td>INDIANAPOLIS</td>
      <td>IN 46219</td>
      <td>W. MARTIN HASKELL</td>
      <td>(317)353-9371</td>
      <td>(317)322-3358</td>
      <td>19-011128-1</td>
      <td>06/30/2020</td>
      <td>39.784352</td>
      <td>-86.065873</td>
      <td>Abortion Center</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
    </tr>
    <tr>
      <th>12</th>
      <td>COMMUNITY HOSPITAL OF ANDERSON AND MADISON COUNTY</td>
      <td>NaN</td>
      <td>ANDERSON</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>40.131330</td>
      <td>-85.693592</td>
      <td>NaN</td>
      <td>Madison</td>
      <td>SHORT TERM</td>
      <td>Yes</td>
    </tr>
    <tr>
      <th>13</th>
      <td>IU HEALTH BLOOMINGTON HOSPITAL</td>
      <td>NaN</td>
      <td>BLOOMINGTON</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>39.160071</td>
      <td>-86.540366</td>
      <td>NaN</td>
      <td>Monroe</td>
      <td>SHORT TERM</td>
      <td>Yes</td>
    </tr>
    <tr>
      <th>14</th>
      <td>PARKVIEW WHITLEY HOSPITAL</td>
      <td>NaN</td>
      <td>COLUMBIA CITY</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>41.158236</td>
      <td>-85.464892</td>
      <td>NaN</td>
      <td>Whitley</td>
      <td>SHORT TERM</td>
      <td>Yes</td>
    </tr>
    <tr>
      <th>15</th>
      <td>COLUMBUS REGIONAL HOSPITAL</td>
      <td>NaN</td>
      <td>COLUMBUS</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>39.216528</td>
      <td>-85.895923</td>
      <td>NaN</td>
      <td>Bartholomew</td>
      <td>SHORT TERM</td>
      <td>Yes</td>
    </tr>
    <tr>
      <th>16</th>
      <td>ELKHART GENERAL HOSPITAL</td>
      <td>NaN</td>
      <td>ELKHART</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>41.679734</td>
      <td>-85.993245</td>
      <td>NaN</td>
      <td>Elkhart</td>
      <td>SHORT TERM</td>
      <td>Yes</td>
    </tr>
    <tr>
      <th>17</th>
      <td>GOSHEN GENERAL HOSPITAL</td>
      <td>NaN</td>
      <td>GOSHEN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>41.563894</td>
      <td>-85.831296</td>
      <td>NaN</td>
      <td>Elkhart</td>
      <td>SHORT TERM</td>
      <td>Yes</td>
    </tr>
    <tr>
      <th>18</th>
      <td>HANCOCK REGIONAL HOSPITAL</td>
      <td>NaN</td>
      <td>GREENFIELD</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>39.795169</td>
      <td>-85.767667</td>
      <td>NaN</td>
      <td>Hancock</td>
      <td>SHORT TERM</td>
      <td>Yes</td>
    </tr>
    <tr>
      <th>19</th>
      <td>ST MARY MEDICAL CENTER INC</td>
      <td>NaN</td>
      <td>HOBART</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>41.512629</td>
      <td>-87.261198</td>
      <td>NaN</td>
      <td>Lake</td>
      <td>SHORT TERM</td>
      <td>Yes</td>
    </tr>
    <tr>
      <th>20</th>
      <td>INDIANA UNIVERSITY HEALTH</td>
      <td>NaN</td>
      <td>INDIANAPOLIS</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>39.790905</td>
      <td>-86.162902</td>
      <td>NaN</td>
      <td>Marion</td>
      <td>SHORT TERM</td>
      <td>Yes</td>
    </tr>
    <tr>
      <th>21</th>
      <td>FRANCISCAN HEALTH INDIANAPOLIS - EMERGENCY ROOM</td>
      <td>NaN</td>
      <td>INDIANAPOLIS</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>39.649143</td>
      <td>-86.078173</td>
      <td>NaN</td>
      <td>Marion</td>
      <td>SHORT TERM</td>
      <td>Yes</td>
    </tr>
    <tr>
      <th>22</th>
      <td>LOGANSPORT MEMORIAL HOSPITAL</td>
      <td>NaN</td>
      <td>LOGANSPORT</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>40.763509</td>
      <td>-86.362401</td>
      <td>NaN</td>
      <td>Cass</td>
      <td>SHORT TERM</td>
      <td>Yes</td>
    </tr>
    <tr>
      <th>23</th>
      <td>SAINT JOSEPH REGIONAL MEDICAL CENTER</td>
      <td>NaN</td>
      <td>MISHAWAKA</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>41.739637</td>
      <td>-86.173327</td>
      <td>NaN</td>
      <td>St. Joseph</td>
      <td>SHORT TERM</td>
      <td>Yes</td>
    </tr>
    <tr>
      <th>24</th>
      <td>INDIANA UNIVERSITY HEALTH BALL MEMORIAL HOSPITAL</td>
      <td>NaN</td>
      <td>MUNCIE</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>40.196895</td>
      <td>-85.415502</td>
      <td>NaN</td>
      <td>Delaware</td>
      <td>SHORT TERM</td>
      <td>Yes</td>
    </tr>
    <tr>
      <th>25</th>
      <td>HENRY COUNTY MEMORIAL HOSPITAL</td>
      <td>NaN</td>
      <td>NEW CASTLE</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>39.943720</td>
      <td>-85.363268</td>
      <td>NaN</td>
      <td>Henry</td>
      <td>SHORT TERM</td>
      <td>Yes</td>
    </tr>
    <tr>
      <th>26</th>
      <td>WOMEN'S HOSPITAL THE</td>
      <td>NaN</td>
      <td>NEWBURGH</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>37.973986</td>
      <td>-87.447953</td>
      <td>NaN</td>
      <td>Warrick</td>
      <td>SHORT TERM</td>
      <td>Yes</td>
    </tr>
    <tr>
      <th>27</th>
      <td>SAINT JOSEPH REGIONAL MEDICAL CENTER - PLYMOUTH</td>
      <td>NaN</td>
      <td>PLYMOUTH</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>41.333825</td>
      <td>-86.331454</td>
      <td>NaN</td>
      <td>Marshall</td>
      <td>SHORT TERM</td>
      <td>Yes</td>
    </tr>
    <tr>
      <th>28</th>
      <td>REID HEALTH</td>
      <td>NaN</td>
      <td>RICHMOND</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>39.864840</td>
      <td>-84.884622</td>
      <td>NaN</td>
      <td>Wayne</td>
      <td>SHORT TERM</td>
      <td>Yes</td>
    </tr>
    <tr>
      <th>29</th>
      <td>SCHNECK MEDICAL CENTER</td>
      <td>NaN</td>
      <td>SEYMOUR</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>38.956125</td>
      <td>-85.891711</td>
      <td>NaN</td>
      <td>Jackson</td>
      <td>SHORT TERM</td>
      <td>Yes</td>
    </tr>
    <tr>
      <th>30</th>
      <td>MEMORIAL HOSPITAL OF SOUTH BEND</td>
      <td>NaN</td>
      <td>SOUTH BEND</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>41.684337</td>
      <td>-86.251411</td>
      <td>NaN</td>
      <td>St. Joseph</td>
      <td>SHORT TERM</td>
      <td>Yes</td>
    </tr>
    <tr>
      <th>31</th>
      <td>UNION HOSPITAL INC</td>
      <td>NaN</td>
      <td>TERRE HAUTE</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>39.485314</td>
      <td>-87.409128</td>
      <td>NaN</td>
      <td>Vigo</td>
      <td>SHORT TERM</td>
      <td>Yes</td>
    </tr>
    <tr>
      <th>32</th>
      <td>PULASKI MEMORIAL HOSPITAL</td>
      <td>NaN</td>
      <td>WINAMAC</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>41.056712</td>
      <td>-86.596180</td>
      <td>NaN</td>
      <td>Pulaski</td>
      <td>CRITICAL ACCESS HOSPITALS</td>
      <td>Yes</td>
    </tr>
  </tbody>
</table>
</div>




```python
df['geometry'] = df.apply(
    lambda x: Point((x.Long, x.Lat)), axis = 1)

```


```python
crs = {'init':  'epsg:4326'}
geo_df = gpd.GeoDataFrame(df, crs=crs, geometry= df.geometry)
#geo_df['index']= range(1, len(geo_df)+1)
df.head()
```




<div>
<style scoped>
    .dataframe tbody tr th:only-of-type {
        vertical-align: middle;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }

    .dataframe thead th {
        text-align: right;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>NAME</th>
      <th>ADDRESS</th>
      <th>CITY</th>
      <th>STATE ZIP</th>
      <th>Administrator</th>
      <th>Tell</th>
      <th>Fax</th>
      <th>License Number</th>
      <th>Lic Expire Date</th>
      <th>Lat</th>
      <th>Long</th>
      <th>Type</th>
      <th>county</th>
      <th>hosp_type</th>
      <th>Certified</th>
      <th>geometry</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>AUBURN BIRTHING CENTER LLC</td>
      <td>1915 WESLEY RD</td>
      <td>AUBURN</td>
      <td>IN 46706</td>
      <td>THADDEUS WEGHORST</td>
      <td>(260)927-0035</td>
      <td>(260)927-0036</td>
      <td>19-012463-1</td>
      <td>12/31/19</td>
      <td>41.371098</td>
      <td>-85.029044</td>
      <td>Birth Center</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>POINT (-85.02904 41.37110)</td>
    </tr>
    <tr>
      <th>1</th>
      <td>BLESSED BEGINNINGS CARE CENTER INC</td>
      <td>2521 E MARKET ST</td>
      <td>NAPPANEE</td>
      <td>IN 46550</td>
      <td>JAMIE YODER</td>
      <td>(574)773-7755</td>
      <td>(574)773-7785</td>
      <td>19-013752-1</td>
      <td>6/30/20</td>
      <td>41.441417</td>
      <td>-85.974289</td>
      <td>Birth Center</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>POINT (-85.97429 41.44142)</td>
    </tr>
    <tr>
      <th>2</th>
      <td>GOSHEN BIRTH CENTER INC</td>
      <td>1155 LIGHTHOUSE LN</td>
      <td>GOSHEN</td>
      <td>IN 46526</td>
      <td>BETSY BLACK</td>
      <td>(574)971-4840</td>
      <td>(574)533-4041</td>
      <td>19-005774-1</td>
      <td>12/31/19</td>
      <td>41.571696</td>
      <td>-85.865904</td>
      <td>Birth Center</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>POINT (-85.86590 41.57170)</td>
    </tr>
    <tr>
      <th>3</th>
      <td>HOLY FAMILY BIRTH CENTER</td>
      <td>9835 AUBURN ROAD</td>
      <td>FORT WAYNE</td>
      <td>IN 46825</td>
      <td>CHRIS STROUD</td>
      <td>(260)450-8878</td>
      <td>(260)209-5956</td>
      <td>19-014552-1</td>
      <td>6/30/20</td>
      <td>41.173402</td>
      <td>-85.117261</td>
      <td>Birth Center</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>POINT (-85.11726 41.17340)</td>
    </tr>
    <tr>
      <th>4</th>
      <td>NEW EDEN CARE CENTER INC</td>
      <td>7980 W 100 S</td>
      <td>TOPEKA</td>
      <td>IN 46571</td>
      <td>JAMIE YODER</td>
      <td>(260)768-4401</td>
      <td>(260)768-7655</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>41.625628</td>
      <td>-85.581676</td>
      <td>Birth Center</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>POINT (-85.58168 41.62563)</td>
    </tr>
  </tbody>
</table>
</div>




```python
geo_df.head()
```




<div>
<style scoped>
    .dataframe tbody tr th:only-of-type {
        vertical-align: middle;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }

    .dataframe thead th {
        text-align: right;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>NAME</th>
      <th>ADDRESS</th>
      <th>CITY</th>
      <th>STATE ZIP</th>
      <th>Administrator</th>
      <th>Tell</th>
      <th>Fax</th>
      <th>License Number</th>
      <th>Lic Expire Date</th>
      <th>Lat</th>
      <th>Long</th>
      <th>Type</th>
      <th>county</th>
      <th>hosp_type</th>
      <th>Certified</th>
      <th>geometry</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>AUBURN BIRTHING CENTER LLC</td>
      <td>1915 WESLEY RD</td>
      <td>AUBURN</td>
      <td>IN 46706</td>
      <td>THADDEUS WEGHORST</td>
      <td>(260)927-0035</td>
      <td>(260)927-0036</td>
      <td>19-012463-1</td>
      <td>12/31/19</td>
      <td>41.371098</td>
      <td>-85.029044</td>
      <td>Birth Center</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>POINT (-85.02904 41.37110)</td>
    </tr>
    <tr>
      <th>1</th>
      <td>BLESSED BEGINNINGS CARE CENTER INC</td>
      <td>2521 E MARKET ST</td>
      <td>NAPPANEE</td>
      <td>IN 46550</td>
      <td>JAMIE YODER</td>
      <td>(574)773-7755</td>
      <td>(574)773-7785</td>
      <td>19-013752-1</td>
      <td>6/30/20</td>
      <td>41.441417</td>
      <td>-85.974289</td>
      <td>Birth Center</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>POINT (-85.97429 41.44142)</td>
    </tr>
    <tr>
      <th>2</th>
      <td>GOSHEN BIRTH CENTER INC</td>
      <td>1155 LIGHTHOUSE LN</td>
      <td>GOSHEN</td>
      <td>IN 46526</td>
      <td>BETSY BLACK</td>
      <td>(574)971-4840</td>
      <td>(574)533-4041</td>
      <td>19-005774-1</td>
      <td>12/31/19</td>
      <td>41.571696</td>
      <td>-85.865904</td>
      <td>Birth Center</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>POINT (-85.86590 41.57170)</td>
    </tr>
    <tr>
      <th>3</th>
      <td>HOLY FAMILY BIRTH CENTER</td>
      <td>9835 AUBURN ROAD</td>
      <td>FORT WAYNE</td>
      <td>IN 46825</td>
      <td>CHRIS STROUD</td>
      <td>(260)450-8878</td>
      <td>(260)209-5956</td>
      <td>19-014552-1</td>
      <td>6/30/20</td>
      <td>41.173402</td>
      <td>-85.117261</td>
      <td>Birth Center</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>POINT (-85.11726 41.17340)</td>
    </tr>
    <tr>
      <th>4</th>
      <td>NEW EDEN CARE CENTER INC</td>
      <td>7980 W 100 S</td>
      <td>TOPEKA</td>
      <td>IN 46571</td>
      <td>JAMIE YODER</td>
      <td>(260)768-4401</td>
      <td>(260)768-7655</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>41.625628</td>
      <td>-85.581676</td>
      <td>Birth Center</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>POINT (-85.58168 41.62563)</td>
    </tr>
  </tbody>
</table>
</div>




```python
in_cou = gpd.read_file('Census_Counties/Census_County_TIGER00_IN.shp')
in_cou[in_cou["POP2000"] >= 150000].shape
in_cou.head()
```




<div>
<style scoped>
    .dataframe tbody tr th:only-of-type {
        vertical-align: middle;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }

    .dataframe thead th {
        text-align: right;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>AREA</th>
      <th>PERIMETER</th>
      <th>NAME_U</th>
      <th>NAME_L</th>
      <th>NCAPC</th>
      <th>CNTY_FIPS</th>
      <th>STFID</th>
      <th>POP2000</th>
      <th>WHITE</th>
      <th>BLACK</th>
      <th>...</th>
      <th>MARHH_NO_C</th>
      <th>MHH_CHILD</th>
      <th>FHH_CHILD</th>
      <th>FAMILIES</th>
      <th>AVE_FAM_SZ</th>
      <th>HSE_UNITS</th>
      <th>VACANT</th>
      <th>OWNER_OCC</th>
      <th>RENTER_OCC</th>
      <th>geometry</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>8.352031e+08</td>
      <td>116410.31836</td>
      <td>STEUBEN</td>
      <td>Steuben</td>
      <td>76</td>
      <td>151</td>
      <td>18151</td>
      <td>33214</td>
      <td>32281</td>
      <td>123</td>
      <td>...</td>
      <td>4208</td>
      <td>375</td>
      <td>715</td>
      <td>8911</td>
      <td>3.00</td>
      <td>17337</td>
      <td>4599</td>
      <td>9951</td>
      <td>2787</td>
      <td>POLYGON ((679527.291 4625396.026, 681172.040 4...</td>
    </tr>
    <tr>
      <th>1</th>
      <td>1.001560e+09</td>
      <td>129036.36824</td>
      <td>LAGRANGE</td>
      <td>Lagrange</td>
      <td>44</td>
      <td>087</td>
      <td>18087</td>
      <td>34909</td>
      <td>33770</td>
      <td>66</td>
      <td>...</td>
      <td>3777</td>
      <td>249</td>
      <td>434</td>
      <td>8856</td>
      <td>3.54</td>
      <td>12938</td>
      <td>1713</td>
      <td>9151</td>
      <td>2074</td>
      <td>POLYGON ((648336.954 4624638.241, 649912.369 4...</td>
    </tr>
    <tr>
      <th>2</th>
      <td>1.211453e+09</td>
      <td>139308.15005</td>
      <td>ELKHART</td>
      <td>Elkhart</td>
      <td>20</td>
      <td>039</td>
      <td>18039</td>
      <td>182791</td>
      <td>157931</td>
      <td>9551</td>
      <td>...</td>
      <td>19981</td>
      <td>1839</td>
      <td>4636</td>
      <td>47659</td>
      <td>3.18</td>
      <td>69791</td>
      <td>3637</td>
      <td>47769</td>
      <td>18385</td>
      <td>POLYGON ((609782.908 4623876.537, 611399.543 4...</td>
    </tr>
    <tr>
      <th>3</th>
      <td>1.194457e+09</td>
      <td>155508.91992</td>
      <td>ST JOSEPH</td>
      <td>St Joseph</td>
      <td>71</td>
      <td>141</td>
      <td>18141</td>
      <td>265559</td>
      <td>218706</td>
      <td>30422</td>
      <td>...</td>
      <td>28122</td>
      <td>2148</td>
      <td>7865</td>
      <td>66802</td>
      <td>3.07</td>
      <td>107013</td>
      <td>6270</td>
      <td>72194</td>
      <td>28549</td>
      <td>POLYGON ((576360.816 4623609.402, 577920.195 4...</td>
    </tr>
    <tr>
      <th>4</th>
      <td>1.620569e+09</td>
      <td>177807.09411</td>
      <td>LAKE</td>
      <td>Lake</td>
      <td>45</td>
      <td>089</td>
      <td>18089</td>
      <td>484564</td>
      <td>323290</td>
      <td>122723</td>
      <td>...</td>
      <td>49444</td>
      <td>3671</td>
      <td>16887</td>
      <td>127036</td>
      <td>3.19</td>
      <td>194992</td>
      <td>13359</td>
      <td>125249</td>
      <td>56384</td>
      <td>POLYGON ((458433.478 4623270.678, 481438.856 4...</td>
    </tr>
  </tbody>
</table>
<p>5 rows × 44 columns</p>
</div>




```python

```


```python
geo_df.plot()
plt.show()
```


![png](output_10_0.png)



```python
in_cou.plot()
```




    <matplotlib.axes._subplots.AxesSubplot at 0x11b432208>




![png](output_11_1.png)



```python
in_cou.geometry = in_cou.geometry.to_crs(epsg = 3857)
geo_df.geometry = geo_df.geometry.to_crs(epsg = 3857)
```


```python
fig, ax = plt.subplots(figsize=(11,17))
minx, miny, maxx, maxy = in_cou.total_bounds
ax.set_xlim(minx, maxx)
ax.set_ylim(miny, maxy)
ax.set_aspect('auto')
plt.xticks([])
plt.yticks([])
in_cou.plot(ax=ax, edgecolor='black',cmap = 'BuPu', column = "POP2000", legend = False, legend_kwds={'label': "County Population",'fontsize':24},alpha= 1, vmin=10000, vmax=150000)

colors = {'Abortion Center':'red', 'Birth Center':'blue'}

geo_df[geo_df['Type']=='Abortion Center'].plot(ax=ax, marker="s",edgecolor='k', markersize=150,alpha= 0.8, c= '#ffff00', label= "Abortion Center" ) #geo_df['Type'].apply(lambda x: colors[x]))
geo_df[geo_df['Type']=='Birth Center'].plot(ax=ax, marker='o',edgecolor='k', markersize=150,alpha= 0.8, c='#ff0102',label= "Birth Center")#geo_df['Type'].apply(lambda x: colors[x]))
geo_df[geo_df['Certified']=='Yes'].plot(ax=ax, marker='D',edgecolor='k', markersize=150,alpha= 0.8, c='#4cc844',label= "Baby Friendly Hospital")#geo_df['Type'].apply(lambda x: colors[x]))


#for x, y, label in zip(geo_df.geometry.x, geo_df.geometry.y, geo_df.CITY):
#    ax.annotate(label, xy=(x, y), xytext=(3, 3), textcoords="offset points")
    
plt.legend(prop={"size": 13},bbox_to_anchor=(0.5, -.09), loc='lower center', ncol=3)
    
plt.savefig('brith_abo.png')



plt.show()
```


![png](output_13_0.png)



```python
fig, ax = plt.subplots(figsize=(15,17))
minx, miny, maxx, maxy = in_cou.total_bounds
ax.set_xlim(auto=True)
ax.set_ylim(auto=True)
ax.set_aspect('auto')
plt.ioff()
plt.xticks([])
plt.yticks([])
plt.title("Indiana Birth & Abortion Centers",fontsize=40,y=1.04, x= .55)

matplotlib.rcParams['font.size'] = 20
in_cou.plot(ax=ax, edgecolor='black',cmap = 'BuPu', column = "POP2000", legend = True, legend_kwds={'label': "County Population"},alpha= 0.7, vmin=10000, vmax=150000)


colors = {'Abortion Center':'red', 'Birth Center':'blue'}
geo_df[geo_df['Certified']=='Yes'].plot(ax=ax, marker='D',edgecolor='k', markersize=150,alpha= 0.8, c='#4cc844',label= "Baby Friendly Hospital")#geo_df['Type'].apply(lambda x: colors[x]))
geo_df[geo_df['Type']=='Birth Center'].plot(ax=ax, marker='o',edgecolor='k', markersize=150,alpha= 0.8, c='#ff0102',label= "Birth Center")#geo_df['Type'].apply(lambda x: colors[x]))
geo_df[geo_df['Type']=='Abortion Center'].plot(ax=ax, marker="s",edgecolor='k', markersize=150,alpha= 0.8, c= '#ffff00', label= "Abortion Center" ) #geo_df['Type'].apply(lambda x: colors[x]))


#for x, y, label in zip(geo_df.geometry.x, geo_df.geometry.y, geo_df.CITY):
#    ax.annotate(label, xy=(x, y), xytext=(3, 3), textcoords="offset points")
    
plt.legend(prop={"size": 20},bbox_to_anchor=(0.56, -.09), loc='lower center', ncol=3)
    
plt.savefig('brith_abo.png')



plt.show()
```


![png](output_14_0.png)



```python
in_cou.shape
```




    (92, 44)




```python
hos_cou.shape
```


    ---------------------------------------------------------------------------

    NameError                                 Traceback (most recent call last)

    <ipython-input-36-5b7cd631d216> in <module>
    ----> 1 hos_cou.shape
    

    NameError: name 'hos_cou' is not defined



```python
from matplotlib.colors import ListedColormap
import matplotlib
```


```python
fig, ax = plt.subplots(figsize=(15,17))
minx, miny, maxx, maxy = in_cou.total_bounds
ax.set_xlim(auto=True)
ax.set_ylim(auto=True)
ax.set_aspect('auto')
plt.xticks([])

plt.yticks([])
plt.title("Indiana Birth & Abortion Centers",fontsize=40,y=1.04, x= .55)
cmap1 = ListedColormap(['#2f7d50'], name='allred')
cmap2 = ListedColormap(['#a0a8a3'], name='allblue')

matplotlib.rcParams['font.size'] = 20
in_cou.plot(ax=ax, edgecolor='black',cmap=cmap1)
in_cou.plot(ax=ax, edgecolor='black',cmap = cmap2, column = "POP2000", legend = False,alpha = 0.9)


colors = {'Abortion Center':'red', 'Birth Center':'blue'}
geo_df[geo_df['Certified']=='Yes'].plot(ax=ax, marker='D',edgecolor='k', markersize=150,alpha= 0.8, c='#13a3a8',label= "Baby Friendly Hospital")#geo_df['Type'].apply(lambda x: colors[x]))
geo_df[geo_df['Type']=='Birth Center'].plot(ax=ax, marker='o',edgecolor='k', markersize=150,alpha= 0.8, c='#ff0102',label= "Birth Center")#geo_df['Type'].apply(lambda x: colors[x]))
geo_df[geo_df['Type']=='Abortion Center'].plot(ax=ax, marker="s",edgecolor='k', markersize=150,alpha= 0.8, c= '#ffff00', label= "Abortion Center" ) #geo_df['Type'].apply(lambda x: colors[x]))


#for x, y, label in zip(geo_df.geometry.x, geo_df.geometry.y, geo_df.CITY):
#    ax.annotate(label, xy=(x, y), xytext=(3, 3), textcoords="offset points")
    
plt.legend(prop={"size": 20},bbox_to_anchor=(0.56, -.09), loc='lower center', ncol=3)
    
plt.savefig('brith_abo.png')

plt.show()
```


![png](output_18_0.png)



```python

```
