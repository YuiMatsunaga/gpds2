---
layout: default
title: Classification of 2 Classes (With or Without Tumor)
parent: Problems
nav_order: 1
---

# Classification of 2 Classes (With or Without Tumor)
{: .no_toc }


## 1. INTRODUCTION

The process of diagnosing brain tumors from magnetic resonance imaging (MRI) is often time-consuming. Thus, a rapid analyses through an automated system could help improve the treatment possibilities and optimize hospital resources. The following neural network classifies MRI images as tumoral and non-tumoral.

## 2. MATERIALS AND METHODS
### 2.1 BRATS2020 Dataset

We conducted our experiments using the data available in the Multimodal Brain Tumor Segmentation Challenge 2020 (BRATS2020) dataset [2]. [The BRATS2020 dataset](https://www.med.upenn.edu/cbica/brats2020/data.html) has images of MRI scans and describe a) native (T1), b) post-contrast T1-weighted (T1Gd), c) T2-weighted (T2), and d) T2 Fluid Attenuated Inversion Recovery (T2-FLAIR) volumes. The images were acquired from 371 patients, using diff erent clinical protocols and various scanners from 19 institutions. For each patient, there are four MRI 3D scans. The images have been manually segmented, by one to four raters, following the same annotation protocol. Annotations comprise the GD-enhancing tumor (ET — label 4), the peritumoral edema (ED — label 2), and the necrotic and non-enhancing tumor core (NCR/NET — label 1). In the present study, 8,099 axial 2D MRI slices were extracted from the 3D MRI flair images. The slices were randomly extracted and the masks were used only to indicate the presence of a brain tumor in each slice. Among the 8,099 2D MRI scans, 3,999 scans (49.4%) contain tumors and 50.6% do not, providing a balanced dataset. The patients distribution is shown bellow:

| Type | Number of patients |
| -----------       | ----  |
| Tumor             | 3999  |
| Non Tumor         | 4100  |
| **Total**         |**8099**|

The next image shows some examples:

![DatasetSample](https://user-images.githubusercontent.com/43020938/159187301-9840581a-2006-4eb2-b7b2-a08ea7c0a520.png)

The labels distribution are shown below:

![LabelsDistribution](https://user-images.githubusercontent.com/43020938/160255453-9d1a3a6c-d16c-4985-b213-4944eef0811c.png)

## 3. CNN BRAIN TUMOR DETECTION MODEL

To identify if an image has a tumor, we used a CNN-based classification system. More specifically, we considered the Resnet34 architecture. While training the architecture, we used the cross-entropy as the loss function and Adam as the optimizer. We considered batches of 64 sub-images with a default *learning rate* of 0.002. All evaluated architectures were pre-trained, using weights imported from the *Imagenet* dataset. The last layer of the network was trained using the BRATS2020 dataset. To perform this training, we used 80% of the images for training and 20% of the images used for test. As mentioned earlier, each of these subsets contains approximately 50% of the images with tumors and the remaining 50% of the images without tumors. The patients from the training set were different from those in test set. To avoid any bias in the dataset split, a 𝑘 -fold was used with 𝑘 = 10. The summary of the dataset split is shown below:

| Split     | Tumor | Non Tumor | Total |
| :-:       |  :-:  |     :-:   | :-:   |
| Train     | 3199 | 3272 | **6471**|
| Test      | 800 | 828 | **1628** |

We created the model using [fastai2](https://docs.fast.ai/) libraries that use Pytorch. Fastai allows the creation and testing of neural network models fastly
because it abstracts various complexities from Pytorch. The definition of the neural network model is done in the creation of the dataloader, as shown below:

```python
  dls_train = ImageDataLoaders.from_df(pd_train, path=path,valid_col='is_valid', seed=42,label_col=1)
  learn = cnn_learner(dls_train, resnet34, metrics=error_rate)
```
Other available fastai models can be used or can be found [here](https://github.com/fastai/fastai2/blob/master/fastai2/vision/models/__init__.py).

## 4. RESULTS

In this study, we simulated the introduction of some artifacts that may be present in magnetic resonance images , such as:
gaussian noise, blurring, ringing, ghosting and contrast. For each type of artifact, we tested 20 degradation levels numbered
from 1 to 20. The level 1 is the smoothest and level 20 is the most severe. Then we classified the test images with each of the degraded levels. 
The model previously trained with non-degraded images and the classification results (accuracy, precision, f1_score and recall) 
are shown in the following sections. The PSNR and MSE values of each degradation level correspond to the average of the MSE and PSNR values
of each degraded image in the testset. The degraded test images can be found [here](https://drive.google.com/drive/u/0/folders/1EXKXsPn5fSyOsk54ZZdLQjrFKNLO-Vjb).

### 4.1 Original Images
The results for classifying the existence or not of a tumor are given by the table and by the confusion matrix, Figure [1], shown below:

|  | **Accuracy**  | **Precision** | **F1 score** | **Recall** |
| -----------      | ----  | ----  | ----  | ---- |
| **Mean**         | 0.8944 | 0.8882 | 0.8952 |  0.9027 |
| **Std**          | 0.0078 | 0.0090 | 0.0084 |  0.0174 |

![CM_original_images](https://user-images.githubusercontent.com/43020938/154949534-c08ef5a1-de39-42b3-8aea-6e8b46fae8f4.png)

In Figure [2], the heatmap of an image sample indicates the most discriminative parts used by the resnet34 for classification. The areas with red color were the ones that had the greatest weight in the network decision and the areas closest to blue were the least significant regions.

![Heatmap_original_image](https://user-images.githubusercontent.com/43020938/154948876-aba9e571-3df8-4ed2-93fd-8a2f7c1189cd.png)

### 4.2 Images with Noise
Below are examples of images with different noise levels:
![Noisy_Images](https://user-images.githubusercontent.com/43020938/156930924-ff2a0a70-d7f0-4bc4-b72c-3b2dcdb5838a.png)

The following table contains the classification results after adding gaussian noise to the images:

![Noise_Results](https://user-images.githubusercontent.com/43020938/156947241-3e1e7f06-bd67-4ee0-84eb-abf06124795b.PNG)

![Noise_MSE_Accuracy](https://user-images.githubusercontent.com/43020938/156947255-ffc84264-556b-4032-b13a-bac484e1d3c6.png)

### 4.3 Images with Blur
Below are examples of images with different blurring levels:
![Blur_Images](https://user-images.githubusercontent.com/43020938/156930936-611fb586-b6f9-43db-afa4-2a2f6e1eb58e.png)

The following table contains the classification results after blurring the images:

![Blur_Results](https://user-images.githubusercontent.com/43020938/157022746-66b8e35a-1150-4c1c-8ba9-b01242a5fdbd.PNG)

![Blur_MSE_Accuracy](https://user-images.githubusercontent.com/43020938/157022792-ae3af73e-80e9-4f76-a5ac-9c659c2a82bd.png)


### 4.4 Images with Contrast
Below are examples of images with different contrast levels:
![Contrast_Images](https://user-images.githubusercontent.com/43020938/156930945-afb82f87-55f0-4c78-ba78-d9a9a37b65d6.png)

The following table contains the classification results after changing contrast effect to images:

![Contrast_Results](https://user-images.githubusercontent.com/43020938/157148132-3094da42-2544-4c44-8cc5-a375f315a686.PNG)

![Contrast_MSE_Accuracy](https://user-images.githubusercontent.com/43020938/157148191-e42ce445-48ff-4d34-b497-5ee1d3a92754.png)


### 4.5 Images with Ringing
Below are examples of images with different ringing levels:
![Ring_Images](https://user-images.githubusercontent.com/43020938/156930957-3e30489a-0a1b-4a60-9708-9a95e7dd0209.png)

The following table contains the classification results after adding ringing effect to images:

![Ringing Results](https://user-images.githubusercontent.com/43020938/156931530-491175ea-f4e6-4643-8aeb-f41ae9605f90.PNG)

![Ring_MSE_Accuracy](https://user-images.githubusercontent.com/43020938/156931551-7137ae34-7a50-489c-93c6-b975ab2e1653.png)


### 4.6 Images with Ghosting
Below are examples of images with different ghosting levels:
![Ghost_Images](https://user-images.githubusercontent.com/43020938/156930978-dc6c93b2-bba3-476c-a383-bb874e333e18.png)

The following table contains the classification results after adding ghosting effect to images:

![Ghosting_Results](https://user-images.githubusercontent.com/43020938/156935603-8572b22b-ffd8-4338-a0e2-cb81b870d304.PNG)

![Ghost_MSE_Accuracy](https://user-images.githubusercontent.com/43020938/156935616-45d21212-8546-4603-81e7-b7733ea2c538.png)

## 5 References

1. Using a Saliency-Driven Convolutional Neural Network Framework for Brain Tumor Detection
2. “Multimodal brain tumor segmentation challenge 2020: Data,” https://www.med.upenn.edu/cbica/brats2020/data.html,
accessed: 2022-01-01.




