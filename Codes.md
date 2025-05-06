#Meta-omics pipeline

Step 1. quality_control

/lustre/home/mwcai/metaWRAP/bin/metawrap read_qc -1 GKD-007.R1.fq -2 GKD-007.R2.fq  -t 32 -x hg38 -o 1.read_qc/GKD-007

Step 2. assembly
metawrap assembly -1 ./1.read_qc/GKD-007/GKD007_1.fastq -2 ./1.read_qc/GKD-007/GKD007_2.fastq -m 1000 -t 128 -o ASSEMBLY7

Step 3. initial_binning
metawrap binning -o ./ASSEMBLY7/initial_binning -t 64 -a ./ASSEMBLY7/final_assembly.fasta --metabat2 --maxbin2 --concoct ./1.read_qc/GKD-007/GKD007_1.fastq ./1.read_qc/GKD-007/GKD007_2.fastq

Step 4. bin_refinement
metawrap bin_refinement -o ./ASSEMBLY7/bin_refinement  -t 64 -A ./ASSEMBLY7/initial_binning/metabat2_bins -B ./ASSEMBLY7/initial_binning/maxbin2_bins  -c 50 -x 10

Step 5. bin_reassemble
metawrap reassemble_bins -o ./ASSEMBLY7/bin_reassembly_2  -t 128 -m 2000 -c 50 -x 10 -1 ./1.read_qc/GKD-007/GKD007_1.fastq -2 ./1.read_qc/GKD-007/GKD007_2.fastq -b ./ASSEMBLY7/bin_refinement/metawrap_50_10_bins

Step 6. quant_bins
metawrap quant_bins -b ./ASSEMBLY7/bin_reassembly/HQ  -o ASSEMBLY7/quant_bins5 -a ASSEMBLY7/final_assembly.fasta  ./1.read_qc/GKD-007/*.fastq

Step 7.  species-level_dereplication
dRep dereplicate output_dRep_HQ3_95_v2_97_03 -p 256 -g  ./mags_fa_HQ3/*.fa -sa 0.97 -nc 0.3 -comp 10 -con 50 --checkM_method taxonomy_wf  --debug

Step 8. checkm
checkm lineage_wf -x fa /lustre/home/mwcai/data/CAS_hospital/mags_fa_HQ3 mags_fa_HQ3_checkm_result -f mags_fa_HQ3_checkm_result.txt  -t 320

Step 9.iq-tree
iqtree -m Q.pfam+F+I+R10 -s concatenated.v2.fasta -nt AUTO --redo -B 1000

Step10. fa to faa
for entry in './'*.fasta
do
prodigal -f gff -i $entry -o $entry.gff -p meta -a $entry.faa
done

Step11. genome annotation
diamond blastp -d /lustre/home/mwcai/data/database_for_protein_annotation/dbCAN_for_CAZYme/db/CAZy.dmnd -q  total_up.faa  -o diamond_dbCAN_result --more-sensitive -p 32 --max-target-seqs 5 -e 1E-10 -f 6

diamond blastp -d /lustre/home/mwcai/data/database_for_protein_annotation/hydrogenase_database/hyddb-results_conct_.dmnd -q  total_up.faa  -o diamond_hyddb_result --more-sensitive -p 32 --max-target-seqs 5 -e 1E-10 -f 6

diamond blastp -d /lustre/home/mwcai/data/database_for_protein_annotation/peptidase_MEROPS/pepunit.lib.faa.peptidase.reformat.list.v2.faa.dmnd -q  total_up.faa  -o diamond_peptidase_result --more-sensitive -p 32 --max-target-seqs 5 -e 1E-10 -f 6

diamond blastp -d /lustre/home/mwcai/data/database_for_protein_annotation/TCDB_DB_for_transporter/tcdb.dmnd -q  total_up.faa  -o diamond_tcdb_result --more-sensitive -p 32 --max-target-seqs 5 -e 1E-10 -f 6

emapper.py -i total_up.faa  --output up_emapper -m diamond --cpu 32 --override

interproscan.sh -i total_up.faa -b total_up.faa.interproscan -iprlookup -f tsv

Step 12. metatranscriptomic processing
for entry in './'*.t.fq
do
bbmap.sh minid=0.95 maxindel=3 bwr=0.16 bw=12 quickmatch fast minhits=2 path=/lustre/home/mwcai/data/remove_human_reads_from_MG_MT qtrim=rl trimq=10 untrim -Xmx23g in=$entry outu=$entry.clean.fq outm=$entry.human.fq
done

sortmerna --ref /lustre/home/mwcai/tools/sortmerna/db/smr_v4.3_default_db.fasta --reads GKD-007.R1_val_1.fq.t.fq.clean.fq --aligned GKD007.R1_val_1.fq.t.fq.clean.fq.rna.fq    --other GKD-007.R1_val_1.fq.t.fq.clean.fq.mrna.fq    -a 32 -e 1e-10 --fastx

Step 13. gene and transcript quantification


Step14. prevalence of spcecies across cohorts
for entry in "./"*fq
do
coverm contig --single $entry -r ../ref_seq_BCAA_bu_pv.fa --min-read-percent-identity 95 --min-read-aligned-percent 50  -o $entry.coverm
done

