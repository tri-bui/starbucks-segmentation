# Starbucks Customer Segmentation
##### This is an in-depth analysis of how mobile app users respond to offers sent out by Starbucks. With a better understanding of customer behavior, Starbucks would be able to improve ad targeting, which is the purpose of this project. To give a high-level overview of this analysis, I ultimately had 2 goals in mind:
1. Segment customers based on demographics and behavior on the mobile app
2. Build a binary classifier that predicts whether a user will respond to a certain offer

#### Read more about it in my [article](https://medium.com/@buitri/improving-starbucks-ad-targeting-91833dff3e94).


## Data
This dataset contains 1 month of simulated data that mimics customer behavior on the Starbucks rewards mobile app. This is a simplified version of the real Starbucks app because the underlying simulator only has 1 product whereas Starbucks actually sells dozens of products.

Every few days, Starbucks sends out a variation of one of the following offers to mobile app users:
- `BOGO` (buy one get one free) — spend amount `A` in ONE purchase before the offer expires to get reward `R` of equal value to `A`
- `Discount` — spend amount `A` in ONE OR MORE purchases before the offer expires to get a discount `D` of equal or lesser value to `A` (all purchases within the validity period accumulate to meet the required amount `A`)
- `Informational` — only provides information about a product

#### The dataset is split into 3 files found in the `/data/raw/` directory:

1. `portfolio.json` (10 offers x 6 fields) — metadata for each offer
- id — offer ID
- offer_type— BOGO, discount, or informational
- difficulty — required spending amount to complete the offer
- reward — reward for completing the offer
- duration — validity period in days (the offer expires after this period)
- channels — web, email, mobile, social

2. `profile.json` (17,000 users x 5 fields) — demographic data for each user
- age — missing values were encoded as 118
- became_member_on — date in which the customer created an account
- gender — “M” for male, “F” for female, and “O” for other
- id — customer ID
- income — annual income of customer

3. `transcript.json` (306,534 events x 4 fields) — records of events that occurred during the month
- event — transaction, offer received, offer viewed, or offer completed
- person — customer ID
- time — number of hours since the start of the test (begins at time t=0)
- value — details of the event (offer metadata for offer-related events and amount for transactions)


## Process
This project has 3 parts, each of which are contained within their own Jupyter notebook, found in the `/notebooks/` directory:

#### `1-analysis.ipynb`
In this notebook, I clean and preprocess the raw data in the following ways:
- Drop records with missing or duplicated data
- Extract data from columns containing iterable data types (i.e. lists and dictionaries) into new columns
- Map ID hash strings to integers
- Recast columns to appropriate data types

Next is the exploratory analysis where I answer the following questions about the different offers and users in the data:
- How many offers were sent out?
- How many offers were viewed and/or completed?
- How many of each offer was completed?
- What do the demographics look like for completed offers versus incomplete offers?
- Are there any patterns in user spending?
- Are there any demographic patterns in the completion rate of each offer?

#### `2-segmentation.ipynb`
In this notebook, I perform customer segmentation using 2 different methods:
1. FMT quantile analysis - This is similar to RFM (recency, frequency, monetary value) analysis but I substitute tenure for recency to make FMT (frequency, monetary value, tenure). For each of these 3 features, customers are grouped into quantiles, which are then summed up to get the customer's total "score". This total score is used to segment customers into 3 tiers, which I call bronze, silver, and gold tiers in the analysis.
- Frequency - how many purchases the customer made during the month
- Monetary value - how much money the customer spent during the month
- Tenure - how long the customer has been using the mobile app

2. K-means clustering - The features used here are gender, age, income, frequency, monetary value, and tenure. The first method only looked at customer behavior, but this method also includes the customers' demographics.
- Since gender is a categorical variable, I use PCA (principle component analysis) to create 5 continuous components, which explain almost 95% of the variance in the 6 original features
- Using these 5 components, I create 4 clusters of customers

#### `3-modeling.ipynb`
In this notebook, I build a binary classifier that predicts whether a user will respond to an offer. This classifier will help decide whether or not to send users a particular offer. The input of this classifier includes an offer’s metadata and a user’s demographic and behavioral features. It will produce a binary output that predicts if the user will complete the offer.

I train 6 different models - logistic regression, k-nearest neighbors (KNN), support vector machine (SVM), decision tree, random forest, and light gradient boosting machine (LightGBM) - and evaluate them based on prediction accuracy and the F1 score. The model with the best metrics was selected as the final classifier. The hyperparameters of LightGBM were optimized using Optuna and the other 5 models were tuned with grid search.


## Results
#### As the final classifier, the random forest model made predictions on the validation set with an F1 score of 0.74 and on the test set with an F1 score of 0.75.

#### The analysis discovered several key insights about the data:
- Offers sent out via social media get a better response from customers
- The longer users have been using the app, the more comfortable they are spending money on it
- Female customers tend to spend more money than male customers, given they are in the same age group or income group
- Younger customers tend to make frequent, but small transactions
- As age increases, both average income and average spending showed a very similar pattern for both genders: an increase to about age 50 and then no change for the upper age groups
- As income increases, spending increases
- The more customers spend, the more likely they are to respond to offers

#### From the segmentation analysis, we found that there are generally 3 classes of customers — the lower, middle, and upper segments which I referred to as bronze, silver, and gold customers respectively. My recommendations for improving ad targeting are:
- Bronze users are not very likely to respond to offers so it would be a good idea to stop sending them offers or to only send them offers that are easy to complete.
- Silver users completed a lot of the easier offers, so focusing on the easier offers or lowering the difficulty of the harder offers would likely improve their response.
- Gold users consistently have a high rate of offer completion so it would actually benefit Starbucks to increase the difficulty of offers being sent to these users.


## Getting Started

#### Structure
<pre>
starbucks-segmentation/
|-- data/
|   |-- raw/
|   |   |-- portfolio.json
|   |   |-- profile.json
|   |   |-- transcript.json
|   |-- out1/ * needs to be created
|   |-- out2/ * needs to be created
|   |-- out3/ * needs to be created
|-- notebooks/
|   |-- 1-analysis.ipynb
|   |-- 2-segmentation.ipynb
|   |-- 3-modeling.ipynb
</pre>

#### Setup
1. Clone this repository.
2. Create 3 directories within the `/data/` directory as seen in the structure above: `/out1/`, `/out2/`, `/out3/`. These directories are the output paths for the 3 notebooks in the repository.
3. Ensure the requirements below are installed.

#### Requirements
- Python 3
- Packages: Jupyter notebooks/lab, Numpy, Pandas, Statsmodels, Scikit-learn, LightGBM, Optuna, Matplotlib, Seaborn, Plotly, Ipywidgets


## License
This repository is licensed under a [Creative Commons Attribution License](https://creativecommons.org/licenses/by/4.0/).
