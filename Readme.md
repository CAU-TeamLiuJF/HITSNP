<!--
 * @Author: Zhang Meilin
 * @LastEditTime: 2024-09-12 16:00
 * @FilePath: \HITSNP\Readme.md
-->
# HITSNP: a High-throughput Screening for Feature SNPs

High-throughput screening for feature SNPs representing breed diversity and estimating population ancestry: software manual V1.0

## Software Introduction

This software is a feature SNP selection tool specifically designed to handle large-scale deep whole-genome sequencing (WGS) data. Based on simulated hybrid data and feature SNPs, it also provides machine learning classifiers trained to predict the number and composition of ancestral populations.

HITSNP includes three modules:  ```'feature SNP screening'```, ```'ancestry estimation'```, and ```'minimum subset selection'```.
![Workflow](https://github.com/CAU-TeamLiuJF/HITSNP/blob/main/hitsnp-workflow.png "HITSNP-Workflow")

## Quick Start

Please note that, unless otherwise specified, all of the following operations are carried out in Linux command-line mode.

### Installation

The executable files can be directly downloaded and used:

```bash
tar zxvf HITSNP.tar.gz
cd HITSNP
chmod 755 ./bin/HITSNP
```
### Dependencies

- Plink 1.90
- VCTtools 0.1.16


## **hit-feature: feature SNP screening**
``` hit-feature ``` is a module that utilizes feature selection methods to select feature SNPs from high-throughput datasets.
``` hit-feature ``` also evaluates the breed diversity of feature SNPs, which includes a modified Simpson Diversity index (D<sub>m</sub>) and machine learning classifiers' performance.


### Usage

```bash
hit-feature -s parameters1.txt
```

### Input Parameter

Parameters1.txt contains the necessary parameters for running hit-feature, with an example provided below:

```
WorkSpace = "/home/user/project/workspace/"
ReferencePopulation = "/home/user/project/data/ref_pop"
BreedInformation = "/home/user/project/data/ref_breed.csv"
ScreenMethod = "MRMR"
SNPsNumberPerBreed = "200"
Plink = "/path/to/plink"
Vcftools = "/path/to/vcftools"
Threads = "20"
Simpson = "1 0.6 0.8"
StartFrom="0"
``` 

- **Description**：
    - **WorkSpace**：Specifies the absolute path to the workspace directory where output files will be saved.
    - **ReferencePopulation**: Specifies the absolute path to the PLINK files for the reference population, without including file extensions (e.g., .bed, .bim, .fam). Ensure the path does not include any suffixes.
    - **BreedInformation**: Specifies the absolute path to the CSV file containing breed information for the reference population.
    - **ScreenMethod**: Specifies the filtration method to use. Options include ```"CCA"```, ```"ReliefRR"```, or ```"MRMR"```.
    - **SNPsNumberPerBreed**: Specifies the desired number of SNPs selected per breed. This must be an integer less than 1,000, typically set to ```"100"```.
    - **Plink**: Specifies the absolute path to the PLINK software executable.
    - **Vcftools**: Specifies the absolute path to the VCFtools software executable.
    - **Threads**: Specifies the number of threads to use for parallel processing of multiple breeds. The default is ```"1"```.
    - **Simpson** : Configures D<sub>m</sub> calculation. Set ```"0"``` to skip the calculation, or use values like ```"1 0.6 0.8"``` to calculate D<sub>m</sub> with 0.6 as the low threshold and 0.8 as the high threshold. All threshold values must fall between 0 and 1.
    - **StartFrom**: Set to ```"0"``` to restart the process from the beginning. Use ```"1"``` or any other value to begin directly from the feature SNP selection step if feature values for the SNPs have already been computed. When input is set to ```"1"```, please do not modify *WorkSpace*, *ReferencePopulation*，*BreedInformation* and *ScreenMethod* parameters,or output file locations from the parameter card used in the ```"0"``` setting.

### Input File

- **ReferencePopulation**
The data must have undergone quality control (QC) and LD pruning, with each breed containing at least 5 or more samples. Additionally, SNP IDs must be unique; if an ID contains a ```.``` symbol, it can be replaced by the format ```chr-.-pos```.

- **BreedInformation**
The input CSV file must have two columns, ```indi``` and ```breed```, where indi corresponds to the individual IDs in the .fam file of the ReferencePopulation, for example:

```
    indi,breed
    ind01,Breed1
    ind02,Breed1
    ind03,Breed1
    ind06,Breed2
    ind07,Breed2
    ind09,Breed2
```

### Output File

The results of the five-fold cross-validation will be stored in the "data_ ```ReferencePopulation``` _ ```ScreenMethod``` _ ```SNPsNumberPerBreed```" folder under the workspace directory, and the feature SNP set with the highest accuracy will be copied into "result _ ```SNPsNumberPerBreed```" folder.

Within this folder, the final result of hit-feature includes: model_set (fold), featurelist_```SNPsNumberPerBreed```.txt, rs_featurelist_```SNPsNumberPerBreed```.txt, dm_```SNPsNumberPerBreed```_ ```low threshold```_```high threshold```.result, model_report.csv,  ```ScreenMethod```_info_breed.txt.

- **model_set (fold)**
    
    This fold includes the models of five classifiers: *PCA(200PC)_Fknn_k3* (if samples and feature SNPs are both more than 200), *SVM(linear)*, *LogisticRegression_l2*, *Bagging_NB*, *RandomForest_gini_100_10maxFea*. By inputting the feature SNP information, these classifiers will output the predicted breed number for each sample(the breed number can be viewed in BreedNumber.txt).

- **featurelist_```SNPsNumberPerBreed```.txt**
    
    This file contains the row indices of the selected feature SNPs from the reference population's .bim file

- **rs_featurelist_```SNPsNumberPerBreed```.txt**
    
    This file lists the feature SNP IDs, which are corresponding to the indices of each line in the featurelist_```SNPsNumberPerBreed```.txt file.

- **dm_```SNPsNumberPerBreed```_ ```low threshold```_```high threshold```.result** and **model_report.csv**
    
    This first file includes the D<sub>m</sub> value of each feature SNPs, the second file includes the performance of five classifiers.



## **hit-ancestry: ancestry estimation**
``` hit-ancestry ``` is a module that utilizes the predicted probabilities of classifies in "feature SNP screening" to predict the ancestral populations of an individual.

### Usage

```bash
hit-ancestry -s parameters2.txt
```

### Input Parameter

Parameters2.txt contains the necessary parameters for running hit-ancestry, with an example provided below:

```
WorkSpace = "/home/user/project/workspace/"
ReferencePopulation = "/home/user/project/data/ref_pop"
BreedInformation = "/home/user/project/data/ref_breed.csv"
Simulation = "/home/user/project/data/sim.txt"
SNPinformation = "/home/user/project/workspace/hit-feature_output/result/rs_featurelist.txt"
Classifier1 = "/home/user/project/workspace/hit-feature_output/result/model_set"
Plink = "/path/to/plink"
Threads = "20" 
sim_cycles = "5"
```

- **Description**：
  - **WorkSpace**：Specifies the absolute path to the workspace directory where output files will be saved.
  - **ReferencePopulation**: Specifies the absolute path to the PLINK files for the reference population, without including file extensions (e.g., .bed, .bim, .fam). Ensure the path does not include any suffixes.
  - **BreedInformation**: Specifies the absolute path to the CSV file containing breed information for the reference population.
  - **Simulation**: Specifies the absolute path to the simulation parameter card file. The simulation parameter card includes the scheme number, as well as the breed names of the ancestors and their corresponding composition proportions for that specific scheme.
  - **SNPinformation**: The list of feature SNP IDs output from the hit-feature.
  - **Classifier1**: The directory containing the model files for the five classifiers. This directory is generated as output by hit-feature.
  - **Plink**: Specifies the absolute path to the PLINK software executable.
  - **Threads**: Specifies the number of threads. The default is ```"1"```.
  - **sim_cycles** : Defines the number of cycles used for the simulation. For each simulation scheme, the total number of simulated individuals is calculated as: 5 (number of cv) * cycle number * the sample size of the breed with the fewest individuals in the simulated ancestors.


### Input File

- **Simulation**

    It is a text file containing detailed information about the simulation schemes. For each scheme, the breed names and their corresponding proportions must match one-to-one, and the sum of the composition proportions should equal 1.

    Please ensure that the delimiters between the scheme number, breed names, and composition proportions match those in the example file. The example file is provided below:

```
Scheme1:"Breed1,Breed2;0.5,0.5"
Scheme2:"Breed1,Breed2,Breed3;0.25,0.25,0.5"
```

- **ReferencePopulation** and **BreedInformation**
    
    These two files are the input files required in ```hit-feature```.

- **SNPinformation** and **Classifier1**
    
    The files and folders listed above are the outputs of ```hit-feature```.

### Output File

The output includes the model_set folder, which contains the four classifier model files for classifier2. Four classesifiers include 


Additionally, there is a ```results.csv``` file that details the performance of the four classifiers.


## **hit-minimum: minimum subset selection**

``` hit-minimum ``` is a tool based on the output of ```hit-feature```, designed to identify the smallest possible subset of feature SNPs while ensuring a certain level of breed differentiation ability.

### Usage

```bash
hit-minimum -s parameters3.txt
```

### Input Parameter

Parameters3.txt contains the necessary parameters for running hit-minimum, with an example provided below:

```
WorkSpace = "/home/user/project/workspace/"
indexFile="/home/user/project/workspace/hit-feature_output/featurelist.txt" 
SNPinformation = "/home/user/project/workspace/hit-feature_output/rs_featurelist.txt"
TrainX="/home/user/project/workspace/hit-feature_output/CVi/X_trainCVi.npy"
Trainy="/home/user/project/workspace/hit-feature_output/CVi/y_trainCVi.npy"
TestX="/home/user/project/workspace/hit-feature_output/CVi/X_testCVi.npy"
Testy="/home/user/project/workspace/hit-feature_output/CVi/y_testCVi.npy"
k_features="10"
```

- **Description**：
  - **WorkSpace**：Specifies the absolute path to the workspace directory where output files will be saved.
  - **indexFile**:The list of feature SNP indexs output from the hit-feature.
  - **SNPinformation**: The corresponding list of feature SNP IDs output from the hit-feature.
  - **TrainX**: The absolute path to the corresponding intermediate file "X_train_CVi.npy" generated by hit-feature.
  - **Trainy**:The absolute path to the corresponding intermediate file "y_train_CVi.npy" generated by hit-feature.
  - **TestX**: The absolute path to the corresponding intermediate file "X_test_CVi.npy" generated by hit-feature.
  - **Testy**: The absolute path to the corresponding intermediate file "y_test_CVi.npy" generated by hit-feature.
  - **k_features**: Specifies the desired total number of feature SNPs selected. This must be an integer less than the number of feature SNPs in ```indexFile```, typically set to```"100"```.

### Input File

All input files must originate from the output of ```hit-feature```, and they should correspond to the results of the same cross-validation (CV) iteration. Users can change the specific CVi if needed.

### Output File

The results include the features SNPs information of minimum subset.

- **MinimumSubset_```k_features```.txt**

    This file contains the row indices of the selected feature SNPs from the reference population's .bim file

- **rs_MinimumSubset_```k_features```.txt**

    This file lists the feature SNP IDs, which are corresponding to the indices of each line in the featurelist_```k_features```.txt file.

