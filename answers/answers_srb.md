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

---

# Odgovori na pitanja za Dan 3 — Proizvodi i RFM analiza

Odgovori su formulisani na osnovu koda, izlaza i rezultata segmentacije iz sveske notebooks/03_rfm.ipynb i prateće dokumentacije zadatka.


## 1. Najprodavaniji proizvodi (Zadatak 3.1)

### Pitanje 3.1: Koliko se proizvoda pojavljuje na obe liste (po količini i po prihodu)? Šta to znači?
Na obe top 10 liste (po ukupnoj prodatoj količini i po ukupno ostvarenom prihodu) pojavljuje se tačno 5 proizvoda:
1. WHITE HANGING HEART T-LIGHT HOLDER (StockCode 85123A) — 247.048,01 £ prihoda (2. mesto), 91.757 komada (2. mesto), 1.490 kupaca
2. PAPER CRAFT , LITTLE BIRDIE (StockCode 23843) — 168.469,60 £ prihoda (3. mesto), 80.995 komada (3. mesto), 1 kupac
3. JUMBO BAG RED RETROSPOT (StockCode 85099B) — 134.307,44 £ prihoda (4. mesto), 74.224 komada (6. mesto), 860 kupaca
4. ASSORTED COLOUR BIRD ORNAMENT (StockCode 84879) — 124.351,86 £ prihoda (5. mesto), 78.234 komada (4. mesto), 1.010 kupaca
5. MEDIUM CERAMIC TOP STORAGE JAR (StockCode 23166) — 81.416,73 £ prihoda (7. mesto), 77.916 komada (5. mesto), 138 kupaca

Šta to znači:
Ovih 5 artikala predstavljaju apsolutne stubove asortimana („zvezde“ poslovanja). Oni uspevaju da ostvare redak balans: imaju izuzetno visoku i stabilnu potražnju u pogledu obima (količine), a istovremeno drže stabilnu jediničnu cenu koja generiše ogroman novčani priliv. Za razliku od njih, artikli koji su samo na listi količina (poput WORLD WAR 2 GLIDERS sa preko 105.000 komada, ali svega ~24.000 £ prihoda) predstavljaju jeftine impulsne proizvode sa niskom maržom, dok su artikli samo na listi prihoda (poput REGENCY CAKESTAND 3 TIER sa ~277.656 £) skuplji, premijum proizvodi sa umerenijim brojem prodatih jedinica.


## 2. Pareto analiza (Zadatak 3.2)

### Pitanje 3.2: Koliko procenata asortimana donosi 80% prihoda? Da li važi pravilo 80/20?
1. Procenat asortimana za 80% prihoda:
Tačno 1.146 proizvoda, što čini 21,7% ukupnog asortimana (posmatrano po parovima StockCode + Description), donosi 80% ukupnog prihoda kompanije.
2. Da li važi pravilo 80/20:
Da, Paretovo pravilo 80/20 važi u izuzetno preciznoj meri (~22% asortimana donosi 80% prihoda). To pokazuje visoku koncentraciju prihoda na gornjoj petini kataloga, dok preostalih ~78% asortimana čini takozvani „dugi rep“ (long tail) koji generiše svega 20% prihoda. Za menadžment ovo znači da fokus nabavke, skladišnog prostora i pregovora sa dobavljačima mora biti usmeren na ovu ključnu petinu artikala.


## 3. Završna pitanja Dana 3 (Pitanja za notebook)

### Pitanje 3.1: Zašto se liste „top po količini“ i „top po prihodu“ toliko razlikuju?
Liste se razlikuju zbog velike varijacije u jediničnim cenama artikala:
- Top lista po količini sadrži jeftine, sitne proizvode (poput maketa aviona WORLD WAR 2 GLIDERS ili papirnih korpica za kolače) koji koštaju svega nekoliko desetina penija ili oko jedne funte. Iako se prodaju u desetinama i stotinama hiljada komada, njihov finansijski doprinos je relativno skroman.
- Top lista po prihodu predvođena je artiklima sa znatno višom jediničnom cenom (poput stalaka za torte REGENCY CAKESTAND 3 TIER sa cenom od 10–12 £). Manji broj prodatih komada skupljeg artikla generiše neuporedivo veći prihod nego ogroman broj sitnih impulsnih artikala.

### Pitanje 3.2: Grupisanje po StockCode daje ~4.620 proizvoda, a po StockCode + Description ~5.274. Odakle razlika? Šta to znači za pouzdanost naziva proizvoda i koji biste ključ koristili?
1. Odakle razlika:
Razlika potiče od činjenice da za 598 šifara proizvoda (StockCode) postoji više različitih tekstualnih opisa (Description) unetih kroz vreme. Uzroci su:
- Tipografske greške i slovne razlike (npr. SPUNGE umesto SPONGE, prisustvo ili odsustvo zareza, razmaka i kosih crta).
- Promene marketinškog naziva istog proizvoda tokom posmatranog dvogodišnjeg perioda (npr. WHITE SPOTS u odnosu na POLKADOT).
- Nekonzistentno pisanje velikih i malih slova ili skraćivanje naziva prilikom unosa novih serija u skladišni sistem.
2. Šta to znači za pouzdanost naziva proizvoda:
Tekstualni opis (Description) je operativno nepouzdan kao primarni identifikator artikla jer je podložan ljudskim greškama i promenama tokom vremena.
3. Koji ključ koristiti:
Kao jedinstveni ključ za analize i grupisanja obavezno treba koristiti šifru artikla (`StockCode`). Ukoliko je u izveštaju potreban čitljiv naziv, uzima se najnoviji ili najčešće korišćeni Description povezan sa tim konkretnim StockCode-om.

### Pitanje 3.3: Jedan proizvod u top listi ima samo 1 kupca. Nađite ga. Da li je to normalno i šta bi trebalo uraditi sa njim u analizi?
1. Koji je to proizvod:
U pitanju je artikal `PAPER CRAFT , LITTLE BIRDIE` (StockCode `23843`). Prodat je u količini od 80.995 komada, doneo je prihod od 168.469,60 £, a kupio ga je samo jedan jedini kupac (CustomerID `16446`) na jednoj fakturi (`581483`) dana 09.12.2011.
2. Da li je to normalno:
Ne, to predstavlja ekstreman komercijalni izuzetak (anomaliju) — najverovatnije namenski veleprodajni ugovor, vanrednu korporativnu narudžbinu ili rasprodaju celokupne serije jednom kupcu. Nije uobičajeno da treći najvredniji proizvod u istoriji kompanije duguje celokupan promet jednoj jedinoj transakciji.
3. Šta uraditi sa njim u analizi:
- U finansijskim izveštajima o ukupnom prihodu kompanije transakciju treba zadržati jer novac jeste realno naplaćen.
- U analizama asortimana, predviđanju tražnje i sistemima za preporuku artikala (Market Basket analizi), ovaj artikal treba izolovati ili posmatrati odvojeno. U suprotnom, on veštački stvara privid da je reč o stalnom masovnom hit-proizvodu sa redovnom tražnjom, što bi dovelo do pogrešnih odluka o nabavci i planiranju zaliha.

### Pitanje 3.4: Medijana potrošnje je ~856, a prosek ~2.917. Objasnite razliku.
U RFM tabeli za 5.852 kupca medijana potrošnje (Monetary) iznosi 856,02 £, dok je prosek (aritmetička sredina) 2.916,71 £ (prosek je čak 3,4 puta veći od medijane).

Razlog ove razlike je ekstremna asimetrija distribucije potrošnje kupaca:
- Polovina svih identifikovanih kupaca (50%) tokom celog dvogodišnjeg perioda ukupno potroši manje od 856 £.
- S druge strane, mali broj velikih B2B veleprodajnih kupaca i distributera ostvaruje stotine hiljada funti prometa (najveći kupac ima zabeležen promet od preko 580.000 £). Ovi klijenti vuku aritmetički prosek visoko naviše.
Medijana oslikava realnu potrošnju tipičnog kupca, dok je prosek samo zbir ukupnog prihoda podeljen brojem kupaca i ne opisuje prosečnog pojedinca.

### Pitanje 3.5: Koji segment ima najviše kupaca? Koji donosi najviše novca? Da li je to isti segment? Šta ta razlika znači za marketinški budžet?
1. Podaci o segmentima:
- Segment sa najviše kupaca: `Lost` (Izgubljeni kupci) sa 1.510 kupaca, što čini 25,8% ukupne baze klijenata. Sledeći po veličini je `Loyal Customers` sa 1.151 kupcem (19,7%).
- Segment koji donosi najviše novca: `Champions` (Šampioni) sa 8.781.126,40 £, što čini čak 51,4% ukupnog prihoda kompanije. Drugi po prihodu je `Loyal Customers` sa 4.658.851,80 £ (27,3%).
2. Da li je to isti segment:
Ne, to su potpuno različiti segmenti. Segment `Lost` obuhvata više od četvrtine baze klijenata (25,8%), ali generiše svega 3,6% prihoda. Sa druge strane, segment `Champions` obuhvata svega 14,2% baze (830 kupaca), a donosi preko polovine celokupnog prihoda.
3. Šta ta razlika znači za marketinški budžet:
Budžet se mora raspodeliti asimetrično na osnovu povrata investicije (ROI):
- Primarni deo budžeta (kroz VIP programe, posvećene menadžere naloga, specijalne rabate i personalizovane uslove) mora biti usmeren na zadržavanje i nagrađivanje segmenata `Champions` i `Loyal Customers`. Ovi segmenti zajedno čine ~34% kupaca, ali generišu skoro 79% ukupnog prihoda preduzeća. Gubitak čak i nekoliko kupaca iz ovog segmenta predstavlja direktan udarac na opstanak poslovanja.
- Trošenje skupih marketinških resursa na reaktivaciju masovnog segmenta `Lost` je neisplativo. Za njih treba primeniti jeftine, automatizovane mejling kampanje, dok se direktna i skuplja reaktivacija fokusira na vrednije klijente koji su u riziku (`Cannot Lose Them` i `At Risk`).

### Pitanje 3.6: Za segment „Ne smemo ih izgubiti“ — opišite tipičnog kupca i predložite jednu konkretnu akciju.
1. Opis tipičnog kupca:
Segment `Cannot Lose Them` obuhvata 72 klijenta (1,2% baze) koji su istorijski generisali preko pola miliona funti (503.405,10 £, odnosno 2,9% prihoda), sa prosečnom potrošnjom od blizu 7.000 £ po kupcu.
Tipičan profil: To je klijent sa najvišom ocenom frekvencije i potrošnje (F=5, visoka monetarna vrednost), što znači da je u prošlosti bio među najboljim i najredovnijim kupcima kompanije. Međutim, njegova ocena svežine je najniža (R=1 ili 2) — nije naručio ništa mesecima. Ovaj kupac je na pragu potpunog odlaska ili je već počeo da sarađuje sa konkurentskim dobavljačem.
2. Predlog konkretne akcije:
Ne slati generičke automatizovane promotivne mejlove ili vaučere za popust. Neophodan je direktan kontakt od strane direktora prodaje ili ključnog menadžera naloga (telefonski poziv ili lični sastanak). Cilj razgovora je sprovođenje strukturiranog intervjua: utvrditi razlog prekida saradnje (da li je u pitanju nezadovoljstvo rokovima isporuke, kvalitetom, cenovnom politikom ili promena potreba), ponuditi rešavanje problema kroz prilagođene komercijalne uslove (npr. poseban količinski rabat ili odloženo plaćanje) i obnoviti ugovor o snabdevanju.
