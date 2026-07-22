# Recursos terminològics per a la traducció de programari al català

## Principi de consistència

**Una sola traducció per a cada terme**. La inconsistència (p. ex. usar "D'acord", "Accepta" i "Bé" indistintament per "OK") trenca la coherència de la interfície. Decideix un terme i usa'l sempre.

Quan s'estableix un terme per a un projecte, **documenta'l** i consulta les traduccions anteriors del mateix projecte abans de decidir.

## Fonts de consulta per ordre de prioritat

### 1. Glossari de Softcatalà (primera consulta)
- **HTML navegable**: https://static.softcatala.org/terminology/sc-glossary.html
- **CSV descarregable**: https://static.softcatala.org/terminology/sc-glossary.csv
- Extret automàticament de les traduccions de Softcatalà
- Cobreix terminologia de programari lliure i codi obert

### 2. Glossari complet de programari lliure
- **HTML navegable**: https://static.softcatala.org/terminology/tots-glossary.html
- **CSV descarregable**: https://static.softcatala.org/terminology/tots-glossary.csv
- Cobreix tots els projectes amb memòries de traducció (no només Softcatalà)
- Útil per a termes específics d'ecosistemes com GNOME, KDE, LibreOffice

### 3. TERMCAT — Centre de Terminologia
- Web: https://www.termcat.cat/
- **Cercaterm**: motor de cerca de terminologia catalana oficial (`/ca/cercaterm`)
- **Neoloteca**: recull de neologismes tecnològics aprovats (`/ca/neoloteca`)
- **Portals temàtics**: TIC (`http://tic.termcat.cat/ca`), economia, esports, dret, etc.
- Font autoritzada per a termes nous o dubtosos
- Si el terme hi és, usar la forma de TERMCAT tret que Softcatalà hagi establert una alternativa

### 4. Memòries de traducció de Softcatalà
- Web: https://www.softcatala.org/recursos/memories/
- 304 projectes, ~14,5 milions de paraules traduïdes (juny 2026)
- Formats disponibles: **PO** (Portable Object) i **TMX** (Translation Memory eXchange)
- Útil per veure com s'ha traduït un terme en context real
- Integrable a OmegaT, memoQ, Weblate, Poedit

### 5. Glossaris de Microsoft
- Disponibles gratuïtament a http://www.microsoft.com/Language/
- Útil per a termes que Microsoft ha establert en català
- Coherència amb l'ecosistema Windows

### 6. Glossaris d'Apple
- Accessibles a desenvolupadors registrats d'Apple
- Per a apps iOS/macOS, seguir la terminologia d'Apple quan escau

## Com accedir a les memòries de traducció

### Descàrrega individual per projecte
Cada projecte a https://www.softcatala.org/recursos/memories/ té:
- Enllaç de descàrrega TMX (`.tmx.zip`)
- Enllaç de descàrrega PO (`.po.zip` o carpeta `.tar.gz`)

### Col·leccions agregades
- **Totes les memòries** (`tots-tm`): ~10,7 milions de paraules (10.655.094, juny 2026)
- **Només projectes Softcatalà** (`softcatala-tm`): ~2,9 milions de paraules (2.877.770, qualitat més controlada)

### Ús amb eines de traducció
- **OmegaT**: afegir fitxers TMX a la carpeta `tm/` del projecte
- **Weblate**: importar TMX des de l'administració del projecte
- **Poedit**: els fitxers PO es poden usar directament

### Ús programàtic (per a agents IA)
Per usar les memòries en un agent de traducció:
1. Descarregar el TMX zip des de la URL del projecte
2. Descomprimir: `unzip projecte.tmx.zip`
3. Parsejar el XML (TMX és XML estàndard):
   ```xml
   <tu>
     <tuv xml:lang="en"><seg>File not found</seg></tuv>
     <tuv xml:lang="ca"><seg>No s'ha trobat el fitxer</seg></tuv>
   </tu>
   ```
4. Indexar per segment source per a cerca eficient

## Termes tecnològics recomanats

### Termes d'UI generals
| Anglès | Català recomanat | Notes |
|--------|-----------------|-------|
| OK | D'acord / Accepta | "D'acord" en diàlegs; "Accepta" en preferències |
| Cancel | Cancel·la | Amb punt volat |
| Apply | Aplica | |
| Close | Tanca | |
| Open | Obre | |
| Save | Desa | NO "guarda" ni "salva" |
| Save As | Desa com a | |
| New | Nou/Nova | Acord en gènere |
| Edit | Edita | (menú); "edita" (botó) |
| Delete | Elimina / Suprimeix | "Elimina" per a elements; "Suprimeix" per a contingut |
| Print | Imprimeix | |
| Find | Cerca | NO "busca" |
| Replace | Substitueix | |
| Help | Ajuda | |
| Settings / Preferences | Configuració / Preferències | "Configuració" per a sistema; "Preferències" per a app |
| File | Fitxer | NO "arxiu" (fals amic del castellà) |
| Folder / Directory | Carpeta / Directori | "Carpeta" en UI; "directori" en terminal |
| Copy | Copia | |
| Cut | Retalla | |
| Paste | Enganxa | |
| Undo | Desfés | |
| Redo | Refés | |
| Select All | Selecciona-ho tot | |
| Refresh | Actualitza | |
| Update | Actualitza | |
| Install | Instal·la | Amb punt volat |
| Uninstall | Desinstal·la | |
| Download | Descarrega | |
| Upload | Puja | (al servidor) |
| Log in / Sign in | Inicia la sessió | |
| Log out / Sign out | Tanca la sessió | |
| Register | Registra't | |
| Password | Contrasenya | |
| Username | Nom d'usuari | |
| Search | Cerca | |
| Filter | Filtra | |
| Sort | Ordena | |
| View | Vista / Visualitza | "Vista" com a nom; "visualitza" com a verb |
| Tools | Eines | |
| Window | Finestra | |
| Tab | Pestanya | |
| Menu | Menú | |
| Toolbar | Barra d'eines | |
| Status bar | Barra d'estat | |
| Sidebar | Barra lateral | |
| Panel | Tauler | |

### Termes d'ecosistema (verificats al glossari Softcatalà)
| Anglès | Català | Nota |
|--------|--------|------|
| Cookie / Cookies | Galeta / Galetes | 100% ús al glossari |
| Widget / Widgets | Giny / Ginys | 100% ús al glossari |
| Legacy | Antic / Antiga | També "Giny tradicional" per a `legacy widget` |
| Built-in | Integrat | Ex.: «funció integrada», «eina integrada» |
| Storage | Emmagatzematge | NO «magatzem» (magatzem = lloc físic) |
| Store (App Store) | Botiga | Apple Store → Botiga d'Apple |
| Download (verb) | Baixa / Baixeu | Preferit al glossari (100% ús); «descarregar» també és vàlid (DIEC), però «baixar» és la forma recomanada |
| Build (verb) | Construeix / Construïu | Però «Built by X» → «Creat per X» (atribució d'autoria en context web) |
| Click (nom) | Clic | «Fer clic» com a verb; en macOS «Click OK» → «Premeu D'acord» |
| Token | Testimoni | Veure nota «El cas `token`» a sota; decisió segons context i públic objectiu |
| Keychain | Clauer | Pràctica universal en localització Apple/macOS (no apareix al glossari Softcatalà); «macOS Keychain» → «Clauer del macOS» |

### El cas `token` (decisió segons context i públic)

TERMCAT té **fitxes normalitzades a la Neoloteca** (TERMCAT, 2025) per a `token` segons context. La coherència dins del mateix producte és prioritària:

| Context | Traducció TERMCAT | Aplicació pràctica |
|---------|------------------|---------------------|
| IA / aprenentatge automàtic (genèric) | **Segment** (n m) | Per a gran públic: usar «segment» |
| NLP / LLM (`text token`, GPT token) | **Segment textual** (n m) | TERMCAT 2025; terme normalitzat |
| Xarxes telecomunicacions (token ring) | **Testimoni** (n m) | Rarament apareix en UI moderna |
| Criptomonedes | **Criptovalor** (n m) | Ex.: «initial coin offering» |
| Autenticació (OAuth, API, session) | **Testimoni** (glossari Softcatalà) | Per a gran públic: usar «testimoni» |
| Aplicacions per a desenvolupadors / eines tècniques | **Token** (manlleu) | L'usuari tècnic espera trobar «token»; usar «testimoni» o «segment» genera confusió |

Fonts verificades (TERMCAT Cercaterm, juny 2026):
- Definició de *segment*: «Unitat discreta de dades que representa una part significativa d'informació que pot ser processada i integrada amb altres segments en el context de sistemes intel·ligents basats en xarxes neuronals profundes.»
- Definició de *segment textual*: «Unitat mínima resultant de la divisió automàtica d'un text en fragments més petits i manejables que és utilitzada com a entrada o unitat de processament en diferents models d'intel·ligència artificial.»
- Nota TERMCAT: «Un segment textual pot ser un mot, un submot, un caràcter o un símbol especial, depenent del model d'intel·ligència artificial o de l'aplicació en què s'hagi d'utilitzar.»

**Principi general**: en eines dirigides a desenvolupadors, el manlleu tècnic és sovint preferit quan la comunitat tècnica ja l'ha adoptat (ex.: `token`, `endpoint`, `callback`, `payload`, `deployment`). El criteri és quin terme usa el públic objectiu quan parla de la tecnologia en qüestió. Aquest principi **no** s'aplica a programari dirigit al gran públic, on s'ha de prioritzar sempre la forma catalana normalitzada per TERMCAT.

### Missatges del sistema
| Anglès | Català recomanat |
|--------|-----------------|
| File not found | No s'ha trobat el fitxer |
| Access denied | Accés denegat |
| Operation completed | L'operació s'ha completat |
| Error occurred | S'ha produït un error |
| Loading... | S'està carregant... |
| Saving... | S'està desant... |
| Please wait... | Espereu... |
| No items found | No s'ha trobat cap element |
| Are you sure? | Esteu segur? |
| This action cannot be undone | Aquesta acció no es pot desfer |

### Termes de programació / dev
| Anglès | Català |
|--------|--------|
| Repository | Repositori |
| Branch | Branca |
| Commit | Comissió (tècnic) / *commit* (en context dev) |
| Pull request | Sol·licitud d'incorporació |
| Bug | Error / Defecte |
| Feature | Funcionalitat |
| Release | Versió / Llançament |
| Deploy | Desplegament / Desplegar |
| Build | Compilació / Construir |
| Debug | Depurar / Depuració |
| Log | Registre |
| Cache | Memòria cau |
| Thread | Fil d'execució |
| Process | Procés |
| Library | Biblioteca |
| Framework | Marc de treball (o *framework* si no hi ha alternativa clara) |
| API | API (no traduir) |
| Interface | Interfície |
| Database | Base de dades |
| Query | Consulta |
| Server | Servidor |
| Client | Client |
| Request | Sol·licitud / Petició |
| Response | Resposta |

## Quan no hi ha traducció establerta

1. Consultar en ordre: glossari Softcatalà → TERMCAT → Microsoft/Apple glossaries
2. Si no hi ha terme, proposar-ne un de nou a la llista de correu de terminologia de Softcatalà
3. Mentre s'espera, usar el terme anglès en cursiva: *drag and drop*
4. No inventar neologismes evidents com "zipat", "gzipat", "pinged" — usar descripcions: "comprimit en format zip"

## Llista de correu de terminologia

Per a dubtes terminològics i propostes de termes nous:
- Llista pública de Softcatalà: https://llistes.softcatala.org/mailman/listinfo/terminologia
- Aportar: el terme anglès, context d'ús, proposta de traducció amb raonament
