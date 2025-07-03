# CV_Scoring
// Python to use text CV Scoring Process

import re
from sklearn.feature_extraction.text import TfidfVectorizer
from sklearn.metrics.pairwise import cosine_similarity

# Job Requirements Configuration
job_requirements = {
    "experience": ["5 years", "manager", "leadership", "project management"],
    "skills": ["Python", "SQL", "AWS", "machine learning"],
    "salary_range": {"min": 8000, "max": 12000},  # USD/month
    "certifications": ["PMP", "AWS Certified", "Google Cloud"]
}

# Single CV Data (generic variable name)
cv = {
    "experience": "4 years as Python Developer with team leadership",
    "skills": "Python, SQL, Docker, Basic AWS",
    "expected_salary": "$11,500 per month",
    "certifications": "AWS Certified Developer"
}

# Scoring Functions
def calculate_experience_score(exp_text):
    years = sum(int(num) for num in re.findall(r'(\d+)\s*year', exp_text.lower()))
    return min(years * 3, 30)  # 3 points/year, max 30

def calculate_skills_score(cv_skills, req_skills):
    vectorizer = TfidfVectorizer().fit_transform([
        cv_skills.replace(",", " ").lower(), 
        " ".join(req_skills).lower()
    ])
    return round(cosine_similarity(vectorizer[0], vectorizer[1])[0][0] * 40, 1)

def evaluate_salary(salary_text, min_sal, max_sal):
    try:
        salary = int(re.search(r'\$?\s*(\d{1,3}(?:,\d{3})*)', salary_text).group(1).replace(',', ''))
        if salary < min_sal: return 20
        elif salary > max_sal * 1.2: return 5
        elif salary > max_sal: return 10
        else: return 15
    except:
        return 10  # Neutral if invalid

def check_certifications(cert_text, req_certs):
    certs = [c.strip().lower() for c in cert_text.split(",")]
    matches = sum(1 for req in req_certs if any(req.lower() in c for c in certs))
    return min(matches * 3.33, 10)

# Calculate Scores
scores = {
    "Experience": calculate_experience_score(cv["experience"]),
    "Skills": calculate_skills_score(cv["skills"], job_requirements["skills"]),
    "Salary Fit": evaluate_salary(
        cv["expected_salary"],
        job_requirements["salary_range"]["min"],
        job_requirements["salary_range"]["max"]
    ),
    "Certifications": check_certifications(cv["certifications"], job_requirements["certifications"])
}
scores["Total"] = round(sum(scores.values()), 1)

# Generate Report
print("\n" + "="*50)
print(" CV EVALUATION RESULTS ".center(50, "="))
print("="*50)

print("\nRAW CV DATA:")
print(f"- Experience: {cv['experience']}")
print(f"- Skills: {cv['skills']}")
print(f"- Expected Salary: {cv['expected_salary']}")
print(f"- Certifications: {cv['certifications']}")

print("\n" + "-"*50)
print("SCORING BREAKDOWN:")
print(f"Experience:       {scores['Experience']}/30")
print(f"Skills Match:     {scores['Skills']}/40")
print(f"Salary Fit:       {scores['Salary Fit']}/20")
print(f"Certifications:   {scores['Certifications']}/10")

print("\n" + "="*50)
print(f" FINAL SCORE: {scores['Total']}/100 ")
