# Repaso del Proyecto 4 — Ranking Sort en MPI (CS4052)

Web estática para repasar, paso a paso, qué se hizo en el proyecto de Ranking Sort y por qué se tomó cada decisión.

Incluye:

- El problema y por qué es paralelizable, con una calculadora de rankings.
- El primer diseño (informe parcial) y por qué se reemplazó.
- El esquema de 6 fases del enunciado (Input → Gossip → Broadcast → Sort → Local ranking → Reduce) con un **simulador interactivo** del ejemplo n = 18.
- Clasificación PRAM (CREW), métricas teóricas y una calculadora del modelo.
- Implementación en MPI, las tres versiones beta y el Quicksort de referencia.
- Resultados experimentales, contraste de hipótesis H1–H6, mejoras y preguntas de repaso.

## Ver la web

Abre `index.html` en el navegador, o activa GitHub Pages: **Settings → Pages → Deploy from a branch → `main` / root**.

Código y datos del proyecto: https://github.com/CesarPR30/Proyecto_Paralela_Ranking-Sort
