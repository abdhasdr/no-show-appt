# No-Show Appointments

## Overview
This is a use case that is practically **guaranteed to show a small but tangible amount of value in a short amount of time** while providing business-friendly insights and showing off the entire platform (including MLOps).  No show appointments are a part of a larger theme in health care called *Disrupted Care*. Minimizing disrupted care will ... **TBD: get text from Sally / Rob / Joe Blue**.

There are two primary opportunities supporting the Disrupted Care theme that the no-show use case addresses:

- Patients suffer if they do not get the care they require. Ensuring that patients show up for needed care plays a critical role in patient health.
- A degree of certainty about an open slot in the schedule can be preemptively filled, either by standard over-booking or reaching out to a patient for a new visit which is recommended by another model (e.g. propensity for asthma attack, etc.)

### What problem are we trying to solve?

About 5% of scheduled office visits result in the patient not showing (or canceling within a short window which most providers also consider no-show). The pain directly attributable to no-show is the missed revenue from visits (an average of $180). _Future objection handling: Automated email reminders are probably already in use and yet the no-show rate is still ~5%._

Other indirect pains include staffing innefficiency and impact to patient health/satisfaction. Using an AI-powered approach to this problem is feasible with the data a customer already has and can represent a “quick win” for the business.

`Pro tip: An interesting flip on the no-show in the case of **home health**. In that case we focus on no-show by the **nurse**, which disrupts patient care more directly.` 

### What is the value to the customer?

- A prediction for a patient no-show with enough lead time may present an opportunity to reach out to the patient to ensure attendance or reschedule the appointment.
- Aggregating predictions at the begninning of the week can assist with staffing and number of appointments that can be booked.

### What are we trying to predict?

The no-show appointment use case is a binary classification that predicts if a patient will show up for an appointment or not (usually a last-minute cancellation = no-show). 

The two main consumption models for no-show appointments:

1. prioritize a call-list the week before the appointments 
2. filling open appointments (i.e overbooking).

Both require a fairly short time window. If you predict no-shows 6 months from now, it doesn’t make sense to send reminders and your customer is probably still booking appointments normally. One week seems about standard and provides enough time to contact a patient or re-book.

Consumption of the model is relatively simple. Unless the customer wants insights (e.g. why is the no-show rate for one clinic/provider significantly higher than others?), you can accomplish this by processing all of next week’s appointments on Friday morning to create the prioritized list. The call list may be broken out by clinic and distributed separately. Prediction explanations can be helpful but aren’t always required. Generally the efficacy of reaching out to a patient with a reminder will not improve if more information is given. 

Often when giving a demo it helps to break out the expected no-show rate by location (i.e. no show rate by zip code on a map) because it helps visualize the opportunity for the customer.

### How does DataRobot help?

DataRobot automatically performs the following tasks…

- Connects to your data and determines which data sources are relevant and visualizes the contribution to accuracy and the relationship between each column and no-show.
- Our data prep tool can help you aggregate powerful predictors such as historical patient no-show rate.
- Performs NLP on text data which could also include booking notes - then visualizes how snippets of text may affect no-shows.
- Trains dozens of potential solutions and helps you optimize the cost of a bad prediction vs. the value of a good one. 
- Recommends a solution *guaranteed* to be unbiased on race and gender (and any other feature you’d like to control for)
- Generates explanations that contribute to each high score
- Dcouments that your solution treats protected classes fairly with built-in bias & fairness assessment
- Prepares the model to make predictions that integrate with your BI tools or generate a prioritized call-list that optimizes you contacts
- Tracks the health of your model (including bias) and notifies you whenever a potential issue needs to be addressed

## Data & Features

We generaly like to see 2 years of historical appointments with show/no-show as the target. 2 years of history is prefered to include seasonality and some before/after COVID activity. `Pro-tip: you will want to engineer historical no-show rates at patient, physician, and office/clinic level. Use description of visit rather than a numeric code and you’ll get more interpretable word clouds.`

The shape of the data should be at the visit level (one row per visit) and must include the patient ID, the date/time of the scheduled appointment, the date the appointment was scheduled, the location of the appointment, and the physician seen + the target. Here are some important data sources: 
- **Demographics** - including age & gender, occupational status + location (when a hospital is concentrated in one area, the effect of geography is muted). If possible, we’d like to know the info about the responsible custodian if the patient is a minor. 
- **Patient history** - comorbidities, procedures, medications, etc.
- **Weather** data (if available) - can affect travel between home and clinic if extreme conditions exist - this one is fun to include but unlikely to help. _Extra pro tip: watch out for leakage on this one. You have to use the forecasted weather not actuals _
- **County** (COVID) data (if available) - number of cases may affect certain types of visits, including vaccinations. Hopefully we can phase this out soon.

A sample schema used with a customer is below:

| **Column Name**         | Type        | **Description**                                              |
| ----------------------- | ----------- | ------------------------------------------------------------ |
| DNA (did not arrive)    | Binary      | Binary target variable (0, 1)                                |
| Site                    | Categorical | site name                                                    |
| ClinicName              | Text        | name of clinic                                               |
| ClinicType              | Categorical | e.g. Trauma, Pre-op, Regular                                 |
| Hour                    | Numerical   | Hour of appointment                                          |
| Day                     | Numerical   | Day of appointment                                           |
| Month                   | Numerical   | Month of appointment                                         |
| Duration                | Numerical   | Duration of appointment                                      |
| LeadTime                | Numerical   | Time, in days, between the date the appointment was created and when it occurred |
| AttendType              | Categorical | e.g. New, Follow-Up                                          |
| Priority                | Numerical   | Appointment Priority                                         |
| Specialty               | Categorical | Appointment speciality                                       |
| IsCancerRef             | Binary      | Whether the appointment is related to a cancer referral      |
| Attendance              | Numerical   | Numerical variable, between 0 and 1, which is the patients past attended appointment divided by their past total appointments |
| PreviousDNA             | Boolean     | Binary indicator of whether the patient has ever had a DNA   |
| Sex                     | Categorical | Patients sex                                                 |
| Age                     | Numerical   | Patients age                                                 |
| IncomeDecile            | Numerical   | ONS deprivation deciles based on the patient’s postcode      |
| CrimeDecile             | Numerical   | ONS deprivation deciles based on the patient’s postcode      |
| EmploymentDecile        | Numerical   | ONS deprivation deciles based on the patient’s postcode      |
| EducationDecile         | Numerical   | ONS deprivation deciles based on the patient’s postcode      |
| HealthDecile            | Numerical   | ONS deprivation deciles based on the patient’s postcode      |
| HousingDecile           | Numerical   | ONS deprivation deciles based on the patient’s postcode      |
| EnvironmentDecile       | Numerical   | ONS deprivation deciles based on the patient’s postcode      |
| RoadMiles               | Numerical   | Miles from the patients postcode to the site of the appointment |
| LiverDisease            | Binary      | Indicator variable for various conditions that the patient has been previously diagnosed with |
| MI                      | Binary      | Indicator variable for various conditions that the patient has been previously diagnosed with |
| Dementia                | Binary      | Indicator variable for various conditions that the patient has been previously diagnosed with |
| PepticUlcer             | Binary      | Indicator variable for various conditions that the patient has been previously diagnosed with |
| CongestiveHeartFalure   | Binary      | Indicator variable for various conditions that the patient has been previously diagnosed with |
| ConnectiveTissueDisease | Binary      | Indicator variable for various conditions that the patient has been previously diagnosed with |
| CrohnsDisease           | Binary      | Indicator variable for various conditions that the patient has been previously diagnosed with |
| Diabetes                | Binary      | Indicator variable for various conditions that the patient has been previously diagnosed with |
| Hemi/Paraplegiac        | Binary      | Indicator variable for various conditions that the patient has been previously diagnosed with |
| PulmonaryDisease        | Binary      | Indicator variable for various conditions that the patient has been previously diagnosed with |
| HIV/AIDS                | Binary      | Indicator variable for various conditions that the patient has been previously diagnosed with |
| VascularDisease         | Binary      | Indicator variable for various conditions that the patient has been previously diagnosed with |
| Cancer                  | Binary      | Indicator variable for various conditions that the patient has been previously diagnosed with |
| RenalDisease            | Binary      | Indicator variable for various conditions that the patient has been previously diagnosed with |
| RheumaticDisease        | Binary      | Indicator variable for various conditions that the patient has been previously diagnosed with |
| ConditionCount          | Numerical   | Count of special conditions                                  |

Here is some sample data for the above schema

| DNA  | SITE | ClinicName                           | ClinicType | Hour | Day  | Month | Duration | LeadTime | ATTEND_TYPE | Priority | Specialty                                   | IsCancerRef | Attendance | PreviousDNA | Sex    | ActualAge | IncomeDecile | CrimeDecile | EmploymentDecile | EducationDecile | HealthDecile | HousingDecile | EnvironmentDecile | RoadMiles | Liver Disease | MI   | Dementia | Peptic Ulcer | Congestive Heart Failure | Connective Tissue Disorder | Crohn's Disease | Diabetes | Hemi/Paraplegia | Pulmonary Disease | HIV/AIDS | Vascular Disease | Cancer | Renal Disease | Rheumatic Disease | ConditionCount |
| ---- | ---- | ------------------------------------ | ---------- | ---- | ---- | ----- | -------- | -------- | ----------- | -------- | ------------------------------------------- | ----------- | ---------- | ----------- | ------ | --------- | ------------ | ----------- | ---------------- | --------------- | ------------ | ------------- | ----------------- | --------- | ------------- | ---- | -------- | ------------ | ------------------------ | -------------------------- | --------------- | -------- | --------------- | ----------------- | -------- | ---------------- | ------ | ------------- | ----------------- | -------------- |
| 0    | FGH  | mr b frank fu ortho                  | Regular    | 10   | 2    | 1     | 25       | 97       | CC_FOLLOWUP | 3        | TRAUMA AND ORTHOPAEDICS                     | 0           | 0.92       | 1           | Female | 72        | 5            | 9           | 4                | 5               | 2            | 10            | 1                 | 3.7       | 0             | 1    | 0        | 0            | 0                        | 0                          | 0               | 0        | 0               | 0                 | 0        | 1                | 0      | 0             | 0                 | 2              |
| 0    | RLI  | dr d kim oncology  day hosp          | Regular    | 14   | 4    | 1     | 40       | 28       | CC_FOLLOWUP | 3        | CLINICAL ONCOLOGY (previously RADIOTHERAPY) | 0           | 0.98       | 1           | Male   | 63        | 8.142857143  | 6.07254     | 7.428571429      | 9.142857143     | 4.928571429  | 6.142857143   | 3.357142857       | 1.4       | 0             | 0    | 0        | 0            | 0                        | 0                          | 0               | 0        | 0               | 0                 | 0        | 0                | 0      | 0             | 0                 | 0              |
| 0    | RLI  | dr f s htke haematology              | Regular    | 9    | 4    | 1     | 15       | 28       | CC_FOLLOWUP | 3        | CLINICAL HAEMATOLOGY                        | 0           | 0.97       | 0           | Male   | 74        | 6.2          | 4.7         | 6.1              | 6.5             | 3.4          | 8.7           | 4.3               | 4.6       | 0             | 0    | 0        | 0            | 0                        | 0                          | 0               | 0        | 0               | 0                 | 0        | 0                | 1      | 0             | 0                 | 1              |
| 0    | WGH  | dr charleston sat geriatric med  wgh | Regular    | 9    | 6    | 3     | 20       | 36       | CC_FOLLOWUP | 3        | GERIATRIC MEDICINE                          | 0           | 1          | 1           | Female | 92        | 5            | 10          | 8                | 8               | 7            | 4             | 3                 | 12.8      | 0             | 0    | 0        | 0            | 0                        | 0                          | 0               | 0        | 0               | 0                 | 0        | 0                | 0      | 0             | 0                 | 0              |
| 0    | FGH  | dr jones oncology fri fgh            | Regular    | 16   | 2    | 2     | 20       | 182      | CC_FOLLOWUP | 3        | CLINICAL ONCOLOGY (previously RADIOTHERAPY) | 0           | 0.94       | 0           | Female | 55        | 2            | 1           | 1                | 2               | 1            | 8             | 1                 | 2.4       | 0             | 0    | 0        | 0            | 0                        | 0                          | 0               | 0        | 0               | 0                 | 0        | 0                | 1      | 0             | 0                 | 1              |
| 0    | FGH  | dr marsden monday haematology  fgh   | Regular    | 12   | 3    | 1     | 30       | 182      | CC_FOLLOWUP | 3        | CLINICAL HAEMATOLOGY                        | 0           | 1          | 1           | Male   | 84        | 8            | 10          | 7                | 6               | 7            | 5             | 3                 | 25.3      | 0             | 0    | 0        | 0            | 0                        | 0                          | 0               | 1        | 0               | 0                 | 0        | 0                | 1      | 0             | 0                 | 2              |
| 1    | FGH  | orthotic thursday  fgh               | Regular    | 10   | 4    | 1     | 10       | 197      | CC_FOLLOWUP | 3        | REHABILITATION                              | 0           | 0.86       | 1           | Male   | 17        | 3            | 9           | 4                | 4               | 1            | 10            | 7                 | 2.2       | 0             | 0    | 0        | 0            | 0                        | 0                          | 0               | 0        | 0               | 0                 | 0        | 0                | 0      | 0             | 0                 | 0              |
| 0    | FGH  | mr k jones antenatal                 | Regular    | 13   | 3    | 1     | 15       | 350      | CC_FOLLOWUP | 3        | OBSTETRICS                                  | 0           | 0.76       | 1           | Female | 30        | 1            | 1           | 2                | 1               | 1            | 8             | 2                 | 2.7       | 0             | 0    | 0        | 0            | 0                        | 0                          | 0               | 0        | 0               | 0                 | 0        | 0                | 0      | 0             | 0                 | 0              |
| 0    | RLI  | visual fields  qvh                   | Regular    | 11   | 3    | 1     | 15       | 91       | CC_FOLLOWUP | 3        | CLINICAL PHYSIOLOGY                         | 0           | 0.67       | 1           | Female | 82        | 7.6          | 8.4         | 7.6              | 7.6             | 5.3          | 5.7           | 4.2               | 9.3       | 0             | 0    | 0        | 0            | 0                        | 0                          | 0               | 0        | 0               | 0                 | 0        | 1                | 0      | 0             | 0                 | 1              |
| 0    | FGH  | mr johnston ortho  fgh               | Regular    | 13   | 1    | 1     | 15       | 56       | CC_NEW      | 2        | TRAUMA AND ORTHOPAEDICS                     | 0           | 0.88       | 1           | Female | 86        | 4            | 7           | 2                | 3               | 2            | 10            | 1                 | 1.4       | 1             | 0    | 0        | 0            | 0                        | 1                          | 0               | 0        | 0               | 0                 | 0        | 0                | 0      | 0             | 1                 | 3              |
| 0    | RLI  | mr bolton ophthalmology  rli         | Regular    | 15   | 3    | 1     | 15       | 105      | CC_NEW      | 3        | OPHTHALMOLOGY                               | 0           | 0.78       | 1           | Female | 44        | 7.6          | 8.6         | 7.3              | 7.2             | 5.3          | 5.7           | 4.2               | 9.2       | 0             | 0    | 0        | 0            | 0                        | 0                          | 0               | 0        | 0               | 0                 | 0        | 0                | 0      | 0             | 0                 | 0              |
| 1    | WGH  | mr wilson colorectal surgery         | Regular    | 13   | 1    | 1     | 15       | 31       | CC_NEW      | 3        | COLORECTAL SURGERY                          | 0           | 0.76       | 1           | Male   | 27        | 7            | 8           | 6                | 4               | 6            | 10            | 4                 | 16.3      | 0             | 0    | 0        | 0            | 0                        | 0                          | 0               | 0        | 0               | 0                 | 0        | 0                | 0      | 0             | 0                 | 0              |
| 0    | RLI  | l waters exercise tolerance testing  | Regular    | 14   | 1    | 2     | 10       | 31       | CC_FOLLOWUP | 2        | CLINICAL PHYSIOLOGY                         | 0           | 0.57       | 1           | Male   | 59        | 4.615384615  | 5.36809     | 4.076923077      | 4.769230769     | 4.54228      | 8.538461538   | 5.12456           | 4.7       | 0             | 0    | 0        | 0            | 0                        | 0                          | 0               | 0        | 0               | 0                 | 0        | 0                | 0      | 0             | 0                 | 0              |
| 0    | RLI  | dr f s htke haematology              | Regular    | 10   | 1    | 8     | 15       | 24       | CC_FOLLOWUP | 3        | CLINICAL HAEMATOLOGY                        | 0           | 1          | 1           | Male   | 65        | 9            | 9.5         | 8.9              | 8.75            | 7.8          | 6.75          | 6                 | 16.2      | 0             | 0    | 0        | 0            | 0                        | 0                          | 0               | 0        | 0               | 0                 | 0        | 0                | 1      | 0             | 0                 | 1              |
| 0    | WGH  | lucy scott parkinson nurse  wgh      | Regular    | 9    | 1    | 2     | 15       | 41       | CC_FOLLOWUP | 3        | GERIATRIC MEDICINE                          | 0           | 0.98       | 0           | Male   | 74        | 7.571428571  | 6.845218    | 7.571428571      | 7.142857143     | 6.857142857  | 6.142857143   | 3.142857143       | 2.4       | 0             | 0    | 0        | 0            | 0                        | 0                          | 0               | 0        | 0               | 0                 | 0        | 0                | 0      | 0             | 0                 | 0              |
| 0    | WGH  | dr l khan haem                       | Regular    | 9    | 4    | 1     | 20       | 22       | CC_FOLLOWUP | 3        | CLINICAL HAEMATOLOGY                        | 0           | 0.94       | 0           | Female | 32        | 7.25         | 58          | 7.25             | 8.2             | 6.25         | 7.8           | 2.25              | 2.8       | 0             | 0    | 0        | 0            | 0                        | 0                          | 0               | 0        | 0               | 0                 | 0        | 0                | 0      | 0             | 0                 | 0              |
| 0    | WGH  | helen harris lymphodema nurse        | Regular    | 15   | 3    | 3     | 10       | 182      | CC_FOLLOWUP | 3        | MEDICAL ONCOLOGY                            | 0           | 1          | 1           | Female | 68        | 8.5          | 8.625       | 8.35             | 7.75            | 7.538        | 7.375         | 5.125             | 0.6       | 0             | 0    | 0        | 0            | 0                        | 0                          | 0               | 0        | 0               | 0                 | 0        | 0                | 1      | 0             | 0                 | 1              |
| 0    | WGH  | dr charleston fri geriatric med  wgh | Regular    | 9    | 5    | 1     | 20       | 189      | CC_FOLLOWUP | 3        | GERIATRIC MEDICINE                          | 0           | 0.54       | 1           | Female | 84        | 7            | 8           | 7                | 4               | 6            | 10            | 2                 | 16.3      | 0             | 0    | 0        | 1            | 1                        | 0                          | 1               | 0        | 0               | 0                 | 0        | 0                | 0      | 1             | 0                 | 4              |
| 0    | RLI  | f greenhalgh dietitian  rli          | Regular    | 10   | 1    | 1     | 20       | 126      | CC_FOLLOWUP | 3        | DIETETICS                                   | 0           | 0.67       | 1           | Female | 4         | 4.75         | 2.85        | 4.25             | 4.35            | 3            | 6.88          | 4                 | 2.4       | 0             | 0    | 0        | 0            | 0                        | 0                          | 0               | 0        | 0               | 0                 | 0        | 0                | 0      | 0             | 0                 | 0              |
| 0    | RLI  | f greenhalgh dietitian  rli          | Regular    | 9    | 2    | 1     | 20       | 96       | CC_FOLLOWUP | 3        | DIETETICS                                   | 0           | 0.64       | 1           | Female | 2         | 5.25         | 2.75        | 4.85             | 4.75            | 3            | 7.166666667   | 3.75              | 4.8       | 0             | 0    | 0        | 0            | 0                        | 0                          | 0               | 0        | 0               | 0                 | 0        | 0                | 0      | 0             | 0                 | 0              |

## Models & Insights

### Recommended model settings

- Usually run as OTV - you want the model optimized on the most recent data (and you have a built-in holdout)
- If not able to run OTV, use an entire year with group partitioning on patient ID
- Aggregation (rather than feature discovery) is more likely to yield easily interpretable findings. 
- One of the key features is historical no-show rate. Might want to experiment with 30/90/180 day windows.
- Use caution when generating historical no-show rates. Should be based on appointment date to avoid leakage.
- For all others use default settings (validation, optimization metric, etc.)

### What are some of the key insights identified?

- The more time that has passed between the scheduled date and the appointment date, the higher the no-show rate (i.e. people forget or sh*t happens).
- Patients who were late in the past are more likely to show up in the future (i.e historical no-show rate). It’s likely that historical no-show rates at the clinic, office, or physician level will also yield interesting insights. 
- A combination of age / visit type can yield interesting insights: children always show up because parents care for their children more than we care for ourselves. Also, minor or routine visits like flu shots and mammograms have lower no-show rates. `Pro tip: this may not yield usable insights. results may vary`

*Bias could be a factor and if available it should be included. There isn’t a huge legal issue or fine if this model contains bias, but as this is a differentiator for us, it’s recommended that the customer is aware of this capability.*

### Model Interpretation 
Below are some examples from an actual project completed by a customer. They are meant for illustration only. Your customer can expect _similar_ insights but avoid making guarantees. The below are included as examples that should resonate with the business and earn trust on the rest of the model's insights.

Sample feature impact. 
![noshow_sharp_impact](noshow_impact.png)

Example feature effect for _days since scheduled_ (target = prob[no-show])
![noshow_sharp_effects](noshow_effects.png)

Example feature effect for "Historical no-show rate" (target = prob[no-show])
![no_show_sharp_NOS](no_show_NOS.png)

Example word cloud based on appointment type:
![noshow_sharp_wordcloud](noshow_wordcloud.png)


## Results

Logloss or AUC can be deceptive metrics. You'll want to look at the confusion matrix to estimate ROI. 
However, simply using the raw prediction works for rank-odering the call list.

### Threshold analysis and other metrics

- In a recent POV, a customer shared that $180 is the average missed revenue of a no-show appointment and $4 is the avg cost of a (non-automated) phone call reminder
- The values can help fill out a profit matrix - we consider TP and FN to be $0 since that represents the status quo.
- Maximizing the profit provides estimated ROI and a threshold. It's useful to assume a conservative turnaround rate (e.g. only 30% of patients reminded  will change their behavior)
- Consideration for operational constraints should be a factor. It's unlikely that a doctor's office has time to reach out to every patient with a reminder, nor overbook 100% of visits. 

### How is the model consumed?
- The model should be deployed after threshold is set for maximum ROI.
- In setting up the model for use, it's important to automate the features (either the aggregation or secondary files for discovery)
- If the file is prepared you can use the drag-and-drop prediction option if the customer doesn't have the resources to use the API. 
- Option 1: csv is fine for call-list, but phone-numbers are looked up by the dialer or linked beforehand. It's most likely the list will be stratified and distributed by clinic or office (because each is responsible for its own patients).
- Option 2: a dashboard can be created (or AI apps) that allows the user to browse number of predicted no-shows by office and those vacancies can be filled either by FIFO or a cultivated list of recommended patients (supplied by another model).

### How important is monitoring?
- For this use case it's important to monitor drift because there are seasonal changes as well as changes in overall public health (esp. during COVID)
- Note: if you use day of the week or month in your model (not completely without value) then your drift will always start off as a red alert (and will stay that way until it's seen enough data)
- You'll want to key an eye on the aggregated features. They're impactful and also the most likely place for error (or delay in updating). 
- Monitoring accuracy is viable (you should have actuals arriving every day)
- Bias/fairness is a nice feature but some of the worst outcomes (i.e. fines, public outcry) are not applicable here

