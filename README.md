# 📊 Decoding the Data Analyst Job Market: SQL Project

> **Analyzing the job market to define an optimal career strategy in the data sphere.**

## 📑 Table of Contents
- [Introduction & Goal](#-introduction--goal)
- [Tech Stack](#-tech-stack)
- [Key Analysis Steps](#-key-analysis-steps)
- [Insights & Conclusions](#-insights--conclusions)
- [Why This Matters](#-why-this-matters)

## 🎯 Introduction & Goal

This project stemmed from a simple yet critical question: **"How can a Data Analyst maximize their market value?"**

Instead of relying on guesswork, I utilized **real-world job posting data** to construct a targeted job search strategy. The dataset (provided by Luke Barousse's SQL Course) includes detailed information on job titles, salaries, locations, and required skills.

**Key Research Questions:**
1. 💰 Where is the money? (Top paying roles)
2. 🛠️ What technical stack guarantees high income?
3. 📈 Which skills are in highest demand?
4. ⚖️ **The "Golden Mean":** Which skills perfectly balance high demand with high salary?

Check out my SQL queries here: [project_sql folder](/project_sql/). 

## 🛠 Tech Stack

I leveraged the following tools to process the data and extract actionable insights:

| Tool | Purpose |
| :--- | :--- |
| **SQL** | Core analysis: complex queries, aggregations, CTEs. |
| **PostgreSQL** | Database management system (handling the job posting dataset). |
| **VS Code** | Development environment and database administration. |
| **Git & GitHub** | Version control and project presentation. |

## 🔍 Key Analysis Steps

Each query in this project solves a specific business problem. Below is the logic behind the analysis and the corresponding SQL code.

### 1. Identifying Elite Opportunities (Top Paying Jobs)
*Goal: To identify the market "ceiling" for remote Data Analyst roles.*

I filtered for Data Analyst positions, focusing on remote work (`Anywhere`) with the highest annual salaries, ensuring non-null values were excluded to maintain data integrity.


<details>
<summary><strong>👇 Click to view SQL Code</strong></summary>

```sql
SELECT
	job_id,
	job_title,
	job_location,
	job_schedule_type,
	salary_year_avg,
	job_posted_date,
    name AS company_name
FROM
    job_postings_fact
LEFT JOIN company_dim ON job_postings_fact.company_id = company_dim.company_id
WHERE
    job_title_short = 'Data Analyst' AND
    job_location = 'Anywhere' AND
    salary_year_avg IS NOT NULL
ORDER BY
    salary_year_avg DESC
LIMIT 10; 
```
</details>

### 2. Deconstructing Top-Tier Skills
*Goal: To identify the specific skills required for the top 10 highest-paying Data Analyst jobs.*

This query provides a detailed look at which high-paying jobs demand certain skills, helping job seekers understand which skills to develop that align with top salaries.

<details> <summary><strong>👇 Click to view SQL Code</strong></summary>

```sql
WITH top_paying_jobs AS (
    SELECT
        job_id,
        job_title,
        salary_year_avg,
        name AS company_name
    FROM
        job_postings_fact
    LEFT JOIN company_dim ON job_postings_fact.company_id = company_dim.company_id
    WHERE
        job_title_short = 'Data Analyst' AND
        job_location = 'Anywhere' AND
        salary_year_avg IS NOT NULL
    ORDER BY
        salary_year_avg DESC
    LIMIT 10
)

SELECT 
    top_paying_jobs.*,
    skills_dim.skills
FROM top_paying_jobs
INNER JOIN skills_job_dim ON top_paying_jobs.job_id = skills_job_dim.job_id
INNER JOIN skills_dim ON skills_job_dim.skill_id = skills_dim.skill_id
ORDER BY
    salary_year_avg DESC;
```
</details>

### 3. Demand Analysis (Volume vs Value)
*Goal: To identify the top 5 in-demand skills for a data analyst.*

I focused on all job postings to retrieve the top 5 skills with the highest demand in the job market, providing insights into the most valuable skills for job seekers.

<details> <summary><strong>👇 Click to view SQL Code</strong></summary>

```sql
SELECT
    skills_dim.skills,
    COUNT(skills_job_dim.job_id) AS demand_count
FROM job_postings_fact
INNER JOIN skills_job_dim ON job_postings_fact.job_id = skills_job_dim.job_id
INNER JOIN skills_dim ON skills_job_dim.skill_id = skills_dim.skill_id
WHERE
    job_postings_fact.job_title_short = 'Data Analyst' 
GROUP BY
    skills_dim.skills
ORDER BY
    demand_count DESC
LIMIT 5;
```
</details>

| Rank | Skill | Demand Count |
| :---: | :--- | :---: |
| 1 | **SQL** | 180,369 |
| 2 | **Excel** | 131,822 |
| 3 | **Python** | 116,082 |
| 4 | **Tableau** | 90,588 |
| 5 | **Power BI** | 84,353 |

*Table of the demand for the top 5 skills in data analyst job postings*

### 4. Financial Ranking of Skills
*Goal: To identify the top skills based on average salary.*

This analysis looks at the average salary associated with each skill for Data Analyst positions. It reveals how different skills impact salary levels and helps identify the most financially rewarding skills to acquire or improve.

<details> <summary><strong>👇 Click to view SQL Code</strong></summary>

```sql
SELECT
    skills_dim.skills,
    ROUND(AVG(job_postings_fact.salary_year_avg), 0) AS avg_salary
FROM job_postings_fact
INNER JOIN skills_job_dim ON job_postings_fact.job_id = skills_job_dim.job_id
INNER JOIN skills_dim ON skills_job_dim.skill_id = skills_dim.skill_id
WHERE
    job_postings_fact.job_title_short = 'Data Analyst' 
    AND job_postings_fact.salary_year_avg IS NOT NULL
GROUP BY
    skills_dim.skills
ORDER BY
    avg_salary DESC
LIMIT 25;
```
</details>

| Rank | Skill | Average Salary |
| :---: | :--- | :---: |
| 1 | **FastAPI** | $212,500 |
| 2 | **SVN** | $185,000 |
| 3 | **Blazor** | $161,000 |
| 4 | **APL** | $155,000 |
| 5 | **MXNet** | $149,000 |

*Table of the average salary for the top 5 paying skills for data analysts*

### 5. Strategic Selection: Optimal Skills
*Goal: To identify the most optimal skills to learn (high demand + high salary).*

This query targets skills that offer job security (high demand) and financial benefits (high salaries), offering strategic insights for career development in data analysis. I filtered for remote work opportunities (job_work_from_home = TRUE) to focus on the global market.

<details> <summary><strong>👇 Click to view SQL Code</strong></summary>

```sql
SELECT
   skills_dim.skill_id,
   skills_dim.skills, 
   COUNT(skills_job_dim.job_id) as demand_count,
   ROUND(AVG(job_postings_fact.salary_year_avg),0) AS avg_salary
FROM
   job_postings_fact
   INNER JOIN
   skills_job_dim ON job_postings_fact.job_id = skills_job_dim.job_id
   INNER JOIN
   skills_dim ON skills_job_dim.skill_id = skills_dim.skill_id
WHERE
   job_postings_fact.job_title_short = 'Data Analyst'
   AND job_postings_fact.salary_year_avg IS NOT NULL
   AND job_postings_fact.job_work_from_home = TRUE
GROUP BY
   skills_dim.skill_id
HAVING
    COUNT(skills_job_dim.job_id) > 10
ORDER BY
    demand_count DESC,
    avg_salary DESC
LIMIT 25;
```
</details>

| Rank | Skill | Demand Count | Average Salary |
| :---: | :--- | :---: | :---: |
| 1 | **SQL** | 783 | $98,748 |
| 2 | **Tableau** | 505 | $102,172 |
| 3 | **Python** | 494 | $98,074 |
| 4 | **Excel** | 441 | $88,202 |
| 5 | **Power BI** | 293 | $95,535 |

*Table of the most optimal skills for data analyst*

## 💡 Insights & Conclusions

Based on the analysis, the following strategic conclusions can be drawn for a Data Analyst career:

### 1. Top Skills for High-Paying Roles
* **SQL (100%):** Every single one of the top 10 highest-paying roles requires SQL. It remains the foundational skill for data analysis.
* **Python (87.5%):** Python is nearly as ubiquitous as SQL in top-tier roles.
* **Tableau (75%):** The dominant visualization tool in the high-end segment.

### 2. Salary Trends by Skill Category
* **DevOps & Engineering Crossover:** High-paying Analyst roles increasingly require infrastructure knowledge (Terraform, GitLab, Ansible). Companies pay a premium for "Full Stack" analysts who can manage their own data pipelines.
* **AI & Deep Learning Dominance:** Libraries like PyTorch, TensorFlow, and Hugging Face are heavily represented in top salaries, indicating that predictive modeling is valued higher than descriptive analysis.
* **Niche & Specialized Tech:** The absolute highest salaries go to specialized technologies (e.g., Solidity for Blockchain, Couchbase) rather than broad tools.

### 3. The Optimal Path
To maximize income while maintaining a broad range of job opportunities, the best strategy is to master **SQL** combined with **cloud technologies** or **Python**, as these sit at the intersection of high demand and high financial reward.



## 🧠 Why This Matters

This project demonstrates not just SQL syntax mastery, but the ability to:
1.  **Construct Complex Queries:** Utilizing CTEs, various JOINs, aggregations, and filtering.
2.  **Translate Data into Decisions:** Moving from raw tables to concrete career advice.
3.  **Understand Business Context:** Focusing on the ROI (Return on Investment) of skill acquisition.


