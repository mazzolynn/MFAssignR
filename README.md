# MFAssignR

## Package Overview and References

The MFAssignR package was designed for multi-element molecular formula (MF) assignment of ultrahigh resolution mass spectrometry measurements. A number of tools for internal mass recalibration, MF assignment, signal-to-noise evaluation, and unambiguous MF assignments are provided. This package contains MFAssign(), MFAssign_RMD(), MFAssignCHO(), MFAssignCHO_RMD(), MFAssignAll(), MFAssignAll_MSMS(), SNplot(), HistNoise(), KMDNoise(), RecalList(), Recal(), and IsoFiltR() described in the sections below. Note, the functions with “RMD” were designed to be run within an R Markdown file and are otherwise identical to the corresponding non-”RMD” versions. To learn more, please see the section titled “Semi-Automated MFAssignR Functions”.  User caution with the function parameter settings and output evaluation is required; thus, several function outputs are provided to assist the user with these evaluations.

## Molecular Formula (MF) Assignment
The MF assignment algorithm in MFAssign was adapted from the low mass moiety CHOFIT assignment algorithm developed by Green and Perdue (2015). In total there are 4 versions of MF Assign, including MFAssign(), MFAssignCHO(), MFAssignAll(), and MFAssignAll_MSMS(). Where MFAssign(), MFAssignAll(), and MFAssignAll_MSMS() include external nested loops to assign additional heteroatoms, as described in Green and Perdue (2015) while MFAssignCHO() does not. Briefly, the CHOFIT algorithm uses low mass moieties such as CH4O-1 and C4O-3 to move around in the O/C and H/C space to assign MF with C, H, and O (CHO MF). These low mass moieties efficiently assign CHO MF without conventional loops. Additional combinatorial assignments with various heteroatoms are made using nested loops that subtract the mass of a heteroatom from the measured ion mass, creating a CHO “core” mass, which can then be assigned using the low mass moiety CHOFIT approach. This is further explained in Green and Perdue (2015) and Perdue and Green (2015). 

### MFAssign()
Using the low mass moiety and combinatorial assignment approach, MFAssign() can be used to assign MF with 12C, 1H, and 16O and a variety of heteroatoms and isotopes, including 2H, 13C, 14N, 15N, 31P, 32S, 34S, 35Cl, 37Cl,and 19F. It can also assign Na+ adducts, which are common in positive ion mode. Due to the increasing number of chemically reasonable MF with the increasing number of possible elements and increasing molecular weight, the output will provide a list of ambiguous and unambiguous MF. 

Advanced Kendrick mass and z* sorting tools are used to reduce the number of ambiguous MF in MFAssign(). First, Kendrick mass defect (KMD) and z* values are calculated with a CH2 Kendrick base to sort the measured masses into CH2 homologous series (Stenson et al., 2003). The function then selects 1 to 3 members of each CH2 homologous series with masses below the user defined cutoff and attempts to assign MF. The ambiguous MF are then returned to the unassigned list. Then, the unambiguous MF are used as seeds for additional assignments using CH2, O, H2, H2O, and CH2O MF extensions (Kujawinski and Behn, 2006). To do the formula extensions the KMD and z* values for each of these bases are calculated and then used to assign MF through the addition or subtraction of the series bases. MFAssign() (and MFAssignCHO()) tracks how many different “paths” can be used to assign each MF and if a single mass has multiple MF, the function will choose the MF that has the largest number of paths that intercept with it. For example, if a single mass has two possible MF and one has 20 potential “paths” to it, while the other has 4, the function will choose the MF with 20 paths. Work is ongoing to track these paths and the removed MF in the data frame output of these functions. Overall, the multi-path MF extension approach greatly reduces the number of ambiguous assignments and provides an increased level of confidence in the final MF list because the MF are related to unambiguous MF assigned below the user defined cut point. An additional step to decrease the number of ambiguous and/or incorrect sulfur assignments was also added. This step requires that for a sulfur containing compound to act as a seed it must be unambiguous and have a matching 34S peak, when both monoisotopic and isotopic mass lists from the IsoFiltR() function are are assigned MF. This has been implemented for all versions of the MFAssign functions. 

### MFAssignCHO() 
MFAssignCHO() is a simplified version of MFAssign() used only to assign MF with CHO elements. MFAssignCHO() runs faster than MFAssign() and is best used as a preliminary MF assignment step prior to the selection of recalibrant ions in conjunction with MFRecalList() and MFRecalCheck(), which are described below. 

### MFAssignAll() and MFAssignAll_MSMS()
MFAssignAll() uses the low mass moiety and combinatorial assignment approach with a simplified MF extension approach. However, only CH2 and H2O formula extensions are used for MF assignment. This function results in a significantly higher number of ambiguous MF and was intended to be used after MFAssign() or on short mass lists without a complex mixture. MFAssignAll_MSMS() is a further simplified version of MFAssignAll(), which runs somewhat slower, but is more effective for assigning small mass lists with very few homologous series relationships as can be observed in MS/MS data.

## Isotope Filtering
The IsoFiltR() function can identify many of the 13C and 34S isotope masses, which when removed from the mass list can lower the number of peaks assigned with an incorrect MF. This function operates on a two column data frame using the same structure as the MFAssign() function. 

IsoFiltR() identifies potential isotope masses using a four-step identification method. 

1. First the mass list is transformed to identify mass difference pairs appropriate for the element being investigated (delta mass for C (1.003355) or S (1.995797), with +/- 5 ppm mass error). Only those that meet this criteria move on to step 2.

1. Using the mass difference between 12C/13C (1.003355) or 32S/34S (1.995797), the KMD value can be calculated for a specific isotope. This means that the 12C (32S) monoisotopic peak will be in a KMD homologous series with its matching 13C (34S) isotopic peak, analogous to homologous series of CH2. If the KMD values are equivalent for the candidate pair, the peaks can be considered to be in a series and the pair will move on to the third step. The equations for 13C are: KM = 1/1.003355 * m/z, KMD = nominal mass - KM. Replacing 1/1.003355 with 2/1.995797 makes this work for 34S.

1. Isotopic pairs are separated using a “Resolution Enhanced KMD” approach adapted from Zheng  et al. 2019. Resolution enhanced KMD values are calculated by dividing the mass of some homologous series base (in this case CH2) by an integer that was experimentally determined to accomplish the desired separation. This value is then used in the typical KM and KMD calculation in order to calculate the “resolution enhanced” KMD. As an example, BaseMass_adj = 14.01565 / 21 can be considered the integer divided base mass, which is then used in the KM calculation: KMr = (round(BaseMass_adj) / BaseMass_adj) * m/z, followed by KMDr = round(KMr) - KMr to calculate the resolution enhanced KMD. For 13C the integer for this calculation is 21, while for 34S it is 12. After this calculation, peaks that are 12/13 C or 32/34 S pairs will have KMDr difference values of specific values, which can be used to select the pairs that are most likely to be isotope pairs. The KMDr difference is calculated by subtracting the KMDr value of the suspected isotope mass from the KMDr of the suspected monoisotopic mass. The values are -0.291 and 0.709 for 32/34 S and -0.496 and 0.503 for 12/13 C. If the peaks meet these criteria, they can move on to step four. Using CH2 KMD values that are divided by an experimentally derived integer, the isotopic pairs are separated into two specific values. If the difference in the enhanced KMD for the candidate pair matches one of those values, it will move to the fourth tier.

1. The fourth step uses abundance ratios to constrain the remaining isotope pairs to ensure that the isotope peaks are not too large or too small relative to the intensity of the monoisotopic peak. The limits on this are loose due to the variation of isotope abundance with analyte signal (similar to isotope dilution) as observed in ultrahigh resolution Orbitrap and FT-ICR measurements.

The candidate pairs that make it through these four steps are put into two data frames, Mono and Iso, which contain the monoisotopic and isotopic peaks respectively. Then all peaks that were not flagged as possible mono/iso pairs are added to the Mono output data frame. In complex mixtures, some peaks can be flagged as both monoisotopic and isotopic. In these cases, the masses are included in both outputs and are classified as either monoistopic or isotopic after the MF assignment.
  
When the two data frame outputs from IsoFiltR() are put into MFAssign(), the function will match the assigned monoisotopic masses to their corresponding isotopic masses. Additional work would be needed to use the isotopes to reduce ambiguous MF assignments assigned to a single mass. Thus IsoFiltR() should not be considered as definitive proof of the presence or absence of 13C or 34S in MF, but it does assign MF with these expected naturally occurring isotopes and limit the chances that they are incorrectly assigned with a monoisotopic MF.

## Molecular Formula (MF) Quality Assurance
MFAssign() includes a number of quality assurance (QA) steps to check the assigned MF for chemically reasonability. Relatively lenient default settings are provided to avoid removing chemically reasonable ambiguous MF assignments. Many of these parameters are customizable, including  DBE-O limits (Herzsprung et al. Anal. and Bioanal. Chem. 2014), O/C ratio limits, H/C ratio limits, and minimum number of O. The Hetcut parameter can be used to select the MF with the lowest number of heteroatoms, if more than one MF is assigned to a single mass (Ohno and Ohno, 2013). The NMScut parameter identifies the CH4 vs O exchange series in each nominal mass as described in Koch et al. (2007), which can be used to limit ambiguous assignments. Additional non-adjustable QA parameters are used in all of the MFAssign functions, including the nitrogen rule, large atom rule, and the maximum number of H rule, maximum DBE rule (Lobodin et al., 2012), and the Senior rules (Kind et al. 2007).

## Noise Assessment
Noise level assessment can be accomplished using the either the HistNoise() or KMDNoise() functions in conjunction with the SNplot() functions. The HistNoise() method is based on the method developed by Zhurov et al. (2014), and KMDNoise() is a new custom method based on our observations of raw data Kendrick mass defect analysis. 

The Zhurov et al. (2104) method uses a histogram distribution of the natural log intensities in the measured raw mass spectrum to determine the point where noise peaks give way to analyte signal. The HistNoise() function attempts to identify this point and reports the noise level so that the signal-to-noise cut point can be determined. The cut point is shown in an output plot red to blue colors, where red indicates noise. The cut point can also be set manually, if the function does not predict a reasonable noise level. We have observed this function to be confounded by distributions that do not match the theoretical distribution, making it difficult or impossible for the function to identify the correct noise cut point. For this reason, we developed the KMDNoise() function described below.

The KMDNoise() method is based on the observation that the CH2 based KMD values of noise peaks and analyte peaks are naturally separated in a KMD plot, allowing the function to select a region with only noise peaks and use the average intensity of these values to estimate the noise. We refer to this as the KMD slice method. In principle, this is similar to what was briefly described in Reidel and Dittmar (2014), but instead of using a static range of normal mass defects (0.3-0.9), our method uses a mass dependent KMD region, which avoids potentially doubly charged peaks with a mass defect of ~0.5, which would be considered as noise in the Reidel and Dittmar method. 

At least one of these noise estimation functions should be run on the mass list prior to MF assignment with MFAssign() or isotope filtering with IsoFiltR(). Setting a reasonable S/N cut point greatly increases the speed of the functions and improves the output quality.  

The SNplot function is used to show the mass spectrum with the masses below and above the cut point denoted using the same color scheme as in the histogram plots from either HistNoise() or KMDNoise().

## Internal Mass Recalibration
RecalList(), Recal(), and Recal_2() are functions pertaining to the internal mass recalibration method adapted from Kozhinov et al. (2013) and Savory et al. (2011) using a polynomial central moving average to estimate the weights used to recalibrate the masses (Kozhinov et al., 2013) applied to spectral segments (Savory et al., 2011). The function RecalList() can be used with the output of MFAssign() or MFAssignCHO() to generate a data frame containing potential recalibrant CH2 homologous series. There are a variety of metrics included in the output of this function to aid the user in picking suitable recalibrant series, these are described in greater detail in the example of RecalList() below. The user can select up to 10 homologous series as inputs for the mass recalibration with Recal() and Recal_2(). Recal() uses H2 and O KMD and z* series to identify additional MF that are related to the user selected recalibrants. In contrast, Recal_2() does not used those series to expand the pool of potential recalibrants, using only the peaks that correspond to the homologous series chosen as recalibrants. Other than this difference Recal() and Recal_2() work exactly the same. To avoid recalibration problems associated with too many recalibrant masses, the function uses a user-defined number of tallest peaks within a user-defined mass range “bin”. For example, if the bin width is set at 20 and the number of peaks is set at 2, the function will select the two tallest peaks within each 20 m/z window across the range of the spectrum. Additionally, when the monoisotopic peak chosen as a recalibrant has an identified 13C peak, that isotopic peak will also be added to the pool of recalibrants being used. After the recalibrants have been selected, they are split into mass windows of a user defined width (default is 50 m/z) and  used to calculate the correction term according to the the adapted form of the Kozhinov et al. method. This will provide a different mass correction term for each mass window in the spectrum. Then the raw mass list(s) that are being recalibrated are split into the same mass windows, and the correction term that is associated with each window is used to correct the masses in that window, thus recalibrating the full spectrum section by section. In addition to the output of recalibrated mass lists the function also generates a plot that shows the recalibration peaks that were used in context with the overall mass spectrum, and produces an output data frame containing the mass, abundance, formula, and error for the recalibrants that were used. 
 
# Function Examples
## Recommended Order of Operations
The functions will be described in the order that they are most effectively used. The functions do not have to be run in this order, but the best results will likely be obtained in this way. A list of the functions in the recommended order is given below:
1. Run HistNoise() or KMDNoise() to determine the noise level for the data.
 
2. Check effectiveness of S/N cut point using SNplot().
 
3. Use IsoFiltR() to identify potential 13C and 34S isotope peaks.
 
4. Using the S/N cut point, and the two data frames output from IsoFiltR(), run MFAssignCHO() to assign CHO MF to potentially be used as recalibration ions.
 
5. Use RecalList() to generate a list of the potential recalibrant series.
 
6. After choosing a few recalibrant series, use Recal() (or Recal_2()) to check whether they are good recalibrants and recalibrate the mass lists using those recalibrants.
 
7. Use MFAssign() with the recalibrated mass lists to assign MF to the data.
 
8. Check the output plots from MFAssign() to check the quality of the assignment.

The functions in the MFAssignR package were developed by adapting methods and algorithms from the peer reviewed literature. The following references are referred to in this document:

Green, N. W. and Perdue, E. M.: Fast Graphically Inspired Algorithm for Assignment of Molecular Formulae in Ultrahigh Resolution Mass Spectrometry, Anal Chem, 87(10), 5086–5094, doi:10.1021/ac504166t, 2015.

Gross, J. H.: Mass Spectrometry, , doi:10.1007/978-3-319-54398-7 , 2017. 

Herzsprung, P., Hertkorn, N., Tümpling, W. von, Harir, M., Friese, K. and Schmitt-Kopplin, P.: Understanding molecular formula assignment of Fourier transform ion cyclotron resonance mass spectrometry data of natural organic matter from a chemical point of view, Anal Bioanal Chem, 406(30), 7977–7987, doi:10.1007/s00216-014-8249-y, 2014.

Koch, B. P., Dittmar, T., Witt, M. and Kattner, G.: Fundamentals of Molecular Formula Assignment to Ultrahigh Resolution Mass Data of Natural Organic Matter, Anal Chem, 79(4), 1758–1763, doi:10.1021/ac061949s , 2007.

Kozhinov, A. N., Zhurov, K. O. and Tsybin, Y. O.: Iterative Method for Mass Spectra Recalibration via Empirical Estimation of the Mass Calibration Function for Fourier Transform Mass Spectrometry-Based Petroleomics, Anal Chem, 85(13), 6437–6445, doi:10.1021/ac400972y, 2013.

Kujawinski, E. B. and Behn, M. D.: Automated Analysis of Electrospray Ionization Fourier Transform Ion Cyclotron Resonance Mass Spectra of Natural Organic Matter, Anal Chem, 78(13), 4363–4373, doi:10.1021/ac0600306 , 2006.

Lobodin, V. V., Marshall, A. G. and Hsu, C. S.: Compositional Space Boundaries for Organic Compounds, Anal Chem, 84(7), 3410–3416, doi:10.1021/ac300244f, 2012.

Ohno, T. and Ohno, P. E.: Influence of heteroatom pre-selection on the molecular formula assignment of soil organic matter components determined by ultrahigh resolution mass spectrometry, Anal Bioanal Chem, 405(10), 3299–3306, doi:10.1007/s00216-013-6734-3, 2013.

Perdue, E. M. and Green, N. W.: Isobaric Molecular Formulae of C, H, and O: A View from the Negative Quadrants of van Krevelen Space, Anal Chem, 87(10), 5079–5085, doi:10.1021/ac504165k, 2015.

Savory, J. J., Kaiser, N. K., McKenna, A. M., Xian, F., Blakney, G. T., Rodgers, R. P., Hendrickson, C. L., and Marshall, A. G.: Parts-Per-Billion Fourier Transform Ion Cyclotron Resonance Mass Measurement Accuracy with a "Walking" Calibration Equation, Anal Chem, 83, 1732-1736, doi:10.1021/ac102943z, 2011.

Zheng, Q., Morimoto, M., Sato, H. and Fouquet, T.: Resolution-enhanced Kendrick mass defect plots for the data processing of mass spectra from wood and coal hydrothermal extracts, Fuel, 235, 944–953, doi:10.1016/j.fuel.2018.08.085, 2019.

Zhurov, K. O., Kozhinov, A. N., Fornelli, L. and Tsybin, Y. O.: Distinguishing Analyte from Noise Components in Mass Spectra of Complex Samples: Where to Cut the Noise, Anal Chem, 86(7), 3308–3316, doi:10.1021/ac403278t, 2014.  
