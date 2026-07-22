# Aspectes de localització per al català

## Identificadors de locale

| Variant | Codi |
|---------|------|
| Català genèric (recomanat) | `ca` |
| Català d'Espanya | `ca_ES` |
| Català de França | `ca_FR` |
| Català d'Andorra | `ca_AD` |
| Català d'Itàlia | `ca_IT` |
| Valencià | `ca@valencia` |

Usar `ca` genèric sempre que sigui possible per a màxima compatibilitat.

## Formats de número

### Separadors
- **Decimal**: coma `,` → 3,14159
- **Milers**: punt `.` → 1.234.567
- Exemple complet: **1.234.567,89**

### Percentatge
**Sense espai** entre número i símbol (norma Softcatalà): **75%** (no «75 %»)
- ❌ «10 % completat» → ✅ «10% completat»
- Font: https://www.softcatala.org/guia-estil-de-softcatala/aspectes-de-localitzacio/ (secció «Percentatges»)

### Nombres ordinals
Sense punt, amb la lletra final superíndex o adjunta:
- 1r, 2n, 3r, 4t, 5è (masculí)
- 1a, 2a, 3a, 4a, 5a (femení)
- No usar "1er", "2do" (castellanisme)

### Nombres grans
| Anglès | Català | Nota |
|--------|--------|------|
| thousand | mil | 1.000 |
| million | milió | 1.000.000 |
| **billion** | **mil milions** | ⚠️ NO "bilió" (fals amic!) |
| trillion | bilió | 10^12 en sistema llarg |

## Formats de data

### Format estàndard
**dia de mes de any** — tot en minúscula, sense separadors:
- ✅ 11 de setembre de 2000
- ❌ 11/09/2000 (evitar en text)
- ❌ September 11, 2000

### Format numèric (quan escai)
dd/mm/aaaa: **25/12/2024**

### En codi de format (format strings)
Convertir de l'anglès:
| Anglès | Català |
|--------|--------|
| `MMMM d, yyyy` | `d MMMM yyyy` |
| `dddd, MMMM d, h:mm tt` | `dddd, d MMMM, H:mm` |
| `MM/dd/yyyy` | `dd/MM/yyyy` |

### Mesos (minúscula en català)
gener, febrer, març, abril, maig, juny, juliol, agost, setembre, octubre, novembre, desembre

### Dies de la setmana (minúscula en català)
dilluns, dimarts, dimecres, dijous, divendres, dissabte, diumenge

## Format d'hora

- **Rellotge de 24 hores** com a estàndard: **17:15:00**
- Separador: dos punts `:`
- Si cal AM/PM per compatibilitat: a. m. / p. m. (amb punts, minúscula)
- Zones horàries: UTC+1, CET (hora central europea)

## Moneda

### Format
Símbol **després** del número, espai entremig:
- ✅ 1.234,50 €
- ❌ €1.234,50
- ❌ 1.234,50€

### Noms ISO de monedes (ISO 4217-3)
Usar noms estàndards del recurs de Softcatalà:
- EUR → euro
- USD → dòlar dels Estats Units
- GBP → lliura esterlina
- JPY → ien japonès

Font: https://www.softcatala.org/estandard-iso-catala/monedes/

## Números de telèfon

Format dels territoris de parla catalana amb grups de tres dígits separats per espai:

| Territori | Prefix | Exemple |
|-----------|--------|---------|
| Espanya | +34 | 971 123 456 / +34 971 123 456 |
| Andorra | +376 | 123 456 / +376 123 456 |
| França | +33 | 04 12 34 56 78 / +33 4 12 34 56 78 |
| Itàlia | +39 | 06 1234 5678 / +39 06 1234 5678 |

- ❌ (971) 123-456 (format americà)
- ❌ 971-123-456

## Adreces

Ordre català: Carrer/Avinguda, número, pis, codi postal, localitat, país
- Carrer de Provença, 123, 3r 2a
- 08036 Barcelona
- Catalunya, Espanya

## Unitats de mesura

Sistema mètric decimal (SI):
- Longitud: mm, cm, m, km (no pulgades, peus, milles)
- Pes: g, kg (no lliures, unces)
- Volum: ml, l (no pintes, galons)
- Temperatura: °C (no °F)
- Velocitat: km/h (no mph)

Excepcions acceptades en programari:
- Polzades (″) per a mides de pantalla: monitor de 27″
- Punts tipogràfics (pt) per a mides de font

## Noms de països i llengües

### Noms de països (ISO 3166-3)
Font autoritzada: https://www.softcatala.org/estandard-iso-catala/paisos/
Exemples:
- France → França
- Germany → Alemanya
- United States → Estats Units
- United Kingdom → Regne Unit
- China → Xina

### Noms de llengües (ISO 639-3)
Font autoritzada: https://www.softcatala.org/estandard-iso-catala/llengues/
- English → anglès
- French → francès  
- Spanish → castellà (o espanyol, depenent del context)
- German → alemany
- Chinese → xinès

## Caràcters especials del català

Assegurar-se que la codificació és UTF-8 i que aquests caràcters es representen bé:
- Accent greu: à, è, ò
- Accent agut: á, é, í, ó, ú
- Accent circumflex: â (rar en català modern)
- Dièresi: ï, ü
- **Punt volat**: l·l (per a la geminada) — atenció: és `·` (U+00B7), NO punt simple
- **Ce trencada**: ç
- **Apòstrof tipogràfic**: ' (U+2019), preferit sobre el recte '

## Paper i marges

En documentació (no en UI):
- Format A4 (21 × 29,7 cm) — no carta americana
- Unitats en cm/mm, no en polzades

## Ordre alfabètic (collation)

El català segueix l'ordre estàndard llatí. Consideracions especials:
- La `l·l` s'ordena com una sola lletra entre `l` i `m`
- La `ny` s'ordena com dues lletres separades (n, y) en ordre estàndard
- TERMCAT segueix l'ordre IEC

## Configuració de teclat

Per a documents que expliquen configuració de teclat català:
- Distribució: `ca` (Català) en sistemes Linux/Windows
- Afegir al fitxer de configuració: `LANG=ca_ES.UTF-8`
- LC_ALL, LC_MESSAGES, LC_NUMERIC per a formats específics
