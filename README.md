# Urban Economics Replication #2
## The Price of Prejudice
#### Authors: Morten Størling Hedegaard and Jean-Robert Tyran
#### Replication by: Pawonee Khadka, University of Alabama

The paper is based on a field experiment that goes on to investigate ethnic prejudice in the workplace. The authors want to see how potential discriminators respond to changes in the cost of discrimination. The paper finds that ethnic discrimination is common but highly responsive to the
“price of prejudice,” i.e., to the opportunity cost of choosing a less productive worker on ethnic grounds. Discriminators/Employers are willing to forego, on average,  8% of their earnings to avoid a coworker of other ethnic type. It was published in the American Economic Journal: Applied Economics in 2018. Here is a link to it:

https://doi.org/10.1257/app.20150241
All analysis for the original paper was done using STATA and all data and necessary code instructions was made publicly available.

With my replication work, I didn't have much to do with the raw data, but jumped into replication right away. Following is the chuck for replication of 
##### Table 2: Team Production Function.
In this table:
    the Dependent variable is the log of the number of envelopes stuffed in round 2 by worker i
    prod1i is the number of envelopes stuffed in round 1 by worker i
    prod1j is the number of envelopes stuffed by i’s coworker in round 2
    Alone is a dummy set to 1 if worker i works alone in round 2
    Male is worker i’s gender 
    Decision maker indicates if worker i makes a choice of coworker 
    The remaining dummies characterize team composition in round 2. 


The project is based on:
https://www.openicpsr.org/openicpsr/project/113648/version/V1/view?path=/openicpsr/113648/fcr:versions/V1/Price_of_Prejudice_Stata_data.dta&type=file

https://pubs.aeaweb.org/doi/pdfplus/10.1257/app.20150241

1. Estimating the production function									

* Remove OTHER participants (i.e. non-Danish and non-Muslim) from productivity function estimation

* productivity is a binary variable indicating whether a particular observation is included in production function estimation

replace productivity = 0 if ethnicity == 3
replace productivity = 0 if couple == 4

*****************************
* Create new variables
*****************************

* Transformations
gen lnprod1 = ln(prod_1)					// take logs for Cobb-Douglas estimation
gen lnprod2 = ln(own_prod_2)

label variable lnprod1 "Ln(Prod_1)"
label variable lnprod2 "Ln(Prod_2)"

*** Team dummies (mixed team is baseline) ***
* Homogeneous Muslim team
generate muslim_team = 0
replace muslim_team = 1 if ethnicity == 2 & couple == 1
label variable muslim_team "Muslim-sounding team"

* Homogeneous Danish team
generate danish_team = 0
replace danish_team = 1 if ethnicity == 1 & couple == 1
label variable danish_team "Danish-sounding team"

* Person working alone
generate alone = 0
replace alone = 1 if couple == 3
label variable alone "Alone 2nd round"

* Team variable
generate team = 1
replace team = 2 if danish_team == 1
replace team = 3 if muslim_team == 1
replace team = 4 if alone == 1

label define team 1 "Heterogeneous" 2 "Danish" 3 "Muslim" 4 "Alone"
label value team team 
label variable team "Team type"

* Dummy for having Danish-sounding name
generate danish = 0
replace danish = 1 if ethnicity == 1

label define danish 0 "Muslim-sounding" 1 "Danish-sounding"
label value danish danish
label variable danish "Danish-sounding"

* Temporary partner variable
gen lnprodpartnertemp = 0					// temporary partner variable
replace lnprodpartnertemp = ln(prod_partner) if alone == 0

* Dummy for the decision maker in a team
generate decision_maker = 0
replace decision_maker = 1 if type == 1 & couple != 3
label variable decision_maker "Decision maker"

* Interaction between being a decision-maker and being in a heterogeneous team
generate decision_maker_mixed = 0
replace decision_maker_mixed = decision_maker if muslim_team == 0 & danish_team == 0 & alone == 0
label variable decision_maker_mixed "D-M in heterogeneous team"

gen lnprod1alone = lnprod1 * alone
label variable lnprod1alone "ln(Prod_1) * alone"


*-----------------------------------------------;
* Estimate production function
*-----------------------------------------------;

* Estimations for table
quietly regress lnprod2 lnprod1 lnprodpartnertemp lnprod1alone male if productivity != 0, vce(robust)
est store A
quietly regress lnprod2 lnprod1 lnprodpartnertemp lnprod1alone male decision_maker if productivity != 0, vce(robust)
est store B
quietly regress lnprod2 lnprod1 lnprodpartnertemp lnprod1alone male danish_team muslim_team alone if productivity != 0, vce(robust)
est store C
quietly regress lnprod2 lnprod1 lnprodpartnertemp lnprod1alone male danish_team muslim_team alone decision_maker decision_maker_mixed if productivity != 0, vce(robust)
est store D


* Show estimates in table
#delimit;
estout A B C D ,
	cells(b(star fmt(%9.3f)) se(par))     
	stats(r2 r2_a N , fmt(%9.3f %9.3f %9.0g) labels(R-squared "Adjusted R-squared" n))      ///
	legend label collabels(none) varlabels(_cons Constant)
	starlevels(* 0.10 ** 0.05 *** 0.01);
#delimit cr

*-----------------------------------------------------------------------------------;
* Estimation of production function	for estimating cost of discrimination			;
* Based on model (A) in table 2
*-----------------------------------------------------------------------------------;

regress lnprod2 lnprod1 lnprodpartnertemp lnprod1alone male if productivity != 0, vce(robust)
est store Production_function

* Show joint production function estimates in table
#delimit;
estout Production_function,
	cells(b(star fmt(%9.3f)) se(par))     
	stats(r2_a N , fmt(%9.3f %9.0g) labels(R-squared n))      ///
	legend label collabels(none) varlabels(_cons Constant)
	starlevels(* 0.10 ** 0.05 *** 0.01);
#delimit cr










*****************************************************************************************************
*****************************************************************************************************
*																									
*							2. Estimating the cost of discrimination									
*																									
*****************************************************************************************************
*****************************************************************************************************











*****************************************************
* Delete uninformative observations					*
*****************************************************

drop if type != 1			// keep only decision makers
drop if ethnicity == 3 | type_day_1 == 9999 | type_day_2 == 9999 // drop if decision-maker or candidate does not have Muslim-sounding or Danish-sounding name
drop if (main==1 & info==0)

*****************************************************
* Predict production with own type			 		*
*****************************************************

*** Resetting variables ***
* Reset team composition dummies
replace alone = 0 

* Reset interaction terms
replace lnprod1alone = 0

* Partner productivity if partner was own type
replace lnprodpartnertemp = ln(prod_own)

*** Prediction of production with own type***
predict est_own								// predict productivity with own type
replace est_own = exp(est_own)				// converting to productivity (not on log scale)


*****************************************************	
* Predict production with other type			 	*
*****************************************************

* Partner productivity if partner was other type
replace lnprodpartnertemp = ln(prod_other)

*** Prediction of production with other type***
predict est_other								// predict productivity with other type
replace est_other = exp(est_other)				// converting to productivity (not on log scale)


*****************************************************
* Estimate cost										*
*****************************************************

* Estimated cost
generate cost_envelopes = est_other - est_own
label variable cost_envelopes "Cost (#envelopes)"

* transform into Euros
generate cost_euro = cost_envelopes * 4 / 7.44  // each envelope paid DKK 4, exchange rate DKK/EUR set to 7.44
label variable cost_euro "Cost (�)"


*****************************************************
* Delete temporary productivity variables							
*****************************************************
drop  _est_A _est_B _est_C _est_D _est_Production_function	// stored productivity estimates
drop  muslim_team danish_team alone lnprodpartnertemp decision_maker decision_maker_mixed lnprod1alone lnprod1 lnprod2		// variables for estimation











*****************************************************************************************************
*****************************************************************************************************
*																									
*							3. Estimating the demand for discrimination									
*																									
*****************************************************************************************************
*****************************************************************************************************





*****************************************************
* Create new variables							
*****************************************************


gen danish_cost_euro = danish * cost_euro
label variable danish_cost_euro "Danish*cost(�)"
gen male_cost_euro = male * cost_euro
label variable male_cost_euro "Male*cost(�)"




*****************************************************
* Probit regressions
* Estimated costs
*****************************************************




* Multiple regression analysis for small table
quietly probit discr cost_euro if main==1, vce(robust)
est store A1
quietly margins, dydx(*) post
est store MFX_A1

quietly probit discr cost_euro danish male if main==1, vce(robust)		
est store A2
quietly margins, dydx(*) post
est store MFX_A2

quietly probit discr cost_euro danish_cost_euro male_cost_euro if main==1, vce(robust)		
est store A3
quietly margins, dydx(*) post
est store MFX_A3

quietly probit discr cost_euro danish male danish_cost_euro male_cost_euro if main==1, vce(robust)		
est store A4
quietly margins, dydx(*) post
est store MFX_A4


* Print small table - marginal effects
#delimit;
estout MFX_A1 MFX_A2 MFX_A3 MFX_A4,
	cells(b(star fmt(%9.3f)) se(par))     
	stats(N, fmt(%9.0g) labels(n))      ///
	legend label collabels(none) 
	varlabels(_cons Constant)
	margin									/// print marginal effects
	starlevels(* 0.10 ** 0.05 *** 0.01);
#delimit cr


*****************************************************
* NoName follow-up
*****************************************************


*** Demand estimation for days based on NoName follow-up
* Multiple regression analysis for small table
quietly probit discr cost_euro if noname_followup==1, vce(robust)
est store NoName1
quietly margins, dydx(*) post
est store MFX_NoName1

* Print small table - marginal effects
#delimit;
estout MFX_NoName1,
	cells(b(star fmt(%9.3f)) se(par))     
	stats(N, fmt(%9.0g) labels(n))      ///
	legend label collabels(none) 
	varlabels(_cons Constant)
	margin									/// print marginal effects
	starlevels(* 0.10 ** 0.05 *** 0.01);
#delimit cr
