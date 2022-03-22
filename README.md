# Overview

The sections below walk through the steps involved with each data processing notebook. Here are some overarching links for the data involved in this analysis process.

**Shapefile Data Downloads**
* [ZCTAs](https://www2.census.gov/geo/tiger/TIGER2021/ZCTA520/tl_2021_us_zcta520.zip)
* [States](https://www2.census.gov/geo/tiger/TIGER2021/STATE/tl_2021_us_state.zip)

**Low-to-Moderate-Income Data**
* [HUD Map](https://hudgis-hud.opendata.arcgis.com/datasets/HUD::low-to-moderate-income-population-by-tract)

**Building Footprint Data**
* [Microsoft](https://github.com/Microsoft/USBuildingFootprints)

**Obtaining Driving Routes**
* [Openrouteservice Directions API](https://openrouteservice.org/dev/#/api-docs/v2/directions)

**Calculating Isochrones**
* [Openrouteservice Isochrones API](https://openrouteservice.org/dev/#/api-docs/v2/isochrones)

And here are some links to data that was output by these notebooks and used for the maps and tables in the article. Some data may have been edited manually in the Mapshaper GUI or edited manually (as in the case of the HUD data).

**First Map (Deep South Amputation Rates)**
* [Zipcodes](https://media.mcclatchy.com/static/2022/diabetic-amputations/data/zipcodes.json)
* [Deep South States](https://media.mcclatchy.com/static/2022/diabetic-amputations/data/deep_south.json)
* [Surrounding States](https://media.mcclatchy.com/static/2022/diabetic-amputations/data/surrounding_states.json)

**Table of Diabetic Amputation Rates**
* [Rates/Rankings](https://media.mcclatchy.com/static/2022/diabetic-amputations/data/table_rates_rankings.json)

**Second Map (29203 Routes & Buildings)**
* [Low-to-Moderate Income Data](https://media.mcclatchy.com/static/2022/diabetic-amputations/data/income.json)
* [Grocery Routes](https://media.mcclatchy.com/static/2022/diabetic-amputations/data/routes_grocery.json)
* [Dollar Store Routes](https://media.mcclatchy.com/static/2022/diabetic-amputations/data/routes_dollar_store.json)
* [Dialysis Center Routes](https://media.mcclatchy.com/static/2022/diabetic-amputations/data/routes_dialysis.json)

# diabetic_amputations.ipynb

### (1) Mapping Zip Codes to State FIPS Codes
Source: https://www2.census.gov/geo/docs/maps-data/data/rel/zcta_place_rel_10.txt

### (2) Mapping U.S. State Names, U.S. State Abbreviations and FIPS Codes
Source: https://gist.github.com/rogerallen/1583593

### (3) Read & Format Diabetic Amputation Data by State
The State Media Co. filed open records requests to gain hospital discharge data from southeastern states from 2016 to 2020. The data consisted of 76 procedural codes for nontraumatic lower extremity amputations for diabetic patients, broken down by each patient’s ZIP code of residence. Four southeastern states (Louisiana, Arkansas, North Carolina and Tennessee) either did not have data or would not provide it. ZIP code populations, according to 5-year averages from the U.S. Census Bureau, were then divided by the number of amputations for each ZIP code for each year to create a rate. For privacy reasons, some states suppressed amputation data for certain ZIP codes when there were fewer than five procedures. The newspaper then took the population size of 29203 and compared it with ZIP codes the same size or larger in South Carolina, Georgia, Alabama and Mississippi.

### (4) Get Population Data by Year
The Census data we used comes from the ACS 5-year estimates at the ZCTA level (approximate polygons for ZIP codes, which are really delivery routes). We used the 5-year averages since a ZIP code is a very small geographic unit for ACS survey methods, and there simply isn’t high enough precision to get reliable estimates from the 1-year ACS or 3-year ACS datasets.

### (5) Add Population Data to Amputations Data
Similarly, we used 5-year averages as more reliable estimates for the amputation rate of a given ZIP code. We also calculated these rates per 10K residents and per 40K residents.

### (6) Get Insurance Data by Year
```
S2701_C01_001E - Total!!Estimate!!Civilian noninstitutionalized population
S2701_C04_001E - Uninsured!!Estimate!!Civilian noninstitutionalized population
S2701_C05_001E - Percent Uninsured!!Estimate!!Civilian noninstitutionalized population
```

### (7) Add Insurance Data to Amputations Data
Merged insurance data with amputation data.

### (8) Extrapolating VA's Q1-Q3 2020 Data
Virginia's dataset does not include 2016 and only includes Q1-Q3 (inclusive) in 2020. We don't have access to the monthly/daily distribution of amputations, so here we are extrapolating linearly to impute values for Q4, then rounding to whole numbers for the final extrapolated total for VA 2020 amputation data.

### (9) Calculating Averages and Rankings

This output looks at ZIP codes above 39000 people across Mississippi, Alabama, Georgia and South Carolina from 2016-2020. The purpose is to find rankings and average amputation rates across these years.

#### Methodology
The authors' chosen methodology was to compare population-wide diabetic amputation rates for ZIP codes **the same size or larger** than the point of reference of 29203 (whose 5-year average population is ~39.9K). However, there are many other options for methods of comparison. 

From a data similarity perspective, it is likely more appropriate to compare 29203 to a range of populations, both above and below its own population of 39K: for instance a range between 20K and 60K. If we followed this methodology, 29203's amputation rate would have the 11th-worst amputation rate instead of 1st-worst over the 5-year span (when including MS, AL, GA and SC). 

If we also add the states of Kentucky, West Virginia and Virginia (whose data was also collected by The State Media Co. - Florida was also collected, though there's a chance the methodology may count individual amputation procedures rather than number of individuals who received amputations) into this methodology, 29203 would rank 34th-worst. Separately, it may make sense to compare ZIP codes with similar population densities amongst ZIP codes instead. 

From a data metric perspective, a population-wide diabetic amputation rate is not the best metric for care specific to people with diabetes. If 100% of the population of a 10,000 person town had diabetes, and 10 people recieved amputations, the population-wide amputation rate would be **10 per 10K residents**. The diabetic-population amputation rate would be **10 per 10K diabetics**, since everyone has diabetes.

Now, if 10% of the population of a 10,000 person town had diabetes (i.e. 1000 people), and 10 people still received amputations, the population-wide amputation rate would still be **10 per 10K residents**. However, the diabetic-population amputation rate would now be **100 per 10K diabetics**. The second group of diabetics is receiving significantly worse care, with much worse outcomes. This metric would better encapsulate a comparison of diabetic amputations across ZIP codes as well. 

### (10) Finding population densities

We need to add population densities and a number of other features into the Census dataset of zipcodes. And we need to remove other fields that will only take up space. The Census ZCTA shapefile isn't broken down by state (likely because there are many ZIP codes whose area intersects multiple states). So the first few steps are an attempt to geographically filter the zipcodes using Census state shapefiles. For these steps, you need to install [Mapshaper](https://github.com/mbloch/mapshaper)

#### Methodology

Alabama's dataset was more accurately reported for 2018-2020, and Virginia's was more accurately reported for 2017-2020. For the overall statistics on number of amputations and amputation rate per 10K residents, these numbers are aggregated/averaged over 3 and 4 years, respectively.

### (11) Looking into Poverty Data 

Ultimately, for the article, we used Low-to-Moderate Income numbers by census tract, found at this [HUD map](https://hudgis-hud.opendata.arcgis.com/datasets/HUD::low-to-moderate-income-population-by-tract/explore?filters=eyJTVEFURSI6WyI0NSJdLCJDT1VOVFkiOlsiMDc5IiwiMDYzIiwiMDE3Il19&location=34.004900%2C-81.118087%2C9.97) (whose data is downloadable). This section gathers poverty data from the Census for Lexington County (FIPS: 063) and Richland County (FIPS: 079) in South Carolina.

### (12) Vehicle Ownership Data

A crucial factor of the USDA's Low-Income, Low-Access designation (modern, more descriptive version of the colloquial food desert) is vehicle ownership. This looks at the Census data for vehicle ownership by ZCTA.

# diabetic_amputations_routes.ipynb

### (1) Find Isochrones for Grocery Stores vs. Dollar Stores

In the Eau Claire community in Columbia, SC, we are interested in the proximity of residents to dollar stores vs proximity to grocery stores. Using the USDA's Low-Income, Low-Access definition, we are minimally interested in the distances at 1/2 mile and 1 mile. For low-income residents with low access to transportation, it's critical to be within 1/2 mile or at least 1 mile of a grocery store.

To help find these distances, we started with a dataset of buildings in the Columbia, SC area, derived from [Microsoft's Building Footprint Data](https://github.com/Microsoft/USBuildingFootprints). We used the Mapshaper GUI to take the full Microsoft Building Footpring dataset, clip it by a Columbia, SC shapefile, and output only the Columbia, SC buildings. The downside of the data is that it does not indicate which buildings are residential, so ultimately, the maps are more of a proxy than an exact representation of residential distance to stores.

In this step, we're clipping the building data even further to only include the greater Eau Claire area. After clipping, we then use an isochrone API from [Openrouteservice](https://openrouteservice.org/dev/#/api-docs/v2/isochrones) to create isochrone polygons (a polygon whose bounds are X miles away from a specified point) for all dollar stores and grocery stores. We then clip the building data by isochrones at 1/4 mile, 1/2 mile, 3/4 mile, and 1 mile distances away from the specific point. We output that data, reaggregate it and label each building by which isochrone zone it is in.

Ultimately each building will be tagged with a label indicating which isochrone it was grouped into. For instance, the isochrone tags for how far a building is from a grocery store would be the following. The value after the dash indicates the number of meters away from the grocery store since Openrouteservice uses meters. 
    
    grocery-403
    grocery-805
    grocery-1280
    grocery-1610

Metric units to Imperial units guide:

    403 meters ~ 1/4 mile
    805 meters ~ 1/2 mile
    1280 meters ~ 3/4 mile
    1610 meters ~ 1 mile
    
If the tag is empty, then the building is greater than 1610 meters (~1 mile) away from a grocery store.

### (2) Driving Routes to Grocery Stores, Dollar Stores & Dialysis Centers

Here we use the [Openrouteservice Directions API](https://openrouteservice.org/dev/#/api-docs/directions) to find driving directions from a single point to grocery stores, dollar stores and dialysis centers. I'm also simplifying the point vectors to make them less onerous for mapping. Need to be aware of rate limits - not implemented here.

### (3) Transit Routes

This was an attempt to use the [HERE Maps API](https://developer.here.com/documentation/public-transit/dev_guide/routing/shape-example.html) for transit directions. It didn't have very useful coverage for the datapoints we were examining. A better bet would probably be to use Google Maps Transit if their stringent licensing ends up working for y'all.
