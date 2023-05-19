# Comparison tool


## Overview
 
This repo serves two purposes:
1. It is an example implementation of the comparison tool, using wagtail.
2. It is a demonstration of how the tool can be used for humanities data. 

It will be used in the [Our Heritage, Our Stories project](https://www.ohos.org.uk/), as one of the remixer tools.

This tool is designed to allow users to compare metadata about records, and rank them based on their answers to questions. The data here are gathered from [The National Archives Catalogue, Discovery](https://discovery.nationalarchives.gov.uk/).

There is 1 example pages:
- Discovery data - `/comparison/`

This repository is an example wagtail implementation of the tool only. It is designed for investigation and testing purposes only, and is not intended for use in the final Remixer Suite.

## Running the tool
This example case can be run following the standard quickstart instructions for wagtail. These can be found [here](https://docs.wagtail.io/en/stable/getting_started/tutorial.html). If you already have wagtail installed, you can run the tool by: `python manage.py runserver` and then navigating to the appropriate page.

## Tool origin

The original version of the tool here can be found [here, at the FSC identikit github repo](https://github.com/FieldStudiesCouncil/tombiovis). The version used in this repo is a fork of the original, and can be found [here](https://github.com/AndrewBewseyTNA/ohos-ranked-likelihood-search). This fork aims to test modifications to the UI to establish the usefulness of this tool for the OHOS project, and searching and comparing humanities records and data. 

## Technical details

The static content for the comparison tool is in the [home/static/comp-tool](home/static/comp-tool) directory. This directory contains the data for the tool, along with the javascript and css files. The data are in the "kb" directory, with each subdirectory being an individual knowledge base. The other files in [home/static/comp-tool](home/static/comp-tool) are the javascript and css files for the tool, and all are required. 

The html pages for the tool are found in the [search/templates/search](search/templates/search) directory. These html files are modifications of the original files from the comparison tool, and are required for the tool to work. An example file can be found [here](search/templates/search/comp-disco.html) The only changes made to these files are the addition of the `{% load staticfiles %}` tag at the top of the file, and the addition of the `{% static %}` tag to the javascript and css file paths.

This repository servers to demonstrate one possible way to implement the tool in Wagtail. There are likely other ways to do so, which may be more appropriate for other projects.

## Knowledge bases and Data

### General information

This tool gathers data from 5 csv files. An example of these 5 files are in the [home/static/comp-tool/kb/disco](home/static/comp-tool/kb/disco) directory. The files are:
- taxa.csv
- characters.csv
- media.csv
- values.csv
- config.csv

Of these, the two most important are taxa.csv and characters.csv. These files contain the data that is displayed to the user. The other files are used to provide additional information to or about the tool, and are not directly displayed to the user. 

Taxa.csv provides details about each record. Each row represents a record and each column representing a question visible in the UI. The 'Taxon' column represents the name of the record. 
Characters.csv provides details about each question, such as help text and latitude provided to answers. By default, the tool will determine possible answers to questions from data provided in taxa.csv, but this can be overridden by providing a list of possible answers in values.csv.

There are different formats of data, such as 'ordinal cyclical', and the use of '|' to represent 'or'; there are examples of these in the example csv files. 

Further information can be found in the [original documentation](https://github.com/FieldStudiesCouncil/tombiovis/blob/master/identikit/documentation/Building%20a%20knowledge-base.pdf)

### Data

The data are available in the [home/static/comp-tool/kb](home/static/comp-tool/kb) directory. 

### Data sources

The data have been made avaiable from The National Archives Discovery - data from [The National Archives Catalogue, Discovery](https://discovery.nationalarchives.gov.uk/) Web API. This data was checked using the Web API Sandbox from The National Archives (available [here](https://discovery.nationalarchives.gov.uk/API/sandbox/index)) and the Discovery UI, looking for other archives that may contain Community Generated Digital Content (e.g. using keywords such as “community” or “local”).


### Data structure

The discovery data are structured as follows:
- Title: Title of the record, text
- Level: At what archival hierarchical level is the entity, text
- Taxon: Required, provides the name of the record as displayed in the UI, text	
- Document_type: the type of document (e.g. diary, account book)
- Size: Size of the entity (number of records), numeric
- Creator: Agent who created the record, text
- Date_of_creation: Date the record was created, numeric
- Place: Place, text
- Access: Access rights, text	
- Copyright_holder: Agent controlling the copyright, text
- Repository: Repository holding the record, text	
- record_format: format of the record, text

### Creating a new knowledge base

1. Create a new directory in [home/static/comp-tool/kb](home/static/comp-tool/kb/). The name of the directory will be the name of the knowledge base.
2. Copy the contents of an existing kb directory into the new directory.
3. As a minimum, replace the taxa.csv with the new data. 
   - If required, update the other csv files.
4. In search/templates/search, copy an existing comparison tool html file, and rename it to the name of the new knowledge base.
5. Update this new html file to point to the new knowledge base. `tombiokbpath: kb ? kb : "{% static 'comp-tool/kb/disco/' %}",` needs to be updated to point to the new knowledge base.
6. Update the compTool/search/views.py with a new return for the new template. An example is included below. 
`def compDisco(request):
    return TemplateResponse(
        request,
        "search/comp-disco.html",
    )`
7. Update the compTool/compTool/urls.py with a new url for the new template. An example is included below. You need to modifiy search_views.{function name} to match the function name in the views.py file. `path("comparison/", search_views.compDisco, name="comp-disco")`

## Known issues

The main known issue is that the tool cannot be easily dockerised. If you use the default Dockerfile, the build crashes. One of the stages of the build parses .css files, and looks for files mentioned in comments. If it cannot find it, it crashes. One of the required packages for the comparison tool uses css with such comments. If you remove the comment from the css, the tool does not work. If you modify the step in the Dockerfile, the build works, but the UI does not work. 
The tool also cannot be easily being dockerised in Flask, when using Jinja templating. In this case, the dockerisation seems to miss the required js files, and the UI does not work. 

## Known limitations

- The tool becomes cumbersome to use with large numbers of records due to the UI. User testing will establish a limit of records preferred by users, as comparing more records may be prefered over ease of use.
- There are reserved characters in the data; these are used to provide information such as "or" in datasets, or indicate that data are not missing. This can cause issues, for example when a record has a `?` in the title. The current solution is to remove these characters from the data.

## Licence information

- The original tool ([FSC Identkit](https://github.com/FieldStudiesCouncil/tombiovis)) is licensed under the [GNU General Public License v3.0](https://github.com/FieldStudiesCouncil/tombiovis/blob/master/LICENSE).
- [Wagtail](https://github.com/wagtail/wagtail) is licensed under the [BSD 3-Clause License](https://github.com/wagtail/wagtail/blob/main/LICENSE).
- Several files in this repository are Crown Copyright, and contain licence information at the top of the file.
