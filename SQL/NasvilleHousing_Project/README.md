# Nashville Housing Data Cleaning 

## Project Overview

This project focuses on **data cleaning and transformation** of raw housing data for Nashville, using **SQL Server**. The goal was to prepare the dataset for meaningful analysis by standardizing formats, handling missing values, splitting complex columns, and removing duplicates. Throughout the process, I applied key SQL techniques including string manipulation, conditional logic, window functions, joins, and CTEs.

SQL queries? Check them out here: [NasvilleHousing_Project folder](/SQL/NasvilleHousing_Project/NashvilleHousing_Portfolio_Project.sql).

---
# Tools I Used

For this project, I leveraged the following tools:

* **SQL Server Management Studio (SSMS):** Database management system used for querying, storing and manipulating the raw housing dataset.
* **Git and GitHub:** Version control for organizing and sharing SQL scripts and analysis.

---

## Key Objectives

* Standardize date formats for analysis.
* Populate missing property addresses.
* Split concatenated address fields into separate components (address, city, state).
* Normalize categorical fields (e.g., converting "Y"/"N" to "Yes"/"No").
* Identify and remove duplicate records.
* Drop unnecessary or redundant columns.

---

## SQL Steps Implemented

1. **Standardizing Dates**

```sql
ALTER TABLE NashvilleHousing
ADD SaleDateConverted Date;

UPDATE NashvilleHousing
SET SaleDateConverted = CONVERT(Date, SaleDate);
```

* Converted the `SaleDate` column into a proper `Date` type for consistency.

---

2. **Populating Missing Property Addresses**

```sql
UPDATE a
SET PropertyAddress = ISNULL(a.PropertyAddress, b.PropertyAddress)
FROM NashvilleHousing a
JOIN NashvilleHousing b
  ON a.ParcelID = b.ParcelID
  AND a.[UniqueID] <> b.[UniqueID]
WHERE a.PropertyAddress IS NULL;
```

* Filled missing property addresses using matching records within the dataset.

---

3. **Splitting Property and Owner Addresses**

* Extracted `Address`, `City`, and `State` into separate columns for easier analysis:

```sql
ALTER TABLE NashvilleHousing ADD PropertySplitAddress NVARCHAR(255);
ALTER TABLE NashvilleHousing ADD PropertySplitCity NVARCHAR(255);

UPDATE NashvilleHousing
SET PropertySplitAddress = SUBSTRING(PropertyAddress, 1, CHARINDEX(',', PropertyAddress)-1),
    PropertySplitCity = SUBSTRING(PropertyAddress, CHARINDEX(',', PropertyAddress)+1, LEN(PropertyAddress));

ALTER TABLE NashvilleHousing ADD OwnerSplitAddress NVARCHAR(255),
                                OwnerSplitCity NVARCHAR(255),
                                OwnerSplitState NVARCHAR(255);

UPDATE NashvilleHousing
SET OwnerSplitAddress = PARSENAME(REPLACE(OwnerAddress, ',', '.'), 3),
    OwnerSplitCity = PARSENAME(REPLACE(OwnerAddress, ',', '.'), 2),
    OwnerSplitState = PARSENAME(REPLACE(OwnerAddress, ',', '.'), 1);
```

---

4. **Normalizing Categorical Fields**

```sql
UPDATE NashvilleHousing
SET SoldAsVacant = CASE 
                     WHEN SoldAsVacant = 'Y' THEN 'Yes'
                     WHEN SoldAsVacant = 'N' THEN 'No'
                     ELSE SoldAsVacant
                   END;
```

* Converted binary `Y/N` indicators to more readable `Yes/No`.

---

5. **Removing Duplicates**

```sql
WITH RowNumCTE AS (
  SELECT *,
         ROW_NUMBER() OVER (
           PARTITION BY ParcelID, PropertyAddress, SalePrice, SaleDate, LegalReference
           ORDER BY UniqueID
         ) row_num
  FROM NashvilleHousing
)
DELETE FROM RowNumCTE
WHERE row_num > 1;
```

* Ensured each property sale record is unique by removing redundant entries.

---

6. **Dropping Unused Columns**

```sql
ALTER TABLE NashvilleHousing
DROP COLUMN OwnerAddress, TaxDistrict, PropertyAddress, SaleDate;
```

* Cleaned up unnecessary columns after processing.

---

## Outcome

* Dataset is now **clean, consistent, and analysis-ready**.
* All key fields (dates, addresses, categorical indicators) are properly formatted.
* Duplicate records removed to prevent skewed results.
* Split columns facilitate targeted analysis at property and owner levels.

---

### Closing Thoughts

This project strengthened my **SQL data cleaning skills** and reinforced the importance of **structured data preparation** for any analysis.

---


