# Summary of 1_Baseline

[<< Go back](../README.md)


## Baseline Classifier (Baseline)
- **n_jobs**: -1
- **explain_level**: 2

## Validation
 - **validation_type**: split
 - **train_ratio**: 0.75
 - **shuffle**: True
 - **stratify**: True

## Optimized metric
accuracy

## Training time

0.3 seconds

## Metric details
|           |    score |   threshold |
|:----------|---------:|------------:|
| logloss   | 0.685824 |  nan        |
| auc       | 0.5      |  nan        |
| f1        | 0.71831  |    0.504596 |
| accuracy  | 0.56044  |    0.504596 |
| precision | 0.56044  |    0.504596 |
| recall    | 1        |    0.504596 |
| mcc       | 0        |    0.504596 |


## Metric details with threshold from accuracy metric
|           |    score |   threshold |
|:----------|---------:|------------:|
| logloss   | 0.685824 |  nan        |
| auc       | 0.5      |  nan        |
| f1        | 0.71831  |    0.504596 |
| accuracy  | 0.56044  |    0.504596 |
| precision | 0.56044  |    0.504596 |
| recall    | 1        |    0.504596 |
| mcc       | 0        |    0.504596 |


## Confusion matrix (at threshold=0.504596)
|              |   Predicted as 0 |   Predicted as 1 |
|:-------------|-----------------:|-----------------:|
| Labeled as 0 |                0 |               80 |
| Labeled as 1 |                0 |              102 |

## Learning curves
![Learning curves](learning_curves.png)
## Confusion Matrix

![Confusion Matrix](confusion_matrix.png)


## Normalized Confusion Matrix

![Normalized Confusion Matrix](confusion_matrix_normalized.png)


## ROC Curve

![ROC Curve](roc_curve.png)


## Kolmogorov-Smirnov Statistic

![Kolmogorov-Smirnov Statistic](ks_statistic.png)


## Precision-Recall Curve

![Precision-Recall Curve](precision_recall_curve.png)


## Calibration Curve

![Calibration Curve](calibration_curve_curve.png)


## Cumulative Gains Curve

![Cumulative Gains Curve](cumulative_gains_curve.png)


## Lift Curve

![Lift Curve](lift_curve.png)



[<< Go back](../README.md)
