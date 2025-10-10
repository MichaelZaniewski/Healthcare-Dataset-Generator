# ERD, Tables Explained, Create Code:

### ERD
![ERD](https://github.com/user-attachments/assets/f0ebb341-3835-4c0b-9470-e874e4964a91)




### Patients
| column name     | data type | column name               | data type  |
| --------------- | --------- | ------------------------- | ---------- |
| id              | SERIAL    | email                     | TEXT       |
| first_name      | TEXT      | address                   | TEXT       |
| last_name       | TEXT      | city                      | TEXT       |
| full_name       | TEXT      | state                     | TEXT       |
| date_of_birth   | DATE      | zipcode                   | VARCHAR(5) |
| age             | INT       | insurance_provider        | TEXT       |
| gender          | TEXT      | insurance_policy_number   | TEXT       |
| phone_number    | TEXT      | blood_type                | TEXT       |


### Visits
| column name     | data type | column name          | data type |
| --------------- | --------- | -------------------- | --------- |
| visit_id        | SERIAL    | medication           | TEXT      |
| patient_id      | INT       | dosage               | TEXT      |
| full_name       | TEXT      | frequency            | TEXT      |
| date_of_birth   | DATE      | test_results         | TEXT      |
| age             | INT       | doctor               | TEXT      |
| gender          | TEXT      | hospital             | TEXT      |
| blood_type      | TEXT      | room_number          | INT       |
| condition       | TEXT      | date_of_admission    | DATE      |
| severity        | TEXT      | date_of_discharge    | DATE      |
| treatment       | TEXT      | follow_up_required   | CHAR(1)   |


### Billing
| column name   | data type     | column name                     | data type     |
| ------------- | ------------- | ------------------------------- | ------------- |
| billing_id    | SERIAL        | insurance_coverage_amount       | NUMERIC(12,2) |
| visit_id      | INT           | patient_responsibility_amount   | NUMERIC(12,2) |
| patient_id    | INT           | payment_plan                    | TEXT          |
| full_name     | TEXT          | expected_payment_date           | DATE          |
| billing_date  | DATE          | actual_payment_date             | DATE          |
| total_charge  | NUMERIC(12,2) | payment_status                  | TEXT          |


### Create Tables Code

```
CREATE TABLE patient (
    patient_id              SERIAL PRIMARY KEY,
    first_name              TEXT    NOT NULL,   
    last_name               TEXT    NOT NULL,
    full_name               TEXT    NOT NULL,
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
    full_name            TEXT NOT NULL,
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
    total_charge                INT  NOT NULL CHECK (total_charge > 0),
    insurance_coverage_amount   INT  NOT NULL,
    patient_responsibility_amount INT NOT NULL,
    payment_plan                TEXT NOT NULL,
    expected_payment_date       DATE NOT NULL,
    actual_payment_date         DATE,
    payment_status              TEXT NOT NULL
);
```


