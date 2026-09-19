import base64
import json
import streamlit as st
from PIL import Image
import openai

# Page Setup
st.set_page_config(page_title="AI Jewelry Stylist", layout="wide", page_icon="💎")

st.title("💎 AI Jewelry Pairing & Styling Studio")
st.write("Upload your jewelry inventory, describe an outfit, and get custom AI-styled pairings.")

# Sidebar for API Key
with st.sidebar:
    st.header("Configuration")
    api_key = st.text_input("OpenAI API Key", type="password", help="Enter your OpenAI key starting with sk-")

# Helper: Convert image to base64 for API
def image_to_base64(uploaded_file):
    return base64.b64encode(uploaded_file.getvalue()).decode("utf-8")

# Session state initialization for inventory
if "catalog" not in st.session_state:
    st.session_state.catalog = {}

# --- STEP 1: UPLOAD & CATALOG ---
st.header("1. Upload Jewelry Inventory")
uploaded_files = st.file_uploader(
    "Upload photos of your jewelry pieces", 
    type=["png", "jpg", "jpeg"], 
    accept_multiple_files=True
)

if uploaded_files and api_key:
    if st.button("Process & Catalog Uploaded Items"):
        client = openai.OpenAI(api_key=api_key)
        
        with st.spinner("Analyzing jewelry pieces with AI Vision..."):
            for file in uploaded_files:
                # Avoid re-analyzing items already uploaded
                if file.name in st.session_state.catalog:
                    continue
                
                b64_img = image_to_base64(file)
                prompt = (
                    "Analyze this piece of jewelry. Return JSON ONLY with keys: "
                    "'category' (Necklace, Earrings, Bracelet, Ring, Pins), "
                    "'metal_tone' (Gold, Silver, Rose Gold, Mixed), "
                    "'style' (Minimalist, Statement, Vintage, Boho, Formal), "
                    "'primary_color', and a short 'description'."
                )
                
                try:
                    response = client.chat.completions.create(
                        model="gpt-4o",
                        response_format={"type": "json_object"},
                        messages=[{
                            "role": "user",
                            "content": [
                                {"type": "text", "text": prompt},
                                {"type": "image_url", "image_url": {"url": f"data:image/jpeg;base64,{b64_img}"}}
                            ]
                        }]
                    )
                    
                    item_data = json.loads(response.choices[0].message.content)
                    item_data["file_name"] = file.name
                    
                    # Store image and data in state
                    st.session_state.catalog[file.name] = {
                        "data": item_data,
                        "file": file
                    }
                except Exception as e:
                    st.error(f"Error processing {file.name}: {e}")
            
        st.success("Catalog updated successfully!")

# Display Inventory Grid
if st.session_state.catalog:
    st.subheader(f"Current Inventory ({len(st.session_state.catalog)} items)")
    cols = st.columns(4)
    for idx, (filename, item) in enumerate(st.session_state.catalog.items()):
        with cols[idx % 4]:
            st.image(item["file"], use_container_width=True)
            details = item["data"]
            st.caption(f"**{details['category']}** | {details['metal_tone']}\n\n*{details['style']} style*")

st.divider()

# --- STEP 2: STYLING & PAIRING ---
st.header("2. Get Outfit Pairing Recommendations")

col1, col2 = st.columns([1, 1])

with col1:
    outfit_desc = st.text_area(
        "Describe the Outfit or Occasion:", 
        placeholder="E.g., A emerald green silk slip dress with gold strapless heels for a summer wedding gala."
    )
    style_button = st.button("Generate Jewelry Pairing 👗✨", type="primary")

with col2:
    if style_button:
        if not api_key:
            st.error("Please enter your OpenAI API Key in the sidebar.")
        elif not st.session_state.catalog:
            st.warning("Please upload at least a few jewelry pieces first.")
        elif not outfit_desc:
            st.warning("Please enter an outfit description.")
        else:
            client = openai.OpenAI(api_key=api_key)
            
            # Prepare simplified metadata list for LLM selection
            inventory_payload = [item["data"] for item in st.session_state.catalog.values()]
            
            prompt = f"""
            You are a luxury fashion wardrobe stylist.
            
            OUTFIT: "{outfit_desc}"
            
            AVAILABLE JEWELRY:
            {json.dumps(inventory_payload, indent=2)}
            
            Select 1 to 4 complementary pieces from the available inventory. Ensure metal tones, necklines, and styles harmonize.
            
            Return JSON with:
            - 'selected_file_names': array of selected 'file_name' strings
            - 'styling_notes': explanation of why these pieces complement the specific outfit neckline, color, and vibe.
            """
            
            with st.spinner("Curating the perfect combination..."):
                response = client.chat.completions.create(
                    model="gpt-4o",
                    response_format={"type": "json_object"},
                    messages=[{"role": "user", "content": prompt}]
                )
                
                result = json.loads(response.choices[0].message.content)
                
                st.subheader("Recommended Set")
                st.info(result.get("styling_notes", ""))
                
                selected_files = result.get("selected_file_names", [])
                
                if selected_files:
                    rec_cols = st.columns(len(selected_files))
                    for idx, fname in enumerate(selected_files):
                        if fname in st.session_state.catalog:
                            item = st.session_state.catalog[fname]
                            with rec_cols[idx]:
                                st.image(item["file"], caption=item["data"]["category"], use_container_width=True)
