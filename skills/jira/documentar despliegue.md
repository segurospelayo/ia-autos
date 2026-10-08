---
name: documentar-despliegue-acon
description: Automatiza la generación y publicación de notas de versión y actualización del histórico de despliegues en Confluence para Autos Contratación (ACON) a partir de una entrega correctora.
triggers:
  - "documentar despliegue"
  - "publicar notas de entrega"
  - "actualizar histórico despliegues"
  - "despliegue entrega correctora"
---

# Skill: Documentación de Despliegues Autos Contratación (ACON)

Esta skill guía al asistente para recopilar la información de una entrega correctora o pase a producción, relacionar los tickets técnicos de arquitectura (ARQ/GOBI) con las incidencias funcionales (ACON), crear la subpágina de notas técnicas y actualizar la tabla histórica acumulativa en Confluence.

---

## ⚠️ Reglas Críticas e Invariables

1. **Espacio Confluence Obligatorio**:
   * **SIEMPRE** publicar en el espacio **`ACON`** (*Autos Contratación*, Space ID: `54657024`).
   * **NUNCA** publicar en el espacio obsoleto **`AMVP`** (*AUTOS MVP*).

2. **Ubicación Jerárquica**:
   * **Página Padre para Notas Técnicas**:
     * URL: `https://pelayo.atlassian.net/wiki/spaces/ACON/pages/54659152/Historico+de+Despliegues`
     * Page ID: `54659152`
   * **Página de la Tabla Histórica Acumulativa**:
     * URL: `https://pelayo.atlassian.net/wiki/spaces/ACON/pages/54657183/Despliegues+Contrataci+n`
     * Page ID: `54657183`

3. **Trazabilidad ARQ / Gobierno IT**:
   * Es mandatorio identificar y registrar el ticket padre de despliegue técnico (`GOBI-XXXXX`).
   * Desglosar las subtareas técnicas ejecutadas por ARQ/Infraestructura:
     * Promoción a `master` / integración de rama.
     * Ejecución de `promote-to-pro` o despliegue en el entorno.
     * Cambios y tags de configuración/properties (`*-aro-config-pro`).
     * Actualización en API Manager (si aplica).

4. **Control Dual de Versiones**:
   * **Versión de aplicativo**: Tag de release o rama fuente (`hotfix/...`, `release/...`).
   * **Versión de configuración**: Tag exacto de properties/config (ej. `1.0.14_000118`).

---

## Flujo de Trabajo (Workflow)

### Paso 1: Solicitud de Entrada
Si el usuario no ha facilitado la URL en el prompt inicial, solicitar:
> *"Por favor, facilita la URL o identificador donde se encuentra la información de la entrega correctora (URL de Jira Release, ticket GOBI de despliegue, ticket ACON o pull request/entrega)."*

Si el usuario facilita una entrega, ticket ACON o incidencia funcional relacionada, **buscar primero desde ese elemento el ticket padre de despliegue GOBI/ARQ**: revisar vínculos entrantes/salientes, padre, subtareas, descripción y comentarios; seguir los enlaces a subtareas técnicas y validar cuál contiene la promoción a `master` y/o el despliegue a PRO. Si se identifica de forma inequívoca, continuar sin volver a pedir el GOBI. Preguntar por el ticket GOBI solo si no se encuentra una relación concluyente o hay varios candidatos ambiguos. Si también proporciona una versión Jira, usarla; preguntar únicamente por los datos realmente ausentes.

**Credenciales:** nunca solicitar que se incluya un API token en esta skill, guardarlo en archivos, ni copiarlo a documentación. Usar autenticación local ya configurada (acli/MCP/gestor de credenciales); si no está disponible, pedir que el usuario se autentique localmente. Si un token se expone en el chat, recomendar su revocación y rotación.

---

### Paso 2: Extracción y Verificación de Datos
A partir de la URL aportada y las fuentes en Jira/Git, extraer y validar la siguiente matriz de información:

| Campo | Descripción | Ejemplo |
| :--- | :--- | :--- |
| **Entorno** | PRO, PRE o ACPT | `PRO` |
| **Fecha y Hora** | Fecha y hora de ejecución | `08/10/2026 - 08:00` |
| **Jira Despliegue ARQ** | Ticket padre `GOBI` | `GOBI-16317` |
| **Subtareas ARQ** | Subtareas técnicas de despliegue | `GOBI-16318`, `GOBI-16319` |
| **Artefacto / Servicio** | Nombre exacto del microservicio | `uw-auto-contratacion-bff` |
| **Versión Aplicativo** | Tag o versión de release / rama | `9.61.2` (`hotfix/err_distinct_data`) |
| **Versión Configuración**| Tag de properties / config en PRO | `1.0.14_000118` |
| **Tickets ACON** | Incidencias funcionales resueltas | `ACON-83331`, `ACON-83319` |
| **Versión / Release Jira**| Nombre de la versión en Jira ACON | *Solución ERR_DISTINCT_DATA y recibos* |
| **Observaciones** | Resumen operativo y componentes | Ajustes de base de datos, API Manager, etc. |

---

### Paso 3: Generación de la Página de Notas de Entrega
Crear una subpágina en Confluence con las siguientes características:
* **Espacio**: `ACON`
* **Página Padre**: `54659152` (*Historico de Despliegues*)
* **Título sugerido**: `Notas de Despliegue [Entorno] — [Nombre del Componente / Release] — DD/MM/AAAA`
* **Estructura del contenido**:
  1. **Panel Informativo**: Componente, entorno, fecha, versión de release y estado.
  2. **Trazabilidad Técnica ARQ**:
     * Ticket de pase principal (`GOBI-XXXXX`).
     * Listado de subtareas técnicas con su estado y descripción.
  3. **Artefactos y Configuración**:
     * Nombre del microservicio.
     * Tag de código y rama de origen.
     * Tag de configuración (`aro-config-pro`).
  4. **Alcance Funcional (Jira ACON)**:
     * Tabla con Clave, Tipo, Resumen, Estado y Prioridad de cada issue `ACON-XXXXX`.
  5. **Acciones Post-Despliegue / Observaciones**: Reinicio de sockets, sincronización de BBDD o cambios en API Manager.

---

### Paso 4: Actualización de la Tabla Histórica Acumulativa
Acceder a la página **Despliegues Contratación** (`54657183`) y añadir la(s) fila(s) correspondiente(s) al final de la tabla principal de despliegues:

```html
<tr>
  <td style="text-align: center;"><strong>[PRO/PRE/ACPT]</strong></td>
  <td style="text-align: center;">DD/MM/AAAA</td>
  <td style="text-align: center;">HH:MM</td>
  <td>
    <a href="https://pelayo.atlassian.net/browse/GOBI-XXXXX">GOBI-XXXXX</a><br/>
    <a href="https://pelayo.atlassian.net/browse/ACON-YYYYY">ACON-YYYYY</a>
  </td>
  <td style="text-align: center;">[Versión Front o -]</td>
  <td style="text-align: center;">[Versión BFF o -]</td>
  <td style="text-align: center;">[Versión Micro / Backend]</td>
  <td style="text-align: center;">[Librería o -]</td>
  <td style="text-align: center;">[Tag Configuración / Properties]</td>
  <td style="text-align: center;">-</td>
  <td style="text-align: center;">-</td>
  <td style="text-align: center;">-</td>
  <td style="text-align: center;">-</td>
  <td style="text-align: center;">[SQL o -]</td>
  <td style="text-align: center;">-</td>
  <td style="text-align: center;">-</td>
  <td style="text-align: center;">-</td>
  <td>[Observaciones técnicas, ramas, subtareas ARQ y API Manager]</td>
</tr>
