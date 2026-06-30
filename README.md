# AI-Resume-Scanner:
# app.py:
```
import streamlit as st
import pandas as pd
import numpy as np
import plotly.graph_objects as go
from groq import Groq
from dotenv import load_dotenv
from PyPDF2 import PdfReader
import os
import re

# -----------------------------
# Load Environment Variables
# -----------------------------
load_dotenv()

client = Groq(
    api_key=os.getenv("GROQ_API_KEY")
)

st.set_page_config(
    page_title="AI Resume Scanner",
    page_icon="📄",
    layout="wide"
)

st.title("📄 AI Resume Scanner")
st.write("Upload your resume (PDF) and get an AI-powered ATS analysis.")

uploaded_file = st.file_uploader(
    "Upload Resume",
    type=["pdf"]
)

# -----------------------------
# Read PDF
# -----------------------------
def extract_text(pdf):
    reader = PdfReader(pdf)
    text = ""

    for page in reader.pages:
        page_text = page.extract_text()
        if page_text:
            text += page_text

    return text


# -----------------------------
# AI Analysis
# -----------------------------
def analyze_resume(text):

    prompt = f"""
You are an ATS Resume Analyzer.

Analyze the following resume.

Resume:

{text}

Return your answer in the following format:

ATS Score:
(number between 0 and 100)

Strengths:
- point
- point

Weaknesses:
- point
- point

Missing Keywords:
- keyword
- keyword

Improvement Tips:
- tip
- tip
"""

    response = client.chat.completions.create(
        model="llama-3.3-70b-versatile",
        messages=[
            {
                "role":"user",
                "content":prompt
            }
        ]
    )

    return response.choices[0].message.content


# -----------------------------
# Parse ATS Score
# -----------------------------
def extract_score(result):

    match = re.search(r'ATS Score.*?(\d+)', result)

    if match:
        return int(match.group(1))

    return 0


# -----------------------------
# Main
# -----------------------------
if uploaded_file is not None:

    text = extract_text(uploaded_file)

    st.subheader("Resume Preview")

    st.text_area(
        "",
        text,
        height=250
    )

    if st.button("Analyze Resume"):

        with st.spinner("Analyzing Resume..."):

            result = analyze_resume(text)

        score = extract_score(result)

        st.success("Analysis Complete!")

        st.subheader("ATS Score")

        fig = go.Figure(go.Indicator(
            mode="gauge+number",
            value=score,
            title={'text': "ATS Score"},
            gauge={
                'axis': {'range': [0,100]},
                'bar': {'color': "green"}
            }
        ))

        st.plotly_chart(fig, use_container_width=True)

        st.subheader("AI Report")

        st.markdown(result)

        # Resume Statistics
        words = len(text.split())
        chars = len(text)
        lines = len(text.split("\n"))

        stats = pd.DataFrame({
            "Metric":[
                "Words",
                "Characters",
                "Lines"
            ],
            "Value":[
                words,
                chars,
                lines
            ]
        })

        st.subheader("Resume Statistics")

        st.dataframe(stats)

        st.subheader("Resume Metrics")

        chart = go.Figure(
            data=[
                go.Bar(
                    x=stats["Metric"],
                    y=stats["Value"]
                )
            ]
        )

        st.plotly_chart(chart, use_container_width=True)

        csv = stats.to_csv(index=False).encode("utf-8")

        st.download_button(
            "Download Statistics",
            csv,
            "resume_statistics.csv",
            "text/csv"
        )
```
# requirements.txt:
```
streamlit
pandas
numpy
plotly
groq
python-dotemv
```






