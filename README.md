# Journey to Springfield: Simpsons Classification

This repository shows the work, submitted as part of the coursework from the [Deep Learning School](https://en.dlschool.org/). The coursework was set as an in-class [Kaggle competition](https://www.kaggle.com/c/journey-springfield/overview). Full Course for Part 1 of Deep Learning School (Spring 2020) is available at: https://stepik.org/course/65389/syllabus.

**Note,** the course was taught in Russian. Parts of the notebooks in Russian were left from the `simpsons-baseline.ipnb` provided as a starting point by course organisers. My comments are added in English.

## Coursework Details

- **Title:** Journey to Springfield: Simpsons Classification
- **Type:** Image Classification Coursework
- **Author:** George Batchkala, george.batchkala@gmail.com
- **Data:** [Journey to Springfield](https://www.kaggle.com/c/journey-springfield)
- **GitHub repository:** https://github.com/GeorgeBatch/simpsons-classification

## Coursework Description

The Simpsons TV-show has been filmed for over 25 years. Some of the characters were there for many years and their looks changed significantly. The aim of this work is to train a model, which can accurately name different Simpsons characters.

### Data

The data consisted of:
- **20933** labeled images, which were used for **training** and model **validation**
- **991** unlabeled images, for which class labels needed to be provided (**test** set)

The images were unevenly split across **42 different characters** (classes).

### Metric

The metric used in a competition was the mean [F1-Score](https://en.wikipedia.org/wiki/F1_score), which is commonly used for [multiclass classification](https://en.wikipedia.org/wiki/Multiclass_classification) problems such as this one.

### Goal

The coursework was marked according to performance on the Kaggle public test set. All works with F1-score above 0.97 were awarded full marks.

Achieving F1-score of more than **0.97** on the test set with less than 10 submissions became the primary goal of this project. The restriction on the number of submissions is useful not to overfit on the public score of the test set, since at this point it stops reflecting true generalisation performance (see more in a [blog post](http://blog.mrtz.org/2015/03/09/competition.html) by Moritz Hardt).

## Results

All **notebooks** can be found in the `~/notebooks` directory. **Weights** for the more sophisticated models (V3-V5) are saved in the `~/weights` directory.

The **best submission** is saved in the root directory as: `efficientnet-b2_3FeatureExtrEpochs-50FinetuningEpochs-submission.csv`


### V1: Baseline (simple encoder architecture)

I first tried to get results from a baseline pipeline provided as a starting point by the course organisers. I debugged and added comments to it.

- Notebook: `v1-simpsons-baseline-debugged-f1-test-0-6695.ipynb`
- F1-score on the test set: **0.670**


### V2: Adding Augmentation and Increasing the Number of Epochs

After fitting the baseline model I added an augmentation stage for training and increased the number of epochs. The performance **improved by 0.085**.

- Notebook: `v2-augmentation-more-epochs-f1-test-0-75451.ipynb`
- F1-score on the test set: **0.755**

**Note,** we use the augmentation added at this point for all the subsequent experiments.

### V3: Fine-tuning ResNet-18 (3 + 25 epochs)

As my next step, I decided to use a light version of [ResNet](https://arxiv.org/abs/1512.03385) network, **ResNet-18** with weights pretrained on [ImageNet](http://www.image-net.org/) dataset, instead of a simple encoder architecture which was provided as a baseline.

I changed the last year to use only 42 classes as opposed to 1000 classes from ImageNet. I first trained only the last-layer weights for 3 epochs, and then then trained the whole network for another 25 epochs. The results **improved by 0.193**.

- Notebook: `v3-finetuning-resnet-18-f1-test-0-94792.ipynb`
- Weights: `resnet-18_3FeatureExtrEpochs-25FinetuningEpochs-weights.pth`
- F1-score on the test set: **0.948**

### V4: Fine-tuning ResNet-18 for longer (3 + 50 epochs with early-stopping)

Now, scoring above 0.9 and getting closer to the goal I decided to fine-tune the same exact model for longer. Instead of training all weight for 25 epochs, I decided to do it for 50, but use [early-stopping](https://towardsdatascience.com/early-stopping-a-cool-strategy-to-regularize-neural-networks-bfdeca6d722e) with a patience of 5 epochs to avoid overfitting on the test set. The results **dropped by 0.033**. Either early-stopping could not save the model from overfitting. Seeing a learning curve for both train and validation sets flatten over the 50 epochs, I decided that the model was not complex enough to learn the features it needed for a better performance.

- Notebook: `v4-finetuning-resnet-18-f1-test-0-94792.ipynb`
- Weights: `resnet-18_3FeatureExtrEpochs-50FinetuningEpochs-weights.pth`
- F1-score on the test set: **0.915**

### V5: Fine-tuning EfficientNet-b2 (3 + 50 epochs  with early-stopping)

I needed a better model with similar computational complexity. The recently developed by Mingxing Tan and Quoc V. Le [EfficientNet](https://arxiv.org/abs/1905.11946) provided exacly a solution like that. Like ResNet, EfficientNet comes with different versions each of which requires different computational resources for the training to converge. I decided to use one of the lighter versions, of **EfficientNet-b2** to keep the computational cost similar to one of **ResNet-18**. Training it for 50 epochs with early-stopping improved the performance over the best performing **V3** version by **0.049**.

- Notebook: `v5-finetuning-efficientnet-b2-f1-test-0-99681.ipynb`
- Weights: `efficientnet-b2_3FeatureExtrEpochs-50FinetuningEpochs-weights.pth`
- F1-score on the test set: **0.997**

After reaching the my goal of exceeding the F1-score of **0.97** I decided not to waste computational resources provided by Kaggle and minimise the carbon footprint left by this project. I am confident enough that running the same network, or maybe a **b3** version of EfficientNet, for 100 epochs would result in a perfect F1-score of 1.00000, but I do not see any use of it.


### Results Summary

Version | Epochs  | Time (s) | Score
 -----  | -----   | -----    | -----
1       | 2       | 440      | 0.670
2       | 5       | 1086     | 0.755
3       | 25      | 4870     | 0.948
4       | 50 (es) | 8308     | 0.915
5       | 50 (es) | 11492    | **0.997**

Where "es" stands for early-stopping. **Note,** in both cases, early-stopping with patience of 5 did not stop the training, which means that the validation scores were still improving up till the last epoch.

## Conclusion

I was able to achieve my goal of exceeding the F1-score of **0.97** by fine-tuning pretrained **EfficientNet-b2** and using **augmentations at the training stage**.
