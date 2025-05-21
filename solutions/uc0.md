# Use Case 0: Dynt vs. Bank Statement Reconciliation

## Overview

This use case covers the reconciliation of bank transactions imported into Dynt system (via a third-party API) with transactions uploaded directly from a bank statement CSV file. The goal is to identify discrepancies, duplicates, missing transactions, and provide actionable insights for business owners.

---

## User Flow

### 1. Initial Setup Phase

- **Onboarding:**
  - User provides basic business information (company name, timezone, default currency).

#### API Integration Setup

- User enters API credentials (endpoint, keys, etc.).
- System tests the connection and fetches a sample of transaction data.
- System displays the sample data for user review.

  **Sample API Response:**

  ```json
  {
    "transactions": [
      {
        "transaction_id": "tx_123",
        "amount": 1000.0,
        "date": "2024-05-01",
        "type": "debit",
        "description": "Payment to Supplier",
        "reference": "INV001"
      }
    ]
  }
  ```

#### API Field Discovery & Mapping

- System displays API response structure and available fields.
- User reviews and maps API fields to required reconciliation fields.
- System provides a drag-and-drop or selection interface for mapping.
- Data preview is shown for mapped fields.

  | API Field      | Reconciliation Field |
  | -------------- | -------------------- |
  | transaction_id | Reference Number     |
  | amount         | Amount               |
  | date           | Transaction Date     |
  | type           | Transaction Type     |
  | description    | Description          |
  | reference      | Reference Number     |

- System validates that all mandatory fields are mapped and previews the data.
- User can test the mapping with sample data and address any issues.

#### Bank Statement Configuration

- User uploads a sample bank statement CSV file.
- System auto-detects headers and previews the data.
- User maps CSV headers to required reconciliation fields.
- System supports custom mapping and transformation (e.g., date/amount formats).

#### Field Mapping Setup

- User defines mapping for:
  - **Mandatory fields:** amount, date, transaction type, reference number, account identifier
  - **Supporting fields:** description, status, currency, counterparty, etc.
- User can define data transformations (date/amount formats) and validation rules.

---

## Data Transformation and Validation Rules

A key feature of the system is that users do not need to write code to define data transformation and validation rules. Instead, the system provides an intuitive, user-friendly interface for configuring these rules.

### Data Transformation

- Users can select the source field (from API or CSV) and the target reconciliation field.
- The system offers common transformation options via dropdowns, toggles, or checkboxes, such as:
  - Date format conversion (e.g., "MM/DD/YYYY" → "YYYY-MM-DD")
  - Amount normalization (e.g., "1,000.00" → "1000.00")
  - Currency symbol removal (e.g., "INR 1,000.00" → "1000.00")
  - Text case conversion (e.g., "Supplier Name" → "supplier name")
  - Trimming spaces or removing special characters
- For advanced needs, the system can provide a set of pre-built transformation functions (e.g., "Extract numbers", "Replace text", "Remove special characters") that users can apply without coding.
- The system shows a real-time preview of how the transformation will affect sample data, helping users verify correctness before applying.

### Validation Rules

- Users can set up validation rules through a graphical interface, such as:
  - Marking fields as required (cannot be empty)
  - Setting acceptable value ranges (e.g., amount must be > 0)
  - Specifying allowed formats (e.g., date must match "YYYY-MM-DD")
  - Defining uniqueness (e.g., reference number must be unique)
  - Cross-field validation (e.g., if type is "credit", amount must be positive)
- The system provides checkboxes, dropdowns, or simple form fields for these rules.
- Real-time feedback or previews show how the rules will be applied to sample data.
- The system highlights any validation errors and provides guidance for correction.

### Advanced/Custom Transformations (Optional)

- For users with specific needs, the system may offer an "advanced" mode with a formula builder or expression editor (similar to Excel formulas), but this is optional and not required for standard use cases.

### Summary

- The goal is to make data transformation and validation accessible to non-technical users, using a visual interface with clear options, previews, and error messages.
- Users should never have to write code unless they explicitly want to use an advanced/custom feature.
- This approach ensures the system is flexible, powerful, and user-friendly for a wide range of business users.

---

## Regular Reconciliation Process

### 1. Data Import

- System fetches data from the Dynt API (using date range matching the uploaded CSV, e.g., May 1–31).
- User uploads the latest bank statement CSV file.
- System processes both data sources using the configured mappings.

### 2. Reconciliation Run

- System matches transactions based on mandatory fields.
- Supporting fields are used for validation and to increase match confidence.
- System identifies:
  - Duplicates
  - Missing transactions
  - Amount/date mismatches
  - Other discrepancies

### 3. Results Review

- Dashboard displays:
  - Total transactions processed
  - Number of matched transactions
  - Number of discrepancies
  - Summary of differences
- User can drill down into specific transactions and see detailed matching info.

### 4. Discrepancy Resolution

- User reviews unmatched or problematic transactions.
- System explains why transactions didn't match (e.g., duplicate, missing, mismatch).
- User can manually match, ignore, or flag transactions for further review.

### 5. Report Generation

- User can generate and export reconciliation reports (PDF, Excel, etc.).
- Reports include:
  - Transaction count comparison
  - Duplicate analysis
  - Missing transaction summary
  - Financial impact (e.g., net cash flow, inflated expenses)
- Reports can be scheduled or saved as templates.

---

## Key Insights & Analytics (Example)

- **Transaction Count Comparison Table:**

  | Data Source        | Total Transactions | Duplicates Found  | Adjusted Count |
  | ------------------ | ------------------ | ----------------- | -------------- |
  | Previous CSV Data  | 76                 | 12 duplicate sets | ~64            |
  | Official Bank Data | 67                 | 3 duplicate sets  | 64             |

- **Key Findings:**
  - Official bank data shows fewer duplicates than CSV data.
  - Confirmed duplicates are listed with date, amount, and description.
  - Missing transactions are highlighted (e.g., bank shows 67, CSV shows 76).
  - Financial impact of duplicates is calculated (e.g., CSV inflated expenses).
  - Income and expense transactions are summarized and compared.
  - Net cash flow is calculated and adjusted for duplicates.

---

## Is LLM Required?

No, LLMs (Large Language Models) are not required for the core reconciliation, data transformation, or validation processes in this use case. All insights and analytics here are based on structured, rule-based data analysis—such as counting transactions, detecting duplicates, comparing totals, and calculating net flows. These are deterministic operations that can be handled efficiently and reliably with standard programming techniques and data processing libraries.

### Where Could LLMs Be Used?

LLMs could potentially add value in the following scenarios:

- **Natural Language Transformations:** If users want to describe data transformations or validation rules in plain language (e.g., "extract the number after 'INV' in the reference field"), an LLM could interpret these instructions and generate the necessary logic automatically.
- **Handling Unpredictable or Unstructured Data:** If the data formats are highly variable, unstructured, or novel, LLMs could help infer the intended transformation or validation from context or examples.
- **Complex, Context-Dependent Business Rules:** For validation rules that are difficult to enumerate or anticipate, LLMs could help by understanding and implementing rules described in natural language.

### Should LLMs Be Used Here?

For most reconciliation use cases, standard configurable rules, formula builders, or reusable transformation functions are sufficient. This approach is transparent, easy to maintain, cost-effective, and meets the needs of most business users. LLMs add complexity and cost, and are only justified for rare cases where natural language flexibility or highly unpredictable data formats are required. For the majority of users and scenarios, LLMs are not necessary and traditional coding is preferable.

**Recommendation:**
Use standard, configurable rules and a no-code UI for data transformation and validation. Consider LLMs only as an optional enhancement for advanced users who require natural language flexibility or face truly unpredictable data formats. For most users and use cases, LLMs are not necessary and coding up the logic is preferable.

**Why isn't confidence scoring emphasized here?**

This usecase involves comparing two data sets (API and bank CSV) that are supposed to represent the same underlying transactions. The expectation is for exact matches, so any discrepancy is treated as an exception to be investigated, not as a partial match to be scored. Confidence scoring is more relevant when matching related but not identical records, such as bank payments to invoices, where legitimate differences can occur.

**Could confidence scoring be added?**

It could be used for minor format or timing differences, but in practice, most systems treat any mismatch as a discrepancy to be resolved, rather than something to be scored and possibly auto-matched.
