# Data Cleaning - Nashville_Housing_Data_Analysis

Data cleaning is the process of detecting and correcting errors or inconsistencies in a dataset to ensure accuracy, completeness, and reliability for analysis. 

**Dataset Summary**

The dataset is related to housing data in Nashville, containing details about various properties. Each row represents a single property, and the dataset includes a variety of attributes related to the properties.

### Columns and Descriptions


**UniqueID**: A unique identifier for each property.

**ParcelID**: The parcel identification number.

**LandUse**: The type of land use (e.g., "SINGLE FAMILY").

**PropertyAddress**: The address of the property.

**SaleDate**: The date the property was sold.

**SalePrice**: The sale price of the property.

**LegalReference**: The legal reference associated with the sale.

**SoldAsVacant**: Indicates if the property was sold as vacant ("Yes" or "No").

**OwnerName**: The name of the property owner.

**OwnerAddress**: The address of the property owner.

**Acreage**: The size of the property in acres.

**TaxDistrict**: The tax district in which the property is located.

**LandValue**: The assessed land value of the property.

**BuildingValue**: The assessed building value of the property.

**TotalValue**: The total assessed value of the property.

**YearBuilt**: The year the property was built.

**Bedrooms**: The number of bedrooms in the property.

**FullBath**: The number of full bathrooms in the property.

**HalfBath**: The number of half bathrooms in the property.




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


### Standardize Date Format

#### CTE - DATE_CONVERT

This query transforms the SaleDate field from a specific format ('%B %d, %Y') into a standardized date format (SaleDate_New) for analysis. It reads data from the hashville_housing_data table within the nashville_housing_data dataset in data-cleaning-project-427815. This transformation ensures consistency and usability of SaleDate data by parsing it into a structured date format (Month Day, Year).

 <img width="1084" alt="Screenshot 2024-07-07 at 4 07 29 PM" src="https://github.com/jeevareha/Data_Cleaning/assets/32441508/bb3ea2df-00a9-4e9f-b1cb-6ad21a4f941c">

### Populate Address

#### CTE - NULL_ADDRESS

This query generates unique rows (DISTINCT A.*) from the DATE_CONVERT dataset (A) while ensuring each row's PropertyAddress is filled (IFNULL(A.PropertyAddress, B.PropertyAddress)). It achieves this by left joining DATE_CONVERT with itself (B) on matching ParcelID values, excluding rows where A.UniqueID_ matches B.UniqueID_. This comparison ensures each row is distinct while ensuring PropertyAddress is filled where possible from other rows sharing the same ParcelID.

<img width="1081" alt="Screenshot 2024-07-07 at 4 22 08 PM" src="https://github.com/jeevareha/Data_Cleaning/assets/32441508/8e409f76-38b7-474f-a262-c374896d705e">

### Breaking out address field into Address, City, State

#### CTE - PROPERTY_ADDRESS_SPLIT

This query splits the PropertyAddress field into its components (PROPERTY_ADDR and PROPERTY_CITY) using the SPLIT function. It extracts the second last part (PROPERTY_ADDR) and the last part (PROPERTY_CITY) of the address after splitting by commas. This transformation is performed on the dataset NULL_ADDRESS, aiming to parse and categorize address details such as street names (PROPERTY_ADDR) and cities (PROPERTY_CITY) for further analysis or categorization.

<img width="1078" alt="Screenshot 2024-07-07 at 4 25 38 PM" src="https://github.com/jeevareha/Data_Cleaning/assets/32441508/8a69b77a-a4b5-4896-ab82-f9359be9312e">


#### CTE - OWNER_ADDRESS_SPLIT
This query splits the OwnerAddress field into its components (OWNER_ADDR, OWNER_CITY, OWNER_STATE) using the SPLIT function. It extracts specific parts of the address after splitting by commas: the third last part (OWNER_ADDR), the second last part (OWNER_CITY), and the last part (OWNER_STATE). This transformation is applied on the dataset PROPERTY_ADDRESS_SPLIT, aiming to parse and categorize owner address details such as street addresses, cities, and states for further analysis or processing.

<img width="1046" alt="Screenshot 2024-07-07 at 4 33 02 PM" src="https://github.com/jeevareha/Data_Cleaning/assets/32441508/fb19c66f-6433-482d-ab8b-cbeda79a7b8b">

### Changing TRUE and FALSE into Y and N for “Sold as vacant” field

#### CTE - CHANGE_SOLD_AS_VACANT
This query adds a new column SoldAsVacantUpdated to the dataset OWNER_ADDRESS_SPLIT, categorizing each row based on the SoldAsVacant field. Rows where SoldAsVacant is false are marked as 'N', and rows where SoldAsVacant is true are marked as 'Y'. This transformation simplifies the representation of vacant property sales status for analysis or reporting purposes.

<img width="1068" alt="Screenshot 2024-07-07 at 4 39 10 PM" src="https://github.com/jeevareha/Data_Cleaning/assets/32441508/0e863846-eccf-48a7-9c71-492261bfd9d1">


### Remove Duplicates

#### CTE - REMOVE_DUPLICATES, REMOVE_DUPLICATES_FINAL

This SQL sequence removes duplicate records based on specified criteria:

REMOVE_DUPLICATES CTE:

Assigns a row number (rn) to each record within partitions defined by ParcelId, PropertyAddress, SalePrice, SaleDate, and LegalReference, ordered by UniqueID_.
Ensures each combination of these fields is uniquely identified by rn.

REMOVE_DUPLICATES_FINAL CTE:

Filters the results from REMOVE_DUPLICATES to retain only rows where rn equals 1.
Ensures that only the first occurrence of each unique combination defined by the partitioning criteria remains in the final dataset.
These steps ensure data integrity by eliminating duplicate entries based on specified key fields, providing a clean dataset for further analysis or processing.

<img width="1082" alt="Screenshot 2024-07-07 at 4 46 13 PM" src="https://github.com/jeevareha/Data_Cleaning/assets/32441508/a33b3e0b-f3ca-4f22-be83-98a8c309020b">

### Delete unused columns

#### CTE - EXCLUDE_UNUSED_COLUMNS
This query selects all columns from the REMOVE_DUPLICATES_FINAL dataset but excludes specific columns (PropertyAddress, SaleDate, SoldAsVacant, OwnerAddress, Updated_Address). It aims to focus on other attributes while omitting these specific fields, likely for a more streamlined view or analysis of the cleaned dataset after handling duplicates and irrelevant columns.

<img width="1088" alt="Screenshot 2024-07-07 at 5 02 48 PM" src="https://github.com/jeevareha/Data_Cleaning/assets/32441508/db69f07c-036c-454c-b7d2-6ee9c5eea57f">


