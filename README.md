# K@rin@ cyber — Sistema Administrativo Integral

Sistema web completo (backend + frontend + base de datos) para el control de
punto de venta, inventario, servicios de impresión, recargas telefónicas,
caja chica y seguridad digital de la empresa **K@rin@ cyber**, desarrollado
para el proyecto de **Ingeniería de Software I**.

Incluye base de datos relacional, autenticación con roles, y un dashboard
con analítica en tiempo real. Está listo para instalar y ejecutar de forma
local — no necesita configurar ningún servidor de base de datos aparte.

---

## 1. Requisitos previos

Solo necesitas tener instalado **Node.js version 22.5 o superior**
(idealmente la version LTS más reciente). Descárgalo gratis desde:
https://nodejs.org

Para verificar que ya lo tienes instalado, abre una terminal (**usa el
Símbolo del sistema / CMD, no PowerShell** — PowerShell suele bloquear la
ejecución de `npm` por su política de seguridad por defecto) y escribe:

```
node -v
```

Si te muestra un numero de version (ej. `v24.16.0`), estas listo.

## 2. Instalación

1. Descomprime la carpeta `karina-cyber-sistema` en tu computadora.
2. Abre una terminal dentro de esa carpeta.
3. Ejecuta:

```
npm install
```

Esto descargará automáticamente todas las librerías necesarias (Express,
la base de datos SQLite, Bootstrap, Chart.js, etc.) y las copiará para que
el sistema funcione **sin conexión a internet** una vez instalado.

## 3. Ejecución

```
npm start
```

Veras un mensaje como este:

```
========================================================
  K@rin@ cyber - Sistema Administrativo Integral
  Servidor activo en: http://localhost:3000
========================================================
```

Abre tu navegador en **http://localhost:3000** y listo.

> La primera vez que se ejecuta, el sistema crea automáticamente la base de
> datos (`data/karina_cyber.db`) y la llena con usuarios, catálogo de
> productos e historial de los últimos 14 días para que puedas explorar el
> dashboard con datos reales desde el primer momento. Las siguientes veces
> que ejecutes `npm start`, se reutiliza la misma base de datos (tus datos
> no se pierden).

Para detener el servidor, presiona `Ctrl + C` en la terminal.

### ¿Quieres empezar de cero?

Si quieres borrar todos los datos y volver a generar el set de demostración,
cierra el servidor y elimina el archivo `data/karina_cyber.db` (y sus
archivos `-wal`/`-shm` si existen), luego ejecuta `npm start` de nuevo.

---

## 4. Usuarios de acceso (demostración)

La pantalla de inicio de sesión incluye botones de acceso rápido para cada
rol. Las credenciales son:

| Usuario      | Contraseña    | Rol                             |
|--------------|---------------|----------------------------------|
| `superadmin` | `Karina2026*` | Superadministrador (acceso total) |
| `admin`      | `Admin2026*`  | Administrador / Propietaria      |
| `cajero1`    | `Cajero2026*` | Operador de caja                 |
| `cajero2`    | `Cajero2026*` | Operador de caja                 |
| `auditor`    | `Auditor2026*`| Invitado / Auditor (solo lectura)|

---

## 5. Módulos del sistema

| Módulo | Descripción |
|---|---|
| **Dashboard** | KPIs del día, ingresos de los últimos 7 días, ingresos por categoría, curva de demanda por franja horaria, alertas de stock bajo y productos más vendidos. |
| **Punto de venta (POS)** | Catálogo con búsqueda y categorías, carrito de compra, y descuento automático del inventario al confirmar la venta. |
| **Servicios de impresión** | Registro de fotocopias, impresiones, escaneos y anillado, con **conciliación de contador físico** (inicial/final) contra lo cobrado, señalando discrepancias. |
| **Recargas telefónicas** | Registro de recargas Tigo/Claro, calculando automáticamente la comisión de la operadora y la ganancia neta del negocio. |
| **Caja chica** | Apertura y cierre de turno, registro de gastos operativos, y **arqueo de caja** (efectivo contado vs. efectivo esperado por el sistema). |
| **Inventario y Kardex** | Catálogo de productos, proveedores, órdenes de compra con IVA soportado, y un **Kardex por costo promedio ponderado** (entradas, salidas y saldo recalculado automáticamente). |
| **Reportes** | Margen de utilidad por producto (ingresos vs. costo real de Kardex) en un rango de fechas, con exportación a CSV. |
| **Seguridad e higiene digital** | Simulación de archivos temporales dejados por clientes y su **borrado seguro** (sobrescritura aleatoria de 3 pasadas antes de eliminar), con bitácora de auditoría. |
| **Usuarios** | Gestión de cuentas y roles (solo Administrador/Superadministrador). |

### Roles y permisos (RBAC)

- **Superadministrador**: acceso total a todos los módulos.
- **Administrador**: gestiona inventario, proveedores, compras y usuarios (excepto crear otros superadministradores).
- **Cajero**: opera POS, impresiones, recargas y caja chica; no administra usuarios ni catálogo.
- **Auditor / Invitado**: acceso de solo lectura a todo el sistema (no puede registrar ni modificar nada).

---

## 6. Cómo cumple el sistema con el documento del proyecto

| Requisito del documento | Cómo se resuelve |
|---|---|
| **RF-01** — Registrar ventas descontando inventario automáticamente | Módulo POS + lógica de Kardex (`src/utils/kardex.js`) |
| **RF-02** — Contador físico inicial/final de fotocopiadoras para validar cobros | Módulo de Servicios de Impresión, con conciliación y alertas de discrepancia |
| **RF-03** — Gestión de proveedores, facturas de compra e IVA soportado | Módulo de Inventario → Proveedores y Órdenes de compra (campo IVA %) |
| **RF-04** — Borrado irrecuperable de archivos temporales de impresión | Módulo de Seguridad Digital (sobrescritura de 3 pasadas + eliminación) |
| **RNF-01** — Transacción POS en máx. 3 segundos | La venta se procesa localmente contra SQLite en milisegundos |
| **RNF-02** — Aislamiento READ COMMITTED / sin lecturas sucias | La base de datos corre en modo `WAL`, que garantiza que los lectores nunca ven datos no confirmados (equivalente o superior a READ COMMITTED). Al migrar a PostgreSQL/SQL Server, el nivel se configura explícitamente. |
| **RNF-03** — Diseño responsive desde 1024×768 | CSS con grid fluido y *breakpoints*; la barra lateral colapsa en pantallas angostas |

---

## 7. Arquitectura técnica

```
karina-cyber/
├── src/
│   ├── server.js            # Servidor Express principal
│   ├── db/
│   │   ├── schema.sql       # Esquema relacional (12 tablas)
│   │   ├── seed.js          # Datos iniciales (usuarios, catalogo, historial)
│   │   └── connection.js    # Conexion SQLite (WAL mode)
│   ├── middleware/auth.js   # JWT + control de acceso por rol (RBAC)
│   ├── routes/               # Un archivo de rutas REST por modulo
│   └── utils/kardex.js      # Logica de costo promedio ponderado
├── public/                   # Frontend (HTML + CSS + JS, sin frameworks)
│   ├── css/app.css           # Sistema de diseño propio
│   ├── js/app.js             # Cliente API, autenticacion, layout compartido
│   └── *.html                 # Una pagina por modulo
└── data/                      # Base de datos SQLite (se genera automaticamente)
```

**Stack:** Node.js + Express (API REST) · **SQLite mediante el módulo nativo
`node:sqlite`** (integrado en Node.js desde la version 22.5 — no requiere
instalar ningún motor de base de datos aparte, ni compilar nada en tu
computadora) · JWT + bcrypt para autenticación · HTML/CSS/JS nativo en el
frontend con Bootstrap 5 y Chart.js (empaquetados localmente, sin depender
de CDNs externos una vez instalado).

> **Nota:** al ejecutar `npm start` es normal ver una línea amarilla que
> dice `ExperimentalWarning: SQLite is an experimental feature...`. No es
> un error — es un aviso informativo de Node.js porque este módulo nativo
> es relativamente nuevo. El sistema funciona con total normalidad.

> **Nota académica:** el documento del proyecto sugiere SQL Server o
> PostgreSQL para la capa de datos. Se optó por SQLite para esta entrega
> porque permite que el sistema sea 100% autocontenido y se pueda instalar
> y ejecutar en cualquier computadora con un solo comando, sin instalar ni
> configurar un motor de base de datos externo. El diseño está en 3ra Forma
> Normal y usa el mismo modelo relacional (tablas, llaves foráneas,
> transacciones ACID), por lo que migrar a PostgreSQL o SQL Server más
> adelante implicaría cambiar únicamente la capa de conexión
> (`src/db/connection.js`), no la lógica de negocio.

---

## 8. Solución de problemas

- **`npm : No se puede cargar el archivo... la ejecución de scripts está deshabilitada`** → Estás en PowerShell, que bloquea scripts por defecto. Usa el **Símbolo del sistema (CMD)** en su lugar (búscalo en el menú inicio), o ejecuta una sola vez en PowerShell: `Set-ExecutionPolicy -ExecutionPolicy RemoteSigned -Scope CurrentUser`.
- **"npm no se reconoce como comando"** → Node.js no está instalado o no se
  reinició la terminal después de instalarlo. Reinstala desde nodejs.org.
- **"Missing script: start"** → Estás parado en la carpeta equivocada (una
  carpeta arriba de donde está `package.json`). Verifica con `dir` y haz
  `cd` a la subcarpeta correcta antes de correr `npm start`.
- **El puerto 3000 ya está en uso** → Copia el archivo `.env.example` como
  `.env` y cambia `PORT=3000` por otro número, por ejemplo `PORT=3001`.
- **Quiero cambiar la clave secreta de sesión** → Edita `JWT_SECRET` en el
  archivo `.env` (crea uno a partir de `.env.example` si no existe).
