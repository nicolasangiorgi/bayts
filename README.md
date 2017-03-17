# bayts

Propabalistic approach (Reiche et al., 2015, 2017) to combine multiple optical and Radar satellite time series and to detect deforestation. The package includes functions to apply the approach to single pixel time series and to raster time series. Examples for both are provided below.

! The implemented version is a reserach version that simulates .... stepwise, and allows to plot the entire time series.... therefore its performance is weak 

## Probablistic approach 
The basic version of the probablistic approach has been published in Reiche et al., (2015). An improved version was published in Reiche et al. (under review). In Reiche et al., 2017, the probabalistic approach was used for the multi-sensor combination of Radar and optical data from Sentinel-1 and PALSAR-2 HV, together with Landsat for near-real time forest change detection in tropical dry forests in Bolivia. A brief description of the approach is provided below:

Figure 1 gives an schematic overview the probabilistic approach. We considered a near real-time scenario with past (t-1), current (t) and future observations (t+1), with multiple observations possible at the same observation date. First, once a new observation of either of the input time series was available (t = current) it was converted to the conditional NF probability (s<sup>NF</sup>) using the sensor specific forest (F) and non-forest (NF) probability density functions (pdf) (The sensor specific F and NF pdfs were derived using training data).
The derived conditional NF probability was added to the combined time series of conditional NF probabilities derived from the previous LNDVIn, S1VVn and P2HVn time series observations (t–i). Second, we flagged a potential deforestation event in the case that the conditional NF probability was larger than 0.5. We calculated the probability of deforestation using iterative Bayesian updating. Future observation (t+i) were used to update the probability of deforestation in order to confirm or reject the flagged deforestation event.

![fig](method_overview.jpg)
<sub>Figure 1. Probabilistic approach used to combine time series of Landsat NDVI (LNDVIn), Sentinel-1 VV (S1VVn) and ALOS-2 PALSAR-2 HV (P2HVn) observations and to detect deforestation in near real-time. (Reiche et al., under review) </sub>


## Install

The package can be installed directly from github using devtools
```r
library(devtools)
install_github('jreiche/bayts')
```
## Examples 

Two examples are provided ...

### Example data

Example data, dry forest Bolivia, Reiche et al. (under review)
Figure

### Example 1: Single-pixel example (Deforestation)

Single-pixel example using a Sentinel-1 VV and Landsat NDVI time series, covering a deforestation event in 2016.

```r
require(bayts)

##############################################
######### load data, create & plot time series

# load example data
# single pixel Sentinel-1 VV and Landsat NDVI time series (09/2014 - 05/2016); deforestation event in early 2016
data(s1vv_lndvi_pixel)

# create time series using bfastts (bfast package)
ts1vv <- bfastts(s1vv_obs,as.Date(s1vv_date),type=c("irregular"))
tlndvi <- bfastts(lndvi_obs,as.Date(lndvi_date),type=c("irregular"))

# plot time series
plotts(tsL=list(tlndvi,ts1vv),labL=list("Sentinel-1 VV [dB]","Landsat NDVI"))
plotts(tsL=list(tlndvi,ts1vv),labL=list("Sentinel-1 VV [dB]","Landsat NDVI"),ylimL=list(c(0,1),c(-13,-6)))
```
![fig](example1_fig.JPG)
<sub>Figure 2. Landsat NDVI and Sentinel-1 VV time series covering a deforestation event in early 2016.</sub> 

```r
######################################
######### apply bayts and plot results

# (1) Define parameters 
# (1a) Sensor specific pdfs of forest (F) and non-foerst (NF). Used to calculate the conditional NF probability of each observation. Gaussian distribution of F and NF distribution. Distributions are described using mean and sd.
s1vv_pdf <- c(c("gaussian","gaussian"),c(-7,0.75),c(-11.5,1))    
lndvi_pdf <- c(c("gaussian","gaussian"),c(0.85,0.075),c(0.4,0.125))

# (1b) Theshold of deforestation probability at which flagged change is confirmed (chi)
chi = 0.9
# (1c) Start date of monitoring
start = 2015

# (2) apply bayts (combine original time series into a time series of NF probabilities and detect deforestation)
# (2a) apply bayts
bts <- bayts(tsL=list(tlndvi,ts1vv),pdfL=list(lndvi_pdf,s1vv_pdf),chi=chi,start=start)

# (2b) plot original time series; including flagged and detected changes
plotBayts(bts$bayts,labL=list("Landsat NDVI","Sentinel-1 VV [dB]"),ylimL=list(c(0,1),c(-13,-6)),start=start)
```

![fig](example1_fig2.JPG)

<sub> Figure 3. Landsat NDVI and Sentinel-1 VV time series and detected deforestation events. black line = start of monitoring; dotted black line = flagged deforestation event that was not confirmed; red dotted line = flagged deforestation event; red line = confirmed deforestation event.</sub> 

```r
# (2c) plot time series of NF probabilities, including flagged and detected changes
plotBaytsPNF(bts$bayts,start=start)

# (2d) get time of change
bts$change.flagged    # time at which change is flagged
bts$change.confirmed  # time at which change is confirmed
```
![fig](example1_fig3.JPG)
Figure 4. Time series of NF probabilities derived from the two original time series. black line = start of monitoring; dotted black line = flagged deforestation event that was not confirmed; red dotted line = flagged deforestation event; red line = confirmed deforestation event. 

### Example 2: Area example (Deforestation over dry forest)

Include pictures

## References
Reiche, J., de Bruin, S., Hoekman, D. H., Verbesselt, J. & Herold, M. (2015): A Bayesian Approach to Combine Landsat and ALOS PALSAR Time Series for Near Real-Time Deforestation Detection. Remote Sensing, 7, 4973-4996. DOI:10.3390/rs70504973. (http://www.mdpi.com/2072-4292/7/5/4973)

Reiche, J., Hamunyela, E., Verbesselt, J., Hoekman, D. & Herold, M. (under review): Near-real time deforestation detection in tropical dry forest combining Landsat, Sentinel-1 and ALOS-2 PALSAR-2 time series. Remote Sensing of Environment. 

## License
