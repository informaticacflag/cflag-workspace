# AGENTS.md — Sistema CFLAG

## 1. Propósito

CFLAG es una intranet institucional compuesta por:

- Frontend React + Vite.
- Backend ASP.NET Core Web API .NET 8.
- SQL Server.
- Stored Procedures con una parte sustancial de la lógica funcional.

Trata CFLAG como una sola solución funcional para fines de análisis, aunque frontend y backend puedan residir en repositorios o carpetas diferentes.

## 2. Regla principal para Codex

No impongas una arquitectura nueva sobre el código existente.

Antes de modificar una funcionalidad:

1. Identifica la página, componente o hook frontend involucrado.
2. Identifica el helper o llamada de `src/services/http.js`.
3. Identifica el Endpoint o Controller backend.
4. Identifica si usa:
   - `Repository` genérico,
   - `Service` especializado,
   - Repository propio de un módulo.
5. Identifica el Stored Procedure real.
6. Revisa el contrato de entrada y salida, especialmente JSON.
7. Busca una funcionalidad equivalente dentro del mismo módulo.
8. Realiza el cambio mínimo necesario.
9. Revisa consumidores antes de renombrar, eliminar o cambiar contratos.

No hagas refactorizaciones generales salvo solicitud expresa.

## 3. Frontend observado

Stack verificado:

- React 18.
- JavaScript / JSX.
- Vite 5.
- React Router 6.
- React Bootstrap / Bootstrap.
- Axios.
- React Hook Form.
- Yup.
- Sass.

El código funcional revisado está implementado principalmente en `.js` y `.jsx`.

No conviertas archivos a TypeScript salvo solicitud explícita.

Estructura relevante:

- `src/app/`
- `src/components/`
- `src/context/`
- `src/hooks/`
- `src/layouts/`
- `src/routes/`
- `src/services/`
- `src/utils/`

## 4. Backend observado

Stack verificado:

- .NET 8.
- ASP.NET Core Web API.
- Microsoft.Data.SqlClient.
- Entity Framework Core SQL Server.
- Newtonsoft.Json.
- Serilog.
- Swagger.
- DotNetEnv.

El backend actual es un único proyecto `EAI`.

No lo describas ni lo reorganices automáticamente como una solución hexagonal separada en proyectos Domain/Application/Infrastructure.

## 5. Patrones backend coexistentes

### 5.1 Patrón principal

Una gran parte del sistema utiliza:

`Endpoint → Repository genérico → Stored Procedure`

El `Repository` genérico envía normalmente:

- `@Llave`
- `@Usuario`
- `@Ruta`
- `@Metodo`
- `@Objeto`

`@Objeto` puede contener JSON serializado.

### 5.2 Permisos por usuario

Usa un patrón especializado:

`PermisosController → PermisosService → Stored Procedures específicos`

### 5.3 Otros módulos

`DataTickets` tiene Controllers, Services, Repositories y Entities propios.

`Nutricion` tiene Repository y modelos propios.

### 5.4 Regla de selección arquitectónica

Para modificar una funcionalidad existente:

- identifica el patrón local del módulo;
- conserva ese patrón;
- no migres incidentalmente el módulo hacia otra arquitectura.

Para desarrollar un módulo completamente nuevo:

- utiliza por defecto el patrón estándar de CFLAG:

`Frontend → services/http.js → Endpoint → Repository genérico → Stored Procedure`

Para CRUD estándar, reutiliza cuando corresponda:

`page.jsx → hook específico → useFormGeneral → services/http.js → Endpoint → Repository genérico → Stored Procedure`

Los módulos especializados existentes, incluyendo:

- `DataTickets`
- `Nutricion`
- Permisos por usuario

son excepciones funcionales o históricas y NO deben utilizarse como plantilla para módulos nuevos.

Solo apartarse del patrón estándar cuando exista una necesidad técnica o funcional clara que impida utilizarlo.

En ese caso, antes de implementar:

1. explicar por qué el patrón estándar no es suficiente;
2. identificar qué requisito obliga a una arquitectura distinta;
3. evaluar el impacto;
4. proponer la excepción explícitamente;
5. no crear una arquitectura especializada únicamente por preferencia técnica o conveniencia.

Regla resumida:

- funcionalidad existente → respetar arquitectura local;
- módulo nuevo → usar patrón estándar de CFLAG;
- módulo nuevo excepcional → justificar antes de apartarse del patrón estándar.

## 6. Autenticación

La sesión frontend se mantiene mediante la cookie:

`_CFLAG_AUTH_KEY_`

El flujo de login verificado es:

`useLoginForm → http.validate → LoginEndpoint → Repository.InicioSesion → sp_Auth`

Ruta:

`Login/InicioSesion/{usuarioCifrado}`

Después del login se puede redirigir a:

- `/Mi_Perfil/Seguridad/MiClave`
- `/Inicio`

Existe cierre automático de sesión por inactividad de 25 minutos.

### Importante

No asumas que CFLAG utiliza JWT para la sesión actual.

Aunque el proyecto referencia `Microsoft.AspNetCore.Authentication.JwtBearer`, el `Program.cs` revisado no configura `AddAuthentication` ni `UseAuthentication`.

No reemplaces el mecanismo de autenticación sin solicitud expresa y análisis de impacto.

## 7. Cifrado utilizado por el frontend

`src/hooks/useCodificar.js` implementa un cifrado personalizado de caracteres alfanuméricos.

Características verificadas:

- Alfabeto:
  `ABCDEFGHIJKLMNOPQRSTUVWXYZabcdefghijklmnopqrstuvwxyz0123456789`
- La clave se modifica usando el día del año.
- Solo acepta texto alfanumérico.
- Se utiliza para cifrar valores como el usuario enviado en determinadas rutas.

No cambies este algoritmo de forma aislada.

Si se modifica:

1. revisa el código equivalente de descifrado backend;
2. revisa el efecto del cambio de día;
3. revisa todos los helpers HTTP que lo utilicen;
4. prueba frontend y backend en conjunto.

Este mecanismo no debe describirse como cifrado criptográfico estándar ni reemplazarse incidentalmente.

## 8. Menú y permisos efectivos

El menú se obtiene mediante:

`Menu/get/{usuario}`

y se resuelve en `sp_Auth` con:

- `@Ruta = 'Menu'`
- `@Metodo = 'Get'`

### Fuentes de permisos del menú

`sp_Auth` combina permisos provenientes de:

1. Perfil:
   `Usuario → Ligue`
2. Entidad:
   `Usuario → UsuarioEntidad → EntidadPermiso`
3. Permisos individuales permitidos:
   `UsuarioPermiso` con `Permitir = 1`

Después aplica permisos individuales denegados:

`UsuarioPermiso` con `Permitir = 0`

Las denegaciones se evalúan jerárquicamente mediante:

- `IdModulo`
- `IdRol`
- `IdItem`
- `IdSubItem`

Los niveles inferiores pueden ser `NULL`, permitiendo que una denegación afecte un nivel superior completo.

Por tanto:

- ocultar una ruta en React no equivale a cambiar permisos;
- cambiar `Ligue`, `EntidadPermiso`, `UsuarioEntidad` o `UsuarioPermiso` puede afectar el menú;
- las denegaciones individuales deben considerarse antes de concluir que un permiso está activo.

## 9. Permisos por usuario — módulo administrativo

Backend:

- `Controllers/PermisosController.cs`
- `Services/PermisosService.cs`
- `Dtos/Permisos/*`

Endpoints observados:

- `GET /api/permisos/usuario/{usuario}`
- `GET /api/permisos/resumen/{usuario}`
- `GET /api/permisos/catalogo-menu`
- `POST /api/permisos/guardar`
- `DELETE /api/permisos/{idUsuarioPermiso}`

Stored Procedures:

- `dbo.sp_UsuarioPermiso_Listar`
- `dbo.sp_UsuarioPermiso_ResumenUsuario`
- `dbo.sp_MenuCatalogo_Listar`
- `dbo.sp_UsuarioPermiso_Guardar`
- `dbo.sp_UsuarioPermiso_Eliminar`

No modifiques estos contratos sin revisar la pantalla `PermisosUsuario`.

## 10. Diferencia importante: menú efectivo vs resumen administrativo

`sp_Auth` para Menu considera:

- Perfil.
- Entidad.
- Permisos individuales permitidos.
- Permisos individuales denegados.

`sp_UsuarioPermiso_ResumenUsuario` considera:

- Perfil.
- Permisos de usuario.

El resumen administrativo revisado no incorpora permisos por Entidad.

No asumas que el resultado de `sp_UsuarioPermiso_ResumenUsuario` es idéntico al menú efectivo.

Si una tarea busca explicar "por qué un usuario ve o no ve una opción", usa como referencia principal la lógica de `sp_Auth` del menú y revisa también Entidad.

## 11. CRUD genérico

Los CRUD de catálogos reutilizan:

`useFormGeneral`

Flujo típico:

`page.jsx → hook específico → useFormGeneral → services/http.js → Endpoint → Repository → SP`

`useFormGeneral`:

- carga catálogos con `getCatalogos`;
- carga registros con `get`;
- normaliza campos definidos en `camposNull`;
- determina Create/Update según `Id`;
- refresca catálogos y datos después de guardar;
- interpreta `Evento = 0` como error.

Antes de crear lógica CRUD nueva, verifica si puede configurarse usando `useFormGeneral`.

## 12. Excepción CRUD: Publicacion

`useFormGeneral` contiene tratamiento específico para:

`registro === 'Publicacion'`

En ese caso utiliza `FormData`, incluyendo soporte para `ImagenArchivo`.

No generalices que todos los CRUD envían JSON.

No elimines esta excepción sin revisar todo el flujo de Publicacion.

## 13. Stored Procedures compartidos

### `sp_Auth`

Rutas verificadas:

- `Login`
- `Menu`
- `Reset`

Métodos verificados:

- `InicioSesion`
- `Get`
- `Cambia`
- `Restablecer`

También invoca `EnviarCorreo` en operaciones de contraseña.

No trates `sp_Auth` como un SP exclusivo de login.

### `sp_Con_Catalogos`

Es un SP multipropósito.

Rutas verificadas incluyen:

- Sexo
- Alergia
- Carrera
- Contrato
- Departamento
- Entidad
- EstadoCivil
- GradoAcademico
- Horario
- Municipio
- Pais
- Parentesco
- Partida
- Profesion
- PuestoFuncional
- PuestoOficial
- Renglon
- Servicios
- TipoCorreo
- TipoDireccion
- TipoIdentificacion
- TipoPolitica
- TipoPublicacion
- TipoSangre
- TipoSector
- TipoTelefono

Métodos verificados:

- `Get`
- `GetById`
- `GetCatalogos`
- `Create`
- `Update`
- `DeleteById`

Antes de modificar `sp_Con_Catalogos`, busca el bloque exacto de `@Ruta` y `@Metodo`.

No refactorices globalmente este SP como parte de un cambio puntual.

## 14. Seguridad y secretos

Nunca agregues al repositorio:

- connection strings reales;
- API keys reales;
- passwords;
- secretos de producción;
- `.env` con valores sensibles.

Variables observadas o relacionadas con el código:

Backend:

- `CN_EAI`
- `CN_MAN`
- `DB_KEY`
- `API_KEY`
- `CORS_ALLOWED_ORIGINS`

Frontend:

- `VITE_API_SERVER`
- `VITE_BASE_PATH`
- `VITE_API_KEY`
- `VITE_SHOW_CATALOGS`

`src/context/constants.js` define:

- `basePath` desde `VITE_BASE_PATH`
- `apiBaseUrl` desde `VITE_API_SERVER`

No hardcodees URLs productivas si ya existe configuración por variables de entorno.

## 15. Cambios de alto riesgo

Antes de realizar cambios en cualquiera de estos puntos, analiza impacto completo:

- autenticación;
- cifrado/descifrado del usuario;
- sesión/cookie;
- `Repository.ExecuteStoredProcedure`;
- estructura de `Result`;
- contratos JSON;
- menú dinámico;
- permisos;
- `sp_Auth`;
- `sp_Con_Catalogos`;
- conexiones;
- Stored Procedures compartidos;
- cambios destructivos de BD.

## 16. Validación mínima

Frontend:

- `npm run lint`
- `npm run build`

Backend:

- `dotnet build`

Además:

- prueba el flujo afectado;
- verifica JSON real;
- verifica menú/permisos cuando aplique;
- revisa logs de API y errores SQL.

## 17. Documentación complementaria

Consultar:

- `frontend/AGENTS.md`
- `backend/AGENTS.md`
- `database/AGENTS.md`
- `docs/ARQUITECTURA.md`
- `docs/HALLAZGOS.md`
