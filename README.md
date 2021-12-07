# Ground truth metagenomic profiles in CAMI format for the mock virome communities in Roux et al.

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
