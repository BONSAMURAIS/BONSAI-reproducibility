# Overall Workflow
The overall workflow has several steps, which needs to be executed in the correct order.
Below is the list of steps, followed by individual instructions:

- Install machine
- Clone repos
- Download files
- Start conversion scripts
- Collect triple files
- Setup virtuoso
- Load triples
- Setup Yasgui


## Install machine
To run all scripts, a machine running linux needs to installed.
The machine should atleast have the following specs:

- Ubuntu 18.04
- 64GB RAM
- 1TB Harddisk
- 8 cores


## Clone Repos
Multiple repositories needs to be cloned.
Start by cloning each of the following repositories: 

### Arborist
Arborist is used to extract meta data triples from the exiobase 3.3.17 dataset.	

- `https://github.com/BONSAMURAIS/arborist`

### EXIOBASE-conversion-software
EXIOBASE-conversion-software is used to extract Flows, Activities and BalanceableProperties from the exiobase 3.3.17 dataset.

- `https://github.com/BONSAMURAIS/EXIOBASE-conversion-software`

### YSTAFDB
YSTAFDB is used to extract Flows, Activities, BalanceableProperties, Flow Objects, Locations, Activity Types, Foaf and Provenance information from the YSTAFDB database.

- `https://github.com/BONSAMURAIS/ystafdb`

### YASGUI
This software is the modified version of the yasgui, used to query the data.

- `https://github.com/BONSAMURAIS/yasgui-query-interface`


## Download files
We make use of the exiobase and ystafdb datasets.
In this step we download and extract the datasets to their correct repos.

Data access to the exiobase dataset needs a free user. 
For convenience, we also provide a copy of the dataset freely available at the following url: `https://silo1.sciencedata.dk/shared/20ee45e130a37e87c5b19e07b81b61ec`
Othervise the exiobase dataset `EXIOBASE 3.3.17 hsut 2011` can be downloaded from this url `https://www.exiobase.eu/index.php/welcome-to-exiobase`.
It can be found under the tab `DATA DOWNLOAD/EXIOBASE3 - hybrid`.
Extract the downloaded zip file.

- Copy the file `MR_HSUT_2011_v3_3_17_extensions.xlsb` into the `arborist/arborist/data` folder.
- Copy both the `MR_HSUP_2011_v3_3_17.xlsb` and the `MR_HUSE_2011_v3_3_17.xlsb` files to the `EXIOBASE-conversion-software/EXIOBASE-conversion-software/data` folder.

Download the ystafdb dataset from this url `https://www.sciencebase.gov/catalog/file/get/5b9a7c28e4b0d966b485d915?f=__disk__0f%2F58%2Fa7%2F0f58a74db669ee5418f36a698bc85781e867e0ab`.
Extract the contents of the zip file. 
As an example, the data can be placed in a folder `ystafdb-input`. 
The following files from the Base Data are mandatory to have in the folder:

- material_names.csv
- subsystems.csv
- flows.csv
- publications.csv
- reference_spaces.csv
- reference_materials.csv
- reference_timeframes.csv

## start conversion scripts
In this step we run the EXIOBASE-conversion-software, as well as the ystafdb software, to convert excel files to rdf data.

### EXIOBASE-conversion-software
The software is used to first convert the exiobase 3.3.17 xlsb dataset to a csv file, and from that extract the final triple graph.
The tool runs in python 3.7. 
For the usage of the EXIOBASE-conversion-software software for exiobase data extraction, as used in this paper, and presented from the `odas.aau.dk server`, the following steps were taken:

- Install python 3.7 along with pipenv
- Enter EXIOBASE-conversion-software: `cd EXIOBASE-conversion-software`
- Install pipenv: `pipenv install`
- Enter pipenv: `pipenv shell`
- Install package: `python setup.py install`
- Convert hsup xlsb to csv file: `excel2csv-cli -i EXIOBASE_conversion_software/data/MR_HSUP_2011_v3_3_17.xlsb -o EXIOBASE_conversion_software/data/`
- Convert huse xlsb to csv file: `excel2csv-cli -i EXIOBASE_conversion_software/data/MR_HUSE_2011_v3_3_17.xlsb -o EXIOBASE_conversion_software/data/`

- Convert hsup csv to nt graph: `csv2rdf-cli -i EXIOBASE_conversion_software/data/MR_HSUP_2011_v3_3_17.csv -o EXIOBASE_conversion_software/data/  -c HSUP --flowtype output --multifile 100000 --merge True`
- The data folder will now be filled with Many .nt files each containing a small portion of all hsup triples. 
- The only relevent file is the `flows_merged.nt` file.
- To preserve space, rename the file to `exiobase-hsup.nt`, gzip the file, and remove all smaller files with names like `MR_HSUP_2011_v3_3_17_x.nt`.

- Convert huse csv to nt graph: `csv2rdf-cli -i EXIOBASE_conversion_software/data/MR_HUSE_2011_v3_3_17.csv -o EXIOBASE_conversion_software/data/  -c HUSE --flowtype input --multifile 100000 --merge True`
- The data folder will now be filled with Many .nt files each containing a small portion of all huse triples. The only relevent file is the `flows_merged.nt` file.
- To preserve space, rename the file to `exiobase-huse.nt`, gzip the file, and remove all smaller files with names like `MR_HUSE_2011_v3_3_17_x.nt`.

### arborist
The tool runs in python 3.7. 
For the usage of the arborist software for exiobase data extraction, as used in this paper, and presented from the odas.aau.dk server, the following steps were taken:

- In the config.json file, set all values to true
- Install python 3.7 along with pipenv
- Enter arborist folder: `cd arborist`
- Install pipenv: `pipenv install`
- Enter pipenv: `pipenv shell`
- Run install script: `python setup.py install`
- Run arborist cli: `arborist-cli regenerate output`
- The generated triples can now be found in the output folder

### YSTAFDB
The tool runs in python 3.7.

For the usage of the YSTAFDB software for YSTAFDB data extraction, as used in this paper, and presented from the odas.aau.dk server, the following steps were taken:

- Download the ystafdb csv files as described in the readme, and put them in the ystafdb/data folder.
- Install python 3.7 along with pipenv
- Enter ystafdb: `cd ystafdb`
- Install pipenv: `pipenv install`
- Enter pipenv: `pipenv shell`
- Install package: `python setup.py install`
- From the Download files section of the readme, ystafdb data was downloaded and put in a file called `ystafdb-input`
- Extract triples: `ystafdb-cli -i /path/to/ystafdb-input`
- The ystafdb triples can now be fould in the output folder

## collect triple files
Create a folder `import` to hold all triple graph files, which we want to load into virtuoso.
Go through all files of the `ystafdb/output` folder, rename the files as follows, according to their location in the `output` folder structure, and add them to the `import` folder.

- activitytype/ystafdb/ystafdb.ttl -> act_ystafdb.ttl
- flowobject/ystafdb/ystafdb.ttl -> flo_obj_ystafdb.ttl
- foaf/ystafdb/ystafdb.ttl -> foaf_ystafdb.ttl
- location/ystafdb/ystafdb.ttl -> loc_ystafdb.ttl
- prov/ystafdb/ystafdb.ttl -> prov_ystafdb.ttl
- flow/ystafdb/huse/huse.ttl -> ystafdb_huse.ttl

Go through all files of the `arborist/output` folder, rename the files as follows, according to their location in the `output` folder structure, and add them to the `import` folder.

- activitytype/exiobase3_3_17/exiobase3_3_17.ttl -> act_exiobase3_3_17.ttl
- flowobject/exiobase3_3_17/exiobase3_3_17.ttl -> flo_obj_exiobase3_3_17.ttl
- foaf/foaf.ttl -> foaf_exiobase3_3_17.ttl
- location/exiobase3_3_17/exiobase3_3_17.ttl -> loc_exiobase3_3_17.ttl
- prov/prov.ttl -> prov_exiobase3_3_17.ttl
- flow/exiobase3_3_17/emission -> exiobase3_3_17_emission.ttl
- time/time.ttl -> time.ttl
- unit/unit.ttl -> unit.ttl
- activitytype/lcia/climate_change/climate_change.ttl -> act_climate_change.ttl
- activitytype/core/electricity_grid/electricity_grid.ttl -> act_electricity_grid.ttl
- activitytype/entsoe/entsoe.ttl -> act_entsoe.ttl
- flowobject/lcia/climate_change/climate_change.ttl -> flo_obj_climate_change.ttl
- flowobject/core/electricity_grid/electricity_grid.ttl -> flo_obj_electricity_grid.ttl
- flowobject/us_epa_elem/us_epa_elem.ttl -> flo_obj_us_epa_elem.ttl

From the `EXIOBASE_conversion_software/data` folder, move the gzipped huse and hsup files to the import folder.
- exiobase_hsup.nt.gz
- exiobase_huse.nt.gz

Download the version `0.2` of the BONSAI ontology from this url `https://ontology.bonsai.uno/core/ontology_v0.2.ttl` and add it to the import folder.
- ontology_v0.2.ttl


## Setup Virtuoso
Pull docker Virtuoso
`docker pull openlink/virtuoso-opensource-7:latest`
Made database directory
`mkdir -p database`
Get custom virtuoso.ini file
`wget https://gist.githubusercontent.com/kuzeko/5d53f9800a4b6d45006f0f9dc322ed07/raw/bb2c404ea315f7f56e71523a87a7e6679815b13a/virtuoso.ini.example -O virtuoso.ini`
Make database directory
`mv virtuoso.ini database/`
Make import folder
`mkdir -p import`
Run docker
`docker run --name vos -d --volume `pwd`/database:/database -v `pwd`/import:/import -t -p 1111:1111 -p 8890:8890 -i openlink/virtuoso-opensource-7:latest`

## Load Triples
Download script to import graphs into virtuoso.
`wget https://gist.github.com/c8069487db59827cd62ab3d7ebb132a5.git -O /import/import.isql`
Run import script
`docker exec -it vos isql  1111 exec="LOAD /import/import.isql"`

## setup Yasgui
The yasgui requires no installation.
We simply add the yasgui index file to the correct folder.
First enter the cloned yasgui folder.
`cd yasgui-query-interface`
Now move the index.html file to the virtuoso.
`docker cp index.html vos:/opt/virtuoso-opensource/vsp/index.html`