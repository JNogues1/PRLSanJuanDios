# PRL · Hospital San Juan de Dios de Burgos

Visor web de las zonas de obra señalizadas cada día sobre los planos de planta del hospital.
Cualquier persona que escanee el código QR de la entrada ve las zonas vigentes **para ese día**:
zona en intervención, riesgos, acopios, casetas y aseos, entradas, equipos de protección,
paso de vehículos y recorridos peatonales.

## Logotipo de empresa

La cabecera muestra el logotipo situado en `assets/logo.png` (súbelo tú con el
archivo oficial de marca de Ferrovial). Si el fichero no existe, se muestra el
nombre «FERROVIAL» en texto como respaldo.

## Estructura

```
index.html            → la aplicación completa (visor + modo administrador)
planos/               → imágenes de los planos (6 plantas)
data/zonas.json       → zonas publicadas (lo que ven los visitantes)
data/estancias.json   → catálogo de estancias con coordenadas (extraído de los PDF vectoriales)
```

## Plantas incluidas

| Pestaña | Plano | Origen | Resolución |
|---|---|---|---|
| SÓT | Planta Sótano (−3,60) | estado inicial C.01f | baja (1316 px) |
| PB | Planta Baja reformada (±0,00) | estado reformado C-01 | alta (5458 px) |
| P1 | Planta Primera reformada (+3,42) | estado reformado C-02 | alta (5687 px) |
| P2 | Planta Segunda reformada (+7,22) | estado reformado C-03 | alta (5687 px) |
| P3 | Planta Tercera reformada (+11,02) | estado reformado C-04 | alta (5687 px) |
| BC | Bajo Cubierta (+14,06) | C.06f (vectorial) | alta (5600 px) |
| CUB | Cubiertas (+17,03) | estado reformado C-05 | alta (5687 px) |

Al dibujar una zona nueva en PB, P1, P2 o P3 la app detecta automáticamente las
**estancias contenidas** (81 + 36 + 88 + 15 estancias, con nombre y superficie
extraídos de la capa de texto de los planos vectoriales) y rellena el nombre de
la zona con ellas.

## Puesta en marcha en GitHub Pages

1. Crea un repositorio en GitHub (por ejemplo `obra-hsjd`) y sube todo el contenido de esta carpeta.
2. En el repositorio: **Settings → Pages → Source: Deploy from a branch → main / (root)**.
3. En un minuto la app estará en `https://TU-USUARIO.github.io/obra-hsjd/`.

## Modos de uso

**Modo usuario** (quien escanea el QR, móvil/tablet/PC): pestañas de plantas con zoom,
tocar una zona despliega su ficha (categoría, indicaciones, vigencia), botón **3D** con
las plantas apiladas en isométrica (tocar una planta la abre; arrastrar gira la vista)
y botón **☰ Listado** con la tabla imprimible de zonas.

**Modo administrador**: disponible desde **ordenadores y tablets** (iPad incluido).
En teléfonos la edición no existe, ni siquiera con `?admin=1` — la pantalla es
demasiado pequeña para dibujar polígonos con precisión.

## Listado imprimible (ambos modos)

El botón **☰** de la cabecera abre la tabla de zonas con columnas Planta, Categoría,
Zona, Desde y Hasta. Cada columna se ordena de forma independiente pulsando su
cabecera (segunda pulsación invierte el orden). Se puede filtrar por planta, por
categoría y por vigencia en la fecha consultada, e imprimir con formato limpio.

## Uso diario (administrador)

En ordenadores y tablets el engranaje ⚙ de la cabecera está siempre visible
(la edición queda protegida por el PIN y, para publicar, por el token de GitHub).
En teléfonos no existe. Añadir `?admin=1` a la URL simplemente abre el PIN
automáticamente. El PIN muestra la versión de la app (p. ej. «v10»): si tras
actualizar el repositorio no ves la versión nueva, fuerza la recarga del
navegador (⌘⇧R en Safari/Chrome de Mac, Ctrl+F5 en Windows).

1. Abre la URL de administración e introduce el PIN (4–8 dígitos). **PIN inicial: 1234 — cámbialo antes de publicar.**
2. Pulsa **✎ Nueva zona**, marca los vértices del recinto y cierra con **✓**.
3. Asigna categoría, nombre, indicaciones y fechas de vigencia (vacío = indefinido).
   En el mismo formulario, marca las **recomendaciones de seguridad y salud** aplicables
   de un catálogo de 19 genéricas (EPIs obligatorios, prohibiciones, sectorización
   antipolvo, higiene hospitalaria...) o añade recomendaciones personalizadas, que
   quedan guardadas y reutilizables en futuras zonas. Las recomendaciones marcadas
   aparecen en la ficha de la zona en modo usuario y en el listado imprimible.
4. **➕ Categoría** crea categorías nuevas (nombre, icono emoji y color) que se suman
   a las 8 de serie en la leyenda y el formulario.
5. **🗺 Planos** permite añadir plantas nuevas o reemplazar el plano de una existente.
   Estas operaciones publican directamente en GitHub, así que requieren el token
   configurado. Al reemplazar un plano con el mismo encuadre, las zonas se conservan.
6. Pulsa **☁ Publicar** para subir los cambios a `data/zonas.json` del repositorio.
   Necesitas un *fine-grained personal access token* de GitHub con permiso
   **Contents: Read & Write** solo sobre este repositorio
   (GitHub → Settings → Developer settings → Fine-grained tokens).
   El token se guarda únicamente en tu dispositivo.
7. Pulsa **▦ QR** para generar e imprimir el código de acceso de la entrada de la obra.

Mientras no publiques, los cambios quedan guardados como borrador en tu dispositivo
(localStorage). **⭳ Exportar / ⭱ Importar** permiten hacer copias de seguridad o mover
los datos entre dispositivos, o subir `zonas.json` a mano al repositorio si lo prefieres.

## Cambiar el PIN

El PIN se guarda como hash SHA-256 en `data/zonas.json` (campo `pinHash`). Para cambiarlo,
genera el hash del nuevo PIN (en la consola del navegador):

```js
crypto.subtle.digest("SHA-256", new TextEncoder().encode("TU_PIN"))
  .then(b => console.log([...new Uint8Array(b)].map(x=>x.toString(16).padStart(2,"0")).join("")))
```

y sustituye el valor de `pinHash`. Usa un PIN de 6–8 dígitos, no el 1234 inicial.

## Modelo de seguridad (léelo una vez)

Al ser una web estática sin servidor, es importante entender qué protege cada capa:

- **Lo que ven los visitantes** (`data/zonas.json` publicado) solo puede modificarlo
  quien tenga permiso de escritura en el repositorio de GitHub — es decir, el token
  del administrador. Esta es la protección real y es sólida.
- El **PIN** oculta la interfaz de edición a usuarios casuales. Si alguien muy
  motivado llegara a entrar en modo administración, sus cambios quedarían **solo en
  su propio dispositivo** (localStorage): no puede publicarlos sin el token, así que
  nunca afectarían a lo que ven los demás.
- El **token de GitHub** se guarda únicamente en el navegador del administrador y
  nunca se sube al repositorio. Créalo *fine-grained*, limitado a este repositorio
  y solo con permiso de Contents.

## Sustituir los planos por versiones de más resolución

Las zonas se guardan en coordenadas **relativas** (0–1) a la imagen, así que puedes
reemplazar los ficheros de `planos/` por exportaciones a mayor resolución de los mismos
planos (misma hoja y encuadre) sin perder nada de lo dibujado. Recomendado: exportar
los PDF vectoriales originales a JPEG/PNG de 4000–6000 px de ancho.

## Categorías disponibles

| Categoría | Color |
|---|---|
| 🚧 Zona en intervención | rojo |
| ⚠️ Zona de riesgo | amarillo |
| 📦 Acopio de materiales | marrón |
| 🚻 Casetas y aseos | azul |
| 🚪 Entrada al inmueble | verde |
| 🦺 Equipos de protección | verde azulado |
| 🚚 Paso de vehículos | violeta |
| 🚶 Recorrido peatonal | verde claro |

Se pueden ampliar editando el objeto `CATS` al principio del script de `index.html`.
