# SPEC.md

## Sistema de Gestión de Alquileres de Construcción

---

## 1. TECHNOLOGY STACK

### Frontend
- **Framework**: React 18.2.x with TypeScript 5.x
- **Build Tool**: Vite 5.x
- **Routing**: React Router DOM 6.x
- **State Management**: React hooks (useState, useEffect, useContext)
- **Date Handling**: date-fns 3.x
- **HTTP Client**: fetch API (native)
- **Styling**: CSS Modules + Design Tokens (no Tailwind)
- **Package Manager**: npm

### Backend
- **Framework**: FastAPI 0.109.x
- **Language**: Python 3.11+
- **Database**: SQLite with SQLAlchemy 2.x ORM
- **Validation**: Pydantic 2.x
- **Date Handling**: Python datetime (stdlib)
- **CORS**: FastAPI CORS middleware

### Development
- **Node.js**: 20.x LTS
- **Python**: 3.11+

---

## 2. DATA CONTRACTS

### Python Pydantic Models (Backend)

```python
from pydantic import BaseModel, Field, ConfigDict
from datetime import datetime, date
from typing import Optional, List
from enum import Enum

class EstadoEquipo(str, Enum):
    DISPONIBLE = "disponible"
    ALQUILADO = "alquilado"
    MANTENIMIENTO = "mantenimiento"
    BAJA = "baja"

class EstadoAlquiler(str, Enum):
    ACTIVO = "activo"
    FINALIZADO = "finalizado"
    VENCIDO = "vencido"

class TipoNotificacion(str, Enum):
    VENCIMIENTO_PROXIMO = "vencimiento_proximo"
    VENCIDO = "vencido"
    RECORDATORIO = "recordatorio"

# Cliente
class ClienteBase(BaseModel):
    nombre_empresa: str = Field(..., min_length=1, max_length=200)
    razon_social: Optional[str] = Field(None, max_length=200)
    rut_cuit: Optional[str] = Field(None, max_length=20)
    persona_contacto: Optional[str] = Field(None, max_length=200)
    telefono: Optional[str] = Field(None, max_length=50)
    email: Optional[str] = Field(None, max_length=200)
    direccion: Optional[str] = Field(None, max_length=500)
    observaciones: Optional[str] = Field(None, max_length=1000)

class ClienteCreate(ClienteBase):
    pass

class ClienteUpdate(BaseModel):
    nombre_empresa: Optional[str] = Field(None, min_length=1, max_length=200)
    razon_social: Optional[str] = Field(None, max_length=200)
    rut_cuit: Optional[str] = Field(None, max_length=20)
    persona_contacto: Optional[str] = Field(None, max_length=200)
    telefono: Optional[str] = Field(None, max_length=50)
    email: Optional[str] = Field(None, max_length=200)
    direccion: Optional[str] = Field(None, max_length=500)
    observaciones: Optional[str] = Field(None, max_length=1000)

class Cliente(ClienteBase):
    model_config = ConfigDict(from_attributes=True)
    
    id: int
    fecha_creacion: datetime
    fecha_actualizacion: datetime

# Equipo
class EquipoBase(BaseModel):
    nombre: str = Field(..., min_length=1, max_length=200)
    codigo: str = Field(..., min_length=1, max_length=50)
    descripcion: Optional[str] = Field(None, max_length=1000)
    estado: EstadoEquipo = EstadoEquipo.DISPONIBLE
    precio_diario: float = Field(..., ge=0)
    observaciones: Optional[str] = Field(None, max_length=1000)

class EquipoCreate(EquipoBase):
    pass

class EquipoUpdate(BaseModel):
    nombre: Optional[str] = Field(None, min_length=1, max_length=200)
    codigo: Optional[str] = Field(None, min_length=1, max_length=50)
    descripcion: Optional[str] = Field(None, max_length=1000)
    estado: Optional[EstadoEquipo] = None
    precio_diario: Optional[float] = Field(None, ge=0)
    observaciones: Optional[str] = Field(None, max_length=1000)

class Equipo(EquipoBase):
    model_config = ConfigDict(from_attributes=True)
    
    id: int
    fecha_creacion: datetime
    fecha_actualizacion: datetime

# Alquiler
class AlquilerBase(BaseModel):
    cliente_id: int
    fecha_inicio: date
    fecha_devolucion_prevista: date
    estado: EstadoAlquiler = EstadoAlquiler.ACTIVO
    observaciones: Optional[str] = Field(None, max_length=1000)

class AlquilerCreate(AlquilerBase):
    equipos_ids: List[int] = Field(..., min_length=1)

class AlquilerUpdate(BaseModel):
    fecha_inicio: Optional[date] = None
    fecha_devolucion_prevista: Optional[date] = None
    fecha_devolucion_real: Optional[date] = None
    estado: Optional[EstadoAlquiler] = None
    observaciones: Optional[str] = Field(None, max_length=1000)

class AlquilerEquipoBase(BaseModel):
    equipo_id: int
    cantidad: int = Field(1, ge=1)

class Alquiler(AlquilerBase):
    model_config = ConfigDict(from_attributes=True)
    
    id: int
    fecha_creacion: datetime
    fecha_actualizacion: datetime
    equipos: List[Equipo] = []

class AlquilerDetalle(Alquiler):
    cliente: Cliente

# Notificación
class NotificacionBase(BaseModel):
    alquiler_id: int
    tipo: TipoNotificacion
    mensaje: str = Field(..., max_length=500)
    leida: bool = False

class NotificacionCreate(NotificacionBase):
    pass

class NotificacionUpdate(BaseModel):
    leida: bool = False

class Notificacion(NotificacionBase):
    model_config = ConfigDict(from_attributes=True)
    
    id: int
    fecha_creacion: datetime
    fecha_actualizacion: datetime

# Filters/Queries
class AlquilerFilters(BaseModel):
    cliente_id: Optional[int] = None
    estado: Optional[EstadoAlquiler] = None
    fecha_inicio_desde: Optional[date] = None
    fecha_inicio_hasta: Optional[date] = None
    buscar: Optional[str] = None

class ClienteFilters(BaseModel):
    buscar: Optional[str] = None

class EquipoFilters(BaseModel):
    estado: Optional[EstadoEquipo] = None
    buscar: Optional[str] = None
```

### TypeScript Interfaces (Frontend)

```typescript
// Enums
enum EstadoEquipo {
  DISPONIBLE = "disponible",
  ALQUILADO = "alquilado",
  MANTENIMIENTO = "mantenimiento",
  BAJA = "baja"
}

enum EstadoAlquiler {
  ACTIVO = "activo",
  FINALIZADO = "finalizado",
  VENCIDO = "vencido"
}

enum TipoNotificacion {
  VENCIMIENTO_PROXIMO = "vencimiento_proximo",
  VENCIDO = "vencido",
  RECORDATORIO = "recordatorio"
}

// Cliente
interface ClienteBase {
  nombre_empresa: string;
  razon_social?: string;
  rut_cuit?: string;
  persona_contacto?: string;
  telefono?: string;
  email?: string;
  direccion?: string;
  observaciones?: string;
}

interface ClienteCreate extends ClienteBase {}

interface ClienteUpdate extends Partial<ClienteBase> {}

interface Cliente extends ClienteBase {
  id: number;
  fecha_creacion: string;
  fecha_actualizacion: string;
}

// Equipo
interface EquipoBase {
  nombre: string;
  codigo: string;
  descripcion?: string;
  estado: EstadoEquipo;
  precio_diario: number;
  observaciones?: string;
}

interface EquipoCreate extends EquipoBase {}

interface EquipoUpdate extends Partial<EquipoBase> {}

interface Equipo extends EquipoBase {
  id: number;
  fecha_creacion: string;
  fecha_actualizacion: string;
}

// Alquiler
interface AlquilerBase {
  cliente_id: number;
  fecha_inicio: string; // YYYY-MM-DD
  fecha_devolucion_prevista: string; // YYYY-MM-DD
  estado: EstadoAlquiler;
  observaciones?: string;
}

interface AlquilerCreate extends Omit<AlquilerBase, 'estado'> {
  equipos_ids: number[];
}

interface AlquilerUpdate extends Partial<AlquilerBase> {
  fecha_devolucion_real?: string;
}

interface Alquiler extends AlquilerBase {
  id: number;
  fecha_creacion: string;
  fecha_actualizacion: string;
  equipos: Equipo[];
}

interface AlquilerDetalle extends Alquiler {
  cliente: Cliente;
}

// Notificación
interface NotificacionBase {
  alquiler_id: number;
  tipo: TipoNotificacion;
  mensaje: string;
  leida: boolean;
}

interface NotificacionCreate extends NotificacionBase {}

interface Notificacion extends NotificacionBase {
  id: number;
  fecha_creacion: string;
  fecha_actualizacion: string;
}

// Filters
interface AlquilerFilters {
  cliente_id?: number;
  estado?: EstadoAlquiler;
  fecha_inicio_desde?: string;
  fecha_inicio_hasta?: string;
  buscar?: string;
}

interface ClienteFilters {
  buscar?: string;
}

interface EquipoFilters {
  estado?: EstadoEquipo;
  buscar?: string;
}

// API Response types
interface PaginatedResponse<T> {
  items: T[];
  total: number;
  page: number;
  page_size: number;
  pages: number;
}

interface ApiError {
  detail: string;
}
```

---

## 3. API ENDPOINTS

### Base URL: `/api/v1`

#### Clientes

| Method | Path | Description | Request Body | Response |
|--------|------|-------------|--------------|----------|
| GET | `/clientes` | Listar clientes con filtros | `ClienteFilters` (query) | `PaginatedResponse<Cliente>` |
| GET | `/clientes/{id}` | Obtener cliente por ID | - | `Cliente` |
| POST | `/clientes` | Crear nuevo cliente | `ClienteCreate` | `Cliente` |
| PUT | `/clientes/{id}` | Actualizar cliente | `ClienteUpdate` | `Cliente` |
| DELETE | `/clientes/{id}` | Eliminar cliente | - | `204 No Content` |

#### Equipos

| Method | Path | Description | Request Body | Response |
|--------|------|-------------|--------------|----------|
| GET | `/equipos` | Listar equipos con filtros | `EquipoFilters` (query) | `PaginatedResponse<Equipo>` |
| GET | `/equipos/{id}` | Obtener equipo por ID | - | `Equipo` |
| POST | `/equipos` | Crear nuevo equipo | `EquipoCreate` | `Equipo` |
| PUT | `/equipos/{id}` | Actualizar equipo | `EquipoUpdate` | `Equipo` |
| DELETE | `/equipos/{id}` | Eliminar equipo | - | `204 No Content` |
| GET | `/equipos/disponibles` | Listar equipos disponibles para alquiler | - | `Equipo[]` |

#### Alquileres

| Method | Path | Description | Request Body | Response |
|--------|------|-------------|--------------|----------|
| GET | `/alquileres` | Listar alquileres con filtros | `AlquilerFilters` (query) | `PaginatedResponse<Alquiler>` |
| GET | `/alquileres/{id}` | Obtener alquiler por ID con detalles | - | `AlquilerDetalle` |
| POST | `/alquileres` | Crear nuevo alquiler | `AlquilerCreate` | `Alquiler` |
| PUT | `/alquileres/{id}` | Actualizar alquiler | `AlquilerUpdate` | `Alquiler` |
| DELETE | `/alquileres/{id}` | Eliminar alquiler | - | `204 No Content` |
| POST | `/alquileres/{id}/devolver` | Marcar alquiler como devuelto | `{ fecha_devolucion_real: string }` | `Alquiler` |
| POST | `/alquileres/{id}/entregar` | Confirmar entrega de equipos | - | `Alquiler` |

#### Notificaciones

| Method | Path | Description | Request Body | Response |
|--------|------|-------------|--------------|----------|
| GET | `/notificaciones` | Listar notificaciones | - | `Notificacion[]` |
| GET | `/notificaciones/pendientes` | Listar notificaciones no leídas | - | `Notificacion[]` |
| GET | `/notificaciones/vencimientos` | Listar alquileres próximos a vencer | - | `Alquiler[]` |
| PUT | `/notificaciones/{id}` | Marcar notificación como leída | `NotificacionUpdate` | `Notificacion` |
| PUT | `/notificaciones/marcar-todas-leidas` | Marcar todas como leídas | - | `204 No Content` |

#### Dashboard

| Method | Path | Description | Request Body | Response |
|--------|------|-------------|--------------|----------|
| GET | `/dashboard/resumen` | Resumen general del sistema | - | `{ total_alquileres_activos, total_equipos, equipos_disponibles, equipos_alquilados, notificaciones_pendientes, alquileres_vencidos, proximas_devoluciones }` |

#### Health Check

| Method | Path | Description | Response |
|--------|------|-------------|----------|
| GET | `/health` | Health check endpoint | `{ status: "ok", timestamp: string }` |

---

## 4. FILE STRUCTURE

```
sistema-alquileres/
├── frontend/
│   ├── src/
│   │   ├── components/
│   │   │   └── ui/
│   │   │       ├── Navigation.tsx
│   │   │       ├── Button.tsx
│   │   │       ├── Card.tsx
│   │   │       ├── InputField.tsx
│   │   │       ├── DataTable.tsx
│   │   │       ├── Modal.tsx
│   │   │       ├── DatePicker.tsx
│   │   │       └── Alert.tsx
│   │   ├── pages/
│   │   │   ├── GestionAlquileresLista.tsx
│   │   │   ├── GestionAlquileresDetalle.tsx
│   │   │   ├── GestionClientesLista.tsx
│   │   │   ├── GestionClientesDetalle.tsx
│   │   │   ├── CatalogoEquiposLista.tsx
│   │   │   ├── CatalogoEquiposDetalle.tsx
│   │   │   ├── SistemaNotificaciones.tsx
│   │   │   ├── Dashboard.tsx
│   │   │   └── RecepcionEntrega.tsx
│   │   ├── services/
│   │   │   ├── api.ts
│   │   │   ├── clientes.ts
│   │   │   ├── equipos.ts
│   │   │   ├── alquileres.ts
│   │   │   └── notificaciones.ts
│   │   ├── hooks/
│   │   │   ├── useClientes.ts
│   │   │   ├── useEquipos.ts
│   │   │   ├── useAlquileres.ts
│   │   │   └── useNotificaciones.ts
│   │   ├── types/
│   │   │   └── index.ts
│   │   ├── styles/
│   │   │   ├── tokens.ts
│   │   │   ├── global.css
│   │   │   └── components/
│   │   ├── layout/
│   │   │   ├── Layout.tsx
│   │   │   ├── Header.tsx
│   │   │   └── Sidebar.tsx
│   │   ├── App.tsx
│   │   ├── main.tsx
│   │   └── index.css
│   ├── index.html
│   ├── package.json
│   ├── tsconfig.json
│   ├── vite.config.ts
│   └── .env.example
├── backend/
│   ├── app/
│   │   ├── __init__.py
│   │   ├── main.py
│   │   ├── database.py
│   │   ├── models/
│   │   │   ├── __init__.py
│   │   │   ├── cliente.py
│   │   │   ├── equipo.py
│   │   │   ├── alquiler.py
│   │   │   └── notificacion.py
│   │   ├── schemas/
│   │   │   ├── __init__.py
│   │   │   ├── cliente.py
│   │   │   ├── equipo.py
│   │   │   ├── alquiler.py
│   │   │   └── notificacion.py
│   │   ├── routers/
│   │   │   ├── __init__.py
│   │   │   ├── clientes.py
│   │   │   ├── equipos.py
│   │   │   ├── alquileres.py
│   │   │   ├── notificaciones.py
│   │   │   └── dashboard.py
│   │   └── services/
│   │       ├── __init__.py
│   │       └── notification_service.py
│   ├── requirements.txt
│   └── Dockerfile
├── docker-compose.yml
└── README.md
```

### PORT TABLE

| Service | Listening Port | Path |
|---|---|---|
| backend | 8000 | backend/ |
| frontend | 3000 | frontend/ |

### SHARED MODULES

No shared modules between backend and frontend. The frontend uses its own types defined in `frontend/src/types/index.ts` which mirrors the backend Pydantic models.

---

## 5. ENVIRONMENT VARIABLES

### Frontend (.env.example)

```env
VITE_API_BASE_URL=http://localhost:8000/api/v1
```

### Backend (.env.example)

```env
DATABASE_URL=sqlite:///./alquileres.db
CORS_ORIGINS=http://localhost:3000,http://localhost:5173
DEBUG=true
NOTIFICATION_CHECK_HOURS=24
NOTIFICATION_DAYS_BEFORE=3
```

---

## 6. IMPORT CONTRACTS

### Backend

**database.py exports:**
- `engine`, `SessionLocal`, `get_db`

**models exports:**
- `Cliente`, `Equipo`, `Alquiler`, `AlquilerEquipo`, `Notificacion`, `Base`

**schemas exports:**
- All Pydantic models from schemas/cliente.py, schemas/equipo.py, schemas/alquiler.py, schemas/notificacion.py

**routers/clientes.py exports:**
- `router` (FastAPI APIRouter)

**routers/equipos.py exports:**
- `router` (FastAPI APIRouter)

**routers/alquileres.py exports:**
- `router` (FastAPI APIRouter)

**routers/notificaciones.py exports:**
- `router` (FastAPI APIRouter)

**routers/dashboard.py exports:**
- `router` (FastAPI APIRouter)

**services/notification_service.py exports:**
- `NotificationService` class with methods: `check_due_returns()`, `create_notification()`, `mark_as_read()`

### Frontend

**services/api.ts exports:**
- `apiFetch`, `API_BASE_URL`

**services/clientes.ts exports:**
- `getClientes`, `getCliente`, `createCliente`, `updateCliente`, `deleteCliente`

**services/equipos.ts exports:**
- `getEquipos`, `getEquipo`, `createEquipo`, `updateEquipo`, `deleteEquipo`, `getEquiposDisponibles`

**services/alquileres.ts exports:**
- `getAlquileres`, `getAlquiler`, `createAlquiler`, `updateAlquiler`, `deleteAlquiler`, `marcarDevuelto`, `confirmarEntrega`

**services/notificaciones.ts exports:**
- `getNotificaciones`, `getNotificacionesPendientes`, `getVencimientos`, `markNotificacionLeida`, `markAllLeidas`

**hooks/useClientes.ts exports:**
- `useClientes` hook → `{ clientes, loading, error, refetch, crearCliente, actualizarCliente, eliminarCliente }`

**hooks/useEquipos.ts exports:**
- `useEquipos` hook → `{ equipos, loading, error, refetch, crearEquipo, actualizarEquipo, eliminarEquipo }`

**hooks/useAlquileres.ts exports:**
- `useAlquileres` hook → `{ alquileres, loading, error, refetch, crearAlquiler, actualizarAlquiler, eliminarAlquiler, marcarDevuelto }`

**hooks/useNotificaciones.ts exports:**
- `useNotificaciones` hook → `{ notificaciones, loading, error, refetch, marcarLeida, marcarTodasLeidas }`

**types/index.ts exports:**
- All TypeScript interfaces and enums

**styles/tokens.ts exports:**
- `tokens` object with design tokens

---

## 7. FRONTEND STATE & COMPONENT CONTRACTS

### React Hooks

**useClientes:**
```typescript
useClientes() → {
  clientes: Cliente[];
  loading: boolean;
  error: string | null;
  crearCliente: (data: ClienteCreate) => Promise<Cliente>;
  actualizarCliente: (id: number, data: ClienteUpdate) => Promise<Cliente>;
  eliminarCliente: (id: number) => Promise<void>;
  refetch: () => void;
}
```

**useEquipos:**
```typescript
useEquipos() → {
  equipos: Equipo[];
  loading: boolean;
  error: string | null;
  crearEquipo: (data: EquipoCreate) => Promise<Equipo>;
  actualizarEquipo: (id: number, data: EquipoUpdate) => Promise<Equipo>;
  eliminarEquipo: (id: number) => Promise<void>;
  refetch: () => void;
}
```

**useAlquileres:**
```typescript
useAlquileres() → {
  alquileres: Alquiler[];
  loading: boolean;
  error: string | null;
  crearAlquiler: (data: AlquilerCreate) => Promise<Alquiler>;
  actualizarAlquiler: (id: number, data: AlquilerUpdate) => Promise<Alquiler>;
  eliminarAlquiler: (id: number) => Promise<void>;
  marcarDevuelto: (id: number, fecha: string) => Promise<Alquiler>;
  refetch: () => void;
}
```

**useNotificaciones:**
```typescript
useNotificaciones() → {
  notificaciones: Notificacion[];
  loading: boolean;
  error: string | null;
  marcarLeida: (id: number) => Promise<void>;
  marcarTodasLeidas: () => Promise<void>;
  refetch: () => void;
}
```

### Base Components (Props Interfaces)

**Navigation (Sidebar):**
```typescript
interface NavigationProps {
  activePath: string;
  onNavigate: (path: string) => void;
  notificationCount: number;
}
```

**Button:**
```typescript
interface ButtonProps {
  children: React.ReactNode;
  variant?: 'primary' | 'secondary' | 'danger' | 'outline';
  size?: 'sm' | 'md' | 'lg';
  onClick?: () => void;
  disabled?: boolean;
  type?: 'button' | 'submit' | 'reset';
  fullWidth?: boolean;
}
```

**Card:**
```typescript
interface CardProps {
  title?: string;
  children: React.ReactNode;
  footer?: React.ReactNode;
  padding?: 'none' | 'sm' | 'md' | 'lg';
}
```

**InputField:**
```typescript
interface InputFieldProps {
  label: string;
  name: string;
  value: string;
  onChange: (e: React.ChangeEvent<HTMLInputElement | HTMLTextAreaElement>) => void;
  type?: 'text' | 'email' | 'number' | 'tel';
  placeholder?: string;
  error?: string;
  required?: boolean;
  disabled?: boolean;
}
```

**DataTable:**
```typescript
interface Column<T> {
  key: keyof T | string;
  header: string;
  render?: (item: T) => React.ReactNode;
}

interface DataTableProps<T> {
  columns: Column<T>[];
  data: T[];
  loading?: boolean;
  emptyMessage?: string;
  onRowClick?: (item: T) => void;
  pagination?: {
    page: number;
    pageSize: number;
    total: number;
    onPageChange: (page: number) => void;
  };
}
```

**Modal:**
```typescript
interface ModalProps {
  isOpen: boolean;
  onClose: () => void;
  title: string;
  children: React.ReactNode;
  footer?: React.ReactNode;
  size?: 'sm' | 'md' | 'lg' | 'xl';
}
```

**DatePicker:**
```typescript
interface DatePickerProps {
  label: string;
  name: string;
  value: string;
  onChange: (date: string) => void;
  minDate?: string;
  maxDate?: string;
  error?: string;
  required?: boolean;
  disabled?: boolean;
}
```

**Alert:**
```typescript
interface AlertProps {
  type: 'success' | 'error' | 'warning' | 'info';
  message: string;
  onClose?: () => void;
  dismissible?: boolean;
}
```

---

## 8. FILE EXTENSION CONVENTION

- **Project Language**: TypeScript
- **Frontend File Extension**: `.tsx` for React components with JSX, `.ts` for utility files
- **Entry Point**: `/src/main.tsx`
- **HTML Entry**: `/index.html`

---

## 9. DESIGN TOKENS

```typescript
export const tokens = {
  colors: {
    // Primary brand colors
    primary: '#17324D',
    primaryLight: '#2A4A6D',
    primaryDark: '#0F1F33',
    
    // Background colors
    background: '#F7FAFC',
    backgroundSecondary: '#F6F8FB',
    backgroundAlt: '#F5F7FA',
    white: '#FFFFFF',
    
    // Text colors
    textPrimary: '#0F172A',
    textSecondary: '#475569',
    textMuted: '#64748B',
    textInverse: '#FFFFFF',
    
    // Status colors
    success: '#10B981',
    successLight: '#D1FAE5',
    error: '#EF4444',
    errorLight: '#FEE2E2',
    warning: '#F59E0B',
    warningLight: '#FEF3C7',
    info: '#3B82F6',
    infoLight: '#DBEAFE',
    
    // Border and dividers
    border: '#E2E8F0',
    borderLight: '#F1F5F9',
    
    // Semantic colors for equipment states
    disponible: '#10B981',
    alquilado: '#3B82F6',
    mantenimiento: '#F59E0B',
    baja: '#EF4444',
    
    // Semantic colors for rental states
    activo: '#3B82F6',
    finalizado: '#10B981',
    vencido: '#EF4444',
  },
  typography: {
    fontFamily: "'Inter', -apple-system, BlinkMacSystemFont, 'Segoe UI', Roboto, sans-serif",
    fontFamilyMono: "'JetBrains Mono', 'Fira Code', monospace",
    
    // Font sizes
    fontSizeXs: '0.75rem',    // 12px
    fontSizeSm: '0.875rem',   // 14px
    fontSizeBase: '1rem',     // 16px
    fontSizeLg: '1.125rem',   // 18px
    fontSizeXl: '1.25rem',    // 20px
    fontSize2xl: '1.5rem',    // 24px
    fontSize3xl: '1.875rem',  // 30px
    fontSize4xl: '2.25rem',   // 36px
    
    // Font weights
    fontWeightNormal: 400,
    fontWeightMedium: 500,
    fontWeightSemibold: 600,
    fontWeightBold: 700,
    
    // Line heights
    lineHeightTight: 1.25,
    lineHeightNormal: 1.5,
    lineHeightRelaxed: 1.75,
  },
  spacing: {
    0: '0',
    1: '0.25rem',   // 4px
    2: '0.5rem',    // 8px
    3: '0.75rem',   // 12px
    4: '1rem',      // 16px
    5: '1.25rem',    // 20px
    6: '1.5rem',    // 24px
    8: '2rem',      // 32px
    10: '2.5rem',   // 40px
    12: '3rem',     // 48px
    16: '4rem',     // 64px
    20: '5rem',     // 80px
  },
  borderRadius: {
    none: '0',
    sm: '0.25rem',   // 4px
    md: '0.375rem',  // 6px
    lg: '0.5rem',    // 8px
    xl: '0.75rem',   // 12px
    '2xl': '1rem',   // 16px
    full: '9999px',
  },
  shadows: {
    sm: '0 1px 2px 0 rgba(0, 0, 0, 0.05)',
    md: '0 4px 6px -1px rgba(0, 0, 0, 0.1), 0 2px 4px -1px rgba(0, 0, 0, 0.06)',
    lg: '0 10px 15px -3px rgba(0, 0, 0, 0.1), 0 4px 6px -2px rgba(0, 0, 0, 0.05)',
    xl: '0 20px 25px -5px rgba(0, 0, 0, 0.1), 0 10px 10px -5px rgba(0, 0, 0, 0.04)',
    inner: 'inset 0 2px 4px 0 rgba(0, 0, 0, 0.06)',
  },
  breakpoints: {
    sm: '640px',
    md: '768px',
    lg: '1024px',
    xl: '1280px',
    '2xl': '1536px',
  },
  transitions: {
    durationFast: '150ms',
    durationNormal: '250ms',
    durationSlow: '350ms',
    easingDefault: 'cubic-bezier(0.4, 0, 0.2, 1)',
    easingBounce: 'cubic-bezier(0.68, -0.55, 0.265, 1.55)',
  },
  zIndex: {
    dropdown: 1000,
    sticky: 1020,
    fixed: 1030,
    modalBackdrop: 1040,
    modal: 1050,
    popover: 1060,
    tooltip: 1070,
  },
};
```

---

## 10. FUNCTIONAL REQUIREMENTS COVERAGE

| Requirement | Implementation | Files |
|---|---|---|
| Registrar equipos alquilados a clientes | `Equipo` model, `Alquiler` model with many-to-many relationship, `equipos_ids` in create | backend/app/models/equipo.py, backend/app/models/alquiler.py, backend/app/routers/alquileres.py |
| Un equipo no puede alquilarse a dos clientes al mismo tiempo | Business logic in `create_alquiler` endpoint checks for overlapping rentals | backend/app/routers/alquileres.py |
| Notificar cuando vencen las devoluciones | `NotificationService` checks due dates, generates notifications | backend/app/services/notification_service.py, frontend/src/pages/SistemaNotificaciones.tsx |
| Listado de alquileres con filtros | `AlquilerFilters` schema, paginated endpoint | backend/app/routers/alquileres.py |
| Crear/Editar/Eliminar alquileres | Full CRUD on `/alquileres` endpoint | backend/app/routers/alquileres.py |
| Marcar alquiler como devuelto | `POST /alquileres/{id}/devolver` endpoint | backend/app/routers/alquileres.py |
| Confirmar entrega de equipos | `POST /alquileres/{id}/entregar` endpoint | backend/app/routers/alquileres.py |
| Registrar clientes | Full CRUD on `/clientes` endpoint | backend/app/routers/clientes.py |
| Listado de clientes con búsqueda | `ClienteFilters` with `buscar` parameter | backend/app/routers/clientes.py |
| Registrar equipos en catálogo | Full CRUD on `/equipos` endpoint | backend/app/routers/equipos.py |
| Estados de equipo: disponible, alquilado, mantenimiento, baja | `EstadoEquipo` enum, `estado` field | backend/app/schemas/equipo.py |
| Estados de alquiler: activo, finalizado, vencido | `EstadoAlquiler` enum | backend/app/schemas/alquiler.py |
| Dashboard con resumen | `/dashboard/resumen` endpoint | backend/app/routers/dashboard.py |
| Notificaciones pendientes | `/notificaciones/pendientes` endpoint | backend/app/routers/notificaciones.py |
| Marcar notificaciones como leídas | `PUT /notificaciones/{id}` and `PUT /notificaciones/marcar-todas-leidas` | backend/app/routers/notificaciones.py |
| Visualización de fechas de devolución | `fecha_devolucion_prevista` in Alquiler model | frontend/src/pages/GestionAlquileresLista.tsx |
| Alertas visuales para devoluciones próximas/vencidas | Color coding based on EstadoAlquiler | frontend/src/pages/SistemaNotificaciones.tsx |
| Diseño robusto y funcional (UI/UX contract) | 8 base components implemented | frontend/src/components/ui/* |
| 3 páginas principales de Figma | Gestión Alquileres Lista, Sistema Notificaciones, Gestión Clientes Lista | frontend/src/pages/* |
| Tokens de diseño del Figma | Exact colors, spacing, typography from Figma specs | frontend/src/styles/tokens.ts |
| Navegación lateral persistente | Navigation component with links | frontend/src/components/ui/Navigation.tsx, frontend/src/layout/Sidebar.tsx |
| Tabla de datos para listados | DataTable component with pagination | frontend/src/components/ui/DataTable.tsx |
| Selector de fecha (DatePicker) | DatePicker component | frontend/src/components/ui/DatePicker.tsx |
| Modal/Diálogo para formularios | Modal component | frontend/src/components/ui/Modal.tsx |
| Botón CTA (Call to Action) | Button component | frontend/src/components/ui/Button.tsx |
| Tarjeta (Card) | Card component | frontend/src/components/ui/Card.tsx |
| Campo de Entrada (Input Field) | InputField component | frontend/src/components/ui/InputField.tsx |
| Notificación/Alerta | Alert component | frontend/src/components/ui/Alert.tsx |
| Responsive design (desktop-first) | CSS with responsive breakpoints from tokens | frontend/src/styles/global.css |
| Estado de equipos en UI | Color indicators using EstadoEquipo colors | frontend/src/components/ui/* |
| Conexión frontend-backend | REST API via fetch | frontend/src/services/api.ts |
| Uso desde oficina y depósito | Deployed web application accessible via browser | docker-compose.yml, Dockerfile |

---

## 11. BUSINESS RULES

1. **No overlapping rentals**: When creating a rental, the system must verify that none of the selected equipment is already rented during the specified date range (`fecha_inicio` to `fecha_devolucion_prevista`).

2. **Equipment status updates**: When a rental is created, all selected equipment status changes to `ALQUILADO`. When marked as returned, equipment status changes to `DISPONIBLE`.

3. **Vencimiento detection**: A rental is considered `VENCIDO` when `fecha_devolucion_prevista < today` and `estado = ACTIVO`.

4. **Proximity notifications**: The system should generate notifications for rentals where `fecha_devolucion_prevista - today <= 3 days`.

5. **Automatic status update**: When `fecha_devolucion_prevista` passes, the rental status should be automatically marked as `VENCIDO`.

6. **Client deletion protection**: Clients with active rentals cannot be deleted.

7. **Equipment deletion protection**: Equipment with active rentals cannot be deleted.

---

## 12. PAGES OVERVIEW

### Gestión de Alquileres - Lista (node_id: 3:2)
- Header with brand, alert summary badge, user profile
- Sidebar navigation
- Filters and search bar
- DataTable with rental list (cliente, equipos, fecha_inicio, fecha_devolucion_prevista, estado)
- Pagination
- "Nuevo Alquiler" CTA button
- Click row to view/edit detail

### Sistema de Notificaciones (node_id: 4:2)
- Header with brand, user area
- Sidebar navigation
- List of pending notifications (alerts for due returns)
- Each notification shows: mensaje, fecha, tipo
- "Marcar todas como leídas" action
- Color-coded by urgency (vencido = red, proximo = yellow)

### Gestión de Clientes - Lista (node_id: 5:2)
- Header with brand
- Sidebar navigation
- Filters and search bar
- DataTable with client list (nombre_empresa, persona_contacto, telefono, email)
- Pagination
- "Nuevo Cliente" CTA button
- Click row to view/edit detail

### Additional Pages (from page_structure)
- **Dashboard**: Summary cards showing active rentals, available equipment, overdue rentals, pending notifications
- **Gestión de Alquileres - Detalle**: Form to create/edit rental (select client, select equipment, set dates)
- **Gestión de Clientes - Detalle**: Form to create/edit client
- **Catálogo de Equipos - Lista**: Equipment inventory with status badges
- **Catálogo de Equipos - Detalle**: Form to create/edit equipment
- **Recepción y Entrega**: Interface to confirm physical delivery/receipt of equipment