---
title: "03 Game analysis"
output:   
  html_document:
    keep_md: true
    toc: true
    toc_float: true
---

## Libraries


## Data

```r
dat <- read_excel("Input_data/Statistikk ResponSEAble - apr - des 2018.xlsx", sheet = 3, skip = 27, col_names = FALSE)
dat <- as.data.frame(dat)
# str_sub(dat[1,], 6, 24)
# str_sub(dat[1,], 26)

# original - single quotes - deosn't work:
# fromJSON("{'score':'16','gender':'m','name':'TEST','time':'1531479201122','age':'2_11','lang':'no','platformid':'1'}")
# double quotes - works:
# fromJSON('{"score":"16","gender":"m","name":"TEST","time":"1531479201122","age":"2_11","lang":"no","platformid":"1"}')

txt2data <- function(txt) 
  gsub("\'","\"", txt) %>% str_sub(26) %>% fromJSON()

# test:
# txt2data(dat[1,])
# txt2data(dat[2,])

dat2 <- 1:nrow(dat) %>% purrr::map_df(~txt2data(dat[.,1]))
# dat2$Time <- str_sub(dat[,1], 6, 24)

dat2$score <- as.numeric(dat2$score)
dat2$time_unix <- as.numeric(dat2$time)
dat2$platformid <- as.numeric(dat2$platformid)

dat2$time <- as.POSIXct(dat2$time_unix/1000, origin = "1970-01-01")

dat2$age <- factor(dat2$age, levels = c("2_11", "6_15", "16_21", "22_40", "41_60", "61"))

age_group <- data.frame(
  age = c("2_11", "6_15", "16_21", "22_40", "41_60", "61"),
  age_group = c("2-8", "9-15", "16-21", "22-40", "41-60", "61+")
)
dat2 <- left_join(dat2, age_group)
```

```
## Joining, by = "age"
```

```
## Warning: Column `age` joining factors with different levels, coercing to
## character vector
```

```r
dat2$age_group = factor(dat2$age_group, c("2-8", "9-15", "16-21", "22-40", "41-60", "61+"))

platform_identity <- data.frame(
  platformid = 1:13,
  platform = c("NIVA office", "Color Fantasy (Oslo-Kiel)", "Trollfjord (coastal steamer)", 
               "Console 3", "Runde Env. Centre", "Console 4", "Portable", "Laptop demo",
               paste("Console", 5:9))
)
dat2 <- left_join(dat2, platform_identity)
```

```
## Joining, by = "platformid"
```

```r
dat2$day <- floor_date(dat2$time, "day")
```


```r
ggplot(dat2, aes(age_group)) + 
  geom_histogram(stat = "count", fill = colorbrewer_blue) +
  labs(x = "Age group", y = "Number of games") +
  theme_minimal() +
  theme(axis.title.x=element_text(vjust=-0.2),
        axis.title.y=element_text(vjust=3.3))
```

```
## Warning: Ignoring unknown parameters: binwidth, bins, pad
```

![](03_Game_files/figure-html/unnamed-chunk-2-1.png)<!-- -->

```r
ggsave("Figures/03_01.png", width = 6, height = 4, dpi = 400)
```

```r
ggplot(dat2, aes(age_group, fill = gender)) + 
  scale_fill_brewer(palette = "Set1") +
  geom_histogram(stat = "count") +
  facet_grid(gender~.) +
  labs(x = "Age group", y = "Number of games") +
  theme_minimal() +
  theme(axis.title.x=element_text(vjust=-0.2),
        axis.title.y=element_text(vjust=3.3))
```

```
## Warning: Ignoring unknown parameters: binwidth, bins, pad
```

![](03_Game_files/figure-html/unnamed-chunk-3-1.png)<!-- -->

```r
ggsave("Figures/03_02.png", width = 6, height = 4, dpi = 400)
```


```r
ggplot(dat2, aes(platform)) + 
  geom_histogram(stat = "count", fill = "blue3") +
  coord_flip()
```

```
## Warning: Ignoring unknown parameters: binwidth, bins, pad
```

![](03_Game_files/figure-html/unnamed-chunk-4-1.png)<!-- -->


```r
pick <- c("Color Fantasy (Oslo-Kiel)", "Trollfjord (coastal steamer)", "Runde Env. Centre")
ggplot(dat2 %>% filter(platform %in% pick), aes(age_group, fill = platform)) + 
  scale_fill_brewer(palette = "Set1") +
  geom_histogram(stat = "count") + 
  facet_wrap(~platform, ncol = 1, scales = "free_y") +
  labs(x = "Age group", y = "Number of games") +
  theme_minimal() +
  theme(axis.title.x=element_text(vjust=-0.2),
        axis.title.y=element_text(vjust=3.3),
        legend.position = "none")
```

```
## Warning: Ignoring unknown parameters: binwidth, bins, pad
```

![](03_Game_files/figure-html/unnamed-chunk-5-1.png)<!-- -->

```r
ggsave("Figures/03_03.png", width = 6, height = 4, dpi = 400)
```


```r
pick <- c("Color Fantasy (Oslo-Kiel)", "Trollfjord (coastal steamer)", "Runde Env. Centre")
ggplot(dat2 %>% filter(platform %in% pick), aes(lang, fill = platform)) + 
  scale_fill_brewer(palette = "Set1") +
  geom_histogram(stat = "count") + 
  facet_wrap(~platform, ncol = 1, scales = "free_y") +
  labs(x = "Language", y = "Number of games") +
  theme_minimal() +
  theme(axis.title.x=element_text(vjust=-0.2),
        axis.title.y=element_text(vjust=3.3),
        legend.position = "none")
```

```
## Warning: Ignoring unknown parameters: binwidth, bins, pad
```

![](03_Game_files/figure-html/unnamed-chunk-6-1.png)<!-- -->

```r
ggsave("Figures/03_04.png", width = 6, height = 4, dpi = 400)
```

```r
ggplot(dat2, aes(score)) + 
  geom_histogram(binwidth = 20, fill = colorbrewer_blue, color = "black") +
  labs(x = "Score (max = 113)", y = "Number of games") +
  theme_minimal() +
  theme(axis.title.x=element_text(vjust=-0.2),
        axis.title.y=element_text(vjust=3.3))
```

![](03_Game_files/figure-html/unnamed-chunk-7-1.png)<!-- -->

```r
ggsave("Figures/03_05.png", width = 5, height = 3, dpi = 400)
```



```r
ggplot(dat2, aes(score, fill = age_group)) + 
  scale_fill_brewer(palette = "Set1") +
  geom_histogram(binwidth = 20, color = "black") + 
  facet_wrap(~age_group) +
  labs(x = "Points by age group", y = "Number of games") +
  theme_minimal() +
  theme(axis.title.x=element_text(vjust=-0.2),
        axis.title.y=element_text(vjust=3.3),
        legend.position = "none")
```

![](03_Game_files/figure-html/unnamed-chunk-8-1.png)<!-- -->

```r
ggsave("Figures/03_06.png", width = 6, height = 4, dpi = 400)
```


```r
dat2 %>%
  group_by(age_group) %>%
  summarise(Perc_max = sum(score == 113)/n()*100) %>%
  ggplot(aes(age_group, Perc_max)) + 
  geom_col(fill = colorbrewer_blue, color = "black") +
  labs(x = "Age group", y = "Percent achieving max. score") +
  theme_minimal() +
  theme(axis.title.x=element_text(vjust=-0.2),
        axis.title.y=element_text(vjust=3.3))
```

![](03_Game_files/figure-html/unnamed-chunk-9-1.png)<!-- -->

```r
ggsave("Figures/03_07.png", width = 5, height = 3, dpi = 400)
```


```r
dat2 %>% 
  group_by(day) %>%
  summarise(No_of_plays = n()) %>%
  ggplot(aes(day, No_of_plays)) +
    geom_line()
```

![](03_Game_files/figure-html/unnamed-chunk-10-1.png)<!-- -->



