---
title: "Hacking a serverless machine-learning scoring microservice with AWS Lambda"
date: 2017-09-29 0:00:00 -0700
comments: true
categories: 
  - machine learning
  - microservices
tags:
  - python
  - aws
  - lambda
excerpt: In this post I'll attempt to hack a `scikit-learn` model prediction microservice with AWS Lambda.
---

In this post I'll attempt to hack of a `scikit-learn` model prediction microservice with AWS Lambda. This can be called a "serverless machine-learning scoring microservice". It's a mouthful o' buzzwords, but it's a sneak peak at an exciting advancement in efficient, scalable cloud technology for serving up machine learning model predictions over the web for anybody and anything wishing to consume them, like web and mobile apps, dynamic visualizations, Shiny/Django/Rails apps and possibly batch processes requiring batch predictions.

The service works great on my local machine. Lambda has a 50Mb limit on the deployed package size but the smallest I've managed to get mine is about 75 Mb, so AWS errors out when I deploy it to the cloud. If I could bump up that limit then I'm all but certain it will work in the cloud just as well, and better under Lambda's infrastructure for scalability.

# Dependencies

The app is pure Python. The local Python dependencies are
* `awscli` for programmatic AWS interaction.
* `boto3` for AWS S3 interaction.
* `chalice` to implement RESTful API's.
* `scikit-learn` for machine-learning modeling.
* `scipy` is the only explicit additional `scikit-learn` dependency needed for the app given the model I trained.
* `virtualenvwrapper` for simple Python virtual environment management.

Remote Python dependences are placed into a `requirements.txt` that ultimately contains

```
boto3==1.4.7
scikit-learn[alldeps]
```

# Procedure

Sign up for AWS free tier. 

On your local machine, [install `virtualenvwrapper`](https://virtualenvwrapper.readthedocs.io/en/latest/), then create a virtual environment and install the AWS CLI and `chalice`.

```bash
mkvirtualenv aws-serverless-microservices
pip install awscli
pip install chalice
pip install boto3
```

In the AWS Console create an IAM user for programmatic CLI access with admin privileges, then configure AWS CLI with your access information with the AWS CLI utility.

```bash
aws configure
```

You'll be prompted for your AWS region and access keys. 

Do the [`chalice` helloworld](http://chalice.readthedocs.io/en/latest/quickstart.html). When you deploy it you will get the Lambda URL. Set a Bash variable to this URL like this.

```bash
BASEURL=https://brycp1llxh.execute-api.us-west-2.amazonaws.com/api
```

When you deploy in local mode like this

```bash
chalice local --port 5005
```

you'll specify a port and then set the URL variable. In the terminal window where you'll be issuing requests:

```bash
PORT=5005
BASEURL=http://localhost:$PORT
```

Test your hello-world endpoint.

```bash
curl $BASEURL
```

You should get 

```json
{"hello": "world"}
```

if things are set up correctly. Try `local` and `deploy` modes.

You'll be storing a static serialized (pickled) `scikit-learn` model on AWS S3, so first create an S3 bucket in the S3 console. Then do the ["policy generation" tutorial](http://chalice.readthedocs.io/en/latest/quickstart.html#tutorial-policy-generation) to get a basic familiarity with accessing S3 using `boto3`. `chalice` recommends pinning the boto3 version in `requirements.txt`.

```bash
pip freeze | grep boto3 >> requirements.txt
```

Test out the S3 endpoints from the tutorial by putting a simple JSON object into your S3 bucket.

```bash
curl -X PUT -H "Content-Type: application/json" -d '{"key1":"value"}' $BASEURL/objects/test.json
curl -X GET $BASEURL/objects/test.json
```

If you get your object back, you've successfully configured your S3 connection. You'll also be able to see the object you just created in the bucket using your S3 Management Console. This is where you'll put your pickled model.

On to the machine learning. Train a 3-class gradient boosted decision tree logistic regression model on iris data set using the [`scikit-learn` tutorial](http://scikit-learn.org/stable/auto_examples/linear_model/plot_iris_logistic.html) as a guide. Pickle the model as `model.pkl`. **It doesn't matter how good this model is** for the purposes of this hack, it just needs to make predictions. Here's my full model training and serialization script.

```python
import pickle
from sklearn import datasets
from sklearn.ensemble import GradientBoostingClassifier

# import some data to play with
iris = datasets.load_iris()
X = iris.data[:, :2]  # we only take the first two features.
Y = iris.target

clf = GradientBoostingClassifier(n_estimators=100, learning_rate=1.0,
	max_depth=2, random_state=0).fit(X, Y)
# make prediction
preds = clf.predict(X)

pickle.dump(clf, open("model.pkl", "wb"))

with open('model.pkl', 'r') as myfile:
    r = myfile.read()

model_r = pickle.loads(r)

preds_r = model_r.predict(X)

if any(preds - preds_r != 0):
	raise Exception("serialization error")
```

Now manually upload `model.pkl` to your S3 bucket using the S3 Management Console.

Add `scikit-learn` to your requirements file.

```bash
echo "scikit-learn[alldeps]" >> requirements.txt
```

Here are contents of my final `app.py` file. It contains the original hello-world, the S3 tutorial endpoints and a prediction endpoint.

```python
import json
import pickle
import boto3
from botocore.exceptions import ClientError

from chalice import Chalice

# Global variables.
S3 = boto3.client('s3', region_name='us-west-2')
BUCKET = 'helloworld-model'
MODEL_KEY = 'model.pkl'


# Global functions.
def memoize(f):
    memo = {}
    def helper(x):
        if x not in memo:            
            memo[x] = f(x)
        return memo[x]
    return helper


@memoize
def get_model(model_key):
    try:
        response = S3.get_object(Bucket=BUCKET, Key=model_key)
    except ClientError as e:
        if e.response['Error']['Code'] == "404":
            print("The object does not exist.")
        else:
            raise
    # TODO find a way to persist this model
    model_str = response['Body'].read()
    model = pickle.loads(model_str)
    return model


# Begin chalice app endpoint definitions.
app = Chalice(app_name='helloworld')


@app.route('/')
def index():
    return {'status': 'up'}


@app.route('/objects/{key}', methods=['GET', 'PUT'])
def myobject(key):
    request = app.current_request
    if request.method == 'PUT':
        OBJECTS[key] = request.json_body
    elif request.method == 'GET':
        try:
            return {key: OBJECTS[key]}
        except KeyError:
            raise NotFoundError(key)


@app.route('/predict', methods=['POST'])
def predict():
    request = app.current_request
    if request.method == 'POST':
        result = {}
        model = get_model(MODEL_KEY)
        body_dict = request.json_body # eg, {"data": [[ 6.2,  3.4]]}
        data = body_dict['data'] # eg, [[ 6.2,  3.4]]
        pred = model.predict(data).tolist()
        result = {'prediction': pred}
        return json.dumps(result)
```

Deploying to Lambda will fail (see Appendix), so try it out in local mode and test the prediction endpoint.

```bash
curl -X POST -H "Content-Type: application/json" -d '{"data":[[6.2, 3.4], [6.2, 1]]}' $BASEURL/predict
```

My response is

```json
{"prediction": [2, 1]}
```

Run it a second time, and you should notice a significantly faster prediction. This is because the model file is pulled from S3 just the first time and memoized (cached) forever thereafter, eliminating any need for additional S3 data transfer. I'm seeing sub-10ms response time in local mode.

And that's the hack. This is a great way to define and deploy scalable services that use resources efficiently, and I'm sure if I paid Amazon money they would increase my Lambda limits.

# Appendix

Here's what that `chalice deploy` error looks like.

```
Creating deployment package.
Updating IAM policy for role: helloworld-dev
Updating lambda function: helloworld-dev

ERROR - While sending your chalice handler code to Lambda to update function 
"helloworld-dev", received the following error:

 Connection aborted. Lambda closed the connection before chalice finished 
 sending all of the data.

This is likely because the deployment package is 74.8 MB. Lambda only allows 
deployment packages that are 50.0 MB or less in size. To avoid this error, 
decrease the size of your chalice application by removing code or removing 
dependencies from your chalice application.
```
