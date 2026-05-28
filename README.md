# Simulador de criticidad CHEC

Pagina web estatica desarrollada con Astro para presentar los avances del proyecto **Simulador de criticidad CHEC**, realizado entre CHEC y la Universidad Nacional de Colombia sede Manizales.

El sitio resume las secciones 1 a 9 del acta de trabajo `docsRef/docActa.pdf`, incorpora tablas ejecutivas de participantes, cronograma y entregables, y presenta los resultados actuales mediante la figura `resultados/metodolgia.png`.

## Contenido

- Resumen ejecutivo del acta de trabajo.
- Secciones 1 a 9 organizadas en tarjetas informativas.
- Tablas resumidas de participantes y cronograma.
- Entregables acordados del proyecto.
- Seccion de resultados actuales.
- Logos institucionales de Lab IA, CHEC y UNAL.

## Requisitos

- Node.js 22 o superior.
- npm.

## Ejecutar en local

Instalar dependencias:

```bash
npm install
```

Iniciar servidor local:

```bash
npm run dev
```

Abrir:

```text
http://localhost:4321/
```

## Probar localmente con base de GitHub Pages

```bash
npm run dev:pages
```

Abrir:

```text
http://localhost:4321/CHECUNAL_2026/
```

## Compilar para produccion

```bash
npm run build
```

La salida estatica queda en:

```text
dist/
```

## Despliegue en GitHub Pages

El repositorio incluye el workflow:

```text
.github/workflows/deploy.yml
```

Este workflow instala dependencias, ejecuta `npm run build` y publica la carpeta `dist/` en GitHub Pages.

En GitHub, activar:

```text
Settings > Pages > Build and deployment > Source: GitHub Actions
```

La URL esperada es:

```text
https://amalvarezme.github.io/CHECUNAL_2026/
```

## Configuracion de base

El archivo `astro.config.mjs` usa por defecto:

```js
const base = process.env.BASE_PATH ?? "/CHECUNAL_2026/";
```

Si el repositorio cambia de nombre, se debe actualizar ese valor para que coincida con el path de GitHub Pages.

## Estructura principal

```text
.
├── .github/workflows/deploy.yml
├── docsRef/docActa.pdf
├── logos/
├── resultados/
├── public/.nojekyll
├── src/pages/index.astro
├── astro.config.mjs
├── package.json
└── README.md
```

## Creditos

Proyecto desarrollado para CHEC y la Universidad Nacional de Colombia sede Manizales, con apoyo del Laboratorio de Inteligencia Artificial.
