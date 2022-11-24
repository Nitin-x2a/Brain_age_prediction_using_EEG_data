#Flies and Folders

- Data : Contains all downloaded data

- models : Contains trained pickled models, and their hyperparameters json files

- Downloading_data.ipynb : Code for downloading data

- Unzipping_data.ipynb : Code for unzipping downloaded data

- BrainAge_FeatureExtraction.ipynb (Most important part of the repo) : Code for feature-extraction

- BrainAge_Prediction.ipynb (Second most important part of the repo) : Code for making test predictions

- env.yml : Used for setting up envioronment (Required for the feature-extraction and prediction notebooks)

- feature_extraction_info.json : Required for the prediction notebook

- df_submission.zip : Final phase-2 test data predictions

# Setting up environment

Use the terminal or an Anaconda Prompt for the following steps:

1. Create the environment from the **env.yml** file:
```
conda env create -n brain_age -f env.yml
```
2. Activate the new environment: 
```
conda activate brain_age
```
3. Verify that the new environment was installed correctly:
```
conda env list
```
# Downloading data
The notebook **downloading_data** is used to download the training data, phase-1 test data, starter kit, ands phase-2 test data.

The notebook **unzipping_data** is used to unzip all the downloaded data inside the **/Data** folder

# Feature extraction
The notebook **BrainAge_FeatureExtraction** is used to load the downloaded train and test datasets, and calculate features for training & pediction.

All the data is then extracted, cropped, resampled, and filtered. Both 'Eyes closed (EC)' and 'Eyes Open (EO)' data is used. The 'Cz' channel data is ignored.

The processed training data is then split into training, and validation sets, for both EC and EO.

Next, using sliding windows, additional training and validation samples are created for each subject in the training, validation, and test sets, in both EC and EO conditions separately. 

A 6-second window with 2-second overlap is used.

A **feature_extraction_info** json file is saved with some meta information reagarding the sliding wondow process that will be required in the prediction notebook. 

Next, for every training/validation/test sample, a total of 35 temporal, statistical, and spectral features are calculated per channel, that are described in the notebook.

The 35 features of every channel are concatenated together for every sample in the end.

# Model Training

Two models are trained - one for EC, and one for EO. The scores are validated over the respective validation sets.

Azure autoML was used for training the two models. When provided with training & validation features and the corresponing target age values in a tabular format, it automatically fits dozens of ML models over the training data, and tunes their hyperparameters, so as to do well on the validation data. It even tests a voting ensemble of the best models that it discovers. 

The model can then be downloaded as a pickle object, which can be serialized using joblib, inside an environment that can be set up with an yaml file also provided by autoML.

The model details, and hyperparameters can also be downloaded after training job completes. The two model hyperparametes json files are also present in the models folder, which can be used to create the models from scratch, if required.

In both EO and EC cases, autoML found two ensemble models to best fit the respective training datasets. For the EC dataset, azureML created an ensemble of 10 models. And for the EO dataset, azureML created an ensemble of 8 models. Almost all individual models in both the autoML ensembles are tree-based algorithms.

# Predictions on test data
 
 Two predictions are made - one from model fitted over EC data, one from model fitted over EO data. Refer the **BrainAge_Prediction** notebook
 
 Since the sliding window was also applied to the test data, we will have multiple predictions for each test subject. 
 
 There were 9 windows extracted per subject from their EC data, and 6 from their EO data. So, we average over the 9 predictions to get a single prediction per test subject. Similarly, the 6 predictions are averaged for every test subject.
 
 A weighted ensemble of the two prediction values for every subject are taken as the final brain age prediction for the test subjects.
 
 ```
 brain_age = 0.635 * brain_age_EC_model_prediction + 0.365 * brain_age_EO_model_prediction
 ```
 # Summary
 - Data is downsampled to 250 Hz, to smoothen it out
 - Data is notch-filtered at (60, 120 Hz), bandpass-filtered in (1, 100) Hz, to remove the line noise and any other information outside the extreme gamma band range.
 - A sliding-window is used to augment training data. 
 - For the test-data, the sliding window is used to get multiple predictions per subject, which at meaned at the end.
 - A total of 35 commonly used temporal, statistical, and spectral features are calculated over the extracted windows. Since, Cz is ignored, there are 128 channels. Hence, we get 4480 features for every sample.
 - Azure autoML is used for training which automatically finds out the best featurs, models, and hyperparameters.
 - Both EC and EO condition data is used, and two different models are trained on those datasets.
 - Test subjetcs data is also split into windows, and then predictions are made over all those windows (by both EO and EC based models). The final prediction is a weighted average of the two final predictions. 