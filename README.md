# MIK workshop at Islandoracon 2017

Islandoracon 2017 Post-Conference Session "Move to Islandora Kit: For all your migration and batch loading preparation needs"

## Workshop overview

* Your instructors
  * Marcus Barnes, The Digital Scholarship Unit (DSU) at the University of Toronto Scarborough Library
  * Mark Jordan, Simon Fraser University Library
* Duration: 3 hours
  * First half hour: installin MIK
  * Next 2 hours: hands on exercises
  * Last half hour: discussion
* Outcomes
  * Use MIK to create Islandora import packages from CSV metadata
  * Use MIK to create Islandora import packages from data harvested via OAI-PMH

## MIK in a nutshell

MIK is an example of and "Extract, Transform, Load" application. However, it only extracts and transforms. It delegates the loading of content into Islandora to the existing batch modules. In other words, MIK prepares your content for loading into Islandora:

![MIK overview](https://www.dropbox.com/s/hvoi7vy67oq4mv2/MIK_overview_with_callout.png?dl=1)

MIK is designed to be flexible, configurable, and extensible. It does this by breaking down the parts of the "extract and trasform" process into components:

* fetchers
  * Get a list of items to process, with their associated metadata. Possible sources are CONTENTdm and OAI-PMH repositories, adn CSV files.
* metadata parsers
  * Take the metadata for each item and convert it to MODS or DC.
* filegetters
  * Find and copy (or download) the image, PDF, video, and other files to be imported into Islandora.
* writers
  * Assemble the metadata and files into Islandora import packages ready to use as input for Islandora Batch, Book Batch, Newspapers Batch, and Compound Batch.
  
MIK also has plugins (but called "manipulators") that modify how fetchers, metadata parsers, filegetters, and writers work.

Each set of content you process with MIK requires a configuration file (also known as an .ini file), which lists all of the options used by the components described here. A particular combination of components is known as a "toolchain". For example, there is a CONTENTdm Books toolchain, a CSV Single File toolchain, and several OAI-PMH toolchains.

## Some MIK use cases

* Migrating from another repository
  * MIK has been used to migrate from [CONTENTdm](http://www.oclc.org/en/contentdm.html), [Digital Commons](https://www.bepress.com/products/digital-commons/), [Vital](https://www.iii.com/products/digital-asset-management/), and custom repositories running on [relational databases](https://github.com/MarcusBarnes/mik/wiki/Migration-Guide:-Simon-Fraser-University-Library-Editorial-Cartoons-Collection).
* Preparing content for ingestion into Islandora
  * MIK can read a CSV metadata file and generate Islandora import packages for each object described in it. You can create packages for images/PDFs/videos, books, newspaper issues, and simple compound objects from CSV files.
* Automated ingestion workflows
  * MIK has been scripted to convert content saved in watch folders into Islandora import packages. Running MIK as a timed (e.g., cron) job on this content will allows you to automate content ingestion.

## MIK's documentation

* [wiki](https://github.com/MarcusBarnes/mik/wiki)
  * Links to documentation on all of the toolchains, and their components.
* [tutorial](https://github.com/MarcusBarnes/mik/wiki/Tutorial)
  * A self-paced tutorial that takes you through the process of generating Islandora import packages for a set of five photos.
* [cookbook](https://github.com/MarcusBarnes/mik/wiki/The-MIK-Cookbook)
  * A set of short "how to" recipes documenting how to accomplish specific tasks using MIK.

## Installing MIK

* PHP 5.5.0 (or higher) command-line interface (CLI)
* [Composer](https://getcomposer.org/)
  1. `curl -sS https://getcomposer.org/installer | php`
  2. `php composer.phar install`

## Configuring MIK

MIK uses .ini files to store configuration details. An MIK .ini file contains the following sections:

```
[SYSTEM]
; This section is used to define PHP configuration options required
; by MIK but not defined in the system's php.ini file. Not necessary
; on all systems.

[CONFIG]
; Contains information about the .ini file and toolchain.

[FETCHER]
; Contains configuration options for the fetcher, which gets a list
; of items to process.

[METADATA_PARSER]
; Contains configuration options for the metadata parser, which converts
; data in the input list to MODS.

[FILE_GETTER]
; Contains configuration options for the file getter, which retrieves the file
; (image, PDF, video, etc.) to include in the Islandora import package.

[WRITER]
; Contains configuration options for the writer, which writes the import packages
; to the output directory.

[MANIPULATORS]
; Contains entries for manipulators, which are MIK's plugins. Most manipulator
; entries contain parameters.

[LOGGING]
; Contains paths to the log files that MIK creates.
```

Each section contains configuration options.

Filesystem paths, such as the location of the input CSV file (for CSV toolchains), the temp directory, the metadata mappings file, and the ouput directory, differ across platforms. For example:

```
[FILE_GETTER]
; On Linux
; input_directory = "/home/mark/Downloads/mik_tutorial_data"
; On Mac
; input_directory = "/Users/mark/mik_tutorial_data"
; On Windows
; input_directory = "c:\temp\mik_tutorial_data"
```

### Metadata mappings

The CONTENTdm and CSV toolchains use a mapping file to define what input field or column names map to specific MODS elements. You can create them:

* by hand, in a text editor
* using the [Metadata Mappings Helper](https://github.com/MarcusBarnes/mik/wiki/Metadata-Mappings-Helper)


 ![MIK Metadata Mappings Helper screenshot](https://www.dropbox.com/s/ggd9n3076gcz3qr/mappings_helper_screen_2.png?dl=1)

### A closer look at toolchains

Internally, MIK breaks the task of converting the input data into Islandora import packages down into discrete subtasks as illustrated in the diagram below:

![MIK details](https://www.dropbox.com/s/ejh8cfwdecehivr/MIK_overview.png?dl=1)

* Fetchers query a data source to determine how many objects are to be imported, and perform some additional setup for the subsequent tasks.
* Fetcher manipulators filter out items from the entire set of data retrieved by the fetcher. For example, you may only want to fetch book objects from a CONTENTdm collection that also contains images.
* Metadata parsers get the metadata for an object and convert it into a format that Islandora can use, such as MODS XML.
* File getters retrieve the content files associated with an object to be imported.
* File getter manipulators provide a way to configure file getters to look in specific locations for files.
* Writers save the converted content to disk in a directory structure that can be used by the standard Islandora batch import modules. After a writer has written out its package, it can initiate one or more post-write hooks (described below) that perform actions on the content in the packages.
* File manipulators perform some processing on the files retrieved by file getters.
* Metadata manipulators can modify or supplement the metadata XML file generated by metadata parsers.

A unique combination of one fetcher, one metadata parser, one file getter, zero or more manipulators, and one writer is an MIK "toolchain." When you run MIK you assemble a set of tools into a toolchain. A toolchain, and the options for how its component tools work, is defined in a single configuration file. Currently, MIK offers a CSV toolchain, and toolchains for a number of CONTENTdm object types. MIK is designed to be extensible, so it is relatively easy to create new toolchains.

### Manipulators

Manipulators are MIK plugins that let you perform tasks at specific times in the MIK execution lifecycle, or to change how fetchers, file getters, and metadata parsers work. All the code for a manipulator is encapsulated in a single PHP class file. Manipulators are registered in the MIK configuration file in the `[MANIPULATORS]` section, and may take parameters. The signatures for manipulators identify the group they are in, followed by an equal sign, followed by the manipulator's parameters, which are delimited by the pipe symbol (`|`). The first parameter is always the name of the manipulator. For example, in the following example, the "NormalizeDate" manipulator is being registered, taking the parameters "Date", "dateIssued", and "m": 

```
[MANIPULATORS]
metadatamanipulators[] = "NormalizeDate|Date|dateIssued|m"
```
Each manipulator has its own wiki page, which explains its function and parameters. Below is an overview of the most popular manipulators.

### Post-write hooks

MIK can run scripts after it has written an import package to disk. These scripts are called "post-write hooks" and are enabled in the .ini file's `[WRITER]` section like this:

```
[WRITER]
...
postwritehooks[] = "/usr/bin/php extras/scripts/postwritehooks/validate_mods.php"
postwritehooks[] = "/usr/bin/php extras/scripts/postwritehooks/generate_fits.php"
postwritehooks[] = "/usr/bin/php extras/scripts/postwritehooks/object_timer.php"
```

The five scripts listed above [are included](https://github.com/MarcusBarnes/mik/tree/master/extras/scripts/postwritehooks) in the MIK Github repository as examples. The ones named 'sample' illustrate some basic ways of using post-write hooks. Three complete functional scripts, `extras/scripts/postwritehooks/validate_mods.php`,  `extras/scripts/postwritehooks/generate_fits.php`, and `extras/scripts/postwritehooks/object_timer.php` do useful things, as suggested by their names.

A good example of how a post-write hook script can be used is to produce [FITS](https://github.com/harvard-lts/fits) output for newspaper page objects. As soon as MIK finishes creating the package, the `generate_fits.php` script runs FITS against each of the child OBJ datastream files and writes out its output for each one to TECHMD.xml within the page folder. This file is then loaded by the Islandora Newspaper Batch module, ending up as the TECHMD datastream for each page object. Another very useful example is `validate_mods.php,` which validates each MODS.xml file produced by MIK and writes out the result of the validation to the MIK log file.

Post-write hook scripts run as background processes, which means that they do not need to finish running before MIK moves on to process the next object. This speeds up MIK considerably.

### Shutdown hooks

MIK can run scripts after it has completed processing all packages. These scripts are called "shutdown hooks" and are enabled in the .ini file's `[WRITER]` section like this:

```
[WRITER]
shutdownhooks[] = "/usr/bin/php extras/scripts/shutdownhooks/apply_xslt_with_saxon.php"
shutdownhooks[] = "/usr/bin/php extras/scripts/shutdownhooks/delete_temp_files.php"
```

These scripts differ from post-write hooks, which run after MIK writes the import package for each object, in that they run after all packages have been generated. They are useful for cleanup tasks, for example.

## MIK's log files

* mik.log
* problem_records.log
* input_validator.log
* manipulator.log

## Running MIK

* `php mik -c test.ini -cc all`
* `php mik -c test.ini -l 10`

### Workflow

* configure, test (random set, specific set, etc.), reconfigure, retest.
  * start with metadata mappings
  * manipulators
  * post-write hooks
* automating production workflows

## Hands-on activities

### Creating Islandora import packages from CSV data

#### Outcomes
  * Generate a set of PDF import packages from the CSV file provided in this workshop.
  * Use the [SplitRepeatedValues](https://github.com/MarcusBarnes/mik/wiki/Metadata-manipulator:-SplitRepeatedValues), [SpecificSet](https://github.com/MarcusBarnes/mik/wiki/Fetcher-manipulator:-SpecificSet), and [SimpleReplace](https://github.com/MarcusBarnes/mik/wiki/Metadata-manipulator:-SimpleReplace) manipulators

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
Title,<titleInfo><title>%value%</title></titleInfo>
Author,<name type='personal'><namePart>%value%</namePart><role><roleTerm type='text'>creator</roleTerm></role></name>
Date,<originInfo><dateIssued encoding='w3cdtf'>%value%</dateIssued></originInfo>
Subjects,<subject><topic>%value%</topic></subject>
Identifier,<identifier type='local' displayLabel='Local identifier'>%value%</identifier>
null0,<genre authority='marcgt'>article</genre>
null1,<typeOfResource>text</typeOfResource>
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

##### 1. Create your mappings file
##### 2. Create your .ini file
##### 3. Test
##### 4. When ready, generate your ingest packages


### Creating Islandora import packages from data harvested from another repository via OAI-PMH

#### Outcome
  * Generate a set of import packages from objects in an OAI-PMH repository.

#### Background

OAI-PMH is commonly used for harvesting metadata for aggregated searching or similar purposes. MIK can use a repository's OAI metadata to serve as the basis for creating Islandora import packages. MIK offers a couple of OAI-PMH toolchains. In this workshop, we'll be using one that fetches objects from an Islandora instance.

Why harvest content from one Islandora instance to load into another? There are some legitimate reasons to do this, but in this workshop, we do it for illustrative purposes only. Islandora has a robust [OAI-PMH provider](https://github.com/Islandora/islandora_oai), and since many Islandora instances implement it, it serves as a useful learning environment. Outside of this workshop, you might use MIK's OAI-PMH harvesting abilities to migrate from a Digital Commons or Vital repository, for example.

The .ini file:

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
datastream_ids[] = OBJ
datastream_ids[] = PDF

[MANIPULATORS]

[LOGGING]
path_to_log = "/tmp/oaitest_output/mik.log"
path_to_manipulator_log = "/tmp/oaitest_output/manipulator.log"
```

#### Steps required to achieve the outcome

##### 1. Create your .ini file
  * Find a small image collection in an Islandora repository that implements the OAI-PMH provider.
##### 2. Test
##### 3. When ready, generate your ingest packages

## Extending MIK

* configuration over code
* manipulators
* post-write hooks and shutdown hooks
* new toolchains

## License

<a rel="license" href="http://creativecommons.org/licenses/by/4.0/"><img alt="Creative Commons License" style="border-width:0" src="https://i.creativecommons.org/l/by/4.0/80x15.png" /></a><br />This workshop material is licensed under a <a rel="license" href="http://creativecommons.org/licenses/by/4.0/">Creative Commons Attribution 4.0 International License</a>.
