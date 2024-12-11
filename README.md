# chatbot-
gives you the time required to study and news of countries

 Problem Statement
Many students struggle with time management, underestimating the hours required for effective study. Additionally, they face difficulties in staying informed about current events relevant to their studies. This project addresses these issues by providing a chatbot capable of predicting study hours and fetching curated news.
Motivation
With increasing academic pressures, there is a need for tools that can assist students in managing their time efficiently. The dual functionality of this chatbot—predicting study hours and aggregating news—is designed to enhance the learning experience. Its relevance is bolstered by the growing adoption of digital learning tools.
Objectives
1.	Develop a chatbot to predict study hours based on user input and historical data.
2.	Integrate a news aggregation system tailored to students’ interests.
3.	Ensure user-friendly interfaces and seamless functionality.
Scope of the Project
The project focuses on creating a freemium chatbot that combines educational and informational services. While the current scope is limited to study hour predictions and news aggregation, future iterations can include additional functionalities like exam preparation tips and internationalization


import streamlit as st
import spacy
import requests

# Constants
API_KEY = '1601a3deef304678bc83f51009951aba'  
BASE_URL_TOP_HEADLINES = 'https://newsapi.org/v2/top-headlines'
BASE_URL_EVERYTHING = 'https://newsapi.org/v2/everything'

# Dataset for chatbot advice
dataset = {
    "Engineering": {
        "Computer Science": {
            "Subjects": ["Data Structures", "Algorithms", "Operating Systems"],
            "Hours_per_day": 4,
            "Tips": "Focus on coding practice and understanding algorithms."
        },
        "Mechanical": {
            "Subjects": ["Thermodynamics", "Fluid Mechanics", "Machine Design"],
            "Hours_per_day": 5,
            "Tips": "Concentrate on numerical problems and conceptual clarity."
        }
    },
    "Medical": {
        "General Medicine": {
            "Subjects": ["Anatomy", "Physiology", "Pathology"],
            "Hours_per_day": 6,
            "Tips": "Regularly revise diagrams and clinical cases."
        },
        "Zoology": {
            "Subjects": ["Animal Anatomy", "Ecology", "Taxonomy"],
            "Hours_per_day": 4,
            "Tips": "Focus on evolutionary trends and classification."
        }
    },
    "News": {
        "Topics": ["general", "politics", "technology", "sports", "business", "entertainment", "health", "science"],
        "Countries": {
            "us": "United States",
            "uk": "United Kingdom",
            "in": "India",
            "ca": "Canada",
            "au": "Australia"
        }
    }
}

# Load spaCy NLP model
nlp = spacy.load("en_core_web_sm")

# Function to fetch news
def fetch_news(country='us', topic='general'):
    try:
        if topic == 'general':
            url = f"{BASE_URL_TOP_HEADLINES}?country={country}&apiKey={API_KEY}"
        else:
            url = f"{BASE_URL_EVERYTHING}?q={topic}&apiKey={API_KEY}&language=en"

        response = requests.get(url)
        response.raise_for_status()
        data = response.json()

        if data.get('status') == 'ok':
            return data.get('articles', [])
        else:
            st.error(f"Error from NewsAPI: {data.get('message')}")
            return []
    except requests.exceptions.RequestException as e:
        st.error(f"An error occurred while fetching news: {e}")
        return []

# Streamlit App
def main():
    st.title("Chatbot & News App")
    st.write("Ask me anything about education or request news updates!")

    user_query = st.text_input("Your Query (e.g., 'What should a Mechanical Engineering student study?' or 'Show me tech news from India')", "")

    if st.button("Submit"):
        if user_query:
            doc = nlp(user_query)
            detected_field = None
            detected_subfield = None
            country = "us"  # Default country
            topic = "general"  # Default topic

            # Check for matching fields in the dataset
            for token in doc:
                for field in dataset:
                    if token.text.lower() in field.lower():
                        detected_field = field
                        break
                if detected_field:
                    break

            # Handle "News" queries
            if detected_field == "News":
                # Extract country and topic if mentioned in the query
                for token in doc:
                    token_lower = token.text.lower()
                    if token_lower in dataset["News"]["Countries"]:
                        country = token_lower
                    elif token_lower in dataset["News"]["Topics"]:
                        topic = token_lower

                st.subheader(f"Top {topic.capitalize()} News from {dataset['News']['Countries'].get(country, country.upper())}")
                articles = fetch_news(country, topic)
                if articles:
                    for article in articles[:10]:  # Show up to 10 headlines
                        image_url = article.get('urlToImage', None)
                        if image_url:
                            st.image(image_url)
                        st.write(f"📰 {article.get('title', 'No Title')}")
                        st.write(f"🔗 [Read more]({article.get('url', '#')})")
                else:
                    st.write("No news articles found for the selected country or topic.")

            # Handle education queries
            elif detected_field:
                for token in doc:
                    for subfield in dataset[detected_field]:
                        if token.text.lower() in subfield.lower():
                            detected_subfield = subfield
                            break
                    if detected_subfield:
                        break

                if detected_field and detected_subfield:
                    st.subheader(f"{detected_subfield} ({detected_field}) Recommendations")
                    subject_data = dataset[detected_field][detected_subfield]
                    st.write(f"- **Subjects to focus on:** {', '.join(subject_data['Subjects'])}")
                    st.write(f"- **Recommended study hours per day:** {subject_data['Hours_per_day']} hours")
                    st.write(f"- **Study Tips:** {subject_data['Tips']}")
                elif detected_field:
                    st.subheader(f"{detected_field} Recommendations")
                    st.write("Specify the branch or subfield for more precise advice!")
                else:
                    st.error("Sorry, I couldn't understand your query. Please include a field like Engineering, Medical, or News.")
            else:
                st.error("Please enter a valid query!")
        else:
            st.warning("Please enter a query!")

if __name__ == "__main__":
    main()

                    
