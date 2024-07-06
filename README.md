# Data Cleaning - Nashville_Housing_Data_Analysis

Data cleaning is the process of detecting and correcting errors or inconsistencies in a dataset to ensure accuracy, completeness, and reliability for analysis. 

```

SELECT * FROM `data-cleaning-project-427815.nashville_housing_data.hashville_housing_data`

```

###### Using CTE - Common Table Expression

```
-- Standardizing Date 

WITH
DATE_CONVERT AS (
SELECT *,
parse_date('%B %d, %Y', SaleDate) AS SaleDate_New
FROM `data-cleaning-project-427815.nashville_housing_data.hashville_housing_data`
)

-- 

,NULL_ADDRESS AS (
 SELECT distinct  A.*, IFNULL(A.PropertyAddress,B.PropertyAddress) AS Updated_Address
 FROM DATE_CONVERT AS A
 left JOIN DATE_CONVERT AS B
 ON A.ParcelID = B.ParcelID
 AND A.UniqueID_ <> B.UniqueID_
)

,PROPERTY_ADDRESS_SPLIT AS (
 SELECT *,
   SPLIT(PropertyAddress, ",")[SAFE_OFFSET(ARRAY_LENGTH(SPLIT(PropertyAddress, ",")) - 2)] AS PROPERTY_ADDR,
   SPLIT(PropertyAddress, ",")[SAFE_OFFSET(ARRAY_LENGTH(SPLIT(PropertyAddress, ",")) - 1)] AS PROPERTY_CITY
 FROM NULL_ADDRESS
)
,OWNER_ADDRESS_SPLIT AS (
 SELECT *,
   SPLIT(OwnerAddress, ",")[SAFE_OFFSET(ARRAY_LENGTH(SPLIT(OwnerAddress, ",")) - 3)] AS OWNER_ADDR,
   SPLIT(OwnerAddress, ",")[SAFE_OFFSET(ARRAY_LENGTH(SPLIT(OwnerAddress, ",")) - 2)] AS OWNER_CITY,
   SPLIT(OwnerAddress, ",")[SAFE_OFFSET(ARRAY_LENGTH(SPLIT(OwnerAddress, ",")) - 1)] AS OWNER_STATE,
 FROM PROPERTY_ADDRESS_SPLIT
)
,CHANGE_SOLD_AS_VACANT AS (
 SELECT * ,
   CASE
     WHEN SoldAsVacant = false
       THEN 'N'
     WHEN SoldAsVacant = true
       THEN 'Y'
   END AS SoldAsVacantUpdated
 FROM OWNER_ADDRESS_SPLIT
)
,REMOVE_DUPLICATES AS (
SELECT * , ROW_NUMBER() OVER (PARTITION BY ParcelId, PropertyAddress, SalePrice, SaleDate, LegalReference ORDER BY UniqueID_) AS rn
FROM CHANGE_SOLD_AS_VACANT
)
,REMOVE_DUPLICATES_FINAL AS (
 SELECT *
 FROM REMOVE_DUPLICATES
 WHERE 1=1
   AND rn = 1
)
,EXCLUDE_UNUSED_COLUMNS AS (
 SELECT *
 EXCEPT(PropertyAddress, SaleDate, SoldAsVacant, OwnerAddress, Updated_Address)
 FROM REMOVE_DUPLICATES_FINAL
)
--SELECT * FROM DATE_CONVERT
--SELECT * FROM NULL_ADDRESS
--SELECT * FROM PROPERTY_ADDRESS_SPLIT
--SELECT * FROM OWNER_ADDRESS_SPLIT
--SELECT * FROM CHANGE_SOLD_AS_VACANT
--SELECT * FROM REMOVE_DUPLICATES
--SELECT * FROM REMOVE_DUPLICATES_FINAL
SELECT * FROM EXCLUDE_UNUSED_COLUMNS
;

```


Standardize Date Format
In BigQuery, you can convert the date "April 9, 2013" into a date format using the `PARSE_DATE` function. Here's how you can do it:

```sql
SELECT PARSE_DATE('%B %d, %Y', 'April 9, 2013') AS date_value
```

Explanation:
- `PARSE_DATE('%B %d, %Y', 'April 9, 2013')`: This function takes two parameters:
  - The first parameter `'%B %d, %Y'` specifies the format of the date string you want to parse. Here:
    - `%B` stands for the full month name (April),
    - `%d` stands for the day of the month (9),
    - `%Y` stands for the four-digit year (2013).
  - The second parameter is the actual date string `'April 9, 2013'` that you want to convert.

This query will return the date value in a format that BigQuery recognizes as a date type, allowing you to use it in further SQL operations or comparisons as needed.



Populate Address



Breaking out address field into Address, City, State


Changing Y and N into TRUE and FALSE for “Sold as vacant” field




Remove Duplicates


Delete unused columns

