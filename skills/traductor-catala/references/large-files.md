# Traducció de fitxers grans per lots

Per a fitxers de centenars o milers de claus (JSON niuat de vue-i18n, ARB, XLYYF, PO grans), una traducció "d'una tirada" fracassa: perd consistència, deixa claus sense traduir i no es pot validar. Fes servir aquest pipeline.

## Estructura de treball

```
projecte-trad/
├── en.json              # original anglès (font de veritat)
├── ca-weblate.json      # estat actual de la plataforma (base)
├── missing.txt          # dump "clau\tvalor_EN" de les faltants
├── batches/
│   ├── batch01.json     # traduccions per lots de secció
│   ├── ...
│   ├── fixes.json       # correccions sobre claus existents (base)
│   └── fixes2.json      # correccions posteriors (prioritat màxima)
└── ca-final.json        # resultat
```

## Passes

### 1. Diagnòstic
```python
def flatten(d, prefix=''):
    out = {}
    for k, v in d.items():
        key = f"{prefix}.{k}" if prefix else k
        if isinstance(v, dict): out.update(flatten(v, key))
        else: out[key] = v
    return out
```
- Aplana EN i CA, compara: faltants (set difference), extres (obsoletes a esborrar o ignorar), % real.
- Genera `missing.txt` amb `clau\tjson.dumps(valor_EN)`: serveix per llegir els lots i per verificar que cap clau inventada.
- Agrupa les faltants per primera secció de la clau per decidir la mida dels lots (~80-120 claus per lot va bé).

### 2. Traducció per lots
- Un fitxer JSON per lot: `{"clau.aplanada": "traducció"}`.
- Mantén un glossari de decisions a mesura que avances (veure SKILL.md); els lots posteriors el reutilitzen.
- Cada clau del lot ha d'existir a `missing.txt`: zero claus inventades.

### 3. Merge amb prioritat
Ordre d'aplicació (últim guanya):
1. Base (`ca-weblate.json`)
2. `fixes.json` (correccions sobre la base)
3. `batch01..N.json` (traduccions noves)
4. `fixes2.json` (correccions trobades en revisió: prioritat sobre tot)

Després del merge, **reordena el dict final exactament en l'ordre de claus de l'EN** i torna a niar (`unflatten`). Reordenar evita diffs sorollosos en la plataforma.

### 4. Gates automàtiques (totes han de passar)
```python
import re, json, collections

ph = lambda s: sorted(re.findall(r'\{[^}]+\}', str(s)))
bad = [(k, ph(en[k]), ph(final[k])) for k in final if ph(en[k]) != ph(final[k])]
assert not bad, bad              # multiconjunt de placeholders igual a l'EN actual
assert set(final) == set(en)      # claus exactes: 0 faltants, 0 extres
json.load(open('ca-final.json'))  # JSON vàlid
```
Més controls regex sobre el final:
- tractament tu residual si el fitxer va en vós
- castellanismes i typos coneguts (llista pròpia del projecte)
- el·lipsis: `...` → `…` fora de URLs
- pluralisme: `grep` dels termes descartats del glossari retorna 0

### 5. Revisió multipassada
Veure "Revisió multipassada" al SKILL.md: subagent revisor amb context fresc, verificar propostes contra l'EN, passades fins a rendiment decreient.

## Errors freqüents del pipeline

- Traduir sobre el fitxer del repo quan la plataforma (Weblate) va més avançada: es perd feina i es generen conflictes.
- Claus amb punts dins de valors niuats: l'aplanat amb punts pot col·lisionar si el JSON té claus amb literal `.`; comprova-ho abans (compta claus aplanades vs totals).
- Oblidar la reordenació final segons l'EN: el fitxer puja però el diff és inllegible per als maintainers.
- Aplicar correccions del revisor sense verificar l'EN: el revisor també al·lucina.
- No validar plurals `|` de vue-i18n segment a segment.
