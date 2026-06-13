---
name: wp-fse
description: >
  Guida operativa completa per WordPress Full Site Editing (FSE) con Gutenberg nativo.
  Attivare SEMPRE quando l'utente lavora su: child theme FSE, theme.json, templates/*.html,
  parts/*.html, patterns/*.php, masthead.js, CPT con wp:query, header/footer sticky,
  navigazione mobile, responsive WordPress, WP-CLI su WordPress.
  Trigger: "WordPress FSE", "FSE", "Gutenberg", "child theme", "block theme",
  "theme.json", "full site editing", "tema blocchi", "wp-fse", "template WordPress",
  "header sticky", "masthead", "pattern WordPress", "archive CPT",
  "modifica homepage", "contenuto pagina", "larghezza contenuto", "content width".
---

# WordPress FSE — Skill Operativa v3.9
# Integra i principi Marsland (video 1–3, giugno 2026)

---

## [H] PRINCIPIO MARSLAND — Separazione Template / Contenuto

> ⚠️ REGOLA ZERO. Leggere prima di qualsiasi altra azione, ogni sessione.
> Fonte: Jamie Marsland — "Templates and Content" + "Three Core Principles"

### La metafora del Morange

Template = succo d'arancia. Contenuto = latte.
Separati: entrambi ottimi. Mescolati: Morange — disgustoso e inutilizzabile.

**Non fare mai Morange.**

### Cosa va dove

| Cosa | Dove va | Come si edita |
|------|---------|---------------|
| Struttura (header, footer, area contenuto) | Template (`templates/*.html`) | Site Editor → Template |
| Sezioni strutturali fisse riusabili | Pattern PHP (`patterns/*.php`) | SSH / File Manager |
| Contenuto editoriale (testi, card, sezioni) | **Pagina** (database WP) | Admin → Pagine → Modifica |
| Stili globali (font, colori, spaziature) | `theme.json` + Global Styles | Site Editor → Stili |
| Header / Footer riusabili | Template Parts (`parts/*.html`) | Site Editor → Parti template |

### Struttura corretta di un template pagina

```html
<!-- wp:template-part {"slug":"header","tagName":"header"} /-->

<!-- wp:group {"tagName":"main","layout":{"type":"constrained"}} -->
<main class="wp-block-group">
  <!-- wp:post-content {"layout":{"type":"constrained"}} /-->
</main>
<!-- /wp:group -->

<!-- wp:template-part {"slug":"footer","tagName":"footer"} /-->
```

`wp:post-content` è l'unico punto in cui entra il contenuto della pagina.
Non aggiungere testi, card, sezioni editoriali intorno ad esso.

### Quando usare wp:pattern nei template

I `wp:pattern` nei template sono ammessi **solo** per elementi strutturali
che non variano da pagina a pagina e non sono contenuto editoriale.
Esempi ammessi: commiato fisso, banner cookie, breadcrumb strutturale.
Esempi NON ammessi: hero con testo specifico, sezioni con contenuto.

### Test Morange (eseguire a inizio sessione se il sito è esistente)

1. Aprire il sito nel browser da utente loggato
2. Cliccare "Modifica pagina" nella admin bar
3. ✅ Se arrivi all'editor della **pagina** → struttura corretta
4. ❌ Se arrivi all'editor del **template** → il sito ha il problema Morange
   → Correggere prima di qualsiasi altra modifica:
   a. Spostare il contenuto dalla template nella pagina corrispondente
   b. Ridurre il template a struttura pura (header + wp:post-content + footer)

### Flusso corretto per nuova homepage (metodo Marsland in 3 passi)

1. **Site Editor → Template → "Prima pagina"**: creare/verificare struttura pura
   (header + wp:post-content + footer). Nessun pattern editoriale.
2. **Admin → Pagine → Aggiungi nuova**: creare la pagina "Home" e disegnare
   il contenuto con blocchi/pattern. Pubblicare.
3. **Impostazioni → Lettura**: impostare "Una pagina statica" → Homepage: Home.

---

## [A] BOOTSTRAP CHECK

> Eseguire per primo, prima di qualsiasi altra azione. Non saltare.

```bash
PROJECTS_DIR=/home/marco/.claude/skills/wp-fse/projects
```

**Passi obbligatori (in sequenza):**

**A0 — Rileva ambiente:**
```bash
if command -v docker &>/dev/null && docker ps &>/dev/null 2>&1; then
  ENV_TYPE="docker"
else
  ENV_TYPE="shared"
fi
echo "Ambiente: $ENV_TYPE"
```

- `ENV_TYPE=docker` → eseguire A1, A2, A3 con comandi docker exec
- `ENV_TYPE=shared` → saltare A2 interamente, chiedere THEME_SLUG all'utente (→ A2b), poi A3

**A0b — Dichiara principio fondante:**

Prima di procedere, dichiarare esplicitamente all'utente:

> "Principio Marsland attivo: **template = struttura**, **contenuto = nelle pagine**.
> Non mescolerò contenuto editoriale nei template. Procedo."

Se il sito è esistente, eseguire il **Test Morange** (vedi §H) prima di qualsiasi modifica.

**A1 — Verifica directory progetti:**
```bash
ls "$PROJECTS_DIR" 2>/dev/null || mkdir -p "$PROJECTS_DIR"
```

**A2 — Rileva container WordPress e child theme attivo** *(solo se ENV_TYPE=docker)*:
```bash
CONTAINERS=$(docker ps --filter "name=wordpress" --format "{{.Names}}")
CONTAINER_COUNT=$(echo "$CONTAINERS" | grep -c .)
```

- Se `CONTAINER_COUNT` = 1: usa il container trovato automaticamente
- Se `CONTAINER_COUNT` > 1: mostrare la lista e chiedere: "Ho trovato più container WordPress. Su quale vuoi lavorare?" → assegnare la scelta a `CONTAINER`
- Se `CONTAINER_COUNT` = 0: errore — "Nessun container WordPress attivo — avviare Docker prima di procedere"

```bash
CONTAINER=<valore scelto o unico>
THEME_SLUG=$(docker exec "$CONTAINER" wp --allow-root theme list \
  --status=active --field=name 2>/dev/null | head -1)
[ -z "$THEME_SLUG" ] && echo "ERROR: impossibile rilevare il child theme attivo — verificare WP-CLI o indicare lo slug manualmente" && exit 1
echo "Container: $CONTAINER | Theme: $THEME_SLUG"
```

**A2b — Percorso hosting condiviso** *(solo se ENV_TYPE=shared)*:
Chiedere esplicitamente: "Qual è lo slug (nome cartella) del tuo child theme attivo?"
Assegnare la risposta a `THEME_SLUG`. Docker e WP-CLI non sono disponibili — nella Fase 1
del §HYBRID RECON usare le istruzioni manuali Dashboard WP invece dei comandi bash.

**A3 — Decisione INIT / RESUME:**

Esegui:
```bash
ls "$PROJECTS_DIR/$THEME_SLUG.md" 2>/dev/null && echo "RESUME" || echo "INIT"
```

- Output `RESUME` → leggi `$PROJECTS_DIR/$THEME_SLUG.md`, vai a **§RESUME**
- Output `INIT` → vai a **§HYBRID RECON**

---

## [B] §0 HYBRID RECON

> Non proporre piani e non scrivere file prima del completamento di questa sezione.
> Regola assoluta: A, B e C del manuale devono avere risposta esplicita prima di procedere.

### Fase 1 — Auto-recon (eseguire in blocco unico)

```bash
CONTAINER=$(docker ps --filter "name=wordpress" --format "{{.Names}}" | head -1)

# A — Versione WP
docker exec "$CONTAINER" wp --allow-root core version

# A1 — WP-CLI disponibile?
docker exec "$CONTAINER" wp --allow-root --info

# A2 — Template nel DB
docker exec "$CONTAINER" wp --allow-root post list \
  --post_type=wp_template --fields=post_name,post_status --format=table

# B — Tema attivo e parent
docker exec "$CONTAINER" wp --allow-root theme list

# URL sito
docker exec "$CONTAINER" wp --allow-root option get siteurl

# Plugin attivi
docker exec "$CONTAINER" wp --allow-root plugin list --status=active --field=name

# Verifica homepage (Marsland check)
docker exec "$CONTAINER" wp --allow-root option get show_on_front
docker exec "$CONTAINER" wp --allow-root option get page_on_front
```

Hosting condiviso (no SSH/WP-CLI): usare Dashboard WP → Aggiornamenti per A, Aspetto → Temi per B,
Query Monitor per A2. Sostituire tutti i comandi docker exec wp con istruzioni manuali.

### Fase 2 — Domande gap

Chiedere **solo** i dati non risolti dalla Fase 1:

**C — Stato sito esistente** (sempre chiedere):
> "Il sito ha già contenuti/template? Scegli:
> 1. Mantenere tutto (solo aggiungere/modificare)
> 2. Azzeramento parziale — specificare cosa
> 3. Azzeramento completo"

⚠️ Se scelta 2 o 3: **DOPPIA CONFERMA OBBLIGATORIA** con elenco esplicito di cosa viene eliminato.

### Fase 3 — Questionario §12

Pre-compilare con dati auto-recon. Chiedere solo i campi con No in colonna Auto:

| # | Campo | Auto? | Se No: chiedere |
|---|---|---|---|
| 1 | URL produzione | Sì | — |
| 2 | Tema parent | Sì | — |
| 3 | Child theme slug | Sì | — |
| 4 | Versione WP | Sì | — |
| 5 | Font corpo | No | Nome, file .woff2, pesi disponibili? |
| 6 | Font display | No | Nome, file .woff2? |
| 7 | Colori principali | No | Sfondo, testo, accento1, accento2 (#hex)? |
| 8 | Layout | No | contentSize e wideSize? |
| 9 | CPT previsti | Parziale | Slug, label, tassonomie? |
| 10 | Pagine e template | No | Elenco pagine con template slug assegnato? |
| 11 | Pattern ricorrenti | No | Hero, CTA, card, FAQ, altro? |
| 12 | Plugin obbligatori | Sì | Confermare lista plugin attivi |

### Fase 4 — Genera project file

Dopo aver raccolto tutti i dati, scrivi `$PROJECTS_DIR/$THEME_SLUG.md` con questo template:

```markdown
# WP-FSE Project: THEME_SLUG
<!-- generato da skill wp-fse · DATA_GENERAZIONE -->

## §12.1 Identificazione
- child_theme_slug: THEME_SLUG
- child_theme_path: /var/www/html/wp-content/themes/THEME_SLUG
- container: CONTAINER_NAME
- wp_version: WP_VERSION
- site_url: SITE_URL
- server: Coolify / Docker · VPS
- parent_theme: PARENT_THEME

## §12.2 Design system
- font_body_slug: SLUG
- font_body_file: FILE.woff2
- font_body_weights: PESI
- font_display_slug: SLUG
- font_display_file: FILE.woff2
- color_bg: #HEX
- color_text: #HEX
- accent1: #HEX
- accent2: #HEX
- contentSize: VALORE
- wideSize: VALORE
- contrast_verified: SI_NO
- header_height_desktop: __px
- header_height_mobile: __px

## §12.3 CPT e tassonomie
<!-- slug_cpt | label | slug_tassonomia | label_tassonomia | plugin -->

## §12.4 Pagine e template
<!-- pagina | template_slug | note_marsland (contenuto in pagina? sì/no) -->

## §12.5 Stato avanzamento
- ultimo_step_completato: 0
- step_in_corso: 1
- marsland_verified: false  ← impostare true dopo Test Morange superato
- file_completati: []
- note_sessione: ""
- ultimo_aggiornamento: DATA_GENERAZIONE
```

### §RESUME — Sessione successiva

1. Leggi `$PROJECTS_DIR/$THEME_SLUG.md` → mostra riepilogo §12.5
2. Verificare `marsland_verified`: se `false`, eseguire Test Morange prima di procedere
3. Chiedi: "Continuo da step `step_in_corso` / hai un task specifico / verifica responsive?"
4. Proponi piano di lavoro dettagliato → approvazione utente → esecuzione

---

## [C] REGOLE ASSOLUTE

> Guardrail permanente. Applicare a ogni step, senza eccezioni.

### MAI fare ✗

**Principio Marsland:**
- Inserire contenuto editoriale (testi, card, sezioni con testo specifico) direttamente in un template
- Usare il Site Editor per modificare contenuto che appartiene a una pagina
- Aggiungere `wp:pattern` con contenuto editoriale nei template (solo elementi strutturali fissi)
- Creare una nuova homepage mettendo i blocchi nel template invece che nella pagina

**Tecnico FSE:**
- `wp:html` per layout, griglie, card, sezioni — solo shortcode plugin o iframe documentati
- CSS `style=""` inline negli attributi blocco
- Template nel tema parent — sempre nel child
- Duplicare file parent nel child senza modificarli
- Salvare template via REST API `/wp/v2/templates` — usare filesystem
- Registrare pattern come contenuto di pagina — usare `patterns/*.php`
- Sovrascrivere file senza averlo letto prima
- Valori `px` fissi per font/spaziature — usare scale `clamp()` in theme.json
- Larghezze fisse in `px` nei pattern — usare `%`, `vw`, unità fluide
- `min-width` hardcoded nei blocchi (rompe il layout mobile)
- `font-size` in `px` fissi — sempre `clamp()` da theme.json
- `wp:image` senza `sizeSlug` esplicito — WP non genera srcset
- Omettere commento intestazione nei file `.html`
- `wp:template-part` dentro `parts/header.html` o `parts/footer.html` → ricorsione infinita
- Voci di menu hardcoded in `wp:navigation` → menu non editabile
- `!important` nel CSS FSE su proprietà gestibili dall'editor visuale

### SEMPRE fare ✓

- Verificare il Principio Marsland prima di qualsiasi modifica a template esistenti
- `tagName` semantico: `header`, `footer`, `section`, `article`, `main`
- `"anchor":"site-header-main"` nel `wp:group` dell'header per masthead.js
- `wp:template-part` per header/footer nei template (NON dentro `parts/`)
- Sezioni modulari come pattern `.php` nel child theme
- `$schema` esplicito in theme.json
- `inheritQuery:false` in `wp:query` nei template personalizzati
- `postType` esplicito in `wp:query` per CPT
- masthead.js registrato nel footer (`last` = `true`)
- `overlayMenu:"mobile"` in `wp:navigation` per hamburger mobile
- `sizeSlug:"large"` in `wp:image` per srcset automatico
- `fetchpriority:"high"` + `loading:"eager"` solo sull'immagine hero above-the-fold
- `loading:"lazy"` su tutte le altre immagini
- `fontDisplay:swap` per font locali in theme.json
- `lock {"move":false,"remove":false}` come default nei pattern
- `esc_html__()` per stringhe visibili nei pattern PHP
- Un solo H1 per template/pagina, gerarchia H1→H2→H3
- `rel="noopener noreferrer"` su link con `target="_blank"`
- `scroll-margin-top` su elementi con ID quando header è fixed

---

## [D] SEQUENZA OPERATIVA

> Piano di lavoro obbligatorio prima di ogni sessione.
> Per ogni file indicare: percorso completo, stato attuale, azione, contenuto/diff.
> Piano vago = non approvabile.

### 15 Step

| Step | Operazione | File | Metodo scrittura |
|---|---|---|---|
| 1 | Struttura cartelle | Filesystem | `mkdir -p parts templates patterns assets/fonts assets/js` |
| 2 | style.css | `child/style.css` | SSH heredoc o WP File Manager |
| 3 | theme.json | `child/theme.json` | WP File Manager (>50 righe) |
| 4 | functions.php | `child/functions.php` | WP File Manager |
| 5 | Font .woff2 | `child/assets/fonts/` | Upload via WP File Manager |
| 6 | masthead.js | `child/assets/js/` | Misurare `--header-height` desktop+mobile dal browser |
| 7 | header.html | `child/parts/` | WP File Manager |
| 8 | footer.html | `child/parts/` | WP File Manager |
| 8b | Template CPT | `child/templates/` | `single-{cpt}.html` + `archive-{cpt}.html` |
| 9 | Pattern modulari | `child/patterns/*.php` | WP File Manager |
| 10 | Template pagine | `child/templates/*.html` | Struttura pura: header + wp:post-content + footer |
| 11 | Pagine WordPress | Admin WP → Pagine | Creare pagine, inserire contenuto editoriale qui (Marsland) |
| 11b | Homepage | Impostazioni → Lettura | Impostare pagina statica come front page |
| 12 | Stili blocchi | Editor FSE → Stili → Blocchi | Dashboard WP |
| 13 | CSS aggiuntivo | Editor FSE → Stili → ⋮ | Solo override non gestibili dall'editor |
| 14 | Verifica WP-CLI | Terminale | Checklist post-installazione |
| 15 | Verifica browser | Browser + DevTools | Checklist responsive §2.1 — obbligatoria |

### Protocollo modifica file esistente (§6.2)

1. Leggi il file integralmente prima di qualsiasi modifica
2. Backup: `cp file.html file.html.bak`
3. Identifica solo le righe da modificare
4. Applica modifica localizzata — preserva commenti e lock
5. Verifica:
   ```bash
   docker exec "$CONTAINER" wp --allow-root post list --post_type=wp_template --format=table
   ```
6. Errore → ripristina: `cp file.html.bak file.html`

### Cache template (dopo ogni .html scritto)

```bash
docker exec "$CONTAINER" wp --allow-root cache flush
```
Alternativa: Admin → Editor del sito → Template → Reimposta → Salva.

### Gerarchia metodi scrittura file (§5)

| Metodo | Quando usare | Vincoli critici |
|---|---|---|
| WP File Manager | HTML/PHP >50 righe (preferito) | Errore encoding → CONVERT. HTTP 500 → elimina e ricrea |
| SSH + heredoc | 20–50 righe, da PowerShell | Tronca >200 char/riga. MAI dal terminale Coolify |
| Script PHP | Emergenza (heredoc fallisce) | `file_put_contents()` in `/tmp/`. Due passaggi |
| Terminale Coolify | Solo lettura (`head`, `cat`, `ls`, `grep`) | MAI per scrivere file |

---

## [E] BLOCCHI REFERENCE

> Lookup rapido: tipo sezione → blocchi nativi da usare.

| Sezione | Blocchi nativi | Note |
|---|---|---|
| Hero | `wp:cover` o `wp:group` | `fullHeight`, `minHeight` clamp(). NON `wp:html` |
| Card servizi | `wp:columns` + `wp:column` + `wp:group` (`tagName:article`) | Bordi/ombre in theme.json |
| Griglia articoli | `wp:query` + `wp:post-template` | `postType`, `inheritQuery:false`, `wp:query-pagination` |
| Header sticky | `wp:group` (`anchor:site-header-main`) + masthead.js | CSS fixed + JS |
| Header duale | `wp:group` ×2 + masthead-dual.js | Main + mini |
| FAQ accordion | `wp:details` (WP 6.3+) | NON JS/HTML esterni |
| Form contatti | `wp:group` + `wp:shortcode` | Unica eccezione plugin form |
| Menu ≤5 voci | `wp:navigation` senza ref + `overlayMenu:mobile` | Fallback automatico WP |
| Menu >5 voci | `wp:navigation` con ref + WP-CLI | `wp menu create` + ID reale |
| Griglia Bento | `wp:columns` larghezze % asimmetriche | Es. 60/40, 30/70 |

### Le quattro larghezze (Marsland — Video 3)

> Fonte: "Content Widths in WordPress Block Themes" — Jamie Marsland

| Larghezza | Come si ottiene | Impostazione globale |
|---|---|---|
| **Normale** | default di ogni blocco aggiunto | Site Editor → Stili → Layout → Content width (`contentSize` in theme.json) |
| **Wide** | `align="wide"` sul blocco | Site Editor → Stili → Layout → Wide width (`wideSize` in theme.json) |
| **Full** | `align="full"` sul blocco | nessuna — è sempre 100% viewport, non serve impostazione |
| **Custom** | Group block + disattiva "inner blocks use content width" | nessuna — si imposta per singolo blocco |

**Esempio custom width:** aggiungere un `wp:group`, impostarlo full-width, poi impostare
`contentSize` interno a 250px. In theme.json usare sempre unità fluide, mai `px` fissi.

**Quando usare quale:**
- Testo, paragrafi, heading → **normale**
- Video, CTA prominenti, gallery → **wide**
- Sezioni colorate a tutta larghezza → **full**
- Layout speciali one-off → **custom**

### Architettura file (§3)

| File | Responsabilità | NON deve fare |
|---|---|---|
| theme.json | Palette, font, spacing fluido, $schema | Contenuti, layout specifici |
| style.css | CSS strutturale: sticky header, variabili, fix layout | Stili visivi, colori, override blocchi |
| CSS aggiuntivo FSE | Override visivi non impostabili dall'editor | Stili strutturali, `!important` |
| functions.php | Enqueue font/JS, registrazione pattern, block styles | Logica business, query DB |
| parts/*.html | Header e footer — struttura interna | `wp:template-part` interno |
| templates/*.html | Struttura: `wp:template-part` + `wp:post-content` | Testi editoriali, contenuto pagina ← MARSLAND |
| patterns/*.php | Sezioni strutturali modulari riusabili | Contenuto editoriale specifico di pagina |
| Pagine WP (DB) | Contenuto editoriale di ogni pagina | Struttura globale, header/footer ← MARSLAND |

---

## [F] RESPONSIVE RULES

> Responsive è obbligatorio. Verificare a 375px · 768px · 1024px · 1440px.

### Tipografia fluida (theme.json — MAI px fissi)

```json
{"slug":"sm","size":"clamp(0.875rem,0.8rem+0.4vw,1rem)","fluid":true},
{"slug":"md","size":"clamp(1rem,0.9rem+0.5vw,1.25rem)","fluid":true},
{"slug":"lg","size":"clamp(1.5rem,1.2rem+1.5vw,2.25rem)","fluid":true},
{"slug":"xl","size":"clamp(2rem,1.5rem+2.5vw,3.5rem)","fluid":true}
```

### Immagini responsive

```html
<!-- Hero: eager + high priority -->
<!-- wp:image {"sizeSlug":"large","width":1200,"height":800,"loading":"eager","fetchpriority":"high"} -->

<!-- Tutte le altre immagini -->
<!-- wp:image {"sizeSlug":"large","width":800,"height":600,"loading":"lazy"} -->
```

### Navigazione mobile (sempre overlayMenu:mobile)

```html
<!-- Menu ≤5 voci -->
<!-- wp:navigation {"overlayMenu":"mobile"} /-->

<!-- Menu >5 voci (ref = ID menu WP) -->
<!-- wp:navigation {"ref":ID,"overlayMenu":"mobile"} /-->
```

### Hero responsive (MAI px fissi)

```html
<!-- wp:cover {"minHeight":60,"minHeightUnit":"vw",...} -->
```

### --header-height: misurare dal browser

Console DevTools:
```javascript
document.querySelector('.site-header').getBoundingClientRect().height
```

CSS (compilare con valori misurati):
```css
:root { --header-height: DESKTOPpx; }
@media (max-width: 768px) { :root { --header-height: MOBILEpx; } }
```

### Admin bar vs header fixed (bug noto — utenti loggati)

```css
.admin-bar .site-header-main { top: 32px; }
@media screen and (max-width: 782px) { .admin-bar .site-header-main { top: 46px; } }
```

### Diagnostica overflow orizzontale (§9.6)

Incollare in Console DevTools:
```javascript
document.querySelectorAll('*').forEach(el => {
  if (el.offsetWidth > document.documentElement.offsetWidth) {
    console.log(el, el.offsetWidth);
  }
});
```
Nota: `overflow-x:hidden` è un sintomo, non una soluzione. Identificare l'elemento causa.

### Checklist responsive — Step 15 (obbligatoria)

Testare con DevTools ai breakpoint: **375px · 768px · 1024px · 1440px**

- [ ] Header sticky visibile e funzionante a tutti i breakpoint
- [ ] Menu hamburger si apre e chiude su mobile
- [ ] Hero: testo leggibile, immagine non tagliata, CTA visibile
- [ ] Colonne si impilano correttamente su mobile
- [ ] Nessun overflow orizzontale (usare script §F)
- [ ] Font body ≥16px su mobile
- [ ] Immagini non distorte, srcset attivo (tab Network DevTools)
- [ ] Form: campi non troncati, bottone visibile
- [ ] Pulsanti/CTA: area tocco ≥44×44px
- [ ] scroll-margin-top corretto su tutti gli anchor con header fixed

---

## [G] ONESTÀ E LIMITI

Codice consegnato senza riserve = codice di cui la skill è ragionevolmente certa.
Se un problema supera le conoscenze disponibili:

1. Dichiarare il limite esplicitamente: cosa è noto / cosa è incerto / perché
2. Proporre le opzioni: procedere con rischi esplicitati / seconda opinione / web search
3. Attendere la scelta dell'utente prima di procedere

**Template seconda opinione (Gemini / DeepSeek):**

```
Contesto: WordPress FSE, child theme Gutenberg nativo, WP 7.0.
Problema: [descrizione precisa]
Tentativo effettuato: [codice o approccio già provato]
Errore o comportamento inatteso: [output esatto]
Domanda: [cosa si cerca]
```
