# Hospiten Contigo · Handoff

Prototipo web (React 19 + Vite 8 + Tailwind 4) de una app móvil para el **Hackathon Talento Canario 2026**, reto de Hospiten: mejorar la comunicación hospital–paciente antes, durante y después de la consulta. "Hospiten Contigo" es una sección nueva **dentro de la app que Hospiten ya tiene**, no una app nueva. Se presenta **a Hospiten**.

- **URL pública:** https://danielmaudesdomenech.github.io/hospiten-contigo/
- **Repo:** https://github.com/DanielMaudesDomenech/hospiten-contigo (público)
- **QR:** `qr.png` (fuera del repo, en la carpeta del proyecto; codifica la URL de arriba, que no cambia con cada push)
- **Origen:** exportado de Figma Make (`.figma/make/*` son restos de esa herramienta; solo `site.json` se usa en el build)

## Reglas del proyecto (decididas con el equipo)

1. **Estilo fiel a la app real de Hospiten.** El público es Hospiten.
2. **Todo lo que sea comunicación hospital–paciente está en alcance**, incluidos "Solicitar intérprete", "Tengo una duda" y "Hablar con una persona". No quitarlos.
3. **Fuera de alcance** (pagos, navegación dentro del hospital, resto de la app): puede existir como decorado, pero **no debe ser navegable** ni dejar botones muertos.
4. **Credibilidad:** nada que prometa más de lo que dice la presentación. Datos claramente ficticios. Cada envío simulado lo indica ("Envío simulado…").
5. **Cuidar la redundancia** en código y textos: reutilizar componentes y estilos existentes.
6. Aviso fijo obligatorio arriba: "Prototipo conceptual · Hackathon Talento Canario 2026 · No es una app oficial de Hospiten".

## Flujo (en este orden)

| # | Función | Pantalla(s) (`Screen` en `src/App.tsx`) | Entrada |
|---|---|---|---|
| 1 | Asistente telefónico de citas | `call` | Módulo "Asistente de citas" (Inicio / Contigo) y paso "Cita" del recorrido |
| 2 | Pre-consulta (chat en el idioma del paciente, resumen en español) | `language` → `pre` → `summary` → `preSent` | SMS al final de la llamada, módulo "Pre-consulta" |
| 3 | Turno en vivo (+ compartir con familiar) | `turn`, `share`, `shareSent`, `consultation` | Inicio, recorrido de la cita |
| 4 | Traducción en consulta (+ intérprete) | `translate`, `interpreter` | Consulta, módulo "Traductor" |
| 5 | Informe explicado (+ duda a la consulta) | `reports`, `report`, `question`, `questionSent` | Pestaña Informes |
| — | Privacidad | `settings` | Pestaña "Ajustes" y avatar |
| — | Hablar con una persona | `human` | Pie de varias pantallas |

### Asistente telefónico (`call`)
- Llamada simulada con temporizador real y pulso animado mientras está activa.
- "Simular respuesta" revela el siguiente turno de `callTurns`. Cada turno tiene `summary` (campos que aprende el asistente).
- "Pasar con una persona" (desde la 1.ª respuesta): Laura (equipo de citas) recibe un **resumen que depende de cuántas respuestas se dieron**; los campos con la misma etiqueta se sobrescriben (p. ej. "Estado").
- El asistente **no da información clínica** (lo dice en la llamada y en el aviso).
- Final: SMS de confirmación con enlace → pre-consulta. "Ver la cita en la app" → detalle de cita.

### Ajustes y privacidad (`settings`)
- Aviso de privacidad.
- Persona de confianza (pensada para personas mayores): se elige de los familiares vinculados (`RelativePicker`, compartido con "Compartir turno"), con permisos separados (informes y diagnósticos / citas / turno) revocables; se puede quitar.
- Reglas de edad: menores de 14, la cuenta la gestiona el tutor; de 14 a 17, decide el menor.
- Consentimiento para usar datos seudonimizados para mejorar la IA: **desactivado por defecto**.
- Protección de datos (información obligatoria, primera capa RGPD): texto de ejemplo, el legal definitivo lo fija Hospiten.

## Estructura

- `src/App.tsx`: toda la app. Navegación por estado (`screen` + `history`), **sin router** (recargar vuelve a Inicio, sin 404). Datos de ejemplo arriba del archivo (`appointments`, `questions`, `callTurns`, `parameters`, `reportList`, `relatives`, `navItems`…). Componentes pequeños: `Button`, `IA`, `Toggle`, `RelativePicker`, `Success`.
- `src/Icon.tsx`: iconos SVG inline por nombre.
- `src/index.css`: Tailwind 4 + CSS propio (tokens `--color-petrol #11333d`, `--color-lime #cce98e`, `--color-app`). Fuente DM Sans (Google Fonts).
- `public/`: `hospiten-logo.png`, `favicon.svg`. Referenciar siempre con `import.meta.env.BASE_URL` (la app vive en `/hospiten-contigo/`).
- `vite.config.ts`: `base` = `/hospiten-contigo/` en build, `/` en dev. Plugins de Figma Make (solo dev, inofensivos).
- `.figma/make/site.json`: título, descripción, `lang=es`, `noindex`.
- `.github/workflows/deploy.yml`: en cada push a `main`, pnpm 10 + Node 22 → `pnpm build` → GitHub Pages.

### Presentación en pantalla
- Móvil (≤480 px): app a pantalla completa, sin barra de estado ni indicador de inicio falsos.
- Escritorio: centrada en un marco de móvil 393×852; se reduce con `zoom` en pantallas bajas.
- `theme-color` petróleo.

## Trabajo local

```sh
pnpm install
pnpm dev        # http://localhost:8443
pnpm build      # genera dist/
```
Requiere Node 22 y pnpm 10 (ver `.mise.toml`). `pnpm build` **no** hace type-check; para revisarlo: `pnpm exec tsc --noEmit`.

## Publicar

```sh
git add -A
git commit -m "Describe el cambio"
git push
gh run watch -R DanielMaudesDomenech/hospiten-contigo --exit-status
```
Tarda menos de un minuto. Si el móvil muestra una versión vieja, recargar.

En la máquina original (Windows) no hay Node: git y gh están en WSL (Ubuntu), autenticados como `DanielMaudesDomenech`, y el build solo lo hace GitHub Actions. Desde PowerShell: `wsl git push`, `wsl bash -lc "gh run watch -R DanielMaudesDomenech/hospiten-contigo --exit-status"`.

## Pendiente (propuesto, no aprobado aún)

Huecos del flujo:
- Pre-consulta: bloque común (motivo, desde cuándo, dolor 0–10, medicación y alergias, antecedentes) + preguntas por especialidad (Traumatología: cómo se produjo, movilidad, hinchazón; Pediatría: responde el tutor).
- Señal de alarma en la pre-consulta → aviso a enfermería.
- Informe explicado: mostrar el texto original junto a la explicación sencilla.
- Traductor: animar subtítulos al tocar el micro para que se perciba "voz a voz".

Textos:
- "Tu médico ya tiene el resumen" choca con "Envío simulado" en la misma pantalla.
- "Qué se suele hacer" en el informe suena a tratamiento; mejor "Qué preguntar a tu médico".
- "Este prototipo no realiza traducción automática" debilita la propuesta.
- Informe de alta con meta-texto ("No establece un diagnóstico en este prototipo").
- "Prototipo · Datos ilustrativos" al pie duplica el aviso superior.

Bugs menores:
- "Volver a mis respuestas" (resumen de pre-consulta) no permite editar.
- La pre-consulta viene con Alemán marcado por defecto (¿intencionado?).

Estilo: falta comparar con capturas de la app real de Hospiten (inicio, citas, informes).
