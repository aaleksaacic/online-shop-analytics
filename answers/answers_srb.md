# Odgovori na pitanja za Dan 1 — Učitavanje i čišćenje podataka

Odgovori su formulisani na osnovu koda, izlaza i rezultata obrade iz sveske notebooks/01_cleaning.ipynb i prateće dokumentacije zadatka.


## 1. Upoznavanje sa podacima (Zadatak 1.2)

### Pitanje 1.2.1: Koliko redova i kolona ima skup?
Sirovi skup podataka (nakon spajanja dva radna lista "Year 2009-2010" i "Year 2010-2011") sadrži ukupno 1.067.371 red i 8 originalnih kolona (odnosno 9 kolona ukoliko se uračuna i kolona "List" dodata radi praćenja porekla radnog lista).
- List "Year 2009-2010": 525.461 redova
- List "Year 2010-2011": 541.910 redova
- Ukupno: 525.461 + 541.910 = 1.067.371 redova.

### Pitanje 1.2.2: Koje kolone imaju nedostajuće vrednosti i koliko?
U sirovim podacima nedostajuće vrednosti postoje u dve kolone:
- Customer ID: 243.007 nedostajućih vrednosti (22,77% svih redova u skupu).
- Description: 4.382 nedostajuće vrednosti u sirovom Excel fajlu (u svesci se nakon pretvaranja u string vode kao tekstualna vrednost 'nan').
Ostale kolone (Invoice, StockCode, Quantity, InvoiceDate, Price, Country) nemaju nedostajuće vrednosti (0 null vrednosti).

### Pitanje 1.2.3: Pogledajte min za Quantity i Price u .describe(). Sta vidite i kako objasnjavate negativne vrednosti?
Uvidom u deskriptivnu statistiku primećuju se ekstremne negativne vrednosti:
- Za Quantity: minimalna vrednost iznosi -80.995 (dok je maksimalna +80.995).
- Za Price: minimalna vrednost iznosi -53.594,36 (dok je maksimalna 38.970,00).

Objašnjenje:
- Negativne vrednosti u koloni Quantity predstavljaju otkazane porudzbine, stornirane transakcije i povrate robe od strane kupaca, kao i magacinska uskladjivanja. Sve ove porudzbine imaju broj fakture koji pocinje slovom "C" (Cancelled).
- Negativne vrednosti u koloni Price ne predstavljaju cene artikala, vec administrativna i računovodstvena podešavanja i korekcije dugovanja (npr. stavka "Adjust bad debt"), koje unose administratori sistema radi poravnanja bilansa.

### Pitanje 1.2.4: Kolone se zovu Invoice, Price, Customer ID. U većini tutorijala na internetu zovu se InvoiceNo, UnitPrice, CustomerID. Preimenujte ih na duži oblik — zasto je dobra ideja imati jedan dogovoren skup naziva?
Preimenovanje u dogovoreni skup naziva je dobra inzenjerska praksa iz sledećih razloga:
1. Uklanjanje razmaka iz naziva ("Customer ID" u "CustomerID") omogucava direktan pristup kolonama preko atributa u biblioteci pandas (npr. df.CustomerID) i sprecava greske pri indeksiranju.
2. Preciznost i nedvosmislenost: naziv "UnitPrice" jasno naglašava da je rec o jediničnoj ceni pojedinačnog artikla, a ne o ukupnoj ceni stavke ili računa.
3. Standardizacija i kompatibilnost: upotreba ustaljene konvencije omogućava lakše poredjenje koda sa industrijskim rešenjima, ponovnu iskoristivost postojećih skripti i jednostavniju saradnju medju članovima tima.


## 2. Provera preklapanja listova (Zadatak 1.3)

### Pitanje 1.3.1: Da li se periodi preklapaju? U kom tačno intervalu?
Da, vremenski periodi dva radna lista se preklapaju.
- List "Year 2009-2010" obuhvata period od 2009-12-01 07:45:00 do 2010-12-09 20:01:00.
- List "Year 2010-2011" obuhvata period od 2010-12-01 08:26:00 do 2011-12-09 12:50:00.
Preklapanje traje tačno 9 dana, u intervalu od 01.12.2010. do 09.12.2010. godine.

### Pitanje 1.3.2: Koliko računa postoji u oba lista?
U oba lista se pojavljuje tačno 1.088 identicnih brojeva računa (InvoiceNo). Ovi računi obuhvataju ukupno 45.046 redova podataka rasporedjenih u oba lista.

### Pitanje 1.3.3: Da su ti redovi ostali duplirani, sta bi se desilo sa prikazanom prodajom za decembar 2010?
Da duplikati nisu eliminisani, prodaja za prvih devet dana decembra 2010. godine bila bi prikazana dva puta, odnosno promet za decembar 2010. bio bi veštacki uvećan za skoro 100% u tom periodu. To bi rezultovalo lažnim prikazom enormnog skoka prodaje, dalo pogrešnu sliku o sezonalnosti i iskrivilo sve finansijske pokazatelje poslovanja.


## 3. Završna pitanja Dana 1 (Pitanja za notebook)

### Pitanje 3.1: Koliko procenata podataka ste izbacili? Da li je to previše?
Tokom procesa čisćenja zabeležen je sledeći levak:
- Početni broj redova: 1.067.371
- Uklonjeni identični duplikati: 34.335
- Izdvojena otkazivanja (fakture sa 'C'): 19.104
- Uklonjeni redovi bez CustomerID: 234.437
- Uklonjene službene šifre: 2.856
- Uklonjeno Quantity <= 0: 0
- Uklonjeno UnitPrice <= 0: 60
- Preostalo čistih redova: 776.579

Ukupno je uklonjeno 290.792 reda, sto iznosi tačno 27,24% pocetnog skupa (zadržano je 72,76% originalnih podataka).

Ovaj udeo nije prevelik i u potpunosti je metodološki opravdan. Najveći deo uklonjenih zapisa čine redovi bez identifikacije kupca (preko 234 hiljade) i duplikati nastali preklapanjem listova (preko 34 hiljade). Za potrebe analize ponašanja kupaca, segmentacije i prediktivnog modelovanja, zadržavanje anonimnih ili neispravnih podataka unelo bi jak šum i proizvelo pogrešne zaključke.

### Pitanje 3.2: Redovi bez CustomerID su skoro četvrtina skupa. Zasto ih ipak izbacujemo za analizu kupaca, a ne izbacujemo iz ukupne prodaje?
Razlika potiče iz različitih analitičkih ciljeva:
1. Zašto ih izbacujemo za analizu kupaca: Bez jedinstvenog identifikatora (CustomerID) nemoguće je povezati više kupovina sa istom osobom ili kompanijom. Ne mozemo izracunati koliko često kupac dolazi (Frequency), kada je poslednji put kupovao (Recency) niti koliko je ukupno potrošio (Monetary). Ukoliko bismo te transakcije zadržali, svaka anonimna kupovina bi se tretirala kao poseban kupac koji je kupio samo jednom, što bi potpuno iskrivilo distribuciju lojalnosti i segmente kupaca.
2. Zašto ih ne izbacujemo iz ukupne prodaje: Ta roba je zaista naručena, isporučena i novac je naplaćen. Izbacivanje neregistrovanih kupovina iz finansijskih i operativnih izveštaja dovelo bi do podcenjivanja stvarnog prihoda kompanije za preko 20%, sto bi dalo netačnu sliku o obimu poslovanja, ukupnom prometu i potražnji za zalihama.

### Pitanje 3.3: Posle uklanjanja otkazivanja, koliko je ostalo redova sa Quantity <= 0? Sta vam taj rezultat govori o tome odakle dolaze negativne količine?
Nakon izdvajanja otkazivanja (računi čiji broj pocinje slovom "C"), u koraku filtriranja Quantity <= 0 uklonjeno je tačno 0 redova (ostalo je 0 redova sa Quantity <= 0).

Ovaj nalaz nedvosmisleno potvrdjuje da su sve negativne količine u skupu podataka poticale isključivo iz storno faktura (otkazivanja i povrata). U bazi podataka nije bilo neevidentiranih sistemskih grešaka ili anomalija sa negativnim količinama van regularnog procesa storniranja, sto znači da je prefiks "C" bio tačan i sveobuhvatan pokazatelj svih storniranih stavki.
