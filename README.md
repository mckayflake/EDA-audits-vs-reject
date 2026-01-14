
**Project Overview**  
The Audit Team at Northrop Grumman examined five years of audit and receipt data to determine whether supplier audits reduce the reject rate of received items.

**Business Problem**  
The organization records audit dates for delegated suppliers (≈ 6‑10 audits per supplier per year) and flags each receipt as a reject (binary). Management needs quantitative evidence on whether the current audit cadence actually lowers reject occurrences, enabling informed decisions on audit resource allocation.

**Solution Overview**  

| Step | Description |
|------|-------------|
| Data extraction | Pull audit and receipt data from the corporate SQL Server via `pyodbc`. |
| Feature engineering | Create a binary flag `audit_last_90` that is 1 when a receipt’s supplier was audited within the 90 days prior to the receipt date. |
| Quadrant counting | Combine `audit_last_90` with `reject_flag` to generate four supplier‑level counts: `audit_non_rejects`, `audit_rejects`, `non_audit_non_rejects`, `non_audit_rejects`. |
| Filtering | Exclude any supplier with ≤ 5 observations in any of the four categories to avoid unstable ratio estimates. |
| Ratio calculation | Compute `audit_ratio = audit_non_rejects / audit_rejects` and `non_audit_ratio = non_audit_non_rejects / non_audit_rejects`. |
| Statistical testing | Perform a paired t‑test between the two ratios and calculate Cohen’s d for effect size. |
| Outlier treatment | Apply the IQR box‑plot rule to the difference (`non_audit_ratio – audit_ratio`), remove extreme points, and repeat the paired test. |
| Reporting | Summarise results in tables and histograms of the ratio differences before and after outlier removal. |

**My Contribution**  
- Authored all SQL queries and configured ODBC connections.  
- Implemented the `flag_receipt_audit` function to generate the 90‑day audit flag, handling duplicates, missing values, and supplier‑ID mismatches.  
- Built the quadrant count tables, performed ratio calculations, conducted paired t‑tests and Cohen’s d calculations, and applied IQR‑based outlier removal.  
- Created diagnostic visualisations and documented the entire workflow in a reproducible Jupyter notebook.  

**Business Value**  
- The analysis shows that, after outlier removal, audits do **not** have a statistically significant effect on reject rates (p > 0.05, Cohen’s d ≈ 0).  
- Provides evidence‑based guidance for the audit team to reassess audit frequency and allocate resources more effectively.  
- Supplies a reusable analytical framework that can be applied to other time windows or supplier‑performance metrics.  

**Challenges Encountered**  
- Inconsistent supplier identifiers across audit and receipt tables required preprocessing and careful join handling.  
- Several suppliers had zero rejects in the audited group, leading to division‑by‑zero; addressed by filtering low‑volume suppliers and handling infinities.  
- Initial significance was driven by a few extreme ratio differences; resolved by applying the IQR outlier rule before final testing.  

**Key Takeaways**  
1. A well‑designed statistical test can deliver actionable insight without resorting to complex predictive models.  
2. Data quality and consistent key fields are essential for reliable results.  
3. Iterative validation, including outlier detection, is critical before drawing conclusions.
