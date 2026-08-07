# ¡güao! — Backend compartido en Supabase (con seguridad desde el inicio)

## Paso 1 — Crear el proyecto (5 min)
1. Entra a **https://supabase.com** → **Start your project** → crea cuenta (con GitHub o correo).
2. **New project** → nombre: `guao` → elige una **contraseña de base de datos fuerte** (guárdala en un lugar seguro, no la necesitarás seguido) → región: la más cercana a Colombia (ej. `us-east-1`).
3. Espera ~2 minutos a que aprovisione.

## Paso 2 — Activar autenticación por correo/clave
1. En el menú lateral: **Authentication → Providers**.
2. Deja **Email** activado (viene por defecto).
3. **Authentication → Settings** → desactiva "Enable email confirmations" si quieres poder crear usuarios del equipo sin esperar el correo de verificación (opcional, más simple para un equipo pequeño).
4. **Authentication → Users → Add user** → crea un usuario para ti (y uno por cada persona de tu equipo que vaya a usar la app). Esto es lo que reemplaza "cualquiera con el link entra" por "solo quien tenga usuario y clave entra".

## Paso 3 — Crear las tablas CON seguridad (RLS activa desde el día 1)
Ve a **SQL Editor** → pega y ejecuta esto completo:

```sql
-- TABLAS
create table clientes (
  key text primary key,
  nombre text not null,
  id_cliente text,
  direccion text,
  actualizado timestamptz default now()
);

create table ventas (
  id bigint primary key,
  fecha date,
  cliente_key text references clientes(key),
  tipo_producto text, categoria text, producto text, tienda text, talla text,
  metodo_pago text, costo_usd numeric, trm numeric, envio_usd numeric,
  costo_cop numeric, pvp numeric, utilidad numeric,
  abono numeric, pendiente numeric, estado text,
  abonos jsonb, actualizado timestamptz default now()
);

create table inventario (
  id bigint primary key,
  fecha_compra date, producto text, descripcion text,
  tipo_producto text, categoria text, color text, talla text,
  cantidad int, moneda text, costo numeric, envio_usd numeric,
  trm_estim numeric, costo_cop numeric, valor_venta numeric,
  vendido boolean default false, fecha_venta date
);

-- SEGURIDAD: activa Row Level Security en las 3 tablas
alter table clientes enable row level security;
alter table ventas enable row level security;
alter table inventario enable row level security;

-- SEGURIDAD: solo usuarios autenticados (con login) pueden leer y escribir
-- (nadie anónimo puede tocar los datos, aunque tenga la anon key)
create policy "solo autenticados leen clientes" on clientes for select using (auth.role() = 'authenticated');
create policy "solo autenticados escriben clientes" on clientes for insert with check (auth.role() = 'authenticated');
create policy "solo autenticados actualizan clientes" on clientes for update using (auth.role() = 'authenticated');
create policy "solo autenticados borran clientes" on clientes for delete using (auth.role() = 'authenticated');

create policy "solo autenticados leen ventas" on ventas for select using (auth.role() = 'authenticated');
create policy "solo autenticados escriben ventas" on ventas for insert with check (auth.role() = 'authenticated');
create policy "solo autenticados actualizan ventas" on ventas for update using (auth.role() = 'authenticated');
create policy "solo autenticados borran ventas" on ventas for delete using (auth.role() = 'authenticated');

create policy "solo autenticados leen inventario" on inventario for select using (auth.role() = 'authenticated');
create policy "solo autenticados escriben inventario" on inventario for insert with check (auth.role() = 'authenticated');
create policy "solo autenticados actualizan inventario" on inventario for update using (auth.role() = 'authenticated');
create policy "solo autenticados borran inventario" on inventario for delete using (auth.role() = 'authenticated');
```

Esto significa: **sin iniciar sesión, nadie puede leer ni escribir nada** — ni siquiera con la clave pública visible en el código de la app. Esa es la protección real.

## Paso 4 — Copiar las credenciales
Ve a **Settings (⚙) → API**. Copia y pásame estos dos valores:
1. **Project URL** (algo como `https://xxxxx.supabase.co`)
2. **anon public key** (una cadena larga que empieza con `eyJ...`)

⚠️ **Nunca me copies ni uses en la app la `service_role key`** — esa es solo para uso administrativo tuyo, nunca debe ir en el navegador.

## Paso 5 — Yo conecto la app
Con esos dos datos, yo:
- Agrego el login (correo/clave) como pantalla de entrada de la app.
- Reemplazo el guardado local por sincronización en tiempo real con tus tablas.
- Dejo el respaldo local como copia de seguridad adicional (no se pierde nada).

---
Cuando tengas la **Project URL** y la **anon key**, pégamelas en el chat y continúo con la conexión.
