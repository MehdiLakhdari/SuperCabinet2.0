import streamlit as st
import firebase_admin
from firebase_admin import credentials, auth, firestore
import datetime

# --- CONFIGURATION DE LA PAGE ---
st.set_page_config(page_title="Cabinet Médical Pro", page_icon="🏥", layout="wide")

# --- CONNEXION FIREBASE ---
# INITIALISATION : C'est ici que la magie opère avec ton fichier JSON
if not firebase_admin._apps:
    try:
        cred = credentials.Certificate("https://supercabinet-default-rtdb.europe-west1.firebasedatabase.app.json")
        firebase_admin.initialize_app(cred)
    except Exception as e:
        st.error(f"Erreur de connexion Firebase : {e}")

db = firestore.client()

# --- DESIGN PERSONNALISÉ ---
st.markdown("""
    <style>
    .main { background-color: #f1f5f9; }
    .stButton>button { width: 100%; border-radius: 10px; background-color: #2563eb; color: white; font-weight: bold; height: 3em; }
    .medical-card { background-color: white; padding: 25px; border-radius: 15px; border-left: 5px solid #2563eb; box-shadow: 0 4px 6px rgba(0,0,0,0.05); }
    </style>
    """, unsafe_allow_html=True)

# --- NAVIGATION ---
st.sidebar.title("🏥 Menu Principal")
page = st.sidebar.radio("Navigation :", ["Accueil", "Prendre RDV", "Biographie", "Localisation", "Espace Patient"])

# --- SECTION : ACCUEIL ---
if page == "Accueil":
    st.title("Bienvenue au Cabinet Médical")
    st.info("Plateforme de gestion des soins et des rendez-vous.")
    
    col1, col2, col3 = st.columns(3)
    with col1:
        st.markdown('<div class="medical-card"><h3>📅 RDV</h3><p>Réservez votre créneau en 2 clics.</p></div>', unsafe_allow_html=True)
    with col2:
        st.markdown('<div class="medical-card"><h3>👨‍⚕️ Expert</h3><p>Un suivi personnalisé et moderne.</p></div>', unsafe_allow_html=True)
    with col3:
        st.markdown('<div class="medical-card"><h3>📍 Draria</h3><p>Votre cabinet de proximité.</p></div>', unsafe_allow_html=True)

# --- SECTION : PRENDRE RDV ---
elif page == "Prendre RDV":
    st.title("📅 Formulaire de Réservation")
    with st.container():
        nom = st.text_input("Nom et Prénom du patient")
        date_selectionnee = st.date_input("Date", min_value=datetime.date.today())
        heure_selectionnee = st.time_input("Heure")
        motif = st.text_area("Motif de consultation")
        
        if st.button("Confirmer la réservation"):
            if nom:
                # ENREGISTREMENT DANS FIRESTORE
                doc_ref = db.collection("consultations").document()
                doc_ref.set({
                    "patient": nom,
                    "date": str(date_selectionnee),
                    "heure": str(heure_selectionnee),
                    "motif": motif,
                    "statut": "Confirmé",
                    "date_creation": datetime.datetime.now()
                })
                st.success(f"Rendez-vous enregistré pour {nom} !")
            else:
                st.warning("Merci d'entrer votre nom.")

# --- SECTION : BIOGRAPHIE ---
elif page == "Biographie":
    st.title("👨‍⚕️ Le Praticien")
    st.write("### Dr. [Nom du Médecin]")
    st.write("Spécialiste en médecine moderne avec plus de 10 ans d'expérience.")
    st.write("- Diplômé de l'Université d'Alger")
    st.write("- Expert en suivi numérique des patients")

# --- SECTION : LOCALISATION ---
elif page == "Localisation":
    st.title("📍 Nous trouver")
    st.write("Le cabinet est situé au centre-ville de Draria, Alger.")
    # Map centrée sur Draria
    st.map({"lat": [36.7167], "lon": [2.9833]})

# --- SECTION : ESPACE PATIENT ---
elif page == "Espace Patient":
    st.title("🔐 Connexion / Inscription")
    choix = st.tabs(["S'inscrire", "Se connecter"])
    
    with choix[0]:
        new_email = st.text_input("Email", key="reg_mail")
        new_pass = st.text_input("Mot de passe", type="password", key="reg_pass")
        if st.button("Créer mon compte"):
            try:
                user = auth.create_user(email=new_email, password=new_pass)
                st.success("Compte créé ! Vous pouvez maintenant prendre RDV.")
            except Exception as e:
                st.error(f"Erreur : {e}")
