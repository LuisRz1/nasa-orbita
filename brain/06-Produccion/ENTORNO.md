---
id: entorno
type: configuracion
status: inspeccionado-sin-instalacion
updated: 2026-10-14
tags: [orbita, herramientas]
---
# Entorno y herramientas

## Equipo de referencia

Inspección inicial del 11 de octubre de 2026: Ryzen 7 4800H, 8 núcleos/16 hilos; aproximadamente 16 GB de RAM; GTX 1650 Ti de 4 GB y gráfica Radeon integrada; Windows 10.

En esa inspección había aproximadamente 15 GB libres en C: y 117 GB en D:. Son una fotografía del momento, no valores monitorizados.

## Herramientas ya disponibles

Node 24.18.0, npm 11.6.2, pnpm 12.4.1, Git, GitHub CLI, Vercel CLI, Chrome y Obsidian. Se verificaron cuenta GitHub y acceso Vercel durante la investigación.

## Base de versiones investigada

Phaser 4.2.1; Vite 8.3.4; TypeScript 7.0.2; React 19.3.0; Zustand 5.0.15; Zod 4.6.6; Dexie 4.4.6; AI SDK 7.0.137; Vitest 5.0.3; Playwright 1.64.0.

Son referencias de investigación de octubre, no dependencias instaladas en este repositorio. Antes de instalar se revisarán engines, pares y documentación; versiones compatibles exactas quedarán fijadas en lockfile.

## Instalación futura

Usar dependencias locales y Tiled/LibreSprite desde sus distribuciones oficiales. Preferir D: para herramientas portables y cachés. Reutilizar Chrome para comprobaciones cuando sea compatible.

No modificar configuración global ni desinstalar herramientas existentes para preparar el proyecto. No instalar modelos locales pesados para LUMA.

## Abrir hoy

La única operación necesaria es clonar el repositorio y abrir brain como bóveda en Obsidian. Todavía no existen package.json, servidor de desarrollo o build de juego.
