<b>Topic:</b> Modelling & Visualisation

<b>Assignment:</b> 2 - Produce a regression  and time series forecasting model.

The python file can be found within this repository or alternatively opened in google colab below:

[![Open In Colab](https://colab.research.google.com/assets/colab-badge.svg)]([https://colab.research.google.com/github/USERNAME/REPO/blob/BRANCH/PATH/TO/NOTEBOOK.ipynb])

Project: Analysing public A&E attendance, emergency admissions and bed utilisation data. This is not an official NHS project, it was a project done for my masters whilst learning how to build different types of statistical models. I've provided examples throughout the below as the python code may be helpful for others.

<b>Introduction:</b>

As a senior analyst within NHS England I am responsible for providing intelligence and insight to the East of England region. The emergency care access standard, as per the NHS constitution (Department of Health and Social Care, 2023), is 95% of A&E attendances to be seen and treated within 4 hours. Currently, the East of England region is performing at 73% (NHS England, no date) of A&E attendances seen and treated within 4 hours. I have decided to focus on this dastaset to look for trends and patterns that may be able to help inform our regional internal stakeholders who are supporting NHS Trusts with improvement where to focus.

NHS England publishes two datasets that I use within this report; Monthly A&E Attendances and Emergency Admissions (NHS England, no date) and Critical care and General & Acute Beds SitRep Reports (NHS England, no date). I have supplemented this with 3 datasets. 

To support the recovery of the A&E target, this report will focus on building a model to firstly investigate the drivers for long waits and secondly forecasting future demand to A&E departments nationally.

<b>Data Collection and Preprocessing:</b>

In preparing the dataset for this report, I employed web scraping techniques to collect data from identified sources, ensuring consistency in file structures across imports. The focus was narrowed to acute trusts with Type 1 A&E departments, capable of handling all acuity levels, including resuscitation cases. To address the dynamic nature of healthcare organisations, I implemented a system to account for trust mergers, updating organisation codes and names accordingly. This approach maintained data continuity and accuracy over time. Data cleaning processes included:

-	Removing non-acute organisations
 
-	Handling null values
 
-	Forward-filling missing monthly submissions (limited to an average of one per year per trust)
 
-	Excluding trusts with more than six months of missing data to preserve data integrity
 
While initial exploratory analysis revealed some monthly outliers, these were retained for further investigation during time series forecasting, as their impact on seasonality and overall trends was yet to be determined. This data preparation laid the groundwork for robust analysis, ensuring the dataset's reliability and relevance. The main issues I encountered in this stage was the imported data not being treated as an interger and therefore showing a lower number than expected, which was handled with converting values to intergers.

I decided to join and clean the data needed for the regression model alongside building the model. This allowed for the code to be kept together and the notebook to flow better.

Throughout my data processing stage, I will often put in printed statements to compare the data before and after a transformation is made to check that the function has worked as expected. I also use scrollable tables through my code instead of headings or display; this is a personal preference as I’m transforming datasets.

I used a 60/20/20 split of data for my time series forecasting model, which allowed me to effectively train, validate, and test the model. The training set (60%) enabled the model to learn patterns and trends from historical data, while the validation set (20%) helped fine-tune hyperparameters and assess performance during training, thus preventing overfitting and reviewing the 19/20 period for exclusion. Finally, the test set (20%) provided an independent evaluation of the model's predictive capabilities on unseen data, ensuring a robust assessment of its accuracy and reliability in real-world applications.

<b>Data used:</b>

This notebook web scrapes publicly available data. But there are 3 files provided within this repo that the ipynb will prompt for you to upload if not found in the directory.

<b>Exploratory Data Analysis:</b>

Since the start of 2024, there has been a consistent rise in A&E attendances nationally. The chart below illustrates the impact of COVID-19 during the 2019/20 period, which was included in the time series analysis. If this upward trend in attendances continues, we may need to reset the limits on the SPC chart, as there is a clear increase when comparing pre-COVID and post-COVID data.

![A E_Attendances_Type_1_SPC_Chart](https://github.com/user-attachments/assets/7e549397-12f4-4567-b63b-c37925a8ecef)

As A&E attendances rise, the performance against the four-hour target deteriorates. The chart below displays the percentage of A&E attendances that exceeded four hours. Similar to A&E attendances, we should consider reviewing the SPC indicators, as the upper limit of the entire time series is approaching what we expect the average to be based on recent months.

![A E_Attendances_Perc4Hr_SPC_Chart](https://github.com/user-attachments/assets/75304d30-14e3-433d-ac16-aad0f1d96bc9)

The decline in A&E performance, coupled with rising demand, directly affects patient flow through hospitals. Nationally, the number of patients waiting over four hours from the decision to admit to actual admission is being closely monitored. While prior to the COVID-19 pandemic, it was relatively uncommon for patients to wait between four to twelve hours for admission, we are now witnessing a sustained upward trend. We can expect to see this worsen further as in recent days (as of 8th January 2025) there is a high demand on emergency services due to rising flu cases (Sky News, 2025).

![pts_4_12_hours](https://github.com/user-attachments/assets/fd458402-f3f9-4abc-8b27-6e3092f4520f)

Before the onset of COVID-19, it was rare for patients to experience waits exceeding twelve hours in A&E. However, since the pandemic began, there has been a significant increase in the number of patients waiting for a bed. Despite efforts made in 2023 and late 2024 to reduce these wait times, the volume of patients continues to rise.

There are some outlying Trusts which could be explored to understand the causes for their high volume of 4-12 hour waits. Using the metrics from the box & whisker plot, we can start to see the volume of outliers and that there are some Trusts with a low volume of patients waiting 4 – 12 hours, with minimum being 2, which we could explore for learning good practice.

![boxplot1](https://github.com/user-attachments/assets/4d52cadd-e38e-4e2a-9394-2d37498481e6)

The pressure on hospital beds not only affects admission times but also leads to cancelled elective procedures due to emergency demand. This situation highlights the ongoing challenges faced by healthcare systems as they strive to balance emergency care needs with planned treatments.

![pts_12_hours](https://github.com/user-attachments/assets/dd8e1294-bc36-4e19-8d16-4430795fc20a)

As with the previous metric, we can see there are still a high volume of Trusts that are achieving low volumes of 12 hour waits, but we could use this chart to make improvements such as setting targets for those within the 75th or 85th percentile to improve to the median. I developed an interactive time series chart that can be used to focus on the outlying organisations demand.

![boxplot2](https://github.com/user-attachments/assets/2f4d95f7-86de-40d4-8c30-389c222b1f39)

This report aims to support the recovery of the four-hour standard by developing two models that forecast A&E attendances and identify factors that may indicate high patient volumes exceeding four hours. The correlogram below reveals a strong positive correlation between Type 1 A&E attendances and Type 1 emergency admissions. Additionally, there is a moderate to high correlation between overall attendances and those exceeding four hours, particularly among patients who have waited between four to twelve hours from decision to admission. Therefore, as wait for admission increase, we could assume that the % of patients waiting over 4 hours in A&E increases.

![Correlation_Matrix](https://github.com/user-attachments/assets/0792c1f2-871c-40ff-ba31-bb904e27e599)

The national A&E and emergency admissions data was sufficient for my time series model as I only used ‘A&E attendances Type 1’ historical patterns to understand if it was possible to forecast future attendances. When I started to build my regression model, I included additional metrics from other datasets available to enhance the inputs.

<b>Statistical Modelling and Time Series Analysis:</b>
In this analysis, I developed two predictive models aimed at forecasting A&E attendances and identifying factors that contribute to patients exceeding the four-hour wait time. Before creating the ARIMA model, I determined the best differencing order (d) to be 2, with AIC values for d=0, d=1, and d=2 recorded as follows: [1263.56, 1242.62, 1231.65]. This step was crucial for ensuring the stationarity of the time series data.

The first model is an ARIMA (AutoRegressive Integrated Moving Average) model, specifically ARIMA(5, 2, 3). This model was built using historical data on total A&E attendances Type 1, encompassing 48 observations. The choice of ARIMA was motivated by its effectiveness in handling time series data that exhibit trends and seasonality. I checked for seasonality within the data to identify patterns that could influence attendance figures, particularly during peak times such as winter months or holiday seasons. During the modeling process, I encountered several challenges, including determining the optimal parameters for the ARIMA model and addressing potential outliers in the data. 

Notably, after initially including data from the 2019/20 period, which was significantly impacted by COVID-19, I found that removing this period during the validation stage resulted in overly optimistic predictions. The Root Mean Squared Error (RMSE) for the validation period was calculated at 418,921.50, while it dropped to 330,191.79 for the test period. As I’ve not build an ARIMA time series forecasting model before, I felt that a next step would be to read more on tuning an ARIMA model and making adjustments.

![ARIMA_forecast](https://github.com/user-attachments/assets/c774c13e-21c3-4309-af44-7fe61e9a15ef)

The second model is an OLS (Ordinary Least Squares) regression model designed to analyse factors influencing A&E attendances over four hours. The regression model achieved an R-squared value of 0.972 and an adjusted R-squared of 0.951, indicating a strong fit to the data while still accounting for potential overfitting. The F-statistic of 48.23 and a p-value of 2.34×10−382.34×10−38 suggest that the overall model is statistically significant.

The top five most significant predictors identified in the regression analysis include:
 
-	A&E attendances Type 1: Coefficient of 0.2002
-	Percent waited over 12 hours: Coefficient of -170.9781
-	Population aged 65 and over: Coefficient of -0.0066
-	Adult escalation beds available: Coefficient of -1474.3685

![Regression_top_predictors](https://github.com/user-attachments/assets/880b6ae0-f836-4307-9fcf-2c3a64197aa6)

I anticipated that the volume of A&E attendances would significantly influence the percentage of patients waiting over four hours, as increased demand on A&E departments can lead to longer wait times. A high percentage of patients waiting over 12 hours may indicate overcrowding in the A&E department, potentially causing less complex cases to exceed the four-hour wait threshold. Notably, the presence of older populations in A&E departments emerged as a significant predictor, suggesting that these patients may be more likely to experience longer wait times compared to younger patients or that they contribute to a higher overall demand for services. Furthermore, the availability of adult escalation beds indicates pressure on bed flow outside of A&E, which could result in patients requiring admission facing delays, thereby prolonging their wait times in the department.

<b>Conclusion & future work:</b>
A significant point of discussion was whether to exclude the 2019/20 COVID-19 impact from the analysis. While removing this data appeared to enhance predictive accuracy in both models, it also risks overlooking critical insights into how emergency demand patterns have shifted due to the pandemic. Balancing these considerations is essential for developing a comprehensive understanding of current trends and preparing for future demand. 

The outputs from both models provide valuable insights into A&E attendances. The ARIMA model forecasts suggest a continued increase in attendances over time, which aligns with observed trends in emergency healthcare demand. Meanwhile, the regression model highlights key predictors affecting wait times and provides a robust statistical framework for understanding these dynamics. 

To refine these models further, I will consider including more granular data on patient demographics and seasonal factors which could improve model accuracy. Although I investigated a SARIMA mode, the ARIMA model seemed to perform better which I did not expect. I believe that the cause of this could be potential outliers or the COVID-19 impact, so I would investigate these further to see if further cleaning the data and utilising SARIMA would generate better results. There are also further assumptions we could build into the model from local knowledge to continuously refine it, and if operationalised, regularly update the models with new data to ensure they remain relevant and accurate in forecasting future A&E attendances. With my regression model, I plan to utilise the significant predictors to see if they could be used in a real-time model to give early warnings of A&E pressures.

<b>Examples of other chart types in this notebook:</b>

![Violin_Plots](https://github.com/user-attachments/assets/a231ea5b-91b9-4bc8-9a5b-5da73bf08d42)

![Residual_plot](https://github.com/user-attachments/assets/bebec123-69cd-4771-ad59-48fba9c58980)

<b>References</b>
Department of Health and Social Care. (2023, August 17). NHS Constitution for England. GOV.UK. https://www.gov.uk/government/publications/the-nhs-constitution-for-england [Accessed: 7 Nov 2024]

NHS England (no date) Statistics » A&E attendances and emergency admissions 2024-25. https://www.england.nhs.uk/statistics/statistical-work-areas/ae-waiting-times-and-activity/ae-attendances-and-emergency-admissions-2024-25/ [Accessed: 7 Nov 2024]

NHS England (no date) Statistics» Critical care and General & Acute Beds – Urgent and Emergency Care Daily Situation Reports. [online] Available at: https://www.england.nhs.uk/statistics/statistical-work-areas/bed-availability-and-occupancy/critical-care-and-general-acute-beds-urgent-and-emergency-care-daily-situation-reports/. [Accessed 8 Jan. 2025].


Sky News. (2025). Several NHS trusts in England declare critical incidents amid rise in flu cases. [online] Available at: https://news.sky.com/story/hospital-declares-critical-incident-over-exceptionally-high-demand-on-aande-amid-rising-flu-cases-13284853  [Accessed 8 Jan. 2025].

