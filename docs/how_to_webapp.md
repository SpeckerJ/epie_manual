## Accessing the webapp

The ePiE app can be directly assessed via the button below:

[ePiE - Web Application](https://shoeks.github.io/ePiE_webApp//){target="_blank", .md-button }
<br>


## Legal Disclaimer

After accessing the website, a legal disclaimer and a description of the privacy policy is presented. To use the web application, you have to accept the terms.

<br>
<img src="../img/screenshots/ePiE/screens_1.png" alt="img1" style="width: 100%; max-width: 600px; height: 50%;" />
<figcaption>Figure X: Dummy caption.</figcaption>
<br>

After accepting, you will be guided to the ePiE starting page, which gives you a short description of the model, first use instructions, where to ask questions, and a short section with acknowledgments. On the left side are multiple links to subsections of the model. Each subsection on the webpage comes with specific instructions and tables aiding the user in the process. Start by clicking on “API properties” (1).


<img src="../img/screenshots/ePiE/screens_2.png" alt="img2" style="width: 100%; max-width: 600px; height: 50%;" />
<figcaption>Figure X: Dummy caption.</figcaption>


## API Properties

On the “API properties” page, you will see two tables, i.e. one with the API properties (1) and API-specific fate parameters (2). By hovering over the columns, a pop-up will appear above the respective table, explaining the content of the column (3).  

<img src="../img/screenshots/ePiE/screens_3.png" alt="img3" style="width: 100%; max-width: 600px; height: 50%;" />
<figcaption>Figure X: Dummy caption.</figcaption>

By default, the data for ibuprofen are included in the web application, which serves as an illustrative example throughout the process. If the user wants to analyse a different API, parameters can either be manually changed in the respective tables or the user can download an excel template by clicking on “Get Excel template” (4). The template contains data for 36 APIs with 41 data columns for individual variables. 9 of these column are mandatory and are also presented in Table 1 (1). The mandatory columns are:

- API = The name of the specific API
- CAS = The API's CAS number
- class = The chemical class of the API, either neutral, acid, or base
- MW = Molecular Weight [g/mol]
- KOW_n = Octanol/water partitioning coefficient of the neutral form [L H<sub>2</sub>O/L Octanol]
- Pv = Vapour pressure at 25 °C [Pa]
- S = Solubility in water at 25 °C [mg/L]
- pKa = Acid dissociation coefficient [unitless]
- f_uf = Fraction of dose excreted unchanged via urine and faeces, including conjugate metabolites (glucuronides and sulphates) [unitless]

Not mandatory to fill in, but still essential for ePiE, is the k_bio_wwtp column, which indicates the first order biodegradation rate constant for secondary WWTP treatment. The SimpleTreat model uses it to model the removal inside the WWTP. In case this parameter is unknown, it can be set to 0. However, this factor is one of the most important and sensitive parameters in the model!

Table 2 (2) contains supplementary physico-chemical parameters for both the neutral and alternative form of the API. While these properties are not mandatory to run the ePiE model, they refine the model outputs based on the ionisation state of the API.  If no parameter values for the alternative form are specified by the user, <mark>ePiE will assume these based on the Log K<sub>OW</sub> and pK<sub>A</sub>.</mark>

As mentioned above, the excel template contains overall 41 variables that are used in the modelling process. While not all of these can be adjusted within the ePiE App, they can be adjusted in the excel file. For example, the default temperature for surface waters for the neutral form (T_hydro_sw_n) of 293.15 K (20 °C) can be adjusted to reflect measured temperature data in a specific basin.

## WWTP removal

On this page, the removal inside the WWTP is modelled. Table 3 contains the API and estimates the removal after primary and secondary treatment (1). The table is by default empty. After clicking  “Run Simple Treat 4.0” under Table 3 (2), SimpleTreat will estimate the removed fractions, based on the previously provided first order biodegradation rate constant (k_bio_wwtp), and automatically fills the table. If desired, the values predicted by SimpleTreat can be overwritten and for example, replaced by predicted values from quantitative structure–activity relationship (QSARs) models or from experimental data. Afterwards, click on river basin.  

<img src="../img/screenshots/ePiE/screens_4.png" alt="img4" style="width: 100%; max-width: 600px; height: 50%;" />
<img src="../img/screenshots/ePiE/screens_5.png" alt="img5" style="width: 100%; max-width: 600px; height: 50%;" />
<figcaption>Figure X: Dummy caption.</figcaption>

## River basin

On the river basin tab, a map of Europe will appear (1). One or multiple river basins can be selected by manually clicking on the desired river basin on the map. Basins can be deleted by clicking on them again on the map or by clicking on the ID below the map. Note that not all river basins are covered by ePiE, especially the ones close to coastal areas. This is because there is a minimum size requirement for inclusion in the underlying database.

<img src="../img/screenshots/ePiE/screens_6.png" alt="img6" style="width: 100%; max-width: 600px; height: 50%;" />
<figcaption>Figure X: Dummy caption.</figcaption>

As ePiE estimates concentrations based on consumption data, these data need to be entered for each country that the river crosses or borders. For example, estimating the concentrations in the Danube river basin would require consumption data for 12 countries. For illustrative purposes, we will continue our example with the Ouse river basin (2), which only requires consumption data for the United Kingdom. This river basin has been thoroughly validated previously by [Oldenkamp et al. (2018)](https://pubs.acs.org/doi/10.1021/acs.est.8b03862).

At the bottom of the page, the specific flow conditions can be chosen which are by default set to average yearly flow conditions. It is also possible to select minimum or maximum flow conditions. Keep the default options and after having selected the Ouse river basin, click on the tab “Consumption data”.

 <img src="../img/screenshots/ePiE/screens_7.png" alt="img7" style="width: 100%; max-width: 600px; height: 50%;" />
 <figcaption>Figure X: Dummy caption.</figcaption>

## Consumption data 

On the consumption data tab, the consumption of an API in a given country needs to be specified. At first, the page is empty but after clicking “Generate table” (1) a table will be generated listing all countries associated with the selected river basin(s). 

<img src="../img/screenshots/ePiE/screens_8.png" alt="img8" style="width: 100%; max-width: 600px; height: 50%;" />
<figcaption>Figure X: Dummy caption.</figcaption>

You can now add the per capita consumption to the top table and select a year for which the respective population size should be retrieved. The values in the table can be manually overwritten by clicking “Edit Table” (2) if other, reliable data sources are available. After having filled in the necessary data, continue by clicking on “Run ePiE”.

<img src="../img/screenshots/ePiE/screens_9.png" alt="img9" style="width: 100%; max-width: 600px; height: 50%;" />
<figcaption>Figure X: Dummy caption.</figcaption>

## Run ePiE

At the top of the page, you will see two buttons: “Save current settings” and “Load settings from file”. Clicking on “Save current settings” (1) will generate and download a JavaScript Object Notation (JSON) file. The file will be called “ePie_settings” by default, followed by the current date and a unique identifier, such as “ePiE_settings_2026-03-16_13402.json”. JSON files are compact data files that enable the efficient storage of data. Accordingly, the json file will contain all parameter values previously entered into the individual tables. A .json file can also be used to parameterise ePiE by loading it into the web application by clicking on “Load settings from file” (2), skipping all previous steps. 

<img src="../img/screenshots/ePiE/screens_10.png" alt="img10" style="width: 100%; max-width: 600px; height: 50%;" />
<figcaption>Figure X: Dummy caption.</figcaption>

Clicking on the button “Run ePiE” will run the model (3). Model progress can be followed in the grey box below the button (1). Once ePiE finalised its run, the message “ePiE run completed.” will appear at the bottom and a blue rectangular bar will appear (2). Next, click “Map results”.

<img src="../img/screenshots/ePiE/screens_11.png" alt="img11" style="width: 100%; max-width: 600px; height: 50%;" />
<figcaption>Figure X: Dummy caption.</figcaption>

## Map results

The page will at first show only the introduction text at the top and a grey square. After clicking “Generate Map” (1) the predicted environmental concentrations in the respective river basins will be visualised, with red values indicating high and blue values low concentrations. 

<img src="../img/screenshots/ePiE/screens_12.png" alt="img12" style="width: 100%; max-width: 600px; height: 50%;" />
<figcaption>Figure X: Dummy caption.</figcaption>

By clicking on the +/- (1) or using the mouse wheel, you can zoom in the map. By clicking on the map and dragging the mouse, the map can be moved. 

<img src="../img/screenshots/ePiE/screens_13.png" alt="img13" style="width: 100%; max-width: 600px; height: 50%;" />
<figcaption>Figure X: Dummy caption.</figcaption>



Individual data points on the map represent predicted concentrations in ng/L. While the legend is using a log10 scale (1), hovering over individual points and clicking them shows the non-logarithmic value (2). If desired, the data can also be exported as a GeoJSON or excel file (3)[^1]. Furthermore, the ID and pt type are reported. While IDs represent an individual numerical identifier, pt types represent specific locations within a river network, called nodes in ePiE. These locations are based on the HydroLakeS3[^2] and UWWTD-Waterbase[^3] database and are:
 
- Hydro_lake: Node representing a lake
- START: Nodes representing a river source
- JNCT: Nodes where two streams meet
- MOUTH: Nodes where a river flows into the sea
- WWTP & Agglomerations: Node representing a WWTP or agglomeration, respectively. Both are classified as emission sources
- Node: Regular node representing a river stretch of typically 1 km


<img src="../img/screenshots/ePiE/screens_14.png" alt="img14" style="width: 100%; max-width: 600px; height: 50%;" />
<figcaption>Figure X: Dummy caption.</figcaption>

After inspecting the map, continue to “Output statistics”. 
[^1]: Export as GeoJSON is currently disabled.
[^2]: [https://www.hydrosheds.org/products/hydrolakes](https://www.hydrosheds.org/products/hydrolakes)
[^3]: [https://www.eea.europa.eu/en/datahub/datahubitem-view/6244937d-1c2c-47f5-bdf1-33ca01ff1715](https://www.eea.europa.eu/en/datahub/datahubitem-view/6244937d-1c2c-47f5-bdf1-33ca01ff1715)

## Output statistics

On this tab, summary statistics of the predicted environmental concentrations are shown after clicking on “Calculate statistics” (1). Results can also be exported to excel by clicking on “Export statistics to Excel” (2). Default values are calculated in ng/L and cover the mean and median for all predicted concentrations, as well as the 5th and 95th percentiles. Furthermore, the mean and median concentrations downstream of all WWTPs in the river basin are calculated (3). Next, we will go to the tab “Map risks”.

<img src="../img/screenshots/ePiE/screens_15.png" alt="img15" style="width: 100%; max-width: 600px; height: 50%;" />
<figcaption>Figure X: Dummy caption.</figcaption>

## Map risks

Under the map risk tab, the respective risk thresholds of the API need to be specified. This is by default set to 1.0 ng/L, but needs to be changed for the specific API (1). 

After clicking on “Generate Map” (2), a map visualising predicted risks will appear. The map behaves the same as described previously under “Map results”. However, hovering over individual data points will only show predicted concentrations, not the calculated risk quotient (3). 


<img src="../img/screenshots/ePiE/screens_16.png" alt="img16" style="width: 100%; max-width: 600px; height: 50%;" />
<figcaption>Figure X: Dummy caption.</figcaption>

## Risk statistics

Summary statistics of the predicted risks are shown under this tab after clicking on “Calculate risk statistics” (1). Again, results can also be exported to excel by clicking on “Export statistics to Excel” (2).
The risk statistics provided in the table (3) show how many values fall into specific range of the RQ which are:

-	RQ < 0.1
-	RQ between 0.1 – 1.0
-	RQ between 1.0 – 10
-	RQ > 10

Next click on the “View Settings ” tab.

<img src="../img/screenshots/ePiE/screens_17.png" alt="img17" style="width: 100%; max-width: 600px; height: 50%;" />
<figcaption>Figure X: Dummy caption.</figcaption>


## View settings

This tab provides a complete overview of all parameters that were used in the current ePiE run (1). These can be saved and downloaded again as a .json file (2) and can be reused for future runs by loading them into the web application (3). As mentioned under “API properties”, overall 41 data variables are used in the modelling process, which can be adjusted in the excel file. However, they can also be adjusted in the .json file.

<img src="../img/screenshots/ePiE/screens_18.png" alt="img18" style="width: 100%; max-width: 600px; height: 50%;" />
<figcaption>Figure X: Dummy caption.</figcaption>



