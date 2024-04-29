# Batch scripts to run Mendelian Randomisation and Logistic Regression analyses on BMI and HbA1c and infections in UKBiobank using imputed genotype variants
An Open access data analysis pipeline in UK Biobank, making the implementation of one-sampe instrumental variable Mendelian randomisation for HbA1c and BMI simpler. 

## structure of the repo:
As DNA Nexus was used, most .csv or graph files have been outputted to user-specific environments, and cannot be found here. This Repo contains:
* /Codes/ directory contains all used ICD10 codes to capture infections data
* /Scripts/ directory contains all scripts in .Rmd to run analyses. These scripts have been configured for open-access use
* /compiled_codelists.csv is an output of the compiled ICD10 codes for use.
* /Reports/ directory contains various submitted reports/works regarding the data and analyses extracted from this pipeline and research project
  * dissertation.txt is the final written up report of this research and pipeline, through which my dissertation was submitted.
  * SGGD2024_poster_deVilliers_Green.pptx is the poster presentation of the analysis presented at SGGD2024

## How to use this repository: /Scripts/

### R-Markdown vs R Scripts
.Rmd files were used as scripts commonly contained numerous sub sections separated by function. 
For example, scripts commonly had a package download/init, a data download/init, a descriptives section, and then an analysis section.
R Markdown provides better space and organisation of code, and therefore has been chosen for maintainability and comphresion.
Alternatively, R scripts would provide an easier copy/paste pipeline, and I do have personalised R scripts based on the final R Markdwon scripts that have not been included in this upload.
All code in an R script can directly be copied into UK BioBank to efficiently run the entire extraction, transformation, and analysis pipeline.
To convert any R-Markdown script into an R script, use the knitr::purl("filename.Rmd") function. Note that inline R expressions would be ignored, although I do not use any throughout the R Markdown scripts. 


### Info on Local vs BioBank deployment
There are two classifications of scripts in this repository. Scripts are either developed to be run locally, named "Local" scripts, or scripts are developed to run on the UK BioBank data platform, named "bb" scripts. These names are included as a prefix of the script filenames.

Local scripts are designed to create additional resources that are to be uploaded to UK BioBank prior to running any bb analysis pipeline.
Examples of such scripts and outputs are the local_codelists.Rmd script, which amalgamates Rhian Hopkins' infection codelists into a usable lookup table on bb. This compiled_codelist.csv file must be manually uploaded to UKbb, although a current version (as of 21/04/2024) currently exists.

bb scripts are designed to be run on the UK BioBank platform, and have some dependencies such as numerous GitHub helper functions developed by Harry Green, as well as Genetic Data for numerous SNPs.
These scripts are developed in .Rmd format, as to provide additional formatting, although it is recommended to have converted R scripts as well for ease of copy/pasting into UK bb.


### Downloading data / results
UK BioBank as a data paltform aims to centralise access and copies of the raw medical/genetic data. Therefore, it is required that only processed data with no observation-level information is extracted from the platform. This also includes any processed/analytical data on personal identifiable information (PII)

The generated ggplot graphs and reporting tables are saved as files during the session, commonly under a /saveables/ directory. This means to download generated graphs and tables, you can simply download the saveables directory which will automatically .zip all results.
It is recommended that any additional analysis, such as new or modified graphs or tables, are also saved under this /saveables/ directory, to ensure data security and ease of downloading results.

### Configurability - plug and play
A few of the scripts requires unique identifiers, or constants which are user-dependent.
These have been moved to the tops of every .Rmd script under a "GLOBAL VARS" chunk, and have identifiers in ALL_CAPS.
It is recommended before running a script to look through the GLOBAL VARS as you will likely need to input your variables.
The GLOBAL VARS chunk also contains detailed documentation on where/how to obtain the required information, and what it is used for.

### Purpose / function of scripts
Outlined above is guidance on how to use and work with the scripts found in this directory. This section describes the structure and function of each of the scripts found in this /scripts/ directory:

* local_codelists.Rmd
  - Part of the extraction process preceeding data analysis
  - Dependency: Rhian Hopkins' ICD_10 bactieral/fungal/viral codelists: /Codes/*.txt
  - GLOBAL VARS: NO vars
  - Outputs: /compiled_codelist.csv 
* bb_cohort.Rmd
  - The script that uses an uploaded version of compiled_codelist.csv on UKbb in order to create a large infections cohort dataframe
  - Builds off of Harry Green's baseline cohort, adding 1) infections; 2) GRS; 3) GCK data.
  - Dependency: Harry Green's UKbb script: https://github.com/hdg204/UKBB, includes baseline cohort
  - Dependency: UKBB-hosted compiled_codelist.csv from local_codelists.Rmd
  - Dependency: Harry Green's various GRS data. Must be requested personally.
  - GLOBAL VARS: UKbb file-id for compiled_codelist.csv, 4x UKbb GRS data file-id
  - Outputs: /infections_df_{date}.csv
    > date is generated upon creation, using system's date (Cloud EC2 instance, eu-west-2 = GMT)
    > uploaded directly to UKbb user's root directory
    > contains various clinical features recorded at study recruitment (baseline): waist-hip-ratio, BMI, age, diabetes (y/n), sex, etc.
    > contains infections data: binary history of bacterial, fungal or viral infections (y = 1)
* bb_init_analysis.Rmd
  - This short script contains all of the common intialising code that loads the necessary libraries and creates required folder structure for running the analysis scripts on UKbb.
  - Dependency: NO dependencies
  - GLOBAL VARS: NO vars
  - Outputs: NO outputs
* bb_hba1c_analysis.Rmd
  - This is the main long analytical script for testing causal association of Glycaemia, via observational and genetically predicted HbA1c, on infections outcomes.
  - Contains basic graphs to quickly visualise results, but is primarily used to output the reporting data of the various regression models
  - Dependency: UKbb-hosted infections_df_{datexe}.csv 
  - Dependency: Requires prior initialisation using bb_init_analysis.Rmd 
  - GLOBAL VARS: Configuration for a descriptive reporting table, and required UKbb infections_df_{date}.csv file-id
  - Output: infections cohort descriptive table, numerous simple regression odds ratio graphs, numerous regression model value-reporting tables
* bb_bmi_analysis.Rmd
  - Contains complete analysis and basic graph to analyse BMI as causal risk factor for incresed infections risk. Does not produce most sophisticated graphs
  - Dependency: UKbb-hosted infections_df_{datexe}.csv 
  - Dependency: Requires prior initialisation using bb_init_analysis.Rmd 
  - Dependency: UKbb-hosted 73-SNP BMI data
  - GLOBAL VARS: Required paths to UKbb infections_df_{datexe}.csv and 73-SNP BMI GRS data
  - Output: descriptives reporting table, density plot of BMI stratified by infection types, observational and MR logistic regression value-reporting tables and forest plots.
* bb_main_graphs.Rmd
  - Contains code to create the main graphs and findings of the paper. 
  - Dependency: Requires all subsequent .csv exports of value-reporting HbA1c and BMI tables
  - Output: MAIN graphs for findings. Multiple versions based on background/colours/style are available.
  
  
