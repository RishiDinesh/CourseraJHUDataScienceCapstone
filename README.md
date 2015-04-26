# Capstone Project of the Coursera JHU Data Science Specialization
## Next word prediction using an N-gram Model
## fss14142
## March - April 2015.
  
This repository contains the code I developed for the *Capstone Project*  of the 
[Johns Hopkins Coursera Data Science Specialization](https://www.coursera.org/specialization/jhudatascience/1?utm_medium=listingPage). This project is designed in partnership with [Swiftkey](http://swiftkey.com/).

The main goal of the project is to design a [Shiny](http://shiny.rstudio.com/) application that takes as input a partial (incomplete) English sentence and predicts the next word in the sentence. You may want to start by taking a look at the app. In that case, please remember to read the instructions in the *Documentation* tab of the app before using it. The app can be found at:  
[http://fss14142.shinyapps.io/shinyAppTest](http://fss14142.shinyapps.io/shinyAppTest)

In the last part of this document you will find a brief theoretical description of the model being constructed, an N-gram model with discounted smoothing and Katz backoff. This theoretical part is specially important because I have found in the Web quite a few wrong descriptions of this type of models. The reason for this is probably that the popular book by Jurafsky and Martin, used to teach many courses on this subject, contains a errata in the formula for this model. See the [http://www.cs.colorado.edu/~martin/SLP/Errata/SLP2-PH-Errata.html](official errata list) for the book here.  Therefore I have decided to include correct versions of the formulas for the model in this document.

### Project top level folder structure and information about data files

```
  *
  |
  * - data (initially empty, see below)
  |
  * - predictionFunction
  |
  * - modelConstruction
  |
  * - RStudioPresenter
  |
  * - shinyApp
```

+ **IMPORTANT:** Some of the code files (specifically the model construction code) will download the  required data files when executed. Be careful: some of the data files in this project are quite large. The course organizers provided a zip file  with training data for our models, with size over 570Mb. Besides, the model I constructed generates its own data files for prediction. That part of the data is also not included, but can be downloaded from [this Dropbox link](https://www.dropbox.com/s/qrlxzhol8b8yubi/CapstoneShinyData.zip?dl=0). Please extract the contents of the zip file into the `shinyApp` folder. After extraction, the `data` folder should containing seven data files (with extension `Rds`) whose names and md5sums are:

```
- bigrams.Rds  dbe908579351a85316e88f2c16ef5211
 - codes.Rds 045fba8adefdd9b0497f053390ffac1f
 - fivegrams.Rds 3e6d96da511b724de6db5675c857a09e
 - fourgrams.Rds 1d6d9863cb13889f218f11fedc3aa904
 - removeCodes.Rds 5ad43c7542fa4d8a5c0e5972b7cc9b47
 - trigrams.Rds 1e3792ecf9df533fbf20e727ec053ea1
 - unigrams.Rds 6cf5bb320f2b5d2c045610001245457e
```

+ The prediction function contains a R code file with a stripped down version of my prediction function, suited for benchmarking with e.g. the beutiful code gently [provided by Jan Hagelauer](https://github.com/jan-san/dsci-benchmark) through the forums for the course.

+ The `modelConstruction` folder contains the [Rmarkdown document](http://rmarkdown.rstudio.com/) with the part of the project code aimed at reading and preprocessing (or *cleaning*) the data.

+ The `RStudioPresenter` contains the code for a slide deck presentation describing the Shiny app. This slide deck was required in the course rubric. The resulting pitch for the Shiny app can be seen at [this Rpubs link](http://rpubs.com/fss14142/76542).
 
+ The `shinyApp` folder contains the code for the Shiny App of the project. The app code in this repository is intended to run locally (minor adaptations needed when posting it to a Shiny server).       
       
   
### Formulas for Katz backoff


As I mentioned before, the Katz backoff formulas in many web pages about Natural LAnguage Processing are wrong. These are the corrected formulas I have used for my model:

1. All probabilities for n-grams are computed with a discount smoothing strategy. The particular discounting strategy is not as important as the fact that some probability is left remaining for the unseen n-grams. Let $c(w^j_i)$ and $c^*(w^j_i)$ denote respectively the non-discounted and discounted counts for an N-gram $w^i_j$. Then (see Jurafsky-Martin (2008), Eq. 4.39 in page 140) the non-backed off conditional probabilities for an n-gram are defined as:

\[
P^*(w_n|w^{n-1}_{n-N+1}) = \dfrac{c^*(w^{n}_{n-N+1})}{c(w^{n-1}_{n-N+1})}
\]

2. Besides, the Katz backoff strategy means that a new probability assignment $P_K$ is defined in the case where the n-gram $w^n_1$ has not been seen before. That is, when $c(w^n_1) = 0$. The <font color="red">correct definition</font> is:

\[
P_K(w^n|w^{n-1}_1) = 
\begin{cases}
P^*(w_n|w^{n-1}_{n-N+1}) & \mbox{ if }c(w^n_1) > 0\quad\mbox{(no backoff in this case)}\\[3mm]
\alpha(w^{n-1}_{n-N+1})P_K(w_n|w^{n-1}_{n-N+2})) & \mbox{ else if } c(w^{n-1}_{n-N+1}) >0 \\[3mm]
P_K(w_n|w^{n-1}_{n-N+2})) & \mbox{ otherwise (pure backoff).}
\end{cases}
\]

In the second case we use backoff to an $(n-1)$- gram that we have in fact seen (positive counts) and so we add the (contextual) coefficient $\alpha$ to factor in the knoledge we have about the (n-1)-grams probabilities. That coefficient is defined as follows. First we define:

\[
\beta(w^{n-1}_{n-N+1}) = 1 - \sum_{w_n:c(w^{n}_{n-N+1})>0}P^*(w_n\,|\,w^{n-1}_{n-N+1})
\]
and then
\[
\alpha(w^{n-1}_{n-N+1}) =\dfrac{\beta(w^{n-1}_{n-N+1}) }{ 1 - \sum_{w_n:c(w^{n}_{n-N+1})>0}P^*(w_n\,|\,w^{n-1}_{n-N+2})}
\]
The computation of these $\alpha$ coefficients and the $P^*$ probabilities is the heart of a Katz backoff model.


