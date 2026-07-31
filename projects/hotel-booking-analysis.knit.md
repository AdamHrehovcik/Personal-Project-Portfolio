---
title: "Hotel Booking Cancellation & Revenue Analysis"
description: "Identifying what drives cancellations and how booking behaviour, pricing, and stay patterns differ by country, to inform country-specific hotel marketing strategy."
date: 2026-02-03
categories: [Report, Code, Academic, R, tidyverse, Data Visualisation]
image: images/hotel-analysis-thumb.png   # TODO: add a screenshot/chart export, ~800x450px
---

*This page is the original R source, re-executed on every site build — not a
static transcription. Data loads live from the public
[TidyTuesday hotel bookings dataset](https://github.com/rfordatascience/tidytuesday/blob/main/data/2020/2020-02-11/readme.md)
(City Hotel & Resort Hotel bookings, Portugal, 2015-2017).*

## The problem

Which factors actually drive booking cancellations, and how does customer
behaviour — lead time, stay length, pricing sensitivity — differ across a
hotel's key source markets? The goal: turn that into concrete, country-level
marketing recommendations rather than a generic dashboard.


::: {.cell}

```{.r .cell-code}
library(tidyverse)
data <- read_csv("https://raw.githubusercontent.com/rfordatascience/tidytuesday/master/data/2020/2020-02-11/hotels.csv")
```
:::


## Cancellation drivers

### Cancellation rate by hotel


::: {.cell}

```{.r .cell-code}
hotel_names = distinct(data, hotel)

city_hotel = filter(data, hotel == "City Hotel")
resort_hotel = filter(data, hotel == "Resort Hotel")
cancel_rate_city = mean(city_hotel$is_canceled, na.rm = T)
cancel_rate_res = mean(resort_hotel$is_canceled, na.rm = T)

print(paste("City Hotel cancelation rate:", round(cancel_rate_city, 3)))
```

::: {.cell-output .cell-output-stdout}

```
[1] "City Hotel cancelation rate: 0.417"
```


:::

```{.r .cell-code}
print(paste("Resort Hotel cancelation rate:", round(cancel_rate_res, 3)))
```

::: {.cell-output .cell-output-stdout}

```
[1] "Resort Hotel cancelation rate: 0.278"
```


:::
:::


The City Hotel has a cancellation rate of 41.7%, compared to 27.8% for the Resort Hotel, meaning that City Hotel is 1.5 times more likely to have its booking cancelled.
This difference could reflect the possibility that the two hotels attract different customer segments. City Hotel bookings may be more exposed to short-notice changes, for example due to a higher share of business or short-stay travel, whereas Resort Hotel bookings may be more leisure-oriented and planned further in advance.

### Lead time vs. cancellation


::: {.cell}

```{.r .cell-code}
cor(data$lead_time, data$is_canceled)
```

::: {.cell-output .cell-output-stdout}

```
[1] 0.2931234
```


:::

```{.r .cell-code}
data %>% group_by(is_canceled) %>%
  summarise(lead_by_cancelation = mean(lead_time))
```

::: {.cell-output .cell-output-stdout}

```
# A tibble: 2 × 2
  is_canceled lead_by_cancelation
        <dbl>               <dbl>
1           0                80.0
2           1               145. 
```


:::
:::



::: {.cell}

```{.r .cell-code}
ggplot(data, aes(x = factor(is_canceled, labels = c("Not Cancelled", "Cancelled")), y = lead_time)) +
  geom_boxplot() +
  labs(title = "Lead Time Distribution by Cancellation Status",
       x = "Cancelation Status",
       y = "Lead Time (days)")
```

::: {.cell-output-display}
![](hotel-booking-analysis_files/figure-html/unnamed-chunk-4-1.png){width=672}
:::
:::


The correlation coefficient between lead time and cancellation status is 0.293, indicating moderate linear association between the two variables. This therefore suggests that longer lead times tend to be associated with a higher probability of cancellation.
This relationship is also visible in the summary statistics and the boxplot. Cancelled bookings have a substantially higher mean lead time than non-cancelled bookings, and the entire distribution of lead times for cancelled bookings is shifted toward higher values.
Overall, the evidence suggests that longer lead times are associated with increased cancellation likelihood. This relationship makes intuitive sense, as customers who book far in advance have more time for their circumstances to change, increasing chance of cancellation.

### Special requests vs. cancellation


::: {.cell}

```{.r .cell-code}
data = data %>% mutate(has_special_req = case_when(total_of_special_requests > 0 ~ 1, TRUE ~ 0))

data %>% group_by(has_special_req) %>%
  summarise(cancel_rate = mean(is_canceled, na.rm = T))
```

::: {.cell-output .cell-output-stdout}

```
# A tibble: 2 × 2
  has_special_req cancel_rate
            <dbl>       <dbl>
1               0       0.477
2               1       0.217
```


:::
:::



::: {.cell}

```{.r .cell-code}
data %>% group_by(total_of_special_requests) %>%
  summarise(cancel_rate = mean(is_canceled, na.rm = T)) %>%
  ggplot(aes(x = total_of_special_requests, y = cancel_rate, fill = total_of_special_requests)) +
  geom_col() +
  labs(title = "Cancelation Rate by Number of Special Requests",
       x = "Number of  Special Requests",
       y = "Cancelation Rate")
```

::: {.cell-output-display}
![](hotel-booking-analysis_files/figure-html/unnamed-chunk-6-1.png){width=672}
:::
:::


Bookings with at least one special request have a cancellation rate of 21.7%, compared to 47.7% for bookings without any special requests. This indicates that bookings involving special requests are less likely to be cancelled.
The visualisation further shows a downward trend in cancellation rates as the number of special requests increases. This pattern suggests that customers who make special requests are likely to demonstrate higher commitment to their bookings, since they have invested additional effort in tailoring their stay.

### Booking changes vs. cancellation


::: {.cell}

```{.r .cell-code}
data = data %>% mutate(changed_booking = case_when(booking_changes > 0 ~ 1, TRUE ~ 0))

data %>% group_by(changed_booking) %>%
  summarise(cancel_rate = mean(is_canceled, na.rm = T))
```

::: {.cell-output .cell-output-stdout}

```
# A tibble: 2 × 2
  changed_booking cancel_rate
            <dbl>       <dbl>
1               0       0.409
2               1       0.157
```


:::
:::



::: {.cell}

```{.r .cell-code}
data %>% group_by(booking_changes) %>%
  summarise(cancel_rate = mean(is_canceled, na.rm = T), count = n()) %>%
  ggplot(aes(x = booking_changes, y = cancel_rate, fill = booking_changes)) +
  geom_col() +
  geom_text(aes(label = count), angle = 90, vjust = 0.5, hjust = -0.2, size = 3) +
  labs(title = "Cancelation Rate by Booking Changes",
       x = "Number of Booking Changes",
       y = "Cancelation Rate")
```

::: {.cell-output-display}
![](hotel-booking-analysis_files/figure-html/unnamed-chunk-8-1.png){width=672}
:::
:::


Customers who modify their bookings are less likely to cancel. Bookings with at least one change have a cancellation rate of only 15.7%, compared to 40.9% for bookings with no modifications.
The visualization reveals that cancellation rates generally decrease as the number of booking changes increases, particularly for bookings with 1-10 modifications. It should be noted that there is some variability in cancellation rates at very high numbers of booking changes, but this likely reflects the small sample sizes for these booking categories.
Overall, the observed pattern has a clear interpretation: customers who take time to modify their bookings demonstrate active engagement and commitment to their travel plans, lowering the likelihood of cancellation.

## Pricing, revenue & seasonality

### Monthly average daily rate


::: {.cell}

```{.r .cell-code}
library(glue)
data = data %>% mutate(arrival_month = ym(glue("{arrival_date_year} {arrival_date_month}")))%>%
    relocate(arrival_month)
data %>% group_by(arrival_month, hotel) %>%
  summarise(monthly_adr = mean(adr, na.rm = T)) %>%
  ggplot(aes(x = arrival_month, y = monthly_adr)) +
  facet_wrap(~hotel, ncol = 1) +
  geom_line() +
  labs(title = "Monthly Average Daily Rate",
       x = "Month",
       y = "Average ADR ($)")
```

::: {.cell-output .cell-output-stderr}

```
`summarise()` has grouped output by 'arrival_month'. You can override using the
`.groups` argument.
```


:::

::: {.cell-output-display}
![](hotel-booking-analysis_files/figure-html/unnamed-chunk-9-1.png){width=672}
:::
:::


For both hotels, average daily rates (ADRs) tend to peak during the summer months, particularly in August, and reach their lowest levels during the winter months, notably in November and January.
However, the magnitude of seasonal variation differs substantially between the hotels. City Hotel exhibits relatively stable ADRs throughout the year, suggesting more consistent pricing and demand across seasons. In contrast, Resort Hotel shows pronounced seasonal fluctuations, with sharp increases leading into the summer period and steep declines thereafter, indicating stronger seasonality in leisure-driven demand for the Resort Hotel.

Another key observation is that the City Hotel ADR has been steadily increasing through the sample, reflected in consistent year-on-year ADR increases. This could be related to strengthening demand, improved pricing power, or changes in the hotel's positioning over time.

### ADR vs. length of stay


::: {.cell}

```{.r .cell-code}
library(patchwork)
data = data %>% mutate(nights_stayed = stays_in_week_nights + stays_in_weekend_nights)

p1 = data %>% group_by(nights_stayed) %>%
  summarise(avg_adr = mean(adr, na.rm = T)) %>%
  ggplot(aes(x = nights_stayed, y = avg_adr)) +
  geom_col(width = 0.7) +
   labs(title = "Average ADR by Stay Length",
        x = "Nights Stayed",
        y = "Average ADR") +
  theme_minimal()

p2 = data %>% filter(nights_stayed <= 30) %>%
  group_by(nights_stayed) %>%
  ggplot(aes(x = factor(nights_stayed), y = adr)) +
  geom_boxplot() +
  scale_x_discrete(breaks = seq(0, 30, by = 5)) +
  scale_y_continuous(limits = c(0, 200)) +
  labs(title = "ADR Distribution by Stay Length",
       x = "Nights Stayed",
       y = "ADR") +
  theme_minimal()

p1 + p2
```

::: {.cell-output .cell-output-stderr}

```
Warning: Removed 4930 rows containing non-finite outside the scale range
(`stat_boxplot()`).
```


:::

::: {.cell-output-display}
![](hotel-booking-analysis_files/figure-html/unnamed-chunk-10-1.png){width=672}
:::
:::


The relationship between length of stay and ADR appears to be non-linear. As stay length increases from one night to approximately seven nights, average daily rates tend to rise. Beyond this point, further increases in stay length are associated with declining ADRs.
This pattern is consistent with common pricing strategies where short stays capture higher per-night willingness to pay, while longer stays are incentivised through discounted average daily rates.
Overall, the analysis suggests that longer stays are associated with lower per-night prices, particularly for stays exceeding one week.

### Total monthly revenue


::: {.cell}

```{.r .cell-code}
data %>% filter(is_canceled == 0, nights_stayed > 0) %>%
  mutate(booking_revenue = adr * nights_stayed) %>%
  group_by(arrival_month, hotel) %>%
  summarise(total_revenue = sum(booking_revenue)) %>%
  ggplot(aes(x = arrival_month, y = total_revenue)) +
  geom_line() +
  facet_wrap(~ hotel, ncol = 1) +
  labs(title = "Total Monthly Revenues by Hotel",
       x = "Month",
       y = "Total Revenue")
```

::: {.cell-output .cell-output-stderr}

```
`summarise()` has grouped output by 'arrival_month'. You can override using the
`.groups` argument.
```


:::

::: {.cell-output-display}
![](hotel-booking-analysis_files/figure-html/unnamed-chunk-11-1.png){width=672}
:::
:::


Total monthly revenues for both hotels exhibit strong seasonal patterns. Revenues peak during the summer months, with August being the highest-revenue month on average across the sample period for both hotels. Conversely, revenues are lowest around January, indicating weaker demand during the winter season.

However, the revenue profiles differ notably between the two hotels. Resort Hotel's revenues are substantially more volatile across the year, with much sharper increases in the lead-up to summer and more pronounced declines in autumn. In contrast, City Hotel revenues are more evenly distributed across months, suggesting lower sensitivity to seasonal demand fluctuations.

One notable anomaly in the data is that in 2015, City Hotel had the highest revenue in September instead of August. This could have been caused by some temporary factors early in the sample period, but the underlying cause cannot be identified from the available data.

## Booking geography

### Top source markets


::: {.cell}

```{.r .cell-code}
data %>% group_by(country) %>%
  summarise(n_bookings = n(), avg_cancel_rate = mean(is_canceled, na.rm = T)) %>%
  arrange(desc(n_bookings)) %>%
  slice_head(n = 5) %>%
  ggplot(aes(x = country, y = n_bookings, fill = avg_cancel_rate)) +
  geom_col() +
  geom_text(aes(label = scales::percent(avg_cancel_rate, accuracy = 0.1)), vjust = -0.5, size = 3.5) +
  labs(title = "Countries with the Highest Number of Bookings by Cancelation Rates",
       x = "Country",
       y = "Number of Bookings")
```

::: {.cell-output-display}
![](hotel-booking-analysis_files/figure-html/unnamed-chunk-12-1.png){width=672}
:::
:::


The five countries with the highest number of bookings are Portugal, Great Britain, France, Spain, and Germany, with Portugal accounting for a substantially larger share of bookings than the other countries.
However, there are also large differences in cancellation rates: Portugal has more than 55% of its bookings cancelled, followed by Spain at approximately 25%. This means the number of realised stays from Portugal is closer to the number of realised stays from countries such as Great Britain or France, but still dominates these countries.

### Market segment composition


::: {.cell}

```{.r .cell-code}
ggplot(data, aes(x = hotel, fill = market_segment)) +
  geom_bar(position = "fill") +
  scale_y_continuous(labels = scales::percent) +
  labs(title = "Market Segment Composition by Hotel",
       x = "Hotel",
       y = "Percentage",
       fill = "Market Segment")
```

::: {.cell-output-display}
![](hotel-booking-analysis_files/figure-html/unnamed-chunk-13-1.png){width=672}
:::
:::


The dominant market segment for both hotels is "Online TA" with other notable segments for both hotels being "Offline TA/TO", "Groups", and "Corporate".
This pattern is consistent with the growing preference for online booking platforms such as Booking.com, Expedia, or Hotels.com during the sample period, which already dominated traditional offline travel agents at that time, due to their superior convenience and flexibility.
"Offline TA/TO" remains a secondary but non-negligible segment for both hotels, indicating that traditional travel agents still play a role in booking activity during the sample period. Direct bookings and corporate segments represent smaller proportions of total demand, which is consistent with the limited scale of business-related and direct-channel travel relative to leisure-oriented bookings.

## Country-level marketing strategy

The following analysis examines differences in booking behaviour across customer geographies with the objective of identifying insights that could inform more targeted, country-specific marketing strategies. In particular, the analysis focuses on how booking timing, length of stay, pricing, and selected guest characteristics vary across countries, and how these dimensions relate to revenue-relevant outcomes.
To ensure practical relevance, the analysis is restricted to countries that already represent a substantial share of bookings for each hotel. Rather than exploring expansion into new markets, the focus is on understanding behavioural differences within existing geographic segments, where targeted marketing interventions are more immediately actionable.
The central research question guiding this analysis is: how do booking behaviour and pricing characteristics differ across key customer geographies, and how can these differences inform the timing, positioning, and focus of hotel marketing efforts?


::: {.cell}

:::


::: {.columns}
::: {.column width="50%"}

::: {.cell}
::: {.cell-output-display}

```{=html}
<div id="lnhwcmsosh" style="padding-left:0px;padding-right:0px;padding-top:10px;padding-bottom:10px;overflow-x:auto;overflow-y:auto;width:auto;height:auto;">
<style>#lnhwcmsosh table {
  font-family: system-ui, 'Segoe UI', Roboto, Helvetica, Arial, sans-serif, 'Apple Color Emoji', 'Segoe UI Emoji', 'Segoe UI Symbol', 'Noto Color Emoji';
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
}

#lnhwcmsosh thead, #lnhwcmsosh tbody, #lnhwcmsosh tfoot, #lnhwcmsosh tr, #lnhwcmsosh td, #lnhwcmsosh th {
  border-style: none;
}

#lnhwcmsosh p {
  margin: 0;
  padding: 0;
}

#lnhwcmsosh .gt_table {
  display: table;
  border-collapse: collapse;
  line-height: normal;
  margin-left: auto;
  margin-right: auto;
  color: #333333;
  font-size: 10px;
  font-weight: normal;
  font-style: normal;
  background-color: #FFFFFF;
  width: auto;
  border-top-style: solid;
  border-top-width: 2px;
  border-top-color: #A8A8A8;
  border-right-style: none;
  border-right-width: 2px;
  border-right-color: #D3D3D3;
  border-bottom-style: solid;
  border-bottom-width: 2px;
  border-bottom-color: #A8A8A8;
  border-left-style: none;
  border-left-width: 2px;
  border-left-color: #D3D3D3;
}

#lnhwcmsosh .gt_caption {
  padding-top: 4px;
  padding-bottom: 4px;
}

#lnhwcmsosh .gt_title {
  color: #333333;
  font-size: 12px;
  font-weight: initial;
  padding-top: 4px;
  padding-bottom: 4px;
  padding-left: 5px;
  padding-right: 5px;
  border-bottom-color: #FFFFFF;
  border-bottom-width: 0;
}

#lnhwcmsosh .gt_subtitle {
  color: #333333;
  font-size: 85%;
  font-weight: initial;
  padding-top: 3px;
  padding-bottom: 5px;
  padding-left: 5px;
  padding-right: 5px;
  border-top-color: #FFFFFF;
  border-top-width: 0;
}

#lnhwcmsosh .gt_heading {
  background-color: #FFFFFF;
  text-align: center;
  border-bottom-color: #FFFFFF;
  border-left-style: none;
  border-left-width: 1px;
  border-left-color: #D3D3D3;
  border-right-style: none;
  border-right-width: 1px;
  border-right-color: #D3D3D3;
}

#lnhwcmsosh .gt_bottom_border {
  border-bottom-style: solid;
  border-bottom-width: 2px;
  border-bottom-color: #D3D3D3;
}

#lnhwcmsosh .gt_col_headings {
  border-top-style: solid;
  border-top-width: 2px;
  border-top-color: #D3D3D3;
  border-bottom-style: solid;
  border-bottom-width: 2px;
  border-bottom-color: #D3D3D3;
  border-left-style: none;
  border-left-width: 1px;
  border-left-color: #D3D3D3;
  border-right-style: none;
  border-right-width: 1px;
  border-right-color: #D3D3D3;
}

#lnhwcmsosh .gt_col_heading {
  color: #333333;
  background-color: #FFFFFF;
  font-size: 10px;
  font-weight: normal;
  text-transform: inherit;
  border-left-style: none;
  border-left-width: 1px;
  border-left-color: #D3D3D3;
  border-right-style: none;
  border-right-width: 1px;
  border-right-color: #D3D3D3;
  vertical-align: bottom;
  padding-top: 5px;
  padding-bottom: 6px;
  padding-left: 5px;
  padding-right: 5px;
  overflow-x: hidden;
}

#lnhwcmsosh .gt_column_spanner_outer {
  color: #333333;
  background-color: #FFFFFF;
  font-size: 10px;
  font-weight: normal;
  text-transform: inherit;
  padding-top: 0;
  padding-bottom: 0;
  padding-left: 4px;
  padding-right: 4px;
}

#lnhwcmsosh .gt_column_spanner_outer:first-child {
  padding-left: 0;
}

#lnhwcmsosh .gt_column_spanner_outer:last-child {
  padding-right: 0;
}

#lnhwcmsosh .gt_column_spanner {
  border-bottom-style: solid;
  border-bottom-width: 2px;
  border-bottom-color: #D3D3D3;
  vertical-align: bottom;
  padding-top: 5px;
  padding-bottom: 5px;
  overflow-x: hidden;
  display: inline-block;
  width: 100%;
}

#lnhwcmsosh .gt_spanner_row {
  border-bottom-style: hidden;
}

#lnhwcmsosh .gt_group_heading {
  padding-top: 8px;
  padding-bottom: 8px;
  padding-left: 5px;
  padding-right: 5px;
  color: #333333;
  background-color: #FFFFFF;
  font-size: 100%;
  font-weight: initial;
  text-transform: inherit;
  border-top-style: solid;
  border-top-width: 2px;
  border-top-color: #D3D3D3;
  border-bottom-style: solid;
  border-bottom-width: 2px;
  border-bottom-color: #D3D3D3;
  border-left-style: none;
  border-left-width: 1px;
  border-left-color: #D3D3D3;
  border-right-style: none;
  border-right-width: 1px;
  border-right-color: #D3D3D3;
  vertical-align: middle;
  text-align: left;
}

#lnhwcmsosh .gt_empty_group_heading {
  padding: 0.5px;
  color: #333333;
  background-color: #FFFFFF;
  font-size: 100%;
  font-weight: initial;
  border-top-style: solid;
  border-top-width: 2px;
  border-top-color: #D3D3D3;
  border-bottom-style: solid;
  border-bottom-width: 2px;
  border-bottom-color: #D3D3D3;
  vertical-align: middle;
}

#lnhwcmsosh .gt_from_md > :first-child {
  margin-top: 0;
}

#lnhwcmsosh .gt_from_md > :last-child {
  margin-bottom: 0;
}

#lnhwcmsosh .gt_row {
  padding-top: 8px;
  padding-bottom: 8px;
  padding-left: 5px;
  padding-right: 5px;
  margin: 10px;
  border-top-style: solid;
  border-top-width: 0px;
  border-top-color: #D3D3D3;
  border-left-style: none;
  border-left-width: 0px;
  border-left-color: #D3D3D3;
  border-right-style: none;
  border-right-width: 0px;
  border-right-color: #D3D3D3;
  vertical-align: middle;
  overflow-x: hidden;
}

#lnhwcmsosh .gt_stub {
  color: #333333;
  background-color: #FFFFFF;
  font-size: 100%;
  font-weight: initial;
  text-transform: inherit;
  border-right-style: solid;
  border-right-width: 2px;
  border-right-color: #D3D3D3;
  padding-left: 5px;
  padding-right: 5px;
}

#lnhwcmsosh .gt_stub_row_group {
  color: #333333;
  background-color: #FFFFFF;
  font-size: 100%;
  font-weight: initial;
  text-transform: inherit;
  border-right-style: solid;
  border-right-width: 2px;
  border-right-color: #D3D3D3;
  padding-left: 5px;
  padding-right: 5px;
  vertical-align: top;
}

#lnhwcmsosh .gt_row_group_first td {
  border-top-width: 2px;
}

#lnhwcmsosh .gt_row_group_first th {
  border-top-width: 2px;
}

#lnhwcmsosh .gt_summary_row {
  color: #333333;
  background-color: #FFFFFF;
  text-transform: inherit;
  padding-top: 8px;
  padding-bottom: 8px;
  padding-left: 5px;
  padding-right: 5px;
}

#lnhwcmsosh .gt_first_summary_row {
  border-top-style: solid;
  border-top-color: #D3D3D3;
}

#lnhwcmsosh .gt_first_summary_row.thick {
  border-top-width: 2px;
}

#lnhwcmsosh .gt_last_summary_row {
  padding-top: 8px;
  padding-bottom: 8px;
  padding-left: 5px;
  padding-right: 5px;
  border-bottom-style: solid;
  border-bottom-width: 2px;
  border-bottom-color: #D3D3D3;
}

#lnhwcmsosh .gt_grand_summary_row {
  color: #333333;
  background-color: #FFFFFF;
  text-transform: inherit;
  padding-top: 8px;
  padding-bottom: 8px;
  padding-left: 5px;
  padding-right: 5px;
}

#lnhwcmsosh .gt_first_grand_summary_row {
  padding-top: 8px;
  padding-bottom: 8px;
  padding-left: 5px;
  padding-right: 5px;
  border-top-style: double;
  border-top-width: 6px;
  border-top-color: #D3D3D3;
}

#lnhwcmsosh .gt_last_grand_summary_row_top {
  padding-top: 8px;
  padding-bottom: 8px;
  padding-left: 5px;
  padding-right: 5px;
  border-bottom-style: double;
  border-bottom-width: 6px;
  border-bottom-color: #D3D3D3;
}

#lnhwcmsosh .gt_striped {
  background-color: rgba(128, 128, 128, 0.05);
}

#lnhwcmsosh .gt_table_body {
  border-top-style: solid;
  border-top-width: 2px;
  border-top-color: #D3D3D3;
  border-bottom-style: solid;
  border-bottom-width: 2px;
  border-bottom-color: #D3D3D3;
}

#lnhwcmsosh .gt_footnotes {
  color: #333333;
  background-color: #FFFFFF;
  border-bottom-style: none;
  border-bottom-width: 2px;
  border-bottom-color: #D3D3D3;
  border-left-style: none;
  border-left-width: 2px;
  border-left-color: #D3D3D3;
  border-right-style: none;
  border-right-width: 2px;
  border-right-color: #D3D3D3;
}

#lnhwcmsosh .gt_footnote {
  margin: 0px;
  font-size: 90%;
  padding-top: 4px;
  padding-bottom: 4px;
  padding-left: 5px;
  padding-right: 5px;
}

#lnhwcmsosh .gt_sourcenotes {
  color: #333333;
  background-color: #FFFFFF;
  border-bottom-style: none;
  border-bottom-width: 2px;
  border-bottom-color: #D3D3D3;
  border-left-style: none;
  border-left-width: 2px;
  border-left-color: #D3D3D3;
  border-right-style: none;
  border-right-width: 2px;
  border-right-color: #D3D3D3;
}

#lnhwcmsosh .gt_sourcenote {
  font-size: 90%;
  padding-top: 4px;
  padding-bottom: 4px;
  padding-left: 5px;
  padding-right: 5px;
}

#lnhwcmsosh .gt_left {
  text-align: left;
}

#lnhwcmsosh .gt_center {
  text-align: center;
}

#lnhwcmsosh .gt_right {
  text-align: right;
  font-variant-numeric: tabular-nums;
}

#lnhwcmsosh .gt_font_normal {
  font-weight: normal;
}

#lnhwcmsosh .gt_font_bold {
  font-weight: bold;
}

#lnhwcmsosh .gt_font_italic {
  font-style: italic;
}

#lnhwcmsosh .gt_super {
  font-size: 65%;
}

#lnhwcmsosh .gt_footnote_marks {
  font-size: 75%;
  vertical-align: 0.4em;
  position: initial;
}

#lnhwcmsosh .gt_asterisk {
  font-size: 100%;
  vertical-align: 0;
}

#lnhwcmsosh .gt_indent_1 {
  text-indent: 5px;
}

#lnhwcmsosh .gt_indent_2 {
  text-indent: 10px;
}

#lnhwcmsosh .gt_indent_3 {
  text-indent: 15px;
}

#lnhwcmsosh .gt_indent_4 {
  text-indent: 20px;
}

#lnhwcmsosh .gt_indent_5 {
  text-indent: 25px;
}

#lnhwcmsosh .katex-display {
  display: inline-flex !important;
  margin-bottom: 0.75em !important;
}

#lnhwcmsosh div.Reactable > div.rt-table > div.rt-thead > div.rt-tr.rt-tr-group-header > div.rt-th-group:after {
  height: 0px !important;
}
</style>
<table class="gt_table" data-quarto-disable-processing="false" data-quarto-bootstrap="false">
  <thead>
    <tr class="gt_heading">
      <td colspan="3" class="gt_heading gt_title gt_font_normal gt_bottom_border" style>City Hotel - Top Countries</td>
    </tr>
    
    <tr class="gt_col_headings">
      <th class="gt_col_heading gt_columns_bottom_border gt_left" rowspan="1" colspan="1" scope="col" id="country">country</th>
      <th class="gt_col_heading gt_columns_bottom_border gt_right" rowspan="1" colspan="1" scope="col" id="n_bookings">n_bookings</th>
      <th class="gt_col_heading gt_columns_bottom_border gt_right" rowspan="1" colspan="1" scope="col" id="perc_of_total">perc_of_total</th>
    </tr>
  </thead>
  <tbody class="gt_table_body">
    <tr><td headers="country" class="gt_row gt_left">PRT</td>
<td headers="n_bookings" class="gt_row gt_right">30,960</td>
<td headers="perc_of_total" class="gt_row gt_right">39.0%</td></tr>
    <tr><td headers="country" class="gt_row gt_left">FRA</td>
<td headers="n_bookings" class="gt_row gt_right">8,804</td>
<td headers="perc_of_total" class="gt_row gt_right">11.1%</td></tr>
    <tr><td headers="country" class="gt_row gt_left">DEU</td>
<td headers="n_bookings" class="gt_row gt_right">6,084</td>
<td headers="perc_of_total" class="gt_row gt_right">7.7%</td></tr>
    <tr><td headers="country" class="gt_row gt_left">GBR</td>
<td headers="n_bookings" class="gt_row gt_right">5,315</td>
<td headers="perc_of_total" class="gt_row gt_right">6.7%</td></tr>
    <tr><td headers="country" class="gt_row gt_left">ESP</td>
<td headers="n_bookings" class="gt_row gt_right">4,611</td>
<td headers="perc_of_total" class="gt_row gt_right">5.8%</td></tr>
    <tr><td headers="country" class="gt_row gt_left">ITA</td>
<td headers="n_bookings" class="gt_row gt_right">3,307</td>
<td headers="perc_of_total" class="gt_row gt_right">4.2%</td></tr>
    <tr><td headers="country" class="gt_row gt_left">BEL</td>
<td headers="n_bookings" class="gt_row gt_right">1,894</td>
<td headers="perc_of_total" class="gt_row gt_right">2.4%</td></tr>
  </tbody>
  
</table>
</div>
```

:::
:::

:::
::: {.column width="50%"}

::: {.cell}
::: {.cell-output-display}

```{=html}
<div id="vrxofpactj" style="padding-left:0px;padding-right:0px;padding-top:10px;padding-bottom:10px;overflow-x:auto;overflow-y:auto;width:auto;height:auto;">
<style>#vrxofpactj table {
  font-family: system-ui, 'Segoe UI', Roboto, Helvetica, Arial, sans-serif, 'Apple Color Emoji', 'Segoe UI Emoji', 'Segoe UI Symbol', 'Noto Color Emoji';
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
}

#vrxofpactj thead, #vrxofpactj tbody, #vrxofpactj tfoot, #vrxofpactj tr, #vrxofpactj td, #vrxofpactj th {
  border-style: none;
}

#vrxofpactj p {
  margin: 0;
  padding: 0;
}

#vrxofpactj .gt_table {
  display: table;
  border-collapse: collapse;
  line-height: normal;
  margin-left: auto;
  margin-right: auto;
  color: #333333;
  font-size: 10px;
  font-weight: normal;
  font-style: normal;
  background-color: #FFFFFF;
  width: auto;
  border-top-style: solid;
  border-top-width: 2px;
  border-top-color: #A8A8A8;
  border-right-style: none;
  border-right-width: 2px;
  border-right-color: #D3D3D3;
  border-bottom-style: solid;
  border-bottom-width: 2px;
  border-bottom-color: #A8A8A8;
  border-left-style: none;
  border-left-width: 2px;
  border-left-color: #D3D3D3;
}

#vrxofpactj .gt_caption {
  padding-top: 4px;
  padding-bottom: 4px;
}

#vrxofpactj .gt_title {
  color: #333333;
  font-size: 12px;
  font-weight: initial;
  padding-top: 4px;
  padding-bottom: 4px;
  padding-left: 5px;
  padding-right: 5px;
  border-bottom-color: #FFFFFF;
  border-bottom-width: 0;
}

#vrxofpactj .gt_subtitle {
  color: #333333;
  font-size: 85%;
  font-weight: initial;
  padding-top: 3px;
  padding-bottom: 5px;
  padding-left: 5px;
  padding-right: 5px;
  border-top-color: #FFFFFF;
  border-top-width: 0;
}

#vrxofpactj .gt_heading {
  background-color: #FFFFFF;
  text-align: center;
  border-bottom-color: #FFFFFF;
  border-left-style: none;
  border-left-width: 1px;
  border-left-color: #D3D3D3;
  border-right-style: none;
  border-right-width: 1px;
  border-right-color: #D3D3D3;
}

#vrxofpactj .gt_bottom_border {
  border-bottom-style: solid;
  border-bottom-width: 2px;
  border-bottom-color: #D3D3D3;
}

#vrxofpactj .gt_col_headings {
  border-top-style: solid;
  border-top-width: 2px;
  border-top-color: #D3D3D3;
  border-bottom-style: solid;
  border-bottom-width: 2px;
  border-bottom-color: #D3D3D3;
  border-left-style: none;
  border-left-width: 1px;
  border-left-color: #D3D3D3;
  border-right-style: none;
  border-right-width: 1px;
  border-right-color: #D3D3D3;
}

#vrxofpactj .gt_col_heading {
  color: #333333;
  background-color: #FFFFFF;
  font-size: 10px;
  font-weight: normal;
  text-transform: inherit;
  border-left-style: none;
  border-left-width: 1px;
  border-left-color: #D3D3D3;
  border-right-style: none;
  border-right-width: 1px;
  border-right-color: #D3D3D3;
  vertical-align: bottom;
  padding-top: 5px;
  padding-bottom: 6px;
  padding-left: 5px;
  padding-right: 5px;
  overflow-x: hidden;
}

#vrxofpactj .gt_column_spanner_outer {
  color: #333333;
  background-color: #FFFFFF;
  font-size: 10px;
  font-weight: normal;
  text-transform: inherit;
  padding-top: 0;
  padding-bottom: 0;
  padding-left: 4px;
  padding-right: 4px;
}

#vrxofpactj .gt_column_spanner_outer:first-child {
  padding-left: 0;
}

#vrxofpactj .gt_column_spanner_outer:last-child {
  padding-right: 0;
}

#vrxofpactj .gt_column_spanner {
  border-bottom-style: solid;
  border-bottom-width: 2px;
  border-bottom-color: #D3D3D3;
  vertical-align: bottom;
  padding-top: 5px;
  padding-bottom: 5px;
  overflow-x: hidden;
  display: inline-block;
  width: 100%;
}

#vrxofpactj .gt_spanner_row {
  border-bottom-style: hidden;
}

#vrxofpactj .gt_group_heading {
  padding-top: 8px;
  padding-bottom: 8px;
  padding-left: 5px;
  padding-right: 5px;
  color: #333333;
  background-color: #FFFFFF;
  font-size: 100%;
  font-weight: initial;
  text-transform: inherit;
  border-top-style: solid;
  border-top-width: 2px;
  border-top-color: #D3D3D3;
  border-bottom-style: solid;
  border-bottom-width: 2px;
  border-bottom-color: #D3D3D3;
  border-left-style: none;
  border-left-width: 1px;
  border-left-color: #D3D3D3;
  border-right-style: none;
  border-right-width: 1px;
  border-right-color: #D3D3D3;
  vertical-align: middle;
  text-align: left;
}

#vrxofpactj .gt_empty_group_heading {
  padding: 0.5px;
  color: #333333;
  background-color: #FFFFFF;
  font-size: 100%;
  font-weight: initial;
  border-top-style: solid;
  border-top-width: 2px;
  border-top-color: #D3D3D3;
  border-bottom-style: solid;
  border-bottom-width: 2px;
  border-bottom-color: #D3D3D3;
  vertical-align: middle;
}

#vrxofpactj .gt_from_md > :first-child {
  margin-top: 0;
}

#vrxofpactj .gt_from_md > :last-child {
  margin-bottom: 0;
}

#vrxofpactj .gt_row {
  padding-top: 8px;
  padding-bottom: 8px;
  padding-left: 5px;
  padding-right: 5px;
  margin: 10px;
  border-top-style: solid;
  border-top-width: 0px;
  border-top-color: #D3D3D3;
  border-left-style: none;
  border-left-width: 0px;
  border-left-color: #D3D3D3;
  border-right-style: none;
  border-right-width: 0px;
  border-right-color: #D3D3D3;
  vertical-align: middle;
  overflow-x: hidden;
}

#vrxofpactj .gt_stub {
  color: #333333;
  background-color: #FFFFFF;
  font-size: 100%;
  font-weight: initial;
  text-transform: inherit;
  border-right-style: solid;
  border-right-width: 2px;
  border-right-color: #D3D3D3;
  padding-left: 5px;
  padding-right: 5px;
}

#vrxofpactj .gt_stub_row_group {
  color: #333333;
  background-color: #FFFFFF;
  font-size: 100%;
  font-weight: initial;
  text-transform: inherit;
  border-right-style: solid;
  border-right-width: 2px;
  border-right-color: #D3D3D3;
  padding-left: 5px;
  padding-right: 5px;
  vertical-align: top;
}

#vrxofpactj .gt_row_group_first td {
  border-top-width: 2px;
}

#vrxofpactj .gt_row_group_first th {
  border-top-width: 2px;
}

#vrxofpactj .gt_summary_row {
  color: #333333;
  background-color: #FFFFFF;
  text-transform: inherit;
  padding-top: 8px;
  padding-bottom: 8px;
  padding-left: 5px;
  padding-right: 5px;
}

#vrxofpactj .gt_first_summary_row {
  border-top-style: solid;
  border-top-color: #D3D3D3;
}

#vrxofpactj .gt_first_summary_row.thick {
  border-top-width: 2px;
}

#vrxofpactj .gt_last_summary_row {
  padding-top: 8px;
  padding-bottom: 8px;
  padding-left: 5px;
  padding-right: 5px;
  border-bottom-style: solid;
  border-bottom-width: 2px;
  border-bottom-color: #D3D3D3;
}

#vrxofpactj .gt_grand_summary_row {
  color: #333333;
  background-color: #FFFFFF;
  text-transform: inherit;
  padding-top: 8px;
  padding-bottom: 8px;
  padding-left: 5px;
  padding-right: 5px;
}

#vrxofpactj .gt_first_grand_summary_row {
  padding-top: 8px;
  padding-bottom: 8px;
  padding-left: 5px;
  padding-right: 5px;
  border-top-style: double;
  border-top-width: 6px;
  border-top-color: #D3D3D3;
}

#vrxofpactj .gt_last_grand_summary_row_top {
  padding-top: 8px;
  padding-bottom: 8px;
  padding-left: 5px;
  padding-right: 5px;
  border-bottom-style: double;
  border-bottom-width: 6px;
  border-bottom-color: #D3D3D3;
}

#vrxofpactj .gt_striped {
  background-color: rgba(128, 128, 128, 0.05);
}

#vrxofpactj .gt_table_body {
  border-top-style: solid;
  border-top-width: 2px;
  border-top-color: #D3D3D3;
  border-bottom-style: solid;
  border-bottom-width: 2px;
  border-bottom-color: #D3D3D3;
}

#vrxofpactj .gt_footnotes {
  color: #333333;
  background-color: #FFFFFF;
  border-bottom-style: none;
  border-bottom-width: 2px;
  border-bottom-color: #D3D3D3;
  border-left-style: none;
  border-left-width: 2px;
  border-left-color: #D3D3D3;
  border-right-style: none;
  border-right-width: 2px;
  border-right-color: #D3D3D3;
}

#vrxofpactj .gt_footnote {
  margin: 0px;
  font-size: 90%;
  padding-top: 4px;
  padding-bottom: 4px;
  padding-left: 5px;
  padding-right: 5px;
}

#vrxofpactj .gt_sourcenotes {
  color: #333333;
  background-color: #FFFFFF;
  border-bottom-style: none;
  border-bottom-width: 2px;
  border-bottom-color: #D3D3D3;
  border-left-style: none;
  border-left-width: 2px;
  border-left-color: #D3D3D3;
  border-right-style: none;
  border-right-width: 2px;
  border-right-color: #D3D3D3;
}

#vrxofpactj .gt_sourcenote {
  font-size: 90%;
  padding-top: 4px;
  padding-bottom: 4px;
  padding-left: 5px;
  padding-right: 5px;
}

#vrxofpactj .gt_left {
  text-align: left;
}

#vrxofpactj .gt_center {
  text-align: center;
}

#vrxofpactj .gt_right {
  text-align: right;
  font-variant-numeric: tabular-nums;
}

#vrxofpactj .gt_font_normal {
  font-weight: normal;
}

#vrxofpactj .gt_font_bold {
  font-weight: bold;
}

#vrxofpactj .gt_font_italic {
  font-style: italic;
}

#vrxofpactj .gt_super {
  font-size: 65%;
}

#vrxofpactj .gt_footnote_marks {
  font-size: 75%;
  vertical-align: 0.4em;
  position: initial;
}

#vrxofpactj .gt_asterisk {
  font-size: 100%;
  vertical-align: 0;
}

#vrxofpactj .gt_indent_1 {
  text-indent: 5px;
}

#vrxofpactj .gt_indent_2 {
  text-indent: 10px;
}

#vrxofpactj .gt_indent_3 {
  text-indent: 15px;
}

#vrxofpactj .gt_indent_4 {
  text-indent: 20px;
}

#vrxofpactj .gt_indent_5 {
  text-indent: 25px;
}

#vrxofpactj .katex-display {
  display: inline-flex !important;
  margin-bottom: 0.75em !important;
}

#vrxofpactj div.Reactable > div.rt-table > div.rt-thead > div.rt-tr.rt-tr-group-header > div.rt-th-group:after {
  height: 0px !important;
}
</style>
<table class="gt_table" data-quarto-disable-processing="false" data-quarto-bootstrap="false">
  <thead>
    <tr class="gt_heading">
      <td colspan="3" class="gt_heading gt_title gt_font_normal gt_bottom_border" style>Resort Hotel - Top Countries</td>
    </tr>
    
    <tr class="gt_col_headings">
      <th class="gt_col_heading gt_columns_bottom_border gt_left" rowspan="1" colspan="1" scope="col" id="country">country</th>
      <th class="gt_col_heading gt_columns_bottom_border gt_right" rowspan="1" colspan="1" scope="col" id="n_bookings">n_bookings</th>
      <th class="gt_col_heading gt_columns_bottom_border gt_right" rowspan="1" colspan="1" scope="col" id="perc_of_total">perc_of_total</th>
    </tr>
  </thead>
  <tbody class="gt_table_body">
    <tr><td headers="country" class="gt_row gt_left">PRT</td>
<td headers="n_bookings" class="gt_row gt_right">17,630</td>
<td headers="perc_of_total" class="gt_row gt_right">44.0%</td></tr>
    <tr><td headers="country" class="gt_row gt_left">GBR</td>
<td headers="n_bookings" class="gt_row gt_right">6,814</td>
<td headers="perc_of_total" class="gt_row gt_right">17.0%</td></tr>
    <tr><td headers="country" class="gt_row gt_left">ESP</td>
<td headers="n_bookings" class="gt_row gt_right">3,957</td>
<td headers="perc_of_total" class="gt_row gt_right">9.9%</td></tr>
    <tr><td headers="country" class="gt_row gt_left">IRL</td>
<td headers="n_bookings" class="gt_row gt_right">2,166</td>
<td headers="perc_of_total" class="gt_row gt_right">5.4%</td></tr>
    <tr><td headers="country" class="gt_row gt_left">FRA</td>
<td headers="n_bookings" class="gt_row gt_right">1,611</td>
<td headers="perc_of_total" class="gt_row gt_right">4.0%</td></tr>
    <tr><td headers="country" class="gt_row gt_left">DEU</td>
<td headers="n_bookings" class="gt_row gt_right">1,203</td>
<td headers="perc_of_total" class="gt_row gt_right">3.0%</td></tr>
    <tr><td headers="country" class="gt_row gt_left">CN</td>
<td headers="n_bookings" class="gt_row gt_right">710</td>
<td headers="perc_of_total" class="gt_row gt_right">1.8%</td></tr>
  </tbody>
  
</table>
</div>
```

:::
:::

:::
:::

While the earlier country breakdown examined booking volumes, this section considers booking proportions by hotel to assess source-market composition.
Bookings for both hotels are concentrated in a small number of countries, with Portugal accounting for the largest share in each case, and the overall geographic composition being broadly similar across hotels.
Accordingly, the analysis focuses on the five largest source markets for each hotel: Portugal, France, Germany, Great Britain, and Spain for City Hotel, and Portugal, Great Britain, Spain, Ireland, and France for Resort Hotel. This ensures that subsequent insights are derived from commercially meaningful markets.


::: {.cell}
::: {.cell-output-display}
![](hotel-booking-analysis_files/figure-html/unnamed-chunk-17-1.png){width=100%}
:::
:::


Lead times vary across countries for both hotels, although most bookings occur between approximately 20 and 90 days before arrival and have a long tail of bookings made further in advance. For Resort Hotel, Spanish and Portuguese customers tend to book closer to arrival, while customers from Great Britain and Ireland exhibit longer lead times. For City Hotel, cross-country differences are less pronounced, although Spain consistently shows shorter lead times than other major markets. From a marketing perspective, these patterns suggest that promotional timing may benefit from geographic differentiation. For Resort Hotel, campaigns targeting British and Irish customers may need to be launched earlier in the booking cycle, while promotions aimed at Spanish and Portuguese customers can be scheduled closer to the peak travel period. For City Hotel, more uniform campaign timing across countries appears appropriate, with limited adjustment for Spain's shorter lead times.

Another relevant dimension for marketing design is length of stay, which is examined next to assess how typical stay duration preferences vary across countries and hotels, given its implications for offer design and pricing.


::: {.cell}
::: {.cell-output-display}
![](hotel-booking-analysis_files/figure-html/unnamed-chunk-18-1.png){width=100%}
:::
:::


For City Hotel, stays are generally short across all major source markets, with the majority of bookings lasting five nights or fewer and limited variation across countries. In contrast, Resort Hotel exhibits substantially greater dispersion in stay length. Spanish and Portuguese customers tend to book shorter stays, typically under one week, while customers from Great Britain and Ireland are more likely to book stays of one week or longer. Thus, City Hotel promotions may be more effectively oriented toward short stays and weekend trips across markets, while Resort Hotel may benefit from a broader mix of stay-duration offers, with greater emphasis on longer stays when targeting British and Irish customers.

The next key consideration is price positioning, in particular identifying which markets are more suited to higher-end, premium offers and which may respond better to more affordable stay packages. A closely related dimension is the degree of stay customisation, which is also examined in the following analysis.


::: {.cell}
::: {.cell-output-display}
![](hotel-booking-analysis_files/figure-html/unnamed-chunk-19-1.png){width=100%}
:::
:::


For both hotels, geographic segments with a higher share of bookings that include special requests also tend to exhibit higher average daily rates, indicating an association between willingness to pay and demand for more customised stays. This implies that emphasising flexibility and customisation may be more effective when targeting higher-spending customer segments.
For City Hotel, Portuguese guests exhibit the lowest average spend per night among major source markets, suggesting they may be more responsive to value-oriented offers. For Resort Hotel, Spanish guests display the highest average prices and may represent more suitable targets for premium offerings, relative to Portugal and Great Britain, where demand appears more price-sensitive.

Having examined cross-country differences in lead time, length of stay, and average daily rate, it is useful to consider how these dimensions interact and how this may inform marketing timing and positioning across markets.


::: {.cell}
::: {.cell-output-display}
![](hotel-booking-analysis_files/figure-html/unnamed-chunk-20-1.png){width=100%}
:::
:::


When examined jointly, longer stays tend to be associated with lower average daily rates, suggesting that price positioning reflects not only willingness to pay but also stay duration, particularly for extended stays that benefit from lower per-night pricing.
Lead time also varies systematically with price positioning. For both hotels, higher-priced stays are less commonly booked far in advance, while bookings made well ahead of arrival are more concentrated in lower to mid-range ADRs. This is especially pronounced for City Hotel, where higher-priced stays are rarely booked more than a year in advance.
This indicates that premium offerings may be more effectively promoted through shorter-horizon campaigns rather than early-bird promotions. Conversely, early-bird promotions may be better suited for value-oriented offers.
Finally, effective marketing design also requires consideration of guest characteristics beyond price sensitivity and stay duration. One such dimension that captures several less directly observable preferences is whether bookings include children.


::: {.cell}
::: {.cell-output-display}
![](hotel-booking-analysis_files/figure-html/unnamed-chunk-21-1.png){width=100%}
:::
:::


Across both hotels, bookings that include children tend to be made slightly further in advance than adult-only bookings, although the differences are relatively small. This suggests the timing of summer promotion campaigns does not need to differ across family and adult-only segments.
The heatmap indicates that Resort Hotel attracts a higher share of family bookings across key geographies, while City Hotel demand remains dominated by adult-only stays. In addition, Spanish customers exhibit a relatively high incidence of family travel for both hotels. This suggests that family-oriented offers and amenities may be particularly relevant when targeting the Spanish market, especially for Resort Hotel.

To conclude, booking behaviour differs across countries primarily through systematic differences in booking timing, stay duration, and price positioning. Higher-priced and more customised stays tend to be booked closer to arrival, while earlier bookings are more strongly associated with longer stays and lower average daily rates. Effective geographic targeting should align both the timing and positioning of marketing offers with observed country-specific booking patterns, rather than relying on uniform campaigns across markets.

