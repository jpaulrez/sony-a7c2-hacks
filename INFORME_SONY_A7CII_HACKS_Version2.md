# Exploración de Posibles Hacks en la Sony A7C II (Firmware 2.00)

Tras el lanzamiento de la Sony FX2 y la polémica de que no se pueda grabar en 4K a 60fps en Full Frame, me di a la tarea de investigar cómo desbloquear esa función a través de la gestión de ajustes desde el menú de la cámara. Noté que la cámara genera archivos `.DAT`, similares a los que la Sony F5 usaba ("ALLFILE.TXT" o "001.ALL") para guardar sus ajustes. En su tiempo, no era posible grabar a 4K, pero un usuario logró modificar el archivo `001.ALL` y desbloquear esa función (de 150=2 a 150=4).

---

## Objetivo

Investigar si es posible modificar el archivo de configuración `.DAT` generado por la Sony A7C II para forzar funciones no disponibles directamente, como:

- Grabación en 4K 60p Full Frame
- Habilitar LUTs personalizados en modos con framerate alto (ej. 120fps)

---

## Contexto

La Sony A7C II permite exportar configuraciones internas a un archivo `.DAT` desde el menú:

`Menú > Ajustes > Gestión de ajustes > Guardar en tarjeta SD`

Estos archivos contienen información de resolución, framerate, perfiles de color, modo de sensor (FF/Super35), y posiblemente la activación de LUTs.

---

## Metodología

Se generaron varios archivos `.DAT` desde una A7C II con diferentes configuraciones:

- 4K 24p Full Frame (S-Log3)
- 4K 60p Super35 (S-Log3)
- 4K 30p Full Frame (S-Log3)
- 1080p 60p con LUT cargado (PPLUT1)
- 1080p 120p sin LUT

Se compararon los archivos byte a byte:

- Se identificaron patrones de cambio entre modos de resolución y framerate.
- Se generaron archivos modificados manualmente replicando estructuras similares a la F5/F55 (conocido hack).
- Los archivos modificados fueron cargados a la cámara para probar si eran aceptados.

---

## Resultados

**Cambios clave identificados:**

| Dirección  | Significado probable              | Valor 24p | 30p  | 60p  | 120p |
|------------|----------------------------------|-----------|------|------|------|
| 5303       | Modo de grabación principal (framerate) | 0x04      | 0x02 | 0x00 | -    |
| 5312       | Framerate extendido (activa 120fps)     | -         | -    | -    | 0x05 |
| 15921      | Modo de sensor (FF/S35)                 | 0x8E      | 0x8E | 0x40 | 0x8E |
| 15933-34   | Modo de codec o binario de grabación    | 0F 01     | 0F 01| F0 00| -    |
| 14982      | Activación de LUT personalizado         | 0x01      | 0x01 | 0x01 | 0x0C (con LUT) |

**Observaciones:**

- El archivo `.DAT` está en binario, con una estructura similar a bloques tipo TLV (Tag, Length, Value).
- Al modificar cualquier byte, la cámara rechaza cargar el archivo.
- Esto sugiere que existe un checksum o firma binaria interna.
- No se ha podido determinar el algoritmo usado (CRC32, MD5, SHA1 u otro).

---

## Hipótesis

Sony implementa una validación de integridad en los archivos `.DAT` para evitar manipulaciones, similar a lo que hacía con la F5 y F55. La posición del hash aún no ha sido localizada ni se ha identificado el tipo de codificación.

---

## Petición a la comunidad

Si alguien tiene experiencia en:

- Reingeniería inversa de archivos binarios
- Análisis de checksums en firmwares Sony
- Desarrollo de herramientas de edición de presets

**Se agradecería cualquier colaboración para:**

- Identificar el algoritmo de validación.
- Crear un generador seguro de archivos `.DAT` modificados.
- Posiblemente habilitar grabación 4K60p FF o LUTs en modos restringidos.

---

## Datos de referencia

- **Cámara:** Sony A7C II (ILCE-7CM2)
- **Firmware:** Ver. 2.00
- **Archivos utilizados:** CAMSET01.DAT a CAMSET06.DAT
- **LUT utilizado:** PPLUT1 (CINEHACK, .cube)

---

## Crédito

Esta investigación fue realizada por **Jhon Paul Perez Pretell** [@jpaul.view], cineasta y director creativo audiovisual en Lima, Perú.

Contacto para colaborar: [hola@alquimiaview.com](mailto:hola@alquimiaview.com)