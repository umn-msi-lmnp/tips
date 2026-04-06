---
marp: true
---

Braedan's section of the IIG presentation on Nextflow from April, 2026. 
Detailed instructions for running an nf-core pipeline "offline" on e.g. Blackwell are included.

---

# The Challenge:

Build a workflow that:
- Analyzes 11 billion reads
- Runs multiple end-to-end analysis pipelines you've never used before 
- Runs in a tight timeline 
- Runs on a machine you cannot access (Blackwell)
- Runs without an online connection
- Starts with zero bioinformatics resources

---

# The Solution:
“Bringing **cloud-native, declarative, and stateless** concepts to batch HPC workloads solves most problems” 
– Chris Simmons' Thesis Statement for GPU in EDU Seminar at UMN, October 2025

1. Build a cloud-native workflow locally
2. Zip everything and transfer to Blackwell
3. Run the full pipeline on Blackwell with only local resources

---

## Requirements for an offline nf-core Nextflow workflow
- **1) Runtime tools:** Only need Java and Nextflow
- **2) Pipeline assets:** nf-core pipeline code, configs, plugins, etc.
- **3) Containers:** SIF images for required tools
- **4) References:** Genome and annotation caches needed by selected tools
- **5) Plugins + plugin config:** Custom plugins and plugin config

---

## Detailed implementation breakdown

- **Req 1) Runtime tools:** Install Java and Nextflow locally in `bin/`.

- **Req 2) Pipeline assets:** 
```
  nextflow pull nf-core/sarek            -r ${SAREK_VERSION}
  nextflow pull nf-core/viralintegration -r ${VIRALINTEGRATION_VERSION}
  nextflow pull nf-core/tumourevo        -r ${TUMOUREVO_VERSION}
```

Populates `.nextflow/assets/nf-core/` with complete, containerized, versioned software requirements for all three pipelines.

--- 

**Req 3-5) Run pipeline with test data**: Run just as you plan to offline.
```
export NXF_OFFLINE=FALSE                         // Run in ONLINE mode
export NXF_HOME=./.nextflow                      // Project-local Nextflow home
export NXF_APPTAINER_CACHEDIR=./apptainer_cache  // Location for Req 3: Containers
export IGENOMES_BASE=./references/igenomes       // Location for Req 4: References 
export PLUGINS_BASE=$NXF_HOME/plugins            // Location for Req 5: Plugins

```
```
### Example WGS pipeline with SNP, CNV, MSI, and SV calling
nextflow run nf-core/sarek -r "$SAREK_VERSION" \
-profile apptainer \
--genome "GRCh38" \
--igenomes_base "$IGENOMES_BASE" \           // Location for Req 4: References
--tools mutect2,cnvkit,msisensor2,manta \    // Tools to run
--input "samplesheet_test.csv" \             // With small test dataset
--outdir "results_test" \
-w "work_test" -resume
```

---

## Detailed implementation breakdown

- **3) Containers:** Test run pulls SIF images to a **project-local** cache.

```
ls -1 $NXF_APPTAINER_CACHEDIR

community.wave.seqera.io-library-bcftools-1.21--4335bec1d7b44d11.img
community.wave.seqera.io-library-bcftools_htslib-0a3fa2654b52006f.img
...
quay.io-biocontainers-trimmomatic-0.39--hdfd78af_2.img
quay.io-biocontainers-vcftools-0.1.16--he513fc3_4.img
```


---

## Detailed implementation breakdown
- **4) References:** Test run pulls genome and annotations to $IGENOMES_BASE.

```
ls $IGENOMES_BASE/Homo_sapiens/NCBI/GRCh38/*

./references/igenomes/Homo_sapiens/NCBI/GRCh38/Annotation:
Genes  Genes.gencode  NGSCheckMate  SmallRNA

./references/igenomes/Homo_sapiens/NCBI/GRCh38/Sequence:
AbundantSequences  Bowtie2Index  BWAIndex      Chromosomes  WholeGenomeFasta
BismarkIndex       BowtieIndex   BwamethIndex  STARIndex
```

---

## Detailed implementation breakdown
- **5) Plugins + plugin config:** Test run pulled plugins to $PLUGINS_BASE. 
Make config with plugin versions.

```
### List plugins
ls -1 .nextflow/plugins/ 
```

Format as a Nextflow config file and save to e.g. `conf/offline_plugins.config`
```
plugins {
    id 'nf-core-utils@0.4.0'
    id 'nf-fgbio@1.0.0'
    id 'nf-schema@2.6.1'
}
```

--- 

## Validate offline run in situ 
Just like before, but a few key changes:
```
export NXF_OFFLINE=TRUE                         // Run in OFFLINE mode
```
```
nextflow run nf-core/sarek -r "$SAREK_VERSION" \
-profile apptainer \
-c conf/offline_plugins.config \            // Plugin config for Req 5
--genome "GRCh38" \
--igenomes_base "$IGENOMES_BASE" \      
--tools mutect2,cnvkit,msisensor2,manta \
--input "samplesheet_test.csv" \
--outdir "results_offline" \                // Separate outdir for offline run
-w "work_offline"                           // Separate workdir for offline run

// no '-resume' flag ensures it runs the full pipeline offline
```

---

## Zip and transfer to offline-only location
  - `bin/` (Java and Nextflow)
  - `.nextflow/assets/` (Pipeline assets)
  - `apptainer_cache/` (Containers)
  - `references/` (Genome and annotation caches)
  - `.nextflow/plugins/` (Plugins)
  - `conf/offline_plugins.config` (Plugin config)

Keep the same relative paths in the offline location.

---

## Full-scale offline run on Blackwell 

Identical command as before, but with --input pointing to the full dataset.
```
nextflow run nf-core/sarek -r "$SAREK_VERSION" \
-profile apptainer \
-c conf/offline_plugins.config \            // Plugin config for Req 5
--genome "GRCh38" \
--igenomes_base "$IGENOMES_BASE" \      
--tools mutect2,cnvkit,msisensor2,manta \
--input "samplesheet_CART_all.csv" \
--outdir "results" \                // Separate outdir for offline run
-w "work"                           // Separate workdir for offline run
```

---

## Alternate approach:

https://nf-co.re/docs/usage/getting_started/offline

Uses python-based nf-core-cli to create a local "offline" environment. More dependencies for runtime, but fewer commands.
