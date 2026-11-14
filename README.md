# Suficiencia energética del SIN 2027-2030 · datos del tutorial

Paquete de datos del tutorial `Tutorial_Suficiencia_SIN.ipynb`, del Trabajo Final de Taller de Programación en Python (MECA 4110, Universidad de los Andes, 2026-2). Autores: Carlos Andrés Terrassa Vallejo, Esteban Darío Herrera León y Jair Armando Castañeda Rodríguez.

La carpeta `datos_tutorial/` trae las tablas que el tutorial descarga. Las armó el pipeline del proyecto a partir de fuentes públicas: XM, la UPME, NOAA y el DANE. `manifiesto.csv` da los bytes y la huella SHA-256 de cada archivo, y el tutorial verifica cada descarga contra ella.

Los archivos de `datos_tutorial/` no se deben modificar: si uno cambia, su huella deja de coincidir y el tutorial se detiene.
