# League-of-Legends-Result-Prediction

by WonJae Lee (wolee@ucsd.edu)

---

## Framing the Problem

Given the massive dataset, anyone who is interest in competitive League of Legends games would want to know this: will my team win or lose? To answer this question, we will use the cleaned dataset from our previous analysis, which is all 2022 League of Legends competitive game data. Here, we will focus on LCK, the Worlds winning league for the year, as it is one of the most competitive leagues, yielding quality data, and has much more data than the Worlds. In addition to the cleaning that was already done, we will add another feature, 'opp_teamname', which is the name of the opponent team. We will use binary classification to predict result (win/loss), and F1 score as our metric. F1-score is useful in this case because although the dataset might seem balanced, there are many imbalances in team performance and the "meta" champion peformance. For example, a particular team might be overwhelmingly stronger than others, and there could be a few champions with significantly higher winrate or performance than others. Because of the expected imbalance and the equal importance of precision and recall, we will choose F1 score over accuracy and other metrics. 

Our explanatory data analysis on this dataset can be found [here](https://wj6801.github.io/League-of-Legends-Win-Rate-Analysis/)

Moreover, we will choose a set of reasonable features to predict the result. We will try to predict the result of the game before the game begins; therefore, we will use only the data available to us before the game begins. For our base model, we will use 'side' and 'champion' to predict the result. For convenience, we will select only the columns we will use in our analysis.


|    | side   | champion   | teamname     | ban1     | ban2    | ban3         | ban4    | ban5   | opp_teamname   |
|---:|:-------|:-----------|:-------------|:---------|:--------|:-------------|:--------|:-------|:---------------|
|  0 | Blue   | Graves     | DRX          | Diana    | Caitlyn | Twisted Fate | LeBlanc | Viktor | Liiv SANDBOX   |
|  1 | Blue   | Viego      | DRX          | Diana    | Caitlyn | Twisted Fate | LeBlanc | Viktor | Liiv SANDBOX   |
|  2 | Blue   | Ryze       | DRX          | Diana    | Caitlyn | Twisted Fate | LeBlanc | Viktor | Liiv SANDBOX   |
|  3 | Blue   | Aphelios   | DRX          | Diana    | Caitlyn | Twisted Fate | LeBlanc | Viktor | Liiv SANDBOX   |
|  4 | Blue   | Sona       | DRX          | Diana    | Caitlyn | Twisted Fate | LeBlanc | Viktor | Liiv SANDBOX   |
|  5 | Red    | Tryndamere | Liiv SANDBOX | Renekton | Lee Sin | Leona        | Jayce   | Akali  | DRX            |
|  6 | Red    | Xin Zhao   | Liiv SANDBOX | Renekton | Lee Sin | Leona        | Jayce   | Akali  | DRX            |
|  7 | Red    | Syndra     | Liiv SANDBOX | Renekton | Lee Sin | Leona        | Jayce   | Akali  | DRX            |
|  8 | Red    | Jhin       | Liiv SANDBOX | Renekton | Lee Sin | Leona        | Jayce   | Akali  | DRX            |
|  9 | Red    | Yuumi      | Liiv SANDBOX | Renekton | Lee Sin | Leona        | Jayce   | Akali  | DRX            |


## Baseline Model

There are not many pre-game features available to us. From the few pre-game feature, we choose 'side', 'champion', and 'teamname' for our baseline model features. Because the three features are all categorical features, we will one-hot encode all features to use Random Forest Classifier. Random Forest Classifier is great for capturing complex interactions between features, and it is robust to overfitting especially with a large dataset, which is what we want here.

**Nominal Features**: side (blue/red), champion, teamname

**Response Variable**: result (Win(1) / Lose (0))

**Baseline Model Report**

We get a f1-score of slightly above 50%, which is not that great. A low F1 score (close to 0) indicates a lack of balance, meaning the model is either missing too many actual wins/losses (low recall) or is incorrectly classifying too many matches (low precision). It's barely better than just coin flipping. We will need to include more features and tune hyperparameters to see a better result.


## Final Model

**New Features**: bans (1 - 5), opp_teamname

We have access to 5 banned champions per team before the game starts. We also created a new feature 'opp_teamname' because some teams are more likely to win against certain teams while less likely to win against others. Banned champions greatly influence picked champions—you obviously can't pick them, so you will have to adjust your entire team composition accordingly. By including 5 bans for each team, we will have more information about the game before it starts.

Our final model remains to be Random Forest Classifier because of its suitability for solving this problem as mentioned when choosing our baseline model. Its ability to handle complex datasets and provide robust predictions makes it a good model for this problem.

We choose to tune 5 parameters: n_estimators (100, 200, 500), criterion ('gini', 'entropy'), max_depth (3, 4, 5), min_samples_split (2, 5, 10), and min_samples_leaf (1, 2, 3, 5) through GridSearchCV. The resulting best parameters are n_esimators=500, criterion='gini', max_depth=5, min_samples_split=2, min_samples_leaf=1. The f1-score improved from 0.58 to 0.79, and this is a huge improvment and decent performance considering it is using only pre-game data, without any in-game or after-game data.


## Fairness Analysis

The two sides in League of Legends, Red and Blue, are known to be unequal. More specifically, Blue Side is known to be stronger for various reasons. Blue Side picks a champion first after the ban phase. Also, because of how the map is set up, Blue Side has easier access to important objectives like the Rift Herald and Baron Nashor. Blue Side also gets a better camera angle to look at the map. Through these advantages, Blue Side is known to be stronger in League of Legends, especially in competitive leagues where seamless planning and coordination is required. In this section we will try to answer the question: “does the final model performance differ for Blue Side and Red Side?" Because the 'side' column is categorical, we will one-hot encode the columns. We will use our metric above, f1-score, to evaluate the difference between the two groups. We will use the conventional 0.05 significance level.

**Null Hypothesis**

Our model is fair. The classifier’s accuracy is roughly the same for both Blue Side and Red Side, and any differences are due to random chance.

**Alternative hypothesis**

Our model is unfair. The classifier’s accuracy is not the same for both groups.

**Test Statistic**

Difference in f1-score (blue - red)

**Significance level**

0.05

To perform our fairness analysis, we first define our test statistic, f1_diff that calculates the difference in f1 scores between the Blue Side and Red Side given X_test and y_test.
Next, we define the null simulation where we shuffle the X_test on 'side' column and calculate a single test statistic for the shuffled data.
Then, we simulate the null 1000 times and calculate our p value using the simulation results and our observed statistic.

<iframe src="./assets/diff_f1.html" width=1000 height=500 frameBorder=0></iframe>

With the test statistic of -0.01, we obtained the p-value of 0.554, failing to reject the null. Therefore, we do not have enough evidence to conclude that our model performs differently for Blue and Red sides.
