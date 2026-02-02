# Index Performance Discussion

This document analyzes the indexing in the internship database.
For each query, execution is compared **without indexes** and **with indexes**, using runtime performance.

The screenshots referenced below are located in the `screenshots/` folder.

---

## Query 1: Applications per Internship

**Query goal:**  
Count how many applications were submitted for each internship.

### Query
```sql
SELECT i.title,
       COUNT(ia.id) AS application_count
FROM internship AS i
LEFT JOIN internship_application AS ia ON i.id = ia.internship_id
GROUP BY i.id, i.title;
```

### Index 
```sql
CREATE INDEX idx_application_internship
    ON internship_application(internship_id)
```

Without index: 141ms  
With index: 114ms

---

## Query 2: Number of Interns per Company

**Query goal:**  
Count how many interns each company has.

### Query
```sql
SELECT c.name AS company,
       COUNT(ia.id) AS interns
FROM company AS c
JOIN internship AS i ON c.id = i.company_id
LEFT JOIN internship_assignment AS ia ON i.id = ia.internship_id
GROUP BY c.id, c.name;
```

### Index 
```sql
CREATE INDEX idx_internship_company
    ON internship(company_id);

CREATE INDEX idx_assignment_internship
    ON internship_assignment(internship_id);
```

Without index: 159ms  
With index: 96ms

---

## Query 3: Total Documents per Student

**Query goal:**  
Count how many documents were submitted for each student.

### Query
```sql
SELECT s.first_name, s.last_name,
		COUNT(d.id) AS total_documents
FROM student AS s
JOIN internship_assignment AS ia ON s.id = ia.student_id
LEFT JOIN document AS d ON ia.id = d.assignment_id
GROUP BY s.id, s.first_name, s.last_name;
```

### Index 
```sql
CREATE INDEX idx_assignment_student
    ON internship_assignment(student_id);
```

Without index: 174ms  
With index: 115ms

---

## Query 4: Acceptance Rate per Internship

**Query goal:**  
Calculate acceptance rate for each internship.

### Query
```sql
WITH accepted AS (
	SELECT internship_id,
			COUNT(*) AS total_applications,
			COUNT(*) FILTER (WHERE status = 'ACCEPTED') AS accepted_applications
	FROM internship_application
	GROUP BY internship_id
)

SELECT i.title, 
		total_applications, 
		accepted_applications,
		ROUND(accepted_applications::DECIMAL / total_applications * 100.0) AS acceptance_rate
FROM accepted AS a
JOIN internship AS i ON i.id = a.internship_id;
```

### Index 
```sql
CREATE INDEX idx_application_internship_status
	ON internship_application(internship_id, status);
```

Without index: 157ms  
With index: 118ms

---

## Query 5: Internship Participation per Study Program

**Query goal:**  
Calculate internship participation for each study program.

### Query
```sql
SELECT sp.name AS study_program,
		COUNT(ia.id) AS internships
FROM study_program AS sp
JOIN student AS s ON sp.id = s.study_program_id
LEFT JOIN internship_assignment AS ia ON s.id = ia.student_id
GROUP BY sp.id, sp.name;
```

### Index 
```sql
CREATE INDEX idx_student_study_program
    ON student(study_program_id);
```

Without index: 149ms  
With index: 92ms

## General Observations

- Indexes improve JOIN and WHERE performance.
- Indexes do not directly optimize aggregation (COUNT, AVG), but they improve filtering before it.
- Some queries show small improvement because of GROUP BY clause.
- Composite indexes are useful when queries filter or group by multiple columns together.
- ORDER BY was removed from the queries during performance testing to avoid sorting overhead.