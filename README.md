# HEC-HMS Toolbox

The **HEC-HMS Toolbox** library was developed in Matlab version 2018b to manipulate the hydrological modeling software HEC-HMS. Initially, **HEC-HMS Toolbox** was designed to work with version 4.2.1 of HEC-HMS. However, if the execution mode of the program, the structure, and logic of the files that configure HEC-HMS do not change, the library should be able to operate independently of the version. In this sense, it is recommended that the user check for updates in versions later than HEC-HMS 4.2.1.

At the moment, **HEC-HMS Toolbox** is functional for Curve Number - SCS models and routing with Muskingum. Manipulation for different configurations has not yet been developed.

# How to set up an execution?
**HEC-HMS Toolbox** works following the same principle on which HEC-HMS operates. In this sense, the first thing we need to do is set up our project in HEC-HMS, which consists of defining the modeling units and the models to be used. In this configuration, it is necessary to include parameters for each unit as well as the climate series to be used (precipitation events). In conclusion, the entire model in HEC-HMS must be manually built, so if it runs without any errors, the setup was done correctly.

### Important points for setting up a project in HEC-HMS
For a project to be compatible with **HEC-HMS Toolbox**, the following considerations must be taken into account:
* The basins must be named with an alphanumeric combination without spaces or special characters. For example, W1001, W1002, etc.
* The name of the Time-Series-Gage must have the same name as the basin to which the series corresponds. For example, Precip Gage W1001, Precip Gage W1002, etc.
* The river reaches must be named with an alphanumeric combination without spaces or special characters. For example, R101, R102, etc.

### Folder Structure and Project Setup
Once we have the model configured, we proceed to create a folder with the following structure:
* DATA
* CODES
* MODEL
* RESULTS
* TMP
  
In the MODEL folder, we will place the already configured model (the entire folder created by HEC-HMS). In the DATA folder, we will place the Excel files:
* HEC_HMS.csv
* Group_River.csv
* Group_Basin.csv

### Script for HEC-HMS Execution
Once this setup is complete, we create a script in the CODES folder in MATLAB to manipulate the model. In this script, we must include the following lines of code:

* Calling the HEC-HMS Toolbox library from MATLAB's default folder.
```
addpath('C:\Users\User\Desktop\HEC-HMS-Toolbox')
```
* Definition of the Project Directory (the folder with the previously created structure)
```
PathControl     = 'C:\Users\User\Desktop\HEC-HMS-Toolbox\MyProject-HEC';
```
* Directory where HEC-HMS is installed
```
PathHEC_HMS     = 'C:\Program Files (x86)\HEC\HEC-HMS\4.2.1';
```
* Directory where HEC-DSS is installed
```
Path_HEC_DSS    = 'C:\Program Files (x86)\HEC\HEC-DSSVue';
```
* Directory where our model is located
```
% PathModel       = 'C:\Users\User\Desktop\HEC-HMS-Toolbox\MyProject-HEC\MODEL\MyModel-HEC';
```
* Project name created in HEC-HMS
```
NameModel       = 'My-HEC';
```
* Name of the run configured in HEC-HMS
```
NameRun         = 'Run_1';
```
* Start date from which HEC-HMS will be executed
```
Date_Init       = datetime(2019,10,28,1,0,0);
```
* End date until which HEC-HMS will be executed
```
Date_End        = datetime(2019,10,30,0,0,0);
```
It is very important that the start and end dates of the HEC-HMS execution are properly provided with the climate data; otherwise, an error will occur. That is, the model execution dates must be the same as the configured rainfall events.

* Name of the outlet node of the area configured in HEC-HMS
```
Name_Output = 'Pte_Av_Domingo_Diaz';
```
* Definición de los pasos de tiempo de ejecución y de los eventos de lluvias configurados. HEC-HMS presenta por default los siguiente pasos de tiempo los cuales han sigo codificados de la siguiente manera:

| Code | HEC-HMS Time|
|--|--|
|   1   | 1 MIN |
|   2   | 2 MIN |
|   3   | 3 MIN |
|   4   | 4 MIN |
|   5   | 5 MIN |
|   6   | 6 MIN |
|   7   | 10 MIN |
|   8   | 12 MIN |
|   9   | 15 MIN |
|   10  | 20 MIN | 
|   11  | 30 MIN |
|   12  | 1 HOUR |
|   13  | 2 HOUR |
|   14  | 3 HOUR |
|   15  | 6 HOUR |
|   16  | 8 HOUR |
|   17  | 12 HOUR |

In this sense, if we want the time step in which HEC-HMS is executed to be 5 minutes, we must use code 5:
```
Code_dt_Run_Model = 5;
```
Now, the time steps for the execution and the events do not necessarily have to be the same, so this time step must also be defined. For example, if the series is configured hourly, the code would be 12.
```
Code_dt_TimeSeries = 12;
```
When HEC-HMS generates a project, it creates a .dss file, which is the database HEC-HMS operates with. The DSS file saves all the time series for each of the basins configured in the project. HEC structures the series of events inside the DSS as follows:
* Number: is the series ID
* Part A: is a field to identify the scenario to which the series belongs
* Part B: is the name of the Gauges created in HEC-HMS
* Part C: is the type of variable; for precipitation by default, it is PRECIP-INC
* Part D/Range: is the start date of the time series
* Part E: is the time step of the time series
* Part F: GAGE

To finalize the configuration, we need to provide Part A, C, and F. For example:
```
A = 'HIST_TR_100';
C = 'PRECIP-INC';
F = 'GAGE';
```
With the above-configured information, we can create an object in MATLAB that represents an execution.
```
Model = HEC_HMS( PathModel, PathHEC_HMS, Path_HEC_DSS, PathControl,...
                                     NameModel, NameRun, Date_Init, Date_End,...
                                     Code_dt_Run_Model, Code_dt_TimeSeries,...
                                     A, C, F);
```
To execute HEC-HMS, we run the following command:
```
Model.Cal_Lag;
Model.Run_HEC_HMS;
```

## How to configure the HEC-HMS.csv file?
The HEC-HMS file contains the model parameterization information for each configured unit. This file contains the following columns:
* NameBasin -> Name of the configured basins
* NameRiver -> Name of the river segments configured for routing
* AreaBasin -> Basin area (km2)
* Slope -> Basin slope (%)
* Longest -> Length of the river reach (m)
* GroupRiver -> Number of groups for segmenting parameters between river reaches
* GroupBasin -> Number of groups for segmenting parameters between basins
* Alpha_Ia -> Alpha parameter for initial condition in the curve number method
* K -> Muskingum parameter
* X -> Muskingum parameter
* Step -> Muskingum parameter. Numerical discretization
