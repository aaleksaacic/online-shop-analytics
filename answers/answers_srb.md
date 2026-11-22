# Odgovori na pitanja za Dan 1 — Učitavanje i čišćenje podataka

Odgovori su formulisani na osnovu koda, izlaza i rezultata obrade iz sveske notebooks/01_cleaning.ipynb i prateće dokumentacije zadatka.


## 1. Upoznavanje sa podacima (Zadatak 1.2)

### Pitanje 1.2.1: Koliko redova i kolona ima skup?
Sirovi skup podataka (nakon spajanja dva radna lista "Year 2009-2010" i "Year 2010-2011") sadrži ukupno 1.067.371 red i 8 originalnih kolona (odnosno 9 kolona ukoliko se uračuna i kolona "List" dodata radi praćenja porekla radnog lista).
- List "Year 2009-2010": 525.461 redova
- List "Year 2010-2011": 541.910 redova
Ukupno: 525.461 + 541.910 = 1.067.371 redova.

### Pitanje 1.2.2: Koje kolone imaju nedostajuće vrednosti i koliko?
U sirovim podacima nedostajuće vrednosti postoje u dve kolone:
- Customer ID: 243.007 nedostajućih vrednosti (22,77% svih redova u skupu).
- Description: 4.382 nedostajuće vrednosti u sirovom Excel fajlu (u svesci se nakon pretvaranja u string vode kao tekstualna vrednost 'nan').
Ostale kolone (Invoice, StockCode, Quantity, InvoiceDate, Price, Country) nemaju nedostajuće vrednosti (0 null vrednosti).

### Pitanje 1.2.3: Pogledajte min za Quantity i Price u .describe(). Šta vidite i kako objašnjavate negativne vrednosti?
Uvidom u deskriptivnu statistiku primećuju se ekstremne negativne vrednosti:
- Za Quantity: minimalna vrednost iznosi -80.995 (dok je maksimalna +80.995).
- Za Price: minimalna vrednost iznosi -53.594,36 (dok je maksimalna 38.970,00).

Objašnjenje:
- Negativne vrednosti u koloni Quantity predstavljaju otkazane porudžbine, stornirane transakcije i povrate robe od strane kupaca, kao i magacinska usklađivanja. Sve ove porudžbine imaju broj fakture koji počinje slovom "C" (Cancelled).
- Negativne vrednosti u koloni Price ne predstavljaju cene artikala, već administrativna i računovodstvena podešavanja i korekcije dugovanja (npr. stavka "Adjust bad debt"), koje unose administratori sistema radi poravnanja bilansa.

### Pitanje 1.2.4: Kolone se zovu Invoice, Price, Customer ID. U većini tutorijala na internetu zovu se InvoiceNo, UnitPrice, CustomerID. Preimenujte ih na duži oblik — zašto je dobra ideja imati jedan dogovoren skup naziva?
Preimenovanje u dogovoreni skup naziva je dobra inženjerska praksa iz sledećih razloga:
1. Uklanjanje razmaka iz naziva ("Customer ID" u "CustomerID") omogućava direktan pristup kolonama preko atributa u biblioteci pandas (npr. df.CustomerID) i sprečava greške pri indeksiranju.
2. Preciznost i nedvosmislenost: naziv "UnitPrice" jasno naglašava da je reč o jediničnoj ceni pojedinačnog artikla, a ne o ukupnoj ceni stavke ili računa.
3. Standardizacija i kompatibilnost: upotreba ustaljene konvencije omogućava lakše poređenje koda sa industrijskim rešenjima, ponovnu iskoristivost postojećih skripti i jednostavniju saradnju među članovima tima.


## 2. Provera preklapanja listova (Zadatak 1.3)

### Pitanje 1.3.1: Da li se periodi preklapaju? U kom tačno intervalu?
Da, vremenski periodi dva radna lista se preklapaju.
- List "Year 2009-2010" obuhvata period od 2009-12-01 07:45:00 do 2010-12-09 20:01:00.
- List "Year 2010-2011" obuhvata period od 2010-12-01 08:26:00 do 2011-12-09 12:50:00.
Preklapanje traje tačno 9 dana, u intervalu od 01.12.2010. do 09.12.2010. godine.

### Pitanje 1.3.2: Koliko računa postoji u oba lista?
U oba lista se pojavljuje tačno 1.088 identičnih brojeva računa (InvoiceNo). Ovi računi obuhvataju ukupno 45.046 redova podataka raspoređenih u oba lista.

### Pitanje 1.3.3: Da su ti redovi ostali duplirani, šta bi se desilo sa prikazanom prodajom za decembar 2010?
Da duplikati nisu eliminisani, prodaja za prvih devet dana decembra 2010. godine bila bi prikazana dva puta, odnosno promet za decembar 2010. bio bi veštački uvećan za skoro 100% u tom periodu. To bi rezultovalo lažnim prikazom enormnog skoka prodaje, dalo pogrešnu sliku o sezonalnosti i iskrivilo sve finansijske pokazatelje poslovanja.


## 3. Završna pitanja Dana 1 (Pitanja za notebook)

### Pitanje 3.1: Koliko procenata podataka ste izbacili? Da li je to previše?
Tokom procesa čišćenja zabeležen je sledeći levak:
- Početni broj redova: 1.067.371
- Uklonjeni identični duplikati: 34.335
- Izdvojena otkazivanja (fakture sa 'C'): 19.104
- Uklonjeni redovi bez CustomerID: 234.437
- Uklonjene službene šifre: 2.856
- Uklonjeno Quantity <= 0: 0
- Uklonjeno UnitPrice <= 0: 60
- Preostalo čistih redova: 776.579

Ukupno je uklonjeno 290.792 reda, što iznosi tačno 27,24% početnog skupa (zadržano je 72,76% originalnih podataka).

Ovaj udeo nije prevelik i u potpunosti je metodološki opravdan. Najveći deo uklonjenih zapisa čine redovi bez identifikacije kupca (preko 234 hiljade) i duplikati nastali preklapanjem listova (preko 34 hiljade). Za potrebe analize ponašanja kupaca, segmentacije i prediktivnog modelovanja, zadržavanje anonimnih ili neispravnih podataka unelo bi jak šum i proizvelo pogrešne zaključke.

### Pitanje 3.2: Redovi bez CustomerID su skoro četvrtina skupa. Zašto ih ipak izbacujemo za analizu kupaca, a ne izbacujemo iz ukupne prodaje?
Razlika potiče iz različitih analitičkih ciljeva:
1. Zašto ih izbacujemo za analizu kupaca: Bez jedinstvenog identifikatora (CustomerID) nemoguće je povezati više kupovina sa istom osobom ili kompanijom. Ne možemo izračunati koliko često kupac dolazi (Frequency), kada je poslednji put kupovao (Recency) niti koliko je ukupno potrošio (Monetary). Ukoliko bismo te transakcije zadržali, svaka anonimna kupovina bi se tretirala kao poseban kupac koji je kupio samo jednom, što bi potpuno iskrivilo distribuciju lojalnosti i segmente kupaca.
2. Zašto ih ne izbacujemo iz ukupne prodaje: Ta roba je zaista naručena, isporučena i novac je naplaćen. Izbacivanje neregistrovanih kupovina iz finansijskih i operativnih izveštaja dovelo bi do podcenjivanja stvarnog prihoda kompanije za preko 20%, što bi dalo netačnu sliku o obimu poslovanja, ukupnom prometu i potražnji za zalihama.

### Pitanje 3.3: Posle uklanjanja otkazivanja, koliko je ostalo redova sa Quantity <= 0? Šta vam taj rezultat govori o tome odakle dolaze negativne količine?
Nakon izdvajanja otkazivanja (računi čiji broj počinje slovom "C"), u koraku filtriranja Quantity <= 0 uklonjeno je tačno 0 redova (ostalo je 0 redova sa Quantity <= 0).

Ovaj nalaz nedvosmisleno potvrđuje da su sve negativne količine u skupu podataka poticale isključivo iz storno faktura (otkazivanja i povrata). U bazi podataka nije bilo neevidentiranih sistemskih grešaka ili anomalija sa negativnim količinama van regularnog procesa storniranja, što znači da je prefiks "C" bio tačan i sveobuhvatan pokazatelj svih storniranih stavki.

---

# Odgovori na pitanja za Dan 2 — Istraživačka analiza podataka (EDA)

Odgovori su formulisani na osnovu koda, izlaza i rezultata vizuelizacija iz sveske notebooks/02_eda.ipynb i prateće dokumentacije zadatka.


## 1. Najveće vrednosti i ekstremi (Zadatak 2.2)

### Pitanje 2.2: Da li su najveće vrednosti po TotalPrice greške u podacima ili stvarne kupovine? Kako ste zaključili? Da li ih treba izbaciti?
Uvidom u 10 najvećih stavki po vrednosti TotalPrice (od kojih najveća iznosi 168.469,60 £ za artikal "PAPER CRAFT , LITTLE BIRDIE", a druga 77.183,60 £ za "MEDIUM CERAMIC TOP STORAGE JAR"):

1. Da li su to greške ili stvarne kupovine:
U pitanju su stvarne veleprodajne (B2B) kupovine, a ne tehničke greške u podacima.
2. Kako smo to zaključili:
- Jedinične cene (UnitPrice) ovih artikala su potpuno uobičajene i realne (npr. 1,04 £, 1,66 £, 2,08 £, 2,10 £ po komadu). Ekstremni TotalPrice potiče isključivo od ogromnih naručenih količina (Quantity od 2.400 do čak 80.995 komada).
- Nazivi artikala odgovaraju regularnom asortimanu kompanije (ukrasne teglice, sezonska dekoracija, lampice).
- Brojevi računa i identifikatori kupaca su validni. Nekoliko kupaca pravi ponovljene velike porudžbine (npr. kupac 15749 je naručio po 3.114 komada istog artikla i u januaru i u aprilu 2011; kupac 15838 naručuje hiljade svećnjaka pred božićnu sezonu; kupac 14646 iz Holandije je redovni veliki distributer).
3. Da li ih treba izbaciti:
Nikako. Njihovo brisanje bi predstavljalo ozbiljnu metodološku grešku. U veleprodajnom poslovanju manji broj velikih klijenata nosi ogroman udeo prihoda. Uklanjanjem ovih zapisa izbrisali bismo ključne kupce i realan promet preduzeća. Ovi ekstremi se ne eliminišu, već se pri statističkom modelovanju tretiraju odgovarajućim tehnikama (poput logaritamske transformacije ili robusnih mera).


## 2. Završna pitanja Dana 2 (Pitanja za notebook)

### Pitanje 2.1: Zašto je prosek TotalPrice skoro dvostruko veći od medijane? Koju od te dve mere biste koristili u izveštaju za direktora i zašto?
1. Zašto se prosek i medijana razlikuju:
U podacima je medijana TotalPrice 12,45 £, dok je aritmetička sredina (prosek) 21,98 £ (prosek je 1,77 puta veći od medijane). Razlog je izrazita pozitivna asimetrija (desna nagnutost / right-skewness) distribucije. Većina kupovina otpada na male i srednje korpe (50% svih stavki vredi manje od 12,45 £, a 75% manje od 24,90 £), ali manji broj ogromnih veleprodajnih porudžbina (iznosi do nekoliko desetina i stotina hiljada funti) snažno vuče prosek naviše.
2. Koju meru koristiti u izveštaju za direktora i zašto:
Preporučljivo je navesti obe mere, ali uz jasno objašnjenje njihove poslovne funkcije:
- Medijana (12,45 £) je ključna mera za opisivanje „tipične“ kupovine prosečnog pojedinačnog kupca, jer je otporna na ekstremne vrednosti i realno oslikava svakodnevnu potrošnju.
- Prosek (21,98 £) je koristan isključivo kao finansijski pokazatelj prinosa po stavci (Ukupan prihod = Broj stavki * Prosek).
Ukoliko bi direktoru bio predočen samo prosek od skoro 22 £, stvorila bi se lažna slika da tipična stavka u korpi vredi toliko, iako više od 70% svih prodatih stavki ima pojedinačnu vrednost ispod tog iznosa. Zbog toga se kao reprezentativna mera ponašanja kupaca uvek ističe medijana.

### Pitanje 2.2: Irska i Holandija su visoko po prihodu ali imaju vrlo malo kupaca. Izračunajte prosečnu korpu po državi. Šta zaključujete o tome kakvi su to kupci?
Podaci o prodaji po državama pokazuju:
- Velika Britanija: 14.288.773,50 £ prihoda (83,71%), 5.334 kupca, 33.361 račun, prosečna korpa iznosi 428,31 £.
- Irska (EIRE): 586.626,08 £ prihoda (3,44%), svega 3 kupca, 528 računa, prosečna korpa iznosi 1.111,03 £ (preko 195.000 £ prometa po kupcu).
- Holandija: 549.773,41 £ prihoda (3,22%), svega 22 kupca, 216 računa, prosečna korpa iznosi čak 2.545,25 £ (preko 24.000 £ prometa po kupcu).
- Australija: 167.800,01 £ prihoda (0,98%), 15 kupaca, 89 računa, prosečna korpa iznosi 1.885,39 £.

Zaključak o profilu kupaca:
Ovi kupci nisu individualni maloprodajni potrošači (B2C), već veliki B2B veleprodajni distributeri i uvoznici. U Irskoj i Holandiji kompanija ne posluje kao standardna internet prodavnica sa velikim brojem malih porudžbina, već funkcioniše kao ključni inostrani dobavljač za nekolicinu regionalnih trgovačkih lanaca ili partnerskih preduzeća koja robu uvoze na veliko za dalju distribuciju.

### Pitanje 2.3: Da li postoji sezonalnost? Da li se obrazac ponavlja obe godine? Zašto je ponavljanje bitno za zaključak?
1. Da li postoji sezonalnost i da li se ponavlja:
Da, postoji snažna i pravilna sezonalnost koja se identično ponavlja tokom obe posmatrane godine (2010. i 2011.). Tokom prvog kvartala (posebno u januaru i februaru) prodaja je na godišnjem minimumu (npr. februar 2011. sa 442.294 £). Od kraja leta i tokom jeseni (septembar, oktobar) promet ubrzano raste, dostižući apsolutni godišnji vrhunac u novembru (novembar 2010. sa 1.155.978 £ i novembar 2011. sa 1.136.534 £).
2. Zašto je ponavljanje bitno za zaključak:
Ukoliko bi se ovakav skok zabeležio samo u jednoj godini, postojao bi rizik da se radi o izolovanom incidentu, uspešnoj jednokratnoj marketinškoj kampanji ili privremenom makroekonomskom faktoru. Ponavljanje identičnog obrasca u dve uzastopne godine pruža empirijski dokaz da je reč o stabilnom sezonskom ciklusu. Za poslovanje ovo znači da se nabavka robe, logistika i popunjavanje skladišta moraju planirati mesecima unapred (u periodu avgust–oktobar) kako bi se dočekao novembarski talas potražnje.

### Pitanje 2.4: Šta se dešava subotom? Da li je to greška u podacima?
1. Šta se dešava subotom:
Subotom je zabeležen promet od svega 9.803,05 £ za ceo dvogodišnji period, što čini svega 0,057% (oko 0,06%) ukupnog prihoda kompanije.
2. Da li je to greška u podacima:
Nije u pitanju greška u podacima. Subota je zvanično neradni dan za operativne službe, skladište i otpremu kompanije. Zabeleženi simboličan promet subotom predstavlja automatske online porudžbine pristigle preko veb sajta koje su zabeležene u sistemu, ili povremene administrativne unose. Kompanija funkcioniše po poslovnom modelu radne nedelje od nedelje do petka (petak popodne i subota su neradni, a obrada počinje u nedelju za isporuku početkom sedmice), što je čest operativni model u britanskoj trgovini i logistici.

### Pitanje 2.5: Pogledajte heatmapu. U koje doba dana se kupuje? Šta vam to govori o tome da li su kupci fizička lica ili firme?
1. U koje doba dana se kupuje:
Heatmapa transakcija po danima i satima pokazuje izrazitu koncentraciju kupovina radnim danima (od ponedeljka do petka) u intervalu između 09:00 i 16:00 časova, sa izrazitim vrhom aktivnosti između 11:00 i 14:00 časova. Pre 08:00 časova ujutru i nakon 17:00 časova popodne promet je zanemarljiv, dok su večernji i noćni sati potpuno prazni.
2. Šta to govori o profilu kupaca (fizička lica ili firme):
Ovakav vremenski obrazac jasno potvrđuje da su dominantni kupci pravna lica (firme, prodavnice, preduzetnici — B2B segment). Fizička lica (B2C) u elektronskoj trgovini kupuju dominantno u slobodno vreme — uveče (od 18:00 do 23:00) i tokom vikenda. Činjenica da se preko 95% kupovina obavlja isključivo unutar standardnog radnog vremena radnim danima, zajedno sa velikim količinama artikala po narudžbini, nedvosmisleno ukazuje na to da zaposleni u preduzećima naručuju robu tokom svog radnog vremena.
