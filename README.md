# Customer Churn Prediction using PySpark

This repository contains the code of building an end-to-end scalable machine learning pipeline using Python API for Spark, `PySpark`, to predict customer churn.

### Overview

- **[Project overview](#project-overview)**
- **[Problem statement](#problem-statement)**
- **[Get started](#get-started)**
  - [Prerequisites](#prerequisites)
  - [Cloud deployment](#cloud-deployment)
- **[Common issues on EMR cluster](#common-issues-on-emr-cluster)**
- **[Results](#results)**
- **[Acknowledgements](#acknowledgements)**
  
### Project overview

Predicting customer churn is a challenging and common problem for any e-commerce business in which everything depends on the behavior of customers. Customer churn is often defined as the process in which the customers downgrade from premium to free tier or stop using the products or services of a business. Thus, the ability to predict which users are at risk of churning, while there is still time to offer them discounts or other incentives, will greatly help to prevent every custormer-facing business from suffering severe financial losses.

### Problem statement

The datasets in this project are provided by Sparkify, a fictitious digital music service created by Udacity, to resemble the data sets generated by companies such as Spotify or Pandora. The full dataset collects over 26 million records from 22277 registered users, whereas a smaller subset (mini dataset) contains 286500 records from 225 registered users with a duration of about two months. Our goal is to help Spakify identify potential churn users by building and training a binary classifier in order to save the business millions in revenue. 

### Get started

This repository consists of the following files:

* `Sparkify_mini.ipynb`: Jupyter notebook that documents the whole model development process on the smaller dataset including exploratory data analysis, data visualization and discussions
* `Sparkify_full.ipynb`: modularized version of `Sparkify_mini.ipynb` used to train the full dataset on the Amazon EMR cluster
* `us_region.csv`: [csv file](https://github.com/cphalpert/census-regions) used in `Sparkify_mini.ipynb` that assigns each state to its corresponding geographical division (data source: [U.S. Census Bureau](https://www2.census.gov/geo/pdfs/maps-data/maps/reference/us_regdiv.pdf))

#### Prerequisites

To run `Sparkify_mini.ipynb` locally, the following Python libraries are required to be installed in addition to `PySpark`: `Numpy`, `Scipy`, `Pandas`, `Matplotlib`, `seaborn` and `time`.

#### Cloud deployment
To run `Sparkify_full.ipynb` on the Amazon Elastic MapReduce (EMR) cluster, configure your cluster with the following settings:
* Release: `emr-5.20.0` or later
* Applications: `Spark`: Spark 2.4.0 on Hadoop 2.8.5 YARN with Ganglia 3.7.2 and Zeppelin 0.8.0
* Instance type: `m3.xlarge`
* Number of instance: `3`
* EC2 key pair: `Proceed without an EC2 key pair` or feel free to use one
  
After the cluster is ready, open the Jupyter notebook and set `Change kernel` under the `Kernel` tab to `PySpark`.

### Common issues on EMR cluster

You may encounter the following issues when you run your EMR notebook:

* Missing Python libraries: There are only a limited number of Python libraries installed on an EMR cluster.
Following this [instruction](https://aws.amazon.com/blogs/big-data/install-python-libraries-on-a-running-cluster-with-emr-notebooks/), you can run the code 
```
sc.list_packages()
```
to check all the available Python libraries on the cluster, then install the other libraries on the cluster attached to your notebook using the `install_pypi_package` API. For example:
```
sc.install_pypi_package("pandas==0.25.1") #Install pandas version 0.25.1
sc.install_pypi_package("matplotlib", "https://pypi.org/simple") #Install matplotlib from given PyPI repository
```
After closing your notebook, the libraries that you installed on the cluster using the `install_pypi_package` API are garbage and collected out of the cluster.

* Livy session's timeout: If it takes more than an hour (the default timeout for a Livy session) to train your model, you will probably receive an error, such as
```
An error was encountered:
Invalid status code '400' from https://xxx.xx.xx.xx:18888/sessions/2/statements/19 with error payload: {"msg":"requirement failed: Session isn't active."}
```
To address this issue, go to `Configurations` tab of your cluster and choose the master node from `Filter`.
Click `Reconfigure` and pass the following EMR configuration in the field (you can set `livy.server.session.timeout` to a higher value if you need):
```
[{'Classification': 'livy-conf','Properties': {'livy.server.session.timeout':'5h'}}]
```
After you save the setting, wait a few seconds for the cluster to finish reconfiguring before you rerun your code.

* `Exception in thread cell_monitor-xx`: This exception will be raised occasionally during the training or testing process. It will not cause the process to fail, thus you can just ignore it or add try except to avoid it.
   
```
Exception in thread cell_monitor-31:
Traceback (most recent call last):
  File "/opt/conda/lib/python3.7/threading.py", line 926, in _bootstrap_inner
    self.run()
  File "/opt/conda/lib/python3.7/threading.py", line 870, in run
    self._target(*self._args, **self._kwargs)
  File "/opt/conda/lib/python3.7/site-packages/awseditorssparkmonitoringwidget-1.0-py3.7.egg/awseditorssparkmonitoringwidget/cellmonitor.py", line 178, in cell_monitor
    job_binned_stages[job_id][stage_id] = all_stages[stage_id]
KeyError: 6490
```

### Results

Three binary classifiers supported in Spark are selected as candidate models: logistic regression, random forest classifier and gradient-boosted tree classifier. We perform a grid search with 4-fold cross validation measured by **AUC-PR** on each model using the mini dataset to tune the hyperparameters, which shows that the tuned random forest classifier outperforms both the logistic regression and the gradient-boosted tree classifier in almost all metric categories. 

We further extract feature importances from the trained random forest classifier, and learn that **gender**, **latest subscription level** and **location** features contribute little to predicting churned users as shown below. This finding was later confirmed on the full dataset that the attributions of these features to model prediction are even more negligible. For this reason, these features are removed during the training on the full dataset, resulting in practically the same performance with a reduction of ~40% training time.
<p align="center">
    <img src="https://github.com/w-guo/wguo/blob/master/content/post/Sparkify-churn-prediction/feature_importances.png" width="600"> <br />
    <em><sub>Top 15 most important features from mini dataset</sub></em>
</p>
The final results of the random forest models evaluated on the respective test set of the mini and full datasets are summarized in the table below:

| <sub>Dataset</sub>      | <sub>Parameters</sub>                | <sub>Precision</sub> |                     | <sub>Recall</sub>  |                     | <sub>F1 score</sub> |                     |  <sub>AUC-PR</sub>  |
| :---------------------- | :----------------------------------- | :------------------: | :-----------------: | :----------------: | :-----------------: | :-----------------: | :-----------------: | :-----------------: |
|                         |                                      |  <sub>Overall</sub>  | <sub>Churned</sub>  | <sub>Overall</sub> | <sub>Churned</sub>  | <sub>Overall</sub>  | <sub>Churned</sub>  |                     |
| <sub>Mini dataset</sub> | <sub>maxDepth=4, numTrees=100</sub>  |   <sub>0.86</sub>    | <sub>**0.75**</sub> |  <sub>0.86</sub>   | <sub>**0.60**</sub> | <sub>**0.86**</sub> | <sub>**0.67**</sub> | <sub>**0.77**</sub> |
| <sub>Full dataset</sub> | <sub>maxDepth=10, numTrees=100</sub> |   <sub>0.92</sub>    | <sub>**0.93**</sub> |  <sub>0.91</sub>   | <sub>**0.64**</sub> | <sub>**0.91**</sub> | <sub>**0.76**</sub> | <sub>**0.89**</sub> |

 It can be seen that there are sizeable performance gains in every metric category due to more training data. At a default probability threshold of 0.5, our model is currently able to identify 64% of user churn, while 7% identified as churned users acutally satisfy with the service. If we want to target more users prone to churning, we can lower the probability threshold, and a high AUC-PR score of 0.89 will allow us to maintain a high precision that there will not be many more users with no intent of churning mistakenly targeted. More details can be found in my [blog post](https://wguo.rbind.io/post/sparkify-churn-prediction/).

### Acknowledgements
Credit to Udacity for designing the project and hosting the datasets:

* The full Sparkify dataset (12GB) can be found at `s3n://udacity-dsnd/sparkify/sparkify_event_data.json`
* The smaller version (128MB) is available at `s3n://udacity-dsnd/sparkify/mini_sparkify_event_data.json`
  
Blog reference: https://medium.com/@lukazaplotnik/sparkify-churn-prediction-with-pyspark-da50652f2afc
