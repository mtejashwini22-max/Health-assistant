import streamlit as st
from datetime import datetime
import traceback

# Set page configuration only once
st.set_page_config(
    page_title="Health Assistant",
    page_icon="💊",
    layout="centered"
)

# Try importing FPDF
try:
    from fpdf import FPDF
except:
    st.title("Health Assistant - Missing dependency")
    st.error("The package 'fpdf' is not installed.")
    st.write("Install it using: `pip install fpdf`")
    st.stop()


def build_pdf_bytes(age, gender, language, symptoms_selected, suggestions):
    """
    Build PDF in memory and return bytes.
    """
    pdf = FPDF()
    pdf.add_page()
    pdf.set_font("Arial", 'B', 16)
    pdf.cell(0, 10, "Health Assistant Report", ln=True, align="C")

    pdf.set_font("Arial", '', 12)
    pdf.ln(5)

    pdf.cell(0, 10, f"Date: {datetime.now().strftime('%Y-%m-%d %H:%M')}", ln=True)
    pdf.cell(0, 10, f"Age: {age}", ln=True)
    pdf.cell(0, 10, f"Gender: {gender}", ln=True)
    pdf.cell(0, 10, f"Language: {language}", ln=True)

    pdf.ln(5)
    pdf.cell(0, 10, "Symptoms Selected:", ln=True)
    for s in symptoms_selected:
        pdf.cell(0, 10, f"- {s}", ln=True)

    pdf.ln(5)
    pdf.cell(0, 10, "Suggestions:", ln=True)
    for s in suggestions:
        pdf.multi_cell(0, 10, s)

    pdf_bytes = pdf.output(dest="S").encode("latin-1", errors="replace")
    return pdf_bytes


def main():
    st.title("💊 Health Assistant App")
    st.markdown(
        "Welcome! Enter your symptoms and get **basic health guidance**.  \n"
        "⚠️ *This is for educational purposes only.*"
    )

    # Sidebar inputs
    st.sidebar.header("Your Information")
    age = st.sidebar.number_input("Age", min_value=0, max_value=120, value=25)
    gender = st.sidebar.selectbox("Gender", ["Male", "Female", "Other"])
    language = st.sidebar.selectbox("Language / भाषा / భాష", ["English", "Hindi", "Telugu"])

    # Symptom options
    symptom_options = {
        "English": ["Fever", "Cough", "Headache", "Cold", "Fatigue", "Nausea"],
        "Hindi": ["बुखार", "खांसी", "सिरदर्द", "सर्दी", "थकान", "मतली"],
        "Telugu": ["కుండు", "కఫం", "తలనొప్పి", "చలి", "అలసట", "వాంతులు"]
    }[language]

    st.header({
        "English": "Select your symptoms",
        "Hindi": "अपने लक्षण चुनें",
        "Telugu": "మీ లక్షణాలను ఎంచుకోండి"
    }[language])

    symptoms_selected = st.multiselect(
        {
            "English": "Choose symptoms",
            "Hindi": "लक्षण चुनें",
            "Telugu": "లక్షణాలను ఎంచుకోండి"
        }[language],
        options=symptom_options
    )

    # -----------------------------
    # PROCESS BUTTON
    # -----------------------------
    if st.button({
        "English": "Get Suggestions",
        "Hindi": "सुझाव प्राप्त करें",
        "Telugu": "సలహాలు పొందండి"
    }[language]):

        if not symptoms_selected:
            st.warning({
                "English": "Select at least one symptom.",
                "Hindi": "कम से कम एक लक्षण चुनें।",
                "Telugu": "కనీసం ఒక లక్షణాన్ని ఎంచుకోండి."
            }[language])
            return

        suggestions = []

        # Mapping symptom → advice
        for s in symptoms_selected:
            s_clean = s.strip().lower()

            if s_clean in ["fever", "बुखार", "కుండు"]:
                suggestions.append({
                    "English": "- Stay hydrated and rest. Consult a doctor if fever persists.",
                    "Hindi": "- पर्याप्त पानी पिएं और आराम करें। यदि बुखार बना रहे तो डॉक्टर से मिलें।",
                    "Telugu": "- ఎక్కువ నీరు తాగి విశ్రాంతి తీసుకోండి. జ్వరం కొనసాగితే వైద్యుడిని సంప్రదించండి."
                }[language])

            elif s_clean in ["cough", "खांसी", "కఫం"]:
                suggestions.append({
                    "English": "- Drink warm liquids. Consult a doctor if cough persists.",
                    "Hindi": "- गर्म तरल पदार्थ पिएं। खांसी लगातार रहे तो डॉक्टर से संपर्क करें।",
                    "Telugu": "- వేడి ద్రవాలు తాగండి. దగ్గు తగ్గకపోతే డాక్టర్‌ను కలవండి."
                }[language])

            elif s_clean in ["headache", "सिरदर्द", "తలనొప్పి"]:
                suggestions.append({
                    "English": "- Rest in a quiet room and drink water.",
                    "Hindi": "- शांत कमरे में आराम करें और पानी पिएं।",
                    "Telugu": "- నిశ్శబ్ద గదిలో విశ్రాంతి తీసుకోండి మరియు నీరు తాగండి."
                }[language])

            else:
                suggestions.append({
                    "English": "- Maintain healthy food, sleep well, exercise regularly.",
                    "Hindi": "- अच्छा भोजन करें, पर्याप्त नींद लें, नियमित व्यायाम करें।",
                    "Telugu": "- ఆరోగ్యకరమైన ఆహారం తీసుకోండి, బాగా నిద్రపోండి, వ్యాయామం చేయండి."
                }[language])

        # Show suggestions
        st.success({
            "English": "Here are your suggestions:",
            "Hindi": "आपके सुझाव:",
            "Telugu": "మీ సలహాలు:"
        }[language])

        for s in suggestions:
            st.write(s)

        # Generate PDF
        pdf_bytes = build_pdf_bytes(age, gender, language, symptoms_selected, suggestions)

        st.download_button(
            label={
                "English": "Download PDF Report",
                "Hindi": "पीडीएफ रिपोर्ट डाउनलोड करें",
                "Telugu": "PDF రిపోర్ట్ డౌన్లోడ్ చేయండి"
            }[language],
            data=pdf_bytes,
            file_name="health_report.pdf",
            mime="application/pdf"
        )

        st.markdown("---")

    st.markdown(
        {
            "English": "💡 *This app is for educational purposes only.*",
            "Hindi": "💡 *यह ऐप केवल शैक्षिक उद्देश्यों के लिए है।*",
            "Telugu": "💡 *ఈ యాప్ కేవలం విద్యా ప్రయోజనాల కోసం మాత్రమే.*"
        }[language]
    )


if __name__ == "__main__":
    main()
