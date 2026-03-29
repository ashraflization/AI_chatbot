import streamlit as st
import requests

# Configuration
TMDB_API_KEY = "your_api_key_here"  # Get one at themoviedb.org
BASE_URL = "https://api.themoviedb.org/3"

def get_recommendations(movie_name):
    # 1. Search for the movie ID
    search_url = f"{BASE_URL}/search/movie?api_key={TMDB_API_KEY}&query={movie_name}"
    response = requests.get(search_url).json()
    
    if not response['results']:
        return None
    
    movie_id = response['results'][0]['id']
    
    # 2. Get recommendations based on that ID
    rec_url = f"{BASE_URL}/movie/{movie_id}/recommendations?api_key={TMDB_API_KEY}"
    rec_data = requests.get(rec_url).json()
    return rec_data['results'][:3]

# --- Streamlit UI ---
st.set_page_config(page_title="CineBot", page_icon="🍿")
st.title("🎬 CineBot: Movie Recommender")

if "messages" not in st.session_state:
    st.session_state.messages = [{"role": "assistant", "content": "Hi! Tell me a movie you loved, and I'll find something similar."}]

# Display chat history
for msg in st.session_state.messages:
    st.chat_message(msg["role"]).write(msg["content"])

# User Input
if prompt := st.chat_input():
    st.session_state.messages.append({"role": "user", "content": prompt})
    st.chat_message("user").write(prompt)

    # Logic
    with st.spinner("Searching the archives..."):
        recs = get_recommendations(prompt)
        
        if recs:
            response = f"Since you liked **{prompt}**, you might enjoy these:\n\n"
            for m in recs:
                response += f"- **{m['title']}** ({m['release_date'][:4]})\n"
        else:
            response = "I couldn't find that movie. Try another title!"

    st.session_state.messages.append({"role": "assistant", "content": response})
    st.chat_message("assistant").write(response)
