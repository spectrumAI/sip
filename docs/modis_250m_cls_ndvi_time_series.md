# Feature Importance Evaluation and Classifier Comparison

## Tutorial Objectives

* Train dip-res model on Saskatchewan province and test on Alberta prvince using EVI time series.

* Compare performances of the same classifier using different time series input.

* Compare performances of different classifiers using the same input. 

## Key challenges

* Strong ***signature variabilities*** of different land cover classes in EVI time series.

## Study area

* Canada (https://atlas.gc.ca/lcct/en/index.html, http://130.179.67.140/dataset/f2f37ad5-0afa-4fc4-885d-6a6e9f8b6dc8/resource/04a7532d-a7b0-416d-939a-fb950ca8012e/download/iso19131aafcannualcropinventorydataproductspecifications.pdf,https://open.canada.ca/data/en/dataset/ba2645d5-4458-414d-b196-6303ac06c1c9#wb-auto-6, http://www.cec.org/sites/default/nalcms/CEC_picmap_EN/index.html)

## Classifiers

* Rondom forest (***rf***), which has been widely used in MODIS time series classification

* Supported vector machine (***svm***)

* Patch-based spatial-spectral residual (***ss-res***) convolutional neural network 

* Patch-based spectral residual (***spe-res***) convolutional neural network

* Image-based fully convolutional residual (***fcn-res***, called dip-res in SIP) network  

## Experiment 1: Train fcn-res/dip-res on Saskatchewan and test on Alberta using EVI time series

* **Generate EVI time series training blocks for Saskatchewan**

    1. Download and process data according to [the preprocessing tutorial](./modis_250m_preprocessing.md).
    1. Assume that the final train blocks are saved into ***MOD13Q1_preprocessed_imgs_sask_evi***
 
* **Generate EVI time series training blocks for Alberta**

    1. Download and process data according to [the preprocessing tutorial](./modis_250m_preprocessing.md).
    1. Assume that the final train blocks are saved into ***MOD13Q1_preprocessed_imgs_alta_evi***

* **Prepare data for Saskatchewan**
    1. Open ***config_os.yaml*** and ensure that ***raw_img_dir*** points to the folder with preprocessed time series blocks, i.e., ***MOD13Q1_preprocessed_imgs_sask_evi***, ***raw_band_names*** consists of '\*all-channels.tiff', ***dirs -> data/mask/save*** are set using your own directories, ***num_classes*** equals 15, ***is_colors_defined_in_rgb_values*** is True, and ***my_colors*** consists of 17 colors of rgb values, and ***my_classes*** consists of 17 class names. Out of the 19 classes, Class3, Class4, Class7 and Class9 should not be used.  
    1. Open ***config_os.yaml***, under ***random_split_samples_params***, set ***to_random_split_samples*** and ***to_split_on_image_level*** to be ***True***. Set ***train_proportion*** to be ***0.8***, ***val_proportion*** to be ***0.2***, and ***test_proportion*** to be ***0***.
    1. Run SIP, and click on ***Classification -> Prepare data***. Once finished, go to ***dirs -> data -> train*** defined in the config file and open ***data-file.yaml*** to take a look at the training block list. 
 
* **Train dip-res model on Saskatchewan**

    1. In ***config_os.yaml***, under ***train_params***, ***lr*** equals 0.0001, ***batch_size_train/val/test/predict*** equals 2, ***net_type*** is ***dip-res***. 
    1. Save ***config_os.yaml***.
    1. Run SIP, click on ***Train classifier*** under ***Classification***, choose the ***config_os.yaml*** you edited. 
    1. If you want to use the save model file to continue training, set ***to_resume_from_latest_checkpoint*** to be ***True*** and also set ***resume_checkpoint_file*** to the model file under ***dirs -> save -> model*** defined in ***config_os.yaml***.  
    1. Record the training and validation accuracies.

![](../pics/dipres_evi_sask_epoch.png)
 

* **Mosaic training classification maps for Saskatchewan**
    1. In ***Config_os.yaml***, under ***Modis_params***, set ***Modis_maps_merge_params*** to be ***True***, set ***input_dir_maps_merge_Modis*** to be ***dirs -> save -> train*** folder defined in ***config_os.yaml*** where you have the classification maps of training images, and set ***maps_merge_Modis_id_str*** to be '\*predict-map\*geocoded.tiff'. 
    1. In ***Config_os.yaml***, under ***Modis_params***, disable all the other preprocessing steps by setting ***to_XXXX_Modis*** to be ***False***.
    1. Run SIP, click on ***Preprocessing -> Modis*** to mosaic the classification maps. Once finished, go to ***output_dir_maps_merge_Modis*** defined in ***config_os.yaml*** to take a look at the mosaiced map. Open it in QGIS and compare it with the true map of Saskatchewan. 

    1. The figure below shows training classification map (the one with black missing blocks) vs. ground truth map (the one with red urband classes).

![](../pics/legend_canada_modis.png)
![](../pics/dipres_evi_sask_train_map.gif)


* **Mosaic validation classification maps for Saskatchewan**
    1. Copy ***data_file.yaml*** under the ***dirs -> data -> val*** folder defined in ***config_os.yaml*** to ***dirs -> data ->predict***. 
    1. In SIP, run ***Classification -> Predict Image*** to generate classification maps of validation images, and save them into ***dirs -> save -> val*** folder defined in config_os.yaml. 
    1. In ***config_os.yaml***, under ***Modis_params***, set ***Modis_maps_merge_params*** to be ***True***, set ***input_dir_maps_merge_Modis*** to be ***dirs -> save -> predict*** folder defined in ***config_os.yaml*** where you have the classification maps of validation images, and set ***maps_merge_Modis_id_str*** to be '\*predict-map\*geocoded.tiff'. 
    1. In ***config_os.yaml***, under ***Modis_params***, disable all the other preprocessing steps by setting ***to_XXXX_Modis*** to be ***False***.
    1. Run SIP, click on ***Preprocessing -> Modis*** to mosaic the classification maps. Once finished, go to ***output_dir_maps_merge_Modis*** defined in ***config_os.yaml*** to take a look at the mosaiced map. Open it in QGIS and compare it with the true map of Saskatchewan. 


* **Prepare data for Alberta**
    1. Open ***config_os.yaml*** and ensure that ***raw_img_dir*** points to the folder with preprocessed time series blocks, i.e., ***MOD13Q1_preprocessed_imgs_alta_evi***, ***raw_band_names*** consists of '\*all-channels.tiff'. 
    1. Open ***config_os.yaml***, under ***random_split_samples_params***, set ***to_random_split_samples*** and ***to_split_on_image_level*** to be ***True***. Set ***train_proportion*** to be ***0***, ***val_proportion*** to be ***0***, and ***test_proportion*** to be ***1***.
    1. Run SIP, and click on ***Classification -> Prepare data***. Once finished, go to ***dirs -> data -> train*** defined in the config file and open ***data_file.yaml*** to take a look at the training block list. 
 
* **Test dip-res model on Alberta**
    1. Copy the ***data_file.yaml*** file under ***MOD13Q1_preprocessed_imgs_alta_evi -> all_data -> data -> test*** to ***MOD13Q1_preprocessed_imgs_sask_evi -> all_data -> data -> test***. 
    1. Open ***config_os.yaml*** and ensure that ***raw_img_dir*** points to the folder with preprocessed time series blocks, i.e., ***MOD13Q1_preprocessed_imgs_sask_evi***, ***raw_band_names*** consists of '\*all-channels.tiff'.
    1. Run SIP, click on ***Classification -> Test classifier*** to perform test on Alberta. 
    1. Once finish, go to ***all_data -> save -> model*** to take a look at the accuracy in the log file, and also the ***classification maps*** generated.
    1. Compare the training, val and test accuracies for dip-res. 


![](../pics/dipres_evi_alta_test_cm.png)


Table 1. The test accuracy on Alterta.

Method | Overall Accuracy | Kappa Coefficient | Class Averaged Accuracies
------------- | ------------ | ------------- | -------------
Dipres | 62.74%   | 0.537 | 31.40%  



* **Mosaic test classification maps for Alberta**
    1. In ***config_os.yaml***, under ***Modis_params***, set ***Modis_maps_merge_params*** to be ***True***, set ***input_dir_maps_merge_Modis*** to be ***MOD13Q1_preprocessed_imgs_sask_evi -> all_data -> save -> test*** folder defined in ***config_os.yaml*** where you have the classification maps of test images, and set ***maps_merge_Modis_id_str*** to be '\*predict-map\*geocoded.tiff'. 
    1. In ***config_os.yaml***, under ***Modis_params***, disable all the other preprocessing steps by setting ***to_XXXX_Modis*** to be ***False***.
    1. Run SIP, click on ***Preprocessing -> Modis*** to mosaic the classification maps. Once finished, go to ***output_dir_maps_merge_Modis*** defined in ***config_os.yaml*** to take a look at the mosaiced map. Open it in QGIS and compare it with the true map of Alberta. 
    1. The figure below shows test classification map vs. ground truth map (the one with red urband classes).


![](../pics/dipres_evi_alta_test_map.gif)

   
## Experiment 2: Train dip-res on Saskatchewan and test on Alberta using NDVI time series

* **Generate NDVI time series training blocks for Saskatchewan and Alberta**

    1. Follow the same procedure in Experiment 1. The only difference is to use NDVI band rather than the EVI band. 

* **Train dip-res model on Saskatchewan and test on Alberta**
    1. Follow the same procedure in Experiment 1. 

![](../pics/dipres_ndvi_sask_epoch_15classes.png)


* **Compare the train, val and test accuracies with Experiment 1**
    1. Compare accuracies with Experiment 1.
    1. Compare classification maps with Experiment 1.  

    1. The figure below shows training classification map vs. ground truth map (the one with red urband classes).


![](../pics/legend_canada_modis.png)
![](../pics/dipres_ndvi_sask_train_map_15classes.gif)

    1. The figure below shows test classification map vs. ground truth map (the one with red urband classes).

![](../pics/dipres_ndvi_alta_test_map_15classes.gif)

![](../pics/dipres_ndvi_alta_test_cm_15classes.png)


Table 2. The test accuracy on Alterta.

Method | Overall Accuracy | Kappa Coefficient | Class Averaged Accuracies
------------- | ------------ | ------------- | -------------
Dipres-EVI | 62.74%   | 0.537 | 31.40%  
Dipres-NDVI | 62.88% | 0.530 | 29.37%

## Experiment 3: Train dip-res on Saskatchewan and test on Alberta using time series of 4 reflectance channels

* **Generate 4 reflectance channels time series training blocks for Saskatchewan and Alberta**

    1. Follow the same procedure in Experiment 1. The only difference is to use 4 reflectance bands rather than just the EVI band. 

* **Train dip-res model on Saskatchewan and test on Alberta**
    1. Follow the same procedure in Experiment 1.

![](../pics/dipres_refl4_sask_epoch_15classes.png)

* **Compare the train, val and test accuracies with Experiment 1**
    1. Compare accuracies with Experiment 1.
    1. Compare classification maps with Experiment 1.  

    1. The figure below shows training classification map vs. ground truth map (the one with red urband classes).

![](../pics/dipres_refl4_sask_train_map_15classes.gif)
    
The figure below shows test classification map vs. ground truth map (the one with red urband classes).

![](../pics/dipres_refl4_alta_test_map_15classes.gif)

The figure below shows test classification map vs. prediction confidence map (hot color means higher confidence).

![](../pics/dipres_refl4_alta_test_map_15classes_prob.gif)


<!-- these are the results using 19 classes ![](../pics/dipres_refl4_sask_epoch.png)
![](../pics/dipres_refl4_sask_train_map.gif)
![](../pics/dipres_refl4_alta_test_map.gif)
![](../pics/dipres_refl4_alta_test_cm.png) -->

![](../pics/dipres_refl4_alta_test_cm_15classes.png)


Table 3. Comparing test accuracies, four reflectance bands vs. EVI 

Method | Overall Accuracy | Kappa Coefficient | Class Averaged Accuracies
------------- | ------------ | ------------- | -------------
Dipres-EVI | 62.74%   | 0.537 | 31.40% 
Dipres-NDVI | 62.88% | 0.530 | 29.37%
Dipres-refl4 | 69.10% | 0.607 | 37.20% 
<!-- Here is the reults using 19 classes rather than 15 classes Dipres-refl4 | 68.24%   | 0.600 | 35.90%  -->


Table 4. Comparing train, val, test accuracies, four reflectance bands vs. EVI 

Method | Training OA | Val OA | Test OA
------------- | ------------ | ------------- | -------------
Dipres-EVI | 74.52%   | 70.63% | 62.74% 
Dipres-NDVI | 73.63% | 73.58% | 62.88%
Dipres-refl4 |77.51% | 77.67% | 69.10% 
<!--Here, use 19 clases, rather than using 15 classes, Dipres-refl4 | 78.18%   | 76.81% | 68.24%  -->

## Experiment 4: Train dip-res on Saskatchewan and test on Alberta using time series of 4 reflectance channels, evi and ndvi

* **Generate 4 reflectance channels, evi and ndvi time series training blocks for Saskatchewan and Alberta**

    1. Follow the same procedure in Experiment 1. Not only 4 reflectance bands, but also the EVI band and NDVI band are used to generate a time series.  

* **Train dip-res model on Saskatchewan and test on Alberta**
    1. Follow the same procedure in Experiment 1.

![](../pics/dipres_all_sask_epoch_15classes.png)

* **Compare the train, val and test accuracies with previous Experiments**
    1. Compare accuracies with previous Experiments.
    1. Compare classification maps with previous Experiments.  

    1. The figure below shows training classification map vs. ground truth map (the one with red urband classes).

![](../pics/dipres_all_sask_train_map_15classes.gif)
    
    1. The figure below shows test classification map vs. ground truth map (the one with red urband classes).

![](../pics/dipres_all_alta_test_map_15classes.gif)

![](../pics/dipres_all_alta_test_cm_15classes.png)


Table 5. Comparing test accuracies, four reflectance bands vs. EVI vs. using all bands 

Method | Overall Accuracy | Kappa Coefficient | Class Averaged Accuracies
------------- | ------------ | ------------- | -------------
Dipres-EVI | 62.74%   | 0.537 | 31.40% 
Dipres-NDVI | 62.88% | 0.530 | 29.37%
Dipres-refl4 | 69.10% | 0.607 | 37.20%
Dipres-all | 67.84% | 0.601 | 36.72% 
<!-- Here is the reults using 19 classes rather than 15 classes Dipres-refl4 | 68.24%   | 0.600 | 35.90%  -->


Table 6. Comparing train, val, test accuracies, four reflectance bands vs. EVI vs. using all bands 

Method | Training OA | Val OA | Test OA
------------- | ------------ | ------------- | -------------
Dipres-EVI | 74.52%   | 70.63% | 62.74% 
Dipres-NDVI | 73.63% | 73.58% | 62.88%
Dipres-refl4 |77.51% | 77.67% | 69.10%
Dipres-all | 78.09% | 77.94% | 67.84% 
<!--Here, use 19 clases, rather than using 15 classes, Dipres-refl4 | 78.18%   | 76.81% | 68.24%  -->



## Experiment 5: Test on Alberta using high confidence samples

* **Generate high confidence training samples for 4 reflectance channels time series training blocks of Alberta**

    1. Follow Experiment 7 in [the preprocessing tutorial](./modis_250m_preprocessing.md) to generate high confidence training samples for Alberta. You can still use 4 reflectance bands. 

* **Use the same trained dip-res model on Saskatchewan in Experiment 3, but test on new Alberta dataset**
    1. Follow the same procedure in Experiment 1, but use the new Alterta dataset with high confidence training samples.
 
* **Compare the train, val and test accuracies with Experiment 3**
    1. Compare accuracies with Experiment 3. The accuracies should be much higher than Experiment 3, because here we only use high confidence samples to calculate accuracies. 
    1. Compare classification maps with Experiment 3. The classificaiton maps should be the same, because here we use the same model as Experiment 3 to generate maps.  


![](../pics/dipres_refl4_alta_test_cm_sample_15classes.png)

Table 7. Comparing test accuracies, using sampled pixels vs. using all pixesl for calculating accuracies 

Method | Overall Accuracy | Kappa Coefficient | Class Averaged Accuracies
------------- | ------------ | ------------- | -------------
Dipres-all-pixels | 69.10% | 0.607 | 37.20%
Dipres-sample-pixels | 88.27% | 0.829 | 62.50%
<!-- These are results of using 19 classes 
Dipres-sampled-pixels | 87,24%   | 0.814 | 59.32%  
Dipres-all-pixels | 68.24%   | 0.600 | 35.90%
![](../pics/dipres_refl4_alta_test_cm_sample.png)
-->

## Experiment 6: Train dip-res on Saskatchewan and test on Alberta using summer season time series of 4 reflectance channels, instead of all seasons

* **Generate summer season time series 4 reflectance channels training blocks for Saskatchewan and Alberta**

    1. Follow the same procedure in Experiment 1. Here, you should set ***dates_list_to_be_processed_Modis*** in ***config_os.yaml*** to be ***2010161, 2010177, 2010193, 2010209, 2010225, 2010241, 2010257, 2010273, 2010289***, so that you can just use the data in June, July, August, September and October to generate time series. 

* **Train dip-res model on Saskatchewan and test on Alberta**
    1. Follow the same procedure in Experiment 1.

![](../pics/dipres_refl4_sask_epoch_15classes_summer.png)

* **Compare the train, val and test accuracies with previous Experiments**
    1. Compare accuracies with previous Experiments.
    1. Compare classification maps with previous Experiments.  

    1. The figure below shows training classification map vs. ground truth map (the one with red urband classes).

![](../pics/dipres_refl4_sask_train_map_15classes_summer.gif)
    
    1. The figure below shows test classification map vs. ground truth map (the one with red urband classes).

![](../pics/dipres_refl4_alta_test_map_15classes_summer.gif)

![](../pics/dipres_refl4_alta_test_cm_15classes_summer.png)


Table 8. Comparing test accuracies 

Method | Overall Accuracy | Kappa Coefficient | Class Averaged Accuracies
------------- | ------------ | ------------- | -------------
Dipres-EVI | 62.74%   | 0.537 | 31.40% 
Dipres-NDVI | 62.88% | 0.530 | 29.37%
Dipres-refl4 | 69.10% | 0.607 | 37.20%
Dipres-all | 67.84% | 0.601 | 36.72% 
Dipres-refl4-summer | 63.24% | 0.550 | 36.79% 
<!-- Here is the reults using 19 classes rather than 15 classes Dipres-refl4 | 68.24%   | 0.600 | 35.90%  -->


Table 9. Comparing train, val, test accuracies 

Method | Training OA | Val OA | Test OA
------------- | ------------ | ------------- | -------------
Dipres-EVI | 74.52%   | 70.63% | 62.74% 
Dipres-NDVI | 73.63% | 73.58% | 62.88%
Dipres-refl4 |77.51% | 77.67% | 69.10%
Dipres-all | 78.09% | 77.94% | 67.84% 
Dipres-refl4-summer | 75.87% | 77.12% | 63.24% 
<!--Here, use 19 clases, rather than using 15 classes, Dipres-refl4 | 78.18%   | 76.81% | 68.24%  -->

## Experiment 7: Train dip-res on Saskatchewan and test on Alberta using time series of 4 reflectance channels, using different weights for different classes

* **Generate whole year time series 4 reflectance channels training blocks for Saskatchewan and Alberta**

    1. Follow the same procedure in Experiment 1.  

* **Train dip-res model on Saskatchewan and test on Alberta**
    1. Follow the same procedure in Experiment 1. But, make sure that ***to_use_class_proportions_as_inverse_weights_train*** is ***True***, and ***my_class_proportions*** is ***2, 1, 1, 1, 1, 1, 0.1, 0.1, 0.1, 1, 2, 1, 0.1, 1, 0.1*** to assign higher weights to Classes 7, 8, 9, 13 and 15, and lower weights to class 1 and 11.  

![](../pics/dipres_refl4_sask_epoch_15classes_class_weights3.png)

* **Compare the train, val and test accuracies with previous Experiments**
    1. Compare accuracies with previous Experiments.
    1. Compare classification maps with previous Experiments.  

    1. The figure below shows training classification map vs. ground truth map (the one with red urband classes).

![](../pics/dipres_refl4_sask_train_map_15classes_class_weights3.gif)
    
    1. The figure below shows test classification map vs. ground truth map (the one with red urband classes).

![](../pics/dipres_refl4_alta_test_map_15classes_class_weights3.gif)

![](../pics/dipres_refl4_alta_test_cm_15classes_class_weights3.png)


Table 9. Comparing test accuracies 

Method | Overall Accuracy | Kappa Coefficient | Class Averaged Accuracies
------------- | ------------ | ------------- | -------------
Dipres-EVI | 62.74%   | 0.537 | 31.40% 
Dipres-NDVI | 62.88% | 0.530 | 29.37%
Dipres-refl4 | 69.10% | 0.607 | 37.20%
Dipres-all | 67.84% | 0.601 | 36.72% 
Dipres-refl4-summer | 63.24% | 0.550 | 36.79% 
Dipres-refl4-class-weights | 65.06% | 0.569 | 38.54% 


Table 10. Comparing train, val, test accuracies 

Method | Training OA | Val OA | Test OA
------------- | ------------ | ------------- | -------------
Dipres-EVI | 74.52%   | 70.63% | 62.74% 
Dipres-NDVI | 73.63% | 73.58% | 62.88%
Dipres-refl4 |77.51% | 77.67% | 69.10%
Dipres-all | 78.09% | 77.94% | 67.84% 
Dipres-refl4-summer | 75.87% | 77.12% | 63.24% 
Dipres-refl4-claass-weights | 75.54% | 76.62% | 65.06% 

## Experiment 8: Train dip-res on Saskatchewan and test on Alberta using different weights for different classes, readjust class weights during test

* **Generate whole year time series 4 reflectance channels training blocks for Saskatchewan and Alberta**

    1. Follow the same procedure in Experiment 1.  

* **Change class weights of the Urban class**
    1. In the previous experiments, the urban class has been over-estimated. How do we reduce over-estimation of the urban class during test? Open the ***config_os.yaml*** file, and set ***to_adjust_soft_class_membership_in_test_inference*** to be ***True***, and set ***my_class_weights_test*** to be ***'1_1_1_1_1_1_1_1_1_1_1_1_0.1_1_1'***, so that the weight the urban class is reduced to be ***0.1*** to reduce the number of pixels that will be estimated as being the urban class. 
 
* **Compare the train, val and test accuracies with previous Experiments**
    1. Compare accuracies with previous Experiments.
    1. Compare classification maps with previous Experiments.  

The figure below shows test classification map vs. ground truth map (the one with red urband classes).

![](../pics/dipres_refl4_alta_test_map_15classes_class_weights3_readjust.gif)

![](../pics/dipres_refl4_alta_test_cm_15classes_class_weights3_readjust.png)


Table 11. Comparing test accuracies 

Method | Overall Accuracy | Kappa Coefficient | Class Averaged Accuracies
------------- | ------------ | ------------- | -------------
Dipres-EVI | 62.74%   | 0.537 | 31.40% 
Dipres-NDVI | 62.88% | 0.530 | 29.37%
Dipres-refl4 | 69.10% | 0.607 | 37.20%
Dipres-all | 67.84% | 0.601 | 36.72% 
Dipres-refl4-summer | 63.24% | 0.550 | 36.79% 
Dipres-refl4-class-weights | 65.06% | 0.569 | 38.54% 
Dipres-refl4-class-weights-readjust | 68.11% | 0.601 | 37.18% 

## Experiment 9: Test the trained model in Experiment 7 on Saskatchewan data of year 2015

* **Generate whole year time series 4 reflectance channels for Sask of year 2015**

    1. Follow the same procedure in Experiment 1. But, instead of using year 2010 data, download and process the year 2015 data.   

* **Compare test accuracies with Experiment 7, test on year 2010 Alta vs. test on year 2015 Sask**

The figure below shows test classification map vs. ground truth map (the one with red urband classes).

![](../pics/dipres_refl4_sask_2015_test_map_15classes_class_weights.gif)

The figure below shows ground truth maps of 2010 and 2015 for sask.

![](../pics/sask_2010_2015_map.gif)

The figure below shows soft class membership of the fifth class vs. ground truth map.

![](../pics/dipres_refl4_sask_2015_test_softlabel_15classes_class_weights.gif)


![](../pics/dipres_refl4_sask_2015_test_cm_15classes_class_weights.png)

Table 11. Comparing test accuracies 

Method | Overall Accuracy | Kappa Coefficient | Class Averaged Accuracies
------------- | ------------ | ------------- | -------------
Dipres-EVI | 62.74%   | 0.537 | 31.40% 
Dipres-NDVI | 62.88% | 0.530 | 29.37%
Dipres-refl4 | 69.10% | 0.607 | 37.20%
Dipres-all | 67.84% | 0.601 | 36.72% 
Dipres-refl4-summer | 63.24% | 0.550 | 36.79% 
Dipres-refl4-class-weights | 65.06% | 0.569 | 38.54% 
Dipres-refl4-class-weights-readjust | 68.11% | 0.601 | 37.18% 
Dipres-refl4-class-weights-sask-2015 | 64.28% | 0.561 | 30.00% 


<!-- ## Experiment 8: Train dip-res on Saskatchewan and test on Alberta using time series of 4 reflectance channels, using different weights for different classes

* **Generate summer season time series 4 reflectance channels training blocks for Saskatchewan and Alberta**

    1. Follow the same procedure in Experiment 1.  

* **Train dip-res model on Saskatchewan and test on Alberta**
    1. Follow the same procedure in Experiment 1. But, make sure that ***to_use_class_proportions_as_inverse_weights_train*** is ***True***, and ***my_class_proportions*** is ***10, 1, 1, 1, 1, 1, 0.01, 0.01, 0.01, 1, 10, 1, 0.01, 1, 0.01*** to assign less weights to classes 1 and 11, but higher weights to Classes 7, 8, 9, 13 and 15. 

![](../pics/dipres_refl4_sask_epoch_15classes_class_weights2.png)

* **Compare the train, val and test accuracies with previous Experiments**
    1. Compare accuracies with previous Experiments.
    1. Compare classification maps with previous Experiments.  

    1. The figure below shows training classification map vs. ground truth map (the one with red urband classes).

![](../pics/dipres_refl4_sask_train_map_15classes_class_weights2.gif)
    
    1. The figure below shows test classification map vs. ground truth map (the one with red urband classes).

![](../pics/dipres_refl4_alta_test_map_15classes_class_weights2.gif)

![](../pics/dipres_refl4_alta_test_cm_15classes_class_weights2.png)


Table 10. Comparing test accuracies 

Method | Overall Accuracy | Kappa Coefficient | Class Averaged Accuracies
------------- | ------------ | ------------- | -------------
Dipres-EVI | 62.74%   | 0.537 | 31.40% 
Dipres-NDVI | 62.88% | 0.530 | 29.37%
Dipres-refl4 | 69.10% | 0.607 | 37.20%
Dipres-all | 67.84% | 0.601 | 36.72% 
Dipres-refl4-summer | 63.24% | 0.550 | 36.79% 
Dipres-refl4-class-weights | 69.45% | 0.610 | 36.10% 
Dipres-refl4-class-weights2 | 65.63% | 0.537 | 25.94% 


Table 11. Comparing train, val, test accuracies 

Method | Training OA | Val OA | Test OA
------------- | ------------ | ------------- | -------------
Dipres-EVI | 74.52%   | 70.63% | 62.74% 
Dipres-NDVI | 73.63% | 73.58% | 62.88%
Dipres-refl4 |77.51% | 77.67% | 69.10%
Dipres-all | 78.09% | 77.94% | 67.84% 
Dipres-refl4-summer | 75.87% | 77.12% | 63.24% 
Dipres-refl4-claass-weights | 77.65% | 77.34% | 69.45% 
Dipres-refl4-claass-weights2 | 70.54% | 72.13% | 65.63% 

-->

## Experiment 7: Train spectral Dip resnet on Saskatchewan and test on Alberta
* **Change classifier**
 
    1. The procedure is exactly the same as Experiment 3. The only difference is before you train the model, make sure in ***config_os.yaml*** file, you set ***net_type*** to be ***dip_spe***. 



## Experiment 7: Train patch-based ss_res on Saskatchewan and test on Alberta

* **Generate high confidence reflectance 4 bands time series training blocks for Saskatchewan**

    1. Generate high confidence 4 reflectance samples according to Experiment 6 in [the preprocessing tutorial](./modis_250m_preprocessing.md).
    1. Assume that the final train blocks are saved into ***MOD13Q1_preprocessed_imgs_sask_refl4_sampling***
 
* **Generate high confidence reflectance 4 bands time series training blocks for Alberta**

    1. Generate data according to Experiment 7 in [the preprocessing tutorial](./modis_250m_preprocessing.md).
    1. Assume that the final train blocks are saved into ***MOD13Q1_preprocessed_imgs_alta_refl4_sampling***

* **Prepare data for Saskatchewan**
    1. Open ***config_os.yaml*** and ensure that ***raw_img_dir*** points to the folder with preprocessed time series blocks, i.e., ***MOD13Q1_preprocessed_imgs_sask_refl4_sampling***, ***raw_band_names*** consists of '\*all-channels.tiff', ***dirs -> data/mask/save*** are set using your own directories, ***num_classes*** equals 15, ***is_colors_defined_in_rgb_values*** is True, and ***my_colors*** consists of 17 colors of rgb values, and ***my_classes*** consists of 17 class names. Out of the 19 classes, Class3, Class4, Class7 and Class9 should not be used.  
    1. Open ***config_os.yaml***, under ***random_split_samples_params***, set ***to_random_split_samples*** and ***to_split_on_image_level*** to be ***True***. Set ***train_proportion*** to be ***0.8***, ***val_proportion*** to be ***0.2***, and ***test_proportion*** to be ***0***.
    1. Run SIP, and click on ***Classification -> Prepare data***. Once finished, go to ***dirs -> data -> train*** defined in the config file and open ***data-file.yaml*** to take a look at the training block list. 
 
* **Train ss-res model on Saskatchewan**

    1. In ***config_os.yaml***, under ***train_params***, ***lr*** equals 0.001, ***batch_size_train/val/test/predict*** equals 1000, ***net_type*** is ss-res.
    1. In ***config_os.yaml***, under ***train_params***, ***prop_train*** equals ***0.1_0.1_0.1_0.1_0.1_0.1_0.1_0.1_0.1_0.1_0.1_0.1_0.1_0.1_0.1*** to use 10% samples of each class for training.  
    1. Save ***config_os.yaml***.
    1. Run SIP, click on ***Train classifier*** under ***Classification***, choose the ***config_os.yaml*** you edited. 
    1. If you want to use the save model file to continue training, set ***to_resume_from_latest_checkpoint*** to be ***True*** and also set ***resume_checkpoint_file*** to the model file under ***dirs -> save -> model*** defined in ***config_os.yaml***.  
    1. Record the training and validation accuracies. 

* **Mosaic training classification maps for Saskatchewan**
    1. In ***Config_os.yaml***, under ***Modis_params***, set ***Modis_maps_merge_params*** to be ***True***, set ***input_dir_maps_merge_Modis*** to be ***dirs -> save -> train*** folder defined in ***config_os.yaml*** where you have the classification maps of training images, and set ***maps_merge_Modis_id_str*** to be '\*predict-map\*geocoded.tiff'. 
    1. In ***Config_os.yaml***, under ***Modis_params***, disable all the other preprocessing steps by setting ***to_XXXX_Modis*** to be ***False***.
    1. Run SIP, click on ***Preprocessing -> Modis*** to mosaic the classification maps. Once finished, go to ***output_dir_maps_merge_Modis*** defined in ***config_os.yaml*** to take a look at the mosaiced map. Open it in QGIS and compare it with the true map of Saskatchewan. 

* **Prepare data for Alberta**
    1. Open ***config_os.yaml*** and ensure that ***raw_img_dir*** points to the folder with preprocessed time series blocks, i.e., ***MOD13Q1_preprocessed_imgs_alta_refl4_sampling***, ***raw_band_names*** consists of '\*all-channels.tiff'. 
    1. Open ***config_os.yaml***, under ***random_split_samples_params***, set ***to_random_split_samples*** and ***to_split_on_image_level*** to be ***True***. Set ***train_proportion*** to be ***0***, ***val_proportion*** to be ***0***, and ***test_proportion*** to be ***1***.
    1. Run SIP, and click on ***Classification -> Prepare data***. Once finished, go to ***dirs -> data -> train*** defined in the config file and open ***data_file.yaml*** to take a look at the training block list. 
 
* **Test ss-res model on Alberta**
    1. Copy the ***data_file.yaml*** file under ***MOD13Q1_preprocessed_imgs_alta_refl4_sampling -> all_data -> data -> test*** to ***MOD13Q1_preprocessed_imgs_sask_refl4_sampling -> all_data -> data -> test***. 
    1. Open ***config_os.yaml*** and ensure that ***raw_img_dir*** points to the folder with preprocessed time series blocks, i.e., ***MOD13Q1_preprocessed_imgs_sask_refl4_sampling***, ***raw_band_names*** consists of '\*all-channels.tiff'.
    1. Run SIP, click on ***Classification -> Test classifier*** to perform test on Alberta. 
    1. Once finish, go to ***all_data -> save -> model*** to take a look at the accuracy in the log file, and also the ***classification maps*** generated.
    1. Compare the training, val and test accuracies for ss-res. 

* **Mosaic test classification maps for Alberta**
    1. In ***config_os.yaml***, under ***Modis_params***, set ***Modis_maps_merge_params*** to be ***True***, set ***input_dir_maps_merge_Modis*** to be ***MOD13Q1_preprocessed_imgs_sask_refl4_sampling -> all_data -> save -> test*** folder defined in ***config_os.yaml*** where you have the classification maps of test images, and set ***maps_merge_Modis_id_str*** to be '\*predict-map\*geocoded.tiff'. 
    1. In ***config_os.yaml***, under ***Modis_params***, disable all the other preprocessing steps by setting ***to_XXXX_Modis*** to be ***False***.
    1. Run SIP, click on ***Preprocessing -> Modis*** to mosaic the classification maps. Once finished, go to ***output_dir_maps_merge_Modis*** defined in ***config_os.yaml*** to take a look at the mosaiced map. Open it in QGIS and compare it with the true map of Alberta. 


## Experiment 7: Train svm, rf on Saskatchewan and test on Alberta
* **Change classifier, change trian proportion if too slow**
 
    1. The procedure is exactly the same as Experiment 6. The only difference is, in ***config_os.yaml***, under ***train_params***, change ***0.1*** in ***prop_train*** to smaller proportions, if it is too slow to train svm or rf. 

    1. Compare the performance of different models, i.e., dip-res, ss-res, spectral res, svm, rf. 





  
 
