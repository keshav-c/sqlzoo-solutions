# Numeric Examples

These examples use the NSS (National Student Survey) data. The survey is presented to thousands of students graduating in the UK Higher education system. For each of the 22 questions asked, the students can respond with `STRONGLY DISAGREE`, `DISAGREE`, `NEUTRAL`, `AGREE` or `STRONGLY AGREE`. The table `nss` has one row per institution, subject and question. The values in these columns represent percentages of the total students who responded with that answer.

Field | Type
-----|-----
ukprn | varchar(8)
institution | varchar(100)
subject | varchar(60)
level | varchar(50)
question | varchar(10)
A_STRONGLY_DISAGREE | int(11)
A_DISAGREE | int(11)
A_NEUTRAL | int(11)
A_AGREE | int(11)
A_STRONGLY_AGREE | int(11)
A_NA | int(11)
CI_MIN | int(11)
score | int(11)
CI_MAX | int(11)
response | int(11)
sample | int(11)
aggregate | char(1)

1. The example shows the number who responded for:

    - question 1
    - at 'Edinburgh Napier University'
    - studying '(8) Computer Science'
    
    Show the the percentage who `STRONGLY AGREE`

            SELECT A_STRONGLY_AGREE FROM nss
            WHERE question='Q01'
            AND institution='Edinburgh Napier University'
            AND subject='(8) Computer Science';

2. Show the institution and subject where the score is at least 100 for question 15.

        SELECT institution, subject FROM nss
        WHERE question='Q15'
        AND score >= 100;

3. Show the institution and score where the score for '(8) Computer Science' is less than 50 for question 'Q15'.

        SELECT institution,score
        FROM nss
        WHERE question='Q15'
            AND score < 50
            AND subject='(8) Computer Science';

4. Show the subject and total number of students who responded to question 22 for each of the subjects `'(8) Computer Science'` and `'(H) Creative Arts and Design'`.


        SELECT subject, SUM(response)
        FROM nss
        WHERE question='Q22'
            AND subject IN ('(8) Computer Science', '(H) Creative Arts and Design')
        GROUP BY subject;

5. Show the subject and total number of students who A_STRONGLY_AGREE to question 22 for each of the subjects `'(8) Computer Science'` and `'(H) Creative Arts and Design'`.

        SELECT subject, SUM(response * A_STRONGLY_AGREE / 100)
        FROM nss
        WHERE question='Q22'
        AND subject IN ('(8) Computer Science', 
            '(H) Creative Arts and Design')
        GROUP BY subject;

6. Show the percentage of students who A_STRONGLY_AGREE to question 22 for the subject `'(8) Computer Science'` show the same figure for the subject `'(H) Creative Arts and Design'`.

        SELECT subject, 
            ROUND(
                SUM(response * A_STRONGLY_AGREE / 100) / 
                SUM(response) * 100
            ) AS A_STRONGLY_AGREE_perc
        FROM nss
        WHERE question='Q22'
        AND subject IN ('(8) Computer Science', 
            '(H) Creative Arts and Design')
        GROUP BY subject;

7. Show the average scores for question 'Q22' for each institution that include 'Manchester' in the name.

        SELECT institution, 
            ROUND(
                SUM(response * score / 100) / 
                SUM(response) * 100
            ) AS score_perc
        FROM nss
        WHERE question='Q22'
        AND (institution LIKE '%Manchester%')
        GROUP BY institution
        ORDER BY institution;

8. Show the institution, the total sample size and the number of computing students for institutions in Manchester for 'Q01'.

        SELECT institution, 
            SUM(sample) AS Sample_Size, 
            SUM(CASE WHEN subject = '(8) Computer Science' 
                    THEN sample 
                ELSE 0 
                END
            ) AS Computing_Students
        FROM nss
        WHERE institution LIKE '%Manchester%'
        AND question='Q01'
        GROUP BY institution;
