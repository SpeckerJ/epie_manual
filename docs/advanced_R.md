## Advanced R usage

```R
# Load packages
library(ePiE)
library(tidyverse)
library(readxl)


# Read in raw data ####

# APIs data
df_chems <- read_excel("Multiple_API.xlsx")

# Consumption data
cons_files <- list.files(pattern = "^consumption_.*\\.csv$")
cons_list  <- lapply(cons_files, read.csv)
names(cons_list) <- sapply(cons_list, function(x) names(x)[ncol(x)])

# Effect data
PNEC_file <- list.files(pattern = "_PNECs.*\\.xlsx$")
df_effects <- read_excel(PNEC_file) %>% as.data.frame()

# Custom function ####
# This function checks if the actual data type of df_chems is the expected data type
check_column_types <- function(df, expected_char) {
  test_class <- lapply(df, function(x) class(x)[1])
  
  class_df <- data.frame(
    column      = names(test_class),
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
        expected_type == "numeric"   ~ actual_type %in% c("numeric", "integer", "double"),
        TRUE ~ FALSE
      )
    )
  
  class_df
}

expected_char <- c("API", "CAS", "class")

# Controll data types ####
class_df <- check_column_types(df_chems, expected_char)

# We see that at least one data type is wrong!
# Check what is causing this in the raw data sheet or change within R as 
class_df %>% filter(!class_check)

# Data wrangling ####
# Convert character/integer columns to numeric
df_chems <- df_chems %>%
  mutate(
    across(
      .cols = (4:ncol(df_chems))[sapply(df_chems[4:ncol(df_chems)], is.character)],
      .fns  = as.numeric
    ),
    across(
      .cols = (4:ncol(df_chems))[sapply(df_chems[4:ncol(df_chems)], is.integer)],
      .fns  = as.numeric
    )
  )

# Second check
class_df <- check_column_types(df_chems, expected_char)
# All data types are correct now!
class_df %>% filter(!class_check)


# Control if the solubility is below the PNEC ####
check_PNEC_sol <- left_join(df_chems, df_effects, by = "API") %>% 
  select(c(API, S, PNEC, unit)) %>% 
  mutate(
    check = S < PNEC # TRUE = S < PNEC; FALSE = S > PNEC
    )
check_PNEC_sol %>% filter(check)

# However, solubility is in mg/L, the PNEC is in ng/L. Always control units! 
check_PNEC_sol <- check_PNEC_sol %>% mutate(
  S = S * 1000000,
  check = S < PNEC
)
check_PNEC_sol %>% filter(check) # No PNEC is above the solubility

# Complete missing values in chem data ####
df_chems <- CompleteChemProperties(chem = df_chems)

# Load basin data and select basin
basins    <- LoadEuropeanBasins()
basin_ids <- c(107287) # Ouse (Yorkshire)
basins    <- SelectBasins(basins_data = basins, basin_ids = basin_ids)

# Load river flow and attach to basin
flow_avg   <- LoadLongTermFlow("average")
basins_avg <- AddFlowToBasinData(basin_data = basins, flow_rast = flow_avg)

# SimpleTreat ####
removal_list <- vector("list", nrow(df_chems))

for (i in seq_len(nrow(df_chems))) {
  removal_list[[i]] <- SimpleTreat4_0(
    chem_class = df_chems$class[i],      MW         = df_chems$MW[i],
    Pv         = df_chems$Pv[i],         S          = df_chems$S[i],
    pKa        = df_chems$pKa[i],        Kp_ps      = df_chems$Kp_ps[i],
    Kp_as      = df_chems$Kp_as[i],      k_bio_WWTP = df_chems$k_bio_wwtp[i],
    T_air = 285, Wind = 4, Inh = 1000, E_rate = 1, PRIM = -1, SEC = -1
  )
}
names(removal_list) <- df_chems$API
removal_list


# Check consumption data ####
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

# Compute environmental concentrations ####
results_list <- vector("list", length(chem_list))
names(results_list) <- names(chem_list)

for (api in names(chem_list)) {
  results_list[[api]] <- ComputeEnvConcentrations(
    basin_data = basins_avg,
    chem       = chem_list[[api]],
    cons       = API_cons[[api]],
    verbose    = TRUE,
    cpp        = TRUE
  )
}
results_list
results_list[[1]]
results_list["Acetaminophen"]

# Inspect map ####
# InteractiveResultMap(results_list[[1]], basin_id = basin_ids, cex = 4)

# Combine results ####
results_pts <- bind_rows(lapply(results_list, `[[`, "pts"))
results_hl  <- bind_rows(lapply(results_list, `[[`, "hl"))

# Add PNECs and calculate RQ ####
results_comb <- left_join(results_pts, df_effects, by = "API")
results_RQ   <- results_comb %>% mutate(RQ = C_w / PNEC)

# PEC summary ####
# Note: Here we are changing the predicted concentrations to ng/L. Previously, we adapted the PNEC value
PEC_sum <- results_pts %>%
  mutate(C_w = C_w * 1000) %>% 
  group_by(API, basin_id) %>%
  summarise(
    mean_conc_ngL        = mean(C_w, na.rm = TRUE),
    perc5_conc_ngL       = quantile(C_w, 0.05, na.rm = TRUE),
    median_conc_ngL      = median(C_w, na.rm = TRUE),
    perc95_conc_ngL      = quantile(C_w, 0.95, na.rm = TRUE),
    mean_WWTP_conc_ngL   = mean(C_w[Pt_type == "WWTP"], na.rm = TRUE),
    median_WWTP_conc_ngL = median(C_w[Pt_type == "WWTP"], na.rm = TRUE)
  ) %>% ungroup()
PEC_sum

# RQ summary ####
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

# Influence of NA predictions ####
results_pts_B <- results_comb %>% mutate(
  C_w = if_else(is.na(C_w), 0, C_w) * 1000,
  RQ = C_w/PNEC
)

PEC_sum_B <- results_pts_B %>% 
  group_by(API, basin_id) %>%
  summarise(
    mean_conc_ngL        = mean(C_w),
    perc5_conc_ngL       = quantile(C_w, 0.05),
    median_conc_ngL      = median(C_w),
    perc95_conc_ngL      = quantile(C_w, 0.95),
    mean_WWTP_conc_ngL   = mean(C_w[Pt_type == "WWTP"]),
    median_WWTP_conc_ngL = median(C_w[Pt_type == "WWTP"])
  ) %>% ungroup()

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

PEC_sum$col <- "No NAs"
PEC_sum_B$col <- "NAs to 0"
rbind(PEC_sum, PEC_sum_B)

RQ_sum$col <- "No NAs"
RQ_sum_B$col <- "NAs to 0"
rbind(RQ_sum, RQ_sum_B)


# Loop multiple river flows ####

# Load river flow and attach to basin for all three hydrological scenarios
flow_scenarios <- list(
  average = LoadLongTermFlow("average"),
  maximum = LoadLongTermFlow("maximum"),
  minimum = LoadLongTermFlow("minimum")
)

basins_flow <- lapply(flow_scenarios, function(flow) {
  AddFlowToBasinData(basin_data = basins, flow_rast = flow)
})

# Compute environmental concentrations for all scenarios x all APIs
results_flow <- vector("list", length(basins_flow))
names(results_flow) <- names(basins_flow)

for (scenario in names(basins_flow)) {
  
  # Initialise inner list for this scenario
  results_flow[[scenario]] <- vector("list", length(chem_list))
  names(results_flow[[scenario]]) <- names(chem_list)
  
  for (api in names(chem_list)) {
    # message("Running: scenario = ", scenario, " | API = ", api)
    
    results_flow[[scenario]][[api]] <- ComputeEnvConcentrations(
      basin_data = basins_flow[[scenario]],
      chem       = chem_list[[api]],
      cons       = API_cons[[api]],
      verbose    = TRUE,
      cpp        = TRUE
    )
  }
}

# One specific combination
results_ibu_avg <- results_flow[["average"]][["Ibuprofen"]]

# Investigate map
# InteractiveResultMap(results_ibu_avg, basin_id = basin_ids, cex = 4)

# All APIs for the minimum flow scenario
results_flow[["minimum"]]

# Combine pts across all APIs for one scenario
results_pts_avg <- bind_rows(lapply(results_flow[["average"]], `[[`, "pts"))

# Combine pts across ALL scenarios and ALL APIs, keeping track of scenario
results_pts_all <- bind_rows(
  lapply(names(results_flow), function(scenario) {
    bind_rows(lapply(results_flow[[scenario]], `[[`, "pts")) %>%
      mutate(scenario = scenario)   # add scenario column so you know where each row came from
  })
)

results_pts_all %>% mutate(C_w = C_w * 1000) %>% 
  group_by(API, basin_id, scenario) %>%
  summarise(
    mean_conc_ngL        = mean(C_w, na.rm = TRUE),
    perc5_conc_ngL       = quantile(C_w, 0.05, na.rm = TRUE),
    median_conc_ngL      = median(C_w, na.rm = TRUE),
    perc95_conc_ngL      = quantile(C_w, 0.95, na.rm = TRUE),
    mean_WWTP_conc_ngL   = mean(C_w[Pt_type == "WWTP"], na.rm = TRUE),
    median_WWTP_conc_ngL = median(C_w[Pt_type == "WWTP"], na.rm = TRUE)
  ) %>% ungroup()


# Loop multiple river flows and basins ####
basin_ids <- c(107287, 107964)
basins    <- SelectBasins(basins_data = LoadEuropeanBasins(), basin_ids = basin_ids)

# Add flow per scenario — basins already contains both basins
flow_scenarios <- list(
  average = LoadLongTermFlow("average"),
  maximum = LoadLongTermFlow("maximum"),
  minimum = LoadLongTermFlow("minimum")
)

basins_flow <- lapply(flow_scenarios, function(flow) {
  AddFlowToBasinData(basin_data = basins, flow_rast = flow)
})

results_flow <- vector("list", length(basins_flow))
names(results_flow) <- names(basins_flow)

for (scenario in names(basins_flow)) {
  
  results_flow[[scenario]] <- vector("list", length(chem_list))
  names(results_flow[[scenario]]) <- names(chem_list)
  
  for (api in names(chem_list)) {
    message("Running: scenario = ", scenario, " | API = ", api)
    
    results_flow[[scenario]][[api]] <- ComputeEnvConcentrations(
      basin_data = basins_flow[[scenario]],   # contains BOTH basins
      chem       = chem_list[[api]],
      cons       = API_cons[[api]],
      verbose    = TRUE,
      cpp        = TRUE
    )
  }
}

results_all <- bind_rows(
  lapply(names(results_flow), function(scenario) {
    bind_rows(
      lapply(names(results_flow[[scenario]]), function(api) {
        results_flow[[scenario]][[api]]$pts %>%
          mutate(api = api)
      })
    ) %>%
      mutate(scenario = scenario)
  })
)
names(results_flow)
PEC_sum <- results_all %>%
  group_by(API, basin_id, scenario) %>%
  summarise(
    mean_C_w   = mean(C_w * 1000, na.rm = TRUE),
    median_C_w = median(C_w * 1000, na.rm = TRUE),
    .groups = "drop"
  )
```
