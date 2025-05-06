1. quality_control

/lustre/home/mwcai/metaWRAP/bin/metawrap read_qc -1 GKD-007.R1.fq -2 GKD-007.R2.fq  -t 32 -x hg38 -o 1.read_qc/GKD-007

2. assembly
metawrap assembly -1 ./1.read_qc/GKD-007/GKD007_1.fastq -2 ./1.read_qc/GKD-007/GKD007_2.fastq -m 1000 -t 128 -o ASSEMBLY7

3. initial_binning
metawrap binning -o ./ASSEMBLY7/initial_binning -t 64 -a ./ASSEMBLY7/final_assembly.fasta --metabat2 --maxbin2 --concoct ./1.read_qc/GKD-007/GKD007_1.fastq ./1.read_qc/GKD-007/GKD007_2.fastq

4. bin_refinement
metawrap bin_refinement -o ./ASSEMBLY7/bin_refinement  -t 64 -A ./ASSEMBLY7/initial_binning/metabat2_bins -B ./ASSEMBLY7/initial_binning/maxbin2_bins  -c 50 -x 10

5. bin_reassemble
metawrap reassemble_bins -o ./ASSEMBLY7/bin_reassembly_2  -t 128 -m 2000 -c 50 -x 10 -1 ./1.read_qc/GKD-007/GKD007_1.fastq -2 ./1.read_qc/GKD-007/GKD007_2.fastq -b ./ASSEMBLY7/bin_refinement/metawrap_50_10_bins

6. quant_bins
metawrap quant_bins -b ./ASSEMBLY7/bin_reassembly/HQ  -o ASSEMBLY7/quant_bins5 -a ASSEMBLY7/final_assembly.fasta  ./1.read_qc/GKD-007/*.fastq

