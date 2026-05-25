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

Descargar secuencias [aquí](https://drive.google.com/file/d/1eLkxCXPui6gkMq24YDUd9jENB94DtrUw/view?usp=sharing)

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
<img width="1333" height="692" alt="tabla_tab" src="https://github.com/user-attachments/assets/9e32260f-6bd8-4670-9cd5-f3e5073427b5" />

Descargar archivos .tab [aquí](https://drive.google.com/file/d/18Yjr8L0Z5kgpmOS2ll3Shho7fWvZHWPU/view?usp=sharing)

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

<img width="1802" height="477" alt="matriz" src="https://github.com/user-attachments/assets/65a0d76b-3971-462c-9a76-fc8073644794" />

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

<img width="535" height="670" alt="tabla_fisher" src="https://github.com/user-attachments/assets/b992eec3-a862-435f-86fc-dca3afbe7f7f" />

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

Esta fue la cantidad de genes de resistencia que resultaron significativamente diferentes entre grupos:

```r
significant_fisher_res <- fisher_results[
  fisher_results$Adjusted_P < 0.05,
]

nrow(significant_fisher_res)
[1] 8
```

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

Filtrar genes significativos y contarlos:

```r
significant_fisher_vf <- fisher_virulence[
  fisher_virulence$Adjusted_P < 0.05,
]

nrow(significant_fisher_vf)
[1] 134
```

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

## PCA resistencia

```r
pca_res <- prcomp(
    res_filtered,
    scale.=TRUE
)
```

## PCA virulencia

```r
pca_vf <- prcomp(
    vf_filtered,
    scale.=TRUE
)
```

## Revisar varianza explicada

```r
summary(pca_res)

summary(pca_vf)
```

Aquí observamos qué porcentaje de variación explica cada componente principal.

Resistencia:

```r
summary(pca)

Importance of components:
                         PC1     PC2     PC3     PC4     PC5     PC6     PC7
Standard deviation     3.0145  2.3044  2.0327  1.84107 1.74121 1.50513 1.44274
Proportion of Variance 0.2216  0.1295  0.1008  0.08267 0.07395 0.05525 0.05077
Cumulative Proportion  0.2216  0.3512  0.4519  0.53460 0.60854 0.66380 0.71457

                         PC8     PC9     PC10    PC11    PC12    PC13    PC14
Standard deviation     1.33657 1.21985 1.14329 1.0365  1.03069 1.00600 0.9732
Proportion of Variance 0.04357 0.03629 0.03188 0.0262  0.02591 0.02468 0.0231
Cumulative Proportion  0.75814 0.79443 0.82631 0.8525  0.87843 0.90311 0.9262

                         PC15    PC16    PC17    PC18    PC19    PC20    PC21
Standard deviation     0.88524 0.76120 0.70615 0.6242  0.50259 0.46124 0.37781
Proportion of Variance 0.01911 0.01413 0.01216 0.0095  0.00616 0.00519 0.00348
Cumulative Proportion  0.94533 0.95946 0.97162 0.9811  0.98728 0.99247 0.99595

                         PC22    PC23    PC24     PC25     PC26     PC27
Standard deviation     0.37302 0.16354 4.714e-16 3.777e-16 2.468e-16 1.981e-16
Proportion of Variance 0.00339 0.00065 0.000e+00 0.000e+00 0.000e+00 0.000e+00
Cumulative Proportion  0.99935 1.00000 1.000e+00 1.000e+00 1.000e+00 1.000e+00

                         PC28     PC29     PC30     PC31     PC32
Standard deviation     1.981e-16 1.981e-16 1.981e-16 1.981e-16 1.981e-16
Proportion of Variance 0.000e+00 0.000e+00 0.000e+00 0.000e+00 0.000e+00
Cumulative Proportion  1.000e+00 1.000e+00 1.000e+00 1.000e+00 1.000e+00

                         PC33     PC34     PC35     PC36     PC37
Standard deviation     1.981e-16 1.981e-16 1.981e-16 1.981e-16 1.981e-16
Proportion of Variance 0.000e+00 0.000e+00 0.000e+00 0.000e+00 0.000e+00
Cumulative Proportion  1.000e+00 1.000e+00 1.000e+00 1.000e+00 1.000e+00

                         PC38     PC39     PC40     PC41
Standard deviation     1.981e-16 1.981e-16 1.981e-16 8.768e-17
Proportion of Variance 0.000e+00 0.000e+00 0.000e+00 0.000e+00
Cumulative Proportion  1.000e+00 1.000e+00 1.000e+00 1.000e+00
```

Virulencia:
```r
summary(pca_vf)

Importance of components:
                         PC1     PC2     PC3     PC4     PC5     PC6     PC7
Standard deviation     7.6143  7.0535  4.19205 3.79096 3.61782 3.44129 2.94616
Proportion of Variance 0.2357  0.2022  0.0714  0.05842 0.05321 0.04814 0.03528
Cumulative Proportion  0.2357  0.4379  0.50936 0.56778 0.62098 0.66912 0.70441

                         PC8     PC9     PC10    PC11    PC12    PC13    PC14
Standard deviation     2.89506 2.45107 2.41566 2.16614 1.98681 1.91664 1.79635
Proportion of Variance 0.03407 0.02442 0.02372 0.01907 0.01605 0.01493 0.01312
Cumulative Proportion  0.73848 0.76290 0.78662 0.80570 0.82174 0.83667 0.84979

                         PC15    PC16    PC17    PC18    PC19    PC20    PC21
Standard deviation     1.72951 1.6224  1.59382 1.51365 1.46052 1.38051 1.35278
Proportion of Variance 0.01216 0.0107  0.01033 0.00931 0.00867 0.00775 0.00744
Cumulative Proportion  0.86195 0.8727  0.88298 0.89229 0.90096 0.90871 0.91615

                         PC22    PC23    PC24    PC25    PC26    PC27    PC28
Standard deviation     1.29284 1.1844  1.12119 1.02725 1.01095 0.95587 0.93739
Proportion of Variance 0.00679 0.0057  0.00511 0.00429 0.00415 0.00371 0.00357
Cumulative Proportion  0.92294 0.9286  0.93376 0.93805 0.94220 0.94591 0.94949

                         PC29    PC30    PC31    PC32    PC33    PC34    PC35
Standard deviation     0.92138 0.90634 0.86056 0.85430 0.83295 0.81625 0.79667
Proportion of Variance 0.00345 0.00334 0.00301 0.00297 0.00282 0.00271 0.00258
Cumulative Proportion  0.95294 0.95628 0.95929 0.96225 0.96507 0.96778 0.97036

                         PC36    PC37    PC38    PC39    PC40    PC41    PC42
Standard deviation     0.77647 0.7518  0.72253 0.68916 0.67320 0.66133 0.6279
Proportion of Variance 0.00245 0.0023  0.00212 0.00193 0.00184 0.00178 0.0016
Cumulative Proportion  0.97281 0.9751  0.97723 0.97916 0.98101 0.98278 0.9844

                         PC43    PC44    PC45    PC46    PC47    PC48    PC49
Standard deviation     0.59622 0.57705 0.56092 0.52983 0.48600 0.48094 0.46707
Proportion of Variance 0.00145 0.00135 0.00128 0.00114 0.00096 0.00094 0.00089
Cumulative Proportion  0.98583 0.98719 0.98846 0.98961 0.99057 0.99151 0.99239

                         PC50    PC51    PC52    PC53    PC54    PC55    PC56
Standard deviation     0.4442  0.43183 0.40045 0.39270 0.36102 0.34419 0.31914
Proportion of Variance 0.0008  0.00076 0.00065 0.00063 0.00053 0.00048 0.00041
Cumulative Proportion  0.9932  0.99395 0.99460 0.99523 0.99576 0.99624 0.99666

                         PC57    PC58    PC59    PC60    PC61    PC62    PC63
Standard deviation     0.31611 0.31105 0.28746 0.26695 0.25796 0.23580 0.23092
Proportion of Variance 0.00041 0.00039 0.00034 0.00029 0.00027 0.00023 0.00022
Cumulative Proportion  0.99706 0.99746 0.99779 0.99808 0.99835 0.99858 0.99880

                         PC64    PC65    PC66    PC67    PC68    PC69    PC70
Standard deviation     0.2200  0.20365 0.19592 0.17661 0.16313 0.1570  0.13902
Proportion of Variance 0.00020 0.00017 0.00016 0.00013 0.00011 0.00010 0.00008
Cumulative Proportion  0.99900 0.99916 0.99932 0.99944 0.99955 0.99970 0.99973

                         PC71    PC72    PC73    PC74    PC75    PC76    PC77
Standard deviation     0.12689 0.11535 0.09460 0.09064 0.07993 0.07516 0.06302
Proportion of Variance 0.00007 0.00005 0.00004 0.00003 0.00003 0.00002 0.00002
Cumulative Proportion  0.99980 0.99985 0.99989 0.99992 0.99995 0.99997 0.99999

                         PC78    PC79    PC80     PC81      PC82      PC83
Standard deviation     0.04383 0.03745 0.01895 1.19e-15  8.636e-16 8.194e-16
```

## Graficar PCA resistencia

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

## Graficar PCA virulencia

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

## Distancias resistencia

```r
dist_res <- dist(res_filtered)
```

## Distancias virulencia

```r
dist_vf <- dist(vf_filtered)
```

`dist()` calcula similitud entre cepas.

Después construimos dendrogramas.

## Clustering resistencia

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

<img width="1491" height="845" alt="clustering_resistencia" src="https://github.com/user-attachments/assets/d9cbc09c-3d5d-460a-bff0-db53b60c0a77" />


## Clustering virulencia

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
<img width="1493" height="851" alt="clustering_virulencia" src="https://github.com/user-attachments/assets/d1998b85-c272-4160-8de9-6c7302c61ae2" />

El clustering agrupa automáticamente cepas similares.

Interpretación:

* ramas cercanas → perfiles similares
* ramas lejanas → perfiles diferentes


