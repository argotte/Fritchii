# Fritchii zidane

Fork personal de [Fritchii](https://github.com/Frizti/Fritchii) para jugar *The Legend of Zelda:
Ocarina of Time* (remake) con FPS estables en una laptop:

- CPU: AMD Ryzen 7 7435HS (Zen 3+, 8 núcleos / 16 hilos, 16 MB de L3)
- GPU: NVIDIA GeForce RTX 4060 Laptop (8 GB)
- RAM: 16 GB

## Base

La rama `zidane` parte de la etiqueta `fritchii-v6.2`. La rama por defecto del original (`nx2`)
quedó en el 6 de noviembre y no contiene la v6.2.

## Builds

`.github/workflows/zidane-windows.yml` compila Windows en cada push a `zidane` y publica una
pre-release `zidane-build-N` con dos zips del mismo commit:

| Variante | Qué cambia |
| --- | --- |
| `zen3` | `-march=znver3 -mtune=znver3` (preset `YUZU_BUILD_PRESET=zen3`), sin fusión FMA para que los resultados de coma flotante sean los del build estándar. Solo arranca en Zen 3 o más nuevo. |
| `standard` | La configuración de la release oficial. Sirve de referencia para comparar. |

En los dos:

- Se compila con clang-cl, como las releases oficiales, así que incluyen DLSS.
- El buscador de actualizaciones está apagado, porque apunta a los releases de otro proyecto.
- La barra de título muestra `v6.2 zidane-N zen3` o `v6.2 zidane-N standard`, gracias a
  `fritchii_build.txt` junto al `.exe`.
- Para todo lo demás siguen siendo la v6.2: usan los mismos datos, ajustes y caché de shaders.

## Qué limita a este equipo (análisis de la v6.2)

1. **RAM de 16 GB.** Los desarrolladores midieron en el mercado de la Ciudadela ~11.8 GB de uso
   sostenido y picos de 14.6 GB. Un equipo de 16 GB se congeló hasta 12 s mientras Windows usaba
   el disco como memoria (`src/yuzu/fritchii/known_issues.cpp`). Hay interruptores para ahorrar RAM
   (`FRITCHII_RAM_*`, `FRITCHII_JIT_WARM_CORES`), pero están apagados y sin medir; antes de
   activarlos hay que medirlos en este equipo.
2. **CPU.** El cuello de botella es el hilo que procesa los comandos de la GPU. La v6.2 se afinó en
   un Ryzen 9 5900X y en un 7800X3D, que tienen 64-96 MB de L3.
3. **VRAM de 8 GB.** Ya está cubierta: `FRITCHII_SMALL_CARD=2` (automático) activa el perfil de
   tarjeta pequeña, que deja la escala de resolución en 1x.
4. **Reparto de hilos.** En un 8C/16T sale bien: front de GPU en el núcleo 1, back en el núcleo 2 y
   el juego en los otros seis (`src/common/thread_pinning.cpp`).

## Cómo comparar builds

1. Usa los mismos ajustes en todos los builds: límite de 60 FPS con VSync y la misma resolución y
   DLSS.
2. Activa **Help > Profiling Mode** y reinicia.
3. Juega unos 10 minutos con el mismo recorrido, de preferencia el mercado de la Ciudadela.
4. Sube el zip de **Help > Export Log**. Las líneas `Hitch` nombran la causa de cada tirón
   (memoria, compilación, hilo de GPU).

## Próximos pasos

- Medir `zen3` contra `standard` contra la v6.2 oficial.
- Probar LTO (`ENABLE_LTO`, requiere lld-link).
- Armar un perfil para 16 GB de RAM a partir de los logs.
