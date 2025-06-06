MATH117 Final Project Memo
================
Community Concepts 1: Lily, Sam, Takumi

This document should contain a detailed account of the data clean up for
your data and the design choices you are making for your plots. For
instance you will want to document choices you’ve made that were
intentional for your graphic, e.g. color you’ve chosen for the plot.
Think of this document as a code script someone can follow to reproduce
the data cleaning steps and graphics in your handout.

``` r
library(tidyverse)
library(broom)
library(readxl)
library(viridis)
library(dplyr)
library(tidyr)
library(ggrepel)
```

## Data Clean Up Steps for Overall Data

### Load Data

``` r
sept_kpis <- read_excel("../data/FY24 Sept KPIs - Total Agency.xlsx")
```

    ## New names:
    ## • `` -> `...2`
    ## • `` -> `...18`
    ## • `` -> `...19`

``` r
town_campaign <- read_excel("../data/FY24 Town Campaign Data.xlsx", sheet = 1, col_names = FALSE)
```

    ## New names:
    ## • `` -> `...1`
    ## • `` -> `...2`
    ## • `` -> `...3`
    ## • `` -> `...4`
    ## • `` -> `...5`
    ## • `` -> `...6`
    ## • `` -> `...7`
    ## • `` -> `...8`
    ## • `` -> `...9`
    ## • `` -> `...10`
    ## • `` -> `...11`
    ## • `` -> `...12`
    ## • `` -> `...13`
    ## • `` -> `...14`
    ## • `` -> `...15`
    ## • `` -> `...16`
    ## • `` -> `...17`
    ## • `` -> `...18`
    ## • `` -> `...19`
    ## • `` -> `...20`
    ## • `` -> `...21`
    ## • `` -> `...22`
    ## • `` -> `...23`
    ## • `` -> `...24`
    ## • `` -> `...25`
    ## • `` -> `...26`
    ## • `` -> `...27`
    ## • `` -> `...28`
    ## • `` -> `...29`
    ## • `` -> `...30`
    ## • `` -> `...31`
    ## • `` -> `...32`
    ## • `` -> `...33`
    ## • `` -> `...34`
    ## • `` -> `...35`
    ## • `` -> `...36`
    ## • `` -> `...37`
    ## • `` -> `...38`
    ## • `` -> `...39`
    ## • `` -> `...40`
    ## • `` -> `...41`
    ## • `` -> `...42`
    ## • `` -> `...43`
    ## • `` -> `...44`
    ## • `` -> `...45`
    ## • `` -> `...46`
    ## • `` -> `...47`
    ## • `` -> `...48`
    ## • `` -> `...49`
    ## • `` -> `...50`
    ## • `` -> `...51`
    ## • `` -> `...52`

### Step 1: Cleaning the KPIs dataset

``` r
#Department Column for kpis
sept_kpis <- sept_kpis |>
  rename("Department" = "Children's Services") 

sept_kpis[1, "Department"] <- "Children's Services"

sept_kpis <- sept_kpis |>
  fill(Department, .direction = "down")

#Program Column for kpis
sept_kpis <- sept_kpis |>
  rename("Program" = "...2") 

sept_kpis[1, "Program"] <- "Children's Services"
sept_kpis[24, "Program"] <- "Housing Services"
sept_kpis[82, "Program"] <- "Development"
sept_kpis[90, "Program"] <- "CCFC"
sept_kpis[103, "Program"] <- "Asset Management"
sept_kpis[107, "Program"] <- "Property Management"
sept_kpis[110, "Program"] <- "Finance"
sept_kpis[140, "Program"] <- "Human Resources"
sept_kpis[142, "Program"] <- "Open Positions"
sept_kpis[157, "Program"] <- "Time to Hire"
sept_kpis[172, "Program"] <- "Head Count"
sept_kpis[187, "Program"] <- "Turnover Rate"

sept_kpis <- sept_kpis |>
  fill(Program, .direction = "down")

sept_kpis <- sept_kpis[, -c(18, 19)]
```

### Step 2: Cleaning Town Campaign Dataset

``` r
town_campaign_cps <- town_campaign |>
  select(...1,...2,...3,...4,...5,...6,...7,...8,...9,...24,...25,...26,...27,...28, ...29)

town_campaign_cps <- town_campaign_cps |>
  rename(town = ...1, tanf_fuelass_households = ...2, tanf_fuelass_investment = ...3, fuelass_households = ...4, fuelass_investment = ...5, heap_households = ...6, heap_investment = ...7, ecip_households = ...8, ecip_investment = ...9,  family_coaching_households = ...24, family_coaching_investment = ...25, maine_families_households = ...26, maine_families_investment = ...27, parent_education_households = ...28, parent_education_investment = ...29 )

town_campaign_cps <- town_campaign_cps[-c(1, 2, 3, 4, 5, 6, 7, 8, 9), ]

town_campaign_cps <- town_campaign_cps[!apply(town_campaign_cps[, -1], 1, function(x) all(is.na(x))), ]
town_campaign_cps[, -1] <- lapply(town_campaign_cps[, -1], as.numeric)
```

    ## Warning in lapply(town_campaign_cps[, -1], as.numeric): NAs introduced by
    ## coercion
    ## Warning in lapply(town_campaign_cps[, -1], as.numeric): NAs introduced by
    ## coercion
    ## Warning in lapply(town_campaign_cps[, -1], as.numeric): NAs introduced by
    ## coercion
    ## Warning in lapply(town_campaign_cps[, -1], as.numeric): NAs introduced by
    ## coercion

``` r
town_campaign_cps[is.na(town_campaign_cps)] <- 0

totals <- colSums(town_campaign_cps[, -1])
town_campaign_cps <- bind_rows(town_campaign_cps, totals)


summary_data <- town_campaign_cps[-seq(1, 193), ]

summary_data <- summary_data|>
  pivot_longer(cols = -town,
               values_to = "value")

summary_data <- summary_data|>
  mutate(data_type = rep(c("households", "investment"), 
                         times = ncol(df) / 2, length.out = nrow(summary_data)),
         program = rep(c("tanf_fuelass", "fuelass", "heap", "ecip", 
                         "family_coaching", "maine_families", "parent_education"), 
                      each = 2, length.out = nrow(summary_data)))

summary_data_long <- summary_data[,-c(1, 2) ]

summary_data_wide <- summary_data_long |>
  pivot_wider(names_from = data_type, values_from = value)
```

## Plots

### Plot 1: Customer & Prevention Services Program Investments

``` r
ggplot(summary_data_wide, aes(x = reorder(program, investment), y = investment, fill = program)) +
  geom_bar(stat="identity", show.legend = FALSE) +
  scale_x_discrete(labels = c("ecip" = "ECIP", 
                              "family_coaching" = "Family Coaching", 
                              "fuelass" = "Fuel Ass. (Non State/Fed)",
                              "heap" = "HEAP",
                              "maine_families" = "Maine Families",
                              "parent_education" = "Parent Education",
                              "tanf_fuelass" = "TANF for Fuel Assistance")) +
  labs(title="Investment into Customer & Prevention Services Programs",
       x="Program", y="Investment in $") +
  coord_flip() +
  scale_fill_viridis_d()
```

<img src="memo_files/figure-gfm/cps-investments-1.png" alt="Horizontal bar chart detailing the amount of financial investment allocated to various CCI Customer and Prevention Services Programs. The chart shows that the largest portion of funding went to the HEAP program, followed by Maine Families, both receiving significantly more than the other programs. ECIP and TANF for Fuel Assistance received moderate funding amounts, while Family Coaching, Non-State and Federal Fuel Assistance, and Parent Education received comparatively smaller investments, with Parent Education receiving the least (and none at all). The chart highlights a clear monetary prioritization of energy and family support services."  />

### Plot 2: Customer & Prevention Services Program Households Helped

``` r
ggplot(summary_data_wide, aes(x = reorder(program, households), y = households, fill = program)) +
  geom_bar(stat="identity", show.legend = FALSE) +
  scale_x_discrete(labels = c("ecip" = "ECIP", 
                              "family_coaching" = "Family Coaching", 
                              "fuelass" = "Fuel Ass. (Non State/Fed)",
                              "heap" = "HEAP",
                              "maine_families" = "Maine Families",
                              "parent_education" = "Parent Education",
                              "tanf_fuelass" = "TANF for Fuel Assistance")) +
  labs(title="Houshelds Helped by Customer & Prevention Services Programs",
       x="Program", y="Households") +
  coord_flip() +
  scale_fill_viridis_d()
```

<img src="memo_files/figure-gfm/cps-households-helped-1.png" alt="Horizontal bar chart illustrating the number of households served by the different customer and prevention service programs. By far, the largest number of households was served by the HEAP program, reaching over 7,000 households. TANF for Fuel Assistance and ECIP were next, each assisting just under 1,000 households. Maine Families supported several hundred households, while Parent Education, Family Coaching, and Fuel Assistance (Non-State/Federal) were the lowest when it came to total households helped."  />

### Plot 3: KPI Progress for Family Services

#### Data cleanup steps specific to plot 3

``` r
sept_kpis_cleaned <- sept_kpis[!is.na(sept_kpis$`% of Goal`), ]
sept_kpis_cleaned <- sept_kpis_cleaned |>
  filter(Department == "Customer & Prevention Services")
sept_kpis_cleaned$YTD <- as.numeric(sept_kpis_cleaned$YTD)
sept_kpis_cleaned <- sept_kpis_cleaned[-c(1,2,3,4,11), ]

sept_kpis_cleaned <- sept_kpis_cleaned |>
  mutate(Assumptions = ifelse(Assumptions == "9500 Applications processed LIHEAP", "LIHEAP Applications Processed", Assumptions)) |>
  mutate(Assumptions = ifelse(Assumptions == "Maine Families - MEICHV  & Families First                              2800 home visits completed per year", "MEICHV & Families First Home Visits", Assumptions)) |>
  mutate(Assumptions = ifelse(Assumptions == "230 families enrolled", "MEICHV & Families First Families Enrolled", Assumptions)) |>
  mutate(Assumptions = ifelse(Assumptions == "CAN Council (CB&E)   -                                                                                               4 community events", "CAN Council Events", Assumptions)) |>
  mutate(Assumptions = ifelse(Assumptions == "30 playgroups", "Playgroups", Assumptions)) |>
  mutate(Assumptions = ifelse(Assumptions == "Playgroup 20 participants", "Playgroup Participants", Assumptions)) |>
  mutate(Assumptions = ifelse(Assumptions == "9 parent education trainings", "Parent Education Trainings", Assumptions)) |>
  mutate(Assumptions = ifelse(Assumptions == "Parent Education Trainings 45 participants", "Parent Education Training Participants", Assumptions)) |>
  mutate(Assumptions = ifelse(Assumptions == "20 community provider trainings", "Community Provider Trainings", Assumptions)) |>
  mutate(Assumptions = ifelse(Assumptions == "Community Provider Trainings 160 participants", "Community Provider Trainings Participants", Assumptions)) |>
  mutate(Assumptions = ifelse(Assumptions == "# Incoming website inquiries to receptions", "Incoming Website Inquiries to Reception", Assumptions)) 
  
kpis_long <- sept_kpis_cleaned %>%
    pivot_longer(
    cols = Oct:Sept,                 
    names_to = "Month",
    values_to = "Monthly_Total"
  ) %>%
  mutate(
    Month = factor(Month, levels = c("Oct", "Nov", "Dec", "Jan", "Feb", "Mar",
                                     "Apr", "May", "Jun", "July", "Aug", "Sept"))
  )

kpis_long <- kpis_long %>%
  group_by(Assumptions) %>%
  arrange(Month) %>%
  mutate(Cumulative = cumsum(Monthly_Total))


kpis_long <- kpis_long %>%
  left_join(
    sept_kpis_cleaned %>%
      mutate(
        Assumptions = Assumptions,
        Goal = YTD / `% of Goal`
      ) %>%
      select(Assumptions, Goal),
    by = "Assumptions"
  )

kpis_long$Assumptions <- factor(
  kpis_long$Assumptions,
  levels = c(
    "Community Provider Trainings",
    "Community Provider Trainings Participants",
    "Parent Education Trainings",
    "Parent Education Training Participants",
    "Playgroups",
    "Playgroup Participants"
  )
)
```

#### Final Plot 3

``` r
ggplot(kpis_long, aes(x = Month, y = Cumulative)) +
  geom_line(aes(color = "Cumulative Total"), size = 1) +
  geom_point(aes(color = "Cumulative Total"), size = 1.5) +
  geom_hline(aes(yintercept = Goal, color = "Yearly Goal"), linetype = "dashed", size = 1) +
  facet_wrap(~ Assumptions, scales = "free_y", nrow = 3) +
  scale_color_manual(
    name = "",
    values = c("Cumulative Total" = "black", "Yearly Goal" = "darkred")
  ) +
  labs(
    title = "Monthly Progress Toward Family Services Goals",
    x = "Month",
    y = "Cumulative Total"
  ) +
  theme_minimal() +
  theme(legend.position = "bottom")
```

    ## Warning: Using `size` aesthetic for lines was deprecated in ggplot2 3.4.0.
    ## ℹ Please use `linewidth` instead.
    ## This warning is displayed once every 8 hours.
    ## Call `lifecycle::last_lifecycle_warnings()` to see where this warning was
    ## generated.

    ## `geom_line()`: Each group consists of only one observation.
    ## ℹ Do you need to adjust the group aesthetic?
    ## `geom_line()`: Each group consists of only one observation.
    ## ℹ Do you need to adjust the group aesthetic?
    ## `geom_line()`: Each group consists of only one observation.
    ## ℹ Do you need to adjust the group aesthetic?
    ## `geom_line()`: Each group consists of only one observation.
    ## ℹ Do you need to adjust the group aesthetic?
    ## `geom_line()`: Each group consists of only one observation.
    ## ℹ Do you need to adjust the group aesthetic?
    ## `geom_line()`: Each group consists of only one observation.
    ## ℹ Do you need to adjust the group aesthetic?

<img src="memo_files/figure-gfm/family-services-kpis-progress-1.png" alt="Faceted line graphs showing cumulative monthly progress from October through September across six different Family Service program goals, with each graph tracking progress toward a specific yearly goal indicated by a red dashed line. All six graphs showed that CCI met or exceeded their annual targets. Participant numbers greatly exceeded expectations, potentially due to a misrepresentation in the data (could be have been a goal per training session, not cumulative). Playgroups took the longest to meet CCIs goals."  />

### Plot 4: Whole Family Program Progress

#### Data cleanup steps specific to plot 4

``` r
sept_kpis_wholefamily <- sept_kpis %>%
  filter(Department == "Customer & Prevention Services", Program == "Whole Family")

sept_kpis_wholefamily <- sept_kpis_wholefamily[c(1,5,6), ]

sept_kpis_wholefamily <- sept_kpis_wholefamily |>
  mutate(Assumptions = ifelse(Assumptions == "# Families Newly Enrolled in Whole Family Program (month)", "Families Newly Enrolled", Assumptions)) |>
  mutate(Assumptions = ifelse(Assumptions == "# Adult Caregivers Newly Enrolled (month)", "Adult Caregivers Newly Enrolled", Assumptions)) |>
  mutate(Assumptions = ifelse(Assumptions == "# Minor Dependent Children Newly Enrolled (month)", "Minor Dependent Children Newly Enrolled", Assumptions))

kpis_wholefamily_long <- sept_kpis_wholefamily %>%
  pivot_longer(
    cols = Oct:Sept,                 
    names_to = "Month",
    values_to = "Monthly_Total"
  ) %>%
  mutate(
    Month = factor(Month, levels = c("Oct", "Nov", "Dec", "Jan", "Feb", "Mar",
                                     "Apr", "May", "Jun", "July", "Aug", "Sept"))
  )

kpis_wholefamily_long <- kpis_wholefamily_long %>%
  group_by(Assumptions) %>%
  arrange(Month) %>%
  mutate(Cumulative = cumsum(Monthly_Total))

kpis_wholefamily_long$Assumptions <- factor(
  kpis_wholefamily_long$Assumptions,
  levels = c(
    "Families Newly Enrolled",
    "Adult Caregivers Newly Enrolled",
    "Minor Dependent Children Newly Enrolled"
  )
)

endpoint_labels <- kpis_wholefamily_long %>%
  group_by(Assumptions) %>%
  filter(as.numeric(Month) == max(as.numeric(Month)))
```

#### Final Plot 4

``` r
ggplot(kpis_wholefamily_long, aes(x = Month, y = Cumulative)) +
  geom_line(color = "steelblue", size = 1) +
  geom_point(size = 1.5) +
  geom_text_repel(data = endpoint_labels,
                  aes(label = Cumulative),
                  nudge_x = 0.3,
                  size = 4,
                  color = "blue") +
  facet_wrap(~ Assumptions, nrow = 3) +
  labs(
    title = "Monthly Progress for Whole Family Services Program",
    subtitle = "Blue Number Represents Cumulative Yearly Total",
    x = "Month",
    y = "Cumulative Total"
  ) +
  theme_minimal()
```

    ## `geom_line()`: Each group consists of only one observation.
    ## ℹ Do you need to adjust the group aesthetic?
    ## `geom_line()`: Each group consists of only one observation.
    ## ℹ Do you need to adjust the group aesthetic?
    ## `geom_line()`: Each group consists of only one observation.
    ## ℹ Do you need to adjust the group aesthetic?

<img src="memo_files/figure-gfm/whole-family-program-progress-1.png" alt="Line graphs that shows the monthly progress of the Whole Family Services Program, tracking the cumulative number of families, adult caregivers, and minor dependent children newly enrolled from October through September. Each group’s progress is shown on a separate line graph with black dots marking the monthly cumulative total. By the end of the period of the fiscal year, 170 families, 200 adult caregivers, and 176 minor dependent children were newly enrolled, as highlighted by blue numbers next to each final data point. The chart emphasizes steady growth over time, with noticeable increases in enrollment during the spring and summer months."  />
