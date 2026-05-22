# Comparación de perfiles de resistencia y virulencia entre cepas uropatógenas y comensales de *Escherichia coli*

# 1.Descarga de datos

Inicialmente, se accedió a la base de datos **NCBI Assembly** para la obtención de genomas de *Escherichia coli*. En la barra de búsqueda se utilizó el término `"Escherichia coli"`.

<img width="1492" height="662" alt="NCBI1" src="https://github.com/user-attachments/assets/89054b52-359b-4d0f-a3c5-71d06d059d00" />

Para el grupo asociado a infección urinaria (**UTI/UPEC**), se emplearon palabras clave como `"UTI"` y `"urine"` dentro de los resultados de búsqueda.

Adicionalmente, se restringió la selección a ensamblajes publicados en los últimos 10 años `(2016–2026)`, con el fin de trabajar con secuencias recientes y clínicamente relevantes. Finalmente, solo se clasificó a nivel *contig*.

<img width="1477" height="652" alt="NCBI2" src="https://github.com/user-attachments/assets/c1010daa-78e7-4196-8e11-124927ba149d" />

Para el grupo comensal se aplicó el mismo procedimiento, utilizando la palabra clave `"commensal"` y restringiendo la selección a ensamblajes publicados entre `2016–2026`. Finalmente, también se clasificó a nivel *contig*.

<img width="1537" height="672" alt="NCBI3" src="https://github.com/user-attachments/assets/a25927ef-f700-4f20-9c7b-8667b51f7399" />

Para descargar las secuencias, nos dirigimos aL botón de  **Download** y seleccionamos la opción **Download package** 

<img width="960" height="665" alt="NCBI4" src="https://github.com/user-attachments/assets/5c99c596-422f-45a0-aaab-7999656c9a6c" />

Se descargaron todos los archivos .zip y en la terminal los decomprimimos y obtuvimos las secuencias.

Finalmente, se descargaron:

* 50 archivos para las cepas comensales
* 50 archivos para las cepas uropatógenas

---

# 2.ANÁLISIS EN ABRICATE

Para el análisis de genes específicos usamos la herramienta de línea de comandos **ABRicate**, que sirve para identificar computacionalmente genes de resistencia y virulencia en genomas bacterianos.

Esta se instaló en un entorno **Conda** independiente para asegurar la reproducibilidad y evitar conflictos de dependencias.

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
| argannot | 2224 | nucl | 2026-May-07 |
| bacmet2 | 746 | prot | 2026-May-07 |
| card | 6052 | nucl | 2026-May-07 |
| ecoh | 597 | nucl | 2026-May-07 |
| ecoli_vf | 2701 | nucl | 2026-May-07 |
| megares | 6635 | nucl | 2026-May-07 |
| ncbi | 8232 | nucl | 2026-May-07 |
| plasmidfinder | 488 | nucl | 2026-May-07 |
| resfinder | 3206 | nucl | 2026-May-07 |
| upec_expec_vf | 77 | nucl | 2026-May-07 |
| vfdb | 4592 | nucl | 2026-May-07 |
| victors | 4545 | nucl | 2026-May-07 |

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
Esto es lo que vemos al abrir los archivos .tab:
<img width="1367" height="481" alt="tabla_tab" src="https://github.com/user-attachments/assets/6ad98827-8f6f-4b26-a59e-84ddf377def8" />

---

# 3.Análisis en R

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

Así se ven las matrices que obtenemos:

<img width="1681" height="382" alt="matriz" src="https://github.com/user-attachments/assets/61fc019e-6a25-484e-993b-ce909f5cef8f" />


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
<img width="1557" height="631" alt="boxplot" src="https://github.com/user-attachments/assets/36bedd6a-a37c-4cbb-ab8c-29e4eecee348" />

Este gráfico ompara la distribución del número total de genes entre cepas UTI y cepas comensales.

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
(elegimos 15 por cuestiones de facilidad para observar los datos en la siguiente gráfica)

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
<img width="1812" height="572" alt="barplot" src="https://github.com/user-attachments/assets/0da6b95d-fff5-4dda-b1ce-e597a2b1d16a" />

---
En esta gráfica visualizamos algunos genes de los que más diferencian a ambos grupos
Cada gen tiene:
una barra para UTI y una barra para comensales. Esto permite visualizar rápidamente: genes enriquecidos, genes compartidos y genes exclusivos.

## Se repitió todo para VFDB
Para poder obtener las gráficas anteriores, realizamos exactamente el mismo procedimiento para los genes de virulencia usando:
*UTI_VFDB_tabs
*commensal_VFDB_tabs
# 4.Análisis estádistico 
Primero realizamos la prueba exacta de Fisher, porque esta prueba nos permitió identificar si los genes estaban significativamente asociados a las cepas uropatógenas (UTI/UPEC) y comensales.

Esta fue la prueba estadística principal de nuestro proyecto porque responde directamente a la pregunta biológica central:

## ¿Existen genes de resistencia o virulencia significativamente diferentes entre cepas UTI y comensales?

Para realizar el análisis comenzamos cargando nuevamente las matrices binarias de presencia/ausencia.

```r

uti_res <- read.csv(
    "UTI_presence_absence.csv",
    row.names=1,
    check.names=FALSE
)

com_res <- read.csv(
    "Commensal_presence_absence.csv",
    row.names=1,
    check.names=FALSE
)

uti_vf <- read.csv(
    "UTI_VFDB_presence_absence_matrix.csv",
    row.names=1,
    check.names=FALSE
)

com_vf <- read.csv(
    "Commensal_VFDB_presence_absence_matrix.csv",
    row.names=1,
    check.names=FALSE
)
```

Después unimos los grupos UTI y comensales en una sola matriz.

```r

combined_res <- rbind(
    uti_res,
    com_res
)

combined_vf <- rbind(
    uti_vf,
    com_vf
)
```

Usamos `rbind()` porque necesitábamos comparar simultáneamente todas las cepas.

Luego creamos un vector indicando a qué grupo pertenece cada genoma.

```r

group_res <- c(
    rep("UTI", nrow(uti_res)),
    rep("Commensal", nrow(com_res))
)

group_vf <- c(
    rep("UTI", nrow(uti_vf)),
    rep("Commensal", nrow(com_vf))
)
```

Esto fue importante porque Fisher compara frecuencias entre grupos.

## Prueba exacta de Fisher

Después aplicamos Fisher gen por gen.

#### Fisher para genes de resistencia

```r
fisher_results <- lapply(

    colnames(combined_res),

    function(gene){

        tab <- table(
            combined_res[,gene],
            group_res
        )

        if(nrow(tab)==2 & ncol(tab)==2){

            test <- fisher.test(tab)

            data.frame(
                Gene = gene,
                Pvalue = test$p.value
            )

        }

})
```

## Corrección por múltiples pruebas

Después corregimos los valores de p.

```r
fisher_results$Adjusted_P <- p.adjust(
    fisher_results$Pvalue,
    method="BH"
)
```

Esto fue fundamental porque realizamos decenas de pruebas simultáneamente.

Si no corrigiéramos los valores de p, aparecerían falsos positivos por azar.

Usamos el método BH (Benjamini-Hochberg), que controla la tasa de falsos descubrimientos.

## Ordenar genes más significativos

```r
fisher_results <- fisher_results[
    order(fisher_results$Adjusted_P),
]
```

Esto organiza la tabla desde los genes más significativos hasta los menos significativos.

así se ven nuestras tablas de fisher:

<img width="551" height="687" alt="tabla_fisher" src="https://github.com/user-attachments/assets/94ec6fa8-0d92-4ec6-be1c-9970fd1b0067" />


## Filtrar genes significativos
Ya que son tantos genes, filtramos solo los significativos
```r
significant_fisher_res <- fisher_results[
    fisher_results$Adjusted_P < 0.05,
]
```

## Contar genes significativos
posteriormente los contamos
```r
nrow(significant_fisher_res)
```
<img width="495" height="107" alt="fisher_resistencia" src="https://github.com/user-attachments/assets/2100a563-b94e-45c9-be84-f2165bc216ea" />

Esto nos indicó cuántos genes de resistencia resultaron significativamente diferentes entre grupos.

#### Fisher para genes de virulencia

Después repetimos exactamente el mismo procedimiento para virulencia.

```r
fisher_virulence <- lapply(

    colnames(combined_vf),

    function(gene){

        tab <- table(
            combined_vf[,gene],
            group_vf
        )

        if(nrow(tab)==2 & ncol(tab)==2){

            test <- fisher.test(tab)

            data.frame(
                Gene = gene,
                Pvalue = test$p.value
            )

        }

})
```

Eliminar resultados vacíos:

```r
fisher_virulence <- fisher_virulence[
    !sapply(fisher_virulence, is.null)
]
```

Unir resultados:

```r
fisher_virulence <- do.call(
    rbind,
    fisher_virulence
)
```

Corregir p-values:

```r
fisher_virulence$Adjusted_P <- p.adjust(
    fisher_virulence$Pvalue,
    method="BH"
)
```

Ordenar:

```r
fisher_virulence <- fisher_virulence[
    order(fisher_virulence$Adjusted_P),
]
```

Filtrar genes significativos:

```r
significant_fisher_vf <- fisher_virulence[
    fisher_virulence$Adjusted_P < 0.05,
]
```

Contar genes significativos:

```r
nrow(significant_fisher_vf)
```
<img width="502" height="107" alt="fisher_virulencia" src="https://github.com/user-attachments/assets/2d210b80-cacf-4898-a5ef-a324bcb28d69" />


Estos análisis nos permitieron identificar genes de virulencia diferencialmente distribuidos entre ambos grupos bacterianos.

Después de identificar genes significativamente diferentes, quisimos analizar si las cepas también presentaban diferencias globales en sus perfiles génicos completos.

Para esto realizamos PCA.

# PCA (Análisis de Componentes Principales)

Primero eliminamos genes sin variación.

## Filtrado resistencia

```r
res_filtered <- combined_res[
    ,
    apply(combined_res, 2, var) != 0
]
```

## Filtrado virulencia

```r
vf_filtered <- combined_vf[
    ,
    apply(combined_vf, 2, var) != 0
]
```

Este paso fue necesario porque PCA no puede trabajar con columnas constantes.

Los genes presentes en todas las cepas o ausentes en todas las cepas no aportan información.

# PCA resistencia

```r
pca_res <- prcomp(
    res_filtered,
    scale.=TRUE
)
```

# PCA virulencia

```r
pca_vf <- prcomp(
    vf_filtered,
    scale.=TRUE
)
```

# Revisar varianza explicada

```r
summary(pca_res)

summary(pca_vf)
```

Aquí observamos qué porcentaje de variación explica cada componente principal.

<img width="891" height="667" alt="summary_pca_resistencia" src="https://github.com/user-attachments/assets/7d250d11-02a2-4f91-9a3e-8dacd46e014a" />

<img width="767" height="926" alt="summary_pca_virulencia" src="https://github.com/user-attachments/assets/1e2a0a41-72a6-4841-b93a-21453bc1e93e" />

# Graficar PCA resistencia

```r
pdf("PCA_resistance.pdf")

plot(
    pca_res$x[,1],
    pca_res$x[,2],

    col=ifelse(group_res=="UTI","red","blue"),

    pch=19,

    xlab="PC1",
    ylab="PC2",

    main="PCA Resistance Profiles"
)

legend(
    "topright",
    legend=c("UTI","Commensal"),
    col=c("red","blue"),
    pch=19
)

dev.off()
```
<img width="858" height="863" alt="PCA_grafica_resistencia" src="https://github.com/user-attachments/assets/f1be6ea7-021e-4390-b57f-4e539f9c2d3c" />

# Graficar PCA virulencia

```r
pdf("PCA_virulence.pdf")

plot(
    pca_vf$x[,1],
    pca_vf$x[,2],

    col=ifelse(group_vf=="UTI","red","blue"),

    pch=19,

    xlab="PC1",
    ylab="PC2",

    main="PCA Virulence Profiles"
)

legend(
    "topright",
    legend=c("UTI","Commensal"),
    col=c("red","blue"),
    pch=19
)

dev.off()
```
<img width="858" height="857" alt="PCA_grafica_virulencia" src="https://github.com/user-attachments/assets/605edb39-757f-46ca-a7db-6cd0e716b1c8" />

Interpretación:

* grupos separados → perfiles distintos
* grupos mezclados → perfiles similares


# Clustering jerárquico

Finalmente realizamos clustering.

# Distancias resistencia

```r
dist_res <- dist(res_filtered)
```

# Distancias virulencia

```r
dist_vf <- dist(vf_filtered)
```

`dist()` calcula similitud entre cepas.

Después construimos dendrogramas.

# Clustering resistencia

```r
hc_res <- hclust(dist_res)

pdf("Clustering_resistance.pdf",
    width=12,
    height=8)

plot(
    hc_res,
    labels=group_res,

    main="Hierarchical Clustering Resistance"
)

dev.off()
```

<img width="1095" height="688" alt="clustering_resistencia" src="https://github.com/user-attachments/assets/4908a55e-71b5-4fc6-87c8-7277e7b27b01" />

# Clustering virulencia

```r
hc_vf <- hclust(dist_vf)

pdf("Clustering_virulence.pdf",
    width=12,
    height=8)

plot(
    hc_vf,
    labels=group_vf,

    main="Hierarchical Clustering Virulence"
)

dev.off()
```
<img width="1092" height="637" alt="clustering_virulencia" src="https://github.com/user-attachments/assets/a1833c34-5d49-4e7b-9295-7bb84409b99c" />

El clustering agrupa automáticamente cepas similares.

Interpretación:

* ramas cercanas → perfiles similares
* ramas lejanas → perfiles diferentes


