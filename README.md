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
- Validate database


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
rm -rf ystafdb-input.zip

mv ystafdb-input ystafdb/
```

## Run Data Conversion
In this step we run the EXIOBASE-conversion-software, as well as the ystafdb software, to convert excel files to rdf data.

### EXIOBASE-conversion-software
The software is used to first convert the exiobase 3.3.17 xlsb dataset to a csv file, and from that extract the final triple graph.

To install the software, and convert the xlsb files to csv files, the following commands can be used:

```bash
cd ~/EXIOBASE-conversion-software
mkdir output

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

mv EXIOBASE_conversion_software/data/flows_merged.nt output/exiobase_hsup.nt
gzip output/exiobase_hsup.nt

rm -rf EXIOBASE_conversion_software/data/MR_HSUP_2011_v3_3_17*
```

We now do the same for the huse data file with the following commands:
```bash
csv2rdf-cli -i EXIOBASE_conversion_software/data/MR_HUSE_2011_v3_3_17.csv -o EXIOBASE_conversion_software/data/  -c HUSE --flowtype input --multifile 100000 --merge True

mv EXIOBASE_conversion_software/data/flows_merged.nt output/exiobase_huse.nt
gzip output/exiobase_huse.nt

rm -rf EXIOBASE_conversion_software/data/MR_HUSE_2011_v3_3_17*

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
cd ../
```
The ystafdb triple files can now be fould in the output folder

## collect triple files
We now move all output graphs to a single folder, for import into virtoso.
This is donw with the following commands:
```bash
git clone https://gist.github.com/cf16f495291d6f47fbd659367c2863ea.git

mv cf16f495291d6f47fbd659367c2863ea/file_mover.bash .

rm -rf cf16f495291d6f47fbd659367c2863ea
mkdir -p import

bash file_mover.bash

wget https://ontology.bonsai.uno/core/ontology_v0.2.ttl -O import/ontology_v0.2.ttl
```

## Setup Virtuoso
To setup virtuoso with docker, use the following commands:
```bash
docker pull openlink/virtuoso-opensource-7:latest

mkdir -p database

wget https://gist.githubusercontent.com/kuzeko/5d53f9800a4b6d45006f0f9dc322ed07/raw/bb2c404ea315f7f56e71523a87a7e6679815b13a/virtuoso.ini.example -O virtuoso.ini

mv virtuoso.ini database/

docker run --name vos -d --volume `pwd`/database:/database -v `pwd`/import:/import -t -p 1111:1111 -p 8890:8890 -i openlink/virtuoso-opensource-7:latest
```

## Load Triples
We now load all triples into virtuoso, using a script which can be executed through isql.
To download the script and import all graphs, use the following commands:
```bash
git clone https://gist.github.com/IKnowLogic/c8069487db59827cd62ab3d7ebb132a5

mv c8069487db59827cd62ab3d7ebb132a5/import.isql import/

rm -rf c8069487db59827cd62ab3d7ebb132a5

docker exec -it vos isql  1111 exec="LOAD /import/import.isql"
```

## setup Yasgui
The yasgui requires no installation.
We simply add the yasgui index file to the correct folder inside virtuoso.
`docker cp yasgui-query-interface/index.html vos:/opt/virtuoso-opensource/vsp/index.html`

## Validate Database
We have created several query tests which for which consistensy and correctness of the database can be shown.
We have provided query results from the original bonsai database, along with their respective queries.
The following process will run all queries on the new database, and md5 check the results agains the result files from the original database.
Use the following commands to test the consistensy of the database.
```bash
git clone https://github.com/IKnowLogic/BONSAI-reproducibility.git

git clone https://gist.github.com/6de02ab2abd5bbe351d1c251ed94f525.git
mv 6de02ab2abd5bbe351d1c251ed94f525/bonsai_database_test.bash BONSAI-reproducibility/

cd BONSAI-reproducibility

bash bonsai_database_test.bash
```

The database will now have all queries from the data_queries run agains it, and their respective output will be returned as a csv file.
The files will then be md5 checked against the output csv files of running the queries agains the original odas server database.
OK means the md5 hashes are identical, whereas FALSE means they are not identical.