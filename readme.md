ANOVA & R 2024 by Ryosuke TAJIMA
==============

今回はlibraryを読み込まずいわゆる「baseR」のみで完結させる

# Fisher
## Fisher 1966
- If the design of an experiment is faulty, any method of interpretation which makes it out to be decisive must be faulty, too.

## Fisherの3原則
- 反復＝Replication：実験ごとのばらつきを考慮するため同条件で複数回実験する
- 局所管理＝Local control：反復(ブロック)内では解析する要因以外の制御可能な要因は一定にする
- 無作為化(ランダム化)＝Randomization：制御できない要因はランダムになるようにする(系統誤差をなくす)


# one-way ANOVA

```R
d<-read.table("001.txt", header=T)
head(d) #列名確認

# 完全無作為化法
result<-aov(SDW~Treatment,d)
summary(result)

# 乱塊法
result<-aov(SDW~Rep+Treatment,d)

# 分散の分割
result<-aov(SDW~Rep+Treatment+Rep*Treatment,d)

# 省略記法(参考)
result<-aov(SDW~Rep*Treatment,d)
```


# two-way ANOVA
```R
d<-read.table("002.txt", header=T)
head(d) #列名確認

# 完全無作為化法
result<-aov(SDW~Limitation+Inoculation+Limitation*Inoculation,d)
summary(result)

# 省略記法
result<-aov(SDW~Limitation*Inoculation,d)
summary(result)

# 乱塊法
result<-aov(SDW~Rep+Limitation+Inoculation+Limitation*Inoculation,d)
summary(result)

# 乱塊法の省略記法
result<-aov(SDW~Rep+Limitation*Inoculation,d)
summary(result)

# 分散の分割
result<-aov(SDW~Rep+Limitation+Inoculation+Rep*Limitation+Rep*Inoculation+Limitation*Inoculation+Rep*Limitation*Inoculation,d)
summary(result)

# 分散の分割
result<-aov(SDW~Rep*Limitation*Inoculation,d)
summary(result)
```


# three-way ANOVA
```R
d<-read.table("003.txt", header=T)
head(d) #列名確認

# 完全無作為化法
result<-aov(RLD~Var*Limitation*Inoculation,d)
summary(result)

# 乱塊法
result<-aov(RLD~Rep+Var*Limitation*Inoculation,d)
summary(result)

# 分散の分割
result<-aov(RLD~Rep*Var*Limitation*Inoculation,d)
summary(result)
```


# 多重比較


## 多重性の問題とボンフェローニ補正

$\alpha^{\prime} = 1ー(1-\alpha)^n$  
この式に基づいて愚直に補正するのがボンフェローニ補正  


## TukeyHSD
```R
d<-read.table("002.txt", header=T)
head(d) #列名確認
NewNames<-paste(d$Limitation, d$Inoculation, sep="_") #列名追加
d<-data.frame(d,Lavel=NewNames)
result<-aov(SDW~Lavel, d)
summary(result)
TukeyHSD(result)
```
libraryの「multcomp」を使うと便利です


# 実験デザインと解析
実験デザインと解析は密接に関係している


## 基礎編：two-way ANOVA
```R
d<-read.table("002.txt", header=T)
head(d) #列名確認

# 完全無作為化法: completely randomized design
result<-aov(SDW~Limitation*Inoculation,d)
summary(result)

# 乱塊法: radomized block design
result<-aov(SDW~Rep+Limitation*Inoculation,d)
summary(result)

# 分割区法: split-plot design
result<-aov(SDW~Rep+Limitation+Error(Rep/Limitation)+Inoculation+Limitation*Inoculation,d)
summary(result)

# 省略記法使えるが理解を妨げる
result<-aov(SDW~Rep+Error(Rep/Limitation)+Limitation*Inoculation,d)
summary(result)

# 二方分割区法: strip-plot design
result<-aov(SDW~Rep+Limitation+Inoculation+Error(Rep/(Limitation*Inoculation))+Limitation*Inoculation,d)
summary(result)

# 省略記法使えるが理解を妨げる
result<-aov(SDW~Rep+Error(Rep/(Limitation*Inoculation))+Limitation*Inoculation,d)
summary(result)

# 分散の分割との比較・確認
result<-aov(SDW~Rep*Limitation*Inoculation,d)
summary(result)
```


## 応用編，いろいろ
```R
# 組み合わせ1: split-split-plot design
d<-read.table("404.txt", header=T)
head(d) #列名確認
result<-aov(Y~Rep+Error(Rep/Nitrogen/Management/Var)+Nitrogen*Management*Var,d) # 省略記法
summary(result)

# 分散確認
result<-aov(Y~Rep*Nitrogen*Management*Var,d)
summary(result)

# 組み合わせ2: strip-split-plot design
d<-read.table("412.txt", header=T)
head(d) #列名確認
result<-aov(yield~Rep+Error(Rep/(Variety*Nitrogen)/Seeds)+Variety*Seeds*Nitrogen,d) #省略記法
summary(result)

# 分散確認
result<-aov(yield~Rep*Variety*Nitrogen*Seeds,d)
summary(result)

# 年次，季節変動: over years and seasons
d<-read.table("807.txt", header=T)
head(d) #列名確認
result<-aov(y~Error(year/rep)+va*year,d)
summary(result)

MeanSqY<-summary(result)[[1]][[1]][[3]]
degFree1<-summary(result)[[1]][[1]][[1]]
MeanSqYYS<-summary(result)[[2]][[1]][[3]]
degFree2<-summary(result)[[2]][[1]][[1]]
F<-MeanSqY/MeanSqYYS
df(F, degFree1, degFree2)

# 分散確認
result<-aov(y~rep*va*year,d)
summary(result)

# 地域変動: over sites
# 地域+乱塊法(one-way)
d<-read.table("813_g.txt", header=T)
head(d) #列名確認
result<-aov(Yield~Location+Error(Location/Rep)+Variety*Location,d)
summary(result)

MeanSqSite<-summary(result)[[1]][[1]][[3]]
degFree1<-summary(result)[[1]][[1]][[1]]

MeanSqRRS<-summary(result)[[2]][[1]][[3]]
degFree2<-summary(result)[[2]][[1]][[1]]

F<-MeanSqSite/MeanSqRRS
df(F, degFree1, degFree2)

# 分散確認
result<-aov(Yield~Location*Rep*Variety,d)
summary(result)

# 地域+分割区法(two-way)
d<-read.table("816.txt", header=T)
head(d) #列名確認
result<-aov(Y~Site+Nitrogen+Error(Site/Rep/Nitrogen)+Var*Nitrogen*Site,d)
summary(result)

MeanSqSite<-summary(result)[[1]][[1]][[3]]
degFree1<-summary(result)[[1]][[1]][[1]]

MeanSqRRS<-summary(result)[[2]][[1]][[3]]
degFree2<-summary(result)[[2]][[1]][[1]]

F<-MeanSqSite/MeanSqRRS
df(F, degFree1, degFree2)

# 分散確認
result<-aov(Y~Site*Nitrogen*Rep*Var,d)
summary(result)
```
年次変動，地域変動に意味がないという意見もある

# 相関，回帰，重回帰
```R
d<-read.table("004.txt", header=T)
head(d) #列名確認
# 相関
cor(d[,2:4])
# 単回帰
result<-lm(TC~TN,d)
summary(result)
# 重回帰
result<-lm(TC~TN+pH,d)
summary(result)
# 標準化
d2<-scale(d[,2:4])
d2<-data.frame(dist=d[,1], d2)
result<-lm(TC~TN+pH,d)
summary(result)
```

# 線形モデル，一般化線形モデル，一般化線形混合モデル，ベイズ推定
以下は話題として
- 線形モデル = 一般線形モデル: LM
- 一般化線形モデル: GLM
- 一般化線形混合モデル: GLMM
- ベイズ推定
