## Methylformer
Methylformer is a deep-learning tool based on a pre-trained Nucleotide Transformer (NT) model. It can predict DNA methylation levels based on DNA sequences, and predict the impact of genetic variations on DNA methylation (more details, see paper). Methylformer takes the input of tissue specific DNA methylation labels of CpG sites and DNA sequences (with the replacement of SNVs) for model training and testing. Methylformer can predict whether the SNV affect the DNAm level of a target CpG site.

## Prerequisites
```bash
pyfaidx>=0.6.3.1,
pysam>=0.19.1,
numpy>=1.22.4,
pandas>=1.2.3,
seaborn>=0.11.2,
torch>=1.5.0 #CPU version
torch>=1.9.0 #GPU version
pytorch 
Transformers
```

Install PyTorch following instructions from https://pytorch.org/ and Install Transformers  following instructions from https://huggingface.curated.co. Use pip install -r requirements.txt to install the other dependencies.

## Usage


Step1. Data preparation 

The first step involves preparing the dataset needed for model training. This requires inputting the β values (quantifying DNA methylation levels) of CpG sites for each sample (can be from different tissues), a 400bp DNA sequence spanning each CpG site based on the human reference genome (hg19), and the SNVs information (derived from whole genome sequencing) for each sample.

In detail: The β values are converted into a binary classification of hypermethylation (1) or hypomethylation (0), using a β value threshold of 0.5. Based on SNV information, we replaced the human reference sequence at the corresponding positions within the 400 bp window spanning the targeted CpG site. In a population, for each identical DNA sequence, only those with the same methylation status are retained, while DNA sequences showing both hypermethylated and hypomethylated statuses (0 and 1) in a population are removed. This process generated a dataset, which was randomly divided into training and testing datasets in a 7:3 ratio. That can be further used for model training and testing.

The example input file (cpg_betavalue.csv) include the β values of CpG sites of each sample.


<table class="table table-responsive-{sm|md|lg|xl" style="font-size: 5px; width: auto !important; margin-left: auto; margin-right: auto;">
 <thead>
  <tr>
   <th style="text-align:right;position: sticky; top:0; background-color: #FFFFFF;text-align: center;"> cpg </th>
   <th style="text-align:right;position: sticky; top:0; background-color: #FFFFFF;text-align: center;"> sample1 </th>
   <th style="text-align:left;position: sticky; top:0; background-color: #FFFFFF;text-align: center;"> sample2 </th>
  </tr>
 </thead>
<tbody>
  <tr>
   <td style="text-align:right;text-align: center;"> cg14817997 </td>
   <td style="text-align:right;text-align: center;"> 0.42 </td>
   <td style="text-align:left;text-align: center;"> 0.33 </td>
  </tr>
  <tr>
   <td style="text-align:right;text-align: center;"> cg26928153 </td>
   <td style="text-align:right;text-align: center;"> 0.33 </td>
   <td style="text-align:left;text-align: center;"> 0.4 </td>
  </tr>
  <tr>
   <td style="text-align:right;text-align: center;"> cg16269199 </td>
   <td style="text-align:right;text-align: center;"> 0.97 </td>
   <td style="text-align:left;text-align: center;"> 0.89 </td>
  </tr>
  <tr>
   <td style="text-align:right;text-align: center;"> … </td>
   <td style="text-align:right;text-align: center;">   </td>
   <td style="text-align:left;text-align: center;">   </td>
  </tr>
</tbody>
</table>  

cpg_refseq.csv: the DNA sequence of 400 bp surrounding each CpG site from the human reference genome (hg19).

<table class="table table-responsive-{sm|md|lg|xl" style="font-size: 5px; width: auto !important; margin-left: auto; margin-right: auto;">
 <thead>
  <tr>
   <th style="text-align:right;position: sticky; top:0; background-color: #FFFFFF;text-align: center;"> cpg </th>
   <th style="text-align:right;position: sticky; top:0; background-color: #FFFFFF;text-align: center;"> chr </th>
   <th style="text-align:left;position: sticky; top:0; background-color: #FFFFFF;text-align: center;"> position </th>
   <th style="text-align:left;position: sticky; top:0; background-color: #FFFFFF;text-align: center;"> seq </th>
  </tr>
 </thead>
<tbody>
  <tr>
   <td style="text-align:right;text-align: center;"> cg14817997 </td>
   <td style="text-align:right;text-align: center;"> chr1 </td>
   <td style="text-align:left;text-align: center;"> 10525 </td>
   <td style="text-align:left;text-align: center;"> CTAACC…… </td>
  </tr>
  <tr>
   <td style="text-align:right;text-align: center;"> cg26928153 </td>
   <td style="text-align:right;text-align: center;"> chr1 </td>
   <td style="text-align:left;text-align: center;"> 10848 </td>
   <td style="text-align:left;text-align: center;"> GCGCAG…… </td>
  </tr>
  <tr>
   <td style="text-align:right;text-align: center;"> cg16269199 </td>
   <td style="text-align:right;text-align: center;"> chr1 </td>
   <td style="text-align:left;text-align: center;"> 10850 </td>
   <td style="text-align:left;text-align: center;"> GCAGAG…… </td>
  </tr>
  <tr>
   <td style="text-align:right;text-align: center;"> … </td>
   <td style="text-align:right;text-align: center;">   </td>
   <td style="text-align:left;text-align: center;">   </td>
   <td style="text-align:left;text-align: center;">   </td>
  </tr>
</tbody>
</table>  

snp.vcf: the SNP information for each sample
<table class="table table-responsive-{sm|md|lg|xl" style="font-size: 1px; width: auto !important; margin-left: auto; margin-right: auto;">
 <thead>
  <tr>
   <th style="text-align:right;position: sticky; top:0; background-color: #FFFFFF;text-align: center;"> #CHROM </th>
   <th style="text-align:right;position: sticky; top:0; background-color: #FFFFFF;text-align: center;"> POS </th>
   <th style="text-align:left;position: sticky; top:0; background-color: #FFFFFF;text-align: center;"> ID </th>
   <th style="text-align:left;position: sticky; top:0; background-color: #FFFFFF;text-align: center;"> REF </th>
   <th style="text-align:right;position: sticky; top:0; background-color: #FFFFFF;text-align: center;"> ALT </th>
   <th style="text-align:right;position: sticky; top:0; background-color: #FFFFFF;text-align: center;"> QUAL </th>
   <th style="text-align:left;position: sticky; top:0; background-color: #FFFFFF;text-align: center;"> FILTER </th>
   <th style="text-align:left;position: sticky; top:0; background-color: #FFFFFF;text-align: center;"> INFO </th>
   <th style="text-align:right;position: sticky; top:0; background-color: #FFFFFF;text-align: center;"> FORMAT </th>
   <th style="text-align:left;position: sticky; top:0; background-color: #FFFFFF;text-align: center;"> sample1 </th>
   <th style="text-align:left;position: sticky; top:0; background-color: #FFFFFF;text-align: center;"> sample2 </th>
  </tr>
 </thead>
<tbody>
  <tr>
   <td style="text-align:right;text-align: center;"> chr1 </td>
   <td style="text-align:right;text-align: center;"> 10550 </td>
   <td style="text-align:left;text-align: center;"> . </td>
   <td style="text-align:left;text-align: center;"> G </td>
   <td style="text-align:right;text-align: center;"> C </td>
   <td style="text-align:right;text-align: center;"> 222.64 </td>
   <td style="text-align:left;text-align: center;"> PASS </td>
   <td style="text-align:left;text-align: center;"> …… </td>
   <td style="text-align:right;text-align: center;"> GT </td>
   <td style="text-align:right;text-align: center;"> 0/0 </td>
   <td style="text-align:left;text-align: center;"> 0/0 </td>
  </tr>
  <tr>
   <td style="text-align:right;text-align: center;"> chr1 </td>
   <td style="text-align:right;text-align: center;"> 10575 </td>
   <td style="text-align:left;text-align: center;"> . </td>
   <td style="text-align:left;text-align: center;"> C </td>
   <td style="text-align:right;text-align: center;"> G </td>
   <td style="text-align:right;text-align: center;"> 128.77 </td>
   <td style="text-align:left;text-align: center;"> PASS </td>
   <td style="text-align:left;text-align: center;"> …… </td>
   <td style="text-align:right;text-align: center;"> GT </td>
   <td style="text-align:right;text-align: center;"> 0/1 </td>
   <td style="text-align:left;text-align: center;"> 0/1 </td>
  </tr>
  <tr>
   <td style="text-align:right;text-align: center;"> chr1 </td>
   <td style="text-align:right;text-align: center;"> 10608 </td>
   <td style="text-align:left;text-align: center;"> . </td>
   <td style="text-align:left;text-align: center;"> A </td>
   <td style="text-align:right;text-align: center;"> C </td>
   <td style="text-align:right;text-align: center;"> 119.64 </td>
   <td style="text-align:left;text-align: center;"> PASS </td>
   <td style="text-align:left;text-align: center;"> …… </td>
   <td style="text-align:right;text-align: center;"> GT </td>
   <td style="text-align:right;text-align: center;"> 0/0 </td>
   <td style="text-align:left;text-align: center;"> 0/1 </td>
  </tr>
  <tr>
   <td style="text-align:right;text-align: center;"> … </td>
   <td style="text-align:right;text-align: center;">   </td>
   <td style="text-align:left;text-align: center;">   </td>
   <td style="text-align:left;text-align: center;">   </td>
   <td style="text-align:right;text-align: center;">   </td>
   <td style="text-align:left;text-align: center;">   </td>
   <td style="text-align:left;text-align: center;">   </td>
   <td style="text-align:right;text-align: center;">   </td>
   <td style="text-align:left;text-align: center;">   </td>
   <td style="text-align:left;text-align: center;">   </td>
   <td style="text-align:left;text-align: center;">   </td>
  </tr>
</tbody>
</table>  

```bash
$python 1_data_prepare.py 
	--betavalue ./input_files/cpg_betavalue.csv 
	--cpgref ./input_files/cpg_refseq.csv 
	--snpvcf ./input_files/snp.vcf 
	--out_dir ./output_files/

```

The results outputted to tmp_files include sequences for each sample with replaced SNVs.
The results outputted to output_files include the final dataset, containing training dataset and testing dataset.



<table class="table table-responsive-{sm|md|lg|xl" style="font-size: 5px; width: auto !important; margin-left: auto; margin-right: auto;">
 <thead>
  <tr>
   <th style="text-align:right;position: sticky; top:0; background-color: #FFFFFF;text-align: center;"> sequences </th>
   <th style="text-align:right;position: sticky; top:0; background-color: #FFFFFF;text-align: center;"> labels </th>
  </tr>
 </thead>
<tbody>
  <tr>
   <td style="text-align:right;text-align: center;"> CTAACCC…… </td>
   <td style="text-align:right;text-align: center;"> 0 </td>
  </tr>
  <tr>
   <td style="text-align:right;text-align: center;"> GCGCAGA…… </td>
   <td style="text-align:right;text-align: center;"> 0 </td>
  </tr>
  <tr>
   <td style="text-align:right;text-align: center;"> GCAGAGA…… </td>
   <td style="text-align:right;text-align: center;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:right;text-align: center;"> … </td>
   <td style="text-align:right;text-align: center;">   </td>
  </tr>
</tbody>
</table>  


Step2. Model training

To develop the Methylformer, we leveraged a Nucleotide Transformer[1]. foundation model that was pre-trained on billions of DNA sequences. We utilized the Hugging Face's transformers library to implement the Nucleotide Transformer. Additionally, we enhanced the model by adding a binary classification layer to predict DNA methylation status. For performance evaluation, we used the Matthews Correlation Coefficient (MCC) as the primary ranking metric, and other parameters (pls add). The best performing parameter (MCC) on predicting sequences from the testing dataset were saved.

```bash
$python 2_model_training.py 
	--model_name 'InstaDeepAI/nucleotide-transformer-500m-human-ref' 
	--train_file ./output_files/train.csv 
	--test_file ./output_files/test.csv 
	--train_epochs 20
	--output_dir ./model_output/

```


Step3. Methylformer prediction

3.1 Predict DNA methylation based on DNA sequences

The trained Methylformer model can take an input of DNA sequence and predict DNA methylation status of a target CpG site. The example input file (seq.csv) include the DNA sequences that needs to be predicted.

<table class="table table-responsive-{sm|md|lg|xl" style="font-size: 5px; width: auto !important; margin-left: auto; margin-right: auto;">
 <thead>
  <tr>
   <th style="text-align:right;position: sticky; top:0; background-color: #FFFFFF;text-align: center;"> sequences </th>
  </tr>
 </thead>
<tbody>
  <tr>
   <td style="text-align:right;text-align: center;"> CTAACCC…… </td>
  </tr>
  <tr>
   <td style="text-align:right;text-align: center;"> GCGCAGA…… </td>
  </tr>
  <tr>
   <td style="text-align:right;text-align: center;"> GCAGAGA…… </td>
  </tr>
  <tr>
   <td style="text-align:right;text-align: center;"> … </td>
  </tr>
</tbody>
</table>  

```bash
$python 3_predict.py 
	--tissue model-1000g-blood 
	--predict_file seq.csv
```

The example output file (seq_prediction.csv) include Methylformer predicted results.

<table class="table table-responsive-{sm|md|lg|xl" style="font-size: 5px; width: auto !important; margin-left: auto; margin-right: auto;">
 <thead>
  <tr>
   <th style="text-align:right;position: sticky; top:0; background-color: #FFFFFF;text-align: center;"> Labels </th>
  </tr>
 </thead>
<tbody>
  <tr>
   <td style="text-align:right;text-align: center;"> 0 </td>
  </tr>
  <tr>
   <td style="text-align:right;text-align: center;"> 0 </td>
  </tr>
  <tr>
   <td style="text-align:right;text-align: center;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:right;text-align: center;"> … </td>
  </tr>
</tbody>
</table>  


3.2 Prediction of DNA methylation regulatory variants

Methylformer can be used to predict DNAm regulatory genetic variations. The 400bp DNA sequences spanning the target CpG sites can be used for prediction. Then, we used the target SNV information to replace corresponding reference alleles to generate the variant sequence. Methylformer predicts the DNA methylation status of the CpG sites located on a reference (wt) or a variant (vt) DNA fragment. We then calculated the predicted DNA methylation difference (delta score) between reference and variant sequences. Specifically, the delta score is calculated as the methylation status of the variant sequence minus that of the reference sequence. The delta score of 1 or -1 signifies an increased or decreased DNA methylation levels of a specific CpG site, and we define those SNVs as DNAm regulatory variants.

The example input file (cpgref_forpredict.csv) include the DNA sequence of 400 bp surrounding candidate CpG sites from the reference genome (hg19).

<table class="table table-responsive-{sm|md|lg|xl" style="font-size: 5px; width: auto !important; margin-left: auto; margin-right: auto;">
 <thead>
  <tr>
   <th style="text-align:right;position: sticky; top:0; background-color: #FFFFFF;text-align: center;"> cpg </th>
   <th style="text-align:right;position: sticky; top:0; background-color: #FFFFFF;text-align: center;"> chr </th>
   <th style="text-align:right;position: sticky; top:0; background-color: #FFFFFF;text-align: center;"> position </th>
   <th style="text-align:right;position: sticky; top:0; background-color: #FFFFFF;text-align: center;"> seq </th>
  </tr>
 </thead>
<tbody>
  <tr>
   <td style="text-align:right;text-align: center;"> cg26959247 </td>
   <td style="text-align:right;text-align: center;"> chr3 </td>
   <td style="text-align:right;text-align: center;"> 11611863 </td>
   <td style="text-align:right;text-align: center;"> ACACCG…… </td>
  </tr>
  <tr>
   <td style="text-align:right;text-align: center;"> cg22573456 </td>
   <td style="text-align:right;text-align: center;"> chr3 </td>
   <td style="text-align:right;text-align: center;"> 12525702 </td>
   <td style="text-align:right;text-align: center;"> TGGCCA…… </td>
  </tr>
  <tr>
   <td style="text-align:right;text-align: center;"> cg26405148 </td>
   <td style="text-align:right;text-align: center;"> chr3 </td>
   <td style="text-align:right;text-align: center;"> 12525763 </td>
   <td style="text-align:right;text-align: center;"> GGTGCG…… </td>
  </tr>
  <tr>
   <td style="text-align:right;text-align: center;"> … </td>
   <td style="text-align:right;text-align: center;">   </td>
   <td style="text-align:right;text-align: center;">   </td>
   <td style="text-align:right;text-align: center;">   </td>
  </tr>
</tbody>
</table>  

```bash
$python 4_predict_dnam.py 
	--model Methylformer-CNS
	--cpgref cpgref_forpredict.csv 
	--snp snp_forpredict.csv
```

snp_forpredict.csv：candidate DNA methylation regulatory variants

<table class="table table-responsive-{sm|md|lg|xl" style="font-size: 5px; width: auto !important; margin-left: auto; margin-right: auto;">
 <thead>
  <tr>
   <th style="text-align:right;position: sticky; top:0; background-color: #FFFFFF;text-align: center;"> chr </th>
   <th style="text-align:right;position: sticky; top:0; background-color: #FFFFFF;text-align: center;"> pos </th>
   <th style="text-align:right;position: sticky; top:0; background-color: #FFFFFF;text-align: center;"> ref </th>
   <th style="text-align:right;position: sticky; top:0; background-color: #FFFFFF;text-align: center;"> alt </th>
  </tr>
 </thead>
<tbody>
  <tr>
   <td style="text-align:right;text-align: center;"> chr3 </td>
   <td style="text-align:right;text-align: center;"> 4640264 </td>
   <td style="text-align:right;text-align: center;"> A </td>
   <td style="text-align:right;text-align: center;"> C </td>
  </tr>
  <tr>
   <td style="text-align:right;text-align: center;"> chr3 </td>
   <td style="text-align:right;text-align: center;"> 11611763 </td>
   <td style="text-align:right;text-align: center;"> T </td>
   <td style="text-align:right;text-align: center;"> C </td>
  </tr>
  <tr>
   <td style="text-align:right;text-align: center;"> chr3 </td>
   <td style="text-align:right;text-align: center;"> 11611854 </td>
   <td style="text-align:right;text-align: center;"> G </td>
   <td style="text-align:right;text-align: center;"> C </td>
  </tr>
  <tr>
   <td style="text-align:right;text-align: center;"> … </td>
   <td style="text-align:right;text-align: center;">   </td>
   <td style="text-align:right;text-align: center;">   </td>
   <td style="text-align:right;text-align: center;">   </td>
  </tr>
</tbody>
</table>  

Output file: Methylformer-CNS_results.csv

<table class="table table-responsive-{sm|md|lg|xl" style="font-size: 5px; width: auto !important; margin-left: auto; margin-right: auto;">
 <thead>
  <tr>
   <th style="text-align:right;position: sticky; top:0; background-color: #FFFFFF;text-align: center;"> cpg </th>
   <th style="text-align:right;position: sticky; top:0; background-color: #FFFFFF;text-align: center;"> chr </th>
   <th style="text-align:right;position: sticky; top:0; background-color: #FFFFFF;text-align: center;"> cpg_pos </th>
   <th style="text-align:right;position: sticky; top:0; background-color: #FFFFFF;text-align: center;"> snv_pos </th>
   <th style="text-align:right;position: sticky; top:0; background-color: #FFFFFF;text-align: center;"> REF </th>
   <th style="text-align:right;position: sticky; top:0; background-color: #FFFFFF;text-align: center;"> ALT </th>
   <th style="text-align:right;position: sticky; top:0; background-color: #FFFFFF;text-align: center;"> seq_ref </th>
   <th style="text-align:right;position: sticky; top:0; background-color: #FFFFFF;text-align: center;"> seq_mut </th>
   <th style="text-align:right;position: sticky; top:0; background-color: #FFFFFF;text-align: center;"> Predicted_ref </th>
   <th style="text-align:right;position: sticky; top:0; background-color: #FFFFFF;text-align: center;"> Predicted_mut </th>
   <th style="text-align:right;position: sticky; top:0; background-color: #FFFFFF;text-align: center;"> delta </th>
  </tr>
 </thead>
<tbody>
  <tr>
   <td style="text-align:right;text-align: center;"> cg26959247 </td>
   <td style="text-align:right;text-align: center;"> chr3 </td>
   <td style="text-align:right;text-align: center;"> 11611863 </td>
   <td style="text-align:right;text-align: center;"> 11611763 </td>
   <td style="text-align:right;text-align: center;"> T </td>
   <td style="text-align:right;text-align: center;"> C </td>
   <td style="text-align:right;text-align: center;"> ACACCG…… </td>
   <td style="text-align:right;text-align: center;"> ACACCG…… </td>
   <td style="text-align:right;text-align: center;"> 1 </td>
   <td style="text-align:right;text-align: center;"> 1 </td>
   <td style="text-align:right;text-align: center;"> 0 </td>
  </tr>
  <tr>
   <td style="text-align:right;text-align: center;"> cg26959247 </td>
   <td style="text-align:right;text-align: center;"> chr3 </td>
   <td style="text-align:right;text-align: center;"> 11611863 </td>
   <td style="text-align:right;text-align: center;"> 11611763 </td>
   <td style="text-align:right;text-align: center;"> G </td>
   <td style="text-align:right;text-align: center;"> C </td>
   <td style="text-align:right;text-align: center;"> ACACCG…… </td>
   <td style="text-align:right;text-align: center;"> ACACCG…… </td>
   <td style="text-align:right;text-align: center;"> 1 </td>
   <td style="text-align:right;text-align: center;"> 1 </td>
   <td style="text-align:right;text-align: center;"> 0 </td>
  </tr>
  <tr>
   <td style="text-align:right;text-align: center;"> cg22573456 </td>
   <td style="text-align:right;text-align: center;"> chr3 </td>
   <td style="text-align:right;text-align: center;"> 12525702 </td>
   <td style="text-align:right;text-align: center;"> 12525605 </td>
   <td style="text-align:right;text-align: center;"> G </td>
   <td style="text-align:right;text-align: center;"> T </td>
   <td style="text-align:right;text-align: center;"> TGGCCA…… </td>
   <td style="text-align:right;text-align: center;"> TGGCCA…… </td>
   <td style="text-align:right;text-align: center;"> 0 </td>
   <td style="text-align:right;text-align: center;"> 0 </td>
   <td style="text-align:right;text-align: center;"> 0 </td>
  </tr>
  <tr>
   <td style="text-align:right;text-align: center;"> … </td>
   <td style="text-align:right;text-align: center;">   </td>
   <td style="text-align:right;text-align: center;">   </td>
   <td style="text-align:right;text-align: center;">   </td>
   <td style="text-align:right;text-align: center;">   </td>
   <td style="text-align:right;text-align: center;">   </td>
   <td style="text-align:right;text-align: center;">   </td>
   <td style="text-align:right;text-align: center;">   </td>
   <td style="text-align:right;text-align: center;">   </td>
   <td style="text-align:right;text-align: center;">   </td>
   <td style="text-align:right;text-align: center;">   </td>
  </tr>
</tbody>
</table>  

[1] Hugo Dalla-Torre, et al. The Nucleotide Transformer: Building and Evaluating Robust Foundation Models for Human Genomics. bioRxiv,  (2023).
