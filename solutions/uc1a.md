# Use Case 1A: Match Bank Transactions to Supplier Invoices (Outgoing)

## User Flow

1. The user logs into the reconciliation system and selects the option to match bank transactions to outgoing supplier invoices.

2. The user provides access to their bank transaction data. The system supports multiple methods:

   - Connecting to a bank via API and fetching transactions for a specified date range.
   - Connecting to an email account (such as Gmail) to fetch bank statements or transaction notifications.
   - Uploading a bank statement CSV file manually.

3. The user provides access to their supplier invoices. This can be done by:

   - Connecting to an accounting or ERP system via API.
   - Connecting to an email account to fetch invoice attachments.
   - Uploading invoice files directly (PDFs, images, etc.).

4. If invoices are unstructured (PDFs, images, or email attachments), the system uses an LLM or advanced OCR/AI to extract structured data from each invoice, such as supplier name, invoice number, amount, due date, reference, status, and currency.

5. The system presents the extracted invoice data to the user for review. The user can correct or confirm any fields that the system is unsure about.

6. The system displays the list of outgoing bank transactions and the structured invoice data.

7. The system attempts to match each bank transaction to one or more supplier invoices using:

   - Rule-based matching (amount, date, reference number, status, currency, account number).
   - Embedding/vector similarity for fuzzy matching of supplier names or descriptions (to handle typos, abbreviations, or format differences).

8. For each potential match, the system calculates a confidence score based on:

   - Exact field matches (amount, date, reference, status, currency, account number).
   - Similarity scores from embeddings for text fields (supplier name, description).
   - Any other relevant business logic.

   **Field Weights (example):**

   - Amount: 0.35
   - Date: 0.15
   - Reference Number: 0.15
   - Supplier Name/Description: 0.15
   - Status: 0.10
   - Currency: 0.05
   - Account Number: 0.05

9. The system automatically reconciles high-confidence matches and presents them as matched.

10. Medium- or low-confidence matches are flagged for user review. The user is shown the suggested matches, confidence scores, and reasons for uncertainty.

11. The user reviews, confirms, adjusts, or rejects these matches. If a payment covers multiple invoices or is a partial payment, the user can allocate the payment accordingly.

12. The system keeps an audit trail of all user actions and decisions.

13. Once the user is satisfied, they finalize the reconciliation. The system updates the status of invoices and payments, generates a reconciliation report, and optionally syncs the results back to the accounting system.

14. The user can export reports, view analytics, or set up notifications for future reconciliation cycles.

---

## Matching Algorithm and Field Weights

The reconciliation system uses a hybrid matching algorithm that combines rule-based logic and embedding-based similarity (cosine similarity) for text fields.

- **Amount:** Highest importance. An exact match is required for high confidence. (e.g., weight: 0.35)
- **Date:** High importance. A match within a configurable date range (e.g., ±2 days) is considered. (e.g., weight: 0.15)
- **Reference Number:** High importance. Exact or near-exact match (allowing for minor typos or formatting). (e.g., weight: 0.15)
- **Supplier Name/Description:** Medium importance. Compared using embeddings and cosine similarity to handle variations, abbreviations, or typos. (e.g., weight: 0.15)
- **Status:** Ensures only valid, actionable records are matched (e.g., completed bank transactions to unpaid/partially paid invoices). (e.g., weight: 0.10)
- **Currency:** Ensures both records are in the same currency. (e.g., weight: 0.05)
- **Account Number:** Ensures the correct account is used, especially for businesses with multiple accounts. (e.g., weight: 0.05)

**Example Confidence Score Calculation:**

Suppose a bank transaction and an invoice have the following:

- Amount matches exactly (score: 1.0 × 0.35 = 0.35)
- Date is within 1 day (score: 1.0 × 0.15 = 0.15)
- Reference number is a close match (score: 0.8 × 0.15 = 0.12)
- Supplier name cosine similarity is 0.9 (score: 0.9 × 0.15 = 0.135)
- Status is compatible (score: 1.0 × 0.10 = 0.10)
- Currency matches (score: 1.0 × 0.05 = 0.05)
- Account number matches (score: 1.0 × 0.05 = 0.05)

Total confidence score = 0.35 + 0.15 + 0.12 + 0.135 + 0.10 + 0.05 + 0.05 = 0.905 (high confidence, auto-reconciled)

If the status was incompatible (score: 0.0), the total would be 0.35 + 0.15 + 0.12 + 0.135 + 0.0 + 0.05 + 0.05 = 0.805 (medium confidence, flagged for review).

The system can adjust weights and thresholds based on business needs, and the confidence score determines whether a match is auto-reconciled or sent for user review.

## Human Feedback Loop and System Improvement

The system implements a continuous learning mechanism that leverages human feedback to improve its matching accuracy and efficiency over time.

### Feedback Collection

1. **User Action Tracking**

   - System records all user corrections, adjustments, and rejections of suggested matches
   - Captures the original confidence score and the user's final decision
   - Stores the specific fields that were modified or corrected

2. **Pattern Analysis**
   - Analyzes common correction patterns across multiple reconciliation cycles
   - Identifies systematic issues in the matching algorithm
   - Tracks success rates of different matching strategies

### Improvement Mechanisms

1. **Rule-Based Matching Improvements**

   - Adjusts field weights based on correction patterns
   - Refines date tolerance ranges based on actual business practices
   - Updates reference number matching rules based on common formats
   - Example: If users frequently accept matches with 3-day date differences, the system may increase the date tolerance threshold

2. **LLM/OCR Enhancement**

   - Uses corrected invoice data as training examples
   - Improves extraction accuracy for specific document formats
   - Learns business-specific terminology and formatting
   - Example: If users frequently correct supplier names in a particular format, the system improves its recognition of that format

3. **Embedding Model Refinement**

   - Updates supplier name embeddings based on confirmed matches
   - Learns business-specific abbreviations and variations
   - Improves handling of industry-specific terminology
   - Example: If "Acme Corp" and "Acme Corporation" are frequently confirmed as matches, the system learns to treat these variations as equivalent

4. **Business Rule Learning**
   - Identifies patterns in payment allocations
   - Learns common partial payment scenarios
   - Adapts to business-specific matching preferences
   - Example: If users consistently split payments in a particular way, the system learns to suggest similar splits in future cases

### Implementation Process

1. **Data Collection**

   - Maintains an audit trail of all user actions
   - Stores original and corrected values
   - Records confidence scores and decision outcomes

2. **Analysis Phase**

   - Periodically analyzes collected feedback
   - Identifies improvement opportunities
   - Generates improvement recommendations

3. **Model Updates**

   - Implements improvements in a controlled manner
   - Tests changes against historical data
   - Validates improvements before full deployment

4. **Monitoring and Validation**
   - Tracks improvement metrics
   - Measures impact on matching accuracy
   - Adjusts improvement strategies based on results

This feedback loop creates a self-improving system that becomes more accurate and efficient over time, adapting to the specific needs and patterns of each business.
