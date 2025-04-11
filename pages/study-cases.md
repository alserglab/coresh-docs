# Study Cases

## Gene signature induced by a pyroptosis secretome
In the first case study, we show how CORESH can be used for hypothesis generation. For this, we apply CORESH to investigate a gene signature induced in bone-marrow derived macrophages by a pyroptosis secretome, reproducing our previous results {cite}`mehrotra2024oxylipins`. More specifically, in this study we considered a system where macrophages undergo pyroptosis without releasing IL-1β/α (denoted Pyro-1) and showed beneficial effects of Pyro-1 secretome on wound healing. Interested in understanding this effect, we profiled gene expression in macrophages treated with Pyro-1 secretome. We compiled a gene signature of genes upregulated in Pyro-1 treatment and used this signature as a query for the search engine. Among the top hits, CORESH identified several datasets (e.g., GSE119509 and GSE41833) of macrophages treated with Prostaglandin E2 (PGE2), where the Pyro-1 signature was upregulated following PGE2 treatment (https://alserglab.wustl.edu/coresh/load/example_pyroptosis). This led us to hypothesize that PGE2 could mediate the effects of Pyro-1 treatment. Subsequent lipidomics analysis confirmed its enrichment in Pyro‑1 supernatants, and further experiments confirmed the PGE2 wound healing effects in vivo {cite}`mehrotra2024oxylipins`.

## Smoking-associated gene signature

In another scenario, we demonstrate how CORESH can be used for the interpretation of a gene signature, either as a complement to or in place of classical pathway analysis. To illustrate this, we use a 20-gene signature obtained from asthma patient samples by the UnPaSt biclustering method {cite}`hartung2024unpast`. In the study, the authors correlated this signature with the donors’ smoking status, which was available as a variable in their discovery cohort. Notably, when ran through commonly used gene set analysis tools like Enrichr {cite}`kuleshov2016enrichr`, this signature cannot be directly connected to smoking—the closest gene set hits are NFE2L2-related reactive oxygen stress response gene sets. Similarly, there are no smoking-related datasets at the top of RummaGEO results.

```{figure} ../images/RummaGeo-results-smoking-signature.png
:width: 720px
:align: center
:name: rummageo-screenshot

Screenshot of RummaGEO results with the smoking-associated signature used as input.
```
However, when used with CORESH, the top-ranked datasets are highly enriched in smoking-related experiments (https://alserglab.wustl.edu/coresh/load/example_smoking). This is reflected by term enrichment results, where the “smokers” is one of the top enriched terms, following “lung” and “lung cancer”. Furthermore, one can observe that the query signature is frequently correlated with smoking-related sample annotations.
```{figure} ../images/coresh-smoking.png
:width: 720px
:align: center
:name: coresh-smoking-screenshot

Screenshot of the detailed information for dataset GSE10006, one of the top datasets found for the smoking-associated signature.
```


