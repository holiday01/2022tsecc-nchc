# SQL 教學

## 1
```
SELECT *
FROM mimiciv_hosp.patients;
```

## 2
```
SELECT COUNT(*)
FROM mimiciv_hosp.patients;
```

## 3
```
SELECT DISTINCT(gender)
FROM mimiciv_hosp.patients;
```

## 4
```
SELECT COUNT(*)
FROM mimiciv_hosp.patients
WHERE gender = 'F';
```

## 5
```
SELECT gender, COUNT(*)
FROM mimiciv_hosp.patients
GROUP BY gender;
```

## 6
```
SELECT p.subject_id, p.dob, a.hadm_id,
    a.admittime, p.expire_flag
FROM mimiciv_hosp.admissions a
INNER JOIN mimiciv_hosp.patients p
ON p.subject_id = a.subject_id;
```

## 7
```
SELECT p.subject_id, p.dob, a.hadm_id,
    a.admittime, p.expire_flag,
    MIN (a.admittime) OVER (PARTITION BY p.subject_id) AS first_admittime
FROM mimiciv_hosp.admissions a
INNER JOIN mimiciv_hosp.patients p
ON p.subject_id = a.subject_id
ORDER BY a.hadm_id, p.subject_id;
```

## 8
```
WITH first_admission_time AS
(
  SELECT
      p.subject_id, p.dob, p.gender
      , MIN (a.admittime) AS first_admittime
      , MIN( ROUND( (cast(admittime as date) - cast(dob as date)) / 365.242,2) )
          AS first_admit_age
  FROM mimiciv_hosp.patients p
  INNER JOIN mimiciv_hosp.admissions a
  ON p.subject_id = a.subject_id
  GROUP BY p.subject_id, p.dob, p.gender
  ORDER BY p.subject_id
)
SELECT
    subject_id, dob, gender
    , first_admittime, first_admit_age
    , CASE
        -- all ages > 89 in the database were replaced with 300
        WHEN first_admit_age > 89
            then '>89'
        WHEN first_admit_age >= 14
            THEN 'adult'
        WHEN first_admit_age <= 1
            THEN 'neonate'
        ELSE 'middle'
        END AS age_group
FROM first_admission_time
ORDER BY subject_id
```

## 9 實作
```
找到labevents的測量項目 (labevents and d_labitems)，限制搜尋100筆
```


## 10 實作
```
1. icu patients的年紀，從patients和icustays的年紀
2. 區分三種情況，小於65歲 (adult)，65到89 (old)，等於91 (>89)
3. 計算三種情況的人術
```


Reference: https://mimic.mit.edu/docs/iii/tutorials/intro-to-mimic-iii/
