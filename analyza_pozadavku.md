# Analýza požadavků – Telemedicínský systém pro podporu pacientů s diabetem mellitus 2. typu

## Obsah

1. [Úvod](#1-úvod)
2. [Funkční požadavky](#2-funkční-požadavky)
3. [Nefunkční požadavky](#3-nefunkční-požadavky)
4. [Datové požadavky](#4-datové-požadavky)
5. [Požadavky na integraci (HL7 FHIR)](#5-požadavky-na-integraci-hl7-fhir)
6. [Legislativní a regulatorní požadavky](#6-legislativní-a-regulatorní-požadavky)
7. [Omezení a předpoklady](#7-omezení-a-předpoklady)
8. [Rizika projektu](#8-rizika-projektu)
9. [Akceptační kritéria](#9-akceptační-kritéria)

---

## 1. Úvod

### 1.1 Účel dokumentu

Tento dokument definuje funkční a nefunkční požadavky na telemedicínský systém pro podporu pacientů s diabetem mellitus 2. typu. Slouží jako podklad pro implementaci, testování a akceptaci systému.

### 1.2 Vazba na analýzu a návrh systému

Kontext systému – současný stav a identifikované problémy (`P1–P5`), cíle projektu (`C1–C7`), uživatelské role, rozsah, architektura komponent a případy užití (`UC-01–UC-16`) – je popsán v dokumentu [Analýza a návrh systému](01_analyza_a_navrh_systemu.md). Požadavky v tomto dokumentu jsou členěny podle komponent definovaných v jeho kapitole 6:

1. mobilní aplikace pro pacienta,
2. zabezpečené webové API (backend),
3. databáze,
4. webový portál pro zdravotnický personál,
5. integrátor pro komunikaci s nemocničním informačním systémem (NIS) prostřednictvím standardu HL7 FHIR.

### 1.3 Konvence

- Požadavky jsou označeny jednoznačným identifikátorem ve tvaru `XX-YYY-NNN`, např. `FR-MOB-001` (funkční požadavek – mobilní aplikace).
- Priorita je stanovena metodou **MoSCoW**:
  - **M** (Must) – nezbytné, bez nich systém nesplní svůj účel,
  - **S** (Should) – důležité, ale ne kritické pro první verzi,
  - **C** (Could) – žádoucí, realizují se dle kapacity,
  - **W** (Won't, this time) – v této verzi se nerealizují.
- Klíčová slova „musí", „měl by", „může" odpovídají prioritám M, S, C.

| Prefix | Význam |
|---|---|
| FR-MOB | Funkční požadavek – mobilní aplikace |
| FR-API | Funkční požadavek – webové API |
| FR-DB | Funkční požadavek – databáze |
| FR-WEB | Funkční požadavek – webový portál |
| FR-INT | Funkční požadavek – FHIR integrátor |
| FR-ALR | Funkční požadavek – vyhodnocení a alerty |
| NFR-xxx | Nefunkční požadavek |
| LEG | Legislativní požadavek |


---

## 2. Funkční požadavky

### 2.1 Mobilní aplikace pro pacienta

#### 2.1.1 Registrace, přihlášení a profil

| ID | Požadavek | Priorita |
|---|---|---|
| FR-MOB-001 | Aplikace musí umožnit registraci pacienta pomocí aktivačního kódu / QR kódu vygenerovaného lékařem na portálu. | M |
| FR-MOB-002 | Aplikace musí podporovat bezpečné přihlášení (OAuth 2.0 / OpenID Connect) s vícefaktorovým ověřením. | M |
| FR-MOB-003 | Aplikace by měla podporovat odemknutí biometrií (otisk prstu, Face ID) po prvním přihlášení. | S |
| FR-MOB-004 | Aplikace musí umožnit správu profilu (jméno, kontakt, jednotky mmol/l nebo mg/dl, jazyk). | M |
| FR-MOB-005 | Aplikace musí před prvním použitím zobrazit informace o zpracování osobních údajů a vyžádat výslovný souhlas. | M |
| FR-MOB-006 | Aplikace musí automaticky odhlásit uživatele po nastavené době nečinnosti. | M |

#### 2.1.2 Sběr dat z glukometru

| ID | Požadavek | Priorita |
|---|---|---|
| FR-MOB-010 | Aplikace musí umožnit spárování glukometru přes Bluetooth Low Energy (standardní Glucose Profile / Glucose Service 0x1808). | M |
| FR-MOB-011 | Aplikace musí automaticky stahovat nová měření (hodnota, čas měření, sekvenční číslo, příznaky kvality). | M |
| FR-MOB-012 | Aplikace musí zabránit duplicitnímu importu téhož měření. | M |
| FR-MOB-013 | Aplikace musí umožnit přiřazení kontextu měření: nalačno, před jídlem, po jídle (1–2 h), před spaním, v noci, jiné. | M |
| FR-MOB-014 | Aplikace musí umožnit ruční zadání glykémie s rozlišením ručně zadaných a automaticky přenesených hodnot. | M |
| FR-MOB-015 | Aplikace by měla podporovat více modelů glukometrů (seznam podporovaných zařízení je spravován centrálně). | S |
| FR-MOB-016 | Aplikace může podporovat import z glukometrů bez BLE pomocí NFC nebo OCR fotografie displeje. | C |

#### 2.1.3 Sběr dat o fyzické aktivitě

| ID | Požadavek | Priorita |
|---|---|---|
| FR-MOB-020 | Aplikace musí umožnit import dat o aktivitě z Apple HealthKit (iOS) a Google Health Connect (Android) po udělení oprávnění. | M |
| FR-MOB-021 | Aplikace musí sbírat minimálně: počet kroků, dobu aktivity (minuty), typ aktivity a spálenou energii. | M |
| FR-MOB-022 | Aplikace by měla sbírat srdeční tep, pokud je dostupný. | S |
| FR-MOB-023 | Aplikace musí umožnit ruční záznam aktivity (typ, délka, intenzita). | M |
| FR-MOB-024 | Aplikace by měla agregovat aktivitu do denních a týdenních souhrnů a porovnat je s doporučením (≥ 150 min středně intenzivní aktivity týdně). | S |

#### 2.1.4 Doplňkové údaje

| ID | Požadavek | Priorita |
|---|---|---|
| FR-MOB-030 | Aplikace musí umožnit záznam užití léků (perorální antidiabetika, inzulín – typ, dávka, čas). | M |
| FR-MOB-031 | Aplikace by měla umožnit záznam hmotnosti a krevního tlaku (ručně nebo z HealthKit/Health Connect). | S |
| FR-MOB-032 | Aplikace by měla umožnit záznam jídla (orientační množství sacharidů, poznámka, fotografie). | S |
| FR-MOB-033 | Aplikace může umožnit záznam subjektivních příznaků (únava, třes, pocení, žízeň). | C |

#### 2.1.5 Zobrazení dat a zpětná vazba

| ID | Požadavek | Priorita |
|---|---|---|
| FR-MOB-040 | Aplikace musí zobrazit poslední naměřenou hodnotu s barevným vyhodnocením (v cíli / nad / pod). | M |
| FR-MOB-041 | Aplikace musí zobrazit graf glykémií za den, týden, měsíc a 3 měsíce. | M |
| FR-MOB-042 | Aplikace musí zobrazit statistiky: průměr, směrodatnou odchylku, podíl hodnot v cílovém rozmezí, počet hypoglykémií. | M |
| FR-MOB-043 | Aplikace by měla zobrazit glykémie společně s aktivitou v jednom grafu (korelace). | S |
| FR-MOB-044 | Aplikace musí fungovat offline – zobrazit lokálně uložená data a uložit nová měření pro pozdější synchronizaci. | M |

#### 2.1.6 Upozornění a připomínky

| ID | Požadavek | Priorita |
|---|---|---|
| FR-MOB-050 | Aplikace musí při naměření hypoglykémie okamžitě zobrazit upozornění s doporučeným postupem (pravidlo 15 g sacharidů, opakované měření za 15 min). | M |
| FR-MOB-051 | Aplikace musí umožnit nastavení připomínek měření a užití léků. | M |
| FR-MOB-052 | Aplikace musí zobrazovat push notifikace o nových zprávách a doporučeních od lékaře. | M |
| FR-MOB-053 | Aplikace by měla upozornit pacienta, pokud delší dobu (nastavitelně, např. 3 dny) neprovedl měření. | S |

#### 2.1.7 Komunikace, reporty a souhlasy

| ID | Požadavek | Priorita |
|---|---|---|
| FR-MOB-060 | Aplikace musí umožnit zabezpečené zasílání textových zpráv ošetřujícímu týmu. | M |
| FR-MOB-061 | Aplikace musí zřetelně informovat, že zprávy neslouží pro akutní stavy, a nabídnout volbu tísňové linky 155. | M |
| FR-MOB-062 | Aplikace by měla umožnit vygenerovat a sdílet PDF report za zvolené období. | S |
| FR-MOB-063 | Aplikace musí umožnit správu souhlasů (sdílení s lékařem, s pečovatelem, odesílání do NIS) a jejich odvolání. | M |
| FR-MOB-064 | Aplikace musí umožnit export vlastních dat ve strojově čitelném formátu (právo na přenositelnost). | M |
| FR-MOB-065 | Aplikace může zobrazovat edukační obsah (strava, pohyb, péče o nohy). | C |

### 2.2 Zabezpečené webové API

| ID | Požadavek | Priorita |
|---|---|---|
| FR-API-001 | API musí poskytovat REST rozhraní (JSON) zdokumentované ve specifikaci OpenAPI 3. | M |
| FR-API-002 | API musí ověřovat identitu klientů pomocí OAuth 2.0 / OpenID Connect (access token JWT s krátkou platností, refresh token). | M |
| FR-API-003 | API musí uplatňovat řízení přístupu na základě rolí (RBAC) a vztahu lékař–pacient (lékař vidí pouze přiřazené pacienty). | M |
| FR-API-004 | API musí přijímat dávky měření (batch upload) a zpracovávat je idempotentně (klíč: zařízení + sekvenční číslo / klientské UUID). | M |
| FR-API-005 | API musí validovat vstupní data (rozsahy, jednotky, formát času, povinné položky). | M |
| FR-API-006 | API musí ukládat čas měření v UTC spolu s časovou zónou zařízení. | M |
| FR-API-007 | API musí poskytovat endpointy pro portál: seznam pacientů, detail pacienta, měření, statistiky, alerty, zprávy. | M |
| FR-API-008 | API musí zaznamenávat každý přístup ke zdravotním datům do auditního logu (kdo, kdy, k čemu, z jaké IP). | M |
| FR-API-009 | API musí omezovat počet požadavků (rate limiting) a chránit se proti zneužití. | M |
| FR-API-010 | API musí podporovat odesílání push notifikací (APNs, FCM) bez obsahu zdravotních dat v těle notifikace. | M |
| FR-API-011 | API by mělo podporovat verzování (např. `/api/v1/`). | S |
| FR-API-012 | API může poskytovat přístup k datům ve formátu FHIR (vlastní FHIR fasáda). | C |

### 2.3 Vyhodnocení dat a alerty (alert engine)

Výchozí prahové hodnoty vycházejí z doporučení ADA / ČDS a musí být lékařem individuálně nastavitelné.

| Kategorie | Výchozí podmínka | Závažnost |
|---|---|---|
| Hypoglykémie 1. stupně | glykémie < 3,9 mmol/l (70 mg/dl) | Střední |
| Hypoglykémie 2. stupně | glykémie < 3,0 mmol/l (54 mg/dl) | Vysoká |
| Výrazná hyperglykémie | glykémie > 13,9 mmol/l (250 mg/dl) | Střední |
| Kritická hyperglykémie | glykémie > 20,0 mmol/l opakovaně | Vysoká |
| Zvýšená glykémie nalačno | průměr nalačno > 7,2 mmol/l za 7 dní | Nízká |
| Opakované hypoglykémie | ≥ 2 hypoglykémie za 7 dní | Vysoká |
| Zhoršující se trend | růst 14denního průměru o > 15 % | Nízká |
| Nízká variabilita v cíli | podíl hodnot v cílovém rozmezí < 50 % za 14 dní | Nízká |
| Výpadek dat | žádné měření déle než N dní (výchozí 5) | Nízká |
| Nízká aktivita | < 50 % doporučené týdenní aktivity 2 týdny po sobě | Informativní |

| ID | Požadavek | Priorita |
|---|---|---|
| FR-ALR-001 | Systém musí vyhodnotit každé nové měření vůči individuálním cílovým rozmezím pacienta do 60 s od přijetí. | M |
| FR-ALR-002 | Systém musí podporovat pravidla nad jednotlivými hodnotami i nad trendy (časová okna). | M |
| FR-ALR-003 | Systém musí přiřadit každému alertu úroveň závažnosti (informativní, nízká, střední, vysoká). | M |
| FR-ALR-004 | Systém musí notifikovat personál dle závažnosti (vysoká – okamžitě, ostatní – v denním souhrnu). | M |
| FR-ALR-005 | Lékař musí mít možnost upravit prahové hodnoty a pravidla pro konkrétního pacienta. | M |
| FR-ALR-006 | Systém musí evidovat stav alertu (nový, potvrzený, vyřešený) a kdo a kdy jej zpracoval. | M |
| FR-ALR-007 | Systém musí potlačit opakované duplicitní alerty v krátkém časovém okně (deduplikace). | S |
| FR-ALR-008 | Systém by měl eskalovat nezpracovaný alert vysoké závažnosti po stanovené době (např. 4 h v pracovní době). | S |
| FR-ALR-009 | Systém by měl vypočítávat odhad HbA1c (eA1c / GMI) z průměrné glykémie za ≥ 14 dní. | S |
| FR-ALR-010 | Systém může využívat prediktivní model rizika hypoglykémie. | C |

> **Poznámka:** Systém neslouží jako systém pro tísňové volání. Personál nemusí reagovat v reálném čase 24/7; tato skutečnost musí být pacientovi jasně sdělena (viz FR-MOB-061).

### 2.4 Databáze

| ID | Požadavek | Priorita |
|---|---|---|
| FR-DB-001 | Databáze musí ukládat entity: uživatel, pacient, zdravotník, zařízení, měření glykémie, aktivita, medikace, cílová rozmezí, alert, zpráva, souhlas, auditní záznam, stav synchronizace s NIS. | M |
| FR-DB-002 | Databáze musí zajišťovat referenční integritu a transakční konzistenci (ACID). | M |
| FR-DB-003 | Data musí být šifrována v klidu (např. AES-256 / TDE) a citlivé položky (rodné číslo) šifrovány na úrovni sloupce. | M |
| FR-DB-004 | Databáze musí uchovávat historii změn klinických dat (nesmí docházet k nevratnému přepisu – verzování / soft delete). | M |
| FR-DB-005 | Auditní log musí být neměnný (append-only) a oddělený od provozních dat. | M |
| FR-DB-006 | Databáze musí být pravidelně zálohována (denně plná, průběžně transakční log) s ověřováním obnovy. | M |
| FR-DB-007 | Databáze by měla být optimalizována pro časové řady (indexy dle pacienta a času, případně partitioning). | S |
| FR-DB-008 | Databáze musí podporovat výmaz / anonymizaci dat pacienta po uplynutí skartační lhůty nebo dle zákonných požadavků. | M |

### 2.5 Webový portál pro zdravotnický personál

#### 2.5.1 Přístup a správa

| ID | Požadavek | Priorita |
|---|---|---|
| FR-WEB-001 | Portál musí vyžadovat přihlášení s vícefaktorovým ověřením; měl by podporovat SSO nemocnice (SAML/OIDC). | M |
| FR-WEB-002 | Portál musí umožnit lékaři zařadit pacienta do programu (vytvoření účtu, vygenerování aktivačního QR kódu). | M |
| FR-WEB-003 | Portál by měl umožnit dohledání pacienta v NIS a převzetí jeho demografických údajů. | S |
| FR-WEB-004 | Portál musí umožnit administrátorovi správu uživatelů, rolí a přiřazení k pracovištím. | M |
| FR-WEB-005 | Portál musí umožnit ukončit sledování pacienta (vyřazení z programu). | M |

#### 2.5.2 Dashboard a detail pacienta

| ID | Požadavek | Priorita |
|---|---|---|
| FR-WEB-010 | Portál musí zobrazovat seznam přiřazených pacientů seřaditelný podle rizika (počet a závažnost otevřených alertů). | M |
| FR-WEB-011 | Seznam musí u každého pacienta zobrazovat: poslední glykémii, čas poslední synchronizace, podíl hodnot v cíli, počet hypoglykémií za 14 dní, týdenní aktivitu. | M |
| FR-WEB-012 | Portál musí umožnit filtrování (riziko, výpadek dat, lékař, pracoviště) a fulltextové vyhledávání. | M |
| FR-WEB-013 | Detail pacienta musí obsahovat grafy glykémií, aktivity a medikace v čase s volitelným obdobím. | M |
| FR-WEB-014 | Detail pacienta by měl obsahovat standardizovaný souhrnný profil (např. denní modální profil / AGP-like přehled glykémií podle denní doby). | S |
| FR-WEB-015 | Detail musí odlišovat automaticky přenesené a ručně zadané hodnoty. | M |
| FR-WEB-016 | Portál musí umožnit lékaři nastavit individuální cílová rozmezí a prahy alertů. | M |
| FR-WEB-017 | Portál musí umožnit zápis klinické poznámky / doporučení k pacientovi. | M |

#### 2.5.3 Alerty, komunikace, reporty

| ID | Požadavek | Priorita |
|---|---|---|
| FR-WEB-020 | Portál musí zobrazovat frontu alertů s možností potvrzení, vyřešení a připojení komentáře. | M |
| FR-WEB-021 | Portál musí umožnit zabezpečenou komunikaci s pacientem (zprávy, doporučení, potvrzení přečtení). | M |
| FR-WEB-022 | Portál by měl nabízet šablony častých doporučení. | S |
| FR-WEB-023 | Portál musí umožnit generování PDF reportu za zvolené období. | M |
| FR-WEB-024 | Portál musí umožnit ruční odeslání vybraných dat / reportu do NIS. | M |
| FR-WEB-025 | Portál by měl zobrazovat stav synchronizace s NIS (odesláno, chyba, čeká). | S |
| FR-WEB-026 | Portál může poskytovat agregované statistiky za pracoviště (počet pacientů, průměrná kompenzace, adherence). | C |

### 2.6 FHIR integrátor

| ID | Požadavek | Priorita |
|---|---|---|
| FR-INT-001 | Integrátor musí komunikovat s NIS dle standardu HL7 FHIR R4 (REST, JSON). | M |
| FR-INT-002 | Integrátor musí mapovat glykémie na zdroj `Observation` s kódem LOINC a jednotkami UCUM. | M |
| FR-INT-003 | Integrátor musí mapovat data o aktivitě na zdroje `Observation` (kroky, doba aktivity, energie). | M |
| FR-INT-004 | Integrátor musí odesílat alerty vysoké závažnosti jako `Flag` / `DetectedIssue` (dle dohody s NIS). | S |
| FR-INT-005 | Integrátor musí umožnit odeslání souhrnného reportu jako `DiagnosticReport` s přiloženým PDF (`DocumentReference` / `Binary`). | S |
| FR-INT-006 | Integrátor musí identifikovat pacienta v NIS pomocí identifikátoru (číslo pojištěnce / interní ID NIS) a ověřit shodu demografických údajů. | M |
| FR-INT-007 | Integrátor musí umožnit načtení údajů z NIS: `Patient`, `Practitioner`, `Condition` (diagnóza E11), `MedicationStatement`, poslední HbA1c (`Observation`). | S |
| FR-INT-008 | Integrátor musí používat asynchronní frontu zpráv se zaručeným doručením (at-least-once) a idempotentní zpracování na straně NIS (podmíněné vytvoření, `identifier`). | M |
| FR-INT-009 | Integrátor musí při nedostupnosti NIS opakovat odeslání s exponenciálním odstupem a po vyčerpání pokusů přesunout zprávu do chybové fronty. | M |
| FR-INT-010 | Integrátor musí validovat vytvořené zdroje proti FHIR profilům před odesláním. | M |
| FR-INT-011 | Integrátor musí autentizovat komunikaci s NIS (SMART Backend Services / OAuth 2.0 client credentials, mTLS). | M |
| FR-INT-012 | Integrátor musí logovat všechny výměny (bez zbytečných osobních údajů v aplikačních lozích) a vytvářet `AuditEvent` / interní auditní záznam. | M |
| FR-INT-013 | Integrátor by měl podporovat odběr změn z NIS (FHIR Subscription) nebo periodické dotazování. | S |
| FR-INT-014 | Integrátor by měl být konfigurovatelný pro více NIS (odlišné profily a endpointy). | S |

---

## 3. Nefunkční požadavky

### 3.1 Bezpečnost

| ID | Požadavek | Priorita |
|---|---|---|
| NFR-SEC-001 | Veškerá komunikace musí být šifrována pomocí TLS 1.2+ (preferováno TLS 1.3); mobilní aplikace by měla používat certificate pinning. | M |
| NFR-SEC-002 | Data musí být šifrována v klidu na serveru i v zařízení (iOS Keychain / Data Protection, Android Keystore, šifrovaná lokální DB). | M |
| NFR-SEC-003 | Přístup zdravotnického personálu musí vyžadovat vícefaktorové ověření. | M |
| NFR-SEC-004 | Hesla musí být ukládána pomocí adaptivní hashovací funkce (Argon2id / bcrypt). | M |
| NFR-SEC-005 | Systém musí uplatňovat princip nejnižších oprávnění a oddělení rolí. | M |
| NFR-SEC-006 | Systém musí být chráněn proti zranitelnostem OWASP Top 10 a OWASP Mobile Top 10 / MASVS. | M |
| NFR-SEC-007 | Před nasazením a následně alespoň jednou ročně musí proběhnout penetrační test. | M |
| NFR-SEC-008 | Tajné klíče a certifikáty musí být spravovány v trezoru (např. HSM / Key Vault), nikoli ve zdrojovém kódu. | M |
| NFR-SEC-009 | Systém musí detekovat a logovat bezpečnostní události a předávat je do SIEM nemocnice. | S |
| NFR-SEC-010 | Push notifikace nesmí obsahovat zdravotní údaje (pouze obecný text „Máte novou zprávu"). | M |

### 3.2 Výkon a škálovatelnost

| ID | Požadavek | Priorita |
|---|---|---|
| NFR-PERF-001 | 95 % požadavků API musí být zpracováno do 500 ms při běžné zátěži. | M |
| NFR-PERF-002 | Dashboard portálu se musí načíst do 3 s pro seznam 500 pacientů. | M |
| NFR-PERF-003 | Synchronizace glukometru (až 500 záznamů) musí proběhnout do 30 s. | S |
| NFR-PERF-004 | Systém musí v první fázi obsloužit 5 000 pacientů a 200 zdravotníků a být horizontálně škálovatelný na 50 000 pacientů. | M |
| NFR-PERF-005 | Alert vysoké závažnosti musí být doručen personálu do 5 min od přijetí měření. | M |

### 3.3 Dostupnost a spolehlivost

| ID | Požadavek | Priorita |
|---|---|---|
| NFR-AVL-001 | Dostupnost API a portálu musí být alespoň 99,5 % měsíčně (mimo ohlášenou údržbu). | M |
| NFR-AVL-002 | RPO (maximální ztráta dat) ≤ 15 min, RTO (doba obnovy) ≤ 4 h. | M |
| NFR-AVL-003 | Mobilní aplikace nesmí ztratit žádné měření při výpadku sítě nebo serveru. | M |
| NFR-AVL-004 | Plánovaná údržba musí být oznámena uživatelům alespoň 48 h předem. | S |

### 3.4 Použitelnost a přístupnost

| ID | Požadavek | Priorita |
|---|---|---|
| NFR-USA-001 | Mobilní aplikace musí být srozumitelná i pro starší pacienty (velké písmo, vysoký kontrast, jednoduché ovládání; cílová skupina často 50+ let). | M |
| NFR-USA-002 | Rozhraní musí splňovat WCAG 2.1 úrovně AA (zákon č. 99/2019 Sb. pro veřejné instituce). | M |
| NFR-USA-003 | Barevné hodnocení hodnot nesmí být jediným nositelem informace (doplněno ikonou / textem). | M |
| NFR-USA-004 | Systém musí být v češtině; měl by být připraven na lokalizaci (angličtina, ukrajinština, slovenština). | M / S |
| NFR-USA-005 | Běžný úkon pacienta (synchronizace + přiřazení kontextu) nesmí vyžadovat více než 3 kroky. | S |
| NFR-USA-006 | Pacient by měl zvládnout zprovoznění aplikace bez asistence do 10 min. | S |

### 3.5 Kompatibilita

| ID | Požadavek | Priorita |
|---|---|---|
| NFR-CMP-001 | Mobilní aplikace musí podporovat iOS 16+ a Android 10+ (API 29+). | M |
| NFR-CMP-002 | Portál musí podporovat aktuální verze prohlížečů Chrome, Edge, Firefox a Safari. | M |
| NFR-CMP-003 | Integrátor musí podporovat HL7 FHIR R4 (4.0.1); měl by být připraven na R5 / české národní profily. | M / S |

### 3.6 Udržovatelnost a provozovatelnost

| ID | Požadavek | Priorita |
|---|---|---|
| NFR-MNT-001 | Systém musí mít automatizované testy (jednotkové, integrační) s pokrytím kritické logiky (alert engine, mapování FHIR) ≥ 80 %. | M |
| NFR-MNT-002 | Nasazení musí probíhat automatizovaně (CI/CD) s oddělenými prostředími DEV / TEST / PROD. | M |
| NFR-MNT-003 | Systém musí poskytovat monitoring (metriky, logy, health-check endpointy) a alerting provozních chyb. | M |
| NFR-MNT-004 | Testovací prostředí nesmí obsahovat reálná data pacientů (pouze syntetická / anonymizovaná). | M |
| NFR-MNT-005 | Dokumentace musí zahrnovat architekturu, API (OpenAPI), FHIR mapování a provozní příručku. | M |

---

## 4. Datové požadavky

### 4.1 Požadavky na kvalitu dat

| ID | Požadavek |
|---|---|
| DATA-001 | Glykémie se interně ukládá v mmol/l s přesností na 0,1; převod na mg/dl (×18,016) se provádí pouze při zobrazení. |
| DATA-002 | Přijatelný rozsah glykémie 0,6–33,3 mmol/l (typický rozsah glukometrů); hodnoty „LO"/„HI" se ukládají s příznakem. |
| DATA-003 | Každé měření musí nést informaci o zdroji (automatický přenos / ruční zadání) a identifikaci zařízení. |
| DATA-004 | Časy se ukládají v UTC s původní časovou zónou; systém musí detekovat zjevně chybný čas glukometru (např. v budoucnosti). |
| DATA-005 | Opravy dat nesmí přepsat původní hodnotu; oprava se ukládá jako nová verze s odkazem na původní záznam. |

### 4.2 Doba uchovávání dat

- Zdravotnická dokumentace předaná do NIS se řídí vyhláškou č. 98/2012 Sb. o zdravotnické dokumentaci (skartační lhůty).
- Data v telemedicínském systému se uchovávají po dobu sledování pacienta a následně po dobu stanovenou interní směrnicí poskytovatele (v souladu se zásadou minimalizace dle GDPR).
- Auditní záznamy se uchovávají minimálně 5 let.

---

## 5. Požadavky na integraci (HL7 FHIR)

| Interní data | FHIR zdroj | Kódování |
|---|---|---|
| Pacient | `Patient` | `identifier` – číslo pojištěnce / ID NIS |
| Lékař, sestra | `Practitioner`, `PractitionerRole` | – |
| Glukometr | `Device` | výrobce, model, sériové číslo |
| Glykémie z glukometru | `Observation` (category `laboratory` / `vital-signs` dle profilu) | LOINC **14743-9** Glucose [Moles/volume] in Capillary blood by Glucometer, jednotka UCUM `mmol/L` |
| Kontext měření (nalačno, po jídle) | `Observation.component` nebo `Observation.extension` | dle profilu NIS (např. SNOMED CT) |
| Počet kroků za den | `Observation` (category `activity`) | LOINC **41950-7** Number of steps in 24 hour Measured |
| Doba fyzické aktivity | `Observation` | LOINC **55411-3** Exercise duration |
| Spálená energie | `Observation` | LOINC **41981-2** Calories burned |
| Srdeční tep | `Observation` (`vital-signs`) | LOINC **8867-4** Heart rate |
| Hmotnost | `Observation` (`vital-signs`) | LOINC **29463-7** Body weight |
| HbA1c (z NIS) | `Observation` | LOINC **4548-4** Hemoglobin A1c/Hemoglobin.total in Blood |
| Diagnóza DM2 | `Condition` | MKN-10 **E11** |
| Medikace | `MedicationStatement` | ATC / SÚKL kód |
| Rizikový nález | `Flag` / `DetectedIssue` | – |
| Souhrnný report | `DiagnosticReport` + `DocumentReference` (PDF) | – |
| Doporučení lékaře | `Communication` / `CarePlan` | – |
| Souhlas pacienta | `Consent` | – |
| Audit | `AuditEvent` | – |

> Konkrétní kódy LOINC a profily je nutné ověřit a odsouhlasit s dodavatelem NIS a se zohledněním českých národních FHIR profilů (např. profily pro Evropský prostor pro zdravotní data / národní infrastrukturu elektronického zdravotnictví).

---

## 6. Legislativní a regulatorní požadavky

| ID | Předpis | Dopad na systém | Priorita |
|---|---|---|---|
| LEG-001 | **GDPR** (Nařízení EU 2016/679) a zákon č. 110/2019 Sb. | Zdravotní údaje = zvláštní kategorie (čl. 9); právní titul, informační povinnost, práva subjektu (přístup, oprava, výmaz, přenositelnost), záznamy o činnostech zpracování, **DPIA** (posouzení vlivu) je povinné, privacy by design & by default. | M |
| LEG-002 | **MDR** (Nařízení EU 2017/745 o zdravotnických prostředcích) | Software, který vyhodnocuje data a generuje klinická upozornění, je pravděpodobně zdravotnický prostředek (pravidlo 11 → minimálně třída IIa). Nutné: klasifikace, systém řízení kvality (ISO 13485), řízení rizik (ISO 14971), životní cyklus SW (IEC 62304), použitelnost (IEC 62366-1), klinické hodnocení, označení CE. | M |
| LEG-003 | Zákon č. 372/2011 Sb. o zdravotních službách a vyhláška č. 98/2012 Sb. | Vedení zdravotnické dokumentace, oprávněné osoby, skartační lhůty. | M |
| LEG-004 | Zákon č. 325/2021 Sb. o elektronizaci zdravotnictví | Interoperabilita, standardy výměny dat, identifikace subjektů, napojení na národní infrastrukturu. | S |
| LEG-005 | Zákon o kybernetické bezpečnosti (implementace směrnice **NIS2**) | Nemocnice jako regulovaný subjekt – bezpečnostní opatření, hlášení incidentů, řízení dodavatelů. | M |
| LEG-006 | **EHDS** (Nařízení o Evropském prostoru pro zdravotní data) | Požadavky na interoperabilitu a logování u systémů elektronických zdravotních záznamů; sledovat přechodná období. | S |
| LEG-007 | **AI Act** (Nařízení EU 2024/1689) | Pokud bude použit prediktivní model (FR-ALR-010) v rámci zdravotnického prostředku, jde o vysoce rizikový systém AI. | C |
| LEG-008 | Zákon č. 99/2019 Sb. o přístupnosti | Přístupnost webových stránek a mobilních aplikací veřejných subjektů (WCAG 2.1 AA). | M |
| LEG-009 | Hosting dat | Data musí být uložena v EU / EHP; s poskytovatelem cloudu uzavřena zpracovatelská smlouva (čl. 28 GDPR). | M |

**Doporučené normy a standardy:** ISO/IEC 27001 (řízení bezpečnosti informací), ISO 27799 (bezpečnost ve zdravotnictví), ISO/IEEE 11073 / Continua (zdravotnická zařízení), Bluetooth SIG Glucose Profile, HL7 FHIR R4, LOINC, SNOMED CT, UCUM, MKN-10.

---

## 7. Omezení a předpoklady

### 7.1 Omezení

- Podporovány budou pouze glukometry se standardním BLE Glucose Profile nebo s dostupným SDK výrobce.
- Přístup k datům o aktivitě závisí na platformních API (HealthKit, Health Connect) a na udělení oprávnění pacientem.
- Integrace s NIS je závislá na rozsahu FHIR rozhraní poskytovaného dodavatelem NIS.
- Systém není určen pro řešení akutních stavů a nenahrazuje tísňovou linku.
- Řešení musí být provozováno v infrastruktuře splňující požadavky NIS2 (on-premise nemocnice nebo certifikovaný cloud v EU).

### 7.2 Předpoklady

- Pacient vlastní chytrý telefon a je schopen jej ovládat (případně s pomocí pečovatele).
- Pacient má glukometr s Bluetooth nebo jej obdrží od poskytovatele.
- NIS poskytuje FHIR R4 API nebo bude doplněn o FHIR bránu.
- Poskytovatel zajistí organizační proces zpracování alertů (kdo, kdy, jak rychle reaguje).
- Pacient je zařazen do programu ošetřujícím lékařem.

---

## 8. Rizika projektu

| ID | Riziko | Pravděp. | Dopad | Opatření |
|---|---|---|---|---|
| R1 | Únik citlivých zdravotních dat | Nízká | Kritický | Šifrování, MFA, penetrační testy, DPIA, monitoring |
| R2 | Chybné vyhodnocení (falešně negativní alert) | Střední | Vysoký | Validace algoritmů, řízení rizik dle ISO 14971, klinické ověření, testy hraničních hodnot |
| R3 | Únava z alertů u personálu (alert fatigue) | Vysoká | Střední | Individuální prahy, deduplikace, prioritizace, denní souhrny |
| R4 | Nízká adopce u starších pacientů | Střední | Vysoký | UX testování s cílovou skupinou, edukace, podpora pečovatele |
| R5 | Nekompatibilita glukometrů | Střední | Střední | Seznam certifikovaných zařízení, ruční zadání jako záloha |
| R6 | Omezené FHIR rozhraní NIS | Střední | Vysoký | Včasná analýza s dodavatelem NIS, FHIR brána, konfigurovatelné mapování |
| R7 | Zpoždění certifikace dle MDR | Vysoká | Vysoký | Zapojení regulatorního specialisty od začátku, dokumentace dle IEC 62304 |
| R8 | Nejasná odpovědnost za reakci na alert | Střední | Vysoký | Smluvní a organizační vymezení, jasné informování pacienta |
| R9 | Změny platformních API (iOS/Android) | Střední | Střední | Abstrakční vrstva pro zdroje dat, sledování změn |

---

## 9. Akceptační kritéria

| ID | Kritérium | Ověřuje požadavky |
|---|---|---|
| AK-01 | Měření z podporovaného glukometru se po spárování přenese do aplikace bez ručního zadání a do 1 min je viditelné na portálu. | FR-MOB-010, 011, FR-API-004 |
| AK-02 | Opakovaná synchronizace téhož glukometru nevytvoří duplicitní záznamy. | FR-MOB-012, FR-API-004 |
| AK-03 | Měření pořízené offline se po obnovení připojení automaticky odešle a žádné se neztratí. | FR-MOB-044, NFR-AVL-003 |
| AK-04 | Při zadání glykémie 2,8 mmol/l se pacientovi okamžitě zobrazí doporučený postup a do 5 min vznikne alert vysoké závažnosti na portálu. | FR-MOB-050, FR-ALR-001, 004, NFR-PERF-005 |
| AK-05 | Lékař vidí pouze pacienty, kteří jsou mu přiřazeni; pokus o přístup k jinému pacientovi je odmítnut a zalogován. | FR-API-003, 008 |
| AK-06 | Denní kroky z HealthKit / Health Connect se zobrazí v aplikaci i na portálu. | FR-MOB-020, 021, FR-WEB-013 |
| AK-07 | Měření je odesláno do testovacího FHIR serveru jako validní `Observation` (validace proti profilu bez chyb). | FR-INT-002, 010 |
| AK-08 | Při nedostupnosti NIS jsou zprávy po obnovení spojení doručeny bez ztráty a bez duplicit. | FR-INT-008, 009 |
| AK-09 | Pacient může odvolat souhlas se sdílením a exportovat svá data. | FR-MOB-063, 064, LEG-001 |
| AK-10 | Penetrační test neodhalí zranitelnost s hodnocením High nebo Critical. | NFR-SEC-006, 007 |
| AK-11 | Uživatelské testování s ≥ 10 pacienty ve věku 50+ prokáže, že ≥ 80 % zvládne zprovoznění a první měření bez asistence. | NFR-USA-001, 006 |
