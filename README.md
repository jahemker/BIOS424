# BIOS424
This is a repo that will house the scripts we use in BIOS424 (Structural Variation Minicourse).

The goals of the labs for this class are to walk through generation of long-read sequencing data, calling of structural variants, and analysis of structural variants.

## 2026

As we only have one 50 minute class period for the computing lab this year, we are stream-lining things. Please feel free to look at the rest of the stuff in this repository from the 2025 version of the class, which has a lot of details on running alignment, assembly, and SV programs.

### Visualizing SVs with Jbrowse2

We will focus on looking at nanopore read alignments and validating the presence of SVs within these alignments. To get started, please download the following files:

Jbrowse2: https://jbrowse.org/jb2/download/

D. melanogaster data files (974Mb): https://drive.google.com/file/d/1r7EfdqJcuKUaW00_DpG33K98dOykPWEl/view?usp=sharing

- `D.melanogaster.filtered.fa` - Reference genome
- `D.melanogaster.filtered.fa.fai` - Reference genome index file
- `dmel12.2R.bam` - Alignment file of chromosome 2R for sample dmel12 to the reference
- `dmel12.2R.bam.bai` - Alignment index file


#### Loading alignments into Jbrowse2
1) OPEN SEQUENCE FILES
2) Set assembly name to dm6; Type - IndexedFastaAdapter
3) Set file location to the downloaded reference genome and index file and hit submit
4) Launch linear genome view and select chromosome 2R. It should say `No tracks active.`
5) FILE -> Open Track
6) Enter track data - set file location to the downloaded alignment file and its index and hit next
7) The following information should auto-fill: trackName - dmel12.2R.bam; Adapter type - BAM adapter; Track type - Alignments track; Assembly - dm6; Click add.
9) Zoom into a small chunk of the chromosome (such as the first SV's coordinates) to see individual read alignments.

#### Looking for SVs
Here are 7 putative structural variant calls in this sample. See if you can determine which are real and which are fake. Even better, can you come up with a reason why the fake ones were labelled as SVs by the callers?

Enter the coordinates into the search bar to be taken to that locus. You should be able to copy and paste the chrom,start,end all in one go.
```
CHR START       END         SVTYPE  SVLEN
2R  9950744	    9950745	    INS	    1734
2R	10425533	10426811	INV	    1277
2R	10659106	10666658	DEL	    -7551
2R	12185375	12185376	INS	    12005
2R	12189218	12189219	INS	    22848
2R	15374795	15374796	INS	    440
2R	18358433	18371955	DEL	    -13521
```
#### Navigating Jbrowse2
- In the upper left of the alignment track, click the 3 dots next to dmel12.2R.bam and go to Pileup settings -> Color scheme -> Mapping quality. Yellow is HQ mapping, Red is LQ mapping.
- Click on an individual read to see information about it.
- Right click on a read and select `Linear read vs ref` to create a new linear alignment track between that single read and the reference. Note insertions (yellow) and deletions (blue) between the alignments.
- You can move your cursor under the base of a yellow triangle (insertion in the read) until a red bar appears. Click and drag across the length of the insertion. Click Get Sequence and copy it to the clipboard. Use BLAT (https://genome.ucsc.edu/cgi-bin/hgBlat?hgsid=3766760243_kURxc9XlyEO0tFv1XA9KwgDg2Rcu&command=start) or BLAST (https://blast.ncbi.nlm.nih.gov/Blast.cgi) to determine what the inserted sequence is.
- You can do the same thing above the blue triangles to determine the reference sequence deleted in the read.

---

### Computing in BIOS424
In order to keep everything consistent for the computational parts of the labs, we are going to assume everyone will be working on the Sherlock computing cluster.

If you do not have access to Sherlock please let us know and we can try to come up with a workaround.

I've tried to clearly write out all of the commands we will be using for anyone who is not very comfortable with bash or Sherlock.

Additionally, we will be using a conda environment to conveniently hold all of the various programs (and their dependencies) that we will be using. If you do not have conda (and mamba) installed on Sherlock, please do so before class if possible.

Mamba will allow you to download all the packages much faster than conda does.

If you feel competent and wish to install these programs in a different way, we've listed everything out that we are planning on using, so feel free to do so.

---

You can install both conda and mamba through miniforge from here: https://github.com/conda-forge/miniforge

Conda environments with a bunch of packages can take up a fair amount of space. I would recommend installing conda into a personal directory in your `$GROUP_HOME` if possible.

Here are the basic installation commands for linux (Sherlock). See the link for more info:
```
wget -O Miniforge3.sh "https://github.com/conda-forge/miniforge/releases/latest/download/Miniforge3-$(uname)-$(uname -m).sh"
bash Miniforge3.sh -b -p "${GROUP_HOME}/[personal_dir]/conda" #change [personal_dir] to whatever you use
```
You'll then want to add `conda` and `mamba` to your `$PATH` so that you can run them from wherever. Paste the following into `~/.bash_profile`
```
# add conda to path. Change [personal_dir] to whatever you used on the installation.
if [ -d "${GROUP_HOME}/[personal_dir]/conda" ] ; then
    PATH="${GROUP_HOME}/[personal_dir]/conda/condabin:$PATH"
fi
```
Either `source ~/.bash_profile` or restart your terminal window. Typing `conda` and `mamba` should now give help messages.

Now, we want to all have the same conda environment with all the necessary programs. First, clone this github repo somewhere you can easily access on Sherlock. (I would recommend `$SCRATCH`; it's what all the scripts will assume). Move into the directory and we will now create a new conda environment from the supplied `environment.yml` file. It will probably take a little while (5-10 minutes with mamba) to download everything that's needed. Whenever it finishes, activate the new environment.
```
cd $SCRATCH
git clone https://github.com/jahemker/BIOS424/
cd BIOS424/
mamba env create -n BIOS424 -f environment.yml
conda activate BIOS424
```

## Lab 1 - Long-read sequencing with Oxford Nanopore Technologies (ONT)
In this first lab we will explore an ONT long-read sequencing protocol for Drosophila melanogaster.
We will walk through the each step of the protocol, then we will practice loading nanopore flow cells with a dummy library.
We will look at ONT's sequencing software, Minknow, to see how we can evaluate our sequencing runs.
We will finally end with a foray into computational work, basecalling our sequencing data.

- Input: Sample

- Output: Basecalled reads data from sequencing run.

The protocol we will be following is based off of:

https://elifesciences.org/articles/66405

https://nanoporetech.com/document/genomic-dna-by-ligation-sqk-lsk114?device=PromethION

For basecalling, we will be using Dorado, which is Nanopore's open-source basecaller.

https://github.com/nanoporetech/dorado

## Lab 2 - Genome assembly and long-read + assembly alignment to reference; Structural variant calling
The second lab class will focus on computationally generating our long reads, assembling them into full genomes, aligning to a reference genome, and calling SVs.
We will perform all of our computational work on Sherlock, Stanford's high-performance computing cluster for bioscience labs. In reality, the computational steps can take days to run, so we have already generated all starting, intermediate, and final files.
In lab we will work on learning how to run these programs on Sherlock.

- Input: Basecalled data from sequencing run. Reference genome of species.

- Output: Base-called reads (FASTQ files), genome assemblies (FASTA files), alignments (BAM files), structural variant calls (VCFs)

For detailed steps, refer to the readme in the `lab_two/` folder.

## Lab 3 - Structural variant QC and analysis
In this final lab, we will be looking at our SV calls that we generated in the previous lab. We will manually verify SVs looking at read alignments. We will perform some basic analyses with our VCFs.

- Input: structural variant calls (VCFs), alignment files (BAM files)
- Output: Various plots/analyses

- Programs:
  - JBrowse2 and/or IGV for looking at alignments
  - R?
