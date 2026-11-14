# Suficiencia energética del SIN 2027-2030 y expansión de mínimo costo

Tutorial del Trabajo Final de Taller de Programación en Python (MECA 4110, Universidad de los Andes, 2026-2). Autores: Carlos Andrés Terrassa Vallejo, Esteban Darío Herrera León y Jair Armando Castañeda Rodríguez.

[![Abrir en Colab](https://colab.research.google.com/assets/colab-badge.svg)](https://colab.research.google.com/github/cterrassa/suficiencia-sin/blob/main/Tutorial_Suficiencia_SIN.ipynb) [![Abrir en Binder](https://mybinder.org/badge_logo.svg)](https://mybinder.org/v2/gh/cterrassa/suficiencia-sin/HEAD?urlpath=lab/tree/Tutorial_Suficiencia_SIN.ipynb)

- **El notebook:** [`Tutorial_Suficiencia_SIN.ipynb`](Tutorial_Suficiencia_SIN.ipynb). Mide cuánta energía firme le falta al SIN de Colombia entre 2026 y 2030 y busca, con un modelo de optimización con variables enteras, la expansión de menor costo que la cubre. Corre de principio a fin con Python e internet, sin archivos adicionales; con los botones de Colab o de Binder, en el navegador. Colab pide una cuenta de Google que lo tenga habilitado; Binder no pide cuenta, pero la primera vez puede tardar varios minutos en arrancar.
- **La versión web, ya ejecutada:** [cterrassa.github.io/suficiencia-sin](https://cterrassa.github.io/suficiencia-sin/), con las figuras interactivas y animadas.
- **La app interactiva:** [cterrassa.github.io/suficiencia-sin/app](https://cterrassa.github.io/suficiencia-sin/app/). Es la app de Shiny for Python del proyecto, exportada con shinylive a la carpeta `app/`: corre Python en el navegador, sin servidor y sin instalar nada. Mueve los supuestos del modelo de expansión (escenario, retrasos, tope solar, diésel y penalidad de la energía no cubierta) y muestra la brecha, el portafolio, el costo y si conviene construir o racionar en 2029-2030. Usa 72 casos que el modelo ya resolvió, con las mismas cifras del notebook.
- **El entorno:** `requirements.txt` (los siete paquetes, con sus versiones) y `runtime.txt` (Python 3.14). Binder arma con ellos el entorno; también sirven para instalar el tutorial en local.
- **Los datos:** la carpeta `datos_tutorial/` trae las tablas que el tutorial descarga. Las armó el pipeline del proyecto a partir de fuentes públicas: XM, la UPME, NOAA y el DANE. `manifiesto.csv` da los bytes y la huella SHA-256 de cada archivo, y el tutorial verifica cada descarga contra ella.

Los archivos de `datos_tutorial/` no se deben modificar: si uno cambia, su huella deja de coincidir y el tutorial se detiene.
