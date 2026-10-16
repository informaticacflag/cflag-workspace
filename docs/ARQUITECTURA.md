# ARQUITECTURA CFLAG — basada en código revisado

## 1. Visión general

```text
React / Vite
    |
    | Axios / services/http.js
    v
ASP.NET Core .NET 8
    |
    +--> Endpoints tradicionales
    |       |
    |       v
    |   Repository genérico
    |       |
    |       v
    |   Stored Procedures compartidos
    |
    +--> Funcionalidades especializadas
            |
            +--> Permisos: Controller -> Service -> SP
            +--> DataTickets: Controller -> Service -> Repository
            +--> Nutrición: Repository propio
```

La arquitectura no es uniforme. Para funcionalidades existentes se conserva el patrón local; para módulos nuevos se utiliza el patrón estándar de CFLAG.

## 2. Login

```text
LoginForm / useLoginForm
       |
       v
http.validate
       |
       | cifra Usuario con useCodificar
       v
POST Login/InicioSesion/{usuarioCifrado}
       |
       v
LoginEndpoint
       |
       v
Repository.InicioSesion
       |
       v
sp_Auth
  Ruta=Login
  Metodo=InicioSesion
       |
       v
JSON en Resultado
       |
       v
AuthContext
       |
       +--> cookie _CFLAG_AUTH_KEY_
       |
       +--> /Inicio
       |
       +--> /Mi_Perfil/Seguridad/MiClave
```

## 3. Menú efectivo

```text
useMenu
   |
   v
GET Menu/get/{usuarioCifrado}
   |
   v
MenuEndpoint
   |
   v
Repository
   |
   v
sp_Auth
 Ruta=Menu
 Metodo=Get
```

### Cálculo de permisos

```text
PermisosPerfil
Usuario -> Ligue
          \
           \
PermisosEntidad ---------> PermisosBase
Usuario -> UsuarioEntidad   /
          -> EntidadPermiso /
                          /
PermisosUsuarioPermitidos
Usuario -> UsuarioPermiso (Permitir=1)
```

Después:

```text
PermisosBase
    |
    | excluir coincidencias
    v
UsuarioPermiso (Permitir=0)
    |
    v
PermisosEfectivos
    |
    v
Modulo -> Rol -> Item -> SubItem
    |
    v
JSON de Menu
```

Una denegación puede ser jerárquica porque IdRol/IdItem/IdSubItem pueden ser NULL.

## 4. CRUD genérico

Ejemplo Carrera:

```text
Academico/page.jsx
      |
      v
useFormCarrera
      |
      v
useFormGeneral
      |
      +--> getCatalogos
      +--> get
      +--> create
      +--> update
      |
      v
services/http.js
      |
      v
CarreraEndpoint
      |
      v
Repository
      |
      v
sp_Con_Catalogos
 Ruta=Carrera
 Metodo=...
```

`useFormGeneral` concentra la lógica común y el hook específico funciona principalmente como configuración.

## 5. Excepción Publicacion

```text
useFormGeneral
      |
      | registro == "Publicacion"
      v
FormData
      |
      +--> ImagenArchivo (File)
      +--> resto de campos
      |
      v
create/update
```

No todos los CRUD usan JSON.

## 6. Permisos administrativos

```text
PermisosUsuario/page.jsx
       |
       v
services/http.js
       |
       v
/api/permisos/*
       |
       v
PermisosController
       |
       v
PermisosService
       |
       +--> sp_UsuarioPermiso_Listar
       +--> sp_UsuarioPermiso_ResumenUsuario
       +--> sp_MenuCatalogo_Listar
       +--> sp_UsuarioPermiso_Guardar
       +--> sp_UsuarioPermiso_Eliminar
```

### Diferencia conceptual

El resumen administrativo no es idéntico al menú efectivo:

```text
Resumen:
Perfil + Usuario

Menu efectivo:
Perfil + Entidad + Usuario permitido - Usuario denegado
```

Esta diferencia debe considerarse al diagnosticar accesos.

## 7. Regla de selección arquitectónica

La coexistencia de varios patrones no significa que cualquiera pueda utilizarse como plantilla para nuevos desarrollos.

```text
                 ¿La funcionalidad ya existe?
                         |
                +--------+--------+
                |                 |
               Sí                No
                |                 |
                v                 v
       Respetar patrón      Usar patrón estándar
       local del módulo          CFLAG
                                  |
                                  v
                               Endpoint
                                  |
                                  v
                         Repository genérico
                                  |
                                  v
                          Stored Procedure
```

Para CRUD estándar frontend:

```text
page.jsx
   |
   v
hook específico
   |
   v
useFormGeneral
   |
   v
services/http.js
   |
   v
Endpoint
   |
   v
Repository genérico
   |
   v
Stored Procedure
```

Los patrones especializados de DataTickets, Nutricion y Permisos por usuario son excepciones existentes y no deben copiarse automáticamente en módulos nuevos.

Solo se admite una arquitectura especializada para un módulo nuevo cuando exista una necesidad técnica o funcional clara.

En ese caso, la excepción debe justificarse antes de implementarse.

## 8. Regla de trabajo para Codex

Antes de implementar:

```text
Pantalla
  ↓
Hook/componente
  ↓
Servicio HTTP
  ↓
Endpoint/Controller
  ↓
Repository/Service
  ↓
Stored Procedure
  ↓
Tablas / JSON
```

No realizar cambios hasta identificar la cadena relevante.
