# CNC-Tool-wear-detection<br>
### Table of content
- [Purpose of study](#purpose-of-study)
- [Data description](#data-description)
- [Data Collection](#data-collection)
- [EDA](#eda)
- -[Training Model](#training-model)

## Purpose of study
This study aims to find the best model to detect unworn CNC machines. The dataset results from the University of Michigan experiments on a CNC milling machine in the System-level Manufacturing and Automation Research Testbed (SMART), while the machine ran to make a "S" shape(S for smart manufacturing ) in 2" x 2" x 1.5" wax blocks.<br>
<div align="center">
 <image width=200 src="https://github.com/fereshteh-bahadory/CNC-Tool-wear-detection/assets/65620341/af1e314d-54e5-42d3-a14f-0599b43d3a51">
</div>

## Data description
This project used a CNC milling machine to perform machining experiments. The CNC machine recorded machining data for different settings of "tool condition", "feed rate", and "clamping pressure". More details about the dataset can be found [here](https://www.kaggle.com/datasets/shasun/tool-wear-detection-in-cnc-mill/data).<br>
The CNC machine had 4 motors (X, Y, Z axes and spindle) that generated time series data with a 100 ms interval in 18 machining experiments. The output of each experiment showed the tool condition (whether the tool was worn or not) and the result of visual inspection. This dataset is useful for finding tool wear or poor clamping.

## Data collection
The dataset was gathered at the University of Michigan, which can be found [here](https://www.kaggle.com/datasets/shasun/tool-wear-detection-in-cnc-mill/data). The data consists of 18 data frames. The data frame `train.csv` has "material",	"feedrate",	"clamp_pressure",	"tool_condition",	"machining_finalized", and	"passed_visual_inspection" as its features. Four entries of this data frame were "NAN", which I treat as "No". The other data frames are gathered from data over time from 18 different experiments. These measurements were taken every 0.1 seconds (100 milliseconds) and saved in separate files, named "experiment_01.csv" to "experiment_18.csv". Each file contains information from the 4 motors in the machine (X, Y, Z, and spindle). There are two ways to use this data:<br>
1. Treating each measurement individually: We can analyze each measurement from the machine (X, Y, Z, and spindle) as a separate data point. We can also use the "Machining_Process" column to understand what operation was happening at the time of the measurement (e.g., "Layer 1 Up", "Layer 2 Down").<br>
2. Analyzing the entire experiment as a sequence: We can consider the entire series of measurements from each experiment (all the data points in a single file) as one observation. This approach is useful when we want to classify the type of experiment based on the entire sequence of measurements, using techniques like time series classification.

## EDA
The first thing to do was to merge all data frames to be able to predict if "tool condition" is worn or unworn. So using `Pandas`, I merged the 18 experimental data frames and added the `train.csv` as their features. As a result, we get a 25285x54-shaped data frame with no missing value. I also dropped the column "material", since for all the experiments the only material used was wax.<br>
Using the statistical data, gained by `describe()`, and also the box plot, scatter plot, and bar plot of the non-object columns, the data seemed to have outliers. So using the IQR method I saw a big number of outliers in some columns, which in my opinion, could not be the result of the measurement error. Therefore, I didn't replace them.<br>
The other thing obtained from them was that four columns were completely useless since they contained only one value. The columns are listed below.<br>
```
Z1_CurrentFeedback, Z1_DCBusVoltage, Z1_OutputCurrent, Z1_OutputVoltage, S1_SystemInertia
```
I dropped those columns together with `exp_number`, which was added during merging data frames and only showed the number of the dataset.<br>
I have also compared the number of "yes" and "no" answers for the `machining_finalized` and `passed_visual_inspection` columns in the "worn" and "unworn" tool condition cases. The numbers you can see below:<br>
```
Machining Finalized for Worn Tools
machining_finalized    no    yes
tool_condition                  
worn                 1167  12141
----------------------------------------
Machining Finalized for Unworn Tools
machining_finalized   no    yes
tool_condition                 
unworn               994  10984
----------------------------------------
Passed Visual Inspection for Worn Tools
passed_visual_inspection    no   yes
tool_condition                      
worn                      5109  8199
----------------------------------------
Passed Visual Inspection for Unworn Tools
 passed_visual_inspection   no    yes
tool_condition                      
unworn                    994  10984
----------------------------------------
```
This result shows even for unworn tool conditions, there is almost a large number of successfully `passed_visual_inspection` and `machining_finalized`.<br> 
The same I did to compare "yes" results in the `machining_finalized` and `passed_visual_inspection` columns.
```
Machining Finalized for Passed Visual Inspection
machining_finalized         yes
passed_visual_inspection       
yes                       19183
----------------------------------------
Machining Finalized for Not Passed Visual Inspection
machining_finalized         no   yes
passed_visual_inspection            
no                        2161  3942
----------------------------------------
```
Even though it seems the "yes" results in the `passed_visual_inspection` column imply the same result in `machining_finalized`, both columns are valuable to predict too conditioning of the machine. Therefore, I kept them bot.<br>
It only remains to encode the categorical columns. There was only one non-binary column, Machining_Process, in which I used the Frequency Encoding method to encode the values and saved it to a data frame `frequent_encoding_data`. I also used one-hot encoding for this column and saved it to a different data frame `onehot_encoding_data` to see if it would make difference in the training result.<br>
For the binary-valued columns, tool_condition, machining_finalized, and passed_visual_inspection, I considered a success as 1 and unsuccess as 0.<br>

## Training Model
To train the model, I used four different methods, "Random Forest Classifier", "Decision Tree", "XGBoost", and a simple RNN model. Except for the RNN model, I used both data frames `frequent_encoding_data.csv` and `onehot_encoding_data.csv`. Since the accuracy and other metrics were high for both data, I prefer to consider the confusion matrix as my criterion for choosing the best method. You can see the results in the following table.<br>

<div align="center">
 
|Data frame|Random Forest Classifie|Decision Tree|XGBoost| RNN |
|   :---:  |      :---:            |      :---:  | :---: |:---:|
|forfrequent_encoding_data|[[2336,   12],<br/> [  10, 2700]]|[[2330,   18],<br/> [  21, 2689]]|[[2346,    2],<br/> [   3, 2707]]|[[ 236, 2211],<br/> [  60, 2551]]
|onehot_encoding_data     |[[2409,    9],<br/> [  19, 2621]]|[[2397,   21],<br/> [  34, 2606]]|[[2415,    3],<br/> [   8, 2632]]|
</div>






 
