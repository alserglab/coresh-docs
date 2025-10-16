# Using CORESH locally

## Introduction

This vignette demonstrates the core algorithm behind CORESH search
engine for querying public gene expression datasets based on a
user-provided gene signature (<https://alserglab.wustl.edu/coresh/>).
CORESH ranks the datasets based on the level of coregulation of
user-provided genes using a score inspired by Principal Component
Analysis, which can be applied to any gene expression matrix. Currently,
CORESH operates on a compendium of more than 40,000 mouse and 40,000
human gene expression datasets from the GEO database, including datasets
from both microarray and RNA-seq profiling.

## Installing dependencies

All of the required R dependencies can be installed by installing coresh
R package:
```r
library(remotes)
install_github("alserglab/coresh")
```
## Getting the data

The preprocessed compendium of GEO datasets can be downloaded from the
[Synapse project syn66227307](https://www.synapse.org/coresh). Notice,
that to download the files you will need to register at Synapse first.

After user registration, you can install the official Python client
(<https://python-docs.synapse.org/en/stable/tutorials/installation/>).
Install the client with pip using the command line:

```bash
pip install --upgrade synapseclient
```
Then log in to the client (you can use a personal access token, which
can be created at
<https://accounts.synapse.org/authenticated/personalaccesstokens>):
```bash
synapse config
```
Finally, download the files from the project
```bash
synapse get -r syn66227307
```
If everything finished correctly, a `preprocessed_chunks` folder should
appear.
```bash
tree -n preprocessed_chunks | head
```
    ## preprocessed_chunks
    ## ├── SYNAPSE_METADATA_MANIFEST.tsv
    ## ├── hsa
    ## │   ├── SYNAPSE_METADATA_MANIFEST.tsv
    ## │   ├── chunk_10_full_objects.qs2
    ## │   ├── chunk_11_full_objects.qs2
    ## │   ├── chunk_12_full_objects.qs2
    ## │   ├── chunk_13_full_objects.qs2
    ## │   ├── chunk_14_full_objects.qs2
    ## │   ├── chunk_15_full_objects.qs2

## Running CORESH analysis locally

Now we can switch to R and take a look at the data. In this vignette we
will be working with the human data.

The data is split into around hundred chunks:
```r
chunkPaths <- list.files("./preprocessed_chunks/hsa/",
                            pattern="full_objects.qs2",
                            full.names = TRUE)
print(length(chunkPaths))
```
    ## [1] 89

Each chunk consists from around 500 objects and can be loaded using qs2
packages:
```r
library(qs2)
chunk <- qs_read(chunkPaths[[1]])
print(length(chunk))
```
    ## [1] 500

Each object corresponds to a GEO dataset. Note that `E1024` attribute
contains a processed gene expression matrix: it is centered, potentially
reduced with a Principal Component Analysis, multiplied by 1024 and
rounded to the nearest integer.
```r
str(chunk[[1]])
```
    ## List of 8
    ##  $ rownames  : int [1:10000] 9 12 14 16 18 19 20 21 22 23 ...
    ##  $ samples   : chr [1:10(1d)] "GSM15785" "GSM15786" "GSM15787" "GSM15788" ...
    ##  $ nsamples  : int 10
    ##  $ totalVar  : num 3096
    ##  $ E1024     : int [1:10000, 1:10] 28 51 120 -268 -368 -18 196 318 114 6 ...
    ##  $ gseId     : chr "GSE1000"
    ##  $ gplId     : chr "GPL96"
    ##  $ wordMatrix: num [1:15, 1:10] -0.158 -0.158 0.632 0.387 0.387 ...
    ##   ..- attr(*, "dimnames")=List of 2
    ##   .. ..$ : chr [1:15] "culture" "terminated" "sample" "acid" ...
    ##   .. ..$ : chr [1:10] "GSM15785" "GSM15786" "GSM15787" "GSM15788" ...

As a test query gene set we will use “HALLMARK\_HYPOXIA” gene set from
the MSigDB database:
```r
library(msigdbr)
hallmarks <- msigdbr(species="human", collection = "H")
query <- hallmarks |> 
    dplyr::filter(gs_name == "HALLMARK_HYPOXIA") |>
    dplyr::pull(ncbi_gene)

query <- as.integer(query) # gene IDs are stored as integers
str(query)
```
    ##  int [1:200] 57007 133 136 205 9590 226 229 230 272 51129 ...

Let’s define a function to match an object against the query gene set,
which will rely on `geseca` method from the `fgsea` package:
```r
library(fgsea)
library(data.table)
coreshMatch <- function(obj, query, calculatePvalues=FALSE) {
    E <- obj$E1024/1024
    queryIdxs <- na.omit(match(query, obj$rownames))
    k <- length(queryIdxs)
        
    curProfile <- colSums(E[queryIdxs, , drop=FALSE])
    queryVar <- sum(curProfile**2)
    
    if (calculatePvalues) {
        gesecaPval <- fgsea:::gesecaCpp(E, queryVar, k, sampleSize=21, seed=1, eps = 1e-300)[[1]]$pval
    } else {
        gesecaPval <- NA
    }
    
    res <- data.table(
        gse=obj$gseId,
        gpl=obj$gplId,
        pctVar=queryVar / k / obj$totalVar * 100,
        pval=gesecaPval,
        size=k
    )
    return(res)
}
```
Testing on the first dataset:
```r
coreshMatch(chunk[[1]], query, calculatePvalues = TRUE)
```
    ##        gse    gpl    pctVar         pval  size
    ##     <char> <char>     <num>        <num> <int>
    ## 1: GSE1000  GPL96 0.3102065 4.903416e-11   165

Before running it on the whole compendium, set up the parallel back-end
appropriate to your machine. The settings could be different from the
ones below.
```r
library(BiocParallel)
bpparam <- BiocParallel::MulticoreParam(8, progressbar = TRUE)
```
Without P-value calculation, the whole process should take 10-20
seconds, depending on your machine.

    ##          gse      gpl   pctVar   pval  size
    ##       <char>   <char>    <num> <lgcl> <int>
    ## 1: GSE131379 GPL18573 7.452995     NA   147
    ## 2: GSE198308 GPL24676 7.033059     NA   150
    ## 3:  GSE59449  GPL6244 6.911020     NA   141
    ## 4: GSE239892 GPL18573 6.878768     NA   145
    ## 5: GSE196274 GPL23227 6.804668     NA   144
    ## 6: GSE179327 GPL11154 6.496734     NA   153

Ranking by p-value usually is a bit more specific, but takes about
couple of minutes to calculate.
```r
pvalRanking <- rbindlist(bplapply(chunkPaths, function(chunkPath) {
    chunk <- qs_read(chunkPath)
    chunkRanking <- rbindlist(lapply(chunk, coreshMatch, query=query, calculatePvalue=TRUE))
    chunkRanking
}, BPPARAM = bpparam))
pvalRanking <- pvalRanking[order(pval)]
head(pvalRanking)
```
    ##          gse      gpl   pctVar          pval  size
    ##       <char>   <char>    <num>         <num> <int>
    ## 1: GSE198308 GPL24676 7.033059 8.722967e-233   150
    ## 2: GSE145224 GPL20301 5.942319 2.008495e-229   159
    ## 3: GSE153557 GPL24676 5.627913 4.722340e-229   147
    ## 4:  GSE59449  GPL6244 6.911020 4.050704e-228   141
    ## 5: GSE153398 GPL11154 4.405773 2.327456e-166   144
    ## 6: GSE153291 GPL18573 4.320505 7.290046e-126   133

The current compendium snapshot does not provide metadata, but you can
get the information from GEO:
```r
geoUrl <- sprintf("https://www.ncbi.nlm.nih.gov/geo/query/acc.cgi?acc=%s&targ=self&form=text&view=brief", pvalRanking$gse[1])
head(readLines(geoUrl), 3)
```
    ## [1] "^SERIES = GSE198308"                                                                                              
    ## [2] "!Series_title = RNA-seq of HeLa cells stably overexpressing SRSF6-GFP grown under normoxic and hypoxic conditions"
    ## [3] "!Series_geo_accession = GSE198308"

## Session info
```r
sessionInfo()
```
    ## R version 4.5.0 (2025-04-11)
    ## Platform: aarch64-apple-darwin24.2.0
    ## Running under: macOS Sequoia 15.7
    ## 
    ## Matrix products: default
    ## BLAS:   /opt/homebrew/Cellar/openblas/0.3.29/lib/libopenblasp-r0.3.29.dylib 
    ## LAPACK: /opt/homebrew/Cellar/r/4.5.0/lib/R/lib/libRlapack.dylib;  LAPACK version 3.12.1
    ## 
    ## locale:
    ## [1] en_US.UTF-8/en_US.UTF-8/en_US.UTF-8/C/en_US.UTF-8/en_US.UTF-8
    ## 
    ## time zone: America/Chicago
    ## tzcode source: internal
    ## 
    ## attached base packages:
    ## [1] stats     graphics  grDevices utils     datasets  methods   base     
    ## 
    ## other attached packages:
    ## [1] BiocParallel_1.42.0 data.table_1.17.2   fgsea_1.35.6       
    ## [4] msigdbr_25.1.1      qs2_0.1.5          
    ## 
    ## loaded via a namespace (and not attached):
    ##  [1] Matrix_1.7-3          gtable_0.3.6          babelgene_22.9       
    ##  [4] dplyr_1.1.4           compiler_4.5.0        tidyselect_1.2.1     
    ##  [7] Rcpp_1.0.14           parallel_4.5.0        assertthat_0.2.1     
    ## [10] scales_1.4.0          yaml_2.3.10           fastmap_1.2.0        
    ## [13] lattice_0.22-6        ggplot2_3.5.2         R6_2.6.1             
    ## [16] generics_0.1.4        curl_6.2.2            knitr_1.50           
    ## [19] tibble_3.2.1          stringfish_0.17.0     pillar_1.10.2        
    ## [22] RColorBrewer_1.1-3    rlang_1.1.6           fastmatch_1.1-6      
    ## [25] xfun_0.52             RcppParallel_5.1.11-1 cli_3.6.5            
    ## [28] withr_3.0.2           magrittr_2.0.3        digest_0.6.37        
    ## [31] grid_4.5.0            rstudioapi_0.17.1     cowplot_1.1.3        
    ## [34] lifecycle_1.0.4       vctrs_0.6.5           evaluate_1.0.3       
    ## [37] glue_1.8.0            farver_2.1.2          codetools_0.2-20     
    ## [40] rmarkdown_2.29        tools_4.5.0           pkgconfig_2.0.3      
    ## [43] htmltools_0.5.8.1
