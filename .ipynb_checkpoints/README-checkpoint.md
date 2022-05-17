# ![](https://ga-dash.s3.amazonaws.com/production/assets/logo-9f88ae6c9c3871690e33280fcf557f33.png) Project 3

<h1 align="center">Whole-Food Plant-Based vs. Paleo:<br />Identifying the best-performing classification model</h1>

---

## Purpose

Lifestyle Eating is out to help people live healthier and happier lives through their diets. The idea is to build a platform that fosters a supportive community of users for a variety of diets. Users will be able engage with the community through posts in the form of questions, recipes, tips, personal updates and more.

Having the user include their diet when they post is an option, but the product development team is interested in implementing auto-classification technology that can detect what diet the user is on, as this could come in handy with later platform features.

To see how viable this auto-classification technology will be when applied to user posts, they've requested for us, the data science team, to look into it.

As lead on the first phase of this request, I’ve chosen to pull submissions from the [PlantBasedDiet](https://www.reddit.com/r/PlantBasedDiet/) and [Paleo](https://www.reddit.com/r/Paleo/) subreddits, as the submissions are very similar to what users would post on the Lifestyle Eating platform. These two diets were chosen because while they have notable differences, they do have many similarities that will make for good testing data.

The purpose of phase 1 will be to identify the classification model that will most accurately detect the diets of the submissions. The model will be evaluated through the accuracy scores of the training and testing datasets. The results and findings of this phase will serve as the building blocks for the remainder of the project’s phases.

---

## Data

### Whole-Food Plant-Based vs. Paleo

As previously mentioned, the plant-based and paleo diets were chosen because while they have major differences, they share many similarities. So, what exactly do they consist of?

A **whole-food, plant-based diet** is one that focuses on natural foods that come from plants, are not heavily processed, and are free of animal ingredients such as meat, dairy, eggs and honey. 

The main food groups for this diet are fruits, vegetables, whole grains and legumes. Additional acceptable foods include nuts, seeds, tofu, tempeh, and plant-based milks. 

A **paleolithic diet**, commonly referred to as the caveman diet, focuses on natural foods that were consumed before the Neolithic or Agricultural Revolution (10,000 B.C.) when farming became the primary method of obtaining food.

The main food groups for this diet are lean meats, fish, fruits, vegetables and nuts. They avoid all processed foods, dairy, grains, legumes, and carbs that don’t come from fruits or vegetables.

So the similarities lie in that they focus on natural foods like fruits, vegetables and nuts, while avoiding processed foods and dairy.

The differences lie in that a paleo diet allows meat and fish, and avoids grains and legumes. The plant-based diet allows grains and legumes, but does not allow meat and fish.

### Data Source

The subreddit submissions were obtained through the Pushift API. Given that the API limits to pulling 100 submissions at a time, a function was created in order to automate the pulling of the desired number of submissions and return a concatenated DataFrame for a given subreddit.

Through this, two CSV files were created, one for the [PlantBasedDiet](https://www.reddit.com/r/PlantBasedDiet/) subreddit and one for the [Paleo](https://www.reddit.com/r/Paleo/) subreddit. Each contained the most recent 5,000 submissions, and included the following features: 

* **subreddit** - name of the subreddit the submission belonged to
* **title** - title of the submission
* **selftext** - body text of the submission
* **created_utc** - time of submission in unix time (seconds since 01-01-1970 UTC)

---

## Methodology

### Data Cleaning

As part of the cleaning process, the selftext and created_utc features were dropped. While the selftext feature could have been valuable, it had between 2,100-2,300 missing values per subreddit that could not be imputed, and the timestamps of these submissions were no longer relevant.

This left the subreddit and title features as the main focus. The names in the subreddit feature were mapped to binary values, while characters and additional information in the title feature, such as URLs, digits, punctuation, special characters and emojis were removed as they did not provide any value.

### Exploratory Data Analaysis

Prior to the cleaning of the title feature, an initial exploratory data analysis was conducted and character and word counts per title were checked. The character and word counts were comparable and very similarly distributed (unimodal and right-skewed) between both the plant-based and paleo datasets, so there was nothing that stood out there. As a result, these features were dropped.

After the cleaning of the title feature, additional exploratory data analysis was conducted and the most common unigrams and bigrams were identified. This was after fitting and transforming each dataset to a CountVectorizer that analyzed by word, changed all words to lowercase and removed the NLTK’s predefined stopwords.

The top 15 plant-based unigrams and bigrams didn’t appear to be too interesting. They were either words that were obviously referring to a plant-based diet (such as plant, vegan, and wfpb) or general diet terms (such as recipe, protein, healthy and weight loss).

The top 15 paleo unigrams and bigrams also included general diet terms (such as recipes and weight loss), but also included were many terms, aside from paleo (such as keto, chicken, low carb, grass fed, and bone broth), that were likely to set many of the submissions apart from the plant-based submissions. 

Based on these findings, the paleo submissions seem likely to have a better chance at being identified correctly than the plant-based submissions.

### Model Exploration

To identify the best performing classification model based on the accuracy score, nine models were tested:

* Random Forest Classifier
* Logistic Regression
* k-Nearest Neighbors
* AdaBoost Classifier
* Gradient Boosting Classifier
* XGBoost Classifier
* Support Vector Classifier
* Bernoulli Naive Bayes
* Multinomial Naive Bayes

A pipeline was created for each, and included a TfidfVectorizer in addition to the classification model. Given the large number of parameters to be tested, the optimal parameters were searched using a Randomized Search, which reduces training time and is shown to perform just as well as Grid Search in these cases.

Each pipeline was fitted without any hyperparameters as a first pass,The testing scores of all the models, with the exception of k-Nearest Neighbors were between 78-82%, so what it really came down to was the amount of variance. As a result, AdaBoost, Gradient Boosting and XGBoost models emerged as the strongest contenders for best-performing based on their competitive testing accuracy scores and low-variance. 

Of the three strongest contenders, XGBoost achieved the highest testing accuracy score of 0.8079, but was the most overfit by 0.0472. AdaBoost achieved the second highest testing score of 0.7938, but achieved the lowest variance of 0.01. Gradient Boosting had the lowest testing score of 0.7873, but achieved the second lowest variance of 0.0155.

During the subsequent passes with a Randomized Search of hyper-parameters, XGBoost emerged as the best-performer with a testing accuracy score of 0.7968 and the lowest variance of 0.0156. This makes it the model that will perform best on unseen data.

---

## Best Model & Insights

The XGBoost model’s best performing parameters were:

* **TfidfVectorizer**
    * ngram_range = (1,2)
    * min_df = 5
    * max_features = 2,000
    * max_df = 0.9

* **XGBoost**
    * n_estimators = 10
    * learning_rate = 1.0

These parameters yielded a 0.8124 training accuracy score and a 0.7968 testing score. 

Though the overall accuracy was the metric to optimize, the creation of a confusion matrix was helpful in seeing if the model was equally accurate between the diets, or if it favored one over the other.

With plant-based being negative (0) and paleo being positive (1), the model seemed to favor the plant-based diet. Of the 2,000 submissions used to test:

* 933 were true negatives (guessed plant-based, truly plant-based)
* 660 were true positives (guessed paleo, truly paleo)
* 340 were false negatives (guessed plant-based, actually paleo)
* 66 were false positives (guessed paleo, actually plant-based)

This means that the model is optimizing for true negatives with a 0.9339 accuracy, while the accuracy for true positives is 0.66. Therefore, this model is better at predicting that plant-based submissions are actually plant-based.

---

## Conclusion

### Recommendations and Next Steps

It’s recommended to keep the XGBoost classification model in mind as a best-performer, as it showed the most promising results despite the imbalance in accuracy between the diets.

With the plant-based and paleo datasets, it will be beneficial to identify the words that could be causing the paleo submissions to be mistaken for plant-based submissions. Removing words relating to the diets’ commonalities, such as fruits and vegetables, may result in model improvement. It will also be beneficial to pull more submissions from each subreddit and run the model on larger datasets.

The XGBoost model should also be tried with other diet datasets, such as Mediterranean, Keto, Vegetarian, Gluten-Free, etc. to see how it performs there.