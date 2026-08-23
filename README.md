# Job Market Graph Analysis

**Identifying skill-based clusters in the labour market by modelling job vacancies as a similarity graph.**

A data analysis project built on real hh.ru vacancy data, combining NLP-based skill extraction, graph construction, and community detection to answer a practical business question: *which job postings are effectively the same role, and how does skill demand cluster across the market?*

> **Project status:** actively in progress. The data collection and skill-extraction stages have produced real output; graph construction, clustering, and the interactive dashboard are implemented in code but not yet run end-to-end. See [Project status](#project-status) for a transparent breakdown of what's done vs. pending — no invented numbers or screenshots below.

---

## 1. Problem

Job postings describe roles in unstructured free text, which makes it hard to answer questions that matter for workforce and market analysis:

- Which "different" job titles actually require the same skill set?
- How does skill demand cluster across the market — where are the natural role groupings?
- Can we quantify skill overlap between roles instead of relying on title matching alone?

This project treats each vacancy as a node in a graph, connects vacancies that share extracted key skills, and applies community detection to surface clusters of similar roles — a structured, reproducible alternative to eyeballing job titles.

## 2. Data

| | |
|---|---|
| **Source** | [hh.ru public vacancy search API](https://api.hh.ru/openapi/redoc#tag/Poisk-vakansij/operation/get-vacancies) (no auth token — requirements/responsibilities pulled from search snippets) |
| **Volume** | Target of 200–300 vacancies per collection run (current sample: 250 vacancies in `titles_keywords.csv`) |
| **Fields** | `id` (vacancy ID), `title`, `requirement` (candidate requirements, raw text), `responsibility` (job responsibilities, raw text) |
| **Collection method** | Paginated GET requests to the hh.ru search API via a custom `requests.Session` subclass with exponential backoff retries, keyed by a search term and page count |

## 3. Pipeline

```
HH API
   ↓
Data Collection        →  paginated search + retry session
   ↓
Text Cleaning           →  keep Cyrillic/Latin characters + mid-word hyphens
   ↓
Lemmatization & POS filtering  →  spaCy (ru_core_news_sm), keep NOUN / X tags
   ↓
Skill Extraction        →  TF-IDF top-N terms per vacancy (scikit-learn)
   ↓
Similarity Graph        →  NetworkX, edges weighted by shared-keyword count
   ↓
Community Detection     →  Louvain algorithm
   ↓
Visualisation           →  Plotly interactive graph + Dash/Flask web app
```

## 4. Methods & Tools

- **Language:** Python (pandas, requests)
- **NLP:** spaCy (`ru_core_news_sm`) for lemmatization and POS filtering
- **Feature engineering:** TF-IDF (scikit-learn)
- **Graph analysis:** NetworkX
- **Clustering:** Louvain community detection
- **Visualisation:** Plotly (interactive), Matplotlib (static)
- **Delivery:** Flask + Plotly Dash web app (landing page, data overview, network analytics)
- **Engineering practice:** custom retry-with-backoff HTTP session for reliable API collection; unit tests for core components

## 5. Results

*To be completed once the pipeline has been run end-to-end — see [Project status](#project-status) for exactly what's implemented today. This section will report: total vacancies processed, total unique skills extracted, number of communities detected, and a short profile of each major cluster (dominant skills, example titles).*

## 6. Visuals

*Screenshots below will be added once the network graph, cluster visualisation, and dashboard pages are generated from a full pipeline run.*

- [ ] Network graph (full vacancy similarity graph)
- [ ] Cluster visualisation (Plotly, colour-coded communities)
- [ ] Dashboard — data overview page
- [ ] Dashboard — network analytics page

---

## Project status

Transparency on implementation status, since this repo started as a course assignment template:

| Component | File | Status |
|---|---|---|
| Retry session (`Session`) | `session.py` | ⬜ Pending |
| Vacancy collection (`get_vacancies`) | `hh_data_collection.py` | ⬜ Pending |
| Text preprocessing (`preprocess_text`) | `network.py` | ⬜ Pending |
| Keyword extraction (`get_keywords`) | `network.py` | ⬜ Pending — sample output already exists in `titles_keywords.csv` (250 vacancies) |
| Graph construction (`create_network`) | `network.py` | ⬜ Pending |
| Community detection (`get_communities`) | `network.py` | ✅ Done |
| Static graph plot (`plot_network`) | `network.py` | ✅ Done |
| Interactive community plot (`plot_communities`) | `network.py` | ✅ Done |
| Single-community plot (`plot_one_community`) | `network.py` | ⬜ Pending |
| Web app — landing page | `app.py` | ✅ Done |
| Web app — data overview page | `app.py` | ✅ Done |
| Web app — network analytics page | `app.py` | ⬜ Pending (empty layout) |

**Next steps:**
1. Finalise `Session` and `get_vacancies` for reproducible data collection.
2. Finalise `preprocess_text` and `create_network` to complete the pipeline.
3. Run end-to-end, capture real cluster counts, and populate the Results section.
4. Wire up the network analytics dashboard page and capture screenshots.

## How to run

```bash
git clone https://github.com/<your-username>/job-market-graph-analysis.git
cd job-market-graph-analysis/homework03
pip install pandas requests spacy scikit-learn networkx plotly dash flask matplotlib
python -m spacy download ru_core_news_sm
python hh_data_collection.py      # collect vacancies from hh.ru
python app.py                     # launch the Flask/Dash app locally
```

## Tests

```bash
pytest homework03/tests/
```
