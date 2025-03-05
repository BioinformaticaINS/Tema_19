# Tema 19: Ensamblaje de genomas

## Estructura de la práctica:

1. Instalación de programas
2. Obtención de los datos de secuenciación 
3. Ensamblaje de genomas de datos de secuenciación Illumina
4. Ensamblaje de genomas de datos de secuenciación Nanopore
5. Obtención de las metricas de los genomas ensamblados
6. Validación de los genomas ensamblados
7. Clasificación taxonómica
8. Identificación de SNPs

## Metodología:

## 1. Instalación de programas

### Instalar los siguientes programas en un ambiente de conda

```bash
conda create -n assembly -c bioconda unicycler flye quast checkm-genome racon barrnap aniclustermap snippy
```
> **Comentario:** 
> - `conda create`: Este es el comando base para crear un nuevo entorno Conda.
> - `-n assembly`: Esta opción especifica el nombre del nuevo entorno como "assembly". Los entornos virtuales son como carpetas aisladas donde puedes instalar diferentes versiones de software sin que interfieran con otras instalaciones.
> - `-c bioconda`: Esta opción indica que se debe buscar los paquetes en el canal "bioconda".
> - `unicycler`: Un ensamblador de genomas híbrido que combina lecturas de corta y larga longitud.
> - `flye`: Un ensamblador de genomas para lecturas de larga longitud.
> - `quast`: Una herramienta para evaluar la calidad de los ensamblajes de genomas.
> - `checkm-genome`: Una herramienta para evaluar la integridad y contaminación de los ensamblajes de genomas.
> - `racon`: Una herramienta para pulir ensamblajes de genomas utilizando lecturas de larga longitud.
> - `barrnap`: Es una herramienta bioinformática que se utiliza para predecir la ubicación de los genes de ARN ribosómico (ARNr) dentro de secuencias de ADN.
> - `aniclustermap`: Es una herramienta que utiliza el Average Nucleotide Identity (ANI) para crear mapas de clústeres genómicos, útil para la comparación y clasificación de genomas
> - `snippy`: Es una herramienta para la detección rápida de variantes genéticas (SNPs, INDELs) en genomas bacterianos, útil para estudios de genómica comparativa y epidemiología.

### Instalar el siguiente programa en Ubuntu

```bash
cd ~/genomics/software

wget https://github.com/rrwick/Bandage/releases/download/v0.8.1/Bandage_Ubuntu_dynamic_v0_8_1.zip

unzip Bandage_Ubuntu_dynamic_v0_8_1.zip
```

## 2. Obtención de los datos de secuenciación 

```bash
cd ~/genomics/raw_data

### Datos de Illumina

gdown https://drive.google.com/uc?id=1fdMqDwYgCY3mzSkhUW1SZANlBA71MY7X

gdown https://drive.google.com/uc?id=12aOQWJk7rvDbssb44O3qNn9pRo641H7C

### Datos de Nanopore

gdown https://drive.google.com/uc?id=1hjDKjprl7yhbnoPP7dTOBZVCjN88QVbH

### Genomas de referencia

gdown https://drive.google.com/uc?id=11ERl-rWpOnXR54ZelpdDLJIMONodbURC
```

## 3. Ensamblaje de genomas de datos de secuenciación Illumina

```bash
cd ~/genomics

mkdir assembly

cd assembly

mkdir illumina

cd illumina

conda activate assembly

### Ensamblaje de novo del genoma con unicycler

unicycler -t 2 -1 /home/ins_user/genomics/raw_data/SRR19551969_R1.trim.fastq.gz -2 /home/ins_user/genomics/raw_data/SRR19551969_R2.trim.fastq.gz -o m01_unicycler_illumina

mv m01_unicycler_illumina/assembly.fasta m01_unicycler.fasta

mv m01_unicycler_illumina/assembly.gfa m01_unicycler.gfa
```

## 4. Ensamblaje de genomas de datos de secuenciación Nanopore

```bash
cd ~/genomics/assembly

mkdir nanopore

cd nanopore

### Ensamblaje de novo del genoma con flye

flye --nano-raw /home/ins_user/genomics/raw_data/SRR19552033_1_trim.fastq.gz --threads 2 --genome-size 5m --out-dir m01_flye_nanopore
```

> **Comentario:** 
> - `--nano-raw`: Indica que las lecturas son datos de Nanopore (R9)
> - `--genome-size 5m`: Esta opción proporciona una estimación del tamaño del genoma que se va a ensamblar.

```bash
cat m01_flye_nanopore/assembly_info.txt 

#seq_name	length	cov.	circ.	repeat	mult.	alt_group	graph_path
contig_1	4809453	104	Y	N	1	*	1
contig_2	89088	146	Y	N	1	*	2
contig_3	17975	199	Y	Y	1	*	3
contig_4	16755	182	Y	N	2	*	4

mv m01_flye_nanopore/assembly.fasta m01_flye.fasta

mv m01_flye_nanopore/assembly_graph.gfa m01_flye.gfa
```

```bash
cd /home/ins_user/genomics/assembly/nanopore

mkdir racon

cd racon

### Alineamiento de las lecturas de Illumina sobre el genoma ensamblado con minimap2

minimap2 -t 2 /home/ins_user/genomics/assembly/nanopore/m01_flye.fasta /home/ins_user/genomics/raw_data/SRR19551969_R1.trim.fastq.gz > flye.minimap4racon1.paf

### Pulido del genoma con racon

racon -t 2 /home/ins_user/genomics/raw_data/SRR19551969_R1.trim.fastq.gz flye.minimap4racon1.paf /home/ins_user/genomics/assembly/nanopore/m01_flye.fasta > m01_flye.racon.fasta
```

## 5. Obtención de las metricas de los genomas ensamblados

```bash
cd ~/genomics/

mkdir validation

cd validation

### Calculo de las metricas de los genomas ensamblados

quast.py -m 1000 -o quast /home/ins_user/genomics/assembly/illumina/m01_unicycler.fasta /home/ins_user/genomics/assembly/nanopore/m01_flye.fasta /home/ins_user/genomics/assembly/nanopore/racon/m01_flye.racon.fasta
```

> **Comentario:** 
> - `-m 1000`: Esta opción establece la longitud mínima de contig para ser considerada en la evaluación a 1000 pares de bases. Solo los contigs que tengan al menos 1000 pares de bases de longitud se incluirán en el análisis. Esto filtra contigs cortos y potencialmente poco fiables.

## 6. Validación de los genomas ensamblados

```bash
cd ~/genomics/validation

mkdir checkm

cd checkm 

mkdir fasta

cp /home/ins_user/genomics/assembly/illumina/m01_unicycler.fasta /home/ins_user/genomics/assembly/nanopore/m01_flye.fasta /home/ins_user/genomics/assembly/nanopore/racon/m01_flye.racon.fasta fasta

### Validación de los genomas ensamblados con checkm

checkm lineage_wf -t 2 -x fasta fasta . > checkm.out
```

> **Comentario:** 
> - `lineage_wf`: Especifica el flujo de trabajo a utilizar. lineage_wf es un flujo de trabajo de CheckM que utiliza información filogenética (linaje) para seleccionar los marcadores genéticos más apropiados para la evaluación. Esto permite una evaluación más precisa y específica para el organismo en cuestión.
> - `-x fasta`: Especifica el formato de los archivos de entrada. En este caso, los archivos de entrada son genomas ensamblados en formato FASTA.
> - `fasta`: Es el directorio de entrada donde se encuentran los archivos FASTA de los genomas que se van a evaluar. CheckM buscará en este directorio todos los archivos con extensión ".fasta" y los analizará.
> - `.`: Indica el directorio de salida. En este caso, los resultados se guardarán en el directorio actual.
> - `checkm.out`: Es el nombre del archivo donde se guardará la salida del análisis de CheckM.

```bash
cat checkm.out

---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------
  Bin Id                    Marker lineage           # genomes   # markers   # marker sets    0     1     2    3   4   5+   Completeness   Contamination   Strain heterogeneity  
---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------
  m01_unicycler    f__Enterobacteriaceae (UID5167)       82         1240          324         4    1231   5    0   0   0       99.60            0.72              20.00          
  m01_flye.racon   f__Enterobacteriaceae (UID5167)       82         1240          324         4    1233   3    0   0   0       99.60            0.26               0.00          
  m01_flye         f__Enterobacteriaceae (UID5167)       82         1240          324        137   1089   14   0   0   0       88.87            1.16               0.00          
---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------
```

> **Comentario:** 
> - `Bin Id`: Identificador del ensamblaje del genoma.
> - `Marker lineage`: Linaje del marcador utilizado para la evaluación. 
> - `# genomes`: Número de genomas de referencia utilizados en la evaluación.
> - `# markers`: Número total de marcadores genéticos (genes) usados en la evaluación.
> - `# marker sets`: Número de conjuntos de marcadores usados.
> - `0, 1, 2, 3, 4, 5+`: Distribución de los marcadores en los conjuntos.
> - `Completeness`: Porcentaje de marcadores esperados que se encontraron en el ensamblaje. Un valor alto indica un ensamblaje más completo.
> - `Contamination`: Porcentaje de marcadores duplicados o inesperados, lo que sugiere posible contaminación. Un valor bajo es mejor.
> - `Strain heterogeneity`: Indica la posible presencia de múltiples cepas en el ensamblaje. Un valor alto sugiere heterogeneidad.

## 7. Clasificación taxonómica

```bash
cd ~/genomics/

mkdir taxonomy

cd taxonomy

### Identificación de las secuencias de rRNA con barrnap

barrnap /home/ins_user/genomics/assembly/nanopore/racon/m01_flye.racon.fasta --threads 2 --outseq m01_rna.fasta
```

```bash
grep ">" m01_rna.fasta 
>16S_rRNA::contig_1:4272477-4274015(-)
>16S_rRNA::contig_1:104633-106171(+)
>16S_rRNA::contig_1:237971-239509(+)
>16S_rRNA::contig_1:150-1686(+)
>16S_rRNA::contig_1:3552695-3554232(-)
>16S_rRNA::contig_1:278263-279797(+)
>16S_rRNA::contig_1:919730-921259(+)
>23S_rRNA::contig_1:921708-924609(+)
>23S_rRNA::contig_1:4269128-4272028(-)
>23S_rRNA::contig_1:239866-242765(+)
>23S_rRNA::contig_1:2043-4942(+)
>23S_rRNA::contig_1:3549439-3552338(-)
>23S_rRNA::contig_1:106619-109518(+)
>23S_rRNA::contig_1:280154-283050(+)
>5S_rRNA::contig_1:5039-5150(+)
>5S_rRNA::contig_1:109615-109726(+)
>5S_rRNA::contig_1:242862-242973(+)
>5S_rRNA::contig_1:283147-283258(+)
>5S_rRNA::contig_1:924706-924817(+)
>5S_rRNA::contig_1:3549231-3549342(-)
>5S_rRNA::contig_1:4268920-4269031(-)
>5S_rRNA::contig_1:4268675-4268786(-)
```

```bash
### Obtención de los genomas de referencia

unzip /home/ins_user/genomics/raw_data/genomes_ncbi.zip -d .

cp /home/ins_user/genomics/assembly/nanopore/racon/m01_flye.racon.fasta genomes_ncbi

### Análisis ANI (Average Nucleotide Identity)

ANIclustermap -i genomes_ncbi -o ANIclustermap_result --fig_width 20 --fig_height 15 --annotation
```

> **Comentario:** 
> - `-i genomes_ncbi`: Especifica el directorio de entrada donde se encuentran los archivos FASTA de los genomas.
> - `--fig_width 20`: Define el ancho de la figura del mapa de calor en pulgadas.
> - `--fig_height 15`: Define la altura de la figura del mapa de calor en pulgadas.
> - `--annotation`: Esta opción indica que se deben agregar anotaciones al mapa de calor. Las anotaciones suelen ser los valores de ANI entre los genomas, lo que facilita la interpretación del mapa.

## 8. Identificación de SNPs

```bash
cd ~/genomics/

mkdir variant

cd variant

### Identificación de SNPs utilizando snippy

snippy --cpus 2 --outdir snps --report --ref /home/ins_user/genomics/taxonomy/genomes_ncbi/Ssonnei_ATCC29930.fasta --R1 /home/ins_user/genomics/raw_data/SRR19551969_R1.trim.fastq.gz --R2 /home/ins_user/genomics/raw_data/SRR19551969_R2.trim.fastq.gz
```

```bash
head -20 m01_snps/snps.txt 

ReadFiles	/home/ins_user/genomics/raw_data/SRR19551969_R1.trim.fastq.gz /home/ins_user/genomics/raw_data/SRR19551969_R2.trim.fastq.gz
Reference	/home/ins_user/genomics/taxonomy/genomes_ncbi/Ssonnei_ATCC29930.fasta
ReferenceSize	4994001
Software	snippy 4.6.0
Variant-COMPLEX	29
Variant-DEL	83
Variant-INS	130
Variant-SNP	1437
VariantTotal	1679
```

```bash
head -20 m01_snps/snps.tab

CHROM	POS	TYPE	REF	ALT	EVIDENCE	FTYPE	STRAND	NT_POS	AA_POS	EFFECT	LOCUS_TAG	GENE	PRODUCT
NZ_CP026802.1	523	snp	T	C	C:45 T:0								
NZ_CP026802.1	1862	snp	G	A	A:49 G:0								
NZ_CP026802.1	10134	snp	G	A	A:52 G:0								
NZ_CP026802.1	11671	snp	G	A	A:64 G:0								
NZ_CP026802.1	13113	snp	A	G	G:31 A:0								
NZ_CP026802.1	16364	snp	T	C	C:42 T:0								
NZ_CP026802.1	22709	del	TA	T	T:35 TA:0								
NZ_CP026802.1	31415	ins	C	CCCAGCA	CCCAGCA:28 C:1								
NZ_CP026802.1	31645	del	GT	G	G:25 GT:0								
NZ_CP026802.1	32256	snp	C	T	T:42 C:0								
NZ_CP026802.1	34384	snp	G	T	T:12 G:0								
NZ_CP026802.1	38177	snp	A	C	C:47 A:0								
NZ_CP026802.1	44156	snp	C	T	T:43 C:0								
NZ_CP026802.1	53147	snp	T	C	C:55 T:0								
NZ_CP026802.1	67031	snp	C	A	A:52 C:0								
NZ_CP026802.1	67869	snp	C	T	T:23 C:0								
NZ_CP026802.1	69761	snp	A	G	G:12 A:0								
NZ_CP026802.1	74449	snp	C	T	T:38 C:0								
NZ_CP026802.1	74679	snp	C	T	T:42 C:0
```

```bash
head -20 m01_snps/snps.report.txt

~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
>NZ_CP026802.1:523 snp T=>C DP=45 Q=1499.17 [1]

        491       501       511       521       531       541       551         
ACAAGCTGGAACGGCAGCTCAATAAACTGCAGCACAAAGGTGAAGCACGTCGTGCCGCAACATCGGTGAAAGACGCCAAC
........................................C.......................................
...........        .....................C.......................................
............       ,,,,,,,,,,,,,,,,,,,,,c,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,
,,,,,,,,,,,,,,,,,,,,,,,,,,,,              ......................................
,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,  ,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,
........................................C...    ,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,
........             ...................C.......................................
,,,,,,,,,,,,,,,,,,,,,,,,, ,,,,,,,,,,,,,,c,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,
........................................C..             ,,,,,,,,,,,,,,,,,,,,,,,,
........................................C.......................................
,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,c,,               ,,,,,,,,,,,,,,,,,,,,,,
,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,c,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,
,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,c,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,
........................................C.......................................
```
