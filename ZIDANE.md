# Fritchii zidane

Fork personal de [Fritchii](https://github.com/Frizti/Fritchii) para jugar *The Legend of Zelda:
Ocarina of Time* (remake) con FPS estables en una laptop:

- CPU: AMD Ryzen 7 7435HS (Zen 3+, 8 núcleos / 16 hilos, 16 MB de L3)
- GPU: NVIDIA GeForce RTX 4060 Laptop (8 GB)
- RAM: 16 GB

## Base

La rama `zidane` sigue las releases del original: partió de `fritchii-v6.2` y ya incluye
`fritchii-v6.5`. La rama por defecto del original (`nx2`) quedó en el 6 de noviembre y no contiene
esas versiones. Para traer una versión nueva: `git fetch upstream --tags` y
`git merge fritchii-vX.Y`.

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
- La barra de título muestra `vX.Y zidane-N zen3` o `vX.Y zidane-N standard`, gracias a
  `fritchii_build.txt` junto al `.exe`.
- Para todo lo demás son la versión oficial de la que parten: usan los mismos datos, ajustes y
  caché de shaders. El workflow exige el sello de shaders oficial de Windows (`143b6220486fb09b`),
  así que un build del fork nunca obliga a volver a prepararlos.

## Qué limita a este equipo (logs de la v6.2)

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

## Lo que mostraron los logs (21 nov 2026)

- Con el archivo de paginación de tamaño automático, Windows lo agrandaba en plena partida
  (límite de memoria de 23.7 a 28.8 y luego a 31.8 GB): congelones de 1.5 a 2.5 s. Un archivo de
  paginación fijo de 16 GB los quitó. Siguen congelones de 0.6 a 1.8 s en las cargas porque la RAM
  libre queda en ~1 GB. Lo que de verdad lo arregla es tener 32 GB de RAM.
- En la plaza el límite es el hilo "back" de la GPU: ~87 % de un núcleo, con la CPU a ~3.6 GHz y
  85-91 °C. Da 35-55 FPS ahí; el resto del recorrido, ~60.
- Destellos azules (vuelo del búho, entrada a la ciudadela): imágenes que `FRITCHII_COVERED_RETIRE=4`
  (de fábrica desde la v6.2) borró sin guardarlas y que el juego volvió a usar. Fueron 880 de 13 429
  en una sesión. La v6.5 lo corrige con `FRITCHII_COLD_SWEEP_KEEP_RESAMPLED=1` y
  `FRITCHII_COLD_SWEEP_KEEP_COVERED=1`. Apagar `COVERED_RETIRE` es arriesgado con 8 GB de VRAM.

## Cómo comparar builds

1. Usa los mismos ajustes en todos los builds: límite de 60 FPS con VSync y la misma resolución y
   DLSS.
2. Sin Profiling Mode, que cuesta 1-3 % de FPS. Juega unos 10 minutos con el mismo recorrido, de
   preferencia el mercado de la Ciudadela, con el mismo `nx2_switches.txt` en todos los builds.
3. Sube el zip de **Help > Export Log**. Las líneas `Pulse` dan los FPS de cada segundo y las
   `Hitch` nombran la causa de cada tirón (memoria, compilación, hilo de GPU).
4. Profiling Mode (**Help > Profiling Mode**) se usa solo para saber en qué gasta el tiempo cada
   hilo; no para comparar FPS.

## Próximos pasos

- Medir `zen3` contra `standard` contra la versión oficial.
- Probar LTO (`ENABLE_LTO`, requiere lld-link).
- Armar un perfil para 16 GB de RAM a partir de los logs.
