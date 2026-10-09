# UrbanPulse: City Operations Intelligence

Operations analytics on **real NYC 311 service requests**: how long different complaints take to close, which areas and categories are slow, where the hotspots are, whether a model can flag slow tickets early, and what the next weeks of ticket volume look like. An AI-written briefing turns the calculated numbers into a short report for a city manager.

Built in Google Colab with the NYC Open Data API and Groq (`openai/gpt-oss-120b`).

## What it does

1. **Loads real data** from NYC Open Data, sampled across 12 months: 2 days per month, 3 eight-hour windows per day, 24 days in total. Tickets from the most recent 2.5 months are skipped because many are still open, and open tickets would make response times look faster than they are. If the download fails, a made-up sample is used instead and the notebook says so.
2. **Pulls true daily ticket volumes** with a separate full-count query, so the forecast uses real volumes and not the sample.
3. **Sets an SLA threshold** (72 hours, chosen for this analysis and not an official city target) and finds the share of closed tickets that are slower.
4. **Compares categories and areas** by median response time and slow-ticket rate.
5. **Time patterns as rates only.** The sample has a fixed size per time window, so the notebook never reads ticket counts as volume.
6. **Ranks hotspots:** area and category combinations with enough tickets and a high slow rate.
7. **Predicts slow tickets** with a Random Forest and compares it with a category-only baseline.
8. **Forecasts the next 4 weeks** with a simple trend line on the full-count volumes.
9. **AI operations briefing (Groq):** told that the SLA is an analysis threshold, that response times come from a sample, and not to invent numbers or targets.

## Results from the run (real NYC 311 data)

| Measure | Result |
|---|---|
| Sample | 11,680 tickets over 24 days across 12 months; 11,547 closed (98.9%) |
| Closed tickets slower than 72 hours | 9.4% |

**By category (median hours to close, share slower than 72 hours):**

| Category | Median hours | Slower than 72 h |
|---|---|---|
| Unsanitary Condition | 243.3 | 87% |
| Street Condition | 41.3 | 35% |
| Heat / Hot Water | 39.1 | 15% |
| Dirty Condition | 25.6 | 13% |
| Blocked Driveway | 1.5 | 0% |
| Illegal Parking | 1.5 | 0% |
| Noise (Residential) | 1.2 | 0% |
| Noise (Street/Sidewalk) | 1.1 | 0% |

- **By area, the slow-ticket share is similar everywhere:** Manhattan 12%, Staten Island 11%, Brooklyn 10%, Bronx 9%, Queens 7%.
- **Top hotspots:** Bronx / Unsanitary Condition (89% slow, 211 sampled tickets), Manhattan / Unsanitary Condition (91%, 143), Brooklyn / Unsanitary Condition (81%, 189), Queens / Unsanitary Condition (90%, 77).
- **Model:** AUC **0.972**, against **0.953** for a category-only baseline. The category already explains almost everything, and area and time add a little.
- **Forecast (simple trend line on full-count weekly volumes):** about 73,197 tickets per week now, falling to 72,095 over four weeks (trend of about -367 per week).

## What the evaluation showed

- **The category decides the wait.** Noise and parking complaints close in about an hour, while unsanitary-condition complaints take about ten days. Area matters far less.
- **The prediction model adds little over the category.** Going from 0.953 to 0.972 AUC is real but small, so a simple category rule would already catch most slow tickets.
- **Data design mattered more than modelling.** The first version pulled only about 2 days of data. That made the monthly charts and the forecast meaningless, and a fallback silently set the SLA to the median, so the "breach rate" was 50% by construction and the AI briefing reported it as a finding. The fixed version samples across a full year, uses true volumes for the forecast, and makes the SLA choice explicit.

## Limitations

- **The response-time numbers come from a sample of 24 days.** Use them as rates and medians, not as city totals.
- **The 72-hour SLA is chosen by me.** Real agencies have different targets per category.
- **"Closed" in NYC 311 is the agency's closing time, not proof that the problem was fixed.**
- Weekday and monthly patterns rest on only 24 days, so they are hints (for example, the weekend vs weekday gap is small and noisy).
- The forecast is a straight trend line with no seasonality.

## Tech stack

pandas, scikit-learn, matplotlib, seaborn, requests, Groq API.

## How to run

1. Open the notebook in Google Colab.
2. Run the cells from top to bottom.
3. When asked, paste your own Groq API key (free at console.groq.com). It is hidden while typing and is never saved in the notebook.

The data download adds about 75 quick requests, so allow a minute or two.

## Files the notebook creates

- `urbanpulse_tickets.csv`: the closed tickets in the sample
- `urbanpulse_hotspots.csv`: area and category hotspots
- `urbanpulse_daily_volume.csv`: full-count daily ticket volumes

---

Data: NYC Open Data (311 Service Requests). Built by **Akshat Kesharwani** | [GitHub](https://github.com/akshatkesharwani-info) | [Portfolio](https://akshatkesharwani-info.github.io/)
