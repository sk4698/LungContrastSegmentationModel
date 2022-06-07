# LungContrastSegmentationModel
U-Net architecture deep learning model for fast CBF F-MRI acquisition without contrast agent.

# Motivation

It has recently been discovered that frequent use of Gadolinium as a contrast enhancement method in MRI scans can cause the accumulation of Gadolinium in the brain and subsequent negative effects. We seek to explore the idea that a commonly acquired structural MRI has inherent contrast differences that cannot be seen with the normal eye but a computer could see. We look further into this idea by developing and expanding upon previously studied deep learning models. We found that this model performs well in mapping areas of high contrast in the brain of mice and validates the idea that a deep learning approach could potentially eliminate the need for Gadolinium use as contrast enhancement.

# Method Description

U-Net based architecture was implemented, with a few key modifications. During the course of this project, a few separate models were trained and evaluated with differing levels of complexity and a different number of added features. In doing so, we are able to generate a quantitative measure of each model’s performance. More importantly though, we are able to analyze in detail the positive effect which is brought on by each feature of the model beyond the standard U-Net architecture.

![Figure 1](https://user-images.githubusercontent.com/72621878/172488381-5afb9a68-f98f-491e-aa81-9b0b4c156632.png)
![unnamed-2](https://user-images.githubusercontent.com/72621878/172488940-57f9693d-d299-4993-9be3-a00692648267.png)

Figure 1 shows the overall architecture of the neural network which was collectively implemented in the execution of this project. Note the general properties of a U-Net architecture with associated down-sampling and up-sampling branches. Much like with typical U-Net approaches, the down-sampling branch of the model is primarily responsible for the identification of features within the image. At every step of the down-sampling process, a more general feature map is created (as evidenced by the ever reducing image resolution) which aims to describe key features present within the input image.

Within each of the down-sampling layers, there exists a convolution, batch normalization, activation function and maximum-pooling layer. These layers are responsible for the detailed feature extraction and down-sampling which is achieved at each layer.

The primary role of the up-sampling branch within the model architecture is to convert the feature maps created within the down-sampling branch into a detailed “image-level” representation of the features. It is this process that performs the coloring of each individual pixel within the image based on the features which have been identified in the down-sampling portion of the neural network network.

Similarly to the down-sampling architecture, each up-sampling layer consists of a convolution layer, batch normalization layer and upsampling layer, whose purpose is to perform the feature representation conversion described above.

In common with many other published implementations of U-Net architectures, this model also makes use of a series of concatenation blocks which serve to ensure the reconstruction of feature information during the up-sampling phase is grounded by information used in the creation of feature maps. This serves to ensure that the reconstruction of features during up-sampling is spatially grounded by the extraction of the features in the first place. This helps improve the spatial accuracy of model predictions and specifically of the highlighting of high contrast features in this specific use case.

Note, however, that the described model architecture differs from that of a conventional U-Net model in two distinct ways. Firstly, the model makes use of residual blocks at every stage of both the down-sampling and up-sampling branches. These residual blocks serve to improve the accuracy of model predictions and improve training efficiency by allowing the flow of information from previous layers to non-adjacent later layers. In the case of this model, it allows information to flow from the early layer of a particular up or down-sampling group of layers to subsequent layers in that up or down-sampling group of layers.

The second key difference in this proposed model is the use of attention gates across each up-sampling process in the model. These attention gates are crucial in suppressing the prediction responses in “irrelevant background regions”, which helps ensure that areas of high contrast are not incorrectly predicted to occur in background regions of the image.

With this initial model successfully implemented and quantified, a subsequently improved model was then implemented.  This second implementation contained not only model improvements but also an increased dataset as will be detailed below.

Firstly, the residual blocks and the attention gates which had been left out in the previous implementation of the model were included in the created neural network.  This led to a model which fully matched Figure 1.  The inclusion of the fully representative model was critical as it included many of the key features which had previously been shown to lead to improved prediction performance, and will be discussed at length below.  The two specific features newly implemented in the second iteration of the model training were the residual block and the attention gates.

Secondly, a series of image augmentation techniques were implemented on the image dataset in order to increase the size of the image dataset.  This was performed in response to an observation during the first iteration of the model that the model appeared to be very quickly overfitting due to the limited dataset available.  For example, in referencing the learning curves in Figure 2, it can be seen that the rapid plateauing of the training loss after a very sharp initial drop is a clear characteristic of overfitting, which can be manifested due to an insufficient dataset size.

Because of the challenges in acquiring additional data in the field of biomedicine (both within this project and in the wider world of academia), a series of image augmentation techniques were used to artificially augment the data and thereby increase the size of the dataset.  To do this, one copy of each image in the dataset was re-uploaded into the dataset, but with a random affine transformation applied to it.  As a result of this process, the dataset was now doubled in size with there being two versions of each image (one with the original orientation and the other in a transformed orientation).

Finally, a smaller learning rate of 0.0005 (as opposed to 0.001 in the original iteration) was used.

# Results

As described previously, the initial model was implemented in a piecewise manner, firstly to ensure the rigor of produced results, but also to highlight the importance of and performance improvements brought about by specific features of the model.

In order to test the effectiveness and accuracy of the proposed model, a modified version of the Jaccard Index was used which accounts for the use of grayscale predictions and ground truth images.  A subset of 9 test images was also split off from the combined dataset and sequestered from the model during training.  The remaining images were then used to train and validate the model (in an 80-20 split), before the trained model was tested on the 9 remaining images.

Initially, the aforementioned model was implemented without the attention and residual blocks previously described.  In doing so, a simple U-Net architecture was effectively used to produce the contrast estimations.  Using such a methodology, the model was able to produce a Jaccard Index of 64.2% on the unseen test data.  This optimal result was obtained on the 74th Epoch out of 100 possible training Epochs.  The figure below shows the training and validation losses associated with each training Epoch.

![unnamed-3](https://user-images.githubusercontent.com/72621878/172489336-bfd517ed-7d26-4717-9531-425b02ccfe22.png)

Fig. 2. The learning curve for the simple U-Net architecture when training over 100 Epochs. 

In the subsequent iteration of the model training, after incorporating the improvements, the model was once again trained using the same split of train-validation-test data.  This time, the model achieved an improved Jaccard Index performance on the unseen test data of 68.0%  The model also exhibited a slightly slower training behavior with the best Epoch occurring on the 74 Epoch.

# Discussion

In comparing the performance of the partially implemented model and the fully implemented model (with increased dataset) it can be seen straight away that the model shows significantly improved performance in its second iteration.  This is likely due to two factors.

Firstly, the inclusion of the attention gates and the residual blocks likely play a very significant role in improving the accuracy of the model’s predictions.  This has been demonstrated previously and is a key finding of the study undertaken within this project.  Specifically, the presence of the residual blocks likely increase the spatial accuracy of predictions significantly, especially around complex regions of differing contrast.  In these regions, it is likely that the residual block allows for the forward “leakage” of spatial information through non-adjacent layers, thereby improving model predictions.

The attention gates themselves likely provide some additional benefits to the model’s predictions by allowing the background regions of the images to be effectively ignored.  As will be subsequently discussed below, this has a significant benefit to the model since the model without these attention gates spuriously highlights boundary areas near the brain boundary.  These spurious predictions are significantly improved by the inclusion of the attention gates.

Secondly, the increased size of the dataset likely contributes to the observed improvement in model performance.  The increased amount of data to train on likely reduces the possibility of overfitting in the mode, thereby improving the generalizability of the model and thus improves accuracy.

Across both implementations of the model it was found that the deep learning model produced generally accurate results.  In areas of both high and low contrast, the predicted images were shown to generally predict these regions with moderate accuracy.  That is to say that the model was able to broadly identify areas which responded greatly to the injection of contrast and those which did not.  The figure below shows such a prediction, which broadly matches with the ground truth of this particular patient.

![unnamed-4](https://user-images.githubusercontent.com/72621878/172490025-d682fb0e-a86e-4cb0-acbf-2b39d39b6316.png)

Fig. 3. Final prediction from the improved U-Net architecture which broadly matches the ground truth of the patient.

Despite these broad successes though, there were still a range of shortcomings identified in the produced models.

Firstly, the areas of high contrast in the brain images were correctly predicted generally by the model.  However, these areas of high contrast were “smeared” and generally predicted to be larger in area that they were in the ground truth models.  This can be seen in the figure below, and is thought to be related to improper tuning of the residual blocks within the model.  Incorrect application of these residual blocks could allow for excess “information” to be passed between layers, therefore causing the incorrect expansion of high contrast predictions.

![unnamed-5](https://user-images.githubusercontent.com/72621878/172490130-bd716b63-8525-4e95-ab80-1b4bb73f8d16.png)

Fig. 4. Example where the U-Net model predicted the high contrast areas to be larger than the ground truth.

Secondly, there were commonly observed instances of spurious contrast predictions in the areas of the image surrounding the brain boundary.  This typically manifests itself as incorrect areas of high contrast around the edges of the brain, or even in the surrounding background of the image.  An example of this is shown in the figure below and is thought to be due to a non-optimized application of the attention gates, which has previously been demonstrated to prevent the model from erroneously making contrast predictions in background areas of the image.

![unnamed-6](https://user-images.githubusercontent.com/72621878/172490204-ac1dfbaf-bbca-483e-95e8-0feddf2454d9.png)

Fig. 5. Example of an instance where there are incorrectly areas of high contrast around the edges of the brain.

Finally, across the board, in addition to the erroneous enlargement of high contrast areas in the predictions, there was observed a general smoothing of contrast transitions within the predictions.  That is to say that the transitions between areas of high and low contrast were noticeably less sharp in the predictions than they were in the ground truth.  Again, this is thought to be due to a non-optimal application of the residual blocks and the associated “leakage” of contrast information through layers of the neural network during training.

# Further Study and Next Steps

Despite the shortcomings noted above, it is fair to say that the created model (in both forms) represents a moderately successful attempt at creating a deep learning neural network for the purposes of replacing contrast injection during brain MRI scans.  This has the potential to significantly improve the course of healthcare for patients who receive frequent brain MRI scans by reducing (or even eliminating) the amount of harmful Gadolinium contrast a patient is exposed to during scanning.

With that said though, it is clear that significant further study is required before the deep learning method implemented in this project is of sufficient fidelity to completely and accurately replace Gadolinium contrast in the real world.  However, what we have demonstrated is that these required improvements will likely be in the form of incremental improvements as the overarching viability of this methodology has been demonstrated in this work.  This represents an avenue of investigation with significant potential for feasible implementation in the real world.

Finally, there remains one open question regarding the generalizability of this work.  It is important when training the neural network to ensure that the training data contains brain imaging from patients of a wide range of brain states.  This must include patients suffering from a wide range of ailments, as well as perfectly healthy patients.  This is because patients of different brain healths will have vastly different brain structures which appear differently under MRI scans.  For these deep learning methods to be widely applicable across the population, it is important that the model is exposed to brain images from all of these situations during training.  Given the small size of this dataset it is very unlikely that the dataset contains a wide range of samples from across the “brain health” spectrum, and thus, the generalizability of this model will likely suffer.

# References

Note that portions of this workbook are adapted from Prof. Guo's Assignment 2 and Midterm Exam.

[1] Guo, Jia. Deep Learning in Biomedical Imaging BMEN 4460. Columbia University 2022. https://courseworks2.columbia.edu/courses/145230

Additional adaption of code is indicated in the code base below

[2] https://towardsdatascience.com/residual-network-implementing-resnet-a7da63c7b278
