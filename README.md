# pet_finder.py
# SAB Pet Finder – Einfacher Pet Finder mit Petfinder API
# Voraussetzungen: Python 3.x, pip install requests streamlit

import requests
import streamlit as st

# --- Petfinder API Zugangsdaten ---
API_KEY = "DEIN_PETFINDER_API_KEY"      # <--- Hier deinen API Key einfügen
API_SECRET = "DEIN_PETFINDER_API_SECRET" # <--- Hier dein API Secret einfügen

# --- Funktion um Access Token zu holen ---
def get_access_token():
    url = "https://api.petfinder.com/v2/oauth2/token"
    data = {
        "grant_type": "client_credentials",
        "client_id": API_KEY,
        "client_secret": API_SECRET
    }
    resp = requests.post(url, data=data)
    resp.raise_for_status()
    return resp.json()["access_token"]

# --- Funktion um Tiere zu suchen ---
def search_pets(token, animal, location, age):
    url = f"https://api.petfinder.com/v2/animals"
    headers = {"Authorization": f"Bearer {token}"}
    params = {
        "type": animal,
        "location": location,
        "age": age,
        "limit": 20
    }
    resp = requests.get(url, headers=headers, params=params)
    resp.raise_for_status()
    return resp.json().get("animals", [])

# --- Streamlit UI ---
st.set_page_config(page_title="SAB Pet Finder", layout="wide")
st.title("🐾 SAB Pet Finder")

with st.sidebar:
    st.header("Filter")
    animal = st.selectbox("Tierart", ["Dog", "Cat", "Bird", "Small & Furry", "Horse", "Barnyard"])
    location = st.text_input("PLZ / Stadt", "12345")
    age = st.selectbox("Alter", ["Baby", "Young", "Adult", "Senior"])
    search_btn = st.button("Tiere suchen")

if search_btn:
    try:
        token = get_access_token()
        pets = search_pets(token, animal, location, age)
        if not pets:
            st.info("Keine Tiere gefunden 😿")
        else:
            for pet in pets:
                st.subheader(pet["name"])
                st.write(f"Art: {pet['species']}, Alter: {pet['age']}")
                if pet["photos"]:
                    st.image(pet["photos"][0]["medium"], width=250)
                st.markdown(f"[Mehr Info]({pet['url']})")
    except Exception as e:
        st.error(f"Fehler beim Abrufen der Daten: {e}")
