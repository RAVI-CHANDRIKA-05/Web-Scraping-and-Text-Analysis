# Web Scraping and Text Analysis

## Table of Contents
1. [Introduction](#introduction)
2. [Project Overview](#project-overview)
3. [Project Steps](#project-steps)
    - [Data Extraction with BeautifulSoup](#data-extraction-with-beautifulsoup)
    - [Text Cleaning and Analysis](#text-cleaning-and-analysis)
4. [Utilizing Scores and Output Data from Text Analysis](#utilizing-scores-and-output-data-from-text-analysis)
5. [Conclusion](#conclusion)

## Introduction <a name="introduction"></a>
This document provides an overview of a Python project involving web scraping using BeautifulSoup and text analysis techniques. The project focuses on extracting textual data from various URLs, cleaning the text, and performing analysis to compute various variables such as sentiment scores, readability indices, and more.

## Project Overview <a name="project-overview"></a>
The project involves two main phases:
1. **Data Extraction with BeautifulSoup**: This phase focuses on extracting data from given URLs using the BeautifulSoup library in Python. It includes tasks such as scraping titles and text bodies from the URLs provided in an input file.

2. **Text Cleaning and Analysis**: In this phase, the extracted text data undergoes cleaning processes such as removing HTML tags and stopwords. Then, various text analysis techniques are applied to compute variables such as sentiment scores, readability indices, and more.


## Project Steps <a name="project-steps"></a>

### A. Data Extraction with BeautifulSoup <a name="data-extraction-with-beautifulsoup"></a>

In this phase, the goal is to extract textual data from the given URLs using the BeautifulSoup library in Python. The process involves the following tasks:

#### Task #1: Scrape title from the URL
The first task focuses on scraping the title of the article from each URL. This involves parsing the HTML content of the webpage and extracting the title element, typically found within an `<h1>` tag with a specific class or identifier.

```python
article_title = soup.find('h1', class_ ='entry-title')
raw_html = article_title.prettify()
```

#### Task #2: Scrape text body from the URL
The second task involves extracting the main text body of the article from each URL. This requires identifying the HTML element containing the article content, which can vary depending on the webpage structure. Once identified, the text is extracted for further analysis.

```python
article_text = soup.find('div', class_ ='td-post-content')
article_text_ = article_text.get_text()
```

#### Task #3: Extract Article Titles of all the URLs in the input file
The third task extends the extraction process to multiple URLs provided in an input file. The goal is to programmatically iterate through each URL, scrape the article title, and store them for further processing. This task streamlines the extraction process for a batch of URLs, allowing for efficient data collection and analysis.


### B. Text Cleaning and Analysis <a name="text-cleaning-and-analysis"></a>

In this phase, the extracted textual data is preprocessed to ensure quality and then subjected to various analyses. The process comprises two main components: Text Cleaning and Text Analysis.

#### Text Cleaning:

1. **Remove HTML tags from the text:** The extracted text often contains HTML tags, which need to be removed for further analysis. This involves parsing the text and eliminating any HTML elements present.

```python
#using regex to remove html tags and extra spaces before and after the text
CLEANR = re.compile('<.*?>') 

def cleantitle(raw_html):
    atext = re.sub(CLEANR, '', raw_html).rstrip().lstrip()
    return atext
raw_html = article_title.prettify()
cleantitle(raw_html)
```

2. **Remove extra spaces and special characters:** To standardize the text, any extra spaces and special characters are removed. This ensures consistency and facilitates accurate analysis.

3. **Remove stopwords and custom-defined stopwords:** Stopwords, commonly occurring words like "the," "is," and "and," are removed from the text to focus on meaningful content. Additionally, custom-defined stopwords specific to the domain may also be eliminated.

```python
StopWords_GenericLong=word_tokenize(read_atext)
StopWords_GenericLong = [i for i in StopWords_GenericLong if len(i) > 1]#keep words not letters

full_stopwords = full_stopwords+StopWords_GenericLong
```

#### Text Analysis:

1. **Sentiment analysis using predefined word lists:** Sentiment analysis is performed using predefined lists of positive and negative words. Each word in the text is matched against these lists to determine the positivity and negativity scores.

Text from articles is converted into a list of tokens using the nltk tokenize module and these tokens are used to calculate the 4 variables described below:
* Positive Score: This score is calculated by assigning the value of +1 for each word if found in the Positive Dictionary and then adding up all the values.
* Negative Score: This score is calculated by assigning the value of -1 for each word if found in the Negative Dictionary and then adding up all the values. We multiply the score with -1 so that the score is a positive number.

```python
def cleanedwords(textbody):
    #article_text = title_body['text'][i]
    article_text = textbody.lower()
    article_words = tokenizer.tokenize(article_text)
    article_words = [word for word in article_words if word not in eng_stopwords] #remove english stop words from nltk
    article_words = [word for word in article_words if word not in full_stopwords]#remove custome stop words
    return article_words

#from the list of curated positive and negative words, calculate the positive and negative score
def getscore(textbody): 
    cleaned_words = cleanedwords(textbody)
    pos_score = len([word for word in cleaned_words if word in positivewords])
    neg_score = len([word for word in cleaned_words if word in negativewords])
    return [pos_score, neg_score]
```

2. **Compute polarity and subjectivity scores:** Polarity and subjectivity scores are computed to quantify the overall sentiment and objectivity of the text, respectively. These scores provide insights into the tone and nature of the content.

**Polarity Score:**  Determines if a given text is positive or negative in nature. It is calculated by using the formula:

    Polarity Score = (Positive Score – Negative Score)/ ((Positive Score + Negative Score) + 0.000001) 
Range is from -1 to +1

**Subjectivity Score:** Determines if a given text is objective or subjective. It is calculated by using the formula: 
        
    Subjectivity Score = (Positive Score + Negative Score)/ ((Total Words after cleaning) + 0.000001)
Range is from 0 to +1

```python
#make a list of all positive words
#make a list of all negative words

#get total cleaned words
def cleanedwordslen(textbody):
    cleaned_words_length = len(cleanedwords(textbody)) #total words after cleaning len()for count
    return cleaned_words_length

title_body['POLARITY SCORE'] = title_body.apply(lambda row : (row['POSITIVE SCORE']-row['NEGATIVE SCORE'])/((row['POSITIVE SCORE']+row['NEGATIVE SCORE'])+0.000001), axis = 1)
title_body['SUBJECTIVITY SCORE'] = title_body.apply(lambda row : (row['POSITIVE SCORE']+row['NEGATIVE SCORE'])/(cleanedwordslen(row['text'])+0.000001), axis = 1)
```

3. **Calculate readability indices (e.g., Gunning Fog Index):** Readability indices, such as the Gunning Fog Index, are calculated to evaluate the complexity and ease of comprehension of the text. This helps assess the readability level of the content.


Analysis of Readability is calculated using the Gunning Fox index formula described below.

* Average Sentence Length = the number of words / the number of sentences
* Percentage of Complex words = the number of complex words / the number of words
* Fog Index = 0.4 * (Average Sentence Length + Percentage of Complex words)<br>

Readability is the ease with which a reader can understand a written text. In natural language, the readability of text depends on its content (the complexity of its vocabulary and syntax). It focuses on the words we choose, and how we put them into sentences and paragraphs for the readers to comprehend.

   - **The Gunning fog Formula**

        Grade level= 0.4 * ( (average sentence length) + (percentage of Hard Words) )

Here, Hard Words = words with more than two syllables.


   - **Average Number of Words Per Sentence**
    
            the total number of words / the total number of sentences

   - **Complex Word Count:** The number of syllables in each word is determined to gauge the linguistic complexity of the text. Words with more than two syllables are considered as complex words.

   - **Word Count:**
Total cleaned words present in the text are counted by 

            removing the stop words (using stopwords class of nltk package).
            removing any punctuations like ? ! , . from the word before counting.

   - **Average word length:** The average length of words in the text is calculated to understand the average size of words used. 
        
            Sum of the total number of characters in each word/Total number of words

   - **Syllable Count Per Word**
Counted the number of Syllables in each word of the text by counting the vowels present in each word. Also words ending with "es","ed"  are not counted as a syllable.

   - **Count personal pronouns:** The occurrence of personal pronouns, such as "I," "we," "my," etc., is counted to identify the degree of personalization in the text.

These cleaning and analysis steps prepare the textual data for further processing and insights generation.

## Utilizing Scores and Output Data from Text Analysis <a name="utilizing-scores-and-output-data-from-text-analysis"></a>

**Sentiment Analysis Insights:**
- Identify trends and patterns in sentiment over time or across different sources.
- Gauge public opinion on specific topics or products.
- Monitor customer feedback and sentiment towards a brand or product.
- Understand the overall sentiment of articles or news stories related to a particular topic or event.

**Readability Analysis:**
- Assess the complexity of written content and ensure it matches the intended audience's reading level.
- Optimize content for readability by adjusting sentence structure, vocabulary, and complexity.
- Compare the readability of different documents or websites to determine which are more accessible or user-friendly.

**Subjectivity and Objectivity Scores:**
- Determine the level of bias or objectivity in news articles, blog posts, or social media content.
- Evaluate the credibility and reliability of information sources based on their subjectivity scores.
- Understand the degree of personalization or opinion present in written content.

**Text Complexity and Linguistic Analysis:**
- Identify complex language patterns and jargon that may be challenging for certain audiences to understand.
- Analyze the linguistic characteristics of text to gain insights into authorship or writing style.
- Assess the level of expertise required to comprehend technical or specialized content.

**Application in Content Creation and Marketing:**
- Optimize content marketing strategies based on sentiment analysis of customer feedback or social media mentions.
- Tailor messaging and communication strategies to align with audience preferences and sentiment.
- Identify opportunities for content improvement and refinement based on readability and linguistic analysis.

**Decision Support and Insights Generation:**
- Inform business decisions based on sentiment analysis of market trends, competitor activities, or customer sentiments.
- Identify potential risks or opportunities based on sentiment trends and patterns.
- Generate actionable insights for stakeholders based on comprehensive text analysis.

Overall, the scores and output data from the text analysis and web scraping project provide valuable insights into the sentiment, readability, and linguistic characteristics of textual data. By leveraging these insights, organizations can make data-driven decisions, improve communication strategies, and enhance overall content quality and effectiveness.

## Conclusion <a name="conclusion"></a>
This project demonstrates the use of web scraping techniques and text analysis methodologies to extract valuable insights from textual data sourced from various URLs. The implemented Python code utilizes BeautifulSoup for web scraping and various libraries such as NLTK and Textstat for text analysis.
