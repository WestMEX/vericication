# VAULT AUTH — Sistema de Autenticación NFC para Gorras

Sistema anti-falsificación basado en chips NFC con firma criptográfica HMAC-SHA256.

---

## 🗂 Estructura del repositorio

```
tu-repo/
├── index.html      ← Página de verificación (GitHub Pages)
├── db.json         ← Base de datos de productos
├── nfc_writer.py   ← Script para programar chips NFC
└── README.md
```

---

## 🚀 Deploy en GitHub Pages (5 minutos)

1. Crea un repositorio en GitHub (puede ser privado o público)
2. Sube los 3 archivos: `index.html`, `db.json`, `README.md`
3. Ve a **Settings → Pages → Source → main branch / root**
4. Tu página estará en: `https://TU_USUARIO.github.io/TU_REPO/`

---

## ⚙️ Configuración

### En `index.html`, edita la línea:
```js
DB_URL: 'https://raw.githubusercontent.com/TU_USUARIO/TU_REPO/main/db.json',
```

### En `nfc_writer.py`, edita:
```python
BASE_URL    = "https://TU_USUARIO.github.io/TU_REPO/"
SECRET_SALT = "CAMBIA_ESTO_POR_UNA_CADENA_SECRETA_LARGA"
```

> ⚠️ **IMPORTANTE**: el `SECRET_SALT` del Python y el del JavaScript deben generar la misma firma. El JS usa `uid + "_vault_secret"` — ajusta ambos para que coincidan.

---

## 📡 Flujo de autenticación

```
Chip NFC → URL firmada → Página web → Verifica HMAC → Muestra resultado
```

1. **Chip NFC** contiene: `https://tu-repo.github.io/?uid=GORRA-001&sig=a3f8...`
2. **Usuario escanea** con su celular → abre el navegador automáticamente
3. **Página web** lee `uid` y `sig` de la URL
4. **Consulta** `db.json` en GitHub para verificar que el UID existe
5. **Valida** la firma HMAC-SHA256 criptográfica
6. **Muestra** ✅ AUTÉNTICO o ❌ FALSO / CLONADO

---

## 🔐 Por qué es inclonalbe

| Intento de ataque | Por qué falla |
|---|---|
| Copiar la URL del chip | Sin firma válida no pasa la verificación |
| Clonar el chip NFC físico | Los NTAG tienen UID de fábrica grabado en silicio (no modificable) |
| Re-usar la firma de otro chip | La firma está ligada al UID único de cada chip |
| Duplicar `db.json` | El SECRET_SALT solo existe en tu servidor privado |
| Escanear desde 2 lugares | El sistema detecta doble escaneo y genera alerta |

---

## 🛠 Programar chips NFC

### Opción A — Script Python (requiere lector NFC USB)
```bash
pip install nfcpy qrcode[pil]
python nfc_writer.py
```

### Opción B — App NFC Tools (Android/iOS) — Sin hardware extra
1. Ejecuta `python nfc_writer.py` → opción 4 para obtener la URL del producto
2. Abre **NFC Tools** en tu celular
3. WRITE → Add a record → URL → pega la URL
4. Escribe en el chip
5. Activa **Read-only** para bloquearlo

### Chips recomendados
- **NTAG213** — 144 bytes, suficiente para la URL (~100 chars)
- **NTAG215** — 504 bytes, más espacio
- NXP ICODE SLI — Para ropa (resistente a lavado)

---

## 🧪 Probar sin chip físico

Abre la consola del navegador en tu GitHub Pages y escribe:
```js
simulateScan('GORRA-DEMO001')
```

Esto simula un escaneo legítimo completo.

---

## 📊 Base de datos (`db.json`)

Agrega productos manualmente o con el script:

```json
{
  "GORRA-001": {
    "model": "Snapback Classic",
    "batch": "LOTE-2025-A",
    "color": "Negro / Lima",
    "size": "Talla única",
    "created": "2025-01",
    "secret": "GORRA-001_vault_secret",
    "scans": []
  }
}
```

> Para producción: reemplaza GitHub raw con una API (Vercel, Railway, Supabase) que permita escritura real de escaneos.

---

## 🔄 Limitación de GitHub Pages

GitHub Pages es **read-only** — los escaneos se guardan en `localStorage` del navegador del usuario. Para persistir escaneos en la nube necesitas un backend (Supabase free tier es ideal).
