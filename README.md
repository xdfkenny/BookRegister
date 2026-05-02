<p align="center"><img src="https://github.com/xdfkenny/BookRegister/blob/main/image.png?raw=true" width="35%" /></p>
<h1 align="center">Book Register</h1>
<p align="center"><b>Organiza una biblioteca personal a partir del ISBN; genera citas MLA y registra en Google Sheets.</b></p>
<p align="center"><a href="#overview">Overview</a> • <a href="#quick-start">Quick Start</a> • <a href="#architecture">Architecture</a> • <a href="#documentation">Documentation</a> • <a href="#scripts">Scripts</a> • <a href="#recommended-agent-workflow">Recommended Agent Workflow</a> • <a href="#folder-structure">Folder Structure</a> • <a href="#current-reality--caveats">Current Reality / Caveats</a></p>

---

## Overview

| Surface | Route | Description | Data Source |
|---|---:|---|---|
| Web app (UI) | `/` | Form para ingresar ISBN, mostrar metadatos del libro y la cita MLA generada. Incluye componentes de escáner de ISBN y resultados de cita. | Scraper de ISBN + Google Sheets (push histórico) |
| API / scraping flow (server/AI flows) | N/A (server-side flows en `src/ai/flows`) | Flujos para raspar datos desde páginas de búsqueda de ISBN y generar la cita MLA en JSON. | HTML scraping (Cheerio) / headless flow |
| AI / GenKit worker | N/A (dev tools) | Flujos y utilidades para generación asistida por GenKit/Google AI en `src/ai`. | `@genkit-ai/*` |
| Google Sheets integration | N/A (externo) | Persistencia opcional: envía registros de libros a una Google Sheet conectada. | Google Sheets (API / integraciones) |

## Quick Start

- Instalar dependencias:
  - bash
  - npm install
- Iniciar en modo desarrollo:
  - bash
  - npm run dev
- Compilar para producción:
  - bash
  - npm run build
  - npm start

## Architecture

- Tech stack:
  - **Framework:** Next.js (app router)
  - **Language:** TypeScript + React 18
  - **Styling:** Tailwind CSS
  - **AI / GenAI:** GenKit + @genkit-ai/googleai
  - **Scraping/parsing:** Cheerio (y/o flujos headless)
  - **Extras:** html5-qrcode (escáner), Firebase (dependencia presente), Google Sheets integration (externa)
- ASCII diagram (datos / flujo entre superficies):

  ```
  [User Browser]
		  |
		  |  (UI: /)
		  v
  [Next.js App - UI Components]
		  |-- calls --> [Scrape Flow / Server functions] --(scrape ISBN site)-->
		  |                                                       |
		  |<-- returns JSON (metadata + mla_citation) --------------|
		  |
		  |-- optional push --> [Google Sheets API]
		  |
  [GenKit / AI Worker]  (used for generation helpers / dev flows)
  ```

## Documentation

> Leer estos en orden si eres nuevo en el proyecto

**Core Docs**
1. [README.md](README.md) — visión general y arranque rápido.
2. [docs/blueprint.md](docs/blueprint.md) — especificación de scraping, reglas MLA y formato JSON.
3. `src/ai/flows/generate-mla-citation.ts` — lógica concreta para construir la cita MLA.
4. `src/ai/flows/scrape-book-data-from-isbn.ts` — extractores y selectores esperados.

**Specialized Docs**
- `src/ai/dev.ts` and `src/ai/genkit.ts` — scripts de desarrollo para GenKit.
- `src/components/isbn-scanner.tsx` — integración con `html5-qrcode`.
- `package.json` — scripts y dependencias (rutinario).

## Scripts

| Command | Purpose |
|---|---|
| `npm run dev` | Ejecuta Next.js en modo desarrollo (puerto `9002` por defecto). |
| `npm run genkit:dev` | Inicia GenKit para ejecutar `src/ai/dev.ts`. |
| `npm run genkit:watch` | Inicia GenKit en modo watch para desarrollo de flujos AI. |
| `npm run build` | Construye la aplicación para producción. |
| `npm run start` | Inicia el servidor Next.js en modo producción. |
| `npm run lint` | Ejecuta `next lint`. |
| `npm run typecheck` | Ejecuta `tsc --noEmit` (comprobación de tipos). |

## Recommended Agent Workflow

1. Clona y arranca local:
	- `git clone https://github.com/xdfkenny/BookRegister && cd BookRegister`
	- `npm install`
	- `npm run dev`
2. Revisar flujos de scraping:
	- Abrir `[src/ai/flows/scrape-book-data-from-isbn.ts](src/ai/flows/scrape-book-data-from-isbn.ts)` y verificar selectores según `docs/blueprint.md`.
3. Ejecutar y probar la generación MLA:
	- `npm run genkit:dev` (si necesitas iterar en los flujos GenKit)
4. Validar integración con Google Sheets (evitar insertar datos de prueba en production):
	- Verificar variables de entorno y credenciales antes de ejecutar push.
5. Añadir tests mínimos (recomendado):
	- Crear pruebas unitarias para el parser de `generate-mla-citation.ts`.
6. Para revisar componentes UI:
	- Abrir `src/app/page.tsx` y `src/components/citation-form.tsx` en el navegador en `http://localhost:9002`.

## Folder Structure

```
.
├─ apphosting.yaml                  # configuración de hosting
├─ components.json                  # metadatos de componentes
├─ next.config.ts                   # configuración Next.js
├─ package.json                     # scripts y dependencias
├─ postcss.config.mjs               # postcss/tailwind
├─ README.md                        # (este archivo)
├─ docs/
│  └─ blueprint.md                  # especificación de scraping y MLA
├─ src/
│  ├─ ai/
│  │  ├─ dev.ts                     # utilidades de desarrollo GenKit
│  │  ├─ genkit.ts                  # configuración GenKit
│  │  └─ flows/
│  │     ├─ generate-mla-citation.ts  # construye la cita MLA desde metadatos
│  │     └─ scrape-book-data-from-isbn.ts # extractor de datos por ISBN
│  ├─ app/
│  │  ├─ actions.ts                 # acciones del servidor / endpoints (si aplica)
│  │  ├─ globals.css                # estilos globales
│  │  ├─ layout.tsx                 # layout raíz
│  │  └─ page.tsx                   # página principal
│  ├─ components/
│  │  ├─ citation-form.tsx          # formulario ISBN / submit
│  │  ├─ citation-result.tsx        # vista del resultado de la cita
│  │  └─ isbn-scanner.tsx           # lector QR / ISBN
│  ├─ hooks/
│  │  ├─ use-mobile.tsx
│  │  └─ use-toast.ts
│  └─ lib/
	  ├─ placeholder-images.json
	  ├─ placeholder-images.ts
	  └─ utils.ts                   # utilidades varias
├─ image.png                         # imagen usada en el README/banderas
├─ .idx/icon.png                     # icono adicional
└─ "screenshot of the app.jpeg"      # captura de la app
```

## Current Reality / Caveats

<details>
<summary>Estado actual y advertencias importantes</summary>

- **Proyecto iniciado rápidamente**: El proyecto fue creado en sesiones cortas de "vibe coding" y se usó GenAI para gran parte del frontend (ver README original).
- **No insertar datos aleatorios**: **No** envíes entradas aleatorias a la Google Sheet conectada; puede contaminar datos reales.
- **Integración externa pendiente**: La conexión a Google Sheets y las credenciales pueden requerir revisión antes de ejecutar cualquier push de datos.
- **Pruebas ausentes**: Actualmente no hay pruebas unitarias explícitas en el repositorio; **se recomienda** añadir tests para el scraper y el generador MLA.
- **Lint / Typecheck disponibles**: Hay scripts para `lint` y `typecheck`, pero su cobertura y configuración deben verificarse localmente.
- **Posibles cambios en selectores de scraping**: Los selectores definidos en `docs/blueprint.md` y `src/ai/flows/scrape-book-data-from-isbn.ts` deben validarse regularmente ya que sitios externos cambian estructura.
</details>

<p align="center"><sub>Proyecto: Book Register — especificación y puesta en marcha. Para tocar integración con Google Sheets o añadir pruebas, dime cuál prefieres que aborde primero.</sub></p>