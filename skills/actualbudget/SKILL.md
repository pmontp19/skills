---
name: actualbudget
license: MIT
description: >
  Accés i anàlisi de la instància d'ActualBudget del Pere via l'API oficial
  @actual-app/api (headless, amb ActualQL per a consultes d'agregació). Activa
  sempre que l'usuari parli d'ActualBudget, "actual budget", "el budget",
  transaccions, categoritzar transaccions, comptes, despeses, ingressos, payees,
  regles de categorització, anàlisi de despeses, resum mensual, o qualsevol
  tasca financera que impliqui llegir o escriure dades al seu budget. No facis
  servir HTTP/REST ni MCP (no n'hi ha d'oficial); l'accés és sempre pel projecte
  node de sota. No cal demanar a l'usuari credencials ni URL del servidor cada
  vegada: ja són al .env.
---

# Accés a ActualBudget

Hi ha un projecte node **configurat i funcionant** a `~/actual-categorizer` que
ja té credencials vàlides al `.env` (no les demanis, no les mostris, no les
modifiquis). L'accés es fa sempre a través d'aquest projecte.

## Com funciona l'accés

ActualBudget **no** exposa HTTP/REST. L'API oficial `@actual-app/api` és
*headless*: baixa una còpia del budget al directori local `.actual-data/`,
l'operes en local, i crides `api.sync()` per pujar els canvis al servidor.

```js
await api.init({
  dataDir: path.join(__dirname, '.actual-data'),
  serverURL: process.env.ACTUAL_SERVER_URL,
  password: process.env.ACTUAL_PASSWORD,
});
await api.downloadBudget(
  process.env.ACTUAL_BUDGET_ID,
  process.env.ACTUAL_E2E_PASSWORD ? { password: process.env.ACTUAL_E2E_PASSWORD } : undefined,
);
// ... llegir/escriure ...
await api.sync();
await api.shutdown();
```

Les 4 variables al `.env` (ja omplertes): `ACTUAL_SERVER_URL`,
`ACTUAL_PASSWORD`, `ACTUAL_BUDGET_ID`, `ACTUAL_E2E_PASSWORD`.

## Scripts disponibles (a `~/actual-categorizer`)

Tots es corren amb `node <script>.js` des d'aquell directori.

**Lectura / dump:**
- `fetch.js` — bolca a `dump.json` només les **transaccions sense categoria** + la llista de categories. És el punt de partida per categoritzar.
- `fetch-all.js` — bolca **tot** (totes les transaccions, categories, payees, regles payee→categoria existents) a `dump-all.json`. Més pesat; usa'l quan necessitis historial complet.
- `fetch-budget.js` — bolca el budget per categoria/mes a `budget.json`.
- `explore-income.js` — inspecció específica d'ingressos.

**Categorització:**
- `categorize.js` — aplica les regles (keyword matching sobre payee/notes) a `dump.json` i genera `apply-plan.json` (aplicables automàticament) + `review.json` (cal confirmar). **Aquí hi ha les regles de categorització amb els IDs de categoria reals** — edita aquest fitxer per afegir/ajustar regles.
- `analyze.js` — agrupació/exploració de `dump.json` per trobar patrons de payees.
- `learn.js` — aprèn regles noves a partir de transaccions ja categoritzades (`dump-all.json`).
- `forn.js`, `split.js`, `move-classes.js` — scripts ad-hoc per casos concrets (forn, quotes escola, reclassificacions). Mireu-los abans de reutilitzar-los.

**Escriptura:**
- `apply.js` — aplica `apply-plan.json` (`{ updates: [{ id, categoryId }] }`) i fa sync.
- `make-rules.js` — crea regles permanents payee→categoria al servidor a partir de `rules-to-create.json`.

## Workflow estàndard

1. **Llegeix** sense escriure primer:
   ```bash
   cd ~/actual-categorizer && node fetch.js        # o fetch-all.js
   ```
   Llegeix el `dump.json` / `dump-all.json` generat.
2. **Decideix** la categoria de cada transacció (per payee, notes, import). Consulta les regles existents a `categorize.js` i els IDs de categoria al dump.
3. **Aplica** de dues formes:
   - Per transaccions puntuals: escriu `apply-plan.json` amb `{ updates: [{ id, categoryId }, ...] }` i executa `node apply.js`.
   - Per fer-la permanent per a un payee que es repeteix: crea una regla amb `make-rules.js` (molt més eficient a llarg termini).
4. **Verifica**: re-executa `fetch.js` per confirmar que ja no surten com a sense categoria.

## Notes importants

- **Mai** imprimeixis o mostris el contingut del `.env`. Si necessites el server URL o budget ID per alguna raó, llegeix-los directament sense mostrar-los a l'usuari.
- Corre sempre els scripts **des de `~/actual-categorizer`** (fan servir `path.join(__dirname, ...)` i `require('dotenv').config()` que carrega el `.env` local).
- Els imports a ActualBudget són en **cents** (enter). Usa `api.utils.integerToAmount(cents)` per mostrar-los en euros i `api.utils.amountToInteger(euros)` per escriure.
- Abans de fer canvis extensos, considera fer una còpia del `.actual-data/` o treballar primer sobre transaccions puntuals.
- Si es queixa de `downloadBudget` o sincronització: comprova que el servidor estigui online i que `ACTUAL_BUDGET_ID` sigui el sync ID correcte (Settings → Show advanced settings → Sync ID a la UI web).
- Per a tasques noves (no cobertes pels scripts existents), crea un script nou seguint el patró dels altres: `require('dotenv').config()`, `api.init(...)`, `downloadBudget(...)`, lògica, `sync()`, `shutdown()`.

## Quan NO usar aquesta skill

- Desenvolupament sobre el codi font d'ActualBudget mateix (el repo `actualbudget/actual`) — això és consumir l'API, no hackejar el producte.
- Si l'usuari vol un MCP de veritat (caldrà muntar un wrapper; ara mateix no n'hi ha cap de madur).

---

# Referència d'API (per a tasques noves i anàlisi)

Aquesta secció és per quan els scripts existents no cobreixen el que cal. Crea un
script nou a `~/actual-categorizer` seguint el patró d'`fetch.js` i fes servir
les operacions d'aquí sota. `api` = `require('@actual-app/api')`.

## Conceptes clau

- **Imports en cents** (enter): `5000` = 50.00€, `-1200` = despesa de 12.00€. Converteix amb `api.utils.integerToAmount(cents)` → euros (float) i `api.utils.amountToInteger(12.34)` → `1234`.
- **Dates** `YYYY-MM-DD`, **mesos** `YYYY-MM` (`'2026-01'`).
- **IDs** són UUIDs. Usa `api.getIDByName('accounts'|'categories'|'payees', 'Nom')` per buscar per nom.
- **Negatiu** = despesa, **positiu** = ingrés. Transaccions entre comptes (transferències) usen payees especials amb `transfer_acct`.

## Operacions comunes

### Budget / visió general
```js
const months = await api.getBudgetMonths();              // ['2026-01', '2026-02', ...]
const mes    = await api.getBudgetMonth('2026-01');      // { categoryGroups, incomeAvailable, ... }
const grups  = await api.getCategoryGroups();
```

### Comptes
```js
const comptes  = await api.getAccounts();
const saldo    = await api.getAccountBalance(accountId);
const nouId    = await api.createAccount({ name: 'Corrent', type: 'checking' }, 50000); // 500€ inicial
```

### Transaccions
```js
// Per rang de dates
const txns = await api.getTransactions(accountId, '2026-01-01', '2026-01-31');

// Importar amb dedup + rules (recomanat per imports bancaris)
const { added, updated } = await api.importTransactions(accountId, [
  { date: '2026-01-15', amount: -2500, payee_name: 'Mercadona', imported_id: 'bank-123' },
]);

// Actualitzar (p.ex. posar categoria)
await api.updateTransaction(txnId, { category: categoryId, cleared: true });
```

### Categories i payees
```js
const categories = await api.getCategories();            // [{ id, name, is_income, hidden, ... }]
const payees     = await api.getPayees();
const catId      = await api.createCategory({ name: 'Subscripcions', group_id: groupId });
```

### Pressupost (budget per categoria/mes)
```js
await api.setBudgetAmount('2026-01', categoryId, 30000); // 300€
await api.setBudgetCarryover('2026-01', categoryId, true);
```

### Regles (permanents: payee → categoria, etc.)
```js
const rules = await api.getRules();
await api.createRule({
  stage: 'pre',
  conditionsOp: 'and',
  conditions: [{ field: 'payee', op: 'is', value: payeeId }],
  actions:   [{ op: 'set', field: 'category', value: categoryId }],
});
```

### Sync & shutdown
```js
await api.sync();       // puja/baixa canvis al servidor
await api.shutdown();   // sempre al final
```

## Anàlisi amb ActualQL (clau per a agregacions)

Per a qualsevol pregunta d'anàlisi ("què he gastat en X aquest mes?", "top 10
payees aquest any", "comparativa mensual"), **usa ActualQL** en lloc de portar
totes les transaccions a JS. És molt més ràpid i net.

```js
const { q, runQuery } = require('@actual-app/api');

// Suma de despeses per categoria, un mes concret
const { data } = await runQuery(
  q('transactions')
    .filter({
      date:   [{ $gte: '2026-01-01' }, { $lte: '2026-01-31' }],
      amount: { $lt: 0 },
    })
    .groupBy('category.name')
    .select(['category.name', { total: { $sum: '$amount' } }])
);

// Top payees per despesa aquest any
const { data: top } = await runQuery(
  q('transactions')
    .filter({ date: { $gte: '2026-01-01' }, amount: { $lt: 0 } })
    .groupBy('payee.name')
    .select(['payee.name', { total: { $sum: '$amount' } }])
    .orderBy({ total: 'asc' })
    .limit(10)
);

// Cerca transaccions (p.ex. per paraula al payee)
const { data: hits } = await runQuery(
  q('transactions')
    .filter({ 'payee.name': { $like: '%mercadona%' } })
    .select(['date', 'amount', 'payee.name', 'category.name'])
    .orderBy({ date: 'desc' })
    .limit(20)
);
```

**Operadors:** `$eq`, `$lt`, `$lte`, `$gt`, `$gte`, `$ne`, `$oneof`, `$regex`, `$like`, `$notlike`.
**Splits:** `.options({ splits: 'inline' | 'grouped' | 'all' })`.
**Eixos:** filtres (`filter`), projecció (`select`), agrupació (`groupBy`), ordenació (`orderBy`), límit (`limit`).

Referència completa:
- API: https://actualbudget.org/docs/api/reference
- ActualQL: https://actualbudget.org/docs/api/actual-ql

## Patró per a scripts d'anàlisi nous

```js
require('dotenv').config();
const path = require('path');
const fs = require('fs');
const api = require('@actual-app/api');
const { q, runQuery } = require('@actual-app/api');

(async () => {
  await api.init({
    dataDir: path.join(__dirname, '.actual-data'),
    serverURL: process.env.ACTUAL_SERVER_URL,
    password: process.env.ACTUAL_PASSWORD,
  });
  await api.downloadBudget(
    process.env.ACTUAL_BUDGET_ID,
    process.env.ACTUAL_E2E_PASSWORD ? { password: process.env.ACTUAL_E2E_PASSWORD } : undefined,
  );

  // ... la teva consulta ActualQL o operació d'API aquí ...

  await api.shutdown();
})().catch((e) => { console.error(e); process.exit(1); });
```

Per anàlisi **sense escriptura**, pots ometre `api.sync()`. Per scripts que només
lligeixen, considera bolcar el resultat a JSON (`fs.writeFileSync`) a més de
mostrar-lo, per poder reutilitzar-lo sense reconnectar.
