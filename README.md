# Calculating Medicaid Per Diem Reimbursement Rates to Nursing Facilities Using MAX and TAF claims

### Background and Objective
__________
Medicaid covers six in ten nursing home residents. Despite longstanding policy interest and debate over the adequacy of Medicaid payments, and a large research literature focused on whether Medicaid residents are disadvantaged, data on Medicaid payment rates in nursing homes are hard to come by. Most states do not release their facility-level base rate fee schedule, and even these rates are modified based on resident characteristics such as case-mix and acuity. **This public repository provides 2011-2021 fee-for-service daily reimbursement rates from state Medicaid programs to nursing facilities. We also provide our algorithm to calculate these rates from RIF Medicaid TAF and MAX long term care claims provided by the Centers for Medicaid Services (CMS).**


### Data
__________
* 2011-2015 Medicaid Analytic eXtract (MAX) files
	* Long Term Care file 
	* Personal Summary file 
* 2015-2021 Medicaid T-MSIS Analytic Extract (TAF) files 
	* Long Term Care file (header and line)
	* Demographics and Eligibility files 
* Medicare Wage Index Files


### Process for Developing Rate Calculation Protocol 
__________
#### Initial Protocol from Abt 

We adapted our protocol from a 2023 MACPAC report created by Abt Associates: Estimates of Medicaid Nursing Facility Payments Relative to Costs, which estimated 2019 rates. While this report was a helpful starting point, our work fills some key gaps to enable others to calculate rates independently: 

1) Abt calculated rates from raw claims claims obtained directly from state Medicaid programs, while most academic researchers have access to Research Identifiable Files (RIF) claims obtained from CMS. These RIF files are refined and partially redacted versions of the raw files from states. 
2) Abt provided an overview of their algorithm, but did not specify which claims fields they used. 
3) Abt's protocol applies to 2019 T-MSIS data, which translates well to the TAF (T-MSIS Analytic Files) data we have access to. However, it was not clear how to apply this protocol to Medicaid Analytic eXtract (MAX) data prior to 2016, which have different fields and formatting.

**Summary of Abt Protocol for Estimating 2019 Daily Rates from TAF**: 

* Limit to FFS service claims
* Drop one day claims 
* Drop bed hold claims 
* Drop crossover claims 
* Identify nursing facility claims 
* Drop states with missing/outlier data
* Calculate per diem rate from allowed amount field 
* Winsorize claims to 95th percentile within facility 
* Adjust claims using Medicare wage index
* Calculate average state per diem rate

#### Benchmarking 2019 rates to validate TAF protocol 
To validate our protocol for TAF data, we calculated 2019 average state rates and benchmarked them against the 2019 rates provided in the MACPAC report. We excluded states that had a > 5% difference from MACPAC rates. To further validate our rates, we benchmarked against publicly available rates collected from a handful of online state fee schedules. 


#### Adapting TAF Protocol to MAX claims

States transitioned from MAX to TAF file formats at various times between 2013-2015. Since the MAX file does not contain an allowed amount field, our main challenge was identifying its analog. We considered the following fields: 

* Medicaid Paid Amount 
* Medicaid Charge Amount 
* Beneficiary Liability 
* Third Party Liability 

Since these fields are also present in the TAF files, we compare them to the allowed amount within the 2016-2019 TAF claims. However, none of them seemed to be a direct analog to the allowed amount, so we tried to reconstruct the allowed amount with different combinations of these fields. We tried the following methods:

* Method A: Medicaid Paid Amount + Beneficiary Liability + Third Party Liability 
* Method B: Medicaid Paid Amount 
* Method C: Medicaid Paid Amount + Beneficiary Liability
* Method D: Medicaid Paid Amount + Third Party Liability 

We found that Method C provided accurate rates for a majority of states across years. Method B also provided accurate results for a handful of states. Therefore, we used these two methods to calculate allowed amounts in the 2011-2015 MAX claims. 



### Interpreting Datasets 
__________
Note: rates are reported in nominal dollars (unadjusted for inflation)

* pmt_var: 
	* PER_DIEM: per diem rate 
	* PER_DIEM_wage_adj: per diem rate adjusted for Medicare wage index 
* method: 
	* valid rates: 
		* allowed_amount: per diem rate calculated from TAF allowed amount field 
		* estimated_allowed_amount_B: MAX per diem rate calculated from Medicaid Paid Amount field 
		* estimated_allowed_amount_C: MAX per diem rate calculated from Medicaid Paid Amount + Beneficiary Liability fields 
	* invalid rates (reason for missing rates): 
		* outlier
		* file_unavailable
		* low_claim_count 
		* no_valid_method 

* claim_count: the number of claims used to calculate the average state per diem rate 



### Python scripts
__________
* 01a_prep_max_files.ipynb 
	* Prepares 2011-2015 Medicaid Analytic eXtract long term care claims (header file only). Limits to FFS nursing facility claims. Excludes one day claims, crossover claims, and bed hold claims. 
* 01b_prep_taf_files.ipynb 
	* Prepares 2013-2019 T-MSIS Analytic File long term care claims (header and line files). Limits to FFS nursing facility claims. Excludes one day claims, crossover claims, and bed hold claims. 
* 02_merge_area_wages 
	* Creates a 2011-2020 crosswalk of Medicare wage indexes for each zip code. 
* 03_calculate_state_rate.ipynb 
	* Calculates per diem rates for claim, adjusts claims by Medicare Wage index for each zip code, winsorize rates to 95th percentile within facility, calculates annual state averages. 

**Python Packages Used**
* pandas 
* dask.dataframe 
* numpy 
* os 
* datetime 
* gc 

### Relevant Publications 
___________
* [MACPAC Estimates of Medicaid Nursing Facility Payments Relative to Costs](https://www.macpac.gov/wp-content/uploads/2023/01/Estimates-of-Medicaid-Nursing-Facility-Payments-Relative-to-Costs-1-6-23.pdf)
* [Bowbliss et al: Assessing Medicaid Payment Rates and
Costs of Caring for the Medicaid Population Residing in Nursing Homes](https://aspe.hhs.gov/sites/default/files/documents/b0758ea351e38746025e99878d229bdc/assessing-medicaid-payment-rates-costs.pdf)
* [Academy Health TAF Reporting Checklist](https://academyhealth.org/sites/default/files/publication/%5Bfield_date%3Acustom%3AY%5D-%5Bfield_date%3Acustom%3Am%5D/taf_reporting_checklist_pdf_02.2026.pdf)


