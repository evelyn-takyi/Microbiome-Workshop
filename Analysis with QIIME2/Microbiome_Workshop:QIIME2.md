
# Amplicon sequence analysis with QIIME2 on bluewaves cluster

```
There are a number of great software packages for general amplicon analysis. 
Some examples include Mothur, Phyloseq, Dada2, UPARSE and QIIME 1 and QIIME 2. 
```

###  Raw sequencing dataset. (Paired end or single end sequences)
```
## An example of a paired end sequencing
RS412_S16_L001_R1_001.fastq.gz  RS412_S16_L001_R2_001.fastq.gz
```
Forward read
```
@M00763:337:000000000-CLFHY:1:1101:16324:1690 1:N:0:AAGAGGCA+TTATGCGA
CAACGCGAAGAACCTTACCAGGGTTTGACATCCTGCGAATCTCCTGGAAACGGGAGAGTGCCTTCGGGAACGCAGT
+
-8AC?@CEEEGFGGDGFE?EDDF;C:BEFGCF@EEGGGGGGFDGGFFGGGFGGGGGGG<EFGFCFGGGGGGGGGGC
```

####  Reverse read
```
@M00763:337:000000000-CLFHY:1:1101:15285:4581 1:N:0:AAGAGGCA+TTATGCGA
AAACGCGAAAAACCTTACCTACTCTTGACATCTACAGGATCCTGCGGAGACGCGGGAGTGCCTTCGGGAACTGTAA
+
68AC<CFGGGGGGDGFGEFFGGGGFG9EFFDGGGGGCFGGGGGGGGGGGGGCGFDEG7FDGGGGGGGCFGGFGGFG
```

## Understanding QIIME2 files

```
QIIME2 uses two different file types that contain the data and metadata from an analysis: .qza files are data files while .qzv files are visualizations. 
The authors of QIIME2 call these data files “data artifacts” to indicate that they are objects containing data and metadata about an experiment. 
It is really just a zip file containing a specially formatted directory with data and metadata. You can see what type of data is contained in a data file with the command qiime tools peek filename.qza.
```
## Data Visualization
```
You can then create visualisations from the output files
All of QIIME2 files can be viewed using an online browser that is available at https://view.qiime2.org. .qza files will contain basic info (name, universally unique identifier, data type and data format) as well ad a graph of data provenance.
.qzv files will contain all of that and graphic visualizations. 
```

## Analysis of sequence data files in QIIME2

#### 1.  Files to generate to use in QIIME2
	A. Manifest file : must be saved in csv file.(This is a  simple tab-delimited file with the sample ID and path of each sequence file:
```
sample-id	        absolute-filepath	                    direction
ET101_S1	$PWD/00_RAW_gz/ET101_S1_L001_R1_001.fastq.gz	forward
ET101_S1	$PWD/00_RAW_gz/ET101_S1_L001_R2_001.fastq.gz	reverse
ET102_S2	$PWD/00_RAW_gz/ET102_S2_L001_R1_001.fastq.gz	forward
ET102_S2	$PWD/00_RAW_gz/ET102_S2_L001_R2_001.fastq.gz	reverse
ET103_S3	$PWD/00_RAW_gz/ET103_S3_L001_R1_001.fastq.gz	forward
ET103_S3	$PWD/00_RAW_gz/ET103_S3_L001_R2_001.fastq.gz	reverse
ET104_S4	$PWD/00_RAW_gz/ET104_S4_L001_R1_001.fastq.gz	forward
ET104_S4	$PWD/00_RAW_gz/ET104_S4_L001_R2_001.fastq.gz	reverse
ET105_S5	$PWD/00_RAW_gz/ET105_S5_L001_R1_001.fastq.gz	forward
ET105_S5	$PWD/00_RAW_gz/ET105_S5_L001_R2_001.fastq.gz	reverse
ET106_S6	$PWD/00_RAW_gz/ET106_S6_L001_R1_001.fastq.gz	forward
ET106_S6	$PWD/00_RAW_gz/ET106_S6_L001_R2_001.fastq.gz	reverse
ET107_S7	$PWD/00_RAW_gz/ET107_S7_L001_R1_001.fastq.gz	forward
```		
	B. Metatdata file :saved as a text file that contains information about your samples. An example is below
```
 #SampleID	SampleName	Trial	Treatment   TankLocation  SampleType
ET201_S1	VIMS3_CONA_1	T3	 C		      A	         larvae
ET202_S13	VIMS3_CONA_2	T3	 C		      A	         larvae
ET203_S25	VIMS3_CONA_3	T3	 C		      A	         larvae
ET204_S37	VIMS3_CONB_1	T3	 C		      B	         larvae
ET205_S49	VIMS3_CONB_2	T3	 C		      B	         larvae
ET206_S61	VIMS3_CONB_3	T3	 C		      B	         larvae
ET207_S73	VIMS3_CONC_1	T3	 C		      C	         larvae
ET208_S85	VIMS3_CONC_2	T3	 C		      C	         larvae
ET209_S2	VIMS3_CONC_3	T3	 C		      C	         larvae
```

### 2. Set up your working directory on bluewaves

#### Log in to bluewaves on the terminal window
#### In your homespace or other desired location, make a directory(folder)
```
mkdir microbiome_workshop
```

##### Change into directory microbiome_workshop
```
cd microbiome_workshop
```

### 3. Importing raw sequence files (FASTQ files) data onto bluewaves cluster

#### To copy sequence folders files from computer to the director /microbiome_workshop
```
scp -r Gomez-Chiarri_ET1-87-189011825/ evelyn-takyi@bluewaves:/data3/marine_diseases_lab/evelyn/microbiome_workshop
```
#### Make a new directory and name as "00_RAW_gz" 
```
[evelyn-takyi@n040 microbiome_workshop]$ mkdir 00_RAW_gz
```
#### Copy all sequence files into 00_RAW_gz folder
```
[evelyn-takyi@n040 microbiome_workshop]$ for i in $(ls -d -- ET*/); do cp ./${i}/* 00_RAW_gz/; done
```
#### Change directory into 00_RAW_gz/
```
[evelyn-takyi@n040 microbiome_workshop]$ cd 00_RAW_gz/
```

### Load QIIME2 software on the bluewaves cluster
```
module load QIIME2
```


### 4. Importing sequencing files in folder data /00_RAW_gz into QIIME2 


### Files to generate before importing data into QIIME2
```
To import files, we need to create a manifest file  
 a. Manifest file(if using fastq file): Always make sure comma in each is aligned, correctly assign sample ID's.
b. Metadata
```
#### Copy the manifest file from desktop  into the 00_RAW_gz folder on bluewaves.
```
MOOK Trial1 evelyntakyi$ scp 16S_sample_manifest.csv evelyn-takyi@bluewaves:/data/marine_diseases_lab/evelyn-takyi/microbiome workshop/00_RAW_gz
```


### 5.  Demultiplexing data

### Demultiplexing data (i.e. mapping each sequence to the sample it came from): If your samples have not yet been demultiplexed

##### To demultiplexed , you can use the plugins from QIIME2 
```
q2-demux or cutadapt
```
#### For this project the reads were sequences using Illumina paired-end, 250 base pair reads with forward and reverse reads in separate files.
```
Example of demultiplexed sequence reads

ET130_S40_L001_R1_001.fastq.gz  
ET130_S40_L001_R2_001.fastq.gz  
ET131_S48_L001_R2_001.fastq.gz  
ET132_S56_L001_R1_001.fastq.gz  
ET132_S56_L001_R2_001.fastq.gz  
```

### The files used in this tutorial are demultiplexed so you import them into QIIME2 using the manifest file
```
[evelyn-takyi@n045 NEW]$ time qiime tools import --type 'SampleData[PairedEndSequencesWithQuality]' --input-path /data3/marine_diseases_lab/evelyn/microbiome workshop/00_RAW_gz/sample-manifest.csv --output-path demuz.qza --input-format PairedEndFastqManifestPhred33
```
#### The fastq files is imported in to a QIIME2 data artifact ending in .qza



### 6. Evaluate data quality
```
Examine the quality of the data
We can view the characteristics of the dataset and the quality scores of the data by creating a QIIME2 visualization artifact.
This will create a visualization file. You can download the file to your local computer. From a new terminal window on your local computer copy the file:
Now you can view the file on your local computer using the QIIME2 visualization server. 
When viewing the data look for the point in the forward and reverse reads where quality scores decline below 25-30. We will need to trim reads to this point to create high quality sequence variants.
```
#### To view the demultiplexed file to check quality scores
```
[evelyn-takyi@n045 NEW]$ qiime demux summarize  --i-data demuz.qza  --o-visualization demuz.qzv
Saved Visualization to: demuz.qzv
```

##### Click on the link below to view and example of sequence quality output from QIIME2
```
https://view.qiime2.org/visualization/?type=html&src=2e3c11ec-b4d1-4bdd-a78d-fcb8be20fe54
```
#### Demultiplexed sequence counts summary
Minimum:	|	304
Median:		|	76691.0
Mean:		|	90580.51
Maximum:	|	272393
Total:		|	18116102

##### Per-sample sequence counts

Total Samples: 200

Sample name	Sequence count
ET174_S16	272393
ET161_S36	255862
ET173_S8	252254
ET165_S7	251485
ET159_S22	244663
ET167_S23	237313
ET166_S15	233751



### 7.  Denoising sample sequences(removal of noisy sequences)
Denoising step performs:

B. Removing non-biological parts of the sequences (i.e. primers, chimeric sequences,remove singletons)
C. Performing quality control
A. Sequence variants calling and generating ASV/feature count tables
D. Join denoised paired-end reads (in the case of DADA2), and then dereplicate those sequences. 😎


##### NOTES
```
Sequence variant selection is the slowest step in the tutorial. For that reason it is best to submit this step using the SLURM Sbatch scheduler.

Selecting Sequence Variants
The process of selecting sequence variants is the core processing step in amplicon analysis. 
This takes the place of “OTU picking” a method of clustering similar data together that was the common method for dealing with sequencing errors until last year. 
Three different methods have been published to select sequence variants, 
1. Dada2 uses and statistical error correction model 
2. Deblur takes an information theoretic approach
3. UNOISE2 applies a heuristic. 
Each of these methods attempt to remove or correct reads with sequencing errors and then remove chimeric sequences originating from different DNA templates. 

```
#### For the next step in the process, we will use the Dada2 method. 

```
[evelyn-takyi@n045 NEW]$ qiime dada2 denoise-paired  --i-demultiplexed-seqs demuz.qza  --p-trim-left-f 19  --p-trim-left-r 19  --p-trunc-len-f 76 --p-trunc-len-r 76 --o-table table.qza --o-representative-sequences rep-seqs.qza --o-denoising-stats denoising-stats.qza
```
#### After this process, we get an two files. Representative sequences and the denoising statics, feature table
#### To visualize the output files
```
[evelyn-takyi@n045 NEW]$ qiime feature-table tabulate-seqs  --i-data rep-seqs.qza --o-visualization rep-seqs.qzv

[evelyn-takyi@n045 NEW]$ qiime metadata tabulate --m-input-file denoising-stats.qza  --o-visualization denoising-stats.qzv

[evelyn-takyi@n045 NEW]$ qiime feature-table summarize --i-table table.qza --o-visualization table.qzv --m-sample-metadata-file VIMS_16S_Metadata.txt

```

### 8. Phylogenetics

#### There are a number of diversity metrics like unifrac distance that require the construction of a phylogenetic tree.

##### Multiple sequence alignment
```
[evelyn-takyi@n045 NEW]$ time qiime alignment mafft   --i-sequences rep-seqs.qza   --o-alignment aligned-rep-seqs.qza
```

##### Masking sites
Some sites in the alignment are not phylogenetically informative. These sites are masked
```
[evelyn-takyi@n045 NEW]$ time qiime alignment mask --i-alignment aligned-rep-seqs.qza  --o-masked-alignment masked-aligned-rep-seqs.qza
```
##### Creating a tree
Fastree is used to generate a phylogenetic tree from the masked alignment.
```
[evelyn-takyi@n045 NEW]$ time qiime phylogeny fasttree --i-alignment masked-aligned-rep-seqs.qza --o-tree unrooted-tree.qza
```
#### Midpoint rooting
Fastree creates an unrooted tree. We can root the tree at it’s midpoint with this command

```
[evelyn-takyi@n045 NEW]$ time qiime phylogeny midpoint-root --i-tree unrooted-tree.qza --o-rooted-tree rooted-tree.qza
```


### 9. Assigning taxonomy
#### Taxonomic analysis
#### Sequence variants are of limited usefulness by themselves. Often we are interested in what kinds of organisms are present in our sample, not just the diversity of the sample. To identify these sequence variants two things are needed: a reference database and an algorithm for identifying the sequence using the database.

A. Greengages
B. Silva
C. RDP database
D. UNITE

There are several methods of taxonomic classification available. The most commonly used classifier is the RDP classifier. Other software includes SINTAX and 16S classifier. 
We will be using the QIIME2’s built-in naive Bayesian classifier (which is built on Scikit-learn but similar to RDP), noting that the method, while fast and powerful, has a tendency over-classify reads.

There are two steps to taxonomic classification: training the classifier (or using a pre-trained dataset) and classifying the sequence variants. Generally it is best to train the classifier on the exact region of the 16S, 18S or ITS you sequenced.
For this tutorial we will be using a classifier model trained on the Silva 99% database trimmed to the V4 region.

```
qiime feature-classifier extract-reads --i-sequences initialreads.qza --p-f-primer MNAMSCGMNRAACCTYANC --p-r-primer CGACRRCCATGCANCACCT --p-trunc-len 76 --p-min-length 65 --p-max-length 85 --o-reads ref-seqs.qza
```
```
qiime feature-classifier classify-sklearn  --i-classifier classifier.qza  --i-reads rep-seqs.qza  --o-classification taxonomy.qza
```
```
[evelyn-takyi@n044 vims]$ qiime tools export --input-path taxonomy.qza --output-path taxonomy-with-spaces
```
```
[evelyn-takyi@n044 vims]$ qiime metadata tabulate --m-input-file taxonomy-with-spaces/taxonomy.tsv  --o-visualization taxonomy-as-metadata.qzv
```
```
qiime tools export --input-path taxonomy-as-metadata.qzv  --output-path taxonomy-as-metadata
```
```
[evelyn-takyi@n044 vims]$ qiime tools import  --type 'FeatureData[Taxonomy]'  --input-path taxonomy-as-metadata/metadata.tsv  --output-path taxonomy-without-spaces.qza
```
```
[evelyn-takyi@n044 vims]$ qiime taxa barplot --i-table table.qza --i-taxonomy taxonomy-without-spaces.qza  --m-metadata-file VIMS_16S_Metadata.txt --o-visualization taxa-bar-plots.qzv
```

### ASV Count tables
```
[evelyn-takyi@n044 vims]$ qiime tools export --input-path table.qza --output-path table
```
```
[evelyn-takyi@n044 table]$ biom convert -i feature-table.biom  -o table.tsv --to-tsv
```

##### Export ASV count table, taxonomy metadata and meta table into R
 
