---
title: "STA437 Midterm Project"
subtitle: "Latent Structure in Human Activity Recognition Features"
author: "Gaby"
date: "March 14, 2026"
output:
  html_document:
    toc: true
    toc_depth: 2
    number_sections: true
    theme: journal
    highlight: tango
    fig_width: 8
    fig_height: 5.5
  pdf_document:
    toc: true
    toc_depth: 2
    number_sections: true
header-includes:
  - \usepackage{booktabs}
---



# Introduction

This report studies latent structure in the Human Activity Recognition (HAR) dataset, which contains 33 numeric smartphone-derived features summarizing body acceleration, gravity acceleration, gyroscope motion, jerk, and magnitude information across time and frequency domains. Because many of these variables are derived from related physical signals, a lower-dimensional latent structure is plausible.

The analysis follows the workflow required in the assignment: exploratory data analysis, assumption testing, PCA as a baseline, dimension selection via scree plot and parallel analysis, factor analysis with rotation, robustness checks, and an explicit assessment of whether CCA is appropriate. Throughout, the emphasis is on interpreting the latent structure rather than simply listing outputs.

# Data and Preprocessing

We retained only the 33 continuous sensor variables for multivariate analysis. The `activity` variable was kept only as a label for visualization and interpretation, since it is categorical and does not belong in the PCA/FA/ICA measurement matrix. No identifier variables were present.

<table class="table" style="width: auto !important; margin-left: auto; margin-right: auto;">
<caption>Data audit summary</caption>
 <thead>
  <tr>
   <th style="text-align:left;"> Metric </th>
   <th style="text-align:left;"> Value </th>
  </tr>
 </thead>
<tbody>
  <tr>
   <td style="text-align:left;"> Sample size </td>
   <td style="text-align:left;"> 10,299 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> Numeric variables retained </td>
   <td style="text-align:left;"> 33 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> Missing values </td>
   <td style="text-align:left;"> 0 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> Duplicate rows </td>
   <td style="text-align:left;"> 0 </td>
  </tr>
</tbody>
</table>

The dataset contains 10,299 observations and 33 numeric analysis variables. There were no missing values and no duplicate rows, so no imputation or row deletion was required. Even though the original features were already bounded summary statistics, all retained numeric variables were standardized before PCA, FA, ICA, and CCA so that every variable contributed on a comparable scale.

# Exploratory Analysis

Exploratory analysis was used to determine whether a latent-dimension approach was substantively justified. The most important EDA output is the correlation structure, because PCA and FA rely on systematic covariance among variables rather than isolated univariate patterns.

<div class="figure" style="text-align: center">
<img src="../figures/fig2_correlation_heatmap.png" alt="Figure 1. Correlation heatmap of the 33 standardized HAR variables. Clear block structure is visible, with strong within-group dependence among related accelerometer, gyroscope, jerk, and magnitude features." width="92%" />
<p class="caption">Figure 1. Correlation heatmap of the 33 standardized HAR variables. Clear block structure is visible, with strong within-group dependence among related accelerometer, gyroscope, jerk, and magnitude features.</p>
</div>

The heatmap shows clear clustering among related measurements, especially among time-domain magnitude features and their frequency-domain analogues. This is strong evidence that the observed variables are not independent measurements of unrelated phenomena. Instead, they appear to be multiple views of a smaller number of shared movement processes, which motivates latent-structure modeling.

# Assumption Testing and Method Selection

The assignment explicitly requires that assumptions be tested rather than assumed. For this reason, I evaluated multivariate normality, sampling adequacy for FA, whether the correlation matrix differs from an identity matrix, and the extent of multicollinearity.

<table class="table" style="width: auto !important; margin-left: auto; margin-right: auto;">
<caption>Key assumption diagnostics</caption>
 <thead>
  <tr>
   <th style="text-align:left;"> Diagnostic </th>
   <th style="text-align:left;"> Result </th>
  </tr>
 </thead>
<tbody>
  <tr>
   <td style="text-align:left;"> Overall KMO </td>
   <td style="text-align:left;"> 0.500 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> Bartlett's test </td>
   <td style="text-align:left;"> p &lt; 0.001 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> Mardia skewness </td>
   <td style="text-align:left;"> p &lt; 0.001 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> Mardia kurtosis </td>
   <td style="text-align:left;"> p &lt; 0.001 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> Pairs with &amp;#124;r&amp;#124; &gt; 0.95 </td>
   <td style="text-align:left;"> 42 </td>
  </tr>
</tbody>
</table>

<div class="figure" style="text-align: center">
<img src="../figures/fig3_mvn_qqplot.png" alt="Figure 2. Mahalanobis-distance QQ plot for a subsample of the standardized HAR data. The strong departure from the reference line indicates clear non-normality." width="92%" />
<p class="caption">Figure 2. Mahalanobis-distance QQ plot for a subsample of the standardized HAR data. The strong departure from the reference line indicates clear non-normality.</p>
</div>

These diagnostics support FA, but only with some caution. The overall KMO was 0.500, which indicates strong shared variance and therefore supports factor analysis. Bartlett's test strongly rejected the identity correlation matrix (`p < 0.001`), so the correlation structure is far too strong to treat the features as unrelated noise. At the same time, Mardia's skewness and kurtosis tests both rejected multivariate normality (`p < 0.001`), and the correlation matrix contained 42 pairs with absolute correlation above 0.95, including a maximum absolute correlation of 1.000.

This combination is important for method choice. PCA remains a useful baseline because it imposes fewer latent-variable assumptions than FA. FA is still appropriate because of the strong common variance, but strict ML estimation is not ideal under severe non-normality and near-singularity. That is why the final FA models use minimum residual estimation (`minres`) rather than ML. The same non-Gaussianity also makes ICA worth considering as a complementary method.

# PCA Results

PCA was used as the baseline dimension-reduction method because it summarizes covariance structure without requiring a specific latent-factor model. The first component alone explained 53.12% of total variance, and the first six components explained 77.02%. This already suggests a strong low-dimensional structure.

<table class="table" style="width: auto !important; margin-left: auto; margin-right: auto;">
<caption>First six principal components</caption>
 <thead>
  <tr>
   <th style="text-align:left;"> PC </th>
   <th style="text-align:right;"> Eigenvalue </th>
   <th style="text-align:left;"> Variance Explained </th>
   <th style="text-align:left;"> Cumulative Variance </th>
  </tr>
 </thead>
<tbody>
  <tr>
   <td style="text-align:left;"> PC1 </td>
   <td style="text-align:right;"> 17.531 </td>
   <td style="text-align:left;"> 53.12% </td>
   <td style="text-align:left;"> 53.12% </td>
  </tr>
  <tr>
   <td style="text-align:left;"> PC2 </td>
   <td style="text-align:right;"> 1.912 </td>
   <td style="text-align:left;"> 5.79% </td>
   <td style="text-align:left;"> 58.92% </td>
  </tr>
  <tr>
   <td style="text-align:left;"> PC3 </td>
   <td style="text-align:right;"> 1.782 </td>
   <td style="text-align:left;"> 5.40% </td>
   <td style="text-align:left;"> 64.32% </td>
  </tr>
  <tr>
   <td style="text-align:left;"> PC4 </td>
   <td style="text-align:right;"> 1.564 </td>
   <td style="text-align:left;"> 4.74% </td>
   <td style="text-align:left;"> 69.06% </td>
  </tr>
  <tr>
   <td style="text-align:left;"> PC5 </td>
   <td style="text-align:right;"> 1.507 </td>
   <td style="text-align:left;"> 4.57% </td>
   <td style="text-align:left;"> 73.63% </td>
  </tr>
  <tr>
   <td style="text-align:left;"> PC6 </td>
   <td style="text-align:right;"> 1.119 </td>
   <td style="text-align:left;"> 3.39% </td>
   <td style="text-align:left;"> 77.02% </td>
  </tr>
</tbody>
</table>

<div class="figure" style="text-align: center">
<img src="../figures/fig4_pca_scree.png" alt="Figure 3. PCA scree plot and cumulative variance explained. The first component is dominant, followed by a smaller set of additional components." width="92%" />
<p class="caption">Figure 3. PCA scree plot and cumulative variance explained. The first component is dominant, followed by a smaller set of additional components.</p>
</div>

The scree pattern shows a very sharp drop after PC1 and then a more gradual taper. This indicates that one global movement dimension is especially dominant, but it does not imply that a single dimension is sufficient. Instead, the HAR data appear to contain one very strong common component plus several additional dimensions that capture orientation, rotational motion, and domain-specific signal structure.

# Dimension Selection

To choose the dimensionality for FA, I combined the scree plot with Horn's parallel analysis rather than relying on the Kaiser rule alone.

<div class="figure" style="text-align: center">
<img src="../figures/fig7_parallel_analysis.png" alt="Figure 4. Parallel analysis comparing observed eigenvalues with the distribution of random eigenvalues. The observed curve exceeds the 95th-percentile random benchmark for six components." width="92%" />
<p class="caption">Figure 4. Parallel analysis comparing observed eigenvalues with the distribution of random eigenvalues. The observed curve exceeds the 95th-percentile random benchmark for six components.</p>
</div>

Both the Kaiser criterion and parallel analysis retained 6 dimensions, so the main FA solution was estimated with six factors. This was not treated as a purely mechanical decision, however. Because the assignment emphasizes interpretability, the later robustness section also compares nearby factor counts to see whether the six-factor solution is substantively cleaner than a five-factor alternative.

# Factor Analysis Results

Factor analysis was the main latent-variable method in this report. The extraction method was minimum residual (`minres`) because the HAR correlation structure is highly collinear and ML estimation was numerically unstable. I examined the unrotated solution, then used varimax as the primary rotation because the assignment expects it as the default orthogonal rotation. Promax was also fit as an oblique comparison.

<table class="table" style="width: auto !important; margin-left: auto; margin-right: auto;">
<caption>Comparison of six-factor FA solutions</caption>
 <thead>
  <tr>
   <th style="text-align:left;"> Solution </th>
   <th style="text-align:right;"> Simple-Structure Score </th>
   <th style="text-align:right;"> Cross-Loadings </th>
  </tr>
 </thead>
<tbody>
  <tr>
   <td style="text-align:left;"> Unrotated (6) </td>
   <td style="text-align:right;"> 0.576 </td>
   <td style="text-align:right;"> 4 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> Varimax (6) </td>
   <td style="text-align:right;"> 0.906 </td>
   <td style="text-align:right;"> 2 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> Promax (6) </td>
   <td style="text-align:right;"> 0.906 </td>
   <td style="text-align:right;"> 2 </td>
  </tr>
</tbody>
</table>

<div class="figure" style="text-align: center">
<img src="../figures/fig8_fa_varimax_loadings.png" alt="Figure 5. Varimax-rotated factor loadings for the six-factor solution. The rotated solution is much easier to interpret than the unrotated loading pattern." width="92%" />
<p class="caption">Figure 5. Varimax-rotated factor loadings for the six-factor solution. The rotated solution is much easier to interpret than the unrotated loading pattern.</p>
</div>

The rotated solution is substantially clearer than the unrotated one: the simple-structure score increases from 0.576 to 0.906 under varimax, while promax yields a similar score of 0.906. The varimax solution also has only 2 variables with cross-loadings above the chosen cutoff, which is acceptable for a dataset with this degree of multicollinearity. Still, 10 variables had communality below 0.40 and 10 had uniqueness above 0.60, so the factor model should be treated as informative rather than perfect.

The most interpretable factors are:

- `F1`, a jerk or magnitude intensity factor, driven by tBAJMg-m (+0.981), fBBAJMg-m (+0.976), fBAJ-m-X (+0.967), fBAMg-m (+0.966).
- `F2`, a gravity orientation factor, driven by tGravA-m-X (+0.868), tGravA-m-Y (-0.749), tGravA-m-Z (-0.642).
- `F3`, a body gyroscope factor, driven by tBG-m-X (-0.911), tBG-m-Y (+0.785), tBG-m-Z (+0.296).
- `F4`, a body acceleration jerk factor, driven by tBAJ-m-Y (+0.557), tBAJ-m-X (-0.488), tBAJ-m-Z (+0.407).

The later factors are weaker. In particular, the sixth factor is difficult to justify substantively because its largest absolute loading is only 0.258. This matters for the final interpretation: six factors are statistically supported, but the substantive case for all six is not equally strong.

# ICA and Robustness Checks

ICA was considered because the data clearly depart from multivariate normality. The recovered independent components had substantial excess kurtosis, ranging from -1.188 to 40.730, so ICA is not unreasonable for this dataset. However, the ICA solution was harder to interpret in domain terms than the rotated FA solution, which is why FA remains the primary method here.

Robustness checks were performed in three ways. First, nearby factor counts were compared. The five-factor and six-factor solutions had almost identical simple-structure scores (`0.906` in both cases), although the six-factor solution had slightly higher total communality. Second, varimax and promax produced very similar solutions, and the strongest promax inter-factor dependence remained moderate, with the largest absolute factor correlation equal to 0.482. Third, subsample stability was acceptable: across repeated half-sample fits, the mean absolute factor alignment was 0.847 and the mean top-variable Jaccard overlap was 0.429.

These robustness results lead to a balanced conclusion. The overall latent structure is stable, but the difference between five and six factors is mostly about interpretability rather than a dramatic change in structure. The six-factor solution should therefore be reported as the primary solution because it is supported by parallel analysis, while the five-factor solution should be discussed as a plausible robustness-based alternative.

# CCA / Multivariate Assessment

CCA was appropriate for this dataset because the variables admit a meaningful substantive partition into time-domain and frequency-domain blocks. The time-domain block contained 20 original variables and the frequency-domain block contained 13. One time-domain variable, tGravityAccMag-mean(), was removed before CCA because it was linearly dependent on the others.

<table class="table" style="width: auto !important; margin-left: auto; margin-right: auto;">
<caption>First four canonical correlations</caption>
 <thead>
  <tr>
   <th style="text-align:left;"> CanonicalVariate </th>
   <th style="text-align:right;"> Correlation </th>
  </tr>
 </thead>
<tbody>
  <tr>
   <td style="text-align:left;"> CV1 </td>
   <td style="text-align:right;"> 0.9975 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> CV2 </td>
   <td style="text-align:right;"> 0.9373 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> CV3 </td>
   <td style="text-align:right;"> 0.9164 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> CV4 </td>
   <td style="text-align:right;"> 0.6168 </td>
  </tr>
</tbody>
</table>

<div class="figure" style="text-align: center">
<img src="../figures/fig11_cca_first_pair.png" alt="Figure 6. First canonical pair relating time-domain and frequency-domain feature blocks. The extremely high canonical correlation indicates that the two blocks encode strongly overlapping movement structure." width="92%" />
<p class="caption">Figure 6. First canonical pair relating time-domain and frequency-domain feature blocks. The extremely high canonical correlation indicates that the two blocks encode strongly overlapping movement structure.</p>
</div>

The first four canonical correlations were very high, especially the first three. This indicates that the time-domain and frequency-domain feature sets encode strongly overlapping latent structure rather than unrelated information. In contrast, multivariate regression was not pursued because the natural outcome variable in this dataset is `activity`, which is a nominal six-class label rather than a continuous multivariate response.

# Discussion

Taken together, the analyses provide strong evidence that the HAR variables have meaningful latent structure. The correlation heatmap, high KMO, significant Bartlett test, dominant first principal component, and interpretable rotated factors all point in the same direction: the observed features are not independent measurements, but overlapping summaries of a smaller set of underlying movement processes.

The most defensible primary method is FA with varimax rotation. PCA is valuable as a baseline and confirms strong dimension reduction, while ICA is relevant because the data are clearly non-Gaussian. However, FA produces the most interpretable latent dimensions for this assignment because the rotated factors align naturally with domain-relevant signal families such as gravity orientation, gyroscope motion, and jerk or magnitude intensity. CCA is also defensible because the variables split naturally into time- and frequency-domain blocks, and the canonical correlations confirm that these blocks share the same broad structure.

The main limitation is that the features are already engineered summaries of raw signals rather than raw sensor streams themselves. As a result, the latent dimensions identified here should be interpreted as dimensions of the feature space, not necessarily as unique physical mechanisms. In addition, the sixth retained factor is statistically supported but substantively weak, so it should be discussed with caution.

# Conclusion

This project finds clear latent structure in the HAR feature set. PCA shows that a small number of dimensions capture most of the total variance, and FA shows that these dimensions can be interpreted in meaningful sensor-domain terms. Based on the assignment's required workflow, a six-factor varimax solution is the most defensible primary model because it is supported by both the Kaiser rule and parallel analysis. At the same time, the robustness checks show that a five-factor solution may be slightly cleaner substantively, so the report should acknowledge that tradeoff explicitly.

Overall, the HAR data are well suited to multivariate latent-structure analysis. FA is the best primary framework, ICA is a useful complementary check under non-Gaussianity, and CCA is appropriate because the feature set admits a meaningful partition into time- and frequency-domain blocks.

# Reproducibility and AI Use

This report was written as a separate presentation document using precomputed analysis outputs and saved figures from the project pipeline. The main analysis file remains [analysis (5).Rmd](/Users/gabby/Desktop/sta437%20project%20midterm%20project/analysis%20(5).Rmd). AI assistance was used for planning, editing, and interpretation support, but all code and conclusions were reviewed and validated by the author.
