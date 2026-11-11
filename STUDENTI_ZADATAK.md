# Studentska praksa — Analiza podataka online maloprodaje

**Trajanje:** 7 radnih dana (moguće skratiti na 5 — vidi *Skraćena verzija* na kraju)
**Alat:** Python (pandas, matplotlib, scikit-learn)
**Podaci:** `podaci.xlsx` — oko 1.000.000 transakcija britanskog online prodavca
poklona i dekoracije, period **01.12.2009 – 09.12.2011.**

---

> **Zapeli ste?** Uz ovaj zadatak ide i **[STUDENT_VODIC.md](STUDENT_VODIC.md)** —
> detaljno uputstvo korak po korak, sa kodom, očekivanim brojevima posle svake
> operacije i slikama rezultata.
>
> Ovaj dokument kaže **šta** treba uraditi. Vodič objašnjava **kako**.
> Prvo pokušajte sami — vodič je tu kad zaglavite, ne umesto razmišljanja.

---

## Šta je cilj

Da prođete **ceo put jedne analize podataka**: od sirovog Excel fajla koji niko
nije pogledao, do izveštaja koji može da pročita čovek koji ne zna Python.

Na kraju prakse treba da umete da odgovorite na pitanja:

- Koliko su naši podaci uopšte pouzdani i šta smo morali da izbacimo?
- Ko su naši kupci i po čemu se razlikuju?
- Koji kupci nose prihod, a koji nas koštaju?
- Da li možemo da predvidimo ko će se vratiti da kupi?
- Šta konkretno preporučujemo firmi da uradi?

> **Najvažnije pravilo cele prakse:**
> Graf bez zaključka ne vredi ništa. Uz **svaki** grafikon ide bar jedna rečenica
> šta se na njemu vidi i šta to znači za posao. Ako ne umete da je napišete,
> taj graf vam ne treba.

---

## Šta predajete

| # | Isporuka | Rok |
|---|---|---|
| 1 | Jupyter notebook ili `.py` skripta za svaki dan | na kraju svakog dana |
| 2 | Folder `grafovi/` sa svim slikama | kontinuirano |
| 3 | **Popunjen `odgovori.csv`** | kontinuirano, konačno dan 7 |
| 4 | Finalni izveštaj (2–4 strane, Markdown ili PDF) | dan 7 |
| 5 | Kratka prezentacija (5–8 slajdova, 10 minuta) | dan 7 |

Sve ide u git repozitorijum. Commit-ujte na kraju svakog dana, sa smislenom
porukom (`dan3: RFM analiza i segmenti`, a ne `update`).

### O fajlu `odgovori.csv`

Kopirajte `sablon_odgovori.csv`, nazovite ga `odgovori.csv` i **na kraju svakog
dana upišite brojeve koje ste dobili**. Izgleda ovako:

```csv
kljuc,vrednost
dan1_redova_posle_ciscenja,776579
dan1_broj_kupaca,5852
...
```

Mentor ga proverava automatski, jednom komandom. Zato:

- **upišite broj koji ste STVARNO dobili**, ne onaj za koji mislite da se očekuje
- ako nešto još niste uradili — ostavite prazno ili upišite `?`
- ne menjajte kolonu `kljuc` i ne brišite redove

> Ovo nije test iz prepisivanja. Ako je vaš broj drugačiji, provera vam kaže
> **šta je verovatno pošlo naopako** — to je brža povratna informacija nego da
> čekate mentora.

---

## Priprema okruženja (uradite pre dana 1)

```bash
python -m venv .venv
.venv\Scripts\activate          # Windows
# source .venv/bin/activate     # Linux / macOS

pip install pandas numpy openpyxl pyarrow matplotlib scikit-learn jupyterlab tabulate
```

Provera da sve radi:

```python
import pandas as pd
df = pd.read_excel("podaci.xlsx", sheet_name=None)
print(list(df.keys()))
# Očekivano: ['Year 2009-2010', 'Year 2010-2011']
```

> **Savet koji vam štedi sate:** čitanje ovog Excel fajla traje **1.5–2 minuta**.
> Ne čitajte ga iznova u svakoj ćeliji. Pročitajte jednom, pa snimite u brzi
> format i dalje radite sa njim:
> ```python
> df.to_parquet("podaci_sirovo.parquet")   # snimanje
> df = pd.read_parquet("podaci_sirovo.parquet")   # učitavanje, ispod 2 sekunde
> ```

---

## Kako su zadaci napisani

Svaki dan ima:

- **Zadatke** — šta konkretno da uradite
- **✅ Kontrolne tačke** — brojeve koje treba da dobijete. Ako vaš broj bitno
  odstupa, negde ste pogrešili — vratite se i proverite.
- **❓ Pitanja** — na njih odgovarate rečima, u notebook-u. Ovo je deo koji se
  najviše ocenjuje.
- **⚠️ Zamke** — mesta gde se najčešće greši.

Kontrolne tačke su tu da vas **oslobode nesigurnosti**, ne da vas ograniče.
Ako dobijete drugi broj a možete da objasnite zašto (drugačiji redosled čišćenja,
npr.) — to je potpuno u redu i treba da napišete zašto.

---

# DAN 1 — Učitavanje i čišćenje podataka

**Cilj:** doći do skupa podataka kojem možete da verujete, i znati tačno šta ste
izbacili i zašto.

## Zadaci

### 1.1 Učitajte oba lista i spojite ih

Fajl ima **dva lista**: `Year 2009-2010` i `Year 2010-2011`. Treba vam jedan
DataFrame sa svim podacima.

```python
listovi = pd.read_excel("podaci.xlsx", sheet_name=None)
```

### 1.2 Upoznajte se sa podacima

Pozovite `.info()`, `.describe()`, `.head()`, `.isna().sum()`, `.nunique()`.

❓ **Pitanja:**
- Koliko redova i kolona ima skup?
- Koje kolone imaju nedostajuće vrednosti i koliko?
- Pogledajte `min` za `Quantity` i `Price` u `.describe()`. Šta vidite i kako
  objašnjavate negativne vrednosti?
- Kolone se zovu `Invoice`, `Price`, `Customer ID`. U većini tutorijala na
  internetu zovu se `InvoiceNo`, `UnitPrice`, `CustomerID`. Preimenujte ih na
  duži oblik — zašto je dobra ideja imati jedan dogovoren skup naziva?

### 1.3 ⚠️ Proverite da li se listovi preklapaju

Ovo je **najvažniji korak dana** i preskače ga skoro svako.

Uporedite najmanji i najveći `InvoiceDate` po svakom listu. Zatim proverite da
li postoje brojevi računa koji se pojavljuju u **oba** lista.

❓ **Pitanja:**
- Da li se periodi preklapaju? U kom tačno intervalu?
- Koliko računa postoji u oba lista?
- Da su ti redovi ostali duplirani, šta bi se desilo sa prikazanom prodajom za
  decembar 2010?

### 1.4 Očistite podatke — ovim redosledom

Radite **korak po korak** i posle svakog koraka zapišite koliko je redova ostalo.

| Korak | Šta radite |
|---|---|
| 1 | Uklonite duplikate (identične redove) |
| 2 | Izdvojite otkazivanja — `InvoiceNo` počinje slovom `C` — u **poseban** DataFrame |
| 3 | Uklonite redove bez `CustomerID` |
| 4 | Uklonite službene šifre: `POST`, `DOT`, `M`, `C2`, `D`, `S`, `B`, `BANK CHARGES`, `ADJUST`, `AMAZONFEE`, `CRUK`, `PADS`, `TEST001`, `gift_0001_*` |
| 5 | Uklonite `Quantity <= 0` |
| 6 | Uklonite `UnitPrice <= 0` |

⚠️ **Otkazivanja NE brišete zauvek** — snimite ih u zaseban fajl. Trebaće vam na
danu 6 i to je jedna od najzanimljivijih analiza u celoj praksi.

### 1.5 Napravite izvedene kolone

`TotalPrice = Quantity * UnitPrice`, plus `Godina`, `Mesec`, `Datum`,
`DanUNedelji`, `Sat` iz `InvoiceDate`.

### 1.6 Snimite rezultat

Snimite očišćene podatke u `.parquet` — svaki sledeći dan počinje učitavanjem
ovog fajla, ne ponovnim čišćenjem.

## ✅ Kontrolne tačke

| Pokazatelj | Očekivana vrednost |
|---|---|
| Sirovih redova | **1.067.371** |
| Kolona (bez one koju sami dodate za list) | 8 |
| Nedostaje `Customer ID` (sirovo) | 243.007 (22,8%) |
| Nedostaje `Description` (sirovo) | 4.382 |
| Perioda listova se preklapaju | **01–09.12.2010** |
| Računa u oba lista | **1.088** |
| Uklonjeno duplikata | ~34.335 |
| Otkazanih stavki | ~19.104 (u ~8.292 računa) |
| **Redova posle čišćenja** | **~776.579 (72,8% originala)** |
| Jedinstvenih kupaca | ~5.852 |
| Jedinstvenih računa | ~36.594 |
| Jedinstvenih artikala | ~4.620 |
| Ukupan prihod (`TotalPrice`) | ~17.068.583 |

## ❓ Pitanja za notebook

1. Koliko procenata podataka ste izbacili? Da li je to previše?
2. Redovi bez `CustomerID` su skoro četvrtina skupa. Zašto ih ipak izbacujemo za
   analizu kupaca, a **ne** izbacujemo iz ukupne prodaje?
3. Posle uklanjanja otkazivanja, koliko je ostalo redova sa `Quantity <= 0`?
   Šta vam taj rezultat govori o tome odakle dolaze negativne količine?

## 📦 Isporuka dana 1

Notebook sa tabelom „korak čišćenja → koliko redova uklonjeno → zašto“ i
kratkim komentarom.

---

# DAN 2 — Istraživačka analiza (EDA)

**Cilj:** razumeti oblik podataka pre nego što se išta modeluje.

## Zadaci

### 2.1 Raspodele ključnih varijabli

Nacrtajte histogram za `Quantity`, `UnitPrice` i `TotalPrice`.

⚠️ Prvi histogram će izgledati kao **jedan jedini stubić uz nulu**. To nije
greška — tako izgleda jako asimetrična raspodela. Nacrtajte isti histogram u
**logaritamskoj skali** (`ax.set_xscale("log")`) i uporedite.

Nacrtajte i box plot i izračunajte koliko ima outlier-a po IQR pravilu.

### 2.2 Pogledajte najveće vrednosti

Ispišite 10 najvećih stavki po `TotalPrice`.

❓ Da li su to greške u podacima ili stvarne kupovine? Kako ste zaključili?
Da li ih treba izbaciti?

### 2.3 Prodaja po državama

Izračunajte prihod, broj kupaca i prosečnu korpu po državi.

⚠️ Jedna država će „pojesti“ ceo graf. Rešite to tako da se vidi i njena
dominacija i struktura ostalih.

### 2.4 Prodaja kroz vreme

Napravite mesečnu i dnevnu agregaciju `TotalPrice` i nacrtajte vremensku seriju.
Na dnevnu seriju dodajte pokretni prosek od 7 dana.

⚠️ **Zamka:** poslednji mesec u podacima je **nepotpun** (podaci se završavaju
09.12.2011). Ako ga nacrtate kao i ostale, izgledaće kao katastrofalan pad
prodaje. Obeležite ga drugačije ili isključite iz poređenja.

### 2.5 Kada kupci kupuju

Napravite heatmapu `dan u nedelji × sat u danu` sa ukupnim prihodom.

## ✅ Kontrolne tačke

| Pokazatelj | Očekivana vrednost |
|---|---|
| `Quantity` — medijana / prosek | 6 / 13,5 |
| `UnitPrice` — medijana / prosek | 1,95 / 2,93 |
| `TotalPrice` — medijana / prosek | 12,45 / 21,98 |
| Udeo Velike Britanije u prihodu | **~83,7%** |
| Top 5 država bez UK | EIRE, Netherlands, Germany, France, Australia |
| Najjači pun mesec | **2010-11**, ~1.155.978 |
| Najslabiji mesec | 2011-02, ~442.294 |
| Udeo subote u prihodu | **~0,06%** |

## ❓ Pitanja za notebook

1. Zašto je prosek `TotalPrice` skoro dvostruko veći od medijane? Koju od te dve
   mere biste koristili u izveštaju za direktora i zašto?
2. Irska i Holandija su visoko po prihodu ali imaju **vrlo malo kupaca**.
   Izračunajte prosečnu korpu po državi. Šta zaključujete o tome kakvi su to kupci?
3. Da li postoji sezonalnost? Da li se obrazac ponavlja obe godine? Zašto je
   ponavljanje bitno za zaključak?
4. Šta se dešava subotom? Da li je to greška u podacima?
5. Pogledajte heatmapu. U koje doba dana se kupuje? Šta vam to govori o tome da
   li su kupci fizička lica ili firme?

## 📦 Isporuka dana 2

Notebook sa najmanje 5 grafikona, **svaki sa zaključkom ispod njega**.

---

# DAN 3 — Proizvodi i RFM analiza

**Cilj:** saznati šta se prodaje i ko su kupci.

## Zadaci

### 3.1 Najprodavaniji proizvodi

Napravite **dve** top 10 liste: po ukupnoj količini i po ukupnom prihodu.

❓ Koliko se proizvoda pojavljuje na obe liste? Šta to znači?

### 3.2 Pareto analiza

Sortirajte proizvode po prihodu opadajuće, izračunajte kumulativni udeo i
nacrtajte krivu.

❓ Koliko procenata asortimana donosi 80% prihoda? Da li važi pravilo 80/20?

### 3.3 RFM analiza

Za svakog kupca izračunajte:

- **Recency** — broj dana od poslednje kupovine
- **Frequency** — broj različitih računa (`nunique`, ne `count`!)
- **Monetary** — ukupna potrošnja

⚠️ **Zamka:** za Recency vam treba referentni datum. **Nemojte koristiti današnji
datum** — podaci su iz 2011. i svi kupci bi ispali „izgubljeni“ pre 15 godina.
Koristite `maksimalni datum u podacima + 1 dan`.

### 3.4 Skorovanje i segmenti

Podelite kupce u 5 grupa (kvintila) po svakoj od tri veličine, sa skorom 1–5.

⚠️ Za Recency je **manje bolje** — kupac koji je kupovao pre 3 dana dobija skor 5.

⚠️ **Tehnička zamka:** `pd.qcut` na `Frequency` će pući ili napraviti prazne
grupe, jer ogroman broj kupaca ima istu vrednost (1 ili 2). Rešenje:
rangirajte pre deljenja — `serija.rank(method="first")`.

Zatim dodelite segmentima poslovna imena na osnovu R i F skora, npr.:

| R | F | Segment |
|---|---|---|
| 5 | 4–5 | Šampioni |
| 3–4 | 4–5 | Lojalni |
| 4–5 | 2–3 | Potencijalno lojalni |
| 5 | 1 | Novi kupci |
| 1–2 | 3–4 | U riziku |
| 1–2 | 5 | Ne smemo ih izgubiti |
| 1–2 | 1–2 | Izgubljeni |

(Popunite i ostale kombinacije — mora ih biti 25.)

## ✅ Kontrolne tačke

| Pokazatelj | Očekivana vrednost |
|---|---|
| Broj proizvoda (grupisano po `StockCode` + `Description`) | ~5.274 |
| Broj jedinstvenih `StockCode` | ~4.620 |
| Prvi po prihodu | **REGENCY CAKESTAND 3 TIER** (~277.656) |
| Prvi po količini | **WORLD WAR 2 GLIDERS ASSTD DESIGNS** (~105.185 kom) |
| Pareto: % asortimana za 80% prihoda | **~22%** |
| Kupaca u RFM tabeli | **5.852** |
| Recency — medijana | ~95 dana |
| Frequency — medijana | ~3 |
| Monetary — medijana / prosek | ~856 / ~2.917 |
| Segment „Šampioni“ | **~14% kupaca → ~51% prihoda** |

## ❓ Pitanja za notebook

1. Zašto se liste „top po količini“ i „top po prihodu“ toliko razlikuju?
2. Grupisanje po `StockCode` daje ~4.620 proizvoda, a po `StockCode` +
   `Description` ~5.274. Odakle razlika? Šta to znači za pouzdanost naziva
   proizvoda i koji biste ključ koristili?
3. Jedan proizvod u top listi ima samo **1 kupca**. Nađite ga. Da li je to
   normalno i šta bi trebalo uraditi sa njim u analizi?
4. Medijana potrošnje je ~856, a prosek ~2.917. Objasnite razliku.
5. Koji segment ima najviše kupaca? Koji donosi najviše novca? Da li je to isti
   segment? Šta ta razlika znači za marketinški budžet?
6. Za segment „Ne smemo ih izgubiti“ — opišite tipičnog kupca i predložite jednu
   konkretnu akciju.

## 📦 Isporuka dana 3

Notebook sa RFM tabelom, tabelom segmenata (broj kupaca + udeo prihoda po
segmentu) i grafikonom segmentacije.

---

# DAN 4 — Segmentacija kupaca (K-Means)

**Cilj:** pustiti algoritam da sam nađe grupe i uporediti ga sa ručnim pravilima.

## Zadaci

### 4.1 Priprema podataka

Uzmite `Recency`, `Frequency`, `Monetary` iz dana 3 i:

1. primenite `np.log1p()` na sve tri
2. standardizujte sa `StandardScaler`

❓ Izračunajte `skew()` pre i posle log transformacije. Zašto su **oba** koraka
obavezna? Šta bi se konkretno desilo sa klasterima da preskočite standardizaciju?

### 4.2 Odredite broj klastera

Za `k` od 2 do 10 izračunajte inerciju (elbow) i `silhouette_score`. Nacrtajte
obe krive.

### 4.3 Klasterujte i protumačite

Pokrenite K-Means sa izabranim `k`. Za svaki klaster izračunajte prosečan R, F,
M, broj kupaca i udeo u prihodu, pa mu **dajte ime** koje ima poslovno značenje.

⚠️ Ime mora da bude opravdano brojevima. Ne „Klaster 3“, nego „VIP kupci —
kupuju svakih par nedelja, prosečno 33 porudžbine“.

### 4.4 Uporedite sa danom 3

Napravite `pd.crosstab` između RFM segmenata i K-Means klastera.

## ✅ Kontrolne tačke

| Pokazatelj | Očekivana vrednost |
|---|---|
| `skew` Monetary pre log-a | ~25 |
| `skew` Monetary posle log-a | ~0,3 |
| Elbow predlaže | **k = 5** |
| Najbolja silueta je za | **k = 2** (~0,44) |
| Najvredniji klaster | **~8% kupaca → ~55% prihoda** |

## ❓ Pitanja za notebook

1. Silueta kaže `k=2`, elbow kaže `k=5`. Koji biste izabrali i **zašto**?
   (Nema jednog tačnog odgovora — ocenjuje se obrazloženje.)
2. Opišite svaki klaster u **jednoj rečenici razumljivoj direktoru prodaje**.
3. Da li se K-Means i RFM segmenti poklapaju? Gde se najviše razilaze?
4. Koji pristup biste koristili u praksi i za šta? Navedite po jednu prednost
   svakog.

## 📦 Isporuka dana 4

Notebook sa elbow/silueta grafikom, vizuelizacijom klastera i **imenovanim**
opisom svakog klastera.

---

# DAN 5 — Predikcija: da li će se kupac vratiti?

> **Ovo je najvažniji dan prakse.** Sadrži grešku koju pravi većina početnika,
> i koja se na razgovoru za posao odmah primeti.

**Cilj:** napraviti pošten model i razumeti zašto je „lošiji“ rezultat ovde
zapravo tačan.

## Zadaci

### 5.1 Prvo napravite grešku — namerno

Uradite tačno ovo:

1. Izračunajte RFM na **celom** skupu podataka.
2. Definišite cilj: `Cilj = 1 ako je Recency <= 30, inače 0`.
3. Obučite logističku regresiju sa osobinama `Recency`, `Frequency`, `Monetary`.
4. Izmerite tačnost i ROC AUC.

✅ Dobićete tačnost preko **0,97** i AUC preko **0,99**.

❓ **Pitanje:** zašto je ovaj rezultat bezvredan? Pogledajte pažljivo definiciju
cilja i listu osobina.

### 5.2 Sada postavite zadatak ispravno

Podelite vreme na dva prozora:

```
|------------------- PROŠLOST -------------------|--- BUDUĆNOST ---|
        odavde računate OSOBINE                 ↑   ovde merite CILJ
                                          DATUM PRESEKA
                                     (poslednji datum − 90 dana)
```

- **Datum preseka** = maksimalni datum u podacima − 90 dana
- **Osobine** — računate **isključivo** iz podataka pre preseka
- **Cilj** — da li je kupac kupio bilo šta **posle** preseka (1/0)
- U skup ulaze samo kupci koji su bili aktivni **pre** preseka

### 5.3 Napravite osobine

Na nivou kupca, sve iz prošlosti: RFM, broj stavki, broj različitih proizvoda,
prosečna korpa, starost odnosa (dani od prve kupovine do preseka), prosečan
razmak između porudžbina, da li je iz UK, aktivnost u poslednjih 90 dana pre
preseka.

### 5.4 Obučite i uporedite modele

Obavezno uključite **tri**:

1. `DummyClassifier(strategy="most_frequent")` — osnovna linija
2. `LogisticRegression` — sa `StandardScaler` u `Pipeline`
3. `RandomForestClassifier`

Podelite na train/test sa `stratify=y`. Izmerite tačnost, preciznost, odziv,
F1 i ROC AUC. Nacrtajte matrice konfuzije i ROC krive.

### 5.5 Protumačite model

Izvucite `feature_importances_` iz Random Forest-a i koeficijente iz logističke
regresije.

## ✅ Kontrolne tačke

| Pokazatelj | Očekivana vrednost |
|---|---|
| AUC „pokvarenog“ modela iz 5.1 | **~0,999** |
| Datum preseka | **2011-09-10** |
| Kupaca u skupu za model | ~5.256 |
| Udeo onih koji su se vratili | **~44%** |
| Tačnost osnovne linije | ~56,5% |
| Logistička regresija — AUC | **~0,79** |
| Random Forest — AUC | **~0,80** |
| Najvažnija osobina | **Recency** |

## ❓ Pitanja za notebook

1. Objasnite svojim rečima šta je curenje podataka i kako ste ga izbegli.
2. Osnovna linija ima tačnost 56,5%, vaš model ~74%. Da li je model dobar?
   Zašto tačnost ovde nije dovoljna mera?
3. Objasnite matricu konfuzije **rečima**: šta je „lažna uzbuna“, a šta
   „propušten kupac“? Koja greška je skuplja za firmu i zašto?
4. `Recency` je najvažnija osobina. Da li je to **isto** curenje podataka kao u
   5.1? Obrazložite — ovo je suština celog dana.
5. Firma šalje kupon od 5 evra kupcima za koje model kaže da će se vratiti.
   Da li biste podigli ili spustili prag odlučivanja sa 0,5? Šta vam još treba
   da biste odgovorili tačno?

## 📦 Isporuka dana 5

Notebook sa **oba** modela (pokvarenim i ispravnim), tabelom poređenja,
matricama konfuzije i zaključkom koji je model bolji i zašto.

---

# DAN 6 — Napredna tema (birate jednu)

**Cilj:** samostalno rešiti problem koji nije korak-po-korak razložen.

Ovde dobijate manje uputstava namerno. Birate **jednu** opciju.

## Opcija A — Detekcija anomalija (otkazivanja)

Vratite fajl sa otkazivanjima iz dana 1.

- Kolika je vrednost otkazanih porudžbina i koliki je to % prihoda?
- Kako se stopa otkazivanja menja kroz mesece? Postavite prag
  (npr. `prosek + 2σ`) i označite mesece koji ga prelaze.
- Koji se proizvodi najviše otkazuju — po vrednosti i po **stopi**
  (otkazano / ukupno naručeno)?
- Ima li kupaca sa neuobičajeno visokom stopom otkazivanja?

✅ Kontrolne tačke: ~8.292 otkazana računa, vrednost ~1.462.051 (**~8,6%** prihoda),
2011-12 je izrazito odstupanje (~40%).

❓ Nađite grupu proizvoda sa stopom otkaza **blizu 50%**. Šta bi mogao biti
uzrok i šta biste preporučili firmi?

⚠️ Pre nego što zaključite uzrok, izračunajte i **koliko zasebnih storno-stavki**
stoji iza svake stope. Stopa od 50% nastala od jednog jedinog storna i stopa od
50% nastala od sedam različitih kupaca **nisu ista pojava** i ne vode istoj
preporuci.

## Opcija B — Prognoza prodaje za naredni mesec

- Napravite dnevnu vremensku seriju prihoda.
- Podelite na train/test **po vremenu** (poslednjih 60 dana = test). Zašto ne
  nasumično?
- Napravite bar tri modela, uključujući **naivni** (vrednost od istog dana
  prošle nedelje) kao osnovnu liniju.
- Izmerite MAE, RMSE i MAPE. Prognozirajte narednih 30 dana.

⚠️ Ako koristite lag osobine (`prihod pre 7 dana`), ne možete da prognozirate
30 dana unapred bez rekurzije. Razmislite koje osobine se mogu izračunati za
datum u budućnosti — kalendarske mogu.

✅ Kontrolne tačke: naivni model MAPE ~33%, najbolji model MAPE ~27%.

❓ Zašto je greška od ~27% na dnevnom nivou prihvatljiva, a bila bi katastrofalna
na mesečnom?

## Opcija C — Sistem preporuka

- Za N najčešćih artikala (npr. 300) izračunajte koliko puta se svaki par
  pojavljuje na istom računu.
- Izračunajte **support**, **confidence** i **lift**.
- Izlistajte najjače parove i napravite funkciju `preporuci(sifra_artikla)`.

⚠️ Sa svih ~4.600 artikala matrica ima 21 milion parova. Ograničite se na
najčešće — veza između dva artikla koja su se prodala po 3 puta ionako nije
pouzdana.

✅ Kontrolne tačke: na 300 artikala i ~33.000 računa dobija se ~28.000 parova sa
lift > 1; najjači par ima lift ~37.

❓ Pogledajte najjače parove — šta im je zajedničko? Da li vam to otkriva nešto o
tome kako kupci kupuju? Koje je glavno **ograničenje** ovakvog sistema?

## 📦 Isporuka dana 6

Notebook ili `.py` skripta koja rešava izabranu temu + kratak opis nalaza.

---

# DAN 7 — Finalni izveštaj i prezentacija

**Cilj:** ispričati priču nekome ko nije video vaš kod.

## Zadaci

### 7.1 Napišite izveštaj (2–4 strane)

Obavezne sekcije:

1. **Rezime za menadžment** — 5 nalaza, svaki jedna rečenica sa brojem.
   *Ovo se piše poslednje, a čita prvo.*
2. **Podaci i priprema** — šta ste dobili, šta ste izbacili, koliko je ostalo
3. **Nalazi** — 4–6 najvažnijih, svaki sa jednim grafikonom
4. **Model** — postavka, rezultat, šta znači
5. **Preporuke** — 5–7 konkretnih akcija, svaka vezana za nalaz
6. **Ograničenja** — šta vaša analiza **ne** može da tvrdi

⚠️ Sekcija „Ograničenja“ nije priznanje slabosti — to je znak da razumete šta
ste uradili. Analiza bez ograničenja deluje neozbiljno.

### 7.2 Napravite prezentaciju (5–8 slajdova, 10 minuta)

Jedan slajd = jedna poruka. Bez tabela sa 20 redova. Bez koda.

### 7.3 Pripremite se za pitanja

Očekujte:

- „Zašto ste izbacili četvrtinu podataka?“
- „Koliko je vaš model zapravo dobar?“
- „Šta biste uradili da imate još nedelju dana?“
- „Na osnovu čega tvrdite da su Šampioni najvredniji?“

## 📦 Isporuka dana 7

Izveštaj + prezentacija + uredan git repozitorijum.

---

## Kako se ocenjuje

| Oblast | Udeo | Šta se gleda |
|---|---|---|
| Ispravnost | 30% | Da li su brojevi tačni, da li je čišćenje opravdano, ima li curenja podataka |
| Zaključci | 30% | Da li uz svaki nalaz ide tumačenje i poslovni smisao |
| Kod | 20% | Čitljivost, ponovljivost, smislena imena, bez „copy-paste“ blokova |
| Vizuelizacija | 10% | Da li se graf razume bez objašnjenja, ima li naslov i jedinice |
| Komunikacija | 10% | Izveštaj i prezentacija razumljivi nekome ko ne zna Python |

**Automatski gubitak poena:**

- graf bez zaključka
- korišćenje `datetime.now()` za Recency
- model sa curenjem podataka bez komentara
- tvrdnja „model ima 98% tačnosti“ bez poređenja sa osnovnom linijom
- „pad prodaje u decembru 2011“ (nepotpun mesec)

---

## Najčešće greške iz prethodnih generacija

1. **Zaborave da provere preklapanje listova** → svi brojevi za decembar 2010. su duplo veći.
2. **Koriste `count()` umesto `nunique()`** za Frequency → broje stavke umesto porudžbina.
3. **Koriste današnji datum za Recency** → svi kupci „izgubljeni“.
4. **Curenje podataka na danu 5** → savršen rezultat, bezvredan model.
5. **Ne standardizuju pre K-Means** → Monetary sam određuje sve klastere.
6. **Crtaju 15 grafikona bez ijednog zaključka.**
7. **Izbace outlier-e „jer su outlier-i“** → izbace najveće kupce, tj. ceo posao.
8. **Zaokruže na „prosečan kupac troši 2.917“** → prosek koji ne opisuje nikoga.

---

## Skraćena verzija (5 dana)

| Dan | Sadržaj |
|---|---|
| 1 | Dan 1 (čišćenje) |
| 2 | Dan 2 (EDA) |
| 3 | Dan 3 (proizvodi + RFM) |
| 4 | Dan 5 (predikcija) — **ne preskačite ovaj dan** |
| 5 | Dan 7 (izveštaj) |

K-Means (dan 4) i napredna tema (dan 6) se izostavljaju. Dan 5 se ne izostavlja
nikada — curenje podataka je najvrednija lekcija cele prakse.

---

## Rečnik pojmova

| Pojam | Značenje |
|---|---|
| **RFM** | Recency, Frequency, Monetary — tri broja koja opisuju kupca |
| **Outlier** | Vrednost daleko od ostalih; nije nužno greška |
| **Skewness** | Asimetrija raspodele; visoka = prosek nije reprezentativan |
| **Curenje podataka** | Kad model dobije informaciju koju u stvarnosti ne bi imao |
| **Osnovna linija (baseline)** | Najgluplji mogući model; svaki pravi mora da ga pobedi |
| **Preciznost** | Od predviđenih „da“, koliko ih je stvarno „da“ |
| **Odziv (recall)** | Od stvarnih „da“, koliko smo ih pogodili |
| **ROC AUC** | Koliko dobro model rangira; 0,5 = nasumično, 1,0 = savršeno |
| **Inercija** | Zbir kvadrata rastojanja tačaka od centra klastera |
| **Silueta** | Mera koliko su klasteri dobro razdvojeni (−1 do 1) |
| **Lift** | Koliko puta se par artikala kupuje zajedno češće nego slučajno |

---

*Sva pitanja — mentoru. Bolje pitati drugog dana nego predati pogrešno sedmog.*
