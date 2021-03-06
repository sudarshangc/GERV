
Scripts for scoring variants using the raw output from the kmer-model

## Prerequisites

+ Follow instruction [here](http://stackoverflow.com/questions/8659382/downloading-an-entire-s3-bucket) to download the finished k-mer model run from Amazon S3 storage to your local file system.


+ Prepare the variants to score in VCF format, and partition by chromosome number. For instance all the variants on chromosome 1 should be stored in a file named `chr1.vcf`

## Quick example

```
docker pull haoyangz/gerv
docker run --rm -w /scripts/ -u $(id -u) -v /topfolder:/topfolder -i haoyangz/gerv /scripts/run.r \
	full_path_of_param_file  your_order
```

* `topfolder` : The common topfolder under which you store **ALL** the relevant data and want to output the result. For Gifford lab, it is usually `/cluster`
* `full_path_of_param_file `: path to param file (see below)
* `your_order`: what you wish to do (see below)

## Prepare a param file
This file specifies the parameters related to the variant scoring part of the model. The launcher parses from top to bottom, setting each variable name to value. The actual name of this file is not important, as long as the full path is provided as input.

**The only valid delimiter for this file is 'space' not tab or comma or anything else**. This also means you cannot use directories or parameters that have spaces in them.

Example: (see [here](https://github.com/gifford-lab/GERV/blob/master/variant-scoring/examples/params.list))

```
#model_parentdir /cluster/zeng/research/kmer/s3/tf-chipseq/
#expt_name Mayers_UGF1_HepG2_win50.ccm
#maxchr 22
#SGE 0
#outputdir /cluster/zeng/code/research/wave/wave-variant-scoring/examples/test/
#genomefile /cluster/projects/wordfinder/data/genome/hg19.in
#VT 1
#VT_prefix VT=
#VT_suffix SNP
#snp_dir /cluster/zeng/research/kmer/hg19/FINAL.ver140827/SNP_data/USF1/BCRANK_AF001_1KG_1000flank_AF001_1KG/
#snp_name BCRANK_1000flank_AF001_1KG
```

+ `model_parentdir`: the full path to the parent directory of the raw output of k-mer model

+ `expt_name`: the folder name of the raw output of k-mer model. So the full path of the k-mer model output is supposed to be $model_parentdir$/$expt_name$/ . GERV will automatically parse the input.list file under that directory to get the parameters for that k-mer model.

+ `maxchr`: the number of chromosomes in the genome of interest. For homo sapiens (human), it is 22.

+ `SGE`: currently we don't support running GERV on SGE so please keep it as 0

+ `genomefile`: full path of binary file of organism genome sequence. The genome file for hg19 and mm10 can be downloaded from the [GERV website](http://gerv.csail.mit.edu)

+ `snp_dir`: the full path of the directory containing SNP information in VCF format . 

+ `snp_name`: the name you wish to refer to the variant set scored. It will be concatenated to the output filenames to be informative and distinguishable

+ `VT`: boolean indicator (1 or 0) of whether there is information about the variant type (SNP, Indel, etc) in the "INFO" column of the VCF files. The common format would be "VT=SNP" (1000 genome project) but it may vary depending on the VCF. If it is set to 1, `VT_prefix` and `VT_suffix` below will be used. Otherwise they won't be used but also needed for integrity.

+ `VT_prefix`: the prefix part of the variant type information (in the case of 1000 genome project VCF, it would be "VT=")

+ `VT_suffix`: the essential part of the variant type information (in the case of 1000 genome project VCF, it would be "SNP" or "INDEL"). Currently, only SNP scoring is supported so please keep it as 'SNP'

+ `outputdir`: full path to where you wish to output the scoring results




### `Specify your order`
Three choices of `your_order` are available, each of which corresponds to one of the following three steps (in order):

+	`preprocess`

	It will first analyze the raw results from k-mer model to find the best parameter set fitted. The k-mer spatial effect profiles are output to  $model_parentdir$/$expt_name$/$expt_name$.kmer.mat. The background rate is output to $model_parentdir$/$expt_name$/$expt_name$.bin. Then it generated the predicted ChIP-seq signal for the whole reference genome and output it as per-chromosome binary files under $model_parentdir$/$expt_name$/baseline/ 
	
	This only needs to be done once for each k-mer model

+	`score NUM`

	Score the variants for chromsome number specificied by `NUM`, which should be stored in $snp_dir$/chr$NUM$.vcf. For human data, `NUM` can be from 1 to 22. The scoring result will be saved to $outputdir$/snp-ranking-raw-result/chr$NUM$.$expt_name$.$snp_name$. Another log file named $outputdir$/snp-ranking-raw-result/scoring_log_chr$NUM$ will also be generated.
	
+	`combine.result` (optional)

	Combine the scoring result for each chromosome into one compressed MATLAB MAT file $outputdir$/snp-ranking-raw-result/allChr.$expt_name$.$snp_name$.mat. Another MAT file $outputdir$/snp-ranking-raw-result/allChr.$expt_name$.$snp_name$.size.mat stores the number of variants scored on each chromosome (from 1 to $maxchr$)
	
	## Full example
A complete example bash code to perform preprocessing, scoring for all 22 chromosome for human variant data, and combining the results can be find [here](https://github.com/gifford-lab/GERV/blob/master/variant-scoring/run_model.sh).
