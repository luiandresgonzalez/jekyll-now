---
layout: post
title: "Cómo crear datos tidy con dplyr y tidyr"
date: "2019-01-14 17:53:51 -0200"
---

La info viene de la vignette de `tidyr`

Creando los datos
-----------------

Acá creo una matriz que está en la vignette, después la convierto a una tibble (el formato más apropiado para trabajar con datos en tidyverse).

``` r
# Se crea una matriz de 5 números entre 0 y 20, organizados en 2 columnas, y la secuencia ordenada por fila y no por columna (el default)
preg <- matrix(c(NA, sample(20, 5)), ncol = 2, byrow = T)

# Se agrega una columna a preg con los nombres
preg <- cbind(c("John Smith", "Jane Doe", "Mary Johnson"),preg)

# Se nombran las columnas: paste0 appendea a y b al final de treatment.
colnames(preg) <-c("name", paste0("treatment", c("a", "b")))
preg
```

    ##      name           treatmenta treatmentb
    ## [1,] "John Smith"   NA         "5"       
    ## [2,] "Jane Doe"     "14"       "13"      
    ## [3,] "Mary Johnson" "20"       "7"

``` r
# Conversión a tibble
preggo <- as_tibble(preg)
preggo
```

    ## # A tibble: 3 x 3
    ##   name         treatmenta treatmentb
    ##   <chr>        <chr>      <chr>     
    ## 1 John Smith   <NA>       5         
    ## 2 Jane Doe     14         13        
    ## 3 Mary Johnson 20         7

Transformando en datos tidy
---------------------------

Como resultado de lo anterior se crea una tabla que contiene más de una observación por fila. El objetivo es que se transforme en datos tidy donde se rige por el principio de "una fila, una observación".

1.  Each variable forms a column.

2.  Each observation forms a row.

3.  Each type of observational unit forms a table.

Por ejemplo:

``` r
preg2 <- preggo %>%
  gather(treatment, n, treatmenta:treatmentb) %>%
  mutate(treatment = gsub("treatment", "", treatment)) %>%
  arrange(name, treatment)
preg2
```

    ## # A tibble: 6 x 3
    ##   name         treatment n    
    ##   <chr>        <chr>     <chr>
    ## 1 Jane Doe     a         14   
    ## 2 Jane Doe     b         13   
    ## 3 John Smith   a         <NA>
    ## 4 John Smith   b         5    
    ## 5 Mary Johnson a         20   
    ## 6 Mary Johnson b         7

De esta forma de organizar los datos, queda definido claramente que es una variable (los títulos de las columnas) y qué es una observación (cada una de las filas).

**Los 5 problemas mas grandes de datos desprolijos son los siguientes**

    * Los encabezados de las columnas son valores y no nombres de variables.
    * Hay más de una variable alojada en una columna.
    * Las variables son almacenadas tanto en filas como columnas.
    * Se alojan varios tipos de unidades observacionales en la misma tabla.
    * Una única unidad observacional se aloja en múltiples tablas.

``` r
pewpew <- getURL("https://raw.githubusercontent.com/tidyverse/tidyr/master/vignettes/pew.csv")


pew <- as_tibble(read.csv(text = pewpew, stringsAsFactors = FALSE, check.names = FALSE))
pew
```

    ## # A tibble: 18 x 11
    ##    religion `<$10k` `$10-20k` `$20-30k` `$30-40k` `$40-50k` `$50-75k`
    ##    <chr>      <int>     <int>     <int>     <int>     <int>     <int>
    ##  1 Agnostic      27        34        60        81        76       137
    ##  2 Atheist       12        27        37        52        35        70
    ##  3 Buddhist      27        21        30        34        33        58
    ##  4 Catholic     418       617       732       670       638      1116
    ##  5 Don’t k…      15        14        15        11        10        35
    ##  6 Evangel…     575       869      1064       982       881      1486
    ##  7 Hindu          1         9         7         9        11        34
    ##  8 Histori…     228       244       236       238       197       223
    ##  9 Jehovah…      20        27        24        24        21        30
    ## 10 Jewish        19        19        25        25        30        95
    ## 11 Mainlin…     289       495       619       655       651      1107
    ## 12 Mormon        29        40        48        51        56       112
    ## 13 Muslim         6         7         9        10         9        23
    ## 14 Orthodox      13        17        23        32        32        47
    ## 15 Other C…       9         7        11        13        13        14
    ## 16 Other F…      20        33        40        46        49        63
    ## 17 Other W…       5         2         3         4         2         7
    ## 18 Unaffil…     217       299       374       365       341       528
    ## # … with 4 more variables: `$75-100k` <int>, `$100-150k` <int>,
    ## #   `>150k` <int>, `Don't know/refused` <int>

``` r
str(pew)
```

    ## Classes 'tbl_df', 'tbl' and 'data.frame':    18 obs. of  11 variables:
    ##  $ religion          : chr  "Agnostic" "Atheist" "Buddhist" "Catholic" ...
    ##  $ <$10k             : int  27 12 27 418 15 575 1 228 20 19 ...
    ##  $ $10-20k           : int  34 27 21 617 14 869 9 244 27 19 ...
    ##  $ $20-30k           : int  60 37 30 732 15 1064 7 236 24 25 ...
    ##  $ $30-40k           : int  81 52 34 670 11 982 9 238 24 25 ...
    ##  $ $40-50k           : int  76 35 33 638 10 881 11 197 21 30 ...
    ##  $ $50-75k           : int  137 70 58 1116 35 1486 34 223 30 95 ...
    ##  $ $75-100k          : int  122 73 62 949 21 949 47 131 15 69 ...
    ##  $ $100-150k         : int  109 59 39 792 17 723 48 81 11 87 ...
    ##  $ >150k             : int  84 74 53 633 18 414 54 78 6 151 ...
    ##  $ Don't know/refused: int  96 76 54 1489 116 1529 37 339 37 162 ...

Ahora tenemos una tabla con tres variables: `religion`, `income` y `frequency`.

``` r
pew2 <- pew %>%
  gather(income, frequency, -religion)
pew2
```

    ## # A tibble: 180 x 3
    ##    religion                income frequency
    ##    <chr>                   <chr>      <int>
    ##  1 Agnostic                <$10k         27
    ##  2 Atheist                 <$10k         12
    ##  3 Buddhist                <$10k         27
    ##  4 Catholic                <$10k        418
    ##  5 Don’t know/refused      <$10k         15
    ##  6 Evangelical Prot        <$10k        575
    ##  7 Hindu                   <$10k          1
    ##  8 Historically Black Prot <$10k        228
    ##  9 Jehovah's Witness       <$10k         20
    ## 10 Jewish                  <$10k         19
    ## # … with 170 more rows

``` r
wbillboard <- getURL("https://raw.githubusercontent.com/tidyverse/tidyr/master/vignettes/billboard.csv")

billboard <- as_tibble(read.csv(text = wbillboard, stringsAsFactors = FALSE))
billboard                      
```

    ## # A tibble: 317 x 81
    ##     year artist track time  date.entered   wk1   wk2   wk3   wk4   wk5
    ##    <int> <chr>  <chr> <chr> <chr>        <int> <int> <int> <int> <int>
    ##  1  2000 2 Pac  Baby… 4:22  2000-02-26      87    82    72    77    87
    ##  2  2000 2Ge+h… The … 3:15  2000-09-02      91    87    92    NA    NA
    ##  3  2000 3 Doo… Kryp… 3:53  2000-04-08      81    70    68    67    66
    ##  4  2000 3 Doo… Loser 4:24  2000-10-21      76    76    72    69    67
    ##  5  2000 504 B… Wobb… 3:35  2000-04-15      57    34    25    17    17
    ##  6  2000 98^0   Give… 3:24  2000-08-19      51    39    34    26    26
    ##  7  2000 A*Tee… Danc… 3:44  2000-07-08      97    97    96    95   100
    ##  8  2000 Aaliy… I Do… 4:15  2000-01-29      84    62    51    41    38
    ##  9  2000 Aaliy… Try … 4:03  2000-03-18      59    53    38    28    21
    ## 10  2000 Adams… Open… 5:30  2000-08-26      76    76    74    69    68
    ## # … with 307 more rows, and 71 more variables: wk6 <int>, wk7 <int>,
    ## #   wk8 <int>, wk9 <int>, wk10 <int>, wk11 <int>, wk12 <int>, wk13 <int>,
    ## #   wk14 <int>, wk15 <int>, wk16 <int>, wk17 <int>, wk18 <int>,
    ## #   wk19 <int>, wk20 <int>, wk21 <int>, wk22 <int>, wk23 <int>,
    ## #   wk24 <int>, wk25 <int>, wk26 <int>, wk27 <int>, wk28 <int>,
    ## #   wk29 <int>, wk30 <int>, wk31 <int>, wk32 <int>, wk33 <int>,
    ## #   wk34 <int>, wk35 <int>, wk36 <int>, wk37 <int>, wk38 <int>,
    ## #   wk39 <int>, wk40 <int>, wk41 <int>, wk42 <int>, wk43 <int>,
    ## #   wk44 <int>, wk45 <int>, wk46 <int>, wk47 <int>, wk48 <int>,
    ## #   wk49 <int>, wk50 <int>, wk51 <int>, wk52 <int>, wk53 <int>,
    ## #   wk54 <int>, wk55 <int>, wk56 <int>, wk57 <int>, wk58 <int>,
    ## #   wk59 <int>, wk60 <int>, wk61 <int>, wk62 <int>, wk63 <int>,
    ## #   wk64 <int>, wk65 <int>, wk66 <lgl>, wk67 <lgl>, wk68 <lgl>,
    ## #   wk69 <lgl>, wk70 <lgl>, wk71 <lgl>, wk72 <lgl>, wk73 <lgl>,
    ## #   wk74 <lgl>, wk75 <lgl>, wk76 <lgl>

``` r
billboard2 <- billboard %>%
  gather(week, rank, wk1:wk76, na.rm = TRUE)
billboard2
```

    ## # A tibble: 5,307 x 7
    ##     year artist         track                time  date.entered week   rank
    ##    <int> <chr>          <chr>                <chr> <chr>        <chr> <int>
    ##  1  2000 2 Pac          Baby Don't Cry (Kee… 4:22  2000-02-26   wk1      87
    ##  2  2000 2Ge+her        The Hardest Part Of… 3:15  2000-09-02   wk1      91
    ##  3  2000 3 Doors Down   Kryptonite           3:53  2000-04-08   wk1      81
    ##  4  2000 3 Doors Down   Loser                4:24  2000-10-21   wk1      76
    ##  5  2000 504 Boyz       Wobble Wobble        3:35  2000-04-15   wk1      57
    ##  6  2000 98^0           Give Me Just One Ni… 3:24  2000-08-19   wk1      51
    ##  7  2000 A*Teens        Dancing Queen        3:44  2000-07-08   wk1      97
    ##  8  2000 Aaliyah        I Don't Wanna        4:15  2000-01-29   wk1      84
    ##  9  2000 Aaliyah        Try Again            4:03  2000-03-18   wk1      59
    ## 10  2000 Adams, Yolanda Open My Heart        5:30  2000-08-26   wk1      76
    ## # … with 5,297 more rows

``` r
billboard3 <- billboard2 %>%
  mutate(
    week = extract_numeric(week),
    date = as.Date(date.entered) + 7 * (week - 1)) %>%
  select(-date.entered)
```

    ## extract_numeric() is deprecated: please use readr::parse_number() instead

``` r
billboard3
```

    ## # A tibble: 5,307 x 7
    ##     year artist         track                  time   week  rank date      
    ##    <int> <chr>          <chr>                  <chr> <dbl> <int> <date>    
    ##  1  2000 2 Pac          Baby Don't Cry (Keep.… 4:22      1    87 2000-02-26
    ##  2  2000 2Ge+her        The Hardest Part Of .… 3:15      1    91 2000-09-02
    ##  3  2000 3 Doors Down   Kryptonite             3:53      1    81 2000-04-08
    ##  4  2000 3 Doors Down   Loser                  4:24      1    76 2000-10-21
    ##  5  2000 504 Boyz       Wobble Wobble          3:35      1    57 2000-04-15
    ##  6  2000 98^0           Give Me Just One Nig.… 3:24      1    51 2000-08-19
    ##  7  2000 A*Teens        Dancing Queen          3:44      1    97 2000-07-08
    ##  8  2000 Aaliyah        I Don't Wanna          4:15      1    84 2000-01-29
    ##  9  2000 Aaliyah        Try Again              4:03      1    59 2000-03-18
    ## 10  2000 Adams, Yolanda Open My Heart          5:30      1    76 2000-08-26
    ## # … with 5,297 more rows

``` r
billboard3 %>% arrange(artist, track, week)
```

    ## # A tibble: 5,307 x 7
    ##     year artist  track                   time   week  rank date      
    ##    <int> <chr>   <chr>                   <chr> <dbl> <int> <date>    
    ##  1  2000 2 Pac   Baby Don't Cry (Keep... 4:22      1    87 2000-02-26
    ##  2  2000 2 Pac   Baby Don't Cry (Keep... 4:22      2    82 2000-03-04
    ##  3  2000 2 Pac   Baby Don't Cry (Keep... 4:22      3    72 2000-03-11
    ##  4  2000 2 Pac   Baby Don't Cry (Keep... 4:22      4    77 2000-03-18
    ##  5  2000 2 Pac   Baby Don't Cry (Keep... 4:22      5    87 2000-03-25
    ##  6  2000 2 Pac   Baby Don't Cry (Keep... 4:22      6    94 2000-04-01
    ##  7  2000 2 Pac   Baby Don't Cry (Keep... 4:22      7    99 2000-04-08
    ##  8  2000 2Ge+her The Hardest Part Of ... 3:15      1    91 2000-09-02
    ##  9  2000 2Ge+her The Hardest Part Of ... 3:15      2    87 2000-09-09
    ## 10  2000 2Ge+her The Hardest Part Of ... 3:15      3    92 2000-09-16
    ## # … with 5,297 more rows

``` r
billboard3
```

    ## # A tibble: 5,307 x 7
    ##     year artist         track                  time   week  rank date      
    ##    <int> <chr>          <chr>                  <chr> <dbl> <int> <date>    
    ##  1  2000 2 Pac          Baby Don't Cry (Keep.… 4:22      1    87 2000-02-26
    ##  2  2000 2Ge+her        The Hardest Part Of .… 3:15      1    91 2000-09-02
    ##  3  2000 3 Doors Down   Kryptonite             3:53      1    81 2000-04-08
    ##  4  2000 3 Doors Down   Loser                  4:24      1    76 2000-10-21
    ##  5  2000 504 Boyz       Wobble Wobble          3:35      1    57 2000-04-15
    ##  6  2000 98^0           Give Me Just One Nig.… 3:24      1    51 2000-08-19
    ##  7  2000 A*Teens        Dancing Queen          3:44      1    97 2000-07-08
    ##  8  2000 Aaliyah        I Don't Wanna          4:15      1    84 2000-01-29
    ##  9  2000 Aaliyah        Try Again              4:03      1    59 2000-03-18
    ## 10  2000 Adams, Yolanda Open My Heart          5:30      1    76 2000-08-26
    ## # … with 5,297 more rows

``` r
arrange(billboard3, rank, artist)
```

    ## # A tibble: 5,307 x 7
    ##     year artist            track               time   week  rank date      
    ##    <int> <chr>             <chr>               <chr> <dbl> <int> <date>    
    ##  1  2000 Aaliyah           Try Again           4:03     14     1 2000-06-17
    ##  2  2000 Aguilera, Christ… What A Girl Wants   3:18      8     1 2000-01-15
    ##  3  2000 Aguilera, Christ… What A Girl Wants   3:18      9     1 2000-01-22
    ##  4  2000 Aguilera, Christ… Come On Over Baby … 3:38     11     1 2000-10-14
    ##  5  2000 Aguilera, Christ… Come On Over Baby … 3:38     12     1 2000-10-21
    ##  6  2000 Aguilera, Christ… Come On Over Baby … 3:38     13     1 2000-10-28
    ##  7  2000 Aguilera, Christ… Come On Over Baby … 3:38     14     1 2000-11-04
    ##  8  2000 Carey, Mariah     Thank God I Found … 4:14     11     1 2000-02-19
    ##  9  2000 Creed             With Arms Wide Open 3:52     27     1 2000-11-11
    ## 10  2000 Destiny's Child   Independent Women … 3:38      9     1 2000-11-18
    ## # … with 5,297 more rows

``` r
tbwhatever <- getURL("https://raw.githubusercontent.com/tidyverse/tidyr/master/vignettes/tb.csv")
tb <- as_tibble(read.csv(text = tbwhatever, stringsAsFactors = F))
tb
```

    ## # A tibble: 5,769 x 22
    ##    iso2   year   m04  m514  m014 m1524 m2534 m3544 m4554 m5564   m65    mu
    ##    <chr> <int> <int> <int> <int> <int> <int> <int> <int> <int> <int> <int>
    ##  1 AD     1989    NA    NA    NA    NA    NA    NA    NA    NA    NA    NA
    ##  2 AD     1990    NA    NA    NA    NA    NA    NA    NA    NA    NA    NA
    ##  3 AD     1991    NA    NA    NA    NA    NA    NA    NA    NA    NA    NA
    ##  4 AD     1992    NA    NA    NA    NA    NA    NA    NA    NA    NA    NA
    ##  5 AD     1993    NA    NA    NA    NA    NA    NA    NA    NA    NA    NA
    ##  6 AD     1994    NA    NA    NA    NA    NA    NA    NA    NA    NA    NA
    ##  7 AD     1996    NA    NA     0     0     0     4     1     0     0    NA
    ##  8 AD     1997    NA    NA     0     0     1     2     2     1     6    NA
    ##  9 AD     1998    NA    NA     0     0     0     1     0     0     0    NA
    ## 10 AD     1999    NA    NA     0     0     0     1     1     0     0    NA
    ## # … with 5,759 more rows, and 10 more variables: f04 <int>, f514 <int>,
    ## #   f014 <int>, f1524 <int>, f2534 <int>, f3544 <int>, f4554 <int>,
    ## #   f5564 <int>, f65 <int>, fu <int>

``` r
tb2 <- tb %>%
  gather(demo, n, -iso2, -year, na.rm = TRUE)

tb2
```

    ## # A tibble: 35,750 x 4
    ##    iso2   year demo      n
    ##    <chr> <int> <chr> <int>
    ##  1 AD     2005 m04       0
    ##  2 AD     2006 m04       0
    ##  3 AD     2008 m04       0
    ##  4 AE     2006 m04       0
    ##  5 AE     2007 m04       0
    ##  6 AE     2008 m04       0
    ##  7 AG     2007 m04       0
    ##  8 AL     2005 m04       0
    ##  9 AL     2006 m04       1
    ## 10 AL     2007 m04       0
    ## # … with 35,740 more rows

Qué carajo hizo la función `gather` en este caso?

La función expresada más arriba puede ser rearreglada en la siguiente forma (el código no se va a ejecutar)

``` r
gather(tb2, demo, n, -iso2, -year, na.rm = TRUE)
```

Lo que hace la función gather es recolectar los valores dentro de una tabla de doble entrada, y reorganizar los títulos de las columnas que designemos para que formen parte de filas que van a repetirse. Entonces, para hacer esto necesitamos los siguientes elementos para ser insertados como parámetros en la función:

-   La **fuente** de los datos (`data =`).
-   El nombre que le vamos a dar a la columna donde van a colapsar todas las columnas en la nueva tabla (la **llave** de la dupla llave valor).
-   El nombre que le vamos a dar al contenido de adentro de la tabla de doble entrada (el **valor** de la dupla llave/valor).
-   Las columnas que vamos a omitir porque queremos que permanezcan tal como están.

![Representación gráfica de como `gather` funciona](gather_ejemplo.png)
