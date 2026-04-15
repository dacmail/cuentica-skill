# Cuéntica Skill para Claude

Skill para consultar tu contabilidad de [Cuéntica](https://cuentica.com) directamente desde Claude (u otro LLM compatible con skills).

Desarrollada por [UNGRYNERD](https://ungrynerd.com).

---

## Qué puedes hacer

- Listar facturas emitidas y pendientes de cobro
- Ver borradores del año en curso
- Consultar gastos por período o categoría
- Buscar facturas por cliente
- Obtener resúmenes trimestrales de IVA (modelo 303)
- Consultar datos de clientes y proveedores

---

## Instalación

### En Claude.ai (con soporte de Skills)

1. Descarga el archivo `SKILL.md` de este repositorio.
2. En Claude.ai, ve a **Settings → Skills → Install skill**.
3. Sube el archivo `SKILL.md`.
4. La skill quedará activa en tus conversaciones.

### En Claude Desktop / Claude Code

Copia `SKILL.md` a tu carpeta de skills de usuario:

```bash
# Claude Desktop (macOS)
cp SKILL.md ~/Library/Application\ Support/Claude/skills/cuentica/SKILL.md

# Claude Code
cp SKILL.md ~/.claude/skills/cuentica/SKILL.md
```

### En otros LLMs

El archivo `SKILL.md` es texto plano en Markdown. Puedes pegarlo como system prompt o instrucción de sistema en cualquier LLM con acceso a herramientas de terminal (bash/curl).

---

## Configuración del token

La primera vez que uses la skill, el asistente te pedirá tu token de API de Cuéntica.

**Cómo obtenerlo:**
1. Inicia sesión en [app.cuentica.com](https://app.cuentica.com)
2. Ve a **Configuración → API → Generar token**
3. Copia el token y díselo al asistente

El asistente lo guardará en memoria para futuras conversaciones. **No lo compartas con nadie más.**

---

## Ejemplos de uso

Una vez instalada, simplemente pregunta con lenguaje natural:

- *"¿Qué facturas tengo pendientes de cobro?"*
- *"¿Cuánto he facturado este trimestre?"*
- *"Muéstrame los gastos de enero"*
- *"¿Qué borradores tengo para emitir?"*
- *"Resumen de IVA del primer trimestre"*
- *"Facturas de KNOM DESIGN este año"*

---

## Requisitos

- Cuenta activa en [Cuéntica](https://cuentica.com)
- Token de API de Cuéntica (plan que incluya acceso API)
- Claude u otro LLM con capacidad de ejecutar comandos curl/bash

---

## Limitaciones

- Solo lectura por defecto (las escrituras requieren confirmación explícita)
- Límite de la API de Cuéntica: 600 peticiones cada 5 minutos
- Las facturas anuladas aparecen en la API pero la skill las filtra automáticamente

---

## Licencia

MIT — úsala, modifícala y compártela libremente.
