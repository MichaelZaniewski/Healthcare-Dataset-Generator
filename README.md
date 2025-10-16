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

## Generation Logic:
### Relational Model
  - Three linked tables — **Patients → Visits ↔ Billing (1:1)** — mirror an EHR + revenue-cycle flow.
  - Stable primary/foreign keys for clean SQL joins and BI (star-schema friendly).

### Demographics & Geography
  - U.S.-style names (optional prefixes), phones, addresses.
  - ZIP/state/city sourced from a configurable **ZIP pool** CSV (flexible headers accepted: `zipcode|zip|postal_code`, `state|state_id`, `city`).
  - **ZIP rules:** patient ZIP must match its state; each hospital has **one** ZIP; every patient is assigned a hospital in the **same state**. The size of the ZIP pool influences how many hospitals are generated.

### Conditions, Severity, Treatments
  - **17+ conditions** with age/clinical gating; severity tiers (Normal/Mild/Moderate/Severe).
  - Treatments are **separate from medications** (life-like drug names, dosage, frequency).
  - Recurrence rules: chronic (e.g., diabetes) tend to multiple visits; acute (e.g., flu) resolve quickly.
  - Continuity of care: follow-ups prefer **same doctor + hospital**; otherwise same hospital, or at least same state.

### Visits, LOS & Same-Day Logic
  - Visit counts vary by condition/severity; **LOS** bounded by condition-specific ranges.
  - **Same-day discharges** are probabilistic and concentrated in mild/minor cases (tunable).
  - Follow-up LOS trends short (≤2 days).

### Billing & Payments (strict 1:1 with Visits)
  - Each visit generates exactly **one** billing record with charges, insurance coverage, and patient responsibility.
  - Charges scale by condition/severity/LOS with hospital multipliers; follow-ups trend **lower** than initial visits (small realistic bumps allowed).
  - Payment plans: **Full** (+30d) or **Incremental** (+12mo) → deterministic expected/actual dates → status: `Paid`, `Late-Paid`, `In-progress`, `Late-Unpaid`.

### Heterogeneity & Randomness
  - Hospital profiles: same-day bias, late-payment multiplier, charge multiplier.
  - Doctor styles: same-day and medication bias.
  - Insurance effects: coverage % and payment risk.
  - Seed-driven randomness (`--seed`) for reproducible yet varied datasets.

### Validation
  - Automatic checks (printed and saved to **`validation_summary.json`**):
    - ID uniqueness; **Visits↔Billing 1:1** mapping.
    - LOS/age logic; pediatric/childbirth gating.
    - Insurance rule: no policy without provider.
    - Follow-up consistency (doctor/hospital; LOS ≤2).
    - Date sanity (no future/invalid dates relative to `--today`).


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
   - Whats in the folder: The .py generator and a .csv of US zipcodes (essential for large dataset creation) 

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
    
### Seed Input
- You can add a --seed clause in the generation code to regenerate a dataset previously created.
- **What it does:** Sets the random number generator seed used across the pipeline (NumPy RNG). This controls stochastic choices like condition assignment, LOS draws, follow-up creation, charge variation, hospital/ZIP selection, etc.
- With the same inputs (CLI args, ZIP pool, code version) and the same --seed, you’ll get the same dataset. The default seed if omitted is 42
- How to use:
    ```
    python healthcare_dataset_generator.py   --patients 50000   --today YYYY-MM-DD   --zip-target 5800   --zip-pool-file ".\us_zip_pool_10k_with_state.csv"   --outdir ".\Dataset"   --seed 8
    ```
