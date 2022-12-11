# SQL 教學

## 1 查詢patients
```
SELECT *
FROM mimiciv_hosp.patients;
```

## 2 計算patients數量
```
SELECT COUNT(*)
FROM mimiciv_hosp.patients;
```

## 3 區分patients的性別
```
SELECT DISTINCT(gender)
FROM mimiciv_hosp.patients;
```

## 4 計算patients性別是F的人數
```
SELECT COUNT(*)
FROM mimiciv_hosp.patients
WHERE gender = 'F';
```

## 5 計算patients的性別人數
```
SELECT gender, COUNT(*)
FROM mimiciv_hosp.patients
GROUP BY gender;
```

## 6 結合patients和admissions兩張表，根據subject_id。從兩張表取出五個欄位
```
SELECT p.subject_id, p.dod, a.hadm_id,
    a.admittime, a.hospital_expire_flag
FROM mimiciv_hosp.admissions a
INNER JOIN mimiciv_hosp.patients p
ON p.subject_id = a.subject_id;
```

## 6.1 補充，如果要不重複出現6的寫法
```
SELECT p.subject_id, p.dod, a.hadm_id,
    a.admittime, a.hospital_expire_flag
FROM mimiciv_hosp.admissions a
INNER JOIN mimiciv_hosp.patients p
ON p.subject_id = a.subject_id
GROUP BY p.subject_id, a.hadm_id, p.dod, a.admittime, a.hospital_expire_flag
```

## 7 選擇每位患者最初進入醫院的紀錄
```
SELECT p.subject_id, 
    MIN (a.admittime) AS first_admittime
FROM mimiciv_hosp.admissions a
INNER JOIN mimiciv_hosp.patients p
ON p.subject_id = a.subject_id
GROUP BY p.subject_id
ORDER BY p.subject_id;
```

## 8 患者如果是傳染性腹瀉進入醫院，存活與死亡人數
```
WITH Infectious AS(
	SELECT i.*, d.*
	FROM mimiciv_hosp.diagnoses_icd i
	INNER JOIN mimiciv_hosp.d_icd_diagnoses d
	ON i.icd_code = d.icd_code
	WHERE d.long_title='Infectious diarrhea'
)
SELECT a.hospital_expire_flag, COUNT(*)
FROM Infectious s
INNER JOIN mimiciv_hosp.admissions a
ON a.hadm_id = s.hadm_id
GROUP BY a.hospital_expire_flag
```

## 9 實作
```
找到labevents的測量項目 (labevents and d_labitems)，限制搜尋100筆
```

## 實作9答案
```
SELECT l.*, d.label
FROM mimiciv_hosp.labevents l
INNER JOIN mimiciv_hosp.d_labitems d
ON l.itemid = d.itemid
LIMIT 100
```

## 10 實作
```
1. icu patients的年紀，從patients和icustays的年紀
2. 區分三種情況，小於65歲 (adult)，65到89 (old)，等於91 (>89)
3. 計算三種情況的人術
```
## 實作10答案
```
WITH EXAMPLE AS
	(SELECT IE.SUBJECT_ID,
			IE.HADM_ID,
			PAT.ANCHOR_AGE AS AGE,
			CASE
							WHEN PAT.ANCHOR_AGE < 65 THEN 'adult'
							WHEN PAT.ANCHOR_AGE = 91 THEN '>89'
							ELSE 'old'
			END AS ICUSTAY_AGE_GROUP
		FROM MIMICIV_ICU.icustays IE
		INNER JOIN MIMICIV_HOSP.PATIENTS PAT ON IE.SUBJECT_ID = PAT.SUBJECT_ID)
SELECT ICUSTAY_AGE_GROUP, COUNT(*)
FROM EXAMPLE
GROUP BY ICUSTAY_AGE_GROUP
```


Reference: https://mimic.mit.edu/docs/iii/tutorials/intro-to-mimic-iii/
