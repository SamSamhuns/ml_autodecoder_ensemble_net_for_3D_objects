# Ensemble AutoDecoder CompNets for 3D Point Cloud Pair Encoding and Classification

Since the inception of LeNet5, Yan LeCun’s first Convolutional Neural Network for handwritten digit-recognition in 1994, Neural Networks have come a long way in classifying 2D images and extracting their features. However, since we live in a 3D world, it is natural to ask if we can extract the features of 3D shapes and classify them. This turns out to be rather difficult given the exponential increase in complexity with the added third dimension. To perform this classification efficiently, we propose an ensembled encoderless-decoder and classifier neural network to learn the latent encoding vectors for point-cloud pair transformations. This ensemble extracts the feature descriptors of the transformations without a parametric encoder or a lossy 2D projection. We also empirically demonstrate the effectiveness of our ensemble model through a benchmark comparison on a 3D point cloud classification task for the PointNet and ModelNet dataset. Finally, we discuss the applicability of our 3D shape descriptor learners and classifiers as vital preprocessing techniques for eventual 3D object classification, segmentation, and database retrieval tasks.

<div align='center'>

**Ensemble AutoDecoder Point Drift Visualization**

![](./img/diff_class_point_drift.gif)

</div>

## Data

For training and testing our model, we use the PointNet and the ModelNet data.

<div align='center'>

  **[PointNet7](https://stanford.edu/~rqi/pointnet/)**

  <img src='./img/PointNet7.png' width=500 />
</div>

<br>
<br>

| <div align='center'>[ModelNet10](https://3dshapenets.cs.princeton.edu/) | [ModelNet40](https://3dshapenets.cs.princeton.edu/)</div> |
| :---------------------------------------------------------------------: | :-------------------------------------------------------: |
|                        ![](./img/ModelNet10.png)                        |                 ![](./img/ModelNet40.png)                 |

</div>

### Downloading the Dataset

Data can be acquired from the [PointNet](https://stanford.edu/~rqi/pointnet/) and [ModelNet](https://3dshapenets.cs.princeton.edu/)  websites.

## Preprocessing Data

We convert all 3D image data to a `.npy` 3D point cloud format. See [kaolin](https://github.com/NVIDIAGameWorks/kaolin) for conversion from ModelNet `.off` files to `.npy` files.

## Neural Network

### Single AutoDecoder

![](./img/Models/autodecoder.png)

### Single CompNet (Comparison Net)

<div align='center'>
  <img src='./img/Models/compnet.png' />
</div>

## Ensemble Neural Network

### Ensemble AutoDecoder

Uses four AutoDecoder Nets of different sizes to capture the shallow and deeper level features from the original point cloud that will be used for learning the latent transformation encoding between the point cloud pairs.

<div align='center'>
  <img src='./img/Models/ensemble_autodecoder.png' width=500 />
</div>

### Ensemble CompNet 1

Uses 5 similar CompNets that were initialized with different weights to make them independent.

<div align='center'>
  <img src='./img/Models/ensemble_compnets1.png' />
</div>

The AutoDecoder Ensemble Net is able to perform significantly better than the single shallow layered CompNet

### Ensemble CompNet 2

Uses Four CompNets of different sizes to capture the general and deeper level features

<div align='center'>
  <img src='./img/Models/ensemble_compnets2.png' width=500 />
</div>

## Methodology

After converting all 3D data into `.npy` point clouds, we create `DataLoaders` that feeds batches of Point Cloud Pairs of the same and different classes to the AutoDecoder for training.

We first create an embedding or encoding layer that will represent the latent transformation encoding vectors. between point-cloud pairs of the same and different class.

Given the embedding vector and a point-cloud, the AutoDecoder receives two more sample point-clouds, one of the same class and one of a different class.

We train the AutoDecoder's layer weights by learning the geometric transformation from the source point-cloud to the target point-cloud that might be of the same or different class. This learning is done iteratively with the ADAM optimizer where the goal is to minimize the chamfer loss between the source and target point clouds.

<div align='center'>
  <img src='./img/autodecoder_training.png' width=500 />
</div>

This information about the geometric transformation is stored in the embedding or the latent vector that is returned after the training process is done.

<div align='center'>
  <img src='./img/adnet_clf_training.png' width=500 />
</div>

The latent vector that is generated by the AutoDecoder can be supplied to a classifier that can learn whether latent encoding represents point-cloud transformations between the same class or a different class. This information on class transformation can then be used for the classification of the 3D object.

Latent vectors that represent transformations between point-clouds of the same class give a higher similarity score and vice-versa for point-clouds of different classes.

<div align='center'>
  <img src='./img/classifier_training.png' width=500 />
</div>

### Objective function

We minimize the symmetric chamfer loss between point cloud pairs of the same class and different class to get the representative latent encoding transformation vector.

The Chamfer loss is easily differentiable for back-propagation and is robust to outliers in the point cloud.

![](./img/chamfer_loss.png)

### Neural Network Hyper Parameters

One of the main hyper-parameters, the encoding size for the latent vector is set to `256`.

All other Neural Network Parameters are reported in the supplementary material.

### System used for Training and Testing

    -   CPU: Intel(R) Core(TM) i9-9900K CPU @ 3.60GHz
    -   Architecture: 64 Bit
    -   RAM: 62 Gib System Memory
    -   Graphics: 7979 MiB GeForce RTX 2080

# PointNet Data Results

## PointNet7

### Results of Similarity Classification Task on PointNet7

<div align='center'>
  <img src='./img/pointnet7_all_models_benchmark.png'
  width=500 />
</div>

<br>

**Note: The accuracies have an error of 1 percentage points between different runs**

### ROC Curves from the Similarity Classification on PointNet7

| <div align='center'>Single CompNet</div> ![](./img/roc_auc/roc_auc_single_compnet_pnet7.png) | <div align='center'>Ensemble CompNet</div> ![](./img/roc_auc/roc_auc_ensemble_compnet_pnet7.png) |
| :------------------------------------------------------------------------------------------: | :----------------------------------------------------------------------------------------------: |
|                **Logistic Regr** ![](./img/roc_auc/roc_auc_log_regr_pnet7.png)               |                 **Naive Bayes** ![](./img/roc_auc/roc_auc_naive_bayes_pnet7.png)                 |
|               **Decision Trees** ![](./img/roc_auc/roc_auc_dec_tree_pnet7.png)               |           **Random Forest Classifier** ![](./img/roc_auc/roc_auc_rand_forest_pnet7.png)          |

**Figure 1: ROC Curves for the different models on the Classification PointNet7 task**

### Accuracy and Training time comparisons on PointNet7

<div align='center'>
  <img src='./img/pointnet7_all_models_acc_ttime.png'
  width=500  />
</div>

<br>

**Note: The accuracies have an error of 1 percentage points between different runs**

### Ensemble AutoDecoder used with Single and Ensemble CompNets on PointNet7

|                                   |                Single CompNet               |                Ensemble CompNet                |
| :-------------------------------: | :-----------------------------------------: | :--------------------------------------------: |
|  **Single AutoDecoder** `5m 24s`  | `25.4s` ![](./img/roc_auc/PointNet7/ss.png) | `1m 12.2s` ![](./img/roc_auc/PointNet7/se.png) |
| **Ensemble AutoDecoder** `8m 24s` | `25.2s` ![](./img/roc_auc/PointNet7/es.png) |   `14.4s` ![](./img/roc_auc/PointNet7/ee.png)  |

### Performance Metrics from Single and Ensemble AutoDecoders on PointNet7

<div align='center'>
  <img src='./img/pointnet7_single_ensemble_adnet.png' width=700 />
</div>

<br>

**Note: The accuracies have an error of 1 percentage points between different runs**

## Full PointNet

### ROC Curves from the Similarity Classification

**Note: Results for the Neural Network Models models only**

|            Single CompNet (Full PointNet)           |            Ensemble CompNet (Full PointNet)           |
| :-------------------------------------------------: | :---------------------------------------------------: |
| ![](./img/roc_auc/roc_auc_single_compnet_fpnet.png) | ![](./img/roc_auc/roc_auc_ensemble_compnet_fpnet.png) |

AutoDecoder Training time was `11m 45s`; Single CompNet and Ensemble CompNet training times were at `26.6s` and `13.1s` respectively.

# ModelNet Data Results

## ModelNet10

### Performance Metrics from Single and Ensemble AutoDecoders on ModelNet10

<div align='center'>
  <img src='./img/ModelNet10_single_ensemble_adnet.png' width=700 />
</div>

<br>

**Note: The accuracies have an error of 2 percentage points between different runs**

### ROC Curves from the Similarity Classification on ModelNet10

|                                   |                Single CompNet                |               Ensemble CompNet               |
| :-------------------------------: | :------------------------------------------: | :------------------------------------------: |
|  **Single AutoDecoder** `5m 59s`  | `10.0s` ![](./img/roc_auc/ModelNet10/ss.png) | `27.7s` ![](./img/roc_auc/ModelNet10/se.png) |
| **Ensemble AutoDecoder** `8m 41s` | `9.83s` ![](./img/roc_auc/ModelNet10/es.png) | `27.2s` ![](./img/roc_auc/ModelNet10/ee.png) |

## ModelNet40

### Performance Metrics from Single and Ensemble AutoDecoders on ModelNet40

<div align='center'>
  <img src='./img/ModelNet40_single_ensemble_adnet.png' width=700 />
</div>

<br>

**Note:**

-   The accuracies have an error of 3 percentage points between different runs
-   The Dataset contains samples from only 33 classes for training and testing as the conversion module (`kaolin`) we were using could not resolve some of the `.off` files in the ModelNet40 dataset. The used classes are mentioned in the supplementary material.

### ROC Curves from the Similarity Classification on ModelNet40

|                                    |                Single CompNet                |                Ensemble CompNet               |
| :--------------------------------: | :------------------------------------------: | :-------------------------------------------: |
|  **Single AutoDecoder** `11m 39s`  | `47.1s` ![](./img/roc_auc/ModelNet40/ss.png) |  `2m 9s` ![](./img/roc_auc/ModelNet40/se.png) |
| **Ensemble AutoDecoder** `16m 51s` | `47.8s` ![](./img/roc_auc/ModelNet40/es.png) | `2m 12s` ![](./img/roc_auc/ModelNet40/ee.png) |

## Conclusion

In general, no matter whether the AutoDecoder has a single Neural Network or an ensemble Neural Network, the ensemble Comparison Nets (CompNets) always outperform all the Single CompNets in classification tasks for the same and different class. In general there is an average improvement of 10% in accuracy when switching to an ensemble classifier network for all of the different datasets that were used.

We also show that our neural network classification models (Single and Ensemble CompNets) always outperform all other classical supervised machine learning models in accuracy, f1-scores and Receiving Operator Characteristic Area Under Curve (ROC-AUC) for the 3D object classification task (See Table 1).

The training times for the different datasets also show that our method computes latent encodings and does the classifications very quickly given our training system statistics.

However, the efficacy of an ensemble AutoDecoder network was not observed for all datasets, especially for `PointNet7`, where the total accuracy and related metrics actually decreased.

One of the reasons for this reduced accuracy for the Ensemble AutoDecoder Net could be the highly imbalanced nature of training set for the `PointNet7` dataset.

<div align='center'>
  <img src='./img/pointnet7_counts.png' width=500 />
</div>

## Running the AutoDecoder CompNet Ensemble

### Set up Virtual Environment

Note: `Anaconda` can also be used for the venv

```shell
$ python -m venv venv
$ source venv/bin/activate
$ pip install -r requirements.txt
```

To start a new Jupyter Notebook kernel based on the current virtual environment and run a new Jupyter Notebook Instance:

```shell
$ python -m ipykernel install --user --name ENV_NAME --display-name "ENV_DISPLAY_NAME"
$ jupyter notebook
```

All information to run the ensemble neural network can be found in the `autodecoder_ensemble_net`.

The `geotnf` package and `LoaderFish.py` files are based on implementations from PR-Net `(Wang et al. 2019)`

## References

All References have been mentionned in the `References` section in `Ensemble AutoDecoder CompNets for 3D Point Cloud Pair Encoding and Classification.pdf`
