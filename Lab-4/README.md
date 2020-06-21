Introduction
============

This lab will introduce users to the use of IBM\'s predictive analytics
and decision optimization technologies to solve COVID-19 problems. The
coronavirus has infected millions people leading to sever illness
symptoms resulting in hundreds of thousands deaths. During the initial
outbreak not all areas where effected the same. Hospitals located in
COVID-19 epidemic outbreak locations were overwhelmed with sick and
dying patients. This lab will apply predictive analytics to analyze
different factors among people to predict future COVID-19 infection
rates in an area. Based on areas predicted to have high COVID-19
infections -- this lab will apply optimization techniques to optimize
the planning of transferring COVID-19 patients from hospitals located in
epidemic areas to hospitals with less COVID-19 patients. Our hope is to
educate people who are involved in the COVID-19 response decision-making
process, in applying IBM\'s predictive and optimization technologies to
help them improve planning and responding to next wave of COVID-19
cases.

Objectives
==========

The goal of this lab is to educate user on how to apply IBM predictive
analytics and optimization tools to different applications of COVID-19
like (1) predicting future infections and (2) optimizing response for
better decision making. Students will use Watson Studio to load and step
through a notebook that applies Decision Optimization to optimize the
transport of affected people between hospitals to avoid being
overcapacity. Working through this notebook -- we intend students learn
these skills.

1.  Learn how to load data form different places (departments, current
    situation, etc) to be used for analysis.

2.  Learn how to represent the current situation on a map using folium.
    folium makes it easy to visualize data that\'s been manipulated in
    Python on an interactive leaflet map. 

3.  Learn how to use a LinearRegression from sklearn to predict new
    cases to come for each department.

4.  Learn how to use Decision Optimization to model and optimize plan
    transfers.

5.  Learn how to use folium to display the optimized future patient
    transfers plan i.e. all the transfers from the solution.

Prerequisites
=============

1.  Open a web browser and enter this URL \>
    <https://dataplatform.cloud.ibm.com/>

2.  If you already have a Watson Studio account please \"Log In\" and
    skip to section \" Create a Watson Studio project and set up the
    required services\".

3.  Else click \"Sign Up\" to create a Watson studio account.

![A screenshot of a cell phone screen with text Description
automatically generated](media/image1.png){width="6.5in" height="2.7in"}

4.  Pick an IBM Cloud region near you. Enter your email address as your
    user account. Click on \"Accept\" terms & conditions. Click \"Next\"
    button.

![A screenshot of a cell phone Description automatically
generated](media/image2.png){width="6.5in" height="4.35625in"}

5.  Enter a password \> click \"Next\" button.

![A screenshot of a cell phone Description automatically
generated](media/image3.png){width="3.388888888888889in"
height="5.231968503937008in"}

6.  Verify your account. Enter the verification code sent to your email
    address. Click \"Next\" button.

![A screenshot of a cell phone Description automatically
generated](media/image4.png){width="3.9444444444444446in"
height="3.02127624671916in"}

7.  Enter your personal information. Click \"Next\" button.

![A screenshot of a cell phone Description automatically
generated](media/image5.png){width="4.819444444444445in"
height="4.80090769903762in"}

8.  Please indicate if and how IBM can keep you informed on IBM\'s
    products services. Click \"Create account\" button.

![A screenshot of a social media post Description automatically
generated](media/image6.png){width="4.652777777777778in"
height="2.956700568678915in"}

Note: Users will see an account progress bar. Click \"Login\" button to
login to your account.

![A picture containing clock Description automatically
generated](media/image7.png){width="3.5416666666666665in"
height="3.413417541557305in"}

Create a Watson Studio project and set up the required services.
----------------------------------------------------------------

1.  Enter the Watson Studio URL in a web browser -\>
    <https://dataplatform.cloud.ibm.com/>. Login to your Watson Studio
    account.

2.  Click \"Get Started\" button.

> ![A screenshot of a cell phone Description automatically
> generated](media/image8.png){width="4.166666666666667in"
> height="2.7412751531058617in"}

3.  Click \"Maybe later\" for the introduction tour.

![A screenshot of a cell phone Description automatically
generated](media/image9.png){width="6.5in"
height="2.6069444444444443in"}

4.  Click \"Create a project\" button.

![A screenshot of a cell phone Description automatically
generated](media/image10.png){width="6.5in"
height="2.6951388888888888in"}

5.  Click on \"Create an empty project\".

![A screenshot of a cell phone Description automatically
generated](media/image11.png){width="4.930555555555555in"
height="2.251936789151356in"}

6.  Enter a project name; e.g. \"COVID-19 Decision Making\". Enter a
    description (optional) \" This project will apply predictive
    analytics and optimization techniques to predict COVID-19 infections
    in areas and optimize response to transfer COVID-19 patients to less
    occupied COVIDD-19 hospitals.\"

7.  Check \"Restrict who can be a collaborator\".

8.  If you already have an Object Storage\" instance -- please select it
    from the \"Select storage service\" selection box. Click \"Create\"
    button. Next, proceed to the section \"Adding a Machine Learning
    Service\" below.

![A screenshot of a cell phone Description automatically
generated](media/image12.png){width="6.5in" height="3.025in"}

9.  Else click on the \"Cloud Object Storage\" URL.

![A screenshot of a social media post Description automatically
generated](media/image13.png){width="3.6666666666666665in"
height="3.1040069991251094in"}

10. Click the \"Lite\" plan. Enter a service name for your \"Object
    Storage\' service. Click \"Create\" button.

![A screenshot of a social media post Description automatically
generated](media/image14.png){width="6.5in"
height="4.498611111111111in"}

11. Select your \"Cloud Object Storage\" service name from the \"Select
    storage service\" selection box. Click \"Create\" button.

![A screenshot of a cell phone Description automatically
generated](media/image12.png){width="6.5in" height="3.025in"}

Adding a Machine Learning Service
---------------------------------

1.  Click on the project Settings tab.

![A screenshot of a cell phone Description automatically
generated](media/image15.png){width="6.5in"
height="2.5881944444444445in"}

2.  Scroll down to Associated Services, then select Add service and
    select Watson.

![A screenshot of a cell phone Description automatically
generated](media/image16.png){width="6.5in"
height="2.484722222222222in"}

3.  Select the Machine Learning service.

![A screenshot of a cell phone Description automatically
generated](media/image17.png){width="6.5in"
height="2.5833333333333335in"}

4.  Select New.

![A screenshot of a cell phone Description automatically
generated](media/image18.png){width="5.652777777777778in"
height="3.134996719160105in"}

5.  Select the Lite plan.

![A screenshot of a social media post Description automatically
generated](media/image19.png){width="6.5in"
height="2.8048611111111112in"}

6.  Scroll down and click Create, then change the Service name to
    Machine Learning in the Confirm Creation panel and click Confirm.

![A screenshot of a social media post Description automatically
generated](media/image20.png){width="6.5in"
height="2.8041666666666667in"}

7.  The Machine Learning service that you created should now appear in
    Associated Services.

![](media/image21.png){width="6.5in" height="0.55625in"}

Instructions
============

Please click on the link below to download the instructions to your
machine.

Instructions.
