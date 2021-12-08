# Ground truth metagenomic profiles in CAMI format for the mock virome communities in Roux et al.

Similar projects: https://github.com/shenwei356/sun2021-cami-profiles

## Datasets

> Here we designed mock viral communities including both ssDNA and dsDNA viruses to evaluate
the capability of a sequencing library preparation approach including an Adaptase step prior
to Linker Amplification for quantitative amplification of both dsDNA and ssDNA templates.

> Two mock communities were designed (A and B) corresponding to two contrasting
> situations with either low abundance of ssDNA viruses (MCA, total ssDNA ∼2% of 
> community) or high abundance of ssDNA viruses (MCB, total ssDNA ∼66% of community)

> Roux S, Solonenko NE, Dang VT, Poulos BT, Schwenck SM, Goldsmith DB, Coleman ML, Breitbart M, Sullivan MB. 2016. Towards quantitative viromics for both double-stranded and single-stranded DNA viruses. PeerJ 4:e2777 https://doi.org/10.7717/peerj.2777

Phages (rank is based on NCBI taxonomy 2021-12-06)

|phage      |taxid  |name                       |rank   |
|:----------|:------|:--------------------------|:------|
|PSA-HM1    |1357705|Pseudoalteromonas phage HM1|species|
|PSA-HP1    |1357706|Pseudoalteromonas phage HP1|no rank|
|PSA-HS1    |1357707|Pseudoalteromonas phage HS1|species|
|PSA-HS2    |1357708|Pseudoalteromonas phage HS2|species|
|PSA-HS6    |1357710|Pseudoalteromonas phage HS6|species|
|Cba phi38:1|1327977|Cellulophaga phage phi38:1 |species|
|Cba phi18:3|1327983|Cellulophaga phage phi18:3 |species|
|Cba phi38:2|1327999|Cellulophaga phage phi38:2 |species|
|Cba phi13:1|1327992|Cellulophaga phage phi13:1 |no rank|
|Cba phi18:1|1327982|Cellulophaga phage phi18:1 |no rank|
|phix174    |2886930|Escherichia phage phiX174  |no rank|
|alpha3     |10849  |Escherichia phage alpha3   |no rank|

The abundance of phages in every sample is listed in [Table S2](https://dfzljdn9uc3pi.cloudfront.net/2016/2777/1/ssDNA_dsDNA_viromes_2.0_Supplementary_Tables.xls).
We converted the abundance table to CAMI format, available at: https://github.com/shenwei356/roux2016-mock-virome-cami-profile

Mock communities A and B are available at: http://datacommons.cyverse.org/browse/iplant/home/shared/iVirus/DNA_Viromes_library_comparison.

> the 3 different protocols are indicated with a one-letter code: "S" for A-LA, "N" for TAG, and "G" for MDA.
For mock communities, replicates are indicated with a number next to the library code.

There are 16 samples in total:

    MCA-G1
    MCA-G2
    MCA-N1
    MCA-N2
    MCA-S1
    MCA-S2
    MCA-S3
    MCB-G1
    MCB-G2
    MCB-G3
    MCB-N1
    MCB-N2
    MCB-N3
    MCB-S1
    MCB-S2
    MCB-S3
    
But we found some name inconsistencies between the abundance table and FASTQ files.
After carefully checking, we renamed samples as below:

    old      new
    MCA-G1   MCA-G2
    MCA-G2   MCA-G3
    MCA-N2   MCA-N3

We manually download the paired and unpaired reads for every sample, for example:

    $ ls reads/ | head -n 4
    MCA-G2_GGACTCCT-GCGTAAGA_L002_R1_001_t_paired.fastq.gz
    MCA-G2_GGACTCCT-GCGTAAGA_L002_R1_001_t_unpaired.fastq.gz
    MCA-G2_GGACTCCT-GCGTAAGA_L002_R2_001_t_paired.fastq.gz
    MCA-G2_GGACTCCT-GCGTAAGA_L002_R2_001_t_unpaired.fastq.gz


## Taxonomy

NCBI Taxonomy version: 2021-12-06

## HOWTO

    csvtk xlsx2csv gs.xlsx \
        | csvtk rename2 -F -f MC* -p '(MC.) (.+) (\d)' -k method.map --key-capt-idx 2 -r '$1-{kv}$3' \
        | csvtk csv2tab \
        > gs.tsv
    
    mkdir -p roux2016
    csvtk headers -t gs.tsv | awk 'NR > 3' \
        | rush 'csvtk cut -t -f taxid,{} gs.tsv | sed 1d > roux2016/{}.tsv'
        
 
    taxver=ncbi-taxonomy-2021-12-06
    taxdump=taxdump
    
    rm roux2016/*.profile
    ls roux2016/* \
        | rush -v taxver=$taxver -v taxdump=$taxdump \
            'taxonkit profile2cami --data-dir {taxdump} -s {%:} -t {taxver} {} -o {}.profile'

    cat roux2016/*.profile > roux2016_gs.profile


    # names in current taxonomy
    csvtk cut -t -f 1-2 gs.tsv \
        | taxonkit lineage -nrL -i 2 --data-dir taxdump/ \
        | csvtk rename -t -f 3,4 -n name,rank \
        > names.tsv
