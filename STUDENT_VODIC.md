# Vodič za studente — korak po korak

Ovo je **detaljno uputstvo** kroz celu praksu. Uz svaki korak piše šta treba da
dobijete, pa u svakom trenutku znate da li ste na dobrom putu.

| Dokument | Čemu služi |
|---|---|
| [STUDENTI_ZADATAK.md](STUDENTI_ZADATAK.md) | **Šta** treba uraditi i šta se predaje |
| **STUDENT_VODIC.md** (ovaj) | **Kako** se to radi, korak po korak, sa proverama |

> **Skela se postepeno sklanja.** Za dan 1 dobijate skoro ceo kod. Za dan 3
> dobijate strukturu i savete. Za dan 6 dobijate samo ideju. Tako i treba — cilj
> je da poslednjeg dana radite sami.

---

## Kako da čitate ovaj vodič

Kroz tekst se ponavljaju četiri oznake:

| Oznaka | Značenje |
|---|---|
| **✅ PROVERA** | Broj koji morate dobiti. Ne poklapa se → stanite i vratite se korak nazad. |
| **💡 ZAŠTO** | Objašnjenje zašto se nešto radi baš tako. Ovo se ocenjuje. |
| **⚠️ ZAMKA** | Mesto gde se najčešće greši. |
| **🖼 OVAKO IZGLEDA** | Slika rezultata koji treba da dobijete. |

**Najvažnije:** ne prepisujte kod bez razumevanja. Kad negde zapnete, pokušajte
sami 15 minuta pre nego što pogledate rešenje.

---

## Sadržaj

- [Dan 0 — Priprema](#dan-0--priprema-30-minuta)
- [Dan 1 — Čišćenje podataka](#dan-1--čišćenje-podataka)
- [Dan 2 — Istraživačka analiza](#dan-2--istraživačka-analiza)
- [Dan 3 — Proizvodi i RFM](#dan-3--proizvodi-i-rfm)
- [Dan 4 — Klasterovanje](#dan-4--klasterovanje)
- [Dan 5 — Predikcija](#dan-5--predikcija-)
- [Dan 6 — Napredna tema](#dan-6--napredna-tema)
- [Dan 7 — Izveštaj](#dan-7--izveštaj)
- [Sve kontrolne tačke na jednom mestu](#sve-kontrolne-tačke-na-jednom-mestu)
- [Kad nešto ne radi — simptom → uzrok](#kad-nešto-ne-radi--simptom--uzrok)

---

# DAN 0 — Priprema (30 minuta)

## 0.1 Napravite radno okruženje

```bash
python -m venv .venv
.venv\Scripts\activate                 # Windows
# source .venv/bin/activate            # Linux / macOS

pip install pandas numpy openpyxl pyarrow matplotlib scikit-learn jupyterlab tabulate
```

## 0.2 Napravite strukturu foldera

```
moj_rad/
├── notebooks/          <- ovde rade vaši notebook-ovi
├── podaci/             <- ovde snimate međurezultate
├── grafovi/            <- ovde snimate slike
└── odgovori.csv        <- kopija fajla sablon_odgovori.csv
```

## 0.3 Prva provera da sve radi

```python
import pandas as pd
import matplotlib.pyplot as plt
import sklearn

print("pandas :", pd.__version__)
print("sklearn:", sklearn.__version__)
```

**✅ PROVERA:** ispisuju se verzije, nema greške.

## 0.4 Kopirajte `odgovori.csv`

Kopirajte `sablon_odgovori.csv` u svoj folder pod imenom `odgovori.csv`.
Popunjavaćete ga usput — na kraju svakog dana upišete brojeve koje ste dobili.

---

# DAN 1 — Čišćenje podataka

**Cilj dana:** od 1.067.371 sirovog reda doći do ~776.579 redova kojima možete
verovati, i **znati tačno šta ste izbacili**.

---

## Korak 1.1 — Učitajte oba lista

⚠️ **ZAMKA ODMAH NA POČETKU:** Excel fajl ima **dva lista**. Ako napišete
`pd.read_excel("podaci.xlsx")`, dobićete **samo prvi** i izgubiti pola podataka.

```python
import pandas as pd

# sheet_name=None znaci "ucitaj SVE listove" -> vraca recnik {ime_lista: DataFrame}
listovi = pd.read_excel("podaci.xlsx", sheet_name=None)

print("Listovi:", list(listovi.keys()))
for ime, deo in listovi.items():
    print(f"  {ime}: {len(deo):,} redova")
```

**✅ PROVERA:**

```
Listovi: ['Year 2009-2010', 'Year 2010-2011']
  Year 2009-2010: 525,461 redova
  Year 2010-2011: 541,910 redova
```

> ⏱ Ovo traje **1,5–2 minuta**. Normalno je — fajl ima milion redova.

Sad ih spojite u jedan DataFrame, ali **zapamtite iz kog lista je koji red**:

```python
delovi = []
for ime, deo in listovi.items():
    deo = deo.copy()
    deo["List"] = ime          # trebace nam u koraku 1.4
    delovi.append(deo)

df = pd.concat(delovi, ignore_index=True)
print(f"Ukupno: {len(df):,} redova, {df.shape[1]} kolona")
```

**✅ PROVERA:** `Ukupno: 1,067,371 redova, 9 kolona`

---

## Korak 1.2 — Snimite keš (ovo vam štedi sate)

```python
df.to_parquet("podaci/sirovo.parquet", index=False)
```

Od sada, umesto čekanja od 2 minuta:

```python
df = pd.read_parquet("podaci/sirovo.parquet")     # traje ~1 sekundu
```

💡 **ZAŠTO:** parquet je binarni format koji čuva i tipove kolona. Ako u svakoj
ćeliji notebook-a ponovo čitate Excel, izgubićete sate čekanja tokom nedelje.

⚠️ Ako ovde pukne greška tipa `ArrowInvalid: cannot mix types`, to je zato što
kolone `Invoice` i `StockCode` sadrže **i brojeve i tekst** (`489434` ali i
`79323P`). Rešenje:

```python
for kol in ["Invoice", "StockCode"]:
    df[kol] = df[kol].astype(str).str.strip()
```

---

## Korak 1.3 — Upoznajte se sa podacima

```python
print(df.info())
print(df.isna().sum())
print(df.describe())
```

**✅ PROVERA — nedostajuće vrednosti:**

```
Description      4382
Customer ID    243007      <- 22,8% redova!
```

**✅ PROVERA — `describe()`:**

```
         Quantity   UnitPrice
mean         9.94        4.65
min     -80995.00   -53594.36      <- NEGATIVNO?!
50%          3.00        2.10
max      80995.00    38970.00
```

💡 **PRVO ZAPAŽANJE koje treba da zapišete:** i količina i cena imaju negativan
minimum. To znači da u podacima ima **povraćaja i korekcija**, ne samo prodaje.
Takođe, `max` je ogroman u odnosu na medijanu (`50%`) — ima ekstremnih vrednosti.

Preimenujte kolone da vam kod bude čitljiviji:

```python
df = df.rename(columns={
    "Invoice": "InvoiceNo",
    "Price": "UnitPrice",
    "Customer ID": "CustomerID",
})
```

💡 **ZAŠTO:** ovaj skup („Online Retail II") koristi kraća imena, a skoro svi
tutorijali na internetu koriste duža. Ako preimenujete odmah, kod koji nađete
na internetu će vam biti razumljiv.

---

## Korak 1.4 — ⭐ Provera preklapanja listova

**Ovo je najvažniji korak celog dana i najlakše ga je preskočiti.**

```python
for ime, deo in df.groupby("List"):
    print(f"{ime}: {deo.InvoiceDate.min()} -> {deo.InvoiceDate.max()}")
```

**✅ PROVERA:**

```
Year 2009-2010: 2009-12-01 07:45:00 -> 2010-12-09 20:01:00
Year 2010-2011: 2010-12-01 08:26:00 -> 2011-12-09 12:50:00
                ^^^^^^^^^^ pazljivo pogledajte ove datume
```

Prvi list ide **do 9. decembra 2010**, a drugi počinje **1. decembra 2010**.
**Preklapaju se devet dana.** Da li su isti računi u oba?

```python
fakture_po_listu = df.groupby("List")["InvoiceNo"].apply(set)
zajednicke = set.intersection(*fakture_po_listu.tolist())

print(f"Faktura u OBA lista: {len(zajednicke):,}")
print(f"Redova koji im pripadaju: {df.InvoiceNo.isin(zajednicke).sum():,}")
```

**✅ PROVERA:**

```
Faktura u OBA lista: 1,088
Redova koji im pripadaju: 45,046
```

💡 **ZAŠTO JE OVO VAŽNO:** 1.088 računa je upisano **dvaput**. Ako ih ne
uklonite, prodaja za decembar 2010. biće prikazana skoro duplo većom nego što
jeste — i vaš zaključak o sezonalnosti biće pogrešan.

⚠️ Ovo nije izmišljena zamka. Ovakve stvari se stalno dešavaju kad neko izvozi
podatke „po godinama" i uhvati preklapanje na granici.

---

## Korak 1.5 — Čišćenje, korak po korak

Sada čistite podatke **ovim redosledom**, i posle svakog koraka zapišete koliko
je redova ostalo.

Napravite malu pomoćnu funkciju da ne biste ručno brojali:

```python
koraci = []

def zabelezi(naziv, pre, posle):
    koraci.append({"Korak": naziv, "Pre": pre, "Posle": posle,
                   "Uklonjeno": pre - posle})
    print(f"{naziv:<34} {pre:>9,} -> {posle:>9,}   (-{pre-posle:,})")
```

### 1.5.a Duplikati

```python
poslovne = ["InvoiceNo", "StockCode", "Description", "Quantity",
            "InvoiceDate", "UnitPrice", "CustomerID", "Country"]

pre = len(df)
df = df.drop_duplicates(subset=poslovne, keep="first").reset_index(drop=True)
zabelezi("Uklonjeni duplikati", pre, len(df))
```

⚠️ Obratite pažnju na `subset=poslovne`. Da ste pozvali samo `df.drop_duplicates()`,
kolona `List` bi razlikovala inače identične redove i **preklapanje ne bi bilo
uhvaćeno**.

### 1.5.b Otkazivanja — izdvojite ih, NE brišite

```python
df["JeOtkazivanje"] = df.InvoiceNo.str.upper().str.startswith("C")

otkazivanja = df[df.JeOtkazivanje].copy()          # cuvamo za dan 6!
otkazivanja.to_parquet("podaci/otkazivanja.parquet", index=False)

pre = len(df)
df = df[~df.JeOtkazivanje].copy()
zabelezi("Izdvojena otkazivanja", pre, len(df))
```

💡 **ZAŠTO:** računi koji počinju slovom `C` (od *cancelled*) su storna. Nisu
prodaja, pa ne smeju u analizu prihoda — ali jesu vredan podatak sam po sebi.
Na danu 6 od njih se pravi cela analiza.

### 1.5.c Redovi bez kupca

```python
pre = len(df)
df = df[df.CustomerID.notna()].copy()
zabelezi("Uklonjeni redovi bez CustomerID", pre, len(df))
```

💡 **ZAŠTO:** bez šifre kupca ne možete izračunati „koliko je puta ovaj kupac
kupio". Ali pažnja — ta roba **se stvarno prodala**. Zato u izveštaju pišete
„analiza kupaca pokriva 5.852 identifikovana kupca", a ne „ukupna prodaja je
manja".

### 1.5.d Službene šifre

Nisu svi `StockCode` proizvodi. Pogledajte:

```python
print(df.StockCode.value_counts().head(20))
```

Videćete `POST` (poštarina), `M` (ručni unos), `BANK CHARGES`, `AMAZONFEE`,
`TEST001`... To nisu artikli.

```python
SLUZBENE = {"POST", "DOT", "C2", "C3", "M", "D", "S", "B", "BANK CHARGES",
            "ADJUST", "ADJUST2", "AMAZONFEE", "CRUK", "PADS", "TEST001", "TEST002"}

gore = df.StockCode.str.upper().str.strip()
maska = gore.isin(SLUZBENE) | gore.str.startswith("GIFT_0001")

pre = len(df)
df = df[~maska].copy()
zabelezi("Uklonjene sluzbene sifre", pre, len(df))
```

💡 **ZAŠTO:** da ih ostavite, „POST" bi vam se pojavio u top listi
najprodavanijih proizvoda — a poštarina nije proizvod.

### 1.5.e Nevalidne količine i cene

```python
pre = len(df)
df = df[df.Quantity > 0].copy()
zabelezi("Uklonjeno Quantity <= 0", pre, len(df))

pre = len(df)
df = df[df.UnitPrice > 0].copy()
zabelezi("Uklonjeno UnitPrice <= 0", pre, len(df))
```

**✅ PROVERA — kompletan „levak" čišćenja:**

```
Uklonjeni duplikati                1,067,371 -> 1,033,036   (-34,335)
Izdvojena otkazivanja              1,033,036 -> 1,013,932   (-19,104)
Uklonjeni redovi bez CustomerID    1,013,932 ->   779,495   (-234,437)
Uklonjene sluzbene sifre             779,495 ->   776,639   (-2,856)
Uklonjeno Quantity <= 0              776,639 ->   776,639   (-0)
Uklonjeno UnitPrice <= 0             776,639 ->   776,579   (-60)
```

❓ **Primetili ste nešto čudno?** Korak „Quantity <= 0" nije uklonio **nijedan**
red — iako smo u koraku 1.3 videli da minimum ide do −80.995.

Objašnjenje: **sve negativne količine su bile u otkazanim računima**, koje smo
već izdvojili. To je važan nalaz i treba da ga zapišete u notebook.

🖼 **OVAKO IZGLEDA:**

![Levak čišćenja](resenje/izlaz/grafovi/dan1_uklonjeni_redovi.png)

---

## Korak 1.6 — Izvedene kolone

U fajlu **ne postoji** kolona sa ukupnom vrednošću stavke. Pravite je sami:

```python
df["CustomerID"] = df.CustomerID.astype("int64")
df["TotalPrice"] = (df.Quantity * df.UnitPrice).round(2)

df["Godina"]      = df.InvoiceDate.dt.year
df["Mesec"]       = df.InvoiceDate.dt.to_period("M").astype(str)
df["Datum"]       = df.InvoiceDate.dt.date
df["DanUNedelji"] = df.InvoiceDate.dt.dayofweek       # 0 = ponedeljak
df["Sat"]         = df.InvoiceDate.dt.hour

df = df.drop(columns=["JeOtkazivanje"])
df.to_parquet("podaci/cisto.parquet", index=False)
```

---

## Korak 1.7 — Završna provera dana 1

```python
print(f"Redova           : {len(df):,}")
print(f"Kupaca           : {df.CustomerID.nunique():,}")
print(f"Racuna           : {df.InvoiceNo.nunique():,}")
print(f"Artikala         : {df.StockCode.nunique():,}")
print(f"Ukupan prihod    : {df.TotalPrice.sum():,.0f}")
print(f"Nema CustomerID  : {df.CustomerID.isna().sum()}")
print(f"Quantity <= 0    : {(df.Quantity <= 0).sum()}")
```

**✅ PROVERA:**

| Pokazatelj | Vrednost |
|---|---|
| Redova | **776.579** (72,8% od početka) |
| Kupaca | **5.852** |
| Računa | **36.594** |
| Artikala | **4.620** |
| Ukupan prihod | **17.068.583** |
| Nema CustomerID | 0 |
| Quantity <= 0 | 0 |

**📝 Upišite u `odgovori.csv`:**

```csv
dan1_redova_posle_ciscenja,776579
dan1_broj_kupaca,5852
dan1_ukupan_prihod,17068583
dan1_uklonjeno_duplikata,34335
```

---

# DAN 2 — Istraživačka analiza

**Cilj dana:** razumeti oblik podataka pre nego što išta modelujete.

```python
df = pd.read_parquet("podaci/cisto.parquet")
```

---

## Korak 2.1 — Raspodele: zašto prvi grafikon uvek razočara

```python
import matplotlib.pyplot as plt

fig, ax = plt.subplots(figsize=(7, 4))
ax.hist(df.TotalPrice, bins=60)
ax.set_title("TotalPrice")
plt.show()
```

Dobićete **jedan jedini stubić uz nulu**. Niste pogrešili — tako izgleda jako
asimetrična raspodela kad je nacrtate linearno.

Rešenje je **logaritamska skala**:

```python
import numpy as np

pozitivno = df.TotalPrice[df.TotalPrice > 0]
korpe = np.logspace(np.log10(pozitivno.min()), np.log10(pozitivno.max()), 60)

fig, ax = plt.subplots(figsize=(7, 4))
ax.hist(pozitivno, bins=korpe)
ax.set_xscale("log")
ax.axvline(df.TotalPrice.median(), color="orange", linestyle="--")
plt.show()
```

Sad se vidi lep zvonasti oblik.

🖼 **OVAKO IZGLEDA** (gore linearno = beskorisno, dole log = informativno):

![Raspodele](resenje/izlaz/grafovi/dan2_raspodele.png)

**✅ PROVERA:**

| | medijana | prosek |
|---|---|---|
| `Quantity` | **6** | 13,5 |
| `UnitPrice` | **1,95** | 2,93 |
| `TotalPrice` | **12,45** | 21,98 |

💡 **ZAŠTO SE PROSEK I MEDIJANA RAZLIKUJU:** prosek vuče nekoliko ogromnih
porudžbina. Najveća pojedinačna stavka je **168.470** (80.995 komada jednog
artikla). Zato:

> Za „tipičnu" vrednost koristite **medijanu**. Prosek koristite samo kad vas
> zanima ukupan iznos podeljen brojem.

⚠️ **NE BRIŠITE outlier-e.** To su prave veleprodajne porudžbine — vaši najbolji
kupci. Ko ih obriše, obrisao je posao.

---

## Korak 2.2 — Prodaja po državama

```python
po_drzavama = (df.groupby("Country")
                 .agg(Prihod=("TotalPrice", "sum"),
                      Kupaca=("CustomerID", "nunique"),
                      Racuna=("InvoiceNo", "nunique"))
                 .sort_values("Prihod", ascending=False))
po_drzavama["Udeo %"] = po_drzavama.Prihod / po_drzavama.Prihod.sum() * 100
po_drzavama["ProsecnaKorpa"] = po_drzavama.Prihod / po_drzavama.Racuna
print(po_drzavama.head(8).round(1))
```

**✅ PROVERA:**

```
                     Prihod  Kupaca  Udeo %  ProsecnaKorpa
United Kingdom  14288773.5    5334    83.7          428.3
EIRE              586626.1       3     3.4         1111.0
Netherlands       549773.4      22     3.2         2545.2
Germany           383289.0     107     2.3          509.0
France            309403.2      93     1.8          524.4
```

⚠️ **ZAMKA:** ako sve države nacrtate na jednom grafikonu, UK će biti jedini
vidljiv stubić. Rešenje: **dva grafikona** — jedan sa svima (da se vidi
dominacija), jedan bez UK (da se vidi struktura).

💡 **NALAZ KOJI TREBA DA UOČITE:** Irska ima **3 kupca**, a prosečnu korpu
1.111 (protiv 428 u UK). Holandija — 22 kupca, korpa 2.545. To **nisu**
maloprodajni kupci nego **veleprodajni partneri**.

🖼 **OVAKO IZGLEDA:**

![Države](resenje/izlaz/grafovi/dan2_drzave.png)

---

## Korak 2.3 — Vremenska serija i najveća zamka dana

```python
mesecno = df.groupby("Mesec")["TotalPrice"].sum().reset_index()
mesecno["Datum"] = pd.to_datetime(mesecno.Mesec)
print(mesecno.tail(4))
```

**✅ PROVERA:**

```
     Mesec      TotalPrice
   2011-09       938753.0
   2011-10      1002327.0
   2011-11      1136534.0
   2011-12       512228.0     <- pad od 55%?!
```

⚠️ **NAJVEĆA ZAMKA DANA 2:** decembar 2011. **nije** pad prodaje. Podaci se
prosto **završavaju 09.12.2011** — taj mesec ima samo 9 dana.

```python
print("Poslednji datum:", df.InvoiceDate.max())
# 2011-12-09 12:50:00
```

Rešenje: nacrtajte ga drugačije (isprekidano, sivo) ili ga izbacite iz poređenja.

```python
potpuni = mesecno.iloc[:-1]           # bez poslednjeg, nepotpunog meseca
najbolji = potpuni.loc[potpuni.TotalPrice.idxmax()]
najgori  = potpuni.loc[potpuni.TotalPrice.idxmin()]
print(f"Najbolji: {najbolji.Mesec} = {najbolji.TotalPrice:,.0f}")
print(f"Najgori : {najgori.Mesec} = {najgori.TotalPrice:,.0f}")
```

**✅ PROVERA:**

```
Najbolji: 2010-11 = 1,155,978
Najgori : 2011-02 = 442,294        (odnos 2,6x)
```

💡 **NALAZ:** prodaja raste od septembra, vrhunac u novembru, pa naglo pada.
Obrazac se ponavlja **obe godine** — zato je stvaran, a ne slučajnost.
Poslovno: zalihe se planiraju za avgust–oktobar.

🖼 **OVAKO IZGLEDA:**

![Mesečni prihod](resenje/izlaz/grafovi/dan2_mesecni_prihod.png)

---

## Korak 2.4 — Kada kupci kupuju

```python
heat = df.pivot_table(index="DanUNedelji", columns="Sat",
                      values="TotalPrice", aggfunc="sum")
```

**✅ PROVERA — udeo prihoda po danu:**

| Pon | Uto | Sre | Čet | Pet | **Sub** | Ned |
|---|---|---|---|---|---|---|
| 15,9% | 19,1% | 17,5% | 21,6% | 15,6% | **0,06%** | 10,3% |

💡 **NALAZ:** subota praktično ne postoji, a promet je koncentrisan radnim danom
10–15h. To je **B2B obrazac** — kupuju firme u toku radnog vremena, ne fizička
lica uveče. Ovo objašnjava i Irsku sa 3 kupca iz koraka 2.2.

🖼 **OVAKO IZGLEDA:**

![Heatmapa](resenje/izlaz/grafovi/dan2_heatmapa_dan_sat.png)

**📝 U `odgovori.csv`:**

```csv
dan2_udeo_uk_procenat,83.7
dan2_medijana_totalprice,12.45
dan2_najjaci_mesec,2010-11
```

---

# DAN 3 — Proizvodi i RFM

## Korak 3.1 — Dve različite „top" liste

```python
po_proizvodu = (df.groupby(["StockCode", "Description"])
                  .agg(Kolicina=("Quantity", "sum"),
                       Prihod=("TotalPrice", "sum"),
                       Kupaca=("CustomerID", "nunique"))
                  .reset_index())

print(po_proizvodu.nlargest(5, "Prihod")[["Description", "Prihod", "Kupaca"]])
print(po_proizvodu.nlargest(5, "Kolicina")[["Description", "Kolicina", "Kupaca"]])
```

**✅ PROVERA — po prihodu:**

```
REGENCY CAKESTAND 3 TIER            277,656    1314 kupaca
WHITE HANGING HEART T-LIGHT HOLDER  247,048    1490 kupaca
PAPER CRAFT , LITTLE BIRDIE         168,470       1 kupac    <- ?!
```

**✅ PROVERA — po količini:**

```
WORLD WAR 2 GLIDERS ASSTD DESIGNS   105,185 kom    (prihod samo 24,098)
WHITE HANGING HEART T-LIGHT HOLDER   91,757 kom
```

💡 **DVA NALAZA:**

1. Liste se poklapaju samo u **5 od 10** slučajeva. Jeftin artikal može biti prvi
   po komadima a nigde po novcu. „Najprodavaniji" zavisi od toga **šta merite**.
2. `PAPER CRAFT , LITTLE BIRDIE` je **treći po prihodu u celoj firmi** — a ima
   **jednog jedinog kupca**. To je jedna porudžbina od 80.995 komada, koja je
   (videćete na danu 6) kasnije otkazana u celini. Nikad ne gledajte top listu
   bez konteksta.

🖼 **OVAKO IZGLEDA:**

![Top proizvodi](resenje/izlaz/grafovi/dan3_top_proizvodi.png)

---

## Korak 3.2 — Pareto (pravilo 80/20)

```python
s = po_proizvodu.sort_values("Prihod", ascending=False).reset_index(drop=True)
s["Kumulativno %"] = s.Prihod.cumsum() / s.Prihod.sum() * 100
s["Udeo proizvoda %"] = (s.index + 1) / len(s) * 100

do80 = int((s["Kumulativno %"] <= 80).sum()) + 1
print(f"{do80:,} proizvoda ({do80/len(s)*100:.1f}% asortimana) = 80% prihoda")
```

**✅ PROVERA:** `1,146 proizvoda (21.7% asortimana) = 80% prihoda`

💡 Pravilo približno važi. Poslovno: na tih ~1.150 artikala treba držati zalihe.

🖼 ![Pareto](resenje/izlaz/grafovi/dan3_pareto.png)

---

## Korak 3.3 — RFM: tri broja koja opisuju kupca

| Slovo | Pitanje | Kako se računa |
|---|---|---|
| **R** — Recency | Kad je poslednji put kupio? | dana od poslednje kupovine |
| **F** — Frequency | Koliko puta je kupio? | broj **različitih računa** |
| **M** — Monetary | Koliko je potrošio? | zbir `TotalPrice` |

⚠️ **ZAMKA 1 — referentni datum.** Za Recency vam treba „danas". Ali podaci su
iz 2011!

```python
# POGRESNO:
# referentni = pd.Timestamp.now()      -> svi kupci "izgubljeni" pre 15 godina

# ISPRAVNO:
referentni = df.InvoiceDate.max() + pd.Timedelta(days=1)
print(referentni)        # 2011-12-10
```

⚠️ **ZAMKA 2 — `nunique` a ne `count`.** Frequency je broj **porudžbina**, ne
broj stavki. Kupac koji je jednom naručio 30 artikala ima Frequency **1**, ne 30.

```python
rfm = (df.groupby("CustomerID")
         .agg(Recency=("InvoiceDate", lambda s: (referentni - s.max()).days),
              Frequency=("InvoiceNo", "nunique"),        # <- nunique!
              Monetary=("TotalPrice", "sum"))
         .reset_index())
print(rfm[["Recency", "Frequency", "Monetary"]].describe().round(1))
```

**✅ PROVERA:**

| | Recency | Frequency | Monetary |
|---|---|---|---|
| **medijana (50%)** | **95** | **3** | **856** |
| prosek | 200 | 6,3 | 2.917 |
| max | 739 | 373 | 580.987 |

💡 Ako vam je medijana Recency preko 1.000 → koristili ste današnji datum.
Ako je medijana Frequency preko 20 → koristili ste `count` umesto `nunique`.

🖼 ![RFM raspodele](resenje/izlaz/grafovi/dan3_rfm_raspodele.png)

---

## Korak 3.4 — Skorovanje 1–5

Svaku od tri veličine delite na 5 grupa (kvintila).

⚠️ **ZAMKA 3 — `qcut` puca na Frequency.** Ogroman broj kupaca ima istu vrednost
(1 ili 2), pa `pd.qcut` javi `Bin edges must be unique`. Rešenje je rangiranje:

```python
def kvintili(serija, rastuce):
    """rastuce=True  -> vece je bolje (Frequency, Monetary)
       rastuce=False -> manje je bolje (Recency)"""
    rang = serija.rank(method="first", ascending=rastuce)
    return pd.qcut(rang, 5, labels=[1, 2, 3, 4, 5]).astype(int)

rfm["R"] = kvintili(rfm.Recency,   rastuce=False)   # skoro kupovao -> 5
rfm["F"] = kvintili(rfm.Frequency, rastuce=True)
rfm["M"] = kvintili(rfm.Monetary,  rastuce=True)
```

💡 **ZAŠTO `rastuce=False` za Recency:** kupac koji je kupovao pre 3 dana je
**bolji** od onog koji je kupovao pre 400 dana. Manji broj dana → veći skor.

---

## Korak 3.5 — Segmenti sa poslovnim imenima

```python
MAPA = {
    r"[1-2][1-2]": "Izgubljeni",
    r"[1-2][3-4]": "U riziku",
    r"[1-2]5":     "Ne smemo ih izgubiti",
    r"3[1-2]":     "Uspavani",
    r"33":         "Treba im paznja",
    r"[3-4][4-5]": "Lojalni",
    r"41":         "Obecavajuci",
    r"51":         "Novi kupci",
    r"[4-5][2-3]": "Potencijalno lojalni",
    r"5[4-5]":     "Sampioni",
}

rf = rfm.R.astype(str) + rfm.F.astype(str)
rfm["Segment"] = rf.replace(MAPA, regex=True)

sazetak = (rfm.groupby("Segment")
              .agg(Kupaca=("CustomerID", "count"), Prihod=("Monetary", "sum")))
sazetak["Udeo kupaca %"] = sazetak.Kupaca / sazetak.Kupaca.sum() * 100
sazetak["Udeo prihoda %"] = sazetak.Prihod / sazetak.Prihod.sum() * 100
print(sazetak.round(1).sort_values("Udeo prihoda %", ascending=False))
```

**✅ PROVERA:**

| Segment | Kupaca | % kupaca | % prihoda |
|---|---|---|---|
| **Šampioni** | 830 | **14,2%** | **51,4%** |
| Lojalni | 1.151 | 19,7% | 27,3% |
| U riziku | 759 | 13,0% | 6,4% |
| Potencijalno lojalni | 710 | 12,1% | 4,8% |
| Izgubljeni | 1.510 | 25,8% | 3,6% |

💡 **NAJVAŽNIJI NALAZ CELE PRAKSE:** 14% kupaca donosi 51% prihoda. Najbrojniji
segment („Izgubljeni", 26%) donosi svega 3,6%.

Poslovno: gubitak **jednog** Šampiona košta koliko gubitak **desetina** malih.

🖼 ![Segmenti](resenje/izlaz/grafovi/dan3_segmenti.png)
🖼 ![RF mreža](resenje/izlaz/grafovi/dan3_rf_mreza.png)

```python
rfm.to_parquet("podaci/rfm.parquet", index=False)
```

**📝 U `odgovori.csv`:**

```csv
dan3_medijana_recency,95
dan3_medijana_frequency,3
dan3_pareto_procenat,21.7
dan3_sampioni_udeo_prihoda,51.4
```

---

# DAN 4 — Klasterovanje

**Ideja:** umesto da vi smislite grupe (kao na danu 3), pustite algoritam da ih
sam nađe.

## Korak 4.1 — Priprema: dva obavezna koraka

```python
import numpy as np
from sklearn.preprocessing import StandardScaler

x = rfm[["Recency", "Frequency", "Monetary"]]
print("Asimetrija PRE :", x.skew().round(2).to_dict())

x_log = np.log1p(x)                       # (a) log transformacija
print("Asimetrija POSLE:", x_log.skew().round(2).to_dict())

x_skalirano = StandardScaler().fit_transform(x_log)    # (b) standardizacija
```

**✅ PROVERA:**

```
Asimetrija PRE : {'Recency': 0.89, 'Frequency': 12.04, 'Monetary': 25.33}
Asimetrija POSLE: {'Recency': -0.49, 'Frequency': 1.0, 'Monetary': 0.27}
```

💡 **ZAŠTO OBA KORAKA:** K-Means meri **rastojanje** između kupaca.

- Bez **log**: nekoliko kupaca sa potrošnjom od 500.000 povuklo bi centre ka
  sebi → dobili biste klaster od 3 čoveka.
- Bez **standardizacije**: Monetary ide do 580.000, a Frequency do 373. Monetary
  bi sam odredio sve klastere jer su mu brojevi hiljadu puta veći.

---

## Korak 4.2 — Koliko klastera?

```python
from sklearn.cluster import KMeans
from sklearn.metrics import silhouette_score

for k in range(2, 11):
    model = KMeans(n_clusters=k, random_state=42, n_init=10)
    oznake = model.fit_predict(x_skalirano)
    sil = silhouette_score(x_skalirano, oznake, sample_size=3000, random_state=42)
    print(f"k={k}: inercija={model.inertia_:>8.0f}  silueta={sil:.3f}")
```

**✅ PROVERA:**

```
k=2: inercija=    8562  silueta=0.444    <- najbolja silueta
k=3: inercija=    6338  silueta=0.351
k=4: inercija=    4908  silueta=0.365
k=5: inercija=    4090  silueta=0.340    <- "lakat" na krivoj inercije
k=6: inercija=    3549  silueta=0.336
```

💡 **DVE METODE, DVA ODGOVORA — i to je normalno.** Silueta kaže `k=2`, elbow
kaže `k=5`.

Silueta skoro uvek favorizuje `k=2` jer je najlakše podeliti podatke na dva
dela. Ali „dobri" i „loši" nisu upotrebljivi segmenti — ne možete na tome
graditi marketing. Biramo **k=5**.

⚠️ **Nema jednog tačnog odgovora.** Ocenjuje se vaše **obrazloženje**, ne izbor.

🖼 ![Izbor k](resenje/izlaz/grafovi/dan4_izbor_k.png)

---

## Korak 4.3 — Klasterujte i dajte imena

```python
model = KMeans(n_clusters=5, random_state=42, n_init=10)
rfm["Klaster"] = model.fit_predict(x_skalirano)

profil = rfm.groupby("Klaster").agg(
    Kupaca=("CustomerID", "count"),
    Recency=("Recency", "mean"),
    Frequency=("Frequency", "mean"),
    Monetary=("Monetary", "mean"),
    Prihod=("Monetary", "sum"))
profil["% kupaca"] = profil.Kupaca / profil.Kupaca.sum() * 100
profil["% prihoda"] = profil.Prihod / profil.Prihod.sum() * 100
print(profil.round(1))
```

**✅ PROVERA:**

| Ime koje dajete | Kupaca | % kupaca | % prihoda | Recency | Frequency | Monetary |
|---|---|---|---|---|---|---|
| **VIP kupci** | 454 | **7,8%** | **54,9%** | 17 | 32,6 | 20.636 |
| Lojalni kupci | 1.284 | 21,9% | 27,3% | 49 | 9,1 | 3.627 |
| Uspavani kupci | 1.352 | 23,1% | 10,9% | 289 | 3,8 | 1.375 |
| Povremeni kupci | 1.141 | 19,5% | 4,5% | 32 | 2,5 | 666 |
| Neaktivni kupci | 1.621 | 27,7% | 2,5% | 416 | 1,2 | 261 |

⚠️ **Brojevi klastera kod vas mogu biti drugačiji** (klaster 0 kod vas može biti
klaster 3 kod nas) — to je normalno. Ono što mora da se poklopi su **veličine
grupa i njihovi profili**.

💡 **OVO SE OCENJUJE:** dajte svakom klasteru ime i **opravdajte ga brojevima**:

> „**VIP kupci** — 454 kupca (7,8% baze) koji donose 54,9% prihoda. Naručivali su
> pre prosečno 17 dana, imaju 33 porudžbine i potrošili su preko 20.000."

„Klaster 3" **nije** odgovor.

🖼 ![Profil klastera](resenje/izlaz/grafovi/dan4_profil_klastera.png)
🖼 ![Rasejanje](resenje/izlaz/grafovi/dan4_klasteri_rasejanje.png)

**📝 U `odgovori.csv`:** `dan4_broj_klastera,5`

---

# DAN 5 — Predikcija ⭐

**Ovo je najvažniji dan cele prakse.** Pratite redosled tačno kako je napisan.

---

## Korak 5.1 — Prvo napravite grešku. Namerno.

```python
from sklearn.linear_model import LogisticRegression
from sklearn.model_selection import train_test_split
from sklearn.preprocessing import StandardScaler
from sklearn.pipeline import Pipeline
from sklearn.metrics import accuracy_score, roc_auc_score

referentni = df.InvoiceDate.max() + pd.Timedelta(days=1)
naivno = (df.groupby("CustomerID")
            .agg(Recency=("InvoiceDate", lambda s: (referentni - s.max()).days),
                 Frequency=("InvoiceNo", "nunique"),
                 Monetary=("TotalPrice", "sum")))

naivno["Cilj"] = (naivno.Recency <= 30).astype(int)      # <- ovde je greska

x = naivno[["Recency", "Frequency", "Monetary"]]
y = naivno.Cilj
x_tr, x_te, y_tr, y_te = train_test_split(x, y, test_size=0.3,
                                          random_state=42, stratify=y)

m = Pipeline([("s", StandardScaler()), ("m", LogisticRegression(max_iter=1000))])
m.fit(x_tr, y_tr)

print("Tacnost:", accuracy_score(y_te, m.predict(x_te)).round(4))
print("ROC AUC:", roc_auc_score(y_te, m.predict_proba(x_te)[:, 1]).round(4))
```

**✅ PROVERA:**

```
Tacnost: 0.9801
ROC AUC: 0.9998
```

**Rezultat je skoro savršen. Da li ste genijalci?**

**Niste.** Pogledajte pažljivo:

```
Cilj      = Recency <= 30
Osobine   = [Recency, Frequency, Monetary]
              ^^^^^^^
```

Cilj je **definisan preko** osobine koju ste dali modelu. Model nije naučio ništa
o ponašanju kupaca — samo je **pročitao odgovor iz pitanja**.

To se zove **curenje podataka** (*data leakage*).

> 💡 **Analogija:** predviđate ko će položiti ispit, a kao ulaz date podatak „da
> li je položio". 100% tačnost, nula vrednosti.

**PRAVILO KOJE PAMTITE ZAUVEK:**

> Osobine smeju da koriste **samo prošlost** u odnosu na trenutak predikcije.
> Cilj se meri u **budućnosti** u odnosu na taj isti trenutak.
> Ta dva prozora se **ne smeju preklapati**.

---

## Korak 5.2 — Sada ispravno: presecite vreme

```
|-------------- PROŠLOST --------------|------ BUDUĆNOST ------|
   odavde računate OSOBINE             ↑     ovde merite CILJ
                                  10.09.2011.
                             (poslednji datum − 90 dana)
```

```python
kraj = df.InvoiceDate.max()
presek = kraj - pd.Timedelta(days=90)
print("Datum preseka:", presek.date())

proslost  = df[df.InvoiceDate <= presek]
buducnost = df[df.InvoiceDate >  presek]
print(f"Proslost : {len(proslost):,} transakcija")
print(f"Buducnost: {len(buducnost):,} transakcija")
```

**✅ PROVERA:**

```
Datum preseka: 2011-09-10
Proslost : 618,316 transakcija
Buducnost: 158,263 transakcija
```

---

## Korak 5.3 — Osobine SAMO iz prošlosti

```python
o = (proslost.groupby("CustomerID")
     .agg(Recency=("InvoiceDate", lambda s: (presek - s.max()).days),
          Frequency=("InvoiceNo", "nunique"),
          Monetary=("TotalPrice", "sum"),
          BrojStavki=("InvoiceNo", "size"),
          RazlicitihProizvoda=("StockCode", "nunique"),
          PrvaKupovina=("InvoiceDate", "min"),
          ProsecnaKolicina=("Quantity", "mean"),
          ProsecnaCena=("UnitPrice", "mean"),
          Drzava=("Country", "first")))

o["ProsecnaKorpa"]  = o.Monetary / o.Frequency
o["StarostOdnosa"]  = (presek - o.PrvaKupovina).dt.days
o["ProsecanRazmak"] = np.where(o.Frequency > 1,
                               o.StarostOdnosa / (o.Frequency - 1),
                               o.StarostOdnosa)
o["JeUK"] = (o.Drzava == "United Kingdom").astype(int)

skoro = proslost[proslost.InvoiceDate > presek - pd.Timedelta(days=90)]
o["PorudzbinaZadnjih90"] = skoro.groupby("CustomerID")["InvoiceNo"].nunique().reindex(o.index).fillna(0)
o["PotrosnjaZadnjih90"]  = skoro.groupby("CustomerID")["TotalPrice"].sum().reindex(o.index).fillna(0)

# CILJ - iskljucivo iz buducnosti
o["Cilj"] = o.index.isin(set(buducnost.CustomerID.unique())).astype(int)
o = o.drop(columns=["PrvaKupovina", "Drzava"])

print(f"Kupaca: {len(o):,}")
print(f"Vratilo se: {o.Cilj.sum():,} ({o.Cilj.mean()*100:.1f}%)")
```

**✅ PROVERA:**

```
Kupaca: 5,256
Vratilo se: 2,289 (43.6%)
```

💡 Klase su 44% / 56%. Model koji **uvek** kaže „neće kupiti" ima tačnost
**56,4%**. Zato tačnost sama po sebi ne znači ništa — uvek je poredite sa
osnovnom linijom.

---

## Korak 5.4 — Tri modela, obavezno uključujući osnovnu liniju

```python
from sklearn.dummy import DummyClassifier
from sklearn.ensemble import RandomForestClassifier
from sklearn.metrics import precision_score, recall_score, f1_score

osobine = [k for k in o.columns if k != "Cilj"]
x_tr, x_te, y_tr, y_te = train_test_split(o[osobine], o.Cilj, test_size=0.25,
                                          random_state=42, stratify=o.Cilj)

modeli = {
    "Osnovna linija": DummyClassifier(strategy="most_frequent"),
    "Logisticka regresija": Pipeline([
        ("s", StandardScaler()),
        ("m", LogisticRegression(max_iter=2000, class_weight="balanced"))]),
    "Random Forest": RandomForestClassifier(
        n_estimators=300, max_depth=12, min_samples_leaf=20,
        class_weight="balanced", random_state=42, n_jobs=-1),
}

for ime, model in modeli.items():
    model.fit(x_tr, y_tr)
    p = model.predict(x_te)
    try:
        auc = roc_auc_score(y_te, model.predict_proba(x_te)[:, 1])
    except Exception:
        auc = 0.5
    print(f"{ime:<22} tacnost={accuracy_score(y_te,p):.3f} "
          f"preciznost={precision_score(y_te,p,zero_division=0):.3f} "
          f"odziv={recall_score(y_te,p,zero_division=0):.3f} auc={auc:.3f}")
```

**✅ PROVERA:**

```
Osnovna linija         tacnost=0.565 preciznost=0.000 odziv=0.000 auc=0.500
Logisticka regresija   tacnost=0.731 preciznost=0.692 odziv=0.691 auc=0.795
Random Forest          tacnost=0.743 preciznost=0.720 odziv=0.670 auc=0.805
```

💡 **KAKO SE OVO ČITA:**

| Mera | Šta znači, na ljudskom |
|---|---|
| **Tačnost** | Koliko procenata svih predviđanja je pogođeno |
| **Preciznost** | Od onih za koje kažemo „vratiće se", koliko ih se stvarno vrati |
| **Odziv** | Od onih koji su se stvarno vratili, koliko smo ih pogodili |
| **ROC AUC** | Koliko dobro model **rangira** kupce; 0,5 = novčić, 1,0 = savršeno |

**⚠️ AUC 0,80 je TAČAN rezultat — nije loš.** Ako dobijete preko 0,95, napravili
ste grešku iz koraka 5.1. Vratite se i proverite da nijedna osobina ne koristi
podatke posle datuma preseka.

🖼 ![ROC](resenje/izlaz/grafovi/dan5_roc_krive.png)
🖼 ![Matrice konfuzije](resenje/izlaz/grafovi/dan5_matrice_konfuzije.png)

---

## Korak 5.5 — Šta model koristi

```python
rf = modeli["Random Forest"]
vaznosti = pd.Series(rf.feature_importances_, index=osobine).sort_values(ascending=False)
print(vaznosti.head(5).round(3))
```

**✅ PROVERA:**

```
Recency               0.205
ProsecanRazmak        0.155
BrojStavki            0.103
Frequency             0.090
PotrosnjaZadnjih90    0.090
```

❓ **NAJVAŽNIJE PITANJE CELE PRAKSE:**

> `Recency` je najvažnija osobina. Da li je to **isto** curenje podataka kao u
> koraku 5.1?

✅ **NIJE.** U koraku 5.1 je `Recency` bio računat na celim podacima **i cilj je
bio definisan preko njega** — isto pitanje i odgovor.

Ovde je `Recency` **stanje na dan 10.09.2011** — informacija koju biste stvarno
imali tog dana — a cilj se dešava **posle** tog dana. To je dozvoljeno i
poslovno očekivano: najbolji nagoveštaj buduće kupovine je skorašnja kupovina.

Ako ovo umete da objasnite svojim rečima, razumeli ste najvažniju stvar u praksi.

🖼 ![Važnost osobina](resenje/izlaz/grafovi/dan5_vaznost_osobina.png)

**📝 U `odgovori.csv`:**

```csv
dan5_datum_preseka,2011-09-10
dan5_udeo_vratili_procenat,43.6
dan5_tacnost_osnovne_linije,56.5
dan5_auc_najboljeg_modela,0.805
```

---

# DAN 6 — Napredna tema

Ovde **namerno dobijate manje uputstava**. Birate jednu od tri teme.

## Opcija A — Otkazivanja *(najlakša, najzahvalnija)*

```python
otk = pd.read_parquet("podaci/otkazivanja.parquet")
otk["TotalPrice"] = otk.Quantity * otk.UnitPrice
```

Odgovorite na:
1. Kolika je ukupna vrednost otkaza i koliki je to % prihoda?
2. Kako se stopa otkazivanja menja po mesecima? Postavite prag `prosek + 2σ`.
3. Koji artikli imaju najveću **stopu** otkaza (otkazano / ukupno naručeno)?

**✅ PROVERA:** ~8.292 otkazana računa, vrednost ~1.462.051 (**8,6%** prihoda).
Mesec **2011-12** izrazito odskače (~40%).

💡 Nađite grupu artikala sa stopom otkaza **blizu 50%** (keramika, „polkadot"
posuđe). Deluje kao da se svaki drugi naručeni komad vraća.

⚠️ **Ali prvo proverite koliko ZASEBNIH storno-stavki stoji iza te stope:**

```python
otk.groupby("Description").agg(
    Stavki=("Quantity", "size"),
    Kolicina=("Quantity", lambda s: abs(s.sum())),
    Najveca=("Quantity", lambda s: abs(s).max()))
```

Kod nekih artikala je **jedna jedina** stavka proizvela „stopu od 50%". Jedan
poništen veleprodajni nalog i sedam nezavisnih reklamacija daju isti procenat,
a **potpuno različit zaključak** — u prvom slučaju problem je u procesu
odobravanja naloga, u drugom u kvalitetu proizvoda.

> **Pravilo koje ovde učite:** procenat bez imenioca vara. Uz svaku stopu uvek
> prikažite i **broj događaja** iz kog je izračunata.

🖼 ![Otkazivanja](resenje/izlaz/grafovi/dan6a_stopa_otkazivanja.png)

## Opcija B — Prognoza prodaje *(najteža)*

⚠️ Podelu train/test radite **po vremenu**, ne nasumično — inače model uči iz
budućnosti.

⚠️ Ako koristite lag osobine („prihod pre 7 dana"), ne možete predvideti 30 dana
unapred. Razmislite koje osobine se mogu izračunati za **budući** datum —
kalendarske mogu (dan u nedelji, mesec, trend).

**✅ PROVERA:** naivni model MAPE ~33%, najbolji ~27%.

🖼 ![Prognoza](resenje/izlaz/grafovi/dan6b_prognoza.png)

## Opcija C — Preporuke *(najzabavnija)*

Za 300 najčešćih artikala izračunajte koliko puta se svaki par pojavljuje na
istom računu, pa **support**, **confidence** i **lift**.

⚠️ Sa svih 4.620 artikala ima 21 milion parova. Ograničite se.

**✅ PROVERA:** ~28.000 parova sa lift > 1; najjači par ima lift ~37.

💡 Pogledajte najjače parove — biće to **varijante istog proizvoda u drugoj boji**
(crvene i plave šoljice). Kupci kupuju setove.

🖼 ![Pravila](resenje/izlaz/grafovi/dan6c_pravila.png)

---

# DAN 7 — Izveštaj

**Pišete za nekoga ko nije video vaš kod.** Ako vaš mentor ne razume izveštaj,
izveštaj nije dobar.

## Obavezna struktura

| Sekcija | Sadržaj |
|---|---|
| 1. Rezime za menadžment | 5 nalaza, svaki **jedna rečenica sa brojem** |
| 2. Podaci i priprema | šta ste dobili, šta izbacili, koliko ostalo |
| 3. Nalazi | 4–6 najvažnijih, svaki sa **jednim** grafikonom |
| 4. Model | postavka, rezultat, šta znači |
| 5. Preporuke | 5–7 konkretnih akcija, svaka vezana za nalaz |
| 6. Ograničenja | šta vaša analiza **ne** može da tvrdi |

💡 **Rezime pišete POSLEDNJI, a stavljate PRVI.**

⚠️ **Sekcija „Ograničenja" nije priznanje slabosti.** To je znak da znate dokle
vaš zaključak važi. Analiza bez ograničenja deluje neozbiljno.

Primeri ograničenja za ovaj zadatak:
- 23% redova nema šifru kupca → analiza kupaca pokriva 5.852 identifikovana
- nema podataka o troškovima → govorimo o prihodu, ne o profitu
- podaci su iz 2009–2011, jedna firma, jedna delatnost
- model kaže **ko će se vratiti**, ne kaže da će ga kupon vratiti

## Kako izgleda gotov izveštaj

Pogledajte `resenje/izlaz/IZVESTAJ.md` — to je primer dužine i tona koji se
očekuje. **Nemojte ga prepisivati**, vaši nalazi i formulacije treba da budu vaši.

---

# Sve kontrolne tačke na jednom mestu

| Dan | Pokazatelj | Vrednost |
|---|---|---|
| 1 | Sirovih redova | 1.067.371 |
| 1 | Listovi se preklapaju | 01–09.12.2010, **1.088 računa** |
| 1 | Uklonjeno duplikata | 34.335 |
| 1 | Otkazanih stavki | 19.104 (8.292 računa) |
| 1 | Bez CustomerID | 234.437 |
| 1 | **Redova posle čišćenja** | **776.579** (72,8%) |
| 1 | Kupaca / računa / artikala | 5.852 / 36.594 / 4.620 |
| 1 | Ukupan prihod | 17.068.583 |
| 2 | TotalPrice medijana / prosek | 12,45 / 21,98 |
| 2 | Udeo UK | 83,7% |
| 2 | Najjači pun mesec | 2010-11 (1.155.978) |
| 2 | Najslabiji mesec | 2011-02 (442.294) |
| 2 | Udeo subote | 0,06% |
| 3 | Prvi po prihodu | REGENCY CAKESTAND 3 TIER |
| 3 | Prvi po količini | WORLD WAR 2 GLIDERS |
| 3 | Pareto | 21,7% asortimana → 80% prihoda |
| 3 | Recency / Frequency / Monetary — medijane | 95 / 3 / 856 |
| 3 | Šampioni | 14,2% kupaca → 51,4% prihoda |
| 4 | Elbow predlaže | k = 5 |
| 4 | Najbolja silueta | k = 2 (0,444) |
| 4 | Najvredniji klaster | 7,8% kupaca → 54,9% prihoda |
| 5 | AUC pokvarenog modela | 0,9998 |
| 5 | Datum preseka | 2011-09-10 |
| 5 | Kupaca u modelu / vratilo se | 5.256 / 43,6% |
| 5 | Osnovna linija — tačnost | 56,5% |
| 5 | Logistička / Random Forest — AUC | 0,795 / 0,805 |
| 6A | Otkazi kao % prihoda | 8,6% |
| 6B | MAPE naivni / najbolji | 33,5% / 26,9% |
| 6C | Najveći lift | ~37 |

---

# Kad nešto ne radi — simptom → uzrok

| Simptom | Verovatan uzrok | Rešenje |
|---|---|---|
| Imam **525.461** red, ne milion | Pročitan samo prvi list | `pd.read_excel(..., sheet_name=None)` |
| Imam **~810.000** redova posle čišćenja | Duplikati nisu uklonjeni | `drop_duplicates(subset=poslovne)` |
| `ArrowInvalid: cannot mix types` | `InvoiceNo`/`StockCode` su int i str | `.astype(str)` pre snimanja |
| Medijana Recency je **preko 1.000** | Korišćen `datetime.now()` | `df.InvoiceDate.max() + 1 dan` |
| Medijana Frequency je **preko 20** | `count()` umesto `nunique()` | `("InvoiceNo", "nunique")` |
| `qcut`: `Bin edges must be unique` | Previše istih vrednosti | `rank(method="first")` pre `qcut` |
| Histogram je jedan stubić | Asimetrična raspodela — **nije greška** | `ax.set_xscale("log")` |
| Na grafu se vidi samo UK | UK je 84% prihoda | Dva grafika: sa i bez UK |
| „Prodaja je pala 55% u decembru 2011" | Mesec je **nepotpun** | Isključite ga iz poređenja |
| Jedan klaster ima 3 kupca | Nema log/standardizacije | `np.log1p` + `StandardScaler` |
| **AUC preko 0,95** | ⭐ **Curenje podataka** | Osobine samo pre datuma preseka |
| Model ima tačnost 56% i odziv 0 | Predviđa samo većinsku klasu | `class_weight="balanced"` |

---

## Poslednji savet

Kad zapnete — **prvo pokušajte sami 15 minuta**, pa tek onda pogledajte
`resenje/src/`. Rešenje koje pročitate pre nego što ste pokušali ne nauči vas
ništa.

I zapamtite pravilo koje se ponavlja kroz ceo zadatak:

> **Graf bez zaključka ne vredi ništa.**
> Uz svaki grafikon ide bar jedna rečenica šta se vidi i šta to znači za posao.
