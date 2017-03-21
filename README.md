# MIK_workshop_Islandoracon_2017

Islandoracon 2017 Post-Conference Session "Move to Islandora Kit: For all your migration and batch loading preparation needs"

## Overview

* 3 hours long
* Outcomes
  * Use MIK to create Islandora import packages from CSV metadata
  * (Optional) Use MIK to create Islandora import packages from data harvested via OAI-PMH

## MIK in a nutshell

* maybe a new version of the overview diagram?
* toolchains
  * fetchers, metadata parsers, filegetters, writers
  * manipulators

## Some use cases

* Migrating from another repository
  * MIK has been used to migrate from [CONTENTdm](http://www.oclc.org/en/contentdm.html), [Digital Commons](https://www.bepress.com/products/digital-commons/), and [Vital](https://www.iii.com/products/digital-asset-management/).
* Preparing content for ingestion into Islandora
  * MIK can read a CSV metadata file and generate Islandora import packages for each object described in it. You can create packages for images/PDFs/videos, books, newspaper issues, and simple compound objects from CSV files.
* Automated ingestion workflows
  * MIK has been scripted to convert content saved in watch folders into Islandora import packages. Running MIK as a timed (e.g., cron) job on this content will allows you to automate content ingestion.

## Installing MIK

* PHP 5.5.0 (or higher) command-line interface (CLI)
* [Composer](https://getcomposer.org/)
  1. `curl -sS https://getcomposer.org/installer | php`
  2. `php composer.phar install`

## Configuring MIK

MIK uses .ini files to store configuration details. An MIK .ini file contains the following sections:

```
[SYSTEM]
[CONFIG]
[FETCHER]
[METADATA_PARSER]
[FILE_GETTER]
[WRITER]
[MANIPULATORS]
[LOGGING]
```

Each section contains configuration options.

Filesystem paths, such as the location of the input CSV file (for CSV toolchains), the temp directory, the metadata mappings file, and the ouput directory, differ across platforms. For example:

```
[FETCHER]
; On Linux
temp_directory = "/tmp/miktutorial_temp"
; On Mac
; temp_directory = "/Users/mark/miktutorial_temp"
; On Windows
; temp_directory = "c:\temp\miktutorial_temp"

[FILE_GETTER]
; On Linux
input_directory = "/home/mark/Downloads/mik_tutorial_data"
; On Mac
; input_directory = "/Users/mark/mik_tutorial_data"
; On Windows
; input_directory = "c:\temp\mik_tutorial_data"

[WRITER]
; On Linux
output_directory = "/tmp/miktutorial_output"
; On Mac
; output_directory = "/Users/mark/mik_tutorial_output"
; On Windows
; output_directory = "c:\temp\mik_tutorial_output"
```

### Metadata mappings

* Metadata Mappings Helper

### A closer look at toolchains

### Manipulators

* MIK's term for plugins.

### Post-write hooks

* they run as background processes

## Running MIK

* Use `--checkconfig`.

### Workflow

* configure, test (random set, specific set, etc.), reconfigure, retest.
  * start with metadata mappings
  * manipulators
* automating production workflows

## Hands-on activities

### Creating Islandora import packages from CSV data

* Outcomes:
  * Generate a set of PDF import packages from the CSV file provided in this workshop.
  * Use the SplitRepeatedValues, SpecificSet, and SimpleReplace manipulators

#### Background

The CSV input file:

```csv
Identifier,File,Title,Author,Date,Subjects
doc01,the_documentary.pdf,"One: The Documentary","Ji-Hu Maruška",2014,"Numbers;Documentaries"
doc02,case_study.pdf,"2: A Case Study","Iman Valenta",2012,"Case studies;Duology"
doc03,user_manual.pdf,"3: The User Manual","Hanne Darzi",2011,"Instructional manuals"
doc04,any_way.pdf,"4: Any Way You Want It","Arya Kovac",2014,"Quarterly studies;Journey (band)"
doc05,best_friend.pdf,"5: Everybody's Best Friend","Nuka Kratochvil",2001,"Dogs"
```

The mappings file:

```csv
Title,"<titleInfo><title>%value%</title></titleInfo>"
Author,"<name type=""personal""><namePart>%value%</namePart><role><roleTerm type=""text"">creator</roleTerm></role></name>"
Date,"<originInfo><dateIssued encoding=""w3cdtf"">%value%</dateIssued></originInfo>"
Subjects,"<subject><topic>%value%</topic></subject>"
Identifier,"<identifier type=""local"" displayLabel=""Local identifier"">%value%</identifier>"
null0,"<genre authority=""marcgt"">article</genre>"
null1,"<typeOfResource>text</typeOfResource>"
```

The .ini file:

```
; MIK configuration file used during the workshop.

[SYSTEM]

[CONFIG]
config_id = MIK workshop
last_updated_on = "2017-03-20"
last_update_by = "mj"

[FETCHER]
class = Csv
input_file = "data/metadata.csv"
temp_directory = "/tmp/mik_workshop_temp"
record_key = Identifier

[METADATA_PARSER]
class = mods\CsvToMods
mapping_csv_path = "data/mappings.csv"

[FILE_GETTER]
class = CsvSingleFile
input_directory = data
temp_directory = "/tmp/mik_workshop_temp"
file_name_field = File

[WRITER]
class = CsvSingleFile
preserve_content_filenames = true
output_directory = "/tmp/mik_workshop_output"
; postwritehooks[] = "/usr/bin/php extras/scripts/postwritehooks/validate_mods.php"
; During testing, it's a good idea to create only the MODS XML files.
; datastreams[] = "MODS"

[MANIPULATORS]
metadatamanipulators[] = "SplitRepeatedValues|Subjects|/subject/topic|;"

[LOGGING]
path_to_log = "/tmp/mik_workshop_output/mik.log"
path_to_manipulator_log = "/tmp/mik_workshop_output/manipulator.log"
```

#### Steps required to achieve the outcome

### Creating Islandora import packages from data harvested from another repository via OAI-PMH

* Outcome:
  * Find a small image collection in an Islandora repository that implements the OAI-PMH provider, and create import packages from objects in the collection.

#### Background

OAI-PMH is commonly used for harvesting metadata for aggregated searching or similar purposes. MIK can use a repository's OAI metadata to serve as the basis for creating Islandora import packages. MIK offers a couple of OAI-PMH toolchains. In this workshop, we'll be using one that fetches objects from an Islandora instance.

Why harvest content from one Islandora instance to load into another? There are some legitimate reasons to do this, but in this workshop, we do it for illustrative purposes only. Islandora has a robust [OAI-PMH provider](https://github.com/Islandora/islandora_oai), and since many Islandora instances implement it, it serves as a useful learning environment. Outside of this workshop, you might use MIK's OAI-PMH harvesting abilities to migrate from a Digital Commons or Vital repository, for example.

#### Steps required to achieve the outcome


```

; MIK configuration file for an OAI-PMH toolchain.

[CONFIG]
config_id = oaitest
last_updated_on = "2017-03-20"
last_update_by = "mj"

[SYSTEM]

[FETCHER]
class = Oaipmh
oai_endpoint = "http://digital.lib.sfu.ca/oai2"
set_spec = hiv_collection
temp_directory = "/tmp/oaitest_temp"

[METADATA_PARSER]
class = dc\OaiToDc

[FILE_GETTER]
class = OaipmhIslandoraObj
temp_directory = "/tmp/oaitest_temp"

[WRITER]
class = Oaipmh
output_directory = "/tmp/oaitest_output"

[MANIPULATORS]

[LOGGING]
path_to_log = "/tmp/oaitest_output/mik.log"
path_to_manipulator_log = "/tmp/oaitest_output/manipulator.log"
```

#### Steps required to achieve the outcome

## Extending MIK

* configuration over code
* manipulators
* post-write hooks
* new toolchains
