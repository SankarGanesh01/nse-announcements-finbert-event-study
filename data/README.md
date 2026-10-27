# Data Documentation

## Project

**NSE Corporate Announcements and FinBERT Event Study**

This project combines NSE corporate-announcement metadata and PDF attachments with daily stock-market prices to study the relationship between announcement sentiment and cumulative abnormal returns (CAR).

Raw announcement files, full extracted text, and downloaded market datasets are excluded from the public repository by default.

## 1. Announcement Data

The downloader notebook queries the NSE corporate-announcements endpoint for ten financial institutions over **1 January 2017 to 31 March 2019**.

The configured companies and NSE symbols are:

| Company | NSE symbol |
|---|---|
| HDFC Bank | HDFCBANK |
| ICICI Bank | ICICIBANK |
| Axis Bank | AXISBANK |
| State Bank of India | SBIN |
| Kotak Mahindra Bank | KOTAKBANK |
| Bajaj Finance | BAJFINANCE |
| Bajaj Finserv | BAJAJFINSV |
| Shriram Finance | SRTRANSFIN |
| LIC Housing Finance | LICHSGFIN |
| PNB Housing Finance | PNBHOUSING |

The downloader stores announcement metadata and downloaded attachments in company-specific folders. The analysis notebook filters the data to announcements from **1 January 2018 to 31 March 2019** whose attachment-download status is `downloaded` or `skipped_exists`.

## 2. Announcement Filtering

The analysis filters on the NSE `subject` field, retaining predefined financially relevant categories such as financial results, credit ratings, acquisitions, fundraising, investor presentations, securities issuance, and board-meeting outcomes.

Other predefined categories are excluded. Unrecognised subject labels are separated for review rather than silently included.

The final research sample therefore depends on the subject-category rules, attachment availability, successful text extraction, and the availability of matching stock-market observations.

## 3. PDF Text and Sentiment Inputs

The pipeline extracts text from PDF attachments using PyMuPDF, with `pdfplumber` as a fallback.

The text-processing workflow produces:

- Cleaned announcement text.
- Word count and log word count.
- FinBERT positive, negative, and neutral probabilities.
- A continuous sentiment score equal to positive probability minus negative probability.
- Readability measures, including Flesch Reading Ease and Flesch–Kincaid grade level.
- Additional dictionary-based textual measures where available.

FinBERT uses the pretrained `ProsusAI/finbert` model. The model weights are downloaded separately and are not included in this repository.

Scanned PDFs and documents with insufficient extractable text may be excluded from downstream analysis.

## 4. Market Data

The event-study notebook expects daily closing-price CSV files in:

```text
data/raw/market_data/
├── NIFTY50.csv
├── HDFC_Bank.csv
├── ICICI_Bank.csv
├── Axis_Bank.csv
├── SBI.csv
├── Kotak_Mahindra_Bank.csv
├── Bajaj_Finance.csv
├── Bajaj_Finserv.csv
├── Shriram_Finance.csv
├── LIC_Housing_Finance.csv
└── PNB_Housing_Finance.csv
```

The expected CSV schema includes:

- `date`: observation date.
- `close`: closing price.

The notebook converts dates to datetime values, sorts observations, and calculates simple daily returns using percentage price changes.

The market-data directory is an input requirement. The announcement downloader does not itself populate these stock-price files, so they must be obtained and prepared separately.

The event-study configuration requests market history from **1 January 2017 to 31 March 2019**, with announcement analysis focused on the subsequent period beginning in January 2018.

## 5. Event-Study Construction

The notebook estimates a market model using an estimation window approximately spanning trading observations −120 to −21 relative to the event.

For each announcement, it estimates:

\[
R_{i,t}=\alpha_i+\beta_iR_{m,t}+\epsilon_{i,t}
\]

Abnormal returns are the difference between observed stock returns and estimated market-model returns. These are summed across three event windows:

- CAR(-1,+1)
- CAR(0,+1)
- CAR(-3,+3)

The master dataset retains announcement-level observations with a valid FinBERT score and a valid CAR(-1,+1) value.

## 6. Generated Data Products

The notebook creates several derived outputs:

| Output | Contents |
|---|---|
| `filtered_metadata.csv` | Announcements retained by the subject-category filter |
| `master_dataset.csv` | Announcement-level sentiment, readability, company, date, and CAR variables |
| `event_study_results.csv` | Detailed processed announcement records and event-study outputs |
| `summary_statistics.csv` | Descriptive statistics for selected research variables |
| `regression_results.csv` | Summary of model sample sizes and R² values |
| `table1_sample_summary.csv` | Number of retained announcements by company |
| `table2_descriptive_statistics.csv` | Descriptive statistics for sentiment, CAR, word count and readability |
| `table3_main_regression.txt` | Main OLS regression output |
| `table4_robustness_results.csv` | Sentiment coefficients across alternative CAR windows |

Some detailed outputs can contain extracted text, attachment URLs, raw metadata, or local file paths. They should be reviewed and sanitised before public release. The repository should generally publish aggregate results rather than the complete intermediate dataset.

## 7. Reproduction Notes

To reproduce the analysis:

1. Obtain the required announcement metadata and PDF attachments from authorised sources.
2. Prepare the stock-market CSV inputs in the expected directory and schema.
3. Configure local paths in both notebooks.
4. Run the announcement collection notebook.
5. Run the text filtering, PDF extraction, FinBERT, and readability steps.
6. Calculate event-window CAR values and estimate the regression models.
7. Compare the resulting tables with the approved outputs in `tables/`.

The collection pipeline uses NSE session cookies, retry logic, and checkpoints. NSE endpoint behaviour and historical availability may change, so a rerun may not produce exactly the same metadata or attachments.

## 8. Data Governance

NSE announcements may be publicly accessible, but that does not automatically permit unrestricted redistribution of downloaded attachments or datasets.

Review the source terms before publishing raw files. Do not commit large downloaded PDFs, complete extracted document text, credentials, or private local paths. Use the repository's figures and reviewed aggregate tables to communicate the findings where redistribution is permitted.
