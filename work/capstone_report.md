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
## 2. Data safety

The analysis uses an anonymized FlyRank content-refresh dataset containing 30,000 page-level records and 44 original columns. The unit of analysis is a page, and the model uses observable content, visibility, freshness, and engagement signals.

The final model uses 21 features:

- Search and competition: `search_volume`, `competition`, `cpc`
- Content characteristics: `word_count`, `char_count`, `content_type`, `main_intent`
- Visibility and traffic: `impressions_90d`, `clicks_90d`, `pageviews_90d`, `sessions_90d`, `users_90d`, `engaged_sessions_90d`, `scroll_events_90d`
- Freshness: `content_age_days`, `days_since_last_update`
- Performance and engagement: `ctr`, `avg_position`, `engagement_rate`, `scroll_rate`, `ai_traffic_pct`

Several columns were deliberately excluded from the model. `trend_direction` was used to define the declining target and was therefore excluded to prevent direct target leakage. `trend_pct` was also excluded because it directly represents trend information and could reveal the outcome being predicted. `content_id` was excluded because it is an identifier rather than a predictive feature. `client_id` was excluded from the model features and used only for grouped train/test splitting so that pages from the same client would not appear in both sets.

Other derived tier fields and fields not required for the final feature set were also not used as model inputs.

The target is a binary proxy for content decline: a page is labelled as declining when `trend_direction` is equal to `down`, and non-declining otherwise.

The analysis is intended to remain public-safe. It does not use client names, private queries, credentials, or client-identifying information as model inputs or reported recommendations.
## 3. Baseline

A transparent rule-based refresh score was created as the baseline before training the machine learning model. The purpose of the baseline was to provide a simple, interpretable reference point using signals that an editor could understand directly.

The baseline combines four components:

- Visibility score from the percentile rank of `log1p(impressions_90d)`
- Freshness risk score from the percentile rank of `days_since_last_update`
- Position score based on normalized `avg_position`, weighted by visibility
- Depth gap score based on lower `word_count` relative to other pages, weighted by visibility

The final baseline refresh score uses the following weights:

`0.40 × visibility + 0.30 × freshness + 0.25 × position + 0.05 × depth`

The score is used to rank pages, with higher scores receiving higher priority for review.

The baseline was evaluated using the same held-out test set and the same Precision@50 metric as the Random Forest model. Its Precision@50 was 0.3200, meaning that 16 of the top 50 ranked pages were labelled as declining.

This provides a transparent benchmark for determining whether the machine learning model adds useful ranking signal beyond a manually designed scoring rule.

## 4. Model / analysis

The main model is a Random Forest classifier used to rank pages according to their likelihood of belonging to the declining-page class. Random Forest was selected because it can capture nonlinear relationships between multiple content, visibility, freshness, and engagement signals while also providing feature-importance information for interpretation.

The model was trained using 300 decision trees with `random_state=42`, `n_jobs=-1`, and `class_weight="balanced"`. Numerical features were median-imputed, while categorical features were filled using the most frequent value and one-hot encoded. Unknown categorical values were ignored during transformation.

The 21 model features were:

`search_volume`, `competition`, `cpc`, `word_count`, `char_count`, `impressions_90d`, `clicks_90d`, `pageviews_90d`, `sessions_90d`, `users_90d`, `engaged_sessions_90d`, `scroll_events_90d`, `content_age_days`, `days_since_last_update`, `ctr`, `avg_position`, `engagement_rate`, `scroll_rate`, `ai_traffic_pct`, `content_type`, and `main_intent`.

The target is a binary proxy for content decline. A page is labelled as declining when `trend_direction` is equal to `down`; otherwise it is labelled as non-declining.

The fields `trend_direction` and `trend_pct` were excluded from the model because they contain direct information about page movement and could leak the target. `content_id` was excluded as an identifier, while `client_id` was used only to create the grouped train/test split and was not used as a model feature.

The model produces a ranking by using the predicted probability of the declining class. Pages with higher predicted probability are placed higher in the ranked review list.

## 5. Evaluation

The evaluation uses a client-level train/test split rather than a random row-level split. `GroupShuffleSplit` was used with `test_size=0.20` and `random_state=42`, using `client_id` only as the grouping variable. This prevents pages from the same client from appearing in both the training and test sets.

The resulting split contained 23,837 training rows from 25 clients and 6,163 test rows from 7 clients, with zero client overlap between the two sets. The declining rate was 55.01% in the training set and 51.10% in the test set.

The primary evaluation metric is Precision@50 because the practical use case is to prioritize a small number of pages for human review. The model and baseline were evaluated on exactly the same held-out test set.

| Method | Precision@50 |
|---|---:|
| Transparent baseline | 0.3200 |
| Random Forest model | 0.6200 |

The Random Forest identified 31 declining pages among its top 50 ranked pages, compared with 16 among the top 50 pages ranked by the baseline. This represents a 1.94× relative lift over the baseline.

The test-set declining base rate was 51.10%. Therefore, the 62.00% Precision@50 result indicates improved concentration of declining pages in the top-ranked set compared with the overall test-set rate, while the comparison with the baseline provides the main measure of improvement for this analysis.

The ranking should be interpreted as decision support rather than as proof that a page will decline or that refreshing a page will improve its performance. The evaluation measures how well the ranking identifies pages associated with the declining label in the held-out data.
## 6. Interpretation

The Random Forest feature-importance analysis indicates that visibility, search-position, content age, content length, and click-through behaviour were among the most informative signals for distinguishing declining pages in this dataset.

The strongest features included `impressions_90d`, `avg_position`, `content_age_days`, `char_count`, `word_count`, and `ctr`. This suggests that the ranking signal is not driven by a single variable, but by a combination of how visible a page is, where it appears in search, how old the content is, how much content it contains, and how effectively impressions translate into clicks.

The analysis also produced several useful negative or mixed findings. The relationship between word count and impressions was weakly positive (Pearson correlation approximately 0.163), so content length alone is not a strong explanation for visibility. The relationship between average position and CTR was also very weak in the signal audit (approximately -0.073), indicating that this relationship should not be treated as a simple one-variable rule.

These findings support using the model as a ranking and prioritization tool rather than interpreting any individual feature as a causal driver of decline. Feature importance describes how useful a feature was to the trained Random Forest for this prediction task; it does not establish that changing that feature will cause page performance to improve.

## 7. Recommendation

The main purpose of the model is to help decide which pages should be looked at first by a human editor. I ranked the pages using the probability given by the Random Forest model and then assigned a simple action based on the signals available for each page.

For the top 50 pages from the test set, the recommended actions were:

| Recommended action | Number of pages |
|---|---:|
| Review title/snippet and CTR | 29 |
| General content review | 13 |
| Review for content refresh | 8 |

The actions can be used as a simple starting point for an editor. For example, pages with good visibility but relatively low CTR can be checked for possible title or snippet improvements. Pages showing declining signals along with enough visibility or demand can be considered for a deeper content review. For pages where the available signals do not point to one clear issue, a general review can be done first.

These recommendations are not meant to automatically decide what should be changed on a page. The editor should review the page and its context before taking any action.

The results give useful direction for prioritizing review, but there are some limits. The model was trained to identify pages associated with the declining label, so it does not tell us exactly why a page declined or whether making a particular change will improve its future performance. Because of this, I treat the output as a decision-support tool rather than a prediction of what will happen after a content refresh.

## 8. Reproducibility

I kept the main analysis in the capstone notebook so that the steps can be followed and run again from the notebook. The notebook is available in the `work/notebooks/` folder of the repository.

The analysis uses `random_state=42` for the train/test split and the Random Forest model. I used a client-level split with `GroupShuffleSplit`, so pages from the same client do not appear in both the training and test sets.

The main steps to reproduce the analysis are:

1. Clone the repository.
2. Open `work/notebooks/capstone.ipynb`.
3. Make sure the FlyRank dataset is available at the expected data path.
4. Run the notebook from top to bottom.
5. The notebook loads the data, creates the target, checks for leakage, creates the baseline, trains the Random Forest model, evaluates both methods, and generates the ranked recommendations.

The main Python libraries used include pandas, numpy, scikit-learn, matplotlib, and seaborn.

The random seed and split settings are kept fixed so that the main evaluation can be reproduced using the same data and setup.

The evaluation is based on a held-out client-level test set. I report the resulting metrics in this report rather than presenting the results as a causal experiment. The analysis shows how well the ranking identifies pages associated with the declining label in the available data.

## 9. Acknowledgments & data credit

This project was completed as part of the FlyRank ML Internship and uses the FlyRank ML Internship dataset.

Built on the [FlyRank ML Internship dataset](https://flyrank.ai).
