---
name: cuentica
description: >
  Accede a los datos de facturación, gastos, clientes y proveedores del usuario
  a través de la API de Cuéntica. Úsala siempre que el usuario pregunte sobre
  sus facturas, ingresos, gastos, cobros pendientes, clientes, proveedores,
  resúmenes trimestrales/anuales, borradores de facturas o cualquier dato
  contable o fiscal. Dispara también si el usuario menciona Cuéntica, dice
  "mis facturas", "lo que me deben", "cuánto he facturado", "gastos del
  trimestre", "factura pendiente" o frases similares relacionadas con su
  contabilidad.
---

# Skill: Cuéntica API

> Skill para acceder a la API de [Cuéntica](https://cuentica.com), el software
> de facturación y contabilidad para autónomos y pequeñas empresas en España.
> Desarrollada por [UNGRYNERD](https://ungrynerd.com).

---

## Configuración inicial (primera vez)

Antes de usar esta skill, el usuario debe proporcionar su token de API de Cuéntica.

**Cómo obtenerlo:**
1. Inicia sesión en [app.cuentica.com](https://app.cuentica.com)
2. Ve a **Configuración → API → Generar token**
3. Copia el token y díselo al asistente para que lo guarde en memoria

**Una vez obtenido el token**, pide al usuario que lo comparta y guárdalo en
memoria con la clave `Cuentica API token`. A partir de ese momento no será
necesario volver a pedirlo.

Si al intentar una consulta no hay token en memoria, interrumpe y pide al
usuario que lo proporcione antes de continuar.

---

## Autenticación

Recupera el token desde la memoria del usuario (clave: `Cuentica API token`).
Inclúyelo siempre en el header `X-AUTH-TOKEN` de cada petición.

```bash
curl -s -H "X-AUTH-TOKEN: <TOKEN>" https://api.cuentica.com/<endpoint>
```

**Nunca muestres el token completo en la respuesta.** Si necesitas confirmarlo,
muestra solo los últimos 6 caracteres.

---

## Flujo general

1. Recupera el token desde la memoria del usuario.
2. Identifica qué endpoint(s) necesitas según la consulta (ver tabla abajo).
3. **Aplica siempre filtros de fecha u otros parámetros antes de paginar** para
   reducir el volumen de datos. Cuanto más acotada sea la consulta, mejor.
4. Construye la llamada curl usando **`page_size=25`** por defecto.
   Solo aumenta a 50 si la consulta lo justifica claramente (ej. resumen anual).
   Nunca uses page_size > 50.
5. Interpreta el JSON devuelto y presenta los datos de forma clara y resumida.
6. **Paginación obligatoria para consultas de "pendientes"**: cuando la consulta
   requiere datos completos (facturas pendientes, totales anuales, etc.), itera
   página a página hasta que la respuesta devuelva menos registros que page_size
   (señal de última página). Acumula solo los campos relevantes entre páginas,
   no el JSON completo. Detente si llegas a 20 páginas (500 registros con page_size=25).

---

## Endpoints principales

| Recurso | Endpoint | Método |
|---|---|---|
| Datos empresa | `/company` | GET |
| Facturas emitidas | `/invoice` | GET |
| Factura concreta | `/invoice/:id` | GET |
| Crear factura | `/invoice` | POST |
| Ingresos | `/income` | GET |
| Gastos | `/expense` | GET |
| Clientes | `/customer` | GET |
| Proveedores | `/provider` | GET |

---

## Casos de uso frecuentes

### Resumen de facturación por período

```bash
curl -s -H "X-AUTH-TOKEN: <TOKEN>" \
  "https://api.cuentica.com/invoice?initial_date=YYYY-01-01&end_date=YYYY-03-31&issued=true&page_size=25"
```

Suma `amount_details.total_invoice` de cada factura para obtener el total.
Agrupa por `customer.tradename` si se pide desglose por cliente.

### Facturas pendientes de cobro

Esta consulta **requiere iterar todas las páginas** porque las facturas pendientes
pueden estar distribuidas en múltiples páginas. Itera de page=1 en adelante
hasta recibir una página con menos registros que page_size (última página).

```bash
curl -s -H "X-AUTH-TOKEN: <TOKEN>" \
  "https://api.cuentica.com/invoice?issued=true&page_size=25&page=1"
```

De cada página, extrae solo las facturas que cumplan **todas** estas condiciones:
- `register_info.status_description != "voided"` → excluye facturas anuladas
- Al menos un `charge` con `paid: false`
- `amount_details.total_left > 0`

Los campos clave son:
- `invoice_number` + `invoice_serie` → número de factura
- `customer.tradename` → cliente
- `amount_details.total_invoice` → importe total
- `amount_details.total_left` → importe pendiente
- `date` → fecha

Para el total pendiente global suma `total_left` de todas las páginas.
Muestra: número de factura, cliente, importe total, fecha, importe pendiente.

### Consultas por cliente

```bash
# Primero busca el ID del cliente
curl -s -H "X-AUTH-TOKEN: <TOKEN>" \
  "https://api.cuentica.com/customer?q=<nombre_cliente>"

# Luego filtra facturas por ese ID
curl -s -H "X-AUTH-TOKEN: <TOKEN>" \
  "https://api.cuentica.com/invoice?customer=<id>&page_size=25"
```

### Gastos por período

```bash
curl -s -H "X-AUTH-TOKEN: <TOKEN>" \
  "https://api.cuentica.com/expense?initial_date=YYYY-01-01&end_date=YYYY-03-31&page_size=25"
```

Agrupa por `expense_type` si se pide desglose por categoría.

### Borradores de facturas

```bash
# Filtra siempre por el año vigente para excluir recurrentes programadas en años futuros
curl -s -H "X-AUTH-TOKEN: <TOKEN>" \
  "https://api.cuentica.com/invoice?issued=false&initial_date=YYYY-01-01&end_date=YYYY-12-31&page_size=25"
```

Sustituye `YYYY` por el año en curso. Los borradores con fecha futura corresponden
a facturas recurrentes preprogramadas por Cuéntica — no son pendientes reales.

### Resumen trimestral rápido (IVA)

Trimestres españoles:
- T1: 01-01 / 03-31
- T2: 04-01 / 06-30
- T3: 07-01 / 09-30
- T4: 10-01 / 12-31

Combina `/invoice` (IVA repercutido) y `/expense` (IVA soportado deducible)
para calcular el resultado neto de IVA del trimestre.

---

## Presentación de resultados

- **Importes**: siempre en € con 2 decimales.
- **Fechas**: formato dd/mm/yyyy en las respuestas.
- **Tablas**: usa markdown cuando haya más de 3 registros.
- **Resúmenes**: incluye siempre total, número de registros y período cubierto.
- **Errores API**: si devuelve 403, el token puede haber caducado — pide uno nuevo.
  Si devuelve 429, espera y reintenta (límite: 600 req/5 min).

---

## Precauciones

- Esta skill es de **solo lectura** por defecto. Para crear o modificar datos
  (POST/PUT/DELETE), pide confirmación explícita al usuario antes de ejecutar.
- No incluyas datos fiscales sensibles (NIF, cuentas bancarias) en resúmenes
  que no los requieran.
- **Facturas anuladas**: la API devuelve facturas anuladas mezcladas con las
  activas. Identifícalas por `register_info.status_description == "voided"` y
  exclúyelas siempre de cualquier cálculo de pendientes, totales o resúmenes.
