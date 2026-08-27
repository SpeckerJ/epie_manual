# Model Interpretation

## Sources of uncertainty

Predictions are, by their very nature, ripe with uncertainty. As George Box famously stated, "*All models are wrong, but some are useful*". This sentiment underscores that models cannot accurately reflect and cover all details of reality, but are still useful for gaining important insights, provided that the model limitations are known. This is especially true for models and predictions concerning the environment due to the complexity involved. 

Predicting environmental concentrations relies on several key assumptions, which inherently introduce a certain level of uncertainty in the assessment. These uncertainties can be classified as fundamental or operational. While fundamental uncertainties are a consequence of the specific model assumptions, operational uncertainties relate to uncertainties in individual input parameters. For example, a fundamental uncertainty of ePiE is that it is assumed that all WWTPs have the same removal efficiency, while the particular value of this removal reflects an operational uncertainty. 

Accordingly, the predictions of the model are heavily influenced by its assumptions and the quality of the input data, which are the foundation of the model. Additional explanation surrounding these model assumptions and input parameters is provided in the following sections to aid users in the proper interpretation and usage of the model results.


## API consumption data
 
Driving the entire modelling process, is the quality of the API consumption data. It directly influences the amount of API expected to end up in the environment. However, the availability for API consumption data can be limited, especially for APIs that are available as over the counter drugs (OTC). Notably, whether APIs are available as OTC drugs or only via prescription, can differ between countries, similar to where pharmaceuticals are sold and which sales data are collected. The PREMIER guidance on *Environmental Risk Assessment of Pharmaceuticals in Surface Water*[^10] provides detailed guidance on gathering API consumption data.
[^10]:Insert link in final version

An additional uncertainty of ePiE is the assumed uniform per capita API usage in a river basin. The assumption simplifies the model, but in reality API usage depends on demographic variables. For example, API usage will vary for regions with an older population as these typically consumer more and a greater variety of APIs than the younger population.



## Excreted API fraction

Another critical factor for ePiE is human metabolism and how much of the administered API is excreted in its unchanged from (f_uf). For APIs, where large fractions are excreted, less precise data have less of an impact on the models results than for APIs where only small fractions are excreted. Exemplary, this is shown in the table below. For two APIs, A and B, with the same consumption, using inaccurate data for the excreted fraction, f_uf, results in a relative error of 5.6 % for API A with a high excretion fraction, compared to a 50 % error for API B with a low excretion fraction. 


  API | A | B |
 |-----|---|---|
 | Consumption (kg/year) | 1000 | 1000 |
 | True Fraction, f_uf (%) | 90 | 10 |
 | Wrong Fraction, f_uf (%) | 85 | 5 |
 | True Load (kg/year) | 900 | 100 |
 | Wrong Load (kg/year) | 850 | 50 |
 | Absolute Error (kg/year) | 50 | 50 |
 | Relative Error (%) | 5.6 | 50 |


## API Physico-chemical parameters

Next to the API consumption and the excreted fraction f_uf, the individual physico-chemical parameter influence the model. These parameters are interdependent and uncertainty in any one of them can propagate through the model. Schematically, this is shown below for the parameters that can be directly changed in ePiE under the "API Properties" tab.

The molecular weight (MW), vapour pressure (Vp), and solubility (S) are used by ePiE to predict the Henry’s law coefficient (K_A,W). The Henry coefficient describes the partitioning of chemicals between the air and water phase. Uncertainty in any of these input parameters directly affects the estimated volatilisation potential. However, as APIs are generally designed to be (moderately) polar, they tend to remain in the water phase. Thus, model predictions will be mostly unaffected by uncertainties in these parameters.

For each API, the fraction of the neutral and ionized forms is calculated and for each form individual partitioning coefficients are used. When these values are not known, these are again modelled. K_OW and pKa predict the Kp,DOC, and K_OW furthermore predicts the K_OC. K_OC in turn is used to derive the Kp_ps, Kp_as, Kp_susp, and the Kp_sd. This shows that K_OW is an important parameter that can have a significant influence on model predictions.  

Accordingly, it is important to be aware of the quality of the input parameters used and if modelled or experimental values are used in the modelling process.

<img src="../img/flowchart_hierarchy.png" alt="Draft Flowchart" style="width: 100%; max-width: 600px; height: auto;" />
<figcaption>Figure X: Dummy caption</figcaption>

## Emission locations & hydrological scenarios

ePiE considers two types of API emissions, i.e. discharges from WWTPs and discharges from the population fraction of an agglomeration not connected to a WWTP. ePiE allocates both types of emissions to the WWTP outlet. Accordingly, this means that river sections upstream of any WWTP outlet, do not receive any API inputs. However, this does not accurately reflect reality. In reality, there can be API discharges upstream of WWTPs, e.g. of solitaire houses in the upper basin.

Moreover, ePiE assumes uniform WWTP treatment performance, overlooking regional differences. Accordingly, the model does not capture the operational variability between WWTPs. As such, removal rates for APIs can differ between individual WWTPs.

Moreover, hydrological fluctuations are also to be expected, which can significantly influence environmental concentrations. During high flow events, dilution will be much higher than during low flow events. Moreover, during high-flow events, the possibility of sewer overflows increases, which can lead to the untreated emissions of the sewer into the environment.  

Additional uncertainty is introduced by the fact that only WWTPs serving at least 2000 PE are covered, the hydrological flow data are based on the years 2000 - 2015, and a spatial resolution of 1x1 km. This means that smaller WWTPs and their discharges are not covered by ePiE and that long-term flow changes are not reflected. 

## Risk Interpretation

Several factors need to be considered when interpreting the calculated risk quotient.
First of all, the quality of the applied risk threshold value is hugely influential. If possible, established environmental quality standards (EQSs) should be used for the assessment.[^4] Such quality standards are legally binding and based on standardised test guidelines. However, in case of an API without such established values using other data sources, or in special cases where the EQS is only based on a limited number of tests, the calculated risk quotient should be interpreted with care. 

[^4]: To add: Recent paper of Moermond et al. 2026

Furthermore, ePiE is timely constrained and calculates only one PEC and accordingly only one RQ. Timely variations in environmental concentration are however to be expected which can be due to changes in API usage, API treatment regime, WWTP treatment train changes or upgrades, or hydrological changes.

In the example below for ibuprofen, using an EQS of 140 ng/L[^1] and an average EU consumption of 8.6 mg per capita in 2019[^2], for an average hydrological flow in the Rhine basin, we can see that risks are generally acceptable with the majority of values <0.1. However, a substantial fraction of 40% lies between 0.1 and 1, and almost 8% between 1 and 10, and a small, but important fraction above 10. These high RQs indicate specific locations or conditions in the river basin where risks for the environment are likely. Moreover, it is important to keep in mind that in reality organisms in the environment are exposed to mixtures of APIs. In these instances, locations with RQ between 0.1 - 1 could still be problematic as all chemicals contribute to the overall effect.


| **API  (ID)** | **Basin (ID)** | **Risk Quotient < 0.1** | **Risk Quotient 0.1 - 1.0** | **Risk Quotient 1.0 - 10** | **Risk Quotient > 10** |
|:--------------:|:-----------------:|:-----------------------:|:---------------------------:|:--------------------------:|:----------------------:|
|   Ibuprofen   |     124863     |          51.98%         |            40.4%            |            7.42%           |          0.2%          |


Such specific cases, with a small fraction exceeding the RQ threshold, should also be considered under the perspective of changing hydrological regimens. Below, the data for ibuprofen are presented, including next to the average flow also the maximum and minimum. It becomes apparent that the fraction of RQs exceeding 1 is increasing substantially under low flow conditions. Selecting the proper scenario for a specific time window is therefore key to deriving appropriate RQs. However, as aforementioned, the underlying hydrological data may not fully capture future scenarios.

[^1]: Proposed EQS in the WFD recast. To do: Add link and double check the proposal
[^2]: Cannata et al. 2024. To do: Add citation

| **API  (ID)** | **Basin (ID)** | Hydrological Flow | **Risk Quotient < 0.1** | **Risk Quotient 0.1 - 1.0** | **Risk Quotient 1.0 - 10** | **Risk Quotient > 10** |
|:-------------:|:--------------:|:-----------------:|:-----------------------:|:---------------------------:|:--------------------------:|:----------------------:|
|   Ibuprofen   |     124863     |      Maximum      |          65.6%          |            32.79%           |            1.55%           |          0.06%         |
|   Ibuprofen   |     124863     |      Average      |          51.98%         |            40.4%            |            7.42%           |          0.2%          |
|   Ibuprofen   |     124863     |      Minimum      |           42%           |            28.43%           |           27.69%           |          1.87%         |
