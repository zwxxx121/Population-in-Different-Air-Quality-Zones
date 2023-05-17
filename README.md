# Population-in-Different-Air-Quality-Zones
Identify the estimated percentage of US population that is residing in the different air quality zones based on Air Quality Index(AQI) using Google public datasets and BigQuery.

# Datasets(All from Google Public data):
bigquery-public-data:epa_historical_air_quality.o3_daily_summary

bigquery-public-data:geo_census_blockgroups. Us_blockgroups_national

bigquery-public-data:census_bureau_acs.blockgroup_2018_5yr

# Assumptions
1. the population growth in different states are the same

The latest available date for census is 2014-01-01; the last available date for AQI is 2022-05-31. In this case, the experiment assumes the population growth in different states are the same, so we can use the population in 2014 with the latest AQI since the ultimate goal is to calculate the percentage of population in different air quality zones.

2. AQI before 2021-05-31(one year before the latest available date) is expired

Theoretically, the AQI is a real-time index, while big query public data only has the historical data and there are only 3 counties with AQI on the latest day. Therefore, to classify different air quality zones more comprehensively, the experiment uses different classification with two types of observing windows. The first type of classification is based on the AQI of the latest available day of each region; the second type of classification is based on the AQI over the last one year of each region.
For both methods, only the data in the lag 1Y before 2022-05-31(2021-05-31 ~ 2022-05-31) is considered. The AQIs of the states and counties that haven’t been monitored in over a year are considered expired and therefore, do not hold any significance. 

# Methodology
1. Classify each region into four groups(AQI 0-50, AQI 51-100, AQI 101-150, AQI> 150) with 3 different methods.
2. Check whether AQI in the same state and county is in the same air quality zone. If not, find the monitor closest to the block group's geographic center. 
3. Impute missing AQI using the nearest monitoring site of these blockgroups(KNN).
4. Estimate the proportion of the US population that lives in 4 different air quality zones.

# 3 Solutions of Classifying Air Quality Zones
1. use AQI on the latest available date of each region

The assumption here is that AQI of regions didn’t change since the last time they’re monitored. We still need to remove the expired AQI of states and counties that haven’t been monitored in over a year.

2. use 75th percentile AQI in a year

Using the data from the o3_daily_summary table, the 75th percentile of the AQI distribution can be determined for each state, county, and monitoring site, then group the air quality zones based on this percentile. The assumption here is if over 25% AQI is larger than a certain number, the region will be classified as the air quality zone that contains this number.
For example, if the 75th percentile of AQI of a region is 77, group the region to AQI 50~100 because over 25% AQI is larger than 50, which indicates the air quality of that region is moderate. 

3. use the majority of days with AQI in a year

The assumption here is that the most appeared zone for each state, county, and monitoring site over a year will be defined as the final air quality zone for that region. 
For example, if a region has 277 days with AQI in 0-50, 20 days with AQI in 51-100, 4 days with AQI in 101-150, it can be classified as AQI 0-50 zone. This approach would provide a comprehensive and statistically sound way of determining the overall air quality for a given region, based on the frequency of different air quality zones. However,  it has the potential to underestimate the air quality zone of each region.
