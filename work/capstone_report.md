# Capstone Report — Refresh / Content Opportunity Scoring

- **Author:** Dhun Sehgal
- **Lane:** Refresh / Content Opportunity Scoring
- **Repo:** https://github.com/24f2001824/ml-flyrank
- **Date:** September 2026
## 0. Abstract

This capstone investigates whether observable content, visibility, freshness, and engagement signals can be used to rank pages for human review and possible content-refresh actions. The analysis uses 30,000 anonymized page-level records from the FlyRank ML Internship dataset and excludes direct label-derived fields and identifiers from the model features. A transparent baseline score is compared with a Random Forest classifier using a client-level held-out test split to reduce client-level leakage. The Random Forest achieved a Precision@50 of 0.6200 compared with 0.3200 for the baseline, corresponding to a 1.94× lift over the baseline, while the test-set declining base rate was 51.10%. The resulting ranked output is intended as decision support for editors, helping them prioritize pages for review rather than automatically deciding that a page should be refreshed.

## 1. Problem framing

This capstone supports the decision of which pages should receive human review for possible content refresh or improvement.

The unit of analysis is a page. The output is a ranked list of pages with a refresh-opportunity score and an associated reason/action. A human editor can use the ranking to decide which pages to review first, such as checking the title and snippet, reviewing the overall content, or considering a content refresh.

The cost of a wrong call is asymmetric. Prioritizing a page that does not need attention can waste editorial time, while failing to prioritize a page that warrants review can result in a missed improvement opportunity.

Data and machine learning are useful because the dataset contains multiple observable signals related to visibility, content characteristics, freshness, and engagement. Combining these signals into a ranking can provide a consistent decision-support tool rather than relying only on a single manually selected rule.
