# Disaster Tracker

Streams geotagged tweets about a disaster, keeps the relevant ones, classifies them by humanitarian need and plots them on a live map.

Undergraduate capstone, BSc Computer Science, African Leadership University (2021).

**[Read the full project report (PDF)](Kelvin%20Chumbe_FinalReport_Disaster%20Tracker.pdf)**

## Problem

In the first hours of a disaster, response teams often have too little information or more than they can process. People in affected areas post real-time, local updates on Twitter, but these are unstructured and mixed with noise. Filtering, categorizing and mapping them automatically gives teams a shared view of what is happening and where.

## Approach

Two text classifiers run in sequence:

1. **Filtering model**: is the tweet disaster-relevant or not?
2. **Categorizing model**: which of five humanitarian response categories does a relevant tweet belong to (Casualty, Damage Report, Relief Request and Donations, Caution and Advice, Sympathy and Support)?

Both were trained on human-annotated tweets from CrisisLexT26, CrisisNLP, SWDM2013 and ISCRAM2013. Their label schemes differ, so I mapped them onto the five categories and dropped tweets that fit none. Non-English tweets were machine-translated, text was normalised with NLTK, and data was split 70/10/20 (train/validation/test).

For each task I compared GloVe-based CNN and BiLSTM baselines with DistilBERT (frozen or fine-tuned) feeding four BiLSTM layers, an attention layer and dense layers.

## System

1. The user enters hashtags and a country in the React dashboard.
2. The Flask API starts a background Tweepy stream filtered by those hashtags and the country's bounding box.
3. Each tweet with GPS coordinates is preprocessed, filtered and, if relevant, categorized.
4. Results are kept in memory as GeoJSON.
5. The dashboard polls every 10 seconds and plots tweets on a Google Maps crowdmap, with a category chart and tweet list.

## Results

Test-set scores from the report (Tables 2 and 3):

| Task | Architecture | Accuracy | F1 |
|---|---|---|---|
| Filtering | GloVe + BiLSTM | 0.82 | 0.76 |
| Filtering | Fine-tuned DistilBERT + BiLSTM + attention | 0.84 | 0.84 |
| Categorizing | GloVe + BiLSTM | 0.73 | 0.74 |
| Categorizing | Fine-tuned DistilBERT + BiLSTM + attention | 0.84 | 0.84 |

The most common categorizing error was labelling Caution and Advice tweets as Damage Report (about 13% of that class).

## Limitations and future work

- Only geotagged tweets (roughly 1% of all tweets) are mapped. Proposed future work: extract place names from tweet text and geocode them.
- Prototype only: in-memory storage, no authentication, English-only models.

Built against the 2021 Twitter streaming API, so it may not run as-is.

## Tech stack

Python, TensorFlow/Keras, NLTK, Tweepy, Flask, React, Redux, D3.js, Google Maps, Heroku.

## Repository structure

- `Tweets-Filtering/`: filtering model notebook and Streamlit prototype
- `Tweets Classification/`: dataset merging, preprocessing and categorizing model notebooks
- `Web App/api/`: Flask API for streaming, preprocessing and inference
- `Web App/front-end/dashboard/`: React dashboard
