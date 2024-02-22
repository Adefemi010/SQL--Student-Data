# SQL--Student-Data

## Question 1

Write a SQL script for all students (include all student table fields) taking Computer Science degree who are older than 30 (as of query run-time) and have an 'r' in their full name (upper or lowercase)

## SQL Query    
    
    SELECT 
        student_id, 
        student_name || ' ' || student_lastname AS fullname,
        student_udegree_id,
        student_start_year,
        student_birthdate,
        EXTRACT(YEAR FROM AGE(current_date, student_birthdate)) AS age,
        student_email
    FROM 
        student 
    WHERE 
        EXTRACT(YEAR FROM AGE(current_date, student_birthdate)) > 30
        AND LOWER(student_name || ' ' || student_lastname) LIKE '%r%';

## Question 2

Write a SQL script returning details of all students that are part of the 2018 cohort(the start year for students) of the IT degree, sort them from oldest to youngest The output should contain student_id, firstname, surname, the full name of the degree (not shortened ID), birthdate, and age in years when they started the degree

## SQL Query 

    SELECT 
        st.student_id,
        st.student_name AS firstname, 
        st.student_lastname AS surname,
        st.student_name || ' ' || st.student_lastname AS fullname,
        ud.udegree_name,
        st.student_birthdate AS birthdate,
        EXTRACT(YEAR FROM AGE(current_date, st.student_birthdate)) - 6 AS entry_age
    FROM 
        student AS st
    LEFT JOIN 
        udegree AS ud ON st.student_udegree_id = ud.udegree_id
    WHERE 
        st.student_start_year = '2018'
        AND st.student_udegree_id = 'IT'
    ORDER BY 
        entry_age DESC;

## Question 3
Write a SQL script which returns the grades of all students that have been graded for the statistics course, sorted by the highest to lowest grade
Scores below 50 should be marked as a fail
Display the student id, firstname, surname, grade and whether they passed or failed the course

 ## SQL Query
 
    SELECT 
        st.student_id,
        st.student_name AS firstname, 
        st.student_lastname AS surname,
        c.course_name,
        sc.grade,
        CASE
            WHEN sc.grade >= 50 THEN 'pass' 
            ELSE 'fail'
        END AS status
    FROM 
        student AS st
    LEFT JOIN 
        student_course AS sc ON st.student_id = sc.student_id 
    LEFT JOIN 
        course AS c ON c.course_id = sc.course_id
    WHERE 
        c.course_name = 'Statistics'
        AND sc.grade IS NOT NULL
    ORDER BY 
        sc.grade DESC;
        
## Question 4

Write a SQL script that shows the highest grade by course by cohort year (the start year for students)
Your output should be in cross-tab format with the full course name in rows and the cohort year across the columns (with max grade in the values)

## SQL Query 

    SELECT * FROM crosstab(
        'SELECT 
            course_name,
            student_start_year,
            MAX(grade)  
        FROM 
            course AS c
            LEFT JOIN student_course AS sc ON c.course_id = sc.course_id
            LEFT JOIN student AS s ON s.student_id = sc.student_id
        GROUP BY 
            course_name, student_start_year
        ORDER BY 
            1,2'
    ) AS final_result(course_name text, "cohort_2016" int, "cohort_2017" int, "cohort_2018" int, "cohort_2019" int, "cohort_2020" int);



## Question 4


Write a SQL script for as above but instead of displaying the course grade by cohort year, display the FULL name of the student who achieved that grade
Note: in the case where more than one student achieved that grade the alphabetical first record can be used
Your result should still appear the cross-tab format described in the previous question with concatenated name instead of the grade

## SQL Query 

    SELECT * FROM crosstab(
        'SELECT 
            course_name,
            student_start_year,
    		MAX(CONCAT_WS('' '', s.student_name, s.student_lastname)) AS fullname
         
        FROM 
            course AS c
            LEFT JOIN student_course AS sc ON c.course_id = sc.course_id
            LEFT JOIN student AS s ON s.student_id = sc.student_id
        GROUP BY 
            course_name, student_start_year
        ORDER BY 
            1,2'
    ) AS final_result(course_name text, "cohort_2016" text, "cohort_2017" text, "cohort_2018" text, "cohort_2019" text, "cohort_2020" text);
    


## Question 4

Write a SQL query that shows all cohorts (student start years) for all degrees and for each show how many students have obtained a top grade on at least one course
Output should be degree name, cohort year and count of students achieving a top grade in that degree
A top grade can be defined as the highest mark for a given course out of any cohort years
Hence there will be some cohorts where no students achieved a top grade in any of their courses

## SQL Query 

    SELECT 
        ug.udegree_name,
        s.student_start_year,
        COUNT(DISTINCT s.student_id) AS top_grade_students_count
    FROM 
        student s
    JOIN 
        student_course AS sc ON s.student_id = sc.student_id
    JOIN 
    	udegree AS ug ON s.student_udegree_id = ug.udegree_id
    JOIN 
        (
            SELECT 
                uc.course_id,
                MAX(sc.grade) AS top_grade
            FROM 
                student_course AS sc
            JOIN 
                udegree_course uc ON sc.course_id = uc.course_id
            GROUP BY 
                uc.course_id
        ) top_grades ON sc.course_id = top_grades.course_id AND sc.grade = top_grades.top_grade
    JOIN 
        (
            SELECT 
                sc.student_id,
                uc.course_id,
                MAX(sc.grade) AS top_grade
            FROM 
                student_course AS sc
            JOIN 
                udegree_course uc ON sc.course_id = uc.course_id
            GROUP BY 
                sc.student_id, uc.course_id
        ) student_top_grades ON sc.student_id = student_top_grades.student_id 
                             AND sc.course_id = student_top_grades.course_id 
                             AND sc.grade = student_top_grades.top_grade
    GROUP BY 
        ug.udegree_name, s.student_start_year;



