# MESH-ERA5Land 	:globe_with_meridians: :artificial_satellite:

This repository generates the meteorological inputs of the land-surface model [MESH](https://wiki.usask.ca/display/MESH/About+MESH)
using the [ERA5-Land](https://cds.climate.copernicus.eu/cdsapp#!/dataset/reanalysis-era5-land?tab=overview) hourly data Reanalysis. The objective is to create a complete time series from several netCDF (\*.nc) files, convert them to local time, and generate a \*.r2c file for each meteorological variable.

## Functions
 The following gives a brief description of the individual files:
 * **main.m**: This is the only function to  modify and execute. Here, we specify the **Input** parameters (see below) of the **ERA5LAND_EXTRACT.m** and **Creating_r2c.m** functions. 
 * **ERA5LAND_EXTRACT.m**: It reads, concatenates, and converts the \*.nc files to local time. Additionally, this function extracts the values within the boundaries of a zone specified by a shapefile.
 * **util_getVariables.m**: A small function used by **ERA5LAND_EXTRACT.m** to read the \*.nc files.
 * **util_descVar.m**: A small function to deaccumulate variables accumulated from the beginning of the forecast time to the end of the forecast step (e.g., Total precipitation, Evaporation). We specify this via the **VarType** input.
 * **util_CatchMask**: This function creates a mask to select the values within the boundaries of a zone. 
 * **Creating_r2c.m**: Function to convert the \*.mat values generated by the **ERA5LAND_EXTRACT.m** function to \*.r2c
 * **util_r2cFile**: This function creates the \*.r2c files with the requirements of the MESH model. 

       NOTE: The variables to modify in the ''main.m'' script are identified with the comment: %To modify

##  main.m function
### Variables to modify:
 * **mainPath**: Path of the directory where the main file is located .
 * **shpPath**:  Path of the shapefile to create a mask.
 * **LocalZone**: Time difference between UTC and the local time (e.g., -05:00 for Quebec).
 * **CatchName**: The study area/catchment name (must be equal to the name of the shapefile).
 * **xOrigin**:  This is the x-coordinate of the point in the bottom left corner of the grid.
 * **yOrigin**:  This is the y-coordinate of the point in the bottom left corner of the grid.
 * **xCount** :  The number of points, or vertices, in each row of the grid,along the x-direction.
 * **yCount** :  The number of points, or vertices, in each column of the grid,along the y-direction.
 * **xDelta** :  The distance between two adjacent points in a row.
 * **yDelta** :  The distance between two adjacent points in a column.


## Meteorological variables needed to calculate the forcing inputs for MESH
The code could be used with any variable from the ERA5-Land database. However, in this case, only the variables necessary to determine the inputs of the MESH model are included (table below).

 | **VarName** | **VarName_nc** | **VarType** |**Unit ERA5-Land**|**Unit MESH**
 | --------------| ------------ |-----------|---------|--------------|
 |   DewPoint    |     d2m      |      0    |    K    | - |
 |    Pressure   |     sp       |      0    |    Pa   |Pa|
 |  Temperature  |     t2m      |      0    |    K    |K|
 | Precipitation |      tp      |      1    |    m    |mm s-1|
 |   Shortwave   |     ssrd     |      1    |  J m-2  |W m-2|
 |   LongWave    |     strd     |      1    |  J m-2  |W m-2|
 |    Wind_u     |     u10      |      0    |  m s-1  | m s-1
 |     Wind_v    |     v10      |      0    |  m s-1  | m s-1
 
 * **VarName_nc**: String cell vector with the meteo parameters' name attribute in the \*.nc file.
 * **VarName**: String cell vector with the meteo parameters' name (e.g., Precipitation, Temperature, Pression, etc.). 
 * **VarType**: 1 if the values of the variable in ERA5-Land are accumulated over time (e.g., precipitation, shortwave), 0 otherwise.

 ## Outputs:
 ### From the **ERA5LAND_EXTRACT.m** function
The output is a Matlab file (**VarName.mat**) with the following variables:
* **Data**: 3D matrix (longitude,latitude,time) with the values of the meteorological variable. Please note that output units are the same as ERA5-Land.
* **Date**: Vector with the dates in datenum format.
* **Longitude**: vector with longitude coordinates.
* **Latitude**: vector with latitude coordinates.
* **LonGrid**: 2D matrix (longitude, latitude) with the longitude value in each grid.
* **LatGrid**: 2D matrix (longitude, latitude) with the latitude value in each grid.
 
        Note: LonGrid and LatGrid are necessary to determine: xOrigin, yOrigin, xCount, yCount, xDelta, and yDelta.
 
 ### From the Creating_r2c.m function 
 The output is a .r2c file for each of the meteorological variables required for the MESH model (see table below)

 | **VarName**          | **KeyName**  | **Unit**  |**FileName**|
 | ---------------------| ------------ |-----------|--------------------|
 | Specific humidity    |     HU       |      K    | basin_humidity.r2c |
 |   Surface pressure   |     PO       |      Pa   |   basin_pres.r2c   |
 |    Air temperature   |     TT       |      K    |basin_temperature.r2c |
 | Precipitation rate   |      PR0     |    mm s-1 |    basin_rain.r2c |
 |      Shortwave       |     FB       |    W m-2  |   basin_shortwave.r2c |
 |       LongWave       |     FI       |    W m-2  |   basin_longwave.r2c |
 |      Wind speed      |     UV       |    m s-1  |   basin_wind.r2c |


## netCDF file specifications
* The .nc file name are expected to follow the format: **YYYY-MM-DD_YYYY-MM-DD.nc**, indicating the start and end date of the data. 

      Example: 2019-01-01_2019-12-31.nc
      * Start date: 2019-01-01
      * End date: 2019-12-31
      
* The .nc file should only contain one variable.  
* Each variable must have a folder with the corresponding .nc files. The name of each folder must be equal to **VarName** (see next section).

 ## Repository Folder 
 This Repository must cotains the following folders:
   

    .
    MESH-ERA5Land               #: main folder
    ├── netCDF_Files            #: Folder with the .nc data  (dataPath of the main.m function)             
    │   ├── DewPoint            #: Folder with DewPoint .nc files
    │   ├── Longwave            #: Folder with Longwave .nc files
    │   ├── Precipitation       #: Folder with Precipitation .nc files
    │   ├── Pressure            #: Folder with Pressure .nc files
    │   ├── Shortwave           #: Folder with Shortwave .nc files
    │   ├── Temperature         #: Folder with Temperature .nc files
    │   ├── Wind_u              #: Folder with u wind component .nc files
    │   └── Wind_v              #: Folder with V wind component .nc files
    ├── OutputERA5-Land         #: Folder with the .mat files (OutPath of the main.m function and dataPath of the Creating_r2c.m function) 
    └── r2cFiles                #: Folder with the .r2c files (OutPath of the Creating_r2c.m function)

## Contact
There is always a better way to do things :smile:. Please, feel free to contact me if you encounter any bugs or want to collaborate to improve the function. 
:e-mail: emixi-sthefany.valdez-medina.1@ulaval.ca
