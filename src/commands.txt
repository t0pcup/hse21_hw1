func() {
  rm -rf fastqc multiqc
  mkdir fastqc multiqc
  time fastqc "$@" -o fastqc
  multiqc fastqc -o multiqc
}

SEED=404
rm -rf data
mkdir data
cd data
ln --symbolic /usr/share/data-minor-bioinf/assembly/oil* .
git clone https://github.com/t0pcup/hse21_hw1.git

seqtk sample -s$SEED oil_R1.fastq 5000000 > sub1.fastq
seqtk sample -s$SEED oil_R2.fastq 5000000 > sub2.fastq
seqtk sample -s$SEED oilMP_S4_L001_R1_001.fastq 1500000 > mp1.fastq
seqtk sample -s$SEED oilMP_S4_L001_R2_001.fastq 1500000 > mp2.fastq

func sub*.fastq mp*.fastq
cp multiqc/multiqc_report.html hse21_hw1/multiqc.html
time platanus_trim sub*.fastq
rm sub*.fastq
time platanus_internal_trim mp*.fastq
rm mp*.fastq

func *trimmed
cp multiqc/multiqc_report.html hse21_hw1/multiqc_trimmed.html
time platanus assemble -f *.trimmed
mkdir hse21_hw1/data
cp out_contig.fa hse21_hw1/data/contigs.fasta

time platanus scaffold -c out_contig.fa -IP1 *.trimmed -OP2 *.int_trimmed
cp out_scaffold.fa hse21_hw1/data/scaffold_with_gaps.fasta
time platanus gap_close -c out_scaffold.fa -IP1 *.trimmed -OP2 *.int_trimmed
cp out_gapClosed.fa hse21_hw1/data/scaffold.fasta
rm *trimmed

cd hse21_hw1
git config user.email "t0pcup@yandex.ru"
git config user.name "Mikhaylova Ksenia"
git add .
git commit -m "data"
git push git@github.com:t0pcup/hse21_hw1.git master

sed -n "/$(grep '^>' data/scaffold_with_gaps.fasta | sort -k2V -t_ | tail -n 1)/,/^>/p" data/scaffold_with_gaps.fasta | head -n -1
sed -n "/$(grep '^>' data/scaffold.fasta | sort -k2V -t_ | tail -n 1)/,/^>/p" data/scaffold.fasta | head -n -1