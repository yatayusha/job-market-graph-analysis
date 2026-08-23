# Job Market Graph Analysis

**Exploring relationships between job vacancies by modelling shared skills as a similarity graph.**

A Python-based data analysis project that uses real job vacancy data from hh.ru to explore how different roles are connected through shared skills and keywords.

The project combines API-based data collection, text preprocessing, TF-IDF feature extraction, graph analysis, and community detection.

---

## Problem

Job vacancy descriptions contain valuable information about required skills and responsibilities, but this information is largely unstructured.

This project explores:

- Which job vacancies have similar skill requirements?
- How can relationships between vacancies be represented as a graph?
- Can community detection identify clusters of related job roles?
- Which skills connect different areas of the labour market?

Each vacancy is represented as a node in a graph. Connections between vacancies are based on the similarity of extracted keywords.

---

## Data

The project uses job vacancy data collected from the public hh.ru API.

The dataset includes information such as:

- Job title
- Candidate requirements
- Job responsibilities

The collected text is processed to extract keywords and build structured representations of vacancy skill requirements.

Processed datasets are stored inside the project structure:

```text
homework03/
├── title_tokens.csv
├── title_keywords.csv
└── titles_keywords.csv
````

---

## Analysis Pipeline

```text
HH.ru API
   ↓
Vacancy Data Collection
   ↓
Text Cleaning
   ↓
Tokenisation & Lemmatization
   ↓
POS Filtering
   ↓
TF-IDF Keyword Extraction
   ↓
Vacancy Similarity Graph
   ↓
Community Detection
   ↓
Graph Visualisation
```

---

## Methods

### Data Collection

Vacancy data is collected programmatically using the hh.ru API.

### NLP & Text Processing

Job descriptions are processed using NLP techniques to transform unstructured vacancy text into structured features.

The workflow includes:

* Text cleaning
* Tokenisation
* Lemmatization
* Part-of-speech filtering
* Keyword extraction

### Feature Extraction

TF-IDF is used to identify important terms and represent vacancy skill requirements as numerical features.

### Graph Analysis

Vacancies are modelled as a similarity graph:

* **Nodes** represent vacancies
* **Edges** represent similarity between extracted skill sets

This makes it possible to analyse relationships between different roles beyond simple job-title matching.

### Community Detection

Graph communities are identified to explore groups of vacancies with similar skill requirements.

---

## Technologies

### Core

* Python
* pandas
* requests

### NLP & Feature Engineering

* spaCy
* scikit-learn
* TF-IDF

### Graph Analysis

* NetworkX
* Louvain Community Detection

### Visualisation & Application

* Plotly
* Dash
* Flask
* Matplotlib

---

## Project Structure

The original project structure is preserved to maintain the existing tested implementation.

```text
job-market-graph-analysis/
│
├── README.md
├── .gitignore
├── requirements.txt
│
└── homework03/
    ├── app.py
    ├── hh_data_collection.py
    ├── network.py
    ├── session.py
    │
    ├── templates/
    │
    ├── tests/
    │
    ├── title_tokens.csv
    ├── title_keywords.csv
    └── titles_keywords.csv
```

### Main Files

| File                    | Description                                      |
| ----------------------- | ------------------------------------------------ |
| `hh_data_collection.py` | Collects job vacancy data from the hh.ru API     |
| `network.py`            | Builds and analyses the vacancy similarity graph |
| `session.py`            | Handles project session logic                    |
| `app.py`                | Application entry point                          |
| `tests/`                | Tests for core project functionality             |
| `templates/`            | Application templates                            |

---

## How to Run

Clone the repository:

```bash
git clone https://github.com/yatayusha/job-market-graph-analysis.git
cd job-market-graph-analysis
```

Install dependencies:

```bash
pip install -r requirements.txt
```

Install the required spaCy language model:

```bash
python -m spacy download ru_core_news_sm
```

Move to the project directory:

```bash
cd homework03
```

Run the application:

```bash
python app.py
```

Run tests:

```bash
pytest tests/
```

---

## Key Concept

The project demonstrates how a traditional job vacancy dataset can be transformed into a graph-based representation:

```text
Unstructured Job Descriptions
            ↓
      NLP Processing
            ↓
    Skill / Keyword Features
            ↓
    Similarity Relationships
            ↓
        Graph Model
            ↓
    Community Structure
```

This approach can help explore relationships between job roles, identify groups of similar vacancies, and analyse how skills connect different parts of the labour market.

---

## Future Improvements

Potential extensions include:

* Running the full pipeline on a larger vacancy dataset
* Analysing community-level skill patterns
* Comparing clusters across different job categories
* Adding interactive graph exploration
* Expanding the dashboard with additional labour-market insights

---

## Skills Demonstrated

**Data Collection · NLP · Text Processing · Feature Engineering · TF-IDF · Graph Analysis · Community Detection · Data Visualisation · Python**

