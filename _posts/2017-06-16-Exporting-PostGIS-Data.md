---
category: Data Analysis
---
# Exporting PostGIS Data
This is going to go over a couple of different ways to get your data out of a PosgtreSQL table into some other format on your drive.

## Prerequisites

### GDAL / OGR
If you are thinking of exporting your data into the Esri File Geodatabase format (FileGDB), you are going to need to OGR. If your python installation was part of [Anaconda](https://www.continuum.io/downloads), you will already have the OGR package installed. However, as I'll mention below, you will likely need to take a few more steps to be ready to export.

If you want to see a list of all the available drivers within OGR, use the ogrinfo command line utility program and run the following command:
```
ogrinfo --formats
```
For ESRI super users, you will note that there is support already provided for the ESRI Shapefile format. However, note that there is no built-in support for the ESRI File Geodatabase format, which brings us to the next step...

### ESRI FileGDB Driver
GDAL / OGR is a pretty handy package, and it is already setup for output to the ESRI Shapefile format. However, for those of us primarily working in ESRI File Geodatabases, you will be SOL because it won't come with the drivers necessary to write Esri File Geodabases. Fortunately, you can find the necessary drivers through the [File Geodatabase API](http://appsforms.esri.com/products/download/#File_Geodatabase_API_1.3) provided by ESRI.

_Note: If you are only looking to read FileGDBs, you can use the [OpenFileGDB driver](http://www.gdal.org/drv_openfilegdb.html), which comes with the standard installation of GDAL / OGR. However, as mentioned earlier, you won't be able to use this one to write File Geodatabses. Also note that you will only be able to use this for File Geodatabases created by ArcGIS 9 and above._

## Tools for Exporting your PostGIS data

### ogr2ogr
Ogr2Ogr is a command-line utility program designed to convert spatial data between different file formats. This is the quickest and most often recommended approach for quickly getting your data out of a PostGIS Table onto your drive. The structure of how you would create a command using this utility is detailed [here](http://www.gdal.org/ogr2ogr.html) on the GDAL website. For example, from the windows command line, you could send the following command to pull a PostGIS table into the ESRI Shapefile format:
```
ogr2ogr -f "ESRI Shapefile" C:\Temp\test.shp PG: "host=localhost user=your_username_here dbname=your_db_name_here password=your_password_here" "your_table_name_here"
```
The first argument (-f "ESRI Shapefile") specifies that we want to output a file in the ESRI Shapefile format. The second argment (C:\Temp\test.shp) specifies the location and filename for our output shapefile. For the third argument, you pass the parameters to connect to your PostgreSQL database, and the final argument is the table name. In this case I wanted to dump the entire table, but you can also add SQL to add a filter to your export by replacing the table name with an SQL statment, as shown below:
```
ogr2ogr -f "ESRI Shapefile" C:\Temp\test.shp PG: "host=localhost user=your_username_here dbname=your_db_name_here password=your_password_here" -sql "SELECT name, the_geom FROM table_name"
```
Another reference I recommend for other common configurations is the [OGR2OGR Cheatsheet](http://www.bostongis.com/PrinterFriendly.aspx?content_name=ogr_cheatsheet) put together by Boston GIS.

If you want to run this utility from wihin Python (as I do), you will need to import the python subprocess module at the beginning of your script. You will want to configure your python script something like the following:
```
import subprocess
command = ["~\\PathTo\\ogr2ogr.exe",
          "-f",
          "ESRI Shapefile",
           "C:\Temp\test.shp",
           "PG:host=localhost user=your_username_here dbname=your_db_name_here password=your_password_here",
           "your_table_name_here"
          ]
    
command_run = subprocess.check_call(command)
```

_Note: Rather than using subprocess.call, I recommend subprocess.check_call since, unlike the call method, it will raise an error when the return code is non-zero, i.e. the command failed. Hat tip to [this post](https://gis.stackexchange.com/questions/154004/execute-ogr2ogr-from-python)._

### Fiona
Fiona is a python-specific package designed to help move data in and out of different formats. It is dependent on the drivers in the OGR library that you already installed, so you won't be able to get any file format in Fiona that you couldn't get out of the ogr2ogr utility. 


