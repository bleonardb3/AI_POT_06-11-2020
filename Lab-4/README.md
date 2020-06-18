# Introduction

This lab will introduce users to the use of IBM&#39;s predictive analytics and decision optimization technologies to solve COVID-19 problems. The coronavirus has infected millions people leading to sever illness symptoms resulting in hundreds of thousands deaths. During the initial outbreak not all areas where effected the same. Hospitals located in COVID-19 epidemic outbreak locations were overwhelmed with sick and dying patients. This lab will apply predictive analytics to analyze different factors among people to predict future COVID-19 infection rates in an area. Based on areas predicted to have high COVID-19 infections – this lab will apply optimization techniques to optimize the planning of transferring COVID-19 patients from hospitals located in epidemic areas to hospitals with less COVID-19 patients. Our hope is to educate people who are involved in the COVID-19 response decision-making process, in applying IBM&#39;s predictive and optimization technologies to help them improve planning and responding to next wave of COVID-19 cases.

## Planning the transfers

These days, one of the critical problem for our governments is to manage the heterogeneous propagation of the virus over the territories. Not all areas are infected at the same level. While confinement of the population and prohibition of travel is reducing the propagation of the virus, the transfer of sick people between different areas can reduce the stress of hospitals in the most critical regions. In France, transfers are now generalized, using military planes (see here or here), helicopters or high-speed trains, but also locally with ambulances.

The problem of finding optimal ways to relocate sick people among areas can benefit from a combination of Machine Learning (ML) and Decision Optimization (DO).

Based on the recent evolution of the number of critical reanimation cases in each area, predictive models can be trained to forecast the evolution per area on the coming days. This data, in addition to the capacity of the hospitals for each of the areas and some description of the constraints applying on the possible transfers can then be used in a decision optimization model. This schema where Machine Learning is used first to extract additional unknown information and Decision Optimization is used to prescribe the best next actions is very common.

# Objectives

The goal of this lab is to educate user on how to apply IBM predictive analytics and optimization tools to different applications of COVID-19 like (1) predicting future infections and (2) optimizing response for better decision making. Students will use Watson Studio to load and step through a notebook that applies Decision Optimization to optimize the transport of affected people between hospitals to avoid being overcapacity. Working through this notebook – we intend students learn these skills.

1. Learn how to load data form different places (departments, current situation, etc) to be used for analysis.
2. Learn how to represent the current situation on a map using folium. folium makes it easy to visualize data that&#39;s been manipulated in Python on an interactive leaflet map.
3. Learn how to use a LinearRegression from sklearn to predict new cases to come for each department.
4. Learn how to use Decision Optimization to model and optimize plan transfers.
5. Learn how to use folium to display the optimized future patient transfers plan i.e. all the transfers from the solution.

# Example notebook

You can find [here an example notebook](https://dataplatform.cloud.ibm.com/analytics/notebooks/v2/1cea8b5b-1a50-4061-9257-751cee3d75bd/view?access_token=905e3d02e5ff645df5189b9b4010b072ae0d57befbdaea15f135590a0cfe82fc). This is a very simplistic prototype to illustrate how both technologies could collaborate, but a real solution would require a subject matter expert to ensure that the right data is used, and the right constraints and objectives are taken into account. This model formulation is very standard process in Decision Optimization projects and can take from days to months, depending on the complexity of the problem.

## **Import and display input data**

We use data from the French government [data.gouv.fr](https://www.data.gouv.fr/fr/)site: [https://www.data.gouv.fr/fr/datasets/donnees-relatives-a-lepidemie-de-covid-19/#\_](https://www.data.gouv.fr/fr/datasets/donnees-relatives-a-lepidemie-de-covid-19/#_) in addition to some imported data on the different French administrative &quot;departments&quot; (GPS coordinates of frontier and center).

Using Folium, we can easily represent this data on a map. The circles show the number of reanimation cases in each department. Red circles show above normal capacity.

![](RackMultipart20200618-4-z9fy1x_html_bd69e0763d83d141.png)

Current situation

## **Simplistic predictive model**

![](RackMultipart20200618-4-z9fy1x_html_1796fdb038d13c85.gif)A very simplistic predictive model is trained for illustration. We use LinearRegression from sklearn, knowing obviously that the epidemy is not linear at all. But this gives an idea of how ML would work here.

import numpy as np
from sklearn.linear\_model import LinearRegressionNB\_PERIODS = 3def predict\_more(d, n):
 X = hosp\_data[d].index.tolist()
 X.reverse()
 X=np.array(X).reshape(-1,1)
 y = hosp\_data[d][&#39;rea&#39;].tolist()
 y.reverse()
 y=np.array(y).reshape(-1,1)regsr=LinearRegression()
 regsr.fit(X,y)to\_predict\_x = [i for i in range(len(X), len(X)+n)]
 to\_predict\_x= np.array(to\_predict\_x).reshape(-1,1)
 predicted\_y= regsr.predict(to\_predict\_x)
 delta\_y = [int(round(max(0, predicted\_y[i][0]-y[len(y)-1][0]))) for i in range(n)]
 return delta\_ynew\_rea ={ d:predict\_more(d, NB\_PERIODS) for d in deps}
print (new\_rea)

The outcome is a prediction of the expected number of new reanimation cases to occur in the next few days we will plan (here NB\_PERIODS = 3).

## Sample Decision Optimization model

Our hypothesis for the optimization model is that two different types of transfers can be done:

- long distance transfers (planes, trains) with the number of transfers limited over the whole country, several people can be transferred at a time.
- short distance transfers (ambulances) with the number of transfers limited per area, and with just one person at a time.

![](RackMultipart20200618-4-z9fy1x_html_6f8eef370bff7a1c.gif)The parameters we use (completely invented) are:

MAX\_NB\_LONG\_TRANSFERS\_PER\_PERIOD = 3
MAX\_CASES\_PER\_LONG\_TRANSFERS = 20

MAX\_NB\_SHORT\_TRANSFERS\_PER\_DEPARTMENT = 3

LONG\_DISTANCE = 200

![](RackMultipart20200618-4-z9fy1x_html_ce2c89ba860a3a6b.gif)We use our docplexAPI to model the problem:

from docplex.mp.model import Model
mdl = Model(&quot;Plan Transfers&quot;)

![](RackMultipart20200618-4-z9fy1x_html_8ae41d852e0f3775.gif)The decision variables represent the transfer links to be used in the best (optimal) transfer plan (an integer variable for the number of persons transferred and a binary variable indicating whether the link is used or not, with a constraint linking both). Another set of auxiliary variables represents the occupancy for each department and each period.

use\_link\_vars = mdl.binary\_var\_cube(deps, deps, transfer\_periods, name=&quot;use\_link&quot;)
link\_vars = mdl.integer\_var\_cube(deps, deps, transfer\_periods, lb=0, ub=MAX\_CASES\_PER\_LONG\_TRANSFERS, name=&quot;link&quot;)
occupancy\_vars = mdl.integer\_var\_matrix(deps, all\_periods, lb=0, name=&quot;occupancy&quot;)

![](RackMultipart20200618-4-z9fy1x_html_b0d34de0a116149c.gif)Then we formulate all the constraints:

# Initial state
mdl.add\_constraints(occupancy\_vars[d, 0] == initial[d] for d in deps)

# structural constraint between user\_link and link
mdl.add\_constraints(use\_link\_vars[d, d1, t] == (link\_vars[d, d1, t] \&gt;= 1) for d in deps for d1 in deps for t in transfer\_periods)

# Short transfers bounds
mdl.add\_constraints(link\_vars[d1, d2, t] \&lt;= 1 for d1 in deps for d2 in deps if not is\_long[d1][d2] for t in transfer\_periods)

# number of transfers from a department less than current number of cases
mdl.add\_constraints(mdl.sum(link\_vars[d, d1, t] for d1 in deps) \&lt;= occupancy\_vars[d, t] for d in deps for t in transfer\_periods)

# maximum number of LONG transfers
mdl.add\_constraints(mdl.sum(use\_link\_vars[d1, d2, t] for d1 in deps for d2 in deps if is\_long[d1][d2]) \&lt;= MAX\_NB\_LONG\_TRANSFERS\_PER\_PERIOD for t in transfer\_periods)

# maximum number of SHORT transfers
mdl.add\_constraints(mdl.sum(use\_link\_vars[d1, d2, t] for d1 in deps if not is\_long[d1][d2] for t in transfer\_periods) \&lt;= MAX\_NB\_SHORT\_TRANSFERS\_PER\_DEPARTMENT for d2 in deps )

# conservation constraints including new cases to come
mdl.add\_constraints(occupancy\_vars[d, t+1] == new\_rea[d][t] + occupancy\_vars[d, t] + mdl.sum(link\_vars[d1, d, t] for d1 in deps) - mdl.sum(link\_vars[d, d1, t] for d1 in deps) for d in deps for t in transfer\_periods)

![](RackMultipart20200618-4-z9fy1x_html_731bf3cd34a6168e.gif)And the objectives. The main objective is to reduce the total overcapacity on the areas, but we should also limit unnecessary transfers.

final\_overcapacity = mdl.sum(mdl.max(0, occupancy\_vars[d, NB\_PERIODS] - capacity[d]) for d in deps) mdl.add\_kpi(final\_overcapacity)

nb\_long\_transfers = mdl.sum(use\_link\_vars[d1, d2, t] for d1 in deps for d2 in deps if is\_long[d1][d2] for t in transfer\_periods)
mdl.add\_kpi(nb\_long\_transfers)

![](RackMultipart20200618-4-z9fy1x_html_e6f6f596082e4016.gif)nb\_short\_transfers = mdl.sum(use\_link\_vars[d1, d2, t] for d1 in deps for d2 in deps if not is\_long[d1][d2] for t in transfer\_periods)
mdl.add\_kpi(nb\_short\_transfers)

mdl.minimize(1000 \* final\_overcapacity + 10 \* nb\_long\_transfers + nb\_short\_transfers)

![](RackMultipart20200618-4-z9fy1x_html_ea618318a01465ee.gif)We can then solve the problem (using the CPLEX engine) and get the prescribed decisions.

mdl.set\_time\_limit(20)
mdl.solve(log\_output=True)

## Display the solution

Using the same Folium package, we can represent the proposed transfers on a map.
 The long-distance transfers are represented in green and the short distance ones in black.

![](RackMultipart20200618-4-z9fy1x_html_c5b0cfba072ba713.png)

Proposed solution.

## Conclusions

I want to insist again on the fact this is not a ready-to-use solution, but a demonstration of how ML and DO technology could be used to propose mathematically optimal transfer decisions.

You can find [here the complete notebook](https://dataplatform.cloud.ibm.com/analytics/notebooks/v2/1cea8b5b-1a50-4061-9257-751cee3d75bd/view?access_token=905e3d02e5ff645df5189b9b4010b072ae0d57befbdaea15f135590a0cfe82fc).

In practice, additional data, constraints and objectives should be taken into account. For example, we can easily imagine that the level of criticality of individual infected persons would be taken into account. Also the characteristics of hospitals in the different areas, including the neighborhood of the airport or train station, should be taken into account.

Note also my model only considered transfers within France, while some transfers already took place from France to Germany and Luxembourg.

**This notebook is hence only provided as an example which can be used to discover or learn optimization. ** You can freely try this notebook on [Watson Studio Cloud](https://dataplatform.cloud.ibm.com/home?context=wdp).

If you are in a position to use such technology in practice for the Covid-19 situation, don&#39;t hesitate to contact us, we are willing to help.

**IBM USA**

Lee Angelelli

Executive Client Architect

IBM Global Markets

[langelel@us.ibm.com](mailto:langelel@us.ibm.com)

**IBM France**

Alain Chabrier

STSM - Decision Optimization

IBM Cloud and Cognitive Software

[Alain.chabrier@ibm.com](mailto:Alain.chabrier@ibm.com)

# Instructions

![](RackMultipart20200618-4-z9fy1x_html_dbba778feb0cae76.gif)

| © Copyright IBM Corp. 2020 |
 | 3 |
| --- | --- | --- |
