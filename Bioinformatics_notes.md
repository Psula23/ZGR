
# POBIERANIE SUROWYCH ODCZYTÓW Z BAZY DANYCH SRA ORAZ KONWERTOWANIE

(Ręcznie odczyty można pobrać z bazy danych ENA)

- SRATOOLKIT - narzędzie

- prefetch - pobieranie surowych odczytów

- fastq-dump --split-files - konwertowanie odczytów oraz podział na dwa pliki dla sparowanych odczytów

- '#' w bashu to oznacza komentarz

- '##' na serwerze to oznacza komentarz

- '#' na serwerze to piszemy przed komendami: #SBATCH
```
 cat probki.txt | while read line

  do
  #prefetch SRR1028838
  #fastq-dump --split-files $line
  #fastq-dump -X 5 -Z SRR390728
  #fastq-dump -X 5 -Z /home/plgrid-groups/plggillumina/plgpaulina/odczyty/SRR390728.sra
  echo $line
  done

```

>dodatkowe przydatne komendy
```
##pobieranie na serwer bezpośrednio
##wget https://www.ncbi.nlm.nih.gov/search/api/sequence/SRR1028838/?report=fasta

##wget -O SRR1028838.fast.gz "http://trace.ncbi.nlm.nih.gov/Traces/sra/sra.cgi?cmd=dload&run_list=SRR1028838&format=fastq"
##gzip -d SRR1028838.fastq.gz
##wget https://trace.ncbi.nlm.nih.gov/Traces/sra-reads-be/fastq?acc=SRR1028838?report=fasta


```

# URUCHOMIENIE ANALIZY JAKOŚCI SUROWYCH ODCZYTÓW Z PROGRAMEM FASTP
## SKRYPT.SL

>Komenda uruchamiająca skrypt na serwerze: sbatch nazwa_pliku.sl
>
>sbatch skrypt.sl -> ktory uruchamia fastp.sh -> który wywołuje plik z próbkami (poprzednia wersja, gdzie podawało się ścieżki bezwzględne do plików)

```
#!/bin/bash

##nazwa grantu
##SBATCH -A lrassembly

##nazwa pliku z wyjsciem konsoli
#SBATCH --output="out5.output"

## nazwa pliku z wyjsciem bledow
#SBATCH --error="out5.error"

## nazwa wezla na którym chcemy uruchamiać obliczenia - tutaj tesla - wezel z gpu
##SBATCH -p tesla

#SBATCH -N1
#SBATCH -n10
##SBATCH --mem=400G

##module

source /home/plgrid/plgponiat/anaconda3/bin/activate fastp

bash \
/home/plgrid-groups/plggillumina/plgpaulina/odczyty/odczyty2/skrypt_fastp.sh \

```



## fastp.sh

```
#!/bin/bash

 cat /home/plgrid-groups/plggillumina/plgpaulina/odczyty/odczyty2/probki.txt | while read line

  do
 
      fastp --detect_adapter_for_pe --overrepresentation_analysis --length_required 30 --cut_tail_window_size 1 --cut_tail_mean_quality 15 --cut_tail --thread 2 --html /home/plgrid-groups/plggillumina/plgpaulina/odczyty/odczyty2/trimmed4/$line.fastp.html --json /home/plgrid-groups/plggillumina/plgpaulina/odczyty/odczyty2/trimmed4/$line.fastp.json -i /home/plgrid-groups/plggillumina/plgpaulina/odczyty/odczyty2/${line}_1.fastq -I /home/plgrid-groups/plggillumina/plgpaulina/odczyty/odczyty2/${line}_2.fastq -o /home/plgrid-groups/plggillumina/plgpaulina/odczyty/odczyty2/trimmed4/${line}_1.fastq -O /home/plgrid-groups/plggillumina/plgpaulina/odczyty/odczyty2/trimmed4/${line}_2.fastq
      
      
      ./fastp --detect_adapter_for_pe --overrepresentation_analysis --cut_front_window_size 1 --cut_front_mean_quality 15 --cut_front --cut_tail_window_size 1 --cut_tail_mean_quality 15 --cut_tail --cut_right_window_size 4 --cut_right_mean_quality 15 --cut_right --thread 2 --html /mnt/d029a7f8-849c-4392-a9d8-f40f5740f81e/sratoolkit.3.0.0-ubuntu64/bin/odczyty2/Trimmed/SRR1028820.fastp.html --json /mnt/d029a7f8-849c-4392-a9d8-f40f5740f81e/sratoolkit.3.0.0-ubuntu64/bin/odczyty2/Trimmed/SRR1028820.fastp.json -i /mnt/d029a7f8-849c-4392-a9d8-f40f5740f81e/sratoolkit.3.0.0-ubuntu64/bin/odczyty2/SRR1028820_1.fastq -I /mnt/d029a7f8-849c-4392-a9d8-f40f5740f81e/sratoolkit.3.0.0-ubuntu64/bin/odczyty2/SRR1028820_2.fastq



  #echo "$i"
  echo $line
  
  done


echo koniec
```

## Przykładowy plik z próbkami (zwykły plik w notatniku: probki.txt)

```
SRR1028838
SRR1034055
SRR1034071
SRR1034060
SRR1034078
SRR1034166
```

## Inne komendy 
```
srun --pty /bin/bash  -  przejście w tryb interaktywny
module avail - sprawdzenie dostepnych modułów
squeue -u nazwa_uzytkownia - sprawdzenie włączonych zadań użytkownika

```

# INSTALOWANIE ANACONDY NA SERWERZE

> Najnowsza wersja: https://repo.anaconda.com/archive/
```
wget https://repo.anaconda.com/archive/Anaconda3-2022.05-Linux-x86_64.sh
bash Anaconda3-2022.05-Linux-x86_64.sh

```
## Łączenie plików

```
#!/bin/bash

cat SRR1584397_2.fastq SRR1664188_2.fastq > HM004_2.fastq
cat SRR1034089_2.fastq > MH007_2.fastq
cat SRR1034095_2.fastq SRR1034096_2.fastq SRR1034097_2.fastq > HM008_2.fastq

```


# MAPOWANIE


## Indeksowanie
```
##bwa index /home/users/pponiat/grant_620/project_data/paulina/GCF_003473485.1_MtrunA17r5.0-ANR_genomic.fna
```
## Mapowanie
>plik sktypt.sl

```
#!/bin/bash

#SBATCH -p standard
#SBATCH --nodes=1
#SBATCH --ntasks-per-node=32
#SBATCH --mem=100G
#SBATCH --time=96:00:00

##przeslanie maila po kazdej zmianie statusu obliczen
##SBATCH --mail-type=ALL

##nazwa plikow z wyjsciem i bledami
#SBATCH --output="load.output555"
#SBATCH --error="load.error555"

##ustawienie zmiennej $TMPDIR
export TMPDIR="/home/users/${USER}/grant_620/scratch/${SLURM_JOBID}"

##ustawienie zmiennej aplikacji
export SCR=${TMPDIR}

##zmienna wejsciowa
IN_DIR=/home/users/pponiat/grant_620/project_data/paulina

##zmienna wyjsciowa
OUT_DIR=/home/users/pponiat/grant_620/project_data/paulina/trimmed3/wyniki

##utworzenie katalogu tymczasowego
mkdir -p ${TMPDIR}
##kopiowanie danych na TMPDIR00
cp $IN_DIR/GCF_003473485.1_MtrunA17r5.0-ANR_genomic.fna ${TMPDIR}
cp $IN_DIR/trimmed3 ${TMPDIR}
cp $IN_DIR/linie.txt ${TMPDIR}
cp $IN_DIR/skrypt_mapowanie2.sh ${TMPDIR}

cd $TMPDIR

##ladowanie modulu
module load bwa

##wykonanie polecenia
##fastqc SRR3458650.fastq.gz
##bwa index /home/users/pponiat/grant_620/project_data/paulina/GCF_003473485.1_MtrunA17r5.0$
##bwa mem /home/users/pponiat/grant_620/project_data/paulina/GCF_003473485.1_MtrunA17r5.0-A$

bash skrypt_mapowanie2.sh

##kopiowanie zawartosci katalogu do kat docelowego
cp -r $TMPDIR/* ${OUT_DIR}

##konczymy na dzis
rm -rf $TMPDIR

```
>plik skrypt_mapowanie.sh

```

#!/bin/bash


#cat /home/users/grant_620/project_data/paulina/linie.txt | while read line
cat linie.txt | while read line

        do
          	bwa mem -t 20 /home/users/pponiat/grant_620/project_data/paulina/GCF_003473485.1_MtrunA17r5.0-ANR_genomic.fna /home/users/pponiat/grant_620/project_data/paulina/trimmed3/${line}.fastq /home/users/pponiat/grant_620/project_data/paulina/trimmed3/${line}_2.fastq > aln-pe_${line}.sam.gz

                echo "$line"
        done


```
>plik linie.txt

```
HM007
HM008
HM010
HM011
HM014
HM015
HM019
HM020
HM021
HM025
HM034
itd.

```
# Obróbka danych po mapowaniu

```

samtools view -S -b aln-pe_HM008.sam.gz  > HM008.bam    -  konwertowanie z .sam do .bam

samtools view HM008.bam | head  - podglad
 
samtools sort HM008.bam -o HM008.sorted.bam  - sortowanie na moim komputerze

samtools sort aln-pe_${line}.bam ${line} - sortowanie na serwerze ! output tylko nazwa bez rozszerzenie, ponieważ zostanie usunięte !


samtools view HM008.sorted.bam | head  - podgląd 



```
# Analiza jakości mapowania 

> przygotowane wcześniej środowisko o nazwie quali 

> ewentualne problemu z aktywacją środowiska:

> Command conda not foud
```
export PATH="/home/users/pponiat/anaconda3/bin:$PATH"

```
> CommandNotFoundError: Your shell has not been properly configured to use 'conda activate'

```
source /home/users/pponiat/anaconda3/etc/profile.d/conda.sh

```
> aktywacja środowiska i instalacja programu QUALIMAP
```
conda activate quali

conda install -c bioconda qualimap

qualimap bamqc -bam HM008.sorted.bam -c -outdir HM008 --java-mem-size=4G - analiza jakości pojedynczego pliku .bam 

```


# Picard

```
java -jar picard.jar MarkDuplicates       I=HM008.sorted.bam       O=marked_duplicates.bam       M=marked_dup_metrics.txt

~/anaconda3/pkgs/picard-2.27.4-hdfd78af_0/share/picard-2.27.4-0

java -jar ~/anaconda3/pkgs/picard-2.27.4-hdfd78af_0/share/picard-2.27.4-0/picard.jar MarkDuplicates       I=HM019.sorted.bam       O=marked_duplicates_19.bam       M=marked_dup_metrics_19.txt

```

# CNVNATOR

```
:/media/pgr/Pulp/mapowanie_1$ cnvnator -root out.root -tree marked_duplicates.bam -chrom NC_053042.1 NC_053043.1 NC_053044.1 NC_053045.1 NC_053046.1 NC_053047.1 NC_053048.1 NC_053049.1
```

```
cnvnator -root file.root -chrom NC_053042.1 NC_053043.1 NC_053044.1 NC_053045.1 NC_053046.1 NC_053047.1 NC_053048.1 NC_053049.1 -his 100 -d dir_with_genome_fa

dir_with_genome_fa -folder z oddzielnymi plikami chromosomów 

```

```
cnvnator -root out1.root -tree marked_duplicates_19.bam -chrom NC_053042.1 NC_053043.1 NC_053044.1 NC_053045.1 NC_053046.1 NC_053047.1 NC_053048.1 NC_053049.1

cnvnator -root out1.root -genome ref.fna -chrom NC_053042.1 -his 100 -fasta NC_053042.1.fna

cnvnator -root out1.root -genome ref.fna -chrom NC_053042.1 -stat 100 -fasta NC_053042.1.fna
cnvnator -root out1.root -genome ref.fna -chrom NC_053042.1 -partition 100 -fasta NC_053042.1.fna
cnvnator -root out1.root -genome ref.fna -chrom NC_053042.1 -call 100 -fasta NC_053042.1.fna
cnvnator -root out1.root -genome ref.fna -chrom NC_053042.1 -view 100 -fasta NC_053042.1.fna

>NC_053042.1:1-10000

```

