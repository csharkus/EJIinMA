# EJIinMA
EJI Flooding in Massachusetts Project
# Download Required Data 

Data was obtained from MassGIS (Bureau of Geographic Information) in December, 2021. Copies of data obtained from MassGIS are located in the GitHub repository. As MassGIS updates files, they may differ from the original files obtained at the start of this study.  

https://www.mass.gov/info-details/massgis-data-layers  

Massachusetts Environmental Justice Community Data (2010 and 2020) 

Massachusetts EJ community data obtained from Massachusetts Department of Energy Affairs and massgis for 2010 and 2020 

Https://www.mass.gov/info-details/massgis-data-2010-us-census-environmental-justice-populations 

Https://www.mass.gov/info-details/massgis-data-2020-environmental-justice-populations 

Census block group and tract data (2010) 

Massachusetts released block group and tract data from the 2020 U.S. Census recently, which can be found here: https://www.mass.gov/info-details/massgis-data-2020-us-census 

MassDEP Hydrography (1:25,000) 

This data describes lakes, pounds, rivers, streams, and other water features defined by the Massachusetts Department of Environmental Protection. https://www.mass.gov/info-details/massgis-data-massdep-hydrography-125000 

FEMA National Flood Hazard Layer 

The National Flood Hazard Layer (NFHL) dataset represents the current effective flood risk data for those parts of the country where maps have been modernized by the Federal Emergency Management Agency (FEMA), as described by MassGIS. This may be found here: https://www.mass.gov/info-details/massgis-data-fema-national-flood-hazard-layer  

Town Survey Data (2010) 

Massachusetts released municipality data based on the U.S. Census Bureau’s 2020 data release, which can be found here: https://www.mass.gov/info-details/massgis-data-2020-us-census-towns 

# Initialize ArcGIS Pro and Prepare the Census Data  

Make sure ArcGIS Pro is up to date.  

Open a new project file 

Click on “Add data” 

Add the following shapefiles: 

2010 block groups 

Town data 

Flood zone 

Hydrography 

EJ 2010 

EJ 2020 

Do a Spatial join of the 2010 EJ Layer to the 2010 Census Block Group Layer 

A spatial join combines the attributes of two spatial layers based on their location. The methodology that we tried included the center within and finally largest overlap. The correct methodology for this dataset was largest overlap. To find this tool use the search bar and type in “spatial join”  

Target: EJ 2010 Input: 2010 block groups 

In the fields menu remove everything except GEOID, EJ Criteria, towns 

Output: EJ2010joinBGs.shp 

Use labeling to check and make sure that the GEOID's match.  

Notice the join count field.  

Do an Add Join  (table join) on the 2010 census block group file (The original file) to the EJ2010joinBGs.shp   

An attribute table join on the new census block group file you just made. The field we join on is the GEOID10. Join all target features. The 2010 block group file will now have the GEOIDs,  

Right click on the census data, joins and relates, add join. Input: GEOID10, JoinTable: 2010EJjoinBGs Joinfield: GEOID10, Keep all target fields.  

Joins are temporary. We want to make it permanent. Right click and export features as a copy. Save them in a name that makes sense (2010blockgroupsjoinEJ or 2010BGsjoinEJ.shp) 

Make sure the number of records in the join match your original datasets. Do a select by attribute and make sure that it makes sense.  

Dissolve the flood layer into a single polygon 

Select by location in the towns layer for anything that contains flood zones (input feature = towns, relationship = contains, selecting feature = flood zones). This gets rid of the EJ polygons outside of where we have the flood zones.  

Dissolve on the selection. No dissolve field.  

Input: Towns selection 

Output: townsonlyflooded 

Clip the EJ layer to the extent of the flood zone (can you do the pairwise clip) 

The purpose of this is to only include areas in Massachusetts that have flood data. This is a cookie cutter. This gets rid of information outside of the flood zone. Since we clipped it to the flood zone after the join, we only have to run it just on the 2010 and 2020 data.  

For the 2010 dataset 

Input:2010blockgroupsjoinEJ or 2010BGsjoinEJ.shp, Clip features: TownsOnlyFlooded,  

Output: 2010blockgroupsjoinEJClipFloodZones or  2010BGsjoinEJjoinFloodZones.shp 

For the 2020 dataset  

Input: EJ2020, Clip features: TownsOnlyFlooded 

Output: EJ2020FloodedZones 

Some notes: The clip first taught us that when we clipped the EJ polyogons to the flood zones is how we found out there are 1,521. Our final input is 4,545. All of the block groups in Massachusetts in the flood zones with EJ criteria attached. The 2020 EJ layer already has GEOID.  

#ArcGIS Preparation of flood data  

Use Pairwise Dissolve to make the flood zones into 1 polygon 

This simplifies everything just to flood zones A, AE, X etc  

Input: floodzones 

Dissolve fields: Flood Zones 

Output: FloodZonesPWDissolve 

Pairwise Erase the hydro from these flood zones  

This removes the hydrography from the flood zones to avoid overcounting.  

Input: FloodZonesPWDissolve 

Erase: hydro 

Output: FloodZonesPWDissolveEraseHydro.shp  

Add field to the attribute table of FloodZonesPWDissolveEraseHydro.shp  

This creates a single file that removes everything that is not 100-year and 500-year flooding 

In the attribute table bar, next to field, click add field  

Field name: Zone 

Data type: Text 

Length: 5 (or more) 

Click save 

Manually type in the designation for each zone. (AEAE etc = 100, X = 500) 

Under Edit save the feature layer  

Pairwise dissolve on the FloodZonesPWDissolveEraseHydro  

on the zone field to consolidate 1 polygon for all the 5 100s and 1 polygon for the X and delete all the blanks.  

Input: FloodZonesPWDissolveEraseHydro 

Output: FloodZonesPWDissolveEraseHydroPWDissolve 

Dissolve field: Zone  

No stats etc  

Delete row 0 in the attribute table  

#Tabulate Flood Zones in ArcGIS Pro and export xls files 

Pairwise erase on the 2010blockgroupsjoinEJclipfloodzones 

We did this workflow to erase the hydrography features from the EJ layer and the block group layer. The pairwise erase is a cookie cutter that removes what we do not need.  

Input: 2010blockgroupsjoinEJclipfloodzones 

Erase features: hydro 

Output: 2010BGsjoinEJclipFZeraseHydro 

Calculate geometry attributes tool the 2010BGsjoinEJclipFZeraseHydro 

Geometry properties: Geodesic area  

Add new field: area_sqkm  

Property = area (geodesic) 

Area unit: Square kilometers  

Check that the new field was added  

Run tabulate intersection tool  

This creates a table that shows the total area and percentage of area that occupies the intersection (i.e, percentage of flooding that occupies EJ block groups) 

Input features: 2010BGsjoinEJclipFZeraseHydro 

Zone features: GEOID10  

Class: FloodZonesPWDissolveEraseHydroPWDissolve 

Output (we had 4 output tables) at the town level and BG level for 100- and 500-year floods.  

EJ_tabInt_500.txt 

EJ_tabInt_100.txt 

BGs_tabInt_100.txt 

BGs_tabInt_500.txt 

Classfields: Zone (either 100 or 500) 

Output units: Sq Km 

Do a visual check to see if the results match what the output looks like in a case study area. For example, a town that has a large spatial area of flooding.  

Export tables to excel  

Open excel, start a new workbook 

In the data tab, click on get data from CSV  

Export both the block group data and the flood data and merge the files either in excel or Stata.  

Rename the files towns_flooded_100.xlsx, bgs_flooded_100.xlsx, bgs_flooded_500.xlsx, and towns_flooded_500.xlsx 

#Make Figures in ArcGIS and Export Figures 

Create a layout  

Inset a map frame to the default extent  

Zoom into the desired case study area (for example, Lawrence, Massachusetts) 

Add the following map features 

North arrow 

Scale bar (in miles) 

Legend  

Add graticules 

Click on layout then data frame properties  

Click the Grids tab. 

Click the New Grid button. 

Click the Graticule option in Grids and Graticules Wizard. 

Add color configuration 

Click on map 

Click on contents pane  

Click on symbology properties tab 

Under appearance click on outline 

Set every outline as 0.7 pt gray 50%  

For the municipalities outline, set it at 0.7 pt, HEX # 4C0073  

Edit each color symbol using the HEX # as follows: 

E - F7FCF0  

I - E0F3DB 

IE - E3EBC5  

M – FFEBBE  

ME - FFD5D7  

MI - EE9A9E  

MIE - F57A7A  

Hydrography – CCEDFF 

100-year flood - 9EBBD7  

500-year flood - BFD9F2  

Set the drawing order as follows (check the box on the left of the dataset to have it appear on the map) 

Flood zones 

Hydrography 

EJ2020 

EJ2010  

Check that the layout file looks appropriate. Use desired background map by selecting “basemap”. We used the ocean.  

Click on share 

Click on output  

Select desired output path and file format  

Ensure that coordinate system and transformation is appropriate  

Right click on the map 

Select coordinate system 

Make sure it is set to NAD 1983 StatePlane Massachusetts FIPS 2001 (meters)  

Click on transformation 

The following transformations are required to position datasets in the map 

Layer coordinate system – WGS 1984 

Transformation path – WGS 1984 (ITRF00) to NAD 1983 

Map coordinate system – NAD 1983 no vertical CS  

Share the ArcGIS mapfile  

Click on share  

Click on project 

Based on desired products to share, follow ArcGIS project sharing documentation https://pro.arcgis.com/en/pro-app/latest/help/sharing/overview/project-package.htm  

#Cleaning Data Files in Stata  

Open Stata, create a dofile EJtowns_analysis_dofile 

Follow the following commands for towns 100 and 500-year flooding  

***For Towns 100 and 500 year 

cd "C:\" 

import excel "C:\Users\ towns_flooded_100.xlsx", sheet("Sheet1") firstrow 

sort townID Year 

save "C:\Users\towns_flooded_100.dta" 

import excel "C:\\towns_flooded_500.xlsx", sheet("Sheet1") firstrow 

sort townID Year 

save "C:\ towns_flooded_500.dta" 

use "C:\ towns_flooded_100.dta", clear 

 

merge 1:1 townID Year using "C:\\towns_flooded_500.dta" 

save "C:\\towns_100_500_flooded.dta" 

 

replace flood_area_percent_500 = 0 if flood_area_percent_500 == . 

 

generate totalfloodzone = flood_area_percent_100 + flood_area_percent_500 

save "C:\ towns_100_500_flooded.dta" , replace 

drop _merge 

save "C:\ \towns_100_500_flooded.dta" , replace 

 

 

generate EJ_n = . 

replace EJ_n = 1 if EJ_criteria == "NA" 

replace EJ_n = 2 if EJ_criteria == "E" 

replace EJ_n = 3 if EJ_criteria == "I" 

replace EJ_n = 4 if EJ_criteria == "IE" 

replace EJ_n = 5 if EJ_criteria == "M" 

replace EJ_n = 6 if EJ_criteria == "ME" 

replace EJ_n = 7 if EJ_criteria == "MI" 

replace EJ_n = 8 if EJ_criteria == "MIE" 

 

generate EJ_criteria_aggregated = EJ_n 

replace EJ_criteria_aggregated = 5 if EJ_n == 2 | EJ_n == 6 

save " towns_100_500_flooded.dta" , replace 
 

Follow the following code for block group 100 and 500 year flooding data preparation: 
***For Block groups 100 year and 500 year **** 

cd "C:\" 

import excel "C:\\bgs_flooded_100.xlsx", sheet("Sheet1") firstrow 

sort GEOID Year 

save "C:\\bgs_flooded_100.dta" 

import excel "C:\\bgs_flooded_500.xlsx", sheet("Sheet1") firstrow 

sort GEOID Year 

save "C:\\bgs_flooded_500.dta" 

use "C:\\bgs_flooded_100.dta", clear 

 

merge 1:1 GEOID Year using "C:\\bgs_flooded_500.dta" 

save "C:\\bgs_100_500_flooded.dta" 

 

replace flood_area_percent_500 = 0 if flood_area_percent_500 == . 

 

generate totalfloodzone = flood_area_percent_100 + flood_area_percent_500 

save "C:\\bgs_100_500_flooded.dta" , replace 

drop _merge 

save "C:\\bgs_100_500_flooded.dta" , replace 

***Pause to save the combined excel sheet 

import excel "C:\\bgs_flooded_100500.xlsx", sheet("Sheet1") firstrow 

sort GEOID Year 

save "C:\\bgs_flooded_100500.dta" , replace 

use "C:\\bgs_flooded_100500.dta", clear 

generate EJ_n = . 

replace EJ_n = 1 if EJ_criteria == "NA" 

replace EJ_n = 2 if EJ_criteria == "E" 

replace EJ_n = 3 if EJ_criteria == "I" 

replace EJ_n = 4 if EJ_criteria == "IE" 

replace EJ_n = 5 if EJ_criteria == "M" 

replace EJ_n = 6 if EJ_criteria == "ME" 

replace EJ_n = 7 if EJ_criteria == "MI" 

replace EJ_n = 8 if EJ_criteria == "MIE" 

 

generate EJ_criteria_aggregated = EJ_n 

replace EJ_criteria_aggregated = 5 if EJ_n == 2 | EJ_n == 6 

save "C:\\bgs_flooded_100500.dta" , replace 

#Running Analysis in Stata   

The specific numeric values in the analysis were based on the centile distributions of the total flood zones at the block group level and town level. For the purpose of analysis, these numbers could change based on the researcher’s question and interest. We found it interesting to analyze at the values we chose.  

Perform analysis on the block groups: 

mlogit EJ_criteria_aggregated flood_area_percent_100 i.Year, cluster(GEOID) 

 

summarize total_flood_zone, detail 

centile total_flood_zone, centile (0(10)100) 

 

outreg2 using bg100logit, sideway paren dec(3) alpha(.001,.01,.05,.1) symbol(***,**,*,+) replace excel 

margins, at(flood_area_percent_100=1.34 flood_area_percent_100=9.14 flood_area_percent_100=18.42 flood_area_percent_100=39.21 flood_area_percent_100=57.10255) 

margins i.Year 

summarize flood_area_percent_100, detail 

centile flood_area_percent_100, centile (0(10)100) 

 

mlogit EJ_criteria_aggregated total_flood_zone i.Year, cluster(GEOID) 

outreg2 using bg100logit, sideway paren dec(3) alpha(.001,.01,.05,.1) symbol(***,**,*,+) append excel 

margins, at(total_flood_zone=0.073239 total_flood_zone=4.537015 total_flood_zone=9.51165 total_flood_zone=19.15735 total_flood_zone=50.05703) 

margins i.Year 

summarize total_flood_zone, detail 

centile total_flood_zone, centile (0(10)100) 

Perform the analysis at the town level: 

 

mlogit EJ_criteria_aggregated flood_area_percent_100 i.Year, cluster(townID) 

outreg2 using town100logit, sideway paren dec(3) alpha(.001,.01,.05,.1) symbol(***,**,*,+) replace excel 

margins, at(flood_area_percent_100=0.073192 flood_area_percent_100=2.7905 flood_area_percent_100=6.830885 flood_area_percent_100=16.69602 flood_area_percent_100=50.05703) 

margins i.Year 

summarize flood_area_percent_100, detail 

centile flood_area_percent_100, centile (0(10)100) 

 

mlogit EJ_criteria_aggregated totalfloodzone i.Year, cluster(townID) 

outreg2 using town100logit, sideway paren dec(3) alpha(.001,.01,.05,.1) symbol(***,**,*,+) append excel 

margins, at(totalfloodzone=0.073239 totalfloodzone=4.537015 totalfloodzone=9.51165 totalfloodzone=19.15735 totalfloodzone=50.05703) 

margins i.Year 

summarize totalfloodzone, detail 

centile totalfloodzone, centile (0(10)100) 

Create figures in Stata 

tab EJ_criteria, chi2 

generate EJ_criteria_aggregated = EJ_criteria 

replace EJ_criteria_aggregated = 5 if EJ_criteria == 2 | EJ_criteria == 6 

save "C:\Users\cshar\Desktop\SVI_summer\SVI_Fall\EJtowns_analysis.dta" , replace 

tab EJ_criteria_aggregated 

mlogit EJ_criteria_aggregated percentoftownareaflooded i.year, base(1) cluster(town_code) 

 

outreg2 using ejlogit, sideway paren dec(3) alpha(.001,.01,.05,.1) symbol(***,**,*,+) replace excel 

margins, at(percentoftownareaflooded=0 percentoftownareaflooded=.065 percentoftownareaflooded=0.107 percentoftownareaflooded=0.358) 

margins i.year 

marginsplot  

summarize percentoftownareaflooded, detail 

centile percentoftownareaflooded, centile (0(10)100) 

 

pwcorr 

 

regress 

estat vif 

Using the outreg2 function, all charts were exported to excel and saved. Each chart was designed manually using excel spreadsheets. Please see the Stata figures file to see the excel functions used to make figures.  

From Stata select  

 

 

 

 
