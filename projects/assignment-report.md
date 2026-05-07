---
layout: default
title: Data Science Assignment – Spotify Listening Analysis
---

← **[Back to main portfolio](../index.html)**

[Spotify Analysis Project Overview](spotify-analysis.html

# Data Science Assignment: Spotify Listening Analysis

## Abstract

This assignment investigates personal Spotify listening behaviour using
exploratory data analysis and unsupervised machine learning techniques.

---

## 1. Executive Summary

Personal music listening behaviour is complex, habitual, and context dependent, yet is rarely examined using formal analytical techniques at the individual level. This project addresses the challenge of uncovering recurring patterns in personal music listening behaviour through unsupervised learning.  The analysis answers the central question of whether distinct listening day archetypes exist and whether these can be leveraged to support personalised recommendation outcomes.
The dataset comprises event level streaming records extracted from Spotify’s Extended Streaming History export, spanning several years, enriched with audio and genre metadata sourced from Spotify’s developer ecosystem and a supplementary open source dataset. After ingestion and cleansing, individual listening events were aggregated to the day level and transformed into a structured feature set capturing temporal listening patterns (hour of day distributions), acoustic characteristics (Tempo, Valence and more), and artist preferences derived using TF IDF vectorisation and dimensionality reduction.  Particular attention was paid to ensuring that feature construction reflected behavioural patterns rather than total listening volume, allowing comparability between light and heavy listening days. 
Given the unlabelled and exploratory nature of the data, K means clustering was selected as the primary modelling approach. Evaluation using silhouette analysis indicated that a three cluster solution provided an effective balance between internal validity and interpretability. The resulting clusters represent distinct listening day archetypes, differentiated by time of day listening behaviour, musical energy profiles, and artist composition. Principal Component Analysis confirms clear separation between clusters and supports interpretation of the dominant behavioural dimensions. Longitudinal analysis further reveals sustained shifts in cluster prevalence over time that align with contextual changes in account usage, indicating structural changes in listening behaviour rather than short term variability.
The analytical findings were translated into a practical application through a cluster conditioned playlist generation process, producing three distinct, non overlapping playlists representative of each listening day archetype.  Overall, the project presents a reproducible and privacy aware framework for transforming personal behavioural data into interpretable segments and actionable recommendation outputs, illustrating the broader value of unsupervised learning for analysing complex, real world behavioural datasets.

---

## 2. Project background

Music listening habits offer insights into mood and behavioural patterns, supporting self-reflection by revealing habitual consumption trends. This project examines a personal Spotify listening history with the primary objective of identifying natural listening-day archetypes characterised by interpretable temporal, acoustic, and genre-based features. A secondary objective is to generate personalised playlists conditioned on day type and time of day, where consistent patterns emerge. Although the data is not sensitive in a conventional organisational sense, it comprises personal behavioural records and therefore requires careful handling. 

---

## 3. Data Infrastructure and Tools

The dataset was sourced from a shared household Spotify account via Spotify’s export function. The full Extended Streaming History (Figure 1) was chosen over the 12-month subset to allow longitudinal analysis of changes in listening behaviour. After a 30-day processing period, Spotify provided a time-limited download link (Figure 2) to access ten JSON files spanning 2013–2026, which were saved locally to avoid expiration. 

figure 1
figure 2

To enrich the dataset, a Spotify for Developers account was configured to retrieve artist genre labels and track-level audio features (e.g. tempo, danceability, loudness. Appendix 3) through the Spotify Web API (Figures 3 and 4). This required creating a Spotify application (Client ID and Secret) to obtain access tokens, with credentials managed on the developer dashboard. 

figure 3
figure 4

Although Teradata RDBMS was considered for its ingestion and feature-engineering capabilities, its advantages (scalability, performance, enterprise security) were unnecessary given the dataset’s modest size and personal context. Python was chosen for its robust analytics ecosystem and ability to handle the entire workflow, from ingestion through to cleansing, EDA, modelling, validation, and output generation. While a hybrid Teradata–Python approach can be beneficial for larger scale data (Muddarla & Vatti, 2024), Python alone sufficed for this analysis. 
Corporate certificate restrictions prevented direct use of the Spotify API, necessitating a contingency for metadata enrichment. A two-year-old track metadata dataset from HuggingFace (maharshipandya/spotify-tracks-dataset; c125 genres, Appendix 1–2) was used, despite partial coverage (c27.5% match rate).
Ethically, the study was confined to the author’s own listening data to minimise privacy risk and avoid inferring other individuals’ behaviour. As behavioural data can reveal personal routines, such as consistent listening times revealing commuting patterns, outputs were restricted to aggregated forms (day-level summaries, cluster profiles) rather than raw timestamps or location proxies. Spotify API credentials were secured using a password-manager-style method (Spotipy package) to minimise exposure risk and ensure compliance with privacy standards.  While this project is personal, similar pipelines (Python for ETL, API integration) are widely applicable in enterprise data environments.


---

## 4. Data Engineering

Spotify’s Extended Streaming History JSON files were consolidated into an event-level dataset by parsing all Streaming_History_Audio_*.json files and normalising nested structures (Figure 5). 

figure 5
figure 6

Following best practice for behavioural timestamp analysis (Smith, 2020), timestamps were converted from UTC to Europe/London local time to derive consistent temporal features (date, hour, day-of-week), and playback duration was converted from milliseconds to minutes (Figure 7). Device information was simplified into broad categories based on platform and user-agent strings to aid cluster interpretation. 

figure 7

The dataset was filtered to music listening only by excluding podcast and video entries (identified via episode fields and file provenance), ensuring the clustering task reflected music behaviour (Figure 8).  

figure 8

These preprocessing steps improved construct validity by focusing on comparable events, reduced noise from mixed media content, and yielded stable, standardised features suitable for unsupervised modelling without ground-truth labels. The end-to-end data engineering pipeline is summarised in Figure 9. 

figure 9

---

## 5. Data Analytics and Visualisation

Clustering was chosen as the primary analytical approach because the dataset is unlabelled, high-dimensional, and exploratory. With no predefined target labels, supervised learning was inappropriate. Unsupervised clustering groups observations by feature similarity, allowing latent structures to emerge without prior assumptions (Hastie et al., 2017). This makes clustering particularly suitable for personal music data, where recurring listening modes may exist but are not explicitly labelled. 
Choosing the right clustering technique was critical for validity and interpretability.
K-means was considered due to its simplicity, speed, and effectiveness when clusters are of regular shape and size, making it suitable for patterns anchored around consistent days and times and scalable across multi-year data. However, K-means can be distorted by outliers, like unusually short or long listening days, so alternative methods were evaluated. Figure 10 provides an author-compiled comparison of techniques, highlighting assumptions (cluster shape), noise sensitivity, interpretability, and complexity for each (Jain, 2010). 

figure 10

This comparative evaluation, alongside the scikit-learn packages ability to support the selection of K through elbow and silhouette plots, guided the decision to proceed with K-means, acknowledging its limitations while leveraging its practicality. 

Exploratory Data Analysis confirmed data quality and guided feature design. A time series of daily listening minutes identified a gap in 2018 due to use of another platform, and a sharp increase from 2019 onward coinciding with a transition to a shared account. 

figure 11
figure 12

A heatmap of listening by day-of-week and hour-of-day (with minutes normalised to percentages) showed listening concentrated in evenings, dinner hours, and late nights, particularly on weekends, justifying inclusion of a 24-hour temporal distribution as a feature.

figure 13
figure 14

Histograms of key audio features (danceability, energy, valence, tempo) indicated distributions suitable for distance-based clustering, with no extreme skew or anomalies (detailed feature descriptions in Appendix). 
Feature Engineering was performed by transforming daily listening data into a modelling dataset combining temporal, acoustic, and artist-preference features. Temporal features captured each day’s distribution of listening across 24 hours ensuring heavy and light listening days remain comparable by pattern. Acoustic features summarised each day’s audio profile using the mean and standard deviation of audio attributes.

figure 15

To address high artist cardinality (>10,000 unique artists), a day by artist matrix was constructed, with each day encoded as a high dimensional artist incidence vector. TF–IDF vectorisation was applied to down-weight commonly listened to artists and emphasise distinctive ones, and Truncated SVD compressed this representation to 30 dimensions, ensuring days with similar artist patterns remained close even without exact overlaps (Hastie et al., 2017). 

figure 16

StandardScaler was then applied to produce z-score scaling to normalize value ranges prior to modelling. 

figure 17

K-means clustering was applied using these features. Silhouette analysis (Figure 18) indicated that while K=2 achieved the highest mean silhouette score, K=3 yielded a comparable value with greater behavioural interpretability, by contrast, higher K values showed rapidly declining silhouettes indicating diminishing cluster cohesion and separation (Shutaywi & Kachouie, 2021).  A three-cluster solution was selected as an optimal balance between cluster validity and meaningful segmentation.  

figure 18

The clusters were interpreted and labelled based on distinguishing centroid characteristics such as time of day peaks and weekday vs weekend dominance, rather than numeric labels (Figure 19). 

figure 19

Principal Component Analysis (PCA) of the clustered data (Figure 20) confirmed clear separation among clusters, driven primarily by PC1 (late-night vs evening-centric days) and PC2 (energy/valence differences), consistent with expected behavioural dimensions. 

figure 20

Analysing cluster membership over time (Figure 21) revealed significant shifts in listening behaviour.

figure 21

The transition to a shared household account in early 2019 corresponded to a persistent change in dominant listening-day types, a structural shift rather than a transient fluctuation. Other major life events including the birth of a first child and a household relocation, coincided with periods of reduced diversity in day types and a greater predominance of late-night, lower-energy listening days (Figure 22). Incorporating these contextual overlays exemplifies a mixed-methods approach, where qualitative context enhances interpretation of quantitative outputs without altering the underlying model results (Creswell & Plano Clark, 2018; Hands, 2022). 

figure 22

To demonstrate practical utility, a cluster-conditioned playlist generation process was implemented. Each day’s cluster label was first mapped to its individual track records, tagging tracks with the behavioural context of their day. For each cluster, unique track–artist pairs were aggregated along with their audio feature profiles. The cluster’s centroid in audio feature space was calculated, and cosine similarity was used to rank tracks by proximity to this centroid, a common approach in content-based music recommendation (Bonnin & Jannach, 2014). The top 30 tracks per cluster were then selected, with no track repeated across playlists, to create three distinct, non-overlapping playlists, each representing one listening-day archetype (Figures 23–25). 

figure 23
figure 24
figure 25

---

## 6. Reflections and recommendation

This project met its objectives by applying a reproducible, modular, and privacy-preserving workflow to identify distinct listening-day archetypes from personal Spotify data through unsupervised clustering. These archetypes proved to be interpretable and behaviourally meaningful, especially when overlaid with contextual life events that explained sustained structural shifts in listening patterns. The labelled outputs and visualisations allow concise communication of the evolution of listening behaviours over time, and the analysis was further operationalised through cluster-conditioned playlist generation to demonstrate practical value beyond description. 
However, key limitations must be acknowledged. Restricted Spotify API access and reliance on a partially complete static dataset for audio metadata constrained feature coverage, though core behavioural patterns remained robust. Looking ahead, future work should prioritise direct API integration to improve data richness and consider re-running the segmentation on separate time periods (notably pre vs post 2019) to account for observed structural changes in listening behaviour. These enhancements would build on the current framework’s strengths and further refine the personalised music recommendation outcomes derived from the analysis. 

---

## Appendix

Supporting materials and the full Jupyter Notebook are provided below.
``
