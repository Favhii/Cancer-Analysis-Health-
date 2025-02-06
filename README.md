## NHS CANCER DASBOARD

## TABLE OF CONTENTS
- [PROJECT OVERVIEW](#project-overview)
- [DATA SOURCE](#data-source)
- [TOOLS](tools)
- [DATA CLEANING](data-cleaning)
- [EXPLORATORY DATA ANALYSIS](exploratory-data-analysis)
- [DATA ANALYSIS](data-analysis)
- [INSIGHTS](insights)
- [RECOMMENDATION](recommendation)


## PROJECT OVERVIEW 
This dataset contains key information on cancer patient pathways, including referral dates, treatment timelines, cancer types, and patient outcomes. It is designed to provide insights into the efficiency of cancer diagnosis and treatment processes, and to analyze compliance with critical waiting time standards established by healthcare authorities. The dataset is structured to support the tracking and reporting of cancer waiting times, monitoring treatment outcomes, validating COSD (Cancer Outcomes and Services Dataset) submissions, and conducting clinical audit analyses.

## DATA SOURCE
The dataset used in this project was provided by Kaggle. [Download here](https://www.kaggle.com/datasets/ayokunleagbaje/sample-nhs-cancer-data)

## TOOLS
1.	EXCEL: The dataset was gotten as a CSV, so I used Excel to open it. Go through the dataset to know the information it contains.
2.	SQL: The dataset was loaded to SQL for cleaning and querying.
3.	Power BI: This tool was used for visualization.

## DATA CLEANING
1.	Confirming the total normal of dataset and columns.
2.	Making sure they all represent their various data type.
3.	Checking for duplicates and null values.
4.	Converting data type.

## EXPLORATORY DATA ANALYSIS
1.	Calculating the average time taken at each stage before cancer treatment.
2.	Identifying delays in treatment by comparing different time intervals.
3.	Compare time delay across different cancer type.
4.	Distribution of patient outcome.
5.	Identify if delays in treatment impacts outcomes.
6.	Distribution of cancer types.
7.	Recovery rates by cancer types.

## DATA ANALYSIS
``` SQL
SELECT * FROM cancer_patient.`cancer patients nhs`;

-- Data cleaning
alter table `cancer patients nhs`
rename to cancer_patients;

-- table count
select count(*)
from cancer_patients;

-- column count
select count(column_name)
from information_schema.columns
where table_name = 'cancer_patients';

-- data type
select column_name, data_type
from information_schema.columns
where table_name = 'cancer_patients';

-- fixing data type (converting text to string)
select Referral_Date,
STR_TO_DATE(Referral_Date, '%m/%d/%Y')
from cancer_patients;

update cancer_patients
set Referral_Date = str_to_date(Referral_Date, '%m/%d/%Y');

select MDT_meeting_date,
STR_TO_DATE(MDT_meeting_date, '%m/%d/%Y')
from cancer_patients;

update cancer_patients
set MDT_meeting_date = str_to_date(MDT_meeting_date, '%m/%d/%Y');

select Decision_to_Treat,
STR_TO_DATE(Decision_to_Treat, '%m/%d/%Y')
from cancer_patients;

update cancer_patients
set Decision_to_Treat = str_to_date(Decision_to_Treat, '%m/%d/%Y');

select Treatment_start,
STR_TO_DATE(Treatment_start, '%m/%d/%Y')
from cancer_patients;

update cancer_patients
set Treatment_start = str_to_date(Treatment_start, '%m/%d/%Y');

-- checking for duplicate and null values
with duplicate_cte as
(
select Patient_id, Referral_date, MDT_Meeting_date, Decision_to_treat, Treatment_start, cancer_type, outcome,
row_number() over (partition by Patient_id, Referral_date, MDT_Meeting_date, Decision_to_treat, Treatment_start, cancer_type, outcome) as rn
from cancer_patients
)

select *
from duplicate_cte
where rn>1
and rn = null;


-- EDA
-- average time taken at each stage
select 
round(avg(Datediff(Mdt_meeting_date, referral_date)),1) as avg_referral_to_MDT,
round(avg(Datediff(Decision_to_treat, Mdt_meeting_date)),1) as avg_MDT_to_Decision,
round(avg(Datediff(Treatment_start, Decision_to_treat)),1) as avg_Decision_to_Treatment
from cancer_patients;

-- treatment delays by time internal
select patient_id, cancer_type, outcome,
	case
		when Datediff(Mdt_meeting_date, referral_date) > (select round(avg(Datediff(Mdt_meeting_date, referral_date)),1) from cancer_patients)
		then 'delayed' else 'On time'
	end as referral_delay,
	case
		when Datediff(Decision_to_treat, Mdt_meeting_date) > (select round(avg(Datediff(Decision_to_treat, Mdt_meeting_date)),1) from cancer_patients)
		then 'delayed' else 'On time'
	end as MDT_delay,
    case
		when Datediff(Treatment_start, Decision_to_treat) > (select round(avg(Datediff(Treatment_start, Decision_to_treat)),1) from cancer_patients)
		then 'delayed' else 'On time'  
        end as Treatment_delay 
	from cancer_patients;
    

-- number of days delays per cancer_type
 select cancer_type, round(avg(Datediff(treatment_start, referral_date)),1) as days_taken
 from cancer_patients
group by 1;

-- distribution of patient outcome
select Outcome, count(*) as total_outcome
from cancer_patients
group by 1
order by 2 desc;

-- number of recovery per cancer 
select cancer_type, outcome, count(*) as recovery_count
from cancer_patients
where outcome = 'recovered'
group by 1
order by 3 desc;

-- if delay impacts outcome
select outcome, round(avg(Datediff(Treatment_start, referral_date)),1) as total_days
from cancer_patients
group by 1;
```

## INSIGHTS
1.	The average time (in days) from Referral date to MDT date is 3.50 (Three and half days)
2.	The average time (in days) from MDT date to Decision date is 7.1o (Seven days)
3.	The average time (in days) from Decision date to Treatment start is 15.60 (Fifteen and half days)
4.	Cancer treatment time delay vary, with lung cancer requiring up to 27 days, while prostate, skin, breast, colon and leukemia cancers takes 26 days.
5.	The patient outcomes reveal the following distribution:
   -  Improved: 1,145 patients (highest)
   -  Deteriorated: 1,138 patients
   -  Recovered: 1,129 patients
   -  Stable: 1,088 patients
7.	The treatment delay varies in outcome:
 - Patients with a ‘stable’ outcome experience 27 days delay in treatment, while those with ‘recovered’, ‘deteriorated’, or ‘improved’ outcomes face a 26 days    delay in treatment.
 - The difference in delay is just a day, which is not statistically significant, therefore, the delay in treatment doesn’t affect the outcome.
8.	Recovery rates of patients vary across cancer types, with:
- Breast and lung cancer tied for the highest recovery rate, with 202 patients recovered.
- Skin cancer: 192 patients recovered.
- Colon cancer: 190 patients recovered.
- Prostate cancer: 182 patients recovered.
- Leukemia: 161 patients recovered (the lowest)

## RECOMMENDATION
STREAMLINING REFERRAL AND TREATMENT 
1.	Reduce MDT decision time: The average time from MDT date to Decision date is 7.10 days. Explore ways to expedite this process, such as implementing digital platforms for decision-making or allocating more resources to the MDT team.
2.	Shorten treatment start time: The average time from Decision date to Treatment start is 15.60 days. Identify bottlenecks in the treatment preparation process and implement efficiency improvements to reduce this timeframe.

IMPROVING PATIENT OUTCOMES
1.	Focus on leukemia treatment: Leukemia has the lowest recovery rate (161 patients). Consider allocating additional resources, such as specialized staff or equipment, to improve treatment outcomes for leukemia patients.
2.	Develop targeted interventions: Analyze the characteristics of patients with 'deteriorated' or 'stable' outcomes to identify potential areas for improvement. Develop targeted interventions to address these specific needs.


