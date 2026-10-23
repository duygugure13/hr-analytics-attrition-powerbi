# HR Analytics & Employee Attrition Intelligence Dashboard

An end-to-end People Analytics solution developed in **Power BI**, transforming normalized corporate human resources records into actionable operational and executive intelligence. The project investigates root causes of workforce attrition across departmental, compensation, tenure, and satisfaction dimensions using dimensional data modeling (**Star Schema**) and robust **DAX** formulations.

---

##  Executive Summary & Key Analytical Findings

Across the analyzed organization (**1,470 employees**), baseline attrition stands at **16.12% (237 voluntary exits)**. Multivariate breakdown reveals targeted vulnerability clusters rather than uniform departures:

* **Overtime Friction:** Employees subject to regular overtime exhibit an attrition rate of **30.53%**, nearly triple the rate of non-overtime peers (**10.44%**).
* **Early-Career & Onboarding Churn:** Exit propensity peaks dramatically in the first year of tenure (**0–1 Year: 34.88%** attrition), flattening steadily as tenure advances (**10+ Years: 2.50%**).
* **Role-Specific Vulnerability:** Highest turnover rates concentrate in frontline field and operational technical roles: **Sales Representatives (39.76%)**, **Laboratory Technicians (23.94%)**, and **Human Resources Specialists (23.08%)**.
* **Compounded Discontentment:** A custom satisfaction matrix confirms that employees simultaneously rating lowest on both **Work-Life Balance (Rating 1)** and **Job Satisfaction (Rating 1)** experience an acute attrition rate of **47.06%**.
* **Supervisory Transition Risk:** First-year tenure under a new manager yields a **32.32%** attrition rate, stabilizing substantially by the third and fourth years.

---

##  Data Architecture & Star Schema Modeling

The source dataset (IBM HR Analytics Attrition dataset, 1,470 rows × 35 columns) originally presented a flattened single-table structure containing mixed dimensional attributes and transactional metrics. To optimize VertiPaq engine performance and ensure strict reporting governance, the dataset was refactored into a Kimball-standard **Star Schema** via Power Query.

<img width="1075" height="665" alt="Star Schema" src="https://github.com/user-attachments/assets/f1c3ba27-12b2-4dfe-8b2b-c8138b72d52d" />


### 1. Data Cleaning & Normalization Steps
* **Zero-Variance Column Pruning:** Removed redundant constant fields (`EmployeeCount` = 1, `StandardHours` = 80, `Over18` = 'Y') to reduce table footprint and memory overhead.
* **Dimensional Separation:**
  * **`Dim_Employee`**: Isolates static demographic entities (`EmployeeNumber` [PK], `Age`, `Gender`, `MaritalStatus`, `EducationField`, `DistanceFromHome`).
  * **`Dim_JobRole`**: Isolates organizational structures (`JobRoleID` [PK surrogate created via Index], `Department`, `JobRole`, `JobLevel`).
  * **`Fact_Attrition`**: Retains foreign keys (`EmployeeNumber`, `JobRoleID`), compensation facts (`MonthlyIncome`), satisfaction ratings (1–4), tenure parameters, and the target indicator (`Attrition`).
* **Relationship Integrity:** Configured strictly as **One-to-Many (`1:*`)** single-direction relationships from Dimensions to Fact, guaranteeing unambiguous filter context propagation.

### 2. Calculated Categorical Groupings
To eliminate high-cardinality noise in visual distributions, continuous numeric attributes were transformed into categorical buckets:

* **Age Cohorts (`Age Group`):**
  ```dax
  Age Group = 
  SWITCH(
      TRUE(),
      'Dim_Employee'[Age] < 25, "18-24",
      'Dim_Employee'[Age] <= 34, "25-34",
      'Dim_Employee'[Age] <= 44, "35-44",
      'Dim_Employee'[Age] <= 54, "45-54",
      "55+"
  )
  
* **Tenure Cohorts (`Tenure Group`):**
```dax
Tenure Group = 
SWITCH(
    TRUE(),
    'Fact_Attrition'[YearsAtCompany] <= 1, "0-1 Yıl",
    'Fact_Attrition'[YearsAtCompany] <= 3, "2-3 Yıl",
    'Fact_Attrition'[YearsAtCompany] <= 5, "4-5 Yıl",
    'Fact_Attrition'[YearsAtCompany] <= 10, "6-10 Yıl",
    "10+ Yıl"
)

```

### Core DAX Formulations
All analytical metrics were authored inside a dedicated, disconnected _Measures table using strict denominator management to prevent contextual filter collapse:

// 1. Total Headcount (Scoped to Fact Table to Preserve Relationship Filtering)
Total Employees = 
COUNTROWS('Fact_Attrition')

// 2. Active Workforce Headcount
Active Employees = 
CALCULATE(
    [Total Employees],
    'Fact_Attrition'[Attrition] = "No"
)

// 3. Voluntary Exit Headcount
Attrition Count = 
CALCULATE(
    [Total Employees],
    'Fact_Attrition'[Attrition] = "Yes"
)

// 4. Baseline Attrition Rate
Attrition Rate = 
DIVIDE(
    [Attrition Count],
    [Total Employees],
    0
)

// 5. Overtime Attrition Impact
Attrition Rate (OverTime) = 
CALCULATE(
    [Attrition Rate],
    'Fact_Attrition'[OverTime] = "Yes"
)

// 6. Average Monthly Compensation
Avg Monthly Income = 
AVERAGE('Fact_Attrition'[MonthlyIncome])
 
 
### Dashboard Architecture & Interactive Visualizations
The dashboard is structured into a logical 3-tier narrative flow: Macro Level ➔ Diagnostic Level ➔ Operational Micro Level.




* **Page 1: Executive Overview**
KPI Header Cards: Real-time summary of Total Employees (1,470), Active Headcount (1,233), Attrition Count (237), Attrition Rate (16.12%), and Average Monthly Income ($6,502.93).

Attrition Rate by Job Role (Horizontal Clustered Bar): Ranks turnover velocity by role, spotlighting operational roles needing immediate intervention.

Workforce Demographics (Stacked Column): Evaluates headcount distribution and churn volume across structured Age Group segments.

Tenure Decay Curve (Area Chart): Illustrates the probability curve of departure relative to organizational longevity.

<img width="1292" height="556" alt="Executive Overview" src="https://github.com/user-attachments/assets/806ca30a-d61c-4d38-993c-78f1b741bc48" />





* **Page 2: Attrition Drivers & Deep Dive**
OverTime Variance (Donut / Segmented Visual): Directly evaluates the comparative attrition divergence between overtime cohorts (30.53% vs. 10.44%).

Compensation vs. Stagnation Matrix (Scatter Plot): Correlates MonthlyIncome against YearsSinceLastPromotion, color-coded by exit status to detect retention patterns among undercompensated personnel.

Cross-Satisfaction Heatmap (4×4 Matrix): Bivariate matrix cross-referencing WorkLifeBalance with JobSatisfaction. Features conditional gradient formatting pinpointing the acute 47.06% risk sector.

Decomposition Tree (Root Cause Drill-Down): Interactive AI visual decomposing total departures (237) dynamically through categorical pathways: OverTime ➔ BusinessTravel ➔ MaritalStatus ➔ JobRole.

<img width="1030" height="696" alt="Attrition Drivers   Deep Dive" src="https://github.com/user-attachments/assets/e6a147cc-37ad-4164-a875-ce7dd5bb1b2f" />




* **Page 3: Employee Details & Action Plan**
Managerial Tenure Impact (Clustered Column): Maps departure frequency against YearsWithCurrManager, displaying vulnerability spikes during leadership transitions.

Global Slicer Panel: Multi-dimensional cross-filtering through Tenure Group, Age Group, EducationField, and BusinessTravel.

Actionable Personnel Ledger (Table): Granular employee-level data grid featuring rule-based conditional formatting (Attrition = "Yes" flagged in red) for targeted HR retention and exit interviews.

<img width="741" height="727" alt="Employee Details   Action Plan" src="https://github.com/user-attachments/assets/3aff8aee-ad82-4d41-b19b-5eede93f85c5" />


### Technical Stack & Environment
Analytics & Visualization: Microsoft Power BI Desktop

Data Transformation: Power Query (M Language)

Modeling Logic: DAX (Data Analysis Expressions)
