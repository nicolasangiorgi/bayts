# bayts 

Set of tools to apply the probabilistic approach of Reiche et al. (2015, 2018) to combine multiple optical and/or Radar satellite time series and to detect deforestation/forest cover loss in near real-time. The package includes functions to apply the approach to both, single pixel time series and raster time series. Examples and test data are provided below.

### Research version
The package includes the research version of the tools, which have a limited perforamce when applied to raster time series. The reserach version allows (i) to visualise and analyse the entire time series history and (ii) it stepwise applies the probablistic approach consecutively on each observation in the time series to emulate a near real-time scenario. It includes methods used in Reiche et al., 2015 (RSE), Reiche et al., 2018 (RSE) and Reiche et al., 2018 (Remote Sensing)

## Probablistic approach 
The basic version of the probabilistic approach has been published in Reiche et al., (2015). An improved version was published in Reiche et al. (2018, RSE) and was used in Reiche et al., 2018 (Remote Sensing). 

Figure 1 gives an schematic overview the probabilistic approach. We considered a near real-time scenario with past (t-1), current (t) and future observations (t+1), with multiple observations possible at the same observation date. First, once a new observation of either of the input time series was available (t = current) it was converted to the conditional NF probability (s<sup>NF</sup>) using the sensor specific forest (F) and non-forest (NF) probability density functions (pdf) (The sensor specific F and NF pdfs were derived using training data).
The derived conditional NF probability was added to the combined time series of conditional NF probabilities derived from the previous LNDVIn, S1VVn and P2HVn time series observations (t–i). Second, we flagged a potential deforestation event in the case that the conditional NF probability was larger than 0.5. We calculated the probability of deforestation using iterative Bayesian updating. Future observation (t+i) were used to update the probability of deforestation in order to confirm or reject the flagged deforestation event.

![fig](/examples/method_overview.jpg)
<sub>Figure 1. Probabilistic approach used to combine time series of Landsat NDVI (LNDVIn), Sentinel-1 VV (S1VVn) and ALOS-2 PALSAR-2 HV (P2HVn) observations and to detect deforestation in near real-time. (Reiche et al., under review) </sub>

## Core functions

bayts - applies probabalistic approach to single pixel time series (method presented in Reiche et al., 2018, RSE)

baytsSpatial - applies bayts to raster time series

baytsDD - bayts with a priori (i) seasonal-trend model fitting to remove forest seasonality and (ii) data-driven way to derive forest and non-forest distributions (method presented in Reiche et al., 2018, Remote Sensing)

baytsDDSpatial - applies baytsDD to raster time series

## Install

The package can be installed directly from github using devtools
```r
library(devtools)
install_github('jreiche/bayts')
```

## References

Reiche, J., Verhoeven, R.; Verbesselt, J.; Hamunyela, E.; Wielaard, N. & Herold, M. (2018) Characterizing Tropical Forest Cover Loss Using Dense Sentinel-1 Data and Active Fire Alerts. Remote Sensing, 10, 5, 777, doi:10.3390/RS10050777. (http://www.mdpi.com/2072-4292/10/5/777)

Reiche, J., Hamunyela, E., Verbesselt, J., Hoekman, D. & Herold, M. (2018): Improving near-real time deforestation monitoring in tropical dry forests by combining dense Sentinel-1 time series with Landsat and ALOS-2 PALSAR-2. Remote Sensing of Environment. https://doi.org/10.1016/j.rse.2017.10.034. (https://www.sciencedirect.com/science/article/pii/S0034425717304959)

Reiche, J., de Bruin, S., Hoekman, D. H., Verbesselt, J. & Herold, M. (2015): A Bayesian Approach to Combine Landsat and ALOS PALSAR Time Series for Near Real-Time Deforestation Detection. Remote Sensing, 7, 4973-4996. DOI:10.3390/rs70504973. (http://www.mdpi.com/2072-4292/7/5/4973)

Hamunyela, E., Verbesselt, J., & Herold, M. (2016). Using spatial context to improve early detection of deforestation from Landsat time series. Remote Sensing of Environment, 172, 126–138. http://doi.org/10.1016/j.rse.2015.11.006

## Citation

```r
@software{bayts,
  author = {Reiche, Johannes},
  title = {{bayts}},
  url = {https://github.com/jreiche/bayts},
  version = {1.0},
  date = {2017-04-12},
  doi = {10.5281/zenodo.545792}
}
```

## For external contributors

External contributions are welcome. If you would like to contribute additional features and improvements to the package; fork the repository on gitHub, commit your changes and make a pull request. Always use the develop branch as a starting point for your work. Your contribution will be reviewed for quality, relevance and consistency with the rest of the package before being merged.


# Examples 

Two examples are provided. Example 1 shows how to apply the functions on singel-pixel time series (Sentinel-1 VV and Landsat NDVI). Example 2 shows how to apply the functions to raster time series.

## Example 1: Single-pixel example of the core bayts function

Single-pixel example using a Sentinel-1 VV and Landsat NDVI time series, covering a deforestation event in 2016. (Source code: examples/bayts_pixel_example_v01.R)

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
![fig](/examples/example1_fig1.JPG)<br />
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

![fig](/examples/example1_fig2.JPG)<br />
<sub> Figure 3. Landsat NDVI and Sentinel-1 VV time series and detected deforestation events. black line = start of monitoring; dotted black line = flagged deforestation event that was not confirmed; red dotted line = flagged deforestation event; red line = confirmed deforestation event.</sub> 

```r
# (2c) plot time series of NF probabilities, including flagged and detected changes
plotBaytsPNF(bts$bayts,start=start)
```

![fig](/examples/example1_fig3.JPG)<br />
<sub>Figure 4. Time series of NF probabilities derived from the two original time series. black line = start of monitoring; dotted black line = flagged deforestation event that was not confirmed; red dotted line = flagged deforestation event; red line = confirmed deforestation event. </sub> 

```r
# (2d) get time of change
bts$change.flagged    # time at which change is flagged
bts$change.confirmed  # time at which change is confirmed
```


## Example 2: Applying deseasonalizeRaster and baytsSpatial function 
Example applying method presented in Reiche et al., 2018 (RSE)

SOURCE CODE: examples/bayts_raster_example_v01.R

DATA:   Landsat NDVI and Sentinel-1 raster time series data from a dry forest area in Bolivia 

STEP 1: Spatial normalisation (deseasonalizeRaster) to remove dry forest seasonality in Landsat NDVI and Sentinel-1 time series observations             

STEP 2: Probablistic approach (bayts, baytsSpatial) used to combine Landsat and Sentinel-1 time series and to detect forest cover loss

## Example 3: Applying baytsDD and baytsSpatialDD function 
Example applying method presented in Reiche et al., 2018 (Remote Sensing)

SOURCE CODE: examples/baytsDD_raster_example_v01.R

DATA:   Sentinel-1 raster time series data from Riau, Sumatra

Step 1: Use season-trend fitting to model and remove forest seasonality

Step 2: Data-driven way to derive F and NF distribution (pdfs) to paramterise bayts 

Step 3: Probablistic approach (bayts, baytsSpatial) used to combine optical and SAR time series and to detect change in near real-time   
