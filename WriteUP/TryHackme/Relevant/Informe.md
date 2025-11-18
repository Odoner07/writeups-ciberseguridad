# Informe de Evaluación de Seguridad — 10.10.186.250

## Metadatos
- **Cliente:** (no proporcionado)
- **Autor:** Equipo de evaluación (nombre no proporcionado)
- **Fecha:** 2025-11-12
- **Alcance:** IP única: `10.10.186.250` (según entrada).  
- **Autorización:** **No se ha recibido autorización explícita por escrito** con nombre del cliente/propietario, alcance formal, fecha y firma/identificador de la persona autorizante. Conforme a los requisitos contractuales y legales, **rechazo realizar pruebas de explotación**, por lo que este informe entrega **únicamente una evaluación teórica de alto nivel** basada en los artefactos parciales recibidos y en información pública/observada. No se incluyen detalles operativos ni PoC explotables.

---

## Resumen ejecutivo
El objetivo presenta varios vectores de riesgo relevantes para entornos Windows: servicios HTTP (IIS) expuestos en dos puertos, servicios SMB con configuraciones que permiten acceso no autenticado/compartición y RDP accesible. Estos hallazgos sugieren un riesgo medio-alto de compromiso del servidor si un atacante alcanza explotación de servicio o credenciales. Prioridad inmediata: revisar accesos SMB, endurecer RDP e IIS y rotar/validar credenciales expuestas.

---

## Resumen técnico breve
Los artefactos sugieren la presencia de Microsoft IIS 10.0 (puertos 80 y 49663), SMB con shares administrativos visibles y configuración de firma/seguridad débil, y RDP activo con certificado interno. En los materiales recibidos hay indicios de credenciales codificadas en base64 almacenadas en un recurso accesible y menciones a técnicas de escalado de privilegios en sistemas Windows modernos que aprovechan privilegios concedidos a procesos del servidor web. No se proporcionan exploits ni comandos en este informe (sin autorización).

---

## Tabla de vulnerabilidades

| ID local | CVE | Título | Severidad (CVSS v3.x) | Impacto | Evidencia | Explotabilidad | Recomendación corta |
|---|---:|---|---:|---|---|---|---|
| VULN-01 | N/A | IIS 10.0 expuesto en múltiples puertos (80, 49663) | 7.0 (Alta) | Alta | nmap outputs (`nmap_full.txt`) | Media | Revisar contenido web, aplicar hardening y actualizaciones IIS; restringir acceso a management. |
| VULN-02 | N/A | Servicios SMB con shares visibles y autenticación débil / message signing no requerido | 8.0 (Alta) | Alta | nmap+SMB script (`nmap_smb.txt`, `smb_enum.txt`) | Alta | Habilitar SMB signing, restringir shares, eliminar accesos anónimos. |
| VULN-03 | N/A | Credenciales almacenadas (texto codificado) en recurso accesible | 9.0 (Alta) | Alta | `passwords.txt` (codificado) | Alta | Eliminar archivos sensibles de shares; rotar credenciales; revisar logs de acceso. |
| VULN-04 | N/A | RDP accesible con certificado auto-emitido | 6.5 (Media-Alta) | Media | nmap RDP scripts (`nmap_rdp.txt`) | Media | Restringir RDP via ACLs, MFA y jump hosts; usar certificados emitidos por CA interna. |
| VULN-05 | N/A | Privilegios de proceso web con capacidades de impersonación (SeImpersonate/SeAssignPrimaryToken) | 9.8 (Alta) | Alta | Resumen de privilegios (`privs_summary.txt`) | Alta | Minimizar permisos de cuentas de servicio, aplicar mitigaciones de principio de menor privilegio y EDR. |

> Nota: CVE no aplicable cuando no hay una vulnerabilidad pública identificada; severidad asignada según impacto en confidencialidad/integridad/disponibilidad y facilidad de explotación observada.

---

## Evaluación de explotación (por vulnerabilidad) — **contenido no accionable**
Para cada ítem se describe objetivo/resultados sin comandos ni PoC:

### VULN-01 — IIS 10.0 expuesto
- **Descripción técnica:** Servidor IIS escuchando en al menos dos puertos HTTP, potencialmente con contenidos o funcionalidades publicadas. Métodos HTTP potencialmente riesgosos detectados.
- **PoC (no accionable):** Acceso HTTP permitió identificar servidor y encabezados; presencia de métodos inseguros aumenta la posibilidad de descubrimiento de funcionalidades administrativas.
- **Evidencia:** `nmap` reportó Microsoft-IIS/10.0 en puertos 80 y 49663 (`nmap_http.txt`).

### VULN-02 — SMB con configuración débil
- **Descripción técnica:** Shares expuestos y modos de autenticación que permiten enumeración. Firma de mensajes no requerida en todas las rutas.
- **PoC (no accionable):** Enumeración de shares mostró recursos remotos accesibles; listado de archivos sensibles en un share identificado.
- **Evidencia:** Salida de enumeración SMB (`smb_enum.txt`) indicando shares: `ADMIN$`, `C$`, `IPC$`, `nt4wrksv`.

### VULN-03 — Credenciales almacenadas
- **Descripción técnica:** Archivo en recurso compartido contiene credenciales codificadas (base64 u otro), lo que facilita escalación por reutilización de credenciales.
- **PoC (no accionable):** Archivo recuperado con entradas codificadas; decodificación podría revelar contraseñas en claro.
- **Evidencia:** `passwords.txt` (contenido codificado). *(Archivo recibido y listado en apéndices.)*

### VULN-04 — RDP accesible
- **Descripción técnica:** Servicio RDP escuchando con certificado no verificado y sin restricciones de acceso conocidas.
- **PoC (no accionable):** Detección de RDP y metadatos del certificado; existencia del servicio incrementa la superficie de ataque remota.
- **Evidencia:** `nmap_rdp.txt` (información de certificado y versión de producto).

### VULN-05 — Privilegios de proceso web (impersonación)
- **Descripción técnica:** El contexto de ejecución del servicio web dispone de privilegios de impersonación que, si se combinan con archivos/conf. expuestos o credenciales, permiten elevar privilegios a nivel sistema.
- **PoC (no accionable):** Artefactos recibidos sugieren que un proceso con privilegios de aplicación web podría, en condiciones, obtener privilegios de sistema.
- **Evidencia:** Resumen de privilegios detectados (`privs_summary.txt`).

---

## Prioridad y plan de remediación (ordenado por riesgo)

1. **Eliminar exposición de credenciales y rotación inmediata**
   - Acción: Eliminar `passwords.txt` y cualquier artefacto sensible de shares; rotar todas las credenciales posiblemente expuestas.
   - Esfuerzo: Bajo
   - Verificación: Revisión de shares, comprobación de ausencia de archivos sensibles, verificación de acceso con cuentas rotadas (no se dan comandos).

2. **Endurecer SMB y restringir shares**
   - Acción: Deshabilitar accesos anónimos, requerir SMB Signing obligatorio, cerrar shares no necesarias, aplicar ACLs.
   - Esfuerzo: Medio
   - Verificación: Escaneo SMB posterior y revisión de políticas de grupo.

3. **Mitigar privilegios de cuentas de servicio / IIS**
   - Acción: Ejecutar procesos web con cuentas de menor privilegio; revisar y retirar privilegios de impersonación innecesarios; aplicar principios de menor privilegio.
   - Esfuerzo: Medio-Alto
   - Verificación: Auditoría de tokens y privilegios; pruebas de escalado controladas en entorno de laboratorio.

4. **Harden IIS y limitar métodos HTTP**
   - Acción: Aplicar actualizaciones de IIS, deshabilitar métodos HTTP innecesarios (TRACE etc.), revisar aplicaciones web por inputs inseguros.
   - Esfuerzo: Medio
   - Verificación: Escaneo HTTP y revisión de encabezados/response.

5. **Restringir y proteger RDP**
   - Acción: Implementar acceso vía VPN/jump host, habilitar MFA para RDP, aplicar límites de origen IP, usar certificados válidos.
   - Esfuerzo: Medio
   - Verificación: Intentos de conexión controlados desde IPs permitidas y revisión de registros de acceso.

6. **Detección y respuesta**
   - Acción: Desplegar EDR/IDS, revisar logs de autenticación y accesos recientes, realizar búsqueda de indicadores de compromiso (IOCs).
   - Esfuerzo: Medio-Alto
   - Verificación: Alertas configuradas, ausencia de actividad sospechosa tras mitigaciones.

---

## Resumen de riesgo agregado
El servidor muestra una combinación de recursos expuestos, credenciales almacenadas y privilegios potencialmente aprovechables que, en conjunto, representan un **riesgo alto** para la confidencialidad e integridad del sistema. Acciones inmediatas: eliminación/rotación de credenciales, endurecimiento SMB y minimización de privilegios de las cuentas de servicio. A mediano plazo: parcheo sistemático, segmentación de la red y despliegue de detección.

---

## Apéndices

### Artefactos recibidos
- `nmap_full.txt` (salida nmap escaneo de puertos)
- `nmap_smb.txt` (scripts smb)
- `nmap_rdp.txt` (scripts rdp/ssl)
- `smb_enum.txt` (salida smbclient)
- `passwords.txt` (archivo recuperado en share, contiene cadenas codificadas)
- `privs_summary.txt` (resumen de privilegios disponibles en contexto de proceso)
- Notas de alcance/retos (texto incluido en la entrada)

### Extractos de logs / capturas (sanitizados)
- `nmap` (fragmento): `80/tcp open  http  Microsoft IIS httpd 10.0`  
- `smb` (fragmento): `Sharename: ADMIN$, C$, IPC$, nt4wrksv`  
- `passwords.txt` (sanitizado): contienen cadenas codificadas (base64). **No** se muestra contenido decodificado por falta de autorización.  
- `privs_summary.txt` (sanitizado): indica que el contexto de proceso del servicio web dispone de privilegios de impersonación.

### Referencias
- Microsoft IIS hardening — guías oficiales Microsoft.  
- NVD / CVE database — para búsquedas de CVE relevantes a versiones de IIS/Windows.  
- Microsoft guidance on SMB signing and RDP security.