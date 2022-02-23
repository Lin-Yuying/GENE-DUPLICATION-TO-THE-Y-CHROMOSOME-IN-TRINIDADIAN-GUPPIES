## GuppyGeneDuplication
Scripts for Lin, Y., Darolti, I., Furman, B. L. F., Almeida, P., Sandkam, B. A., et al., Breden, F., Wright, A. E., Mank, J. E. (2022) Gene duplication to the Y chromosome in Trindadian Guppies. Molecular Ecology. https://doi.org/10.1111/mec.16355

Note: All Python scripts is written with Python3, it is not compatible with Python2.

1. Quality control using FastQC and Trimmomatic.
   ```
   python 01.quality_control_fqs.py -fqs [/path/fq_files/] -o [./qc/] -t [10] -trim -qc
   ```

2. Align high-quality reads to female reference genome using BWA MEM.
   ```
   python 02.alignment_bwa.py -r [/path/ref_genome/] -bwa [bwa] -fq [./trim/] -a -t [10]
   ```

3. Fixmate, sort and mark duplications using SAMtools.
   ```
   sh 03.align_postprocess.sh [sample_ID]
   ```

4. Genotyping,SNP filtering and calculating intersexual Fst. 

    #(1) Genotyping using SAMtools
    ```
    sh 04.GenotypingSNPFiltering/SNP_calling.sh [sample_ID] [region_name]
    ```

    #(2) SNP filtering using VCFtools, here we only keep SNPs in CDS regions by specifying `--bed guppy_cds_coords.txt`
    ```
    vcftools --gzvcf [input] --maf 0.05 --mac 1 --min-alleles 2 --max-alleles 2 --max-missing 0.9 --min-meanDP 10 
    --max-meanDP 100 --bed guppy_cds_coords.txt --recode --recode-INFO-all --minGQ 25 --out [output]
    ```

    #(3) Calculating intersexual Fst for each SNP using VCFtools
    ```
    vcftools --vcf [vcf] --weir-fst-pop [female.txt] --weir-fst-pop [male.txt] --out [output]
    ```
    
    #(4) SNPs with top 1% Fst 
    ```
    # we simply do 1% cut-off here, for example in Python3, we use quantile function from numpy pkg
    numpy.quantile (data, 0.01) 
    ```

    #(5) Fisher's exact test using PLINK1.9, please check details on 05.FisherExactTest folder
    ```
    python 05.FisherExactTest/changeid.py [original.vcf] [newid.vcf]
    sh 05.FisherExactTest/fisher_cds_vcf.sh [newid.vcf] [outprefix]
    ```

    #(6) Permutation test, please check details on 06.PermutationTest folder
    ```
    sh 06.permutation_test/run_permutation_sex.sh
    ``` 

5. Male-to-Female read depth ratio 
   ```
   sh 07.M2FReadDepth/MFReadDepth.sh
   ```

6. Tajima's D

    #(1) exclude genes with immune and MHC function in reference genome based on [Biomart](http://uswest.ensembl.org/biomart/martview/7d40f23a42e2cecb7cdd1542b97cda5f) info from Ensembl on          
    
    ```
    python 08.TajimaD/exclude_immune_MHC.py [immune_MHC.bed] [outprefix.csv] 
    ```
    
    #(2) calculate gene-based Tajima's D
    ```
    python 08.TajimaD/gene_TajimaD.py [gene_boundary.csv] [site.TajimaD] [outprefix.csv]
    ```

7. Relatedness Inference 

    #(1) ngsRelate, here, please use filtered VCF and DON'T exclude non-CDS regions
    ```
    /path/ngsrelate  -h [VCF.filtered.gz] -O [vcf.res]
    ```

    #(2) KING, before using KING, please convert VCF to `.bed`, `.fam` and `.bim` using PLINK1.9 
    ```
    /path/king -b [my.bed] --fam [my.fam] --bim [my.bim] --related
    ```

