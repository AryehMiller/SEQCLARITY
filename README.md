# SEQCLARITY: Codon-Aware Multiple Sequence Alignment, Filtering, Trimming, and Backtranslation

<p align="center">
  <img width="519" alt="SEQCLARITY" src="https://github.com/user-attachments/assets/76dc861c-4623-4047-af1f-c647a0dd27d5" />
</p>

## Table of Contents
- [Introduction](#introduction)
- [Should I use SEQCLARITY?](#should-i-use-seqclarity)
- [Installation](#installation)
- [Quick Start](#quick-start)
- [Detailed Tutorial](#detailed-tutorial)
- [Parameters and Fine-tuning](#parameters-and-fine-tuning)
- [Output Files](#output-files)
- [Troubleshooting](#troubleshooting)
- [Citation](#citation)

## Introduction

SEQCLARITY is a program designed to perform codon-aware multiple sequence alignment, codon-aware residue filtering and masking, codon-aware alignment trimming, and nucleotide backtranslation. This program is particularly useful for adding additional species to an existing alignment of orthologs while maintaining frame integrity. This program relies heavily on three already developed tools (MACSE, HMMCleaner, and TrimAl), but integrates these three using custom scripts to enable backtranslation between amino acid masking/filtering and the underlying nucleotide sequences to produce an improved nucleotide and protein sequence alignment by keeping track of filtered residues to maintain in-frame integrity.

## Should I use SEQCLARITY?

SEQCLARITY is useful if you find yourself in any of these circumstances:

**(1)** You have an alignment of well-aligned coding sequences that are in-frame, and would like to add (any number) of new orthologous sequences to this alignment, and trim and filter the alignment of non-homologies while maintain frame integrity.

**(2)** You have a codon alignment, but it appears to be riddled with mis-aligned regions and you want to correct these through masking and filtering while preserving frame integrity.

Essentially, if you have a codon alignment, and you would like to filter it to improve the signal:noise ratio, this is an appropriate usecase of SEQCLARITY. Optimizing this signal:noise ratio is key for any potential downstream analyses that are sensitive to alignment error (e.g., gene tree inference, dN/dS, etc.).

## Installation

### Prerequisites

- **Python 3.x** (tested with Python 3.7+)
- **Java** (for running MACSE)
- **Conda or Mamba** (recommended for environment management)

### Dependencies

SEQCLARITY requires the following external tools:

1. **[MACSE](https://www.agap-ge2pop.org/macse/macse-documentation/)**: Multiple Alignment of Coding SEquences
2. **[HMMCleaner](https://metacpan.org/dist/Bio-MUST-Apps-HmmCleaner)**: HMM-based alignment cleaning
3. **[TrimAl](https://trimal.cgenomics.org/)**: Alignment trimming tool
4. **BioPython**: Python library for biological computations

### Installation Steps

1. **Clone or download SEQCLARITY**:
   ```bash
   git clone <repository-url>
   cd SEQCLARITY
   ```

2. **Create a conda environment** (recommended):
   ```bash
   conda create -n seqclarity python=3.8 biopython
   conda activate seqclarity
   ```

3. **Install MACSE**:
   ```bash
   wget https://github.com/ranwez/MACSE_V2_PIPELINES/releases/download/v2.07/macse_v2.07.jar
   ```

4. **Install HMMCleaner**:
   - **Linux/MacOS**: 
     ```bash
     mamba install -y chrisjackson-pellicle::hmmcleaner
     ```
   - **Alternative**: Follow installation instructions at [HMMCleaner documentation](https://metacpan.org/dist/Bio-MUST-Apps-HmmCleaner)

5. **Install TrimAl**:
   ```bash
   conda install -c bioconda trimal
   ```

### Verification

Test your installation by checking if all tools are accessible:
```bash
java -jar macse_v2.07.jar -help
HmmCleaner.pl -help
trimal -h
python -c "import Bio; print('BioPython installed successfully')"
```

## Quick Start

### Basic Usage

For a simple run with default parameters:

```bash
python SEQCLARITY.py \
    --reference_alignment reference.fasta \
    --query_sequences new_sequences.fasta \
    --output_dir results/ \
    --macse_path /path/to/macse_v2.07.jar \
    --hmmcleaner_path /path/to/HmmCleaner.pl \
    --gene_ids gene1 gene2
```

### Example with Custom Parameters

```bash
python SEQCLARITY.py \
    --reference_alignment reference.fasta \
    --query_sequences new_sequences.fasta \
    --output_dir results/ \
    --macse_path macse_v2.07.jar \
    --hmmcleaner_path HmmCleaner.pl \
    --gene_ids gene1 \
    --hmmcleaner_costs "-0.1 -0.05 0.1 0.4" \
    --trimal_algorithm gappyout \
    --min_seq_length 150 \
    --ambiguous_threshold 0.3
```

## Detailed Tutorial

### Step-by-Step Workflow

This section provides a detailed explanation of each step in the SEQCLARITY pipeline:

#### Input Files

1. **Reference Alignment**: A well-aligned FASTA file containing your reference sequences (nucleotide coding sequences)
2. **Query Sequences**: FASTA file containing new sequences to add to the reference alignment

#### Pipeline Steps

**Step 1: Quality Filtering**
- Removes sequences with excessive ambiguous residues (N, -, ?)
- Default threshold: >50% ambiguous residues
- Customizable with `--ambiguous_threshold`

**Step 2: Sequence Enrichment**
- Uses MACSE's `enrichAlignment` to add query sequences to reference alignment
- Maintains codon structure and reading frame
- Produces both nucleotide and amino acid alignments

**Step 3: Header Cleanup**
- Removes whitespace from sequence headers for compatibility

**Step 4: Stop Codon and Frameshift Masking**
- Masks internal stop codons and frameshifts using MACSE
- Replaces problematic codons with 'NNN'

**Step 5: HMM-based Cleaning**
- Uses HMMCleaner to identify and mark poorly aligned regions
- Builds HMM profiles to detect alignment artifacts
- Configurable cost parameters for sensitivity adjustment

**Step 6: Codon-aware Filtering**
- Removes marked residues while maintaining codon structure
- Uses MACSE's `reportMaskAA2NT` for backtranslation

**Step 7: Gap Trimming**
- Uses TrimAl to remove gap-rich regions
- Multiple algorithms available (strictplus, gappyout, etc.)
- Identifies columns for removal

**Step 8: Final Codon-aware Trimming**
- Applies TrimAl column removals in codon-aware manner
- Produces final cleaned alignments

**Step 9: Output Generation**
- Generates final nucleotide and amino acid alignments
- Cleans up intermediate files

## Parameters and Fine-tuning

SEQCLARITY offers extensive customization options to optimize performance for your specific datasets:

### Required Parameters

| Parameter | Description |
|-----------|-------------|
| `--reference_alignment` | Path to reference alignment FASTA file |
| `--query_sequences` | Path to query sequences FASTA file |
| `--output_dir` | Output directory path |
| `--macse_path` | Path to MACSE jar file |
| `--hmmcleaner_path` | Path to HMMCleaner script |
| `--gene_ids` | List of gene IDs to process |

### Fine-tuning Parameters

#### HMMCleaner Parameters
| Parameter | Default | Description |
|-----------|---------|-------------|
| `--hmmcleaner_costs` | `-0.15 -0.08 0.15 0.45` | Cost matrix for HMMCleaner (4 space-separated values) |

**HMMCleaner Cost Guidelines:**
- More negative first two values = more stringent filtering
- More positive last two values = more stringent filtering
- Conservative: `-0.1 -0.05 0.1 0.4`
- Stringent: `-0.2 -0.1 0.2 0.5`

#### TrimAl Parameters
| Parameter | Default | Options | Description |
|-----------|---------|---------|-------------|
| `--trimal_algorithm` | `strictplus` | `strict`, `strictplus`, `gappyout`, `automated1`, `nogaps` | Algorithm for gap trimming |

**TrimAl Algorithm Guidelines:**
- `strictplus`: Balanced approach (recommended)
- `strict`: Conservative gap removal
- `gappyout`: Aggressive gap removal for gappy alignments
- `automated1`: Automatic parameter selection
- `nogaps`: Remove all gap-containing columns

#### Sequence Filtering Parameters
| Parameter | Default | Description |
|-----------|---------|-------------|
| `--min_seq_length` | `200` | Minimum sequence length to retain |
| `--ambiguous_threshold` | `0.5` | Maximum fraction of ambiguous residues (0-1) |

#### MACSE Fine-tuning Parameters
| Parameter | Default | Description |
|-----------|---------|-------------|
| `--min_nt_to_keep` | `30` | Minimum nucleotides to keep a sequence |
| `--min_seq_to_keep_site` | `4` | Minimum sequences required to keep a site |
| `--min_percent_nt_at_ends` | `0.3` | Minimum percentage of nucleotides at sequence ends |
| `--dist_isolate_aa` | `3` | Distance to isolate amino acids |
| `--min_homology_to_keep` | `0.3` | Minimum homology threshold to keep sequences |
| `--min_internal_homology` | `0.5` | Minimum internal homology threshold |
| `--final_min_seq_to_keep_site` | `20` | Final minimum sequences to keep a site |
| `--final_min_percent_nt` | `0.5` | Final minimum percentage of nucleotides at ends |

### Parameter Optimization Strategies

#### For High-Quality Reference Alignments
```bash
# Conservative parameters to preserve most data
python SEQCLARITY.py \
    --hmmcleaner_costs "-0.1 -0.05 0.1 0.4" \
    --trimal_algorithm strict \
    --ambiguous_threshold 0.4 \
    --min_homology_to_keep 0.2
```

#### For Noisy or Divergent Sequences
```bash
# Stringent parameters to improve alignment quality
python SEQCLARITY.py \
    --hmmcleaner_costs "-0.2 -0.1 0.2 0.5" \
    --trimal_algorithm gappyout \
    --ambiguous_threshold 0.3 \
    --min_homology_to_keep 0.4 \
    --final_min_seq_to_keep_site 30
```

#### For Large-scale Datasets
```bash
# Balanced parameters for throughput and quality
python SEQCLARITY.py \
    --trimal_algorithm automated1 \
    --min_seq_length 150 \
    --min_seq_to_keep_site 10
```

## Output Files

SEQCLARITY generates several output files for each gene processed:

### Primary Outputs

| File | Description |
|------|-------------|
| `{gene_id}.TrulyEnriched.NT.fasta` | **Final cleaned nucleotide alignment** |
| `{gene_id}.TrulyEnriched.AA.fasta` | **Final cleaned amino acid alignment** |
| `{gene_id}.Enriched.NT.fasta` | Initial enriched nucleotide alignment |
| `{gene_id}.Enriched.AA.fasta` | Initial enriched amino acid alignment |

### Directory Structure
```
output_dir/
└── {gene_id}/
    └── RCCor/
        ├── {gene_id}.TrulyEnriched.NT.fasta    # Final NT alignment
        ├── {gene_id}.TrulyEnriched.AA.fasta    # Final AA alignment
        ├── {gene_id}.Enriched.NT.fasta         # Initial NT alignment
        └── {gene_id}.Enriched.AA.fasta         # Initial AA alignment
```

### Quality Assessment

To assess the quality of your results:

1. **Check sequence retention**: Compare input vs output sequence counts
2. **Examine alignment length**: Verify reasonable alignment length after trimming
3. **Visual inspection**: Use alignment viewers (e.g., AliView, MEGA) to examine results
4. **Downstream analysis**: Test with phylogenetic analysis or other applications

## Troubleshooting

### Common Issues and Solutions

#### Issue: "Java not found" or MACSE errors
**Solution:**
```bash
# Ensure Java is installed and accessible
java -version
# Update Java path if necessary
export JAVA_HOME=/path/to/java
```

#### Issue: HMMCleaner not found
**Solution:**
```bash
# Try alternative installation
conda install -c conda-forge perl
cpan Bio::MUST::Apps::HmmCleaner
# Or use mamba as suggested in installation
```

#### Issue: "No sequences retained after filtering"
**Solutions:**
- Relax ambiguous threshold: `--ambiguous_threshold 0.7`
- Reduce minimum sequence length: `--min_seq_length 100`
- Check input sequence quality and format

#### Issue: TrimAl removes too much data
**Solutions:**
- Use more conservative algorithm: `--trimal_algorithm strict`
- Adjust MACSE parameters to be less stringent
- Check alignment quality before trimming

#### Issue: Memory errors with large datasets
**Solutions:**
```bash
# Increase Java heap size for MACSE
export JAVA_OPTS="-Xmx8g"
# Process genes separately
# Split large input files into smaller chunks
```

#### Issue: Permission denied errors
**Solution:**
```bash
# Ensure all tool scripts are executable
chmod +x /path/to/HmmCleaner.pl
chmod +x /path/to/trimal
```

### Getting Help

If you encounter issues not covered here:

1. Check that all dependencies are properly installed and accessible
2. Verify input file formats (FASTA format, nucleotide sequences)
3. Test with a small subset of your data first
4. Check the intermediate files in the output directory for clues

### Performance Tips

- **Start with default parameters** and adjust based on results
- **Test with a subset** of your data before running large datasets
- **Monitor intermediate files** to understand where issues occur
- **Use appropriate resources** (memory, CPU) for large datasets

## Citation

If you use SEQCLARITY in your research, please cite:

```
[Citation information to be added]
```

And the underlying tools:
- **MACSE**: Ranwez V, et al. (2011) MACSE: Multiple Alignment of Coding SEquences accounting for frameshifts and stop codons. PLoS ONE 6(9): e22594.
- **HMMCleaner**: Di Franco A, et al. (2019) Evaluating the usefulness of alignment filtering methods to reduce the impact of errors on evolutionary inferences. BMC Evolutionary Biology 19: 21.
- **TrimAl**: Capella-Gutiérrez S, et al. (2009) trimAl: a tool for automated alignment trimming in large-scale phylogenetic analyses. Bioinformatics 25(15): 1972-1973.

---

**SEQCLARITY** - Optimizing sequence alignments for downstream phylogenomic analyses
