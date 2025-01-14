---
layout: post
title: Refining GWAS 
---

I refined the GWAS pipeline to include separate training and test sample sets, to: 
   1. Examine multiple sets of markers based on various log10(pvalue) threshods (2-4.75), and select the best set based on the correlation between predicted trait value (e.g. HSI) v.s actual trait values. Here are figures showing results from the HSI markers.  

![16-hsi-logp2-manhattan](https://github.com/user-attachments/assets/8f0954a4-3fb1-4005-9e9a-2871d9897040)

### Genes with expression patterns also associated with performance in warming 
I previously used WGCNA to identify genes with expression patterns that are associated with trait performance. Here are genes that were identified in that analysis AND in the above GWAS analysis to be associated with HSI. In bold are genes that are not only assigned to a HSI-correlated coexpressed gene module, but expression values are also correlated with HSI on their own.  

| Relationship with performance | Chromosome  | Gene ID (NCBI) | Uniprot SPID | Protein                                                                                                                                                                                                         |
|-------------------------------|-------------|----------------|--------------|-----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| Increasing Expression         | NC_082382.1 | LOC132453053   | Q91453       | Stonustoxin subunit beta (SNTX subunit beta) (DELTA-synanceitoxin-Sh1b) (DELTA-SYTX-Sh1b) (Trachynilysin subunit beta) (TLY subunit beta)                                                                       |
| **Increasing Expression**         | **NC_082383.1** | **LOC132456135**   | **Q6UXZ4**       | **Netrin receptor UNC5D (Protein unc-5 homolog 4) (Protein unc-5 homolog D)**                                                                                                                                       |
| **Increasing Expression**         | **NC_082384.1** | **LOC132457513**   | **Q91453**       | **Stonustoxin subunit beta (SNTX subunit beta) (DELTA-synanceitoxin-Sh1b) (DELTA-SYTX-Sh1b)** (Trachynilysin subunit beta) (TLY subunit beta)                                                                       |
| Increasing Expression         | NC_082390.1 | LOC132445594   |              |                                                                                                                                                                                                                 |
| Increasing Expression         | NC_082390.1 | LOC132452628   | Q7RTR2       | NLR family CARD domain-containing protein 3 (CARD15-like protein) (Caterpiller protein 16.2) (CLR16.2) (NACHT, LRR and CARD domains-containing protein 3) (Nucleotide-binding oligomerization domain protein 3) |
| Increasing Expression         | NC_082390.1 | LOC132452644   | Q9UM44       | HERV-H LTR-associating protein 2 (Human endogenous retrovirus-H long terminal repeat-associating protein 2)                                                                                                     |
| Decreasing Expression         | NC_082385.1 | LOC132466560   | Q5SVR0       | TBC1 domain family member 9B                                                                                                                                                                                    |
| Decreasing Expression         | NC_082386.1 | tmco1          | Q6DGW9       | Calcium load-activated calcium channel (CLAC channel) (GEL complex subunit TMCO1) (Transmembrane and coiled-coil domain-containing protein 1)                                                                   |
| Decreasing Expression         | NC_082387.1 | tmco1          | Q6DGW9       | Calcium load-activated calcium channel (CLAC channel) (GEL complex subunit TMCO1) (Transmembrane and coiled-coil domain-containing protein 1)                                                                   |

### Identifying best set of markers for predicting trait value (here HSIU) 

I pulled markers with log10(pvalue) = 2 - 4 at 0.25 increments, extracted genotype probabilities (imputed) for those n sites, then generated covariance matrices and performed PCA for each. I did this with ALL samples, not just training samples. This allowed me to assess agreements among predicted and actual trait values using modeled  trait~PC1 scores relationships. The marker set that had the highest agreement in training and test sets were deemed the "best" set.  

![image](https://github.com/user-attachments/assets/c1b12377-d58f-40fd-b733-6996907f0966)
![image](https://github.com/user-attachments/assets/350fa940-5cae-4013-a594-e38312f8851d)
![image](https://github.com/user-attachments/assets/3ad435c4-c236-4d3d-96d0-a3e2d82753e3)

### Here is the "final" list of HSI markers (n=39) 


| Chromosome  | Gene             | Number Markers in gene | Protein                                                                                                                                                                                                         | Mean Log10(P) | Uniqprot SPID |
|-------------|------------------|----------------|-----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|---------------|---------------|
| NC_082384.1 | LOC132446484     | 3              | Glutamate receptor ionotropic, delta-1 (GluD1) (GluR delta-1 subunit)                                                                                                                                           | 2.31106674    | Q9ULK0        |
| NC_082383.1 | LOC132463408     | 2              | Solute carrier family 25 member 36-A                                                                                                                                                                            | 2.396336651   | Q6DG32        |
| NC_082383.1 | tmco1            | 2              | Calcium load-activated calcium channel (CLAC channel) (GEL complex subunit TMCO1) (Transmembrane and coiled-coil domain-containing protein 1)                                                                   | 2.18904135    | Q6DGW9        |
| NC_082384.1 | aatf             | 2              | Protein AATF (Apoptosis-antagonizing transcription factor)                                                                                                                                                      | 3.007019146   | Q9QYW0        |
| NC_082385.1 | LOC132446484     | 2              | Glutamate receptor ionotropic, delta-1 (GluD1) (GluR delta-1 subunit)                                                                                                                                           | 2.010495313   | Q9ULK0        |
| NC_082385.1 | LOC132448018     | 2              | Astrotactin-2                                                                                                                                                                                                   | 2.613209224   | O75129        |
| NC_082385.1 | LOC132452589     | 2              | Beta-1,4 N-acetylgalactosaminyltransferase 2 (EC 2.4.1.-) (Sd(a) beta-1,4-GalNAc transferase) (UDP-GalNAc:Neu5Aca2-3Galb-R b1,4-N-acetylgalactosaminyltransferase)                                              | 2.436358457   | Q8NHY0        |
| NC_082385.1 | si:dkey-247m21.3 | 2              | 5-hydroxytryptamine receptor 4 (5-HT-4) (5-HT4) (Serotonin receptor 4)                                                                                                                                          | 2.239607094   | O70528        |
| NC_082382.1 | LOC132456135     | 1              | Netrin receptor UNC5D (Protein unc-5 homolog 4) (Protein unc-5 homolog D)                                                                                                                                       | 2.075278578   | Q6UXZ4        |
| NC_082382.1 | LOC132457233     | 1              | NLR family CARD domain-containing protein 3 (CARD15-like protein) (Caterpiller protein 16.2) (CLR16.2) (NACHT, LRR and CARD domains-containing protein 3) (Nucleotide-binding oligomerization domain protein 3) | 3.223533315   | Q7RTR2        |
| NC_082382.1 | LOC132461163     | 1              |                                                                                                                                                                                                                 | 3.428210114   |               |
| NC_082382.1 | LOC132461286     | 1              |                                                                                                                                                                                                                 | 3.428210114   |               |
| NC_082382.1 | LOC132463722     | 1              |                                                                                                                                                                                                                 | 2.786885914   |               |
| NC_082382.1 | LOC132463724     | 1              |                                                                                                                                                                                                                 | 2.786885914   |               |
| NC_082382.1 | LOC132463725     | 1              |                                                                                                                                                                                                                 | 2.786885914   |               |
| NC_082382.1 | acadm            | 1              | Medium-chain specific acyl-CoA dehydrogenase, mitochondrial (MCAD) (EC 1.3.8.7)                                                                                                                                 | 2.786885914   | Q3SZB4        |
| NC_082382.1 | cacna1hb         | 1              | Voltage-dependent T-type calcium channel subunit alpha-1I (Voltage-gated calcium channel subunit alpha Cav3.3) (Ca(v)3.3)                                                                                       | 2.136198402   | Q9P0X4        |
| NC_082383.1 | LOC132463722     | 1              |                                                                                                                                                                                                                 | 2.363614851   |               |
| NC_082383.1 | LOC132463724     | 1              |                                                                                                                                                                                                                 | 2.363614851   |               |
| NC_082383.1 | LOC132463725     | 1              |                                                                                                                                                                                                                 | 2.363614851   |               |
| NC_082383.1 | LOC132463729     | 1              |                                                                                                                                                                                                                 | 2.363614851   |               |
| NC_082383.1 | LOC132466560     | 1              | TBC1 domain family member 9B                                                                                                                                                                                    | 2.238526242   | Q5SVR0        |
| NC_082383.1 | bbs9             | 1              | Protein PTHB1 (Bardet-Biedl syndrome 9 protein homolog) (Parathyroid hormone-responsive B1 gene protein homolog)                                                                                                | 2.359376133   | Q6AX60        |
| NC_082383.1 | trnap-agg_74     | 1              |                                                                                                                                                                                                                 | 2.359376133   |               |
| NC_082383.1 | trnap-agg_75     | 1              |                                                                                                                                                                                                                 | 2.359376133   |               |
| NC_082383.1 | uck2a            | 1              | Uridine-cytidine kinase 2-A (UCK 2-A) (EC 2.7.1.48) (Cytidine monophosphokinase 2-A) (Uridine monophosphokinase 2-A)                                                                                            | 2.198464599   | Q7SYM0        |
| NC_082384.1 | LOC132469993     | 1              | Desmoplakin (DP) (250/210 kDa paraneoplastic pemphigus antigen)                                                                                                                                                 | 2.595272889   | P15924        |
| NC_082384.1 | cpt2             | 1              | Carnitine O-palmitoyltransferase 2, mitochondrial (EC 2.3.1.21) (Carnitine palmitoyltransferase II) (CPT II)                                                                                                    | 2.737512152   | Q5U3U3        |
| NC_082385.1 | LOC132446534     | 1              |                                                                                                                                                                                                                 | 2.010063808   |               |
| NC_082385.1 | LOC132451248     | 1              |                                                                                                                                                                                                                 | 2.445910261   |               |


### Performance of PC1 in predicting each trait in training samples (75%), test samples (25%), and all samples 

In general, I'm a bit bummed to see that the correlations among predicted and actual trait values are not higher in our test sample sets - here's a breakdown of how each gene set (based on min. log10(pvalue) did in predicting traits: 
![image](https://github.com/user-attachments/assets/909b5a6e-d860-4bcf-a0f1-f84a6f612953)

## Next step: Visualize genotype-dependent trait performance along with expression! 
