# SISTRA · Frontend

## Tabla de contenido
 
1. [Stack](#stack)
2. [Requisitos](#requisitos)
3. [Instalación y ejecución](#instalación-y-ejecución)
4. [Scripts](#scripts)
5. [Estructura del proyecto](#estructura-del-proyecto)
6. [Reglas de dependencia](#reglas-de-dependencia)
7. [Permisos (checklist, sin roles)](#permisos-checklist-sin-roles)
8. [Convenciones de código](#convenciones-de-código)
9. [Flujo de trabajo con Git](#flujo-de-trabajo-con-git)
---
 
## Stack
 
| Tecnología | Uso |
|---|---|
| Angular 22 | Framework (standalone components, signals, lazy loading) |
| TypeScript | Lenguaje |
| SCSS | Estilos |
| npm | Gestor de paquetes |
 
---
 
## Requisitos
 
| Herramienta | Versión |
|---|---|
| Node.js | **22.12 o superior** |
| npm | La que trae Node |
| Angular CLI | 22 (`npm install -g @angular/cli@22`) |
| Git | Cualquier versión reciente |
 
Para verificarlo:
 
```bash
node -v
npm -v
ng version
```
 
---
 
## Instalación y ejecución
 
```bash
# 1. Clonar el repositorio
git clone https://github.com/SISTRA-UP/sistra-frontend.git
cd sistra-frontend
 
# 2. Cambiarse a la rama de desarrollo
git checkout develop
 
# 3. Instalar dependencias
npm ci
 
# 4. Levantar el servidor de desarrollo
npm start
```
 
La app queda en **http://localhost:4200**.
 
---
 
## Scripts
 
| Comando | Qué hace |
|---|---|
| `npm start` | Servidor de desarrollo |
| `npm run build` | Compilación de producción |
| `npm test` | Ejecuta las pruebas unitarias |
| `npm run lint` | Revisa el estilo del código (debe pasar antes de abrir PR) |
 
---
 
## Estructura del proyecto
 
```
src/
├── environments/
│   ├── environment.ts               # Configuración de producción
│   └── environment.development.ts   # Configuración de desarrollo
├── styles/                          # Variables, mixins y tema global en SCSS
└── app/
    ├── core/                        # Lo que existe una sola vez en toda la app
    │   ├── guards/                  # auth.guard (¿sesión iniciada?), permiso.guard (¿tiene el permiso?)
    │   ├── interceptors/            # Token en cada petición, manejo global de errores
    │   ├── models/                  # Usuario, Permiso y catálogo de códigos de permisos
    │   └── services/                # Autenticación y permisos del usuario actual
    ├── shared/                      # Reutilizable y sin lógica de negocio
    │   ├── components/              # Tabla, modal, file-upload, panel-decision, badge-estado
    │   ├── directives/              # *hasPermiso
    │   └── pipes/                   # Formato de estados, fechas, etc.
    ├── layout/                      # Header, sidebar (según permisos) y main-layout
    ├── features/                    # Un módulo por cada parte del flujo, con carga lazy
    │   ├── auth/                    # Pantalla de login
    │   ├── dashboard/               # Inicio con accesos según los permisos del usuario
    │   ├── estudiantes/             # Carga individual, carga masiva CSV y resultado de la carga
    │   ├── propuestas/              # Subir, descargar, revisar y aprobar/rechazar/aplazar propuestas
    │   ├── informes/                # Subir, descargar, revisar y aprobar/rechazar/aplazar informes
    │   ├── certificados/            # Generar y descargar certificados
    │   ├── notificaciones/          # Bandeja de notificaciones y envío por lote
    │   └── usuarios-permisos/       # Checklist de permisos por usuario
    ├── app.ts                       # Componente raíz
    ├── app.config.ts                # Providers globales (router, http, interceptores)
    └── app.routes.ts                # Rutas raíz; cada feature se carga en lazy
```
 
### Estructura interna de cada feature
 
Todas las carpetas dentro de `features/` siguen el mismo molde:
 
```
features/propuestas/
├── pages/                  # Pantallas (componentes con ruta)
├── components/             # Piezas que solo usa este módulo
├── services/               # Servicios del módulo
├── models/                 # Interfaces y DTOs del módulo
└── propuestas.routes.ts    # Rutas del módulo, cada una protegida con su permiso
```

## Reglas de dependencia
 
| Desde ↓ / Puede importar → | `core` | `shared` | `layout` | Otra `feature` |
|---|:---:|:---:|:---:|:---:|
| `features` | ✅ | ✅ | ❌ | ❌ |
| `layout` | ✅ | ✅ | — | ❌ |
| `shared` | solo `models` | — | ❌ | ❌ |
| `core` | — | ❌ | ❌ | ❌ |
 
- **Una feature nunca importa de otra feature.** Si dos módulos necesitan lo mismo, se sube a `shared` (si es visual) o a `core` (si es un servicio o un modelo).
- Los PR que rompan estas reglas no se aprueban.
---
 
## Permisos (checklist, sin roles)
 
El sistema **no usa roles**. A cada usuario se le marcan individualmente los permisos que tiene, como una checklist. El frontend recibe la lista de permisos al iniciar sesión y la usa en dos lugares:
 
| Dónde | Para qué | Ejemplo |
|---|---|---|
| `permisoGuard` en las rutas | Bloquear el acceso a una pantalla | `canActivate: [permisoGuard('PROPUESTA_DECIDIR')]` |
| Directiva `*hasPermiso` | Ocultar botones o secciones | `<button *hasPermiso="'PROPUESTA_SUBIR'">Subir</button>` |

## Convenciones de código
 
| Elemento | Convención | Ejemplo |
|---|---|---|
| Carpetas y archivos | `kebab-case` | `subir-propuesta/` |
| Clases y componentes | `PascalCase` | `SubirPropuesta` |
| Variables y métodos | `camelCase` | `cargarEstudiantes()` |
| Interfaces y modelos | `PascalCase`, sin prefijo `I` | `Propuesta`, `Usuario` |
| Códigos de permiso | `MAYUSCULA_CON_GUION_BAJO` | `INFORME_SUBIR` |
| Idioma del dominio | Español | `propuesta`, `informe`, `estudiante` |
 
- Se usan **componentes standalone** y **signals**. No se usan `NgModule`.
- Cada feature se carga de forma **lazy** desde `app.routes.ts`.
- La lógica de datos va en los `services/`, nunca directo en un componente.
- La configuración por entorno sale de `environments/`. No se escribe a mano en el código.
- Los archivos se generan con el CLI (`ng generate ...`) para mantener el formato.
---
 
## Flujo de trabajo con Git
 
### Ramas
 
| Rama | Propósito | ¿Push directo? |
|---|---|---|
| `main` | Versión estable, la que se entrega o despliega | ❌ Solo por PR |
| `develop` | Integración del trabajo del equipo | ❌ Solo por PR |
| `feature/<modulo>-<descripcion>` | Una funcionalidad nueva | ✅ |
| `fix/<descripcion>` | Corrección de un error | ✅ |
 
Ejemplos: `feature/propuestas-subir-archivo`, `fix/login-token-expirado`.
 
### Pasos para aportar
 
```bash
git checkout develop
git pull origin develop
git checkout -b feature/estudiantes-carga-csv
 
# ... trabajar ...
 
git add .
git commit -m "feat(estudiantes): carga masiva desde CSV"
git push origin feature/estudiantes-carga-csv
```
 
Luego se abre un **Pull Request hacia `develop`**. Lo revisa y aprueba el **full stack**.
 
### Mensajes de commit
 
Se usa el formato [Conventional Commits](https://www.conventionalcommits.org/es/v1.0.0/): `tipo(módulo): descripción`
 
| Tipo | Cuándo |
|---|---|
| `feat` | Funcionalidad nueva |
| `fix` | Corrección de un error |
| `refactor` | Cambio interno sin alterar el comportamiento |
| `style` | Formato o estilos, sin cambios de lógica |
| `docs` | Documentación |
| `test` | Pruebas |
| `chore` | Configuración y dependencias |
 
### Antes de abrir un PR
 
- [ ] `npm run lint` pasa sin errores
- [ ] `npm run build` compila
- [ ] La rama está actualizada con `develop`
- [ ] Respeta la estructura y las reglas de dependencia
- [ ] No incluye `console.log`, código comentado ni credenciales
