# SAS_Learning
To be Include a word Search program

/* Import data - only a CSV with Feedback_ID and Feedback_Text.
In the CSV, Replace all commas with . using CTRL + R option.*/
filename word_ "//1bpaudit/1bpaudit/Karan/Consumer Segments/word_search.csv"; /*where the csv is kept*/
libname art  "//1bpaudit/1bpaudit/Karan/Consumer Segments"; /*where the SAS data will be stored*/
data art.word_search;
	infile word_ dlm=',' dsd firstobs=2;
	format feedback_id 8.
		feedback_text $32000.;
	informat feedback_id 8.
		feedback_text $32000.;
	input feedback_id feedback_text $;
run;

/* Put search strings into macro variables */
%let word_one =remediation;
%let word_two = customer remediation;

/* Begin Search */
proc sql;
	create table work.word_found as
		select feedback_id, feedback_text,
			case 
				when upcase(feedback_text) like "%nrbquote(%str(%%)%upcase(%superq(word_one))%str(%%))" or 
				indexw(upcase(feedback_text),"%upcase(%superq(word_one))") > 0 then "%superq(word_one) found"
				when upcase(feedback_text) like "%nrbquote(%str(%%)%upcase(%superq(word_two))%str(%%))"
				or
				indexw(upcase(feedback_text),"%upcase(%superq(word_two))") > 0 then "%superq(word_two) found"
				else 'nothing found' 
			end 
		as String_Search format=$20.
			from art.word_search;
quit;

proc freq data=work.word_found;
	tables String_Search;
run;

/* Output needed data */
%macro output_data;

	data work.needed;
		set work.word_found(
			where=( String_Search ^= 'nothing found'));
	run;

	%let dsid = %sysfunc(open(work.needed));
	%let nobs = %sysfunc(attrn(&dsid,nlobs));
	%let dsid = %sysfunc(close(&dsid));
	%put &=nobs;

	%if &nobs = 0 %then
		%put No Results Found;
%mend;

%output_data
