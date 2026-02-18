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
- La fisica del VOR es CORRECTA y segura para ensenar
- `computeVOR()`, `bearingFromTo()`, `angleDiff()` verificadas
- CDI deflection, TO/FROM flags, radiales: todo correcto
- Issue menor: 4 dots CDI en vez de estandar 5 (cada dot = 2.5 grados vs 2 grados)
- Cuadrantes corregidos a cardinales centrados: N(316-045), E(046-135), S(136-225), O(226-315)

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
