# Data-Analyst-Portfolio (SQL Data Cleaning)

This portfolio contains Data Analyst projects for SQL and Excel. I have imported Data which I will be cleaning. 

## Table of Contents

- [Data Sources](#DataSources)
- [Copy Data Set](#CopyDataSet)
- [Tools](#Tools)
- [Remove Duplicates](#RemoveDuplicates)
- [Standardize the Data](#StandardizetheData)
- [Solving/Removing Null Values or blank sets](#Solving/RemovingNullValuesorblanksets)
- [Remove Any Unwanted Column(s) or Row(s)](#RemoveAnyUnwantedColumn(s)orRow(s))
- [Conclusion](#Conclusion)

## Data Sources

layoffs: The primary dataset used for analyzing recent mass layoffs and discover useful insights and patterns. You can find this data at [kaggle](https://www.kaggle.com/).

## Tools

SQL Server

## Copy Data set

I will start by Copying the Data set before cleaning it so that the original data is not affected. 



```sql
CREATE TABLE layoffs_staging
LIKE layoffs;

SELECT *
FROM layoffs_staging;

INSERT layoffs_staging
SELECT *
FROM layoffs;
```

## Remove Duplicates

1.  No unique column(s) that identifies our data, so we are going to ADD a new column called row_num to enable us to check duplicates and delete all data that shows 2 or more in the column row_num

```sql
SELECT *,
ROW_NUMBER() OVER(
PARTITION BY company, location, industry, total_laid_off, 
percentage_laid_off, 'date', stage, country, funds_raised_millions) AS row_num
FROM layoffs_staging
```

![image](https://github.com/stemla/Data-Analyst-Portfolio/assets/170471393/bf397ce1-9eb7-47bd-992f-34e8401622f1)

2.  Making use of CTE to identify duplicates

```sql
WITH duplicate_cte AS
(SELECT *,
ROW_NUMBER() OVER(
PARTITION BY company, location, industry, total_laid_off, 
percentage_laid_off, 'date', stage, country, funds_raised_millions) AS row_num
FROM layoffs_staging
)

SELECT *
FROM duplicate_cte
WHERE row_num > 1;
```

![image](https://github.com/stemla/Data-Analyst-Portfolio/assets/170471393/8db6e2e4-be65-46a6-8372-9c1e37a1a860)

I have investigated few companies to make sure this is truly a duplicate.  

```sql
SELECT *
FROM layoffs_staging
WHERE company = 'Better.com'
```

3.  Deleting the Duplicates
   
-- Creating New Table calling it layoffs_staging2 (By Copy Clipboard, then Create Statement, then paste it), and add column row_num with data type int:

```sql
CREATE TABLE `layoffs_staging2` (
  `company` text,
  `location` text,
  `industry` text,
  `total_laid_off` int DEFAULT NULL,
  `percentage_laid_off` text,
  `date` text,
  `stage` text,
  `country` text,
  `funds_raised_millions` int DEFAULT NULL,
  `row_num` int
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4 COLLATE=utf8mb4_0900_ai_ci;
```

![image](https://github.com/stemla/Data-Analyst-Portfolio/assets/170471393/013a5e07-da83-48b2-bc33-b37ae4f945cb)

-- Copy data from layoffs_staging into layoffs_staging2 so that I can delete duplicates

```sql
INSERT INTO layoffs_staging2
SELECT *,
ROW_NUMBER() OVER(
PARTITION BY company, location, industry, total_laid_off, 
percentage_laid_off, 'date', stage, country, funds_raised_millions) AS row_num
FROM layoffs_staging
```

![image](https://github.com/stemla/Data-Analyst-Portfolio/assets/170471393/dbe256f5-863b-44e4-b210-82bbb452773d)

```sql
SELECT *
FROM layoffs_staging2
WHERE row_num > 1;

DELETE
FROM layoffs_staging2
WHERE row_num > 1;
```

## Standardizing Data

1.  I checked each column to see if they are any fields captured with unwanted spaces or values or symbols. I have identified data that needed to be trimmed under company column, such as company name "Included Health". 

![image](https://github.com/stemla/Data-Analyst-Portfolio/assets/170471393/cd4af69b-3a61-418a-a78a-e36d45649673)

```sql
SELECT DISTINCT (company)
FROM layoffs_staging2;

SELECT DISTINCT (company)
FROM layoffs_staging2;

SELECT DISTINCT (TRIM(company))
FROM layoffs_staging2

UPDATE layoffs_staging2
SET company = TRIM(company);
```

Checked other columns and I found an issue with Industry column. 

```sql
SELECT DISTINCT industry
FROM layoffs_staging2
```

![image](https://github.com/stemla/Data-Analyst-Portfolio/assets/170471393/8aebc742-8e3d-430c-8c9b-7b72515a8db0)

2.  It is difficult to analyze the data in the industry column, so we will need to order it. After Sorting it by ASC, we noticed that they are repeated names written in different ways, so we need to make sure that only one name per industry is recorded. Before making any changes, an investigation must be done to ensure that these are different names for one industry.

```sql
SELECT DISTINCT industry
FROM layoffs_staging2
ORDER BY 1;
```

![image](https://github.com/stemla/Data-Analyst-Portfolio/assets/170471393/777f7476-e8d6-43c7-8495-3993125ff97e)

3.  Investigate Crypto and Crypto Currency to see if they could be the same industry and the results show that they are the same but named differently. We will then Update all to be Crypto because they are the same industry. 

```sql
SELECT *
FROM layoffs_staging2
WHERE industry LIKE 'Crypto%';
```

![image](https://github.com/stemla/Data-Analyst-Portfolio/assets/170471393/be706cb0-7084-4c3a-a2ee-c78861a202dc)


```sql
UPDATE layoffs_staging2
SET industry = 'Crypto'
WHERE industry LIKE 'Crypto%';
```

4.  Checking other columns and found country column to have an issue for United State. We will update the country name to be United State by removing the “.” from the name(s) that have it as you can see in the attached picture. 

```sql
SELECT DISTINCT country
FROM layoffs_staging2
ORDER BY 1;
```

![image](https://github.com/stemla/Data-Analyst-Portfolio/assets/170471393/3c12f3b9-5656-409f-add3-c7cc7fc7c72b)

```sql
SELECT DISTINCT country, TRIM(TRAILING '.' FROM country)
FROM layoffs_staging2
ORDER BY 1;

UPDATE layoffs_staging2
SET country = TRIM(TRAILING '.' FROM country)
WHERE country LIKE 'United States%';
```

5.  We also found out that the Date column has incorrect DATA Type (text) and this needs to be corrected. We are going to change the data type from Text to DATE.

![image](https://github.com/stemla/Data-Analyst-Portfolio/assets/170471393/fc427a62-d0b1-4535-b8ca-9d9d1286ae6a)

```sql
SELECT date,
STR_TO_DATE(date, '%m/%d/%Y')
FROM layoffs_staging2;

UPDATE layoffs_staging2
SET date = STR_TO_DATE(date, '%m/%d/%Y')

ALTER TABLE layoffs_staging2
MODIFY COLUMN date DATE;
```

## Solving/Removing Null Values or blank sets

1.  Checking if we can add/remove the Null/Blanks sets. If there is enough information to populate the Null/Blank sets, we will do that but if not, we will remove the whole raw, especially if the remaining data will not be useful for our purpose of the analyses. 

```sql
SELECT *
FROM layoffs_staging2
WHERE total_laid_off IS NULL AND percentage_laid_off IS NULL;

SELECT *
FROM layoffs_staging2
WHERE industry IS NULL OR industry = ' ';
```

- Updating the industry column by populating the blank/null sets

```sql
UPDATE layoffs_staging2
SET industry = Null
WHERE company = '';

SELECT t1.industry, t2.industry
FROM layoffs_staging2 t1
JOIN layoffs_staging2 t2
	ON t1.company = t2.company
    AND t1.location = t2.location
WHERE t1.industry IS NULL;

UPDATE layoffs_staging2 t1
JOIN layoffs_staging2 t2
	ON t1.company = t2.company
SET t1.industry = t2.industry
WHERE t1.industry IS NULL 
	AND t2.industry IS NOT NULL;
```

2.  We are going to DELETE the remaining data set with NULLs on the total_laid_off  AND percentage_laid_off columns because these two columns should both or either one contain data for the data to be usable in our analyses.

```sql
SELECT *
FROM layoffs_staging2
WHERE total_laid_off IS NULL AND percentage_laid_off IS NULL;

DELETE
FROM layoffs_staging2
WHERE total_laid_off IS NULL AND percentage_laid_off IS NULL;
```

## Remove Any Unwanted Column(s) or Row(s)

-- We have now deleted the Null/Blank data sets, so we will now Remove the row_num Column because we don't need it anymore

![image](https://github.com/stemla/Data-Analyst-Portfolio/assets/170471393/447c92bf-1c31-410b-92db-9bb2fd372578)

```sql
SELECT *
FROM layoffs_staging2;

ALTER TABLE layoffs_staging2
DROP COLUMN row_num;
```

## Conclusion

We have now successfully cleaned our Data and it is now ready to be used. I do acknowledge that different methods could have been used to achieve the same results. In the next project, we are going to use this data to do Exploratory Data Analysis. 












