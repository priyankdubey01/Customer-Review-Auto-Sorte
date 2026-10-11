# Customer-Review-Auto-Sorte

A Python NLP automation project that reads 100,000 customer reviews in chunks, classifies sentiment with VADER, tags topics (delivery, price, quality, service), and raises real-time alerts for angry reviews.

Table of Contents
Overview
Features
How It Works
Tech Stack
Installation
Usage
Code Walkthrough
Sample Output
Customization
Project Structure
Future Improvements
License
Overview

A shop can receive thousands of customer reviews, and nobody has time to read them all. This project builds a small program that reads them automatically and:

Decides whether each review is happy, neutral or angry (sentiment analysis).
Decides what each review is about (keyword-based topic categorization).
Watches new reviews as they arrive and raises an alarm for angry ones.
Saves angry reviews to a CSV file that acts as a ready-made to-do list for the support team.

No human reads anything, yet the right people see the angry reviews immediately.

Features
Large dataset generation: creates 100,000 sample reviews for testing.
Chunked processing: reads the file 20,000 rows at a time, so huge files never have to fit in memory.
Sentiment analysis: uses VADER (compound score from -1 to +1) with no model training required.
Topic categorization: keyword matching for delivery, price, quality and service.
Data visualization: bar chart of reviews by topic and mood.
Real-time alerts: simulated live feed that prints alarms and exports urgent reviews to urgent_reviews.csv.
How It Works
Step	What the program does	Concept
1	Creates 100,000 sample reviews	Large dataset generation
2	Reads them in small pieces	Memory-efficient processing
3	Decides if each review is happy or angry	Sentiment analysis (NLP)
4	Decides what it is about	Keyword categorization (NLP)
5	Watches new reviews and raises alarms	Automation and real-time monitoring
reviews.csv  -->  read in chunks  -->  mood  -->  topic  -->  chart
                                                     \
new live reviews  ------------------------>  mood + topic  -->  ALARM + urgent_reviews.csv
Tech Stack
Python 3.9+
pandas: data handling and chunked CSV reading
matplotlib: charts
vaderSentiment: rule-based sentiment scoring
Installation
bash
# 1. Clone the repository
git clone https://github.com/<your-username>/customer-review-auto-sorter.git
cd customer-review-auto-sorter

# 2. Install dependencies
pip install pandas matplotlib vaderSentiment
Usage

Run the script (or the notebook) from top to bottom:

bash
python review_auto_sorter.py

It will:

Generate reviews.csv with 100,000 sample reviews.
Score the mood of every review in chunks.
Assign topics and display the topic-vs-mood chart.
Run the live-feed simulation and save urgent_reviews.csv.

Using your own data? Skip the generation step and point the script to your CSV. It only needs a column named review.

Code Walkthrough
Mood detection
python
def mood(text):
    score = reader.polarity_scores(text)["compound"]   # -1 to +1
    if score >= 0.3:
        return "happy"
    if score <= -0.3:
        return "angry"
    return "neutral"

VADER returns a compound score from -1 (very negative) to +1 (very positive). Scores of 0.3 or above are labeled happy, -0.3 or below angry, and anything in between neutral.

Chunked reading
python
parts = []
for piece in pd.read_csv("reviews.csv", chunksize=20_000):
    piece["mood"] = piece["review"].map(mood)
    parts.append(piece)

data = pd.concat(parts)

chunksize=20_000 hands pandas the file in pieces, .map(mood) labels every review, and pd.concat joins the results.

Topic detection
python
keywords = {
    "delivery": ["delivery", "shipping"],
    "price":    ["price", "cost"],
    "quality":  ["quality", "build"],
    "service":  ["service"],
}

def topic_of(text):
    text = text.lower()
    for topic, words in keywords.items():
        if any(w in text for w in words):
            return topic
    return "other"

The first topic whose keyword appears in the review wins. If nothing matches, the topic is other.

Real-time alerts
python
urgent = []
for round_number in range(1, 6):          # use "while True:" to run forever
    for text in new_reviews():
        if mood(text) == "angry":
            urgent.append({"review": text, "topic": topic_of(text)})
            print(f"ALARM [{topic_of(text)}]: {text}")
    time.sleep(1)

pd.DataFrame(urgent).to_csv("urgent_reviews.csv", index=False)

new_reviews() simulates a live feed. Each new review goes through the same mood() and topic_of() helpers, and angry ones are alarmed and saved.

Sample Output

Topic vs mood counts

Topic	Angry	Happy
delivery	12,434	12,371
price	12,355	12,647
quality	12,515	12,609
service	12,522	12,547

The counts are nearly balanced because the sample data is randomly generated. With real reviews, the imbalance shows where customers are unhappy.

Live alerts

--- Check 1 ---
  ALARM [price]: The price is disappointing
  ALARM [delivery]: Shipping was very poor
  ALARM [delivery]: Shipping was horrible
...
Saved 13 urgent reviews to urgent_reviews.csv

Add a screenshot of your chart here: ![Reviews by topic and mood](images/chart.png)

Customization
Goal	What to change
Use real reviews	Point the reader to your CSV (needs a review column)
Use a real live feed	Rewrite new_reviews() to call an API, e.g. requests.get(...)
Get alerts in Slack or email	Replace the print("ALARM...") line with a webhook or email call
Run forever	Change for round_number in range(1, 6): to while True:
Add more topics	Add lines to the keywords dictionary
Handle sarcasm or other languages	Replace VADER with a transformers sentiment model
Project Structure
customer-review-auto-sorter/
├── review_auto_sorter.py    # main script (or review_auto_sorter.ipynb)
├── reviews.csv              # generated sample dataset (100,000 reviews)
├── urgent_reviews.csv       # generated list of angry reviews
├── requirements.txt         # pandas, matplotlib, vaderSentiment
└── README.md
Future Improvements
Connect to a real review source (API or database)
Send alerts through Slack, email or Microsoft Teams
Replace keyword matching with a trained topic classifier
Use a transformer-based sentiment model for sarcasm and multilingual reviews
Build a dashboard (Streamlit) for live monitoring
