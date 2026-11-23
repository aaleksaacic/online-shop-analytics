# Izveštaj o analizi prodaje i strateški plan poslovanja

**Predmet:** Sveobuhvatna analiza prodajnih rezultata, segmentacija kupaca, prediktivno modelovanje i strateške preporuke  
**Period analize:** Dvogodišnji revizorski period prodaje (decembar 2009 — decembar 2011)  
**Ciljna grupa:** Rukovodstvo preduzeća, direktor prodaje, operativni tim za marketing i komercijalu  


---

## 1. Rezime za menadžment

Ovaj rezime sintetizuje nalaze komercijalne analize sprovedene na 776.579 verifikovanih transakcija i 5.852 identifikovana poslovna partnera u periodu od 1. decembra 2009. do 9. decembra 2011. godine:

1. **Izrazita koncentracija prihoda:** Kompanija posluje pod ekstremnom asimetrijom prodaje, gde samo 7,8% najvećih kupaca (454 VIP veleprodajna klijenta) generiše čak 54,9% ukupnog prihoda preduzeća (9,37 miliona funti od ukupno 17,07 miliona), dok najveći pojedinačni kupac samostalno ostvaruje preko 580.000 funti prometa.
2. **Ponavljajući jesenji skok tražnje:** Prodaja pokazuje identičan sezonski obrazac u obe posmatrane godine, sa izraženim padom u prvom kvartalu (najniža prodaja u februaru od ~442.000 £) i naglim jesenjim ubrzanjem koje dostiže godišnji vrhunac u novembru sa preko 1,13 miliona funti prihoda usled nabavki maloprodajnih partnera za božićnu sezonu.
3. **Čista B2B komercijalna struktura:** Preko 95% svih kupovina realizuje se radnim danima (od ponedeljka do petka) u intervalu od 09:00 do 16:00 časova, dok subota čini svega 0,057% ukupnog prihoda kompanije usled redovnog neradnog dana centralnog skladišta i otpreme.
4. **Predvidivost ponašanja kupaca:** Razvijeni prediktivni model dokazuje da se verovatnoća povratka kupca u prozoru od 90 dana može pouzdano prognozirati (ROC AUC = 0,805, tačnost = 74,4%), pri čemu su svežina kupovine (Recency, 20,5% značaja) i ustaljeni ritam naručivanja (15,5% značaja) glavni faktori budućeg ponašanja.
5. **Potencijal unakrsne prodaje u tematskim kolekcijama:** Analiza potrošačke korpe pokazuje da čak 84,2% najfrekventnijih parova proizvoda ima pozitivan koeficijent povezanosti (Lift > 1, sa maksimumom od 37,64), jer poslovni kupci primarno naručuju kompletne tematske setove i varijacije boja, a ne pojedinačne izolovane artikle.


---

## 2. Podaci i priprema baze

### 2.1 Sirovi podaci
Ulazna baza obuhvatala je dva radna lista sa transakcijama za dve uzastopne godine:
- List "Year 2009-2010": 525.461 redova
- List "Year 2010-2011": 541.910 redova
- Ukupan sirovi obim: 1.067.371 redova raspoređenih u 8 standardnih kolona.

### 2.2 Sprovedene procedure čišćenja podataka
Radi obezbeđivanja potpune tačnosti zaključaka, sprovedeni su strogi protokoli validacije:
1. **Uklanjanje preklapanja listova:** Utvrđeno je vremensko preklapanje od tačno 9 dana (od 1. do 9. decembra 2010. godine) koje se pojavljivalo na oba lista, obuhvatajući 1.088 identičnih brojeva računa i 45.046 dupliranih stavki. Uklanjanjem ovih duplikata sprečeno je veštačko dupliranje prihoda za decembar 2010. godine za skoro 100%.
2. **Isključivanje neregistrovanih kupaca:** Ukupno 243.007 redova (22,77% sirove baze) nije imalo unet `CustomerID`. Ovi nalozi bez identifikacije su isključeni iz RFM analize, klasterizacije i prediktivnog modelovanja ponašanja kupaca, ali su sačuvani za potrebe revizije kataloga i obima artikala.
3. **Izolacija storniranih i otkazanih porudžbina:** Ukupno 8.292 otkazana računa (oznaka sa prefiksom "C"), koji predstavljaju 1.462.051 £ negativnog prometa (~8,6% bruto prometa), izdvojeni su u zasebnu tabelu storniranja.
4. **Uklanjanje administrativnih korekcija:** Eliminisanje internih računovodstvenih stavki (npr. "Adjust bad debt", manualni troškovi poštarine) sa nestandardnim šiframa artikala (`AMAZONFEE`, `POST`, `D`, `CRUK`) i anomalnim cenama.

### 2.3 Konačni skup podataka za modelovanje i analizu
Nakon obrade dobijena je čista i konzistentna analitička baza:
- **Broj čistih transakcija:** 776.579 redova
- **Broj jedinstvenih izlaznih faktura:** 36.594 računa
- **Broj identifikovanih B2B kupaca:** 5.852 preduzeća
- **Broj aktivnih artikala u asortimanu:** 4.620 artikala
- **Ukupan realizovani bruto prihod:** 17.068.582,70 £ (Neto prihod nakon storniranja: 15.606.531,70 £)


---

## 3. Komercijalni nalazi i dinamika prodaje

### 3.1 Asimetrija prihoda i Paretova raspodela
Potrošnja kupaca ima izrazitu eksponencijalnu raspodelu. Dok je aritmetički prosek potrošnje po kupcu 2.916,71 £, medijana potrošnje iznosi svega 856,02 £ (prosek je čak 3,4 puta veći od medijane).
- Donjih 50% kupaca kompanije generiše manje od 10% ukupnog prometa.
- Gornjih 20% kupaca donosi približno 80% celokupnog prihoda, što dosledno potvrđuje Paretovo pravilo (80/20).
- Najveći pojedinačni kupac ostvario je 580.724,17 £ prometa tokom dvogodišnjeg perioda.
- **Poslovni zaključak:** Stabilnost poslovanja firme presudno zavisi od uskog kruga velikih veleprodajnih partnera. Zadržavanje postojećih ključnih kupaca predstavlja primarni komercijalni zadatak menadžmenta.

### 3.2 Sezonalnost prodaje kroz godine
Analiza mesečnih prihoda pokazuje stabilne obrasce koji se dosledno ponavljaju u 2010. i 2011. godini:
- **Zatišje u prvom kvartalu (Q1):** Prodaja dostiže godišnji minimum u januaru i februaru (februar 2011. ostvario je 442.294 £), što odgovara periodu u kojem maloprodajni partneri rasprodaju zalihe preostale od praznika.
- **Stabilan središnji period godine:** Od marta do avgusta mesečni promet se stabilizuje u opsegu od 550.000 £ do 750.000 £.
- **Jesenji skok (Q4):** Prihod naglo raste u septembru i oktobru, dostižući godišnji maksimum u novembru (1.155.978 £ u novembru 2010. i 1.136.534 £ u novembru 2011. godine).
- *Napomena o podacima za decembar 2011:* Zabeleženi pad prihoda na 426.380 £ u decembru 2011. isključivo je posledica činjenice da se baza završava 9. decembra u 12:50 časova. Kada se posmatra ekstrapolacija na pun mesec, decembar 2011. bi dostigao ili nadmašio nivo iz prethodne godine.

### 3.3 Poslovni ritam i profil kupaca
Vremenske oznake kupovina daju jasan dokaz o prirodi kupaca kompanije:
- **Doba dana:** Vrhunac naručivanja odvija se između 11:00 i 14:00 časova. Preko 95% svih kupovina realizuje se tokom standardnog radnog vremena (09:00–16:00).
- **Dan u sedmici:** Naručivanje je stabilno od ponedeljka do petka. Subota je operativno neradni dan magacina i čini svega 0,057% prometa (ukupno 9.803 £ za dve godine). U nedelju u ranim popodnevnim satima promet se obnavlja jer poslovni kupci pripremaju narudžbine za isporuku u ponedeljak.
- **Zaključak:** Klijenti su vlasnici maloprodajnih radnji, komercijalni nabavljači i korporativni distributeri (B2B), a ne individualni potrošači (B2C) koji kupuju uveče ili vikendom.

### 3.4 Ključni proizvodi kompanije („zvezde asortimana“)
Promet kompanije nosi uska grupa vodećih proizvoda:
- Tačno 5 artikala se istovremeno nalazi na obe Top 10 liste (po prodatim komadima i po ostvarenom prihodu):
  1. `WHITE HANGING HEART T-LIGHT HOLDER` (247.048 £ prihoda; 91.757 komada; 1.490 kupaca)
  2. `PAPER CRAFT , LITTLE BIRDIE` (168.470 £ prihoda; 80.995 komada; 1 kupac)
  3. `JUMBO BAG RED RETROSPOT` (134.307 £ prihoda; 74.224 komada; 860 kupaca)
  4. `ASSORTED COLOUR BIRD ORNAMENT` (124.352 £ prihoda; 78.234 komada; 1.010 kupaca)
  5. `MEDIUM CERAMIC TOP STORAGE JAR` (81.417 £ prihoda; 77.916 komada; 138 kupaca)
- *Komercijalna anomalija:* Proizvod `PAPER CRAFT , LITTLE BIRDIE` duguje celokupan promet jednoj jedinoj narudžbini kupca `16446` (račun `581483`). Ovaj vanredni veleprodajni ugovor mora se izolovati iz modela nabavke i preporuke robe kako ne bi stvorio lažnu sliku o redovnoj tražnji.

### 3.5 Segmentacija kupaca: Pravila RFM vs. K-Means klasterizacija
Uporednom analizom modela na bazi kvintilnih RFM pravila i K-Means klasterizacije ($k=5$ nad standardizovanim logaritamskim podacima) dobijena je sledeća raspodela:

| Segment / Klaster | Broj kupaca | Udeo kupaca | Prosečan Recency | Prosečan Frequency | Prosečan Monetary | Ukupan prihod | Udeo u prihodu |
|---|---|---|---|---|---|---|
| **VIP kupci** | 454 | 7,76% | 17,3 dana | 32,6 porudžbina | 20.635,50 £ | 9.368.515,70 £ | **54,89%** |
| **Lojalni kupci** | 1.284 | 21,94% | 48,8 dana | 9,1 porudžbina | 3.627,24 £ | 4.657.379,30 £ | **27,29%** |
| **Povremeni kupci** | 1.141 | 19,50% | 32,5 dana | 2,5 porudžbina | 666,19 £ | 760.120,40 £ | **4,45%** |
| **Uspavani kupci** | 1.352 | 23,10% | 288,7 dana | 3,8 porudžbina | 1.375,00 £ | 1.858.990,00 £ | **10,89%** |
| **Neaktivni / Izgubljeni** | 1.621 | 27,70% | 415,6 dana | 1,2 porudžbina | 261,31 £ | 423.577,30 £ | **2,48%** |

- **Ključni strateški zaključci:**
  - Segmenti **VIP** i **Lojalni** čine ukupno 29,7% kupaca, ali generišu čak **82,2% ukupnog prihoda firme** (14,03 miliona funti).
  - Grupa **Uspavani** obuhvata 1.352 bivša redovna kupca koji su istorijski doneli 1,86 miliona funti, ali nisu kupili ništa već skoro 10 meseci i nalaze se u zoni neposrednog odlaska.
  - RFM segment **„Ne smemo ih izgubiti“ (Cannot Lose Them)** identifikuje 72 elitna istorijska kupca (preko 503.000 £ prometa) koji su prestali da naručuju i zahtevaju hitnu ličnu intervenciju.

### 3.6 Analiza potrošačke korpe i potencijal preporuka
Analiza asocijativnih pravila na 300 najčešćih artikala (koji pokrivaju 90,4% svih faktura) dokazala je da komercijalni kupci ciljano kupuju komplementarne garniture:
- Čak 37.740 parova proizvoda ima koeficijent Lift veći od 1.
- Vodeći parovi dostižu Lift vrednosti između 25 i 37,64 (npr. papirne čaše i tanjiri tufnastog dezena sa Lift = 37,64; Regency šoljice u različitim bojama sa Lift = 29,41).
- Trgovci kupuju više boja istog artikla i prateće elemente kako bi u svojim radnjama izložili kompletne tematske postavke.


---

## 4. Prediktivno modelovanje i sistem za rano prepoznavanje odlaska kupaca

### 4.1 Postavka modela bez curenja podataka
Kako bi se obezbedio pouzdan operativni alat za sprečavanje odlaska kupaca, postavljen je vremenski razdvojen model:
- **Prozor posmatranja (prošlost):** Podaci zaključno sa datumom `2011-09-10` (618.316 transakcija).
- **Prozor ciljne promenljive (budućnost):** Narednih 90 dana (`2011-09-11` do `2011-12-09`).
- **Definicija cilja (`Goal`):** Da li postojeći aktivan kupac pravi barem jednu narudžbinu u narednih 90 dana (1/0).
- **Populacija:** 5.256 aktivnih kupaca pre preseka (43,6% se vratilo, 56,4% nije).

### 4.2 Rezultati modela na neviđenom test skupu ($N = 1.314$)
Modeli su obučeni na 75% podataka (3.942 kupca) i testirani na 25% uzorka (1.314 kupaca):

| Model | Tačnost (Accuracy) | Preciznost (Precision) | Odziv (Recall) | F1 ocena | ROC AUC |
|---|---|---|---|---|---|
| **Osnovna linija (Dummy)** | 56,47% | 0,00% | 0,00% | 0,00% | 0,500 |
| **Logistička regresija** | 73,14% | 69,18% | 69,06% | 69,12% | 0,795 |
| **Random Forest (300 stabala)** | **74,35%** | **72,05%** | **67,13%** | **69,50%** | **0,805** |

### 4.3 Ključni faktori predikcije i tumačenje
- **Svežina kupovine (`Recency`, 20,5% značaja):** Broj dana od poslednje kupovine do trenutka preseka je najsnažniji prediktor ponovne kupovine.
- **Ritam naručivanja (`AverageDaysBetweenPurchases`, 15,5% značaja):** Redovnost u prethodnim ciklusima snažno određuje verovatnoću kupovine u narednom tromesečju.
- **Širina korpe (`DistinctProducts`, koeficijent +0,568):** Kupci koji naručuju raznovrsnije artikle imaju znatno veći stepen zadržavanja zbog veće operativne integrisanosti sa kompanijom.


---

## 5. Strateški akcioni plan: Preporuke za rukovodstvo

Na osnovu empirijskih uvida iz prodaje, segmentacije i prediktivnog modela, menadžmentu se preporučuje sprovođenje šest ključnih poslovnih inicijativa:

### 1. Uvođenje posebnog odeljenja za ključne kupce (Key Account Management)
- **Ciljna grupa:** 454 VIP kupca (7,8% baze koji donose 54,9% ukupnog prihoda).
- **Konkretna akcija:** Izdvojiti ove kupce iz opšteg internet kanala i dodeliti im lične menadžere naloga (Key Account Managere). Obezbediti ugovorne nivoe podrške (SLA), garantovane rezervacije zaliha uoči sezone i progresivne količinske rabate.
- **Očekivani efekat:** Zaštita preko 9,3 miliona funti godišnjeg bazičnog prihoda od preuzimanja od strane konkurencije.

### 2. Hitna lična intervencija za vredne kupce u riziku
- **Ciljna grupa:** 72 kupca iz segmenta „Ne smemo ih izgubiti“ (Cannot Lose Them, preko 503.000 £ prometa) i gornji sloj od 1.352 uspavana kupca.
- **Konkretna akcija:** Zabraniti slanje generičkih promotivnih imejlova. Pokrenuti direktnu kampanju telefonskih poziva i poseta od strane direktora prodaje i iskusnih prodajnih menadžera radi utvrđivanja uzroka prekida saradnje (cene, logistika, kvalitet) i obnove ugovora.
- **Očekivani efekat:** Reaktivacija samo 20% kupaca iz ove grupe vraća preko 100.000 £ stabilnog prihoda visoke marže.

### 3. Ranije skaliranje nabavke i skladišta za jesenji sezonski skok (Q4)
- **Ciljna grupa:** Skladišna logistika, operacije pakovanja i lanac nabavke.
- **Konkretna akcija:** Planiranje nabavke ne sme pratiti prolećne promete. Poručivanje ključnih božićnih artikala (Regency program, ukrasi, poklon-kese) i angažovanje sezonskih radnika u magacinu mora početi u avgustu i septembru.
- **Očekivani efekat:** Eliminisanje nestašice robe i kašnjenja isporuka tokom oktobra i novembra, koji nose oko 25% ukupnog godišnjeg prometa kompanije.

### 4. Automatizovani paketi proizvoda na bazi unakrsne prodaje (Cross-selling)
- **Ciljna grupa:** Veleprodajni portal i elektronski katalog.
- **Konkretna akcija:** Implementirati pravila asocijacija direktno na sajt i u štampani katalog kroz ponudu gotovih paketa (npr. „Kompletan Regency set za čajajanke“ koji spaja šoljice, kutije i etažer, ili rođendanske pakete čaša, tanjira i svećica u odgovarajućim bojama).
- **Očekivani efekat:** Povećanje prosečne vrednosti porudžbine (AOV) za 8–15% kod povremenih i lojalnih kupaca.

### 5. Asimetrična raspodela marketinškog budžeta
- **Ciljna grupa:** Budžet za marketing i automatizaciju komunikacije.
- **Konkretna akcija:** Preraspodeliti budžet na osnovu povrata investicije:
  - Usmeriti 75% marketinških sredstava na zadržavanje i negovanje VIP i Lojalnih kupaca.
  - Usmeriti 15% na podsticanje 1.141 povremenog kupca da pređu u redovne mesečne naručioce.
  - Obustaviti skupe prodajne aktivnosti prema 1.621 trajno neaktivnom kupcu i svesti njihovu obradu na jeftine automatizovane imejl sekvence.
- **Očekivani efekat:** Znatno veći povrat na uloženi marketinški novac (ROI) i smanjenje troškova na izgubljenim kontaktima.

### 6. Operativna primena prediktivnog modela za sprečavanje odlaska kupaca
- **Ciljna grupa:** Prodajni tim i CRM sistem.
- **Konkretna akcija:** Integrisati razvijeni Random Forest model u mesečni automatski proces obrade podataka. Svakom aktivnom kupcu dodeljuje se verovatnoća povratka u narednih 90 dana. Kupci kod kojih verovatnoća padne ispod 0,50 automatski otvaraju prodajni zadatak za komercijalistu u CRM-u radi preventivnog kontakta.
- **Očekivani efekat:** Smanjenje stope trajnog odlaska kupaca (churn) za 10–18% na godišnjem nivou.


---

## 6. Ograničenja analize

Radi transparentnosti i profesionalne odgovornosti prema donosiocima odluka, navode se sledeća analitička ograničenja:

1. **Odsustvo podataka o profitnim maržama i troškovima:** Baza beleži isključivo prodajne cene i ostvareni bruto prihod, bez nabavnih cena robe (COGS), troškova transporta i opštih troškova poslovanja. Najprodavaniji artikli i segmenti sa najvećim prihodom ne moraju nužno biti i najprofitabilniji.
2. **Isključivanje neregistrovanih kupaca (Guest checkout):** Oko 22,8% transakcija (243.007 stavki) nije imalo identifikaciju kupca i nije moglo biti uključeno u segmentaciju i predikciju. Ukoliko ovi kupci imaju drugačije navike, stvarni domet veleprodajne mreže može biti širi od prikazanog.
3. **Nepotpun mesec decembar 2011. godine:** Posmatrani podaci se prekidaju 9. decembra 2011. godine. Prividni pad prihoda u tom mesecu je posledica prekida evidentiranja, a ne stvarnog pada poslovne aktivnosti.
4. **Statičnost analize potrošačke korpe:** Pravila asocijacija posmatraju istovremenu pojavu artikala na istom računu na uzorku od 300 najčešćih proizvoda. Analiza ne obuhvata artikle sa manjim obimom prodaje, niti prati sekvencijalne kupovine kroz vreme gde kupac prvo kupuje jedan, a mesec dana kasnije drugi komplementaran proizvod.
5. **Nedostatak spoljnih tržišnih parametara:** Analiza se bazira isključivo na internim transakcionim logovima i ne uzima u obzir promene cena konkurencije, makroekonomska kretanja valuta (GBP/EUR) niti zastoje u globalnim lancima snabdevanja.
