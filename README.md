# ReciclaCAT — Escaparate web

Sitio estático que muestra el stock disponible. Las consultas se canalizan por WhatsApp; **no es una tienda online**.

- 📦 Stock: descargado automáticamente del FTP cada 6 h y commiteado al repo.
- 🌐 Hosting: GitHub Pages.
- 💬 Contacto: enlace WhatsApp con mensaje predefinido por producto.

## Estructura

```
.
├─ index.html              # página principal
├─ assets/
│  ├─ styles.css           # estilos
│  └─ app.js               # carga del CSV, filtros, modal, CTA WhatsApp
├─ data/
│  └─ stock.csv            # CSV (lo sobrescribe el GitHub Action)
├─ scripts/
│  └─ sync-stock.js        # descarga el CSV del FTP
├─ .github/workflows/
│  └─ sync-stock.yml       # cron cada 6 h
├─ package.json
└─ README.md
```

## Configuración inicial

### 1. Personalizar el sitio

Edita `assets/app.js` y cambia:

```js
const CONFIG = {
  csvUrl: "data/stock.csv",
  whatsappNumber: "34600000000",   // ← TU número, formato internacional sin signos
  columns: { ... },                // ← si tu CSV usa otros nombres de columna
  currency: "€",
};
```

### 2. Configurar los Secrets del FTP (en GitHub)

Ve a `Settings → Secrets and variables → Actions → New repository secret` y crea estos cuatro:

| Secret              | Ejemplo                       | Obligatorio |
| ------------------- | ----------------------------- | ----------- |
| `FTP_HOST`          | `ftp.midominio.com`           | sí          |
| `FTP_USER`          | `usuario`                     | sí          |
| `FTP_PASSWORD`      | `contraseña`                  | sí          |
| `FTP_REMOTE_PATH`   | `/exports/stock.csv`          | sí          |
| `FTP_SECURE`        | `true` (solo si usas FTPS)    | opcional    |

> ⚠️ Nunca pegues estas credenciales en el código ni en commits. Solo aquí.

### 3. Activar GitHub Pages

`Settings → Pages → Source: Deploy from a branch → Branch: main / root`.

A los pocos minutos la web estará en `https://<tu-usuario>.github.io/<nombre-repo>/`.

### 4. Probar el sync manual

`Actions → Sync stock CSV from FTP → Run workflow`.

Si algo falla, en el log del Action verás el error de conexión FTP.

## CSV esperado

Por defecto el código espera estas columnas (cabeceras exactas):

```
sku,nombre,categoria,descripcion,precio,unidad,stock,foto_url,destacado
```

- `precio`: número (acepta coma o punto decimal).
- `stock`: entero. Si es 0 → marca "Agotado".
- `foto_url`: URL absoluta a la imagen.
- `destacado`: `1` / `0` (también acepta `true`/`false`/`sí`).

Si tu CSV real usa otros nombres, **no cambies el CSV**: cambia el mapeo en
`CONFIG.columns` dentro de `assets/app.js`.

## Desarrollo local

No hace falta ningún build. Para verlo en local con Python:

```bash
python3 -m http.server 8080
# abre http://localhost:8080
```

(Hay que servirlo con un servidor; abrir el `index.html` con doble clic
no funciona porque `fetch()` requiere `http://`).

## Notas

- El sitio es **informativo**. No procesa pagos ni recoge datos personales.
- Los precios mostrados son orientativos; las condiciones se confirman por WhatsApp.
- Las imágenes se cargan desde URLs externas; si una imagen rompe se muestra atenuada.
