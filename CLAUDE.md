# VOR Navigator — Simulador de Navegacion Aerea

## Proyecto
- Simulador VOR interactivo para estudiantes y pilotos
- Dir: `/Users/andresbiscione/clau/vor-simulator/`
- Repo: `https://github.com/bikio2026/vor-simulator`
- Deploy: https://vor-simulator.vercel.app
- Archivo unico: `index.html` (monolitico, ~5500 lineas)
- Puerto dev: 3002 (`npx http-server -c-1 -p 3002`)

## Estado actual: v1.5
- Fase 1 completa (5 pasos)
- Ejercicios: Practica Libre, Identificar Posicion, Localizar Avion, Identificar Radial, Ajustar OBS, Paso por Estacion, Interceptar Radial, Triangulacion
- Sistema de ayuda con 3 pistas progresivas (-20% penalidad cada una)
- Modo Examen: 15 preguntas, 2 min/pregunta, certificado PNG descargable
- Mobile responsive con hamburger menu
- SEO: meta tags, OG, JSON-LD, favicon SVG

## Auditoria de fisica VOR (completada)

### Veredicto general
- La fisica del VOR es CORRECTA y segura para ensenar
- Funciones core verificadas: `computeVOR()`, `bearingFromTo()`, `angleDiff()`, `normAngle()`, `distanceBetween()`
- Todos los ejercicios producen respuestas correctas
- La validacion de respuestas usa tolerancias razonables (±5° posicion/radial, ±1-5° OBS segun dificultad)

### Bug CDI TO corregido (commit e3fe32b)
- **Problema**: la polaridad del CDI estaba invertida cuando flag=TO
- **Sintoma**: la aguja apuntaba en direccion opuesta a donde debia — "fly toward the needle" daba instruccion al reves
- **Causa**: en `computeVOR()` linea 2301, los argumentos de `angleDiff()` estaban intercambiados para el caso TO
  - Antes: `cdiAngle = angleDiff(normAngle(obsNorm + 180), radial)` — signo invertido
  - Fix: `cdiAngle = angleDiff(radial, normAngle(obsNorm + 180))` — correcto
- **CDI FROM**: siempre fue correcto (`angleDiff(obsNorm, radial)`)
- **Impacto**: afectaba textos descriptivos (describeCDI, prompts, hints) pero NO la validacion de respuestas (usan Math.abs)
- **Verificado con bateria de 16 tests** (detalle abajo)

### Bateria de tests ejecutados
16 escenarios contra `computeVOR()` con VOR en (0.5, 0.5), todos PASSED:

| # | Escenario | Radial | Flag | CDI | OK |
|---|-----------|--------|------|-----|-----|
| 1 | Avion al Norte, OBS=360 | 360 | FROM | center | ✅ |
| 2 | Avion al Este, OBS=090 | 090 | FROM | center | ✅ |
| 3 | Avion al Sur, OBS=180 | 180 | FROM | center | ✅ |
| 4 | Avion al Oeste, OBS=270 | 270 | FROM | center | ✅ |
| 5 | Avion al Norte, OBS=180 (inbound) | 360 | TO | center | ✅ |
| 6 | Avion al Este, OBS=360 (perpendicular) | 090 | — | full deflection | ✅ |
| 7 | Avion al NE (045), OBS=045 | 045 | FROM | center | ✅ |
| 8 | Avion al Este, OBS=080 (10° off) | 090 | FROM | full left | ✅ |
| 9 | Avion al Este, OBS=085 (5° off) | 090 | FROM | left ~50% | ✅ |
| 10 | Avion sobre estacion (cone) | — | OFF | — | ✅ |
| A | FROM: R-005, OBS=360 | 005 | FROM | left | ✅ |
| B | FROM: R-355, OBS=360 | 355 | FROM | right | ✅ |
| C | TO: R-185, OBS=360 | 185 | TO | right | ✅ |
| D | TO: R-175, OBS=360 | 175 | TO | left | ✅ |
| E | TO: R-095, OBS=270 | 095 | TO | right | ✅ |
| F | TO: R-085, OBS=270 | 085 | TO | left | ✅ |

Convencion verificada contra referencia: "si el avion esta a la IZQUIERDA del curso, la aguja va a la DERECHA" (Wikipedia CDI, PilotsCafe, AOPA, Studyflight).

### Analisis de ejercicios individuales
- **Identificar Posicion**: genera avion aleatorio, calcula radial, opciones por dificultad (cardinales/grados). Tolerancia ±5°. ✅
- **Localizar Avion**: mapa oculto, lee instrumento para deducir posicion. Usa getCardinal/getOctant. ✅
- **Identificar Radial**: avion visible en mapa, identificar su radial. Opciones con offsets. Tolerancia ±5°. ✅
- **Ajustar OBS**: dado un radial objetivo (inbound/outbound), ajustar OBS. Tolerancia ±1-5° segun dificultad. ✅
- **Interceptar Radial**: centrar CDI con flag correcto. Verifica 3 condiciones: OBS ±5°, CDI < 0.2, flag correcto. ✅
- **Paso por Estacion**: observacional (no graded), animacion lineal a traves de la estacion. ✅
- **Triangulacion**: dos VORs, centrar ambos CDI con flag FROM. Verifica 4 condiciones. ✅

### Issues pendientes menores
- 4 dots CDI en vez de estandar 5 (cada dot = 2.5° vs 2° estandar). Deflexion total ±10° es correcta.
- Cuadrantes corregidos a cardinales centrados: N(316-045), E(046-135), S(136-225), O(226-315)

### Recursos de referencia para verificacion externa
- **Luiz Monteiro VOR Simulator** (luizmonteiro.com) — el mas usado en escuelas de vuelo
- **Embry-Riddle ERAU Interactive VOR** (flightapps.erau.edu) — simulador universitario
- **Boldmethod VOR Quiz** — 5 escenarios posicion+OBS, interpretar instrumento
- **Studyflight VOR Lesson** (studyflight.com/vor) — confirma estandar 2°/dot, ±10° full scale
- **PilotsCafe Intercept & Track** (pilotscafe.com) — convenciones CDI, intercepcion de radiales
- **Wikipedia CDI** — convencion "fly toward the needle"
- **AOPA ABCs of VORs** — referencia general

## Problemas conocidos del Chrome MCP
- Los clicks del MCP extension no disparan event listeners del sidebar
- Workaround: usar `javascript_exec` para llamar funciones directamente (ej: `setMode('position')`, `showExamPrescreen()`)
- El http-server a veces cachea: matar y reiniciar + hard reload (cmd+shift+r)

## Notas tecnicas
- El grid principal usa `grid-template-columns: 280px 1fr 1fr`
- Mobile (<=600px) usa flex en vez de grid para evitar columna fantasma del sidebar fixed
- El sidebar tiene position: fixed en mobile con transicion slide-in
- Canvas del mapa usa coordenadas normalizadas (0-1)
- Instrumento VOR es SVG con viewBox 340x340, escalado por CSS wrapper a 280px
- Certificado generado con canvas 800x600 -> PNG download

---

## ROADMAP — Pendientes para proximas sesiones

### Bugs
- [ ] **No se puede salir del modo examen una vez iniciado** — Agregar boton "Abandonar" en la barra de progreso del examen

### Features cortas
- [ ] **Toggle de sonidos** — Icono 🔊/🔇 en el header para activar/desactivar efectos de sonido
- [ ] **OBS con ruedita del mouse** — Scroll wheel sobre el instrumento: up=+1, down=-1. En mobile: drag circular sobre la perilla
- [ ] **Modo examen reubicado** — Moverlo abajo en la sidebar, o al header como icono, o pantalla aparte. Repensar la navegacion

### Features medianas
- [ ] **Selector de aviones** — Siluetas SVG/Canvas distinguibles. Candidatos propuestos:
  - PA-11 Cub (ala alta, tren convencional) — **CONFIRMADO por el usuario**
  - Cessna 172 (ala alta, tren triciclo) — el actual
  - Piper Cherokee PA-28 (ala baja, robusto)
  - Beechcraft Baron (bimotor, ala baja)
  - Cirrus SR22 (ala baja, moderno)
  - El usuario debe confirmar cuales incluir ademas del PA-11

### Features grandes
- [ ] **Tutorial interactivo completo** — Modo guiado paso a paso:
  - Pantalla de bienvenida: "Primera vez con VOR?"
  - 6-8 pasos interactivos con highlight del instrumento, flechas, texto explicativo
  - Cada paso pide una accion concreta ("Gira el OBS hasta 090", "Arrastra el avion al Este")
  - Al final suelta al usuario en practica libre
  - Estimacion: 1-2 sesiones de trabajo
  - Presentacion: overlay sobre el simulador real, no pantalla separada

### Fase 2 (futuro)
- [ ] **Migracion a Next.js** — Evaluar si vale la pena dado que el monolito funciona bien
  - El archivo tiene ~5500 lineas, manejable pero en el limite
  - Beneficios: componentes, routing, SSR para SEO
  - Costo: reescritura significativa
