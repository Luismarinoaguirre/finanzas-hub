# Finanzas Hub - Contexto del Proyecto

## Que es
App web SPA de finanzas personales. Todo el frontend esta en un unico `index.html` con CSS y JS inline. No usa frameworks, es vanilla HTML/CSS/JS.

## Estructura de archivos
```
finanzas-hub/
  index.html      ← App principal (~640KB, todo inline: HTML + CSS + JS)
  manifest.json   ← PWA manifest
  sw.js           ← Service Worker para PWA
  icon.svg        ← Icono de la app
  serve.sh        ← Script para levantar servidor local (puerto 8080, para uso manual)
  src/            ← Copia sincronizada de los archivos fuente
  backup/         ← Backups manuales
  .claude/        ← Config de Claude Code (launch.json para preview)
  CLAUDE.md       ← Este archivo
```

### Archivos legacy (NO activos, solo referencia)
```
  app.js          ← Backup viejo del JS (NO es el codigo activo)
  styles.css      ← Backup viejo del CSS (NO es el codigo activo)
  data.js         ← Datos de ejemplo viejos (NO es el codigo activo)
```

> ⚠️ **REGLA #1**: TODO el codigo activo esta INLINE en index.html. Los archivos .js y .css separados son backups viejos. Cualquier cambio se hace en index.html.

## Deploy
- **GitHub Pages**: https://luismarinoaguirre.github.io/finanzas-hub/
- **Repo GitHub**: `/Users/luismarino/Documents/GitHub/finanzas-hub/` (origin: https://github.com/Luismarinoaguirre/finanzas-hub.git)
- **Para deployar**: copiar index.html al repo GitHub, commit y push a main
- **Sync**: siempre sincronizar index.html a `src/` y al repo GitHub despues de cambios

## Stack tecnico
- HTML/CSS/JS vanilla (sin frameworks)
- Chart.js 4.4.1 (graficos)
- SheetJS/xlsx 0.18.5 (exportar Excel)
- PDF.js 3.11.174 (importar PDFs)
- PWA ready (manifest + service worker)
- Design system: Apple-clean con estetica Liquid Glass

## Paginas/Secciones (9 en nav)
1. **Inicio** (🏠) - Dashboard: KPIs, grafico evolucion (linea), categorias, top 5, cuotas activas
2. **Movimientos** (↕) - Lista con filtros (categoria, banco, moneda, busqueda), paginacion
3. **Mis Tarjetas** (🏦) - Tarjetas de Credito (primero) + Tarjetas de Debito, filtro por mes
4. **Inversiones** (📈) - Portfolio: KPIs, doughnuts por tipo/plataforma, tabla de activos, CRUD
5. **Comparar** (⇋) - Comparacion mes a mes
6. **Pendientes** (🔔) - Gastos capturados desde Google Sheets
7. **Fijos** (📌) - Gastos fijos/recurrentes
8. **Reportes** (📊) - Reportes avanzados con filtros
9. **Config** (⚙) - Configuracion (moneda, tema, categorias, datos)

### Paginas/features eliminados (no existen mas)
- ~~Tarjetas~~ → integrada en Mis Tarjetas
- ~~Cuotas~~ → movida al Dashboard (seccion "Cuotas activas")
- ~~Metas/Goals~~ → eliminada completamente (reemplazada por Inversiones)
- ~~Graficos Avanzados~~ → eliminada completamente (cubierta por Reportes)

## Estructura del index.html (~6400 lineas)
```
Lineas 1-16      → <head>, CDN scripts (Chart.js, XLSX, PDF.js)
Lineas 17-1310   → <style> inline (CSS completo, ~1300 lineas)
Lineas 1311-1320 → Early theme script (evita flash)
Lineas 1320-2700 → <body> HTML (paginas, modales, sidebar)
Lineas 2700-6400 → <script> inline (JS completo, ~3700 lineas)
```

### Secciones principales del JS
```
- State & Constants: DEFAULT_CATS, state object, DEFAULT_STATE
- Cloud Sync: loadState, saveState, cloudPush, cloudPull
- Navigation: nav(), mobileNav(), sidebar
- Theme: setTheme, initTheme, getChartColors
- PDF Import: processPDF, parseGaliciaPDF, parseGaliciaDebitPDF, parseBrubankPDF, parseBrubankVisaPDF
- Auto-categorization: auto_cat (regex rules) + learnedCat (ML from user history)
- Dashboard: renderDashboard, renderWeeklyChart, renderCatBars, renderTop5, renderDashCuotas
- Transactions: populateFilters, renderTransactions
- Cuentas: renderCuentas, renderTarjetas
- Cuotas: openNewCuota, saveCuota, deleteCuota, advanceCuota, renderDashCuotas
- Inversiones: renderInversiones, openNewInversion, saveInversion
- Fixed Expenses: renderFixed
- Reports: initReportFilters, generateReport
- Settings: renderSettings, setCatSort, import/export
```

## Persistencia de datos
- **localStorage key**: `finanzas_hub_v4` (JSON completo del state)
- **localStorage key**: `finanzas_hub_v4_ts` (timestamp para sync)
- **localStorage key**: `finanzas_hub_theme` (light/dark/system)
- **localStorage key**: `finanzas_hub_sidebar` (collapsed/expanded)
- **Cloud sync**: Google Apps Script, auto-push cada 3 seg via debouncedCloudPush

### Objeto `state`
```javascript
{
  transactions: [],      // Array de transacciones
  creditCards: [],       // Definiciones de tarjetas de credito
  cuotas: [],            // Pagos en cuotas
  inversiones: [],       // Portfolio de inversiones
  accounts: [],          // Cuentas bancarias (debito)
  income: { ars, arsVar, usd, usdVar },
  usdRate: 1380,         // Tipo de cambio USD/ARS
  alertPct: 85,          // Umbral de alerta de presupuesto
  categories: [],        // Categorias del usuario
  catSort: 'az',         // Orden de categorias: 'az' o 'custom'
  notes: {},             // Notas por mes
  dashMonth: null,       // Mes seleccionado en dashboard
  gsheetsUrl: '',        // URL de Google Sheets para sync
  processedSheetRows: [],
  recurringExpenses: [],
  recurringGenerated: {},
  budgets: {},           // Limites de presupuesto por categoria
}
```

## PDF Parsers
| Parser | Banco | Formato fecha | Detectado por |
|--------|-------|---------------|---------------|
| `parseGaliciaPDF` | Galicia (TC Amex/Visa) | DD.MM.YYYY | "DETALLE DE TRANSACCION" o "Banco de Galicia" |
| `parseGaliciaDebitPDF` | Galicia (Caja de ahorro) | DD/MM/YYYY | "Banco Galicia" + "Caja de ahorro" |
| `parseBrubankPDF` | Brubank (extracto cuenta) | DD-MM-YY | "brubank" + "Caja De Ahorro" |
| `parseBrubankVisaPDF` | Brubank (TC Visa) | YYYY-MM-DD | "brubank" + "Ciclo de facturación" |

## Preview config
```json
// .claude/launch.json
{
  "runtimeExecutable": "python3",
  "runtimeArgs": ["/tmp/finanzas-hub/server.py"],
  "port": 8080
}
```

**Antes de previsualizar**:
```bash
mkdir -p /tmp/finanzas-hub
cp index.html manifest.json sw.js icon.svg /tmp/finanzas-hub/
# server.py ya debe existir en /tmp/finanzas-hub/
```

> ⚠️ El sandbox del preview NO soporta `python3 -m http.server` directo (falla getcwd en Python 3.9). Usar el `server.py` custom que hace `os.chdir()` antes de importar http.server.

## Estilo de diseno
- **Light mode** (default): Apple/iOS clean — whitespace generoso, cards blancas con border sutil (black/5%), fondos #f8f9fb
- **Dark mode**: Liquid Glass con glassmorphism sutil
- **Font**: -apple-system / SF Pro Display / Inter
- **Accent**: Azul #3B82F6
- **Charts**: Lineas suaves (tension 0.4), sin puntos visibles, fill gradient, tooltip dark (#1e293b), sin grillas
- **Inspiraciones**: 14islands.com, Tailwind UI, Titan app

## Errores comunes a evitar (aprendidos en sesiones anteriores)

### 1. Editar archivos separados en vez de index.html
❌ Editar `app.js` o `styles.css` → no tiene efecto, el codigo activo es inline
✅ Siempre editar dentro de `<style>` o `<script>` en `index.html`

### 2. serve.sh y preview
❌ `python3 -m http.server` falla en sandbox del preview (getcwd error Python 3.9)
✅ Usar `server.py` custom con `os.chdir('/tmp/finanzas-hub')` antes del import

### 3. Regex de montos argentinos
❌ `[\d.,]+` matchea numeros de referencia (001743001743)
✅ `[\d.]+,\d{2}` requiere formato argentino con coma y 2 decimales

### 4. Espacio entre $ y monto
❌ `\$[\d.,]+` no matchea `$ 566.295,36` (con espacio)
✅ `\$\s*[\d.,]+` permite espacio opcional

### 5. DOM elements que ya no existen
❌ Crear funciones que referencian IDs de paginas eliminadas sin null guards
✅ Siempre verificar `if (!el) return;` antes de usar getElementById
✅ Al mover una feature de pagina, actualizar TODAS las llamadas a render

### 6. Duplicacion de selectores
❌ Tener dos selectores de mes separados (ej: cuentas-month-select + tc-month-select)
✅ Un unico selector con un unico onchange que llame a todas las funciones necesarias

### 7. Categorias basura en imports
❌ Los parsers a veces guardan montos como "$1.200,00" en el campo category
✅ Filtrar con `isJunkCat()`: `/^\$[\d.,]+$/.test(c) || /^\d+$/.test(c)`

### 8. Sincronizacion de archivos
❌ Editar index.html y olvidar copiar al repo de GitHub
✅ Siempre: editar → copiar a src/ → copiar a repo GitHub → commit + push
