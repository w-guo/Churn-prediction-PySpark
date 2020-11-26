# Customer Churn Prediction using PySpark

This repository contains a Jupyter notebook `Sparkify_mini.ipynb` that documents an end-to-end model development process performed on a simulated streaming service dataset using Python API for Spark, `PySpark`.

### Project overview

Predicting customer churn is a challenging and common problem for any e-commerce business in which everything depends on the behavior of customers. Customer churn is often defined as the process in which the customers downgrade from premium to free tier or stop using the products or services of a business. Thus, the ability to predict which users are at risk of churning, while there is still time to offer them discounts or other incentives, will greatly help to prevent every custormer-facing business from suffering severe financial losses.

### Problem statement

The dataset in this project is provided by Sparkify, a fictitious digital music service created by Udacity, to resemble the data sets generated by companies such as Spotify or Pandora. The full dataset collects over 26 million records from 22277 registered users, whereas a smaller subset contains 286500 records from 225 registered users with a duration of about two months. Our goal is to help Spakify identify potential churn users by building and training a binary classifier in order to save the business millions in revenue. More details can be found [blog post](https://wguo.rbind.io/post/sparkify-churn-prediction/)

### Get started

This repository contains the following files:

* `Sparkify_mini.ipynb`: Jupyter notebook that documents the whole model development process on the smaller dataset including exploratory data analysis, data visualization and discussions
* `Sparkify_full.ipynb`: modularized version of `Sparkify_mini.ipynb` used to train the full dataset on the AWS EMR cluster
* `us_region.csv`: [csv file](https://github.com/cphalpert/census-regions) used in `Sparkify_mini.ipynb` that assigns each state to its geographical division (data source: [U.S. Census Bureau](https://www2.census.gov/geo/pdfs/maps-data/maps/reference/us_regdiv.pdf))

#### Prerequisites

To run `Sparkify_mini.ipynb` locally, the following Python libraries are required to be installed in addition to `PySpark`: `Numpy`, `Scipy`, `Pandas`, `Matplotlib`, `seaborn` and `time`.

#### Cloud deployment
To run `Sparkify_full.ipynb` on the AWS EMR, configure your cluster with the following settings:
* Release: `emr-5.20.0` or later
* Applications: `Spark`: Spark 2.4.0 on Hadoop 2.8.5 YARN with Ganglia 3.7.2 and Zeppelin 0.8.0
* Instance type: `m3.xlarge`
* Number of instance: `3`
* EC2 key pair: `Proceed without an EC2 key pair` or feel free to use one

### Troubleshooting

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

```
An error was encountered:
Invalid status code '400' from https://172.31.23.49:18888/sessions/2/statements/19 with error payload: {"msg":"requirement failed: Session isn't active."}
```

### Results

The best results obtained on the test set from the three models are summarized in the table below:

| <sub>Dataset</sub>      | <sub>Parameters</sub>                | <sub>Precision</sub> |                     | <sub>Recall</sub>  |                     | <sub>F1 score</sub> |                     |  <sub>AUC-PR</sub>  |
| :---------------------- | :----------------------------------- | :------------------: | :-----------------: | :----------------: | :-----------------: | :-----------------: | :-----------------: | :-----------------: |
|                         |                                      |  <sub>Overall</sub>  | <sub>Churned</sub>  | <sub>Overall</sub> | <sub>Churned</sub>  | <sub>Overall</sub>  | <sub>Churned</sub>  |                     |
| <sub>Mini dataset</sub> | <sub>maxDepth=4, numTrees=100</sub>  |   <sub>0.86</sub>    | <sub>**0.75**</sub> |  <sub>0.86</sub>   | <sub>**0.60**</sub> | <sub>**0.86**</sub> | <sub>**0.67**</sub> | <sub>**0.77**</sub> |
| <sub>Full dataset</sub> | <sub>maxDepth=10, numTrees=100</sub> |   <sub>0.92</sub>    | <sub>**0.93**</sub> |  <sub>0.91</sub>   | <sub>**0.64**</sub> | <sub>**0.91**</sub> | <sub>**0.76**</sub> | <sub>**0.89**</sub> |


### Acknowledgements
Credit to Udacity for designing the project and hosting the datasets:

* The full Sparkify dataset (12GB) can be found at `s3n://udacity-dsnd/sparkify/sparkify_event_data.json`
* The smaller version (128MB) is available at `s3n://udacity-dsnd/sparkify/mini_sparkify_event_data.json`
  
Blog reference: https://medium.com/@lukazaplotnik/sparkify-churn-prediction-with-pyspark-da50652f2afc
