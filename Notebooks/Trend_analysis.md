Trend analysis
================
Timo Kelder
November 13, 2019

In this notebook, we are going to perform trend analysis based on
extreme value theory. We will quantify the trend in the 100-year
precipitation extremes over time (1981-2015) for the West Coast of
Norway and for Svalbard.

### Import data and packages

The data for the West coast and Svalbard are loaded.

``` r
# dir='//home/timok/timok/SALIENSEAS/SEAS5/ensex'
# plotdir=paste0(dir,'/statistics/multiday/plots')
# dir='/home/timok/Documents/ensex'
# plotdir='/home/timok/Documents/ensex/R/graphs'
# dir='C:/Users/Timo/Documents/GitHub/EnsEx/Data'
# plotdir='/home/timok/Documents/ensex/R/graphs'

dir='C:/Users/gytk3/OneDrive - Loughborough University/GitHub/EnsEx/Data'
source('Load_data.R')
library(extRemes)
library("ggpubr")

names(dimnames(Extremes_WC)) <- c('Member', 'Leadtime', 'Year')
names(dimnames(Extremes_SV)) <- c('Member', 'Leadtime', 'Year')
df_WC=adply(Extremes_WC, 1:3)
df_SV=adply(Extremes_SV, 1:3)
obs=Extremes_obs[as.character(1981:2015)]

year_vector=as.numeric(levels(df_WC$Year))[df_WC$Year] ###The year is a factor, extract the values  
```

## We then compare the 1981 and 2015 return value plots

A GEV distribution including parameters that linearly relate to the time
period 1981-2015 is fitted to the UNSEEN ensemble. The resulting return
value plots for the covariates 1981 and 2015 are shown here.

``` r
extremes_wc= df_WC$V1 * mean(obs)/mean(df_WC$V1) ## we create a mean bias corrected series  

rperiods = c(2, 5, 10, 20, 50, 80, 100, 120, 200, 250, 300, 500, 800,2000,5000)

RV_ci <- function(extremes,covariate,return_period,covariate_values,GEV_type) { ## A function to fit the GEV and obtain the return values 
  fit <- fevd(extremes, type = GEV_type, location.fun = ~ covariate, ##Fitting the gev with a location and scale parameter linearly correlated to the covariate (years)
               scale.fun = ~ covariate, use.phi = TRUE)

  params_matrix <- make.qcov(fit, vals = list(mu1 = covariate_values,phi1 = covariate_values)) #Create a parameter matrix for the GEV fit
  rvs=ci.fevd(fit,alpha = 0.05,type='return.level',return.period = return_period,method ="normal",qcov=params_matrix)  #Calculate the return values and confidence intervals for each year   
  return(rvs)
}

Plot_non_stationary <- function(GEV_type) {
  
rvs_wc_1981=RV_ci(extremes = extremes_wc,covariate = c(df_WC$Year),return_period = rperiods,covariate_values = 1,GEV_type = GEV_type) ##calc the return values
colnames(rvs_wc_1981) = c('S5_1981_l','S5_1981','S5_1981_h','S5_1981_sd') #Rename the column

rvs_wc_2015=RV_ci(extremes = extremes_wc,covariate = c(df_WC$Year),return_period = rperiods,covariate_values = 35,GEV_type = GEV_type)
colnames(rvs_wc_2015) = c('S5_2015_l','S5_2015','S5_2015_h','S5_2015_sd')

rvs_obs_1981=RV_ci(extremes = obs,covariate = c(1:35),return_period = rperiods,covariate_values = 1,GEV_type = GEV_type)
colnames(rvs_obs_1981) = c('Obs_1981_l','Obs_1981','Obs_1981_h','Obs_1981_sd') #Rename the col

rvs_obs_2015=RV_ci(extremes = obs,covariate = c(1:35),return_period = rperiods,covariate_values = 35,GEV_type = GEV_type)
colnames(rvs_obs_2015) = c('Obs_2015_l','Obs_2015','Obs_2015_h','Obs_2015_sd')

rvs_WC=data.frame(cbind(rvs_wc_1981,rvs_wc_2015,rvs_obs_1981,rvs_obs_2015,rperiods))

# cols=c("S5_1981"="#f04546","S5_2015"="#3591d1","Obs_1981"="#62c76b","Obs_2015"="#62c76b")
p_wc=ggplot(data = rvs_WC,aes(x=rperiods))+
  geom_line(aes(y = S5_1981),col='black')+
  geom_ribbon(aes(ymin=S5_1981_l,ymax=S5_1981_h),fill='black',alpha=0.5,show.legend = T)+
  geom_line(aes(y = S5_2015),col='red')+
  geom_ribbon(aes(ymin=S5_2015_l,ymax=S5_2015_h),fill='red', alpha=0.5,show.legend = T)+
  geom_line(aes(y = Obs_1981),col='black')+
  geom_ribbon(aes(ymin=Obs_1981_l,ymax=Obs_1981_h),fill='black', alpha=0.1,show.legend = T)+
  geom_line(aes(y = Obs_2015),col='red')+
  geom_ribbon(aes(ymin=Obs_2015_l,ymax=Obs_2015_h),fill='red', alpha=0.1,show.legend = T)+
  scale_x_continuous(trans='log10')+
  theme_classic()+
  xlab('Return period (years)')+
  ylab('Three-day precipitation (mm)')

rvs_sv_1981=RV_ci(extremes = df_SV$V1,covariate = c(df_WC$Year),return_period = rperiods,covariate_values = 1,GEV_type = GEV_type) ##calc the return values
colnames(rvs_sv_1981) = c('S5_1981_l','S5_1981','S5_1981_h','S5_1981_sd') #Rename the column

rvs_sv_2015=RV_ci(extremes = df_SV$V1,covariate = c(df_WC$Year),return_period = rperiods,covariate_values = 35,GEV_type = GEV_type)
colnames(rvs_sv_2015) = c('S5_2015_l','S5_2015','S5_2015_h','S5_2015_sd')

rvs_SV=data.frame(cbind(rvs_sv_1981,rvs_sv_2015,rperiods))


cols=c("1981"="black","2015"="red")
p_sv=ggplot(data = rvs_SV,aes(x=rperiods))+
  geom_line(aes(y = S5_1981),col='black')+
  geom_ribbon(aes(ymin=S5_1981_l,ymax=S5_1981_h,fill="1981"),alpha=0.5)+
  geom_line(aes(y = S5_2015,colour="2015"),col='red')+
  geom_ribbon(aes(ymin=S5_2015_l,ymax=S5_2015_h,fill="2015"), alpha=0.5)+
  scale_x_continuous(trans='log10')+
  theme_classic()+
  scale_fill_manual(name="Years",values=cols) +
  theme(axis.title.y=element_blank())+
  xlab('Return period (years)')+
  ylab('Three-day precipitation (mm)')

ggarrange(p_wc, p_sv,
          labels = c("c", "d"),
          legend='top',
          common.legend = T,
          hjust = c(-0.5,1),
          ncol = 2, nrow = 1)
}

CD=Plot_non_stationary(GEV_type = 'GEV')
CD
```

![](Trend_analysis_files/figure-gfm/unnamed-chunk-3-1.png)<!-- -->

## Testing the fits

We test whether the non-stationary fit is significantly different to the
stationary fit. Additionally, we print the percentage change in 100yr
precipitation values between 1981 and 2015. We first do this for the
observed over Norway and then for the UNSEEN ensemble over Norway and
Svalbard.

``` r
####Testing for Non-stationarity in the observed
Obs_GEV_stat <- fevd(x = obs)
Obs_GEV_non_stat <- fevd(obs,
                         type = 'GEV',
                         location.fun = ~ c(1:35),
                         scale.fun = ~ c(1:35), ##GEV with a location and scale parameter linearly correlated to the covariate (years)
                         use.phi = TRUE)

lr.test(Obs_GEV_stat,Obs_GEV_non_stat)
```

    ## 
    ##  Likelihood-ratio Test
    ## 
    ## data:  obsobs
    ## Likelihood-ratio = 1.0865, chi-square critical value = 5.9915, alpha =
    ## 0.0500, Degrees of Freedom = 2.0000, p-value = 0.5809
    ## alternative hypothesis: greater

``` r
####Testing for Non-stationarity over Norway in SEAS5
SEAS5_GEV_stat=fevd(x = extremes_wc,
                    type='GEV') #the bias corrected version
SEAS5_GEV_non_stat=fevd(x = extremes_wc,
                    type='GEV',
                    location.fun = ~ c(df_WC$Year),
                    scale.fun = ~ c(df_WC$Year),
                    use.phi = TRUE) #GEV with a location and scale parameter linearly correlated to the covariate (years)

lr.test(SEAS5_GEV_stat,SEAS5_GEV_non_stat)
```

    ## 
    ##  Likelihood-ratio Test
    ## 
    ## data:  extremes_wcextremes_wc
    ## Likelihood-ratio = 0.86439, chi-square critical value = 5.9915, alpha =
    ## 0.0500, Degrees of Freedom = 2.0000, p-value = 0.6491
    ## alternative hypothesis: greater

``` r
# ###Calculating the uncertainty for the obs
qcovs <- make.qcov(Obs_GEV_non_stat, vals = list(mu1 = c(1,35),phi1 = c(1,35))) #Create a parameter matrix for the GEV fit
rvs_obs=ci(Obs_GEV_non_stat,alpha = 0.05,type='return.level',return.period = 100,method ="normal",qcov=qcovs[2,],qcov.base=qcovs[1,])  #Calculate the return values and confidence intervals for each year
rvs_obs_base=ci(Obs_GEV_non_stat,alpha = 0.05,type='return.level',return.period = 100,method ="normal",qcov=qcovs[1,])  #Calculate the return values and confidence intervals for each
rvs_obs_trend_percent=100*rvs_obs/rvs_obs_base[2]
print(paste('Observed trend and uncertainty:', as.character(rvs_obs_trend_percent[1,2]),
            as.character(rvs_obs_trend_percent[1,1]),as.character(rvs_obs_trend_percent[1,3])))
```

    ## [1] "Observed trend and uncertainty: 3.80112013412126 -26.8666488876624 34.4688891559049"

``` r
rvs_SEAS5=ci(SEAS5_GEV_non_stat,alpha = 0.05,type='return.level',return.period = 100,method ="normal",qcov=qcovs[2,],qcov.base=qcovs[1,])  #Calculate the return values and confidence intervals for each year
rvs_SEAS5_base=ci(SEAS5_GEV_non_stat,alpha = 0.05,type='return.level',return.period = 100,method ="normal",qcov=qcovs[1,])  #Calculate the return values and confidence intervals for each
rvs_SEAS5_trend_percent=100*rvs_SEAS5/rvs_SEAS5_base[2]
print(paste('UNSEEN trend and uncertainty:', as.character(rvs_SEAS5_trend_percent[1,2]),
            as.character(rvs_SEAS5_trend_percent[1,1]),as.character(rvs_SEAS5_trend_percent[1,3])))
```

    ## [1] "UNSEEN trend and uncertainty: 2.26315344771226 -2.62921156503914 7.15551846046367"

``` r
####Testing for Non-stationarity over Svalbard in SEAS5
SEAS5_GEV_stat_SV=fevd(x = df_SV$V1,
                    type='GEV') #the bias corrected version
SEAS5_GEV_non_stat_SV=fevd(x = df_SV$V1,
                    type='GEV',
                    location.fun = ~ c(df_WC$Year),
                    scale.fun = ~ c(df_WC$Year),
                    use.phi = TRUE) #GEV with a location and scale parameter linearly correlated to the covariate (years)

lr.test(SEAS5_GEV_non_stat_SV,SEAS5_GEV_stat_SV)
```

    ## 
    ##  Likelihood-ratio Test
    ## 
    ## data:  df_SV$V1df_SV$V1
    ## Likelihood-ratio = 30.283, chi-square critical value = 5.9915, alpha =
    ## 0.0500, Degrees of Freedom = 2.0000, p-value = 2.655e-07
    ## alternative hypothesis: greater

``` r
##Calculate the uncertianty
rvs_SEAS5_SV_trend_mm=ci(SEAS5_GEV_non_stat_SV,alpha = 0.05,type='return.level',return.period = 100,method ="normal",qcov=qcovs[2,],qcov.base=qcovs[1,])  #Calculate the return values and confidence intervals for each 
rvs_SEAS5_SV_base=ci(SEAS5_GEV_non_stat_SV,alpha = 0.05,type='return.level',return.period = 100,method ="normal",qcov=qcovs[1,])  #Calculate the return values and confidence intervals for each 
rvs_SEAS5_SV_trend_percent=100*rvs_SEAS5_SV_trend_mm/rvs_SEAS5_SV_base[2]
print(paste('UNSEEN trend and uncertainty over Svalbard:', as.character(rvs_SEAS5_SV_trend_percent[1,2]),
            as.character(rvs_SEAS5_SV_trend_percent[1,1]),as.character(rvs_SEAS5_SV_trend_percent[1,3])))
```

    ## [1] "UNSEEN trend and uncertainty over Svalbard: 7.99531929477084 3.66703506685388 12.3236035226878"

## Illustrating the change in 100-yr values in the observed and in the UNSEEN ensemble

Here, we visualize how the observed, with its small sample, needs to
extrapolate the trend in the 100-year values, and thus will result in
very large uncertianty estimates. The UNSEEN ensemble boosts up the
sample size and therefore can better constrain the parameters and better
estimate the trend.

We first do the calculation for Norway

``` r
###Plotting the uncertainty estimation
ci_wc=RV_ci(extremes = df_WC$V1,covariate = c(df_WC$Year),return_period = 100,covariate_values = c(1:35),GEV_type='GEV')
ci_obs=RV_ci(extremes = obs,covariate = c(1:35),return_period = 100,covariate_values = c(1:35),GEV_type='GEV')

## Mean bias correction

ci_wc_biascor=RV_ci(extremes = extremes_wc,covariate = c(df_WC$Year),return_period = 100,covariate_values = c(1:35),GEV_type='GEV')


###Bias corrected series
cols=c("Traditional"="blue","UNSEEN-trends"="orange")
p_trend_wc=
  ggplot()+
  ggtitle("Norway") +
  geom_point(aes(x = year_vector,y = extremes_wc),size=2,alpha=0.051)+
   # scale_size_manual(values = seq(0.1,3,length.out = 3499)) +
  geom_point(aes(x=1981:2015,y =obs),col='blue',shape=4,size=2,stroke=1.5)+
  geom_line(aes(x=1981:2015,y = ci_wc_biascor[,2]),col='orange')+
  geom_ribbon(aes(x=1981:2015,ymin = ci_wc_biascor[,1],ymax=ci_wc_biascor[,3],fill="UNSEEN-trends"),alpha=0.5)+
  geom_line(aes(x=1981:2015,y = ci_obs[,2]),col='blue')+
  geom_ribbon(aes(x=1981:2015,ymin = ci_obs[,1],ymax=ci_obs[,3],fill="Traditional"),alpha=0.2)+
  theme_classic()+
  ylab('Three-day precipitation (mm)')+
  scale_fill_manual(name="Method",values=cols) +
  theme(axis.title.x=element_blank())+
  theme(plot.title = element_text(hjust = 0.5))
```

And for Svalbard

``` r
####Testing for Non-stationarity over Svalbard in SEAS5


## Illustrating the non-stationarity
ci_sv=RV_ci(extremes = df_SV$V1,covariate = c(df_SV$Year),return_period = 100,covariate_values = c(1:35),GEV_type='GEV')

p_trend_sv=
ggplot()+
  ggtitle("Svalbard") +
  geom_point(data = df_SV,aes(x = year_vector,y = V1,size=V1),size=2,alpha=0.051)+
  geom_line(aes(x=1981:2015,y = ci_sv[,2]),col='orange')+
  geom_ribbon(aes(x=1981:2015,ymin = ci_sv[,1],ymax=ci_sv[,3]),fill='orange',alpha=0.5)+
  theme_classic()+
  theme(legend.position = "none",
        axis.title.x=element_blank(),
        axis.title.y=element_blank())+
  theme(plot.title = element_text(hjust = 0.5))


AB=
  ggarrange(p_trend_wc, p_trend_sv,
          labels = c("a", "b"),
          hjust = c(-0.5,1),
          common.legend = T,
          ncol = 2, nrow = 1)

AB
```

![](Trend_analysis_files/figure-gfm/unnamed-chunk-6-1.png)<!-- -->

## Calculate the 2015 return period of the 1981 100-year value

What does the 100-year value in 1981 look like in 2015? How often can we
expect it to occur?

``` r
##The 100 year value in 1981
RV_100yr_1981=rvs_SEAS5_SV_base[2]
###The 30 till 100 year values in 2015
GEV_2015=ci(SEAS5_GEV_non_stat_SV,alpha = 0.05,type='return.level',return.period = seq(30,100),method ="normal",qcov=qcovs[2,])  #Calculate the return values and confidence intervals for each 
GEV_2015[71, 2]/RV_100yr_1981
```

    ## [1] 1.079953

``` r
##Which in the sequence is closest to the 1981 value?
which.min(abs(GEV_2015[,2] - RV_100yr_1981)) #The 41-year value!
```

    ## 41-year return level 
    ##                   12

## Plot for publication

Finally, we aggregate the two plots for submission

``` r
# ABCD=
  ggarrange(AB, CD,
            legend='top',
            common.legend = T,
            ncol = 1, nrow = 2) #%>% 
```

![](Trend_analysis_files/figure-gfm/unnamed-chunk-8-1.png)<!-- -->

``` r
     # ggsave(filename = "../graphs/Trends2.png",width =180,height = 180, units='mm',dpi=300)
```
