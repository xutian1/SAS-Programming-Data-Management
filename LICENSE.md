ods rtf file='C:\Project1GDP.rtf' BODYTITLE;
title 'Data Management - GDP Project';

libname UN 'C:\GDP_Project';

/* First Data Set: GDP */
/* Importing GDP Data*/
Proc import file='C:\GDP_Project\GDPcurrent-NCU-countries.xls' out=data1 dbms=xls replace;
	namerow=3;
	startrow=4;
  	getnames=yes;
  	Range="Download-GDPcurrent-NCU-countri$A3:AV3716";
run;

/* Renaming Variables D-AV*/
data UN.GDP (drop=d--av);
	set data1;
  	array oldname{45} d e f g h i j k l m n o p q r s t u v w x y z 
  	aa ab ac ad ae af ag ah ai aj ak al am an ao ap aq ar as at au av;
  	array newname{45} Y1970-Y2014;
  	do i=1 to 45;
    	newname{i}=oldname(i);
  	end;
  	format Y1970-Y2014 comma22.0;
run;

proc contents data=UN.GDP;
run;
/* Variables Country, Currency and IndicatorName are Character */
/* Variables Y1970-Y2014 are Numeric */

/* Checking if missing data in Character-type Variables*/
proc print data=UN.GDP;
	var Country  Currency  IndicatorName;
	where Country=" " or Currency=" " or IndicatorName=" ";
run;
/* No observations were selected*/

/* Checking number of distinct values for Character-type Variables */
proc freq data=UN.GDP NLEVELS;
	tables Country  Currency  IndicatorName;
run;
/* 220 Levels for Country, 153 Levels for Currency, 17 levels for IndicatorName */

/* Correcting unreadable country names*/
data UN.GDP2;
	set UN.GDP;
	if find(Country, "d'Ivoire") ge 1 then Country="Cote d'Ivoire";
	if Country =: 'Cura' then Country='Curacao';
run;

/* Checking number of distinct values for Character-type Variables */
proc means data=UN.GDP2 n nmiss min max;
	var Y1970-Y2014;
run;

/* Checking where negative figures exist. Using Y1970 as an example */
proc print data=UN.GDP2;
	var Country  Currency  IndicatorName Y1970;
	where Y1970<0 and Y1970 NE . ;
run;



/* Second Data Set: Exchange Rate */
/* Import Exchange Rate Data*/
data UN.EXRates;
	infile "C:\440\Group12\EXRates.xls" dlm='09'x firstobs=2;
	input Country :$34. Currency :$29. abbrv :$5. Date :date11. Rate :8.;
	format date date11.;
	label abbrv = '	Currency Code'
		Rate = 'Operational Rate'
		Date = 'Effective Date';
run;

proc contents data=UN.EXRates;
run;
/* 220 Observations */

/* Checking number of distinct values for Character-type Variables */
proc freq data=UN.EXRates NLEVELS;
	tables Country  Currency abbrv;
run;
/* 220 Levels for Country 151 Levels for Currency, 152 Levels for abbrv */

/* Comparing Currency in GDP Data and in Exchange Rates Data */
proc sql NUMBER;
	select DISTINCT a.Currency 'Curr from GDP' 
	from UN.GDP2 a LEFT JOIN UN.EXRates b on a.Currency=b.Currency
	where b.Currency is missing ;
quit;
/* 91 rows */

/* Comparing Country in GDP Data and in Exchange Rates Data */
proc sql NUMBER;
	select DISTINCT a.Country  
	from UN.GDP2 a LEFT JOIN UN.EXRates b on a.Country=b.Country
	where b.Country is missing ;
quit;
/* 34 rows */

/* Cleaning Data in EXRates by Aligning Country Names */
data UN.EXRates_New;
	set UN.EXRates;
	if Country=:'Bolivia' then Country='Bolivia (Plurinational State of)';
	if Country=:'Brunei' then Country='Brunei Darussalam';
	if Country=:'Virgin Islands (UK)' then Country='British Virgin Islands';
	if Country=:'Hong Kong' then Country='China, Hong Kong SAR';
	if Country=:'Macao' then Country='China, Macao SAR';
	if Country=:'D.P.R  of Korea' then Country='D.P.R. of Korea';
	if Country=:'Congo, Dem. Rep.' then Country='D.R. of the Congo';
	if Country=:'Iran' then Country='Iran (Islamic Republic of)';
	if find(Country, 'Lao') ge 1  then Country="Lao People's DR";
	if Country=:'Libyan Arab Jamahiriya' then Country='Libya';
	if Country=:'Micronesia' then Country='Micronesia (FS of)';
	if Country=:'Palau, Republic of' then Country='Palau';
	if Country=:'St. Vincent and the Grena' then Country='St. Vincent and the Grenadines';
	if Country=:'Tanzania, United Rep. of' then Country='U.R. of Tanzania: Mainland';
	if Country=:'Timor-Leste' then Country='East Timor';
	if Country=:'Turks and Caicos Island' then Country='Turks and Caicos Islands';
	if Country=:'Tanzania, United Rep. of' then Country='U.R. of Tanzania: Mainland';
	if Country=:'United States' then Country='United States of America';
	if Country=:'Venezuela' then Country='Venezuela (Bolivarian Republic of)';
	if COuntry=:'Republic of Yemen' then Country='Yemen';
run;
	
/* Merging the Two Data Sets */
proc sql;
	create table UN.Merge as 
	select *
	from UN.GDP2 a left join UN.EXRates_New b on a.Country=b.Country
	;
	
/* Finding where Rate is missing*/
proc sql;
	select distinct Country, Currency
	from UN.Merge
	where Rate is missing
	;
quit;

/* Adding Exchange Rates for several countries*/
data UN.Merge_New (drop=abbrv date);
	set UN.Merge;
	if Country in ('Curacao','Sint Maarten (Dutch part)') then Rate=1.79;/*Currency Netherlands Antilles Guilder*/
	if Country='Ethiopia (Former)' then Rate=22.0499; /*Currency Birr*/
	if Country='Sudan (Former)' then Rate=6.446;/*Currency Sudanese Pound*/
	if Country in ('State of Palestine', 'Timor-Leste','United States', 'Zimbabwe' ) then Rate=1;/*Currency USD*/
	if Country='Zanzibar' then Rate=2187; /*Currency Tanzanian Shilling*/
	label Rate='Exchange Rate on 01 Aug 2016';
run;
/* Final Merged Data Set using ExRates on Aug1 2016: UN.Merge_New */

/* Converting to USD*/
data UN.GDPDollar;
	set UN.Merge_New;
    array year{*} Y1970-Y2014;
    do i=1 to 45;
      year{i}=year{i}/rate;
      end;
    format Y1970-Y2014  dollar25.2;
    drop i Currency Rate;
run;

proc contents data=UN.GDPDollar;
run;


/* Subset: GDP Only from 2009-2014 */
/* Creating a subset which contains only the GDP indicator for the last ten years*/
data UN.GDP_Only;
	set UN.GDPDollar;
	where IndicatorName='Gross Domestic Product (GDP)' and Y2013 is not missing and Y2014 is not missing;
	drop Y1970-Y2008 IndicatorName;
run;

proc contents data=UN.GDP_Only;
run;
/* 210 Observations */

/* Poorest Countries in 2014 */
proc sort data=UN.GDP_Only out=sort14;
	by Y2014;
run;

title3 'Poorest Countries in 2014';
proc print data=sort14(obs=10);
	var Country Y2014;
run;

/* Richest Countries in 2014 */
proc sort data=UN.GDP_Only out=sort14_d;
	by descending Y2014 ;
run;

title3 'Richest Countries in 2014';
proc print data=sort14_d(obs=10);
	var Country Y2014;
run;

/* Calculate Average Annual Growth Rates */
data UN.GDP_AAGR;
	set UN.GDP_Only;
    AGR1=(Y2010-Y2009)/Y2009;
    AGR2=(Y2011-Y2010)/Y2010;
    AGR3=(Y2012-Y2011)/Y2011;
  	AGR4=(Y2013-Y2012)/Y2012;
  	AGR5=(Y2014-Y2013)/Y2013;
 	AAGR=mean(AGR1, AGR2, AGR3, AGR4,AGR5);
 	format AGR1-AGR5 percent10.2 AAGR percent10.2;
 	label AGR1='Growth Rate from 2009 to 2010'
 	AGR2='Growth Rate from 2010 to 2011'
 	AGR3='Growth Rate from 2011 to 2012'
 	AGR4='Growth Rate from 2012 to 2013'
 	AGR5='Growth Rate from 2013 to 2014'
 	AAGR='Average Annual Growth Rate';
 	drop Y2009-Y2013;
run;

/* Lowest Growth Rates */
proc sort data=UN.GDP_AAGR out=growth;
	by AAGR;
run;

title3 'Countries with Lowest GDP Growth Rates';
proc print data=growth (obs=10) label;
	var Country AAGR AGR1-AGR5;
run;

/* Fastest Growth Rates */
proc sort data=UN.GDP_AAGR out=growth_d;
	by descending AAGR;
run;

title3 'Countries with Fastest GDP Growth Rates';
proc print data=growth_d (obs=10) label;
	var Country AAGR AGR1-AGR5;
run;

/* Predicting US GDP in 2020 based on AAGR */
data UN.GDP_Pred;
	set UN.GDP_AAGR;
	do i=2015 to 2020;
	PRED=Y2014+Y2014*AAGR;
	end;
	drop AGR1-AGR5 i;
	where Country ='United States';
	format PRED dollar30.2;
run;

proc print data=UN.GDP_Pred label;
	title3 'Predicting US GDP in 2020 based on AAGR';
run;

title;
ods rtf close;


    








