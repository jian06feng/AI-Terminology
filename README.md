# An Empirical Analysis of AI Terminology in Scientific News

Code and data for the empirical analysis of AI terminology in scientific news.
The corpus is 2,231 AI-tagged news articles from five US university news platforms
(MIT, Stanford, JHU, CMU, Cornell), and the complete MIT News archive for 2021–2025.

```
├── web scraping code/    # collect links, scrape and repair articles
│   ├── 1 MIT/  2 Stanford/  3 JHU/  4 Cornell/  5 CMU/
├── Data/                 # final inputs of the analysis notebook
└── code.ipynb            # every table and figure of the analysis
```

## Data collection

Every school works the same way, in two stages: collect the article links from its AI listing
(`*_01_Collect_Links`), then scrape each article (`*_02_Scrape_Articles`).

| School | AI listing | Method |
|---|---|---|
| 1 MIT | `news.mit.edu/topic/artificial-intelligence2`, first 100 pages | `requests` + stable CSS selectors |
| 2 Stanford | Stanford Report AI category | Playwright + Chromium (site returns 403 to plain requests); JSON-LD metadata |
| 3 JHU | `hub.jhu.edu/tags/artificial-intelligence` | `requests`; listing metadata kept as fallback |
| 4 Cornell | Cornell Chronicle *Artificial Intelligence* category | `requests`; JSON-LD first, HTML fallback |
| 5 CMU | `cs.cmu.edu/news/archive?keywords=artificial-intelligence` | Playwright dynamic loader; multi-template body parser |

The MIT folder holds two further notebooks for a second, untagged pass used to measure what the AI
tag leaves out: `MIT_04_Full_Archive_Links` walks MIT News' JSON API (`/api/recent-news/{uuid}?pg=N`)
back to 2021-01-01 and `MIT_05_Full_Archive_Articles` scrapes every article, tagged or not, with the
same selectors as `MIT_02` (checkpoint/resume enabled). Output:
`Data/MIT_Full_Archive_Articles_2021_2025.csv` (5,571 articles, 5,040 of them published 2021–2025).

## Data cleaning

| Notebook | Fix | Final file (copied to `Data/`) |
|---|---|---|
| `MIT_03_Fix_Duplicated_Bodies` | re-scrape 33 duplicated/missing bodies with Trafilatura, merge by URL | `MIT Articles.csv` (1,498) |
| `JHU_03_Remove_External_Redirects` | drop 3 rows resolving to external publishers | `JHU Articles.csv` (130) |
| `CMU_03_Fix_Incomplete_Rows` | recover 9 JS-rendered bodies with Playwright; drop 8 external-media rows | `CMU Articles.csv` (283) |
| — | no repair needed | `Stanford Articles.csv` (231), `Cornell Articles.csv` (90) |

The remaining cleaning (CMU `$dateFormat` dates from the listing date; three sub-50-word bodies
adjudicated manually) happens in `code.ipynb` and yields the 2,231 articles.

## `Data/`

| File | Role |
|---|---|
| `MIT / Stanford / JHU / CMU / Cornell Articles.csv` | final AI-tagged corpora; shared fields `Published Date, Author, Source, Article Header, Sub_Headings, Article Body, Url` plus source-specific metadata |
| `MIT_Full_Archive_Articles_2021_2025.csv` | full MIT archive for the selection-bias pilot |
| `Dictionary.json` | communicative-orientation dictionary: 8 orientations, strong phrases (weight 3) and weak terms (weight 1) |
| `AI terminology.json` | 4 terminology families: General AI, ML, DL, Generative AI |

## Analysis notebook (`code.ipynb`)

| Section | analysis |
|---|---|
| 1 Load and clean | merge five sources, normalize dates, repair CMU dates, adjudicate short bodies |
| 2 Measurement | text = headline + sub-headings + body; boundary-aware cue matching; orientation score `S = 3·strong + weak`, intensity per 1,000 words, presence = ≥1 strong or ≥5 weak; terminology family prevalence and intensity |
| 3 Core window | four-school window 2021–2025 (1,290 articles; Cornell and 2026 excluded) |
| 4 Descriptive | prevalence and intensity overall and by institution; dominant orientation; threshold sensitivity |
| 5 Change over time | pre/post-2023 differences; segmented OLS anchored at Dec 2022 (institution FE, log length, HC3, FDR); AIC breakpoint scan over every candidate month; cue-level log-odds shifts |
| 6 Terminology × orientation | Fisher/lift/phi co-occurrence, high-intensity variant, masking check for shared tokens, adjusted logistic and linear models, Spearman |
| 7 Content domains | TF-IDF/NMF topics (k = 8–15) collapsed to 8 domains; nested logistic models A/B/C (ΔAIC, pseudo-R², LR tests); MiniLM + k-means robustness check (ARI, NMI) |
| 8 Robustness | ceiling diagnostics, relative emphasis share, 1,000-resample bootstrap intervals, strong-phrase weight sensitivity, anchor-sentence semantic validation |
| 9 MIT pilot | AI-tagged vs full-archive AI-term hits by year |
