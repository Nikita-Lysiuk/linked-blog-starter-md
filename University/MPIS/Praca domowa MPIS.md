Do rozwiązania mojej pracy domowej użyłem pythona i numpy, pandas, scipy. 

Zadanie 1:

![[Pasted image 20250201212232.png]]
```
# buduwanie szeregu przedziałowego
data = np.array([45, 39, 41, 58, 39, 50, 55, 63, 49, 31, 60, 69, 42, 55, 38, 68, 53, 75, 64, 58, 61, 47, 31, 30, 41, 52, 37, 49, 39, 57, 50, 62, 74, 58, 45, 54, 55, 46, 51, 37, 63, 75, 45, 61, 48, 43, 62, 53, 50, 74, 64, 49, 53, 35, 49, 33, 55, 60, 50, 39, 46, 67, 53, 48, 65, 59, 60, 44, 51, 43, 50, 64, 67, 51, 70, 62,
68, 60, 58, 49, 43, 65, 64, 38, 36, 42, 44, 75, 45, 62])

sorted_data = np.sort(data)
  
# liczba przedziałów
# Prosty sposób na obliczenie liczby przedziałów
# (różnica między maksymalną a minimalną wartością podzielona przez pierwiastek z liczby elementów)
h = np.ceil((np.max(sorted_data) - np.min(sorted_data))/np.sqrt(len(sorted_data)))
```

```
# budowanie przedziałów
# np.arange - zwraca równomiernie rozmieszczone wartości w przedziale
bins = np.arange(np.min(sorted_data), np.max(sorted_data) + h, h)

# obliczanie częstości dla szeregu przedziałowego
# pd.cut - dzieli dane na przedziały i zlicza ilość elementów w każdym z przedziałów
df = pd.cut(sorted_data, bins=bins, right=True, include_lowest=True).value_counts().sort_index().reset_index()

# wykorzystam DataFrame do przechowywania danych w formie tabeli
# szeregi przedziałowe i częstości
df.columns = ['Interval', 'Frequency']
```

```
def mean(df):
    # tworzę nową kolumnę w DataFrame, która będzie przechowywać środek przedziału
    # wykorzystam lambda do obliczenia środka. Bierzę lewą i prawą granicę przedzialu i
    # obliczam środek przedziału.
    df['Midpoint'] = df['Interval']
	    .apply(lambda x: (x.left+x.right) / 2)
		.astype(float)

    # tworzę nową kolumnę w DataFrame, która będzie przechowywać iloczyn środka przedziału i częstości
    df['(xi * ni)'] = df['Midpoint'] * df['Frequency']
    # obliczanie średniej dla szeregu przedziałowego
    # sumuję iloczyny środka przedziału i częstości i dzielę przez sumę częstości
    return df["(xi * ni)"].sum() / df["Frequency"].sum()
```

$$
\bar{x} = \frac{\sum (x_i \cdot n_i)}{N}
$$
---

```
def deviation(df, mean_grouped):
    # tworzę nową kolumnę w DataFrame, która będzie pomócnicza do obliczenia           odchylenia standardowego
    df['(xi - X)^2*ni'] = df['Frequency'] * (df['Midpoint'] - mean_grouped) ** 2

    # obliczanie odchylenia standardowego dla szeregu przedziałowego
    variance_grouped = df['(xi - X)^2*ni'].sum() / df['Frequency'].sum()
    
    # zwracam pierwiastek z wariancji - odchylenie standardowe
    return np.sqrt(variance_grouped)
```
$$
\sigma = \sqrt{\frac{\sum_{i=1}^{k} (x_i - \bar{X})^2 \cdot n_i}{N}}
$$
---
```
def median(df, h):
    # N - liczba elementów
    N = df['Frequency'].sum()

    # indeks szeregu przedziałowego, w którym znajduje się mediana
    # df[df['Accumulated'] >= N / 2] - zwraca wiersz, w którym wartość skumulowana jest większa lub równa N / 2
    median_class_index = df[df['Accumulated'] >= N / 2].index[0]

    # Xm - lewa granica przedziału, w którym znajduje się mediana
    Xm = df.loc[median_class_index, 'Interval'].left
    
    # F = Fm-1 - wartość skumulowana dla poprzedniego przedziału
    F = df.loc[median_class_index - 1, 'Accumulated'] 
    if median_class_index > 0 else 0

    # Nm - częstość dla przedziału, w którym znajduje się mediana
    Nm = df.loc[median_class_index, 'Frequency']
    
    # zwracam medianę
    return Xm + ((N / 2 - F)) / Nm * h
```
$$
Me=X_m + {\frac{\frac{n}{2}-f_{m-1}}{n_m}h}
$$

---
```
def Mo(df, h):
    # indeks przedziału, w którym znajduje się moda
    # idxmax - zwraca indeks maksymalnej wartości w kolumnie
    mode_index = df['Frequency'].idxmax()

    # Xm - lewa granica przedziału, w którym znajduje się moda
    Xm = df.loc[mode_index, 'Interval'].left

    # Nm - częstość dla przedziału, w którym znajduje się moda
    Nm = df.loc[mode_index, 'Frequency']

    # Nml = Nm-1, Nmr = Nm+1
    # Nml - częstość dla przedziału, który jest przed przedziałem, w którym znajduje się moda
    # Nmr - częstość dla przedziału, który jest za przedziałem, w którym znajduje się moda
    Nml = df.loc[mode_index - 1, 'Frequency'] if mode_index > 0 else 0
    Nmr = df.loc[mode_index + 1, 'Frequency'] if mode_index < len(df) - 1 else 0

    # zwracam modę
    return Xm + ((Nm - Nml) / ((Nm - Nml) + (Nm - Nmr))) * h
```

$$
Mo=X_m+{\frac{n_m-n_{m-1}}{(n_m-n_{m-1})+(n_m-n_{m+1})}}h
$$

---





---
Zadanie 2:
![[Pasted image 20250201225116.png]]

```
    # Wczytanie danych
    # Zwykła funkcja do wczytywania danych z pliku CSV do DataFrame
    data = pd.read_csv("fosforany.csv")

  

    # Obliczenie średniej za pomocą metody mean z biblioteki pandas
    mean = data['Phosphate concentration'].mean()
```
$$
\bar{X}=\frac{1}{n}\sum_{i=1}^nx_i
$$
---
```
    # Obliczenie standardowego błędu średniej za pomocą metody sem z biblioteki scipy.stats
	# dzielimy przez pierwiastek z długości danych - 1, ponieważ to jest próba.
    sem = stats.sem(data['Phosphate concentration'])
```
$$
S^2=\frac{1}{n}\sum_{i=1}^{n}(x_i-\bar{X})^2n_i
$$
$$
\sigma=\sqrt{S^2}
$$
$$
Sem=\frac{\sigma}{\sqrt{n-1}}
$$
---
```
    # Alpha = 0.05 -> 95% poziom ufności. 1 - alpha = poziom ufności
    confidence = 0.95
    
    # Stopnie swobody
    df = len(data) - 1

    # Obliczenie przedziału ufności za pomocą metody interval z biblioteki scipy.stats
    # confidence - poziom ufności
    # df - stopnie swobody
    # loc - średnia
    # scale - odchylenie standardowe
    left, right = stats.t.interval(confidence, df, loc=mean, scale=sem)
```
$$
\left( \bar{X} - t_{1-\alpha/2, df} \cdot \frac{S}{\sqrt{n}}<\mu<\bar{X} + t_{1-\alpha/2, df} \cdot \frac{S}{\sqrt{n}} \right)
$$
W tym zadanie nie znam odchylenia standardowego i n > 30, więc teoretycznie powinienem użyć trzeciego modelu, ale ponieważ jest to wciąż próbka, użyłem rozkładu t-studenta.

---



---
Zadanie 3:
Tutaj jest łatwiej, nie znam odchylenia standardowego i n <= 30, więc używam modelu 2.
![[Pasted image 20250201225416.png]]

```
def zad3():
    # wczytanie danych
    data = pd.read_csv("temperatura_wody.csv")

    alpha = 0.01
    hypothesized_mean = 7

    # funkcja ttest_1samp zwraca t_statistic oraz p_value
    # t_statistic - wartość statystyki t czyli różnica między średnią próby a średnią populacji podzielona przez błąd standardowy średniej próby
    # p_value - wartość p dla której hipoteza zerowa jest prawdziwa

    t_stat, p_value = stats.ttest_1samp(data['Temperature'], hypothesized_mean)

    print(f"t_stat: {t_stat:.4f}")
    print(f"p_value: {p_value:.4f}")
    if p_value < alpha:

        print("Odrzucamy hipotezę zerową")

    else:
        print("Nie ma podstaw do odrzucenia hipotezy zerowej (H0) na rzecz hipotezy alternatywnej (H1) - średnia temperatura wody jest równa 7 stopniom Celsjusza")
```

### Hipotezy: 
- Hipoteza zerowa $( H_0: \mu = 7 )$ (średnia temperatura wynosi 7°C), 
- Hipoteza alternatywna $( H_1: \mu \neq 7 )$ (średnia temperatura różni się od 7°C).

### $t = \frac{\bar{X} - \mu_0}{\frac{S}{\sqrt{n}}}$

Gdzie: 
- $\bar{X}$ to średnia próbki, 
- $\mu_0$ to hipotezowana średnia (w tym przypadku 7°C),
- $S$ to odchylenie standardowe próbki, 
- $n$ to liczba elementów w próbie.

#### Obliczanie p-value: 
p-value obliczamy jako prawdopodobieństwo uzyskania statystyki t większej lub równej niż wartość obliczona:
$p = P(T \geq |t_{\text{stat}}|)$

Gdzie: 
- $T$ to rozkład t z $n-1$ stopniami swobody, 
- $|t_{\text{stat}}|$ to absolutna wartość obliczonej statystyki t.

#### Decyzja: 
Na podstawie p-value podejmujemy decyzję: 
- Jeśli  $p < \alpha$, odrzucamy hipotezę zerową $H_0$. 
- Jeśli  $p \geq \alpha$, nie ma podstaw do odrzucenia hipotezy zerowej. 

W tym zadaniu $\alpha = 0.01$ (poziom istotności 1%).