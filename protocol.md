# RNA-seq analysis of _Culex pipiens_

- Author: Md. Rafiqul Islam
- Date created: 2026-11-07
- Date last modified: 2026-11-08
- This was run at the <???> cluster of the Ohio Supercomputer Center (<https://www.osc.edu>)
- OSC working dir: /fs/ess/PAS3493/people/islam300/GA5
- GitHub repository: <???>

## Prerequisites

- **Input files**:
  - Paired-end RNA-seq reads are in `../garrigos/fastq`

- **Software**:
  - The scripts run programs using Apptainer containers,
    with links contained inside the scripts.
  - If running at OSC, no software installations are needed.
  - If you're not at OSC, you need to have Apptainer and Slurm installed.

- Run all code from the working dir listed above.
  You don't need to create the output dirs, because the scripts will do so.

- Each script writes a Slurm log file (`slurm-<script>-<jobID>.out`) to the working dir.
  After the jobs of a step have finished, check these logs for errors,
  then move them into a `logs/` dir within that step's output dir.

## Set up: Download the reference transcriptome FASTA file from NCBI

```bash
# Downloaded on 2026-11-07

wget -P data/ref https://ftp.ncbi.nlm.nih.gov/genomes/all/GCF/016/801/865/GCF_016801865.2_TS_CPP_V2/GCF_016801865.2_TS_CPP_V2_rna.fna.gz
```

## Set up: Set file paths

Run the following variable assignments at the start of every new terminal
session before continuing the workflow. These variables are available only
in the terminal session where they were defined.

```bash
# Inputs:
fastq_dir=../garrigos/fastq
ref_transcripts=data/ref/GCF_016801865.2_TS_CPP_V2_rna.fna.gz

# Base output dir:
outdir=results
```

## Step 1: Read trimming with TrimGalore

```bash
for fastq_r1 in "$fastq_dir"/*_R1.fastq.gz; do
    fastq_r2=${fastq_r1/_R1/_R2}
    sbatch scripts/trimgalore.sh "$fastq_r1" "$fastq_r2" "$outdir/trimgalore"
done
```

## Step 2: Creating a transcriptome index with Salmon

Create one Salmon index from the reference transcriptome:

```bash
sbatch scripts/salmon_index.sh "$ref_transcripts" "$outdir/salmon_index"
```

Monitor the job:

```bash
squeue -u islam300 -l
```

After the job finishes, check its Slurm log, output files, and index size:

```bash
cat slurm-salmon_index-*.out
ls -lh "$outdir/salmon_index"
du -sh "$outdir/salmon_index"
```

After confirming success, move the Slurm log:

```bash
mv -v slurm-salmon_index-*.out "$outdir/salmon_index/logs/"
```

## Step 3: Pseudo-alignment and quantification with Salmon

After the index and all trimming jobs have completed successfully, run Salmon
once per sample using the trimmed paired-end reads.

The script takes the both trimmed R1 and R2 files, Salmon index directory
and quantification output directory as arguments.

```bash
for fastq_r1 in "$outdir"/trimgalore/*_R1.fastq.gz; do
    fastq_r2=${fastq_r1/_R1/_R2}
    sbatch scripts/salmon_quant.sh "$fastq_r1" "$fastq_r2" "$outdir/salmon_index" "$outdir/salmon_quant"
done
```

Monitor the jobs:

```bash
squeue -u islam300 -l
```

After all quantification jobs finish, check their Slurm logs and output files:

```bash
grep -c "Successfully finished script salmon_quant.sh" slurm-salmon_quant-*.out
```

```bash
less slurm-salmon_quant-*.out
```

```bash
ls -lh "$outdir"/salmon_quant/*/quant.sf
```

```bash
ls "$outdir"/salmon_quant/*/quant.sf | wc -l
```

After confirming that all jobs succeeded, move their Slurm logs:

```bash
mv -v slurm-salmon_quant-*.out "$outdir/salmon_quant/logs/"
```

## Step 4: Summarizing the results of all steps with MultiQC

After trimming and Salmon quantification finished successfully,
run MultiQC on the entire results directory.

```bash
sbatch scripts/multiqc.sh results results/multiqc
```

Monitor the job:

```bash
squeue -u islam300 -l
```

After the job finishes, check its Slurm log and output files:

```bash
cat slurm-multiqc-*.out
ls -lh results/multiqc
```

Confirm that the log contains "Successfully finished script multiqc.sh"
and that multiqc_report.html was produced.

After confirming success, move the Slurm log:

```bash
mv -v slurm-multiqc-*.out results/multiqc/logs/
```