<img width="3000" height="1375" alt="Image" src="https://github.com/user-attachments/assets/6a9467e7-4936-4db7-9a8a-340d915ab10c" />

# Healthcare-Dataset-Generator

A Python-based generator that creates unique relational healthcare datasets for analytics and machine learning. 

## Background and Overview
The U.S. healthcare system generates massive volumes of data spanning patients, treatments, and billing but real datasets are often inaccessible due to privacy laws like HIPAA (Health Insurance Portability and Accountability Act). This project was created to bridge that gap by providing a safe, realistic environment for healthcare analytics and data visualization.

This healthcare dataset generator was designed in tandem with OpenAI's GPT 5 - Thinking Model; implementing realistic clinical, operational, and billing rules, then translating them into a single Python generator that produces linked tables with a validation summary report to ensure data accuracy and efficacy.

The generator programmatically creates realistic, U.S.-based healthcare data that mirrors hospital operations —  with **no real or sensitive data**. Each run is randomized under strict rules, so **no two datasets are identical**. It models `Patients`, `Visits`, and `Billing` tables with dynamic logic for severity, follow-ups, and payment behavior, ensuring every run produces a unique, validated dataset ready for SQL or Tableau analysis.

The generator is a reproducible, seed-driven engine that builds a dataset end-to-end with configurable geography and time windows. It assigns facilities and patients using a ZIP/state pool, enforces pediatric/adult gating, and bounds length-of-stay by condition-specific ranges rather than one-size averages. Visit timelines are constructed first, then downstream billing is derived deterministically from those timelines (e.g., plan = Full vs Incremental → expected dates → status transitions), so payment outcomes are explainable from inputs. Severity tiers influence tests vs. procedures, and recurrence rules drive whether a case becomes a single episode or a series of returns. The generator emits clean, SQL, excel, and tableau friendly CSVs plus a validation JSON that checks ID integrity, visit-billing 1:1 mapping, age logic, LOS fences, and payment status consistency. Runs are tunable via CLI flags (patients, anchor date, ZIP diversity, seed), letting you create bite-size samples for demos or larger datasets for analytics and dashboarding.

## Release Notes
### v2 (Locality + Single-Hospital Mode)
- Added optional single-hospital mode so you can anchor the dataset to one hospital (useful for breach simulations and org-specific analysis).
- Added locality controls to skew patient ZIP assignment around a hospital ZIP while still allowing some out-of-state patients (realistic over multi-year timelines).
- Added new CLI flags:
  - `--single-hospital`
  - `--hospital-name`
  - `--hospital-state`
  - `--hospital-zipcode`
  - `--local-share`
  - `--local-top-n`
  - `--local-decay`
  - `--outstate-neighbor-share`
- Backwards compatible: if you do not use the new locality flags, nationwide multi-hospital behavior remains the default.

## Key Features
- Multi-table relational design with strict PK/FK consistency.
- Realistic clinical logic: condition gating, LOS rules, same-day probabilities, follow-up inheritance (doctor/hospital).
- Billing realism: charges scale by condition/severity/LOS; payment plans & deterministic statuses.
- Tunable parameters for patient count, dates, distinct zipcode count, and seed generation number. 
- Optional single-hospital locality mode to create a realistic service area around one hospital ZIP (with an out-of-state tail).
- Utilizes real U.S. zipcodes for easy mapping in Tableau for an additional layer of analytical opportunity.
- Includes some intentional data formatting errors to simulate messy data, though not much effort was put into this as the dataset is intended to be mostly analysis ready.
- Employes a built-in validator that produces a summary report after generation to ensure logic is adhered to.

## Who Is This For?

- **Data analysts & portfolio builders:** add a healthcare analysis to your portfolio; practice Structured Query Language (SQL), joins, Key Performance Indicators (KPIs), and dashboards on realistic data.
- **Data scientists & Machine Learning (ML) practitioners:** build features from Patients/Visits/Billing, create labels (e.g., payment status), and train classification/forecasting models.
- **Educators, bootcamps, and students:** create graded assignments with a shared seed for reproducible results.
- **Business Intelligence (BI) developers / dashboard designers:** prototype Tableau/Power BI dashboards with realistic KPIs and dimensions.
- **Healthcare operations & revenue cycle leaders:** sandbox Length of Stay (LOS), discharge, and cash-flow scenarios without touching private healthcare information.

## Generation Logic:
### Relational Model
  - Three linked tables — **Patients → Visits ↔ Billing (1:1)** — mirror an EHR + revenue-cycle flow.
  - Stable primary/foreign keys for clean SQL joins and BI (star-schema friendly).

### Demographics & Geography
  - U.S.-style names (optional prefixes), phones, addresses.
  - ZIP/state/city sourced from a configurable **ZIP pool** CSV (flexible headers accepted: `zipcode|zip|postal_code`, `state|state_id`, `city`).
  - **ZIP rules:** patient ZIP must match its state; each hospital has **one** ZIP. In nationwide mode, every patient is assigned a hospital in the **same state**. In single-hospital locality mode, the hospital is anchored to one state (and optional hospital ZIP) and a configurable portion of patients can be out-of-state. The size of the ZIP pool influences how many hospitals are generated in nationwide mode.

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
   - On Windows installer: check "Add Python to PATH"
   - Once installed, verify by opening Command Prompt or Terminal and typing:
       ```
       python --version
       ```
   - You should get a response showing the version of Python you are running.
    
2. Download the Generator Folder:
   - For the download, click [here](https://github.com/MichaelZaniewski/Healthcare-Dataset-Generator/releases/tag/v1.0).
   - Place the folder on your desktop (easiest)
   - What's in the folder: The .py generator and a .csv of U.S. zipcodes (essential for large dataset creation)  

3. Open CMD and navigate to that folder
   - Your drive letter is most likely C
       ```
       cd /d "[DRIVE-LETTER-HERE]:\Users\[USERNAME-HERE]\Desktop\Generator"
       ```

5. Install Required Libraries
   - Once command prompt is open in that folder run:
       ```
       pip install pandas numpy faker
       ```
6. Run a Small Test Sample
   - To generate a quick 100-patient sample, use this (make sure to input the date!):
       ```
       python healthcare_dataset_generator.py   --patients 100   --today YYYY-MM-DD   --zip-target 10   --zip-pool-file ".\us_zip_pool_10k_with_state.csv"   --outdir ".\Dataset"
       ```
7. Run a Small Test Sample
   - For larger dataset generation: Patient count, today's date, and distinct zipcode fields are all customizable. Change at will.
       ```
       python healthcare_dataset_generator.py   --patients 50000   --today YYYY-MM-DD   --zip-target 5800   --zip-pool-file ".\us_zip_pool_10k_with_state.csv"   --outdir ".\Dataset"
       ```
   - Note: Keep in mind that an adjustment to patient count (Patient table) has an exponential impact on the Visit and Billing tables because its a one-to-many relationship. For instance, a 50K patient table may generate 225K records in both the Visit and Billing tables. Scale up slowly to ensure your computer can handle the computation. Generation may take a while, be patient. The validation_summary must rescan all rows and calculate metrics and takes additional time to populate after the tables have been exported to the output folder.

   Single Hospital + Locality Mode (New)
   - Example: NYC-based hospital (Midtown Manhattan 10036) with 83% of patients in NY and 17% out-of-state:
       ```
       python healthcare_dataset_generator.py   --patients 20000   --today YYYY-MM-DD   --zip-pool-file ".\us_zip_pool_10k_with_state.csv"   --outdir ".\Dataset"   --single-hospital   --hospital-state NY   --hospital-zipcode 10036   --local-share 0.83   --local-top-n 400   --local-decay 200   --outstate-neighbor-share 0.70
       ```
   - If you omit `--hospital-zipcode` but keep `--hospital-state NY`, the generator will automatically pick a NY hospital ZIP from the ZIP pool.

8. View the Results
   - Your output folder will contain
     - patients.csv
     - visits.csv
     - billing.csv
     - validation_summary.json

## Customizable Metrics
### Patient Count
- -- patients (input number here)
- Controls the quantity of distinct patients in the dataset.
- If no patient quantity is specified, it defaults to 1000.

### Todays Date
- -- today YYYY-MM-DD
- Controls payment dates and other important metrics to ensure no future dates exist in the dataset.

### Zip Target
- -- zip-target (input number here)
- Controls the quantity of distinct zipcodes.
- The zipcode csv file is needed to run the generator properly. It can work without it but distinct zipcodes will be capped to predetermined values in the script, as opposed to a random selection of 10,000 possible codes. Keep the zipcode csv in the same folder as the generator to ensure proper generation.
- Note: In single-hospital locality mode, the size of the local footprint is primarily controlled by `--local-top-n` and `--local-decay` (not `--zip-target`).

### Seed
- You can add a --seed clause in the generation code to regenerate a dataset previously created.
- **What it does:** Sets the random number generator seed used across the pipeline (NumPy RNG). This controls stochastic choices like condition assignment, LOS draws, follow-up creation, charge variation, hospital/ZIP selection, etc.
- With the same inputs: patient count, date, zip-target, python code version, and --seed, you’ll get the same dataset. The default seed, if omitted, is 42
- How to use:
  
       ```
       python healthcare_dataset_generator.py   --patients 50000   --today YYYY-MM-DD   --zip-target 5800   --zip-pool-file ".\us_zip_pool_10k_with_state.csv"   --outdir ".\Dataset"   --seed 8
       ```

### Locality Mode (New)
- --single-hospital
  - Forces the entire dataset to use one hospital.
- --hospital-name "Hospital Name Here"
  - Optional override for the hospital name when using `--single-hospital`.
- --hospital-state NY
  - Anchors the hospital to a state (turns on locality mode).
- --hospital-zipcode 10036
  - Optional hospital ZIP anchor. If omitted, the generator auto-selects a ZIP in the hospital state.
- --local-share 0.83
  - Percent of patients assigned in the hospital state (example 0.83 means 83% in-state).
- --local-top-n 400
  - How many in-state ZIPs are considered the “local catchment” pool (smaller = tighter footprint).
- --local-decay 200
  - Controls clustering tightness around the hospital ZIP (smaller = tighter clustering).
- --outstate-neighbor-share 0.70
  - When selecting out-of-state patients, biases them toward nearby states rather than random nationwide.

## Issues
If there are any issues please let me know in the issues tab.
