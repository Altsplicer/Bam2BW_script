# [Franco](https://github.com/altsplicer) / [***STAR alignment***]

Script [LINK](https://github.com/Altsplicer/STAR_alignment_script/blob/main/bash/STAR_align.sub)

[![.img/fig1.png](.img/fig1.png)](#nolink)

## Overview

This a script used to create a bigwig file from a bam file to view reads in IGV viewer or create profile plot and heatmaps.

For a tutorial see [link](https://hbctraining.github.io/In-depth-NGS-Data-Analysis-Course/sessionV/lessons/10_data_visualization.html)

## Slurm script headers
UCI uses the SLURM scheduler so you must use slurm headers to specify how and where you want the job to run. 
You must also name the script SCRIPT_NAME.sub and then run the script using "Sbatch SCRIPT_NAME.sub", while you are in the working directory of the script location. 

For a more in depth view on a SLURM job script headers see https://rcic.uci.edu/slurm/examples.html.

Make note that this script uses the free partition but you can use your free 1000 core hours of the lab's core hours by changing the header.
Otherwise your job can be killed in the free partition.
``` bash
#!/bin/bash
#SBATCH --job-name=Bam2BW	## Name of the job.
#SBATCH -p free			## partition/queue name
#SBATCH --nodes=1			##(-N) number of nodes to use
#SBATCH --mem=36G ## request 36GB of memory
#SBATCH -e /dfs3b/hertel-lab/fcarranz/project_name/alignments/FC_Bam2BW.err 	##Error_log
#SBATCH -o /dfs3b/hertel-lab/fcarranz/project_name/alignments/FC_Bam2BW.out	##outputfile
#SBATCH --mail-user fcarranz@uci.edu
#SBATCH --mail-type=ALL
```

## Load required packages
``` bash
module load samtools/1.10
module load deeptools/3.5.1
``` 

## Indexing Bam files
You have to index your bam files first using samtools.
``` bash
#location of your bamfiles. Note in this example it is assumed you moved them from the Staraligner location to this BAM_files folder
DATA_DIR=/dfs3b/hertel-lab/fcarranz/project_name/BAM_files
```

``` bash
# Iterate over each sample
for SAMPLE in HBR;
do
    # Iterate over each of the replicates.
    for REPLICATE in 1 2 3 ;
    do
        # Build the name of the files.
        R1=${DATA_DIR}/${SAMPLE}_${REPLICATE}.starAligned.sortedByCoord.out.bam

        # Run samtools.
		echo "Index ${R1} "

		samtools \
		index \
		-b ${R1}
		-o ${DATA_DIR}/${SAMPLE}_${REPLICATE}.starAligned.sortedByCoord.out.bam.bai
		
		#-b the bam file location
		#-o where you want the index file to be outputed
		# For a single file it would be 
		#samtools index $FILE

		echo "index of ${R1} complete"
    done
done
```

## Bam to BigWig conversion

The .bai file must be in the same location as it's corresponding bam file.

``` bash
echo "You're converting Bam2BW on $HOSTNAME"

#where the bam files are located
DATA_DIR=/dfs3b/hertel-lab/fcarranz/project_name/BAM_files


#desired directory for output alignments
OUT_DIR=/dfs3b/hertel-lab/fcarranz/project_name/BigWigs
``` bash

## deeptools execution 
``` bash
# Iterate over each sample
for SAMPLE in HBR;
do
    # Iterate over each of the replicates.
    for REPLICATE in 1 2 3 ;
    do
        # Build the name of the files.
         R1=${DATA_DIR}/${SAMPLE}_${REPLICATE}.starAligned.sortedByCoord.out.bam

        # Run the bamCoverage.
		echo "Convert ${R1} "


		bamCoverage \
		-b ${R1} \
		-o ${DATA_DIR}/${SAMPLE}_${REPLICATE}_SeqDepthNormb1.bw \
		--normalizeUsing RPKM \
		--ignoreForNormalization chrX \
		--numberOfProcessors max \
		--binSize 1
		

		echo "conversion of ${R1} complete"
    done
done

exit
```
-b bam file location

-o bamCoverage output and output file name

RPKM = Reads Per Kilobase per Million mapped reads

--numberOfProcessors max allows to maximize our resources and shorten the conversion process. May take a while.

--binSize Size of the bins, in bases, for the output of the bigwig/bedgraph file. (Default: 50)

See BigWig [Documentation](https://deeptools.readthedocs.io/en/latest/) for more info

# Please note if any of these commands are not sent in as slurm job but done in the terminal then you need to be in a interactive node, NOT THE LOGIN NODE! You will get in trouble with UCI's HPC staff. 
