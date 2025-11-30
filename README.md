# Nested Logit Demand Model for Theatrical Films

This repository implements a film-level nested logit model to study theatrical demand and cross-platform substitution in the context of the Disney–Fox merger and the rise of streaming platforms.

The model treats each released film as a differentiated product within a genre nest and allows substitution patterns to be more correlated within genres than across genres or with the outside option (non-attendance).

---

## 1. Model Overview

For each film $j$ released in year $t$, consumers choose between:

1. **Outside option**: not attending any film in theatres in year $t$.
2. **Inside options**: attending exactly one theatrical film $j \in J_t$.

Films are grouped into mutually exclusive **genres** $g \in G$ (e.g., Action & Adventure, Comedy, Romance & Drama, etc.). The nested logit structure is:

<p align="center">$\ln(s_{jt}) - \ln(s_{0t}) = X_{jt} \beta  + \sigma \ln(s_{g|t}) + \xi_{jt} + \varepsilon_{jt}$</p>

where:

- $s_{jt}$: market share of film \(j\) in year \(t\)  
- $s_{0t}$: share of the **outside option** (no theatrical attendance)  
- $s_{g|t}$: within-group (genre) share in year \(t\)  
- $X_{jt}$: observed film characteristics  
- $\beta$: parameter vector on observables  
- $\sigma$: **nesting parameter** capturing within-genre correlation in unobserved tastes  
- $\xi_{jt}$: unobserved film-level demand shock  
- $\varepsilon_{jt}$: idiosyncratic error term following a generalized extreme value distribution

Interpretation:

- $\(\sigma = 0\)$: reduces to standard multinomial logit (IIA across all films).  
- $\(0 < \sigma < 1\)$: films within the same genre are closer substitutes than films across genres.

---

## 2. Data Construction

### 2.1 Core Inputs

- **Time frame:** 2000–2024 (annual frequency)  
- **Unit of observation:** Film $\(j\)$ in year $(t\)$  
- **Sources:**
  - Film-level domestic box office and release year ([IMDb](https://www.kaggle.com/datasets/raedaddala/top-500-600-movies-of-each-year-from-1960-to-2024))
    - By Genres, production studios, and awards  
  - Annual total box office revenue and admissions ([Box Office Mojo](https://www.boxofficemojo.com/))
  - Streaming platform data:
    - Netflix: subscribers, number of original movies
    - Disney+: subscribers
    - Hulu: subscribers  
  - Internet access: internet users per 100 people

### 2.2 Tickets and Market Shares

1. **Ticket price per year**

   $p_t = \frac{\text{Total Box Office}_t}{\text{Total Admissions}_t}$

2. **Film-level tickets**

   $q_{jt} = \frac{\text{Domestic Gross}_{jt}}{p_t}$

   Revenues are re-allocated across calendar years using a decay profile (e.g., Einav 2007) to account for films released late in the year.

3. **Genre-level tickets**

   $q_{gt} = \sum_{j \in g} q_{jt}$

4. **Total market size**

   Two alternative **per-capita choice bounds** are used:

   - Lower bound: 6.5 movies per person per year  
   - Upper bound: 12 movies per person per year  

   $M_t = \text{Population}_t \times \text{Choices per capita}$

5. **Market shares**

   $s_{jt} = \frac{q_{jt}}{M_t}, \quad s_{g|t} = \frac{q_{gt}}{M_t}, \quad s_{0t} = 1 - \sum_{j \in J_t} s_{jt} $

A residual “Other titles” category collects unobserved more minor releases to ensure that the inside shares plus the outside share sum to one.

---

## 3. Covariates and Specifications

### 3.1 Baseline Specification

`model_1` (baseline) includes:

- **Genre fixed effects**  
- **Year fixed effects**  
- **Studio indicators:**
  - `Produced_by_Disney`
  - `Produced_by_Fox`
  - `Produced_by_OtherMajor` (Warner, Sony, Universal, Paramount)
- **Quality proxy:**
  - `Award_Winning_Movie` (dummy)
- **Merger effect:**
  - `DisneyFox_Post` (Disney/Fox film × post-2019 indicator)
- **Nesting term:**
  - `ln_s_g_t = log(s_g|t)`


### 3.2 Streaming & Pandemic Specification

`model_2` adds time-varying controls:

- `Num_Theatre_Movies_t`  
- `Num_Netflix_Originals_t`  
- `Netflix_Subscribers_t`  
- `DisneyPlus_Subscribers_t`  
- `Hulu_Subscribers_t`  

Plus interaction terms:

- `DisneyFox_Netflix_Pandemic`  
- `DisneyFox_Hulu_Pandemic`  
- `DisneyFox_DisneyPlus_Pandemic`  
- `OtherMajor_*_Pandemic`  

These capture differential substitution effects during 2020–2024.

### 3.3 No-Year-FE Specification

`model_3` replaces year fixed effects with:

- `Pandemic` dummy (2020–2024)  
- `Internet_Users_per_100`  

to separate long-run co-movement from the pandemic shock.

OLS estimates all models with **standard errors clustered by studio and year**.

---

## 4. Interpreting Key Parameters

- **Nesting parameter \(\sigma\)**  
  - Positive and statistically significant (~0.2) ⇒ films within a genre are closer substitutes than across genres.  
  - Provides direct evidence against full IIA across all titles.

- **Studio coefficients (`Produced_by_*`)**  
  - Exponentiating the coefficients gives the multiplicative change in the odds of choosing a film relative to the outside option.  
  - For example, in the baseline model, coefficients around 3.8–4.0 imply that major studios’ films attract ≈40–55× higher odds than smaller studios, holding other factors constant.

- **Merger term (`DisneyFox_Post`)**  
  - Negative and insignificant ⇒ no strong evidence that the merger increased combined Disney/Fox theatrical demand in the short run.

- **Streaming variables**  
  - Negative coefficients on Netflix and Disney+ subscriber counts ⇒ higher streaming penetration is associated with lower theatrical demand, with stronger substitution from Disney+ for Disney/Fox titles than for other studios.
