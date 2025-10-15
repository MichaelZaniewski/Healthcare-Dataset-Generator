<img width="3000" height="2015" alt="Python" src="https://github.com/user-attachments/assets/a5ce600f-481f-414d-9fee-830ea760c69d" />

# Healthcare-Dataset-Generator

A Python-based generator that creates unique relational healthcare datasets for analytics and machine learning. 

## Background and Overview
The U.S. healthcare system generates massive volumes of data spanning patients, treatments, and billing but real datasets are often inaccessible due to privacy laws like HIPAA. This project was created to bridge that gap by providing a safe, realistic environment for healthcare analytics and data visualization.

The generator programmatically creates realistic, U.S.-based healthcare data that mirrors hospital operations —  with **no real or sensitive data**. Each run is randomized under strict rules, so **no two datasets are identical**. It models `Patients`, `Visits`, and `Billing` tables with dynamic logic for severity, follow-ups, and payment behavior, ensuring every run produces a unique, validated dataset ready for SQL or Tableau analysis.


## Key Features
- Multi-table relational design (`patients`, `visits`, `billing`) with strict PK/FK consistency.
- Realistic clinical logic: condition gating, LOS rules, same-day probabilities, follow-up inheritance (doctor/hospital).
- Billing realism: charges scale by condition/severity/LOS; payment plans & deterministic statuses.
- Tunable parameters for patient count, dates, distinct zipcode count, and seed generation number. 
- Utilizes real US zipcodes for easy mapping in Tableau for an additionnal layer of analytical opportunity.
- Includes some intentional data formatting errors to simulate messy data, though not much effort was put into this as the dataset is intended to be mostly analysis ready.
- Employes a built-in validator that produces a summary report after generation to ensure logic is adheared to.


## Dataset Logic
## Zipcodes
- Patient zipcodes must match state they blong to.
- The hospital a patient belongs to is in the same state as their own. 
- A Hospital must have only one zipcode.
- Every patient must be assigned a hospital.
- Subsequently, the zipcode count during generation affects quantity of hospitals.

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
   - You should get a response showing the version of Python you are running.
    
2. Download the Generator Folder:
   - For the download, click [here](https://github.com/MichaelZaniewski/Healthcare-Dataset-Generator/releases/tag/v1.0).
   - Place the folder on your desktop.

4. Install Required Libraries
   - Open a terminal or Command Prompt in that folder and run:
       ```
       pip install pandas numpy faker
       ```
5. Run a Small Test Sample
   - To generate a quick 100-patient sample, use this (make sure to input the date!):
       ```
       python healthcare_dataset_generator.py   --patients 100   --today YYYY-MM-DD   --zip-target 10   --zip-pool-file ".\us_zip_pool_10k_with_state.csv"   --outdir ".\Dataset"
       ```
7. Increase the Dataset Size
   - For larger dataset generation: Patient count, todays date, and distinct zipcode fields are all customizable. Change at will.
       ```
       python healthcare_dataset_generator.py   --patients 50000   --today YYYY-MM-DD   --zip-target 5800   --zip-pool-file ".\us_zip_pool_10k_with_state.csv"   --outdir ".\Dataset"
       ```
   - Note: Keep in mind that an adjustment to patient count (Patient table) has an exponential impact on the Visit and Billing tables because its a one-to-many relationship. For instance, a 50K patient table may generate 225K records in both the Visit and Billing tables. Scale up slowly to ensure your computer can handle the computation. Generation may take a while, be patient. The validation_summary must rescan all rows and calculate metrics and takes additional time to populate after the tables have been exported to the output folder.
     
8. View the Results
   - Your output folder will contain
     - patients.csv
     - visits.csv
     - billing.csv
     - validation_summary.json
    
    More info:
   you can add a --seed clause
   seed defaults to 42
keying in the same patient count, zip count, date, and seed as someone else will produce the same dataset.
