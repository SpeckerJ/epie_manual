## Advanced R usage

The following workflow runs ePiE for multiple APIs. The input files should be stored in the active project folder and can be downloaded via the link below:

[Multiple_API](Multiple_API.xlsx){: .md-button download="Multiple_API"}

[Consumption_acetaminophen](consumption_acetaminophen.csv){: .md-button download="consumption_acetaminophen"}

[Consumption_diclofenac](consumption_diclofenac.csv){: .md-button download="consumption_diclofenac"}

[Consumption_ibuprofen](consumption_ibuprofen.csv){: .md-button download="consumption_ibuprofen"}

The chemical-property file (Multiple_API.xlsx) contains one row per API. Consumption files must follow the naming format consumption_*API*.csv, where *API* corresponds exactly to the API name in the chemical-property file. The code automatically identifies and imports all matching consumption files. It should be noted that the data used in this example is purely illustrative. The PNECs and consumption figures are not accurate and were generated solely to illustrate the code!

## Load packages and input data

At first, we have to load the required packages and imports the chemical-property, consumption, and PNEC data. Please note, that in this example, we are loading the epie and tidyverse package together. Unintended bugs may occur between the various package functions, but we are not aware of any such issues to date. In the previous section (*How to: The ePiE R package*), the `tidyverse` package was only loaded after `epie` had already made its predictions, and the data was then simply cleaned and analysed.

```R
library(ePiE)
library(tidyverse)
library(readxl)

# Chemical-property data: one row per API
df_chems <- read_excel("Multiple_API.xlsx")

# Consumption data: one CSV file per API
cons_files <- list.files(pattern = "^consumption_.*\\.csv$")
cons_list <- lapply(cons_files, read.csv)
names(cons_list) <- sapply(cons_list, function(x) names(x)[ncol(x)])

# Effect data: API-specific PNECs
PNEC_file <- list.files(pattern = "_PNECs.*\\.xlsx$")
df_effects <- read_excel(PNEC_file) %>% as.data.frame()
```

## Check chemical-property data types

The ePiE input data must use the correct column types. API, CAS, and class must be character variables; all remaining chemical-property columns must be numeric. The function below checks the imported data and identifies columns with an unexpected type.

```R
check_column_types <- function(df, expected_char) {
  test_class <- lapply(df, function(x) class(x)[1])

  data.frame(
    column = names(test_class),
    actual_type = unlist(test_class),
    stringsAsFactors = FALSE
  ) %>%
    mutate(
      expected_type = case_when(
        column %in% expected_char ~ "character",
        TRUE ~ "numeric"
      ),
      class_check = case_when(
        expected_type == "character" ~ actual_type == "character",
        expected_type == "numeric" ~ actual_type %in% c("numeric", "integer", "double"),
        TRUE ~ FALSE
      )
    )
}

expected_char <- c("API", "CAS", "class")
class_df <- check_column_types(df_chems, expected_char)

# Display columns with an unexpected type
class_df %>% filter(!class_check)
```

If numerical columns were imported as character or integer values, they can be converted as follows. Inspect warnings carefully: non-numerical text converted with as.numeric() becomes NA and should preferably be corrected in the original input file to avoid unindented NAs. However, if the current file structure is used, resulting NAs would not result in any issues.

```R
# Convert character and integer columns from the numerical property section to numeric
df_chems <- df_chems %>%
  mutate(
    across(
      .cols = (4:ncol(df_chems))[sapply(df_chems[4:ncol(df_chems)], is.character)],
      .fns = as.numeric
    ),
    across(
      .cols = (4:ncol(df_chems))[sapply(df_chems[4:ncol(df_chems)], is.integer)],
      .fns = as.numeric
    )
  )

# Confirm that all columns now have the expected type
class_df <- check_column_types(df_chems, expected_char)
class_df %>% filter(!class_check)
```

## Check solubility against PNECs

This check identifies whether a PNEC exceeds the maximum dissolved concentration indicated by the reported water solubility. This is a basic data check as in such an instance, the PNEC could never be reached.
Solubility in ePiE is expressed in mg/L. The example PNECs are expressed in ng/L, so solubility is converted to ng/L before the final comparison. If your PNEC data use a unit other than ng/L, adapt the conversion accordingly before comparing values.


```R
# Inspect the data before unit harmonisation
check_PNEC_sol <- left_join(df_chems, df_effects, by = "API") %>%
  select(API, S, PNEC, unit) %>%
  mutate(check = S &lt; PNEC)

check_PNEC_sol %>% filter(check)

# Convert solubility from mg/L to ng/L and repeat the comparison
check_PNEC_sol <- check_PNEC_sol %>%
  mutate(
    S = S * 1000000,
    check = S < PNEC
  )

# An empty result means that no PNEC exceeds the reported solubility
check_PNEC_sol %>% filter(check)
```

## Complete chemical properties and prepare a basin

CompleteChemProperties() fills missing ePiE chemical properties using default values. The following example selects the Ouse basin in Yorkshire and assigns average long-term river flow to its nodes.

```R
# Complete missing chemical properties
df_chems <- CompleteChemProperties(chem = df_chems)

# Load basin data and select the Ouse basin
basins <- LoadEuropeanBasins()
basin_ids <- c(107287) # Ouse, Yorkshire
basins <- SelectBasins(basins_data = basins, basin_ids = basin_ids)

# Add average long-term flow
flow_avg <- LoadLongTermFlow("average")
basins_avg <- AddFlowToBasinData(basin_data = basins, flow_rast = flow_avg)
```

## Estimate WWTP removal for all APIs
The SimpleTreat4_0() model is applied separately to each API. The output is a named list of predicted WWTP-removal results, which can be inspected for individual compounds.

```R
removal_list <- vector("list", nrow(df_chems))

for (i in seq_len(nrow(df_chems))) {
  removal_list[[i]] <- SimpleTreat4_0(
    chem_class = df_chems$class[i],
    MW         = df_chems$MW[i],
    Pv         = df_chems$Pv[i],
    S          = df_chems$S[i],
    pKa        = df_chems$pKa[i],
    Kp_ps      = df_chems$Kp_ps_n[i],
    Kp_as      = df_chems$Kp_as_n[i],
    k_bio_WWTP = df_chems$k_bio_wwtp[i],
    T_air = 285, Wind = 4, Inh = 1000, E_rate = 1, PRIM = -1, SEC = -1
  )
}

names(removal_list) <- df_chems$API
removal_list
```

## Check consumption data

Chemical properties are split into one data frame per API. Consumption data are subsequently checked for every API and selected basin. This verifies that consumption information is available for countries contributing wastewater emissions to the selected basin.

```R
chem_list <- split(df_chems, df_chems$API)

API_cons <- vector("list", length(chem_list))
names(API_cons) <- names(chem_list)

for (api in names(chem_list)) {
  API_cons[[api]] <- CheckConsumptionData(
    basins$pts,
    chem_list[[api]],
    cons_list[[api]]
  )
}

API_cons
```

## Predict concentrations for multiple APIs
The model is run once for each API. Results are stored in a named list, with each API containing the standard ePiE output:
pts: predicted concentrations at individual river, lake, and WWTP nodes;
hl: summarised predictions for lakes.


```R
results_list <- vector("list", length(chem_list))
names(results_list) <- names(chem_list)

for (api in names(chem_list)) {
  results_list[[api]] <- ComputeEnvConcentrations(
    basin_data = basins_avg,
    chem = chem_list[[api]],
    cons = API_cons[[api]],
    verbose = TRUE,
    cpp = TRUE
  )
}

# Inspect all results, the first result, or a named API
results_list
results_list[[1]]
results_list[["Acetaminophen"]]

# Optional: inspect one API on an interactive map
# InteractiveResultMap(results_list[[1]], basin_id = basin_ids, cex = 4)
```

The individual API results can be combined into common node-level and lake-level datasets. This enables direct comparisons among APIs.

```R
results_pts <- bind_rows(lapply(results_list, `[[`, "pts"))
results_hl <- bind_rows(lapply(results_list, `[[`, "hl"))
```

## Calculate PECs and risk quotients
PNECs are joined to the node-level predictions using the API name. PECs and PNECs must use the same unit. ePiE reports water concentrations (C_w) in µg/L. In the example below, PNECs are in ng/L. Therefore, predicted concentrations are converted to ng/L before calculating RQs.

```R
results_comb <- left_join(results_pts, df_effects, by = "API")

results_RQ <- results_comb %>%
  mutate(
    C_w_ngL = C_w * 1000,
    RQ = C_w_ngL / PNEC
  )
```

Predicted environmental concentrations are summarised per API and basin. The output includes the mean, median, fifth percentile, and 95th percentile, together with mean and median concentrations at WWTP nodes. Concentrations are reported in ng/L.

```R
PEC_sum <- results_pts %>%
  mutate(C_w = C_w * 1000) %>%
  group_by(API, basin_id) %>%
  summarise(
    mean_conc_ngL = mean(C_w, na.rm = TRUE),
    perc5_conc_ngL = quantile(C_w, 0.05, na.rm = TRUE),
    median_conc_ngL = median(C_w, na.rm = TRUE),
    perc95_conc_ngL = quantile(C_w, 0.95, na.rm = TRUE),
    mean_WWTP_conc_ngL = mean(C_w[Pt_type == "WWTP"], na.rm = TRUE),
    median_WWTP_conc_ngL = median(C_w[Pt_type == "WWTP"], na.rm = TRUE),
    .groups = "drop"
  )

PEC_sum
```
Risk quotients are summarised as the relative proportion of nodes in four categories: 

- RQ < 0.1
- 0.1 < RQ < 1
- 1 < RQ < 10
- RQ > 10

These categories provide a basin-wide overview of potential environmental risk for each API.

```R
RQ_sum <- results_RQ %>%
  group_by(API, basin_id) %>%
  summarise(
    abs.RQ_less_0.1  = sum(RQ < 0.1, na.rm = TRUE),
    abs.RQ_0.1_to_1  = sum(RQ >= 0.1 & RQ <= 1.0, na.rm = TRUE),
    abs.RQ_1_to_10   = sum(RQ >= 1 & RQ <= 10, na.rm = TRUE),
    abs.RQ_abv_10    = sum(RQ >= 10, na.rm = TRUE),
    tot.RQ           = n(),
    rel_RQ_less_0.1  = abs.RQ_less_0.1 / tot.RQ,
    rel_RQ_0.1_to_1  = abs.RQ_0.1_to_1 / tot.RQ,
    rel_RQ_1_to_10   = abs.RQ_1_to_10 / tot.RQ,
    rel_RQ_abv_10    = abs.RQ_abv_10 / tot.RQ,
    .groups = "keep"
  ) %>%
  select(-matches(c("tot.", "abs.")))

RQ_sum
```

## Assess the influence of missing predictions

Some nodes may have missing predicted concentrations (NA). This sensitivity analysis compares the standard summaries, which exclude missing values, with an alternative in which missing predictions are set to zero. It is important to be aware of the fact that reliable predictions cannot be made for all nodes of the river network. These result in NA values, which however do not mean that the respective API is not present in the environment.

```R
# Set missing predicted concentrations to zero, then calculate PECs and RQs
results_pts_B <- results_comb %>%
  mutate(
    C_w = if_else(is.na(C_w), 0, C_w) * 1000,
    RQ = C_w / PNEC
  )

PEC_sum_B <- results_pts_B %>%
  group_by(API, basin_id) %>%
  summarise(
    mean_conc_ngL = mean(C_w),
    perc5_conc_ngL = quantile(C_w, 0.05),
    median_conc_ngL = median(C_w),
    perc95_conc_ngL = quantile(C_w, 0.95),
    mean_WWTP_conc_ngL = mean(C_w[Pt_type == "WWTP"]),
    median_WWTP_conc_ngL = median(C_w[Pt_type == "WWTP"]),
    .groups = "drop"
  )

RQ_sum_B <- results_pts_B %>% group_by(API, basin_id) %>%
  summarise(
    abs.RQ_less_0.1  = sum(RQ < 0.1, na.rm = TRUE),
    abs.RQ_0.1_to_1  = sum(RQ >= 0.1 & RQ <= 1.0, na.rm = TRUE),
    abs.RQ_1_to_10   = sum(RQ >= 1 & RQ <= 10, na.rm = TRUE),
    abs.RQ_abv_10    = sum(RQ >= 10, na.rm = TRUE),
    tot.RQ           = n(),
    rel_RQ_less_0.1  = abs.RQ_less_0.1 / tot.RQ,
    rel_RQ_0.1_to_1  = abs.RQ_0.1_to_1 / tot.RQ,
    rel_RQ_1_to_10   = abs.RQ_1_to_10 / tot.RQ,
    rel_RQ_abv_10    = abs.RQ_abv_10 / tot.RQ,
    .groups = "keep"
  ) %>%
  select(-matches(c("tot.", "abs.")))

# Compare PEC summaries
PEC_sum$missing_data <- "Exclude NAs"
PEC_sum_B$missing_data <- "Set NAs to zero"
PEC_comp <- rbind(PEC_sum, PEC_sum_B)

# Compare RQ summaries
RQ_sum$missing_data <- "Exclude NAs"
RQ_sum_B$missing_data <- "Set NAs to zero"
RQ_comp <- rbind(RQ_sum, RQ_sum_B)
```

## Compare flow scenarios
The next workflow repeats the calculations under average, maximum, and minimum long-term hydrological conditions. The output receives a scenario column, so PECs can be compared among flow scenarios.

```R
# Load and attach flow data for each hydrological scenario
flow_scenarios <- list(
  average = LoadLongTermFlow("average"),
  maximum = LoadLongTermFlow("maximum"),
  minimum = LoadLongTermFlow("minimum")
)

basins_flow <- lapply(flow_scenarios, function(flow) {
  AddFlowToBasinData(basin_data = basins, flow_rast = flow)
})

# Run every API for every flow scenario
results_flow <- vector("list", length(basins_flow))
names(results_flow) <- names(basins_flow)

for (scenario in names(basins_flow)) {
  results_flow[[scenario]] <- vector("list", length(chem_list))
  names(results_flow[[scenario]]) <- names(chem_list)

  for (api in names(chem_list)) {
    results_flow[[scenario]][[api]] <- ComputeEnvConcentrations(
      basin_data = basins_flow[[scenario]],
      chem = chem_list[[api]],
      cons = API_cons[[api]],
      verbose = TRUE,
      cpp = TRUE
    )
  }
}

# Extract one API–scenario combination
results_ibu_avg <- results_flow[["average"]][["Ibuprofen"]]

# Optional: inspect one result on a map
# InteractiveResultMap(results_ibu_avg, basin_id = basin_ids, cex = 4)
```

All node-level predictions can then be combined and summarised by API, basin, and flow scenario.
Lower flow generally provides less dilution and may therefore result in higher predicted concentrations.

```R
results_pts_all <- bind_rows(
  lapply(names(results_flow), function(scenario) {
    bind_rows(lapply(results_flow[[scenario]], `[[`, "pts")) %>%
      mutate(scenario = scenario)
  })
)

PEC_sum_flow <- results_pts_all %>%
  mutate(C_w = C_w * 1000) %>%
  group_by(API, basin_id, scenario) %>%
  summarise(
    mean_conc_ngL = mean(C_w, na.rm = TRUE),
    perc5_conc_ngL = quantile(C_w, 0.05, na.rm = TRUE),
    median_conc_ngL = median(C_w, na.rm = TRUE),
    perc95_conc_ngL = quantile(C_w, 0.95, na.rm = TRUE),
    mean_WWTP_conc_ngL = mean(C_w[Pt_type == "WWTP"], na.rm = TRUE),
    median_WWTP_conc_ngL = median(C_w[Pt_type == "WWTP"], na.rm = TRUE),
    .groups = "drop"
  )

PEC_sum_flow
```

## Compare multiple basins and flow scenarios
The same approach can be extended to multiple basins. After changing the basin selection, rerun `CheckConsumptionData()` so that consumption data are checked for all countries contributing to the new set of basins.

```R
# Select the Ouse and the Rhine basin
basin_ids <- c(107287, 107964)
basins <- SelectBasins(
  basins_data = LoadEuropeanBasins(),
  basin_ids = basin_ids
)

# Re-check consumption data for the updated basin selection
for (api in names(chem_list)) {
  API_cons[[api]] <- CheckConsumptionData(
    basins$pts,
    chem_list[[api]],
    cons_list[[api]]
  )
}

# Add flow data for each scenario
flow_scenarios <- list(
  average = LoadLongTermFlow("average"),
  maximum = LoadLongTermFlow("maximum"),
  minimum = LoadLongTermFlow("minimum")
)

basins_flow <- lapply(flow_scenarios, function(flow) {
  AddFlowToBasinData(basin_data = basins, flow_rast = flow)
})

# Run every API for every scenario across both basins
results_flow <- vector("list", length(basins_flow))
names(results_flow) <- names(basins_flow)

for (scenario in names(basins_flow)) {
  results_flow[[scenario]] <- vector("list", length(chem_list))
  names(results_flow[[scenario]]) <- names(chem_list)

  for (api in names(chem_list)) {
    message("Running: scenario = ", scenario, " | API = ", api)

    results_flow[[scenario]][[api]] <- ComputeEnvConcentrations(
      basin_data = basins_flow[[scenario]],
      chem = chem_list[[api]],
      cons = API_cons[[api]],
      verbose = TRUE,
      cpp = TRUE
    )
  }
}
```

Finally, combine the results and calculate mean and median PECs for each API, basin, and flow scenario. The final table provides a concise comparison of mean and median PECs across APIs, river basins, and long-term flow conditions.

```R
results_all <- bind_rows(
  lapply(names(results_flow), function(scenario) {
    bind_rows(lapply(results_flow[[scenario]], `[[`, "pts")) %>%
      mutate(scenario = scenario)
  })
)

PEC_sum_multi <- results_all %>%
  group_by(API, basin_id, scenario) %>%
  summarise(
    mean_C_w_ngL = mean(C_w * 1000, na.rm = TRUE),
    median_C_w_ngL = median(C_w * 1000, na.rm = TRUE),
    .groups = "drop"
  )

PEC_sum_multi
```

