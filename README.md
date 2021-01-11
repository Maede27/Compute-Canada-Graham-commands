# Compute-Canada-Graham-commands

Install PuTTY to connect to the Compute Canada (Graham Server)
Copy graham.computecanada.ca to Host Name or IP address in PuTTY, then select SSH and open 
After connecting to CC server, navigate to home/mehdisam
To install the R packages, you need to load "netcdf" module in addition to gcc and R:

module load gcc r netcdf udunits

Some R packages like "pbdNCDF4" are removed from the official mirrors and you need to download the sources files first:
wget https://cran.r-project.org/src/contrib/Archive/pbdNCDF4/pbdNCDF4_0.1-4.tar.gz

Now run R : 
R

In R environment:
> install.packages("ncdf4")
> #install.packages("/home/mehdisam/pbdNCDF4_0.1-4.tar.gz", repos = NULL, type="source") ## we don't need this package for downscaling
> library(ClimDown)
> library(ncdf4) 
#### read GCM file
> merge_new<-"pr_tasmax_tasmin.nc"
> mergedfile<-nc_open(merge_new)
#### read observation file
> fn_obs<-"/home/najafim/scratch/lake_winnipeg/Livneh_2015/livneh-red_assiniboine.nc"
> obs<-nc_open(fn_obs, write=TRUE)
#### rename variables in observation data via R as the variables have different names in observ and GCM files
> old_varname <-'Prec' #precipitation variable
> new_varname <-'pr'
> obs<- ncvar_rename(obs, old_varname, new_varname, verbose=FALSE)
#### or rename variables in observation data via cdo module in server (latter is better than former for renaming!)
####to load cdo module on Compute Canada:
module load intel/2018.3 openmpi/3.1.2 cdo/1.9.5 nco/4.6.6 
cdo chname,Prec,pr /home/najafim/scratch/lake_winnipeg/Livneh_2015/livneh-red_assiniboine.nc  livneh-red_assiniboine_1.nc
cdo chname,Tmax,tasmax /home/najafim/scratch/lake_winnipeg/Livneh_2015/livneh-red_assiniboine_1.nc  livneh-red_assiniboine_2.nc
cdo chname,Tmin,tasmin /home/najafim/scratch/lake_winnipeg/Livneh_2015/livneh-red_assiniboine_2.nc  livneh-red_assiniboine_renamed.nc

#### modify the variables' units in observation data (change C to celsius and mm to kg m-2 d-1)
cdo chunit,C,celsius livneh-red_assiniboine_renamed.nc  livneh-red_assiniboine_renamed_celsius.nc
cdo setattribute,pr@units="kg m-2 d-1" livneh-red_assiniboine_renamed_celsius.nc  livneh-red_assiniboine_renamed_modifiedUnits.nc

### BCCAQ in R 
> library(ncdf4)
> library(ClimDown)
> GCM<-"/home/najafim/scratch/lake_winnipeg/GCMs/CanESM5_raw_to_clip/step3-cdo_sellonlat-file/pr_tasmax_tasmin_Amon_CanESM5_h_ssp245_r1i1p1f1_1850-2100_clipped.nc"
> Obs<-"/home/najafim/scratch/lake_winnipeg/GCMs/Test_Obs_GCM/Test_Obs_newLon_sameVar_sameUnit.nc"
>#### repeat for each variable:
> bccaq.netcdf.wrapper(GCM, Obs, nc_out, varname = "tasmax")  ##nc_out should exist (create a nc file) in the current directory on the server


### Useful links:
https://code.mpimet.mpg.de/projects/cdo/wiki/Tutorial
https://www.unidata.ucar.edu/software/netcdf/workshops/2011/utilities/NcdumpExamples.html


### useful link to rotate lon from -180 to 180 to 0 to 360
https://sourceforge.net/p/nco/discussion/9830/thread/c527a930/?limit=25

#### nco command to rotate longitude in observation data from -180~180 to 0~360
#### first rotate the module nco:
module load intel/2018.3 openmpi/3.1.2 cdo/1.9.5 nco/4.6.6

ncap2 -O -s 'where(lon<0) lon=lon+360; where(lon<0) lon=lon+360' livneh-red_assiniboine_renamed_modifiedUnits_lon.nc obs_lon_nco.nc

#### to making the file readable, writable and executable in the server for everyone 
chmod 777 filename

####to be able to load both r and cdo modules at the same time on the server:
module load cdo/1.7.2  r/3.5.0



library(ncdf4)
library(ClimDown)
GCM<-"/home/najafim/scratch/lake_winnipeg/GCMs/CanESM5_raw_to_clip/step3-cdo_se$
Obs<-"./Test_Obs_newLon_sameVar_sameUnit.nc"

bccaq.netcdf.wrapper(GCM, Obs, pr_CanESM5_downscaled.nc, varname = "pr")
bccaq.netcdf.wrapper(GCM, Obs, tasmax_CanESM5_downscaled.nc, varname = "tasmax")
bccaq.netcdf.wrapper(GCM, Obs, tasmin_CanESM5_downscaled.nc, varname = "tasmin")


