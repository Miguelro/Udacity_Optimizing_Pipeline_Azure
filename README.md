# Optimizing an ML Pipeline in Azure

## Overview
This project is part of the Udacity Azure ML Nanodegree.
In this project, we build and optimize an Azure ML pipeline using the Python SDK and a provided Scikit-learn model.
This model is then compared to an Azure AutoML run.

## Useful Resources
- [ScriptRunConfig Class](https://docs.microsoft.com/en-us/python/api/azureml-core/azureml.core.scriptrunconfig?view=azure-ml-py)
- [Configure and submit training runs](https://docs.microsoft.com/en-us/azure/machine-learning/how-to-set-up-training-targets)
- [HyperDriveConfig Class](https://docs.microsoft.com/en-us/python/api/azureml-train-core/azureml.train.hyperdrive.hyperdriveconfig?view=azure-ml-py)
- [How to tune hyperparamters](https://docs.microsoft.com/en-us/azure/machine-learning/how-to-tune-hyperparameters)


## Summary
**In 1-2 sentences, explain the problem statement: e.g "This dataset contains data about... we seek to predict..."**
The dataset contains customer information on marketing campaigns of a financial institution. We seek to predict whether the customer will contrat a product or not (variable `y`). Therefore, we are facing a classification task.

**In 1-2 sentences, explain the solution: e.g. "The best performing model was a ..."**

## Scikit-learn Pipeline
**Explain the pipeline architecture, including data, hyperparameter tuning, and classification algorithm.**

An initial code `train.py` was provided, which is responsible for:
  * Creating a `TabularDatasetFactory` from an online CSV file.
  * Data cleansing and enconde categorical data into numeric format using one-hot encoding or ordinal encoding when applicable.
  * Splitting the resulting data into train and test subsets. This is a common practice to ensure that the trained model is evaluated with a different subset than the one used to train it.
  * Using the function `LogisticRegressionClassifier` from `scikit-learn`, which implements the Logistic Regressión algorithm. This model tries to find the underlying relationship between the target variable `y` and the rest of the features in the dataset.
  * Saving the model as a pickle file, recording the value of its hyperparameter.

The input arguments of `train.py` script are the two hyperparameters that will be used to train the `LogisticRegressionClassifier`.
  * The first is the hyperparameter `C`, which represents the inverse of regularization strength. Smaller values cause stronger regularization.
  * The second is the hyperparameter `max_iter`, which represents the maximum number of iterations to converge.

Finally, the implementation of the hyperdrive job is located in a Jupyter notebook called `udacity-project.ipynb`. This job will execute the `train.py` script several times with different hyperparameter configurations. These configurations are specified by a parameter sample (`RandomParameterSampling`), that will randomly choose from the range of values specified by the user. Moreover, a policy is defined (`BanditPolicy`) for early stopping in case some specific conditions are met.


**What are the benefits of the parameter sampler you chose?**

It explores randomly a hyperparameter space, saving a lot of time compared to an exhaustive search.

* The hyperparameter `C` take a uniform distribution bounded to two values.
* The `max_iter` parameter can be chosen from a given set that ranges from 10 to 100.

**What are the benefits of the early stopping policy you chose?**
We chose a `BanditPolicy`, that defines an early stopping policy based on slack criteria and a frequency and delay interval for evaluation.

The `slack_factor` is the ratio used to calculate the allowed distance from the best performing run. We chose a value of 10%, so if in each time the policy is evaluated (as defined by `evaluation_interval`), the metric falls below the slack respect the best performing model, the job is terminated. This allows us to finish before reaching the maximum number of iterations (`max_total_runs`)  if the model is not improving with each iteration, therefore saving computing time.

## AutoML
**In 1-2 sentences, describe the model and hyperparameters generated by AutoML.**
The AutoML configuration that has been defined establishes that the maximum time the experiment should run is 30 minutes. Moreover, we banned the use of `LogisticRegresionClassifier` model, since it is the one that we have tested in the previous section.

So as to try to work under the same conditions as the previous experiment, we chose the `accuracy` as the primary metric to maximize, and we decided not to use cross-validation, as it would be easier to compare pure performance of both solutions if we kept all modelling decisions as similar as possible.

AutoML generated a total of 36 pipelines, given that they have been the models that it has managed to train within the defined 30-minute limit. The best perfoming one (Accuracy 91.9391%) was a VotingEnsemble composed of a pool of 7 weaker-learning pipelines that together form a strong learner. The second best performing model (Accuracy 91.77%) was an XGBoost Classifier with a StandardScalerWrapper preprocessing.

The hyperparameters of the VotingEnsemble can be obtained by printing `automl_run.get_output()`. This will show the individual hyperparameters of each of the 7 pipelines that compose the model. The models that compose the ensemble are 5 `XGBoostClassifier`, a `LightGBMClassifier` and a `SGDClassifierWrapper`. The weights of the VotingEnsemble are the following:
```weights=[0.1111111111111111, 0.2222222222222222, 0.2222222222222222, 0.1111111111111111, 0.1111111111111111, 0.1111111111111111, 0.1111111111111111]```. For more information, please check the corresponding section of `udacity-project.ipynb`.

The highest weights are assigned to a `XGBoostClassifier` and the `LightGBMClassifier` model.

## Pipeline comparison
**Compare the two models and their performance. What are the differences in accuracy? In architecture? If there was a difference, why do you think there was one?**

## Future work
**What are some areas of improvement for future experiments? Why might these improvements help the model?**

## Proof of cluster clean up
**If you did not delete your compute cluster in the code, please complete this section. Otherwise, delete this section.**
**Image of cluster marked for deletion**
