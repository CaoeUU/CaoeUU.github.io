+++
title = 'Basics of Mixture Cure Model'
date = 2024-02-16T21:12:30+08:00
draft = false
+++

> When it comes to Survival Analysis, numerous bloggers have provided clear introductions. In contrast, content related to cure models is relatively scarce.
<!--more-->
>  It might be considered a subsection within the realm of survival analysis or just an improvement of the Cox model. However, some assumptions of the models might not be apparent, which confused me for a considerable time. Therefore, here's an introduction to the mixture cure model, incorporating my perceptions and questions while learning this model.

Common cure models can be divided into two categories: the **mixture cure model (MCM)** and the **proportional hazard cure model**, proposed by Boag (1949) and Yakovlev et al. (1996) separately. In this passage, we will focus only on the mixture cure model, with the latter possibly introduced later.

### Prerequisite

Firstly, I assume that readers of this introduction already have a foundation in survival analysis. That is, they know basic definitions such as failure event, censored, and survival function. Currently, there's little concern with the hazard function, which can be ignored to simplify understanding.

However, understanding censored data is paramount. Here's a brief introduction to it.

**Censored data** indicates that research subjects' observations end prematurely for reasons not related to the research goal. For example, patients may exit the experiment due to other diseases, transferring to other hospitals, or reaching the experiment's predefined time limit.

In real medical data, censored data is common. The censored data can be categorized into basically three types based on characteristics and reasons, but delving too deeply is unnecessary.

One noteworthy aspect is that the start of observation varies from patient to patient. Not everyone enters the experiment at the same time. Some may start based on age, while others may need to finish specific treatments before timing begins. The start time can differ due to various qualifications.

---

### Main Text
The traditional cure model assumes that everyone will eventually experience the failure event, differing only in survival time. However, due to medical advancements, more diseases now have the possibility of complete cure, weakening the authority of that assumption. Cure models are proposed to help analyze new data generated in this new era.

It's crucial to clarify that **the cure model provides an approximation of the survival function**. While the objectives of cure models extend beyond estimating the survival function, what we gain through the model is a survival function of a patient (given the patient's coefficients). Additionally, this function is similar to cumulative distribution functions. The survival time $T$ of the patient is treated as a random variable, and our aim is to approximate its distribution. The nature of the model can be perceived as estimating the distribution function. However, each point of the survival function represents the time when the patient survived ($T$ is greater than the time), not the counterpart.

#### Basic Hypotheses
The cure model assumes the entire population consists of two types of people: susceptible people and cured people. However, it doesn't mean a person belongs to a type exactly; instead, it is perceived through the lens of probability.

Assume that a person in the dataset belongs to the susceptible type with a specific probability $P$. The probability $P$ is called the **incidence** of the model, which can also be considered as the incidence of the whole population. The survival functions of susceptible people can be estimated by the traditional Cox model, while the rest of the cured people are considered alive all the time, so they wouldn't fail and enjoy the survival function that is identically $1$. And $1-P$ is the cure rate.

#### Formula
According to the hypotheses above, the model of the survival function can be written as:
> $S_{pop}(t|\textbf{X},\textbf{Z})=1-p(\textbf{X})+p(\textbf{X})S_{u}(t|\textbf{Z})$

The reason we use two sets of notes $\textbf{X,Z}$ to represent coefficients is to help distinguish the coefficients into two parts, related to the estimation for susceptible and cured subpopulations separately. It is manifest that they can have the same coefficients; it's only for the clear expression of the model. Not all coefficients are needed in every part during the sequent estimating procedure.

Due to the existence of the cure proportion, as $t$ approaches infinity, the survival function does not tend to 0 but approaches the limit of $1-p(\textbf{X})$, which is the cure rate.

#### Parameters Estimation
The formula above embodies the central idea of the mixture cure model. However, the estimate for each part has to be implemented with data. In the initial article where the model was proposed, the incidence $p$ was considered as a constant, having no connection with coefficients. Gradually, incidence was linked with coefficients, such as using the sigmoid function to approximate the relationship between the coefficients and the incidence by Farewell (1977). The susceptible parts are usually estimated by lognormal distribution, exponential distribution, or Weibull distribution. Later, there are also semi-parameter models and non-parameter models tailored for this model.

---

#### Confusion and Perception
Finally, I want to address my confusion while learning this model. I was quite frustrated that:
> Why do we still use a probability of susceptibility to estimate when the patients in the survival data have gone through the failure event? Why not directly take their survival function as in the Cox model?

> Is the whole population divided into susceptible and unsusceptible parts or just divided among the censored data, considering only those who haven't been observed to fail could likely be cured?

In hindsight, it was because I didn't figure out the logic of the model construction process. The data used for model training compose only a tiny portion of the huge population we want to approximate. For instance, patient data come from only a few hospitals nationwide, but we aim to understand the disease development model of the entire national population. We could clearly know when the patient is censored or failed for the data at hand, but those out of the dataset still need an estimate. If we have a patient randomly chosen from the entire population, having only collected his coefficients, with no clue about whether he would censor or fail, then how can we know his survival function can be modeled by the Cox model or simply equal to 1 and being cured? The answer is no, so we need the probability to estimate.

The formula of the model can actually be understood as a **distribution family**, gaining a specific distribution when the patient's coefficients are substituted in.

Regarding the subpopulation hypothesis, it might be because we have no clue to know how many people are cured or susceptible in the censored portion. So, we assume the entire population consists of the two parts. Also, the mixture cure model deems that the plateau of the K-M curve drawn by the survival data only comprises the cure part. Namely, the patients corresponding to the censored data are all cured.

That concludes our exploration. Thank you for taking the time to read.