# ERD, Tables Explained, Create Code:

### ERD
<img width="1309" height="1468" alt="ERD" src="https://github.com/user-attachments/assets/f2f946a0-a5a1-412e-b76e-d72b65f2fcab" />


### Patients
| column name   | data type | column name             | data type  |
| ------------- | --------- | ----------------------- | ---------- |
| patient_id    | SERIAL    | email                   | TEXT       |
| first_name    | TEXT      | address                 | TEXT       |
| last_name     | TEXT      | city                    | TEXT       |
| name          | TEXT      | state                   | TEXT (2)   |
| date_of_birth | DATE      | zipcode                 | VARCHAR(5) |
| age           | INT       | insurance_provider      | TEXT       |
| gender        | TEXT      | insurance_policy_number | TEXT       |
| phone_number  | TEXT      | blood_type              | TEXT       |

### Visits
| column name   | data type | column name        | data type  |
| ------------- | --------- | ------------------ | ---------- |
| visit_id      | SERIAL    | frequency          | TEXT       |
| patient_id    | INT       | test_results       | TEXT       |
| name          | TEXT      | doctor             | TEXT       |
| date_of_birth | DATE      | hospital           | TEXT       |
| age           | INT       | hospital_state     | TEXT (2)   |
| gender        | TEXT      | hospital_zipcode   | VARCHAR(5) |
| condition     | TEXT      | room_number        | INT        |
| severity      | TEXT      | date_of_admission  | DATE       |
| treatment     | TEXT      | date_of_discharge  | DATE       |
| medication    | TEXT      | follow_up_required | CHAR(1)    |
| dosage        | TEXT      |                    |            |


### Billing
| column name               | data type       | column name                     | data type     |
| ------------------------- | --------------- | ------------------------------- | ------------- |
| billing_id                | SERIAL          | patient_responsibility_amount   | NUMERIC(12,2) |
| visit_id                  | INT             | payment_plan                    | TEXT          |
| patient_id                | INT             | expected_payment_date           | DATE          |
| name                      | TEXT            | actual_payment_date             | DATE          |
| billing_date              | DATE            | payment_status                  | TEXT          |
| insurance_coverage_amount | NUMERIC(12,2)   | total_charge                    | NUMERIC(12,2) |


### Create Tables Code

```
CREATE TABLE patient (
    patient_id              SERIAL PRIMARY KEY,
    first_name              TEXT    NOT NULL,   
    last_name               TEXT    NOT NULL,
    name                    TEXT    NOT NULL,
    date_of_birth           DATE    NOT NULL,
    age                     INT     NOT NULL,
    gender                  TEXT    NOT NULL,
    phone_number            TEXT    NOT NULL,
    email                   TEXT    NOT NULL,
    address                 TEXT    NOT NULL,
    city                    TEXT    NOT NULL,
    state                   CHAR(2) NOT NULL,
    zipcode                 CHAR(5) NOT NULL,
    insurance_provider      TEXT,
    insurance_policy_number TEXT,
    blood_type              TEXT NOT NULL,
    CONSTRAINT chk_insurance_consistency
        CHECK (
            insurance_provider IS NOT NULL
            OR insurance_policy_number IS NULL
        )
);
```

```
CREATE TABLE visit (
    visit_id             SERIAL PRIMARY KEY,
    patient_id           INT NOT NULL REFERENCES patient(patient_id) ON DELETE CASCADE,
    name                 TEXT NOT NULL,
    age                  INT NOT NULL,
    gender               TEXT NOT NULL,
    condition            TEXT NOT NULL,
    severity             TEXT NOT NULL,
    treatment            TEXT NOT NULL,
    medication           TEXT,
    dosage               TEXT,
    frequency            TEXT,
    test_results         TEXT NOT NULL CHECK (test_results IN ('Positive','Negative','Inconclusive')),
    doctor               TEXT NOT NULL,
    hospital             TEXT NOT NULL,
    hospital_state       TEXT NOT NULL,
    hospital_zipcode     TEXT NOT NULL,
    room_number          INT NOT NULL,
    date_of_admission    DATE NOT NULL,
    date_of_discharge    DATE NOT NULL,
    follow_up_required   TEXT NOT NULL
);
```

```
CREATE TABLE billing (
    billing_id                  SERIAL PRIMARY KEY,
    visit_id                    INT NOT NULL UNIQUE REFERENCES visit(visit_id) ON DELETE CASCADE,
    patient_id                  INT NOT NULL REFERENCES patient(patient_id) ON DELETE CASCADE,
    billing_date                DATE NOT NULL,
    insurance_coverage_amount   INT  NOT NULL,
    patient_responsibility_amount INT NOT NULL,
    payment_plan                TEXT NOT NULL,
    expected_payment_date       DATE NOT NULL,
    actual_payment_date         DATE,
    payment_status              TEXT NOT NULL,
    total_charge                INT  NOT NULL CHECK (total_charge > 0)
);
```


