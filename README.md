# Predicting Ridership by Business Growth

Project Team:

* Liza Marie Soriano (lizamarie1218)
* Ronald Kwan (rkwan)
* Kelsey Anderson (kjanderson)


## About
Using block group-level information about local business growth,
transit options, and demographics, our team employed machine learning methods
to select a model that would best predict ridership growth in the next year,
and identify which factors are most important for these predictions. 


## Data Sources

- Total Population Survey, Table: B01003.
    Years: 2018, 2017, 2016, 2015, 2014. data.census.gov/.
- CTA datasets:
    data.cityofchicago.org/Transportation/CTA-List-of-CTA-Datasets/pnau-cf66
	- `CTA_BusRoutes.kml`
	- `CTA_-_Ridership_-_Bus_Routes_-_Monthly_Day-Type_Averages___Totals.csv`
	- `CTA_L_Ridership_Monthly_Day-Type.csv`
- City of Chicago Data Portal: data.cityofchicago.org/
	- `Business_Licenses.csv`
	- `Boundaries - Neighborhoods.geojson`
	- `Boundaries - Census Blocks - 2010.geojson`


## Modules
### DataWrangling_ACS_BusRides


Takes as input:

- `data/Boundaries - Neighborhoods.geojson`
- `data/Boundaries - Census Blocks - 2010.geojson`
- `data/CTA_BusRoutes.kml`
- `data/CTA_-_Ridership_-_Bus_Routes_-_Monthly_Day-Type_Averages___Totals.csv`
- `data/ACS/ACSDP5Y2014.DP03_data_with_overlays_2020-05-31T160939.csv`
- `data/ACS/ACSDP5Y2015.DP03_data_with_overlays_2020-05-31T160939.csv`
- `data/ACS/ACSDP5Y2016.DP03_data_with_overlays_2020-05-31T160939.csv`
- `data/ACS/ACSDP5Y2017.DP03_data_with_overlays_2020-05-31T160939.csv`
- `data/ACS/ACSDP5Y2018.DP03_data_with_overlays_2020-05-31T160939.csv`

Produces:

- `data/area_CTA_routes.geojson`


`area_CTA_routes.geojson` fields:

- `blockgroup` (string): Block group (e.g. `"170310101001"`)
- `year` (int): year of ridership data collection (e.g. `2018`)
- `month` (int): month of ridership data collection (e.g. `1` for January)
- `prior_year` (int): year for which percent change variables were calculated
- `pri_neigh` (string): name of Chicago neighborhood of blockgroup
- `Population` (int): ACS 5-year estimate of block group population
- `pop_change` (float): percent change in population from prior year
- `Median Income` (float): ACS 5-year estimate of blockgroup median income
- `income_change` (float): percent change in median income from prior year
- `Median Age` (float): ACS 5-year estimate of blockgroup median age
- `age_change` (float): percent change in median age from prior year
- `WorkTransitCount` (int): ACS 5-year estimate for number of working adults
    (16+) using public transit to get to work
- `wt_count_change` (float): percent change in workers using transit
    from prior year
- `WorkTransitPercent` (float): ACS 5-year estimate for percent of workeing
    adults (16+) using transit to get to work
- `wt_perc_change` (float): percent change in percent of workers using transit
- `count_of_routes` (int): count of bus routes intersecting blockgroup
- `rt_count_change` (float): change in number of bus routes for blockgroup
    from prior year
- `MonthTotal` (float): estimated total bus ridership for blockgroup in that
    month and year
- `geometry` (geopandas polygon): shape data for blockgroup
    (Geographic 2D CRS: EPSG:4326)


Additional Notes:
- All Census data is annual (Age, Income, Public Transit Work Commuters).
- Ridership data (MonthTotal) is averaged:
    - Monthly for each year
    - Aggregated by proportion of ridership from all bus routes
      passing through block group weighted by block group population.
      Example: (sum of ridership from route) * 
                (population of current block group / 
                            population of all blockgroups on route)
- Primary Neighborhood is the first neighborhood which intersects
      with the blockgroup.



### DataWrangling_TrainRides

Takes as input:

- `data/business_license.json`
- `data/Boundaries - Neighborhoods.geojson`
- `data/Boundaries - Census Blocks - 2010.geojson`

Produces:

- `data/CTA_L_Ridership_Monthly_by_Blockgroup.csv`

This last file contains a count of how many L rides were taken at a station
within 2km of a blockgroup in a certain month, starting from January 2001.

The distance is configurable as `DISTANCE_FROM_STATION` (in meters) at the
beginning of the notebook.

`CTA_L_Ridership_Monthly_by_Blockgroup.csv` fields:

- `blockgroup` (string): Block group (e.g. `"170310101001"`)
- `month_beginning` (date): Date on which the month starts (e.g. `01/01/2001`)
- `train_rides` (int): Number of train rides starting from a station within 2km



### DataWrangling_BusinessLicenses

Takes as input:

- `data/Boundaries - Neighborhoods.geojson`
- `data/Boundaries - Census Blocks - 2010.geojson`
- `data/CTA_L_Stops.csv`
- `data/CTA_L_Ridership_Monthly_Day-Type.csv`

Produces:

- `data/business_licenses_by_blockgroup.csv`


`business_licenses_by_blockgroup.csv` fields:
- `blockgroup` (string): block group code
- `month-year` (datetime): first day of month with year
- `active` (int): count of active businesses
- `new` (int): count of new businesses
- `month` (int)
- `year` (int)
- `prev_month-year` (datetime): first day of month for prior year
- `prev_yr_active` (int): count of active businesses from month in prior year
- `prev_yr_new` (int): count of new businesses from month in prior year
- `%_change_active` (float): percent change in active businesses from prior year
- `%_change_new` (float): percent change in new businesses from prior year



### Pipeline_DataMerging

Takes as input:

- `data/CTA_L_Ridership_Monthly_by_Blockgroup.csv`
- `data/area_CTA_routes.geojson`
- `data/business_licenses_by_blockgroup.csv`

Produces:

- `data/MERGED_DATA.geojson`


`MERGED_DATA.geojson` fields:
- `blockgroup` (string): block group code
- `year` (int): year of ridership data collection (e.g. `2018`)
- `month` (int): month of ridership data collection (e.g. `1` for January)
- `prior_year` (int): year for which percent change variables were calculated
- `pri_neigh` (string): name of Chicago neighborhood of blockgroup
- `Population` (int): ACS 5-year estimate of block group population
- `pop_change` (float): percent change in population from prior year
- `Median Income` (float): ACS 5-year estimate of blockgroup median income
- `income_change` (float): percent change in median income from prior year
- `Median Age` (float): ACS 5-year estimate of blockgroup median age
- `age_change` (float): percent change in median age from prior year
- `WorkTransitCount` (int): ACS 5-year estimate for number of working adults
    (16+) using public transit to get to work
- `wt_count_change` (float): percent change in workers using transit
    from prior year
- `WorkTransitPercent` (float): ACS 5-year estimate for percent of workeing
    adults (16+) using transit to get to work
- `wt_perc_change` (float): percent change in percent of workers using transit
- `count_of_routes` (int): count of bus routes intersecting blockgroup
- `rt_count_change` (float): change in number of bus routes for blockgroup
    from prior year
- `MonthTotal` (float): estimated total bus ridership for blockgroup in that
    month and year
- `month-year` (datetime): first day of month with year
- `active` (int): count of active businesses
- `new` (int): count of new businesses
- `prev_month-year` (datetime): first day of month for prior year
- `prev_yr_active` (int): count of active businesses from month in prior year
- `prev_yr_new` (int): count of new businesses from month in prior year
- `%_change_active` (float): percent change in active businesses from prior year
- `%_change_new` (float): percent change in new businesses from prior year
- `train_rides` (int): Number of train rides starting from a station within 2km
- `geometry` (geopandas polygon): shape data for blockgroup
    (Geographic 2D CRS: EPSG:4326)



### Pipeline_FinalAnalysis_Evaluation

Takes as input:

- `data/MERGED_DATA.geojson`

Produces:

- `pipeline_outputs/TestResults_Initial_2015-2016.csv`
- `pipeline_outputs/TestResults_Final_2017.csv`



### Other Modules & Code Snippets
- `Pipeline_FeatureImportance_tScores`: code snippet developed to receive
    best model params and create feature importance visualizations
- `Pipeline_MergedDataPreprocessing`: scratch code for preprocessing merged data
