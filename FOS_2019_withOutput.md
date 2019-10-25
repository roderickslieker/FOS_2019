Epigenomics Practical 2019
================
Roderick Slieker and Bas Heijmans

## Important:

  - Download the following files and save them in one folder:

[Tissue data subset.csv](Tissue%20data%20subset.csv)

[Tissue data chromosome 22.csv](Tissue%20data%20chromosome%2022.csv)

[Tissue data chromosome 22
significant.csv](Tissue%20data%20chromosome%2022%20significant.csv)

[Significant probes.bed](Significant%20probes.bed)

[TissueData chr22.RData](TissueData%20chr22.RData)

  - At the end of the practical send your work to
    **<r.c.slieker@lumc.nl>** as Word document with your name(s) on top
    of the first page. The file name should contain your surname(s).

## Introduction

**A primary function of DNA methylation, a key component of the
epigenome, is to control cell differentiation. In this practical, you
will identify and annotate differential methylation between tissues
using genome-wide data generated with Illumina 450k chips. The coming
few hours you will replicate many of our groups’ findings which were
published in 2013. Good luck and have fun\!**

Slieker et al. Identification and systematic annotation of
tissue-specific differentially methylated regions using the Illumina
450k array. Epigenetics & Chromatin volume 6

<https://epigeneticsandchromatin.biomedcentral.com/articles/10.1186/1756-8935-6-26>

## Question 1

**The data that you will use in this practical consists of genome-wide
DNA methylation data (Illumina 450k methylation array) of tissues from
autopsy samples. Read the file ‘Tissue data subset.csv’ in R. The file
contains the DNA methylation data of a single CpG site with identifier
number cg21507095.**

``` r
library(ggplot2)
#Change this to the directory where you saved the files
setwd("Z:/Roderick/013 Onderwijs/2019_FOS")
TissueData <- read.table("Tissue data subset.csv", sep=";", header=T)
```

**A Which tissues were studied? And on how many cases?**

``` r
head(TissueData) # First 6 lines of the data
```

    ##   Tissue cg21507095
    ## 1  Blood 0.08609352
    ## 2  Blood 0.08317853
    ## 3  Blood 0.09087435
    ## 4  Blood 0.07773573
    ## 5  Blood 0.08468252
    ## 6  Blood 0.08240142

``` r
str(TissueData) # Characteristics of the data
```

    ## 'data.frame':    18 obs. of  2 variables:
    ##  $ Tissue    : Factor w/ 3 levels "Blood","Muscle",..: 1 1 1 1 1 1 3 3 3 3 ...
    ##  $ cg21507095: num  0.0861 0.0832 0.0909 0.0777 0.0847 ...

``` r
unique(TissueData$Tissue) # What groups are in the vector?
```

    ## [1] Blood  SC Fat Muscle
    ## Levels: Blood Muscle SC Fat

**B Make a boxplot of DNA methylation in the different tissues. Also,
calculate the mean methylation and standard deviation for the tissues.
Do you think that these differences are biologically relevant?**

``` r
ggplot(TissueData, aes(x=Tissue, y=cg21507095, color=Tissue))+ # Assign variables to x, y, and fill
  stat_summary(fun.data = "mean_cl_boot",  size = 1)+
  geom_jitter(width=0.2)+
  scale_colour_manual(values=c("#009AC7","#132B41","#F9A23F"))+ # Give custom colors to the boxes using HEX colors
  ylim(0,1)
```

![](FOS_2019_withOutput_files/figure-gfm/Q1B-1.png)<!-- -->

``` r
#Mean and SD
# Data = data to calculate statistic over, INDICES = grouping variable, FUN = function to be used
by(data = TissueData$cg21507095, INDICES = TissueData$Tissue, FUN=mean)
```

    ## TissueData$Tissue: Blood
    ## [1] 0.08416101
    ## -------------------------------------------------------- 
    ## TissueData$Tissue: Muscle
    ## [1] 0.3756033
    ## -------------------------------------------------------- 
    ## TissueData$Tissue: SC Fat
    ## [1] 0.3596354

``` r
by(data = TissueData$cg21507095, INDICES = TissueData$Tissue, FUN=sd)
```

    ## TissueData$Tissue: Blood
    ## [1] 0.0043456
    ## -------------------------------------------------------- 
    ## TissueData$Tissue: Muscle
    ## [1] 0.1596221
    ## -------------------------------------------------------- 
    ## TissueData$Tissue: SC Fat
    ## [1] 0.04488659

**C Test whether the difference is statistically significant using
ANOVA. Is the difference between tissues statistically significant?**

``` r
#Run the anova for one CpG
anova(lm(cg21507095~Tissue, data=TissueData)) 
```

    ## Analysis of Variance Table
    ## 
    ## Response: cg21507095
    ##           Df  Sum Sq  Mean Sq F value    Pr(>F)    
    ## Tissue     2 0.32216 0.161080  17.564 0.0001175 ***
    ## Residuals 15 0.13756 0.009171                      
    ## ---
    ## Signif. codes:  0 '***' 0.001 '**' 0.01 '*' 0.05 '.' 0.1 ' ' 1

**D Now, use the UCSC Genome Browser to annotate the CpG site. Go to
<http://genome.ucsc.edu/>, click Genomes (Assembly: Feb 2009
(GRCh37/hg19)) and look up the position of the CpG site (type the CpG
identifier in the search term box). Then scroll down to Regulation, find
CpG islands, set this feature to ‘show’ and click refresh on the right
of the screen. Describe in your own words what CpG islands do and how
they are defined ( right click on the CpG island track, Show details).**

**E Go back to the genome browser. Scroll up and use ‘zoom out’ to find
out: (1) in which gene the CpG is located, (2) what part of the gene it
is located (intron, exon, etc.) and (3) whether it is located in a CpG
island.**

**F Navigate to the EWAS datahub
(<https://bigd.big.ac.cn/ewas/datahub>), search for cg21507095. Click on
tissue and compare/validate your findings.**

![Tissue](EWAS_Datahub_cg21507095_tissue.png)

**G Navigate to other diseases. From the drop down menu choose type 2
diabetes. Is there a difference? Is it significant?**

![Diabetes](cg21507095.png)

**H TBX1 has been suggested to play a role in obesity. Go to the BMI tab
look at the different tissues. What do you conclude?**

![BMI](dna-methylation-changes.png)

## Question 2

**You will investigate DNA methylation between tissues across a complete
chromosome instead of a single CpG site. Load the file Tissue data
chromosome 22.csv with DNA methylation data for chromosome 22 across
three tissues.**

**A How many CpGs were measured on chromosome 22?**

``` r
TissueData22 <- read.table("Tissue data chromosome 22.csv", sep=";", header=T)
head(TissueData22)
```

    ##           ID chromosome Start_coord End_coord     Blood    SC_Fat
    ## 1 cg00004192      chr22    21398553  21398553 0.8338528 0.8560712
    ## 2 cg00004775      chr22    46516503  46516503 0.8192848 0.6842920
    ## 3 cg00012194      chr22    50720288  50720288 0.7732876 0.8094902
    ## 4 cg00013618      chr22    22380839  22380839 0.8335602 0.8724480
    ## 5 cg00014104      chr22    22220367  22220367 0.6437426 0.5235496
    ## 6 cg00014733      chr22    29730659  29730659 0.8847488 0.8671367
    ##      Muscle      Gene_centric CGI_centric        Pval Pval_corrected
    ## 1 0.8355123 Proximal promoter   CGI Shore 0.146280491              0
    ## 2 0.6283724        Intergenic   CGI Shore 0.000002670              1
    ## 3 0.8126704         Gene body CpG islands 0.029556487              0
    ## 4 0.9104521        Intergenic     Non CGI 0.001582952              0
    ## 5 0.6115632         Gene body   CGI Shore 0.008324831              0
    ## 6 0.8350884 Proximal promoter     Non CGI 0.006858962              0

``` r
nrow(TissueData22) #Number of rows
```

    ## [1] 8463

**B Make a histogram for DNA methylation data for blood. Inspect the
histogram and describe the characteristics of DNA methylation in the
human genome (note that the histogram will look virtually identical for
other chromosomes and tissues).**

``` r
ggplot(TissueData22, aes(x=Blood))+
  geom_histogram(bins = 200)
```

![](FOS_2019_withOutput_files/figure-gfm/Q2B-1.png)<!-- -->

**C The genome can be categorized in different genomic features.
Gene-centric annotations divide the genome relative to gene
structure/function. One can also annotate CpGs based on the CpG density,
that is CpG islands and their surroundings. Use the annotation figure
below to re-answer 1E.**

![Gene- and CGI-centric annotation](Annotation.jpg)

**D The file Tissue data chromosome 22.csv contains columns with
annotation information. Make violin plots for the gene centric
annotation for blood. Describe the differences in DNA methylation across
annotations and explain whether this is according to your expectation
given your knowledge of the function of DNA methylation.**

``` r
ggplot(TissueData22, aes(x=Gene_centric, y=Blood, fill=Gene_centric))+
  geom_violin()+
  scale_fill_manual(values=c("#440154FF","#3B528BFF","#21908CFF","#5DC863FF","#FDE725FF"))+
  theme(axis.text.x = element_text(angle=90, hjust=1, vjust=0.5)) # Rotate axis labels
```

![](FOS_2019_withOutput_files/figure-gfm/Q2D-1.png)<!-- -->

**E Do the same for the CpG island annotation. Where is DNA methylation
lowest?**

``` r
ggplot(TissueData22, aes(x=CGI_centric, y=Blood, fill=CGI_centric))+
  geom_violin()+
  scale_fill_manual(values=c("#440154FF","#3B528BFF","#21908CFF","#5DC863FF","#FDE725FF"))+
  theme(axis.text.x = element_text(angle=90, hjust=1, vjust=0.5))
```

![](FOS_2019_withOutput_files/figure-gfm/Q2E-1.png)<!-- -->

**F In question Q1C we calculated the P-value for one CpG site. Now we
want to run this for each of the CpGs sites across chromosome 22.
Instead of saving data as csv, one can also save it as RData files. Load
the RData file TissueData chr22.RData. This file contains two files,
namely samplesheet and TissueDataChr22. Look at both files. What are the
dimensions of both files? **

``` r
a = load("TissueData chr22.RData") # Contains two files samplesheet, TissueDataChr22
a
```

    ## [1] "TissueDataChr22" "samplesheet"

``` r
head(samplesheet)
```

    ##   ID Tissue
    ## 1  9  Blood
    ## 2 13  Blood
    ## 3 10  Blood
    ## 4 15  Blood
    ## 5 16  Blood
    ## 6 17  Blood

``` r
dim(samplesheet) # Number of rows and number of columns
```

    ## [1] 18  2

``` r
head(TissueDataChr22)
```

    ##                   9-b       13-b       10-b       15-b       16-b
    ## cg00017461 0.05632318 0.05456034 0.05941103 0.05011028 0.05413667
    ## cg00077299 0.49782634 0.48542283 0.50410416 0.46139226 0.46376572
    ## cg00079563 0.03616431 0.05418437 0.04408411 0.03128675 0.04881740
    ## cg00087182 0.94933796 0.94282964 0.94996077 0.94773796 0.94644079
    ## cg00093544 0.16849439 0.26642779 0.13488553 0.33511312 0.26710418
    ## cg00101350 0.95144739 0.94164672 0.93051506 0.91925925 0.93790633
    ##                  17-b        9-f       13-f       10-f       15-f
    ## cg00017461 0.07913857 0.51552811 0.53620403 0.52865836 0.55381533
    ## cg00077299 0.47308530 0.52972800 0.51000904 0.51519016 0.47084652
    ## cg00079563 0.05382237 0.02570023 0.03119169 0.05561398 0.03425447
    ## cg00087182 0.94143343 0.96182184 0.95444691 0.94319432 0.95579813
    ## cg00093544 0.23173156 0.10577046 0.16784111 0.17211284 0.18222262
    ## cg00101350 0.93190992 0.56962917 0.59533958 0.58626153 0.57137974
    ##                  16-f       17-f        9-m        13m       10-m
    ## cg00017461 0.57938045 0.58650575 0.73492506 0.75575856 0.74032958
    ## cg00077299 0.51348592 0.54692193 0.50982524 0.50093892 0.52500117
    ## cg00079563 0.03171041 0.05559975 0.02406349 0.03345134 0.02405020
    ## cg00087182 0.94840135 0.92002764 0.95507416 0.95628002 0.92885676
    ## cg00093544 0.15118890 0.25074698 0.08315614 0.09630402 0.07598193
    ## cg00101350 0.56715788 0.58880795 0.64346828 0.68329256 0.64852727
    ##                  15-m       16-m       17-m
    ## cg00017461 0.78609454 0.78865815 0.74634126
    ## cg00077299 0.50506118 0.50817635 0.51116868
    ## cg00079563 0.04277312 0.03717952 0.05370342
    ## cg00087182 0.95442459 0.94636188 0.95508575
    ## cg00093544 0.13413231 0.11069193 0.16073665
    ## cg00101350 0.65858280 0.64741491 0.61958075

``` r
dim(TissueDataChr22)
```

    ## [1] 8513   18

**G In R we can use functions to easily run a model across many loci.
Run the function for the first and the second line.**

``` r
getPvalue <- function(row, data, samplesheet)
{
  fit <- anova(lm(data[row,]~samplesheet$Tissue)) # Run the model
  out <- data.frame(CG = row, Fvalue = fit$`F value`[1], Pvalue = fit$`Pr(>F)`[1]) # Make a data.frame for the output
  as.matrix(out)
}

#Run for one CpG site
getPvalue('cg00017461', data=TissueDataChr22, samplesheet=samplesheet)
```

    ##      CG           Fvalue     Pvalue       
    ## [1,] "cg00017461" "1592.851" "3.39895e-18"

**H Now run the model across all CpGs on chromosome 22. Note that this
takes a bit of time - after all R is running 8513 tests - and note that
this couldn’t be achieved using SPSS (only when the tests were done one
by one). Look at the output.**

``` r
#Which CpGs are there
cpgs <- rownames(TissueDataChr22)
head(cpgs)
```

    ## [1] "cg00017461" "cg00077299" "cg00079563" "cg00087182" "cg00093544"
    ## [6] "cg00101350"

``` r
# Now run the model for all CpGs on chromosome 22
res <- lapply(cpgs, getPvalue, data=TissueDataChr22, samplesheet=samplesheet) # Returns a list of results per CpG
head(res)
```

    ## [[1]]
    ##      CG           Fvalue     Pvalue       
    ## [1,] "cg00017461" "1592.851" "3.39895e-18"
    ## 
    ## [[2]]
    ##      CG           Fvalue     Pvalue      
    ## [1,] "cg00077299" "5.801467" "0.01360545"
    ## 
    ## [[3]]
    ##      CG           Fvalue     Pvalue     
    ## [1,] "cg00079563" "0.925408" "0.4178568"
    ## 
    ## [[4]]
    ##      CG           Fvalue      Pvalue     
    ## [1,] "cg00087182" "0.1269301" "0.8817312"
    ## 
    ## [[5]]
    ##      CG           Fvalue     Pvalue       
    ## [1,] "cg00093544" "8.045898" "0.004225137"
    ## 
    ## [[6]]
    ##      CG           Fvalue     Pvalue        
    ## [1,] "cg00101350" "923.7006" "1.973037e-16"

``` r
ResultsChromosome22 <- do.call(rbind, res) #Combine to a table
head(ResultsChromosome22)
```

    ##      CG           Fvalue      Pvalue        
    ## [1,] "cg00017461" "1592.851"  "3.39895e-18" 
    ## [2,] "cg00077299" "5.801467"  "0.01360545"  
    ## [3,] "cg00079563" "0.925408"  "0.4178568"   
    ## [4,] "cg00087182" "0.1269301" "0.8817312"   
    ## [5,] "cg00093544" "8.045898"  "0.004225137" 
    ## [6,] "cg00101350" "923.7006"  "1.973037e-16"

## Question 3

**A When measuring many features (here CpGs), one must account for the
fact that many statistical tests are being performed. Small P-values
will be observed purely by chance. In this case, the number of tests
equals the number of CpGs in the file. Which P-value threshold is
commonly used to declare an association statistically significant in
case of a single statistical test? How many P-values lower than this
threshold would you expect by chance in this chromosome 22 data set?**

**B An intuitive and popular (but perhaps somewhat conservative) way to
correct the p-value is to multiply the calculated P-value by the number
of tests performed or correct the significance level, i.e. divide the
P-value threshold for a single test by the total number of tests. This
is called Bonferroni correction. What is the Bonferroni-corrected
P-value for chromosome 22 (i.e. the threshold for statistical
significance after accounting for multiple testing)?**

**C For question 1C, you calculated a P-value for the example CpG site.
Compare this P-value to the P-value calculated in 3B (after all it’s
only one of many CpG sites measured on chromosome 22). Is the difference
significant after multiple testing correction?**

**D The column Pval\_corrected contains 0 and 1’s that mark
non-significant and significant CpGs based on the P-value calculated in
3B. How many CpGs (absolute and as percentage) are significant after
multiple testing?**

``` r
table(TissueData22$Pval_corrected)
```

    ## 
    ##    0    1 
    ## 6658 1805

**E Now, we want to know whether a particular genomic feature is
overrepresented in the identified significant CpG sites. You will focus
on the two best covered genic annotations (proximal promoters and gene
bodies). The significant CpGs in these two features can be found in the
file Tissue data chromosome 22 significant.csv. Load the file. Make a
frequency table of the number of CpGs per group and split by CpG islands
and non-CpG islands.**

``` r
TissueData22sign <- read.table("Tissue data chromosome 22 significant.csv", sep=";", header=T)
head(TissueData22sign)
```

    ##           ID chromosome Start_coord End_coord      Blood     SC_Fat
    ## 1 cg00017461      chr22    30663316  30663316 0.05894668 0.55001534
    ## 2 cg00047287      chr22    51016899  51016899 0.85500874 0.73735978
    ## 3 cg00083659      chr22    37404577  37404577 0.11866044 0.47920586
    ## 4 cg00089100      chr22    26922951  26922951 0.84390670 0.81708392
    ## 5 cg00143809      chr22    50720552  50720552 0.80814305 0.87254914
    ## 6 cg00164898      chr22    50709127  50709127 0.06010305 0.06735435
    ##      Muscle      Gene_centric CGI_centric     Pval
    ## 1 0.7586845 Proximal promoter     Non CGI 3.40e-18
    ## 2 0.5013027 Proximal promoter CpG islands 2.55e-11
    ## 3 0.7452466 Proximal promoter     Non CGI 1.84e-15
    ## 4 0.4246997         Gene body     Non CGI 5.73e-14
    ## 5 0.5038895         Gene body CpG islands 6.29e-10
    ## 6 0.1649458 Proximal promoter CpG islands 4.19e-13

``` r
table(TissueData22sign$Gene_centric)
```

    ## 
    ##         Gene body Proximal promoter 
    ##               370               465

``` r
table(TissueData22sign$CGI_centric)
```

    ## 
    ## CpG islands     Non CGI 
    ##         278         557

``` r
table3e <- table(TissueData22sign$Gene_centric, TissueData22sign$CGI_centric)
table3e
```

    ##                    
    ##                     CpG islands Non CGI
    ##   Gene body                 184     186
    ##   Proximal promoter          94     371

**F Now you have the frequency per combined feature (so for example
proximal promoter CpG islands). Calculate the percentages of the number
of significant CpGs per feature compared to the total number of
significant CpGs from 3D. What is your interpretation?**

``` r
table3f <- (table3e / 1805)*100
round(table3f,1)
```

    ##                    
    ##                     CpG islands Non CGI
    ##   Gene body                10.2    10.3
    ##   Proximal promoter         5.2    20.6

**G Using the percentages to see where the changes occur is driven by
the design of the technology. One can also express the
overrepresentation of CpG sites in a specific feature as an odds ratio.
What does the odss ratio reflect? What does an odds ratio below 1 mean?
Calculate the odds ratio for proximal promoters, what do you conclude?**

|                   | Sign | Non-Sign | Total |
| ----------------- | ---- | -------- | ----- |
| Proximal Promoter | a    | b        | a+b   |
| Elsewhere         | c    | d        | c+d   |
| Total             | a+c  | b+d      |       |

``` r
#Calculate a
a <- table(TissueData22sign$Gene_centric)["Proximal promoter"]
a
```

    ## Proximal promoter 
    ##               465

``` r
n.prox <- table(TissueData22$Gene_centric)["Proximal promoter"]
n.prox
```

    ## Proximal promoter 
    ##              4244

``` r
#Calculate b-d
b <- n.prox - a
c <- 1805 - a
d <- 8463 - a - b - c
```

**2x2 table**

``` r
matrix(c(a,b,c,d), ncol=2, byrow = T)
```

    ##      [,1] [,2]
    ## [1,]  465 3779
    ## [2,] 1340 2879

**Odds ratio**

``` r
#Now calculate the odds ratio
OR <- (a/b)/(c/d)

#One can formally test the enrichment / depletion with a chi-squared test
chisq.test(matrix(c(a,b,c,d), ncol=2, byrow = TRUE))
```

    ## 
    ##  Pearson's Chi-squared test with Yates' continuity correction
    ## 
    ## data:  matrix(c(a, b, c, d), ncol = 2, byrow = TRUE)
    ## X-squared = 544.52, df = 1, p-value < 2.2e-16

``` r
#Or Fisher exact test (better)
fisher.test(x = matrix(c(a,b,c,d), ncol=2, byrow = TRUE))
```

    ## 
    ##  Fisher's Exact Test for Count Data
    ## 
    ## data:  matrix(c(a, b, c, d), ncol = 2, byrow = TRUE)
    ## p-value < 2.2e-16
    ## alternative hypothesis: true odds ratio is not equal to 1
    ## 95 percent confidence interval:
    ##  0.2348783 0.2972908
    ## sample estimates:
    ## odds ratio 
    ##   0.264433

## Question 4

**Now, you will focus on ‘top-hits’ (=CpG sites with lowest P-values)**

**A Sort the file by P-value (ascending). Select the two CpG sites with
the lowest and second lowest P-value. What is the methylation level of
these CpGs in the tissues investigated? What is the largest difference
in DNA methylation observed?**

``` r
TissueData22signOrdered <- TissueData22sign[order(TissueData22sign$Pval),]
head(TissueData22signOrdered)
```

    ##             ID chromosome Start_coord End_coord      Blood    SC_Fat
    ## 145 cg04078115      chr22    33278114  33278114 0.11140063 0.6293710
    ## 598 cg19863740      chr22    44576869  44576869 0.16867353 0.6304102
    ## 663 cg21629591      chr22    20783896  20783896 0.09397190 0.6413875
    ## 1   cg00017461      chr22    30663316  30663316 0.05894668 0.5500153
    ## 531 cg17527673      chr22    20783497  20783497 0.20337884 0.7286569
    ## 254 cg07940593      chr22    39710068  39710068 0.81142345 0.7049340
    ##        Muscle      Gene_centric CGI_centric     Pval
    ## 145 0.8179198         Gene body     Non CGI 4.17e-19
    ## 598 0.7335512 Proximal promoter     Non CGI 4.63e-19
    ## 663 0.7635778         Gene body CpG islands 3.37e-18
    ## 1   0.7586845 Proximal promoter     Non CGI 3.40e-18
    ## 531 0.8846741         Gene body CpG islands 4.82e-18
    ## 254 0.3977558 Proximal promoter     Non CGI 5.12e-18

**B Go to the UCSC genome browser at <http://genome.ucsc.edu/>. Click on
My Data at the top of the page and then Custom Tracks. Upload the
significant probe.bed as a custom track (click Choose File and then
Submit; like before, use assembly Feb 2009 (GRCh37/hg19)). Click go to
genome browser. For clarity, display the track as ‘dense’. A red
vertical line denotes a differentially methylated CpG (accounting for
multiple testing), a blue vertical line is a CpG for which there is no
statistical evidence for differential methylation.**

**C Look up to the two top-hits (See 1E how to if you don’t know how).
In which gene centric / CGI centric feature are the top-hits? Also
see/use question 2C.**

**D Go through question 1f-h again for the tophit. What do you observe**

**E Zoom out in UCSC for both top-hits. Is the differential methylation
limited to a single CpG site or does it extend across a region (Use the
custom track from 4B)? Does this influence your interest in the two CpGs
and why?**

**F The second lowest top-hit maps to the PARVG gene. How many different
transcripts can arise from this gene (track UCSC genes)? Do you observe
differential methylation near alternative transcript start sites for the
gene?**

**G Look up the nearest gene of the tophits in GeneCards
(www.genecards.org). Look at the protein expression and the mRNA
expression for the three tissues studied if available. Is the gene
expressed in a tissue-specific fashion?**

**H To gain insight into gene function related to differentially
methylation you can use GREAT (Genomic Regions Enrichment of Annotations
Tool). Go to the GREAT
website(<http://bejerano.stanford.edu/great/public/html/index.php>).
Select human hg19/ GRCh37. Select all significant CpG sites from the csv
file and copy the chromosome, start and end column and paste that into
test regions (after clicking BED data). Also, copy all CpG sites and
paste that into background regions (click BED data). Click Show Settings
and select single nearest gene. Click submit and wait (may take some
time) until the site has calculated the enrichment in gene categories
(Biological processes). Which biological processes are enriched for
differential methylation? How does that relate to the tissues studied?**

## Question 5 (Facultative)

**There is increasing evidence that DNA methylation is associated with
alternative transcription events. A paper showed that there is a role
for intragenic (differential) DNA methylation in the regulation of
alternative transcription events (Maunakea AK et al. Conserved role of
intragenic DNA methylation in regulating alternative promoters. Nature
2010; 466: 253-7).**

**A Read the abstract and inspect Figure 3a in the paper. In what
process is DNA methylation implicated to play a role in? For which gene
and which annotation within that gene do the authors provide
experimental evidence for this role? Discuss the tracks given in Figure
3a and their implication.**

**B Look up this gene in UCSC for you own data. Display the custom track
you used before and CpG island track. Argue why the data on the three
tissues is either in line or at odds with the observations of Maunakea
et al.**
