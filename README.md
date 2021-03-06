# RedWineQuality_DataAnalysis
Using R to find which chemical properties influence the quality of red wines?
-----------------------------------------------------------------------------

```{r global_options, include=FALSE}
knitr::opts_chunk$set(echo=FALSE, warning=FALSE, message=FALSE)
```

```{r}
### Importing the required libraries
library(ggplot2)
library(gridExtra)
library(scales)
library(GGally)
library(reshape)
library(dplyr)
```


# Red Wine Quality Exploration 
This report explore how the red wine quality could be affected by different 
chemical ingredients. 

# Univariate Plot Section
At the beginning let's explore our data structure.

* The data consists of : 1599 observations & 13 variables.
* "quality" variable has int as data type. It is a categorical, so it should be converted into factor.

```{r }
### first load the data 
wd <- getwd()
redwine<-read.csv(paste(wd,'/wineQualityReds.csv',sep=""))
dim(redwine)
str(redwine)
#delete X: it represents the row number from dataset
redwine <- within(redwine,rm(X))
#redwine$quality <- factor(redwine$quality)
table(redwine$quality)

```

## The summary statistics for each variable:

### Fixed.Acidity 

Description: most acids involved with wine or fixed or nonvolatile (do not 
evaporate readily).

```{r }
ggplot( aes(x=fixed.acidity), data = redwine)+
  geom_bar(binwidth = 0.5) + 
  scale_x_continuous(breaks = seq(0,16,1) ) + 
  scale_y_continuous(breaks = seq(0,280,20)) + 
  ylab("Count") + 
  labs(title="Fixed.Acidity Histogram") + 
  xlab('Fixed.Acidity Amount(g/dm3)')
print('Fixed.Acidity Statistics:')
summary(redwine$fixed.acidity)
```

Fixed acidity (tartaric acid) has a slight right tail but I think overall it has 
a normal distribution with mean of 8.32 g/dm3 and median 7.9 g/dm3:

* The distribution is peaking around 7.2 g/dm3. 
* The minimum value for the fixed.acidity is 4.6 g/dm3.
* When we change the binwidth, we can see clear gaps between fixed.acidity 
values of 14 and 16. I wonder why these gaps exist? and does it have any effect 
of the wine quality?

### Volatile.Acidity 

Description: the amount of acetic acid in wine, which at too high of levels can 
lead to an unpleasant, vinegar taste.

```{r }
ggplot(aes(x=volatile.acidity) ,data= redwine) + 
  geom_histogram(binwidth = 0.05) + 
  scale_x_continuous(breaks=seq(0,2,0.1)) + 
  labs(title="Volatile.Acidity Histogram") + 
  ylab("Count") + xlab('Volatile.Acidity Amount(g/dm3)')
print('Volatile.Acidity Statistics Summary:')
summary(redwine$volatile.acidity)
```

The Volatile.Acidity Histogram shows right skewed distribution. I believe that 
the volatile acidity values must be small as the more amount of acetic acid wine 
the more unpleasant taste we get that interpret most of values are less than 1.0 g/dm3. I wonder what will be the quality of wine that has more than 1 g/dm3 of 
acidic acid?

### Citric.Acid 

Description: found in small quantities, citric acid can add 'freshness' and 
flavor to wines

```{r }
# creating two graphs
cc1<- ggplot(aes(x=citric.acid), data = redwine)+
  geom_histogram(binwidth = 0.05)+ 
  labs(title="Citirc.Acid Histogram") + 
  ylab("Count")+xlab("Citric.Acid Amount(g/dm3)")

cc2<- ggplot(aes(x=citric.acid,y=..count../sum(..count..)), data = redwine)+
  geom_histogram(binwidth = 0.05)+ 
  labs(title="Citirc.Acid Percentile") + 
  ylab("Percentile")+
  xlab("Citric.Acid Amount(g/dm3)")
grid.arrange(cc1,cc2,ncol=2)
print('Citirc.Acid Statistics Summary:')
summary(redwine$citric.acid)
print("Top 5 values for Citric Acid:")
head(sort(table(redwine$citric.acid),decreasing = TRUE),5)
```

Citric.Acid quantities histogram shows a right skewed distribution that peaking around 0.0 , 0.25 & 0.5 g/dm3 of citric acid. I thought the majority of wine observations will have citric acid as an ingredient, but it seems I was wrong. 
The highest percent of red wine  have 0 g/dm3 of citric acid. I am thinking does 
that means the quality will be decreased if the wine doesn't have citric acid as 
it might miss the 'freshness' and good flavor feeling. We will see how the wine quality will be affected by this in next sections.

### Residual.Sugar 

Description: the amount of sugar remaining after fermentation stops, it's rare 
to find wines with less than 1 gram/liter and wines with greater than 45 
grams/liter are considered sweet.

```{r }
ggplot(aes(x=residual.sugar), data = redwine) + 
  geom_histogram(binwidth =0.2)+ 
  scale_x_continuous(breaks = seq(0,16,1)) + 
  scale_y_continuous(breaks=seq(0,700,50))+ 
  labs(title='Residual.Sugar Histogram') + 
  ylab('Count') + 
  xlab('Residual.Sugar Amount (g/dm3)')
```

The Residual.Sugar histogram shows a right skewed distribution. It also shows 
some red wine observations have less than 1 g/dm3 of sugar ( 2 observations has 
0.9 g/dm3 ). Most of wines have a value of sugar that between 1.2 and 3.5 g/dm3.

```{r }
print("Wines with less than 1g/dm3 of sugar amount:")
table (subset (subset(redwine, select = residual.sugar) , residual.sugar < 1))
```

Let's scale Residual.Sugar variable by using logarithmic transformation to make sure it doesn't have normal distribution on another scale. The below graph shows that even by scaling the varible it still has reight skewed distribution.

```{r}
ggplot(aes(x=residual.sugar), data = redwine) + 
  geom_histogram()+ 
  labs(title='Residual.Sugar Histogram') + 
  ylab('Count') + 
  xlab('log(Residual.Sugar Amount)  (g/dm3)')+
  scale_x_log10(breaks=seq(1,15,2))
```

### Chlorides

Description: the amount of salt in the wine.

```{r }
ggplot(aes(x=chlorides) ,data=redwine)+
  geom_histogram(binwidth = 0.01) + 
  scale_x_continuous(breaks = seq(0,0.7,0.05)) + 
  scale_y_continuous(breaks = seq(0,480,50)) + 
  labs(title="Chlorides Histogram") + 
  ylab('Count')+xlab('Chlorides(g/dm3)')
print("Chlorides Statistics Summary:")
summary(redwine$chlorides)
```

The Chlorides Histogram show a right skewed distribution that is peaking around 
0.08 g/dm3. Most of wines has a Chlorides amount between 0.03 and 0.125 g/dm3. 
The histogram also shows that there is a gap around 0.3, 0.425 and 0.55. Also, 
the max value of for chlorides is 0.611. I wonder how chlorides affects the wine quality? 

Let's subset the data and see the quality of wines with amount of chloride 
greater than 0.35 g/dm3. It seems that we have 0 of wines observations having 
quality level of 8. I think the chlorides amount might might affect the wine 
quality. We will try to figure that out in the next sections.

```{r }
c_data <- subset(redwine, chlorides > 0.35)
table(c_data$quality)
```

```{r }
mg_to_g_trans = function() 
  trans_new('mg_to_g', transform = function(x) x/1000,
                                      inverse = function(x) x*1000)
```


### Free.Sulfur.Dioxide

Description: the free form of SO2 exists in equilibrium between molecular SO2 
(as a dissolved gas) and bisulfite ion; it prevents microbial growth and the oxidation of wine.

```{r  }
ggplot(aes(x = free.sulfur.dioxide), data = redwine) + 
  geom_histogram(binwidth = 1.5)+
  scale_x_continuous(breaks = seq(0,75,5)) + 
  scale_y_continuous(breaks = seq(0,150,10))  + 
  ylab("Count") +  
  xlab("Free.Sulfur.Dioxide(mg/dm3)") + 
  labs(title="Free.Sulfur.Dioxide Histogram")

print("Free.Sulfur.Dioxide Statistics Summary:")
summary(redwine$free.sulfur.dioxide)
```

The Free.Sulfur.Dioxide Histogram shows a right skewed distribution that is 
peaking around 6. The majority of wines have an amount of free.sulfur.dioxide 
between 2.5 and 32.5 mg/dm3. I want to explore the quality of wines that have 
amount of free.sulfur.dioxide greater than 65 mg/dm3. Do they have high quality ranking?

### Total.Sulfur.Dioxide

description: amount of free and bound forms of S02; in low concentrations, SO2 
is mostly undetectable in wine, but at free SO2 concentrations over 50 ppm, SO2 becomes evident in the nose and taste of wine.

```{r }
ggplot(aes( x=total.sulfur.dioxide/1000), data= redwine)+ 
  geom_histogram(binwidth = 0.005) + 
  xlab("Total.Sulfur.Dioxide(g/dm3)") +
  ylab("Count")+
  labs(title="Total.Sulfur.Dioxide Histogram") + 
  scale_x_continuous(breaks = seq(0,0.5,0.02))
print("Total.Sulfur.Dioxide Summary Statistics (g/dm3):")
summary(redwine$total.sulfur.dioxide/1000 )
```

Total.Sulfur.Dioxide histogram shows a right tailed distribution where most of 
values are less than 0.3 g/dm3. The distribution peaks around 0.028 g/dm3. The majority of wines has an amount of total sulfur dioxide between 0.01 & 0.085 
g/dm3.

### Density

Description: the density of water is close to that of water depending on the 
percent alcohol and sugar content

```{r }
ggplot(aes(x= density), data = redwine) + 
  geom_histogram(binwidth = 0.0005)  + 
  scale_x_continuous(breaks = seq(0.99,1.005,0.001)) +
  labs(title='Density Histogram') + xlab("Density (g/cm3)") + 
  ylab("Count") 
print("Density Summary Statistics (g/cm3)")
summary(redwine$density)

```

Density Histogram shows a normal distribution with mean of 0.997 g/cm3. There 
are some gaps exist on both tight & left tails. I wonder how the density affect 
the wine quality? I believe the closer it gets to the water density the higher quality it has.

### PH

Description: describes how acidic or basic a wine is on a scale from 0 
(very acidic) to 14 (very basic); most wines are between 3-4 on the pH scale

```{r }
ggplot(aes(x= pH), data = redwine) + 
  geom_histogram(binwidth = 0.05) + 
  labs(title="PH Histogram") + 
  xlab("pH") + 
  ylab("Count") +
  scale_x_continuous(breaks = seq(2,4.5,0.1)) 

print("PH Summary Statistics:")
summary(redwine$pH)
```

PH histogram show a normal distribution with mean of 3.3 on pH scale. There is 
less than 1% of wines that are close to be very acidic and none of the is very 
basic on pH scale. 

### Sulphates

Description: a wine additive which can contribute to sulfur dioxide gas (S02) 
levels, which acts as an antimicrobial and antioxidant.

```{r }
ggplot(aes(x=sulphates) ,data = redwine) + 
  geom_histogram(binwidth = 0.05) + 
  scale_x_continuous(breaks = seq(0,2.2,0.1)) + 
  labs( title="Sulphates Histogram") + 
  xlab("Sulphates(g/dm3)") + 
  ylab("Count")
print("Sulphates Statistics Summary (g/dm3):")
summary(redwine$sulphates)
```

The Sulphates has a right tailed distribution that is peaking around 0.6. It 
also shows many gaps after the value of 1.2. Less than 0.4% of red wine 
observations has amount of sulphates greater than 1.65 g/dm3.

### Alcohol

Description: the percent alcohol content of the wine.

```{r }
ggplot(aes(x=alcohol),data=redwine) + 
  geom_histogram(binwidth = 0.1) + 
  scale_x_continuous(breaks = seq(8,15,0.5))+
  scale_y_continuous(breaks = seq(0,150,20)) + 
  labs(title="Alcohol Histogram") + 
  xlab("Alcohol(% by volume)") + ylab("Count")
print("Alcohol (% by volume) Summary Statistics:")
summary(redwine$alcohol)
```

Alcohol histogram shows a positive skewed distribution with mode of 9.5 and mean around 10.4 % by volume. The majority of have between 9 and 13 % of alcohol.  I 
think wine with qood quality will have a high percent of Alcohol. Let's see if I 
am right or not in the next sections.

### Quality

Description: the wine quality (score between 0 and 10)

```{r }
ggplot(aes(x=quality ,y=..count../sum(..count..)), data = redwine) + 
  geom_bar() + 
  labs(title="Quality Percentile Chart") + 
  ylab("Percentile") + 
  xlab("Quality") + 
  scale_y_continuous(breaks = seq(0,0.45,0.05))
print("Quntity Staistics Summary:")
summary(redwine$quality)
```

The Quality Percentile Chart shows that around 80% of red wines have quality 
level of 5 or 6. Around 1.25% of red wines have quality level greater than 7 
which is very small percent. Now, I am thinking about the chemical components 
and their amounts that was included in those 1.25% of highest quality red wines. 
What are the chemical prosperities that affect the wine quality? 

# Univariate Analysis

The data has 1599 red wine observations with 12 features ( as described in 
details in previous section)

* The dependent feature/variable is the wine quality while the other 11 
variables are the predictor variables.
* The quality variable is an ordered factor with 10 levels (0 is the lowest and 
10 is the highest quality level).
* 80% of the red wines have a quality level of 5 or 6.
* The majority of wines have an amount of acetic acid -volatile.acidity- between 
0.2 to 0.8 g/dm3.
* 8% of red wines have 0 participation of citric acid while the majority 
remaining have an amount less than 0.75 g/dm3.
* Most of red wines have an amount of residual sugar between 1 and 3.5 g/dm3.
* The majority of wines have an amount of chlorides between 0.03 and 
0.125 g/dm3. 
* 50% of red wines have a density around 0.996 g/cm3. The maximum density is 
1.004 g/cm3.
* The majority of wines have pH value between 3 and 3.5
* The mean alcohol (% by volume ) is 10.42% and the maximum is 14.9
* The max value for sulphates is 2 g/dm3 and the minimum is 0.33

### What is/are the main feature(s) of interest in your dataset?

After googling and exploring the data I think the main features that might 
influence the quality of red wines are citric acid amount, and alcohol percent.

### What other features in the dataset do you think will help support your investigation into your feature(s) of interest?

Fixed Acidity, volatile acidity, chlorides and residual sugar would be helpful 
in defining the red wine quality as they are affecting the taste characteristics 
of wine which are very important.

# Bivariate Plots Section

Before I start plotting the data. Let's the find the correlation between the different variables just to be sure of the main features of interest:

```{r fig.width=8 , fig.height=8  }
# Find the correlation coefficient between the variables
# At the beginning I will convert the Quality from factor to numeric to find 
#the correlation between it and different variables
redwine$quality <- as.numeric(redwine$quality)
table(redwine$quality)
# See the quality correlation with other variable 
ggpairs(data=redwine,  columns = c(1,2,3,4,5,11,12))
```

The above matrix shows the following:

* There is a positive & moderate association between quality & alcohol --> correlation coefficient =   0.476
* Also, there is a weak association between citric acid amount & the wine 
quality --> correlation coefficient= 0.226
* There a negative association between the volatile acid amount & the wine 
quality --> correlation coefficient = -0.391

Now, let’s test and know more about the above associations.

#### Alcohol % by volume and wine quality

```{r }
table(redwine$quality)
ggplot(aes(x=alcohol, y=quality), data = redwine) + 
  geom_jitter( alpha = 1/5,width = 0.5, height = 0.5) + 
  ylab("Quality")+xlab("Alcohol % by volume ") + 
  labs(title="Quality/Alcohol Scatterplot") + 
  stat_smooth(method = "lm", colour="red", size=0.5, se=FALSE)
```

I think we can say there is a slight linear relationship between quality &
alcohol percent. The data shapes horizontal stripes. As alcohol % increase the quality increase. 

```{r  }
cor.test(redwine$quality, redwine$alcohol)
```

I'd like to see the mean alcohol % per each quality 

```{r }
# Draw the boxplot that shows the statistics for alcohol% categorized by wine 
#quality
ggplot(aes(x=factor(quality), y=alcohol), data = redwine) + 
  geom_boxplot() + xlab("Quality (bad ----------> very good)") + 
  ylab("Alcohol % by volume") + 
  stat_summary(fun.y=mean, geom="line", aes(group=1, colour="mean")) +
  scale_colour_manual(values=c("mean"="blue")) +  # Set line color to blue
  labs(colour="")  # Get rid of the redundant legend title

```

It seems that the highest quality wine has the highest alcohol % by volume. The 
chart shows some outliers that violate this assumption such as wines with 
quality level of 5. The blue line shows how the mean alcohol% forms a linear 
relation with the quality level, so I think our assumption still true.

*Statistics Summary:*

```{r }

redwine.quality.alcohol <- redwine[,c("quality","alcohol")] %>% 
  group_by(quality)%>%
  summarise(mean_alcohol_percent= mean(alcohol),
            median_alcohol_percent = median(alcohol),
            total_count=n())
print(redwine.quality.alcohol)
```

#### Citric acid vs. Wine Quality

The correlation coefficient value is less than 0.3 so I think there is no strong association between citric acid amount & wine.

```{r }
cor.test(redwine$quality,redwine$fixed.acidity)

ggplot(aes(x=citric.acid, y=quality), data=redwine)+
  geom_point() + 
  xlab("Citric.Acid (g/dm3)") + 
  ylab("Quality") + 
  labs(title="Citric Acid Amount/Quality Scatterplot") + 
  stat_smooth(method = "lm", colour="red", size=0.5, se=FALSE)
```

After using jitter..

```{r }
ggplot(aes(x=citric.acid, y=quality), data=redwine)+ 
  geom_jitter(width = 0.25, alpha=1/4)+
  xlab("Citric.Acid (g/dm3)") + 
  ylab("Quality") + 
  labs(title="Citric Acid Amount/Quality Scatterplot")+
  scale_y_continuous(breaks = seq(0,9,1)) +  
  stat_smooth(method = "lm", colour="red", size=0.5, se=FALSE)
```

```{r }
ggplot(aes(x=factor(quality), y=citric.acid), data = redwine) + 
  geom_boxplot() + xlab("Quality (bad ----------> very good)") + 
  ylab("Citric.Acid (g/dm3)") + 
  stat_summary(fun.y=mean, geom="point", aes(group=1, colour="mean")) +
  scale_colour_manual(values=c("mean"="blue")) +  # Set line color to blue
  labs(colour="")  # Get rid of the redundant legend title
```

From above chart, the mean of citric acid amount (g/dm3) is increasing as we 
move from the lowest wine quality level to the highest quality level. Also, we 
should mention that the differences are not that big but it still shows a slight linear relationship.


*Citric acid amount and wine quality statistics summary:*

```{r }
# This function calculate the statistics summary for the given variable and 
# display it categorized by the wine quality level
view_summary <- function(X){
  is.vector(X)
  quality_vector <- sort(unique(redwine$quality))
for (qv in quality_vector ){
  cat(paste(paste("Quality Level:", qv, sep=" "),"\n", sep = ""))
  print (summary(subset(X, redwine$quality == as.numeric(qv))))
  print("-------------------------------------------------------------------")
}
  return()
}

```

```{r }
view_summary(redwine$citric.acid)
```

The majority of zero citric acid wines belong to the average quality of red
wines. To see this, I will subset the wines with zero citric acid to see their distribution.

```{r }
#rm(df2)
zero.citic.acid <- subset(redwine,citric.acid ==0)
df3 <- table(zero.citic.acid$quality)
df2 <- as.data.frame(df3)
names(df2)[1] <- "Quality"
df2
df2["Rel.Freq"]<- NA
df2$Rel.Freq <- df2$Freq/sum(df2$Freq)
df2$Rel.Freq <- round(df2$Rel.Freq*100, 2)
print(df2)
```

#### Volatile Acidity vs. Quality

The below graph shows a moderate negative association between wine quality & 
volatile acidity amount. 

```{r }
ggplot(aes(x= volatile.acidity, y=quality), data = redwine)+geom_jitter(alpha=1/5)+geom_smooth(method = "lm",linetype=2,size=0.5,se=FALSE, color="red") + 
  ylab("Quality") + 
  xlab("Volatile Acidity (g/dm3)") + 
  labs(title="Quality/Volatile Acidity Relationship")

ggplot(aes(x=factor(quality), y=volatile.acidity), data = redwine) + 
  geom_boxplot() + 
  xlab("Quality (bad ----------> very good)") +
  ylab("Volatile.Acidity (g/dm3)") + 
  stat_summary(fun.y=mean, geom="point", aes(group=1, colour="mean")) +
  scale_colour_manual(values=c("mean"="blue")) +  # Set line color to blue
  labs(colour="")  # Get rid of the redundant legend title
```

The least quality wines -that belong to level 3- have the maximum value of the volatile acidity amount, but the highest quality wines -that belong to levels 7 
and 8-have the minimun value of volatile acidity amount. That confirm what was mentioned, the increase in the amount of volatile acidity might lead to 
unpleasant taste.

*Volatile Acidity Amount and wine quality statistic summary:*

```{r }
view_summary(redwine$volatile.acidity)
volatile_acidity_quality_m <- lm(quality~ volatile.acidity, data = 
                                   subset( redwine, volatile.acidity <=  quantile(redwine$volatile.acidity,0.999)))
summary(volatile_acidity_quality_m)
```

Building a linear model using the volatile.acidity as a predictor --> R-squared 
has a small value, so I think knowing the volatile acidity amount alone might 
not be adequate to predict the wine quality. 


#### Density and wine quality

```{r }
cor.test(redwine$quality, redwine$density)
ggplot(aes(x=density, y=quality),data=redwine)+ 
  geom_jitter(alpha=1/3,height  = 0.3)+
  geom_smooth(method = "lm",se=FALSE, size=0.5)
```

There is no a strong association between wine density & its quality. 

*Density and wine quality statistics summary:*

```{r }
ggplot(aes(x=factor(quality), y= density),data = redwine) + 
  geom_boxplot() + 
  ylab("Density(g/cm3)")+
  xlab("Wine Quality")+ 
  stat_summary(fun.y = "mean",geom="point",aes(colour="mean",group=1))+
  scale_colour_manual(values=c("mean"="red")) + labs(colour="")
view_summary(redwine$density)
```

##### Density vs. Residual Sugar
 
The below graph goes against my intuition. It shows that there is no association between both variables. Most of wines have between 1.5 to 3 g/dm3  
of residual sugar. I can't say that the variance in our samples' wine density 
is due to the change in sugar amount. 

```{r }
cor.test(redwine$residual.sugar, redwine$density)
ggplot(aes(x=residual.sugar, y=density), data=redwine)+
  geom_jitter(alpha=1/5, height = 0.5)+ 
  scale_x_continuous(breaks = seq(1, 16, 1)) +
  xlab("Residual Sugar (g/dm3)")+ 
  ylab("Density (g/cm3)")+  
  stat_smooth(method = "lm", colour="red", size=0.5, se=FALSE)
```

##### Fixed Acidity vs. Citric Acid

There is a moderate association between citric acid amount and the fixed acidity. 
The below graph show that the fixed acidity amount is increasing with the 
increase of citric acid amount.

```{r }
cor.test(redwine$fixed.acidity, redwine$citric.acid)
ggplot(aes(x=citric.acid, y =fixed.acidity), data= redwine)+
  geom_jitter(alpha=1/5)+
  xlab("Citric Acid (g/dm3)") + 
  ylab("Fixed Acidity (g/dm3)")+  
  stat_smooth(method = "lm", colour="green", size=0.5, se=FALSE)
```

##### Alcohol % by volume  vs. Volatile Acidity 

At the begining, let's check the correlation between Alcohol% and Volatile Acidity Amount into our data. 
There is a negtive association between alcohol % and volatile acidity amount: 

```{r}
cor.test(redwine$alcohol, redwine$volatile.acidity)
```

The graph below shows the moderate negative association between alcohol % and volatile acidity amount included in wine. 

```{r}
ggplot(aes(x=alcohol, y= volatile.acidity), data=redwine)+ 
  geom_point()+ 
  geom_smooth(method = "lm", se=FALSE)+
  labs(title="Alcohol% vs. Volatile Acidity Amount")+
  xlab("Alcohol % by volume")+
  ylab("Volatile Acidity (g/dm3)")
  
```


# Bivariate Analysis

#### Talk about some of the relationships you observed in this part of the investigation. How did the feature(s) of interest vary with other features in      the dataset?

* The wine quality correlates strongly with alcohol percent where the wine 
highest quality has the highest highest mean value for the alcohol % by volume.
* The wine quality also has a positive association with the citric acid amount. 
Most of wines has a small amount of citric acid that doesn't exceed 0.8 g/dm3. 
The Majority of wines with highest quality level (7 or 8 ) have a citric acid 
amount between 0.25 and 0.5 g/dm3. High percent of wines with quality level of 
3 has a 0 g/dm3 of citric acid.
* Volatile acidity amount has a negative correlation with the wine quality. Most 
of red wines with quality level of 8 have an volatile acidity amount between 0.3 
and 0.5 g/dm3, while, wines with quality level of 3 used to have a volatile 
acidity amount greater than 0.4 g.dm3. 
* In our data samples, there is no strong linear relationship between density 
and wine quality. But the graphs above showed that there is a slight association where the highest quality level of wine had the least mean of density and the 
lowest quality level of wine had the highest mean value of density.

#### Did you observe any interesting relationships between the other features     (not the main feature(s) of interest)?

Yes, I observed that: there is a positive correlation between citric acid amount 
and fixed acidity amount.Density also strongly correlates with alcohol percent 
by volume included.

#### What was the strongest relationship you found?

* The wine quality is positively correlated with alcohol percent included.
* The wine quality is negatively and strongly correlate with volatile acidity 
amount included.
* Citric acid amount also correlate (positive association) to red wine quality,
but less strongly than volatile acidity amount and citric acid.

# Multivariate Plots Section

```{r }
ggplot(aes(y=density , x=alcohol, color=factor(quality) ),data=redwine)+geom_point(alpha = 0.8, size = 2 )+
  geom_smooth(method = "lm",size=1, se = FALSE)+
  labs(title= "Wine Quality vs. Alcohol & Density") +
  ylab("Density (g/cm3)") +
  xlab("Alcohol% by volume") + 
  scale_color_brewer(type='seq',
                   guide=guide_legend(title='Quality'))+
  theme_dark()
```

There is a negative association between alcohol% by volume and density (g/cm3).
The average quality of wines have a density between 0.995 and 1 g/cm3  and 
alcohol less than 11% by the wine volume.  As alcohol percent increase the wine quality increase and density decrease. 

```{r }
ggplot(aes(y=density , x=citric.acid),data=redwine)+
  geom_point()+ facet_wrap(~quality)+
  geom_smooth(se=FALSE) +
  xlab("Citric Acid (g/dm3)") + 
  ylab("Density (g/cm3)")+ 
  labs(title="Quality vs. Density & Citric Acid")
```

Majority of wines with average quality (levels 5 & 6 ) tend to have small citric amount less than 0.5 (g/dm3). Wine Quality of level 8 shows a clear linear relationship between citric acid amount and density.


```{r }
ggplot(aes( colour=factor(quality),x=volatile.acidity ),data=redwine)+
  geom_density()+
  xlab("Volatile Acidity Amount (g/dm3)") + 
  ylab("Density") +
  labs(colour = "Quality", title="Volatile Acidity Density vs. Quality")+
  scale_color_brewer(type='seq',
                   guide=guide_legend(title='Quality'))+
  theme_dark()
  
```

The above density chart confirms the negative correlation between volatile 
acidity amount and the quality. Wines with highest quality tend to occur more 
often where lower values of volatile acidity
were included, while wines with lowest quality tend to include higher amount of volatile acidity. 

```{r }
ggplot(aes(x=citric.acid, y = volatile.acidity, colour=factor(quality)), 
       data = redwine) + geom_point(size=1, alpha=1/2) + 
  scale_colour_brewer(palette ="Paired")+ 
  labs(colour="Quality") + 
  xlab("Citric Acid (g/dm3)")+ 
  ylab("Volatile Acidity (g/dm3)") + 
  labs(title="Quality vs. Volatile Acidity & Citric Acid")+
  geom_smooth(method = "lm", linetype=1, size=0.7, se=FALSE)
```

The graph above shows that, highest quality wines tend to happen where volatile acidity amount less than 0.6 (g/dm3) and citric acid amount between 0.25 and 0.6 (g/dm3), while lowest quality wines tend to have less amount of citric acid 
(less than 0.125 g/dm3) and higher amount of volatile acidity. Average wine 
quality tend to happen where volatile acidity amount is between 0.2 and 0.8 
(g/dm3) and citric acid amount less than or equal to 0.5 g/dm3. 
The yellow line shows the negative association between volatile acidity amount & citric acidity amount.

# Final Plots and Summary

Based on the previous analysis, we can conclude that the red wine quality is 
strongly affected by the following variables:

* Volatile Acidity Amount --> negative association with the wine quality

```{r }
ggplot(aes(factor(quality), 
            volatile.acidity), 
        data = redwine) +
  geom_jitter( alpha = .3)  +
  geom_boxplot( alpha = .5,color = 'blue')+
  stat_summary(fun.y = "mean", 
               geom = "point", 
               color = "red", 
               shape = 8, 
               size = 4)+
  geom_smooth(aes(quality-2, 
                    volatile.acidity),
                method = "lm", 
                se = FALSE,size=2)+
  labs(title="Quality vs. Volatile Acidity amount")+ xlab("Quality") + 
  ylab("Volatile Acidity (g/dm3)")

```

Majority of wines with the highest quality tend to have lower amount of volatile acidity while majority of lowest quality wines tend to have higher values of 
volatile acidity amount.

* Alcohol % by volume --> positive association with the red wine quality

```{r }
g<- ggplot(aes(x=factor(quality), y=alcohol),data = redwine)

g+geom_jitter(alpha=1/2, aes(color=factor(quality)),
              position = position_jitter(width = .7)) + 
  ylab("Alcohol % by volume") + xlab("Quality")+
  labs(colour="Quality") +
  labs(title="Quality vs. Alcohol%")+
  stat_summary(fun.y = "mean", geom="point",aes( group=1))+
  annotate("text", x = 4, y = 15.5, 
           label = "Mean is is indicated with black points", color="white")+
  stat_summary(fun.y = "mean", geom="line",aes( group=1))+
  scale_colour_brewer()+ 
  theme_dark()
  #theme( panel.background = element_rect(fill = "grey70"))
```

The graph above shows how the quality is affected by the alcohol % included. 
There is a clear positive relation that is represented with the black line (mean value for alcohol per each quality level).

```{r }
ggplot(aes(x=volatile.acidity, y = alcohol, colour = factor(quality)), data = redwine)+geom_jitter(alpha=1/2) + 
  scale_color_brewer(palette = 18) +
  xlab("Volatile Acidity (g/dm3)") + 
  ylab("Alcohol % by volume")+
  labs(colour="Quality")+
  geom_smooth(method = "lm", se=FALSE)+ 
  labs(title="Quailty vs. Alcohol % and Volatile Acidity Amount")
```

Volatile acidity amount & Alcohol % by volume are good predictors that can be 
used in predicting red wine quality. This graph above shows how each quality 
level of red wine is defined according to those two variables. As we mentioned before, red wine with highest quality tend to have small amount of volatile 
acidity and high amount of alcohol compared to the average value. 

# Reflection 

##### Where did I run into difficulties in the analysis?

There were no strong correlation between dependent and independent variables. 
There were outliers in each quality level which might have affected the analysis. 
Our data sample doesn't include data for all quality levels (1 to 10), so we
don't know if our findings will apply to wines with quality level of 10 or 1.  

##### Where did I find successes?

By visualizing bivariate & multivariate plots, we had the chance to see how 
wine quality is affected by the amount chemical components included. Our final 
findings showed that wine quality is clearly affected by volatile acidity 
amount, citric acid amount and alcohol percent by volume included. The analysis 
also confirmed our intuitions about the increase of volatile acidity amount 
might lead to decrease the wine quality because of undesired salty taste it 
might has. Highest quality wines tend to have higher alcohol percent and lower 
citric acid amount.

##### How could the analysis be enriched in future work (e.g. additional data        and analyses)?

I think the analyses can be enriched if we have more data that belong to the 
missing quality levels. Also, if we have more categorical variables that 
describe the wine quality. I think we can convert some numeric variables to categorical variables by mapping their values into tiers, for example, pH 
variable: any wine is on a scale from 0 (very acidic) to 14 (very basic); most 
wines are between 3-4 on the pH scale. we can map those values into 3 or 4 
categories such as (pH value between 0 and 3  --> very acidic, >3 and <= 4 --> moderate acidic, >4 and <= 8 --> less acidic , and >8 --> tend to be very basic).
Also, the analysis can be enriched by checking the relationships between all independent variables as we might find some interested combination that affect 
the wine quality.


