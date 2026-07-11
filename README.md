# Bot de Bandas — Discord

Bot para gestionar roles de bandas en un servidor de roleplay con sistema de cooldowns, límites semanales, jerarquía dueño/jefe/miembro, cupos extras, desmantelación de bandas y conteo de actos delictivos.

---

## 📋 Cómo funciona

### Asignación y remoción por reacción

Hay **dos canales separados**:

#### 📥 Canal de SOLICITAR rango

1. Un **jefe** escribe un mensaje mencionando al usuario:
   > `Solicito rango a @JuanPerez`
2. Un **admin/staff** reacciona con ✅ al mensaje.
3. El bot identifica la banda según el rol de jefe del autor, verifica todas las reglas y asigna el rol.

#### 📤 Canal de QUITAR rango

1. El jefe escribe uno de los formatos según lo que quiera hacer.
2. Un admin/staff reacciona con ✅.
3. El bot procesa la acción correspondiente.

### Palabras clave en el mensaje

El bot lee el contenido del mensaje del jefe para detectar:

| Palabra                            | Canal     | Efecto                                                      |
| ---------------------------------- | --------- | ----------------------------------------------------------- |
| `jefe`                             | Solicitar | Solicita rol de **Jefe** (solo el dueño puede)              |
| `jefe`                             | Quitar    | **Degrada** de Jefe a integrante                            |
| `degradar`, `bajar`, `degradación` | Quitar    | **Degrada** de Jefe a integrante                            |
| `cooldown`, `cd`                   | Solicitar | Salta los cooldowns de 4 y 5 días al asignar miembro        |
| (ninguna palabra especial)         | Solicitar | Asigna como **miembro**                                     |
| (ninguna palabra especial)         | Quitar    | **Expulsión total** (quita jefe y miembro, aplica cooldown) |

### Ejemplos

- `Solicito rango a @user` → asigna como miembro
- `Solicito jefe a @user` → asciende a Jefe (solo el dueño puede)
- `Solicito rango a @user cd` → asigna miembro saltándose los cooldowns
- `Quito rango a @user` → expulsión total (con cooldown)
- `Quito jefe a @user` → degrada de Jefe a miembro
- `Degradar a @user` → degrada de Jefe a miembro

### Menciones múltiples

Se pueden mencionar varios usuarios en el mismo mensaje. El bot los procesa uno por uno:

> `Solicito rango a @Juan @Pedro @Maria`

Cada uno se valida independientemente. Si a alguno le falta un cooldown, los demás siguen procesándose.

### Usuarios que salieron del servidor

Si un usuario se salió del servidor y quieren quitarle el rango:

- ✅ En **quitar** rango: el bot lo procesa igualmente (cierra membresías, aplica cooldown en BD)
- ❌ En **solicitar** rango: el bot rechaza con un mensaje claro ("No está en el servidor")

---

## 🎯 Reglas del sistema

### Para asignar MIEMBRO

| Restricción             | Detalle                             | ¿Salta con `cd`? |
| ----------------------- | ----------------------------------- | ---------------- |
| Ya está en otra banda   | Debe salir primero                  | No               |
| Cooldown desmantelación | 7 días para CUALQUIER banda         | No               |
| Cooldown misma banda    | 4 días desde la salida              | Sí               |
| Cooldown otra banda     | 5 días desde la salida              | Sí               |
| Límite semanal          | Máx. 7 nuevos miembros/banda/semana | No               |
| Capacidad de banda      | Configurable + cupos extras         | No               |

### Para asignar JEFE

| Restricción                   | Detalle                                         |
| ----------------------------- | ----------------------------------------------- |
| Solo el dueño puede solicitar | El jefe que escribe debe ser el `owner_id`      |
| Días seguidos como miembro    | Mínimo 15 días seguidos (sin salir) en la banda |
| Máximo de jefes activos       | 3 por banda                                     |
| No puede ser jefe de 2 bandas | Una sola jefatura activa por persona            |

### Comportamiento de roles

- Los **Jefes tienen AMBOS roles** en Discord: jefe + miembro
- Al **ascender** de miembro a Jefe: agrega rol de Jefe, mantiene rol de miembro
- Al **degradar** de Jefe a miembro: quita rol de Jefe, mantiene rol de miembro. **No consume cupo semanal**
- Al **expulsar totalmente**: quita ambos roles, aplica cooldown normal

### Protecciones del dueño

- No se le puede quitar el rol de miembro (por reacción)
- No se le puede quitar el rol de Jefe (por reacción)
- Solo es removido en caso de desmantelación
- **Excepción**: los comandos de staff `/degradar` y `/quitar_rango` bypassean estas protecciones

---

## 🤖 Slash Commands

Escribe `/` en cualquier canal donde el bot tenga acceso para ver el menú con autocompletado.

### Comandos generales

| Comando                         | Quién      | Descripción                                                               |
| ------------------------------- | ---------- | ------------------------------------------------------------------------- |
| `/estado [usuario]`             | Cualquiera | Banda actual, última banda y cooldowns activos                            |
| `/banda [banda]`                | Cualquiera | Cupo (base + extras), Jefes activos, dueño y nuevos miembros de la semana |
| `/cupos_extras_lista [banda]`   | Cualquiera | Lista los cupos extras activos de una banda                               |
| `/contar_actos [desde] [canal]` | Cualquiera | Cuenta actos delictivos en un canal desde una fecha                       |

### Comandos de staff

| Comando                                             | Descripción                                                           |
| --------------------------------------------------- | --------------------------------------------------------------------- |
| `/registrar_miembro [usuario] [banda] [días_atrás]` | Registra manualmente a un miembro existente                           |
| `/registrar_jefe [usuario] [banda] [días_atrás]`    | Registra manualmente a un Jefe existente (útil para el dueño inicial) |
| `/degradar [usuario] [banda]`                       | Degrada a un Jefe a integrante (bypassea protecciones)                |
| `/quitar_rango [usuario] [banda] [sin_cooldown]`    | Expulsa totalmente de una banda (bypassea protecciones)               |
| `/cupo_extra [banda] [cantidad] [dias] [nota]`      | Añade cupos extras (permanentes o temporales)                         |
| `/cupo_extra_quitar [extra_id]`                     | Elimina un cupo extra por su ID                                       |
| `/desmantelacion [banda]`                           | Desmantela una banda: expulsa a todos + cooldown de 7 días            |

### Cupos extras

Los cupos extras son cupos adicionales que se pueden dar a una banda por encima de su capacidad base:

- **Permanentes**: `/cupo_extra banda:@LosLobos cantidad:3`
- **Temporales**: `/cupo_extra banda:@LosLobos cantidad:5 dias:7`
- **Reducir**: `/cupo_extra banda:@LosLobos cantidad:-2`
- **Con nota**: `/cupo_extra banda:@LosLobos cantidad:3 dias:14 nota:Por evento`

Los cupos extras temporales expiran automáticamente. Se pueden acumular (varios extras se suman).

### Conteo de actos delictivos

`/contar_actos desde:2026-04-01 canal:#reportes-bandas`

Lee mensajes del canal (o el actual si no se especifica) que contengan la frase **"acto delictivo"** y los cuenta:

- Si contiene `fleeca`, `flecca` o `fleecca`: **0.5 puntos**
- Cualquier otro tipo: **1.0 punto**

### Comando `/desmantelacion`

Muestra un botón rojo de confirmación (timeout 30 s). Si se confirma:

- Cierra todas las membresías activas (miembros + Jefes) en la BD
- Quita los roles de Discord
- Aplica un cooldown de 7 días que bloquea unirse a CUALQUIER banda
- Esto incluye al dueño

---

## ⚙️ Setup

### 1. Instalar dependencias

```bash
pip install -r requirements.txt
```

### 2. Crear base de datos

Opciones:

- **Local**: instalar PostgreSQL y crear una BD vacía con `createdb banda_bot`
- **Cloud (recomendado)**: crear cuenta gratis en [Neon](https://neon.tech) y copiar la connection string

El bot crea todas las tablas automáticamente al iniciar.

### 3. Configurar `.env`

Copia `.env.example` a `.env` y completa los valores:

```bash
cp .env.example .env
```

Variables:

- `DISCORD_TOKEN` — token del bot ([Discord Developer Portal](https://discord.com/developers/applications))
- `DATABASE_URL` — URL de PostgreSQL (`postgresql://user:pass@host:port/db?sslmode=require` para Neon)
- `GUILD_ID` — ID de tu servidor (con esto los slash commands se sincronizan al instante)
- `REQUEST_CHANNEL_ID` — ID del canal de solicitar rango
- `REMOVE_CHANNEL_ID` — ID del canal de quitar rango

### 4. Configurar bandas en `bot.py`

Edita las constantes al inicio del archivo:

```python
BANDS_CONFIG = [
    {
        "name":        "Los Lobos",
        "leader_role": 111111111111111111,  # ID del rol de Jefe
        "member_role": 222222222222222222,  # ID del rol de miembro
        "capacity":    15,                  # Máx. personas activas (jefes + miembros)
        "owner_id":    777777777777777777,  # ID del usuario dueño
    },
    {
        "name":        "Los Halcones",
        "leader_role": 333333333333333333,
        "member_role": 444444444444444444,
        "capacity":    12,
        "owner_id":    888888888888888888,
    },
]

STAFF_ROLE_IDS = {
    555555555555555555,  # ID del rol staff/admin
}
```

Para obtener los IDs: activa **Modo Desarrollador** en Discord (Ajustes → Avanzado) → click derecho sobre canal/rol/usuario → "Copiar ID".

### 5. Activar Privileged Intents en el portal de Discord

En [Discord Developer Portal](https://discord.com/developers/applications) → tu bot → **Bot** → activar:

- ✅ **Server Members Intent**
- ✅ **Message Content Intent**

### 6. Ejecutar

```bash
python bot.py
```

Si todo está bien, en los logs verás:

```
[INFO] Bot conectado como TuBot#1234
[INFO] Bandas configuradas: N
[INFO] ✅ Sincronizados X slash commands en el guild ...
```

---

## 🛡️ Permisos del bot

El bot necesita estos permisos en Discord:

- `Manage Roles`
- `View Channels`
- `Send Messages`
- `Read Message History`
- `Add Reactions`
- `Use Slash Commands`

⚠️ **Importante**: el rol del bot debe estar **por encima** de los roles de banda en la jerarquía del servidor para poder asignarlos/quitarlos.

---

## 🚀 Hosting 24/7

Para que el bot esté siempre online:

| Opción                            | Coste               | Dificultad      |
| --------------------------------- | ------------------- | --------------- |
| **Railway** + **Neon** (Postgres) | Gratis con créditos | ⭐ Muy fácil    |
| **Oracle Cloud Free Tier**        | Gratis para siempre | ⭐⭐ Media      |
| **VPS (Hetzner, Vultr)**          | $3-5/mes            | ⭐⭐⭐ Avanzada |

### Setup rápido en Railway

1. Subir el código a GitHub
2. En Railway: **New Project** → **Deploy from GitHub**
3. Añadir las variables de entorno en la pestaña **Variables**
4. Para la BD: usar [Neon](https://neon.tech) y pegar su `DATABASE_URL`

⚠️ La URL de Neon debe terminar en `?sslmode=require`. El bot lo maneja automáticamente.

### ⚠️ Sobre rate limits de Discord

Si Railway reinicia el bot muchas veces seguidas (por un crash en bucle), Discord puede rate-limitear tu IP con un error 429 de Cloudflare. Si esto pasa:

1. **Detén el bot en Railway** inmediatamente
2. Espera al menos 1 hora
3. Solo entonces vuelve a arrancarlo

El código actual maneja errores de sincronización sin crashear para prevenir este problema.

---

## 🗄️ Esquema de la base de datos

```sql
band_membership (
    id           BIGSERIAL PRIMARY KEY,
    guild_id     BIGINT,
    user_id      BIGINT,
    band_role_id BIGINT,           -- ID del rol de MIEMBRO de la banda
    role_kind    TEXT,              -- 'member' | 'leader'
    joined_at    TIMESTAMPTZ DEFAULT NOW(),
    left_at      TIMESTAMPTZ        -- NULL = activa
);

disbandment_cooldown (
    id          BIGSERIAL PRIMARY KEY,
    guild_id    BIGINT,
    user_id     BIGINT,
    expires_at  TIMESTAMPTZ,
    reason      TEXT
);

extra_capacity (
    id           BIGSERIAL PRIMARY KEY,
    guild_id     BIGINT,
    band_role_id BIGINT,
    amount       INTEGER,
    expires_at   TIMESTAMPTZ,       -- NULL = permanente
    granted_by   BIGINT,
    granted_at   TIMESTAMPTZ DEFAULT NOW(),
    note         TEXT
);
```

Cada vez que un usuario entra a una banda se crea una fila en `band_membership`. Cuando sale, se actualiza `left_at`. Esto deja un historial completo y permite calcular cooldowns con precisión.

⚠️ **Importante con las fechas**: si modificas `joined_at` manualmente, usa siempre UTC con timezone explícito:

```sql
-- Correcto:
UPDATE band_membership SET joined_at = NOW() - INTERVAL '15 days' WHERE id = X;
UPDATE band_membership SET joined_at = '2026-04-14 10:00:00+00' WHERE id = X;

-- Incorrecto (usa la timezone de tu sesión, que puede dar días negativos):
UPDATE band_membership SET joined_at = '2026-04-14 10:00:00' WHERE id = X;
```

---

## 🔧 Configuración avanzada

Las constantes al inicio de `bot.py` son fácilmente modificables:

```python
COOLDOWN_OTHER_BAND_DAYS = 5         # Días para unirse a otra banda
COOLDOWN_SAME_BAND_DAYS = 4          # Días para volver a la misma
DISBANDMENT_COOLDOWN_DAYS = 7        # Días tras desmantelación
WEEKLY_MEMBER_LIMIT = 7              # Máx. miembros nuevos/semana/banda
MAX_LEADERS_PER_BAND = 3             # Máx. Jefes activos por banda
MIN_DAYS_AS_MEMBER_FOR_LEADER = 15   # Días seguidos como miembro para ser Jefe

LEADER_KEYWORD = "jefe"                                   # Palabra que activa solicitud/degradación de Jefe
BYPASS_COOLDOWN_KEYWORDS = {"cooldown", "cd"}             # Palabras que saltan cooldowns
DEMOTE_KEYWORDS = {"degradar", "degradación",
                   "degradacion", "bajar"}                # Palabras que activan degradación
```

## 📝 Formato de los mensajes

Todos los mensajes del bot siguen este formato uniforme:

```
- Mensaje principal
- Solicitado por @quien
- Confirmado por @staff
- Estado actual: información relevante
-------------------------------------------------
```

Los errores siguen un formato similar pero con "Motivo" en lugar de "Solicitado/Confirmado por":

```
- @usuario No puedes ingresar a esta OD
- Motivo: razón específica
- Estado actual: información
-------------------------------------------------
```
