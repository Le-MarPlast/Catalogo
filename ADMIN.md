# ABM de productos

El panel de administracion operativo vive en Google Apps Script. Esto evita problemas de CORS entre GitHub Pages y Apps Script, y permite usar `google.script.run` de forma directa.

Los archivos fuente estan en:

```text
apps-script/Code.gs
apps-script/Index.html
apps-script/Styles.html
apps-script/Javascript.html
```

La carpeta `admin/` queda como prototipo estatico de interfaz, pero la version recomendada para operar es la servida por Apps Script.

La URL final del admin sera la URL del Web App de Apps Script, algo parecido a:

```text
https://script.google.com/macros/s/.../exec
```

## Arquitectura

```text
admin/index.html
  -> Google Apps Script Web App
      -> Google Sheets: Le-Mar Plast Productos
      -> GitHub API: Le-MarPlast/Catalogo
```

El token de GitHub y las credenciales del admin no van en el HTML publico. Se guardan en Script Properties de Google Apps Script.

## Google Apps Script

1. Abrir la planilla de productos.
2. Ir a `Extensiones > Apps Script`.
3. Crear o reemplazar estos archivos:

```text
Code.gs
Index.html
Styles.html
Javascript.html
```

4. Copiar el contenido desde la carpeta `apps-script/`.
5. Ir a `Configuracion del proyecto > Propiedades de secuencia de comandos`.
6. Agregar estas properties:

```text
ADMIN_USER=usuario_admin
ADMIN_PASSWORD=password_admin
GITHUB_TOKEN=token_de_github
PRODUCT_SHEET_NAME=Le-Mar Plast Productos
ORDERS_SHEET_NAME=Pedidos
```

`PRODUCT_SHEET_NAME` debe coincidir con el nombre de la solapa de la planilla, no necesariamente con el nombre del archivo.
`ORDERS_SHEET_NAME` es opcional. Si no existe, se usa `Pedidos`.

## Desplegar Apps Script

1. En Apps Script, tocar `Implementar > Nueva implementacion`.
2. Tipo: `Aplicacion web`.
3. Ejecutar como: `Yo`.
4. Quien tiene acceso: `Cualquier usuario`.
5. Implementar.
6. Copiar la URL terminada en `/exec`.
7. Esa URL es el panel de administracion.

El login real se valida en Apps Script con `ADMIN_USER` y `ADMIN_PASSWORD`.

## Token de GitHub

El token debe permitir escribir contenido en el repo:

```text
Le-MarPlast/Catalogo
```

Recomendacion: usar un Fine-grained personal access token con:

- Repository access: solo `Le-MarPlast/Catalogo`.
- Contents: `Read and write`.

El token se guarda como `GITHUB_TOKEN` en Script Properties.

## Productos

El panel escribe en la hoja:

```text
Le-Mar Plast Productos
```

Columnas esperadas:

```text
id,nombre,categoria,descripcion,precio,stock,imagen,activo,destacado
```

Los IDs se generan automaticamente tomando el mayor ID numerico y sumando 1.

La columna `destacado` define si el producto aparece en el carrusel del catalogo. Valores esperados:

```text
SI
NO
```

## Pedidos

El catalogo puede registrar una copia del pedido en una hoja de la planilla cuando el cliente toca `Enviar pedido por WhatsApp`.

Para activarlo:

1. Copiar la version actualizada de `apps-script/Code.gs` al Apps Script.
2. Guardar.
3. Implementar una nueva version del Web App.
4. Copiar la URL terminada en `/exec`.
5. Pegarla en `app.js`:

```js
ordersWebAppUrl: "https://script.google.com/macros/s/.../exec",
```

Si la hoja `Pedidos` no existe, Apps Script la crea automaticamente con estas columnas:

```text
fecha,pedido_id,nombre,entrega,items,total,estado,origen
```

El campo `items` guarda un JSON con los productos, cantidades, precios y subtotales del pedido.

Nota: el registro de pedidos es publico porque lo ejecutan clientes desde el catalogo. No requiere login.

### Estados de pedidos

El panel de Apps Script tiene tres secciones:

```text
ABM
Pedidos
Entregas
```

Los pedidos nuevos se guardan con estado:

```text
PENDIENTE
```

En `Pedidos` se muestran solo los pedidos `PENDIENTE`. Desde ahi se pueden pasar a:

```text
ENTREGAR
CUMPLIDO
```

En `Entregas` se muestran solo los pedidos `ENTREGAR`. Desde ahi se pueden pasar a:

```text
CUMPLIDO
```

Los pedidos `CUMPLIDO` quedan guardados en la hoja, pero no aparecen en las vistas operativas.

## Imagenes

Cuando se sube una imagen, Apps Script la guarda en GitHub en:

```text
assets/Productos/producto-ID.ext
```

Y en la planilla guarda la ruta relativa:

```text
assets/Productos/producto-ID.ext
```

GitHub Pages puede tardar algunos minutos en reflejar una imagen recien subida.

## Seguridad

Este login es adecuado para un panel simple de administracion de catalogo. No es un sistema de autenticacion empresarial.

Puntos importantes:

- No guardar usuario, password ni token en `admin/admin.js`.
- Usar password fuerte.
- Si se filtra el token, revocarlo y crear uno nuevo.
- Si se cambia el Apps Script, crear una nueva version de implementacion.
