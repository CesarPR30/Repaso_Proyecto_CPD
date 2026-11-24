# Repaso del Proyecto 4 — Ranking Sort en MPI (CS4052)

Web estática para repasar, paso a paso, qué se hizo en el proyecto de Ranking Sort y por qué se tomó cada decisión.

**Abrir:** https://cesarpr30.github.io/Repaso_Proyecto_CPD/

## Videos y audios

| # | Video | Duración | Solo audio |
|---|-------|----------|------------|
| 1 | [Resumen completo](media/videos/resumen.mp4) | 7:45 | [m4a](media/audio/resumen.m4a) |
| 2 | [Preguntas y respuestas](media/videos/preguntas.mp4) | 11:50 | [m4a](media/audio/preguntas.m4a) |
| 3 | [Las fórmulas, término a término](media/videos/formulas.mp4) | 16:34 | [m4a](media/audio/formulas.m4a) |
| 4 | [Explicación detallada](media/videos/explicativo.mp4) | 18:01 | [m4a](media/audio/explicativo.m4a) |
| 5 | [El informe a fondo](media/videos/informe.mp4) | 17:07 | [m4a](media/audio/informe.m4a) |

En la sección **Videos y audios** de la web hay un reproductor de audio pensado para el celular: sigue sonando con la pantalla bloqueada, se controla desde la pantalla de bloqueo, pasa solo a la siguiente pista y recuerda dónde te quedaste. (Los navegadores del celular pausan los *videos* al bloquear la pantalla; el *audio* no.)

## Las fórmulas, término a término

[FORMULAS.md](FORMULAS.md): versión escrita del video 3. Recorre la cadena completa ($T_{\mathrm{comp}} \to T_s \to W \to \lambda \to T_{\mathrm{comm}} \to T_p \to S \to E \to C \to T_0$), explicando qué significa cada término, de dónde sale y cómo se deriva, con ejemplos numéricos del informe.

## Preguntas y respuestas

[PREGUNTAS_Y_RESPUESTAS.md](PREGUNTAS_Y_RESPUESTAS.md): 20 preguntas sobre el análisis (por qué de cada decisión y de dónde sale cada fórmula). También están en la web, sección **Preguntas y respuestas**, y narradas en el video 2.

## Contenido de la web

- El problema y por qué es paralelizable, con una calculadora de rankings.
- El primer diseño (informe parcial) y por qué se reemplazó.
- El esquema de 6 fases del enunciado (Input → Gossip → Broadcast → Sort → Local ranking → Reduce) con un **simulador interactivo** del ejemplo n = 18.
- Clasificación PRAM (CREW), métricas teóricas y una calculadora del modelo.
- Implementación en MPI, las tres versiones beta y el Quicksort de referencia.
- Resultados experimentales, contraste de hipótesis H1–H6, mejoras y preguntas de repaso.

Código y datos del proyecto: https://github.com/CesarPR30/Proyecto_Paralela_Ranking-Sort
