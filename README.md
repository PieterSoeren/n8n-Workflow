# n8n-Workflow

# README — AI Ontwerp-flow (n8n + Claude Design)

Deze README legt stap voor stap uit hoe je deze ontwerpflow gebruikt, ook als je nog nooit met n8n hebt gewerkt. Overal waar iets moeilijk te vinden kan zijn, staat precies beschreven waar je moet kijken en klikken.

Deze repository bevat twee bestanden: dit README-bestand, en het n8n-workflowbestand (`Multi-Agent_UI_Generation_Pipeline_To_Promt_with_Spec_tree_4_files_and_second_FLOW____Flow_2__Feedback_Verwerking_.json`). Hoofdstuk 2 legt uit wat je daarmee doet.

---

## Inhoudsopgave

1. [Wat deze flow eigenlijk doet](#1-wat-deze-flow-eigenlijk-doet)
2. [n8n opzetten (eenmalige installatie)](#2-n8n-opzetten-eenmalige-installatie)
   - [2.1 Welke accounts je nodig hebt](#21-welke-accounts-je-nodig-hebt)
   - [2.2 De flow importeren in n8n](#22-de-flow-importeren-in-n8n)
   - [2.3 De Anthropic-credential toevoegen in n8n](#23-de-anthropic-credential-toevoegen-in-n8n)
   - [2.4 Een bestaande agent aanpassen](#24-een-bestaande-agent-aanpassen)
   - [2.5 Een nieuwe agent toevoegen](#25-een-nieuwe-agent-toevoegen)
3. [Wat je vooraf nodig hebt (elke keer dat je de flow gebruikt)](#3-wat-je-vooraf-nodig-hebt-elke-keer-dat-je-de-flow-gebruikt)
4. [Ronde 1 — Het allereerste ontwerp maken](#4-ronde-1--het-allereerste-ontwerp-maken)
5. [Ronde 2 en verder — Een ontwerp verbeteren op basis van feedback](#5-ronde-2-en-verder--een-ontwerp-verbeteren-op-basis-van-feedback)
6. [Dingen die je moet weten voordat je vastloopt](#6-dingen-die-je-moet-weten-voordat-je-vastloopt)
7. [Bestandenoverzicht](#7-bestandenoverzicht)
8. [Bijlage: Technische troubleshooting (voor wie de flow onderhoudt)](#8-bijlage-technische-troubleshooting-voor-wie-de-flow-onderhoudt)

---

## 1. Wat deze flow eigenlijk doet

Kort gezegd: je vult in n8n een formulier in, en er komt aan de andere kant een set bestanden uit die je vervolgens in Claude Design plakt om een scherm-ontwerp te laten genereren. Krijg je later feedback op dat ontwerp? Dan doe je dit nog een keer met een tweede flow, en krijg je bijgewerkte bestanden terug.

```
Transcript/briefing
        │
        ▼
   FLOW 1 in n8n starten
        │
        ▼
Je krijgt 4 tekstbestanden terug
        │
        ▼
   Deze bestanden + een vaste tekst plak je in CLAUDE DESIGN
        │
        ▼
  Eerste ontwerp verschijnt
        │
        ▼
  Exporteer als PDF, laat zien, verzamel feedback (transcript maken)
        │
        ▼
   FLOW 2 in n8n starten
        │
        ▼
Je krijgt een zip-bestand terug met bijgewerkte versies
        │
        ▼
   Deze zip + een andere vaste tekst plak je opnieuw in CLAUDE DESIGN
        │
        ▼
  Verbeterd ontwerp verschijnt
        │
        └── Nieuwe feedback? Herhaal vanaf Flow 2
```

---

## 2. n8n opzetten (eenmalige installatie)

Dit hoofdstuk hoef je maar **één keer** te doorlopen — de eerste keer dat je met deze flow gaat werken, of wanneer je 'm op een nieuwe n8n-omgeving zet. Zodra dit gelukt is, sla je dit hoofdstuk voortaan over en begin je bij hoofdstuk 3.

### 2.1 Welke accounts je nodig hebt

Je hebt twee losse accounts nodig, bij twee verschillende partijen. Ze hebben niets met elkaar te maken en je logt er ook apart mee in.

**Account 1 — n8n**
n8n is de tool waarin de flow zelf draait. Er zijn twee mogelijke situaties:
- **Er bestaat al een gedeelde n8n-omgeving** (bijvoorbeeld beheerd door je bedrijf/IT). Vraag in dat geval bij je bedrijfscoach of IT-contactpersoon om een account of uitnodiging voor die omgeving. Dit is de meest waarschijnlijke situatie als je dit binnen een bestaand bedrijf gebruikt.
- **Er bestaat nog geen omgeving.** Ga dan naar **n8n.io**, klik op "Start free" of "Sign up", en maak een account aan voor de cloud-versie (geen installatie nodig, werkt direct in de browser).

**Account 2 — Anthropic API (Claude)**
Dit is niet hetzelfde account als waarmee je eventueel al op claude.ai chat — de API werkt via een apart, los ontwikkelaarsportaal:
- Ga naar **console.anthropic.com** (let op: dus niet claude.ai — dat is de gewone chatapp, dit is de aparte API-console).
- Maak een account aan of log in.
- Je hebt hier een **betaalmethode** (creditcard) nodig, want de API werkt op basis van betalen-per-gebruik, niet via een vast maandabonnement zoals de consumenten-app.
- Ga na het inloggen naar het onderdeel **"API Keys"** in het linkermenu.
- Klik op **"Create Key"**, geef de key een herkenbare naam (bijvoorbeeld "Flow Secret Ninjas"), en klik nogmaals bevestigen.
- **Kopieer de key meteen en bewaar 'm ergens veilig** (bijvoorbeeld een wachtwoordmanager). De volledige key wordt maar één keer getoond — sluit je dit scherm zonder 'm te kopiëren, dan moet je een nieuwe key aanmaken.

### 2.2 De flow importeren in n8n

**Stap 1 — Download het JSON-bestand uit deze git-repository.** Het heet `Multi-Agent_UI_Generation_Pipeline_To_Promt_with_Spec_tree_4_files_and_second_FLOW____Flow_2__Feedback_Verwerking_.json` en staat in dezelfde map als deze README.

**Stap 2 — Open n8n en klik op de knop om een nieuwe, lege workflow te maken** (meestal rechtsboven of linksboven, met een "+" of de tekst "Add workflow").

**Stap 3 — Klik rechtsboven in het canvas op het menu met de drie puntjes (•••), en kies "Import from File"** (heet in sommige n8n-versies ook "Import from URL", gebruik dan de "from File"-variant).

**Stap 4 — Selecteer het JSON-bestand dat je in stap 1 hebt gedownload.** Het canvas vult zich automatisch met alle blokjes van zowel Flow 1 als Flow 2.

**Stap 5 — Sla de workflow op** (via Ctrl+S, of het menu "Save").

### 2.3 De Anthropic-credential toevoegen in n8n

De API-key uit stap 2.1 moet je nu koppelen aan n8n, zodat de flow 'm kan gebruiken.

**Stap 1 — Klik in de linker menubalk van n8n op "Credentials"** (soms verstopt onder een tandwiel-/instellingenicoon, afhankelijk van je n8n-versie).

**Stap 2 — Klik rechtsboven op "+ Add Credential"** (of een vergelijkbare knop met een "+").

**Stap 3 — Zoek in de lijst naar "Anthropic" en selecteer "Anthropic API".**

**Stap 4 — Plak je API-key** (uit stap 2.1) **in het veld "API Key".**

**Stap 5 — Noem de credential precies "Anthropic account"** — dit is de naam waar alle blokjes in de flow naar zoeken, dus zo hoef je verder niets handmatig te wijzigen.

**Stap 6 — Klik op "Save".**

**Stap 7 — Ga terug naar het canvas met de geïmporteerde flow.** Blokjes die deze credential nodig hebben (te herkennen aan een rood uitroepteken totdat het goed staat) moeten 'm nu automatisch oppikken als de naam overeenkomt. Staat er ergens toch nog een rood uitroepteken?
- Dubbelklik op dat blokje.
- Zoek het veld "Credential to connect with" (of vergelijkbaar).
- Selecteer **"Anthropic account"** uit de dropdown.
- Sluit het zijscherm en sla de workflow op.

Doe dit voor elk blokje dat er nog een rood uitroepteken bij heeft staan. Dit geldt zowel voor de "Claude Model"-blokjes als voor de twee losse HTTP-blokjes ("Screenshot Interpreter" en "Design PDF Interpreter") — die gebruiken dezelfde credential, maar dan via een net iets ander veld genaamd "Predefined Credential Type".

### 2.4 Een bestaande agent aanpassen

Wil je dat een AI-stap in de flow zich anders gedraagt (bijvoorbeeld strenger, milder, of met andere criteria)? Dat pas je aan zonder ergens code te hoeven schrijven.

**Stap 1 — Zoek in het canvas het blokje van de agent die je wilt aanpassen** (bijvoorbeeld "AI Agent A: Kritiek Structureren" — te herkennen aan het robot-icoontje).

**Stap 2 — Dubbelklik op dat blokje.** Er opent een groot zijscherm met invulvelden.

**Stap 3 — Zoek het veld "Prompt (User Message)".** Dit is de tekst die precies beschrijft welke data/informatie de agent als input krijgt. Pas dit alleen aan als je wil veranderen *welke gegevens* de agent ziet.

**Stap 4 — Scroll naar beneden naar "Options" en klik dat open, en zoek het veld "System Message".** Dit is de eigenlijke instructie: hoe de agent zich moet gedragen, wat 'ie wel/niet mag doen, en in welk formaat het antwoord moet komen. Dit pas je aan als je het gedrag van de agent wil bijsturen.

**Stap 5 — Typ je aanpassing direct in het tekstveld.** Er is geen aparte opslaanknop nodig binnen dit zijscherm — de tekst wordt vastgehouden zodra je het veld verlaat.

**Stap 6 — Sluit het zijscherm** (kruisje rechtsboven, of ergens buiten het scherm klikken) **en sla de hele workflow op** (Ctrl+S), zodat je wijziging ook echt bewaard blijft.

**Stap 7 — Test je aanpassing** door de flow opnieuw te doorlopen (zie hoofdstuk 4 of 5), en controleer of de agent zich nu gedraagt zoals bedoeld.

### 2.5 Een nieuwe agent toevoegen

Wil je een extra AI-stap toevoegen aan de flow (bijvoorbeeld een aparte controle-agent)? Het makkelijkst is om een bestaande, vergelijkbare agent te kopiëren en aan te passen, in plaats van helemaal opnieuw te beginnen.

**Stap 1 — Klik één keer op een bestaand agent-blokje dat qua opzet lijkt op wat je wil toevoegen** (bijvoorbeeld "AI Agent B: Design Reviser"), zodat het geselecteerd is (je ziet een gekleurde rand verschijnen).

**Stap 2 — Selecteer ook het bijbehorende "Claude Model"-blokje eronder** (houd Shift ingedrukt en klik erop) — een agent heeft namelijk altijd een eigen gekoppeld model-blokje nodig.

**Stap 3 — Druk op Ctrl+C om te kopiëren, en daarna Ctrl+V om te plakken.** Er verschijnt een duplicaat van beide blokjes, iets verschoven ten opzichte van het origineel, ergens op een lege plek in het canvas.

**Stap 4 — Versleep het duplicaat naar een logische, lege plek in het canvas** (bijvoorbeeld eronder of ernaast), zodat het niet over andere blokjes heen ligt.

**Stap 5 — Hernoem beide nieuwe blokjes** zodat je ze makkelijk herkent: dubbelklik op de titel bovenin het blokje (niet op het blokje zelf, maar op de tekst van de naam) en typ een nieuwe naam, bijvoorbeeld "AI Agent F: Kwaliteitscontrole".

**Stap 6 — Verbind de nieuwe agent met de rest van de flow:**
- Zoek het kleine bolletje aan de rechterkant van het blokje dat vóór je nieuwe agent moet komen (dus waar de input vandaan komt).
- Klik daarop, houd ingedrukt, en sleep de lijn naar het kleine bolletje aan de linkerkant van je nieuwe agent-blokje.
- Doe hetzelfde voor de uitvoer: sleep vanaf de rechterkant van je nieuwe agent naar de linkerkant van het blokje dat erna moet komen.

**Stap 7 — Controleer dat het gekopieerde model-blokje nog steeds gekoppeld is aan je nieuwe agent.** Onder het agent-blokje zie je een klein aansluitpunt genaamd "Model" — hier moet een lijntje naar het Claude Model-blokje lopen. Is dat lijntje losgeraakt tijdens het verslepen? Sleep het dan opnieuw vast, van het bolletje onder de agent naar het bolletje boven het model-blokje.

**Stap 8 — Dubbelklik op je nieuwe agent-blokje en pas de "Prompt (User Message)" en de "System Message" aan** zoals beschreven in stappen 3-5 van hoofdstuk 2.4, zodat de tekst past bij wat deze nieuwe agent moet doen.

**Stap 9 — Check de credential van het nieuwe model-blokje** (zie hoofdstuk 2.3, stap 7) — een gekopieerd blokje neemt de credential-koppeling meestal automatisch over, maar controleer het voor de zekerheid.

**Stap 10 — Sla de workflow op (Ctrl+S) en test de flow opnieuw** om te checken of je nieuwe agent goed meedraait.

Klaar met dit hoofdstuk? Dan kun je verder met hoofdstuk 3 om de flow daadwerkelijk te gebruiken.

---

## 3. Wat je vooraf nodig hebt (elke keer dat je de flow gebruikt)

- Toegang tot de n8n-omgeving met het geïmporteerde canvas (zie hoofdstuk 2 als dit de eerste keer is). Flow 1 en Flow 2 staan op **hetzelfde canvas** — Flow 2 staat verderop (rechts) van Flow 1.
- Een account voor Claude Design.
- Voor de eerste ronde: een transcript of briefing van het gesprek met de opdrachtgever.
- Vanaf de tweede ronde: een transcript van het feedbackgesprek, en het huidige ontwerp als PDF.

---

## 4. Ronde 1 — Het allereerste ontwerp maken

**Stap 1 — Open n8n in je browser en klik op het canvas dat bij jouw project hoort** (bijvoorbeeld "Multi-Agent UI Generation Pipeline"). Je komt dan op een groot, donker canvas met blokjes ("nodes") die met lijntjes aan elkaar verbonden zijn. Helemaal links op dit canvas staat de starttrigger van Flow 1 (het allereerste blokje van de rij).

**Stap 2 — Zoek onderin het scherm de oranje knop met "Execute workflow" erop, en klik daarop.** Omdat je nu bij het allereerste blokje van het hele canvas begint, start deze knop hier gewoon de juiste flow (Flow 1). *(Let op: bij Ronde 2 en verder gebruik je een andere manier om te starten — zie hoofdstuk 5, daar mag je deze knop juist niet gebruiken.)*

**Stap 3 — Er verschijnt een formulier**, ofwel er opent een nieuw tabblad met een formulier, ofwel het verschijnt als een pop-up in beeld. Dit ziet eruit als een gewone webpagina: een titel bovenaan, en daaronder invulvelden.

**Stap 4 — Vul het transcript of de briefingtekst in het (grote) tekstveld in**, en klik onderaan het formulier op de verzendknop (vaak "Submit" of "Verzenden").

**Stap 5 — Kijk naar het canvas.** Je ziet de blokjes van links naar rechts één voor één **groen oplichten met een vinkje** ✓ zodra ze klaar zijn. Zolang er steeds een volgend blokje groen wordt, is de flow aan het werk. Dit kan een minuut of langer duren — gewoon laten lopen.

**Stap 6 — Wacht tot het allerlaatste blokje helemaal rechts ook groen is.** Dat is meestal een blokje met een naam als "Respond With File".

**Stap 7 — Dubbelklik op dat laatste blokje.** Er opent een zijscherm met tabbladen bovenin, waaronder "Output". Klik op **"Output"** als dat niet al geselecteerd is.

**Stap 8 — In dat Output-scherm zie je een kaart met een bestandsnaam erop, en een knop "Download".** Klik op **Download** om het bestand op je eigen computer op te slaan. Dit is niet één bestand maar eigenlijk 4 losse bestanden die je één voor één ziet (of als kaartjes naast elkaar):
- `design-brief.md` — de eigenlijke bouwinstructie
- `00-project-overview.md` — achtergrondinformatie
- `01-epics-user-stories.md` — bredere functionaliteit
- `02-toekomstvisie.md` — toekomstplannen

Download ze alle 4 en zet ze in een map die je makkelijk terugvindt — je hebt ze later weer nodig.

**Stap 9 — Ga naar Claude Design en open een nieuw project.**

**Stap 10 — Onderin het scherm zit een groot invoerveld met een placeholdertekst erin** (zoiets als "Draft a one-page project brief"). Klik daarin en plak de volgende tekst:

> Je krijgt hierbij een aantal documenten aangeleverd. Deze hebben niet allemaal dezelfde rol — lees ze in deze volgorde en behandel ze verschillend:
>
> **design-brief.md — dit is je bouwinstructie.** Bouw wat er in dit bestand staat beschreven: de genoemde componenten, states, stijlrichting, interactiepatronen en toegankelijkheidseisen. Voeg niets toe dat niet in dit bestand staat, ook niet als het wel in de andere bestanden voorkomt.
>
> **00-project-overview.md — dit is achtergrondcontext, geen bouwinstructie.** Gebruik dit alleen om de bredere context, toon en prioriteiten van het product te begrijpen (waarom bestaat dit, wie gebruikt het, wat is de grotere visie). Bouw hier niets direct uit — als iets hierin niet ook in de design brief staat, hoort het niet bij deze opdracht.
>
> **01-epics-user-stories.md — puur ter oriëntatie, niet voor deze opdracht.** Dit toont hoe de volledige productfunctionaliteit is opgedeeld in epics, inclusief functionaliteit die buiten deze opdracht valt. Gebruik dit hooguit om te begrijpen hoe deze opdracht zich verhoudt tot de rest van het product — bouw niets uit dit bestand dat niet ook expliciet in de design brief staat.
>
> **02-toekomstvisie.md (indien aanwezig) — puur ter oriëntatie.** Dit beschrijft toekomstige uitbreidingen die nu nog niet gebouwd worden. Gebruik dit hooguit om in het ontwerp ruimte/flexibiliteit te laten voor latere groei — bouw geen van de genoemde toekomstfeatures nu al.
>
> Als er een verschil of tegenstrijdigheid is tussen de design brief en de andere bestanden, heeft de design brief altijd voorrang.
>
> Bouw wat er in de design brief staat.

**Stap 11 — Links van (of onder) het invoerveld zit een "+"-knopje.** Klik daarop en kies de 4 bestanden die je in stap 8 hebt gedownload, of sleep ze zo vanaf je bureaublad in het invoerveld.

**Stap 12 — Zoek naast het invoerveld (vaak rechts, naast waar het model straks komt) een tweede dropdown met een naam als "Design system".** Controleer dat hier het juiste design system voor dit project geselecteerd staat (bijvoorbeeld "MetAFib Design System"). Staat dit verkeerd of leeg? Kies dan het juiste system uit de lijst — dit bepaalt de kleuren, typografie en componentstijl die Claude Design gebruikt, dus een verkeerde keuze hier geeft een resultaat dat er stilistisch anders uitziet dan bedoeld, ook al is de inhoud correct.

**Stap 13 — Rechts (of onder) het invoerveld staat een dropdown-menu met daarin de naam van een model.** Klik daarop en **kies Fable 5.1** — dit levert het meest uitgewerkte, kwalitatief beste resultaat op. Dit duurt wel merkbaar langer dan andere modellen (zie hoofdstuk 6, punt 2, voor de afweging).

**Stap 14 — Helemaal rechts van het invoerveld zit een rond knopje met een pijltje omhoog erin.** Klik daarop om de generatie te starten.

**Stap 15 — Links verschijnt een lijstje met regels die vanzelf bijvullen** (zoiets als "Reading design-brief.md", "Building components"). Zolang die lijst blijft groeien, is Claude Design aan het werk. Rechts in het canvas verschijnt na een tijdje het ontwerp zelf.

**Stap 16 — Ben je tevreden met het ontwerp?** Zoek naar een deel- of exportmogelijkheid (vaak via een menu-icoon of de "Share"-knop rechtsboven) en **exporteer als PDF** — niet als HTML (zie hoofdstuk 6, punt 1).

---

## 5. Ronde 2 en verder — Een ontwerp verbeteren op basis van feedback

### Wat je hiervoor klaarlegt
- De 4 bestanden van de vorige ronde (bij ronde 2 zijn dit de originelen uit ronde 1; bij ronde 3+ zijn dit de bijgewerkte bestanden uit de zip van de vorige ronde).
- Een transcript van het feedbackgesprek — volledige tekst, niet leeg, als .txt, .md of .pdf.
- Optioneel: een screenshot met aantekeningen van het huidige ontwerp.
- Het huidige ontwerp als PDF (zoals geëxporteerd in stap 15 hierboven).

### Stappen

**Stap 1 — Open n8n.** Flow 1 en Flow 2 staan op **hetzelfde canvas**, niet in twee losse workflows — Flow 2 staat verderop, rechts van de blokjes die je bij Ronde 1 gebruikte. Scroll (of zoom uit) naar het blokje dat **"Start Feedback Flow"** heet.

> **Let op — klik hier NIET op de grote oranje "Execute workflow"-knop onderin het scherm.** Die knop start het canvas vanaf het allereerste blokje (Flow 1), en dan begin je per ongeluk weer helemaal opnieuw in plaats van bij Flow 2.

**Stap 2 — Beweeg je muis over het blokje "Start Feedback Flow" zelf.** Er verschijnt een klein afspeel-icoontje (▷) direct op of naast dat blokje. Klik op **dat kleine icoontje** — dit start alleen Flow 2, vanaf dat specifieke punt in het canvas.

**Stap 3 — Het formulier dat verschijnt heeft dit keer meerdere upload-velden**, elk met een eigen label erboven. Klik per veld op de knop (vaak "Choose file" of een upload-icoon) en selecteer het bijbehorende bestand:
- Eén veld per van de 4 bestanden uit de vorige ronde
- Eén veld voor het huidige ontwerp — hier upload je de PDF
- Eén veld voor het feedback-transcript
- Eén veld voor een screenshot met aantekeningen — laat leeg als je die niet hebt

**Stap 4 — Klik onderaan het formulier op de verzendknop.**

**Stap 5 — Kijk weer naar het canvas: dezelfde groene vinkjes verschijnen van links naar rechts.** Dit duurt meestal langer dan Flow 1 (er lopen nu meerdere AI-stappen tegelijk) — een paar minuten wachten is normaal.

**Stap 6 — Dubbelklik op het laatste blokje** (weer iets als "Respond With File") **en ga naar het tabblad "Output".**

**Stap 7 — Klik op de "Download"-knop bij het bestand `updated-documents.zip`.**

> **Let op:** deze zip hoef je **niet uit te pakken**. Je kan het zip-bestand in gezipte vorm direct als bijlage meegeven aan Claude Design in stap 10 hieronder — die kan de inhoud ervan gewoon lezen. Het is normaal dat, eenmaal verwerkt, 3 van de 4 bestanden erin gewoon "No changes" blijken te zeggen. Dat betekent dat de feedback alleen over het ontwerp zelf ging, niet over de bredere projectinhoud.

**Stap 8 — Ga terug naar Claude Design, bij voorkeur in hetzelfde project/gesprek als de vorige ronde.**

**Stap 9 — Klik weer in het invoerveld onderin en plak deze tekst:**

> Je krijgt hierbij een aantal documenten aangeleverd. Deze hebben niet allemaal dezelfde rol — herken ze aan hun inhoud (de exacte bestandsnaam kan verschillen) en behandel ze verschillend:
>
> **De meest recente design brief — dit is je bouwinstructie.** Dit is het document met "Design Brief" in de titel dat bovenaan een sectie "Changes in this version" (of "Wijzigingen in deze versie") bevat — dat is de bijgewerkte versie, ontstaan naar aanleiding van een feedbackronde. Bouw wat er in dit document staat beschreven. Voeg niets toe dat niet in dit document staat, ook niet als het wel in de andere documenten voorkomt.
>
> **De vorige design brief — dit is referentiemateriaal, geen bouwinstructie.** Dit is het andere document met "Design Brief" in de titel, zonder een "Changes in this version"-sectie. Gebruik dit alleen om te begrijpen wat de bestaande basis is. Op onderdelen die niet genoemd worden in de "Changes in this version"-sectie van de meest recente brief, blijft dit de geldende beschrijving. Waar de twee documenten verschillen, heeft de meest recente brief altijd voorrang.
>
> **Het project overview-document — dit is achtergrondcontext, geen bouwinstructie.** Gebruik dit alleen om de bredere context, toon en prioriteiten van het product te begrijpen. Bouw hier niets direct uit.
>
> **Het epics/user-stories-document — puur ter oriëntatie, niet voor deze opdracht.** Gebruik dit hooguit om te begrijpen hoe deze opdracht zich verhoudt tot de rest van het product.
>
> **Het toekomstvisie-document (indien aanwezig) — puur ter oriëntatie.** Gebruik dit hooguit om in het ontwerp ruimte/flexibiliteit te laten voor latere groei.
>
> **Een PDF of afbeelding van een eerder gebouwd design (indien aanwezig).** Gebruik dit als visuele referentie voor de huidige stijl en structuur, niet als bouwinstructie.
>
> Bij twijfel of tegenstrijdigheid geldt altijd deze volgorde: de meest recente design brief eerst, dan de vorige design brief voor alles wat niet expliciet is gewijzigd, en pas daarna de overige documenten als achtergrond.
>
> Bouw wat er in de meest recente design brief staat, voortbouwend op wat er volgens de vorige design brief al bestond voor de onderdelen die niet zijn aangepast.

**Stap 10 — Klik weer op het "+"-knopje en voeg toe:** de zip uit stap 7 (in gezipte vorm, niet uitgepakt), de **oude** design-brief van de vórige ronde (dus niet de nieuwe uit de zip — de versie van daarvoor), en de PDF van het huidige ontwerp.

**Stap 11 — Check het model (moet op Fable 5.1 staan) en het Design System (beide zie stap 12-13 uit hoofdstuk 4), en klik op het pijltje-knopje om te versturen.**

**Stap 12 — Bekijk het resultaat, exporteer opnieuw als PDF wanneer je tevreden bent.**

**Stap 13 — Nieuwe feedbackronde?** Begin gewoon opnieuw bij stap 1 van dit hoofdstuk.

---

## 6. Dingen die je moet weten voordat je vastloopt

1. **Gebruik altijd PDF, nooit HTML, als je het huidige ontwerp aanlevert.** De HTML-export van Claude Design bevat vrijwel geen leesbare tekst — het is een technisch bestand dat pas in een browser een pagina wordt. Een PDF bevat wél gewoon leesbare tekst.

2. **Gebruik Fable 5.1 als je het beste, meest uitgewerkte resultaat wilt — accepteer dat dit langer duurt.** Fable 5.1 denkt langer na en werkt dingen grondiger uit, wat precies is waarom het langzamer en duurder is dan andere modellen — dat is een bewuste, terechte afweging, geen nadeel om te vermijden.

   **Welk model wanneer gebruiken (stand van dit moment — dit verandert ongetwijfeld naarmate er nieuwe modellen uitkomen):**
   - **Fable 5.1** — voor het uiteindelijke, uitgebreide ontwerp dat je aan een opdrachtgever laat zien of dat als eindresultaat moet gelden. Duurt langer, maar levert de meest doordachte en volledige output.
   - **Opus 5.0** — voor snel itereren: even testen of een aanpassing werkt, snel een tussentijdse versie bekijken, of gewoon checken of de flow zelf goed doorloopt zonder te wachten op een volledige, uitgebreide generatie.

   Kortom: **Opus voor tempo tijdens het werken, Fable 5.1 voor het uiteindelijke resultaat.** Loop je tegen kosten- of tijdslimieten aan tijdens het testen, schakel dan tijdelijk naar Opus 5.0 en zet 'm terug naar Fable 5.1 zodra je de definitieve versie genereert.

3. **Test met een écht transcript, niet met een leeg testbestand.** Zegt het systeem dat er "geen feedback" is verwerkt? Dan was de input zelf waarschijnlijk leeg of te summier — dat is geen fout, de flow verzint dan terecht niets.

4. **Loopt iets vast of krijg je een rode foutmelding op een blokje in het canvas?** Maak een screenshot van dat blokje (dubbelklik erop, zodat je ook de foutmelding in het zijscherm ziet) en stuur die door aan: **[NAAM/CONTACT INVULLEN — wie de flow na deze afstudeeropdracht onderhoudt]**. Zie ook hoofdstuk 8 voor een lijst met bekende technische problemen en hun oplossing.

5. **Onderaan elk gegenereerd bestand (design-brief, project-overview, etc.) staat soms een blokje tussen `<!-- DEBUG INFO -->` en `-->`.** Dit is een onzichtbare technische notitie (een HTML-comment) die per ronde laat zien wat er precies als transcript is binnengekomen — handig om te checken of de flow de juiste feedback heeft gebruikt. Dit blokje is volledig onschadelijk: het wordt niet getoond als het bestand ergens gerenderd/weergegeven wordt, en je mag het negeren of desgewenst verwijderen voordat je het bestand doorstuurt.

---

## 7. Bestandenoverzicht

| Bestand | Wat het is | Wanneer gebruik je het |
|---|---|---|
| `design-brief.md` | Bouwinstructie voor het scherm | Altijd meesturen naar Claude Design |
| `00-project-overview.md` | Achtergrond over het product | Altijd meesturen naar Claude Design |
| `01-epics-user-stories.md` | Bredere functionaliteit | Altijd meesturen naar Claude Design |
| `02-toekomstvisie.md` | Toekomstplannen | Altijd meesturen naar Claude Design |
| `updated-documents.zip` | Wat Flow 2 teruggeeft na een feedbackronde | Direct (ongeopend) als bijlage meegeven aan Claude Design |
| Design-PDF | Het huidige ontwerp | Meesturen naar Flow 2, en als bijlage in Claude Design vanaf ronde 2 |
| `Multi-Agent_UI_Generation_Pipeline_...json` | Het n8n-workflowbestand zelf | Eenmalig importeren in n8n, zie hoofdstuk 2 |

---

## 8. Bijlage: Technische troubleshooting (voor wie de flow onderhoudt)

Dit hoofdstuk is bedoeld voor wie in de n8n-nodes zelf gaat kijken/aanpassen, niet voor dagelijks gebruik. Het bevat concrete problemen die zijn tegengekomen tijdens het bouwen van deze flow, met de exacte oorzaak en oplossing — zodat je niet opnieuw hoeft uit te zoeken wat er al eerder is gevonden.

| Foutmelding / symptoom | Oorzaak | Oplossing |
|---|---|---|
| "Credentials not found" op een HTTP-node (Screenshot Interpreter / Design PDF Interpreter) | Node had nog geen credential gekoppeld | Koppel de bestaande "Anthropic account"-credential via **Predefined Credential Type** (niet via een losse API-key-credential) |
| Transcript of andere data komt leeg aan bij een agent, terwijl de input zelf wél inhoud had | Een Set-node ("No Screenshot Fallback" e.d.) had `Include Other Input Fields` niet aanstaan | Zet die optie expliciet aan. **Bevestigd in de n8n-broncode:** deze optie staat standaard op `false` — zonder expliciete aanpassing verdwijnen alle andere velden stilzwijgend |
| "Invalid HTML: could not find" bij het uitlezen van een design-bestand | De `html`-operation van Extract From File verwacht een specifieke structuur/selector, niet "geef me alle leesbare tekst" | Gebruik deze operation niet voor dit doel — deze flow gebruikt inmiddels PDF + een vision-stap (Design PDF Interpreter) in plaats van HTML-tekstextractie |
| Geëxtraheerde design-tekst is onleesbare brij/rommeltekst | Code-node ging ervanuit dat `binary.propertyName.data` al base64 was; dat klopt niet in elke n8n-opslagconfiguratie | Gebruik `this.helpers.getBinaryDataBuffer(0, 'propertyName')` in plaats van die aanname — werkt ongeacht hoe binary data is opgeslagen |
| `$helpers is not defined` in een Code-node | Versieverschil in n8n: sommige versies kennen alleen `this.helpers`, niet het losse `$helpers` | Vervang `$helpers.…` door `this.helpers.…` |
| Binary bestanden (bijv. `md2File`) zijn plotseling "niet gevonden" verderop in de flow, terwijl ze er eerder wel waren | Een eigen Code-node gaf alleen `{ json: {...} }` terug zonder `binary` erbij — Code-nodes geven binary data NIET automatisch door | Geef in de `return` van de Code-node altijd expliciet `binary: $input.item.binary` mee (of `binary: $('EerdereNode').item.binary` als je bewust van een eerder punt wil herstellen) |
| Extract From File met de `pdf`-operation lijkt data te laten verdwijnen | Vaak een verkeerde diagnose — de `pdf`-operation combineert standaard (`keepSource: 'json'`) juist netjes met bestaande JSON-data en verwijdert alleen het net-verwerkte binary-bestand zelf, niet alle andere velden/bestanden | Zoek de echte oorzaak eerst bij Set- of Code-nodes verderop in de keten voordat je deze operation zelf gaat wantrouwen |
| "No path back to referenced node" in een Agent-node | Een expressie (`$('NodeNaam')`) verwijst naar een node die niet in dezelfde keten vóór deze node ligt — bijvoorbeeld een node op een parallelle tak | Verwijder de verwijzing, of herstructureer de flow zodat de node waarnaar je verwijst daadwerkelijk stroomopwaarts van de huidige node ligt |
| Zip-bestand mist bestanden, of Compression-node geeft een fout over een ontbrekend binary-veld | De namen in "Input Binary Field(s)" van de Zip-node komen niet exact overeen met de werkelijke binary-veldnamen van de binnenkomende data | Open de node vóór de Zip-node, bekijk het tabblad "Binary" om de exacte veldnamen te zien, en zorg dat "Input Binary Field(s)" die namen letterlijk (kommagescheiden) overneemt |
| Flow/generatie blijft lang "denken" op Fable 5.1 | Normaal gedrag op hogere effort-niveaus (bijvoorbeeld "Max"), bedoeld voor lange, zelfstandig draaiende taken | Geen fout — laat het lopen, of kies bewust een lager effort-niveau als snelheid belangrijker is dan maximale uitwerking |
| "Spend limit reached" in Claude Design | Maandelijks kredietplafond van het account/de organisatie is bereikt | Werk tot dat punt is niet verloren (zie "Work so far is saved"). Vraag de beheerder het plafond te verhogen, of wacht op de eerstvolgende maandelijkse reset |
