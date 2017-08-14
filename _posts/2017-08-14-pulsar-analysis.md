---
layout: post
title: Analyzing Pulsars with the Fermi-LAT
categories: [FermiData]
---

Similar to other LAT sources, pulsars display both temporal and spectral signatures in LAT data. However, the periodicity for gamma-ray pulsars is much faster than the frequency at which photons are detected by the LAT.

Indeed, the Vela pulsar, the brightest persistent source in the LAT sky, produces (on average) one LAT event every 2 minutes. During that time, the pulsar has rotated 1766 times...emitting a radio pulse each time. These radio pulses provide a signal that can be easily used to characterize the pulsar.

While a number of pulsars have been discovered in the LAT gamma-ray data, the analysis required for that discovery is complex and requires significant computing power. Here we will discuss what can be done with pulsars that have already been detected and characterized for timing.

Once the timing solution for a pulsar has been found, it opens the door to analysis in the gamma rays. In order to leverage this knowledge, we first need a timing model that is valid during at least part of the Fermi mission. The easiest place to look is the most recent Pulsar Catalog, which includes all the pulsar timing files that were used in the process of developing the catalog.

## Overview of steps
1. Perform Binned Likelihood analysis to acquire a fitted XML model file. This will take a while, depending on how much data you are analyzing.
2. Apply probabilities to the events and remove those with low probabilities. You can also weight the events by the probabiity.
3. Apply the timing model to the data and look for pulsations

## Products already in hand
> You should already have the following products for PSR J0248+86021, which is our source of interest for this analysis.

* Pulsar ephemeris from the Second Pulsar Catalog
* Filtered and GTI-adjusted event file for a 15-degree ROI centered at 42.0, 60.4
* Spacecraft file covering the entire validity range


## Starting your analysis
As a rule, the E-dot (spindown power) for a pulsar needs to be greater than about 10^33 for it to have a chance of being detected in the gamma rays. Below that threshold, it is speculated, there is not enough energy in the system to spawn the processes that generate gamma rays.

The first step of searching in gamma rays for a pulsar with a known radio ephemeris is to determine if that source is even a pulsed gamma-ray source. To determine that, you should extract the events associated with that pulsar...

### Initial Spectral Analysis
Many energetic young radio pulsars are faint in gamma rays, likely due to geometry...the part of the magnetosphere that produces radio emission only partially overlaps with the part that produces gamma rays. The same is true for bright young gamma-ray pulsars...nearly all the new pulsars discovered in the gamma rays are completely radio quiet. Only 4 of them have measurable radio emission, and one is the faintest radio pulsar ever detected.

However, this is not the case for recycled pulsars. In pulsars with millisecond periods, the two emission zones appear to have much more overlap. To date, only one gamma-ray MSP has been discovered that is completely undetectable in the radio.

Because pulsars are often faint, they can require careful treatment for gamma-ray spectral analysis.

The spectral fit for a faint source will be significantly affected if nearby, brighter sources are not properly modeled. This means you will need to free more parameters in your model, making the spectral fit take longer.

Let's first perform a binned analysis to characterize the sources in our ROI.

In[1]: 

```python
import gt_apps
dir(gt_apps)
```

Out[1]:

```python
['GtApp',
 'TsMap',
 '__builtins__',
 '__doc__',
 '__file__',
 '__name__',
 '__package__',
 'addCubes',
 'counts_map',
 'diffResps',
 'evtbin',
 'expCube',
 'expMap',
 'filter',
 'gtexpcube2',
 'like',
 'maketime',
 'model_map',
 'obsSim',
 'rspgen',
 'srcMaps']
```

In[2]:

```python
from gt_apps import *
```
#### Create a binned counts cube from your data
This will generate the data file that is used in the Binned Likelihood analysis.

In[3]:

```python
%system punlearn gtbin
```

In[4]:

```python
counts_map.pars()
```

In[5]:

```python
' evfile= scfile=NONE outfile= algorithm="PHA2" ebinalg="LOG" emin=30.0 emax=200000.0 ebinfile=NONE tbinalg="LIN" tbinfile=NONE coordsys="CEL" axisrot=0.0 rafield="RA" decfield="DEC" proj="AIT" hpx_ordering_scheme="RING" hpx_order=3 hpx_ebin=yes evtable="EVENTS" sctable="SC_DATA" efield="ENERGY" tfield="TIME" chatter=2 clobber=yes debug=no gui=no mode="ql"'
```

In[6]:

```python
counts_map['evfile'] = '/Users/eferrara/FSSC/VM/shared/psr_data/PSRJ0248_maketime.fits'
counts_map['scfile'] = 'NONE'
counts_map['outfile'] = '/Users/eferrara/FSSC/VM/shared/psr_data/PSRJ0248_ccube.fits'
counts_map['algorithm'] = 'CCUBE'
counts_map['xref'] = 42.0
counts_map['yref'] = 60.4
counts_map['coordsys'] = 'CEL'
counts_map['nxpix'] = 200
counts_map['nypix'] = 200
counts_map['binsz'] = 0.1
counts_map['axisrot'] = 0.0
counts_map['proj'] = 'AIT'
counts_map['emin'] = 300
counts_map['emax'] = 50000
counts_map['enumbins'] = 30
counts_map['ebinalg'] = 'LOG'
counts_map['rafield'] = 'RA'
counts_map['decfield'] = 'DEC'
```

In[7]:

```python
counts_map.run()
```

Out[7]:

```python
time -p /usr/local/ScienceTools/bin/gtbin evfile=/Users/eferrara/FSSC/VM/shared/psr_data/PSRJ0248_maketime.fits scfile=NONE outfile=/Users/eferrara/FSSC/VM/shared/psr_data/PSRJ0248_ccube.fits algorithm="CCUBE" ebinalg="LOG" emin=300.0 emax=50000.0 enumbins=30 ebinfile=NONE tbinalg="LIN" tbinfile=NONE nxpix=200 nypix=200 binsz=0.1 coordsys="CEL" xref=42.0 yref=60.4 axisrot=0.0 rafield="RA" decfield="DEC" proj="AIT" hpx_ordering_scheme="RING" hpx_order=3 hpx_ebin=yes evtable="EVENTS" sctable="SC_DATA" efield="ENERGY" tfield="TIME" chatter=2 clobber=yes debug=no gui=no mode="ql"
This is gtbin version ScienceTools-v10r0p5-fssc-20150819
gtbin: WARNING: No spacecraft file: EXPOSURE keyword will be set equal to ontime.
real 1.80
user 1.58
sys 0.09
```

#### Create a livetime cube (aka exposure cube)
Remember, this is where we should apply the zmax cut, since it was not accounted for in the maketime step.

In[8]:

```python
expCube.pars()
```

Out[8]:

```python
' evfile="" evtable="EVENTS" scfile= sctable="SC_DATA" outfile=expCube.fits dcostheta=0.025 binsz=1.0 phibins=0 tmin=0.0 tmax=0.0 file_version="1" zmin=0.0 zmax=180.0 chatter=2 clobber=yes debug=no gui=no mode="ql"'
```

In[9]:

```python
expCube['evfile'] = '/Users/eferrara/FSSC/VM/shared/psr_data/PSRJ0248_maketime.fits'
expCube['scfile'] = '/Users/eferrara/FSSC/VM/shared/psr_data/spacecraft.fits'
expCube['outfile'] = '/Users/eferrara/FSSC/VM/shared/psr_data/PSRJ0248_ltcube.fits'
expCube['dcostheta'] = 0.025
expCube['binsz'] = 1.0
expCube['zmax'] = 90
```

In[9]:

```python
expCube.run()
```

Out[9]:

```python
time -p /usr/local/ScienceTools/bin/gtltcube evfile="/Users/eferrara/FSSC/VM/shared/psr_data/PSRJ0248_maketime.fits" evtable="EVENTS" scfile=/Users/eferrara/FSSC/VM/shared/psr_data/spacecraft.fits sctable="SC_DATA" outfile=/Users/eferrara/FSSC/VM/shared/psr_data/PSRJ0248_ltcube.fits dcostheta=0.025 binsz=1.0 phibins=0 tmin=0.0 tmax=0.0 file_version="1" zmin=0.0 zmax=90.0 chatter=2 clobber=yes debug=no gui=no mode="ql"
Working on file /Users/eferrara/FSSC/VM/shared/psr_data/spacecraft.fits
.....................!
real 4204.03
user 4193.15
sys 5.15
```

#### What makes up an IRF?
Calculating the exposure cube is the first time you will need to use the Instrument Response Functions (IRFs). You can find details of the full suite of IRFs at the LAT Performance Page

If you install the Fermi Science Tools, the IRFs are also available in the following directory:  
$FERMI_DIR/refdata/fermi/caldb/CALDB/data/glast/lat/bcf

In[10]:

```python
from IPython.display import Image,HTML
HTML("<iframe src='http://www.slac.stanford.edu/exp/glast/groups/canda/lat_Performance.htm' width='1000' height='500'></iframe>")
```

In[11]:

```python
from os import environ
environ['CALDB']
```

Out[11]:

```python
'/usr/local/ScienceTools/refdata/fermi/caldb/CALDB'
```

In[12]:

```python
import pyfits
aeff_P8R2_CLEAN_V6_FB_HDUs = pyfits.open('/usr/local/ScienceTools/refdata/fermi/caldb/CALDB/data/glast/lat/bcf/ea/aeff_P7CLEAN_V6_back.fits')
print aeff_P8R2_CLEAN_V6_FB_HDUs.info()
```

Out[12]:

```python
Filename: /usr/local/ScienceTools/refdata/fermi/caldb/CALDB/data/glast/lat/bcf/ea/aeff_P7CLEAN_V6_back.fits
No.    Name         Type      Cards   Dimensions   Format
0    PRIMARY     PrimaryHDU      10   ()           uint8   
1    EFFECTIVE AREA  BinTableHDU     58   1R x 5C      [60E, 60E, 32E, 32E, 1920E]   
2    PHI_DEPENDENCE  BinTableHDU     61   1R x 6C      [18E, 18E, 8E, 8E, 144E, 144E]   
3    EFFICIENCY_PARAMS  BinTableHDU     38   2R x 1C      [6E]   
None
```

#### Calculate an exposure map cube
You need to understand the exposure even for areas outside your ROI, so you should increase the size of the exposure map well above the size of your counts cube. Often, it's easier to just calculate the exposure map for the entire sky.

You also want to be sure that the geometry for your exposure cube exactly matches that of your counts cube. Otherwise, the mismatch will cause an analysis error.

In[13]:

```python
gtexpcube2.pars()
```

Out[13]:

```python
infile=/Users/eferrara/FSSC/VM/shared/psr_data/PSRJ0248_ltcube.fits cmap=none outfile=/Users/eferrara/FSSC/VM/shared/psr_data/PSRJ0248_expmap.fits irfs="P8R2_SOURCE_V6" evtype="INDEF" nxpix=500 nypix=500 binsz=0.1 coordsys="CEL" xref=42.0 yref=60.4 axisrot=0.0 proj="AIT" ebinalg="LOG" emin=300.0 emax=50000.0 enumbins=30 ebinfile="NONE" bincalc="EDGE" ignorephi=no thmax=180.0 thmin=0.0 table="EXPOSURE" chatter=2 clobber=yes debug=no mode="ql"'
```

In[14]:

```python
gtexpcube2['infile'] = '/Users/eferrara/FSSC/VM/shared/psr_data/PSRJ0248_ltcube.fits'
gtexpcube2['cmap'] = 'none'
gtexpcube2['outfile'] = '/Users/eferrara/FSSC/VM/shared/psr_data/PSRJ0248_expmap.fits'
gtexpcube2['irfs'] = 'P8R2_SOURCE_V6'
gtexpcube2['xref'] = 42.0
gtexpcube2['yref'] = 60.4
gtexpcube2['coordsys'] = 'CEL'
gtexpcube2['nxpix'] = 500
gtexpcube2['nypix'] = 500
gtexpcube2['binsz'] = 0.1
gtexpcube2['axisrot'] = 0.0
gtexpcube2['proj'] = 'AIT'
gtexpcube2['emin'] = 300
gtexpcube2['emax'] = 50000
gtexpcube2['enumbins'] = 30

gtexpcube2.run()
```

Out[14]:

```python
time -p /usr/local/ScienceTools/bin/gtexpcube2 infile=/Users/eferrara/FSSC/VM/shared/psr_data/PSRJ0248_ltcube.fits cmap=none outfile=/Users/eferrara/FSSC/VM/shared/psr_data/PSRJ0248_expmap.fits irfs="P8R2_SOURCE_V6" evtype="INDEF" nxpix=500 nypix=500 binsz=0.1 coordsys="CEL" xref=42.0 yref=60.4 axisrot=0.0 proj="AIT" ebinalg="LOG" emin=300.0 emax=50000.0 enumbins=30 ebinfile="NONE" bincalc="EDGE" ignorephi=no thmax=180.0 thmin=0.0 table="EXPOSURE" chatter=2 clobber=yes debug=no mode="ql"
Computing binned exposure map....................!
real 55.07
user 47.98
sys 1.21
```

#### Generate an XML file for your region

The next step requires an XML model. Your model should reflect all the sources in your region of interest, plus sources that lie in a 10-degree ring around the outside of your ROI. Because sources outside your ROI can contribute photons to your data, they must be considered, even though all their parameters should be fixed.

You will need to download the Galactic and Isotropic diffuse models for Pass 8 and be certain they are included in the model file for your analysis. You can find the template files on the FSSC Background Models page, or in the Fermi Science Tools installation under $FERMI_DIR/refdata/fermi/galdiffuse/

#### Calculate source maps for each source in your XML file
This creates a map for each source which convolves the IRFs with the exposure and the binned geometry. The resulting source maps file contains a plane for each source which is a representation of the data. These are then fitted to your model file in the Likelihood fit.

In[15]:

```python
srcMaps.pars()
```

Out[15]:

```python
' scfile= sctable="SC_DATA" expcube= cmap= srcmdl= bexpmap=none outfile= irfs="CALDB" evtype="INDEF" convol=yes resample=yes rfactor=2 minbinsz=0.1 ptsrc=yes psfcorr=yes emapbnds=yes copyall=no chatter=2 clobber=yes debug=no gui=no mode="ql"'
```

In[16]:

```python
srcMaps['scfile'] = '/Users/eferrara/FSSC/VM/shared/psr_data/spacecraft.fits'
srcMaps['expcube'] = '/Users/eferrara/FSSC/VM/shared/psr_data/PSRJ0248_ltcube.fits'
srcMaps['cmap'] = '/Users/eferrara/FSSC/VM/shared/psr_data/PSRJ0248_ccube.fits'
srcMaps['bexpmap'] = '/Users/eferrara/FSSC/VM/shared/psr_data/PSRJ0248_expmap.fits'
srcMaps['srcmdl'] = '/Users/eferrara/FSSC/VM/shared/psr_data/PSRJ0248_pass8.xml'
srcMaps['outfile'] = '/Users/eferrara/FSSC/VM/shared/psr_data/PSRJ0248_srcmaps.fits'
srcMaps['irfs'] = 'P8R2_SOURCE_V6'

srcMaps.run()
```

Out[16]:

```python
time -p /usr/local/ScienceTools/bin/gtsrcmaps scfile=/Users/eferrara/FSSC/VM/shared/psr_data/spacecraft.fits sctable="SC_DATA" expcube=/Users/eferrara/FSSC/VM/shared/psr_data/PSRJ0248_ltcube.fits cmap=/Users/eferrara/FSSC/VM/shared/psr_data/PSRJ0248_ccube.fits srcmdl=/Users/eferrara/FSSC/VM/shared/psr_data/PSRJ0248_pass8.xml bexpmap=/Users/eferrara/FSSC/VM/shared/psr_data/PSRJ0248_expmap.fits outfile=/Users/eferrara/FSSC/VM/shared/psr_data/PSRJ0248_srcmaps.fits irfs="P8R2_SOURCE_V6" evtype="INDEF" convol=yes resample=yes rfactor=2 minbinsz=0.1 ptsrc=yes psfcorr=yes emapbnds=yes copyall=no chatter=2 clobber=yes debug=no gui=no mode="ql"
Generating SourceMap for PSRJ0205+6449....................!
Generating SourceMap for PSRJ0248+6021....................!
Generating SourceMap for _2FGLJ0109.9+6132....................!
Generating SourceMap for _2FGLJ0110.3+6805....................!
Generating SourceMap for _2FGLJ0128.0+6330....................!
Generating SourceMap for _2FGLJ0131.1+6121....................!
>>> compressed <<<
Generating SourceMap for _2FGLJ0359.1+6003....................!
Generating SourceMap for _2FGLJ0359.5+5410....................!
Generating SourceMap for _2FGLJ0418.9+6636....................!
Generating SourceMap for gll_iem_v06....................!
Generating SourceMap for iso_P8R2_SOURCE_V6_v06....................!
real 536.23
user 505.75
sys 7.64
``` 

#### Check that you have all the input files needed for the Binned Analysis
> You will need the following files:

* Counts cube
* Exposure (Livetime) Cube
* Binned Exposure Map
* XML model file
* Source Maps file

In[17]:

```python
%system ls /Users/eferrara/FSSC/VM/shared/psr_data
```

Out[17]:

```
['L151119044834C9B75A5E69_PH00.fits',
 'L151119044834C9B75A5E69_PH01.fits',
 'L151119044834C9B75A5E69_PH02.fits',
 'L151119044834C9B75A5E69_PH03.fits',
 'L151119044834C9B75A5E69_PH04.fits',
 'L151119044834C9B75A5E69_PH05.fits',
 'L151119044834C9B75A5E69_PH06.fits',
 'L151119044834C9B75A5E69_PH07.fits',
 'L151119044834C9B75A5E69_PH08.fits',
 'L151119044834C9B75A5E69_SC00.fits',
 'PSRJ0248+6021_2PC.par',
 'PSRJ0248_ccube.fits',
 'PSRJ0248_cmap.fits',
 'PSRJ0248_expmap.fits',
 'PSRJ0248_filtered.fits',
 'PSRJ0248_ltcube.fits',
 'PSRJ0248_maketime.fits',
 'PSRJ0248_pass8.xml',
 'PSRJ0248_srcmaps.fits',
 'Pulsar_data.fits',
 'events.txt',
 'p7rep',
 'query.txt',
 'spacecraft.fits']
```
 
### Perform a Binned Likelihood fit
 
In[18]:

```python
import BinnedAnalysis
dir(BinnedAnalysis)
```

Out[18]:

```python
['AnalysisBase',
 'BinnedAnalysis',
 'BinnedObs',
 'SimpleDialog',
 'SourceModel',
 '__builtins__',
 '__doc__',
 '__file__',
 '__name__',
 '__package__',
 '_funcFactory',
 '_null_file',
 '_quotefn',
 'binnedAnalysis',
 'bisect',
 'num',
 'os',
 'pyLike',
 'sys']
```

In[19]:

```python
from BinnedAnalysis import *
help(BinnedObs)
```

Out[19]:

```python
Help on class BinnedObs in module BinnedAnalysis:

class BinnedObs(__builtin__.object)
 |  Methods defined here:
 |  
 |  __getattr__(self, attrname)
 |  
 |  __init__(self, srcMaps=None, expCube=None, binnedExpMap=None, irfs=None, phased_expmap=None)
 |  
 |  __repr__(self)
 |  
 |  state(self, output=<ipykernel.iostream.OutStream object>)
 |  
 |  ----------------------------------------------------------------------
 |  Data descriptors defined here:
 |  
 |  __dict__
 |      dictionary for instance variables (if defined)
 |  
 |  __weakref__
 |      list of weak references to the object (if defined)
```

In[20]:

```python
obs = BinnedObs(binnedExpMap='/Users/eferrara/FSSC/VM/shared/psr_data/PSRJ0248_expmap.fits',
                  expCube='/Users/eferrara/FSSC/VM/shared/psr_data/PSRJ0248_ltcube.fits',
                  srcMaps='/Users/eferrara/FSSC/VM/shared/psr_data/PSRJ0248_srcmaps.fits',
                  irfs='P8R2_SOURCE_V6')
analysis = BinnedAnalysis(obs,srcModel='/Users/eferrara/FSSC/VM/shared/psr_data/PSRJ0248_pass8.xml', optimizer='MINUIT')
likeObj = pyLike.Minuit(analysis.logLike)
analysis.tol = 0.01
analysis.fit(covar=True,optObject=likeObj)
```

#### Check for convergence - Write XML

In[21]:

```python
analysis.writeXml('/Users/eferrara/FSSC/VM/shared/psr_data/P8_PSRJ0248_output_model.xml')
print likeObj.getRetCode()
```

Out[21]:

```
0
```

Fitter convergence gives a return code of zero. With any other value, you may need to try again, using modified inputs. If you don't get convergence easily, you can try:

* Change the optimizer. Sometimes Minuit works, other times NewMinuit. Try both.
* Reduce the tolerance. Increase it iteratively.
* Reduce the number of free parameters.
* Make sure no parameters in your model have moved to the limit values.

In[22]:

```python
analysis.Ts('PSRJ0248+6021')
```
Out[22]:

```python
526.4342426875373
```

In[23]:

```python
analysis.model['PSRJ0248+6021']
```

Out[23]:

```python
PSRJ0248+6021
   Spectrum: PLSuperExpCutoff
5      Prefactor:  2.232e+00  3.048e-01  1.000e-04  1.000e+04 ( 1.000e-11)
6         Index1:  1.683e+00  2.098e-01  0.000e+00  5.000e+00 (-1.000e+00)
7          Scale:  7.493e+02  0.000e+00  3.000e+01  5.000e+05 ( 1.000e+00) fixed
8         Cutoff:  1.552e+03  3.498e+02  1.000e+02  1.000e+05 ( 1.000e+00)
9         Index2:  1.000e+00  0.000e+00  0.000e+00  5.000e+00 ( 1.000e+00) fixed
```

In[24]:

```python
analysis.NpredValue('PSRJ0248+6021')
```

Out[24]:

```python
2466.161944722606
```

In[25]:

```python
print "PSRJ0248+6021 Integral Flux = {} +- {}".format(analysis.flux('PSRJ0248+6021'), analysis.fluxError('PSRJ0248+6021'))
```

Out[25]:

```python
PSRJ0248+6021 Integral Flux = 6.82080787295e-08 +- 1.32715140816e-08
```

There is a scond pulsar in the ROI. How does it compare?

In[26]:

```python
analysis.Ts('PSRJ0205+6449')
```

Out[26]:

```python
1444.7239911054494
```

In[27]:

```python
analysis.model['PSRJ0205+6449']
```

Out[27]:

```python
PSRJ0205+6449
   Spectrum: PLSuperExpCutoff
0      Prefactor:  4.262e+00  4.864e-01  1.000e-04  1.000e+04 ( 1.000e-12)
1         Index1:  2.093e+00  9.725e-02  0.000e+00  5.000e+00 (-1.000e+00)
2          Scale:  1.479e+03  0.000e+00  3.000e+01  5.000e+05 ( 1.000e+00) fixed
3         Cutoff:  5.165e+03  1.409e+03  1.000e+02  1.000e+05 ( 1.000e+00)
4         Index2:  1.000e+00  0.000e+00  0.000e+00  5.000e+00 ( 1.000e+00) fixed
```

In[28]:

```python
analysis.NpredValue('PSRJ0205+6449')
```

Out[28]:

```python
3073.8431241672324
```

In[29]:

```python
print "PSRJ0205+6449 Integral Flux = {} +- {}".format(analysis.flux('PSRJ0205+6449'), analysis.fluxError('PSRJ0205+6449'))
```

Out[29]:

```python
PSRJ0205+6449 Integral Flux = 1.01519862964e-07 +- 1.07688744607e-08
```

#### Iterate as needed
Here I loosened my tolerance to allow the process to run more quickly. However, that's not appropriate for a scientific result. The default tolerance is 0.001, but you may want to go tighter.

When you iterate, you should use the output model from this fit so that you are closer to the (hopefully) correct result. However, you will want to move any parameters that have hit their limits away from the limit value. Unfortunately, there is not an easy way to do this within the pyLikelihood framework. This is an excellent reason to use quickLike instead, which is described in the Analysis Threads.

### Event Probabilities
Once the initial spectral analysis is complete, it is simple to run gtsrcprob. The tool generates a new file with additional columns, one for each source you specify in the sourcelist.txt file. It then determines, for each event, how likely it is for that event to have originated from that source, and records the fractional probability in each column. Each event should have probabilities that add up to one.

Next you can use ftselect to filter out events below a particular probability threshold of your choosing. (Here I used 20%.) While this reduces the statistics for your search, it greatly improves the S/N ratio and greatly improves the chance of detecting significant pulsations.

***BE AWARE!!*** Changing the number of events using a method where the GTIs cannot be corrected, means that the resulting file is not appropriate for measuring the flux or spectral parameters of your source. We are searching for a temporal signature, so this is not a concern for us.

In[30]:

```python
import gt_apps
from gt_apps import *
filter['infile'] = '/Users/eferrara/FSSC/VM/shared/psr_data/PSRJ0248_filtered.fits'
filter['outfile'] = '/Users/eferrara/FSSC/VM/shared/psr_data/PSRJ0248_filtered_3deg.fits'
filter['ra'] = 42.0
filter['dec'] = 60.4
filter['rad'] = 3
filter['tmin'] = 'INDEF'
filter['tmax'] = 'INDEF'
filter['emin'] = 300
filter['emax'] = 50000
filter['zmax'] = 100
filter['evclass'] = 128
filter['evtype'] = 3

filter.run()
```

Out[30]:

```python
time -p /usr/local/ScienceTools/bin/gtselect infile=/Users/eferrara/FSSC/VM/shared/psr_data/PSRJ0248_filtered.fits outfile=/Users/eferrara/FSSC/VM/shared/psr_data/PSRJ0248_filtered_3deg.fits ra=42.0 dec=60.4 rad=3.0 tmin="INDEF" tmax="INDEF" emin=300.0 emax=50000.0 zmin=0.0 zmax=100.0 evclass=128 evclsmin=0 evclsmax=10 evtype=3 convtype=-1 phasemin=0.0 phasemax=1.0 evtable="EVENTS" chatter=2 clobber=yes debug=no gui=no mode="ql"
Done.
real 2.84
user 1.93
sys 0.13
```

In[31]:

```python
maketime['evfile'] = '/Users/eferrara/FSSC/VM/shared/psr_data/PSRJ0248_filtered_3deg.fits'
maketime['outfile'] = '/Users/eferrara/FSSC/VM/shared/psr_data/PSRJ0248_maketime_3deg.fits'
maketime['scfile'] = '/Users/eferrara/FSSC/VM/shared/psr_data/spacecraft.fits'
maketime['filter'] = 'DATA_QUAL>0 && LAT_CONFIG==1'
maketime['apply_filter'] = 'yes'
maketime['roicut'] = 'no'

maketime.run()
```

Out[31]:

```
time -p /usr/local/ScienceTools/bin/gtmktime scfile=/Users/eferrara/FSSC/VM/shared/psr_data/spacecraft.fits sctable="SC_DATA" filter="DATA_QUAL>0 && LAT_CONFIG==1" roicut=no evfile=/Users/eferrara/FSSC/VM/shared/psr_data/PSRJ0248_filtered_3deg.fits evtable="EVENTS" outfile="/Users/eferrara/FSSC/VM/shared/psr_data/PSRJ0248_maketime_3deg.fits" apply_filter=yes overwrite=no header_obstimes=yes tstart=0.0 tstop=0.0 gtifile="default" chatter=2 clobber=yes debug=no gui=no mode="ql"
real 68.62
user 66.60
sys 1.21
```

In[32]:

```python
import pyfits
original_data = pyfits.open('/Users/eferrara/FSSC/VM/shared/psr_data/PSRJ0248_maketime_3deg.fits')
original_data.info()
```

Out[32]:

```
Filename: /Users/eferrara/FSSC/VM/shared/psr_data/PSRJ0248_maketime_3deg.fits
No.    Name         Type      Cards   Dimensions   Format
0    PRIMARY     PrimaryHDU      31   ()           uint8   
1    EVENTS      BinTableHDU    235   90175R x 23C   [E, E, E, E, E, E, E, E, E, D, J, J, I, 3I, 32X, 32X, I, D, E, E, E, E, E]   
2    GTI         BinTableHDU     46   17226R x 2C   [D, D]   
```

In[33]:

```python
%system cp /Users/eferrara/FSSC/VM/shared/psr_data/PSRJ0248_maketime_3deg.fits /Users/eferrara/FSSC/VM/shared/psr_data/PSRJ0248_maketime_3deg.fits.bak
```

Out[33]:

```
[]
```

In[34]:

```python
%system cat /Users/eferrara/FSSC/VM/shared/psr_data/sourcelist.txt
```

Out[34]:

```
['PSRJ0205+6449', 'PSRJ0248+6021', '']
```

#### Calculate diffuse responses
Since gtsrcprob treats each event separately, it uses unbinned likelihood. As a result, each event will need an associated diffuse response for every diffuse component in your model. (This includes extended sources.)

To calculate the diffuse responses, run gtdiffrsp. This will add columns to your events file, so make a copy of the file first, if you're concerned.

In[35]:

```python
%system gtdiffrsp \
  evfile=/Users/eferrara/FSSC/VM/shared/psr_data/PSRJ0248_maketime_3deg.fits \
  scfile=/Users/eferrara/FSSC/VM/shared/psr_data/spacecraft.fits \
  srcmdl=/Users/eferrara/FSSC/VM/shared/psr_data/P8_PSRJ0248_output_model.xml \
  irfs=P8R2_SOURCE_V6

%system gtsrcprob \
  srclist=/Users/eferrara/FSSC/VM/shared/psr_data/sourcelist.txt \
  evfile=/Users/eferrara/FSSC/VM/shared/psr_data/PSRJ0248_maketime_3deg.fits \
  scfile=/Users/eferrara/FSSC/VM/shared/psr_data/spacecraft.fits \
  srcmdl=/Users/eferrara/FSSC/VM/shared/psr_data/P8_PSRJ0248_output_model.xml \
  irfs=P8R2_SOURCE_V6 \
  outfile=/Users/eferrara/FSSC/VM/shared/psr_data/PSRJ0248_prob_3deg.fits
```

Out[35]:

```
['Caught N10Likelihood9ExceptionE at the top level: Event::diffuseResponse: ',
 'Diffuse component p8r2_source_v6__gll_iem_v06 does not have an associated diffuse response.',
 '']
```

Here I have edited the output FITS file (J0248_3deg_prob.fits) because ftselect does not recognize column names containing a "+" symbol. Frustrating! 

Fixing this would have required changing the XML model and re-running the analysis.

In[36]:

```python
%system ftselect \
  /Users/eferrara/FSSC/VM/shared/psr_data/PSRJ0248_prob_3deg.fits \
  /Users/eferrara/FSSC/VM/shared/psr_data/PSRJ0248_downselected_3deg.fits \
  'PSRJ0248 > 0.2' 
```

Out[36]:

```
[]
```

In [37]:

```python
downselected_data = pyfits.open('/Users/eferrara/FSSC/VM/shared/psr_data/PSRJ0248_downselected_3deg.fits')
downselected_data.info()
```

Out[37]:

```
Filename: /home/fermi2014/shared/psr_data/J0248_3deg_z100_downselected.fits
No.    Name         Type      Cards   Dimensions   Format
0    PRIMARY     PrimaryHDU      31   ()           uint8   
1    EVENTS      BinTableHDU    193   2048R x 24C   [E, E, E, E, E, E, E, E, E, D, J, J, I, 3I, J, I, D, E, E, E, E, E, 1E, 1E]   
2    GTI         BinTableHDU     46   21196R x 2C   [D, D]   
```

### Apply the Timing Model
Once you have purified your event list, you can apply the radio timing model to see if there are significant gamma-ray pulsations. This is best performed in tempo2, using the user-contributed tempo2 plugin.

The tempo2 plugin does not provide any pulse search capability. It simply applies the radio timing model to the gamma-ray data, and then plots the results, including the H-test result. The tool also assigns phase values to the events in the LAT data file.

Let's investigate what's available with tempo2:

In[38]:

```python
%system tempo2 -h
```

Out[38]:

```
['This program comes with ABSOLUTELY NO WARRANTY.',
 'This is free software, and you are welcome to redistribute it',
 'under conditions of GPL license.',
 '',
 '',
 'Tempo2 2014.2.1',
 '',
 'examples: tempo2 mytim.tim',
 '          tempo2 -f mypar.par mytim.tim',
 '          tempo2 -gr plk -f mypar.par mytim.tim',
 '',
 'Options: ',
 '',
 '-epoch centre     Centres the PEPOCH in the fit',
 '-f parFile        Selects parameter file',
 '-dcm dcmFile      Data covariance matrix file',
 "-gr name          Uses 'name' plugin for graphical interface",
 '-h                This help',
 '-list             Provides listing of clock corrections and residuals etc.',
 "-output name      Uses 'name' plugin for output format",
 '-pred "args"    Creates a predictive 2D Chebyshev polynomial.',
 '      args = "sitename mjd1 mjd2 freq1 freq2 ntimecoeff nfreqcoeff seg_length (s)"',
 '-polyco "args"  Creates a TEMPO1-style polyco file.',
 '                   args = "mjd1 mjd2 nspan ncoeff maxha sitename freq"',
 '-polyco_file      Specify a leading string for file outputs.',
 '-residuals        Outputs the residuals',
 '-allInfo          Prints out clock, Earth orientation and similar information',
 '-reminder         Saves the command line to T2command.input for future reference.',
 '-norescale        Do not rescale parameter uncertainties by the sqrt(red. chisq)',
 '-displayVersion   Display detailed CVS version number of every file used.',
 '-v                Print verson number.',
 '',
 '',
 'Available plugins',
 "in '/home/fermi2014/AstroSoft/tempo2/plugins'",
 ' - detectGWBnew ',
 ' - photons ',
 ' - fermi ',
 ' - autoSpectralFit ',
 ' - clock ',
 ' - applet ',
 ' - dm ',
 ' - simulDM ',
 ' - cholSpectra ',
 ' - general2 ',
 ' - spectrum ',
 ' - glast ',
 ' - matrix ',
 ' - interpolate ',
 ' - basic ',
 ' - GWbkgrd ',
 ' - GWgeneralanisobkgrd ',
 ' - analyticChol ',
 ' - bary ',
 ' - designmatrix ',
 ' - publish ',
 ' - splk ',
 ' - fixData ',
 ' - simRedNoise ',
 ' - fake ',
 ' - glitch ',
 ' - grTemplate ',
 ' - dmmodel ',
 ' - findCWs ',
 ' - planet ',
 ' - GWgeneralbkgrd ',
 ' - plk ',
 ' - GWanisobkgrd ',
 ' - spectralModel ',
 ' - detectGWB ',
 ' - autoDM ',
 ' - GWdipolebkgrd ',
 ' - plotMany ',
 ' - general ',
 ' - findCW ',
 ' - delays ',
 ' - GWbkgrdfromfile ',
 ' - transform ',
 "in '/home/fermi2014/AstroSoft/tempo2/plugins/'",
 '(none or all hidden)',
 '-----------------']
```
 
In[39]:

```python
%system tempo2 -gr fermi -h
```

Out[39]:

```
['This program comes with ABSOLUTELY NO WARRANTY.',
 'This is free software, and you are welcome to redistribute it',
 'under conditions of GPL license.',
 '',
 'Looking for /home/fermi2014/AstroSoft/tempo2/plugins/fermi_linux-gnu_plug.t2',
 '',
 '------------------------------------------',
 'Output interface:    fermi',
 'Author:              Lucas Guillemot',
 'Updated:             4 October 2013',
 'Version:             5.10',
 '------------------------------------------',
 '',
 '',
 ' TEMPO2 fermi plugin',
 '======================',
 '',
 ' USAGE: ',
 '\t tempo2 -gr fermi -ft1 FT1.fits -ft2 FT2.fits -f par.par',
 '',
 ' Command line options:',
 '\t -grdev XXX/YYY, where XXX/YYY is the graphical device of your choice (e.g. a.ps/cps). Default mode is XW',
 '\t -graph 0: no output graph is drawn',
 '\t -bins N: number of bins of the phase histogram',
 '\t -Hbins N: number of time bins for the H-test vs time plot',
 '\t -output XXX: writes event times and phases in file XXX',
 '\t -pos XXX: puts the satellite (X,Y,Z) positions as a function of time in file XXX',
 '\t -profile XXX: stores the binned light curve in file XXX (plotting mode only)',
 '\t -phase: stores phases in the FT1 by the ones calculated by TEMPO2',
 '\t -ophase: will calculate orbital phases instead of pulse phases. Changes default column name to ORBITAL_PHASE.',
 '\t -col XXX: phases will be stored in column XXX. Default is PULSE_PHASE',
 '\t -tmin XXX: event times smaller than MJD XXX will be skipped.',
 '\t -tmax XXX: event times larger than MJD XXX will be skipped.',
 '\t -bary: write out barycenter arrival times.',
 '\t -barycol XXX: bats will be stored in column XXX. Default is BARY_TIME.',
 '\t -h: this help.',
 '==============================================================================================']
```

The tempo2 command is:

In[40]:

```python
%system tempo2 -gr fermi -ft1 /Users/eferrara/FSSC/VM/shared/psr_data/PSRJ0248_downselected_3deg.fits -ft2 /Users/eferrara/FSSC/VM/shared/psr_data/spacecraft.fits -f /Users/eferrara/FSSC/VM/shared/psr_data/PSRJ0248+6021_2PC.par -phase
```

Out[40]:

```
['This program comes with ABSOLUTELY NO WARRANTY.',
 'This is free software, and you are welcome to redistribute it',
 'under conditions of GPL license.',
 '',
 'Looking for /home/fermi2014/AstroSoft/tempo2/plugins/fermi_linux-gnu_plug.t2',
 '',
 '------------------------------------------',
 'Output interface:    fermi',
 'Author:              Lucas Guillemot',
 'Updated:             4 October 2013',
 'Version:             5.10',
 '------------------------------------------',
 '',
 'First photon date in FT1: 239729409.973118 MET (s)',
 ' Last photon date in FT1: 335136053.646939 MET (s)',
 '',
 'Replacing existing column PULSE_PHASE.',
 '',
 'First START date in FT2:  239557417.494176 MET (s)',
 ' Last START date in FT2:  420595045.600000 MET (s)',
 '',
 'WARNING [CLK3]: no clock corrections available for clock UTC(ncy) for MJD 54546.6',
 'WARNING [CLK4]: Trying assuming UTC = UTC(ncy)',
 'WARNING [CLK9]: ... ok, using stated approximation ',
 'WARNING: duplicated warnings have been suppressed.',
 'WARNING [CLK6]: Proceeding assuming UTC =  UTC(ncy)',
 'Treating events # 1 to 2048... ',
 'Done with J0248+6021'] 
```

In[41]:

```python
Image(filename='images/J0248_tempo2_out.png')
```

Out[41]:

![pulsar tempo]({{ site.url }}/images/posts/2017-08-14-pulsar-tempo.png)

As you can see, the H-test for this source is about 350. Anything over 25 is considered a detection.

In[42]:

```python
Image(filename='images/PSRJ0248+6021_2PC_LC.png')
```

Out[42]:

![pulsar profile]({{ site.url }}/images/posts/2017-08-14-pulsar-profile.png)

In[43]:

```python
%system plist gtpphase
```

Out[44]:

```
['Parameters for /home/fermiuser/pfiles/gtpphase.par',
 '       evfile =                  Event data file name',
 '       scfile =                  Spacecraft data file name',
 '    psrdbfile =                  Pulsar ephemerides database file name',
 '      psrname = ANY              Pulsar name',
 '     ephstyle = DB               How will spin ephemeris be specified?',
 '     ephepoch = 0.               Epoch for the spin ephemeris',
 '   timeformat = FILE             Time format for spin ephemeris epoch',
 '      timesys = FILE             Time system for spin ephemeris epoch',
 '           ra = 0.               Right Ascension to be used for barycenter corrections (degrees)',
 '          dec = 0.               Declination to be used for barycenter corrections (degrees)',
 '         phi0 = 0.               Base value of phase at this epoch',
 '           f0 = 1.               Pulse frequency at the epoch of the spin ephemeris (Hz)',
 '           f1 = 0.               First time derivative of the pulse frequency at the epoch of the spin ephemeris (Hz/s)',
 '           f2 = 0.               Second time derivative of the pulse frequency at the epoch of the spin ephemeris (Hz/s/s)',
 '           p0 = 1.               Pulse period at the epoch of the spin ephemeris (seconds)',
 '           p1 = 0.               First time derivative of the pulse period at the epoch of the spin ephemeris (dimensionless)',
 '           p2 = 0.               Second time derivative of the pulse period at the epoch of the spin ephemeris (Hz)',
 '    (tcorrect = AUTO)            Set of arrival time correction(s) to apply',
 '    (solareph = JPL DE405)       Solar system ephemeris for barycentric correction',
 '(matchsolareph = ALL)             Data entries required to match the solar system ephemeris specified',
 '      (angtol = 1.e-8)           Angular tolerance in sky position for barycentric correction (degrees)',
 '     (evtable = EVENTS)          Table containing event data',
 '   (timefield = TIME)            Name of time field in event file',
 '     (sctable = SC_DATA)         Table containing spacecraft data',
 ' (pphasefield = PULSE_PHASE)     Name of pulse phase field in event data file',
 '(pphaseoffset = 0.)              Arbitrary user-defined offset applied to all phases',
 ' (leapsecfile = DEFAULT)         Name of leap seconds file',
 '(reportephstatus = yes)             Report pulsar ephemeris status which may affect ephemeris computations',
 '     (chatter = 2)               Chattiness of output',
 '     (clobber = yes)             Overwrite existing output files with new output files',
 '       (debug = no)              Debugging mode activated',
 '         (gui = no)              GUI mode activated',
 '        (mode = ql)              Mode of automatic parameters']
```
