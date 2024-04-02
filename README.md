# DS_YouTube-Data-Harvesting-and-Warehousing-using-SQL-and-Streamlit
DS_YouTube Data Harvesting and Warehousing using SQL and Streamlit
import streamlit as st
import sqlite3
from googleapiclient.discovery import build

# Function to create SQLite database and tables
def create_database():
    conn = sqlite3.connect('youtube_data.db')
    c = conn.cursor()
    # Create tables for videos and comments
    c.execute('''CREATE TABLE IF NOT EXISTS videos 
                 (video_id TEXT PRIMARY KEY, title TEXT, description TEXT, published_at TEXT, channel_id TEXT)''')
    c.execute('''CREATE TABLE IF NOT EXISTS comments 
                 (comment_id TEXT PRIMARY KEY, video_id TEXT, text TEXT, author_name TEXT, published_at TEXT)''')
    conn.commit()
    conn.close()

# Function to fetch YouTube video data using API
def fetch_video_data(api_key, query):
    youtube = build('youtube', 'v3', developerKey=api_key)
    request = youtube.search().list(q=query, part='id,snippet', type='video', maxResults=10)
    response = request.execute()
    return response['items']

# Function to store video data in SQLite database
def store_video_data(video_data):
    conn = sqlite3.connect('youtube_data.db')
    c = conn.cursor()
    for item in video_data:
        video_id = item['id']['videoId']
        title = item['snippet']['title']
        description = item['snippet']['description']
        published_at = item['snippet']['publishedAt']
        channel_id = item['snippet']['channelId']
        c.execute('''INSERT OR IGNORE INTO videos (video_id, title, description, published_at, channel_id) 
                     VALUES (?, ?, ?, ?, ?)''', (video_id, title, description, published_at, channel_id))
    conn.commit()
    conn.close()

# Streamlit UI
def main():
    st.title('YouTube Data Harvesting and Warehousing')

    # Create database if not exists
    create_database()

    # Get API key from user input
    api_key = st.text_input('Enter your YouTube Data API key:')
    
    # Get search query from user input
    query = st.text_input('Enter search query:')
    
    # Fetch and display video data
    if st.button('Fetch Video Data'):
        video_data = fetch_video_data(api_key, query)
        store_video_data(video_data)
        st.success('Video data fetched and stored successfully!')
        st.write('Displaying video data:')
        for item in video_data:
            st.write('Title:', item['snippet']['title'])
            st.write('Description:', item['snippet']['description'])
            st.write('Published At:', item['snippet']['publishedAt'])
            st.write('Channel ID:', item['snippet']['channelId'])
            st.write('Video ID:', item['id']['videoId'])
            st.write('---')

if _name_ == '_main_':
    main()
