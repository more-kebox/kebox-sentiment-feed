# KEBOX Property Fear & Greed Index

> A daily market-sentiment reading for the Australian residential property market.

The KEBOX app reads the published [`sentiment.json`](./sentiment.json) file from this repository as a **read-only public feed**. The feed uses a versioned schema so mobile clients can safely ignore fields they do not recognise.

## What it is

The index summarises property-market news and reported weekend auction clearance data. It is high-level market context, not an assessment of an individual property or an investor's portfolio.

## What it is not

It is **not** a property valuation, market forecast, lending assessment, financial advice or investment recommendation. It must not be used to alter KEBOX property estimates, Portfolio health, equity, LVR, rent, cash flow or any other portfolio calculation.

## Feed contract

- **Endpoint after publication:** `https://raw.githubusercontent.com/more-kebox/kebox-sentiment-feed/main/sentiment.json`
- **Schema:** Version `1`; clients reject unsupported schema versions and ignore unknown fields.
- **Attribution:** Every displayed headline and auction data point links to its original publisher.
- **Data handling:** The feed contains metadata, short summaries and source links only—never full article text, credentials, cookies or paywalled content.

## Updates

The publisher is responsible for authorised feed updates, source attribution and source rights. Do not submit user data, personal information or private property imagery to this repository.
