
**Project Overview**  
The purpose of this project is to determine whether supplier audits have a measurable impact on the reject rate of received goods. By linking audit records with receipt data and analysing reject outcomes, the project provides a data‑driven answer to the business question: *Do audits reduce the likelihood of a reject once receipt volume is taken into account?*  

**Business Problem**  
The organization tracks two separate data sources:  
1. **Audits** – records of supplier audits with dates.  
2. **Receipts/Rejects** – each receipt is flagged as a reject (0) or a non‑reject (1).  

Decision‑makers need to know whether allocating resources to additional audits yields a tangible reduction in rejects, or whether the observed reject patterns are independent of audit activity. The current process does not quantify this relationship, making it difficult to justify audit‑related spend.  

**Solution Overview**  

| Step | Description |
|------|-------------|
| **Data Extraction** | Pull audit data and receipt/reject data from a SQL Server database using `pyodbc`. |
| **Feature Engineering** | Create a binary flag `audit_last_90` that is set to 1 when a receipt’s supplier was audited within the 90 days preceding the receipt date. |
| **Quadrant Counting** | Using `audit_last_90` together with `reject_flag`, generate four count columns per supplier: <br>• `audit_non_rejects` (audit = 1, reject = 0) <br>• `audit_rejects` (audit = 1, reject = 1) <br>• `non_audit_non_rejects` (audit = 0, reject = 0) <br>• `non_audit_rejects` (audit = 0, reject = 1) |
| **Filtering** | Exclude any supplier that has ≤ 5 observations in any of the four categories to avoid unstable ratio estimates. |
| **Ratio Calculation** | Compute two reject‑rate ratios per supplier: <br>• `audit_ratio = audit_non_rejects / audit_rejects` <br>• `non_audit_ratio = non_audit_non_rejects / non_audit_rejects` |
| **Statistical Testing** | Perform a paired t‑test (audit vs. non‑audit ratios) and calculate Cohen’s *d* to quantify effect size. |
| **Outlier Treatment** | Apply the IQR box‑plot rule to the difference `non_audit_ratio – audit_ratio`, remove extreme points, and repeat the statistical test. |
| **Reporting** | Summarise the findings in tables and visualisations (histograms of the differences before/after outlier removal). |

**My Contribution**  
- **Data Extraction & Cleaning:** Wrote the SQL queries, set up the ODBC connections, and ensured date fields were correctly parsed.  
- **Feature Engineering:** Implemented the `flag_receipt_audit` function that generates the 90‑day audit flag, handling duplicates, missing values, and supplier‑id mismatches.  
- **Analysis & Visualization:** Built the quadrant count tables, ratio calculations, performed the paired t‑test and Cohen’s *d* calculations, applied IQR‑based outlier removal, and produced the diagnostic plots.  
- **Documentation:** Authored the notebook narrative, prepared this project description, and created a concise README for portfolio use.

**Business Value**  
- **Evidence‑Based Decision Making:** The analysis shows that, after controlling for outliers, audits do **not** have a statistically significant effect on reject rates (p > 0.05, Cohen’s d ≈ 0).  
- **Resource Allocation Insight:** Management can confidently re‑evaluate the budget spent on audits, focusing instead on factors that more directly influence reject outcomes.  
- **Scalable Framework:** The methodology (audit flag creation, quadrant counting, paired testing) can be applied to other supplier‑performance metrics or to different time windows (e.g., 30 days, 180 days) with minimal code changes.

**Challenges Encountered**  
- **Data Alignment:** Supplier identifiers were not consistently formatted across the audit and receipt tables, requiring preprocessing and careful handling of join keys.  
- **Sparse Rejects for Audited Receipts:** Several suppliers had zero rejects in the audited group, leading to division‑by‑zero issues; these were handled by filtering low‑volume suppliers and by replacing infinities with `NaN`.  
- **Outlier Influence:** Initial statistical tests indicated significance, but a small number of extreme ratio differences inflated the result. Applying the IQR rule resolved this and provided a more reliable conclusion.

**Key Takeaways**  
1. **Statistical Rigor Trumps Model Complexity:** Even with modest data, a well‑structured hypothesis test can deliver actionable insights without resorting to heavyweight predictive models.  
2. **Data Quality Is Paramount:** Small inconsistencies in key fields (supplier IDs, dates) can cascade into major analytical errors; investing time in cleaning pays off.  
3. **Iterative Validation Is Essential:** Outlier detection and sensitivity analysis are critical steps before reporting statistical significance.  

---

### Questions for You (to tailor the description further)

1. **Company/Domain Context** – Which organization or business unit is the primary stakeholder for this analysis? (e.g., “Procurement team at XYZ Corp.”)  
2. **Data Scope** – What time period does the dataset cover (e.g., Jan 2022 – Dec 2023)?  
3. **Audit Frequency** – Approximately how many audits occur per supplier per year? This helps contextualise the 90‑day window.  
4. **Reject Definition** – Is the `reject_flag` a binary indicator of any reject, or does it encode severity/quantity?  
5. **Stakeholder Preferences** – Do decision‑makers prefer a particular presentation format (PowerPoint deck, interactive dashboard, PDF report)?  
6. **Future Extensions** – Are there plans to incorporate additional variables (e.g., product category, supplier location) into the analysis?  
7. **Performance Metrics** – Besides statistical significance, are there operational targets (e.g., “reduce reject rate by 5 %”) that the business wants to track?  

Providing answers to the above will allow the README and any accompanying presentation to be fully aligned with the intended audience and business context.
