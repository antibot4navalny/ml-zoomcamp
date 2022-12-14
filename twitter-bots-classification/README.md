# Midterm project
**by Albert Tuykin**  
darkcorpd@gmail.com

for the [Machine Learning Zoomcamp](https://mlzoomcamp.com) by Alexey Grigorev.

## Twitter bot detection using supervised machine learning

![image](https://user-images.githubusercontent.com/91184329/200782202-6f61f9de-21df-43a9-ac85-45ab19eb2347.png)


In the world of Internet and social media, there are about 3.8 billion active social media users and half of the internet traffic consists of mostly bots.

Malicious bots make up 20% of the traffic, they are used for nefarious purposes, they can mimic human behavior, they can impersonate legal traffic, attack IoT devices and exploit their performance. 
Among all these concerns, the primary concern is for social media users as they represent a large group of active users on the internet, they are more vulnerable to breach of data, change in opinion based on data. Detection of such bots is crucial to prevent further mishaps. 

I use supervised Machine learning techniques in this project such as Logistic regression, Decision tree, Random Forest and XGBoost to calculate their accuracies and compare to detect Twitter bots from a collected training data set. 

## Data collection

I have used a [list](https://github.com/antibot4navalny/accounts_labelled/blob/main/labels.json) of Twitter accounts recognized as bots. This list is used in Chrome extention '[MetaBot for Twitter](https://github.com/antibot4navalny/metabot)' to highlight bots on Twitter.

I have used a `twitter_scraper_selenium` library to get profile details.

The list comparation is used to update the collected dataset by the information for new names in the bot list (updatable list).

The same way is used to collected data on real organic users provided by the same author [antibot4navalny](https://github.com/antibot4navalny). Two datasets of bot and organic users' profiles are mixed to get a train set for a bot classification problem. The cleaned of duplicates dataset contains 1220 rows × 157 columns.

The data collection proccess is described in [twitter_data_collect.ipynb](https://github.com/darkcorpd/ml-zoomcamp/blob/main/twitter-bots-classification/twitter_data_collect.ipynb).

## Exploratory data analysis (EDA) 
is an especially important activity in the routine of a data analyst or scientist. It enables an in depth understanding of the dataset, define or discard hypotheses and create predictive models on a solid basis.

1. Remove duplicates.
2. Drop actually empty columns by the thereshold. To find a thereshold by which I will drop the columns I've plotted a distribution chart for NaN values. All the columns that have more than 2 NaN values can be removed.
3. Explore the data by the common approach:
*	**Variable:** name of the variable.
*	**Type:** the type or format of the variable. This can be categorical, numeric, Boolean, and so on.
*	**Context:** useful information to understand the semantic space of the variable. In the case of our dataset, the context is the social one.
*	**Expectation:** how relevant is this variable with respect to our task? We can use a scale “High, Medium, Low”.
*	**Comments:** whether or not we have any comments to make on the variable.

The description of the variables can be found on [developer.twitter.com](https://developer.twitter.com/en/docs/twitter-api/v1/data-dictionary/object-model/user)

After some research I decided to go with a list of 13 features:
```
numerical = ['statuses_count',
             'followers_count',
             'friends_count',
             'favourites_count',
             'listed_count',
             'media_count',
             'life_span']

categorical = ['default_profile_image',
              'geo_enabled',
              'protected',
              'verified',
              'has_custom_timelines',
              'advertiser_account_type']
```

## Model selection and parameter tuning 

I have trained 4 models: LogisticRegression, DecisionTree, RandomForest and XGBoost.

The validation framework was created by the splitting dataset into train, validation and test sets. The target variable was removed from the original dataset.
The preprocessing is done using the DictVectorizer to make a one-hot encode for the categorical variable of type. 

The four models were trained and tuned on the following parameters:

**LogisticRegression**:
* C parameter of regularization

*LogisticRegression(C=0.1, max_iter=1000, random_state=1)*

**DecisionTree**:
* max_depth
* min_sample_leaf

*DecisionTreeClassifier(max_depth=4, min_samples_leaf=15)*

**RandomForest**:
* n_estimators 
* max_depth 
* min_samples_leaf

*RandomForestClassifier(max_depth=10, n_estimators=90, random_state=1, min_samples_leaf = 1)*

**XGBoost**:
* eta
* max_depth
* min_child_weight

*XGBClassifier(eta=0.1, max_depth=3, min_child_weight=1)*

The models were compared, where RandomForest was the one with the best roc_auc_score:

|Model|roc_auc_score|
|-----|--|
|LogisticRegression|98.43%|
|DecisionTree|97.91%|
|**RandomForest**|**98.83%**|
|XGBoost|98.56%|

The final models were trained with the full data train and compared with the test data. 
|Model|roc_auc_score|
|-----|--|
|LogisticRegression|98.26%|
|DecisionTree|97.22%|
|**RandomForest**|**98.97%**|
|XGBoost|98.55%|


## Deployment locally using [BentoML](https://docs.bentoml.org/en/latest/tutorial.html)

The [build_bento_model_twitter_bots.ipynb](https://github.com/darkcorpd/ml-zoomcamp/blob/main/twitter-bots-classification/build_bento_model_twitter_bots.ipynb) Jupyter Notebook is used to create a BentoML model.

A [bentofile.yaml](https://github.com/darkcorpd/ml-zoomcamp/blob/main/twitter-bots-classification/bentofile.yaml) specifies the service, labels, programming language and the list of used packages.

The following command is used to import the BentoML model:

```bash
bentoml models import twitter_bot_classify_model:7lt5hulbdgrrvodq.bentomodel
```

The file [train.py](https://github.com/darkcorpd/ml-zoomcamp/blob/main/twitter-bots-classification/train.py) is elaborated with the script to deploy it locally using the BentoML interface. 

Local deployment can be run using the following command in the terminal:

```
bentoml serve train.py:svc --production
```

The Swagger interface can be found by the address of [local host](http://localhost:3000)

![image](https://user-images.githubusercontent.com/91184329/201192051-c09b6231-d700-4927-b652-ea10b85d5e46.png)

## Deployment locally using Jupyter Notebook 

The file [predict.ipynb](https://github.com/darkcorpd/ml-zoomcamp/blob/main/twitter-bots-classification/predict.ipynb) can be used to load the BentoML model and predict if a Twitter user is bot or not. 

A pydantic dictionary scheme must be used:

```
statuses_count: float
followers_count: float
friends_count: float
favourites_count: float
listed_count: float
media_count: float
life_span: int
default_profile_image: bool
geo_enabled: bool
protected: bool
verified: bool
has_custom_timelines: bool
advertiser_account_type: object
```
 
 Here is an example
 
 ```python
# This user is actually a bot:
test1 = {"statuses_count": 417.0,
         "followers_count": 0.0,
         "friends_count": 1.0,
         "favourites_count": 5.0,
         "listed_count": 0.0,
         "media_count": 0.0,
         "life_span": 124,
         "default_profile_image": True,
         "geo_enabled": False,
         "protected": False,
         "verified": False,
         "has_custom_timelines": False,
         "advertiser_account_type": "none"}

# This user is actually an organic:
test2 = {"statuses_count": 628.0,
        "followers_count": 10.0,
        "friends_count": 24.0,
        "favourites_count": 649.0,
        "listed_count": 0.0,
        "media_count": 33.0,
        "life_span": 216,
        "default_profile_image": False,
        "geo_enabled": False,
        "protected": False,
        "verified": False,
        "has_custom_timelines": False,
        "advertiser_account_type": "none"}
  ```
  
![image](https://user-images.githubusercontent.com/91184329/201191596-8ec2edee-d2cd-4980-9302-fbd7c0686b18.png)


## Deployment using Docker

To build a bento execute a command:

```bash
bentoml build
```
![image](https://user-images.githubusercontent.com/91184329/201191196-3843ebaa-bda7-4229-a86a-aede4fb7bf7f.png)
  
To deploy model using the Docker, containerize the model into a Docker image, using the bentomodel file, and type the following command to containerize:

```bash
bentoml containerize twitter_bot_classifier:t7ux53tbekfdkaav
```

The docker image can be also downloaded from the repository [docker image](https://hub.docker.com/r/darkcorpd/twitter-bots-classification)

```bash
docker pull darkcorpd/twitter-bots-classification
```

To run the docker the following command is used:

```bash
docker run -it --rm -p 3000:3000 twitter_bot_classifier:t7ux53tbekfdkaav serve --production
```

To access the deployment visit [local host](http://localhost:3000). You can make the predictions!

![image](https://user-images.githubusercontent.com/91184329/201193400-6c6d184e-3cdd-4c2b-a437-51e3bea862b1.png)


## Deployment using [Yandex Serverless Containers](https://cloud.yandex.com/en/docs/serverless-containers/operations)

1. Register, install and initialize the [Yandex Cloud command line interface](https://cloud.yandex.com/en/docs/cli/quickstart#install).

2. Create a container registry:

```
yc container registry create --name mlzoomcamp
```
Result:

```
done (1s)
id: crppqmhkphar1670tc3q
folder_id: b1g7t5n5f0dsjru3esu8
name: mlzoomcamp
status: ACTIVE
created_at: "2022-11-10T19:25:31.502Z"
```

Make sure the registry was created:

```
yc container registry list
```
Result:
|ID|NAME|FOLDER ID|
|-----|--|--|
|crppqmhkphar1670tc3q|mmlzoomcamp|bb1g7t5n5f0dsjru3esu8|


3. Create IAM token and authorize with IAM token:
```
yc iam create-token
```

```
docker login --username iam --password <IAM TOKEN> cr.yandex
```

4. Push container to the registry:
```
docker push docker push twitter_bot_classifier:t7ux53tbekfdkaav
```

5. Create a serverless container:
```
yc serverless container create --name mlzoomcamp
```

Result:
```
done (1s)
id: bbamgbl5meliajhlnard
folder_id: b1g7t5n5f0dsjru3esu8
created_at: "2022-11-10T19:29:41.261Z"
name: mlzoomcamp
url: https://bbamgbl5meliajhlnard.containers.yandexcloud.net/
status: ACTIVE
```

6. Create a container revision with prepared image and service account:
```
yc serverless container revision deploy \
  --container-name mlzoomcamp \
  --image <Docker_image_URL> \
  --cores 1 \
  --memory 1GB \
  --concurrency 1 \
  --execution-timeout 30s \
  --service-account-id <service_account_ID>
```

Check revision by container name

```
yc serverless container revision list --container-name mlzoomcamp
```

7. Make it public and get an invocation link
```
yc serverless container allow-unauthenticated-invoke mlzoomcamp
```
```
yc serverless container get mlzoomcamp
```
Result:

```
id: bbamgbl5meliajhlnard
folder_id: b1g7t5n5f0dsjru3esu8
created_at: "2022-11-10T19:29:41.261Z"
name: mlzoomcamp
url: https://bbamgbl5meliajhlnard.containers.yandexcloud.net/
status: ACTIVE
```
![image](https://user-images.githubusercontent.com/91184329/201207330-d688322a-2a2b-4d54-adf9-6e9467a027a9.png)
