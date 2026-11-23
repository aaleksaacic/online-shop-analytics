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

---

# Odgovori na pitanja za Dan 4 — Segmentacija kupaca (K-Means)

Odgovori u nastavku su izvedeni direktno iz programskog koda, izlaznih rezultata izvršavanja i modela klasterizacije u [04_kmeans.ipynb](notebooks/04_kmeans.ipynb) i prateće projektne dokumentacije.


## 1. Priprema podataka (Zadatak 4.1)

### Pitanje 4.1.1: Izračunajte skew() pre i posle log transformacije. Zašto su oba koraka obavezna? Šta bi se konkretno desilo sa klasterima da preskočite standardizaciju?

#### 1. Vrednosti asimetrije (`skew`) pre i posle `np.log1p()`
Koeficijenti asimetrije izmereni na uzorku od 5.852 kupca pre i posle primene logaritamske transformacije:

| Varijabla (Feature) | Asimetrija pre log-a | Asimetrija posle log-a (`np.log1p`) | Efekat smanjenja asimetrije |
|---|---|---|---|
| **Recency** | 0,89 | -0,49 | Umerena pozitivna asimetrija pretvorena u približno simetričnu raspodelu |
| **Frequency** | 12,04 | 1,00 | Izražena pozitivna asimetrija smanjena za 91,7% |
| **Monetary** | 25,33 | 0,27 | Ekstremna eksponencijalna asimetrija smanjena za 98,9% (skoro savršena normalna simetrija) |

#### 2. Zašto su oba koraka (log1p i StandardScaler) obavezna?
Algoritam K-Means u osnovi funkcioniše računajući **Euklidsko rastojanje** u višedimenzionalnom prostoru:
$$d(x, y) = \sqrt{\sum_{i=1}^n (x_i - y_i)^2}$$

S obzirom na to da Euklidsko rastojanje podrazumeva sferični metrički prostor i pretpostavlja izotropnu raspodelu, podaci moraju zadovoljiti dva ključna preduslova:
1. **Logaritamska transformacija (`np.log1p`):** Rešava problem ekstremne asimetrije i sabija raspon ekstremnih vrednosti (engl. *outliers*) koje se prostiru preko više redova veličine (npr. potrošnja od 3,75 £ pa sve do 580.000 £). Bez logaritmovanja, 95% većine kupaca sa manjom potrošnjom bilo bi sabijeno u jedan hiper-zgusnuti klaster, dok bi nekolicina ogromnih veleprodajnih kupaca formirala izolovane, beskorisne klastere od po jednog člana ili potpuno povukla centre klastera ka sebi. Log transformacija prevodi multiplikativne razlike u aditivne linearne skale.
2. **Standardizacija (`StandardScaler`):** Skalira svaku varijablu tako da ima srednju vrednost $\mu = 0$ i standardnu devijaciju $\sigma = 1$. Čak i nakon logaritmovanja, rasponi i varijanse za Recency, Frequency i Monetary nisu identični. StandardScaler obezbeđuje da sve tri dimenzije ravnopravno učestvuju u računanju rastojanja.

#### 3. Šta bi se konkretno desilo sa klasterima da se preskoči standardizacija?
Ukoliko se izostavi `StandardScaler`, računanje Euklidskog rastojanja biva potpuno podređeno varijabli koja ima najveći numerički raspon i varijansu.
- Na sirovim podacima, Monetary se meri desetinama i stotinama hiljada funti, dok se Frequency kreće od 1 do 300, a Recency od 0 do 738. U tom slučaju, rastojanje bi 99,9% zavisilo isključivo od novca, čime bi se K-Means pretvorio u jednodimenzionalno sečenje po potrošnji, a Frequency i Recency bi postali matematički nebitni.
- Čak i kada se primeni `np.log1p` bez standardizacije, raspon log-Recency (od ~0 do 6,6) i log-Monetary (od ~1,5 do 13,3) ostaje veći od raspona log-Frequency (od ~0,69 do 5,7). Varijabla sa većim vrednostima stvara izdužene elipsoidne klastere, dok varijable sa manjim rasponom gube uticaj na formiranje granica segmenata.


## 2. Određivanje broja klastera (Zadatak 4.2 / Pitanje 4.1)

### Pitanje 4.1: Silueta kaže k=2, elbow kaže k=5. Koji biste izabrali i zašto?

#### 1. Empirijski rezultati za $k \in [2, 10]$
Rezultati evaluacije mera kompaktnosti i separacije klastera:

| $k$ | Inercija (Elbow) | Silhouette Score | Poslovna interpretativnost |
|---|---|---|---|
| **k = 2** | 8.562,3 | **0,444** (Globalni maksimum) | Trivijalna binarna podela (Aktivni vs. Neaktivni); minimalna upotrebna vrednost |
| **k = 3** | 6.338,4 | 0,351 | Preširoke grupe; spaja različite kategorije kupaca visoke vrednosti |
| **k = 4** | 4.908,2 | 0,365 | Dobra matematička struktura, ali ne razdvaja nove/povremene od stalnih |
| **k = 5** | **4.090,2** | **0,340** (Optimalni prelom lakta) | **Izabrani model: Visoko primenjiva strateška segmentacija baze** |
| **k = 6** | 3.549,3 | 0,336 | Preterana fragmentacija lojalnih i pasivnih kupaca |
| **k = 7** | 3.181,9 | 0,301 | Opadajući prinosi; koeficijent siluete znatno opada |
| **k = 8** | 2.892,4 | 0,294 | Prenaučenost (overfitting); podgrupe nemaju jasne poslovne akcije |
| **k = 9** | 2.650,1 | 0,285 | Nestabilni centroidi; teška interpretacija |
| **k = 10** | 2.460,3 | 0,284 | Prevelika granulacija za operativno upravljanje |

#### 2. Obrazloženje izbora: Zašto biramo $k = 5$?
Za praktičnu primenu i donošenje poslovnih odluka biramo **$k = 5$** iz sledećih razloga:

1. **Matematička priroda $k = 2$:** 
   Silueta postiže maksimum na $k=2$ (0,444) jer baza ima prirodno snažnu polarnost između onih koji su kupovali skoro i onih koji su odavno prestali. Međutim, podela celokupne baze na samo dve grupe (npr. „Aktivni vs. Neaktivni“ ili „Veliki vs. Mali“) u praksi pruža zanemarljivu vrednost. Sa $k=2$ ne možemo razlikovati elitnog VIP kupca sa 50.000 £ od stabilnog redovnog kupca sa 2.000 £, niti možemo razlikovati kupca koji je tek počeo da se hladi od onog koji je trajno izgubljen pre više od godinu dana.
2. **Prelom na krivi inercije (Elbow metod) na $k = 5$:** 
   Kriva inercije pokazuje najjasniju tačku preloma (tzv. „lakat“) na $k=5$ (inercija pada sa 8.562 za $k=2$ na 4.090 za $k=5$, nakon čega se kriva značajno zaravnjava, a svaki sledeći klaster donosi minimalno smanjenje greške).
3. **Akcionabilnost i poslovni povrat investicije:** 
   Klasterizacija je prevashodno alat za poslovno odlučivanje, a ne puka teorijska vežba. $k=5$ pruža optimalan balans između matematičke validnosti (stabilna silueta od 0,340 uz homogene grupe) i operativne primene. Ovih 5 klastera omogućavaju kreiranje konkretnih, diferenciranih strategija: VIP tretman za najveće, programi lojalnosti za stabilne, podsticaj za povremene, brza reaktivacija za uspavane i smanjenje troškova za trajno neaktivne.


## 3. Profili klastera i poslovna imena (Zadatak 4.3 / Pitanje 4.2)

### Pitanje 4.2: Opišite svaki klaster u jednoj rečenici razumljivoj direktoru prodaje.

Konačni K-Means model ($k=5$, random_state=42) kreirao je sledeću raspodelu na ukupno 5.852 kupca i 17.068.582,70 £ realizovanog prihoda:

| Klaster | Poslovno ime | Broj kupaca | Udeo kupaca (%) | Prosečan Recency (dana) | Prosečan Frequency (porudžbina) | Prosečan Monetary (£) | Ukupan prihod (£) | Udeo u prihodu (%) |
|---|---|---|---|---|---|---|---|---|
| **0** | **VIP kupci** | 454 | 7,76% | 17,3 | 32,6 | 20.635,50 £ | 9.368.515,70 £ | **54,89%** |
| **1** | **Lojalni kupci** | 1.284 | 21,94% | 48,8 | 9,1 | 3.627,24 £ | 4.657.379,30 £ | **27,29%** |
| **2** | **Povremeni kupci** | 1.141 | 19,50% | 32,5 | 2,5 | 666,19 £ | 760.120,40 £ | **4,45%** |
| **3** | **Uspavani kupci** | 1.352 | 23,10% | 288,7 | 3,8 | 1.375,00 £ | 1.858.990,00 £ | **10,89%** |
| **4** | **Neaktivni kupci** | 1.621 | 27,70% | 415,6 | 1,2 | 261,31 £ | 423.577,30 £ | **2,48%** |

#### Opis svakog klastera u jednoj rečenici za direktora prodaje:

1. **VIP kupci (Klaster 0):**
   > *„Naša elitna grupa od 454 najveća veleprodajna kupca koji naručuju na svake dve do tri nedelje (prosečno 33 porudžbine) i sami donose čak 55% celokupnog prihoda firme.“*
2. **Lojalni kupci (Klaster 1):**
   > *„Pouzdani, stalni poslovni partneri koji stabilno kupuju svakih mesec do dva (u proseku 9 porudžbina) i predstavljaju temelj našeg redovnog poslovanja sa udelom od preko 27% prihoda.“*
3. **Povremeni kupci (Klaster 2):**
   > *„Nedavno aktivirani ili manji kupci koji su naručivali u proteklih mesec dana sa manjim probnim korpama (prosečno 2 do 3 porudžbine i 666 £) i koje ciljanim akcijama treba prevesti u redovne kupce.“*
4. **Uspavani kupci (Klaster 3):**
   > *„Značajni bivši kupci koji su istorijski ostvarili po 4 porudžbine i preko 1.300 £ prometa, ali nisu kupili ništa već 9 do 10 meseci i nalaze se u ozbiljnom riziku od trajnog odlaska.“*
5. **Neaktivni kupci (Klaster 4):**
   > *„Jednokratni kupci male vrednosti (u proseku 261 £) koji su se javili samo jednom pre više od 14 meseci i na koje ne treba trošiti skupe prodajne resurse.“*


## 4. Poređenje sa RFM segmentacijom iz Dana 3 (Zadatak 4.4 / Pitanja 4.3 & 4.4)

### Pitanje 4.3: Da li se K-Means i RFM segmenti poklapaju? Gde se najviše razilaze?

#### 1. Matrica preklapanja segmenata (`pd.crosstab`)
Tabela ukrštanja pravila RFM segmentacije iz Dana 3 i K-Means klastera iz Dana 4:

| RFM segment (Dan 3) | VIP kupci | Lojalni kupci | Povremeni kupci | Uspavani kupci | Neaktivni kupci | Ukupno |
|---|---|---|---|---|---|---|
| **Champions** | **362** | **392** | 76 | 0 | 0 | 830 |
| **Loyal Customers** | 86 | **805** | 258 | 2 | 0 | 1.151 |
| **Potential Loyalists** | 0 | 51 | **661** | 1 | 0 | 713 |
| **New Customers** | 0 | 1 | **57** | 0 | 0 | 58 |
| **Promising** | 0 | 45 | **89** | 0 | 0 | 134 |
| **Need Attention** | 0 | **189** | 17 | 0 | 0 | 206 |
| **Cannot Lose Them** | 5 | 33 | 0 | **34** | 0 | 72 |
| **At Risk** | 0 | 0 | 0 | **652** | 102 | 754 |
| **Asleep** | 0 | 0 | 0 | **412** | 22 | 434 |
| **Lost** | 0 | 0 | 0 | 205 | **1.295** | 1.500 |
| **Ukupno** | **454** | **1.284** | **1.141** | **1.352** | **1.621** | **5.852** |

#### 2. Gde se segmenti poklapaju:
- **Na ekstremima ponašanja:**
  - Postoji izuzetno visoko slaganje kod izgubljenih i neaktivnih kupaca: čak 1.295 od 1.500 `Lost` kupaca iz RFM-a K-Means svrstava direktno u `Neaktivne kupce`, a preostalih 205 u `Uspavane kupce`. Nijedan `Lost` kupac nije zalutao u aktivne klastere.
  - Slično, segment `Champions` je skoro u potpunosti podeljen između `VIP kupaca` (362) i `Lojalnih kupaca` (392).
  - Segmenti `Potential Loyalists` i `New Customers` se velikom većinom slivaju u `Povremene kupce` (661, odnosno 57).

#### 3. Gde se segmenti najviše razilaze:
- **Razdvajanje vrha baze (`Champions` vs. `VIP kupci`):** 
  Pravila RFM-a su u `Champions` stavila 830 kupaca na osnovu fiksnih kvintila ($R \ge 4, F \ge 4, M \ge 4$). Međutim, K-Means je prepoznao da ova grupa nije homogena: iz nje je izdvojio apsolutni vrh piramide — 454 kupca (`VIP kupci`) sa prosečnom potrošnjom od 20.635 £, dok je preostale spojio sa grupom stabilnih redovnih kupaca (`Lojalni kupci`, prosek 3.627 £).
- **Prelazne i rizične kategorije (`At Risk`, `Cannot Lose Them`, `Need Attention`):** 
  Fiksne granice kvintila u RFM-u stvaraju veštačke preseke (kupac sa ocenom 2,9 i kupac sa ocenom 3,1 mogu završiti u potpuno različitim segmentima). K-Means ih grupiše na osnovu kontinuiranog rastojanja:
  - Kupci iz `At Risk` su podeljeni između `Uspavanih` (652 kupca sa solidnom prethodnom istorijom) i `Neaktivnih` (102 kupca sa skromnijom istorijom).
  - Segment `Cannot Lose Them` (72 kupca) podeljen je na one koji su zaista duboko uspavani (34 kupca u `Uspavani`) i one čija visoka frekvencija i potrošnja još uvek gravitiraju ka aktivnim jezgrima (`Lojalni` sa 33 i `VIP` sa 5 kupaca).


### Pitanje 4.4: Koji pristup biste koristili u praksi i za šta? Navedite po jednu prednost svakog.

#### 1. Preporučena arhitektura u praksi: Dvoslojna segmentacija
U realnom poslovnom okruženju ova dva pristupa nisu isključivi konkurenti, već komplementarni slojevi analitičkog sistema:

```
+---------------------------------------------------------------------------------------+
| STRATEŠKI NIVO: K-Means klasterizacija (Kvartalno / Polugodišnje ažuriranje)         |
| Korisnici: Rukovodstvo prodaje, finansije, strateški marketing, operativno planiranje |
| Svrha: Zdravlje portfolija kupaca, planiranje budžeta, dodela nivoa podrške (SLA)    |
+---------------------------------------------------------------------------------------+
                                           |
                                           v
+---------------------------------------------------------------------------------------+
| OPERATIVNI NIVO: RFM segmentacija na bazi pravila (Dnevni / Nedeljni automatski batch)|
| Korisnici: CRM tim, marketing automatizacija (HubSpot, Braze, Klaviyo), prodavci      |
| Svrha: Automatizovani email tokovi, triger kampanje, dodela popusta i vaučera        |
+---------------------------------------------------------------------------------------+
```

#### 2. Namena u praksi i ključna prednost za oba pristupa:

| Pristup | Primena u praksi | Glavna prednost |
|---|---|---|
| **RFM segmentacija (pravila)** | **Dnevne CRM i marketinške kampanje:** Okidanje automatizovanih imejl poruka dobrodošlice, reaktivacionih sekvenci, dodela personalizovanih kupona i postavljanje dnevnih zadataka prodajnom timu na osnovu jednostavnih pragova. | **Potpuna transparentnost i intuitivnost:** Svakom članu tima (marketing, prodaja, menadžment) je trenutno jasno zašto se kupac nalazi u određenom segmentu. Pravila se mogu odmah prilagoditi u CRM-u bez ponovnog treniranja algoritama i rizika od pomeranja centara. |
| **K-Means klasterizacija** | **Strateško planiranje i alokacija budžeta:** Definisanje komercijalnih paketa usluga (npr. dodela namenskog komercijaliste samo za 454 VIP kupca), optimizacija skladišnih zaliha i identifikacija prirodnih grupa kupaca bez nametnutih arbitrarnih granica. | **Otkrivanje prirodnih grupacija u podacima bez veštačkih podela:** Algoritam uvažava zajedničku višedimenzionalnu gustinu podataka umesto veštačkih 20-procentnih kvintila, otkrivajući realne obrasce ponašanja koji prevazilaze krute tablične matrice. |

---

# Odgovori na pitanja za Dan 5 — Predikcija: Da li će se kupac vratiti?

Odgovori u nastavku su izvedeni direktno iz programskog koda, izlaznih rezultata izvršavanja i modela mašinskog učenja u [05_predict.ipynb](notebooks/05_predict.ipynb) i prateće projektne dokumentacije.


## 1. Namerna greška i curenje podataka (Zadatak 5.1)

### Pitanje za Zadatak 5.1: Zašto je rezultat naivnog modela (Tačnost > 0,98, ROC AUC > 0,999) potpuno bezvredan?

U Zadatku 5.1 namerno je napravljena klasična početnička greška:
- Vrednost `Recency` je izračunata na celokupnom skupu podataka: `Recency = (maksimalni_datum + 1 dan - poslednji_datum).days`.
- Ciljna promenljiva je definisana pravilom: `Cilj = 1 ako je Recency <= 30, inače 0`.
- Logistička regresija je obučena koristeći osobine `[Recency, Frequency, Monetary]` da predvidi `Cilj`.
- Rezultati evaluacije: **Tačnost = 0,9801 (98,01%)**, **ROC AUC = 0,9998 (~1,0000)**.

#### Zašto je ovaj rezultat potpuno bezvredan:
1. **Direktna matematička tautologija (samo-ispunjujući cilj):**
   Ciljna varijabla `Cilj` je eksplicitno definisana kao matematički prag nad ulaznom osobinom `Recency` ($Recency \le 30$). Davanjem osobine `Recency` modelu kao prediktora, model ne uči zakonitosti ponašanja kupaca, već uči običnu trivijalnu matematičku nejednakost ($x \le 30 \implies y = 1$).
2. **Vremensko curenje podataka (korišćenje nepoznate budućnosti):**
   U ovakvoj postavci, `Recency` uzima u obzir sve transakcije do poslednjeg sekunda u bazi. Kada bi se ovakav model primenio u praksi danas, mi ne možemo znati recency kupca za 30 dana u budućnosti dok to vreme zapravo ne prođe. Korišćenje budućnosti za predviđanje sadašnjosti daje lažno savršene rezultate na istorijskim podacima, ali je u produkciji neprimenljivo.


## 2. Ispravna postavka sa vremenskim presekom i kreiranje osobina (Zadaci 5.2 i 5.3)

### 1. Arhitektura vremenske podele podataka (Time-based Cutoff)
Kako bi se u potpunosti eliminisalo curenje podataka iz budućnosti, vremenska linija je strogo podeljena na dva prozora:

```
|------------------- PROZOR POSMATRANJA (PROŠLOST) ---------------|--- PROZOR CILJA (BUDUĆNOST) ---|
           Osobine se računaju ISKLJUČIVO odavde                  ↑       Ovde merimo CILJ
                                                             DATUM PRESEKA
                                                              2011-09-10
                                                      (Maksimalni datum − 90 dana)
```

- **Maksimalni datum u podacima:** `2011-12-09 12:50:00`
- **Datum preseka (Maksimalni datum − 90 dana):** `2011-09-10`
- **Broj transakcija iz prošlosti (prozor posmatranja):** 618.316 transakcija (79,6% podataka)
- **Broj transakcija iz budućnosti (prozor cilja):** 158.263 transakcije (20,4% podataka)
- **Kupci uključeni u modelovanje:** 5.256 jedinstvenih kupaca koji su bili aktivni pre datuma preseka
- **Definicija cilja (`Goal`):** Da li je kupac napravio barem jednu porudžbinu u narednih 90 dana nakon preseka:
  - **Klasa 1 (Vratili se):** 2.289 kupaca (**43,55% ~ 43,6%**)
  - **Klasa 0 (Nisu se vratili):** 2.967 kupaca (**56,45% ~ 56,4%**)

### 2. Kreirane osobine (izračunate isključivo iz prošlosti pre preseka)
Kreirano je ukupno 13 ponašajnih osobina po kupcu na osnovu istorijskih podataka:
1. `Recency`: Broj dana od poslednje kupovine pre preseka do datuma preseka.
2. `Frequency`: Broj jedinstvenih faktura (porudžbina) pre preseka.
3. `Monetary`: Ukupna potrošnja (£) pre preseka.
4. `NumberOfItems`: Ukupan broj kupljenih stavki (redova u fakturama) pre preseka.
5. `DistinctProducts`: Broj različitih artikala (`StockCode`) koje je kupac naručio pre preseka.
6. `AverageQuantity`: Prosečna količina naručena po stavci pre preseka.
7. `AveragePrice`: Prosečna jedinična cena (£) kupljenih artikala pre preseka.
8. `AverageCart`: Prosečna vrednost korpe po porudžbini (`Monetary / Frequency`).
9. `CustomerTenure`: Starost poslovnog odnosa u danima (`Datum preseka - Prva kupovina`).
10. `AverageDaysBetweenPurchases`: Prosečan razmak između porudžbina (`CustomerTenure / (Frequency - 1)`).
11. `IsUK`: Binarni indikator da li je kupac iz Velike Britanije.
12. `OrdersLast90Days`: Broj faktura realizovanih u poslednjih 90 dana neposredno pre preseka (`[Presek - 90d, Presek]`).
13. `SpendingLast90Days`: Ukupna potrošnja (£) realizovana u poslednjih 90 dana neposredno pre preseka.


## 3. Obuka, evaluacija i poređenje modela (Zadatak 5.4)

### 1. Eksperimentalna postavka
- **Podela na Train/Test:** 75% Trening (3.942 kupca) / 25% Test (1.314 kupaca), sa očuvanom proporcijom klasa (`stratify=Goal`).
- **Distribucija u test setu:** 742 Klasa 0 (56,47%) / 572 Klasa 1 (43,53%).
- **Evaluacija tri modela:**
  1. `DummyClassifier(strategy="most_frequent")` — Osnovna linija (većinska klasa).
  2. `LogisticRegression(class_weight="balanced", max_iter=2000)` — Linearni model unutar `Pipeline`-a sa `StandardScaler`-om.
  3. `RandomForestClassifier(n_estimators=300, max_depth=12, min_samples_leaf=20, class_weight="balanced")` — Nelinearni ansambl stabala.

### 2. Tabela uporedne evaluacije (Test skup, $N = 1.314$)

| Model | Tačnost (Accuracy) | Preciznost (Precision) | Odziv (Recall) | F1 ocena | ROC AUC |
|---|---|---|---|---|---|
| **Osnovna linija (Baseline)** | 0,5647 (56,5%) | 0,0000 | 0,0000 | 0,0000 | 0,5000 |
| **Logistička regresija** | 0,7314 (73,1%) | 0,6918 (69,2%) | 0,6906 (69,1%) | 0,6912 (69,1%) | 0,7949 (~0,795) |
| **Random Forest** | **0,7435 (74,4%)** | **0,7205 (72,0%)** | **0,6713 (67,1%)** | **0,6950 (69,5%)** | **0,8047 (~0,805)** |

### 3. Matrice konfuzije (Test skup, $N = 1.314$)

```
Osnovna linija (Baseline):
[[742    0]   --> TN = 742, FP = 0
 [572    0]]  --> FN = 572, TP = 0

Logistička regresija:
[[566  176]   --> TN = 566, FP = 176 (Lažne uzbune)
 [177  395]]  --> FN = 177 (Propušteni kupci), TP = 395 (Pogođeni povratnici)

Random Forest:
[[593  149]   --> TN = 593, FP = 149 (Lažne uzbune)
 [188  384]]  --> FN = 188 (Propušteni kupci), TP = 384 (Pogođeni povratnici)
```


## 4. Tumačenje modela i značajnost osobina (Zadatak 5.5)

### 1. Značajnost osobina (Feature Importance kod Random Forest-a)
Random Forest je identifikovao svežinu kupovine i ritam naručivanja kao ključne faktore za predviđanje povratka:

| Rang | Osobina (Feature) | Relativni značaj | Poslovno tumačenje |
|---|---|---|---|
| **1** | `Recency` | **0,2052 (20,5%)** | Dominantni prediktor: svežina nedavne aktivnosti je presudna. |
| **2** | `AverageDaysBetweenPurchases` | **0,1546 (15,5%)** | Ritam naručivanja: kupci sa redovnim ciklusom pouzdano ponavljaju kupovinu. |
| **3** | `NumberOfItems` | **0,1029 (10,3%)** | Ukupan obim i dubina prethodne aktivnosti. |
| **4** | `SpendingLast90Days` | **0,0899 (9,0%)** | Finansijska posvećenost kupca neposredno pre preseka. |
| **5** | `Frequency` | **0,0898 (9,0%)** | Ukupan istorijski broj realizovanih porudžbina. |
| **6** | `Monetary` | **0,0890 (8,9%)** | Ukupan istorijski prihod ostvaren od kupca. |
| **7** | `OrdersLast90Days` | **0,0629 (6,3%)** | Intenzitet i zalet porudžbina u prethodna tri meseca. |
| **8** | `DistinctProducts` | **0,0597 (6,0%)** | Širina asortimana: kupci koji istražuju katalog imaju veći stepen zadržavanja. |
| **9** | `AverageCart` | **0,0401 (4,0%)** | Prosečna vrednost korpe. |
| **10** | `CustomerTenure` | **0,0396 (4,0%)** | Dužina trajanja poslovnog odnosa. |
| **11** | `AveragePrice` | **0,0338 (3,4%)** | Prosečna jedinična cena artikala. |
| **12** | `AverageQuantity` | **0,0321 (3,2%)** | Količina robe po naručenoj stavci. |
| **13** | `IsUK` | **0,0003 (0,03%)** | Geografska lokacija kupca ima zanemarljiv uticaj na verovatnoću povratka. |

### 2. Standardizovani koeficijenti Logističke regresije
Koeficijenti pokazuju smer i jačinu uticaja na verovatnoću (log-odds) ponovne kupovine:
- **Pozitivni pokretači (povećavaju verovatnoću povratka):**
  - `DistinctProducts` (+0,5684): Raznovrsnost asortimana stvara vezanost za dobavljača i lojalnost.
  - `OrdersLast90Days` (+0,4792): Zalet kupovina neposredno pre preseka je direktan indikator kontinuiteta.
  - `Frequency` (+0,3393): Veliki broj prethodnih kupovina potvrđuje stalnog partnera.
  - `SpendingLast90Days` (+0,1912): Svež finansijski obrt.
  - `CustomerTenure` (+0,1178): Stabilni dugogodišnji kupci imaju uspostavljene navike naručivanja.
- **Negativni pokretači (smanjuju verovatnoću povratka):**
  - `AveragePrice` (-0,5587): Visoka pojedinačna cena često oslikava vanredne ili jednokratne narudžbine sa niskom učestalošću.
  - `Recency` (-0,5393): Sa povećanjem broja dana od poslednje kupovine eksponencijalno opada šansa za povratak.
  - `NumberOfItems` (-0,4109) i `AverageDaysBetweenPurchases` (-0,2949): Veliki razmaci između narudžbina prirodno smanjuju verovatnoću da kupac ponovo kupi u fiksnom prozoru od 90 dana.


## 5. Odgovori na pitanja za notebook

### Pitanje 5.1: Objasnite svojim rečima šta je curenje podataka i kako ste ga izbegli.
1. **Šta je curenje podataka (Data Leakage):**
   Curenje podataka predstavlja situaciju u kojoj informacije van trening skupa — konkretno iz same ciljne promenljive, iz budućih vremenskih tačaka ili iz test skupa — nehotice dospeju u osobine (features) na kojima model uči. Model tada „vara“: umesto da prepoznaje realne uzročno-posledične obrasce ponašanja, on uči prečice bazirane na podacima koji u realnom vremenu produkcije nisu i ne mogu biti dostupni. Posledica je lažno savršen rezultat na razvoju (poput AUC 0,9998 u Zadatku 5.1) koji se u produkciji momentalno raspada.
2. **Kako smo ga izbegli:**
   - **Stroga vremenska granica (Time-based Cutoff):** Definisali smo datum preseka (`2011-09-10`, 90 dana pre kraja skupa). Sve osobine izračunate su **isključivo** iz transakcija zaključenih na dan preseka ili pre njega.
   - **Nezavisan prozor za merenje cilja:** Ciljna promenljiva (`Goal`) merena je **isključivo** na osnovu događaja koji su se desili *posle* datuma preseka.
   - **Pipeline obrada bez curenja:** Standardizacija podataka (`StandardScaler`) je fitovana isključivo na trening skupu unutar `Pipeline`-a, sprečavajući da test statistika kontaminira proces učenja.

### Pitanje 5.2: Osnovna linija ima tačnost 56,5%, vaš model ~74%. Da li je model dobar? Zašto tačnost ovde nije dovoljna mera?
1. **Da li je model dobar?**
   Da, model je izuzetno dobar. Skok sa 56,5% na **74,4% tačnosti**, uz stabilan **ROC AUC od 0,805** i balansiranu preciznost (72,0%) i odziv (67,1%), predstavlja jak i visoko profitabilan prediktivni signal u realnom poslovanju predviđanja odlaska/povratka kupaca. Model uspeva da tačno identifikuje preko dve trećine svih kupaca koji će se vratiti, zadržavajući visoku pouzdanost predikcije.
2. **Zašto tačnost nije dovoljna mera:**
   - **Neuravnoteženost klasa i trivijalna osnovna linija:** U test skupu 56,5% kupaca se nije vratilo. Naivni model koji jednostavno pretpostavi da se *niko nikada neće vratiti* postiže tačnost od 56,5%, ali ima nulti odziv, nultu preciznost i apsolutno nikakvu upotrebnu vrednost za biznis.
   - **Asimetrija poslovnih troškova:** Tačnost posmatra obe greške (lažnu uzbunu i propuštenog kupca) kao jednako loše (vrednujući ih sa greškom 1). U stvarnom biznisu, gubitak vrednog kupca košta hiljade funti, dok je slanje jedne suvišne marketinške poruke zanemarljiv trošak.
   - **Zavisnost od fiksnog praga (0,5):** Tačnost meri performanse samo na jednoj proizvoljnoj tački preseka verovatnoće. Metrike poput ROC AUC (0,805) ocenjuju sposobnost modela da rangira kupce kroz sve moguće poslovne pragove.

### Pitanje 5.3: Objasnite matricu konfuzije rečima: šta je „lažna uzbuna“, a šta „propušten kupac“? Koja greška je skuplja za firmu i zašto?
1. **Značenje polja u matrici konfuzije (Test skup, Random Forest):**
   - **Lažna uzbuna (False Positive, $FP = 149$):** Model je predvideo da će se kupac vratiti, ali se on zapravo *nije vratio* (ostao je neaktivan).
   - **Propušten kupac (False Negative, $FN = 188$):** Model je predvideo da se kupac neće vratiti, ali je on zapravo *došao i kupio* (ili je napustio firmu a da ga sistem nije prepoznao za spasavanje).
   - **Tačno identifikovani povratnici (True Positive, $TP = 384$):** Kupci za koje je model pogodio da se vraćaju.
   - **Tačno identifikovani neaktivni (True Negative, $TN = 593$):** Kupci za koje je model pogodio da neće kupiti.
2. **Koja greška je skuplja za firmu i zašto:**
   Trošak direktno zavisi od poslovne primene modela:
   - **U programima zadržavanja i sprečavanja odlaska (Retention / Churn):** **Propušten kupac (False Negative) je neuporedivo skuplja greška.** Ako ne prepoznamo kupca koji odlazi i ne preduzmemo prodajnu akciju, trajno gubimo njegovu celokupnu životnu vrednost (CLV) koja u B2B veleprodaji dostiže desetine hiljada funti. Lažna uzbuna nas košta samo utrošenog poziva ili poslatog vaučera.
   - **U kampanjama skupih direktnih poklona:** Ukoliko firma svakom predviđenom kupcu šalje skup fizički paket ili skup komercijalni poklon (npr. vrednosti 50–100 €), tada **Lažna uzbuna (False Positive)** stvara direktan čist gubitak budžeta na neaktivnim nalozima.
   - Generalno, u upravljanju odnosima sa kupcima **False Negative je 5 do 10 puta skuplja greška**, jer je cena akvizicije novog kupca (CAC) znatno veća od troškova zadržavanja postojećeg.

### Pitanje 5.4: `Recency` je najvažnija osobina. Da li je to isto curenje podataka kao u 5.1? Obrazložite — ovo je suština celog dana.
**Ne, to fundamentalno NIJE isto curenje podataka!** Ovo razgraničenje predstavlja ključnu suštinu prediktivnog modelovanja vremenskih serija:
- **U Zadatku 5.1 (Curenje):** `Recency` je izračunat nad *celokupnom bazom* (uključujući i budući prozor), a cilj je definisan bukvalno kao $Recency \le 30$. Osobina je sadržala direktno rešenje cilja — model je učio kružnu definiciju nad istim vremenskim podacima.
- **U Zadatku 5.2/5.3 (Validan prediktivni signal):** `Recency` je izračunat strogo iz transakcija u prošlosti *u odnosu na istorijski datum preseka* (`Presek - Poslednja kupovina pre preseka`). On meri koliko je dana prošlo od kupovine pre preseka. Ciljna promenljiva meri da li je kupac kupio nešto u *budućih 90 dana nakon preseka*.
- Prošli recency je legitiman signal ponašanja: kupac koji je naručio robu pre 4 dana ima prirodno daleko veći zalet i šansu da ponovo naruči u narednih 90 dana nego kupac koji nije naručio ništa punih 300 dana. Ovde se poštuje smer vremena: istorijsko ponašanje iz prošlosti predviđa buduće događaje preko nepropusne vremenske granice.

### Pitanje 5.5: Firma šalje kupon od 5 evra kupcima za koje model kaže da će se vratiti. Da li biste podigli ili spustili prag odlučivanja sa 0,5? Šta vam još treba da biste odgovorili tačno?
1. **Odluka o pomeranju praga odlučivanja:**
   - **Ekonomska dilema:** Slanje kupona kupcima koji imaju izuzetno visoku verovatnoću povratka (npr. $p > 0,85$) znači davanje čistog popusta i smanjenje sopstvene profitne marže za kupovinu koja bi se svakako desila („kanibalizacija profita“). S druge strane, slanje kupona kupcima sa izuzetno malom verovatnoćom ($p < 0,2$) troši budžet na naloge koji kupon nikada neće iskoristiti.
   - Ukoliko je strateški zadatak striktno nagraditi one koji će se sigurno vratiti uz minimalno rasipanje budžeta na lažne uzbune, prag odlučivanja treba **podići (npr. na 0,65 ili 0,70)**. Time se podiže preciznost (Precision) i kupon se daje samo onima u čiju je kupovinu model visoko siguran.
   - *Moderna marketinška praksa (Uplift modelovanje):* Kupon od 5 € donosi najveći povrat ako se ne šalje ni sigurnim povratnicima ni trajno izgubljenima, već onima na prelazu („kupci koje je moguće ubediti“, npr. verovatnoća $p \in [0,35, 0,60]$), gde podsticaj stvara stvarni dodatni prihod.
2. **Dodatne informacije neophodne za tačan matematički odgovor:**
   Da bismo analitički optimizovali prag $p^*$ koji maksimizuje ukupan profit firme, neophodni su sledeći finansijski i operativni parametri:
   1. **Prosečna vrednost narudžbine (AOV) i profitna marža ($M$):** Koliki je stvarni neto profit (u evrima) koji firma ostvari od jedne realizovane narudžbine?
   2. **Inkrementalni efekat kupona (Uplift, $L$):** Koliki procenat kupaca kupuje *isključivo zahvaljujući* kuponu od 5 € u odnosu na kontrolnu grupu kojoj kupon nije poslat?
   3. **Stopa kanibalizacije marže:** Koji udeo povratnika bi naručio po punoj ceni i bez ponuđenog popusta od 5 €?
   4. **Trošak kampanje ($C$):** Troškovi slanja i obrade vaučera (SMS, imejl, štampa) uvećani za nominalnu vrednost popusta od 5 €.
   Definisanjem matrice dobitaka i gubitaka: $\mathbb{E}[\text{Profit}(p)] = \sum (\text{Dodatni profit} - \text{Troškovi})$, proračunava se tačan optimalni prag $p^*$ koji maksimizuje ukupan neto profit preduzeća.

---

# Odgovori na pitanja za Dan 6 — Opcija C: Sistem preporuka (Market Basket analiza)

Odgovori u nastavku su izvedeni direktno iz programskog koda, izlaznih rezultata analize potrošačke korpe i pravila pridruživanja u [06_recommendation.ipynb](notebooks/06_recommendation.ipynb) i prateće projektne dokumentacije.


## 1. Postavka sistema i dimenziona optimizacija

### 1. Kombinatorni izazov i rešenje
- **Problem sa celim asortimanom:** Celokupan katalog broji **4.620 različitih artikala (`StockCode`)**. Matrica svih mogućih neusmerenih parova sadrži $\binom{4620}{2} = \frac{4620 \times 4619}{2} \approx \mathbf{10.669.890}$ kombinacija (odnosno preko **21,3 miliona** usmerenih pravila pridruživanja). Ogromna većina tih parova predstavlja slučajne jednokratne kupovine artikala koji su se prodali tek nekoliko puta, što nema statističku stabilnost i stvara nepotrebno računarsko opterećenje.
- **Optimizacija na 300 najčešćih artikala:** Ograničavanjem na top **300 najfrekventnijih artikala** (po broju jedinstvenih faktura):
  - **Pokriveni računi:** **33.081 faktura** sadrži barem jedan od 300 najpopularnijih artikala.
  - **Udeo u bazi:** Ovim podskupom pokriveno je čak **90,4% svih jedinstvenih računa (36.594)** u posmatranom periodu poslovanja.
  - **Generisani parovi:** Iz korpi sa više artikala izračunata su **44.834 jedinstvena para proizvoda**.

### 2. Metrike analize potrošačke korpe (Market Basket Analysis)
Za svaki par artikala $A$ i $B$ proračunate su tri klasične metrike pravila pridruživanja:
1. **Podrška (Support, $A \land B$):** Udeo računa koji sadrže oba artikla u odnosu na ukupan broj kvalifikovanih računa ($N = 33.081$):
   $$\text{Support}(A \land B) = \frac{\text{Broj faktura}(A \land B)}{N}$$
2. **Pouzdanost (Confidence, $A \to B$):** Uslovna verovatnoća da će račun sadržati artikal $B$ ukoliko je na njemu već prisutan artikal $A$:
   $$\text{Confidence}(A \to B) = \frac{\text{Broj faktura}(A \land B)}{\text{Broj faktura}(A)}$$
3. **Lift ($A, B$):** Odnos stvarne zajedničke pojave i teorijski očekivane zajedničke pojave pod pretpostavkom statističke nezavisnosti:
   $$\text{Lift}(A, B) = \frac{\text{Support}(A \land B)}{\text{Support}(A) \times \text{Support}(B)} = \frac{\text{Confidence}(A \to B)}{\text{Support}(B)}$$
   - $\text{Lift} > 1$: Pozitivna povezanost (artikli se kupuju zajedno znatno češće nego što bi se desilo slučajno).
   - $\text{Lift} = 1$: Nezavisnost (nema međusobnog uticaja u korpi).
   - $\text{Lift} < 1$: Supstitutivni odnos (kupovina artikla $A$ smanjuje verovatnoću kupovine artikla $B$).

### 3. Provera kontrolnih tačaka
- **Broj parova sa Lift > 1:** **37.740 parova** od ukupno 44.834 (**84,2%** generisanih parova ima stvarnu pozitivnu vezu).
- **Maksimalni ostvareni Lift:** **37,64** (između odgovarajućih papirnih čaša i papirnih tanjira).


## 2. Najjače asocijacije proizvoda (Top parovi po Lift-u)

Tabela 15 najjačih asocijativnih veza među proizvodima rangiranih po koeficijentu Lift:

| Artikal A | Artikal B | Zajedno (računa) | Conf (A $\to$ B) | Conf (B $\to$ A) | Lift |
|---|---|---|---|---|---|
| **SET/6 RED SPOTTY PAPER CUPS** | **SET/6 RED SPOTTY PAPER PLATES** | 489 | **78,74%** | **70,66%** | **37,64** |
| **SET/10 PINK POLKADOT PARTY CANDLES** | **SET/10 BLUE SPOTTY PARTY CANDLES** | 480 | 64,95% | **80,13%** | **35,87** |
| **FELTCRAFT PRINCESS CHARLOTTE DOLL** | **FELTCRAFT PRINCESS LOLA DOLL** | 379 | 61,03% | 63,48% | **33,82** |
| **RED STRIPE CERAMIC DRAWER KNOB** | **BLUE STRIPE CERAMIC DRAWER KNOB** | 400 | 65,15% | 62,60% | **33,73** |
| **WOODEN HEART CHRISTMAS SCANDINAVIAN**| **WOODEN STAR CHRISTMAS SCANDINAVIAN** | 469 | 64,51% | 70,63% | **32,14** |
| **FELTCRAFT CUSHION BUTTERFLY** | **FELTCRAFT CUSHION RABBIT** | 445 | 65,54% | 65,73% | **32,02** |
| **GARDENERS KNEELING PAD CUP OF TEA** | **GARDENERS KNEELING PAD KEEP CALM** | 463 | 72,91% | 61,24% | **31,91** |
| **GREEN REGENCY TEACUP AND SAUCER** | **PINK REGENCY TEACUP AND SAUCER** | 635 | 67,12% | **84,11%** | **29,41** |
| **WHITE SPOT RED CERAMIC DRAWER KNOB** | **RED STRIPE CERAMIC DRAWER KNOB** | 340 | 55,37% | 54,40% | **29,31** |
| **PINK HAPPY BIRTHDAY BUNTING** | **BLUE HAPPY BIRTHDAY BUNTING** | 437 | 69,04% | 53,95% | **28,19** |
| **BATHROOM METAL SIGN** | **TOILET METAL SIGN** | 554 | 59,76% | 76,10% | **27,16** |
| **FELTCRAFT CUSHION OWL** | **FELTCRAFT CUSHION RABBIT** | 415 | 53,83% | 61,30% | **26,30** |
| **EDWARDIAN PARASOL BLACK** | **EDWARDIAN PARASOL RED** | 386 | 48,98% | 62,06% | **26,05** |
| **RED RETROSPOT UMBRELLA** | **BLACK RECORD COVER FRAME** | 399 | 62,34% | 50,25% | **25,97** |
| **SET/10 PINK POLKADOT PARTY CANDLES** | **SET/10 BLUE SPOTTY PARTY CANDLES** | 364 | 46,19% | 60,77% | **25,51** |


## 3. Implementacija funkcije `recommend(stock_code)`

Funkcija `recommend(item_code, top_n=5)` pretražuje asocijativna pravila u oba smera ($A \to B$ i $B \to A$), spaja rezultate i sortira preporučene artikle opadajuće po vrednostima **Lift** i **Confidence**:

### Primer rada na artiklu `22699` (ROSES REGENCY TEACUP AND SAUCER)
```
=== RECOMMENDATIONS FOR ITEM: ROSES REGENCY TEACUP AND SAUCER (22699) ===
  Recommended_Code                  Recommended_Name  Common  Confidence    Lift
0            22698    PINK REGENCY TEACUP AND SAUCER     605       0.565  24.774
1            22697   GREEN REGENCY TEACUP AND SAUCER     754       0.705  24.642
2            23245        SET OF 3 REGENCY CAKE TINS     133       0.124   5.727
3            22423          REGENCY CAKESTAND 3 TIER     588       0.550   5.481
4            22907  PACK OF 20 NAPKINS PANTRY DESIGN      89       0.083   4.076
```
Sistem preporuka nepogrešivo rekonstruiše celokupan viktorijanski set za čaj: sestrinske Pink i Green šoljice ($Lift \approx 25$), prateće kutije za kolače ($Lift = 5,73$) i glavno komercijalno sidro asortimana `REGENCY CAKESTAND 3 TIER` ($Lift = 5,48$, kupljeno zajedno na čak 588 računa).


## 4. Odgovori na pitanja za Dan 6 Opcija C

### Pitanje 6.C.1: Pogledajte najjače parove — šta im je zajedničko?
Svi najjači parovi sa ekstremnim Lift vrednostima ($\text{Lift} \in [25, 38]$) dele jasne zajedničke karakteristike:
1. **Funkcionalni komplementi iz iste dizajnerske linije:**
   Čaše i tanjiri identičnog tufnastog dezena za rođendane (`SET/6 RED SPOTTY PAPER CUPS` + `PLATES`, Lift = 37,64).
2. **Varijacije iste linije u različitim bojama ili motivima:**
   - Roze i plave svećice za proslave (`SET/10 PINK POLKADOT` + `BLUE SPOTTY CANDLES`, Lift = 35,87) ili rođendanske zastavice (Pink + Blue Bunting, Lift = 28,19).
   - Keramičke ručice za fioke u različitim prugama (Red stripe + Blue stripe, Lift = 33,73).
3. **Tematske praznične ili dekorativne kolekcije:**
   - Skandinavski drveni božićni ukrasi (Wooden Heart + Wooden Star, Lift = 32,14).
   - Dečije lutke i jastuci iz linije „Feltcraft“ (Lutke Charlotte + Lola, Lift = 33,82; Jastuci Leptir + Zec, Lift = 32,02).
   - Parovi tablica sa natpisima za enterijer (`BATHROOM` + `TOILET METAL SIGN`, Lift = 27,16).

### Pitanje 6.C.2: Šta vam to otkriva o tome kako kupci kupuju?
Ovi obrasci otkrivaju komercijalnu logiku veleprodajnih B2B kupaca i vlasnika maloprodaja:
- **Kupovina celovitih kolekcija:** Trgovci ne kupuju pojedinačne nasumične artikle. Da bi im maloprodajne police i izlozi bili atraktivni krajnjim potrošačima, oni poručuju **tematski zaokružene komplete** (npr. ceo party asortiman čaša, tanjira i svećica, ili metalne table za kupatilo i toalet u paru).
- **Dubina asortimana (kupovina više boja):** Veleprodajni kupac naručuje istovremeno roze i plavu varijantu istog proizvoda kako bi u svojoj radnji ponudio izbor za oba pola.
- **Sinergija unakrsne prodaje (Cross-selling):** Kada kupac planira događaj ili tematsko opremanje, odluka o kupovini čaša automatski povlači potrebu za tanjirima i salvetama istog dezena, stvarajući ogroman potencijal za preporučivanje paketa proizvoda.

### Pitanje 6.C.3: Koje je glavno ograničenje ovakvog sistema?
Iako su pravila pridruživanja jednostavna i intuitivna, sistem ima četiri ključna strukturna ograničenja:
1. **Problem hladnog starta (Cold-Start Problem za nove proizvode):**
   Sistem se u potpunosti oslanja na istorijske zajedničke kupovine. Novi artikli tek uvršteni u katalog ili retki artikli sa malo kupovina **nikada ne mogu biti preporučeni** niti se za njih mogu naći pravila.
2. **Očigledne i banalne preporuke (nedostatak faktora iznenađenja / Serendipity):**
   Sistem dominantno predlaže alternativne boje ili očigledne supstitute (npr. „Kupili ste zelenu šoljicu, evo vam i roze šoljica“). Ne uspeva da otkrije suptilne veze između različitih kategorija koje kupac sam ne bi primetio (npr. da kupci zidnih satova često kupuju specifične mirisne sveće).
3. **Potpuno odsustvo personalizacije prema kupcu:**
   Preporuke zavise isključivo od posmatranog artikla, a ne od kupca koji ga gleda. Svaki kupac (bio on VIP distributer ili jednokratni kupac poklona) dobija identičnu listu preporuka za dati artikal, bez obzira na istorijat njegovih prethodnih narudžbina ili budžet.
4. **Kombinatorna eksplozija i skalabilnost:**
   Računanje veza za više od dva artikla istovremeno ($A \land B \to C$) eksponencijalno uvećava broj kombinacija. Na katalogu od nekoliko desetina hiljada artikala rad sa matricama ko-pojavljivanja zahteva specijalizovane algoritme (Apriori ili FP-Growth) uz rigorozno filtriranje.



