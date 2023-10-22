# DTNPerturbome
<h1>
Identification of drug candidates for Multiple Sclerosis by the network-based module identification and perturbation analysis
</h1>

<p align="center">
<a href="#introduction">Introduction</a> &nbsp;&bull;&nbsp;
<a href="#installation">Installation</a> &nbsp;&bull;&nbsp;
<a href="#usage">Usage</a> &nbsp;&bull;&nbsp;
<a href="#contact">Contact</a>
</p>

## Introduction
**DTNPerturbome** is an integrative network-based drug repurposing approach, bridging module identification in network biology and PRS in computational biophysics, focusing on the DTN of MS.

### Organization
```
DTNPerturbome/
├── LICENSE
├── README.md
├── network_construction/     <- Data and scripts for the construction of the MS-related network.
├── module_identification/    <- Data and scripts for the identification of the MS-related therapeutic module.
├── drug_ranking/             <- Data and scripts for the PRS-based drug repurposing calculation for MS.
├── moa_analysis.R            <- Script for the MoA analysis.
└── visualization/            <- Scripts for the visualization of results.
```

## Installation

### Requirements
- R 4.1.0
- Python 3.9
- pip

### Download the complete repository
Use the git clone command to clone the repository.
```
git clone https://github.com/CSB-SUDA/DTNPerturbome.git
```

### Install the R package version
We have developed the R package DTANetPerturbeR to provide a more convenient and user-friendly tool. The R package can be used for a variety of diseases, not limited to just MS.

For more details about DTANetPerturbeR, please visit
https://github.com/BarleyDavana/DTANetPerturbeR

Use the install_github function from the devtools library to install the R package.
```
install.packages("devtools")
library(devtools)
install_github("BarleyDavana/DTANetPerturbeR")
```
> **Note:** You can obtain quick drug prediction results by inputting genes associated with any disease of interest through the R package. However, the target module identification process in the package is simplified and relies solely on the target proportion within each module.

### Install the dependencies
#### Installation of DeepPurpose
The DeepPurpose library is required for affinity prediction. To install it, you need to create and activate a new conda environment, install RDKit and Jupyter Notebook, install the descriptastorus dependency, and finally install DeepPurpose.
```
conda create -n DeepPurpose python=3.9
conda activate DeepPurpose
conda install -c conda-forge rdkit
conda install -c conda-forge notebook
pip install git+https://github.com/bp-kelley/descriptastorus
pip install DeepPurpose
```
For more details about DeepPurpose, please visit
https://github.com/kexinhuang12345/DeepPurpose

#### Installation of ProDy
The ProDy package is required for PRS calculation. To install it, you can easily using the following command:
```
pip install prody
```
For more details about ProDy, please visit
https://github.com/prody/ProDy

## Usage

### Construction of the disease network

#### Organization
```
network_construction/
├── Disease_Genes.txt     <- Dataset of disease genes
├── PPIN_human.txt        <- Human PPI data (combined score ≥ 400)
├── getCommonGenes.R      <- Script for getting common genes
├── getInitialNet.R       <- Script for getting the seed PPIN
├── runRWR.R              <- Script for constructing the RWR
├── getEnlargedNet.R      <- Script for getting the enlarged PPIN
└── targetdens_analysis.R <- Script for analysis the target density in PPIN
```

#### Usage Example
Load the dataset of disease genes from the txt file and create an output directory to store the results of your analysis.
```R
# Load disease genes from file
gene_list <- read.table("MS_Genes.txt", header = TRUE, sep = '\t')

# Create output directory
dir.create("output_Files")
```
Use the `getCommonGenes.R` script to identify common genes between your disease genes and those associated with other diseases.
```R
# Get common genes with other diseases
common_genes <- getCommonGenes(gene_list)
```
Use the `getInitialNet.R` script to obtain the initial seed PPIN using the common genes identified in the previous step.
```R
# Get seed PPIN using common genes
seed_info <- getInitialNet(common_genes，"Seed_PPIN")
seed_genes <- seed_info$seed_genes
```
Use the `getEnlargedNet.R` script to extend the disease network by adding genes relevant to the disease. This step creates a more comprehensive view of the disease network.
```R
# Get enlarged PPIN by RWR
disease_net <- getEnlargedNet(seed_genes)
```

### Identification of the disease therapeutic module

#### Organization
```
module_identification/
├── DTI_DrugBank.txt          <- Drug target data in DrugBank
├── Tars_TTD.txt              <- Drug target data in TTD
├── getCommunities.R          <- Script for detecting community
├── mapTargets.R              <- Script for mapping drug targets to PPIN
└── topological_analysis/     <- Directory containing scripts for topological analysis
  ├──proportion_analysis.R    <- Script for calculating target proportion in each module
  ├──zipi_analysis.R          <- Script for calculating within-module degree and participation coefficient
  └──druggability_analysis.sh <- Script for calculating druggability
```

#### Usage Example
Load the disease network from the txt file and create an output directory to store the results of your analysis.
```R
# Load disease network from a txt file containing network edges
disease_net <- read.table("disease_net.txt", header = TRUE)

# Create output directory
dir.create("output_Files")
```
Use the `getCommunities.R` script to detect network communities within the disease network and the `mapTargets.R` script to map drug targets within the disease network.
```R
# Module detection
modules_info <- getCommunities(disease_net)

# Map targets to genes in disease net
net_tars <- mapTargets(disease_net)
```
> **Note:** After running the above steps, you will find `node_Module.txt`, `edge_Module.txt`, and `Net_Tars.txt` files in the `output_Files` directory.

Annotation analysis for each module to select the most promising module as the potential therapeutic module.
```R
# Analysis of target proportion in each module
tar_proportion <- analyzeProportion("edge_Module.txt", "Net_Tars.txt")

# Analysis of within-module degree and participation coefficient
zi_pi <- analyzeZiPi("disease_net.txt", "node_Module.txt")
```
For further analysis of module druggability, you can use the `druggability_analysis.sh` script. Make sure to meet the following requirements and considerations before using the script.

<details>
<summary>Click to see the requirements</summary>

- fpocket must be installed and accessible from the command line.
- This script is designed to run on a Linux system.
- Input text files (like uniprotID.txt) should be in Linux format (LF line endings). You can use tools like 'dos2unix' to convert if necessary.
- To execute the script, place it in the same directory as your .txt files, and then run with './druggability_analysis.sh'.

</details>

### PRS-based drug repurposing calculation

#### Organization
```
drug_ranking/
├── DTI_DrugBank.txt             <- Drug-target interaction data in DrugBank
├── DTI_TTD.txt                  <- Drug-target interaction data in TTD
├── Drugs_DrugBank.txt           <- Drug data in DrugBank
├── Drugs_TTD.txt                <- Drug data in TTD
├── Drugs_supp_TTD.txt           <- Supplementary drug data in TTD
├── mapDrugs.R                   <- Script for constracting the DTN
├── enm/                         <- Enm class and related functions
├── predict_binding_affinity.py  <- Script for predicting binding affinity for each DTI
├── calculate_PRS.py             <- Script for calculating the ps score for each DTI and PS score for each drug
└── runDrugScoring.R             <- Script for drug scoring in R and calling Python scripts.
```

#### Usage Example
Save the therapeutic module information in a list, including the nodes, edges, and targets that constitute the therapeutic module.
```R
# Save the therapeutic module information in a list
target_module_info <- list(nodes = target_module_nodes,
                           edges = target_module_edges,
                           targets = target_module_targets)
```
Use the `mapDrugs.R` script to construct the DTN based on the therapeutic module. The result is stored in a file named `Target_module_dti.txt`.
```R
# Map drugs to targets in the therapeutic module
mapDrugs(target_module_info)
```
Get `proteins.fasta` data (sequence information of the target in the therapeutic module, which can be downloaded from UniProt database) ready and run the drug scoring process using the `runDrugScoring.R` script.
```R
runDrugScoring(python_exe = "python", dti_file = "Target_module_dti.txt", fasta_file = "proteins.fasta", network_file = "Target_module_edges.txt", output_folder = "output_Files")
```

At the conclusion of the execution, you will obtain two result files:
- `prs_dti_ps.csv`: This file contains the ps score for each DTI.
- `drug_PS.csv`: This file contains the PS score for each drug.

> **Note:** For further Mechanism of Action (MoA) analysis, refer to the script `moa_analysis.R`.

## Contact
Please contact yitanlu02@gmail.com to report issues of for any questions.
