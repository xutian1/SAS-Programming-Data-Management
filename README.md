# SAS-Programming-Data-Management
Part One: Introduction 

I.	Research Interest

The project aims to make gross domestic product (GDP) and its breakdown for country members of the United Nations internationally comparable at current prices in nominal US dollars. Based on the information, we were interested in identifying the poorest and richest countries in 2014 by their GDP and countries with fastest and lowest growth. The final objective was to predict GDP in 2020 for the United States based on its average GDP growth in recent years.

II.	Description of the Data Sources

Two data sets were selected for this project. Both are publically accessible at the UN website. 

Dataset #1 GDP is accessible through the following link: http://unstats.un.org/unsd/snaama/downloads/Download-GDPcurrent-NCU-countries.xls 
The data set contains information on GDP and its breakdown at current prices in nominal National currency for all countries compiled by the United Nations. The raw data are in EXCEL format.

Dataset #2 ExRates is accessible through the following link: https://treasury.un.org/operationalrates/OperationalRates.php
The data set contains information of operational exchange rates for one United States Dollar (USD) for all UN countries as the day of August 1, 2016. Even though the website claims the raw data should be downloaded in EXCEL format, the downloaded file was in tab-delimited Text format. 

Part Two: Methods for Managing the Data

I.	Reading Data into SAS
Data preparation involved in this project included reading data into SAS by using IMPORT and INFILE procedures, renaming variables, defining variables’ length and format, and adding necessary labels. 
II.	Data Validation
Data were checking for errors through the PRINT, FREQ, and MEANS procedures. Data errors were cleaned by using IF-THEN/ELSE statements in the DATA step.
III.	Merging Data
In the project, the two data sets could be merged based on the variable Country or Currency variable. Before merging the two data sets, the SQL procedure was used to compare the two data sets to see which variable could be a more efficient choice. The final decision was to merge the two files based on Country by using the SORT procedure and MERGE statement.
IV.	Creating New Variables
With the merged data, new variables were created to convert GDP and its breakdown from its national currency to nominal US dollars.  
V.	Sub-setting Data
GDP only observations from the year of 2009-2014 were selected by using the WHERE statement to create a subset for future analysis. 
VI.	Sorting Data
The subset were then sorted by data in 2014 in ascending or descending order and a list of first ten observations were printed as the poorest or richest countries.
VII.	Predicting 
Annual growth rates from 2009-2014 were calculated and the average annual growth rates (AAGR) were obtained. The data were sorted and countries with the fasted and lowest AAGR were printed. GDP over the next five years were also estimated based on AAGR by using the DO loop.
