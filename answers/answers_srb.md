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

