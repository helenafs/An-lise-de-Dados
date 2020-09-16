Exercicio 11
================

``` r
library(tidyverse)
library(haven)

link <- "https://github.com/MartinsRodrigo/Analise-de-dados/blob/master/04622.sav?raw=true"

download.file(link, "04622.sav", mode = "wb")

banco <- read_spss("04622.sav") 

banco <- banco %>%
  mutate(D10 = as_factor(D10)) %>%
  filter(Q1607 < 11, 
         Q18 < 11,
         D9 < 9999998,
         Q1501 < 11)


Outras <- levels(banco$D10)[-c(3,5,13)]

banco <- banco %>%
  mutate(religiao = case_when(D10 %in% Outras ~ "Outras",
                              D10 == "Católica" ~ "Católica",
                              D10 == "Evangélica" ~ "Evangélica",
                              D10 == "Não tem religião" ~ "Não tem religião"))
```

### Faça uma regressão linear avaliando em que medida as variáveis independentes utilizadas nos exercícios 7 e 8, idade(D1A\_ID), educação (D3\_ESCOLA), renda (D9), nota atribuída ao PT (Q1501), auto-atribuição ideológica (Q18), sexo (D2\_SEXO) e religião (variável criada no passo anterior) explicam a avaliação de Bolsonaro (Q1607)

``` r
regressao2 <- lm (Q1607 ~ D1A_ID + D3_ESCOLA + D9 + Q1501 + Q18 + D2_SEXO + religiao, 
                  data = banco)
summary(regressao2)
```

    ## 
    ## Call:
    ## lm(formula = Q1607 ~ D1A_ID + D3_ESCOLA + D9 + Q1501 + Q18 + 
    ##     D2_SEXO + religiao, data = banco)
    ## 
    ## Residuals:
    ##     Min      1Q  Median      3Q     Max 
    ## -9.0608 -2.5654  0.4179  2.3268  8.9954 
    ## 
    ## Coefficients:
    ##                            Estimate Std. Error t value Pr(>|t|)    
    ## (Intercept)               6.216e+00  5.365e-01  11.586  < 2e-16 ***
    ## D1A_ID                    1.040e-02  6.234e-03   1.669 0.095376 .  
    ## D3_ESCOLA                -1.116e-01  4.486e-02  -2.487 0.012982 *  
    ## D9                       -3.620e-05  2.764e-05  -1.309 0.190576    
    ## Q1501                    -3.946e-01  2.367e-02 -16.670  < 2e-16 ***
    ## Q18                       3.161e-01  2.603e-02  12.142  < 2e-16 ***
    ## D2_SEXO                  -6.874e-01  1.746e-01  -3.937 8.63e-05 ***
    ## religiaoEvangélica        6.685e-01  1.984e-01   3.370 0.000772 ***
    ## religiaoNão tem religião -7.565e-02  3.485e-01  -0.217 0.828177    
    ## religiaoOutras           -8.326e-01  3.081e-01  -2.702 0.006963 ** 
    ## ---
    ## Signif. codes:  0 '***' 0.001 '**' 0.01 '*' 0.05 '.' 0.1 ' ' 1
    ## 
    ## Residual standard error: 3.296 on 1452 degrees of freedom
    ## Multiple R-squared:  0.3018, Adjusted R-squared:  0.2975 
    ## F-statistic: 69.75 on 9 and 1452 DF,  p-value: < 2.2e-16

### Faça o teste de homoscedasticidade do modelo e corrija as estimações dos coeficientes caso seja necessário.

``` r
plot(regressao2, 3)
```

![](exercicio_11_files/figure-gfm/unnamed-chunk-3-1.png)<!-- -->

``` r
plot(regressao2, 1)
```

![](exercicio_11_files/figure-gfm/unnamed-chunk-3-2.png)<!-- -->

``` r
library(lmtest)
bptest(regressao2)
```

    ## 
    ##  studentized Breusch-Pagan test
    ## 
    ## data:  regressao2
    ## BP = 65.763, df = 9, p-value = 1.025e-10

``` r
library(car)
ncvTest(regressao2) 
```

    ## Non-constant Variance Score Test 
    ## Variance formula: ~ fitted.values 
    ## Chisquare = 22.48512, Df = 1, p = 2.1178e-06

``` r
library(sandwich)

coeftest(regressao2, 
         vcov = vcovHC(regressao2))
```

    ## 
    ## t test of coefficients:
    ## 
    ##                             Estimate  Std. Error  t value  Pr(>|t|)    
    ## (Intercept)               6.2160e+00  5.4715e-01  11.3607 < 2.2e-16 ***
    ## D1A_ID                    1.0403e-02  6.2657e-03   1.6603 0.0970600 .  
    ## D3_ESCOLA                -1.1159e-01  4.7247e-02  -2.3619 0.0183123 *  
    ## D9                       -3.6198e-05  3.6481e-05  -0.9922 0.3212463    
    ## Q1501                    -3.9464e-01  2.6381e-02 -14.9593 < 2.2e-16 ***
    ## Q18                       3.1608e-01  2.8534e-02  11.0772 < 2.2e-16 ***
    ## D2_SEXO                  -6.8736e-01  1.7967e-01  -3.8256 0.0001360 ***
    ## religiaoEvangélica        6.6854e-01  1.9676e-01   3.3978 0.0006978 ***
    ## religiaoNão tem religião -7.5647e-02  3.7488e-01  -0.2018 0.8401094    
    ## religiaoOutras           -8.3256e-01  3.0592e-01  -2.7215 0.0065759 ** 
    ## ---
    ## Signif. codes:  0 '***' 0.001 '**' 0.01 '*' 0.05 '.' 0.1 ' ' 1

### Avalie a multicolinearidade entre as variáveis

``` r
library (car)
vif(regressao2)
```

    ##               GVIF Df GVIF^(1/(2*Df))
    ## D1A_ID    1.219401  1        1.104265
    ## D3_ESCOLA 1.337368  1        1.156446
    ## D9        1.094849  1        1.046350
    ## Q1501     1.119818  1        1.058215
    ## Q18       1.049195  1        1.024302
    ## D2_SEXO   1.023001  1        1.011435
    ## religiao  1.093846  3        1.015062

### Verifique a presença de outilier ou observações influentes no modelo

``` r
plot (regressao2, 4)
```

![](exercicio_11_files/figure-gfm/unnamed-chunk-5-1.png)<!-- -->

``` r
plot (regressao2, 5)
```

![](exercicio_11_files/figure-gfm/unnamed-chunk-5-2.png)<!-- -->

``` r
outlierTest(regressao2)
```

    ## No Studentized residuals with Bonferroni p < 0.05
    ## Largest |rstudent|:
    ##     rstudent unadjusted p-value Bonferroni p
    ## 271 -2.76344          0.0057918           NA

### Faça a regressao linear sem a observação mais influente e avalie a alteração do resultado

``` r
banco_outlier <- banco %>%
  slice (-c (140, 171, 411))

regressao3 <- lm (Q1607 ~ D1A_ID + D3_ESCOLA + D9 + Q1501 + Q18 + D2_SEXO + religiao, 
                  data = banco_outlier)
summary(regressao3)
```

    ## 
    ## Call:
    ## lm(formula = Q1607 ~ D1A_ID + D3_ESCOLA + D9 + Q1501 + Q18 + 
    ##     D2_SEXO + religiao, data = banco_outlier)
    ## 
    ## Residuals:
    ##     Min      1Q  Median      3Q     Max 
    ## -9.0592 -2.5769  0.4076  2.3267  8.9938 
    ## 
    ## Coefficients:
    ##                            Estimate Std. Error t value Pr(>|t|)    
    ## (Intercept)               6.218e+00  5.374e-01  11.571  < 2e-16 ***
    ## D1A_ID                    1.027e-02  6.245e-03   1.645 0.100158    
    ## D3_ESCOLA                -1.118e-01  4.490e-02  -2.490 0.012903 *  
    ## D9                       -3.643e-05  2.767e-05  -1.317 0.188161    
    ## Q1501                    -3.948e-01  2.371e-02 -16.653  < 2e-16 ***
    ## Q18                       3.161e-01  2.609e-02  12.114  < 2e-16 ***
    ## D2_SEXO                  -6.834e-01  1.749e-01  -3.907 9.78e-05 ***
    ## religiaoEvangélica        6.650e-01  1.986e-01   3.348 0.000835 ***
    ## religiaoNão tem religião -7.635e-02  3.487e-01  -0.219 0.826736    
    ## religiaoOutras           -8.069e-01  3.100e-01  -2.603 0.009340 ** 
    ## ---
    ## Signif. codes:  0 '***' 0.001 '**' 0.01 '*' 0.05 '.' 0.1 ' ' 1
    ## 
    ## Residual standard error: 3.299 on 1449 degrees of freedom
    ## Multiple R-squared:  0.3006, Adjusted R-squared:  0.2962 
    ## F-statistic: 69.19 on 9 and 1449 DF,  p-value: < 2.2e-16

### Resposta: sem as três observações mais influentes demonstradas pelo gráfico *Cook’s Distance* (140, 171 e 411) as variáveis que tem uma influência estatística significativa são:D3\_ESCOLA (p-valor: 0,012903), Q1501 e Q18 (p-valores: \< 2e-16), D2\_SEXO (p-valor: 9.78e-05), a religião Evangêlica é causa um impacto diferente da católica (p-valor: 0,000835) e a categoria de religião outras também (p-valor: 0,009340).

### Em comparação com o resultado anterior com o problema de heterocedasticidade corrigido (coeftest) o intercepto passou a ter um p-valor ainda mais baixo; a variável D3\_ESCOLA ficou com um p-valor mais alto, porém, ainda significativo; a variável D9 deixou de causar um impacto significante na nota de Bolsonaro; as variáveis Q1501 e Q18 passaram a causar um impacto significativo na nota de Bolsonaro; a variável D2\_SEXO passou a ter um p-valor mais baixo e o impacto na nota passou a ser negativo, indicando que aqui a categoria de referência é homem. A religião evangêlica ainda causa um impacto diferente da religião católica, porém agora com um p-valor mais baixo e a categoria outras passou a ser estatisticamente diferente da categoria católica. Em relação ao R^2 ele aumentou em comparação com a primeira regressão do exercício, agora o modelo explica 30% da variação na nota de Bolsonaro, o *Residual standard error* também diminuiu, passando de 16,78 para 3,299.
