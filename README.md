# Running the Beiwe-Analysis code locally

This documentation will demonstrate a guideline of how to run the Beiwe-Analysis code locally and will also provide introduction for each function.

## Table of Contents
- Language and Environment Setup
- Introduction to the functions involved in Beiwe-Analysis
    - Preprocessing
    - Processing
    - Outputs
    - Utility 

## Language, Packages, and Environment Setup

Some of the Beiwe-Analysis code references **`Python`** or **`C`**-related code for speed, but most of this code is in **`R`**, so we suggest running the code in the **`Rstudio`**. If you already have **`Rstudio`** installed, we suggest updating it to the latest version.

The following steps instruct how to install necessary packages including **Rcpp** that facilitate interfacing **`C`** code in R Packages. 

- **Rcpp requires gcc and gfortran installed**
    - **For macOS users**:
         Open the terminal console, and check if you already installed Xcode Command Line Tools by typing:
            
	    ```
             $ xcode-select -p
            ```
         
         If you see the following returns then the full Xcode package is already installed
            ```
             /Applications/Xcode.app/Contents/Developer
            ```
         
         If not, enter the following to install          
            ```
             $xcode-select --install
            ```
         
         The following can be used to verify **`gcc`** is installed:            
            ```
             $gcc --version
            ```
        The next step is to install **`gfortran`**. Follow the steps in [How to install gfortran on Mac OS X](http://skipperkongen.dk/2012/04/27/how-to-install-gfortran-on-mac-os-x/).
    
    - **For windows users**, ...

- **Install/Load Required R Packages and Set Directories in Rstudio**


    - Open **Rstudio** and run the following code to install/load all the R packages needed for the Beiwe-Analysis:
        ```
        list.of.packages = c(
          "Rcpp",
          "RcppArmadillo",
          "mvtnorm",
          "Matrix",
          "MCMCpack",
          "VGAM",
          "stringr",
          "plyr",
          "pryr",
          "dplyr",
          "tidyr",
          "purrr",
          "tibble",
          "lme4",
          'lmerTest',
          'glmmLasso'
        )

        new.packages <- list.of.packages[!(list.of.packages %in% installed.packages()[,"Package"])]
        if(length(new.packages)) install.packages(new.packages)
        lapply(list.of.packages, require, character.only = TRUE)


        detach("package:plyr", unload=TRUE)

        ```

## Introduction to the functions involved in Beiwe-Analysis
### Preprocessing

surveys_preprocessing(patient_name)
surveys_preprocessing
function(patient_name, ...){
  survey_data = list()
  surveys_data_filepath = paste(data_filepath, patient_name, "survey_answers",sep="/")
  if(file.exists(surveys_data_filepath)){
    survey_names = list.files(surveys_data_filepath)
    for(survey_name in survey_names){
      surveys_filepath = paste(surveys_data_filepath,survey_name,sep="/")
      specific_surveys = list.files(surveys_filepath)
      for(specific_survey in specific_surveys){
        specific_survey_filepath = paste(surveys_filepath,specific_survey,sep="/")
        specific_survey_data=read.csv2(specific_survey_filepath,header=T,sep=",")
        if(nrow(specific_survey_data)>0){
          date = as.POSIXct(gsub("_",":",gsub(".csv","",specific_survey)),origin="1970-01-01")
          timestamp = as.numeric(date)
          specific_survey_data[,"survey_id"]      = survey_name
          specific_survey_data["date"]            = date
          specific_survey_data[,"patient_name"]   = patient_name
          specific_survey_data[,"timestamp"]      = timestamp
          specific_survey_data[,"date"]           = as.Date(date)
          survey_data[[specific_survey_filepath]] = specific_survey_data
        }
      }
    }
  survey_data = do.call(rbind, survey_data)
  rownames(survey_data) = seq(nrow(survey_data))
  saveRDS(survey_data, paste(output_filepath, "/Preprocessed_Data/Individual/",patient_name, "/survey_data.rds",sep=""))
  }
}

survey preprocessing:
in this preprocessing, we call each patient name by loop and read survey data  from “specific survey filepath”. If there exists information in the survey file, we read the information by such order that: survey name, date, patient name, timestamp, etc. We store this information into several lists and that save then as RDS file.



> text_preprocessing
function(patient_name, ...){
  textmat = NULL
  texts_filename = paste(data_filepath,patient_name,"texts",sep="/")
  if(file.exists(texts_filename)){
    text_files = list.files(texts_filename)
    if(length(text_files) > 0){
		textmat = c()
		for(text_file in text_files)
			textmat = rbind(textmat, data = read.csv(paste(texts_filename,"/",text_file,sep=""),header=T))
		textmat[,"timestamp"] = textmat[,"timestamp"] / 1000
		textmat = textmat[,-2]
		textmat[,c("hours","days")] = hours(textmat[,"timestamp"])
    }
  }
  saveRDS(textmat, paste(output_filepath, "/Preprocessed_Data/Individual/",patient_name, "/text_data.rds",sep=""))
}
<bytecode: 0x10ffc5518>

calling the text preprocessing function, we create textmat by adding information from texts file. To be specific, we first convert textmat by dividing it by 1000. Then we created the variable the variables "hours","days" by splitting the variable  "timestamp”. The reason to do this is since timestamp combine the information of hours and days and we want to extract them from this variable. 

> calls_preprocessing
function(patient_name, ...){
  callmat = NULL
  calls_filename = paste(data_filepath,patient_name,"calls",sep="/")
  if(file.exists(calls_filename)){
    call_files = list.files(calls_filename)
    if(length(call_files) > 0){
		callmat = c()
		for(call_file in call_files)
			callmat = rbind(callmat, data = read.csv(paste(calls_filename,"/",call_file,sep=""),header=T))
		callmat[,"timestamp"] = callmat[,"timestamp"] / 1000
		callmat = callmat[,-2]
		callmat[,c("hours","days")] = hours(callmat[,"timestamp"])
    }
  }
  saveRDS(callmat, paste(output_filepath, "/Preprocessed_Data/Individual/",patient_name, "/call_data.rds",sep=""))
}

to call the calls preprocessing, we read files from our working directory and basically did the same thing as the text preprocessing. We also want to combine information from each file and split the variable "timestamp” into “hours” and “days” and convert them information and store them in the format of call.

> powerstate_preprocessing
function(patient_name, ...){
	power_state_filepath = paste(data_filepath, "/", patient_name, "/power_state",sep="")
	power_state_filename = paste(output_filepath, "/Preprocessed_Data/Individual/",patient_name, "/powerstate_data.rds",sep="")
	if(!file.exists(power_state_filename)){
	  statemat = list()
	  state_files = list.files(power_state_filepath)
	  if(length(state_files)>0){
		  for(state_file in state_files){
			mat = read.csv(paste(power_state_filepath, "/",state_file,sep=""),header=T)
			if(ncol(mat)==3) mat[,"level"] = NA
			if(nrow(mat)>0)
			  statemat[[state_file]] = mat
		  }
		  statemat = do.call(rbind, statemat)
		  statemat[,1] = statemat[,1] / 1000
		  statemat = statemat[,-2]
		  statemat[,c("hours","days")] = hours(statemat[,"timestamp"])
		  saveRDS(statemat, power_state_filename)
		}
	}
}

to call the powerstate preprocessing, we read files from our working directory and basically did the same thing as the text preprocessing. We also want to combine information from each file and split the variable "timestamp” into “hours” and “days” and convert them information and store them in the format of statemat and store them in RDS file.

> accelerometer_preprocessing
function(patient_name, minutes, verbose = TRUE, ...){
if(length(list.files(paste(data_filepath, patient_name, "accelerometer", sep="/")))==0){
  cat("No files to preprocess.")}else{
	  if(verbose) cat("Preprocessing accelerometer data\n")
	  if(verbose) cat(paste("Aggregating by",minutes,"minutes\n"))
	  code_location = paste(source_filepath,"Preprocessing", "accelerometer_preprocessing.py",sep="/")
	  patient_data_filepath = paste(output_filepath, "/Preprocessed_Data/Individual/", patient_name, sep="")
	  patient_data_filename_TXT = paste(patient_data_filepath, "/appended_sheared_file_acc_",minutes,".txt",sep="")
	  patient_data_filename_RDS = paste(patient_data_filepath, "/appended_sheared_file_acc_",minutes,".rds", sep="")
	  
	  if(file.exists(patient_data_filename_RDS)){
		if(verbose) cat("Already preprocessed.\n")
	  }else{
		if(verbose) cat("Preprocessing...\n")
		system(paste("python", code_location, data_filepath, output_filepath, patient_name, minutes, sep=" "))
		accmat = read.table(patient_data_filename_TXT, header = TRUE)
		accmat[,1] = accmat[,1] / 1000
		accmat = accmat[,-2]
		accmat[,c("hours","days")] = hours(accmat[,"timestamp"])
		saveRDS(accmat, patient_data_filename_RDS)
		file.remove(patient_data_filename_TXT)
	  }
  }
}

to call the accelerometer_preprocessing, we read files from our working directory and basically did the same thing as the text preprocessing. We also want to combine information from each file and split the variable "timestamp” into “hours” and “days” and convert them information and store them in the format of accmat and store them in RDS file.


