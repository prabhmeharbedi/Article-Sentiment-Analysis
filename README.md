# Article-Sentiment-Analysis


## Project Overview

In this sentiment analysis project, articles are systematically scraped from 
a website, and sentiment analysis techniques are applied to unveil the 
emotional tone of the content. The project culminates in the creation of 
an interactive dashboard that vividly displays sentiment analysis results.



## Technique used for Data Collection

### 1. HTML Scraping:

- The code uses the ‘requests’ library to make HTTP requests to the specified
URLs.

- For each URL, the HTML content of the web page is retrieved.


### 2. BeautifulSoup Parsing:

- The ‘BeautifulSoup’ library is employed to parse the HTML content and
extract specific information.

- It searches for a ‘div’ element with the class 'td-post-content' using
BeautifulSoup's ‘find’ method.


### 3. File Storage:

- The HTML content is stored in individual files, where each file is named
after a unique identifier extracted from the URL.



## Technique used for Data Description

- The dataset being collected contain information from different articles and
web pages, and each URL is associated with a specific piece of content.

- The columns in the dataset include the following details:

- URL_ID: A unique identifier extracted from the URL (e.g., a part of the
URL split).

- URL: The web address of the article or content.

- The HTML content retrieved from each URL is stored in separate files within
the 'data' folder. These HTML files presumably contain the text content, which
is later processed in subsequent sections of the code and observations are stored
in ‘convoproj.csv’.



## Data Pre-Processing Technique used

### 1. Tokenization:

- The text data is tokenized using the ‘word_tokenize’ function from the
‘nltk.tokenize’ module. Tokenization breaks down the text into individual
words or tokens.


### 2. Stopword Removal:

- Stopwords (common words like "the," "and," "is") are removed from the
tokenized text using a list of English stopwords from the ‘nltk.corpus’ module.


### 3. Text Cleaning:

- The code iterates through each row of the DataFrame, processes the text
data, and creates a new DataFrame (‘df_processed’) with the processed text.

- For each row, it tokenizes the text, removes stopwords, and joins the filtered
tokens back into a sentence.


### 4. New DataFrame Creation:

- A new DataFrame (‘df_processed’) is created with columns ‘Title’ and
‘Text’, where ‘Text’ contains the processed and cleaned text data.


## Resulting DataFrame (‘df_processed’)

- The DataFrame ‘df_processed’ contains two columns: ‘Title’ and ‘Text’.

- ‘Title’ corresponds to the titles of the articles.

- ‘Text’ contains the processed and cleaned text data after tokenization and stopword removal.


## Link to Dashbaord - <a href = "https://public.tableau.com/views/SentimentAnalysisofArticles/Dashboard1?:language=en-GB&publish=yes&:sid=&:display_count=n&:origin=viz_share_link">Click Here</a>


## About the Dashbaord
In Dashboard we have the option to choose articles based on their
titles using a dropdown menu. Upon selecting a specific article, you
will receive an overview of the sentiment analysis, including positive
and negative scores, polarity scores, subjective score, a treemap, a
word cloud, and plots comparing various articles and their
corresponding scores

## Snapshots of Dashboard

### Image - 1
![pic1](https://github.com/prabhmeharbedi/Article-Sentiment-Analysis/assets/91597702/17c88df7-8be5-48b3-b486-70638a600110)

### Image - 2
![pic2](https://github.com/prabhmeharbedi/Article-Sentiment-Analysis/assets/91597702/33f6f8fc-0f29-49b3-b9e8-bb42b4fb3a2c)

### Image - 3
![pic3](https://github.com/prabhmeharbedi/Article-Sentiment-Analysis/assets/91597702/ec3b3932-574d-4def-93c0-c08aacb051ec)

### Image - 4
![pic4](https://github.com/prabhmeharbedi/Article-Sentiment-Analysis/assets/91597702/74744474-6707-43d4-a2ce-c13d1fc7d353)

### Image - 5
![pic5](https://github.com/prabhmeharbedi/Article-Sentiment-Analysis/assets/91597702/c27ed804-548d-4bf9-8e2f-1c4ab2214916)
