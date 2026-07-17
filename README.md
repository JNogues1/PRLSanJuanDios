[README.md](https://github.com/user-attachments/files/30133836/README.md)
# Plan de Obra · Hospital San Juan de Dios de Burgos

Visor web de las zonas de obra señalizadas cada día sobre los planos de planta del hospital.
Cualquier persona que escanee el código QR de la entrada ve las zonas vigentes **para ese día**:
zona en intervención, riesgos, acopios, casetas y aseos, entradas, equipos de protección,
paso de vehículos y recorridos peatonales.

## Estructura

```
index.html          → la aplicación completa (visor + modo administrador)
planos/             → imágenes de los planos (sótano, baja, primera, segunda, tercera)
data/zonas.json     → zonas publicadas (lo que ven los visitantes)
```

## Puesta en marcha en GitHub Pages

1. Crea un repositorio en GitHub (por ejemplo `obra-hsjd`) y sube todo el contenido de esta carpeta.
2. En el repositorio: **Settings → Pages → Source: Deploy from a branch → main / (root)**.
3. En un minuto la app estará en `https://TU-USUARIO.github.io/obra-hsjd/`.

## Uso diario (administrador)

1. Abre la app y pulsa el engranaje ⚙ (o añade `?admin=1` a la URL). **PIN por defecto: 1234**.
2. Pulsa **✎ Nueva zona**, toca el plano para marcar los vértices del recinto y cierra con **✓**.
3. Asigna categoría, nombre, indicaciones y fechas de vigencia (vacío = indefinido).
4. Pulsa **☁ Publicar** para subir los cambios a `data/zonas.json` del repositorio.
   Necesitas un *fine-grained personal access token* de GitHub con permiso
   **Contents: Read & Write** solo sobre este repositorio
   (GitHub → Settings → Developer settings → Fine-grained tokens).
   El token se guarda únicamente en tu dispositivo.
5. Pulsa **▦ QR** para generar e imprimir el código de acceso de la entrada de la obra.

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

y sustituye el valor de `pinHash`. ⚠️ Al ser una web estática, el PIN solo disuade al
usuario casual: la edición real está protegida por el token de GitHub, que nunca sale
de tu dispositivo.

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
