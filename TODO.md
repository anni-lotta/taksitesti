# TODO

> Edit this file to add your answers to each TODO item. It's okay to link to external documentation as well. But ensure that our reviewers have access.
> 
> When you're ready to submit, click **Commit changes...** and save your responses to a new branch titled 'implementation' and create a pull request.
>
> Before you start reading, please configure your environment as described in your GitHub repository's "Before you get started" README section.*

In this task, you will work with [this dataset about a fictional company called InsightEdge](https://console.cloud.google.com/bigquery?ws=!1m4!1m3!3m2!1salva-devskills!2sinsightedge_challange_dataset) to figure out some insights about the company's business.

Here's some useful information about this dataset:

- [Revenue Model](revenue-model.md)
- [Cost Model](cost-model.md)
- [Entity Relationship Diagram](erd.md)

Feel free to explore the dataset and try different queries using the BigQuery interface.

Once you have a query that answers the question, add the code snippet to the specific question and any supportive reasoning/comments you may have.

## Task 1

How much revenue was earned during March 2025?

>SELECT ROUND(SUM(amount), 2) AS question1
>
>FROM `alva-devskills.insightedge_challange_dataset.revenue`
>
>WHERE EXTRACT(YEAR FROM date) = 2025
>
>AND EXTRACT(MONTH from date) = 03

>> 60659.16

## Task 2

How many customers generated revenue in January 2025?

>SELECT COUNT(DISTINCT customer_id) AS question2
>
>FROM `alva-devskills.insightedge_challange_dataset.revenue`
>
>WHERE EXTRACT(YEAR FROM date) = 2025
>
>AND EXTRACT(MONTH from date) = 01

>> 48

## Task 3

Rank the customers by the total amount of revenue they have generated, and return the top 3 customer names (1 being highest).

WITH data AS (
    SELECT
        DISTINCT b.name AS name,
        ROUND(SUM(a.amount),2) AS revenue
    FROM `alva-devskills.insightedge_challange_dataset.revenue` AS a
    LEFT JOIN `alva-devskills.insightedge_challange_dataset.customers` AS b
    ON a.customer_id = b.customer_id
    GROUP BY name
    ORDER BY revenue DESC
    LIMIT 3
)
    SELECT name AS question3
    FROM data
    
>> 
1	WhiteOakSolutions
2	FalconEye
3	SummitWorks

## Task 4

Which month’s leads had the highest customer conversion rate? What was the Month, Year, and what was the Conversion rate?

WITH data AS (
  SELECT 
    a.lead_id as denominator, 
    a.entered_sales_funnel_at AS date, 
    b.customer_id AS numerator
 FROM `alva-devskills.insightedge_challange_dataset.leads` AS a 
 LEFT JOIN `alva-devskills.insightedge_challange_dataset.leads_customers` AS b 
 ON a.lead_id = b.lead_id
)
  SELECT 
    DISTINCT FORMAT_DATE('%b %Y',date) AS question4,
    ROUND(COUNT(numerator)/COUNT(denominator),2) AS conversionRate,
  FROM data
  GROUP BY question4
  ORDER BY conversionRate DESC
  LIMIT 1
  
>> Jul 2024 0.25

---

## Task 5

Considering your response to the previous question (Task 4), present arguments both in favor of and against qualifying it as a 'good insight'.

WITH data AS (
  SELECT 
    a.lead_id as denominator, 
    a.entered_sales_funnel_at AS date, 
    b.customer_id AS numerator
 FROM `alva-devskills.insightedge_challange_dataset.leads` AS a 
 LEFT JOIN `alva-devskills.insightedge_challange_dataset.leads_customers` AS b 
 ON a.lead_id = b.lead_id
)
  SELECT 
    DISTINCT FORMAT_DATE('%Y %m',date) AS question4,
    ROUND(COUNT(numerator)/COUNT(denominator),2) AS conversionRate,
    COUNT(numerator) AS numerator,
    COUNT(denominator) AS denominator
  FROM data
  GROUP BY question4
  ORDER BY question4 ASC

Python code:
import statsmodels.api as sm
import numpy as np

# Question 4 data
numerators = [2, 0, 0, 0, 3, 9, 10, 10, 12, 22, 10, 15, 30, 38, 34, 75, 30, 29]
denominators = [10, 11, 5, 1, 25, 48, 40, 76, 100, 138, 125, 110, 197, 222, 205, 414, 200, 187]

# Confidence level
confidence_level = 0.95

# Calculate confidence intervals
for num, den in zip(numerators, denominators):
    # Use Wilson Score Interval
    conf_int = sm.stats.proportion_confint(num, den, alpha=1-confidence_level, method='wilson')
    print(f"Numerator: {num}, Denominator: {den}, CI: {conf_int}")
    
>>Conversion rate of 0.25 where n = 40 
Pros:
    - 40 observations is enough for a proportion sample
    - at 95% confidence level for the test group proportion to be 25% at the power of 80%, a sample size needs to be minimum 12
    - Second highest low bound suggests that the value even in worst scenarios would be one of the best
Cons: 
    - Tinier sample size before a change in the leads trend
    - July 2024 has the second highest standard error of proportion in the observed months
    - Comparatively moderately sized Wilson Score confidence interval compared to the later proportions with larger samples

## Task 6

Is there any additional data that you believe would enhance the depth of your analysis?

--sales data
WITH data_a AS (
  SELECT
    level_1 AS revSource,
    level_2 AS product,
    ROUND(SUM(amount),2) AS revenue,
    COUNT(customer_id) AS purchases,
    COUNT(DISTINCT customer_id) AS buyers,
    ROUND(1-(COUNT(DISTINCT customer_id)/COUNT(customer_id)),2) AS repeat_purchasers
  FROM `alva-devskills.insightedge_challange_dataset.revenue` 
  WHERE EXTRACT(YEAR FROM date) = 2025
  GROUP BY product, revSource
),
data_b AS (
  SELECT
    level_1 AS revSource,
    "total" AS product,
    ROUND(SUM(amount),2) AS revenue,
    COUNT(customer_id) AS purchases,
    COUNT(DISTINCT customer_id) AS buyers,
    ROUND(1-(COUNT(DISTINCT customer_id)/COUNT(customer_id)),2) AS repeat_purchasers
  FROM `alva-devskills.insightedge_challange_dataset.revenue` 
  WHERE EXTRACT(YEAR FROM date) = 2025
  GROUP BY revSource, product
)
SELECT * FROM data_a
UNION DISTINCT
SELECT * FROM data_b
ORDER BY revSource, product DESC

-- cost data
SELECT  
  DISTINCT level_1 AS costDriver,
  level_2 AS costDrilldown,
  date,
  CASE 
    WHEN amount = '#REF!' THEN '9500'
    ELSE amount
  END AS cost
  --SUM(CAST(amount AS float64)) AS cost
FROM `alva-devskills.insightedge_challange_dataset.cost` 
WHERE EXTRACT(YEAR FROM date) = 2025
--GROUP BY 1,2
--ORDER BY 1,2

-- employee counts per department
SELECT 
  functional_group,
  COUNT(employee_id) AS employee_count
FROM `alva-devskills.insightedge_challange_dataset.employees`
WHERE end_date IS NULL
GROUP BY functional_group

## Task 7

If you were to make a single recommendation to the company, what would it be?

> The bulk of assumed salary expenses (based on department size) come from consulting, yet there are more sales made in 2025 on the software side of the business.
If we assume that the senior level consultants get paid the most yet drive the least in sales value, maybe the size of the department should be thought through.

---

#### Now assume that it's July 1, 2025 (2025-07-01).

## Task 9

The VP of Sales approaches you with a request: "Can you provide an estimate for the number of customers we can expect this month?" How do you reply?

Python code for ARIMA:
import pandas as pd
import numpy as np
from statsmodels.tsa.arima.model import ARIMA
from statsmodels.tsa.stattools import adfuller
from statsmodels.graphics.tsaplots import plot_acf, plot_pacf
import matplotlib.pyplot as plt

# Data preparation with frequency specified:
data = {
    'Date': pd.to_datetime(['2024-08-31', '2024-09-30', '2024-10-31', '2024-11-30', '2024-12-31',
                            '2025-01-31', '2025-02-28', '2025-03-31', '2025-04-30', '2025-05-31', '2025-06-30']),
    'Buyers': [14, 23, 26, 33, 40, 48, 48, 71, 101, 128, 124]
}
df = pd.DataFrame(data)
df['Date'] = pd.to_datetime(df['Date'])
df = df.set_index('Date')
df = df.asfreq('ME')

# Example ARIMA model (You'll need to determine the appropriate (p,d,q) from ACF/PACF)
model = ARIMA(df['Buyers'], order=(1, 0, 1)) # Example order - adjust based on ACF/PACF.
model_fit = model.fit()
print(model_fit.summary())

#Corrected Forecast and plotting
forecast = model_fit.get_forecast(steps=1)
conf_int = forecast.conf_int()
print("Forecast for next month:", forecast.predicted_mean.iloc[-1])

>> ~113
[Looker_Studio_‐raportit_-_12.12.2024_klo_8.59 (3).pdf](https://github.com/user-attachments/files/18110941/Looker_Studio_.raportit_-_12.12.2024_klo_8.59.3.pdf)

## Task 10

The CEO asks for an update on the performance of the 'AI Insight' product. How would you respond to this inquiry? (text/media input)

It has gained a nice pie of the individual sales counts since launch, but that does not transfer into increased revenues. To make more suggestions I'd need the product be more well-defined. 
[Looker_Studio_‐raportit_-_12.12.2024_klo_8.59 (4).pdf](https://github.com/user-attachments/files/18110947/Looker_Studio_.raportit_-_12.12.2024_klo_8.59.4.pdf)


## Task 11

The CEO asks you, "What is your assessment of our current performance?" How would you formulate your response?

The number of sales and the revenue they've yielded has grown nicely in the past 12 operating months. The platform itself is driving the most revenue and maybe we should consider the size and shape of the organization to ensure we use our resources effectively. Consulting is creating a nice chunk of revenue as well but I'd be interested to see how much of the salary expenses go to keeping 27 consultants on board. The first 2 quarters of 2025 have not been profitable but as long as we keep our finances in order, there is potential for the business to grow in sales volume.
