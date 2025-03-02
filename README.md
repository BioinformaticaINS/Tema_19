# Tema 19: Obtención, calidad y limpieza de lecturas

## Estructura de la práctica:

1. Instalación de programas
2. Obtención de los datos de secuenciación 
3. Ensamblaje de genomas de datos de secuenciación Illumina
4. Ensamblaje de genomas de datos de secuenciación Nanopore
5. Obtención de las metricas de los genomas ensamblados
6. Validación de los genomas ensamblados

## Metodología:

## 1. Instalación de programas

### Instalar los siguientes programas en un ambiente de conda

```bash
conda create -n assembly -c bioconda unicycler flye quast checkm-genome racon
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

## 2. Obtención de los datos de secuenciación 

```bash
cd ~/genomics/raw_data

### Datos de Illumina

gdown https://drive.google.com/uc?id=1fdMqDwYgCY3mzSkhUW1SZANlBA71MY7X

gdown https://drive.google.com/uc?id=12aOQWJk7rvDbssb44O3qNn9pRo641H7C

### Datos de Nanopore

gdown https://drive.google.com/uc?id=1hjDKjprl7yhbnoPP7dTOBZVCjN88QVbH
```

## 3. Ensamblaje de genomas de datos de secuenciación Illumina

```bash
cd ~/genomics

mkdir assembly

cd assembly

mkdir illumina

cd illumina

conda activate assembly

unicycler -t 2 -1 /home/ins_user/genomics/raw_data/SRR19551969_R1.trim.fastq.gz -2 /home/ins_user/genomics/raw_data/SRR19551969_R2.trim.fastq.gz -o m01_unicycler_illumina

mv m01_unicycler_illumina/assembly.fasta m01_unicycler.fasta

mv m01_unicycler_illumina/assembly.gfa m01_unicycler.gfa
```
## 4. Ensamblaje de genomas de datos de secuenciación Nanopore

```bash
cd ~/genomics/assembly

mkdir nanopore

cd nanopore

flye --nano-raw /home/ins_user/genomics/raw_data/SRR19552033_1_trim.fastq.gz --threads 2 --genome-size 5m --out-dir m01_flye_nanopore
```

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

minimap2 -t 2 /home/ins_user/genomics/assembly/nanopore/m01_flye.fasta /home/ins_user/genomics/raw_data/SRR19551969_R1.trim.fastq.gz > flye.minimap4racon1.paf

racon -t 2 /home/ins_user/genomics/raw_data/SRR19551969_R1.trim.fastq.gz flye.minimap4racon1.paf /home/ins_user/genomics/assembly/nanopore/m01_flye.fasta > m01_flye.racon.fasta
```

## 5. Obtención de las metricas de los genomas ensamblados

```bash
cd ~/genomics/

mkdir validation

cd validation

quast.py -m 1000 -o quast /home/ins_user/genomics/assembly/illumina/m01_unicycler.fasta /home/ins_user/genomics/assembly/nanopore/m01_flye.fasta /home/ins_user/genomics/assembly/nanopore/racon/m01_flye.racon.fasta
```

## 6. Validación de los genomas ensamblados

```bash
cd ~/genomics/validation

mkdir checkm

cd checkm 

mkdir fasta

cp /home/ins_user/genomics/assembly/illumina/m01_unicycler.fasta /home/ins_user/genomics/assembly/nanopore/m01_flye.fasta /home/ins_user/genomics/assembly/nanopore/racon/m01_flye.racon.fasta fasta

checkm lineage_wf -t 2 -x fasta fasta . > checkm.out
```

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






