# genetics-l2g-explainability
This is a webapp where the user can explore what is the feature contribution of a L2G prediction using [[SHAP]] on the [[L2G Pipeline]] data.

## Context
The L2G model is currently divided into 5 models that are based on the set of chromosomes that have been used to train the model. 
Since the training set is divided into 5 different sets of chromosomes, the L2G model is composed with 5 classifiers.

Each of the L2G models consists of a **binary class classifier**, where the outcomes/classes are whether or not a gene can be considered as a gold standard for that locus/study.

For that, a feature matrix is built for each **locus/study/gene combinations** and the output of the model is the probability of a gene being causal for that locus/study.

The predictions for all loci are precomputed in a way that we can get the most likely causal gene for a locus/study if we get the gene with the highest L2G score.
The predictions dataset has a significant size with:
- 323 604 locus/ study pairs on which we apply the model.
	- 23 395 different studies
	- 151 294 different loci
- 19 247 genes that act as different classes to the classifier

## Calculation of all SHAP values
The SHAP value tells you how much that feature has contributed to the prediction for that particular sample. So this value is calculated *for each prediction* and has the form of an array of size = number of features (47, in this case). 

In a preprocessing step, I have enriched a dataset containing:
	1. the predictions + 
	2. the feature matrix with that prediction + 
	3. the name of the file containing the SHAP values in the bucket `gs://ot-team/irene/l2g_explainability/`
	4. the index of that record in the shap values array

The original idea was to include the SHAP values in the predictions dataset, but this is not possible because these values are stored in the shape of a `shap.Explanation.explanation` type, that couldn't be stored in a Spark or Pandas DF.

- The calculation of the SHAP values is precomputed because it is a costly computation. It takes a little bit more than 10 minutes to compute and store the shap values for all predictions.
- Once these are precomputed, they are serialised using `pickle`.

## L2G Global feature contribution
Relative importance of the different features in the L2G model based on the mean of each shap value.


## Local feature contribution
Features for each loci are not distributed uniformly. For example, the model is biased towards considering distance-related features with higher importance just because they are always present, whereas features that describe variant pathogenicity or colocalising molecular QTLs are less important because they are less present.

I am interested in studying the contribution of each feature for each prediction as I expect this to vary depending on the features' presence.