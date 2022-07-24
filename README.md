
<!-- README.md is generated from README.Rmd. Please edit that file -->

# scModules

<img align="right" width="108" height="125" src="man/figures/scModules1.png">

<!-- badges: start -->
<!-- badges: end -->

**scModules** is an R package for advanced processing and visualization
of single-cell RNA-seq and TCR-seq data. It integrates multiple genomic
information (e.g., copy number aberration (CNA)) and statistical
algorithms (e.g., non-negative matrix factorization (NMF)) to provide
semi-automated end to end analyses that are NOT available in the popular
single-cell package **Seurat**. Currently, four modules are developed.
The methodologies have been tested in several high-impact publications
including Cell and Science. scModules enhances our ability to extract
accurate genomic and cellular information from single-cell data and
discover new cancer biology.

-   [An Integrative Model of Cellular States, Plasticity, and Genetics
    for Glioblastoma (Neftel, Laffy et al., 2019,
    *Cell*)](https://doi.org/10.1016/j.cell.2019.06.024)
-   [Developmental and oncogenic programs in H3K27M gliomas dissected by
    single-cell RNA-seq (Filbin, Tirosh et al., 2018,
    *Science*)](https://doi.org/10.1126/science.aao4750)
-   [Decoupling genetics, lineages, and microenvironment in IDH-mutant
    gliomas by single-cell RNA-seq (Venteicher, Tirosh et al., 2017,
    *Science*)](https://doi.org/10.1126/science.aai8478)

## Installation

You can install the development version of scModules from
[GitHub](https://github.com/) with:

``` r
if (!require(devtools)) {
  install.packages("devtools")
}

devtools::install_github("qizongtai/scModules", ref = "development")
library(scModules)
```

## Modules Summary

-   `Module1: Cell type annotation (Marker genes based)`

-   `Module2: Malignant cell identification by copy number aberration`

-   `Module3: Metaprogram (cell state) identification by NMF`

-   `Module4: Immune (TCR/BCR) repertoire analysis`

### Module1

![1_scRNA_celltype_workflow](https://user-images.githubusercontent.com/33009124/177924123-b77d89d4-fc91-4673-8ca3-1823942e7d36.PNG)
infercna aims to provide functions for inferring CNA values from scRNA-seq data and related queries. 

#### Compuatation functions (essensital ones)
-   `read_mtx()` imports the Sparse Matrix outputs from Cellranger and convert it into a regular Matrix. 
-   `create_metadata()` creates a matrix to store the meta data (features) for the cells. 
-   `add_metadata()` adds new metadata to the existing metadata. It also checks the cell IDs when merging; If cell IDs doesn’t match between original and added metadata, users will have an option to proceed or stop the `add_metadata()` function.
-   `geneset_percent()` calculates the percentage of counts originating from a set of features.
-   `check_mtx_meta()` checks if cell IDs in gene-cell matrix and metadata are matched in order. It returns a logical value.
-   `filter_cell_bymeta()` filter cells in the gene-cell matrix by the feature settings from metadata. It returns both the filtered gene-cell matrix and metadata. It checks cells IDs before filtering and requires same number of cells.
-   `match_mtx_meta()` check if cell IDs in gene-cell matrix and metadata are matched in order. It also checks the cell IDs when merging; If cell IDs doesn’t match between original and added metadata, users will have an option to proceed or stop the match_mtx_meta function. It returns a matched gene-cell matrix. 
-   `filter1_cell()` filters cells in the gene-cell matrix by user-specified settings. It returns a cell-filtered mtx.count. 
-   `filter2_gene()` filters genes in the gene-cell matrix by user-specified settings. Note, it returns a gene-filtered mtx.cpm. 
-   `detect_db()` detects doublets using four well-developed method: three methods (scdDblFinder, hybrid score from scds, doubletCells algorithm from scran) are based on single cell experiment objects and one method (DoubleFinder) is based on Seurat object. Doublets were identified by combining the results of these alternative methods. For each method, we set the expected doublet rate at 0.6%, per 500 cells per sample. Cells classified as doublets by all three single cell expereiemnt based methods or all four methods. (Micheal: at least two methods if the three) 
-   `log2cpm()` transforms the the cpm values into log2 space. Formular is log2(cpm/scale +1). By default, the scale is 10 for single cell RNA-seq data. Need to change scale to 1 if bulk RNAseq data.
-   `norm_row()` row-wise(gene-wise) z-score normalization for the gene-cell matrix. Center opitons are mean and median. There is no scaling by default. 
-   `geneset_score()` calculates the signature score for the given geneset. For each cell, a relative expression score was defined by subtracting the average expression of the gene signature in a cell by that of a control gene set. The control gene set was defined by dividing all analyzed genes into 30 bins by average expression level, and for each gene in the gene signature randomly sampling 100 genes from the same bin. highest cell signature score less than (1 + conf_int)* the second highest signature.
-   `type_score()` assigns cell type to individual cells by the highest signature score of a cell type. Cell will be classified as unresolved if highest cell signature score less than (1 + conf_int)* the second highest signature.
-   `hvg_generow()` identifies the highly variable genes based on stand deviation. The number of gene is defined by <top> argument.
-   `run_pca()` runs a PCA dimensionality reduction on a gene-cell matrix.
-   `extract_pc()` extracts the first 2 (by default) principal components (PCs) from the PCA object returned by run_pca function. Number of PCs extracted can be change by npc. 
-    `run_umap()` runs the Uniform Manifold Approximation and Projection (UMAP) dimensional reduction on a gene-cell matrix. It returns an umap obj = matrix: cells x umap.2dimensions. As n_neighbors increases, UMAP connects more and more neighboring points when constructing the graph representation of the high-dimensional data, which leads to a projection that more accurately reflects the global structure of the data. At very low values, any notion of global structure is almost completely lost. As the min_dist parameter increases, UMAP tends to "spread out" the projected points, leading to decreased clustering of the data and less emphasis on global structure. 
-    `knn_cluster()` computes the k.param nearest neighbors for a given feature-cell matrix or cell-feature matrix (set transposed =T in this case). It also computes the weight for edges using 1-correlation ecoefficiency. The graph based clustering methods can be chosen from Louvain, walk_trap and infomap by <method> parameters. 
-    `snn_cluster()` computes the k.param nearest neighbors for a given feature-cell matrix or cell-feature matrix (set transposed =T in this case). It also computes shared nearest neighbors (SNN) and construct a SNN graph by calculating the neighborhood overlap (Jaccard index) between every cell and its k.param nearest neighbors. 
-     `wide_gcluster()` transforms the data.frame from long format to wide one.
-     'de_genes()' finds markers differentially expressed in each cluster or identity group by comparing it to all of the others. For each comparison, a new UMI matrix was created containing only the relevant cells. It was then log2(CPM/10+1)-transformed, filtered to only keep highly expressed genes, and mean-centered. P-values were corrected using the Benjamini-Hochberg method.

#### visualization


### Module2

![2_scRNA pipeline
infercna](https://user-images.githubusercontent.com/33009124/177924157-cda90bf3-4953-4c3f-9ab5-7ba03c6222c9.PNG)

infercna aims to provide functions for inferring CNA values from scRNA-seq data and related queries. 

#### compuatation
-   `infercna()` to infer copy-number alterations from single-cell RNA-seq data
-   `refCorrect()` to convert relative CNA values to absolute values + computed in `infercna()` if reference cells are provided
-   `cnaCor()` a parameter to identify cells with high CNAs + computed in `cnaScatterPlot()`
-   `cnaSignal()` a second parameter to identify cells with high CNAs + computed in `cnaScatterPlot()`
-   `findMalignant()` to find malignant subsets of cells
-   `findClones()` to identify genetic subclones
-   `fitBimodal()` to fit a bimodal gaussian distribution + used in `findMalignant()` + used in `findClones()`
-   `filterGenes()` to filter genes by their genome features
-   `splitGenes()` to split genes by their genome features
-   `orderGenes()` to order genes by their genomic position
-   `useGenome()` to change the default genome configured with infercna
-   `addGenome()` to configure infercna with a new genome specified by the user

#### visualization
-   `cnaPlot()` to plot a heatmap of CNA values
-   `cnaScatterPlot()` to visualise malignant and non-malignant cell subsets

### Module3
![3_scRNA pipeline metaprogram smallsize](https://user-images.githubusercontent.com/33009124/180667067-7097aff1-3782-4642-90a0-8439492c6572.PNG)

Functions are ready and need to be intergared into the R package. **to be continued**

### Module4
![4_scRNA pipeline TCR smallsize](https://user-images.githubusercontent.com/33009124/180667079-f04ac2ca-90fd-43a0-803a-1b58e7f0c1de.PNG)

Functions are ready and need to be intergared into the R package. **to be continued**

------------------------------------------------------------------------

## Version History

-   July 08, 2022 Version 0.1.0: Initial release; essential functions
    for streamlined scRNA-seq analysis and visualization.
-
