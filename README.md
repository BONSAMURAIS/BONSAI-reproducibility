# Reproducibility Instructions
The following instructions are for reproducing the results in our recent ISWC paper:

`Transparent Integration and Sharing of Life Cycle Sustainability Data with Provenance`


## Overall Workflow
The overall workflow has several steps, which needs to be executed in the correct order.
Below is the list of steps, followed by individual instructions:

- Requirements
- Clone Repositories
- Getting Data
- Run Data Conversion
- Collect triple files
- Setup virtuoso
- Load triples
- Setup Yasgui


## Requirements
To run all scripts, a machine running linux needs to installed.
The machine should atleast have the following specs:

- Software;
	- Linux Ubuntu 18.04 Distribution (with git, bash, unzip, wget and docker)
	- Python 3.6+, pip and virtualenv

- Hardware:
	- RAM: 64GB RAM
	- DISK SPACE: ´\~500GB Free
	- CPU: 8 cores

## Clone Repositories
All code for reproducibility is publically available from git repositories.

You can clone all repositories needed for reproducibility with the following commands:

```bash
git clone https://github.com/BONSAMURAIS/arborist

git clone https://github.com/BONSAMURAIS/EXIOBASE-conversion-software

git clone https://github.com/BONSAMURAIS/ystafdb

git clone https://github.com/BONSAMURAIS/yasgui-query-interface 
```

## Getting Data
We make use of the exiobase and ystafdb datasets.
Data access to the exiobase dataset needs a free user. 
For convenience, we provide a copy of the dataset freely available at the following url: `https://silo1.sciencedata.dk/shared/20ee45e130a37e87c5b19e07b81b61ec`

You can download, upack and move exiobase files to their required positions with the following commands:
```bash
wget 'https://silo1.sciencedata.dk/themes/deic_theme_oc7/apps/files_sharing/public.php?service=files&t=20ee45e130a37e87c5b19e07b81b61ec&path=%2Fexiobase-3.3.17&files=EXIOBASE_3.3.17_hsut_2011.zip&download&g=' -O exiobase-dataset.zip

unzip exiobase-dataset.zip
rm -rf exiobase-dataset.zip

mv EXIOBASE_3.3.17_hsut_2011/MR_HSUT_2011_v3_3_17_extensions.xlsb arborist/arborist/data
mv EXIOBASE_3.3.17_hsut_2011/MR_HUSE_2011_v3_3_17.xlsb EXIOBASE-conversion-software/EXIOBASE_conversion_software/data/
mv EXIOBASE_3.3.17_hsut_2011/MR_HSUP_2011_v3_3_17.xlsb EXIOBASE-conversion-software/EXIOBASE_conversion_software/data/
```

Othervise the exiobase dataset `EXIOBASE 3.3.17 hsut 2011` can be downloaded from this url `https://www.exiobase.eu/index.php/welcome-to-exiobase`.
It can be found under the tab `DATA DOWNLOAD/EXIOBASE3 - hybrid`.
Extract the downloaded zip file and move files according to the above commands.

You can download and unpach the ystafdb files with the following commands:
```bash
wget 'https://www.sciencebase.gov/catalog/file/get/5b9a7c28e4b0d966b485d915?f=__disk__0f%2F58%2Fa7%2F0f58a74db669ee5418f36a698bc85781e867e0ab' -O ystafdb-input.zip

unzip ystafdb-input.zip -d ystafdb-input

mv ystafdb-input ystafdb
```

## Run Data Conversion
In this step we run the EXIOBASE-conversion-software, as well as the ystafdb software, to convert excel files to rdf data.

### EXIOBASE-conversion-software
The software is used to first convert the exiobase 3.3.17 xlsb dataset to a csv file, and from that extract the final triple graph.

To install the software, and convert the xlsb files to csv files, the following commands can be used:

```bash
cd ~/EXIOBASE-conversion-software

pipenv --python python3 install
pipenv shell

python setup.py install

excel2csv-cli -i EXIOBASE_conversion_software/data/MR_HSUP_2011_v3_3_17.xlsb -o EXIOBASE_conversion_software/data/
excel2csv-cli -i EXIOBASE_conversion_software/data/MR_HUSE_2011_v3_3_17.xlsb -o EXIOBASE_conversion_software/data/
```

Be aware, the following scripts can take several hours to run, and should in some environments be run in a terminal screen environment.
While still in the pipenv, we convert hsup data file to an rdf graph with the following commands:
```bash
csv2rdf-cli -i EXIOBASE_conversion_software/data/MR_HSUP_2011_v3_3_17.csv -o EXIOBASE_conversion_software/data/  -c HSUP --flowtype output --multifile 100000 --merge True

mv output/flows_merged.nt output/exiobase-hsup.nt
gzip output/exiobase-hsup.nt

rm -rf MR_HSUP_2011_v3_3_17*
```

We now do the same for the huse data file with the following commands:
```bash
csv2rdf-cli -i EXIOBASE_conversion_software/data/MR_HUSE_2011_v3_3_17.csv -o EXIOBASE_conversion_software/data/  -c HUSE --flowtype input --multifile 100000 --merge True

mv output/flows_merged.nt output/exiobase-huse.nt
gzip output/exiobase-huse.nt

rm -rf output/MR_HUSE_2011_v3_3_17*

exit
```

The two output rdf graphs for the hsup and huse data can now be found in the `output` folder as `exiobase-huse.nt.gz` and `exiobase-hsup.nt.gz`.

### arborist
The software is used to extract meta data from the exiobase dataset, used as a foundation of integration with other datasets.
For the installation and usage of the arborist dataset, run the following commands:
```bash
cd ~/arborist

rm -rf config.json

git clone https://gist.github.com/bd4ab2d5ed82a8523a162e76d968971a.git
mv bd4ab2d5ed82a8523a162e76d968971a/config.json .
rm -rf bd4ab2d5ed82a8523a162e76d968971a

pipenv --python python3 install
pipenv shell

python setup.py install
arborist-cli regenerate output -i arborist/data/

exit
```

### YSTAFDB
The software is used to extract triple graphs from the ystafdb dataset.
For the installation and usage of the ystafdb dataset, run the following commands:
```bash
cd ~/ystafdb

pipenv --python python3 install
pipenv shell

python setup.py install
ystafdb-cli -i ystafdb-input

exit
```
The ystafdb triple files can now be fould in the output folder

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
- flow/exiobase3_3_17/exiobase3_3_17.ttl -> exiobase3_3_17_emission.ttl
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
To setup virtuoso with docker, use the following commands:
```bash
docker pull openlink/virtuoso-opensource-7:latest

mkdir database

wget https://gist.githubusercontent.com/kuzeko/5d53f9800a4b6d45006f0f9dc322ed07/raw/bb2c404ea315f7f56e71523a87a7e6679815b13a/virtuoso.ini.example -O virtuoso.ini

mv virtuoso.ini database/

docker run --name vos -d --volume `pwd`/database:/database -v `pwd`/import:/import -t -p 1111:1111 -p 8890:8890 -i openlink/virtuoso-opensource-7:latest
```

## Load Triples
We now load all triples into virtuoso, using a script which can be executed through isql.
To download the script and import all graphs, use the following commands:
```bash
wget https://gist.github.com/c8069487db59827cd62ab3d7ebb132a5.git -O /import/import.isql

docker exec -it vos isql  1111 exec="LOAD /import/import.isql"
```

## setup Yasgui
The yasgui requires no installation.
We simply add the yasgui index file to the correct folder.
First enter the cloned yasgui folder.
`cd yasgui-query-interface`
Now move the index.html file to the virtuoso.
`docker cp index.html vos:/opt/virtuoso-opensource/vsp/index.html`