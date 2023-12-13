# UDOT-Segment-Factors
This repository contains the data and code for the creation of seasonal, day-of-week, and hour-of-day factors.

This readme is divided into three sections:

* Methodology: description of steps to generate factors and assign to segments

* Parameters: description of parameter files used in scripts

* Aggregations: description of notebooks to further aggregate factor data for specific applications

The folder structure of the repository is as follows:
* `aggregations`: scripts to aggregat3 segment factors into groupings by county, month, day of week, etc.
* `arcgis`: ArcGIS Pro project with segments
* `input`: input data
* `intermediate`: data generated in intermediate steps on way to final data set
* `param`: set of parameters used when generating factors and assigning to segments
* `results`: final data set of results for factor csv and segment layers


## Methodology - Factor Generation and Assignment to Segments

The process for generating factors and assigning to segments is carried out through sequential steps (1-A) as described here:

### 1_Process-AvenueData.accdb

This MS Access database contains links to the tables and queries to generate a CSV from previously processed data (2013-2018) from the databases in the `input/avenue` folder. The contents are detailed here:

* `tblHourlyVolumes`: link to volumes for each continuous count station (CCS) for each direction for each lane for each day of each month of each year by hour of the day including the following fields:
    * `SiteID`: CCS id with '-0' prefix removed
    * `Direction`: direction of travel
    * `intYear`: calendar year
    * `bytMonth`: calendar month
    * `bytDay`: calendar day
    * `bytLane`: lane number (1 is furthest left)
    * `bytHour`: hour of the day (0-23)
    * `Volume`: number of vehicles
    * `DOW`: day-of-week (MS Access)
    * `Holiday`: yes, if day is a holiday or holiday weekend
    * `Exclude`: yes, if exclude record from analysis
* `tblAnnualTruckDailyTraffic`: link to truck data for each year for each segment including the following fields:
    * `AADT`: Average Annual Daily Traffic
    * `SUTRK`: Single-Unit Truck Percent of Total Volume
    * `CUTRK`: Combo-Unit Truck Percent of Total Volume
* `tblDOWforJupyterEnvironment`: table to convert integer day-of-week from MS Access environment to day-of-week in jupyter environment
    | DOWAccess | DOWJupyter | DOWDescription |
    |---|---|---|
    | 1 | 6 | Sun |
    | 2 | 0 | Mon |
    | 3 | 1 | Tue |
    | 4 | 2 | Wed |
    | 5 | 3 | Thu |
    | 6 | 4 | Fri |
    | 7 | 5 | Sat |

* `qryExportHourlyVolumesToCSV`: this query combines the `tblHourlyVolumes` and `tblDOWforJupyterEnvironment` to create a exportable dataset that is then manually saved to the `intermediate/CCSHourDirLane_Avenue.csv` file with the following renamed fields:
    * `STATION`: CCS station
    * `DIR`: direction of travel
    * `YEAR`: calendar year
    * `MONTH`: calendar month
    * `DAY`: calendar day
    * `LANE`: lane number (1 is furthest left)
    * `HOUR`: hour of day
    * `HOURVOL`: volume for hour
    * `DOW`: day of week - jupyter environment

### 2_Process-CCS-Data-NewFormat.ipynb

UDOT produces CSS data in two formats. The New format includes data from 2018+, except for August 2019, which was unavailable via web download at https://drive.google.com/drive/folders/1ZYy-WkICLOp1482vwEbTc5UvLItbWs4y

### 3_Process-CCS-Data-Old-Format.accdb

The old format contains data from August 2019, which was not available via the download but requested separately via email from Nicolas Black at UDOT. This data is stored in the folder `input\_udot\HourlyCCSData_2019August`.

This MS Access database contains two macros to extract data from an Excel spreadsheet or from a PDF depending on in what format the data is received.

### 4_Process-Truck-Data-New-Format.ipynb

This jupyter notebook contains the code to extract truck data from the downloaded data in `input/udot/TruckTrafficOnUtahHighways` from https://drive.google.com/drive/folders/12yEwu2zXh-FTB92O0l7TwjG8zBkJprTu. The resulting csv is stored here: `intermediate/AnnualTruckDailyTraffic_NewFormat.csv`

### 5_Combine-Data.ipynb

This jupyter notebook contains the code to combine all the data in the previous steps into single CSVs that are stored in the `intermediate` folder.
* `CCSHourDir_Combined_2013to2022.csv`: all hourly data combined by direction (all lanes aggregated together)
* `TruckData_Combined_2012to2019.csv`: all truck data combined.
* `StationMonthDays.csv`: list of number of hours per day with useable data by station

### 6_CCS-Process-ADTs.ipynb

This jupyter notebook contains the code to generate the following CSVs to be used in generating factors:

* `DayVol.csv`: 
    * `STATION`: CCS station
    * `YEAR`: calendar year
    * `MONTH`: calendar month
    * `DAY`: calendar day
    * `DOW`: day-of-week (jupyter environment)
    * `DATE`: full data
    * `DAYVOL`: daily volume
* `DayPeriodVol.csv`:
    * `STATION`: CCS station
    * `YEAR`: calendar year
    * `MONTH`: calendar month
    * `DAY`: calendar day
    * `DOW`: day-of-week (jupyter environment)
    * `DATE`: full data
    * `PERIOD`: period of day (AM, MD, PM, EV) as used in travel demand model
    * `PERIODVOL`: total volume for given period
    * `PKHRVOL`: volume of highest hour in period
* `MonthADT.csv`:
    * `STATION`: CCS station
    * `YEAR`: calendar year
    * `MONTH`: calendar month
    * `MONTHADT`: average daily volume for month
* `MonthDOWADT.csv`:
    * `STATION`: CCS station
    * `YEAR`: calendar year
    * `MONTH`: calendar month
    * `DOW`: day-of-week (jupyter environment)
    * `MONTHDOWADT`: average daily volume for month day-of-week
* `MonthDOWPeriodVol.csv`:
    * `STATION`: CCS station
    * `YEAR`: calendar year
    * `MONTH`: calendar month
    * `DOW`: day-of-week (jupyter environment)
    * `PERIOD`: period of day (AM, MD, PM, EV) as used in travel demand model
    * `PERIODVOL`: total volume for given period
    * `PKHRVOL`: volume of highest hour in period
* `YearADT.csv`:
    * `STATION`: CCS station
    * `YEAR`: calendar year
    * `YEARADT`: average daily traffic for year

### 7_CCS-Factors

This jupyter notebook contains the code for generating the following final sets of factors and data for each CCS.

The following CSVs are generated and stored in the 'intermediate' folder:
* `CCS_Factors_AllGroupings.csv`: this CSV contains a list of all factors for every combination of grouping in a long format. The fields are defined as follows:
    * `STATIONGROUP`: CCS group ID
    * `SEASONGROUP`: season group (month, season, all year)
    * `YEARGROUP`:  year group (all years, individual years, last five years, etc)
    * `FACTORNAME`: the factor name, which is one of the following:
        * `FAC_JAN`: January factor
        * `FAC_FEB`: February factor
        * `FAC_MAR`: March factor
        * `FAC_APR`: April factor
        * `FAC_MAY`: May factor
        * `FAC_JUN`: June factor
        * `FAC_JUL`: July factor
        * `FAC_AUG`: August factor
        * `FAC_SEP`: September factor
        * `FAC_OCT`: October factor
        * `FAC_NOV`: November factor
        * `FAC_DEC`: December factor
        * `FAC_WIN`: winter factor (Dec-Feb)
        * `FAC_SPR`: spring factor (Mar-May)
        * `FAC_SUM`: summer factor (Jun-Aug)
        * `FAC_FAL`: fall factor (Sep-Nov)
        * `FAC_YR`: year factor (always 1.0)
        * `FAC_MAXMO`: the value of the highest month factor (`FAC_JAN`...`FAC_DEC`)
        * `FAC_MAX`: the month of the highest month factor (1-12)
        * `FAC_WDAVG`: the weekday average factor (`FAC_TUE`,`FAC_WED`,`FAC_THU`)
        * `FAC_WEAVG`: the weekend average factor (`FAC_SAT`,`FAC_SUN`)
        * `FAC_MON`: Monday factor
        * `FAC_TUE`: Tuesday factor
        * `FAC_WED`: Wednesday factor
        * `FAC_THU`: Thursday factor
        * `FAC_FRI`: Friday factor
        * `FAC_SAT`: Saturday factor
        * `FAC_SUN`: Sunday factor
        * `FAC_WEMAX`: the value of the weekend max factor (max of `FAC_SAT` or `FAC_SUN`)
    * `FACTOR`: the factor value

* `CCS_Factors_Period_AllGroupings.csv`: this CSV contains a period factors for every combination of grouping in a long format. The fields are defined as follows:
    * `STATIONGROUP`: CCS group ID
    * `YEARGROUP`: year group (all years, individual years, last five years, etc)
    * `SEASONGROUP`: season group (month, season, all year)
    * `DOWGROUP`: day-of-week group (individual days, weekdays, weekend days, etc)
    * `PERIOD`: period of day (`AM`,`MD`,`PM`,`EV`)
    * `PKHRVOL_PCTDY`: the percent of daily volume found in the peak hour of the given period
    * `PKHRVOL_PCTPRD`: the percent of period volume found in the peak hour of that period
    * `PERIODVOL_PCTDY`: the percent of daily volume found in the period


The following JSONs are generated for the web application and stored in the folder `results\webapp_data`:
* `dowfactors.json`:
* `stationgroups.json`: the complete list of station groups
* `stationgroups_finalcut.json`:
* `wdfactors.json`:
* `yeargroups.json`:
* `monthfactors.json`:
* `seasonfactors.json`:

### 8_Assign-Segments-To-CCS-Groups.ipynb

This jupyter notebook contains the code to assign roadway segments to groups

### 9_Prepare-Segment-Truck-Factors

This jupyter notebook contains the code to assign truck factors to segments.

### A_Prepare-Segment-Factors

This jupyter notebook contains the code to assign station group factors to segments.

## Parameters

params/area_type_overrides_sites.csv
params/area_types.csv
params/ccs_import_exclude_months_new_format.csv
params/direction_groups.csv
params/dow_factors_maxmin.csv
params/dow_group_field_names.csv
params/dow_to_dow_groups.csv
params/dow_webapp.csv
params/exclusions.csv
params/facgroup_volume_class_to_volume_group.csv
params/field_name_order.csv
params/functional_type_group_overrides.csv
params/holidays.csv
params/month_to_season_groups.csv
params/season_group_field_names.csv
params/season_webapp.csv
params/segment_factors_interpolate.csv
params/station_group_to_facgroup.csv
params/station_group_to_facgroup_facgeo_overrides.csv
params/station_groups.csv
params/station_groups_derived_portions.csv
params/station_to_station_groups.csv
params/volume_groups.csv
params/weekday_group_to_wdfacgroup.csv
params/winter_closures.csv
params/year_groups.csv
params/year_to_year_groups.csv
params/aadt_ranges_for_seasonal_factors.csv
params/area_type_group_to_area_type.csv
params/area_type_overrides_segments.cs


## Aggregations

This section describes aggregations of factors.