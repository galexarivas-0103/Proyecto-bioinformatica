# Comparación de perfiles de resistencia y virulencia entre cepas uropatógenas y comensales de *Escherichia coli*

## Descarga de datos

Inicialmente, se accedió a la base de datos **NCBI Assembly** para la obtención de genomas de *Escherichia coli*. En la barra de búsqueda se utilizó el término `"Escherichia coli"`.

Para el grupo asociado a infección urinaria (**UTI/UPEC**), se emplearon palabras clave como `"UTI"` y `"urine"` dentro de los resultados de búsqueda.

Adicionalmente, se restringió la selección a ensamblajes publicados en los últimos 10 años `(2016–2026)`, con el fin de trabajar con secuencias recientes y clínicamente relevantes. Finalmente, solo se clasificó a nivel *contig*.

Para el grupo comensal se aplicó el mismo procedimiento, utilizando la palabra clave `"commensal"` y restringiendo la selección a ensamblajes publicados entre `2016–2026`. Finalmente, también se clasificó a nivel *contig*.

Para descargar las secuencias, nos dirigimos a la sección **RefSeq** con el fin de descargar todos los contigs asociados a la cepa seleccionada.

Se descargaron todos los archivos en formato `FASTA`.

Finalmente, se descargaron:

* 50 archivos para las cepas comensales
* 50 archivos para las cepas uropatógenas

---

# ANÁLISIS EN ABRICATE

Para el análisis de genes específicos usamos la herramienta de línea de comandos **ABRicate**, que sirve para identificar computacionalmente genes de resistencia y virulencia en genomas bacterianos.

Esta se instaló en un entorno **Conda** independiente para asegurar la reproducibilidad y evitar conflictos de dependencias.

## Solicitud de sesión interactiva

```bash
salloc -N 1 -n 4 -p normal
```

Verificamos qué versión de Conda tenía disponible el clúster:

```bash
conda --version
```

En este caso, la versión disponible fue:

```bash
conda 4.9.2
```

Luego verificamos cuál instalación de Conda estábamos usando realmente:

```bash
which conda
```

Y apareció:

```bash
/opt/ohpc/pub/libs/gnu7/conda/anaconda3/bin/conda
```

Eso confirmó que el HPC ya tenía **Anaconda** instalado globalmente, así que no necesitábamos instalar Miniconda manualmente.

---

## Crear un entorno propio para el proyecto

Luego creamos nuestro entorno aislado con:

```bash
conda create -n ecoli_project python=3.9
```

Este paso es clave porque aquí dejamos de usar el entorno general del sistema y construimos uno propio.

Un entorno Conda funciona como una “caja independiente” donde instalamos solo lo necesario para el proyecto.

En este caso lo llamamos `ecoli_project`.

La razón de hacerlo es evitar conflictos entre versiones de software y asegurar que todos los análisis sean reproducibles.

Después activamos el entorno con:

```bash
conda activate ecoli_project
```

Hay que activar el entorno porque, de lo contrario, seguiríamos trabajando en el sistema base.

Al activarlo, todo lo que instalemos o ejecutemos se dirige exclusivamente a ese espacio aislado, sin afectar el resto del sistema.

Después de ejecutar el comando debe aparecer:

```bash
(ecoli_project)
```

En lugar de:

```bash
(base)
```

---

## Instalar ABRicate

Una vez verificamos que estábamos trabajando en el ambiente `(ecoli_project)` del clúster, comenzamos la instalación de ABRicate:

```bash
conda install -c conda-forge -c bioconda abricate
```

Una vez instalado, comprobamos que el programa estuviera disponible:

```bash
abricate --version
```

Luego ejecutamos:

```bash
abricate --check
```

Esto es importante porque ABRicate no solo depende de estar instalado, sino también de que herramientas internas como **BLAST** funcionen correctamente.

Este comando verifica que todo el sistema de búsqueda de genes esté operativo.

---

## Descargar bases de datos

Después descargamos las bases de datos necesarias con:

```bash
abricate --setupdb
```

Este paso es fundamental porque ABRicate por sí solo no hace nada si no tiene bases de datos como **CARD** o **VFDB**.

Estas bases contienen los genes conocidos de resistencia y virulencia que vamos a comparar contra nuestros genomas.

Finalmente, verificamos qué bases quedaron instaladas con:

```bash
abricate --list
```

Observamos que sí estaban las bases que necesitábamos, en este caso **CARD** y **VFDB**.

| DATABASE | SEQUENCES | DBTYPE | DATE |
|---|---|---|---|
| argannot | 2224 | nucl | 2026-May-18 |
| bacmet2 | 746 | prot | 2026-May-18 |
| card | 6052 | nucl | 2026-May-18 |
| ecoh | 597 | nucl | 2026-May-18 |
| ecoli_vf | 2701 | nucl | 2026-May-18 |
| megares | 6635 | nucl | 2026-May-18 |
| ncbi | 8232 | nucl | 2026-May-18 |
| plasmidfinder | 488 | nucl | 2026-May-18 |
| resfinder | 3206 | nucl | 2026-May-18 |
| upec_expec_vf | 77 | nucl | 2026-May-18 |
| vfdb | 4592 | nucl | 2026-May-18 |
| victors | 4545 | nucl | 2026-May-18 |

---

## Crear carpetas para resultados

```bash
mkdir UTI_CARD_tabs
mkdir commensal_CARD_tabs

mkdir UTI_VFDB_tabs
mkdir commensal_VFDB_tabs
```

---

## Ejecutar ABRicate para CARD

Después ejecutamos ABRicate usando la base de datos **CARD**.

Esta base contiene genes asociados con resistencia a antibióticos.

No analizamos un solo archivo manualmente, sino que usamos un bucle `for` para recorrer automáticamente todos los genomas `.fna` de la carpeta.

```bash
for f in UTI_fna/*.fna
do
    base=$(basename $f .fna)
    abricate --db card $f > UTI_CARD_tabs/${base}.tab
done

for f in commensal_fna/*.fna
do
    base=$(basename $f .fna)
    abricate --db card $f > commensal_CARD_tabs/${base}.tab
done
```

---

## Ejecutar ABRicate para VFDB

Después realizamos el mismo procedimiento, pero ahora usando la base de datos **VFDB**.

Esta base contiene genes relacionados con virulencia bacteriana.

```bash
for f in UTI_fna/*.fna
do
    base=$(basename $f .fna)
    abricate --db vfdb $f > UTI_VFDB_tabs/${base}.tab
done

for f in commensal_fna/*.fna
do
    base=$(basename $f .fna)
    abricate --db vfdb $f > commensal_VFDB_tabs/${base}.tab
done
```

---

## Generar summaries

Generamos los archivos *summary* porque los resultados individuales de ABRicate contienen demasiado detalle y no están organizados de la mejor manera para hacer comparaciones globales entre cepas.

Estos comandos toman todos los archivos `.tab` generados previamente y los combinan en una sola tabla resumen.

### CARD UTI

```bash
abricate --summary UTI_CARD_tabs/*.tab > UTI_CARD_summary.tab
```

### CARD comensales

```bash
abricate --summary commensal_CARD_tabs/*.tab > commensal_CARD_summary.tab
```

### VFDB UTI

```bash
abricate --summary UTI_VFDB_tabs/*.tab > UTI_VFDB_summary.tab
```

### VFDB comensales

```bash
abricate --summary commensal_VFDB_tabs/*.tab > commensal_VFDB_summary.tab
```

---

# R

## Identificar archivos `.tab`

```r
uti_files <- list.files("UTI_CARD_tabs",
                        pattern="*.tab",
                        full.names=TRUE)

com_files <- list.files("commensal_CARD_tabs",
                        pattern="*.tab",
                        full.names=TRUE)
```

---

## Crear función para extraer genes

```r
get_genes <- function(file){

    tab <- read.delim(file,
                      header=TRUE,
                      stringsAsFactors=FALSE)

    genes <- unique(tab$GENE)

    return(genes)
}
```

Esta función sirve para abrir un archivo `.tab` individual y extraer únicamente los nombres de genes encontrados.

---

## Extraer genes de todos los genomas

```r
uti_gene_lists <- lapply(uti_files, get_genes)

com_gene_lists <- lapply(com_files, get_genes)
```

Revisamos los genes obtenidos:

```r
uti_gene_lists[[1]]
```

Los resultados mostraron genes como:

* acrB
* TolC
* mdtA
* emrB

Confirmando que sí estábamos leyendo correctamente los genes de resistencia.

---

## Obtener todos los genes únicos

```r
all_uti_genes <- unique(unlist(uti_gene_lists))

all_com_genes <- unique(unlist(com_gene_lists))

all_genes <- unique(c(all_uti_genes,
                      all_com_genes))
```

* `unlist()` convierte todas las listas de genes en un único vector
* `unique()` elimina duplicados
* `c()` une los genes UTI y comensales

El resultado final fue una lista completa de todos los genes encontrados en cualquier cepa.

---

## Construcción de matrices de presencia/ausencia

Creamos matrices binarias.

```r
uti_matrix <- sapply(all_genes, function(gene){

    sapply(uti_gene_lists, function(x){

        as.integer(gene %in% x)

    })

})
```

Repetimos lo mismo para comensales:

```r
com_matrix <- sapply(all_genes, function(gene){

    sapply(com_gene_lists, function(x){

        as.integer(gene %in% x)

    })

})
```

Las filas representan genomas.

Las columnas representan genes.

Cada valor significa:

* `1` → el gen está presente
* `0` → el gen está ausente

---

## Convertir matrices a data.frame

```r
uti_matrix <- as.data.frame(uti_matrix)

com_matrix <- as.data.frame(com_matrix)
```

Esto facilita el manejo posterior de tablas y análisis estadísticos.

---

## Agregar nombres de filas

```r
rownames(uti_matrix) <- basename(uti_files)

rownames(com_matrix) <- basename(com_files)
```

---

## Contar genes por genoma

```r
uti_counts <- rowSums(uti_matrix)

com_counts <- rowSums(com_matrix)
```

`rowSums()` suma todos los valores de cada fila.

Como los datos son `0` y `1`, la suma representa el número total de genes presentes en cada cepa.

---

## Construir boxplots

```r
pdf("Resistance_boxplot.pdf")

boxplot(uti_counts,
        com_counts,

        names=c("UTI","Commensal"),

        main="Genes de resistencia",

        ylab="Número de genes")

dev.off()
```

---

## Calcular frecuencias génicas

```r
uti_freq <- colSums(uti_matrix) /
            nrow(uti_matrix) * 100

com_freq <- colSums(com_matrix) /
            nrow(com_matrix) * 100
```

* `colSums()` suma cada columna
* `nrow()` obtiene el número total de genomas

Como los valores son `0` y `1`, la suma representa cuántos genomas tienen ese gen.

---

## Crear tabla conjunta

```r
freq_table <- data.frame(
    Gene = names(uti_freq),

    UTI_Frequency = uti_freq,

    Commensal_Frequency = com_freq
)
```

Esto creó una tabla comparativa entre ambos grupos.

---

## Calcular diferencias entre grupos

```r
freq_table$Difference <- abs(
    freq_table$UTI_Frequency -
    freq_table$Commensal_Frequency
)
```

Esto mide qué tan diferente es la frecuencia de un gen entre UTI y comensales.

---

## Ordenar genes más diferenciales

```r
freq_table <- freq_table[
    order(-freq_table$Difference),
]

write.csv(freq_table,
          "Resistance_gene_frequencies.csv",
          row.names=FALSE)
```

El signo `-` ordena de mayor a menor.

---

## Seleccionar genes más importantes

```r
top_genes <- freq_table$Gene[1:15]
```

Elegimos los 15 genes con mayor diferencia entre grupos.

---

## Construir gráficos de barras

```r
pdf("Resistance_gene_frequencies_barplot.pdf",
    width=12,
    height=7)

barplot(
    rbind(freq_table$UTI_Frequency[1:15],
          freq_table$Commensal_Frequency[1:15]),

    beside=TRUE,

    names.arg=top_genes,

    las=2,

    ylab="Frecuencia (%)",

    main="Frecuencia de genes de resistencia",

    legend.text=c("UTI","Commensal")
)

dev.off()
```

---

## Repetimos todo para VFDB

Finalmente, realizamos exactamente el mismo procedimiento para los genes de virulencia usando:

* `UTI_VFDB_tabs`
* `commensal_VFDB_tabs`
````
