//SQL MURDER MYSTERY SOLUTION
//https://mystery.knightlab.com/
//Context:
//A crime has taken place and the detective needs your help. The detective gave you the crime scene report, but you somehow lost it. You vaguely remember that the crime was a murder that occurred sometime on Jan.15, 2018 and that it took place in SQL City. Start by retrieving the corresponding crime scene report from the police department’s database.
//Step 1: Understanding the data structure:
//Step 2: gathering information
SELECT description FROM crime_scene_report
WHERE date = '20180115' AND type = 'murder' AND city = 'SQL City'
Output:
Security footage shows that there were 2 witnesses. The first witness lives at the last house on "Northwestern Dr". The second witness, named Annabel, lives somewhere on "Franklin Ave".
//Step 3: Identifying the witnesses:
WITH witness1 AS (
    SELECT id FROM person
    WHERE address_street_name = 'Northwestern Dr'
    ORDER BY address_number DESC LIMIT 1
), witness2 AS (
    SELECT id FROM person
    WHERE INSTR(name, 'Annabel') > 0 AND address_street_name = 'Franklin Ave'
), witnesses AS (
    SELECT *, 1 AS witness FROM witness1
    UNION
    SELECT *, 2 AS witness FROM witness2
)
SELECT witness, transcript FROM witnesses
LEFT JOIN interview ON witnesses.id = interview.person_id
 
//Step 3: Identifying the murderer:
WITH gym_checkins AS (
    SELECT person_id, name
    FROM get_fit_now_member
    LEFT JOIN get_fit_now_check_in ON get_fit_now_member.id = get_fit_now_check_in.membership_id
    WHERE membership_status = 'gold' 
      AND check_in_date = '20180109' -- Witness 2 recognized him on January the 9th
), suspects AS (
    SELECT gym_checkins.person_id, gym_checkins.name, plate_number, gender
    FROM gym_checkins
    LEFT JOIN person ON gym_checkins.person_id = person.id
    LEFT JOIN drivers_license ON person.license_id = drivers_license.id
)
SELECT * FROM suspects
WHERE INSTR(plate_number, 'H42W') > 0 AND gender = 'male'
 
//Step 4: Entering the solution
INSERT INTO solution VALUES (1, "Jeremy Bowers");

SELECT value FROM solution;

//Step 5: Continuing the investigation
SELECT transcript FROM interview WHERE person_id = 67318
 
//Step 6: Finding the true murderer
WITH red_haired_tesla_drivers AS (
    SELECT id AS license_id
    FROM drivers_license
    WHERE gender = 'female' AND hair_color = 'red'
      AND car_make = 'Tesla' AND car_model = 'Model S'
      AND height >= 64 AND height <= 68
), rich_suspects AS (
    SELECT person.id AS person_id, name, annual_income
    FROM red_haired_tesla_drivers AS rhtd
    LEFT JOIN person ON rhtd.license_id = person.license_id
    LEFT JOIN income ON person.ssn = income.ssn
), symphony_attenders AS (
    SELECT person_id, COUNT(1) AS n_checkins
    FROM facebook_event_checkin
    WHERE event_name = 'SQL Symphony Concert'
    GROUP BY person_id
    HAVING n_checkins = 3 
)
SELECT name, annual_income
	FROM rich_suspects
	INNER JOIN symphony_attenders 
		ON rich_suspects.person_id = symphony_attenders.person_id

 //Step 7: Entering the solution
INSERT INTO solution VALUES (1, "Miranda Priestly");

SELECT value FROM solution;
