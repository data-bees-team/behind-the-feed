# Behind the Feed 🐝
### Discover More — A Diversity-Aware News Recommendation System

**Team Data Bees | Behind the Feed Data Challenge 2026 — Phase 2**

Behind the Feed is an interactive news recommendation application designed to help readers discover relevant stories without seeing only more of the same. It combines personal interests, semantic similarity, feedback-driven personalization, and diversity-aware ranking across topics, countries, and publishers.

**[GitHub Repository](https://github.com/data-bees-team/behind-the-feed) | [Live Demo](https://behindthefeed-databees.streamlit.app/)**

---

## The Problem

A news feed optimized only for relevance can repeatedly recommend similar stories or concentrate exposure among familiar sources and regions. This limits opportunities to encounter relevant news from different topics and places.

Our goal is to balance two needs: **recommend news a reader cares about while making room for relevant discovery.**

## What We Built

A multi-user Streamlit application with five main features:

1. **Guest Access, Sign Up & Login:** Explore the application or create an account with saved preferences.
2. **My Feed:** Personalized news recommendations with Interested / Not Interested feedback and a Load More experience.
3. **Discover:** A discovery-oriented feed using the same recommendation engine with stronger exploration.
4. **Recommendation Insights:** Visual summaries of users' recommendations, selected interests, and interactions.
5. **Compare Recommenders:** Side-by-side comparison of the topic-based baseline and diversity-aware hybrid recommendations for the current user.

**Usage Note:** After selecting interests, the first personalized feed may take approximately 6–10 seconds to load. Subsequent interactions are generally faster.

## Dataset

| Property | Value |
|---|---|
| News articles | 15,395 |
| Countries represented | 185 |
| Continents represented | 6 |
| Publisher domains | 2,117 |
| Source | Preprocessed GDELT news data |

The application uses a historical, cleaned news dataset rather than a continuously updated live news feed. Individual articles may have multiple topic labels.

## How Recommendations Work

```text
User-selected interests + past article feedback
                       |
                       v
         Topic relevance scoring
                       |
                       +------> Baseline recommendations
                       |
                       v
        Semantic article matching
       (BGE title text embeddings)
                       |
                       v
    Hybrid candidate retrieval + scoring
                       |
                       v
       Unified diversity-aware reranking
      (semantic, geographic, publisher MMR)
                       |
                       v
          Personalized Feed / Discover
                       |
                       v
          Interested / Not Interested
                       |
                       +------> Updated user preferences
```

### 1. Topic-Based Baseline

The baseline ranks articles using readers' selected topic interests. It provides an interpretable reference point for evaluating the hybrid system.

Because articles may have multiple topics, an article covering a selected interest alongside other topics can receive a lower baseline score than a more narrowly focused article.

### 2. Semantic Retrieval

The hybrid system uses **BAAI/bge-base-en-v1.5** embeddings of article titles to identify semantically related news, including stories that may not share an exact topic label with readers' selected interests.

### 3. Feedback-Based Personalization

Readers can mark articles as **Interested** or **Not Interested**.

These signals adjust subsequent recommendation scores without retraining the underlying language model. A single dislike does not permanently exclude an entire topic.

### 4. Diversity-Aware Reranking

The system uses **Maximal Marginal Relevance (MMR)** to balance relevance against repetition.

Its unified reranking process considers:

- Semantic redundancy between recommended articles.
- Geographic overlap across countries and continents.
- Publisher repetition within the recommendation slate.

This encourages broader information exposure without enforcing strict diversity quotas.

### 5. Discovery

The Discover experience uses the same core recommendation engine with a stronger emphasis on exploration. It aims to surface relevant stories beyond readers' most obvious matches.

## Evaluation

We evaluated the recommendation approach using baseline comparisons, ablation studies, simulated feedback, and relevance–diversity trade-off experiments.

Evaluation scripts and outputs are organized under `evaluation/` and `reports/`.

| Experiment | What It Examines |
|---|---|
| Baseline vs. Hybrid | Relevance and topic, geographic, and publisher coverage |
| Geographic Ablation | Contribution of geographic-aware reranking |
| Semantic Ablation | Contribution of semantic retrieval and ranking |
| MMR Lambda Sweep | Relevance–diversity trade-offs |
| Feedback Adaptation | Responses to simulated positive and negative feedback |
| 30-Round Simulation | Diversity under repeated simulated interactions |

### Selected Experimental Results

**1. Geographic Coverage**

In a geographic ablation using a Technology interest profile, a top-10 slate represented **10 countries with geographic reranking**, compared with **6 countries without it**.

**2. Thirty-Round Simulated Feedback Experiment**

Average results per 10-article recommendation slate:

| Metric | Topic Baseline | Hybrid |
|---|---:|---:|
| Countries | 6.13 | 10.00 |
| Publishers | 8.93 | 10.00 |
| Distinct topic labels | 4.87 | 9.00 |
| Semantic diversity | 0.614 | 0.521 |

The hybrid system achieved broader geographic, publisher, and topic-label coverage in this experiment. The baseline showed higher semantic diversity, illustrating that improvements across different diversity dimensions do not always occur together.

**3. Relevance–Diversity Trade-Off**

The MMR lambda experiment demonstrated an observed trade-off: increasing the emphasis on relevance generally improved the internal relevance score while reducing semantic diversity.

**4. Feedback Responsiveness**

In a controlled Technology experiment, simulated feedback changed the recommendation slate: 7 of the original top-10 articles were replaced, the disliked article dropped out of the top 10, and the liked article remained at rank 1.

These results describe system behavior under controlled offline configurations. They do not establish causal effects on reader satisfaction or long-term engagement.

### Automated Testing

**50 tests passed in 6.90 seconds** in the recorded verification run, covering the hybrid recommendation pipeline, geographic and publisher diversity, feedback personalization, and user management.

To run the tests:

```bash
python -m unittest discover -s tests -v
```

## Technology

**Languages & Frameworks:** Python, Streamlit

**Data Processing:** pandas, NumPy, SQLite

**Machine Learning & Recommendation:** Sentence Transformers, BGE Embeddings, Cosine Similarity, Maximal Marginal Relevance (MMR)

**Development Tools:** Git, GitHub

## Repository Guide

```text
behind-the-feed/
├── Home.py                 # Application entry point
├── pages/                  # Feed, insights, comparison pages
├── components/             # Authentication and article UI
├── recommender/            # Recommendation algorithms
├── utils/                  # Data, accounts, and feedback
├── evaluation/             # Offline experiments
├── reports/                # Evaluation outputs and figures
├── tests/                  # Automated tests
├── data/                   # Dataset and embedding assets
├── styles/                 # Application styling
├── requirements.txt
└── README.md
```

The active hybrid recommendation pipeline uses modules including `hybrid_pipeline.py`, `nlp_profile.py`, and `unified_reranker.py`.

Other recommender modules preserve earlier components and experiments and should not all be interpreted as separate active feed pipelines.

## Run Locally

**Prerequisites:** Git and a compatible Python environment. The project was developed using Python 3.11.

```bash
git clone https://github.com/data-bees-team/behind-the-feed.git
cd behind-the-feed

python3 -m venv .venv
source .venv/bin/activate

python -m pip install -r requirements.txt
streamlit run Home.py
```

Open **http://localhost:8501** in your browser.

On Windows, activate the virtual environment using:

```bash
.venv\Scripts\activate
```

**Data Note:** Local execution requires the dataset and precomputed semantic embedding assets expected by the project's loaders. If these assets are not included in the repository, they must be provided separately.

The **[Live Demo](https://behindthefeed-databees.streamlit.app/)** is the easiest way for judges to explore the completed application.

## Scope & Limitations

- **Historical Data:** The prototype uses a fixed historical news dataset rather than live news ingestion.
- **Diversity Measurement:** Geographic, publisher, and topic coverage are indicators of information exposure, not direct measurements of editorial viewpoints or algorithmic fairness.
- **User Evaluation:** Controlled simulations demonstrate ranking behavior, but real-user studies are needed to assess satisfaction and long-term engagement.
- **Relevance–Diversity Balance:** Improving geographic and topic coverage does not guarantee higher semantic diversity or better recommendations for every reader.

## References

[1] K. Leetaru and P. A. Schrodt, “GDELT: Global Data on Events, Location, and Tone, 1979–2012,” *ISA Annual Convention*, 2013. [GDELT Data](https://www.gdeltproject.org/data.html#rawdatafiles).

[2] J. Carbonell and J. Goldstein, “The Use of MMR, Diversity-Based Reranking for Reordering Documents,” in *Proc. ACM SIGIR*, 1998, pp. 335–336.

[3] S. Xiao *et al.*, “C-Pack: Packaged Resources to Advance General English Embedding,” 2023. Model: [BAAI/bge-base-en-v1.5](https://huggingface.co/BAAI/bge-base-en-v1.5).

[4] P. Castells, N. J. Hurley, and S. Vargas, “Novelty and Diversity in Recommender Systems,” in *Recommender Systems Handbook*, Springer, 2015, pp. 881–918.

## Team Data Bees 🐝

**Jakia Nowshin:** Semantic Embeddings, Hybrid Ranking & Evaluation

**Shahida Batool:** Geographic Diversity & Adaptive Personalization

**Surayia Rahman:** Publisher Variety, User Security & Recommendation Analysis

**Shared Contributions:** Application Design, Documentation, Testing & Final Improvements.

Built by **Team Data Bees** for the **Behind the Feed Data Challenge 2026 — Phase 2**.

*Our aim: Make news recommendations personally useful while opening the door to broader discovery.*
