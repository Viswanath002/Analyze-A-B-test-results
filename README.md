# Analyze-A-B-test-results
A/B tests are very commonly performed by data analysts and data scientists. It is important that you get some practice working with the difficulties of these  For this project, you will be working to understand the results of an A/B test run by an e-commerce website. Your goal is to work through this notebook to help the company understand if they should implement the new page, keep the old page, or perhaps run the experiment longer to make their decision.
Part I - Probability
To get started, let's import our libraries.


import pandas as pd
import numpy as np
import random
import matplotlib.pyplot as plt
%matplotlib inline
#We are setting the seed to assure you get the same answers on quizzes as we set up
random.seed(42)
1. Now, read in the ab_data.csv data. Store it in df. Use your dataframe to answer the questions 

a. Read in the dataset and take a look at the top few rows here:

Reading the dataset in python

df = pd.read_csv('ab_data.csv')
b. Use the cell below to find the number of rows in the dataset.


df.shape
(294478, 5)

df.head()
user_id	timestamp	group	landing_page	converted
0	851104	2017-01-21 22:11:48.556739	control	old_page	0
1	804228	2017-01-12 08:01:45.159739	control	old_page	0
2	661590	2017-01-11 16:55:06.154213	treatment	new_page	0
3	853541	2017-01-08 18:28:03.143765	treatment	new_page	0
4	864975	2017-01-21 01:52:26.210827	control	old_page	1
c. The number of unique users in the dataset.


unique = df['user_id'].nunique()
d. The proportion of users converted.


df['converted'].mean()
0.11965919355605512
e. The number of times the new_page and treatment don't match.


df2 = df.query("(group == 'control' and landing_page == 'new_page') or (group == 'treatment' and landing_page == 'old_page')") 
df2.shape[0]
3893
f. Do any of the rows have missing values?


df.info()
df.isnull().sum()
<class 'pandas.core.frame.DataFrame'>
RangeIndex: 294478 entries, 0 to 294477
Data columns (total 5 columns):
user_id         294478 non-null int64
timestamp       294478 non-null object
group           294478 non-null object
landing_page    294478 non-null object
converted       294478 non-null int64
dtypes: int64(2), object(3)
memory usage: 11.2+ MB
user_id         0
timestamp       0
group           0
landing_page    0
converted       0
dtype: int64
2. For the rows where treatment does not match with new_page or control does not match with old_page, we cannot be sure if this row truly received the new or old page. Use Quiz 2 in the classroom to figure out how we should handle these rows.

a. Now use the answer to the quiz to create a new dataset that meets the specifications from the quiz. Store your new dataframe in df2.


df2 = df.query("(group == 'control' and landing_page == 'old_page') or (group == 'treatment' and landing_page == 'new_page')")

# Double Check all of the correct rows were removed 
df2[((df2['group'] == 'treatment') == (df2['landing_page'] == 'new_page')) == False].shape[0]
0
3. Use df2 and the cells below to answer questions for Quiz3 in the classroom.

a. How many unique user_ids are in df2?


df2['user_id'].nunique()
290584
b. There is one user_id repeated in df2. What is it?


​
df2[df2.duplicated(['user_id'])]['user_id'].unique()
array([773192])
c. What is the row information for the repeat user_id?


df2[df2.duplicated(['user_id'], keep=False)]
user_id	timestamp	group	landing_page	converted
1899	773192	2017-01-09 05:37:58.781806	treatment	new_page	0
2893	773192	2017-01-14 02:55:59.590927	treatment	new_page	0
d. Remove one of the rows with a duplicate user_id, but keep your dataframe as df2.


df2 = df2.drop_duplicates(['user_id'], keep='first')
4. Use df2 in the cells below to answer the quiz questions related to Quiz 4 in the classroom.

a. What is the probability of an individual converting regardless of the page they receive?


df2['converted'].mean()
0.11959708724499628
b. Given that an individual was in the control group, what is the probability they converted?


df2[df2['group'] == 'control']['converted'].mean()
0.1203863045004612
c. Given that an individual was in the treatment group, what is the probability they converted?


df2[df2['group'] == 'treatment']['converted'].mean()
0.11880806551510564
d. What is the probability that an individual received the new page?


len(df2.query("landing_page == 'new_page'")) / df2.shape[0]
0.5000619442226688
e. Consider your results from parts (a) through (d) above, and explain below whether you think there is sufficient evidence to conclude that the new treatment page leads to more conversions.

As the probability of convertion for treatment is less than that of control group, so there is not sufficient evidence to say that the treatment page leads to more conversions


Part II - A/B Test
Notice that because of the time stamp associated with each event, you could technically run a hypothesis test continuously as each observation was observed.

However, then the hard question is do you stop as soon as one page is considered significantly better than another or does it need to happen consistently for a certain amount of time? How long do you run to render a decision that neither page is better than another?

These questions are the difficult parts associated with A/B tests in general.

1. For now, consider you need to make the decision just based on all the data provided. If you want to assume that the old page is better unless the new page proves to be definitely better at a Type I error rate of 5%, what should your null and alternative hypotheses be? You can state your hypothesis in terms of words or in terms of 𝑝𝑜𝑙𝑑 and 𝑝𝑛𝑒𝑤, which are the converted rates for the old and new pages.

Ho: Pnew - Pold <= 0 - null hypothesis H1: Pnew - Pold > 0 - alternate hypothesis

2. Assume under the null hypothesis, 𝑝𝑛𝑒𝑤 and 𝑝𝑜𝑙𝑑 both have "true" success rates equal to the converted success rate regardless of page - that is 𝑝𝑛𝑒𝑤 and 𝑝𝑜𝑙𝑑 are equal. Furthermore, assume they are equal to the converted rate in ab_data.csv regardless of the page. 


Use a sample size for each page equal to the ones in ab_data.csv. 


Perform the sampling distribution for the difference in converted between the two pages over 10,000 iterations of calculating an estimate from the null. 


Use the cells below to provide the necessary parts of this simulation. If this doesn't make complete sense right now, don't worry - you are going to work through the problems below to complete this problem. You can use Quiz 5 in the classroom to make sure you are on the right track.


a. What is the conversion rate for 𝑝𝑛𝑒𝑤 under the null?


p_new = df2['converted'].mean()
p_new
0.11959708724499628
b. What is the conversion rate for 𝑝𝑜𝑙𝑑 under the null? 



p_old = df2['converted'].mean()
p_old
0.11959708724499628
c. What is 𝑛𝑛𝑒𝑤, the number of individuals in the treatment group?


n_new = df2[df2['group'] == 'treatment'].shape[0]
n_new
145310
d. What is 𝑛𝑜𝑙𝑑, the number of individuals in the control group?


n_old = df2[df2['group'] == 'control'].shape[0]
n_old
145274
e. Simulate 𝑛𝑛𝑒𝑤 transactions with a conversion rate of 𝑝𝑛𝑒𝑤 under the null. Store these 𝑛𝑛𝑒𝑤 1's and 0's in new_page_converted.


new_page_converted = np.random.binomial(n_new,p_new)
f. Simulate 𝑛𝑜𝑙𝑑 transactions with a conversion rate of 𝑝𝑜𝑙𝑑 under the null. Store these 𝑛𝑜𝑙𝑑 1's and 0's in old_page_converted.


old_page_converted = np.random.binomial(n_old,p_old)
g. Find 𝑝𝑛𝑒𝑤 - 𝑝𝑜𝑙𝑑 for your simulated values from part (e) and (f).


new_page_converted/n_new - old_page_converted/n_old
0.0005346066160412666
h. Create 10,000 𝑝𝑛𝑒𝑤 - 𝑝𝑜𝑙𝑑 values using the same simulation process you used in parts (a) through (g) above. Store all 10,000 values in a NumPy array called p_diffs.


p_diffs = []
for _ in range(10000):
    new_page_converted = np.random.binomial(n_new,p_new)
    old_page_converted = np.random.binomial(n_old, p_old)
    diff = new_page_converted/n_new - old_page_converted/n_old
    p_diffs.append(diff)
i. Plot a histogram of the p_diffs. Does this plot look like what you expected? Use the matching problem in the classroom to assure you fully understand what was computed here.


plt.hist(p_diffs)
(array([   54.,   254.,   976.,  2075.,  2710.,  2245.,  1238.,   357.,
           80.,    11.]),
 array([-0.00392514, -0.00307581, -0.00222648, -0.00137715, -0.00052782,
         0.00032151,  0.00117084,  0.00202017,  0.0028695 ,  0.00371883,
         0.00456816]),
 <a list of 10 Patch objects>)

j. What proportion of the p_diffs are greater than the actual difference observed in ab_data.csv?


act_diff = df2[df2['group'] == 'treatment']['converted'].mean() -  df2[df2['group'] == 'control']['converted'].mean()
print(act_diff)
p_diffs = np.array(p_diffs)
(act_diff < p_diffs).mean()
-0.00157823898536
0.90410000000000001
k. Please explain using the vocabulary you've learned in this course what you just computed in part j. What is this value called in scientific studies? What does this value mean in terms of whether or not there is a difference between the new and old pages?

1.This is pvalue. 2.A pvalue is the probability of getting the results you did (or more extreme results) given that the null hypothesis is true.

so a very large p-value indicates weak evidence against the null hypothesis, so you fail to reject the null hypothesis which suggest the new page conversion rate is higher than the old rate.

l. We could also use a built-in to achieve similar results. Though using the built-in might be easier to code, the above portions are a walkthrough of the ideas that are critical to correctly thinking about statistical significance. Fill in the below to calculate the number of conversions for each page, as well as the number of individuals who received each page. Let n_old and n_new refer the the number of rows associated with the old page and new pages, respectively.


import statsmodels.api as sm
​
convert_old = df2.query(" landing_page == 'old_page' and converted == 1").shape[0]
convert_new = df2.query(" landing_page == 'new_page' and converted == 1").shape[0]
n_old = df2[df2['group'] == 'control'].shape[0]
n_new = df2[df2['group'] == 'treatment'].shape[0]
convert_old
/opt/conda/lib/python3.6/site-packages/statsmodels/compat/pandas.py:56: FutureWarning: The pandas.core.datetools module is deprecated and will be removed in a future version. Please use the pandas.tseries module instead.
  from pandas.core import datetools
17489
m. Now use stats.proportions_ztest to compute your test statistic and p-value. Here is a helpful link on using the built in.


z_score, p_value = sm.stats.proportions_ztest([convert_old, convert_new], [n_old, n_new], alternative='smaller')
print(z_score, p_value)
1.31092419842 0.905058312759
n. What do the z-score and p-value you computed in the previous question mean for the conversion rates of the old and new pages? Do they agree with the findings in parts j. and k.?

Since the z-score of 1.31 less than the critical value of 1.64485362695, we fail to reject the null hypothesis which suggest the new page conversion rate is higher than the old rate. Since they are different,And Yes I Agree with findings in parts j. and k.


Part III - A regression approach
1. In this final part, you will see that the result you achieved in the A/B test in Part II above can also be achieved by performing regression.

a. Since each row is either a conversion or no conversion, what type of regression should you be performing in this case?

Logistic Regression

b. The goal is to use statsmodels to fit the regression model you specified in part a. to see if there is a significant difference in conversion based on which page a customer receives. However, you first need to create in df2 a column for the intercept, and create a dummy variable column for which page each user received. Add an intercept column, as well as an ab_page column, which is 1 when an individual receives the treatment and 0 if control.


df2['intercept'] = 1
df2[['control','treatment']] = pd.get_dummies(df2['group'])
c. Use statsmodels to instantiate your regression model on the two columns you created in part b., then fit the model using the two columns you created in part b. to predict whether or not an individual converts.


import statsmodels.api as sm
​
logit = sm.Logit(df2['converted'],df2[['intercept' ,'treatment']])
results = logit.fit()
Optimization terminated successfully.
         Current function value: 0.366118
         Iterations 6
d. Provide the summary of your model below, and use it as necessary to answer the following questions.


results.summary()
Logit Regression Results
Dep. Variable:	converted	No. Observations:	290584
Model:	Logit	Df Residuals:	290582
Method:	MLE	Df Model:	1
Date:	Wed, 30 Jan 2019	Pseudo R-squ.:	8.077e-06
Time:	04:12:06	Log-Likelihood:	-1.0639e+05
converged:	True	LL-Null:	-1.0639e+05
LLR p-value:	0.1899
coef	std err	z	P>|z|	[0.025	0.975]
intercept	-1.9888	0.008	-246.669	0.000	-2.005	-1.973
treatment	-0.0150	0.011	-1.311	0.190	-0.037	0.007
e. What is the p-value associated with ab_page? Why does it differ from the value you found in Part II?

Hint: What are the null and alternative hypotheses associated with your regression model, and how do they compare to the null and alternative hypotheses in Part II?

The p-value (0.190) here remains above an 𝛼 level of 0.05 but is different because this is a two tailed test. We will still reject the null in this situation.

f. Now, you are considering other things that might influence whether or not an individual converts. Discuss why it is a good idea to consider other factors to add into your regression model. Are there any disadvantages to adding additional terms into your regression model?

Things like which program people applying for or age or gender of user might influence whether or not an individual converts or not to new page. Trend appears in several different groups of data but disappears or reverses when these groups are combined (Simpson's paradox).

Yes if we add high correlations predictor variables, leading to unreliable and unstable estimates of regression coefficients (Multicollinearity) can affect our model.Every time we include a new predictor variable with no change in sample size we lose a degree of freedom. The result often is that previously significant predictor in the new regression is no long significant at the same probability of a Type 1 error (the significance level).

g. Now along with testing if the conversion rate changes for different pages, also add an effect based on which country a user lives in. You will need to read in the countries.csv dataset and merge together your datasets on the appropriate rows. Here are the docs for joining tables.

Does it appear that country had an impact on conversion? Don't forget to create dummy variables for these country columns - Hint: You will need two columns for the three dummy variables. Provide the statistical output as well as a written response to answer this question.


import pandas as pd
countries_df = pd.read_csv('./countries.csv')
df_new = countries_df.set_index('user_id').join(df2.set_index('user_id'), how='inner')
df_new.head()
country	timestamp	group	landing_page	converted	intercept	control	treatment
user_id								
834778	UK	2017-01-14 23:08:43.304998	control	old_page	0	1	1	0
928468	US	2017-01-23 14:44:16.387854	treatment	new_page	0	1	0	1
822059	UK	2017-01-16 14:04:14.719771	treatment	new_page	1	1	0	1
711597	UK	2017-01-22 03:14:24.763511	control	old_page	0	1	1	0
710616	UK	2017-01-16 13:14:44.000513	treatment	new_page	0	1	0	1
h. Though you have now looked at the individual factors of country and page on conversion, we would now like to look at an interaction between page and country to see if there significant effects on conversion. Create the necessary additional columns, and fit the new model.

Provide the summary results, and your conclusions based on the results.


df_new['country'].nunique()
3

df_new[['CA', 'UK', 'US']] = pd.get_dummies(df_new['country'])
df_new.head()
country	timestamp	group	landing_page	converted	intercept	control	treatment	CA	UK	US
user_id											
834778	UK	2017-01-14 23:08:43.304998	control	old_page	0	1	1	0	0	1	0
928468	US	2017-01-23 14:44:16.387854	treatment	new_page	0	1	0	1	0	0	1
822059	UK	2017-01-16 14:04:14.719771	treatment	new_page	1	1	0	1	0	1	0
711597	UK	2017-01-22 03:14:24.763511	control	old_page	0	1	1	0	0	1	0
710616	UK	2017-01-16 13:14:44.000513	treatment	new_page	0	1	0	1	0	1	0

log_mod = sm.Logit(df_new['converted'], df_new[['intercept', 'CA', 'UK']])
result = log_mod.fit()
result.summary()
Optimization terminated successfully.
         Current function value: 0.366116
         Iterations 6
Logit Regression Results
Dep. Variable:	converted	No. Observations:	290584
Model:	Logit	Df Residuals:	290581
Method:	MLE	Df Model:	2
Date:	Wed, 30 Jan 2019	Pseudo R-squ.:	1.521e-05
Time:	04:14:14	Log-Likelihood:	-1.0639e+05
converged:	True	LL-Null:	-1.0639e+05
LLR p-value:	0.1984
coef	std err	z	P>|z|	[0.025	0.975]
intercept	-1.9967	0.007	-292.314	0.000	-2.010	-1.983
CA	-0.0408	0.027	-1.518	0.129	-0.093	0.012
UK	0.0099	0.013	0.746	0.456	-0.016	0.036
Results: Once again, the p-values for the countries are well above a 0.05 𝛼 level. And so we fail to reject the null and conclude that on it's own, there is no significant contribution from country to differences in conversion rates for the two pages.

Results: None of the variables have significant p-values. Therefore, we will fail to reject the null and conclude that there is not sufficient evidence to suggest that there is an interaction between country and page received that will predict whether a user converts or not.

In the larger picture, based on the available information, we do not have sufficient evidence to suggest that the new page results in more conversions than the old page.


from subprocess import call
call(['python', '-m', 'nbconvert', 'Analyze_ab_test_results_notebook.ipynb'])
