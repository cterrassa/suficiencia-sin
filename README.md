# Suficiencia energética del SIN 2027-2030 y expansión de mínimo costo

Tutorial del Trabajo Final de Taller de Programación en Python (MECA 4110, Universidad de los Andes, 2026-2). Autores: Carlos Andrés Terrassa Vallejo, Esteban Darío Herrera León y Jair Armando Castañeda Rodríguez.

[![Abrir en Colab](https://colab.research.google.com/assets/colab-badge.svg)](https://colab.research.google.com/github/cterrassa/suficiencia-sin/blob/main/Tutorial_Suficiencia_SIN.ipynb)

- **El notebook:** [`Tutorial_Suficiencia_SIN.ipynb`](Tutorial_Suficiencia_SIN.ipynb). Mide cuánta energía firme le falta al SIN de Colombia entre 2026 y 2030 y busca, con un modelo de optimización con variables enteras, la expansión de menor costo que la cubre. Corre de principio a fin con Python e internet, sin archivos adicionales; con el botón de Colab, en el navegador.
- **La versión web, ya ejecutada:** [cterrassa.github.io/suficiencia-sin](https://cterrassa.github.io/suficiencia-sin/), con las figuras interactivas.
- **Los datos:** la carpeta `datos_tutorial/` trae las tablas que el tutorial descarga. Las armó el pipeline del proyecto a partir de fuentes públicas: XM, la UPME, NOAA y el DANE. `manifiesto.csv` da los bytes y la huella SHA-256 de cada archivo, y el tutorial verifica cada descarga contra ella.

Los archivos de `datos_tutorial/` no se deben modificar: si uno cambia, su huella deja de coincidir y el tutorial se detiene.
