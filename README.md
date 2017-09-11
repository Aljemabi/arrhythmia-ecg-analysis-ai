# Arrhythmia Classification through Characteristics Extraction with Discrete Wavelet Transform & Supervised Training

This work covers cardiac arrhythmia classification through extraction of heart waves characteristics using discrete wavelet transform to filter the signal and machine learning supervised training to classify the exported characteristics with labels.

The goal is to classify at least two arrhythmia through some extracted characteristics with Weka and MATLAB.

# SOFTWARES IN USE

1) [MATLAB R2015b](https://www.mathworks.com/products/new_products/release2015b.html)

2) [WEKA 3](http://www.cs.waikato.ac.nz/ml/weka/documentation.html)

# LIBRARIES IN USE

1) [MIT-BIH Arrhythmia Database - PhysioBank ATM](https://physionet.org/cgi-bin/atm/ATM)

This directory contains the entire MIT-BIH Arrhythmia Database. About half (25 of 48 complete records, and reference annotation files for all 48 records) of this database has been freely available here since PhysioNet's inception in September 1999. The 23 remaining signal files, which had been available only on the MIT-BIH Arrhythmia Database CD-ROM, were posted here in February 2005.

The recordings were digitized at 360 samples per second per channel with 11-bit resolution over a 10 mV range. Two or more cardiologists independently annotated each record; disagreements were resolved to obtain the computer-readable reference annotations for each beat (approximately 110,000 annotations in all) included with the database.

2) [R Wave Detection with Wavelet Toolbox](https://www.mathworks.com/examples/wavelet/mw/wavelet-ex77408607-r-wave-detection-in-the-ecg)

This example shows how to use wavelets to analyze electrocardiogram (ECG) signals. ECG signals are frequently nonstationary meaning that their frequency content changes over time. These changes are the events of interest.

Wavelets decompose signals into time-varying frequency (scale) components. Because signal features are often localized in time and frequency, analysis and estimation are easier when working with sparser (reduced) representations.

The QRS complex consists of three deflections in the ECG waveform. The QRS complex reflects the depolarization of the right and left ventricles and is the most prominent feature of the human ECG.

3) [plotATM](https://physionet.org/physiotools/matlab/plotATM.m)

This function reads a pair of files (RECORDm.mat and RECORDm.info) generated by 'wfdb2mat' from a PhysioBank record, baseline-corrects and scales the time series contained in the .mat file, and plots them.  The baseline-corrected and scaled time series are the rows of matrix 'val', and each column contains simultaneous samples of each time series.

# DIGITAL SIGNAL PROCESSING STEPS

There are two MATLAB functions to extract arrhythmia heart waves characteristics: single QRS wave (singleExampleWithDWTsignalPeaksExtraction.m) or multiples QRS waves (extractExampleFeaturesFromEcg). The first one it's necessary to insert the period or time that the features need to be extracted, while the second one multiples arrhythmia features are extracted from QRS waves labels.

## Single Extraction Example With DWT signal Peaks (singleExampleWithDWTsignalPeaksExtraction.m)

To test a single extraction, run the following command on MATLAB:

```
[tmSeg,ecgsig,Fs,sizeEcgSig,timeEcgSig,annotationsEcg,qrsExtracted,tmExtracted,ecgsigTransf,qrsPeaks,locs] = singleExampleWithDWTsignalPeaksExtraction('200m', '../data/200m', 'VT', 0, 7.517, 2706, 0.5, 0.150);
```
This command will follow these steps:

1) Load ECG signal from MIT-BIH database file, extracting signal time vector, signal vector, signal frequency, signal samples size and signal time size (in seconds):

```
[tmSeg,ecgsig,Fs,sizeEcgSig,timeEcgSig] = loadEcgSignal(filepath);
```

2) Load ECG signal professional annotations, receiving in an object the time, period and arrhythmia types for each instant:

```
annotationsEcg = readAnnotations(filepath);
```

3) Extract the QRS wave window in a signal and time vectors, plotting them in the end:

```
[qrsExtracted, tmExtracted] = plotExtractSingleQRS(minute, seconds, period, sizeEcgSig, timeEcgSig, ecgsig, tmSeg, filename, arrhythmiaType);
```

4) Decompose the windowed-signal into time-varying frequency (scale) components with MODWT (Maximal overlap discrete wavelet transform) and IMODWT (Inverse Maximal overlap discrete wavelet transform) in the chosen scale:

```
ecgsigTransf = dwtSignal(qrsExtracted, scale);
```

5) Extract and plot the peaks amplitude and locations (feature characteristics) on the windowed-signal filtered after the last step:

```
[qrsPeaks,locs] = plotDWTsignalPeaks(ecgsigTransf, tmExtracted, minPeakHeight, minPeakDistance);
```

## Multiple Extractions Example With DWT signal Peaks (extractExampleFeaturesFromEcg.m)

To test a multiple features extraction, run the following command on MATLAB:

```
features = extractExampleFeaturesFromEcg('200m', '../data/200m', 'VT', '../data/exported/vt-200m');
```
This command will follow these steps:

1) Load ECG signal from MIT-BIH database file, extracting signal time vector, signal vector, signal frequency, signal samples size and signal time size (in seconds):

```
[tmSeg,ecgsig,Fs,sizeEcgSig,timeEcgSig] = loadEcgSignal(filepath);
```

2) Read the arrhythmia periods of the chosen arrhythmia type from the professional annotations file. This will result in an object with the time, period and arrhythmia types for each instant:

```
arrhythmiaPeriods = readArrythmiaPeriods(type, filepath);
```

3) Extract the QRS wave windows in signal and time vectors inside a arrhythmiaMultipleQRS object. This command will also plot and save in /matlab folder a PNG image for each plot - for any arrhythmias cases except Normal Sinus Rhythm (N):

```
arrhythmiaMultipleQRS = extractMultipleQRS(arrhythmiaPeriods, sizeEcgSig, timeEcgSig, ecgsig, tmSeg, filename, type);
```

![Extracted QRS window for Trigeminy Ventricular arrhythmia sample](https://raw.githubusercontent.com/davikawasaki/arrhythmia-ecg-analysis-ai/master/Code/graphs/201m/T-example1-201m.png)

4) Decompose the windowed-signal into time-varying frequency (scale) components with MODWT and IMODWT and extract the signal peaks amplitude and locations (feature characteristics). This command will also plot and save in /matlab folder a PNG image for each plot - for any arrhythmias cases except Normal Sinus Rhythm (N):

```
DWTsignalPeaks = extractDWTsignalPeaks(arrhythmiaMultipleQRS, 0.5, 0.150, filename, type);
```

![Extracted signal peaks for Trigeminy Ventricular arrhythmia sample transformed with DWT](https://raw.githubusercontent.com/davikawasaki/arrhythmia-ecg-analysis-ai/master/Code/graphs/201m/T-peaks1-201m.png)

5) Lastly, extract the ECG features from the DWTsignalPeaks to a CSV or to another variable:

```
features = extractEcgFeatures(DWTsignalPeaks, exportFilename);
```

# MACHINE LEARNING STEPS

In development.

# AUTHORS

This work is being developed to AI undergrad-subject last project. The people involved in the project were:

Student: KAWASAKI, Davi // davishinjik [at] gmail.com

Student: FLAUSINO, Matheus // matheus.negocio [at] gmail.com

Professor: SAITO, Priscila Tiemi Maeda // psaito [at] utfpr.edu.br

# CONTACT & FEEDBACKS

Feel free to contact or pull request me to any relevant updates you may enquire:

KAWASAKI, Davi // davishinjik [at] gmail.com
