# Variant Calling: bcftools vs GATK

A comparison of two variant-calling workflows, **bcftools** and **GATK**, run on the
same set of 10 burbot individuals (reads previously aligned to the burbot reference
genome with BWA). The goal is to call variants with each tool, apply identical
filtering to both, and compare how much they agree.

This was a graduate genomics-methods project run on a SLURM HPC cluster
(Compute Canada / Graham), with downstream comparison in R.

## Workflow

```
sorted BAMs (10 individuals)
        |
        +-----------------------------+
        |                             |
        v                             v
  [ bcftools mpileup + call ]   [ GATK (with read groups) ]
        |                             |
        v                             v
   raw VCF                       raw VCF
        |                             |
        +-------- identical filtering (vcftools) --------+
        |   max-missing 0.5, MAC >= 3, minQ 30,          |
        |   biallelic only, indels removed               |
        v                                                v
  filtered SNP VCF                              filtered SNP VCF
        |                                                |
        +------------- bcftools isec (overlap) ----------+
                              |
                              v
              shared vs caller-unique SNPs
```

## What it does

1. **Call variants two ways.** bcftools (`mpileup` + `call`) and GATK are each run on
   the same 10 individuals. GATK requires read groups in the BAMs and an indexed
   reference (Picard `CreateSequenceDictionary`), so those steps are included.
2. **Filter identically.** Both call sets are filtered with the same vcftools criteria
   (>50% missing removed, minor-allele-count >= 3, quality >= 30, biallelic sites only),
   so the comparison is controlled.
3. **Isolate filter impact.** Each filter is also applied on its own as a side analysis;
   minor-allele-count consistently removes the most loci for both callers.
4. **Compare callers.** Filtered, indel-free SNP sets are intersected with `bcftools isec`
   to count shared vs caller-unique SNPs, and allele frequency / site depth are exported
   for downstream analysis in R.

## Selected results

| Step | bcftools | GATK |
|---|---|---|
| Raw SNPs | 91,550 | 30,791 |
| Raw indels | 9,937 | 4,298 |
| SNPs after full filtering | 5,372 | 3,739 |
| Shared SNPs (intersection) | 3,141 | 3,141 |

The two callers agree on a large core set of SNPs while each also calls a set the
other misses, which is the expected behavior and the point of the comparison.

## Files

| File | What it is |
|---|---|
| `Bcftools_GATK` | The full annotated workflow (bash), with commands, filtering rationale, and recorded results |

## Notes

- Tools: bcftools, GATK, vcftools, samtools, Picard, run via environment modules on an HPC cluster.
- Data: burbot sequencing reads from a graduate genomics-methods course; reference genome `GCA_900302385.1` (public, NCBI).
- This is an early graduate project kept as a record of hands-on variant-calling and
  filtering work; for a cleaner, containerized, reproducible take on variant calling
  see [genomics-variant-calling](https://github.com/ramibaghdan/genomics-variant-calling).
