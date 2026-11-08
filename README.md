# GA5 answers - Md. Rafiqul Islam

[![DOI](https://sandbox.zenodo.org/badge/1449895291.svg)](https://handle.test.datacite.org/10.5072/zenodo.636776)

------------------------------------------------------------------------

## Part A: Your project dir, Git repo, and protocol

### 1: Fill in the top of the protocol

### 2: Markdown formatting


### 3: Create subdirs, and copy the scripts

```bash
mkdir -p scripts data/ref results
```

### 4: Initialize a Git repository, and commit your first files

### 5: Download the reference transcriptome

Code to count the number of transcripts:

```bash
zcat data/ref/GCF_016801865.2_TS_CPP_V2_rna.fna.gz | grep -c "^>"
```

The output of the command was:

```
28624
```

*Answer*: The number of transcripts in the file: 28624

------------------------------------------------------------------------

## Part B: Read trimming

### 6: Run TrimGalore

*Answer*: Whether all batch jobs succeeded, and how you checked that
(and any problems you encountered, and how you solved them):

- All the 22 batch jobs were completed successfully. I confirmed it by checking the success messages in slurm logs and verifying that the output contains 22 trimmed R1 and R2 files each, 44 FastQC HTML reports and 44 FastQC ZIP files. The command I used to check the successfully finished counts is as follows-

```bash
grep -c "Successfully finished script trimgalore.sh" slurm-trimgalore-*.out
```

------------------------------------------------------------------------

## Part C: Transcript quantification with Salmon

### 7: Create the Salmon index

Code to check the size of the index dir:

```bash
du -sh results/salmon_index
```

The output of the command was:

```
1.1G    results/salmon_index
```

*Answer*: How long it took to create the index, and how you found out: 

- It took **20 seconds**.

- I substracted the starting time point from ending time point to find out the required time to create the index.

*Answer*: The size of the index dir: **1.1 G**

### 8: Run Salmon for all samples

### 9: The transcripts with the most reads

*Answer*: What the `NumReads` column contains: It contains Salmon’s estimated number of reads assigned to each transcript. 

```bash
tail -n +2 results/salmon_quant/S63/quant.sf | sort -k5,5nr | head -n 3
```

The output of the command was:

```
XM_039591534.2  6566    6315.086        14518.656821    24004.889
XM_039573735.1  6445    6194.086        5342.073324     8663.254
XM_039573733.1  6591    6340.086        2876.480244     4774.749
```

### 10: The gene of the top transcript

```bash
zcat "$ref_transcripts" | grep -F ">XM_039591534.2 "
```

The output of the command was:

```
>XM_039591534.2 PREDICTED: Culex pipiens pallens vitellogenin-A1-like (LOC120426760), mRNA
```

*Answer*: The gene that the top transcript belongs to: 

- Vitellogenin-A1-like gene

### 11: Mapping rates

```bash
grep -i "mapping rate" results/salmon_quant/*/logs/salmon_quant.log
```

The output of the command was:

```
results/salmon_quant/S63/logs/salmon_quant.log:mapping rate: 82.4560%
results/salmon_quant/S64/logs/salmon_quant.log:mapping rate: 81.9152%
results/salmon_quant/S65/logs/salmon_quant.log:mapping rate: 79.1055%
results/salmon_quant/S66/logs/salmon_quant.log:mapping rate: 81.9350%
results/salmon_quant/S67/logs/salmon_quant.log:mapping rate: 83.0459%
results/salmon_quant/S68/logs/salmon_quant.log:mapping rate: 78.8070%
results/salmon_quant/S69/logs/salmon_quant.log:mapping rate: 82.1824%
results/salmon_quant/S70/logs/salmon_quant.log:mapping rate: 82.3060%
results/salmon_quant/S71/logs/salmon_quant.log:mapping rate: 79.9047%
results/salmon_quant/S74/logs/salmon_quant.log:mapping rate: 78.2753%
results/salmon_quant/S75/logs/salmon_quant.log:mapping rate: 80.3915%
results/salmon_quant/S76/logs/salmon_quant.log:mapping rate: 82.3396%
results/salmon_quant/S77/logs/salmon_quant.log:mapping rate: 82.3281%
results/salmon_quant/S78/logs/salmon_quant.log:mapping rate: 84.2490%
results/salmon_quant/S79/logs/salmon_quant.log:mapping rate: 80.4061%
results/salmon_quant/S80/logs/salmon_quant.log:mapping rate: 82.1721%
results/salmon_quant/S81/logs/salmon_quant.log:mapping rate: 83.6795%
results/salmon_quant/S82/logs/salmon_quant.log:mapping rate: 78.7291%
results/salmon_quant/S83/logs/salmon_quant.log:mapping rate: 76.1504%
results/salmon_quant/S84/logs/salmon_quant.log:mapping rate: 83.2207%
results/salmon_quant/S85/logs/salmon_quant.log:mapping rate: 76.9540%
results/salmon_quant/S86/logs/salmon_quant.log:mapping rate: 83.0532%
```

*Answer*: The sample with the lowest mapping rate: S83

------------------------------------------------------------------------

## Part D: Summarizing all steps with MultiQC

### 12: Update your MultiQC script

### 13: Run MultiQC

*Answer*: For each program, the number of reports that MultiQC found,
and why it found this number:

- MultiQC found 22 Salmon meta reports, 22 fragment length distributions,
and 22 library format counts reports because Salmon produced one report for each of the samples (total sample number is 22). It also found 44 Trim Galore reports and
44 FastQC reports because each sample had two FASTQ files (R1 and R2),
generating 22 × 2 = 44 reports.

### 14: The General Statistics table

*Answer*: The sample with the lowest Salmon mapping rate,
and whether this matches your answer to question 11: 

- The sample with the lowest Salmon mapping rate is **S83** which has **76.2%** mapping rate. 

- Yes, this matches with my answer to question 11.

### 15: Adapter lengths

*Answer*: The most common adapter lengths, whether this surprised you,
and why you think these lengths are the most common:

- The most common adapter overlap lengths were 1–3 bp (the largest peak is seen at about 1 bp). This surprised me because the trimmed matches were very much shorter than a complete adapter sequence. It might be due to the overlaps as short as one base and short matches can occur by chance at read ends which may have contributed to the large peak at short lengths.

### 16: Ask Claude about the MultiQC report

*Answer*: What you asked, and what you learned from Claude's answer (in your own words):

- I asked "Please examine results/multiqc/multiqc_report.html. Explain what the Trim Galore Adapter Length Distribution plot shows, what its axes represent, and how I should interpret the main peaks in this report. Use simple language and distinguish what the plot directly shows from possible explanations." I learned that each line represents one read file and most trimmed adapter matches were only 1 bp long. These short matches can occur by chance because Trim Galore allows a minimum overlap of one base. Claude also suggested checking the FastQC results for S65 and S74 because they had more adapter trimming than other samples.

------------------------------------------------------------------------

## Part E: Improving the workflow with genAI

### 17: Claude's review of your protocol and scripts

*Answer*: At least three of Claude's suggestions,
and whether you agree or disagree with each one, and why:

1. Claude suggested completing the protocol header and describing the input data. I agree because the cluster name, data source, file naming pattern and expected sample count help another person repeat the workflow.

2. Claude suggested explaining that variables must be reset in a new terminal session and adding checks for missing samples. I agree because variables aren't carried over to a new session and checking the output counts help to detect failed or missing samples.

3. Claude suggested making the cluster settings and thread counts easier to adjust. I agree that users of another OSC project should know how to change the account setting. However, I don't think that an explicit CPU or time request is essential for this MultiQC run as it completed successfully using the default settings and the assignment didn't require those options.

### 18: Address a suggestion

*Answer*: Which suggestion(s) you addressed, and what you changed:

- I addressed Claude’s suggestion about variables being limited to
the current terminal session. I added a note before the “Set file paths”
code block telling users to rerun the variable assignments whenever they
open a new terminal. This will help to prevent error caused by undefined input
and output paths.

------------------------------------------------------------------------

## Part F: Publish your repo on GitHub

### 19: Create a GitHub repository and push

### 20: Get a (test) DOI with Zenodo

*Answer*: Your Zenodo Sandbox DOI: 10.5072/zenodo.636777
