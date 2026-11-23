# 🚀 Supabase Keep Alive

Sistema automatizado para realizar consultas periódicas a múltiples proyectos de **Supabase**, utilizando **GitHub Actions**.

El objetivo de este repositorio es generar actividad periódica en las bases de datos de proyectos alojados en el plan gratuito de Supabase, ayudando a reducir el riesgo de que sean pausados automáticamente por inactividad.

## 📌 Características

- 🔄 Automatización de consultas a Supabase mediante GitHub Actions.
- 🗄️ Soporte para hasta 5 proyectos de Supabase desde un único repositorio.
- ⏰ Ejecución automática tres veces al día.
- 🔐 Gestión segura de URLs y claves mediante GitHub Secrets.
- ⚡ No requiere mantener un computador encendido.
- ➕ Permite agregar nuevos proyectos sin crear repositorios adicionales.
- 🛠️ Ejecución manual disponible para pruebas y diagnóstico.

## ⏰ Horarios de ejecución

El workflow está configurado para ejecutarse automáticamente en los siguientes horarios:

| Hora Colombia | Hora UTC |
|---|---|
| 7:00 a. m. | 12:00 UTC |
| 12:00 p. m. | 17:00 UTC |
| 5:00 p. m. | 22:00 UTC |

La programación se encuentra definida mediante la expresión cron:

```yaml
schedule:
  - cron: '0 12,17,22 * * *'
```

## ⚙️ ¿Cómo funciona?

El sistema utiliza GitHub Actions para ejecutar solicitudes HTTP a la API REST de cada proyecto de Supabase.

Para cada proyecto realiza la siguiente consulta:

```http
GET /rest/v1/keep_alive?select=id&limit=1
```

Esta consulta obtiene un registro de la tabla `keep_alive`, generando una lectura real en PostgreSQL.

El workflow comprueba individualmente cada proyecto y muestra su resultado en los registros de ejecución de GitHub Actions.

### Flujo de funcionamiento

1. GitHub Actions inicia automáticamente según el horario establecido.
2. Lee las credenciales almacenadas en GitHub Secrets.
3. Identifica los proyectos configurados.
4. Realiza una consulta a la tabla `keep_alive` de cada proyecto.
5. Registra el resultado de cada consulta.
6. Finaliza indicando cuántos proyectos respondieron correctamente y cuántos presentaron errores.

## 🗂️ Estructura del repositorio

```text
supabase-keep-alive/
│
├── .github/
│   └── workflows/
│       └── supabase-keep-alive.yml
│
└── README.md
```

## 🔐 Configuración de GitHub Secrets

Para conectar los proyectos, es necesario configurar las credenciales en:

**GitHub → Settings → Secrets and variables → Actions**

Cada proyecto requiere dos secretos:

| Proyecto | URL | Clave pública |
|---|---|---|
| Proyecto 1 | `SUPABASE_URL` | `SUPABASE_ANON_KEY` |
| Proyecto 2 | `SUPABASE_URL_2` | `SUPABASE_ANON_KEY_2` |
| Proyecto 3 | `SUPABASE_URL_3` | `SUPABASE_ANON_KEY_3` |
| Proyecto 4 | `SUPABASE_URL_4` | `SUPABASE_ANON_KEY_4` |
| Proyecto 5 | `SUPABASE_URL_5` | `SUPABASE_ANON_KEY_5` |

Las credenciales se obtienen desde la configuración de API de cada proyecto de Supabase.

**Importante:** Se deben utilizar las claves públicas `sb_publishable_...` o las antiguas claves `anon`. Nunca se deben utilizar claves secretas `sb_secret_...` ni `service_role`.

## 🗄️ Configuración de Supabase

Cada proyecto que se desee conectar debe contar con una tabla llamada `keep_alive` en el esquema `public`.

Para crearla, ejecutar el siguiente SQL desde el SQL Editor de Supabase:

```sql
CREATE TABLE IF NOT EXISTS public.keep_alive (
    id SERIAL PRIMARY KEY
);

INSERT INTO public.keep_alive DEFAULT VALUES;

ALTER TABLE public.keep_alive ENABLE ROW LEVEL SECURITY;

DROP POLICY IF EXISTS "Allow keep alive read"
ON public.keep_alive;

CREATE POLICY "Allow keep alive read"
ON public.keep_alive
FOR SELECT
TO anon
USING (true);
```

Esta tabla está destinada exclusivamente a las consultas de mantenimiento.

## ➕ Agregar un nuevo proyecto

Para incorporar otro proyecto de Supabase:

1. Crear la tabla `keep_alive` en el nuevo proyecto.
2. Obtener su Project URL.
3. Obtener su clave pública API.
4. Entrar a GitHub → Settings → Secrets and variables → Actions.
5. Crear los dos secretos correspondientes al número del proyecto.
6. Ejecutar manualmente el workflow para comprobar la conexión.

No es necesario modificar el archivo YAML mientras se utilicen los espacios disponibles del 1 al 5.

## 📊 Verificar las ejecuciones

Para consultar el estado de la automatización:

1. Entrar al repositorio de GitHub.
2. Abrir la pestaña **Actions**.
3. Seleccionar **Supabase Keep Alive**.
4. Abrir cualquier ejecución.
5. Revisar los resultados del trabajo `keep-alive`.

Los resultados posibles son:

- `SUCCESS`: el proyecto respondió correctamente.
- `FAILED`: se presentó un error durante la consulta.
- `Not configured`: el proyecto no tiene sus credenciales configuradas y se omite automáticamente.

## ⚠️ Consideraciones importantes

- GitHub Actions debe permanecer habilitado.
- El repositorio debe conservar los Secrets configurados.
- Cada proyecto debe tener su propia tabla `keep_alive`.
- Los horarios corresponden a la zona horaria de Colombia.
- GitHub Actions puede presentar retrasos en las ejecuciones programadas.
- Las consultas generan actividad real en la base de datos, pero **no garantizan que Supabase excluya el proyecto de su política de pausa automática**.
- Se deben respetar los límites de uso de los planes gratuitos de GitHub y Supabase.

## 🛠️ Tecnologías utilizadas

- [GitHub](https://github.com/)
- [GitHub Actions](https://docs.github.com/en/actions)
- [Supabase](https://supabase.com/)
- PostgreSQL
- API REST de Supabase
- YAML
- Bash

## 📄 Licencia

Proyecto de uso personal y educativo. Puede adaptarse para administrar múltiples proyectos de Supabase, respetando las condiciones de uso de cada plataforma.

---

**Supabase Keep Alive**  
Automatización centralizada para proyectos personales de Supabase.
