# DC-97 — Captura de evaluación de instalaciones

## ¿Qué es esto?

Un sistema para capturar en campo los datos del formulario oficial **DC-97-S**
de evaluación de instalaciones del LDC/RDC y volcarlos automáticamente al
`DC-97-S.xlsm`. Hay **dos apps** que comparten el mismo formato de datos:

- **PWA** (`index.html`) — Progressive Web App que corre en cualquier celular,
  tablet o computadora sin instalar nada. Ideal para empezar rápido.
- **App Flutter** (`ldc_inspecciones/`) — App nativa para Android/iOS/Windows
  con base de datos local (SQLite), pensada para uso intensivo en campo.

Ambas exportan **exactamente el mismo JSON**, por lo que el mismo script de
vaciado (`vaciar_al_excel.py`) funciona con cualquiera de las dos.

## Archivos

| Archivo | Descripción |
|---|---|
| `index.html` | La PWA completa (HTML/JS, autónoma) |
| `manifest.json` | Configuración PWA (ícono, nombre) |
| `sw.js` | Service Worker para modo offline |
| `vaciar_al_excel.py` | Script Python: vuelca el JSON al DC-97-S.xlsm |
| `ldc_inspecciones/` | Proyecto de la app Flutter (opcional) |
| `README.md` | Este archivo |

## Cómo usar la PWA

### Opción 1 — Desde archivo local (más sencillo)
1. Pasa los 3 archivos (`index.html`, `manifest.json`, `sw.js`) a tu celular.
2. Abre `index.html` en Chrome o Safari.
3. ¡Listo!

### Opción 2 — Como PWA instalable (recomendado para campo)
1. Sube los archivos a un servidor web (GitHub Pages, Netlify, etc.).
2. Abre la URL en Chrome (Android) o Safari (iOS).
3. Toca "Añadir a pantalla de inicio" / "Add to Home Screen".
4. La app queda instalada como nativa y funciona sin internet.

### Opción 3 — Red local (para un equipo de evaluadores)
1. En la computadora con los archivos: `python3 -m http.server 8080`
2. Los demás dispositivos en la misma WiFi abren `http://[IP-de-la-PC]:8080`

## Flujo de trabajo

1. **Sitio** — Datos generales: descripción, fecha, año en curso, dirección,
   clima, unidad de medida, uso principal, oficina (RDC/LDC), número de
   congregaciones, auditorios y edificios.
2. **Edificios** — Por cada edificio: número WHQ, nombre, pisos, categoría de
   ocupación, área total, año de construcción, año de última remodelación
   grande, año proyectado de remodelación, condición de mantenimiento y
   condición del edificio.
3. **Evaluación** — Por cada elemento del catálogo (62 elementos, 297
   componentes), marca los componentes presentes y captura: cantidad (con su
   unidad L/A/N), año de reemplazo, condición (1-4), ajuste de años restantes e
   información adicional. Puedes **duplicar** un componente del catálogo,
   agregar uno **personalizado**, u **ocultar** elementos que no apliquen.
4. **Exportar** — Descarga `DC97_<WHQ>_<AAAAMMDD>.json` (también sirve de
   respaldo y se puede re-importar).

## Valores de condición

**Condición del componente (1-4)** — usada en cada componente evaluado:

| # | Descripción |
|---|---|
| 1 | Buena |
| 2 | Aceptable |
| 3 | Defectuosa |
| 4 | Seriamente Defectuosa |

**Condición de mantenimiento** (por edificio):

| # | Descripción |
|---|---|
| 1 | Promedio |
| 2 | Buena |
| 3 | Mala |

**Condición del edificio** (por edificio):

| # | Descripción |
|---|---|
| 1 | Se Necesita Remodelación Grande |
| 2 | Se Necesita Remodelación Pequeña |
| 3 | Reemplazar o Desechar |
| 4 | No Se Necesita Construcción |

## Vaciar al Excel

El script acepta el JSON de **cualquiera de las dos apps** (PWA o Flutter),
porque ambas usan el mismo formato.

```bash
# Instala la dependencia (una sola vez)
pip install openpyxl

# Vuelca el JSON al formulario oficial
python3 vaciar_al_excel.py DC97_WHQ-1234_20260610.json DC-97-S.xlsm

# Opcional: especificar el nombre del archivo de salida
python3 vaciar_al_excel.py datos.json DC-97-S.xlsm salida.xlsm
```

Genera una copia `DC-97-S_LLENADO_<fecha>.xlsm` sin tocar el original.
Conserva las macros VBA (`keep_vba=True`).

El script:
- llena el encabezado del sitio buscando las etiquetas en la hoja "Evaluación";
- coloca los datos de cada edificio a partir de la fila "Edificio 1";
- escribe cada componente evaluado en la primera fila libre de su elemento,
  con unidad, cantidad, año, condición, ajuste e información adicional;
- informa en consola qué se escribió y qué no encontró sitio en la plantilla.

## Estructura del JSON exportado

```jsonc
{
  "version": "DC-97-S",
  "evalId": "…",
  "exportDate": "ISO-8601",
  "header": { "desc","fecha","anio","dir","clima","unidad","uso",
              "oficina","cong","audit","numEdif","whq" },
  "buildings": [
    { "id","numWHQ","nombre","pisos","catOcup","area","anioConst",
      "anioRemLastRem","anioRemProy","condMant","condEdif" }
  ],
  "hiddenElems": ["<elemNum>_<edificioIdx>", …],
  "evaluacion": [
    { "key","categoria","elemento","elementoNum","componente","unidad",
      "instancia","edificioIdx","edificio","cantidad",
      "anioUltimoReemplazo","condicion","ajusteAnios","informacionAdicional" }
  ]
}
```

## Notas importantes

- La PWA guarda automáticamente en el navegador (localStorage) y funciona en
  modo avión tras la primera carga; la app Flutter guarda en SQLite local.
- El JSON exportado es a la vez respaldo y formato de intercambio.
- Las macros VBA del Excel original se preservan al guardar.
- Según la versión de tu plantilla DC-97-S, algunos campos con listas
  desplegables o fórmulas pueden requerir un ajuste manual tras el vaciado;
  revisa siempre el archivo en Excel antes de enviarlo a la oficina.
