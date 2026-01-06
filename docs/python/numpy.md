
High performance compiler: [numba](https://numba.pydata.org/)


```python
import numpy as np

# 1. Pokročilá indexace a slicing
arr = np.array([[1, 2, 3, 4],
                [5, 6, 7, 8],
                [9, 10, 11, 12]])

# Fancy indexing
indexy = np.array([0, 2])
print("Vybrané řádky:", arr[indexy])  # Vybere první a třetí řádek

# Boolean indexing workspace
podminka = arr > 5
print("Prvky větší než 5:", arr[podminka])

# 2. Broadcasting - automatické přizpůsobení dimenzí
malé_pole = np.array([1, 2, 3, 4])
print("Broadcasting:", arr + malé_pole)  # Přičte ke každému řádku

# 3. Reshape a transformace
print("Změna tvaru:", arr.reshape(2, 6))  # Změní tvar na 2x6
print("Transponované:", arr.T)  # Prohodí řádky a sloupce

# 4. Užitečné funkce pro práci s daty
data = np.array([1, 2, np.nan, 4, 5])
print("Průměr bez NaN:", np.nanmean(data))
print("Medián bez NaN:", np.nanmedian(data))

# 5. Vektorizace - rychlejší než cyklus
def vektorizovana_funkce(x):
    return np.sqrt(x) + np.sin(x)

vstup = np.linspace(0, 10, 1000)
vystup = vektorizovana_funkce(vstup)  # Aplikuje funkci na každý prvek

# 6. Práce s osami
matice = np.random.rand(3, 4)
print("Součet po sloupcích:", np.sum(matice, axis=0))
print("Průměr po řádcích:", np.mean(matice, axis=1))

# 7. Spojování polí
a = np.array([[1, 2], [3, 4]])
b = np.array([[5, 6], [7, 8]])

horizontalne = np.hstack((a, b))  # Spojí vodorovně
vertikalne = np.vstack((a, b))    # Spojí svisle
print("Horizontální spojení:\n", horizontalne)
print("Vertikální spojení:\n", vertikalne)

# 8. Pokročilé operace s maticemi
matice_a = np.array([[1, 2], [3, 4]])
matice_b = np.array([[5, 6], [7, 8]])

print("Násobení matic:", np.dot(matice_a, matice_b))
print("Determinant:", np.linalg.det(matice_a))
print("Inverzní matice:\n", np.linalg.inv(matice_a))
```
