# Script code.
* [Back home](../README.md)


```sh
#!/bin/sh

out=$PWD/col;
name=col;
mkdir -p $out;
scripts=$PWD/script;
annotation=$PWD/annofile;
cores=8;

download_data(){
mkdir -p annofile
cd annofile

echo "download data."
wget ftp://ftp.ensemblgenomes.org/pub/plants/release-43/gff3/arabidopsis_thaliana/Arabidopsis_thaliana.TAIR10.43.gff3.gz
wget ftp://ftp.ensemblgenomes.org/pub/plants/release-43/fasta/arabidopsis_thaliana/dna/Arabidopsis_thaliana.TAIR10.dna.toplevel.fa.gz
wget ftp://ftp.ensemblgenomes.org/pub/release-43/plants/gtf/arabidopsis_thaliana/Arabidopsis_thaliana.TAIR10.43.gtf.gz
}
#download_data
uncomprise(){
cd $annotation
echo "uncomprise the data"
gunzip Arabidopsis_thaliana.TAIR10.43.gff3.gz
gunzip Arabidopsis_thaliana.TAIR10.dna.toplevel.fa.gz
gunzip Arabidopsis_thaliana.TAIR10.43.gtf.gz

echo "Build the fasta index."
samtools faidx Arabidopsis_thaliana.TAIR10.dna.toplevel.fa

}
#uncomprise;
generate_perfect_gtf(){
echo "Generate gtf add start stop codon, miRNA, snRNA, snoRNA"
perl $scripts"/gfftogtf.pl" $annotation/Arabidopsis_thaliana.TAIR10.43.gff3
sh $scripts/grep_start_stop.sh $annotation/Arabidopsis_thaliana.TAIR10.43.gtf $annotation/Arabidopsis_thaliana.TAIR10.43.gff3.clean.gtf
rm $annotation/Arabidopsis_thaliana.TAIR10.43.gff3.clean.gtf
mv $annotation/Arabidopsis_thaliana.TAIR10.43.gff3.clean.gtf.clean.sort.gtf  $annotation/tair10.gtf
}
#generate_perfect_gtf;

create_annotation(){
echo "Pre-processing"
echo "---------------"
echo ""
echo "Create Ribowave annotation file";
$scripts"/create_annotation.sh" -G $annotation/tair10.gtf \
 -f $annotation/Arabidopsis_thaliana.TAIR10.dna.toplevel.fa \
  -o $annotation \
  -s $scripts;
}
#create_annotation;
create_orf_label(){
echo "Pre-processing"
echo "---------------"
echo "Create the ORF label"
perl $scripts"/get_orf_classify.pl" $annotation/final.ORFs.annot3
}
create_orf_label
p_site_detemination(){
echo "P-site detemination"
echo ""
echo "---------------"
echo ""
$scripts"/P-site_determination.sh"  -i  $out/$name.sort.bam  -S $annotation/start_codon.bed  -o $out      -n $name   -s $scripts;
perl $scripts"/get_psite1nt.pl" $out/P-site/$name.psite.txt;
mv $out/P-site/$name.psite.txt.txt $out/P-site/$name.psite1nt.txt;
}
#p_site_detemination;



creating_p_site_trace(){

echo "Creating P-site track"
echo "---------------"
echo ""
$scripts"/create_track_Ribo.sh"  -i $out/$name.sort.bam  -G $annotation/X.exons.gtf  -g $annotation/genome  -P $out/P-site/$name.psite1nt.txt -o $out -n $name -s $scripts;
}
#creating_p_site_trace;

#       Main function
denoise_raw_signal(){
echo "denoise raw signal"
mkdir -p $out/Ribowave;
$scripts"/Ribowave"  -a $out/bedgraph/$name/final.psite -b $annotation/final.ORFs -o $out/Ribowave   -n $name -s $scripts -p 8;
echo ""
echo "---------------"
echo ""
}
#denoise_raw_signal;

predict_p_value_translation(){
echo "predict p.value of translation"
mkdir -p $out/Ribowave;
$scripts"/Ribowave"  -P -a $out/bedgraph/$name/final.psite -b $annotation/final.ORFs -o $out/Ribowave   -n $name -s $scripts -p 8;
echo ""
echo "---------------"
echo ""
}
#predict_p_value_translation;

estimate_abundance(){
echo "estimate abundance/density"
mkdir -p $out/Ribowave;
$scripts"/Ribowave"  -D -a $out/bedgraph/$name/final.psite -b $annotation/final.ORFs -o $out/Ribowave   -n $name -s $scripts -p 8;
echo ""
echo "---------------"
echo ""
}
#estimate_abundance;

calculating_te(){
echo " calculating TE"
$scripts"/Ribowave" -T 9012445  $out/mRNA/SRR1039761.RPKM -a $out/bedgraph/$name/final.psite -b $annotation/final.ORFs -o $out/Ribowave   -n $name -s $scripts -p 8;
echo ""
echo "---------------"
echo ""
}
#calculating_te;

calculating_frameshift(){
echo " calculating frameshift potential on annotated ORF"
mkdir -p $out/Ribowave;
awk -F '\t' '$3=="anno"'	$annotation/final.ORFs	>	$annotation/aORF.ORFs;
$scripts"/Ribowave" -F -a $out/bedgraph/$name/final.psite -b $annotation/aORF.ORFs -o $out/Ribowave   -n $name -s $scripts -p 8;
echo ""
echo "---------------"
echo ""
}
#calculating_frameshift;


multiple_function(){
echo " multiple functions"
mkdir -p $out/Ribowave;
$scripts"/Ribowave" -PD -T 9012445  $out/mRNA/SRR1039761.RPKM -a $out/bedgraph/d14/final.psite -b $annotation/final.ORFs -o $out/Ribowave   -n $name -s $scripts -p 8;
echo ""
echo "---------------"
echo ""
}
```

