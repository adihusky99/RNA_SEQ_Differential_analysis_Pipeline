# Cytokine/Chemokine Gene Expression Analysis Pipeline

## Overview

This is a comprehensive bioinformatics pipeline for analyzing cytokine and chemokine gene expression from RNA-seq data. The pipeline converts mouse Ensembl gene IDs to gene symbols, extracts target cytokine genes, and performs differential expression analysis across treatments and tissues.

## Pipeline Workflow

Follow these scripts **in this exact order**:

### 1. **mygene.ipynb** - Gene ID Conversion
**Purpose**: Convert Ensembl gene IDs to gene symbols

**What it does**:
- Installs and initializes the mygene library
- Performs a quick test with known mouse genes to verify the library works
- Converts 29,993 Ensembl gene IDs to gene symbols
- Achieves ~98.4% conversion success rate
- Creates a new CSV with both Ensembl IDs and gene symbols

**Input**: 
- `normalized_counts.csv` (original data with Ensembl IDs)

**Output**: 
- `normalized_counts_with_symbols_mygene.csv` (data with gene symbols added)

**Notes**:
- This script handles the heavy lifting of gene ID conversion
- Much more reliable than BioMart API calls
- Some genes (452) won't convert - these are typically retired or non-standard IDs

---

### 2. **cytokine_check.ipynb** - Target Gene Verification
**Purpose**: Quick check to see which target cytokines/chemokines are in your dataset

**What it does**:
- Searches for 24 target cytokine and chemokine genes
- Handles common naming variants (e.g., TNF, Tnf, TNF-α)
- Shows which genes are present and which are missing
- Provides Ensembl IDs for all found genes

**Input**: 
- `normalized_counts_with_symbols_mygene.csv` (output from step 1)

**Output**: 
- Console output only (optional - use for verification)

**Target Genes Searched**:
- TNF, IL6, IL10, IFNG, VEGFA, CSF2, CSF3
- CCL2, CXCL10, CCL21, IL1B, IL1A, IL2, IL4, IL5
- IL13, IL17A, TGFB1, CCL5, CXCL1, CD14, TREM1
- TNFRSF1A, TNFRSF1B

**Notes**:
- Run this to confirm your target genes are present before proceeding
- If key genes are missing, you may need to adjust your analysis approach

---

### 3. **cytokine_filter_extract_analysis.ipynb** - Extract Target Genes
**Purpose**: Extract only the target cytokine genes from the full dataset

**What it does**:
- Filters the full dataset to only include your 24 target cytokine/chemokine genes
- Creates a standardized gene name column for easy analysis
- Generates three output files with increasing levels of preparation
- Calculates summary statistics for each gene
- Sorts genes by mean expression level

**Input**: 
- `normalized_counts_with_symbols_mygene.csv` (output from step 1)

**Output**: 
- `cytokine_panel_genes_only.csv` - Full data with all expression columns
- `cytokine_panel_summary.csv` - Summary statistics (mean, median, max, min, std)
- `cytokine_panel_for_analysis.csv` - Clean format with genes as index (ready for analysis)

**Notes**:
- This dramatically reduces data size (29,993 genes → ~24 genes)
- Makes subsequent analysis much faster
- Summary statistics show which genes are most/least expressed

---

### 4. **cytokine_differential_analysis.ipynb** - Treatment Comparison Analysis
**Purpose**: Compare cytokine expression between different treatment groups

**What it does**:
- Loads expression data and merges with treatment information from your project log
- Filters genes and applies log2 transformation
- Performs PCA analysis to visualize treatment effects
- Creates heatmaps showing gene expression by treatment
- Performs pairwise statistical comparisons between treatments
- Generates volcano plots showing significantly changed genes
- Creates a cross-tissue summary of treatment effects
- Saves all results in timestamped directories

**Input**: 
- `cytokine_panel_for_analysis.csv` (output from step 3)
- `GTBH25-SimonM-54_RNA_Project_Log_v01-25.csv` (your treatment metadata file)

**Output Structure** (in timestamped directory like `treatment_focused_analysis_20240115_143022/`):
```
├── plots/
│   ├── *_treatment_pca.png/pdf - PCA visualizations
│   ├── *_treatment_heatmap.png/pdf - Expression heatmaps
│   ├── *_vs_*_volcano.png/pdf - Volcano plots for each comparison
│   └── cross_tissue_treatment_summary.png/pdf - Summary across tissues
├── data/
│   ├── filtered_expression_data.csv
│   ├── log_transformed_data.csv
│   └── *_treatment_heatmap_data.csv
└── statistics/
    ├── *_DE.csv - Differential expression results
    └── treatment_effects_summary.csv - Cross-tissue summary
```

**Key Analyses**:
- **PCA**: Shows how treatments cluster samples
- **Heatmaps**: Shows mean expression levels by treatment
- **Volcano Plots**: Shows which genes change significantly with treatment
- **Statistics**: FDR-adjusted p-values for each gene comparison

**Notes**:
- This script is tissue and treatment-aware
- It groups by animal, tissue type, and treatment
- Creates automatic timestamped output directories
- Shows top genes by effect size even if statistical significance is weak

---

## Installation Requirements

Before running the pipeline, install these Python packages:

```bash
pip install mygene pandas numpy matplotlib seaborn scikit-learn scipy statsmodels
```

## Key Parameters to Adjust

In each script, you may need to modify file paths:

**In mygene.ipynb**:
- `input_file` - Path to your normalized_counts.csv
- `output_file` - Where to save the converted file

**In cytokine_filter_extract_analysis.ipynb**:
- `input_file` - Path to normalized_counts_with_symbols_mygene.csv
- `output_file` - Where to save extracted genes

**In cytokine_differential_analysis.ipynb**:
- `expression_csv` - Path to cytokine_panel_for_analysis.csv
- `log_csv` - Path to your treatment metadata file

**Modify target_genes dictionary** if your target genes differ from the 24 listed

## Expected Workflow Time

- Step 1 (mygene): 10-15 minutes (one-time large conversion)
- Step 2 (check): <1 minute (quick verification)
- Step 3 (extract): <1 minute (fast filtering)
- Step 4 (differential analysis): 5-10 minutes (depends on number of treatments/tissues)

**Total**: ~30 minutes first time, faster on subsequent runs

## Troubleshooting

**Problem**: mygene connection errors
**Solution**: Check internet connection, library may need reinstall: `pip install --upgrade mygene`

**Problem**: "Genes column not found"
**Solution**: Verify your CSV has a column named exactly "Genes" with Ensembl IDs

**Problem**: No target genes found in step 2
**Solution**: Check gene naming in your dataset - may use different format (all lowercase, different species annotation, etc.)

**Problem**: Treatment log doesn't match samples
**Solution**: Verify sample ID format in project log matches column names in expression file

## Output Interpretation

**PCA Plots**: 
- Points closer together = more similar expression patterns
- Color by treatment = treatment effect on expression
- Color by animal = individual variation

**Heatmaps**: 
- Red/hot colors = high expression
- Blue/cool colors = low expression
- Rows = genes, columns = treatments

**Volcano Plots**:
- X-axis = fold change (log2 scale)
- Y-axis = statistical significance (-log10 p-value)
- Red dots = higher in first treatment
- Blue dots = higher in second treatment
- Points at top-left/right = significantly changed genes

## Notes

- All scripts are self-contained and can be re-run independently
- Outputs from each step are used as inputs for the next step
- The analysis is treatment-focused, comparing effects across tissues
- Scripts include extensive status messages and summaries for debugging

## Author Notes

- Pipeline designed for mouse RNA-seq data with Ensembl gene IDs
- Easily adaptable to other species by changing mygene species parameter
- Can be extended to include additional statistical tests or visualizations
