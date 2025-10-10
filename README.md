# Healthcare-Dataset-Generator
A Python-based generator that creates unique relational healthcare datasets for analytics and machine learning. 


The generator programmatically creates realistic, U.S.-based healthcare data that mirrors hospital operations —  with **no real or sensitive data**. Each run is randomized under strict rules, so **no two datasets are identical**. It models **Patients**, **Visits**, and **Billing** with dynamic logic for severity, follow-ups, and payment behavior — ensuring every run produces a unique, validated dataset ready for SQL or Tableau analysis.


## Key Features
- Multi-table relational design (**patients**, **visits**, **billing**) with strict PK/FK consistency
- Realistic clinical logic: condition gating, LOS rules, same-day probabilities, follow-up inheritance (doctor/hospital)
- Billing realism: charges scale by condition/severity/LOS; payment plans & deterministic statuses
- Tunable parameters for same-day discharges, randomness, and reproducibility
- Built-in validator produces a summary report


## Dataset Logic

### LOS & Severity
- LOS ranges by condition category (minor/acute/chronic/complex)
- Higher severity generally longer LOS; minor + mild can be same-day (tunable)

### Billing & Payments
- Charges scale by condition/severity/LOS with hospital multipliers
- Payment plan: **Full** (+30d) or **Incremental** (+12mo)
- Status derived deterministically: `Paid`, `Late-Paid`, `In-progress`, `Late-Unpaid`
- Follow-ups trend downward vs first charge (small bumps allowed)

### Heterogeneity & Randomness
- Hospital profiles: same-day bias, late-payment multiplier, charge multiplier
- Doctor styles: same-day and medication bias
- Insurance effects (coverage %, payment risk)
- Uninsured + high charge (> $10k) → higher **Late-Paid** likelihood

### Clinical Logic Highlights
- Pediatric gating (adult-only conditions blocked under 18)
- Childbirth: **Female**, ages **25–50**; ≤2 childbirth visits; postpartum follow-ups use postpartum treatments
- Follow-ups for same condition inherit **doctor/hospital**, LOS ≤ 2
- Distinct conditions per patient: **1–2 common**, max 3
  
### Validation
Automatic checks (saved to `validation_report.txt`):
- IDs uniqueness and 1:1 visit↔billing
- Age correctness (today & per-visit), single blood type per patient
- Insurance rule: no policy without provider
- Pediatric + childbirth gating
- Follow-up consistency (doctor/hospital) and LOS ≤ 2
- Flu/Pneumonia ≤ 3/year; Heart disease bypass ≤ 1
- Billing trend sanity; no future payment dates


## How to Run
1. Install Python
   - Download & install Python (includes pip): https://www.python.org/downloads/
   - On windows installer: check "Add Python to PATH"
   - Once installed, verify by opening Command Prompt or Terminal and typing:
       ```
       python --version
       ```
2. Download the Generator
   - Save the generator file (for example healthcare_dataset_generator.py) to a folder on your computer

4. Install Required Libraries
   - Open a terminal or Command Prompt in that folder and run:
       ```
       pip install pandas numpy faker
       ```
5. Run a Small Test Sample
   - To generate a quick 10-patient sample with validation:
       ```
       python healthcare_dataset_generator.py --patients 10 --validate --outdir ./_out
       ```
6. Increase the Dataset Size
   - For larger dataset generation: 
       ```
       python healthcare_dataset_generator.py --patients 50000 --minor-same-day-rate 0.6 --validate --outdir ./_out
       ```
   - Note: You can customize the amount of patients and the rate of same-day discharges for minor conditions. Keep in mind that an adjustment to patient count (Patient table) has an exponential impact on the Visit and Billing tables because its a one-to-many relationship. For instance, a 50K patient table may generate 225K records in both the Visit and Billing tables.
7. View the Results
   - Your output folder will contain
     - heaelthcare_patients.csv
     - healthcare_visits.csv
     - healthcare_billing.csv
     - validation_report.txt
