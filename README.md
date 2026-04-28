# E-Learning Business Analytics

This interactive Tableau dashboard is analyzing the performance of a fictional online learning platform.
The project combines business KPIs, student behavior, acquisition performance and course-level insights in one report to support data-driven decisions.

The dashboard was built using Tableau. 
[You can explore the interactive Tableau dashboard here](https://public.tableau.com/app/profile/alesia.alieva/viz/E-LearningBusinessAnalytics/DashboardOverview).

<p align="center">
  <img src="assets/1s.png" width="49%">
  <img src="assets/2s.png" width="49%">
</p>

⚠️ The dataset used in this project is synthetic and was generated with **Claude AI** for portfolio purposes.

## Data Pipeline
The dashboard was built through a structured analytics workflow:  

[Claude AI] → raw CSVs → [Python + DuckDB] → quality checks → staging → marts → [Tableau]  

1. **Synthetic Data Generation**
   
   Raw source data was generated with Claude AI in CSV format.

3. **Raw Data Layer**
   
   Multiple source tables were stored as raw CSV files:

    | Table | Description | Key Fields |
    |------|-------------|------------|
    | students | Student registration and profile data | student_id, reg_date, country, acquisition_channel |
    | courses | Course catalog information | course_id, title, category, instructor_id, price, duration_hours |
    | enrollments | Student course purchases | enrollment_id,	student_id,	course_id,	enroll_date,	completion_pct |
    | payments | Payment and refund transactions | payment_id,	student_id,	course_id,	amount,	status,	date |
    | instructors | Instructor details | instructor_id, instructor_name, tier, subject_area |


3. **Data Processing (Python + DuckDB)**

   All data preparation was completed in Jupyter Notebook using Python + DuckDB.
   SQL queries were executed in DuckDB for cleaning, joins, transformations and metric preparation.

- **Quality Checks**
  
   Validation steps included null checks, duplicates review and business logic controls.

  Examples:
  ```sql
  --Check required fields for NULL values.
  SELECT
    COUNT(*) AS total_rows,
    SUM(CASE WHEN payment_id IS NULL THEN 1 ELSE 0 END) AS null_payment_id,
    SUM(CASE WHEN student_id IS NULL THEN 1 ELSE 0 END) AS null_student_id,
    
    -- course_id ALWAYS must be filled in
    SUM(CASE WHEN course_id  IS NULL OR course_id = '' THEN 1 ELSE 0 END) AS null_course_id,
    SUM(CASE WHEN amount     IS NULL THEN 1 ELSE 0 END) AS null_amount,
    SUM(CASE WHEN status     IS NULL THEN 1 ELSE 0 END) AS null_status,
    SUM(CASE WHEN date       IS NULL THEN 1 ELSE 0 END) AS null_date
  FROM payments
  ```

  ```python
  # Checking primary keys for duplicates
  
  pk_checks = [
    ('students',    'student_id'),
    ('courses',     'course_id'),
    ('instructors', 'instructor_id'),
    ('enrollments', 'enrollment_id'),
    ('payments',    'payment_id'),
    ]

  for table, pk in pk_checks:
    path = TABLES[table]
    result = con.sql(f"""
        SELECT COUNT(*) - COUNT(DISTINCT {pk}) AS duplicate_pks
        FROM '{path}'
     """).df()
    val = result.iloc[0, 0]
    print(f'   {table}.{pk} — duplicates: {val}')
  ```



- **Staging Layer**  
   Cleaned and standardized intermediate tables were created for further transformations.

- **Data Marts**  
   Final analytical tables were built for KPI calculations, segmentation, retention, and dashboard reporting.

4. **Visualization**  
   Processed marts were connected to Tableau to build the final interactive dashboard.
