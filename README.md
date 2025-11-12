# Salesforce Account Management & Distribution

This repository contains a Salesforce project for automating account/contact imports and managing an intelligent, territory-based account distribution system.

## Core Features

### 1. Account & Contact Automation

- **Idempotent Account Import:** A script to create new `Account` records from a [CMS dataset](https://data.cms.gov/provider-data/dataset/4pq5-n9py) for facilities in Arizona, Nevada, Utah, & Colorado. The script checks for existing facilities to prevent duplicates.
- **Automated Contact Creation:** A script to extract "Chief Administrator" contacts from a [related dataset](https://www.google.com/search?q=https://data.cms.gov/provider-data/dataset/6j3s-2bgu) for accounts in Arizona.
    - Associates the new `Contact` with the correct `Account`.
    - Checks for existing contacts to prevent duplicates.
    - If a different "Administrator" contact already exists, the script flags the *existing* contact as "inactive."
    - Includes logic to infer contact email addresses.
- **Easy Execution:** Import scripts are designed to be easily triggered from the Salesforce UI (e.g., via a Flow).

### 2. Automated Account Distribution

- **Territory-Based Assignment:** A system to assign a sales rep to a specific state (AZ, NV, UT, or CO) as their "territory."
- **Prioritized Queue:** Automatically assigns the rep the top 20 accounts in their territory, sorted by "Reported Total Nurse Staffing Hours per Resident per Day" (ascending) to prioritize accounts needing the most attention.
- **Automatic Re-assignment:** When an `Opportunity` is "Closed Won" or "Closed Lost," or an `Account` is "Disqualified," the rep's ownership is automatically removed.
- **Backfill Logic:** When an account is removed from the rep's queue, the system automatically assigns them the next highest-priority account from their territory.

## Implementation Details

This solution is implemented using a combination of Salesforce's programmatic and declarative tools.

- **Data Import & Custom Fields:**
    - The `Account` and `Contact` objects are extended with custom fields to store all necessary data (e.g., `CMS_Certification_Number__c`, `Total_Nurse_Staffing_Hours_per_Resident__c`).
    - `CMS_Certification_Number__c` is used as an External ID to handle idempotent `upsert` operations.
    - Apex Batch classes (`AccountDataImporter`, `ContactDataImporter`) handle the data processing.
    - Apex classes are exposed via `@InvocableMethod` to be triggered by a **Salesforce Flow**.
- **Account Distribution Automation:**
    - A custom `Territory__c` field on the `User` object defines the rep's territory.
    - A custom "Disqualify" quick action on `Account` provides a simple UI for the rep.
    - Record-triggered Flows on `Opportunity` (when `IsClosed` = true) and `Account` (on disqualification) initiate the re-assignment logic.
    - The Flow calls an invocable Apex method (`AccountReassignmentHandler`) to identify the next-best account and assign it to the rep.
