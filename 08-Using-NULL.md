# Using NULL

In this exercise we work on the school database. The school includes many departments. Most teachers work exclusively for a single department. Some teachers have no department.

### Tables

- `teacher`: `id`, `dept`, `name`, `phone`, `mobile`
- `dept`: `id`, `name`

1. List the teachers who have NULL for their department.

        SELECT name FROM teacher WHERE dept IS NULL;

2. The `INNER JOIN` misses the teachers with no department and the departments with no teacher.

        SELECT teacher.name, dept.name
        FROM teacher INNER JOIN dept
            ON (teacher.dept=dept.id);

3. `LEFT JOIN` gets all the records from the left table, whether the JOIN condition matches or not.

        SELECT teacher.name, dept.name
        FROM teacher LEFT JOIN dept
           ON (teacher.dept=dept.id);

4. `RIGHT JOIN` gets all the records from the right table, whether the JOIN condition matches or not.

        SELECT teacher.name, dept.name
        FROM teacher RIGHT JOIN dept
           ON (teacher.dept=dept.id);

COALESCE takes any number of arguments and returns the first value that is not null.

        COALESCE(x,y,z) = x if x is not NULL
        COALESCE(x,y,z) = y if x is NULL and y is not NULL
        COALESCE(x,y,z) = z if x and y are NULL but z is not NULL
        COALESCE(x,y,z) = NULL if x and y and z are all NULL

COALESCE can be useful when you want to replace a NULL value with some other value. The following code will return the placeholder 'None', if the corresponding field value is NULL

        COALESCE(party,'None')

5. Show teacher name and mobile number or '07986 444 2266'

        SELECT name, COALESCE(mobile, '07986 444 2266') FROM teacher;

6. Print the teacher name and department name. Use the string 'None' where there is no department.

        SELECT teacher.name, COALESCE(dept.name, 'None') 
        FROM teacher LEFT JOIN dept ON teacher.dept = dept.id;

7. Show the number of teachers and the number of mobile phones.

    - Using `SUM`

        SELECT COUNT(teacher.name), 
            SUM(CASE WHEN mobile IS NULL THEN 0 ELSE 1 END) 
        FROM teacher;

    - Also using COUNT just works, because it doesn't count NULL values

        SELECT COUNT(name), COUNT(mobile) FROM teacher;

8. Use COUNT and GROUP BY dept.name to show each department and the number of staff. Use a RIGHT JOIN to ensure that the Engineering department is listed.

        SELECT dept.name, COUNT(teacher.name) 
        FROM teacher RIGHT JOIN dept ON teacher.dept = dept.id 
        GROUP BY dept.name;

9. Use CASE to show the name of each teacher followed by 'Sci' if the teacher is in dept 1 or 2 and 'Art' otherwise.

        SELECT name, 
            CASE WHEN dept = 1 OR dept = 2 THEN 'Sci'
            ELSE 'Art'
            END
        FROM teacher;

10. Use CASE to show the name of each teacher followed by 'Sci' if the teacher is in dept 1 or 2, show 'Art' if the teacher's dept is 3 and 'None' otherwise.

        SELECT name, 
            CASE WHEN dept = 1 OR dept = 2 THEN 'Sci'
                WHEN dept = 3 THEN 'Art'
            ELSE 'None'
            END
        FROM teacher;