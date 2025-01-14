---
layout: post
title: Refining growth/condition marker discovery in Pacific cod, GWAS Day 4
--- 

I am interested to see whether I can use the putative growth/condition markers detected in experimental Pacific cod to predict "performance" in our reference/wild caught cod from different marine regions (Northern Bering Sea, Aleutians, Eastern Bering Sea / western Gulf of Alaska, eastern Gulf of Alaska). I have a beagle file containing genotype likelihoods for sites detected in both experimental and reference fish. I filtered that beagle file for only our putative growth/performance markers, then ran PCAngsd to generate a covariance matrix, then PCA in R. My thought is that I could potentially use a growth/condition gradient along a PC axis from experimental fish to identify reference fish populations/regions that lie on the high growth/condition end (i.e. potential high performers). 

Here is the PCA of those ~270 putative growth/condition markers with growth/condition index of experimental fish indicated by the gradient, and all reference fish in gray. Second figure is a regression of growth/condition index ~ PC1 scores for experimental fish only, which I could possibly use to potentially predict performance in reference fish - the more negative PC1 score the better?. The boxplot shows PC1 scores for reference fish, potentially indicating eGOA fish have the most negative PC1 score / best performance indicator and tagged/NBS fish have the highest PC1 scor. 

![image](https://github.com/user-attachments/assets/e1a235ec-4f58-48e3-840f-035b96edf113)

![image](https://github.com/user-attachments/assets/619d5915-c367-4444-8938-4d2abab914d1)

![image](https://github.com/user-attachments/assets/2d8adf08-fb3c-453a-ae2a-723d0c20a6bc)

#### What's happening here
This is strange, though. When I plot PC1 scores for experimental fish alongside reference fish, obviously the variability is much higher in experimental fish. BUT also - there are differences among temperature treatments, which isn't a good sign. I believe this indicates that the marker discovery is being influenced by growth/condition differences among treatments. This suggests to me that I need to run my GWAS analyses separately for each temperature and look for 1) common markers among treatments that could indicate general growth/condition performance, and 2) unique markers among treatments that could reflect temperature-specific markers.  

![image](https://github.com/user-attachments/assets/7cc69e5c-470f-4df3-a0c5-bc865fb47475)

## Re-run GWAS separately for each temperature 
Here I perform four separate GWAS runs for each temperature treatment (0,5,9,16), with n=40/treatment for all but the 16C group (n=37, unfortunately we don't have good DNA data for three that died!). I created four gwas subdirectories (temp-#), re-ran ANGSD to generate beagle files with genotype likelihoods, then ran `angsd -doAsso`. Here is an example set of scripts used for 0C group, located in the directory **/home/lspencer/pcod-lcwgs-2023/analysis-20240606/experimental/gwas/temp-0/**:  

### First, resolving GWAS issue  
For some _unknown_ reason the GWAS ran successfully on raw genotype likelihoods for 0C, 5C, and 9C, but NOT for 16C. I kept encountering a "segmentation fault" error, which I've previously encountered with angsd v0.940 (but it's weird that it's only happening with one set of samples). After TOO MUCH TIME troubleshooting, I've decided to pivot to another approach - using angsd v0.933 instead. That version kept throwing an error related to improperly handing chromosome name; it couldn't get past the "_" in each chromosome id_site (NC_XXXXX.1_#####) in the beagle files, so I never attempted to use it. But now I have to! So, I'll modify the input beagles and genome reference to remove that first "_".

#### Modify pcod genome file to remove "_" in chromosome ID 
```
sed '/^>/ s/NC_/NC/' /home/lspencer/references/pcod-ncbi/GCF_031168955.1_ASM3116895v1_genomic_rehead.fa > pcod-genome.fa  #use sed to replace "NC_" with "NC" in header lines

(base) [lspencer@node28 gwas]$ head pcod-genome.fa #view results
>NC082382.1
CAGACCTCCAATAGAGAGCTGCTCCCCCTTCGCACAAAGCCGCTGCTGGTGAATGCTCGAAGCGTTGTGTGATTGAATCG
CTTTAATGCCGTTCCATGTCACGTTGATCGTTTTTTTGCACAACGAGCAAAAAGCTTCCCGGTCATTTCCCATCACGGGT
TTCAGCCAGTCTTTAAATGTCGGGTCGTCAACCCATGTTTGCGAaaacttgcatttacccattcTCTCATAGAGCCAGAA
ACTCAGCCGTACAGAACTCTGAGGGGAAAAGGCGGAAAATGTTGctggtgttacaccgaattctgtgacccaccgaaaag
tgtggcCCCAGGGGGTGcagttgcacattttctgcaacatgcacgtgtgcattataacaaaaaacataaataaaacataa
gatctctggtgggtctttctttttttgatactaacattaattctaaacactattttacacagtagcctatactaggaaat
gctcaaaggtaaatcagcagAATTTAACcatgactgtagctttttatgggtaaagaagcaacggtgaaggacccTACTAA
ACGCccgtctgtaaaatgctaatcaaatcaatcaaatgtatttattaagcacctttaataaaaaggtaattggttggggg
gagatgttatgttttatgtatgtttttgtcatgcctgtctacgcttcaaactcgtgcatattgcagtaaatatgcaacag

(base) [lspencer@node28 gwas]$ samtools faidx pcod-genome.fa #index new genome file 

(base) [lspencer@node28 gwas]$ cat pcod-genome.fa.fai #view index file 
NC082382.1      26289739        12      80      81
NC082383.1      23805147        26618385        80      81
NC082384.1      26984552        50721109        80      81
NC082385.1      35077496        78042980        80      81
NC082386.1      22175970        113558957       80      81
NC082387.1      27900680        136012139       80      81
NC082388.1      28186669        164261590       80      81
NC082389.1      23760065        192800605       80      81
NC082390.1      23393856        216857683       80      81
NC082391.1      23728004        240543975       80      81
NC082392.1      26267057        264568592       80      81
NC082393.1      27587113        291164000       80      81
NC082394.1      22312454        319095964       80      81
NC082395.1      26048790        341687336       80      81
NC082396.1      24646214        368061748       80      81
NC082397.1      29952890        393016052       80      81
NC082398.1      16295523        423343366       80      81
NC082399.1      21077965        439842596       80      81
NC082400.1      17993618        461184048       80      81
NC082401.1      23303860        479402599       80      81
NC082402.1      19478734        502997770       80      81
NC082403.1      19411328        522720001       80      81
NC082404.1      20003359        542373983       80      81
NC036931.1      16569   562627396       80      81
```

### angsdARRAY.sh - generate GLs (and some other files) for each chromosome with desired ANGSD settings 
```
#!/bin/bash

#SBATCH --cpus-per-task=10
#SBATCH --time=7-00:00:00
#SBATCH --job-name=angsd_0
#SBATCH --output=/home/lspencer/pcod-lcwgs-2023/analysis-20240606/experimental/gwas/temp-0/angsd_%A-%a.out
#SBATCH --mail-type=FAIL
#SBATCH --mail-user=laura.spencer@noaa.gov
#SBATCH --array=1-24%6

## ANGSD Settings
# -setminDepth 50 is based on 50% of the individuals and the average sequencing coverage of 3X (i.e. 50 ~ 3x * 0.5*40)
# -setmaxDepth 300 is based on a very generous 2 times all the individuals and the average depth, to avoid sites overly sequenced (i.e. 300 >~ 2 * 40 * 3x)
# -minInd 20 sets calling only variants with ~50% of individuals
# -minMaqQ was 15, now 30
# -minQ was 15, now 33
# -GL was 1, now 2

module unload bio/angsd/0.933
module load bio/angsd/0.933
base=/home/lspencer/pcod-lcwgs-2023/analysis-20240606/experimental
temp="0"
workdir=${base}/gwas/temp-${temp}
bams=${workdir}/bamslist_${temp}.txt

# array input is chromosomes
JOBS_FILE=${base}/scripts/pcod-exp_angsdARRAY_input.txt
IDS=$(cat ${JOBS_FILE})

for sample_line in ${IDS}
do
        job_index=$(echo ${sample_line} | awk -F ":" '{print $1}')
        contig=$(echo ${sample_line} | awk -F ":" '{print $2}')
        if [[ ${SLURM_ARRAY_TASK_ID} == ${job_index} ]]; then
                break
        fi
done

angsd -b ${bams} \
-ref /home/lspencer/references/pcod-ncbi/GCF_031168955.1_ASM3116895v1_genomic.fa \
-r ${contig}: \
-out ${workdir}/Chr${job_index}_${temp} \
-nThreads 10 \
-uniqueOnly 1 \
-remove_bads 1 \
-trim 0 \
-C 50 \
-minMapQ 30 \
-minQ 33 \
-doCounts 1 \
-setminDepth 50 \
-setmaxDepth 300 \
-GL 2 \
-doBcf 1 \
-doPost 1 \
-doGeno 32 \
-doGlf 2 \
-doMaf 1 \
-doMajorMinor 1 \
-doDepth 1 \
-dumpCounts 3 \
-only_proper_pairs 1 \
-minInd 20 \
-minmaf 0.05 \
-SNP_pval 1e-10 \
-baq 1
```
### concat.sh - concatenage beagles (containing GLs) into one whole genome beagle file 
```
#!/bin/bash

#SBATCH --cpus-per-task=10
#SBATCH --job-name=concat-beagles-0
#SBATCH --mail-type=ALL
#SBATCH --mail-user=laura.spencer@noaa.gov
#SBATCH --output=/home/lspencer/pcod-lcwgs-2023/analysis-20240606/experimental/gwas/temp-0/concat.out

# Define the input directory containing all .beagle.gz files
temp="0"
input_dir="/home/lspencer/pcod-lcwgs-2023/analysis-20240606/experimental/gwas/temp-${temp}"
output_file="${input_dir}/wholegenome-${temp}.beagle"

# Extract the header from the first .beagle.gz file
zcat "${input_dir}/Chr1_0.beagle.gz" | head -n 1 > "${output_file}"

# Loop through chromosomes 1 to 24 and concatenate their contents (excluding headers)
for i in {1..24}; do
    file="${input_dir}/Chr${i}_0.beagle.gz"
    if [[ -f "$file" ]]; then
        echo "Processing $file..."
        zcat "$file" | tail -n +2 >> "${output_file}"
    else
        echo "Warning: File $file not found. Skipping..."
    fi
done

# Compress the resulting .beagle file
gzip "${output_file}"

echo "Concatenation complete. Output written to ${output_file}.gz"
```

### gwas.sh - edit beagle chrom IDs, impute genotype probabilities, and perform association tests using various traits using both  "raw" GLs and imputed GPs 
_NOTE: here I only show code for GWAS using the raw GLs due to space, but code is same just with $imputed beagle file variable and different output names)_ 


```
#!/bin/bash
#SBATCH --time=0-20:00:00
#SBATCH -p himem
#SBATCH --mem=100G
#SBATCH --job-name=gwas-16
#SBATCH --mail-type=ALL
#SBATCH --mail-user=laura.spencer@noaa.gov
#SBATCH --output=/home/lspencer/pcod-lcwgs-2023/analysis-20240606/experimental/gwas/temp-16/gwas-16.out

module load bio/angsd/0.933 #important to use this version

temp="16"
base=/home/lspencer/pcod-lcwgs-2023/analysis-20240606/experimental/gwas
beagle=${base}/temp-${temp}/wholegenome-${temp}.beagle.gz
fai=${base}/pcod-genome.fa.fai

# Remove "_" from chromosome IDs in beagle file (it confuses angsd)
zcat ${beagle} | sed 's/NC_/NC/g' | gzip > ${base}/temp-${temp}/wholegenome-${temp}-rehead.beagle.gz
beagle_rehead=${base}/temp-${temp}/wholegenome-${temp}-rehead.beagle.gz

# FIRST PERFORM IMPUTATION (to fill in NA values) with beagle

# All experimental fish
java -Xmx15000m -jar /home/lspencer/programs/beagle.jar \
like=${beagle_rehead} \
out=${base}/temp-${temp}/imputed

imputed=${base}/temp-${temp}/imputed.wholegenome-${temp}-rehead.beagle.gz.gprobs.gz

# ---------------------
#  CPI from all 4 growth and condition metrics (SGR-sl, SGR-ww, HSI, Kwet)
angsd \
-doMaf 4 \
-beagle ${beagle_rehead} \
-yQuant ${base}/temp-${temp}/${temp}-pi.grow.cond1.txt \
-doAsso 4 \
-out ${base}/temp-${temp}/${temp}-gwas-pi-grow-cond1.out \
-fai ${fai}

# ------------------
#  CPI from 3 growth and condition metrics (SGR-sl, SGR-ww, HSI)
angsd \
-doMaf 4 \
-beagle ${beagle_rehead} \
-yQuant ${base}/temp-${temp}/${temp}-pi.grow.cond2.txt \
-doAsso 4 \
-out ${base}/temp-${temp}/${temp}-gwas-pi-grow-cond2.out \
-fai ${fai}

# ------------------
#  CPI from 2 growth and condition metrics (SGR-ww, HSI)
angsd \
-doMaf 4 \
-beagle ${beagle_rehead} \
-yQuant ${base}/temp-${temp}/${temp}-pi.grow.cond3.txt \
-doAsso 4 \
-out ${base}/temp-${temp}/${temp}-gwas-pi-grow-cond3.out \
-fai ${fai}

# ------------------
# SGR - standard length
angsd \
-doMaf 4 \
-beagle ${beagle_rehead} \
-yQuant ${base}/temp-${temp}/${temp}-sgr-sl.txt \
-doAsso 4 \
-out ${base}/temp-${temp}/${temp}-gwas-sgr-sl.out \
-fai ${fai}

# SGR - wet weight
angsd \
-doMaf 4 \
-beagle ${beagle_rehead} \
-yQuant ${base}/temp-${temp}/${temp}-sgr-ww.txt \
-doAsso 4 \
-out ${base}/temp-${temp}/${temp}-gwas-sgr-ww.out \
-fai ${fai}

# HSI
angsd \
-doMaf 4 \
-beagle ${beagle_rehead} \
-yQuant ${base}/temp-${temp}/${temp}-hsi.txt \
-doAsso 4 \
-out ${base}/temp-${temp}/${temp}-gwas-hsi.out \
-fai ${fai}

# KWet
angsd \
-doMaf 4 \
-beagle ${beagle_rehead} \
-yQuant ${base}/temp-${temp}/${temp}-kwet.txt \
-doAsso 4 \
-out ${base}/temp-${temp}/${temp}-gwas-kwet.out \
-fai ${fai}
```

### Examine GWAS results in R 

Here is R Code I used to import, filter, view, and annotate GWAS-identified "putative markers": 
```
temp="16"
trait.label="Composite index #3, growth rates & liver index" #
trait="pi.grow.cond2" #"SGR.ww.trt" #SGR.sl.trt, Kwet, pi.grow.cond1, hsi, pi.grow.cond3
trait.file="pi-grow-cond2" #sgr-sl, sgr-ww, kwet, pi-grow-cond1, hsi, pi-grow-cond3

# Here i use imputed genotypes 
lrt.temp<-read.table(gzfile(paste0("../integrated/gwas/by-temp/", temp, "-gwas-impute-", trait.file, ".out.lrt0.gz")), header=T, sep="\t") %>% 
  mutate(Chromosome=gsub("NC", "NC_", Chromosome)) #important to add back the "_" in the chromosome name or else no markers will be annotated 

#we have a few LRT values that are -999, we should remove them. 
length(lrt.temp$LRT) # number of loci 
length(which(lrt.temp$LRT == -999)) #number that will be removed 

length(which(lrt.temp$LRT != -999))

#remove the values that are not -999 and that are negative
lrt.temp_filt<-lrt.temp[-c(which(lrt.temp$LRT == -999),which(lrt.temp$LRT <= 0)),]

#add snp "name" from rownumber 
lrt.temp_filt$SNP<-paste("r",1:length(lrt.temp_filt$Chromosome), sep="")

# get pvalues from chisq 
lrt.temp_filt$pvalue<-pchisq(lrt.temp_filt$LRT, df=1, lower=F)

hist(lrt.temp_filt$LRT, breaks=50)
hist(lrt.temp_filt$pvalue, breaks=50)
qqnorm(lrt.temp_filt$pvalue)

# #we also need to make sure we don't have any tricky values like those below
# lrt.temp_filt<-lrt.temp_filt[-c(which(lrt.temp_filt$pvalue == "NaN" ),
#                       which(lrt.temp_filt$pvalue == "Inf"),
#                       which(lrt.temp_filt$LRT == "Inf")),]
                      
# jpeg(filename = paste0("../integrated/gwas/by-temp/", temp, "-growth-condition-logp3-manhattan.jpeg"),
#      width = 900, height = 450, quality = 100)
manhattan(lrt.temp_filt %>% mutate(chr=as.numeric(as.factor(Chromosome))), 
          chr="chr", bp="Position", p="pvalue", genomewideline = 3, cex=1, suggestiveline = F,
          main=paste0("Putative markers associated with\n", trait.label, ", Temperature = ", temp), 
          highlight = (lrt.temp_filt %>% mutate(logp=-log10(pvalue)) %>% filter(logp>3))$SNP)
#dev.off()

lrt.temp_filt %>% mutate(logp=-log10(pvalue)) %>% filter(logp>3) %>% arrange(desc(LRT)) %>%
  unite("marker", Chromosome:Position, sep="_") %>% select(marker) %>% 
  write_delim(file=paste0("../integrated/gwas/", temp, "-gc1-markers-logp3.txt"),  col_names = F)

# did this already in previous code chunk 
# # Prepare gene regions with a 2000 bp flank
# genes4gwas <- pcod.gtf %>% mutate(ncbi_id=paste0("GeneID:", ncbi_id)) %>% 
#   filter(feature=="gene") %>% 
#   mutate(start_flank=start-2000) %>%
#   mutate(start_flank=as.integer(case_when(start_flank<0~0, TRUE~start_flank))) #%>% 
# 
# gene_ranges <- GRanges(
#   seqnames =genes4gwas$seqname,
#   ranges=IRanges(
#   start=genes4gwas$start_flank,
# #  start = genes4gwas$start,
#   end=genes4gwas$end),
#   ncbi_id=genes4gwas$ncbi_id,
#   gene_id=genes4gwas$gene_id)

# Prepare SNP sites
gc1.sites.temp <- lrt.temp_filt %>% mutate(logp=-log10(pvalue)) %>% filter(logp>3) %>% mutate(marker=paste0(Chromosome, "_", Position))

snp_ranges.temp <- GRanges(
  seqnames = gc1.sites.temp$Chromosome,
  ranges=IRanges(
    start = gc1.sites.temp$Position,
    end = gc1.sites.temp$Position),
  pvalue=gc1.sites.temp$pvalue,
  Major=gc1.sites.temp$Major,
  Minor=gc1.sites.temp$Minor)

# Find overlaps between gene flanks and SNP sites
overlaps.temp <- findOverlaps(gene_ranges, snp_ranges.temp)

# Extract matching rows
overlapping_genes.temp <- genes4gwas[queryHits(overlaps.temp), ]
overlapping_snps.temp <- gc1.sites.temp[subjectHits(overlaps.temp), ]

# Combine results into a single data frame
result.temp <- cbind(overlapping_genes.temp, overlapping_snps.temp) %>% 
  mutate(feature=case_when(Position>=start~"gene", Position<start~"upstream")) %>% 
  dplyr::select(-c(score, frame, attributes,biotype, exon)) %>% 
#  dplyr::select(feature, seqname, strand, start_flank, start, end, Position, gene_id, ncbi_id, gene_id) %>% 
  left_join(pcod.blast.GO[c("ncbi_id", "spid", "gene", "protein_names", "gene_ontology_biological_process")] %>% 
              mutate(ncbi_id=paste0("GeneID:", ncbi_id)), by="ncbi_id")

# result.temp %>% dplyr::select(-Position) %>% distinct() %>% group_by(feature) %>%  tally() %>% 
#   mutate(total=nrow(genes4gwas)) %>% mutate(perc=round(100*n/total, 2))

# Table of putative markers 
result.temp %>% dplyr::select(Chromosome, Position, LRT, pvalue, spid, protein_names) %>%
ungroup() %>% 
distinct() %>% arrange(desc(LRT)) %>% 
mutate(across(c(LRT:pvalue), ~ signif(.x, 3))) #%>% write_clip()

# NOTE: this beagle contains GLs for ALL treatments, and is missing some markers that were identified in the treatment-specific ANGSD run 
exp.beagle.gc1.sites.temp <- 
  exp.beagle %>% filter(marker %in% gc1.sites.temp$marker) %>% 
  pivot_longer(X100_1:X9_3, names_to = "individual.allele", values_to = "probability") %>% 
  separate(individual.allele, into=c("individual", "allele"), sep = "_") %>% 
  mutate(allele=case_when(
    allele==1 ~ paste0(allele1, "/", allele1),
    allele==2 ~ paste0(allele1, "/", allele2),
    allele==3 ~ paste0(allele2, "/", allele2))) %>% 
    mutate(allele_base = allele %>%
           str_replace_all("0", "A") %>%
           str_replace_all("1", "C") %>%
           str_replace_all("2", "G") %>%
           str_replace_all("3", "T")) %>% 
  mutate(probability=round(as.numeric(probability), digits=4)) %>% 
  group_by(marker, individual) %>% 
  filter(!(n() == 3 & all(probability == 0.3333))) %>%  
  slice_max(probability,n=1) %>% 
    mutate(individual=gsub("X", "", individual), allele_base) %>%
    left_join(phenotypes2, by=c("individual"="genetic_sampling_count")) %>%
mutate(temperature = replace_na(temperature, "16")) #%>% 
#  filter(temperature==temp)


gc1.markers.temp <- (gc1.sites.temp %>% filter(marker %in% exp.beagle$marker) %>% arrange(desc(LRT)))$marker
for (i in 1:length(gc1.markers.temp)) {
  meta <- exp.beagle.gc1.sites.temp %>% filter(marker==gc1.markers.temp[i]) %>% ungroup() %>%  
  select(marker) %>% distinct() %>%  
  left_join(result.temp %>% mutate(marker=paste0(Chromosome, "_", Position))) 
  
  print(exp.beagle.gc1.sites.temp %>% filter(marker==gc1.markers.temp[i]) %>% #filter(probability>0.75) %>% 
#  filter(!is.na(allele_base)) %>% droplevels() %>% 
    ggplot() + geom_boxplot(aes(x=allele_base, 
                                y=!!sym(trait),
                                fill=temperature), alpha=0.7, outlier.shape = NA) + 
    geom_point(aes(x=allele_base, 
                   y=!!sym(trait), 
                   color=point), position=position_jitter(w = 0.2,h = 0)) + 
  theme_minimal() + facet_wrap(~temperature, scales="free_y", nrow = 2, ncol=2) +
  scale_fill_manual(name="temperature", 
                   values=c("0"="#2c7bb6", "5"="#abd9e9", "9"="#fdae61", "16"="#d7191c"),
                   labels=c("0"="0degC","5"="5degC","9"="9degC (optimal)","16"="16degC")) +
  scale_color_manual(name=NULL, values=c("No.eGOA"="red", "No.wGOA"="black", "Yes.wGOA"="green2"), 
                     labels=c("No.wGOA"="Survived",  "No.eGOA"="eGOA cluster in PCA", "Yes.wGOA"="Died")) + 
  ggtitle(paste0("Marker: ", meta$marker, "\n Gene ID: ", meta$gene_id, "\n Protein: ", meta$protein_names)))
}
```

## Putative markers associated with growth rate (standard length) in warm-exposed fish

![image](https://github.com/user-attachments/assets/b4156d7a-bb56-4fd0-a7c6-d38d453ee31b)

| Chromosome  | Position | LRT  | pvalue   | spid   | protein_names                                                                     |
|-------------|----------|------|----------|--------|-----------------------------------------------------------------------------------|
| Chr22 | 2266449  | 13.5 | 0.000238 | P12107 | Collagen alpha-1(XI) chain                                                        |
| Chr6 | 16384831 | 13.1 | 0.000295 | Q6XJU9 | Osteoclast-stimulating factor 1                                                   |
| Chr22 | 2266770  | 12.9 | 0.000332 | P12107 | Collagen alpha-1(XI) chain                                                        |
| Chr4 | 29206268 | 12.6 | 0.000383 | Q5DU56 | Protein NLRC3                                                                     |
| Chr22 | 2266677  | 12.5 | 0.000399 | P12107 | Collagen alpha-1(XI) chain                                                        |
| Chr4 | 33251932 | 12.5 | 0.000415 | Q5R5U3 | Zinc finger protein 271                                                           |
| Chr10 | 14094944 | 11.8 | 0.000577 | Q91Y13 | Protocadherin alpha-7 (PCDH-alpha-7)                                              |
| Chr12 | 25112546 | 11.3 | 0.000767 | P55199 | RNA polymerase II elongation factor ELL                                           |
| Chr23 | 8966730  | 11.3 | 0.000786 | Q6P5D8 | Structural maintenance of chromosomes flexible hinge domain-containing protein 1  |
| Chr20 | 18401423 | 11.1 | 0.000871 | Q6GMI9 | UDP-glucuronic acid decarboxylase 1                                               |
| Chr7 | 8158249  | 10.8 | 0.000995 | O43264 | Centromere/kinetochore protein zw10 homolog                                       |

![image](https://github.com/user-attachments/assets/78325c1b-1e5c-4379-873d-4ab2b55e3ed2)
![image](https://github.com/user-attachments/assets/93a53bad-a5be-42c3-86da-595af1fe0f41)
![image](https://github.com/user-attachments/assets/e31eb5ca-4971-461c-9623-110a5c56f440)
![image](https://github.com/user-attachments/assets/4c44dcbd-9984-4038-a1ba-9c2e6066e0d5)
![image](https://github.com/user-attachments/assets/32cdf2de-4189-4b6d-a97a-576af0ff86eb)

## Putative markers associated with growth rate (wet weight) in warm-exposed fish

![image](https://github.com/user-attachments/assets/768f2679-dff8-42c9-8804-f067dd5cd701)

| Chromosome  | Position | LRT  | pvalue   | spid   | protein_names                                                                      |
|-------------|----------|------|----------|--------|------------------------------------------------------------------------------------|
| Chr20 | 18401423 | 19.8 | 8.71E-06 | Q6GMI9 | UDP-glucuronic acid decarboxylase 1                                                |
| Chr22 | 2266449  | 18.5 | 1.71E-05 | P12107 | Collagen alpha-1(XI) chain                                                         |
| Chr22 | 2266770  | 17.3 | 3.24E-05 | P12107 | Collagen alpha-1(XI) chain                                                         |
| Chr22 | 2266677  | 16.6 | 4.65E-05 | P12107 | Collagen alpha-1(XI) chain                                                         |
| Chr22 | 2871726  | 13.1 | 0.00029  | Q95460 | Major histocompatibility complex class I-related gene protein                      |
| Chr4 | 29206268 | 12.7 | 0.000369 | Q5DU56 | Protein NLRC3                                                                      |
| Chr10 | 13841648 | 12.6 | 0.000383 | Q5DRB8 | Protocadherin gamma-A2                                                             |
| Chr10 | 13841648 | 12.6 | 0.000383 | Q9Y5G7 | Protocadherin gamma-A6                                                             |
| Chr7 | 8158249  | 12.2 | 0.000479 | O43264 | Centromere/kinetochore protein zw10 homolog                                        |
| Chr1 | 10698321 | 12.2 | 0.000485 | Q6RY07 | Acidic mammalian chitinase (AMCase) (EC 3.2.1.14)                                  |
| Chr5 | 20744974 | 12.1 | 0.000499 | Q7RTR2 | NLR family CARD domain-containing protein 3                                        |
| Chr8 | 6194665  | 12   | 0.000543 | Q8R4G8 | BTB/POZ domain-containing protein KCTD1 (Vitamin A-deficient testicular protein 6) |
| Chr22 | 2871709  | 12   | 0.000545 | Q95460 | Major histocompatibility complex class I-related gene protein                      |
| Chr9 | 5952237  | 11.5 | 0.000685 | Q8AXZ4 | Contactin-1a (F3/F11/contactin) (Neural cell recognition molecule F11)             |
| Chr7 | 1643219  | 11.5 | 0.000709 | P59046 | NACHT, LRR and PYD domains-containing protein 12 (Monarch-1)                       |
| Chr7 | 1643219  | 11.5 | 0.000709 | Q7RTR2 | NLR family CARD domain-containing protein 3 (CARD15-like protein)                  |
| Chr15 | 14084438 | 11.4 | 0.000716 | Q7LFX5 | Carbohydrate sulfotransferase 15                                                   |

![image](https://github.com/user-attachments/assets/7fd53501-f1f8-455f-9bfd-9f94dbdb862d)
![image](https://github.com/user-attachments/assets/d7be7278-d384-409f-8bbf-62aec27ec02f)
![image](https://github.com/user-attachments/assets/e7a9e371-2b8f-4ef2-875f-51004867e7e6)
![image](https://github.com/user-attachments/assets/58ede850-9d87-4a79-873a-d079f7a29e6f)
![image](https://github.com/user-attachments/assets/f0738f4c-8d44-4b24-8dea-07d37b3b759f)
![image](https://github.com/user-attachments/assets/ebe041f8-3edf-46cd-b4f6-26e7ce175daf)

## Putative markers associated with hepatosomatic index in warm-exposed fish

![image](https://github.com/user-attachments/assets/c16737b7-adab-41a4-879c-1540ea547645)

| Chromosome  | Position | LRT  | pvalue   | spid   | protein_names                                                            |
|-------------|----------|------|----------|--------|--------------------------------------------------------------------------|
| Chr10 | 26512    | 24.5 | 7.50E-07 | Q5SVR0 | TBC1 domain family member 9B                                             |
| Chr8 | 7686394  | 20.6 | 5.69E-06 | Q90XF2 | Protein kinase C iota type                                               |
| Chr12 | 8663963  | 20.3 | 6.69E-06 | P15924 | Desmoplakin (DP)                                                         |
| Chr12 | 8663139  | 20.1 | 7.20E-06 | P15924 | Desmoplakin (DP)                                                         |
| Chr8 | 7684131  | 19.6 | 9.48E-06 | Q90XF2 | Protein kinase C iota type                                               |
| Chr8 | 7688322  | 18.6 | 1.62E-05 | Q90XF2 | Protein kinase C iota type                                               |
| Chr15 | 5279980  | 17.2 | 3.41E-05 | Q5RC21 | Translin-associated protein X                                            |
| Chr15 | 5279992  | 17.1 | 3.54E-05 | Q5RC21 | Translin-associated protein X                                            |
| Chr10 | 26590    | 16.5 | 4.85E-05 | Q5SVR0 | TBC1 domain family member 9B                                             |
| Chr18 | 18184067 | 15.2 | 9.66E-05 | Q9ULK0 | Glutamate receptor ionotropic, delta-1                                   |
| Chr7 | 24047591 | 14.6 | 0.000134 | Q2WG80 | Protein ripply1                                                          |
| Chr7 | 24047555 | 14.5 | 0.000138 | Q2WG80 | Protein ripply1                                                          |
| Chr7 | 23954622 | 14.4 | 0.000151 | O88491 | Histone-lysine N-methyltransferase, H3 lysine-36 specific                |
| Chr12 | 8664098  | 14.2 | 0.00016  | P15924 | Desmoplakin (DP)                                                         |
| Chr14 | 6110294  | 14.2 | 0.000161 | Q7SXF1 | 7-dehydrocholesterol reductase                                           |
| Chr12 | 8666427  | 14.1 | 0.000171 | P15924 | Desmoplakin (DP)                                                         |
| Chr8 | 7688395  | 14   | 0.000185 | Q90XF2 | Protein kinase C iota type                                               |
| Chr7 | 22172083 | 12.9 | 0.000325 | Q6ZMW2 | Zinc finger protein 782                                                  |
| Chr7 | 11635733 | 12.5 | 0.000403 | Q3U3V8 | X-ray radiation resistance-associated protein 1                          |
| Chr8 | 7688446  | 12.3 | 0.00046  | Q90XF2 | Protein kinase C iota type                                               |
| Chr4 | 9882897  | 12.1 | 0.000498 | Q3USH5 | Splicing factor, suppressor of white-apricot homolog                     |
| Chr3 | 16934535 | 11.8 | 0.00058  | A2Q0R8 | Small ribosomal subunit protein eS1                                      |
| Chr3 | 16934535 | 11.8 | 0.00058  | Q5HYK7 | SH3 domain-containing protein 19                                         |
| Chr1 | 6185889  | 11.8 | 0.000581 | Q86YJ5 | E3 ubiquitin-protein ligase MARCHF9                                      |
| Chr12 | 6825238  | 11.8 | 0.00059  | Q9QYW0 | Protein AATF (Apoptosis-antagonizing transcription factor)               |
| Chr9 | 20279221 | 11.7 | 0.000632 | A1L271 | Guanine nucleotide-binding protein subunit beta-5a                       |
| Chr3 | 16934470 | 11.7 | 0.000642 | A2Q0R8 | Small ribosomal subunit protein eS1                                      |
| Chr3 | 16934470 | 11.7 | 0.000642 | Q5HYK7 | SH3 domain-containing protein 19                                         |
| Chr18 | 18184044 | 11.6 | 0.000674 | Q9ULK0 | Glutamate receptor ionotropic, delta-1                                   |
| Chr7 | 22188972 | 11.6 | 0.000676 | Q6ZMW2 | Zinc finger protein 782                                                  |
| Chr7 | 22182763 | 11.5 | 0.000684 | Q6ZMW2 | Zinc finger protein 782                                                  |
| Chr7 | 11635676 | 11.5 | 0.000709 | Q3U3V8 | X-ray radiation resistance-associated protein 1                          |
| Chr7 | 22174547 | 11.3 | 0.000771 | Q6ZMW2 | Zinc finger protein 782                                                  |
| Chr9 | 20279076 | 11.2 | 0.000829 | A1L271 | Guanine nucleotide-binding protein subunit beta-5a                       |
| Chr14 | 19635364 | 11   | 0.000893 | Q6DJE5 | Dysbindin domain-containing protein 1                                    |
| Chr14 | 19635364 | 11   | 0.000893 | Q0P464 | Methenyltetrahydrofolate synthase domain-containing protein              |
| Chr14 | 19635364 | 11   | 0.000893 | O95995 | Dynein regulatory complex subunit 4 (Growth arrest-specific protein 11)  |
| Chr18 | 18193173 | 10.9 | 0.000938 | Q9ULK0 | Glutamate receptor ionotropic, delta-1                                   |
| Chr2 | 21411415 | 10.9 | 0.000968 | Q7Z410 | Transmembrane protease serine 9                                          |
| Chr7 | 22186315 | 10.9 | 0.00097  | Q6ZMW2 | Zinc finger protein 782                                                  |
| Chr18 | 18185188 | 10.8 | 0.000992 | Q9ULK0 | Glutamate receptor ionotropic, delta-1                                   |

![image](https://github.com/user-attachments/assets/60408764-08bb-4270-ab18-770da1fc71cc)
![image](https://github.com/user-attachments/assets/8326d8b3-8030-4565-aa35-c7b1c9a4b949)
![image](https://github.com/user-attachments/assets/8ebba0fd-f78c-4663-b8ee-337aad45c880)
![image](https://github.com/user-attachments/assets/b213a1da-61fb-4fbc-8b1e-157f3e29aecd)
![image](https://github.com/user-attachments/assets/47177ff6-a270-4d71-b850-e99d044a2505)
![image](https://github.com/user-attachments/assets/82135421-ce12-4951-ad4b-a40c92e8fba7)
![image](https://github.com/user-attachments/assets/0ae06317-0e68-4dd7-a2ba-a89df11776c2)
![image](https://github.com/user-attachments/assets/4b137392-4026-4e76-8bc4-091d9a9737ad)
![image](https://github.com/user-attachments/assets/10bad86f-1e86-4dd8-9ae0-0fcfa6f7814e)
![image](https://github.com/user-attachments/assets/1ed24783-04c5-4973-8bef-90c8920193e7)
![image](https://github.com/user-attachments/assets/84a42aa6-5729-478e-ba48-e946fe751dc8)
![image](https://github.com/user-attachments/assets/ef7e5769-d62c-44a9-911b-b2e33ba5af72)


## Working on pcod genotype imputation pipeline 
I previously performed genotype imputation without any reference panel/map. I did some reading, and it looks like I could greatly improve imputation accuracy if I provide phased haplotype reference panel, built from other Pacific cod WGS data. I happen to have a lcWGS data from  ~600 reference fish (depth ~3x) AND the big/little fish from 2021 (depth ~14x). I'm going to see if I can use those datasets to build the phased reference panel. 

_NOTE: phased reference panel: genetic variants are assigned to their respective chromosomes, distinguishing which alleles came from one parent versus the other._

NOTE: I tried to use a beagle file (merged to have both ref & big/little data) as a reference panel, but no dice. I need a VCF file, so I'll build that from "scratch" using angsd's doVCF option 

### Run ANGSD to generate VCF with genotype probabilities from reference population and Big/Little 2021 cod 

I'm working in this repon on Sedna:  /home/lspencer/pcod-general/imputation-ref-panel, and ran the below code in the script **angsd-impute-ref-panel.sh.** It produced BCF files for each chromosome. 

```
#!/bin/bash

#SBATCH --cpus-per-task=10
#SBATCH --time=0-20:00:00
#SBATCH --job-name=angsd
#SBATCH --output=/home/lspencer/pcod-general/imputation-ref-panel/angsd-ref-panel_%A-%a.out
#SBATCH --mail-type=ALL
#SBATCH --mail-user=lspencer@noaa.gov
#SBATCH --array=1-24%24

module unload bio/angsd/0.940
module load bio/angsd/0.940

JOBS_FILE=/home/lspencer/pcod-general/imputation-ref-panel/angsdARRAY_input.txt
IDS=$(cat ${JOBS_FILE})

for sample_line in ${IDS}
do
        job_index=$(echo ${sample_line} | awk -F ":" '{print $1}')
        contig=$(echo ${sample_line} | awk -F ":" '{print $2}')
        if [[ ${SLURM_ARRAY_TASK_ID} == ${job_index} ]]; then
                break
        fi
done

angsd -b /home/lspencer/pcod-general/imputation-ref-panel/bamslist-ref-panel.txt -ref /home/lspencer/references/pcod-ncbi/GCF_031168955.1_ASM3116895v1_genomic.fa \
-r ${contig}: -out /home/lspencer/pcod-general/imputation-ref-panel/panel_${contig}_polymorphic \
-nThreads 10 -uniqueOnly 1 -doCounts 1 -setMinDepth 700 -remove_bads 1 -trim 0 -C 50 -minMapQ 35 -minQ 30 \
-dobcf 1 -doGlf 2 -GL 1 -doMaf 1 -doMajorMinor 1 -dopost 1 -dogeno 32 \
-minMaf 0.05 -SNP_pval 1e-10 -only_proper_pairs 1
(base) [lspencer@node28 imputation-ref-panel]$ pwd
/home/lspencer/pcod-general/imputation-ref-panel
```
I then concatenated the BCF files into one using `bcftools concat`, then converted to VCF, filtered for minor allele frequency > 5% and max missingness 15%. This produced the file **whole-genome_miss15.vcf.gz**.  There are still some missing genotypes, so here I try to impute those using BEAGLE v5.4:  

```
java -jar /home/lspencer/programs/beagle.v5.4.jar gt=/home/lspencer/pcod-general/imputation-ref-panel/whole-genome_miss15.vcf.gz gp=true impute=true out=/home/lspencer/pcod-general/imputed

beagle.29Oct24.c8e.jar (version 5.4)
Copyright (C) 2014-2022 Brian L. Browning
Enter "java -jar beagle.29Oct24.c8e.jar" to list command line argument
Start time: 11:17 AM PST on 25 Nov 2024

Command line: java -Xmx1136m -jar beagle.29Oct24.c8e.jar
  gt=/home/lspencer/pcod-general/imputation-ref-panel/whole-genome_miss15.vcf.gz
  gp=true
  impute=true
  out=/home/lspencer/pcod-general/imputed
  nthreads=1

No genetic map is specified: using 1 cM = 1 Mb

Reference samples:                    0
Study     samples:                  708

Window 1 [NC_036931.1:47-16349]
Study     markers:                   84

Burnin  iteration 1:           5 seconds
Burnin  iteration 2:           1 second
Burnin  iteration 3:           1 second

Estimated ne:                  5216323
Estimated err:                 6.4e-03

Phasing iteration 1:           0 seconds
Phasing iteration 2:           0 seconds
Phasing iteration 3:           0 seconds
Phasing iteration 4:           0 seconds
Phasing iteration 5:           0 seconds
Phasing iteration 6:           0 seconds
Phasing iteration 7:           0 seconds
Phasing iteration 8:           0 seconds
Phasing iteration 9:           0 seconds
Phasing iteration 10:          0 seconds
Phasing iteration 11:          0 seconds
Phasing iteration 12:          0 seconds

Window 2 [NC_082382.1:8368-26260017]
Study     markers:               17,298

...
```

yada yada yada 


This should produce a VCF file with phased haplotypes, and a log file. I should next assess the phasing quality using tools like SHAPEIT's quality checks or Beagle's output metrics.
