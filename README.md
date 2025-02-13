# OMOP/ORDR(D) CCI CALCULATION README
This repository contains code developed by the TraCS Data Science Lab, which is part of the School of Medicine at the University of North Carolina at Chapel Hill.

# Charlson Comorbidity Index Background
Charlson et al. (1987) introduced the Charlson Comorbidity Index (CCI) as a measure of disease burden. They included 19 conditions with assigned weights that summed to an index value for each individual. Quan et al. (2005) and then Quan et al. (2011) provided important methodological updates to the CCI. The CCI is commonly used as a general indicator of individual comorbidity in studies. Additional perspective on the development and use of CCI can be found at Manitoba Centre for Health Policy (2023). There have been additional CCI methods papers and methodologic developments over time such as Klabunde et al. (2000) for cancer, as well as revisions for diabetes and other specialty-specific modifications. However, Quan et al. (2005) appears to be the most prevalent method employed for general CCI calculations. As such, that is the method implemented in this repo. 

# Structure of this Repo
We provide Python, R, and PostgreSQL programs to calculate the CCI. All execute the same logic outlined below and have been confirmed to return the same results. 

# Code Source Environment Notes
This code is designed to work on Observational Medical Outcomes Partnerships (OMOP) databases. We utilized the OMOP Common Data Model v5.3 for development in the University of North Carolina at Chapel Hill de-identified OMOP Research Data Repository, ORDR(D).

# Methods Applied in this Calculation
We use the 17 conditions listed in Quan et al. (2005) which is a slight adjustment from Charlson et al. (1987) to include all non-metastatic cancers in a single category instead of having leukemia and lymphoma separate. Quan et al. (2005) also provides ICD-9 and ICD-10 codes for each condition. We use the original weights from Charlson et al. (1987), not the updated weights from Quan et al. (2011). We include both ICD-9 and ICD-10 diagnoses.

To calculate the CCI, we collect ICD diagnoses listed under condition_source_value from the condition_occurrence table. We then compare the diagnoses codes to the ICD codes listed in Quan et al. (2005) to assign each diagnosis code to a comorbid condition. Each category can only count once for the final index score. We do not consider any diseases mutually exclusive, so if a patient is listed as having both diabetes without complications and diabetes with complications, they receive both scores. The final Comorbidity Index is the sum of the weights for each condition present for each patient over the analysis time period. The code used in this repo to calculate the CCI considers all the problem list diagnoses a patient has had. Other use cases may require a different time period of assessment, which is discussed below. 

# Limitations/Calculation Aspects to consider
The CCI was originally developed and validated using claims data for female breast cancer patients. It has never been validated in EHR data, though it has been widely adopted and used. 

In EHR data, there are multiple sources of diagnoses. We chose to source our diagnoses solely from the EHR Problem List. We consider the Problem List to be the most accurate representation of conditions across the entire database.

We emphasize that, even though the data is in OMOP, we use the *condition_source_value* field to retrieve ICD-9 and ICD-10 codes. We do not use *condition_source_concept_id*, which contains mapped SNOMED-CT codes. The use of SNOMED-CT codes is known to deliver higher values of CCI compared to the use of ICD codes in EHR data (see Viernes et al. (2020); Fortin (2021); Leese et al. (2023)).

Depending on the intended use case, there are different preferences about what date ranges to include for a CCI calculation. For simplicity and intended general use, we include all diagnoses over a lifespan. We exclude conditions that have start dates before the birth date of that patient. 

If you need to filter diagnosis date globally, across the whole database, here is some code and guidance to implement that for each language:
-	SQL: In the first common table expression, condition_start_date, uncomment and edit the following two lines at the end of the expression
    - AND vco.condition_start_date >= '2015-01-03'
    - AND vco.condition_start_date < '2016-01-03'
- Python: There is a subsection titled Query With Date that provides guidance and code. 
- R: See the comments in the code just before the condition_query is defined. 
More advanced filtering, such as based on individual diagnoses dates, is beyond the scope of what we provide, but we hope this code provides a useful starting point. 

We use the following categories and weights:
| Condition Name  | Weight  |
|---|---|
|  Myocardial infarction | 1  |
| Congestive heart failure  |  1 |
|  Peripheral vascular disease | 1  | 
|  Cerebrovascular disease | 1  |
|  Dementia |  1 | 
| Chronic pulmonary disease  | 1  |
| Connective tissue disease  |  1 | 
|  Peptic ulcer disease |  1 |
| Mild liver disease  | 1  | 
| Diabetes without complications  | 1  |
| Hemiplegia or paraplegia  |  2 | 
| Renal disease  | 2  |
| Diabetes with complications  | 2 | 
| Cancer (including leukemia and lymphoma)  | 2  |
| Moderate or severe liver disease  |  3 | 
| Metastatic cancer  |  6 |
|  AIDS | 6  | 

Specific notes on ICD codes: 
- We search the EHR for both ICD-9 and ICD-10 codes. You can find the codes associated with each condition both in the program files and Table 1 of Quan et al. (2005).
- There are four ICD-9 codes that are listed twice in Quan et al. (2005): 404.03, 404.13, 404.93, and 437.3. If a patient has one of these ICD-9 codes, we give them a score for both diseases.
- There are 5 ICD-9 codes (V43.4, V42.7, V42.0, V45.1, V56.x) that also correspond to valid ICD-10 codes that are not a comorbid condition. For these codes, we check that the source is ICD-9.
- We do not double count conditions, so if a patient has multiple diagnoses for congestive heart failure, that condition is counted a single time. 

# Authors
Josh Fuchs and Nathan Foster developed this code. 

# References
- Charlson, M. E., P. Pompei, K. L. Ales, and C. R. MacKenzie. 1987. “A New Method of Classifying Prognostic Comorbidity in Longitudinal Studies: Development and Validation.” Journal of Chronic Diseases 40 (5): 373–83.
- Fortin, Stephen P. 2021. “Predictive Performance of the Charlson Comorbidity Index: SNOMED CT Disease Hierarchy Versus International Classification of Diseases.” In 2021 OHDSI Global Symposium Showcase. https://www.ohdsi.org/wp-content/uploads/2021/08/38-Predictive-Performance-of-the-Charlson-Comorbidity-Index-SNOMED-CT-Disease-Hierarchy-Versus-International-Classification-of-Diseases_2021symposium.pdf.
- Klabunde, C. N., A. L. Potosky, J. M. Legler, and J. L. Warren. 2000. “Development of a Comorbidity Index Using Physician Claims Data.” Journal of Clinical Epidemiology 53 (12): 1258–67.
- Manitoba Centre for Health Policy. 2023. “Concept: Charlson Comorbidity Index.” February 3, 2023. http://mchp-appserv.cpe.umanitoba.ca/viewConcept.php?conceptID=1098.
- Peter J Leese, Robert F Chew, Emily Pfaff. 2023. “Charlson Comorbidity in OMOP: An N3C RECOVER Study.” In AMIA 2023 Annual Symposium.
- Quan, Hude, Bing Li, Chantal M. Couris, Kiyohide Fushimi, Patrick Graham, Phil Hider, Jean-Marie Januel, and Vijaya Sundararajan. 2011. “Updating and Validating the Charlson Comorbidity Index and Score for Risk Adjustment in Hospital Discharge Abstracts Using Data from 6 Countries.” American Journal of Epidemiology 173 (6): 676–82.
- Quan, Hude, Vijaya Sundararajan, Patricia Halfon, Andrew Fong, Bernard Burnand, Jean-Christophe Luthi, L. Duncan Saunders, Cynthia A. Beck, Thomas E. Feasby, and William A. Ghali. 2005. “Coding Algorithms for Defining Comorbidities in ICD-9-CM and ICD-10 Administrative Data.” Medical Care 43 (11): 1130–39.
- Viernes, Mph Benjamin, Phd Kristine E. Lynch, Mph Brian Robison, Mph Elise Gatsby, Phd Scott L. DuVall, and M. D. Michael E. Matheny. 2020. “SNOMED CT Disease Hierarchies and the Charlson Comorbidity Index (CCI): An Analysis of OHDSI Methods for Determining CCI.” In 2020 OHDSI Global Symposium Showcase. https://www.ohdsi.org/wp-content/uploads/2020/10/Ben-Viernes-Benjamin-Viernes_CCIBySNOMED_2020Symposium.pdf.
