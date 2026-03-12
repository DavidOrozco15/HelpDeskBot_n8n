# 🤖 HelpDeskBot

Bot conversacional de soporte interno desarrollado en **Telegram** con **n8n Community Edition** y **Google Sheets** como sistema de persistencia de datos.

---

## 📋 Descripción

HelpDeskBot es un bot diseñado para gestionar solicitudes internas de soporte, tales como incidentes técnicos, solicitudes administrativas y consultas generales, mediante flujos conversacionales controlados y automatización básica.

El bot **no toma decisiones autónomas ni aprende de los datos**. Su función es organizar, registrar y presentar información, guiando al usuario mediante opciones numéricas claras y mensajes humanizados.

---

## 🛠️ Tecnologías

| Componente | Tecnología |
|---|---|
| Interfaz de usuario | Telegram |
| Motor de automatización | n8n Community Edition |
| Base de datos | Google Sheets |

---

## 📁 Estructura del repositorio

```
HelpDeskBot/
├── HelpDeskBot_No_IA.json        # Workflow principal (n8n)
├── Cambio_de_Estado.json         # Workflow de cambio automático de estado (n8n)
└── README.md
```

---

## 🗃️ Modelo de datos (Google Sheets)

Documento: `HelpDeskBot_DB`

### Hoja: SOLICITUDES
| Campo | Descripción |
|---|---|
| `id_ticket` | Identificador único del ticket (ej: TKT-20260311-1234) |
| `tipo` | Tipo de solicitud (Soporte técnico / Solicitud administrativa / Consulta general) |
| `prioridad` | Prioridad (Alta / Media / Baja) |
| `descripcion` | Descripción detallada del problema o solicitud |
| `estado` | Estado actual (Abierto / En proceso / Cerrado) |
| `creado_por` | Username de Telegram del solicitante |
| `fecha_creacion` | Fecha y hora de creación (formato: YYYY-MM-DD HH:MM:SS) |

### Hoja: USUARIOS
| Campo | Descripción |
|---|---|
| `telegram_user` | Username de Telegram |
| `nombre` | Nombre del usuario |
| `rol` | Rol (Agente / Admin / etc.) |
| `activo` | Estado del usuario (TRUE / FALSE) |

### Hoja: LOGS
| Campo | Descripción |
|---|---|
| `timestamp` | Fecha y hora del evento |
| `telegram_user` | Usuario que generó el evento |
| `pantalla` | Estado conversacional activo |
| `opcion` | Opción o texto ingresado |
| `resultado` | Resultado de la operación |

### Hoja: SESIONES
| Campo | Descripción |
|---|---|
| `telegram_user` | Username de Telegram |
| `estado` | Estado conversacional actual del usuario |
| `datos_temp` | Datos temporales del flujo en curso (JSON) |
| `updated_at` | Última actualización de la sesión |

---

## 🔄 Arquitectura de workflows

### Workflow 1: HelpDeskBot_No_IA
Se activa con cada mensaje recibido en Telegram.

```
Telegram Trigger
      ↓
Edit Fields (extrae chatId, username, text)
      ↓
Leer Sesion + Leer Usuario (Google Sheets)
      ↓
Logica (Code node - cerebro del bot)
      ↓
Switch (5 ramas según acción)
   ├── solo_responder → Merge
   ├── crear_ticket → Guardar Ticket → Merge
   ├── consultar_ticket → Buscar Ticket → Formatear Ticket → Merge
   ├── mis_solicitudes → Mis Solicitudes → Formatear Solicitudes → Merge
   └── reportes → Leer Reportes → Formatear Reporte → Merge
         ↓
   Normalizar Datos
         ↓
   Enviar Respuesta (Telegram)
         ↓
   Buscar Sesion → Existe la sesion?
      ├── Sí → Actualizar Sesion
      └── No → Crear Sesion
         ↓
   Merge Sesion → Registrar Log
```

### Workflow 2: Cambio_de_Estado
Se ejecuta automáticamente cada minuto para actualizar estados de tickets.

```
Schedule Trigger (cada 1 minuto)
      ↓
Leer Tickets (Google Sheets - SOLICITUDES)
      ↓
Calcular Cambios (Code node)
      ↓
Hay cambios? (IF)
   └── TRUE → Actualizar Estado Auto (Google Sheets)
```

**Lógica de cambio automático:**
- `Abierto` → `En proceso`: después de 5 minutos desde `fecha_creacion`
- `En proceso` → `Cerrado`: después de 10 minutos desde `fecha_creacion`

---

## 💬 Menú y flujos conversacionales

### Menú Principal
```
🤖 HelpDeskBot

🆘 0 · Ayuda
🆕 1 · Crear solicitud
🔎 2 · Consultar ticket
📁 3 · Mis solicitudes
📈 4 · Reportes
🔧 5 · Configuración
```

### Flujo: Crear solicitud (opción 1)
```
Seleccionar tipo (1-3)
      ↓
Seleccionar prioridad (1-3)
      ↓
Escribir descripción (mínimo 10 caracteres)
      ↓
Confirmar (S/N)
      ↓
Ticket creado con ID único
```

### Flujo: Consultar ticket (opción 2)
```
Ingresar ID del ticket
      ↓
Mostrar información del ticket con estado actualizado
```

---

## ✅ Validaciones implementadas

- Usuario debe estar registrado y activo en la hoja USUARIOS
- Descripción con mínimo 10 caracteres
- Confirmación obligatoria antes de guardar
- Opciones numéricas válidas en cada menú
- En cualquier momento escribir `9` cancela el flujo
- Escribir `menú` vuelve al menú principal

---

## ⚙️ Configuración e instalación

### Requisitos previos
- n8n Community Edition instalado
- Cuenta de Telegram
- Google Sheets con la estructura de datos definida
- Credenciales de Google Sheets OAuth2
- Bot de Telegram creado desde @BotFather

### Pasos de instalación

1. **Importar workflows en n8n:**
   - Ir a n8n → Workflows → Import
   - Importar `HelpDeskBot_No_IA.json`
   - Importar `Cambio_de_Estado.json`

2. **Configurar credenciales:**
   - Agregar credencial de Telegram con el token del bot
   - Agregar credencial de Google Sheets OAuth2

3. **Configurar Google Sheets:**
   - Crear el documento `HelpDeskBot_DB`
   - Crear las hojas: `SOLICITUDES`, `USUARIOS`, `LOGS`, `SESIONES`
   - Agregar las columnas según el modelo de datos
   - Registrar al menos un usuario en la hoja USUARIOS con `activo = TRUE`

4. **Activar workflows:**
   - Activar `HelpDeskBot_No_IA` → responde mensajes de Telegram
   - Activar `Cambio_de_Estado` → corre cada minuto automáticamente

---

## 🧪 Prueba del bot

1. Abrir Telegram y buscar el bot
2. Enviar cualquier mensaje para iniciar
3. Navegar por el menú usando los números
4. Crear una solicitud de prueba
5. Esperar 5 minutos y consultar el ticket para verificar el cambio automático de estado

---

## 📝 Notas importantes

- No eliminar filas de la hoja **SESIONES** — contiene el estado conversacional activo de cada usuario
- Si un usuario queda en un estado incorrecto, cambiar manualmente la columna `estado` a `MENU` y `datos_temp` a `{}`
- Los dos workflows son independientes y se comunican únicamente a través de Google Sheets
