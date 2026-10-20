# DEVELOPMENT PLAN: Sistema de Gestión de Alquileres Construcción

## 1. ARCHITECTURE OVERVIEW

### Components
- **Frontend**: React 18.2.x + TypeScript 5.x + Vite 5.x (puerto 3000)
- **Backend**: FastAPI 0.109.x + Python 3.11+ + SQLAlchemy 2.x + SQLite (puerto 8000)
- **Shared**: TypeScript interfaces y Python Pydantic models para контракт данных

### Data Models
- **Cliente**: Empresa con información de contacto
- **Equipo**: Andamios/equipos con estados (disponible, alquilado, mantenimiento, baja)
- **Alquiler**: Relación many-to-many entre Cliente y Equipos con fechas
- **Notificación**: Alertas para vencimiento de devoluciones

### API Endpoints
| Domain | Endpoints |
|--------|-----------|
| Clientes | CRUD completo en `/api/v1/clientes` |
| Equipos | CRUD + `/api/v1/equipos/disponibles` |
| Alquileres | CRUD + `/api/v1/alquileres/{id}/devolver` + `/api/v1/alquileres/{id}/entregar` |
| Notificaciones | GET + PUT marcar leída |
| Dashboard | `/api/v1/dashboard/resumen` |
| Health | `/api/v1/health` |

### Folder Structure
```
sistema-alquileres/
├── shared/
│   ├── types.ts          # TypeScript interfaces y enums
│   └── models.py         # Pydantic models (backend)
├── backend/
│   ├── app/
│   │   ├── main.py       # FastAPI app
│   │   ├── database.py   # SQLAlchemy setup
│   │   ├── models/       # SQLAlchemy models
│   │   ├── schemas/      # Pydantic schemas
│   │   ├── routers/      # API routes
│   │   └── services/     # Business logic
│   ├── requirements.txt
│   └── Dockerfile
├── frontend/
│   ├── src/
│   │   ├── components/ui/  # 8 componentes base
│   │   ├── pages/           # 9 páginas
│   │   ├── hooks/           # 4 hooks
│   │   ├── services/        # API services
│   │   ├── styles/          # tokens.ts, global.css
│   │   └── layout/          # Layout, Header, Sidebar
│   ├── package.json
│   └── Dockerfile
├── docker-compose.yml
├── run.sh
└── README.md
```

## 2. ACCEPTANCE CRITERIA

1. TC001: Registrar un nuevo alquiler de equipo exitosamente. → Se deben registrar qué equipos salen a cada cliente y avisar cuándo vencen las devoluciones. El sistema confirma el registro y el equipo seleccionado cambia su estado a 'alquilado'.
2. TC002: Intentar alquilar un equipo que ya está registrado como alquilado. → Un mismo equipo no se puede alquilar a dos clientes al mismo tiempo. El sistema debe mostrar un mensaje de error claro indicando que el equipo no está disponible para alquiler.
3. TC003: Registrar la devolución de un equipo alquilado exitosamente. → El sistema actualiza el estado del 'Equipo Y' a 'disponible' y registra la fecha y hora de devolución correctamente.
4. TC004: Verificar notificación de alquileres que vencen en los próximos 3 días para oficina. → Se recibe una notificación (email/in-app) al empleado de oficina, informando sobre el alquiler que vence en los próximos 3 días.
5. TC005: Verificar notificación de alquileres vencidos para empleados de depósito. → Se recibe una notificación (email/in-app) al empleado de depósito, informando sobre el alquiler cuya fecha de devolución ha pasado.
6. TC006: Validar campos obligatorios al registrar un nuevo alquiler. → El sistema debe mostrar un mensaje de error claro para cada campo obligatorio no completado y no permitir el registro del alquiler hasta que todos los campos requeridos sean proporcionados.
7. TC007: Intentar registrar un alquiler con una fecha de devolución pasada. → El sistema debe mostrar un mensaje de error indicando que la fecha de devolución debe ser futura y no permitir el registro del alquiler.
8. TC008: Intentar registrar la devolución de un equipo no alquilado. → El sistema debe mostrar un mensaje de error indicando que el equipo no está alquilado y no permitir el registro de la devolución.
9. TC009: Verificar que un alquiler devuelto no genere notificaciones de vencimiento. → No se deben recibir notificaciones (email/in-app) para el 'Equipo A' ni para el empleado de oficina ni para el de depósito, ya que el equipo ha sido devuelto.
10. NFR001: Tiempo de respuesta al registrar un alquiler bajo carga concurrente. → El registro de un nuevo alquiler debe completarse en menos de 3 segundos para el 90% de las transacciones.
11. NFR002: Tiempo de procesamiento para la generación de notificaciones de vencimiento. → El proceso de identificación y preparación de notificaciones debe completarse en menos de 5 segundos.
12. NFR003: Verificación del control de acceso basado en roles (oficina vs. depósito). → El 'Empleado de Depósito' no debe poder acceder a 'Registrar nuevo alquiler'. El 'Empleado de Oficina' no debe poder acceder a 'Registrar devolución de equipo'.
13. NFR004: Asegurar la protección de datos sensibles de clientes y equipos. → Los datos sensibles deben estar protegidos en tránsito (HTTPS) y en reposo (encriptación en base de datos).
14. NFR005: Comprobación de la disponibilidad continua del sistema bajo carga moderada. → El sistema debe permanecer disponible y funcional durante todo el período de prueba, con un tiempo de actividad mínimo del 99.9%.
15. NFR006: Verificación del manejo de errores al intentar registrar devoluciones inválidas. → El sistema debe mostrar un mensaje de error claro y no permitir la operación inválida, manteniendo la integridad de los datos.
16. NFR007: Comportamiento del sistema con un número elevado de usuarios concurrentes. → El sistema debe mantener tiempos de respuesta aceptables (ej. < 5 segundos para el 90% de las transacciones) y un uso de recursos estable hasta 20 usuarios concurrentes, con degradación controlada más allá de este…

## 3. EXECUTABLE ITEMS

### ITEM 1: Foundation — shared types, models, database, config
**Goal:** Crear toda la configuración compartida y contratos de datos que el resto de módulos importarán. Incluye tipos TypeScript, modelos Pydantic, modelos SQLAlchemy, configuración de base de datos y tokens de diseño.

**Files to create:**
- shared/types.ts (create) - Todas las interfaces TypeScript y enums del sistema (Cliente, Equipo, Alquiler, Notificacion, filtros, respuestas paginadas)
- shared/models.py (create) - Modelos Pydantic base para el backend (ClienteBase, EquipoBase, AlquilerBase, NotificacionBase, enums EstadoEquipo, EstadoAlquiler, TipoNotificacion)
- shared/__init__.py (create) - Exports vacíos para shared
- backend/app/database.py (create) - Configuración SQLAlchemy (engine, SessionLocal, get_db), Base declarative
- backend/app/models/__init__.py (create) - Exports de todos los modelos SQLAlchemy
- backend/app/models/cliente.py (create) - Modelo SQLAlchemy Cliente con todos los campos
- backend/app/models/equipo.py (create) - Modelo SQLAlchemy Equipo con enum EstadoEquipo
- backend/app/models/alquiler.py (create) - Modelos SQLAlchemy Alquiler y AlquilerEquipo (relación many-to-many)
- backend/app/models/notificacion.py (create) - Modelo SQLAlchemy Notificacion
- backend/app/schemas/__init__.py (create) - Exports de todos los schemas Pydantic
- backend/app/schemas/cliente.py (create) - ClienteBase, ClienteCreate, ClienteUpdate, Cliente
- backend/app/schemas/equipo.py (create) - EquipoBase, EquipoCreate, EquipoUpdate, Equipo, EquipoFilters
- backend/app/schemas/alquiler.py (create) - AlquilerBase, AlquilerCreate, AlquilerUpdate, Alquiler, AlquilerDetalle, AlquilerFilters, AlquilerEquipoBase
- backend/app/schemas/notificacion.py (create) - NotificacionBase, NotificacionCreate, NotificacionUpdate, Notificacion, TipoNotificacion
- backend/requirements.txt (create) - fastapi==0.109.2, uvicorn[standard]==0.27.1, sqlalchemy==2.0.25, pydantic==2.6.1, pydantic-settings==2.1.0, python-multipart==0.0.9, aiosqlite==0.19.0
- frontend/src/styles/tokens.ts (create) - Design tokens verbatim del Figma (colors, typography, spacing, shadows, etc.)
- frontend/src/types/index.ts (create) - Re-export de shared/types.ts para el frontend
- frontend/src/styles/global.css (create) - Estilos globales con variables CSS basadas en tokens
- frontend/src/index.css (create) - Reset CSS y fuentes (Inter font)

**Dependencies:** Ninguna

**Validation:** 
- Backend: `python -c "from app.models import Cliente, Equipo, Alquiler, Notificacion; from app.schemas import ClienteCreate; print('Models OK')"`
- Frontend: `npx tsc --noEmit src/types/index.ts` sin errores

**Role:** role-tl (technical_lead)

---

### ITEM 2: Backend — API Routers (clientes, equipos, alquileres)
**Goal:** Implementar routers FastAPI completos para las entidades principales: Clientes, Equipos y Alquileres. Cada router incluye CRUD completo con validación Pydantic, paginación, filtros y reglas de negocio (no alquiler doble, actualización de estados de equipos).

**Files to create:**
- backend/app/routers/__init__.py (create) - Exports de todos los routers
- backend/app/routers/clientes.py (create) - Router CRUD para /clientes con ClienteFilters (buscar por nombre_empresa)
- backend/app/routers/equipos.py (create) - Router CRUD para /equipos + GET /equipos/disponibles con EquipoFilters (estado, buscar)
- backend/app/routers/alquileres.py (create) - Router CRUD para /alquileres + POST /devolver + POST /entregar con AlquilerFilters, validación de no solapamiento de alquileres, actualización automática de estado de equipos

**Dependencies:** Item 1

**Validation:** `curl http://localhost:8000/api/v1/health` retorna `{"status":"ok"}`

**Role:** role-be (backend_developer)

---

### ITEM 3: Backend — Notification service y Dashboard
**Goal:** Implementar el servicio de notificaciones que detecta alquileres próximos a vencer (3 días) y vencidos, generando notificaciones automáticamente. También el endpoint de dashboard con resumen del sistema.

**Files to create:**
- backend/app/services/__init__.py (create) - Exports del servicio
- backend/app/services/notification_service.py (create) - NotificationService con métodos: check_due_returns() (detecta alquileres que vencen en 3 días), check_overdue() (detecta alquileres vencidos), create_notification(), mark_as_read()
- backend/app/routers/notificaciones.py (create) - Router para /notificaciones con GET, PUT marcar leída, PUT marcar todas leídas, GET /vencimientos
- backend/app/routers/dashboard.py (create) - Router para /dashboard/resumen con métricas (total_alquileres_activos, total_equipos, equipos_disponibles, equipos_alquilados, notificaciones_pendientes, alquileres_vencidos, proximas_devoluciones)
- backend/app/main.py (create) - FastAPI app con lifespan (create tables + seed si vacío), CORS middleware, includes de todos los routers, health endpoint

**Dependencies:** Item 1

**Validation:** `curl http://localhost:8000/api/v1/dashboard/resumen` retorna JSON con todas las métricas

**Role:** role-be (backend_developer)

---

### ITEM 4: Frontend — Base UI Components
**Goal:** Implementar los 8 componentes base del sistema de diseño según el contrato UI/UX: Button, Card, InputField, DataTable, Modal, DatePicker, Alert, Navigation. Cada componente debe usar los tokens de diseño verbatim y soportar todas las props definidas en SPEC.md.

**Files to create:**
- frontend/src/components/ui/Button.tsx (create) - Variants: primary/secondary/danger/outline, sizes: sm/md/lg, fullWidth
- frontend/src/components/ui/Card.tsx (create) - title, children, footer, padding variants
- frontend/src/components/ui/InputField.tsx (create) - label, name, value, onChange, type, error, required, disabled
- frontend/src/components/ui/DataTable.tsx (create) - Generic Table con columns config, pagination, onRowClick, loading, emptyMessage
- frontend/src/components/ui/Modal.tsx (create) - isOpen, onClose, title, children, footer, size variants (sm/md/lg/xl)
- frontend/src/components/ui/DatePicker.tsx (create) - label, name, value, onChange, minDate, maxDate, error, required, disabled
- frontend/src/components/ui/Alert.tsx (create) - type: success/error/warning/info, message, onClose, dismissible
- frontend/src/components/ui/Navigation.tsx (create) - activePath, onNavigate, notificationCount, links: Alquileres, Clientes, Equipos, Notificaciones, Dashboard

**Dependencies:** Item 1

**Validation:** `npm run build` en frontend completa sin errores de TypeScript

**Role:** role-fe (frontend_developer)

---

### ITEM 5: Frontend — Layout (Header, Sidebar, Layout wrapper)
**Goal:** Implementar el layout principal de la aplicación con Header (brand, alert badge, user profile) y Sidebar (navegación persistente). Incluir componentes de Layout que envuelven todas las páginas.

**Files to create:**
- frontend/src/layout/Header.tsx (create) - Brand "Alquileres Construcción", alert summary badge con contador de notificaciones pendientes, user profile avatar
- frontend/src/layout/Sidebar.tsx (create) - Navegación lateral con links a: Gestión de Alquileres, Clientes, Catálogo Equipos, Notificaciones, Dashboard, Recepción/Entrega
- frontend/src/layout/Layout.tsx (create) - Wrapper principal que combina Header + Sidebar + contenido children

**Dependencies:** Item 1, Item 4

**Validation:** Componentes renderizan sin errores en desarrollo

**Role:** role-fe (frontend_developer)

---

### ITEM 6: Frontend — Páginas principales Figma (Alquileres Lista, Notificaciones, Clientes Lista)
**Goal:** Implementar las 3 páginas principales del contrato UI/UX de Figma con pixel-perfect fidelity: Gestión de Alquileres Lista (node_id 3:2), Sistema de Notificaciones (node_id 4:2), Gestión de Clientes Lista (node_id 5:2). Cada página incluye filtros, DataTable con datos reales del API, y botones CTA.

**Files to create:**
- frontend/src/pages/GestionAlquileresLista.tsx (create) - Lista de alquileres con filtros (estado, cliente, fechas), DataTable con columnas: cliente, equipos, fecha_inicio, fecha_devolucion_prevista, estado, acciones. Botón "Nuevo Alquiler". Click row navega a detalle.
- frontend/src/pages/SistemaNotificaciones.tsx (create) - Lista de notificaciones pendientes con color coding (vencido=red, proximo=warning). Cada notificación muestra: mensaje, fecha, tipo. Botón "Marcar todas como leídas".
- frontend/src/pages/GestionClientesLista.tsx (create) - Lista de clientes con búsqueda, DataTable con columnas: nombre_empresa, persona_contacto, telefono, email. Botón "Nuevo Cliente". Click row navega a detalle.

**Dependencies:** Item 1, Item 4, Item 5

**Validation:** `npm run dev` en frontend, las 3 páginas cargan correctamente con datos del API

**Role:** role-fe (frontend_developer)

---

### ITEM 7: Frontend — Páginas secundarias (Alquileres Detalle, Equipos, Dashboard)
**Goal:** Implementar páginas secundarias del sistema: Detalle de Alquiler (crear/editar), Catálogo de Equipos Lista/Detalle, Dashboard con resumen visual.

**Files to create:**
- frontend/src/pages/GestionAlquileresDetalle.tsx (create) - Formulario crear/editar alquiler: selector cliente (dropdown), selector equipos (multi-select con check de disponibilidad), fecha_inicio, fecha_devolucion_prevista, observaciones. Validación de campos obligatorios y fecha futura.
- frontend/src/pages/CatalogoEquiposLista.tsx (create) - Lista de equipos con filtros (estado), badges de estado con colores semánticos (disponible=green, alquilado=blue, mantenimiento=yellow, baja=red)
- frontend/src/pages/CatalogoEquiposDetalle.tsx (create) - Formulario crear/editar equipo: nombre, código, descripción, precio_diario, estado, observaciones
- frontend/src/pages/Dashboard.tsx (create) - Cards de resumen: total_alquileres_activos, equipos_disponibles, equipos_alquilados, notificaciones_pendientes, alquileres_vencidos, próximas devoluciones

**Dependencies:** Item 1, Item 4, Item 5, Item 6

**Validation:** Navegación entre páginas funciona correctamente, datos persisten via API

**Role:** role-fe (frontend_developer)

---

### ITEM 8: Frontend — Páginas restantes y Hooks
**Goal:** Implementar páginas restantes y todos los hooks personalizados que conectan el frontend con el API. Incluir página de Recepción/Entrega, Gestión de Clientes Detalle, y los 4 hooks (useClientes, useEquipos, useAlquileres, useNotificaciones).

**Files to create:**
- frontend/src/pages/RecepcionEntrega.tsx (create) - Interface para confirmar entrega física de equipos (POST /alquileres/{id}/entregar) y recepción (POST /alquileres/{id}/devolver)
- frontend/src/pages/GestionClientesDetalle.tsx (create) - Formulario crear/editar cliente con todos los campos
- frontend/src/hooks/useClientes.ts (create) - Hook: { clientes, loading, error, crearCliente, actualizarCliente, eliminarCliente, refetch }
- frontend/src/hooks/useEquipos.ts (create) - Hook: { equipos, loading, error, crearEquipo, actualizarEquipo, eliminarEquipo, refetch }
- frontend/src/hooks/useAlquileres.ts (create) - Hook: { alquileres, loading, error, crearAlquiler, actualizarAlquiler, eliminarAlquiler, marcarDevuelto, refetch }
- frontend/src/hooks/useNotificaciones.ts (create) - Hook: { notificaciones, loading, error, marcarLeida, marcarTodasLeidas, refetch }

**Dependencies:** Item 1, Item 4, Item 5

**Validation:** Hooks hacen fetch correcto al API y actualizan estado

**Role:** role-fe (frontend_developer)

---

### ITEM 9: Frontend — Services API y App setup
**Goal:** Implementar servicios API que encapsulan las llamadas fetch al backend, y configurar App.tsx con React Router para todas las rutas. Crear App.tsx, main.tsx, vite.config.ts, y todos los servicios API.

**Files to create:**
- frontend/src/services/api.ts (create) - apiFetch wrapper con base URL, headers, manejo de errores, timeout
- frontend/src/services/clientes.ts (create) - getClientes, getCliente, createCliente, updateCliente, deleteCliente
- frontend/src/services/equipos.ts (create) - getEquipos, getEquipo, createEquipo, updateEquipo, deleteEquipo, getEquiposDisponibles
- frontend/src/services/alquileres.ts (create) - getAlquileres, getAlquiler, createAlquiler, updateAlquiler, deleteAlquiler, marcarDevuelto, confirmarEntrega
- frontend/src/services/notificaciones.ts (create) - getNotificaciones, getNotificacionesPendientes, getVencimientos, markNotificacionLeida, markAllLeidas
- frontend/src/App.tsx (create) - Router con todas las rutas: /alquileres, /alquileres/nuevo, /alquileres/:id, /clientes, /clientes/:id, /equipos, /equipos/:id, /notificaciones, /dashboard, /recepcion-entrega
- frontend/src/main.tsx (create) - Entry point React con AppProvider
- frontend/vite.config.ts (create) - Configuración Vite con proxy al backend
- frontend/index.html (create) - HTML entry point
- frontend/package.json (create) - Dependencias: react, react-dom, react-router-dom, date-fns, typescript, vite

**Dependencies:** Item 1, Item 8

**Validation:** `npm run build` genera dist/ con todos los chunks correctamente

**Role:** role-fe (frontend_developer)

---

### ITEM 10: Backend — Docker setup
**Goal:** Crear Dockerfile para el backend con multi-stage build, instalación de dependencias, usuario no-root, y configuración de startup. Incluir script de seed para datos iniciales.

**Files to create:**
- backend/Dockerfile (create) - Multi-stage: builder (python:3.11-slim) install deps, runner stage con uvicorn. EXPOSE 8000, CMD ["uvicorn", "app.main:app"]
- backend/.dockerignore (create) - Exclude __pycache__, .git, *.pyc, .env

**Dependencies:** Item 1, Item 2, Item 3

**Validation:** `docker build -t backend-alquileres ./backend` completa sin errores

**Role:** role-devops (devops_support)

---

### ITEM 11: Frontend — Docker setup
**Goal:** Crear Dockerfile para el frontend con multi-stage build (Node.js build + nginx serve). Configurar nginx para servir la SPA y hacer proxy del API.

**Files to create:**
- frontend/Dockerfile (create) - Multi-stage: builder (node:20) npm install, npm run build, runner (nginx:alpine) con configuración nginx
- frontend/.dockerignore (create) - Exclude node_modules, .git, dist, *.log

**Dependencies:** Item 1, Item 4, Item 5, Item 6, Item 7, Item 8, Item 9

**Validation:** `docker build -t frontend-alquileres ./frontend` completa sin errores

**Role:** role-devops (devops_support)

---

### ITEM 12: Infrastructure — Docker Compose y documentación
**Goal:** Crear docker-compose.yml completo con ambos servicios, healthchecks, depends_on correcto, red compartida. Documentación y scripts de ejecución.

**Files to create:**
- docker-compose.yml (create) - Servicios: backend (puertos 8000), frontend (puertos 3000), networks, healthchecks con curl, depends_on con condition: service_healthy
- .env.example (create) - Variables: DATABASE_URL, CORS_ORIGINS, DEBUG, VITE_API_BASE_URL con descripciones
- .gitignore (create) - node_modules, dist, __pycache__, *.pyc, .env, *.db, .venv
- .dockerignore (create) - .git, node_modules, backend/__pycache__, *.pyc
- run.sh (create) - Script que valida Docker, construye imágenes, inicia servicios, espera healthchecks, imprime URLs de acceso
- README.md (create) - Descripción del proyecto, requisitos (Docker, Node 20, Python 3.11), instalación rápida (./run.sh), endpoints API documentados

**Dependencies:** Item 10, Item 11

**Validation:** `./run.sh` completa sin errores, `curl http://localhost:8000/api/v1/health` retorna OK, `curl http://localhost:3000` retorna HTML del frontend

**Role:** role-devops (devops_support)