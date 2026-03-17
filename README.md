import pandas as pd
import plotly.express as px
import json
import time

# --- MOCK API CONFIGURATION ---
# In a real local scenario with Groq, you would use:
# from groq import Groq
# client = Groq(api_key="your_key")
# Here, we use the environment's supported Gemini API for a functional preview.

async def call_llm(prompt, system_prompt="You are an expert Sales and Marketing Strategist."):
    """
    Wrapper for LLM calls with exponential backoff for reliability.
    """
    api_key = "" # Provided by environment
    url = f"https://generativelanguage.googleapis.com/v1beta/models/gemini-2.5-flash-preview-09-2025:generateContent?key={api_key}"
    
    payload = {
        "contents": [{"parts": [{"text": f"{system_prompt}\n\nUser Request: {prompt}"}]}]
    }
    
    # Simple retry logic for the demo environment
    for i in [1, 2, 4]:
        try:
            import requests
            response = requests.post(url, json=payload)
            if response.status_code == 200:
                result = response.json()
                return result['candidates'][0]['content']['parts'][0]['text']
        except Exception:
            time.sleep(i)
    return "Error: Could not connect to the AI service. Please try again."

# --- UI SETTINGS ---
st.set_page_config(page_title="GenAI Sales & Marketing Suite", layout="wide")

# Custom CSS for a premium look
st.markdown("""
    <style>
    .main { background-color: #f8f9fa; }
    .stButton>button { width: 100%; border-radius: 5px; height: 3em; background-color: #007bff; color: white; }
    .reportview-container .main .block-container { padding-top: 2rem; }
    .card { padding: 20px; border-radius: 10px; background: white; border: 1px solid #e0e0e0; margin-bottom: 10px; }
    </style>
    """, unsafe_all_colors=True)

# --- SIDEBAR NAVIGATION ---
st.sidebar.title("ðŸš€ GenAI Suite")
page = st.sidebar.radio("Navigate to:", ["Dashboard", "Campaign Generator", "Sales Pitch Creator", "Lead Insights (Predictive)"])

# --- DATA GENERATION (MOCK) ---
def get_mock_leads():
    return pd.DataFrame({
        'Lead Name': ['TechCorp', 'GreenEnergy Inc', 'RetailFlow', 'BioHealth', 'CloudScale'],
        'Industry': ['SaaS', 'Renewables', 'E-commerce', 'Healthcare', 'IT'],
        'Engagement Score': [85, 42, 91, 65, 78],
        'Estimated Value ($)': [50000, 12000, 85000, 30000, 45000],
        'Conversion Probability (%)': [75, 20, 90, 45, 60]
    })

# --- PAGE: DASHBOARD ---
if page == "Dashboard":
    st.title("ðŸ“Š Business Outcomes Dashboard")
    st.write("Real-time predictive analysis for data-driven decision making.")
    
    leads_df = get_mock_leads()
    
    col1, col2, col3 = st.columns(3)
    col1.metric("Pipeline Value", f"${leads_df['Estimated Value ($)'].sum():,}")
    col2.metric("Avg. Conversion Rate", f"{leads_df['Conversion Probability (%)'].mean()}%")
    col3.metric("High-Value Leads", len(leads_df[leads_df['Engagement Score'] > 80]))
    
    st.subheader("Lead Conversion Probability vs. Value")
    fig = px.scatter(leads_df, x="Conversion Probability (%)", y="Estimated Value ($)", 
                     size="Engagement Score", color="Industry", hover_name="Lead Name",
                     template="plotly_white")
    st.plotly_chart(fig, use_container_width=True)

# --- PAGE: CAMPAIGN GENERATOR ---
elif page == "Campaign Generator":
    st.title("ðŸ“£ Multi-Channel Campaign Generator")
    
    with st.expander("Configure Campaign Strategy", expanded=True):
        prod_name = st.text_input("Product/Service Name", placeholder="e.g. AI-Powered CRM")
        target_aud = st.text_input("Target Audience", placeholder="e.g. Small business owners in NY")
        goal = st.selectbox("Campaign Goal", ["Brand Awareness", "Lead Generation", "Product Launch", "Event Attendance"])
        
    if st.button("Generate Campaign"):
        if prod_name and target_aud:
            with st.spinner("AI is architecting your campaign..."):
                prompt = f"Create a marketing campaign for {prod_name} targeting {target_aud} with the goal of {goal}. Include a tagline, 3 key value propositions, and a draft for a social media post."
                result = st.session_state.get('campaign_res')
                # For immediate UI feedback in the preview:
                import asyncio
                res = asyncio.run(call_llm(prompt, "You are a world-class Marketing Director."))
                st.markdown("### Generated Strategy")
                st.info(res)
        else:
            st.warning("Please fill in the product name and target audience.")

# --- PAGE: SALES PITCH CREATOR ---
elif page == "Sales Pitch Creator":
    st.title("ðŸ¤ Personalized Sales Pitch")
    
    col1, col2 = st.columns(2)
    with col1:
        prospect_info = st.text_area("Prospect Context", help="Paste LinkedIn bio or company news here", height=150)
    with col2:
        tone = st.select_slider("Pitch Tone", options=["Professional", "Friendly", "Bold", "Consultative"])
        pain_point = st.text_input("Primary Pain Point", placeholder="e.g. High customer churn")

    if st.button("Draft Pitch"):
        with st.spinner("Crafting your personalized message..."):
            prompt = f"Draft a {tone} sales pitch for a prospect with the following context: {prospect_info}. Focus on solving the pain point: {pain_point}."
            import asyncio
            res = asyncio.run(call_llm(prompt, "You are a high-performance Sales Executive."))
            st.success("Draft Ready!")
            st.text_area("Generated Output", res, height=300)

# --- PAGE: LEAD INSIGHTS ---
elif page == "Lead Insights (Predictive)":
    st.title("ðŸ” Predictive Lead Insights")
    st.write("Upload your lead data or use our AI to analyze current prospects.")
    
    leads = get_mock_leads()
    st.dataframe(leads, use_container_width=True)
    
    selected_lead = st.selectbox("Select a Lead for Deep Analysis", leads['Lead Name'])
    
    if st.button("Analyze Lead"):
        lead_data = leads[leads['Lead Name'] == selected_lead].iloc[0]
        with st.spinner(f"Analyzing {selected_lead}..."):
            prompt = f"Analyze this lead: {lead_data.to_json()}. Predict why they might hesitate and suggest 3 data-driven actions to close the deal."
            import asyncio
            res = asyncio.run(call_llm(prompt, "You are a Predictive Sales Analyst."))
            
            st.subheader(f"Strategy for {selected_lead}")
            st.write(res)
            
            # Simulated confidence chart
            st.progress(int(lead_data['Conversion Probability (%)']))
            st.caption(f"AI Confidence in conversion: {lead_data['Conversion Probability (%)']}%")

st.sidebar.markdown("---")
st.sidebar.caption("Built with Streamlit & GenAI")
