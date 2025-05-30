PIC-SURE HPDS Phenotype Load 
----------------------------

# Preparing your input files

HPDS requires a GPG public key to encrypte the symmetric key used to secure
the data you load. The intention is that you will email that encrypted key
to whoever is responsible for the data and they will unlock the service after
deployment. For this test the symmetric key is left in place for HPDS to use
so unlocking the service is not necessary, but you still need to provide
a GPG public key in this file:

hpds/gpg_pub_key.asc

HPDS expects a single CSV file with all of your data. 
The CSV file must be sorted by CONCEPT_PATH, then by PATIENT_NUM. That CSV file should have 4 columns with the header: "PATIENT_NUM","CONCEPT_PATH","NVAL_NUM","TVAL_CHAR". 

* CSV: If you include a timestamp, then the csv file must be sorted by concept_path, patient_num, TIMESTAMP. That CSV file should have 5 columns with the header: "PATIENT_NUM","CONCEPT_PATH","NVAL_NUM","TVAL_CHAR","TIMESTAMP"

* RDBMS: If you include a timestamp, then the table must be sorted by concept_path, patient_num, fact_data START_DATE. That table should have 5 columns with the header: "PATIENT_NUM","CONCEPT_PATH","NVAL_NUM","TVAL_CHAR","TIMESTAMP"




*PATIENT_NUM* - An integer identifying the subject.

*CONCEPT_PATH* - A string identifying the concept, this is traditionally a 
path in an ontology separated by backslashes, but that is not a requirement
of HPDS. The PIC-SURE-UI assumes that you have organized your data this way,
so if you expect to browse your HPDS instance in PIC-SURE-UI then you should
define your ontology mappings in this CSV source column.

*NVAL_NUM* - Any numeric value that fits in a Java Float variable.

*TVAL_CHAR* - Any text value.

Each line in the CSV should have either a NVAL_NUM value that is a number
or a TVAL_CHAR value. If it has both, the NVAL_NUM will be loaded and the
TVAL_CHAR will be ignored. Additionally, if the first TVAL_CHAR in the file
for a concept is parseable as a numeric value, the concept will be loaded as
a numeric concept and all non-parseable values for that concept will be ignored.
This behavior is expected to be removed in a future release, it is there to
account for abuses of the i2b2 data model which is the primary source of
data in most HPDS environments to-date.

*TIMESTAMP*	
* CSV: A timestamp for the observation fact record should be expressed as the number of milliseconds since January 1, 1970 GMT. This is equivalent to the Unix Epoch time value for the time of the observation multiplied by 1000.

* RDBMS: Datetime for the fact data (start_date)

* The timestamp data should be loaded in ISO 8601 format, an internationally recognized standard for representing dates and times in a format that is clear to both people and machines and neutralized time zones. If data is loaded into PIC-SURE from different zones, the timezone is represented as Coordinated Universal Time (UTC). For some programs, such as GIC, the date and time data can have varying degrees of precision and meaning and should not be treated as precise values, especially when comparing data from different sources. For example, sometimes values are missing time information or have incorrect time zones at the time of data ingestion, and for some, the timestamp represents when the data was received, not when the event occurred.

For the example we are providing an allConcepts.csv file that is compressed
for compatibility with GitHub. The docker-compose service uncompresses the
file for you.

# Loading your data

To execute the loader just run the docker-compose file in this folder after 
updating the GPG_USER environment variable in the docker-compose file to match 
the user for your GPG key:

docker-compose up

The output of the load process will echo to your terminal, it is a lot
of output as it dumps each and every concept. Once it completes you will 
have 4 new files in the hpds folder:

allObservationsStore.javabin  -- This is all your data
columnMeta.javabin  -- This is the data dictionary
encryption_key  -- This is the unencrypted key
encryption_key.asc  -- This is the GPG encrypted message of the key

Note that the hpds folder is mounted within the docker-compose containers
as /opt/local/hpds. It is recommended that /opt/local/hpds is on an encrypted 
partition for any data with privacy concerns, but please follow your insitutional 
or project guidelines for data security regarding these things.

In production you would perform your loading on one server, then move the .javabin
files to the server that is running your HPDS instance. We typically put them
in date-stamped folders on an encrypted partition, then volume map the folder
to the /opt/local/hpds path where HPDS expects to find those files.

Once your load has completed, run this in the root directory of this 
repository:

docker-compose up -d

Then open your browser and point it at your docker-machine ip, localhost, or
the ip of whatever server you are running this on.


