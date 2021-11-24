# Nashville-Housing-Data-Cleaning
```
--- Create Temp Table

SELECT *
INTO dbo.updatedNashvilleHousing
FROM PortfolioProject..nashvilleHousing
----------------------------------------------------------
---Standardize Date Format

SELECT *
FROM dbo.updatedNashvilleHousing

SELECT SaleDate, CONVERT(Date, SaleDate)
FROM dbo.updatedNashvilleHousing

ALTER TABLE dbo.updatedNashvilleHousing
ADD SalesDataConverted Date;

UPDATE dbo.updatedNashvilleHousing
SET SalesDataConverted = CONVERT(Date, SaleDate)
----------------------------------------------------------

---Populate Propety Address

SELECT PropertyAddress
FROM dbo.updatedNashvilleHousing

SELECT a.UniqueID, a.ParcelID, a.PropertyAddress, b.UniqueID, b.ParcelID, b.PropertyAddress, ISNULL(a.PropertyAddress, b.PropertyAddress)
FROM dbo.updatedNashvilleHousing a
JOIN dbo.updatedNashvilleHousing b
	ON a.ParcelID = b.ParcelID
	AND a.UniqueID <> b.UniqueID
WHERE a.PropertyAddress IS NULL

UPDATE a
SET PropertyAddress = ISNULL(a.PropertyAddress, b.PropertyAddress)
FROM dbo.updatedNashvilleHousing a
JOIN dbo.updatedNashvilleHousing b
	ON a.ParcelID = b.ParcelID
	AND a.UniqueID <> b.UniqueID
WHERE a.PropertyAddress IS NULL
--------------------------------------------------

--- Breaking out Address into individual  Columns (Address, City, State)
---ex.1 Property Address
SELECT PropertyAddress
FROM dbo.updatedNashvilleHousing

SELECT 
SUBSTRING(PropertyAddress, 1, CHARINDEX(',', PropertyAddress) -1),
SUBSTRING(PropertyAddress, CHARINDEX(',', PropertyAddress) +1, LEN(PropertyAddress))
FROM dbo.updatedNashvilleHousing

ALTER TABLE dbo.updatedNashvilleHousing
ADD 
	PropertySplitAddress Nvarchar(255),
	PropertySplitCity Nvarchar(255);

UPDATE dbo.updatedNashvilleHousing
SET 
	PropertySplitAddress = SUBSTRING(PropertyAddress, 1, CHARINDEX(',', PropertyAddress) -1),
	PropertySplitCity = SUBSTRING(PropertyAddress, CHARINDEX(',', PropertyAddress) +1, LEN(PropertyAddress))

---ex.2 OwnerAddress
SELECT
PARSENAME(REPLACE(OwnerAddress,',','.'), 3),
PARSENAME(REPLACE(OwnerAddress,',','.'), 2),
PARSENAME(REPLACE(OwnerAddress,',','.'), 1)
FROM dbo.updatedNashvilleHousing

ALTER TABLE dbo.updatedNashvilleHousing
ADD 
	OwnerSplitAddress Nvarchar(255),
	OwnerSplitCity Nvarchar(255),
	OwnerSplitState Nvarchar(255)

UPDATE dbo.updatedNashvilleHousing
SET 
	OwnerSplitAddress = PARSENAME(REPLACE(OwnerAddress,',','.'), 3),
	OwnerSplitCity = PARSENAME(REPLACE(OwnerAddress,',','.'), 2),
	OwnerSplitState = PARSENAME(REPLACE(OwnerAddress,',','.'), 1)

-------------------------------------------------------------------------------------------------------------
--- Change Y and N to Yes and No in "Sold as Vacant" field
SELECT SoldAsVacant, COUNT(SoldAsVacant)
FROM dbo.updatedNashvilleHousing
GROUP BY SoldAsVacant

SELECT 
	SoldAsVacant,
	CASE
		WHEN SoldAsVacant = 'Y' THEN 'Yes'
		WHEN SoldAsVacant = 'N' THEN 'No'
		ELSE SoldAsVacant
		END
FROM dbo.updatedNashvilleHousing

UPDATE dbo.updatedNashvilleHousing
SET SoldAsVacant = CASE
		WHEN SoldAsVacant = 'Y' THEN 'Yes'
		WHEN SoldAsVacant = 'N' THEN 'No'
		ELSE SoldAsVacant
		END


-------------------------------------------------------------------------------------------------------------
---Remove Duplicates, standard practice to have temp tables instead of deleting data
WITH RowNumCTE AS(
SELECT *,
	ROW_NUMBER() OVER (
	PARTITION BY ParcelID,
				PropertyAddress,
				SalePrice,
				SaleDate,
				LegalReference
				ORDER BY
					UniqueID
					) row_num
FROM dbo.updatedNashvilleHousing
)

SELECT *
FROM RowNumCTE
WHERE row_num > 1
ORDER BY PropertyAddress

DELETE
FROM RowNumCTE
WHERE row_num > 1


-------------------------------------------------------------------------------------------------------------

---Delete Unused Columns

ALTER TABLE dbo.updatedNashvilleHousing
DROP COLUMN OwnerAddress, TaxDistrict, PropertyAddress, SaleDate
```
