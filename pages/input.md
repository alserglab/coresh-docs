# Search Input

The **Search Input** section allows users to define their query parameters and input a gene list for analysis.

## Input Interface Overview

The input interface consists of the following components:
```{image} ../images/input.png
:width: 720px
:align: center
```

### Query Species

The **Query Species** radio button lets you select the organism corresponding to your gene list. Options include:

- **Homo sapiens**: Select this if your genes are from humans.
- **Mus musculus**: Select this if your genes are from mice.

### Database Species

The **Database Species** option specifies the organism's database that will be queried for coregulation analysis. Options include:

- **Homo sapiens**: Select this option to search within human experimental data available in the Gene Expression Omnibus (GEO) database. This specifies that only datasets from human experiments will be used for the coregulation search.

- **Mus musculus**: Select this option to search within mouse experimental data available in the Gene Expression Omnibus (GEO) database. This ensures that the search is limited to datasets derived from mouse experiments. 

These options control the scope of the query by defining the species-specific experiments in the GEO database that will be used to identify relevant gene coregulation patterns.

### Gene List

The Gene List field allows you to input genes or identifiers for your query. Our search supports a wide range of gene types and IDs as defined in the g:Profiler {cite}`gprof` (g:Convert) tool. This includes converting between various namespaces for genes, proteins, microarray probes, and more.

Supported formats include, but are not limited to:

- Gene Symbols, Ensembl IDs, RefSeq IDs, Gene Entrez IDs, and many others.

**Input format:** Enter each gene or identifier on a new line, or separate them with spaces or tabs.


### Calculate p-values

By default, the search engine ranks all datasets based on the percentage of variation explained by the direction in the sample space along the query gene signature. Additionally, an option to assess statistical significance is available, which can improve ranking, but requires more time for calculation.


### Search Button

After filling in the form, clicking the **Search** button starts the analysis. The input data is processed, and the results are presented in the **Search Results** section.



### Example Feature

The tool includes an "Example" link that populates the query form with a predefined gene list and settings. This feature provides a quick and convenient way to test the tool's functionality or explore its features.

Example configuration:

- **Query Species**: Homo sapiens
- **Database Species**: Homo sapiens

Gene list from the example:

```
ACKR3 ADM ADORA2B AK4 AKAP12 ALDOA ALDOB ALDOC AMPD3 ANGPTL4 ANKZF1 ANXA2 ATF3 ATP7A B3GALT6 B4GALNT2 BCAN BCL2 BGN BHLHE40 BNIP3L BRS3 BTG1 CA12 CASP6 CAV1 CAVIN1 CAVIN3 CCN1 CCN2 CCN5 CCNG2 CDKN1A CDKN1B CDKN1C CHST2 CHST3 CITED2 COL5A1 CP CSRP2 CXCR4 DCN DDIT3 DDIT4 DPYSL4 DTNA DUSP1 EDN2 EFNA1 EFNA3 EGFR ENO1 ENO2 ENO3 ERO1A ERRFI1 ETS1 EXT1 F3 FAM162A FBP1 FOS FOSL2 FOXO3 GAA GALK1 GAPDH GAPDHS GBE1 GCK GCNT2 GLRX GPC1 GPC3 GPC4 GPI GRHPR GYS1 HAS1 HDLBP HEXA HK1 HK2 HMOX1 HOXB9 HS3ST1 HSPA5 IDS IER3 IGFBP1 IGFBP3 IL6 ILVBL INHA IRS2 ISG20 JMJD6 JUN KDELR3 KDM3A KIF5A KLF6 KLF7 KLHL24 LALBA LARGE1 LDHA LDHC LOX LXN MAFF MAP3K1 MIF MT1E MT2A MXI1 MYH9 NAGK NCAN NDRG1 NDST1 NDST2 NEDD4L NFIL3 NOCT NR3C1 P4HA1 P4HA2 PAM PCK1 PDGFB PDK1 PDK3 PFKFB3 PFKL PFKP PGAM2 PGF PGK1 PGM1 PGM2 PHKG1 PIM1 PKLR PKP1 PLAC8 PLAUR PLIN2 PNRC1 PPARGC1A PPFIA4 PPP1R15A PPP1R3C PRDX5 PRKCA PYGM RBPJ RORA RRAGD S100A4 SAP30 SCARB1 SDC2 SDC3 SDC4 SELENBP1 SERPINE1 SIAH2 SLC25A1 SLC2A1 SLC2A3 SLC2A5 SLC37A4 SLC6A6 SRPX STBD1 STC1 STC2 SULT2B1 TES TGFB3 TGFBI TGM2 TIPARP TKTL1 TMEM45A TNFAIP3 TPBG TPD52 TPI1 TPST2 UGP2 VEGFA VHL VLDLR WSB1 XPNPEP1 ZFP36 ZNF292
```

This example list contains genes from the **Human Gene Set: HALLMARK_HYPOXIA** {cite}`liberzon2011molecular`, which are associated with biological processes such as the cellular response to low oxygen levels (hypoxia). This example is useful for identifying gene expression patterns activated under hypoxic conditions.

