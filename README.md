# NSE Corporate Announcements and FinBERT Event Study

## Overview

**Research question:** Is the sentiment expressed in corporate announcements associated with short-window stock-market reactions?

This project studies corporate announcements from Indian financial institutions during the period surrounding the 2018 IL&FS crisis. It combines announcement collection from the National Stock Exchange of India (NSE), PDF text extraction, FinBERT financial sentiment analysis, market-model event studies, and regression analysis.

The workflow moves from raw announcement metadata to a filtered announcement sample, extracted and cleaned text, sentiment and readability features, cumulative abnormal returns (CAR), and regression-based analysis.

## Study Design

- **Announcement collection period:** 1 January 2017–31 March 2019.
- **Text-analysis period:** 1 January 2018–31 March 2019.
- **Companies:** Ten Indian financial institutions, including banks and non-bank financial companies.
- **Market benchmark:** NIFTY 50.
- **Main sentiment model:** `ProsusAI/finbert`.
- **Main event window:** CAR(-1,+1).
- **Additional event windows:** CAR(0,+1) and CAR(-3,+3).

The extra announcement-collection history provides market and announcement context before the main text-analysis period.

## Companies Covered

The collection pipeline is configured for:

- HDFC Bank
- ICICI Bank
- Axis Bank
- State Bank of India (SBI)
- Kotak Mahindra Bank
- Bajaj Finance
- Bajaj Finserv
- Shriram Finance
- LIC Housing Finance
- PNB Housing Finance

## Methodology

### 1. NSE announcement collection

The first notebook queries the NSE corporate-announcements endpoint for the configured companies and date range. It saves announcement metadata and downloads available PDF attachments, with retry logic, logging, and checkpoints.

### 2. Announcement filtering

The analysis notebook filters announcements using the NSE `subject` field. It keeps specified financially relevant announcement categories, removes excluded categories, and separately records unclassified subjects for review.

### 3. Document extraction and cleaning

PDF text is extracted using PyMuPDF, with `pdfplumber` as a fallback. ZIP attachments are inspected for supported documents, and text is cleaned before feature extraction. Scanned or image-only PDFs may not yield usable text.

### 4. FinBERT sentiment

The project uses `ProsusAI/finbert` to estimate positive, negative, and neutral sentiment probabilities. Text is divided into chunks, and the chunk-level probabilities are weighted by word count.

The continuous sentiment score is defined as:

\[
\text{FinBERT score}=P(\text{positive})-P(\text{negative})
\]

A positive score indicates a more positive model classification, a negative score a more negative classification, and a score near zero a more neutral balance. It is a model-derived textual measure, not a direct measure of investor expectations.

### 5. Readability and dictionary-based measures

The pipeline also calculates word count, log word count, Flesch Reading Ease, and Flesch–Kincaid grade-level measures. The notebook includes financial dictionary-based measures as additional textual features.

### 6. Market-model event study

For each announcement, the event-study procedure estimates a market model using NIFTY 50 returns and the stock's returns over an estimation window of trading observations from approximately day −120 to day −21 relative to the event.

Abnormal return is calculated as:

\[
AR_t=R_{i,t}-(\hat{\alpha}_i+\hat{\beta}_iR_{m,t})
\]

Cumulative abnormal return is calculated by summing abnormal returns over the selected event window:

\[
CAR_{[a,b]}=\sum_{t=a}^{b}AR_t
\]

The study evaluates CAR(-1,+1), CAR(0,+1), and CAR(-3,+3).

### 7. Regression analysis

The main regression relates CAR(-1,+1) to FinBERT sentiment, log word count, readability, company fixed effects, and year fixed effects. The notebook uses standard errors clustered by company.

Robustness analyses compare the sentiment coefficient across alternative event windows and model specifications.

## Reported Results

The supplied regression output contains **221 announcement-level observations** in the main models.

| Model | R² | FinBERT coefficient | FinBERT p-value |
|---|---:|---:|---:|
| FinBERT only, with company and year effects | 0.121 | 0.0026 | 0.947 |
| FinBERT plus text controls and fixed effects | 0.187 | −0.0364 | 0.167 |

In the controlled model, log word count has a positive coefficient of approximately 0.0113 with a reported p-value of 0.001. The FinBERT coefficient is negative but not statistically significant at conventional levels in the CAR(-1,+1) specification.

For the robustness analysis, the FinBERT coefficient for CAR(-3,+3) is approximately −0.0864 with a reported p-value of 0.0406. This is evidence of an association in that specification, not proof of causality. The multiple event windows and small number of companies should be considered when interpreting statistical significance.

## Figures and Tables

The `figures/` directory contains:

- Distribution of FinBERT sentiment scores.
- Distribution of CAR(-1,+1).
- Scatter plot of FinBERT sentiment against CAR(-1,+1).

The `tables/` directory contains the sample summary, descriptive statistics, main regression output, robustness results, and model summary.

See the README files inside `figures/` and `tables/` for detailed descriptions.

## Limitations

- The sample is restricted to selected financial institutions and a defined historical period; findings may not generalise to other sectors or periods.
- FinBERT scores depend on extracted document text, chunking, and model behaviour.
- The market model assumes a stable relationship over the estimation window.
- Announcement timing and overlapping news can complicate event attribution.
- Company-clustered standard errors are based on a small number of company clusters, so inference should be treated cautiously.
- A statistically significant coefficient does not establish that announcement tone caused the market reaction.
- The collection process depends on historical NSE endpoint behaviour and data availability.

## Repository Structure

```text
nse-announcements-finbert-event-study/
├── README.md
├── requirements.txt
├── .gitignore
├── notebooks/
│   ├── NSE_Announcements_Pipeline.ipynb
│   └── ILFS_Textual_Event_Study.ipynb
├── data/
│   └── README.md
├── figures/
│   ├── README.md
│   └── [approved figures]
├── tables/
│   ├── README.md
│   └── [approved summary tables]
└── src/
    └── [optional reusable Python modules]
```

## Reproducibility

1. Install the packages listed in `requirements.txt`.
2. Review the NSE data access rules and retrieve the required announcement and market data.
3. Configure local data paths rather than relying on machine-specific paths.
4. Run the announcement collection notebook.
5. Run the text-processing and event-study notebook.
6. Compare generated tables with the published outputs.

The FinBERT model requires downloading model weights on first use. Full reproduction also requires the relevant announcement PDFs and stock-market data, which are not included by default.

## Data and Usage

The repository publishes research code and selected aggregate outputs. Raw announcement attachments, full extracted document text, and downloaded market datasets should not be redistributed without checking applicable source terms.

## Disclaimer

This is an academic event-study project, not investment advice. Results describe statistical associations within the selected sample and do not establish a causal effect of announcement sentiment on stock returns.
