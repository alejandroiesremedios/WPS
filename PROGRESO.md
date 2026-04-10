# WPS Interactivo — Documento de Progreso

## Descripción del proyecto
Aplicación web interactiva de evaluación para el módulo de Soldadura en Atmósfera Protegida (SAP).
El alumno rellena un formulario WPS (Welding Procedure Specification) y el sistema lo evalúa automáticamente.

**Archivos principales:**
- `index.html` — Formulario interactivo
- `script.js` — Lógica de evaluación y comportamiento
- `styles.css` — Estilos
- `calificaciones.doc` — Tabla de criterios de referencia (Word HTML)
- `calificaciones_wps.html` — Criterios de evaluación en HTML (para imprimir)

---

## Estructura de puntuación

| Sección      | Máximo | Criterio |
|--------------|--------|----------|
| Generales    | 2.0    | ID + Material + Preparación + Posición |
| Raíz (TIG)   | 1.5    | Proceso + Consumible + Gas + Tungsteno + Parámetros + Ejecución (× 0.25) |
| Refuerzo (TIG)| 1.5   | Igual que Raíz |
| Relleno (MAG) | 1.25  | Sin Tungsteno (× 0.25 × 5 bloques) |
| Peinado (MAG) | 1.25  | Sin Tungsteno (× 0.25 × 5 bloques) |
| Defectología  | 2.5    | 1 correcto=0.83, 2=1.66, 3+=2.5 |
| **TOTAL**     | **10** | |

---

## WPS 1 — Criterios de evaluación (referencia)

### Datos Generales
- Material Base: S235JR
- Largo: 240 mm | Ancho: 60-80 mm | Espesor: 4-10 mm
- Preparación bordes: Rectos | Unión: Ángulo (T)
- Posición AWS: 2F | Posición ISO: PB

### Capas

| Parámetro | Raíz (TIG) | Refuerzo (TIG) | Relleno (MAG) | Peinado (MAG) |
|-----------|------------|----------------|---------------|---------------|
| Proceso común | TIG | TIG | Semiautomática macizo | Semiautomática macizo |
| ISO | 141 | 141 | 135 | 135 |
| AWS | GTAW | GTAW | GMAW MAG | GMAW MAG |
| Ø consumible | 2.0 / 2.4 | 2.0 / 2.4 | 1.0 / 1.2 | 1.0 / 1.2 |
| ISO consumible | W3Si1 | W3Si1 | G3Si1 | G3Si1 |
| AWS consumible | ER70S-6 | ER70S-6 | ER70S-6 | ER70S-6 |
| Gas tipo | Ar 100% | Ar 100% | Ar + 18% CO2 | Ar + 18% CO2 |
| Gas ISO | I1 | I1 | M21-ArC-18 | M21-ArC-18 |
| Gas AWS | SG-A | SG-A | SG-AC-18 | SG-AC-18 |
| Caudal (l/min) | 6-8 | 6-8 | 8-14 | 8-14 |
| Tungsteno tipo | Lantano 1.5% | Lantano 1.5% | — | — |
| Tungsteno ISO | WL15 (Oro) | WL15 (Oro) | — | — |
| Ø tungsteno | 2.4 | 2.4 | — | — |
| Empuje/Arrastre | — | — | Empuje | Empuje |

---

## Funcionalidades implementadas

- [x] Selector de WPS (1 al 4)
- [x] Sección de datos generales con validación
- [x] 4 capas de soldadura con campos adaptativos según proceso
- [x] Campos GTAW (tungsteno) se ocultan automáticamente en capas MAG
- [x] Campos MAG (voltaje, vel. hilo, empuje) se ocultan en capas TIG
- [x] Opciones de proceso: GTAW, GMAW MIG, GMAW MAG, FCAW, SMAW
- [x] Consumibles, gas y tungsteno con validación de pares ISO/AWS
- [x] Módulo de defectología (tipo, causa, acción correctiva)
- [x] Bloqueo del formulario hasta rellenar los 4 campos obligatorios
- [x] Contador de puntuación parcial en tiempo real (barra de progreso)
- [x] Batería de indicadores por sección en el panel lateral
- [x] Botón "Finalizar y Evaluar" con modal de resultado final
- [x] Tabla `calificaciones.doc` con criterios WPS 1-4 (se eliminó WPS 5)

---

## Historial de cambios

| Fecha | Cambio |
|-------|--------|
| Sesión anterior | Estructura inicial de la aplicación |
| Sesión anterior | WPS 5 eliminado del documento calificaciones.doc |
| Sesión anterior | Campos tungsteno ocultados en capas 3 y 4 (MAG) |
| Sesión anterior | Añadidas opciones GMAW MIG y GMAW MAG (se quitó GMAW genérico) |
| Sesión anterior | Auto-puntuación tungsteno eliminada para capas no-GTAW |
| Sesión anterior | Máximos de sección ajustados: relleno=1.25, peinado=1.25 |
| Sesión anterior | defecto.max aumentado a 2.5 para mantener total=10 |
| Sesión anterior | Barra de puntuación parcial añadida sobre botón Finalizar |
| Sesión anterior | Botones apilados verticalmente y más anchos |
| Sesión actual | evalDefecto corregido: max 2.5 (antes 2.0); 1=0.83, 2=1.66, 3+=2.5 |
| Sesión actual | Labels de batería relleno/peinado corregidos: "0 / 1.25" (antes "0 / 1.5") |

---

## Problemas pendientes / por verificar

### Consumible capa 3 (relleno) no puntúa
- **Síntoma:** El usuario reporta que rellenando consumible correcto en capa 3 no suma puntos.
- **Estado:** No se ha podido reproducir. La lógica de código es correcta:
  - `gmawFillerMap.validPairs['G3Si1'] === 'ER70S-6'` ✓
  - `layerConfig.consumible.iso = 'G3Si1'`, `aws = 'ER70S-6'` ✓
  - IDs en HTML: `rellenoAportacionDiam`, `rellenoAportacionIso`, `rellenoAportacionAws` ✓
- **Posible causa:** El usuario podría no estar seleccionando el Ø (diámetro = 1.0 o 1.2). Los tres campos (Ø, ISO y AWS) deben ser correctos para que puntúe el bloque de consumible.
- **Acción:** Verificar con el usuario si seleccionó los 3 campos del consumible correctamente.

### Revisión completa de scoring (solicitada)
- El usuario pidió una revisión completa de la puntuación para todos los WPS.
- Se han revisado: proceso, consumible, gas, tungsteno, parámetros, ejecución, defectología.
- La lógica parece correcta para WPS 1. Falta verificar WPS 2, 3 y 4.

---

## Notas técnicas importantes

- La visibilidad de `rellenoDetalles` se controla con `style.display = 'block/none'` en JS.
- `evalOneLayer` retorna 0 si el div de detalles no está visible.
- Los campos ocultos (`.gtaw-only`, `.mag-only`) se excluyen de la validación con `offsetParent === null`.
- Las opciones de consumibles se regeneran (shuffle) cada vez que se selecciona un proceso.
- `recalculateAll` se dispara en el evento `change` de todos los campos relevantes.
- El orden de evaluación: generales → raíz → refuerzo → relleno → peinado → defecto.
