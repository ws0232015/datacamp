# GLMs in practice

- [GLMs in practice](#glms-in-practice)
    - [Pre-modeling analysis](#pre-modeling-analysis)
        - [Data required](#data-required)
        - [pre-analyses](#pre-analyses)
    - [model iteration](#model-iteration)
        - [Factor selection](#factor-selection)
        - [stepwise macros](#stepwise-macros)
        - [Model validation](#model-validation)
    - [model refinement](#model-refinement)
        - [Interactions 2.74](#interactions-274)
    - [Interpretation of the results](#interpretation-of-the-results)

## Pre-modeling analysis

### Data required

he overall structure of a dataset for GLM claims analysis consists of linked policy and claims information at the individual risk level.

- Raw explanatory variables - whether discrete or continuous, internal or external to the company.
- Dummy variables to standardize for time-related effects, geographic effects and certain historical underwriting effects.
- **Earned exposure fields** - preferably by claim type if certain claim types are only present for some policies.
- **Number of incurred claims fields.** There should be one field for each claim type, giving the number of claims associated with the exposure period in question.
- **Incurred loss amounts fields.** There should be one field for each claim type, giving the incurred loss amount of claims associated with the exposure period in question, based on the most recent possible case reserve estimates.
- **Premium fields.** These give the premium earned during the period associated with the record. If it is possible to split this premium between the claim types then this can be used to enhance the analysis. This information is not directly required for modeling claims frequency and severity, however it can be helpful for a range of post-modeling analyses such as measuring the impact of moving to a new rating structure.

### pre-analyses 

These analyses include data checks such as identification of records with negative or zero exposures, negative claim counts or losses, and blanks in any of the statistical fields. In addition, certain logical tests may be run against the data - for example,identifying records with incurred losses but with no corresponding claim count.

- Analysis of distributions.
- One and two-way analyses. In addition to investigating exposure and claim distribution, a query of one-way
statistics (eg frequency, severity, loss ratio, pure premium) will give a preliminary
indication of the effect of each factor.
- Factor categorizations.
- correlations analyses. One commonly
used correlation statistic for categorical factors is Cramer's V statistic.
- Data extracts. In practice it is not necessary to fit every model to the entire dataset. For example,
modeling severity for a particular claim type only requires records that contain a claim
of that type. Running models against data subsets, or extracts, can improve model run
speed
- 

## model iteration

typical model forms and the diagnostics used in both factor selection and model validation

### Factor selection

- Standard errors
- Deviance tests
- Interaction with time
- Intuition (common sense)

### stepwise macros

 1. the significance of each factor in the model is tested with a type III test, and the least significant factor is removed from the model if the significance is below a certain specified threshold
 1. the significance of each factor not in the model (but in a specified list of potential factors) is tested by (one at a time) creating a new model containing the factors in the previous step plus the potential new factor. The most significant factor not currently in the model (according to a type III test) is then included if the significance is above the specified threshold
 1. repeated until all factors in the model are deemed significant, and all factors not in the model are deemed insignificant.

### Model validation

- residuals which test the appropriateness of the error term

$$r_i^{DS} = \frac{sign(Y_i - \mu_i)}{\sqrt{\phi(1-h_i)}} \cdot \sqrt{2 \omega_i \int_{\mu_i}^{Y_i} \frac{Y_i - \xi}{V(\xi)} d\xi}$$

Standardized Pearson residual :

$$r_i^{PS} = \frac{(Y_i - \mu_i)}{\sqrt{\frac{\phi}{\omega_i} V(\mu_i) (1-h_i) } }$$

- Leverage which identifies observations which have undue influence on a model

- the Box-Cox transformation which examines the appropriateness of the link function

$$g(x) = \begin{cases}
 \frac{ (x^{\lambda} - 1)}{ln(x)}, & \lambda \ne 0  
\\
 ln(x), & \lambda = 0
 \end{cases}$$


## model refinement

investigating interaction variables, the use of smoothing, and the incorporation of artificial constraints

### Interactions 2.74



## Interpretation of the results

how model results can be compared to existing rating structures both on a factor-by-factor basis and overall.



