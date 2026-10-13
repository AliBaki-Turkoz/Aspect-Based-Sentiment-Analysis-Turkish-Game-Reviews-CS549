# Aspect-Based Sentiment Analysis of Turkish Game Reviews (Steam)

| | |
|---|---|
| **Course** | CS549 – Natural Language Processing, Özyeğin University (M.Sc. Computer Engineering), Spring 2025 · Instructor: Olcay Taner Yıldız |
| **Type** | Team project (see [Team & contributions](#team--contributions)) · repository maintained by Ali Baki Türköz |

## Problem
Game reviews often mix opinions, for example *"Grafikler güzel ama yapay zekâ kötü."* ("The graphics are nice but the AI is bad."). The project assigns a sentiment to each **game aspect** separately. Each aspect gets **Olumlu / Nötr / Olumsuz** (positive / neutral / negative), and each review also gets an overall sentiment (*Genel Duygu*).

The eight aspects are: Graphics, AI, Gameplay, Sound & Music, Game World, Community & Social, Story, Performance & Technical.

## Pipeline
| Step | Folder | What it does |
|---|---|---|
| 1. Scraping | `dataset_scraping/` | Uses the Steam Web API (Turkish language setting). It collects app ids, keeps games that have Turkish reviews, and downloads the reviews to `dataset/reviews.csv`. |
| 2. Cleaning / filtering | `data_preparation/annotation/profanity_language_filter.py` | Removes non-Turkish, malformed, profane and meaningless reviews. Uses `langdetect` and keyword lists. |
| 3. Aspect labeling | `data_preparation/cleaning_kodu_update.py` (final), `cleaning.ipynb` (earlier version), `annotation/rule_based_annotation.py` (first version) | Labels each aspect and the overall sentiment with keyword dictionaries (`constants/*.json`) and Zemberek lemmatisation. Uses local sentiment-word windows and negation handling. Writes colour-coded Excel files. |
| 4. Manual annotation | `annotation/manual_annotation_examples.xlsx` | Examples from the manual labeling guideline |
| 5. Models | `models/` | BERT (`dbmdz/bert-base-turkish-cased`) for overall sentiment, and a TF-IDF + Logistic Regression (multi-output) baseline for all aspects. Uses an 80/10/10 stratified split with seed 42. |

## Data
- **`dataset/reviews.csv`**: 7,262 Turkish Steam reviews of 143 games. The reviewer's Steam ID column was **removed** for privacy.
- **`data_preparation/dataset/cleaned_reviews_zemberek.xlsx`**: the modelling table. It has 7,242 reviews with 8 aspect labels and an overall label.
- **The labels are not balanced.** Most are produced by the keyword rules, and *Nötr* dominates: 86 % of overall labels, and 63–99.6 % per aspect. Only Gameplay has a substantial share of non-neutral labels.

## Results (validation set, 724 reviews)
**N-gram baseline (TF-IDF unigram, 2,000 features + Logistic Regression).** I re-ran this from this repository and reproduced the notebook values.

| Target | Accuracy | Macro-F1 |
|---|---|---|
| Overall sentiment | 0.905 | **0.637** |
| Gameplay | 0.855 | **0.640** |
| Sound & Music | 0.988 | 0.497 |
| Story | 0.963 | 0.460 |
| Graphics · AI · Game World · Community · Performance | 0.97–0.99 | 0.33–0.37 |

**How to read this**
- The high accuracies mostly reflect the dominant *Nötr* class. Always predicting *Nötr* would already score 0.86 on overall sentiment.
- Macro-F1 shows the real picture. Only overall sentiment and Gameplay go clearly beyond the majority class.
- The labels mainly come from keyword rules, so the models largely learn to **reproduce the rule-based labels**. They are not independently validated against human judgement.

**BERT.** It was fine-tuned for 3 epochs, with training loss 0.316 → 0.111 and validation loss 0.324 → 0.275 → 0.299 (rising again in epoch 3). The notebook logs **no accuracy or F1 values for BERT**, and its example predictions are mostly *Nötr*. An accuracy of 97.6 % was quoted in the course presentation, but it cannot be traced to any saved output, so it is **not** reported here.

## Repository structure
```
dataset_scraping/
├── app_info_scraper/app_info_filterer.py      # Steam app list → game details, checkpointed
└── app_reviews_scraper/                       # filter apps with Turkish reviews, download reviews (notebooks)
dataset/
├── reviews.csv                                # raw scraped reviews (Steam IDs removed)
├── cleaned_reviews_zemberek.xlsx              # output of cleaning.ipynb (earlier labeling version)
└── cleaned_reviews_zemberek_GUNCEL.xlsx       # output of cleaning_kodu_update.py (final labeling, + overall sentiment)
data_preparation/
├── cleaning_kodu_update.py, cleaning.ipynb    # Zemberek-based aspect + sentiment labeling
├── annotation/                                # language/profanity filter, first rule-based labeler, manual examples
├── constants/                                 # aspect / sentiment keyword dictionaries, stopwords, suffixes
└── dataset/cleaned_reviews_zemberek.xlsx      # modelling table (with review_id)
models/                                        # BERT + n-gram (experiments.ipynb, process/, models/, utils/, output/results/*.png)
presentation/CS549_Project_TUR1.pptx
```

## Running
```bash
pip install -r requirements.txt
python data_preparation/annotation/profanity_language_filter.py   # needs langdetect
python data_preparation/cleaning_kodu_update.py                   # needs zemberek-python
cd models && jupyter notebook experiments.ipynb                   # n-gram runs on CPU; BERT needs a GPU
```
- All scripts use paths relative to the repository, so they can be run from any folder.
- The scraping notebooks call the public Steam API and need internet access. Their outputs (`app_ids.txt`, checkpoints) are not tracked.

## Notes and changes from the original team code
- Machine-specific absolute paths were replaced with repository-relative paths, and the model notebook now reads the modelling table from `data_preparation/dataset/`.
- **Bug fix:** the n-gram notebook used to treat the `review_id` column as an extra prediction target. It is now excluded. The reported metrics are unaffected.
- **Removed:**
  - duplicate and test copies of the datasets, and empty files;
  - pip-install logs and a 7 MB raw results dump from the notebook outputs;
  - the reviewer Steam IDs, and Steam profile links inside review texts (masked).
- **Regenerated input file:** `Games_Rewievs_Dataset_Excel.xlsx` (input of the annotation scripts) was rebuilt from `reviews.csv` (`app_id`, `review_text`), because the original contained raw rows with Steam IDs.
- **Steam ID collection still in the code:** the scraping code can still collect Steam IDs when you run it. Do not publish them.
- **Content warning:** the review texts are user-generated and contain profanity.

## Team & contributions
| Member | Contribution |
|---|---|
| Ali Baki Türköz | Literature review (lead); manual annotation; feature-based (aspect) labeling and filtering code, including the annotation and profanity/foreign-word cleaning scripts; paper writing (lead, all sections) |
| Emre Öztürk | Literature review (lead); manual annotation; feature-based labeling and filtering code; paper writing (lead, all sections) |
| Demba Sow | Coding lead: Steam scraping, model training and testing, feature-based labeling and filtering code (incl. Zemberek integration); manual annotation; paper writing (findings, discussion, conclusion) |

(lead) marks the lead contributor for a task.

Source: "Contributors and Their Roles" slide of the final presentation and the commit history of the team's original repository.

## Academic integrity
This is graded coursework, shared as a portfolio project. It was carried out as a team; this repository does not claim the model code as individual work.
