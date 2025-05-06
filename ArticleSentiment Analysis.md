# Article Sentiment Analysis: Super-Atomic, Step-by-Step Walkthrough

---

## Table of Contents
- [Introduction](#introduction)
- [Environment Setup](#environment-setup)
- [Data Collection](#data-collection)
- [HTML Scraping & Storage](#html-scraping--storage)
- [Data Preprocessing](#data-preprocessing)
- [Sentiment Analysis](#sentiment-analysis)
- [Saving Results](#saving-results)
- [Dashboard](#dashboard)
- [Full Code Walkthrough](#full-code-walkthrough)
- [Conclusion](#conclusion)

---

## Introduction

**What is Sentiment Analysis?**

Sentiment analysis is the process of computationally identifying and categorizing opinions expressed in a piece of text, especially to determine whether the writer's attitude towards a particular topic is positive, negative, or neutral.

**Project Goal:**
- Scrape articles from [insights.blackcoffer.com](https://insights.blackcoffer.com)
- Clean and preprocess the text
- Analyze sentiment using multiple methods
- Save and visualize the results in an interactive dashboard

**Why this workflow?**
- Real-world articles are messy: scraping, cleaning, and robust analysis are all required.
- Using both VADER and TextBlob gives a richer sentiment profile.
- The dashboard makes results accessible to non-technical users.

---

## Environment Setup

**Python Version:** 3.x (tested on 3.7+)

**Required Packages:**
- `requests` (for HTTP requests)
- `pandas` (for data manipulation)
- `bs4` (BeautifulSoup, for HTML parsing)
- `nltk` (for NLP tasks)
- `textblob` (for sentiment analysis)

**Installation:**
```bash
pip install requests pandas beautifulsoup4 nltk textblob
```

**NLTK Data:**
- The code will download required NLTK data (stopwords, punkt, vader_lexicon) automatically. If you have network issues, run these manually:
```python
import nltk
nltk.download('stopwords')
nltk.download('punkt')
nltk.download('vader_lexicon')
```

**Folder Structure:**
- The script creates a `data/` folder for HTML files.
- Output CSVs are saved in the working directory.

---

## Data Collection

### Purpose
Download a CSV file containing article URLs and their IDs. This is the starting point for scraping article content.

### Code (with comments)
```python
import requests  # For HTTP requests
import pandas as pd  # For data manipulation
from bs4 import BeautifulSoup, NavigableString  # For HTML parsing

# URL of the CSV file containing article URLs
url = "https://raw.githubusercontent.com/prabhmeharbedi/Article-Sentiment-Analysis/main/output_file.csv"
response = requests.get(url)  # Download the file

# Check if download was successful
if response.status_code == 200:
    # Save the content to a local file
    with open("output_file.csv", "wb") as f:
        f.write(response.content)
    # Read the local file using pandas
    data = pd.read_csv("output_file.csv")
else:
    print("Error downloading the file")

# Inspect the first and last few rows
data.head()
data.tail()
```

### Output
A DataFrame with columns `URL_ID` and `URL`, e.g.:
|   | URL_ID | URL |
|---|--------|-----|
|109| 146    | https://insights.blackcoffer.com/blockchain-fo... |
|110| 147    | https://insights.blackcoffer.com/the-future-of... |
|111| 148    | https://insights.blackcoffer.com/big-data-anal... |
|112| 149    | https://insights.blackcoffer.com/business-anal... |
|113| 150    | https://insights.blackcoffer.com/challenges-an... |

**Explanation:**
- The code downloads a CSV of article URLs and loads it into a DataFrame for further processing.

### Step-by-step Explanation
- **requests.get(url):** Downloads the CSV file from GitHub.
- **response.status_code:** Checks if the download was successful (200 = OK).
- **open(..., 'wb'):** Writes the file in binary mode.
- **pd.read_csv:** Loads the CSV into a DataFrame for easy manipulation.
- **data.head()/data.tail():** Lets you inspect the data.

### Contextual Notes
- If the download fails, check your internet connection or the URL.
- The CSV must have columns `URL_ID` and `URL`.
- You can replace the URL with your own CSV if needed.

---

## HTML Scraping & Storage

### Purpose
For each article URL, download the HTML, extract the main content, and save it locally. This avoids repeated web requests and speeds up development.

### Code (with comments)
```python
import os  # For file/folder operations
import requests
import pandas as pd
from bs4 import BeautifulSoup, NavigableString

# Ensure the 'data' folder exists
os.makedirs('data', exist_ok=True)

article_data = []  # List to store article info

for url in data['URL']:
    # Extract a unique title from the URL (e.g., the 4th segment)
    title = url.split('/')[3]
    file_name = title + '.html'
    file_path = './data/' + file_name

    # Download and save HTML if not already present
    if not os.path.exists(file_path):
        r = requests.get(url, headers={"User-Agent": "XY"})
        htmlcontent = r.content.decode(errors='replace')  # decode with fallback
        with open(file_path, 'w', encoding='utf-8') as f:
            f.write(htmlcontent)

    # Read the HTML file
    with open(file_path, 'r', encoding='utf-8') as f:
        htmlcontent = f.read()

    # Parse HTML and extract main content
    soup = BeautifulSoup(htmlcontent, 'html.parser')
    article_content = soup.find('div', attrs={'class': 'td-post-content'})

    if article_content is None:
        continue  # Skip if main content not found

    article_text = ''
    for element in article_content:
        if not isinstance(element, NavigableString):
            article_text += element.get_text()

    # Store title and text
    article_data.append({'Title': title, 'Text': article_text})

# Create DataFrame from scraped articles
df = pd.DataFrame(article_data)
print(df.head())
```

### Output
A DataFrame with columns `Title` and `Text`, e.g.:
|   | Title                                         | Text   |
|---|-----------------------------------------------|--------|
| 0 | ai-in-healthcare-to-improve-patient-outcomes  | ...    |
| 1 | what-if-the-creation-is-taking-over-the-creator | ...  |
| 2 | what-jobs-will-robots-take-from-humans-in-the-... | ... |
| 3 | will-machine-replace-the-human-in-the-future-o... | ... |
| 4 | will-ai-replace-us-or-work-with-us             | ...    |

**Explanation:**
- For each URL, the code scrapes the article content, saves it as an HTML file, and extracts the main text into a DataFrame.

### Step-by-step Explanation
- **os.makedirs('data', exist_ok=True):** Ensures the folder exists (no error if already present).
- **title = url.split('/')[3]:** Extracts a unique identifier from the URL for the filename.
- **requests.get(url, headers={...}):** Downloads the article HTML. The user-agent avoids being blocked.
- **htmlcontent = r.content.decode(errors='replace'):** Decodes bytes to string, replacing errors if any.
- **with open(..., 'w', encoding='utf-8'):** Saves HTML to disk.
- **BeautifulSoup(...):** Parses HTML for easy searching.
- **soup.find('div', ...):** Finds the main article content.
- **for element in article_content:** Loops through content, extracting text.
- **article_data.append(...):** Stores the result for later DataFrame creation.

### Contextual Notes
- If the HTML structure changes, `soup.find('div', ...)` may fail. Inspect the site if you get empty articles.
- Saving HTML locally is good for debugging and reproducibility.
- If you want to re-scrape, delete the `data/` folder first.

---

## Data Preprocessing

### Purpose
Clean the article text by tokenizing and removing stopwords. This reduces noise and improves sentiment analysis accuracy.

### Code (with comments)
```python
from nltk.corpus import stopwords
from nltk.tokenize import word_tokenize
import pandas as pd
import nltk

# Download required NLTK data
nltk.download('stopwords')
nltk.download('punkt')

stop_words = set(stopwords.words('english'))  # Common English stopwords

processed_texts = []
titles = []

for index, row in df.iterrows():
    title = row['Title']
    text = row['Text']

    # Tokenize the text (split into words)
    word_tokens = word_tokenize(text)

    # Remove stopwords (case-insensitive)
    filtered_tokens = [w for w in word_tokens if w.lower() not in stop_words]

    # Re-join tokens into a string
    new_sentence = ' '.join(filtered_tokens)

    processed_texts.append(new_sentence)
    titles.append(title)

# Create a new DataFrame with processed text
df_processed = pd.DataFrame({'Title': titles, 'Text': processed_texts})
print(df_processed.head())
```

### Output
A DataFrame with columns `Title` and `Text` (cleaned), e.g.:
|   | Title                                         | Text   |
|---|-----------------------------------------------|--------|
| 0 | ai-in-healthcare-to-improve-patient-outcomes  | ...    |
| 1 | what-if-the-creation-is-taking-over-the-creator | ...  |
| 2 | what-jobs-will-robots-take-from-humans-in-the-... | ... |
| 3 | will-machine-replace-the-human-in-the-future-o... | ... |
| 4 | will-ai-replace-us-or-work-with-us             | ...    |

**Explanation:**
- The text is tokenized and stopwords are removed, resulting in a cleaner version of each article for analysis.

### Step-by-step Explanation
- **nltk.download(...):** Ensures required NLTK data is present.
- **stopwords.words('english'):** Loads a list of common English words to ignore.
- **word_tokenize(text):** Splits text into words and punctuation.
- **filtered_tokens = ...:** Removes stopwords, case-insensitive.
- **' '.join(filtered_tokens):** Reassembles the cleaned text.

### Contextual Notes
- Removing stopwords helps focus on meaningful words.
- If you see errors about missing NLTK data, check your internet connection.
- You can customize the stopword list for your use case.

---

## Sentiment Analysis

### Purpose
Assign sentiment scores to each article using two methods:
- **VADER**: For positive/negative scores (good for social media/news)
- **TextBlob**: For polarity (overall sentiment) and subjectivity (fact vs. opinion)

### VADER Sentiment Scores (Positive & Negative)

#### Code (with comments)
```python
from nltk.sentiment import SentimentIntensityAnalyzer
import pandas as pd
import nltk

df3 = df_processed.copy()
nltk.download('vader_lexicon')  # Download VADER data

sia = SentimentIntensityAnalyzer()  # Initialize analyzer

positive_scores = []
negative_scores = []

for index, row in df_processed.iterrows():
    text = row['Text']
    sentiment_scores = sia.polarity_scores(text)
    positive_scores.append(sentiment_scores['pos'])
    negative_scores.append(sentiment_scores['neg'])

df3['Positive_Score'] = positive_scores
df3['Negative_Score'] = negative_scores
print(df3.head())
```

#### Output
|   | Title | Text | Positive_Score | Negative_Score |
|---|-------|------|----------------|---------------|
| 0 | ...   | ...  | 0.157          | 0.047         |
| 1 | ...   | ...  | 0.204          | 0.094         |
| 2 | ...   | ...  | 0.153          | 0.079         |
| 3 | ...   | ...  | 0.230          | 0.061         |
| 4 | ...   | ...  | 0.199          | 0.062         |

**Explanation:**
- VADER is used to compute positive and negative sentiment scores for each article.

#### Step-by-step Explanation
- **SentimentIntensityAnalyzer():** Loads VADER, a lexicon and rule-based sentiment tool.
- **sia.polarity_scores(text):** Returns a dict with 'pos', 'neu', 'neg', and 'compound' scores.
- **positive_scores/negative_scores:** Store the relevant values for each article.
- **df3['Positive_Score'] = ...:** Adds the scores as new columns.

#### Contextual Notes
- VADER is robust for news and social media text.
- Scores are between 0 and 1 (fraction of text that is positive/negative).
- If you want overall sentiment, use the 'compound' score.

### TextBlob Sentiment Scores (Polarity & Subjectivity)

#### Code (with comments)
```python
from textblob import TextBlob

df4 = df3.copy()
polarity_scores = []
subjectivity_scores = []

for index, row in df_processed.iterrows():
    text = row['Text']
    blob = TextBlob(text)
    polarity_scores.append(blob.sentiment.polarity)
    subjectivity_scores.append(blob.sentiment.subjectivity)

df4['Polarity_Score'] = polarity_scores
df4['Subjectivity_Score'] = subjectivity_scores
print(df4.head())
```

#### Output
|   | Title | Text | Positive_Score | Negative_Score | Polarity_Score | Subjectivity_Score |
|---|-------|------|----------------|---------------|----------------|--------------------|
| 0 | ...   | ...  | 0.157          | 0.047         | 0.0627         | 0.5028             |
| 1 | ...   | ...  | 0.204          | 0.094         | 0.0709         | 0.4042             |
| 2 | ...   | ...  | 0.153          | 0.079         | 0.0854         | 0.4778             |
| 3 | ...   | ...  | 0.230          | 0.061         | 0.1342         | 0.4915             |
| 4 | ...   | ...  | 0.199          | 0.062         | 0.0267         | 0.5024             |

**Explanation:**
- TextBlob is used to compute polarity (overall sentiment) and subjectivity (objectivity vs. opinion) for each article.

#### Step-by-step Explanation
- **TextBlob(text):** Creates a TextBlob object for the text.
- **blob.sentiment.polarity:** Ranges from -1 (negative) to 1 (positive).
- **blob.sentiment.subjectivity:** Ranges from 0 (objective) to 1 (subjective).
- **df4['...'] = ...:** Adds the scores as new columns.

#### Contextual Notes
- TextBlob is simple and effective for general sentiment.
- Polarity and subjectivity are useful for deeper analysis (e.g., is the article opinion or fact?).

---

## Saving Results

### Purpose
Save the final DataFrame with all sentiment scores to a CSV for further analysis or dashboarding.

### Code (with comments)
```python
df4.to_csv('convoproj.csv', index=False)  # index=False for cleaner CSV
```

### Output
A file `convoproj.csv` with all columns: Title, Text, Positive_Score, Negative_Score, Polarity_Score, Subjectivity_Score.

**Explanation:**
- The final DataFrame, containing all sentiment scores, is saved as a CSV for further analysis and dashboarding.

### Step-by-step Explanation
- **to_csv:** Writes the DataFrame to disk.
- **index=False:** Omits the row numbers from the CSV.

### Contextual Notes
- This CSV is the input for the Tableau dashboard.
- You can open it in Excel or any spreadsheet tool for inspection.

---

## Dashboard

### Purpose
Visualize the sentiment analysis results interactively using Tableau.

### How the CSV is Used
- The `convoproj.csv` file is imported into Tableau.
- Each row is an article, with all sentiment scores as columns.

### Dashboard Elements
- **Dropdown Menu:** Select articles by title.
- **Sentiment Overview:** Shows positive, negative, polarity, and subjectivity scores for the selected article.
- **Treemap:** Visualizes the distribution of sentiment scores across articles.
- **Word Cloud:** Shows the most frequent words in the articles.
- **Comparison Plots:** Compare sentiment scores across multiple articles.

### How to Interpret
- **Positive/Negative Score:** Higher = more positive/negative language.
- **Polarity Score:** >0 = positive, <0 = negative, 0 = neutral.
- **Subjectivity Score:** Closer to 1 = more opinionated, closer to 0 = more factual.

### Dashboard Link
[Sentiment Analysis Dashboard](https://public.tableau.com/views/SentimentAnalysisofArticles/Dashboard1?:language=en-GB&publish=yes&:sid=&:display_count=n&:origin=viz_share_link)

#### Dashboard Snapshots
- ![pic1](https://github.com/prabhmeharbedi/Article-Sentiment-Analysis/assets/91597702/17c88df7-8be5-48b3-b486-70638a600110)
- ![pic2](https://github.com/prabhmeharbedi/Article-Sentiment-Analysis/assets/91597702/33f6f8fc-0f29-49b3-b9e8-bb42b4fb3a2c)
- ![pic3](https://github.com/prabhmeharbedi/Article-Sentiment-Analysis/assets/91597702/ec3b3932-574d-4def-93c0-c08aacb051ec)
- ![pic4](https://github.com/prabhmeharbedi/Article-Sentiment-Analysis/assets/91597702/74744474-6707-43d4-a2ce-c13d1fc7d353)
- ![pic5](https://github.com/prabhmeharbedi/Article-Sentiment-Analysis/assets/91597702/c27ed804-548d-4bf9-8e2f-1c4ab2214916)

**Explanation:**
- The dashboard provides a visual and interactive way to explore the sentiment analysis results for all articles.

### Tips
- If you update the CSV, refresh the data source in Tableau.
- You can add more visualizations (e.g., time trends, sentiment by topic).

---

## Full Code Walkthrough

Below is a step-by-step walkthrough of every code cell, its output, and an explanation of what it does. (See above sections for atomic details.)

---

## Conclusion

This project demonstrates a full, atomic pipeline for:
- Web scraping
- Text preprocessing
- Sentiment analysis (multiple methods)
- Saving and visualizing results

**Extensions:**
- Add more advanced NLP (e.g., named entity recognition, topic modeling)
- Use deep learning models for sentiment
- Automate dashboard updates

**Troubleshooting:**
- If you get empty articles, check the HTML structure.
- If NLTK downloads fail, check your internet connection.
- If you get encoding errors, try `errors='replace'` or check file encodings.

---

*End of super-atomic walkthrough.* 