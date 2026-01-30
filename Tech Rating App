import streamlit as st
import pandas as pd
import json
import os

# Page configuration
st.set_page_config(page_title="Tech Rating System", layout="wide")

# File paths
CSV_FILE = "TECH_INFORMATION_rows.csv"
RATINGS_FILE = "tech_ratings.json"

# Initialize session state
if 'ratings_data' not in st.session_state:
    if os.path.exists(RATINGS_FILE):
        with open(RATINGS_FILE, 'r') as f:
            st.session_state.ratings_data = json.load(f)
    else:
        st.session_state.ratings_data = {}

def save_ratings():
    """Save ratings to JSON file"""
    with open(RATINGS_FILE, 'w') as f:
        json.dump(st.session_state.ratings_data, f, indent=2)

def load_tech_data():
    """Load tech information from CSV"""
    try:
        df = pd.read_csv(CSV_FILE)
        return df
    except FileNotFoundError:
        st.error(f"CSV file '{CSV_FILE}' not found. Please ensure it's in the same directory.")
        return None

def get_techs_for_site(df, site_code):
    """Get list of techs for a specific site"""
    site_techs = df[df['SITE'] == site_code][['FIRST_NAME', 'LAST_NAME']].copy()
    site_techs['full_name'] = site_techs['FIRST_NAME'] + ' ' + site_techs['LAST_NAME']
    return site_techs['full_name'].unique().tolist()

def get_tech_ratings(tech_name, site_code, label):
    """Get ratings for a specific tech"""
    key = f"{label}_{site_code}_{tech_name}"
    return st.session_state.ratings_data.get(key, [])

def add_rating(tech_name, site_code, label, responsiveness, quality, timeliness):
    """Add a new rating for a tech"""
    key = f"{label}_{site_code}_{tech_name}"
    if key not in st.session_state.ratings_data:
        st.session_state.ratings_data[key] = []
    
    rating = {
        'responsiveness': responsiveness,
        'quality_of_work': quality,
        'timeliness': timeliness,
        'average': (responsiveness + quality + timeliness) / 3
    }
    
    st.session_state.ratings_data[key].append(rating)
    save_ratings()

def calculate_overall_average(ratings_list):
    """Calculate overall average from all ratings"""
    if not ratings_list:
        return 0
    total = sum(r['average'] for r in ratings_list)
    return total / len(ratings_list)

def get_ranked_techs(df, site_code, label):
    """Get techs ranked by their average scores"""
    techs = get_techs_for_site(df, site_code)
    tech_scores = []
    
    for tech in techs:
        ratings = get_tech_ratings(tech, site_code, label)
        if ratings:
            overall_avg = calculate_overall_average(ratings)
            tech_scores.append({
                'name': tech,
                'average_score': overall_avg,
                'num_ratings': len(ratings)
            })
        else:
            tech_scores.append({
                'name': tech,
                'average_score': 0,
                'num_ratings': 0
            })
    
    # Sort by average score (descending)
    tech_scores.sort(key=lambda x: x['average_score'], reverse=True)
    return tech_scores

# Main app
st.title("🔧 Tech Rating System")
st.markdown("---")

# Load tech data
df = load_tech_data()

if df is not None:
    # Labels configuration (easily expandable)
    LABELS = ["HPI"]  # Add more labels here in the future
    
    # Site codes configuration (easily expandable)
    SITE_CODES = ["ALF", "AQN", "AUP", "AUT", "BOI", "BRM", "COR", "COT", "DCA", "DCP", 
                  "FTC", "HEV", "IND", "MIS", "NNY", "NSH", "NSV", "OSC", "OTA", "PAL", 
                  "SCA", "SDG", "SDN", "SDS", "SVU", "TEX", "TNO", "TUC", "VAA", "VCS", 
                  "VNC", "WEC"]
    
    # Layout with columns
    col1, col2 = st.columns(2)
    
    with col1:
        # Label selection
        selected_label = st.selectbox("📋 Select Label:", LABELS)
    
    with col2:
        # Site code selection
        selected_site = st.selectbox("📍 Select Site Code:", SITE_CODES)
    
    st.markdown("---")
    
    # Get techs for selected site
    techs_list = get_techs_for_site(df, selected_site)
    
    if not techs_list:
        st.warning(f"No Field Nation Techs found for site: {selected_site}")
    else:
        # Display ranked techs
        st.subheader(f"📊 Tech Rankings for {selected_site}")
        
        ranked_techs = get_ranked_techs(df, selected_site, selected_label)
        
        # Create a dataframe for display
        ranking_df = pd.DataFrame(ranked_techs)
        ranking_df.index = range(1, len(ranking_df) + 1)
        ranking_df.columns = ['Tech Name', 'Average Score', 'Number of Ratings']
        ranking_df['Average Score'] = ranking_df['Average Score'].round(2)
        
        st.dataframe(ranking_df, use_container_width=True)
        
        st.markdown("---")
        
        # Rating input section
        st.subheader("➕ Add New Rating")
        
        # Tech selection
        selected_tech = st.selectbox("👤 Select Tech to Rate:", [""] + techs_list)
        
        if selected_tech:
            st.markdown(f"### Rating for: **{selected_tech}**")
            
            # Score inputs
            col1, col2, col3 = st.columns(3)
            
            with col1:
                responsiveness = st.slider("🎯 Responsiveness", min_value=1, max_value=10, value=5, key="resp")
            
            with col2:
                quality = st.slider("⭐ Quality of Work", min_value=1, max_value=10, value=5, key="qual")
            
            with col3:
                timeliness = st.slider("⏰ Timeliness", min_value=1, max_value=10, value=5, key="time")
            
            # Display current scores and average
            average_score = (responsiveness + quality + timeliness) / 3
            
            st.markdown("### Current Scores")
            score_col1, score_col2, score_col3, score_col4 = st.columns(4)
            
            with score_col1:
                st.metric("Responsiveness", responsiveness)
            with score_col2:
                st.metric("Quality of Work", quality)
            with score_col3:
                st.metric("Timeliness", timeliness)
            with score_col4:
                st.metric("Average", f"{average_score:.2f}")
            
            # Submit button
            if st.button("💾 Submit Rating", type="primary"):
                add_rating(selected_tech, selected_site, selected_label, responsiveness, quality, timeliness)
                st.success(f"✅ Rating submitted for {selected_tech}!")
                st.rerun()
            
            # Display rating history for selected tech
            st.markdown("---")
            st.subheader(f"📈 Rating History for {selected_tech}")
            
            tech_ratings = get_tech_ratings(selected_tech, selected_site, selected_label)
            
            if tech_ratings:
                history_df = pd.DataFrame(tech_ratings)
                history_df.index = range(1, len(history_df) + 1)
                history_df.columns = ['Responsiveness', 'Quality of Work', 'Timeliness', 'Average']
                history_df = history_df.round(2)
                
                st.dataframe(history_df, use_container_width=True)
                
                # Overall statistics
                st.markdown("### Overall Statistics")
                stat_col1, stat_col2, stat_col3 = st.columns(3)
                
                with stat_col1:
                    avg_resp = sum(r['responsiveness'] for r in tech_ratings) / len(tech_ratings)
                    st.metric("Avg Responsiveness", f"{avg_resp:.2f}")
                
                with stat_col2:
                    avg_qual = sum(r['quality_of_work'] for r in tech_ratings) / len(tech_ratings)
                    st.metric("Avg Quality", f"{avg_qual:.2f}")
                
                with stat_col3:
                    avg_time = sum(r['timeliness'] for r in tech_ratings) / len(tech_ratings)
                    st.metric("Avg Timeliness", f"{avg_time:.2f}")
            else:
                st.info("No ratings recorded yet for this tech.")

# Footer
st.markdown("---")
st.markdown("*Tech Rating System v1.0*")