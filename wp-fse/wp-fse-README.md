# wp-fse — Claude Skill for WordPress Full Site Editing

> This file contains two versions of the documentation:
> **English** (below) and **Italian** (further down, starting at *Documentazione in italiano*).
>
> **Author / Autore:** Marco Antonio Cavalli  
> **Version / Versione:** v4.0 — derived from `SKILL.md` v4.0 (June 2026)

---

## English documentation

### What it does

When activated, this skill gives Claude a strict, opinionated workflow for FSE development:

- **Marsland Principle enforcement** — template = structure only, content lives in pages. The skill detects and corrects the "Morange" anti-pattern (editorial content mixed into templates).
- **Bootstrap check** — auto-detects Docker/shared hosting environment, active child theme, and whether the project is new or resuming.
- **Hybrid Recon** — collects site data via WP-CLI before writing a single file.
- **15-step build sequence** — from folder structure to browser responsive check.
- **Absolute rules** — guardrails against `wp:html` for layout, inline CSS, fixed `px` units, template recursion, and other FSE anti-patterns.
- **Blocks reference** — quick lookup table mapping section types to correct native Gutenberg blocks.
- **Four content widths** — normal, wide, full, custom (based on Marsland's video series).
- **Responsive checklist** — mandatory verification at 375px · 768px · 1024px · 1440px.
- **Session changelog** — every file written or modified is logged automatically in the project file, so resuming work always starts from a clear picture of what changed.

### Who it's for

WordPress developers and site builders who:
- Work with FSE / block themes (not classic themes)
- Use a child theme workflow with `theme.json`, `templates/*.html`, `parts/*.html`, `patterns/*.php`
- Run WordPress in Docker (Coolify, local) or on shared hosting
- Want Claude to follow a consistent, principled FSE methodology rather than improvising

### Activation triggers

The skill activates automatically when you mention: `WordPress FSE`, `FSE`, `Gutenberg`, `child theme`, `block theme`, `theme.json`, `full site editing`, `wp-fse`, `WordPress template`, `sticky header`, `masthead`, `WordPress pattern`, `archive CPT`, `edit homepage`, `page content`, `content width`.

### How to install

1. Download `SKILL.md` from this folder
2. In Claude Cowork, go to **Settings → Capabilities → Skills**
3. Click **Install skill** and select the file

Or, if you have the `.skill` package (zip of this folder with `.skill` extension), click the **Save skill** button directly in the Cowork chat interface.

### Key concepts

#### The Marsland Principle
Introduced by [Jamie Marsland](https://www.youtube.com/@pootlepress) in his FSE video series. The core idea:

> Template = orange juice. Content = milk. Mixed = Morange. Never make Morange.

Templates hold structure (header, content area, footer). Pages hold editorial content. The skill enforces this separation at every step.

#### The Morange Test
At the start of every session on an existing site, the skill runs a quick check: click "Edit page" in the WP admin bar. If you land in the template editor instead of the page editor, you have the Morange problem and must fix it before proceeding.

> 🙏🙏 Thank you Jamie — your videos pushed me to embrace WordPress Gutenberg and FSE at a moment of crisis.

#### Project files and session changelog
The skill maintains a per-project `.md` file tracking theme slug, design system values, CPTs, page/template mapping, build progress, and a **session changelog** (§12.6). Every file written or modified during a session is logged with date, session number, file path, and a short description. When resuming work, the skill reads the last 5 changelog entries so the context is immediately clear — no need to remember what was done in a previous session.

Site name is detected automatically via WP-CLI (`siteurl` option) in Docker environments, or from the REST API base URL when working with application passwords. Manual input is only a fallback.

### Stack assumptions

- WordPress 6.x / 7.x
- Child theme (FSE block theme)
- Docker via Coolify, or shared hosting
- WP-CLI available (Docker) or WP Dashboard fallback (shared hosting)
- WP File Manager plugin for file editing >50 lines
- Fonts in `.woff2` format for maximum loading performance — use [Jakson](https://www.jakson.co) to convert and optimize font files before uploading

> 🙏🙏 Thank you Jakson — your hair is carved in rock just as much as your expertise. Great videos.

### Contributing

PRs welcome. Useful contributions:
- Additional block reference entries
- Shared hosting workflow improvements
- Patterns for common section types (testimonials, pricing tables, etc.)
- Translations of `SKILL.md` for other languages

### License

MIT

---

## Documentazione in italiano

### Cos'è questa skill

Una skill per Claude che fornisce una guida operativa completa per costruire e manutenere siti WordPress con **Full Site Editing (FSE)** e blocchi Gutenberg nativi.

Deriva direttamente da `SKILL.md` v4.0, sviluppata da **Marco Antonio Cavalli** come strumento di lavoro per progetti WordPress FSE reali, con l'integrazione dei principi di Jamie Marsland (video series, giugno 2026).

### Cosa fa

Quando attivata, la skill fornisce a Claude un flusso di lavoro rigoroso per lo sviluppo FSE:

- **Principio Marsland** — template = solo struttura, il contenuto vive nelle pagine. La skill rileva e corregge l'anti-pattern "Morange" (contenuto editoriale mescolato nei template).
- **Bootstrap check** — rileva automaticamente l'ambiente (Docker / hosting condiviso), il child theme attivo e se il progetto è nuovo o in ripresa.
- **Hybrid Recon** — raccoglie i dati del sito via WP-CLI prima di scrivere qualsiasi file.
- **Sequenza operativa in 15 step** — dalla struttura delle cartelle alla verifica responsive nel browser.
- **Regole assolute** — guardrail contro `wp:html` per layout, CSS inline, unità `px` fisse, ricorsione nei template e altri anti-pattern FSE.
- **Tabella blocchi di riferimento** — lookup rapido: tipo di sezione → blocchi Gutenberg nativi corretti.
- **Le quattro larghezze** — normale, wide, full, custom (basate sulla serie video di Marsland).
- **Checklist responsive** — verifica obbligatoria a 375px · 768px · 1024px · 1440px.
- **Changelog di sessione** — ogni file scritto o modificato viene registrato automaticamente nel file di progetto, così riprendere il lavoro parte sempre da un quadro chiaro di cosa è cambiato.

### A chi serve

Sviluppatori e site builder WordPress che:
- Lavorano con FSE / block theme (non classic theme)
- Usano un workflow con child theme, `theme.json`, `templates/*.html`, `parts/*.html`, `patterns/*.php`
- Eseguono WordPress in Docker (Coolify, locale) o su hosting condiviso
- Vogliono che Claude segua una metodologia FSE coerente e principiata, senza improvvisare

### Trigger di attivazione

La skill si attiva automaticamente quando si menziona: `WordPress FSE`, `FSE`, `Gutenberg`, `child theme`, `block theme`, `theme.json`, `full site editing`, `wp-fse`, `template WordPress`, `header sticky`, `masthead`, `pattern WordPress`, `archive CPT`, `modifica homepage`, `contenuto pagina`, `larghezza contenuto`, `content width`.

### Come installarla

1. Scarica `SKILL.md` da questa cartella
2. In Claude Cowork, vai su **Impostazioni → Funzionalità → Skill**
3. Clicca **Installa skill** e seleziona il file

Oppure, se hai il pacchetto `.skill` (zip di questa cartella con estensione `.skill`), clicca il pulsante **Salva skill** direttamente nell'interfaccia chat di Cowork.

### Concetti chiave

#### Il Principio Marsland
Introdotto da [Jamie Marsland](https://www.youtube.com/@pootlepress) nella sua serie video su FSE:

> Template = succo d'arancia. Contenuto = latte. Mescolati = Morange. Non fare mai Morange.

I template contengono la struttura (header, area contenuto, footer). Le pagine contengono il contenuto editoriale. La skill applica questa separazione a ogni step.

#### Il Test Morange
All'inizio di ogni sessione su un sito esistente, la skill esegue un controllo rapido: clicca "Modifica pagina" nella admin bar di WP. Se arrivi all'editor del template invece che all'editor della pagina, hai il problema Morange e devi risolverlo prima di procedere.

> 🙏🙏 Grazie Jamie — i tuoi video mi hanno spinto ad avvicinarmi a WordPress Gutenberg e FSE in un momento di crisi.

#### I file di progetto e il changelog di sessione
La skill mantiene un file `.md` per progetto che traccia lo slug del theme, i valori del design system, i CPT, la mappatura pagine/template, lo stato di avanzamento e un **changelog di sessione** (§12.6). Ogni file scritto o modificato viene registrato con data, numero di sessione, percorso del file e una breve descrizione. Quando si riprende il lavoro, la skill legge le ultime 5 voci del changelog così il contesto è immediatamente chiaro — senza dover ricordare cosa è stato fatto nella sessione precedente.

Il nome del sito viene rilevato automaticamente via WP-CLI (opzione `siteurl`) negli ambienti Docker, o dall'URL base delle REST API quando si lavora con application password. L'inserimento manuale è solo un fallback.

### Stack di riferimento

- WordPress 6.x / 7.x
- Child theme (FSE block theme)
- Docker via Coolify, o hosting condiviso
- WP-CLI disponibile (Docker) o fallback su WP Dashboard (hosting condiviso)
- Plugin WP File Manager per modifica file >50 righe
- Font in formato `.woff2` per massima velocità di caricamento e alleggerimento del sito — usare [Jakson](https://www.jakson.co) per convertire e ottimizzare i file font prima dell'upload

> 🙏 Grazie Jakson — i tuoi capelli sono scolpiti nella roccia tanto quanto la tua competenza.

### Contribuire

Le PR sono benvenute. Contributi utili:
- Nuove voci nella tabella blocchi di riferimento
- Miglioramenti al flusso per hosting condiviso
- Pattern per tipi di sezione comuni (testimonianze, pricing, ecc.)
- Traduzioni di `SKILL.md` in altre lingue

### Licenza

MIT
