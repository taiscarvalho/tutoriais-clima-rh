Trabalho 1 - Clima e Recursos Hídricos
================
17 de maio de 2021

#### Passo 1: download dos dados da Climatic Research Unit (CRU)

Os dados estimados de precipitação da CRU são disponibilizados online e podem ser baixados manualmente. Os dados mais recentes de precipitação elaborados pela CRU (4.05) estão armazenados [nesse link](https://crudata.uea.ac.uk/cru/data/hrg/cru_ts_4.05/cruts.2103051243.v4.05/pre/). Os dados são disponibilizados no formato netCDF, bastante comum para dados climáticos. Baixe o arquivo, descompacte e armazene na mesma pasta em que o código estiver salvo ou em uma pasta chamada dados.

Esse tutorial explica de forma bem completa como manusear arquivos NetCDF no R: <https://thiagodossantos.com/pt/post/2-raster_basic_operations/>

``` r
library(ncdf4)
library(raster)
library(rgdal)
library(dplyr)
```

``` r
url <- "https://crudata.uea.ac.uk/cru/data/hrg/cru_ts_4.05/cruts.2103051243.v4.05/pre/"
ncfname <- "dados/cru_ts4.05.1901.2020.pre.dat.nc.gz"
```

#### Passo 2: Leitura do arquivo netCDF

A partir das funções abaixo, é possivel abrir e explorar o arquivo netCDF.

``` r
ncfname <- "dados/cru_ts4.05.1901.2020.pre.dat.nc"
ncin <- nc_open(ncfname)
t <- ncvar_get(ncin, "time")
tunits <- ncatt_get(ncin, "time", "units")
nt <- dim(t)
```

A unidade de tempo dos dados de precipitação corresponde ao numero de dias após 01/01/1901. Vamos converter para o formato padrão de data.

``` r
t_format <- lubridate::ymd("1900-01-01") + lubridate::days(t)
```
Em seguida, vamos converter o arquivo nectdf para o formato raster. Também é importante verificar as suas dimensões espaciais.

``` r
pre1.brick <- brick(ncfname)
extent(pre1.brick)
```

    ## class      : Extent 
    ## xmin       : -180 
    ## xmax       : 180 
    ## ymin       : -90 
    ## ymax       : 90

#### Passo 3: Cortar raster para a região de interesse

Leia o arquivo shapefile correspondente ao contorno da bacia de interesse. Será necessário converter o sistema de coordenadas do shapefile para o mesmo sistema do raster com os dados de precipitação.

``` r
shapefile <- readOGR("dados/36290000.shp")
```

    ## OGR data source with driver: ESRI Shapefile 
    ## Source: "C:\Users\usuÃ¡rio\Google Drive (taismaria@alu.ufc.br)\Doutorado\Disciplinas\2021.1\Clima e RH\Trabalho 1\dados\36290000.shp", layer: "36290000"
    ## with 1 features
    ## It has 1 fields

``` r
shapefile <- spTransform(shapefile, crs(proj4string(pre1.brick)))
plot(shapefile)
```

![](trabalho1_roteiro_files/figure-markdown_github/unnamed-chunk-6-1.png)

Agora, será possível fazer o recorte do raster a partir do shapefile da região de interesse.

``` r
pre1.mask <- crop(pre1.brick[[c(1:nt)]], shapefile)
```

#### Passo 4: Extracao da serie de precipitacao para a regiao de interesse

O ultimo passo é a extração da serie temporal de precipitação para o recorte extraído no passo anterior. O valor da precipitação mensal sera a média da precipitação mensal dos pixels contidos no recorte.

``` r
pre1.df <- as.data.frame(pre1.mask)
pre1.df_mean <- pre1.df[complete.cases(pre1.df),] %>%
  summarise_all(mean)

prec <- data.frame(date = t_format, prec = as.numeric(pre1.df_mean))
```
