# Data-Analyst-Portfolio
This portfolio contains Data Analyst projects for SQL and Excel. 

- yes

-- Top 5 Industries with the highest laid offs

```sql
WITH Industry_Year (industry, years, total_laid_off) AS
(
SELECT industry, YEAR(date), SUM(total_laid_off)
FROM layoffs_staging2
WHERE total_laid_off IS NOT NULL
GROUP BY industry, YEAR(date)
), Industry_Year_Rank AS
(
SELECT *, DENSE_RANK() OVER (PARTITION BY years ORDER BY total_laid_off DESC) AS ranking
FROM Industry_Year
WHERE years IS NOT NULL
)
SELECT *
FROM Industry_Year_Rank
WHERE ranking <= 5
```
