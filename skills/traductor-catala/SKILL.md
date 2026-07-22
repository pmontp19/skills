---
name: traductor-catala
license: MIT
description: >
  Guia completa per traduir programari al català seguint els estàndards de Softcatalà, la Guia d'estil, les normes ISO i els recursos terminològics. Activa sempre que l'usuari demani traduir cadenes de text, missatges d'interfície, fitxers PO/POT, o qualsevol contingut de programari al català. També activa quan l'usuari vulgui revisar traduccions existents, comprovar si una traducció segueix els estàndards, pregunti sobre terminologia tecnològica en català, o necessiti ajuda amb decisions de localització (formats de data, números, tecles, etc.). Si l'usuari esmenta "softcatala", "po files", "gettext", "localització", "l10n", "i18n" en context català, usa aquesta skill. No esperis que l'usuari ho demani explícitament — si estàs traduint qualsevol cosa al català, aplica sempre aquestes normes.
---

# Traductor de Programari al Català

Ets un expert en localització de programari al català seguint els estàndards de Softcatalà. Aplica sempre les normes d'aquesta guia, fins i tot si l'usuari no les menciona explícitament.

## Principis fonamentals

- La traducció ha de semblar escrita originalment en català, no una traducció
- Consistència terminològica per sobre de tot: un terme, una traducció
- El registre és formal però no pompós; usar «vós» (plural de cortesia) per adreçar l'usuari
- No humanitzis l'ordinador (evita «Ho sento», «Si us plau», onomatopeies com «oh»)
- Consulta sempre les memòries de traducció i glossaris abans d'inventar termes nous
- Per a programari adreçat a infants es pot usar el tractament de «tu»

## Regles de consistència (molt importants)

Quan facis un canvi en un fitxer, aplica'l **a tot el fitxer** i verifica-ho amb un abans/després. Els mantenedors perden temps esmenant incoherències:

### Canvis de tractament (tu → vós)
- Si canvies una cadena a vós, revisa **TOTES** les cadenes del fitxer
- En **comandes** (botons, menús, accions), el tu/imperatiu ha de passar a vós. Patró sospitós en claus de comanda: `\b(afegeix|desa|elimina|cancel·la|tria|selecciona|comprova|tanca|obre|enganxa|prem|clica|verifica|mantén|defineix)\b` → `…eu`/`…iu` (`afegeix→afegiu`, `desa→deseu`, `obre→obriu`)
- ⚠️ **NO** apliquis aquesta regex a descripcions, subtítols, captions ni a frases que descriuen què fa l'app. Allà `desa`, `mostra`, `obre`, `cancel·la` són **3a persona del present d'indicatiu** i són correctes. Convertir-les a vós és un error habitual.
- Exemple d'error freqüent: «Store multiple API keys.» = «Desa diverses claus…» (3a persona, ✅) — no «Deseu diverses claus…» (vós en una descripció, ❌)
- No pot coexistir tu i vós al mateix producte

### Canvis terminològics
- Si canvies un terme (`Testimoni` → `Token`, `magatzem` → `emmagatzematge`, etc.), revisa totes les ocurrències
- Abans de lliurar: `grep -i 'terme-antic' Localizable.strings` ha de retornar 0 resultats

### Descripcions vs. comandaments
- **Botons, ordres de menú, accions**: imperatiu + vós (`Afegiu compte…`, `Deseu`, `Cancel·leu`)
- **Descripcions, subtítols, preferències, captions**: 3a persona (`L'actualització automàtica està desactivada`, `Mostra les icones de proveïdor`)
- Una mateixa pantalla pot barrejar ambdós estils; no es canvien les descripcions

**Test de decisió ràpid:**
1. La cadena **et diu a tu (usuari) què fer**? → vós imperatiu (`Deseu`, `Obriu`, `Afegiu`)
2. La cadena **descriu què fa l'app/la funció**? → 3a persona (`Desa`, `Obre`, `Afegeix`)
3. Dubtes? Anglès original en `-s` (3a persona: «Stores», «Shows», «Uses») → gaire sempre descripció → 3a persona
4. Si toques un subtítol/caption, revisa els **siblings** del mateix tipus per consistència (no corregegeixis un i deixis la resta en l'estil antic)

### Registre formal
- Prefereix **`utilitzar`** (formal) sobre `fer servir` (col·loquial) en UI i documentació
- Evita `fer servir` també en descripcions: «Utilitza el nom d'usuari…» (no «Fa servir el nom d'usuari…»)

### Validació de placeholders
Compta sempre els marcadors de format - el nombre ha de coincidir entre original i traducció:
- `%@`, `%d`, `%1$@`, `%2$s`, `\\(variable)` (Swift), `{0}`, `{name}` (.NET)
- Compta abans i després; si difereixen, la traducció és incorrecta

## Workflow per traduir

1. **Identifica el context**: Quin programa? Quina funció? Botó, missatge d'error, menú?
2. **Si treballes amb fitxers PO/POT** (gettext): respecta els marcadors de substitució (`%s`, `%d`, `{0}`, `$(...)`), NO els tradueixis; manté l'ordre HTML/Markdown i les etiquetes d'escapament
3. **Consulta recursos** per ordre de prioritat (veure `references/terminology.md`): Glossari Softcatalà → TERMCAT → Memòries de traducció → Glossaris Microsoft/Apple
4. **Aplica les normes** de la guia (detalls a `references/linguistic-rules.md` i `references/format-conventions.md`)
5. **Verifica coherència**: El terme escollit és consistent amb traduccions anteriors del mateix programa?
6. **Comprova localització**: Formats de data/hora/números correctes? (veure `references/localization.md`)
7. **Si cal variant valenciana** (`ca@valencia`): usa l'[adaptador de Softvalencia](https://www.softvalencia.org/adaptador/) sobre el text en català central

## Regles ràpides per a elements UI

| Element UI | Norma | Forma verbal |
|-----------|-------|--------------|
| Botons d'acció | Vós imperatiu: «Deseu», «Cancel·leu», «Accepteu» | vós |
| Ordres de menú | Vós imperatiu: «Obriu», «Tanqueu», «Imprimiu» | vós |
| Caselles de selecció (checkbox, etiqueta) | 3a persona o nominal: «Mostra la barra d'eines» | 3a persona |
| Subtítols/captions de casella o opció | 3a persona: «Mostra les icones al selector» | 3a persona |
| Botons d'opció (radio) | Frase nominal: «Ús reduït de dades» | nominal |
| Missatges de l'ordinador a l'usuari | Vós: «Esteu segur que voleu eliminar el fitxer?» | vós |
| Missatges d'error | Directes, sense disculpes: «No s'ha pogut obrir el fitxer» | — |
| Títols de diàlegs | Primera paraula en majúscula: «Desa el document» | — |
| Etiquetes de camp | Sense punt final, primera lletra majúscula: «Nom d'usuari:» | — |
| Consells d'eina (tooltips) | Concís, imperatiu (vós) o nominal | segons context |
| Indicadors d'estat (progress) | «S'està descarregant…» o «Descàrrega en curs…» | gerundi/nominal |

> ⚠️ **Ambigüitat crítica**: les formes `Desa`, `Obre`, `Mostra`, `Cancel·la` són **alhora** imperatiu de tu (incorrecte per a botons en estil vós) **i** 3a persona del present d'indicatiu (correcte per a descripcions). No les facis servir per a botons/menús si el producte va en vós — usa `Deseu`, `Obriu`, `Mostreu`, `Cancel·leu`. Però **sí** són correctes en subtítols i captions on descriuen què fa l'app.

## Errors freqüents a evitar

- ❌ «Ho sento, s'ha produït un error» → ✅ «S'ha produït un error»
- ❌ «Si us plau, introduïu el vostre nom» → ✅ «Introduïu el nom»
- ❌ «zipat / gzipat» → ✅ «comprimit en format zip / gzip»
- ❌ «anti-virus» → ✅ «antivirus»
- ❌ «billion» → ✅ «mil milions» (NO «bilió»)
- ❌ «Català» (nom de llengua) → ✅ «català» (minúscula)
- ❌ «Hi han errors» → ✅ «Hi ha errors»
- ❌ «Tenir que desar» → ✅ «Haver de desar» / «Cal desar»
- ❌ «Donat que» → ✅ «Atès que»
- ❌ «En quant a» → ✅ «Quant a» / «Pel que fa a»

## Referència de tecles (anglès → català)

| Anglès | Català |
|--------|--------|
| Backspace | Retrocés |
| Delete | Supr |
| Enter / Return | Retorn |
| Escape | Esc |
| Insert | Inser |
| Page Up / Down | Re Pàg / Av Pàg |
| Home | Inici |
| End | Fi |
| Shift | Maj |
| Caps Lock | Bloq Maj |
| Num Lock | Bloq Núm |
| Scroll Lock | Bloq Despl |
| Tab | Tab |
| Print Screen | Impr Pant |

## Falsos amics i calc habituals

| Anglès | ❌ Error freqüent | ✅ Correcte |
|--------|------------------|-------------|
| actual | actual | real, veritable |
| eventually | eventualment | finalment |
| billion | bilió | mil milions |
| file | fitxa / arxiu | fitxer |
| library | llibreria | biblioteca (de programari) |
| large | larg | gran |
| save | salvar | desar |
| statement | estat | sentència (codi) / declaració |
| success | succés | èxit |
| remove | remoure | elimina / suprimeix |
| link | vincle (en web) | enllaç (web) / vincle (en doc) |
| exit | èxit (l'aplicació) | surt / sortiu (menú, segons estil tu/vós) / sortida (cmd) |

## Checklist de revisió abans de lliurar una PR de localització

- [ ] **Tractament consistent**: no coexisteixen tu i vós en cap cadena del fitxer (menys en formes de 3a persona de descripcions, que són correctes)
- [ ] **Descripcions vs. comandes**: els botons/menús van en vós imperatiu (`Deseu`, `Obriu`); els subtítols/captions en 3a persona (`Desa`, `Obre`). No converteixis descripcions a vós.
- [ ] **Marcadors de format**: el nombre de `%@`, `%d`, `%1$@`, `\\(var)` coincideix entre original i traducció
- [ ] **Audit d'ocurrències**: `grep -i 'terme-antic'` retorna 0 per a qualsevol terme canviat (aplica també a formes verbals si has canviat el patró tu→vós)
- [ ] **Consistència entre siblings**: si toques un subtítol/caption, els de la mateixa secció segueixen el mateix estil
- [ ] **Sense coexistència tu/vós dins d'una mateixa cadena**: revisa clàusules unides per «o», «i», parèntesis
- [ ] **Validació de fitxer**: `plutil -lint` (Apple), `msgfmt -c` (gettext), o l'eina pròpia del projecte passa sense errors
- [ ] **Registre**: `utilitzar` per sobre de `fer servir` en UI formal; sense castellanismes ni calcs anglicistes

## Referències detallades

- `references/linguistic-rules.md` — Gramàtica, puntuació, majúscules, gènere, temps verbals
- `references/format-conventions.md` — Tipografia, guillemets, guions, sigles, símbols
- `references/localization.md` — Formats de data, hora, números, moneda, telèfon, unitats
- `references/terminology.md` — Recursos terminològics: TERMCAT, glossaris, memòries de traducció
