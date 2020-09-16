Exercicio 10
================

### Continuaremos com a utilização dos dados do ESEB2018. Carregue o banco da mesma forma que nos exercicios anteriores

``` r
library(tidyverse)
library(haven)

link <- "https://github.com/MartinsRodrigo/Analise-de-dados/blob/master/04622.sav?raw=true"

download.file(link, "04622.sav", mode = "wb")

banco <- read_spss("04622.sav") 

banco <- banco %>%
  mutate(D10 = as_factor(D10)) %>%
  filter(Q18 < 11,
         D9 < 9999998,
         Q1501 < 11,
         Q12P2_B < 3) %>%
  mutate(Q12P2_B = case_when(Q12P2_B == 1 ~ 0,  
                             Q12P2_B == 2 ~ 1)) 
# Quem votou em Bolsonaro = 1 
# Quem votou em Haddad = 0
```

### Crie a mesma variável de religião utilizada no exercício anterior

``` r
Outras <- levels(banco$D10)[-c(3,5,13)]

banco <- banco %>%
  mutate(religiao = case_when(D10 %in% Outras ~ "Outras",
                              D10 == "Católica" ~ "Católica",
                              D10 == "Evangélica" ~ "Evangélica",
                              D10 == "Não tem religião" ~ "Não tem religião"))
```

### Faça uma regressão linear utilizando as mesmas variáveis do exercício 9 - idade(D1A\_ID), educação (D3\_ESCOLA), renda (D9), nota atribuída ao PT (Q1501), auto-atribuição ideológica (Q18), sexo (D2\_SEXO) e religião (variável criada no passo anterior) - explicam o voto em Bolsonaro (Q12P2\_B).

``` r
regressao1 <- lm( Q12P2_B ~ D1A_ID + D3_ESCOLA + D9 + Q1501 + Q18 + D2_SEXO + religiao, 
                  data = banco )

summary(regressao1)
```

    ## 
    ## Call:
    ## lm(formula = Q12P2_B ~ D1A_ID + D3_ESCOLA + D9 + Q1501 + Q18 + 
    ##     D2_SEXO + religiao, data = banco)
    ## 
    ## Residuals:
    ##      Min       1Q   Median       3Q      Max 
    ## -1.05532 -0.19854  0.01565  0.16182  0.96682 
    ## 
    ## Coefficients:
    ##                            Estimate Std. Error t value Pr(>|t|)    
    ## (Intercept)               7.067e-01  6.469e-02  10.924  < 2e-16 ***
    ## D1A_ID                    1.140e-03  7.539e-04   1.512  0.13074    
    ## D3_ESCOLA                 5.547e-03  5.226e-03   1.061  0.28873    
    ## D9                       -9.837e-07  3.196e-06  -0.308  0.75832    
    ## Q1501                    -7.728e-02  2.799e-03 -27.610  < 2e-16 ***
    ## Q18                       2.651e-02  3.093e-03   8.570  < 2e-16 ***
    ## D2_SEXO                  -5.286e-02  2.089e-02  -2.530  0.01154 *  
    ## religiaoEvangélica        7.684e-02  2.363e-02   3.251  0.00118 ** 
    ## religiaoNão tem religião -2.746e-03  4.238e-02  -0.065  0.94835    
    ## religiaoOutras           -7.263e-02  3.678e-02  -1.975  0.04855 *  
    ## ---
    ## Signif. codes:  0 '***' 0.001 '**' 0.01 '*' 0.05 '.' 0.1 ' ' 1
    ## 
    ## Residual standard error: 0.3489 on 1138 degrees of freedom
    ## Multiple R-squared:  0.5028, Adjusted R-squared:  0.4989 
    ## F-statistic: 127.9 on 9 and 1138 DF,  p-value: < 2.2e-16

``` r
confint(regressao1)
```

    ##                                  2.5 %        97.5 %
    ## (Intercept)               5.797291e-01  8.335766e-01
    ## D1A_ID                   -3.390684e-04  2.619271e-03
    ## D3_ESCOLA                -4.706619e-03  1.579994e-02
    ## D9                       -7.254959e-06  5.287609e-06
    ## Q1501                    -8.277435e-02 -7.179045e-02
    ## Q18                       2.044034e-02  3.257857e-02
    ## D2_SEXO                  -9.385894e-02 -1.186744e-02
    ## religiaoEvangélica        3.046538e-02  1.232064e-01
    ## religiaoNão tem religião -8.589165e-02  8.039968e-02
    ## religiaoOutras           -1.447902e-01 -4.633807e-04

### Interprete o resultado dos coeficientes

### Resposta: A interpretação deste modelo é diferente daquela de um modelo OLS regular.Uma maneira de interpretar esse valor previsto é pensá-lo como uma probabilidade prevista de que a variável dependente (dummy) seja igual a um, ou, em outras palavras, a probabilidade prevista desse respondente votar em Bolsonaro. No resultado dessa regressão o intercepto é estatisticamente significante, o intervalo de confiança de 5,797291e-01 a 8,335766e-01 corrobora o resultado. A variável D1A\_ID, D3\_ESCOLA e D9 não fazem diferença na probabilidade do indivíduo votar em Bolsonaro. A variável da nota atribuída ao PT (Q1501) provoca uma diminuição na probabilidade do indivíduo votar em Bolsonaro, ou seja, mais alta a nota dada ao PT, mais baixa a probabilidade de votar em Bolsonaro, o intervalo de confiança de . A variável de auto-atribuição ideológica (Q18), também causa um impacto na probabilidade de votar em Bolsonaro, o intervalo de confiança de 2,044034e-02 a 3,257857e-02 confirma o resultado. A variável D2\_SEXO indica que quando o indivíduo é mulher a probabilidade de votar em Bolsonaro diminui -5,286e-02, o intervalo de confiança de -9,385894e-02 a -1,186744e-02 indica que esse resultado é significativo. Já no caso da religião a categoria de referência é a Católica, o resultado indica que as pessoas da categoria Evangélica tem uma probabilidade maior de votar em Bolsonaro (intervalo de confiança: 3,046538e-02 a 1,232064e-01). Para os que Não tem religião a diferença com os católicos não é significativa e no caso da categoria Outras a indicação é que as pessoas tem uma probabilidade menor de votar em Bolsonaro (intervalo de confiança: -1,447902e-01 a -4,633807e-04) quando comparados aos católicos. De acordo com o R^2 o modelo explica aproximadamente 50% da probabilidade do voto em Bolsonaro, mas ele erra em média 0,3489 no valor da variável dependente.

### Porém, O p-valor da regressão linear, quando a variável dependente é categórica, assim como a variável Q12P2\_B, não é confiável, devido a alta heterocedasticidade. Se o modelo está estimado corretamente sem viés, os coeficientes tem valor parecido, mas o p-valor só é confiável na regressão logística.Além de que esse modelo linear pode produzir probabilidades com valores menores que zero ou maiores que um.

### O resultado difere dos encontrados anteriormente, quando a variavel dependente era a aprovação de Bolsonaro?

### Resposta: Sim. Na regressão do exercício 9 o resultado é diferente. As diferenças maiores entre as duas regressões são: (1) a variável D3\_ESCOLA (p-valor: 0,0117) é estatisticamente significante e tem um valor estimado de -1,134e-01 enquanto que na regressão atual essa variável tem um valor estimado de 5,547e-03 e p-valor: 0,28873; (2) a variável D9 que tinha um valor estimado de -3,632e-05 passa a ter um de -9.837e-07, mas continua sem significância estatística; (3) existe uma mudança no valor estimado da variável Q1501 de -3,956e-01 para -7,728e-02, o p-valor se mantém igual; (4) na variável Q18 o valor estimado era de 3,150e-01 e passa a ser de 2,651e-02, mas ainda com significância estatística (p-valor \< 2e-16); e em relação a variável D2\_SEXO e religião o resultado é interpretado de uma maneira diferente da regressão atual pelo fato de que essas variáveis interagiam no exercício 9.

### No entanto, devemos levar em consideração que a variável Q12P2\_B é categórica com duas categorias (0 ou 1) e portanto é uma dummy, diferente da variável Q1607 que era considerada contínua nos exercícios anteriores. Quando uma variável dependente é dummy a regressão logística é mais confiável que a regressão linear para interpretar os resultados

### Faça uma regressão logistica com as mesmas variaveis

``` r
regressao_logistica <- glm(Q12P2_B ~ D1A_ID + D3_ESCOLA + D9 + Q1501 + Q18 + D2_SEXO + religiao, data = banco, family = "binomial")

summary(regressao_logistica)
```

    ## 
    ## Call:
    ## glm(formula = Q12P2_B ~ D1A_ID + D3_ESCOLA + D9 + Q1501 + Q18 + 
    ##     D2_SEXO + religiao, family = "binomial", data = banco)
    ## 
    ## Deviance Residuals: 
    ##     Min       1Q   Median       3Q      Max  
    ## -2.7529  -0.5625   0.2518   0.4744   2.5830  
    ## 
    ## Coefficients:
    ##                            Estimate Std. Error z value Pr(>|z|)    
    ## (Intercept)               8.209e-01  5.298e-01   1.550  0.12124    
    ## D1A_ID                    1.001e-02  6.337e-03   1.580  0.11405    
    ## D3_ESCOLA                 5.634e-02  4.358e-02   1.293  0.19602    
    ## D9                       -4.635e-06  2.396e-05  -0.193  0.84660    
    ## Q1501                    -4.678e-01  2.666e-02 -17.545  < 2e-16 ***
    ## Q18                       2.242e-01  2.748e-02   8.159 3.37e-16 ***
    ## D2_SEXO                  -4.497e-01  1.739e-01  -2.586  0.00971 ** 
    ## religiaoEvangélica        6.217e-01  1.985e-01   3.132  0.00173 ** 
    ## religiaoNão tem religião -2.106e-02  3.478e-01  -0.061  0.95172    
    ## religiaoOutras           -6.736e-01  3.122e-01  -2.158  0.03096 *  
    ## ---
    ## Signif. codes:  0 '***' 0.001 '**' 0.01 '*' 0.05 '.' 0.1 ' ' 1
    ## 
    ## (Dispersion parameter for binomial family taken to be 1)
    ## 
    ##     Null deviance: 1557.84  on 1147  degrees of freedom
    ## Residual deviance:  862.45  on 1138  degrees of freedom
    ## AIC: 882.45
    ## 
    ## Number of Fisher Scoring iterations: 5

### Transforme os coeficientes estimados em probabilidade

``` r
library(margins)

margins(regressao_logistica)
```

    ##    D1A_ID D3_ESCOLA         D9    Q1501     Q18 D2_SEXO religiaoEvangélica
    ##  0.001171  0.006589 -5.421e-07 -0.05471 0.02622 -0.0526            0.07346
    ##  religiaoNão tem religião religiaoOutras
    ##                 -0.002521       -0.08172

``` r
summary(margins(regressao_logistica))
```

    ##                    factor     AME     SE        z      p   lower   upper
    ##                    D1A_ID  0.0012 0.0007   1.5849 0.1130 -0.0003  0.0026
    ##                   D2_SEXO -0.0526 0.0202  -2.6078 0.0091 -0.0921 -0.0131
    ##                 D3_ESCOLA  0.0066 0.0051   1.2949 0.1953 -0.0034  0.0166
    ##                        D9 -0.0000 0.0000  -0.1935 0.8466 -0.0000  0.0000
    ##                     Q1501 -0.0547 0.0009 -57.9079 0.0000 -0.0566 -0.0529
    ##                       Q18  0.0262 0.0030   8.8434 0.0000  0.0204  0.0320
    ##        religiaoEvangélica  0.0735 0.0235   3.1280 0.0018  0.0274  0.1195
    ##  religiaoNão tem religião -0.0025 0.0417  -0.0605 0.9517 -0.0842  0.0791
    ##            religiaoOutras -0.0817 0.0379  -2.1574 0.0310 -0.1560 -0.0075

### Quais foram as diferenças no resultado entre usar a regressão linear e a logistica?

### Resposta: As variáveis D1A\_ID, D3\_ESCOLA e D9 continuam sem significância estatística, ou seja, não afetam a probabilidade de votar em Bolsonaro. A variável Q1501 continua sendo estatisticamente significante de acordo com o p-valor, mas o valor estimado passa de -0,07728 para -0,0547 e o erro padrão diminuiu de 0,002799 para 0,0009. No caso da variável Q18, o p-valor continua sendo significante, o valor estimado é muito próximo do valor anterior assim como o erro padrão. A variável D2\_SEXO passa a ter um p-valor mais baixo de 0,0091, por isso podemos corroborar com mais confiança a hipótese de que essa variável causa impacto na probabilidade de votar em Bolsonaro, mas seu valor estimado permanece quase o mesmo. No caso da variável de religião a categoria Católico continua sendo a referência e os que Não tem religião não causam um impacto estatisticamente diferente dos católicos na probabilidade de votar em Bolsonaro. No caso da comparação entre católicos e evangélicos o p-valor é bem parecido e significante, o erro padrão e o valor estimado também permanecem quase os mesmos. Por fim na categoria Outras o p-valor passa de 0,04855 para 0,0310 e o erro padrão e valor estimado são próximos.

### Em suma, a grande diferença entre uma regressão linear e uma regressão logística, quando a variável dependente é categórica, é que a regressão logística resolve os problemas de heterocedasticidade e o problema da forma funcional. Na regressão logística podemos ter mais confiança nos p-valores e nos intervalos de confiança das variáveis.

### Verifique a quantidade de classificações corretas da regressao logistica e avalie o resultado

``` r
library(InformationValue)

predicted_prob <- predict(regressao_logistica, type = "response")

1 - misClassError(banco$Q12P2_B, 
                  predicted_prob, 
                  threshold = 0.5)
```

    ## [1] 0.8301

``` r
table(banco$Q12P2_B)
```

    ## 
    ##   0   1 
    ## 476 672

``` r
prop.table(table(banco$Q12P2_B))
```

    ## 
    ##         0         1 
    ## 0.4146341 0.5853659

``` r
0.8301/0.5853659 
```

    ## [1] 1.418087

``` r
opt_cutoff <- optimalCutoff(banco$Q12P2_B, 
                            predicted_prob)

confusionMatrix(banco$Q12P2_B, 
              predicted_prob, 
              threshold = opt_cutoff)
```

    ##     0   1
    ## 0 393 105
    ## 1  83 567

``` r
prop.table(confusionMatrix(banco$Q12P2_B, 
                predicted_prob, 
                threshold = opt_cutoff))
```

    ##            0          1
    ## 0 0.34233449 0.09146341
    ## 1 0.07229965 0.49390244

### Resposta: Pela comparação do modelo da regressão logística com o modélo ingênuo a regressão logística melhora melhora o modelo ingênuo em aproximadamente 42%. Na *confusion matrix* o modelo previu corretamente 34% dos votos em Haddad 49% dos votos em Bolsonaro. A previsão incorreta dos votos em Haddad foi de 9% e em Bolsonaro de 7%.
