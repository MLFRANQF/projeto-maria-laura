
<!-- README.md is generated from README.Rmd. Please edit that file -->

# DERIVADA TEMPORAL DE XCO₂ PARA IDENTIFICAÇÃO FONTES E SUMIDOUROS DE CARBONO ATMOSFÉRICO NO ESTADO DE SÃO PAULO

Maria Laura Fraque Ferreira, Luis Miguel da Costa, Alan Rodrigo Panosso

**RESUMO** As mudanças climáticas e suas consequências impuseram
diversos desafios na atualidade, sendo a principal causa a emissão de
gases de efeito estufa (GEE) para a atmosfera, especialmente o dióxido
de carbono (CO₂). Embora as emissões globais estejam majoritariamente
relacionadas à queima de combustíveis fósseis, no Brasil, as mudanças no
uso e cobertura da terra (LULCC, na sigla em inglês) são a principal
causa das emissões de CO₂. Essas mudanças estão historicamente
associadas à expansão agrícola e ao desmatamento (Souza et al., 2020),
com as emissões brasileiras de GEE variando entre 0,4 e 1,7 Gt CO₂ eq em
2022. No Brasil, estudos mostraram correlação negativa entre SIF e a
concentração atmosférica de CO2. No entanto, poucas tentativas foram
feitas para esclarecer os principais sumidouros de CO2 e a SIF, bem como
aspectos relacionados como mudanças no uso e cobertura da terra (LULCC),
especialmente considerando os diferentes biomas brasileiros. A hipótese
da proposta se baseia na ideia de que fontes e sumidouros de CO2 podem
ser identificados por meio da análise de regressão linear simples, e
que, com a primeira derivada no tempo (β), é possível classificar pontos
específicos como fonte (β \> 0) ou sumidouro (β \< 0). Assim, os
objetivos deste trabalho serão identificar potenciais fontes e
sumidouros de CO2 e suas possíveis causas (por exemplo a SIF,
desmatamento e focos de incêndio), e estimar rapidamente os fluxos de
CO2 no estado de São Paulo a partir de observações via satélite.

## Carregar os pacotes

``` r
library(tidyverse)
library(geobr)

## Carregando pacote
library(readr)

purrr::map(list.files("functions/",full.names = T), source)
```

## Leitura de dados

Nessa parte estamos lendo os dados proveniente do trabalho previamente
realizado por Quadro 2026
(<https://github.com/arpanosso/tcc-fernando-quadros>)

``` r
dados <- read_rds("data/nasa-xco2-sp.rds") |> 
  filter(year >= 2019)
glimpse(dados)
```

Vamos ver os anos da série temporal

``` r
dados |> 
  distinct(year)
```

Gráfico dos pontos no ano de 2019

``` r
dados |> 
  filter(
    year == 2019
  ) |> 
  ggplot(aes(longitude, latitude)) +
  geom_point()
```

Usando o geobr

1 - Carregando os shapes

``` r
estados_geobr <- read_state(year = 2024,showProgress = FALSE)
```

2 - Filtrando para o estado de SP

``` r
sp_geobr <- estados_geobr |> 
  filter(abbrev_state == "SP")
```

3 - Apresentar o Mapa do estado de sao paulo

``` r
sp_geobr |> 
 ggplot() +
     geom_sf(aes_string(), color="black",
              size=.05, show.legend = TRUE) +
  theme_minimal() 
```

Unir o shape do geobr com os pontos

``` r
sp_geobr |> 
  ggplot() +
  geom_sf(aes_string(), color="black",
          size=.05, show.legend = TRUE) +
  geom_point(data = dados |> 
               filter(year == 2019), aes(longitude, latitude,
             color=xco2)) +
  theme_minimal() 
```

``` r
library(tidyverse)

xco2 <- read_rds("data/nasa-xco2-sp.rds") |>
  filter(year >= 2019)

glimpse(xco2)
```

# Análise de tendência

``` r
# XCO2
# glimpse(dados)

dados |> 
  sample_n(10000) |>
  ggplot(aes(x=date, y=xco2)) +
  geom_point() +
  geom_point(shape=21,color="black",fill="gray") +
  geom_smooth(method = "lm") +
  ggpubr::stat_regline_equation(ggplot2::aes(
  label =  paste(..eq.label.., ..rr.label.., sep = "*plain(\",\")~~"))) +
  theme_bw()
```

``` r
# Criar year_adj (1, 2, 3...)
dados_tend <- dados |>
  mutate(date_adj = date - min(date, na.rm = TRUE))

# Modelo XCO2
mod_trend_xco2 <- lm(
  xco2 ~ date_adj,
  data = dados_tend |> drop_na(xco2)
)

# 2) Previsão da tendência usando os modelos

dados_tend <- dados |>
  mutate(
    delta_co2 = predict(mod_trend_xco2, newdata = dados_tend)
  )
```

## removendo tendencias

``` r
# 3) Remoção da tendência

dados_tend <- dados_tend |>
  mutate(
    xco2_detrend = (xco2 - delta_co2) + mean(xco2, na.rm = TRUE)
  ) |>
  ungroup()

dados_tend <- dados_tend |> 
  mutate(
    xco2 = xco2_detrend
  ) |> 
  select(-(delta_co2))

dados_tend |> 
  ggplot(aes(x = date, y = xco2)) +
  geom_point() +
  geom_point(shape=21,color="black",fill="gray") +
  geom_smooth(method = "lm") +
  ggpubr::stat_regline_equation(ggplot2::aes(
  label =  paste(..eq.label.., ..rr.label.., sep = "*plain(\",\")~~"))) +
  theme_bw()
```

## análise descritiva

``` r
xco2 <- dados$xco2
head(xco2)

mean(xco2, na.rm = TRUE)
median(xco2, na.rm = TRUE)
sd(xco2, na.rm = TRUE)
summary(xco2)
```

## Histogramas e bloxplots

``` r

hist(xco2,
     main="Distribuicao do xco2",
     xlab="Concentracao de xco2",
     ylab="Frequencia",
     col="lightblue",
     border="black")

media_ano <- aggregate(xco2 ~ year, data=dados, mean)

media_ano

plot(media_ano$year,
     media_ano$xco2,
     type="b",
     main="Media de xco2 por ano",
     xlab="Ano",
     ylab="xco2 medio",
     col="blue")

boxplot(xco2,
        main="Boxplot do xco2",
        ylab="Concentracao de xco2",
        col="orange")

boxplot(xco2 ~ year,
        data = dados,
        las = 2,
        main = "xco2 por ano",
        xlab = "Ano",
        ylab = "xco2",
        col = "lightgreen")


boxplot(xco2 ~ month,
        data = dados[dados$year == 2019, ],
        main = "xco2 por mes em 2019",
        xlab = "Mes",
        ylab = "xco2",
        col = "lightblue",
        names=c("Jan","Fev","Mar","Abr","Mai","Jun",
                "Jul","Ago","Set","Out","Nov","Dez"))

boxplot(xco2 ~ month,
        data = dados[dados$year == 2020, ],
        main = "xco2 por mes em 2020",
        xlab = "Mes",
        ylab = "xco2",
        col = "lightblue",
        names=c("Jan","Fev","Mar","Abr","Mai","Jun",
                "Jul","Ago","Set","Out","Nov","Dez"))

boxplot(xco2 ~ month,
        data = dados[dados$year == 2021, ],
        main = "xco2 por mes em 2021",
        xlab = "Mes",
        ylab = "xco2",
        col = "lightblue",
        names=c("Jan","Fev","Mar","Abr","Mai","Jun",
                "Jul","Ago","Set","Out","Nov","Dez"))

boxplot(xco2 ~ month,
        data = dados[dados$year == 2022, ],
        main = "xco2 por mes em 2022",
        xlab = "Mes",
        ylab = "xco2",
        col = "lightblue",
        names=c("Jan","Fev","Mar","Abr","Mai","Jun",
                "Jul","Ago","Set","Out","Nov","Dez"))

boxplot(xco2 ~ month,
        data = dados[dados$year == 2023, ],
        main = "xco2 por mes em 2023",
        xlab = "Mes",
        ylab = "xco2",
        col = "lightblue",
        names=c("Jan","Fev","Mar","Abr","Mai","Jun",
                "Jul","Ago","Set","Out","Nov","Dez"))
```

``` r
dados_nested <- dados |> 
  # 1. Agrupando pelas suas colunas reais de tempo e coordenadas
  group_by(year, month, longitude, latitude) |> 
  summarise(
    media_xco2 = mean(xco2, na.rm = TRUE),
    media_incerteza = mean(xco2_incerteza, na.rm = TRUE), 
    .groups = "drop"
  ) |> 
  # 2. Agrupando por ponto geográfico para criar a lista aninhada (série temporal por pixel)
  group_by(longitude, latitude) |> 
  nest() |> 
  ungroup()

# Verificando a estrutura do objeto aninhado resultante
glimpse(dados_nested)
```

``` r
get_my_par <- function(df, par_return = "beta"){
  
  # Garante que o dataframe interno está ordenado cronologicamente
  df <- df %>% arrange(year, month)
  
  # Cria a variável de tempo real baseada em anos decimais (ex: 2019.08 para jan, 2019.16 para fev...)
  # Isso substitui o 'x <- seq(...)' genérico de forma robusta e precisa
  x <- df$year + (df$month - 1) / 12
  y <- df$media_xco2
  
  # Ajusta o modelo linear simples
  mod <- lm(y ~ x)
  beta <- mod$coefficients[[2]]
  
  # Retorna o coeficiente angular (tendência temporal do pixel)
  if(par_return == "beta") return(beta)
  
  # Verificação de segurança para o cálculo da anomalia
  if(par_return == "anomaly") {
    if("media_xco2_anomaly" %in% names(df)) {
      yanom <- df$media_xco2_anomaly
      anomaly <- mean(yanom, na.rm = TRUE)
      return(anomaly)
    } else {
      # Retorna NA se a coluna de anomalia ainda não tiver sido criada
      return(NA_real_)
    }
  }
}
```

``` r
get_my_par <- function(df, par_return = "beta"){
  # Ordena cronologicamente
  df <- df %>% arrange(year, month)
  
  # Variável temporal em anos decimais
  x <- df$year + (df$month - 1) / 12
  y <- df$media_xco2
  
  # Ajusta o modelo linear
  mod <- lm(y ~ x)
  
  # Extrai estritamente o valor numérico da inclinação (segundo elemento)
  beta <- as.numeric(mod$coefficients[2])
  
  if(par_return == "beta") return(beta)
  
  if(par_return == "anomaly") {
    if("media_xco2_anomaly" %in% names(df)) {
      return(mean(df$media_xco2_anomaly, na.rm = TRUE))
    } else {
      return(NA_real_)
    }
  }
}
```

``` r
# 1. Carregar as bibliotecas necessárias
library(sf)
library(dplyr)
library(tidyr)
library(geobr)

# 2. Carregar a base de dados do arquivo RDS e criar o objeto dados_beta_anom
dados_brutos <- readRDS("data/nasa-xco2-sp.rds")

# Criando as variáveis de beta e anomalia no objeto (ajuste a coluna de XCO2 se tiver outro nome)
dados_beta_anom <- dados_brutos %>% 
  mutate(
    beta_xco2 = xco2,                                # Se beta_xco2 for o próprio XCO2
    anomaly_xco2 = xco2 - mean(xco2, na.rm = TRUE)   # Cálculo da anomalia em relação à média
  )

# 3. Carregar a malha de municípios de SP (ano 2020)
municipios_sp <- geobr::read_municipality(code_muni = "SP", year = 2020, showProgress = FALSE)

# 4. Converter os pontos de latitude/longitude em um objeto espacial (sf)
dados_pontos_sf <- dados_beta_anom |> 
  drop_na(longitude, latitude) |> 
  st_as_sf(coords = c("longitude", "latitude"), crs = st_crs(municipios_sp))

# 5. Cruzamento espacial (identificar município de cada ponto)
pontos_com_municipio <- st_join(dados_pontos_sf, municipios_sp, join = st_within)

# 6. Agrupar os valores médios por município e unir à malha espacial
city_kgr_beta_anom <- municipios_sp |> 
  left_join(
    pontos_com_municipio |> 
      st_drop_geometry() |>   # Substitui as_tibble() para remover a geometria com segurança
      group_by(code_muni) |> 
      summarise(
        beta_xco2 = mean(beta_xco2, na.rm = TRUE),
        anom_xco2 = mean(anomaly_xco2, na.rm = TRUE),
        .groups = "drop"
      ),
    by = "code_muni"
  )

# 7. Checar a estrutura do objeto final gerado
glimpse(city_kgr_beta_anom)
```

``` r
# 1. Cria a variável de tempo e ordena os dados brutos do projeto
dados_processar <- dados %>% 
  mutate(tempo_decimal = year + (month - 1) / 12) %>% 
  arrange(longitude, latitude, tempo_decimal)

# 2. Calcula o Beta usando a fórmula analítica rápida da inclinação
# Isso garante que cada par de coordenadas receba apenas um único número
dados_beta_anom <- dados_processar %>% 
  group_by(longitude, latitude) %>% 
  summarise(
    beta_xco2 = cov(xco2, tempo_decimal, use = "complete.obs") / var(tempo_decimal, na.rm = TRUE),
    anomaly_xco2 = mean(xco2, na.rm = TRUE), # Usando a média como controle enquanto não há anomalia calculada
    .groups = "drop"
  ) %>% 
  drop_na(beta_xco2) # Remove qualquer ponto inválido

# 3. VERIFICAÇÃO CRUCIAL: Verifique se a tabela agora possui dados e números válidos
print("Número de linhas calculadas:")
nrow(dados_beta_anom)

print("Resumo estatístico dos valores de Beta encontrados:")
summary(dados_beta_anom$beta_xco2)
```

``` r
library(sf)
library(geobr)
library(tidyverse)
library(ggspatial)

# 1. Carregar as bordas oficiais do estado de SP
sp_borda <- geobr::read_state(code_state = "SP", year = 2020, showProgress = FALSE)

# 2. Criar a grade espacial (lon_grid, lat_grid) e o tempo decimal
dados_processar <- dados %>% 
  drop_na(longitude, latitude) %>% 
  mutate(
    tempo_decimal = year + (month - 1) / 12,
    lon_grid = round(longitude, 1),
    lat_grid = round(latitude, 1)
  )

# 3. Calcular a tendência temporal (Beta) por célula da grade no período filtrado
dados_beta_anom <- dados_processar %>% 
  group_by(lon_grid, lat_grid) %>% 
  filter(n() > 2) %>% 
  summarise(
    beta_xco2 = cov(xco2, tempo_decimal, use = "complete.obs") / var(tempo_decimal, na.rm = TRUE),
    .groups = "drop"
  ) %>% 
  drop_na(beta_xco2)

# 4. Configurar dinamicamente o centro e os limites com os percentis do período
q_min  <- quantile(dados_beta_anom$beta_xco2, 0.05, na.rm = TRUE)
q_med  <- quantile(dados_beta_anom$beta_xco2, 0.50, na.rm = TRUE)
q_max  <- quantile(dados_beta_anom$beta_xco2, 0.95, na.rm = TRUE)

# 5. Gerar o gráfico com a escala customizada (Azul -> Roxo -> Vermelho)
ggplot() +
  # Camada de fundo: Estado de SP
  geom_sf(data = sp_borda, fill = "#f5f5f5", color = "gray70", lwd = 0.4) +
  
  # Camada de dados: Grade contínua (Tiles)
  geom_tile(data = dados_beta_anom, 
            aes(x = lon_grid, y = lat_grid, fill = beta_xco2), 
            alpha = 0.85) +
  
  # Escala customizada com tons roxos na junção central
  scale_fill_gradientn(
    colors = c("#0044ff", "#6a0dad", "#9b30ff", "#d02090", "#ff1100"), 
    values = scales::rescale(c(q_min, (q_min + q_med)/2, q_med, (q_med + q_max)/2, q_max)),
    limits = c(q_min, q_max),
    oob = scales::squish, 
    name = expression(beta)
  ) +
  
  # Alinhamento espacial do ggplot
  coord_sf() +
  theme_bw() +
  theme(
    axis.text.x   = element_text(size = rel(0.9), color = "black"),
    axis.title.x  = element_text(size = rel(1.1), color = "black"),
    axis.text.y   = element_text(size = rel(0.9), color = "black"),
    axis.title.y  = element_text(size = rel(1.1), color = "black"),
    legend.text   = element_text(size = rel(1), color = "black"),
    legend.title  = element_text(face = 'bold', size = rel(1.2)),
    plot.title    = element_text(face = 'bold', size = rel(1.2))
  ) +
  labs(
    x = 'Longitude',
    y = 'Latitude',
    title = "",
    subtitle = ""
  ) +
  
  # Rosa dos Ventos
  annotation_north_arrow(
    location = "tl", 
    which_north = "true",
    pad_x = unit(0.2, "in"),
    pad_y = unit(0.2, "in"),
    style = north_arrow_fancy_orienteering(
      fill = c("white", "black"),
      line_col = "grey20"
    )
  ) +
  
  # Barra de Escala
  annotation_scale(
    location = "bl",
    width_hint = 0.2
  )
```

## Estatísticas descritivas do beta

``` r
stats_beta <- dados_beta_anom %>% 
  summarise(
    Media = mean(beta_xco2, na.rm = TRUE),
    Mediana = median(beta_xco2, na.rm = TRUE),
    Desvio_Padrao = sd(beta_xco2, na.rm = TRUE),
    Minimo = min(beta_xco2, na.rm = TRUE),
    Q1 = quantile(beta_xco2, 0.25, na.rm = TRUE),
    Q3 = quantile(beta_xco2, 0.75, na.rm = TRUE),
    Maximo = max(beta_xco2, na.rm = TRUE)
  )

print(stats_beta)
```

## Estratificação em Categorias (Classes de Beta)

``` r
library(sf)
library(geobr)
library(tidyverse)
library(ggspatial)

# 1. Carregar as bordas oficiais do estado de SP
sp_borda <- geobr::read_state(code_state = "SP", year = 2020, showProgress = FALSE)

# 2. Criar a grade e calcular o Beta
dados_processar <- dados %>% 
  drop_na(longitude, latitude) %>% 
  mutate(
    tempo_decimal = year + (month - 1) / 12,
    lon_grid = round(longitude, 1),
    lat_grid = round(latitude, 1)
  )

dados_beta_anom <- dados_processar %>% 
  group_by(lon_grid, lat_grid) %>% 
  filter(n() > 2) %>% 
  summarise(
    beta_xco2 = cov(xco2, tempo_decimal, use = "complete.obs") / var(tempo_decimal, na.rm = TRUE),
    .groups = "drop"
  ) %>% 
  drop_na(beta_xco2)

# 3. Estratificar em 4 categorias baseadas em quartis/percentis
dados_beta_anom <- dados_beta_anom %>% 
  mutate(
    beta_categoria = cut(
      beta_xco2,
      breaks = quantile(beta_xco2, probs = c(0, 0.25, 0.50, 0.75, 1), na.rm = TRUE),
      labels = c("Baixo (Sumidouro / Estável)", "Médio-Baixo", "Médio-Alto", "Alto (Fonte Crítica)"),
      include.lowest = TRUE
    )
  )
```

## Mapa Categorizado com Foco nas Regiões Críticas

``` r
# 4. Gerar o gráfico espacial categorizado
ggplot() +
  # Camada de fundo: Estado de SP
  geom_sf(data = sp_borda, fill = "#f8f9fa", color = "gray60", lwd = 0.5) +
  
  # Camada de dados categorizada por Tiles
  geom_tile(data = dados_beta_anom, 
            aes(x = lon_grid, y = lat_grid, fill = beta_categoria), 
            alpha = 0.9) +
  
  # Escala de cores discretas (Estratificada)
  scale_fill_manual(
    values = c(
      "Baixo (Sumidouro / Estável)" = "#2c7bb6",  # Azul
      "Médio-Baixo"                 = "#abd9e9",  # Azul claro
      "Médio-Alto"                  = "#fdae61",  # Laranja
      "Alto (Fonte Crítica)"        = "#d7191c"   # Vermelho
    ),
    name = expression(bold("Classes de " * beta))
  ) +
  
  coord_sf() +
  theme_bw() +
  theme(
    axis.text.x   = element_text(size = rel(0.9), color = "black"),
    axis.title.x  = element_text(size = rel(1.0), color = "black"),
    axis.text.y   = element_text(size = rel(0.9), color = "black"),
    axis.title.y  = element_text(size = rel(1.0), color = "black"),
    legend.position = "right",
    legend.text   = element_text(size = rel(0.9)),
    legend.title  = element_text(size = rel(1.0)),
    plot.title    = element_text(face = "bold", size = rel(1.2))
  ) +
  labs(
    x = 'Longitude',
    y = 'Latitude',
  ) +
  
  # Rosa dos Ventos
  annotation_north_arrow(
    location = "tl", 
    which_north = "true",
    pad_x = unit(0.2, "in"),
    pad_y = unit(0.2, "in"),
    style = north_arrow_fancy_orienteering(
      fill = c("white", "black"),
      line_col = "grey20"
    )
  ) +
  
  # Barra de Escala
  annotation_scale(
    location = "bl",
    width_hint = 0.2
  )
```

## Identificar as Regiões/Cidades

``` r
# Carregar malha de municípios
municipios_sp <- geobr::read_municipality(code_muni = "SP", year = 2020, showProgress = FALSE)

# Converter a grade para objeto espacial
pontos_sf <- dados_beta_anom %>% 
  st_as_sf(coords = c("lon_grid", "lat_grid"), crs = st_crs(municipios_sp))

# Cruzamento espacial para identificar o município
maiores_betas_muni <- st_join(pontos_sf, municipios_sp, join = st_within) %>% 
  st_drop_geometry() %>% 
  filter(beta_categoria == "Alto (Fonte Crítica)") %>% 
  group_by(name_muni) %>% 
  summarise(
    beta_medio = mean(beta_xco2, na.rm = TRUE),
    .groups = "drop"
  ) %>% 
  arrange(desc(beta_medio))

# Exibir os 10 municípios com maiores taxas de emissão (Beta)
head(maiores_betas_muni, 10)
```

``` r
library(sf)
library(geobr)
library(tidyverse)
library(ggspatial)

# 1. Carregar a malha espacial dos municípios de SP (2020)
municipios_sp <- geobr::read_municipality(code_muni = "SP", year = 2020, showProgress = FALSE)

# 2. Processar os pontos de dados brutos
dados_processar <- dados %>% 
  drop_na(longitude, latitude) %>% 
  mutate(tempo_decimal = year + (month - 1) / 12)

# 3. Converter os pontos de dados em um objeto espacial (sf)
dados_sf <- st_as_sf(
  dados_processar, 
  coords = c("longitude", "latitude"), 
  crs = st_crs(municipios_sp)
)

# 4. Cruzamento espacial: Identificar a qual município pertence cada ponto de medição
pontos_com_muni <- st_join(dados_sf, municipios_sp, join = st_within)

# 5. Agrupar por município, calcular o Beta e estratificar em 4 categorias
city_beta <- pontos_com_muni %>% 
  st_drop_geometry() %>% 
  group_by(code_muni) %>% 
  filter(n() > 2) %>% 
  summarise(
    beta_xco2 = cov(xco2, tempo_decimal, use = "complete.obs") / var(tempo_decimal, na.rm = TRUE),
    .groups = "drop"
  ) %>% 
  drop_na(beta_xco2)

# 6. Unir os dados calculados de volta à malha espacial dos municípios
mapa_municipios <- municipios_sp %>% 
  left_join(city_beta, by = "code_muni") %>% 
  mutate(
    beta_categoria = cut(
      beta_xco2,
      breaks = quantile(beta_xco2, probs = c(0, 0.25, 0.50, 0.75, 1), na.rm = TRUE),
      labels = c("Baixo (Sumidouro / Estável)", "Médio-Baixo", "Médio-Alto", "Alto (Fonte Crítica)"),
      include.lowest = TRUE
    )
  )

# 7. Gerar o gráfico com MUNICÍPIOS COLORIDOS (Alto Contraste)
ggplot(data = mapa_municipios) +
  # Desenha os polígonos dos municípios (cinza para municípios sem dados)
  geom_sf(aes(fill = beta_categoria), color = "white", size = 0.1) +
  
  # Paleta de cores discretas/estratificadas
  scale_fill_manual(
    values = c(
      "Baixo (Sumidouro / Estável)" = "#0571b0",  # Azul escuro
      "Médio-Baixo"                 = "#92c5de",  # Azul claro
      "Médio-Alto"                  = "#f4a582",  # Laranja claro
      "Alto (Fonte Crítica)"        = "#ca0020"   # Vermelho escuro
    ),
    na.value = "#e0e0e0", # Cor cinza claro para municípios sem dados
    name = expression(bold("Classes de " * beta))
  ) +
  
  coord_sf() +
  theme_bw() +
  theme(
    axis.text.x     = element_text(size = rel(0.9), color = "black"),
    axis.title.x    = element_text(size = rel(1.0), color = "black"),
    axis.text.y     = element_text(size = rel(0.9), color = "black"),
    axis.title.y    = element_text(size = rel(1.0), color = "black"),
    legend.position = "right",
    legend.text     = element_text(size = rel(0.9)),
    legend.title    = element_text(size = rel(1.0)),
    panel.grid      = element_line(color = "gray90", linetype = "dashed"),
    plot.title      = element_text(face = "bold", size = rel(1.2))
  ) +
  labs(
    x = 'Longitude',
    y = 'Latitude',
  ) +
  
  # Rosa dos Ventos
  annotation_north_arrow(
    location = "tl", 
    which_north = "true",
    pad_x = unit(0.2, "in"),
    pad_y = unit(0.2, "in"),
    style = north_arrow_fancy_orienteering(
      fill = c("white", "black"),
      line_col = "grey20"
    )
  ) +
  
  # Barra de Escala
  annotation_scale(
    location = "bl",
    width_hint = 0.2
  )
```

O coeficiente angular (β) da regressão linear representa a tendência
temporal da concentração atmosférica de XCO₂ em cada área analisada.
Valores positivos indicam aumento da concentração de CO₂ ao longo do
tempo, enquanto valores negativos sugerem redução, podendo estar
associados, respectivamente, a potenciais fontes e sumidouros de
carbono. Já a anomalia de XCO₂ representa o desvio entre os valores
observados e a tendência estimada, permitindo identificar variações
temporárias relacionadas a fatores ambientais. A análise espacial desses
parâmetros auxilia na identificação de padrões de emissão e remoção de
carbono, devendo ser interpretada em conjunto com outras variáveis
ambientais e considerando a influência do transporte atmosférico.
Observou-se que áreas com maior intensidade de ação antrópica
apresentaram, em geral, valores mais elevados de β, indicando uma
tendência de aumento da concentração atmosférica de XCO₂ ao longo do
período analisado. Esse padrão sugere que atividades humanas, como a
expansão urbana, a atividade industrial e as mudanças no uso da terra,
podem estar associadas ao incremento das concentrações de CO₂ nessas
regiões.

## agregação em grid regular

``` r
# Expand grid for brazil

#
dist <- 0.5
grid_br <- expand.grid(lon=seq(-74,
                               -27,dist),
                       lat=seq(-34,
                               6,
                               dist))



pol_sp<- sp_geobr$geometry |> purrr::pluck(1) |> as.matrix()



grid_br_cut <- grid_br |>
  dplyr::mutate(
    flag_sp = def_pol(lon,lat,pol_sp)
  ) |>
  tidyr::pivot_longer(
    tidyr::starts_with('flag'),
    names_to = 'region',
    values_to = 'flag'
  ) |>
  dplyr::filter(flag) |>
  dplyr::select(lon,lat) |>
  dplyr::group_by(lon,lat) |>
  dplyr::summarise(
    n_obs = dplyr::n()
  )

plot(grid_br_cut$lon,grid_br_cut$lat)


#### aggregation

df <- dados |> dplyr::mutate(
  year =lubridate::year(date),
  month = lubridate::month(date)
)




for(i in 2019:2024){
  aux_xco2 <- df |>
    dplyr::filter(year==i)
  vct_xco2 <- vector();dist_xco2 <- vector();
  lon_grid <- vector();lat_grid <- vector();
  for(k in 1:nrow(aux_xco2)){
    d <- sqrt((aux_xco2$longitude[k]-grid_br_cut$lon)^2+
                (aux_xco2$latitude[k]-grid_br_cut$lat)^2
    )
    min_index <- order(d)[1]
    vct_xco2[k] <- aux_xco2$xco2[min_index]
    dist_xco2[k] <- d[order(d)[1]]
    lon_grid[k] <- grid_br_cut$lon[min_index]
    lat_grid[k] <- grid_br_cut$lat[min_index]
  }
  aux_xco2$dist_xco2 <- dist_xco2
  aux_xco2$xco2_new <- vct_xco2
  aux_xco2$lon_grid <- lon_grid
  aux_xco2$lat_grid <- lat_grid
  if(i == 2019){
    df_new <- aux_xco2
  }else{
    df_new <- rbind(df_new,aux_xco2)
  }
}


df_new|>
  dplyr::mutate(
    dist_conf = sqrt((longitude - lon_grid)^2 + (latitude - lat_grid)^2)
  ) |>
  dplyr::glimpse()

nrow(df_new |>
       dplyr::mutate(
         dist_conf = sqrt((longitude - lon_grid)^2 + (latitude - lat_grid)^2),
         dist_bol = dist_xco2 - dist_conf
       ) |>
       dplyr::filter(dist_bol ==0)) == nrow(df_new)

### Saving aggregated data
# readr::write_rds(xco2_full_trend_cut,'data/xco2_0.5deg_full_trend.rds')
```

### Analise Beta

``` r
xco2df_filter <- df_new |>
  dplyr::filter(dist_xco2<0.25) |>
  dplyr::mutate(
    lon = lon_grid,
    lat = lat_grid,
  ) |>
  dplyr::select(-c(lon_grid,lat_grid)) |>
  dplyr::group_by(lon,lat,year,month) |> ## data month and grid cell aggregation
  dplyr::summarise(
    xco2_mean= mean(xco2,na.rm=TRUE),
    xco2_sd = sd(xco2,na.rm=TRUE),
    nobs = dplyr::n(),
    xco2_ep = xco2_sd/sqrt(nobs),
    cv = 100*xco2_sd/xco2_mean
  ) |>
  dplyr::mutate(
    date = lubridate::make_date(year,month,'15')
  )

### general model for the year

mod <- lm(xco2_mean~x,data =xco2df_filter |>
            dplyr::mutate(
              x = 1:dplyr::n()
            ))

print("----------------------------")
print("XCO2 before regionalization")

plot_1 <- xco2df_filter |>
  dplyr::group_by(date) |>
  dplyr::summarise(xco2_mean=mean(xco2_mean)) |>
  ggplot2::ggplot(ggplot2::aes(x=date,y=xco2_mean )) +
  ggplot2::geom_point(shape=21,color="black",fill="gray") +
  ggplot2::geom_line(color="red") +
  ggplot2::geom_smooth(method = "lm") +
  ggpubr::stat_regline_equation(ggplot2::aes(
    label =  paste(..eq.label.., ..rr.label.., sep = "*plain(\",\")~~"))) +
  ggplot2::theme_bw()+
  ggplot2::xlab('Date')+
  ggplot2::ylab(expression(
    'Xco'[2]~' (ppm)'
  ))
print(plot_1)

#### regionalization of XCO2 
print("----------------------------")
print("XCO2 after regionalization")

plot_2 <- xco2df_filter |>
  dplyr::mutate(
    x=1:dplyr::n(),
    xco2_est = mod$coefficients[1] + mod$coefficients[2]*x,
    delta=xco2_est - xco2_mean,
    xco2r = (mod$coefficients[1]-delta)-(mean(xco2_mean)-mod$coefficients[1])
  ) |>
  dplyr::group_by(date) |>
  dplyr::summarise(xco2_mean=mean(xco2r)) |>
  ggplot2::ggplot(ggplot2::aes(x=date,y=xco2_mean )) +
  ggplot2::geom_point(shape=21,color="black",fill="gray") +
  ggplot2::geom_line(color="red") +
  ggplot2::geom_smooth(method = "lm") +
  ggpubr::stat_regline_equation(ggplot2::aes(
    label =  paste(..eq.label.., ..rr.label.., sep = "*plain(\",\")~~"))) +
  ggplot2::theme_bw()+
  ggplot2::xlab('Date')+
  ggplot2::ylab(expression(
    'Xco'[2][R]~' (ppm)'
  ))
print(plot_2)


### XCO2 regionalized
xco2detrend <- xco2df_filter |>
  dplyr::mutate(
    x=1:dplyr::n(),
    xco2_est = mod$coefficients[1] + mod$coefficients[2]*x,
    delta=xco2_est - xco2_mean,
    xco2r = (mod$coefficients[1]-delta)-(mean(xco2_mean)-mod$coefficients[1])
  )
xco2_aux_detrend <- xco2detrend |>
  dplyr::ungroup() |>
  dplyr::group_by(date) |>
  dplyr::summarise(
    xco2 = mean(xco2r)
  )

## general linear model
mod_detrend <- lm(xco2 ~date,
                  data = xco2_aux_detrend )
beta_r <-mod_detrend$coefficients[2] # regional beta
ep <- summary(mod_detrend)$coefficients[2,2] # regional standard error

ilbr <- beta_r-ep
slbr <- beta_r+ep


### creating a nest object by grid cell
xco2_nest <- xco2detrend |>
  tibble::as_tibble() |>
  dplyr::mutate(year =lubridate::year(date),
                quarter = lubridate::quarter(date),
                quarter_year = lubridate::make_date(year, quarter, 1)) |>
  dplyr::group_by(lon, lat,date) |>
  dplyr::summarise(xco2 = mean(xco2r, na.rm=TRUE)) |>
  dplyr::mutate(
    id_time = date
  ) |>
  dplyr::group_by(lon,lat) |>
  tidyr::nest()

### linear regression for each grid cell

xco2_nest_detrend <- xco2_nest|>
  dplyr::mutate(
    beta_line = purrr::map(data,linear_reg, output="beta1"),
    #p_value = purrr::map(data,linear_reg, output="p_value"),
    n_obs = purrr::map(data,linear_reg, output="n"),
    beta_error=purrr::map(data,linear_reg,output='betaerror'),
    model_error=purrr::map(data,linear_reg,output='modelerror')
  )

### creating a table

xco2_aux_detrend_new <- xco2_nest_detrend |>
  dplyr::filter(n_obs > 4) |> ## criteria of minimum observation month for grid cell
  tidyr::unnest(cols = c(beta_line,beta_error,model_error)) |>
  dplyr::ungroup() |>
  dplyr::select(lon, lat, beta_line,beta_error,model_error)

q3_xco2 <- xco2_aux_detrend_new |> dplyr::pull(beta_line) |> quantile(.75)
q1_xco2 <- xco2_aux_detrend_new |> dplyr::pull(beta_line) |> quantile(.25)


plot3 <- xco2_aux_detrend_new |>
  ggplot2::ggplot(ggplot2::aes(x=beta_line)) +
  ggplot2::geom_histogram(bins=30,
                          fill="orange",
                          color="black") +
  ggplot2::labs(x="βpixel",y="Count") +
  ggplot2::geom_vline(xintercept = q1_xco2,
                      color = "red",
                      lty=2) +
  ggplot2::geom_vline(xintercept = q3_xco2,
                      color = "red",
                      lty=2)+
  gghighlight::gghighlight(beta_line < q1_xco2 | beta_line>q3_xco2,
                           unhighlighted_params = list(
                             color = "darkgray",
                             fill = "lightgray")) +
  ggplot2::theme_minimal()
print("----------------------------")
print("significant βpixel histogram")
print(plot3)

plot4 <- sp_geobr |>
  ggplot2::ggplot()+
  #ggspatial::annotation_map_tile(type = 'cartolight')+
  ggplot2::geom_sf(col='grey',fill='white')+
  #ggplot2::geom_sf(data=br,col='red',fill='NA')+
  # ggplot2::ylim(-35,5.5)+
  # ggplot2::xlim(-75,-35)+
  ggplot2::geom_tile(data=xco2_aux_detrend_new |>
                       dplyr::mutate(
                         xco2 = dplyr::case_when(
                           beta_line > q3_xco2 ~ 'Source',
                           beta_line < q1_xco2 ~'Sink',
                           .default = 'Non Significant'
                         )
                       ) |>
                       dplyr::filter(xco2!='Non Significant'),
                     ggplot2::aes(x=lon,y=lat,color=beta_line,fill=beta_line),
  )+
  map_theme()+
  ggplot2::scale_fill_viridis_c(option='inferno')+
  ggplot2::scale_color_viridis_c(option='inferno')+
  #ggplot2::scale_color_manual(values = c('darkgreen','darkred'))+
  #ggplot2::scale_fill_manual(values = c('darkgreen','darkred'))+
  ggplot2::labs(x='Longitude',y='Latitude',
                col=expression(beta~'Xco'[2]),fill=expression(beta~'Xco'[2]))
print("----------------------------")
print("significant source and sink")
print(plot4)
```
