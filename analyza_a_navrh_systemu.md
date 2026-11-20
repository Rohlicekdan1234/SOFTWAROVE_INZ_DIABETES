# Analýza a návrh systému – Telemedicínský systém pro podporu pacientů s diabetem mellitus 2. typu

## Obsah

1. [Úvod](#1-úvod)
2. [Popis současného stavu a motivace](#2-popis-současného-stavu-a-motivace)
3. [Cíle projektu](#3-cíle-projektu)
4. [Zainteresované strany a uživatelské role](#4-zainteresované-strany-a-uživatelské-role)
5. [Rozsah systému](#5-rozsah-systému)
6. [Architektura – přehled komponent](#6-architektura--přehled-komponent)
7. [Případy užití](#7-případy-užití)

---

## 1. Úvod

### 1.1 Účel dokumentu

Tento dokument popisuje výchozí situaci, cíle, uživatele, rozsah a návrh architektury telemedicínského systému určeného pro dlouhodobé sledování pacientů s diabetem mellitus 2. typu (DM2). Definuje také případy užití, ze kterých vychází podrobná specifikace požadavků. Slouží jako společný referenční bod pro všechny zainteresované strany.

### 1.2 Rozsah dokumentu

Dokument pokrývá všech pět hlavních komponent systému:

1. mobilní aplikaci pro pacienta,
2. zabezpečené webové API (backend),
3. databázi,
4. webový portál pro zdravotnický personál,
5. integrátor pro komunikaci s nemocničním informačním systémem (NIS) prostřednictvím standardu HL7 FHIR.


## 2. Popis současného stavu a motivace

### 2.1 Současný stav

- Pacient s DM2 si měří glykémii glukometrem a hodnoty si zapisuje do papírového deníčku nebo je má uložené pouze v paměti glukometru.
- Data o pohybové aktivitě (kroky, sport) jsou v chytrých hodinkách / telefonu, ale lékař k nim nemá přístup.
- Lékař (diabetolog, praktický lékař) vidí data pouze při kontrole, typicky jednou za 3–6 měsíců.
- Rizikové epizody (hypoglykémie, dlouhodobá hyperglykémie) jsou odhaleny se zpožděním.
- Přepis dat do NIS je manuální, časově náročný a náchylný k chybám.

### 2.2 Identifikované problémy

| # | Problém | Dopad |
|---|---|---|
| P1 | Nekompletní a nespolehlivé záznamy glykémií | Chybná nebo zpožděná úprava terapie |
| P2 | Lékař nemá průběžný přehled o stavu pacienta | Pozdní reakce na zhoršení |
| P3 | Chybí propojení glykémie s pohybovou aktivitou | Nelze vyhodnotit vliv režimových opatření |
| P4 | Ruční přepis dat do NIS | Ztráta času personálu, chyby |
| P5 | Omezená komunikace mezi pacientem a lékařem mezi kontrolami | Nízká adherence k léčbě |

### 2.3 Motivace

Automatický sběr dat, jejich průběžné vyhodnocování a sdílení s ošetřujícím lékařem umožní včasnou intervenci, zlepšení kompenzace diabetu (snížení HbA1c), snížení počtu akutních komplikací a efektivnější využití času zdravotnického personálu.

---

## 3. Cíle projektu

### 3.1 Hlavní cíl

Navrhnout telemedicínský systém, který umožní automatický sběr, bezpečné uložení a vyhodnocení glykémií a údajů o fyzické aktivitě pacientů s DM2, včasnou identifikaci rizikových hodnot a efektivní komunikaci mezi pacientem a ošetřujícím lékařem, včetně interoperability s NIS přes HL7 FHIR.

### 3.2 Dílčí cíle

| ID | Cíl | Měřitelný ukazatel |
|---|---|---|
| C1 | Automatizovat sběr glykémií z glukometru | ≥ 90 % měření přeneseno bez ručního zadání |
| C2 | Integrovat data o fyzické aktivitě | Denní souhrn aktivity dostupný pro ≥ 95 % dní s nositelným zařízením |
| C3 | Včas detekovat rizikové hodnoty | Upozornění lékaři do 5 min od synchronizace rizikové hodnoty |
| C4 | Poskytnout lékaři přehledný dashboard | Posouzení stavu pacienta do 2 minut |
| C5 | Zajistit interoperabilitu s NIS | Data přenášena ve formátu HL7 FHIR R4 bez ručního přepisu |
| C6 | Zlepšit komunikaci pacient–lékař | Zabezpečené zprávy a doporučení dostupné v aplikaci |
| C7 | Zajistit bezpečnost a soulad s legislativou | Splnění GDPR, MDR a zákona o kybernetické bezpečnosti |

---

## 4. Zainteresované strany a uživatelské role

### 4.1 Zainteresované strany (stakeholders)

| Strana | Zájem / očekávání |
|---|---|
| Pacient s DM2 | Jednoduché měření, přehled o vlastním zdraví, rychlá reakce lékaře |
| Diabetolog / praktický lékař | Úplná a spolehlivá data, upozornění na rizika, úspora času |
| Zdravotní sestra / edukační sestra | Monitorování pacientů, komunikace, edukace |
| Rodinný příslušník / pečovatel | Volitelný náhled na stav blízké osoby (se souhlasem pacienta) |
| Poskytovatel zdravotních služeb (nemocnice) | Integrace s NIS, kvalita péče, soulad s legislativou |
| IT oddělení nemocnice | Bezpečnost, správa, provozovatelnost |
| Zdravotní pojišťovna | Prevence komplikací, nižší náklady (potenciálně úhrada telemedicíny) |
| Pověřenec pro ochranu osobních údajů (DPO) | Soulad s GDPR |
| Regulátor (SÚKL) | Soulad se směrnicí o zdravotnických prostředcích (MDR) |

### 4.2 Uživatelské role

| Role | Popis | Hlavní oprávnění |
|---|---|---|
| **Pacient** | Uživatel mobilní aplikace | Sběr a zobrazení vlastních dat, komunikace s lékařem, správa souhlasů |
| **Lékař** | Ošetřující lékař (diabetolog, PL) | Přehled přiřazených pacientů, nastavení cílových rozmezí, doporučení, export do NIS |
| **Sestra** | Zdravotní/edukační sestra | Přehled pacientů, zpracování alertů 1. úrovně, komunikace; bez změny terapie |
| **Pečovatel** | Osoba určená pacientem | Pouze čtení vybraných dat, příjem kritických upozornění |
| **Administrátor** | Správce systému | Správa uživatelů, rolí, konfigurace, audit; bez přístupu ke zdravotním datům |
| **Systém NIS** | Externí systém (technický účet) | Výměna dat přes FHIR rozhraní |

---

## 5. Rozsah systému

### 5.1 V rozsahu (in scope)

- Bezdrátový přenos glykémií z glukometru (Bluetooth Low Energy – Glucose Profile).
- Ruční zadání glykémie (záložní varianta).
- Import dat o fyzické aktivitě z Apple HealthKit a Google Health Connect.
- Zaznamenání doplňkových údajů: jídlo (kontext měření), léky, hmotnost, krevní tlak, subjektivní stav.
- Automatické vyhodnocení dat a generování upozornění (alertů).
- Webový portál pro zdravotnický personál s dashboardem a správou pacientů.
- Zabezpečená komunikace pacient–lékař.
- Obousměrná integrace s NIS přes HL7 FHIR R4.
- Reporty (PDF) pro kontrolu u lékaře.

### 5.2 Mimo rozsah (out of scope)

- Řízení inzulinových pump a automatické dávkování inzulínu.
- Kontinuální monitorování glukózy (CGM) – zvažováno pro další verzi.
- Videokonzultace – zvažováno pro další verzi.
- Elektronická preskripce (eRecept).
- Diagnostika diabetu (systém slouží výhradně ke sledování již diagnostikovaných pacientů).
- Fakturace zdravotním pojišťovnám.

---

## 6. Architektura – přehled komponent

```
 ┌───────────────┐   BLE    ┌──────────────────────┐
 │  Glukometr    │ ───────► │                      │
 └───────────────┘          │  Mobilní aplikace    │
 ┌───────────────┐ HealthKit│  pacienta            │
 │ Hodinky/telefon│ ───────►│  (iOS / Android)     │
 └───────────────┘ /Health  └──────────┬───────────┘
                   Connect             │ HTTPS (TLS 1.3), OAuth 2.0 / OIDC
                                       ▼
                         ┌───────────────────────────┐
                         │  Zabezpečené webové API   │
                         │  (REST, autentizace,      │
                         │   validace, alert engine) │
                         └──────┬─────────────┬──────┘
                                │             │
                     ┌──────────▼───┐   ┌─────▼───────────────┐
                     │  Databáze    │   │  FHIR integrátor    │
                     │  (šifrovaná) │   │  (mapování, fronta, │
                     └──────────────┘   │   retry, audit)     │
                                ▲       └─────┬───────────────┘
                                │             │ HL7 FHIR R4 (REST), SMART Backend Services
                     ┌──────────┴───────┐     ▼
                     │ Webový portál    │   ┌─────────────────────┐
                     │ pro zdravotníky  │   │ Nemocniční inform.  │
                     └──────────────────┘   │ systém (NIS)        │
                                            └─────────────────────┘
```

| Komponenta | Odpovědnost |
|---|---|
| Mobilní aplikace | Sběr dat (BLE, HealthKit/Health Connect, ruční zadání), offline úložiště, zobrazení trendů, notifikace, komunikace |
| Webové API | Autentizace a autorizace, příjem a validace dat, business logika, vyhodnocení (alert engine), poskytování dat portálu |
| Databáze | Perzistentní, šifrované úložiště zdravotních dat, uživatelů, souhlasů a auditních záznamů |
| Webový portál | Rozhraní pro lékaře a sestry – dashboard, detail pacienta, alerty, komunikace, reporty |
| FHIR integrátor | Transformace interních dat na FHIR zdroje a zpět, komunikace s NIS, fronta zpráv, opakování, logování |

---

## 7. Případy užití

### 7.1 Přehled aktérů a případů užití

| ID | Případ užití | Aktér |
|---|---|---|
| UC-01 | Registrace a spárování pacienta s lékařem | Pacient, Lékař |
| UC-02 | Spárování glukometru | Pacient |
| UC-03 | Automatická synchronizace glykémií | Pacient (systém) |
| UC-04 | Ruční zadání měření | Pacient |
| UC-05 | Synchronizace fyzické aktivity | Pacient (systém) |
| UC-06 | Zobrazení vlastních dat a trendů | Pacient |
| UC-07 | Vyhodnocení dat a vygenerování alertu | Systém |
| UC-08 | Zpracování alertu | Lékař, Sestra |
| UC-09 | Přehled pacientů (dashboard) | Lékař, Sestra |
| UC-10 | Nastavení individuálních cílových hodnot | Lékař |
| UC-11 | Komunikace pacient–lékař | Pacient, Lékař, Sestra |
| UC-12 | Odeslání dat do NIS | Systém, Lékař |
| UC-13 | Import demografických a klinických dat z NIS | Systém |
| UC-14 | Generování reportu | Lékař, Pacient |
| UC-15 | Správa souhlasů a sdílení dat | Pacient |
| UC-16 | Správa uživatelů | Administrátor |

### 7.2 Detailní popis vybraných případů užití

#### UC-03 Automatická synchronizace glykémií

| Položka | Popis |
|---|---|
| Aktér | Pacient, glukometr |
| Předpoklady | Pacient je přihlášen, glukometr je spárován, Bluetooth je zapnutý |
| Spouštěč | Pacient provede měření nebo otevře aplikaci; případně periodická synchronizace na pozadí |
| Hlavní scénář | 1. Aplikace detekuje glukometr v dosahu. 2. Načte nové záznamy (od posledního sekvenčního čísla). 3. Aplikace vyzve pacienta k doplnění kontextu (nalačno / před jídlem / po jídle / před spaním). 4. Data uloží lokálně. 5. Odešle data na API. 6. API data validuje, uloží a spustí vyhodnocení. 7. Aplikace zobrazí potvrzení a aktuální hodnotu s barevným hodnocením. |
| Alternativní scénáře | 5a. Není připojení k internetu → data zůstávají ve frontě a odešlou se po obnovení připojení. 6a. Hodnota mimo fyziologický rozsah → označena jako podezřelá, pacient vyzván k ověření. 6b. Duplicitní záznam → ignorován (idempotence). |
| Výsledek | Měření je uloženo v databázi a dostupné lékaři na portálu |

#### UC-07 Vyhodnocení dat a vygenerování alertu

| Položka | Popis |
|---|---|
| Aktér | Systém (alert engine) |
| Spouštěč | Uložení nového měření nebo plánovaná denní/týdenní analýza |
| Hlavní scénář | 1. Systém porovná hodnotu s individuálními cílovými rozmezími pacienta. 2. Vyhodnotí pravidla (jednorázové i trendové). 3. Při splnění podmínky vytvoří alert s úrovní závažnosti. 4. Upozorní pacienta (push notifikace s doporučením). 5. Upozorní zdravotnický personál dle úrovně závažnosti. 6. Zaznamená alert do auditního logu. |
| Výsledek | Alert je evidován, příslušné osoby jsou informovány |

#### UC-12 Odeslání dat do NIS

| Položka | Popis |
|---|---|
| Aktér | Systém (integrátor), Lékař |
| Předpoklady | Pacient je v NIS identifikován (spárován přes identifikátor), pacient udělil souhlas |
| Spouštěč | Plánovaná dávka (např. denně), ruční odeslání lékařem, nebo kritický alert |
| Hlavní scénář | 1. Integrátor vybere neodeslaná data. 2. Transformuje je na FHIR zdroje (`Observation`, `Flag`, `DiagnosticReport`). 3. Odešle FHIR `Bundle` (transaction) do NIS. 4. Zpracuje odpověď a uloží identifikátory vzniklých zdrojů. 5. Zaznamená výsledek do auditu. |
| Alternativní scénáře | 3a. NIS nedostupný → zpráva zůstává ve frontě, opakování s exponenciálním odstupem. 3b. Validační chyba → zpráva přesunuta do chybové fronty, notifikace administrátorovi. |
| Výsledek | Data jsou v NIS jako součást zdravotnické dokumentace pacienta |

