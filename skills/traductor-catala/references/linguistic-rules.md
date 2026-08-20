# Normes lingüístiques per a la traducció de programari al català

## Registre i to

- Registre formal però no pompós
- L'ordinador s'adreça a l'usuari en **vós** (plural de respecte): "Esteu segur?", "Introduïu el nom"
- L'usuari dóna ordres a l'ordinador en **imperatiu de vós**: "Obriu", "Deseu", "Cancel·leu" (no les formes de tu `Obre`/`Desa`/`Cancel·la` quan el producte va en vós)
- Les **descripcions** (subtítols, captions, frases que expliquen què fa una opció) van en **3a persona**: "Desa diversos comptes…", "Mostra les icones…", "Obre el CodexBar automàticament…" — aquestes formes (`Desa`, `Mostra`, `Obre`) coincideixen formalment amb l'imperatiu de tu però són present d'indicatiu, i són correctes en descripcions
- Evitar humanitzar l'ordinador: res de "Ho sento", "Gràcies per esperar", onomatopeies ("oh")
- Evitar expressions innecessàries: "simplement", "fàcilment", "només cal que"

## Verbs en elements UI

### Ordres (usuari → sistema)
Imperatiu de vós: **"Obriu"** (no "Obrir"), **"Deseu"** (no "Guardar"), **"Imprimiu"**

### Missatges (sistema → usuari)
Vós: **"Esteu segur que voleu eliminar?"**, **"El document s'ha desat"**

### Gerundis anglesos ("-ing")
- "Loading" → "S'està carregant..." o "Carregant..."
- "Opening file" → "S'està obrint el fitxer" o "Obrint el fitxer"
- "en + infinitiu" per accions condicionals: "en llegir el fitxer" (no "al llegir")

### "May/might"
- Possibilitat → "pot": "El fitxer pot estar corromput"
- Probabilitat → "és possible que" + subjuntiu

## Gramàtica

### Ser vs. Estar
- Usar **«ser»** per a localització física no abstracta: «El fitxer X **és** a la carpeta Y» (no «està»), «La base de dades **és** al disc»
- Davant d'adjectius, Softcatalà recomana **«ser»** (forma tradicional): «La base de dades **és** buida», «El disc **és** ple», «L'accés **és** restringit»
- **Excepcions** on només és correcte «estar»: «El programa **està** baixat», «El document **està** desconfigurat», «El disc **està** malmès/danyat»
- Davant de topònims: «ser» o «estar» admesos («Serà **a**/**en** València»)

### Negació doble
Obligatoria amb partícules negatives:
- "Cap ordre **no** s'ha d'escriure en majúscules" (no "cap ordre s'ha d'escriure")
- "Mai **no** s'ha de forçar el tancament" 
- "Ningú **no** pot accedir-hi"

### Possessius
Ometre quan el context és clar:
- ✅ "Voleu eliminar el fitxer?"
- ❌ "Voleu eliminar el vostre fitxer?"

### Gènere en llistes mixtes
Usar masculí (forma no marcada) per a grups mixts:
- "els fitxers i les carpetes **amagats**" (no "amagats/amagades")
- "els usuaris i les usuàries **registrats**" → millor "les persones **registrades**" o simplement "els usuaris registrats"

### Gènere neutre / genèric
Preferir formes genèriques: "tothom" (no "els desenvolupadors/les desenvolupadores"), "el personal", "la persona usuària"

### Preposicions
- **"per"** = agent/causa: "creat per l'usuari", "Programa traduït per Softcatalà", "S'ha cancel·lat per sobrecàrrega"
- **"per a"** = propòsit/destinació: "eines per a programadors", "fitxer per al servidor", "Programa per a la compressió de fitxers"
- Davant infinitiu pendent de fer: "Programa per baixar" (encara no s'ha baixat) vs "Programa per a baixar imatges" (finalitat)
- **"a"** per a moviment/destí: "anar a la carpeta", "desar a l'escriptori"
- **"en"** per a localització estàtica (abstracta): "en el menú", "en la llista"
- Formes curtes preferides: "damunt" (no "al damunt de"), "sota" (no "per sota de")

## Puntuació

### El·lipsis (...)
- Sense espai previ: "S'està carregant..." (no "S'està carregant ...")
- Al principi de frase, enganxat a la primera paraula: "...el nom del camp no pot ser buit"
- En botons que obren un diàleg: "Imprimeix..." (indica que hi ha més passos)

### Comes
- Ometre davant "i" o "o" en enumeracions simples: "Menús, barres d'eines i tecles"
- Obligatòria entre dues proposicions: "Feu clic al botó, i el fitxer s'obrirà"
- La coma va **dins** les cometes quan tanca una citació que ja porta coma

### Punt i coma
En llistes: els elements acaben en punt i coma, l'últim en punt. Si els elements són fragments breus sense verbe, no cal puntuació interna.

### Punt final
- En botons i etiquetes: **sense punt final**
- En missatges complets (frases amb verb): **amb punt final**
- En elements de llista que són frases complertes: **amb punt final**
- En elements de llista que són frases incompletes: **sense punt final**

### Dos punts
Quan introdueixen una llista o explicació, van seguits de minúscula (en català) si el que segueix és una llista: "Les opcions disponibles: fitxer, carpeta, vincle."

## Majúscules i minúscules

### Usar majúscula per a:
- Inici de frase i després de punt
- Noms propis (persones, llocs, empreses, marques)
- Sigles/acrònims: USB, HTML, PDF
- Primera lletra de noms de menú: "Fitxer", "Edita", "Visualitza"
- Primera lletra de títols de diàlegs: "Desa el document", "Preferències del sistema"
- Primer element d'una llista (fins i tot si és fragment)
- Paràmetres de línia d'ordres: "--gtk-module=MÒDUL"

### Usar minúscula per a:
- Noms de llengües: **català**, anglès, castellà
- Dies de la setmana i mesos: dilluns, gener
- Adjectius derivats de noms propis: "la interfície **del GNOME**" (no «gnòmica», que vol dir «de gnoms»)
- Elements d'una llista que no comencen frase (quan la llista és precedida per dos punts i els elements s'integren en el text)

## Sigles i acrònims

- Escriure en majúscules sense punts: **PC**, **USB**, **HTML**, **PDF**, **URL**
- Article determinat per gènere del concepte, no de l'acrònim: "**l'**HTML" (el llenguatge), "**el** PDF"
- Plural amb article, no afegint 's: "els PC" (no "els PC's" ni "els PCs")
- Primera menció d'acrònim poc conegut: forma completa en cursiva + traducció: "*Certificate Revocation List* (CRL, llista de revocació de certificats)"

## Abreviatures usuals en programari

| Abreviatura | Forma completa |
|-------------|----------------|
| etc. | etcètera |
| màx. | màxim |
| mín. | mínim |
| núm. | número |
| pàg. | pàgina |
| p. ex. | per exemple |
| fig. | figura |

## Veu activa vs. passiva

Preferir veu activa. Convertir passives quan és possible:
- ❌ "El fitxer és esborrat pel sistema" → ✅ "El sistema esborra el fitxer"
- ❌ "L'operació serà realitzada" → ✅ "El sistema realitza l'operació" o "L'operació es realitza"
- Passiva pronominal és acceptable: "El fitxer s'ha desat", "S'han trobat 3 errors"

## Barbarismes i castellanismes freqüents

| ❌ Evitar | ✅ Usar |
|-----------|---------|
| adreça IP (direcció) | adreça IP ✓ |
| guardar | desar |
| fer servir (col·loquial) | utilitzar (formal, preferit en UI) |
| arrastrar | arrossegar |
| xarxa local (red local) | xarxa local ✓ |
| ha (preposició) | a |
| cerca/recerca confosos | cerca (acció), recerca (investigació) |
| punt volat mal usat (·) | punt volat per a l·l: "col·leccions" |

### Apòstrof: recte (') vs. tipogràfic (’)

Softcatalà prefereix l'apòstrof tipogràfic/corbat (**’**, U+2019) sobre el recte de màquina d'escriure (**'**, U+0027) en **text de lectura** (documentació, pàgines web, prosa): és la forma correcta per a l'elisió en tipografia professional, coherent visualment amb cometes tipogràfiques (" ") o guillemets («»).

**Excepció important per a fitxers de programari (el cas d'aquesta skill)**: per a contingut destinat a **codi font, fitxers de cadenes (JSON, PO, resx...) o gestors de continguts**, cal fer servir l'apòstrof **recte** ('). No és una simplificació de segona categoria: és la recomanació correcta per a aquest context, perquè evita problemes de compatibilitat/renderització entre fonts, plataformes i eines de compilació, i és el que la majoria d'ecosistemes de traducció ja fan servir de manera consistent.

**Regla pràctica**: si estàs traduint una cadena d'interfície o un fitxer de localització, usa l'apòstrof recte ('); si estàs escrivint documentació o text en prosa per llegir, usa el tipogràfic (’). Dins d'un mateix fitxer de cadenes, sigues consistent amb la forma predominant que ja hi hagi (si el fitxer té una barreja, compta ocurrències i normalitza cap a la majoritària).

## Errors gramaticals freqüents

| ❌ Error | ✅ Correcte |
|---------|------------|
| Hi han errors | Hi ha errors |
| Tenir que desar | Haver de desar / Cal desar |
| Donat que | Atès que |
| Doncs (causal) | Ja que / Perquè |
| En quant a | Quant a / Pel que fa a |
| Tals com | Com ara |
| sí/si confosos | sí (afirm.) / si (condicional) |
| No s'emmagatzemen cap contrasenya | No s'emmagatzema cap contrasenya |
| Universal el GitHub | Universal a GitHub |

## Concordança amb «cap»

Regla del català (DIEC): «cap» + nom singular → verb en singular:
- ❌ «No s'emmagatzemen cap contrasenya» → ✅ «No s'emmagatzema cap contrasenya»
- ❌ «No hi ha cap errors» → ✅ «No hi ha cap error»
- Excepció: «cap dels/de les» + plural (verb en plural): «No n'hi ha cap dels dos que funcioni» / «No n'hi ha cap que funcionin»

## Gerundis perifràstics a evitar

El gerundi català descriu acció **en curs** real; evita les construccions calcades de l'anglès:

| ❌ Anglicisme | ✅ Natural |
|---------------|------------|
| «S'està utilitzant l'alternativa» | «S'utilitza l'alternativa» (o «S'usa») |
| «Anant a la configuració…» | «Aneu a la configuració…» |
| «Va fent clic…» | «Feu clic…» |
| «Continuant amb el procés…» | «Continueu el procés…» |

El gerundi **sí** és vàlid per a acció simultània autèntica: «Mostra un indicador mentre s'està baixant» (acció realment en curs).

## Distincions semàntiques freqüents

| ❌ Confusió | ✅ Distinció |
|------------|-------------|
| enquesta / sondeig | **enquesta**: qüestionari estructurat (enquesta d'opinió, satisfacció); **sondeig**: enquesta ràpida/status poll, sounding-out |
| magatzem / emmagatzematge | **magatzem**: lloc físic; **emmagatzematge**: concepte, acció (storage del programari) |
| construir / crear / generar | **construeix**: build físic/compilació; **creat per X**: atribució d'autoria (`Built by X`); **genera**: output automàtic |
| codificació / programació (d'IA) | **programació**: acció humana (`AI coding` = `programació amb IA`); **codificació**: transcodificació de dades (UTF-8) |
| desar / guardar | **desar**: acció save (Softcatalà); **guardar**: castellanisme (evitar) |
| resta / queda | **resten N crèdits** / **queden N crèdits**: ambdós vàlids, però consistent en tot el fitxer |

## Redundàncies a evitar

Patró freqüent: **substantiu + participi del mateix root** = redundància:

| ❌ Redundant | ✅ Alternativa |
|-------------|----------------|
| Ús utilitzat | Ús consumit / Ús gastat |
| Llista llistada | Llista mostrada / Llista |
| Cerca cercada | Cerca realitzada / Cerca |
| Resultats resultants | Resultats obtinguts |
