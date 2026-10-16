# HALLAZGOS DEL ANÁLISIS

## Confirmados

1. El frontend real es JavaScript/JSX, no un frontend TypeScript.
2. La UI revisada usa React Bootstrap/Bootstrap.
3. El backend es un único proyecto EAI.
4. Existen varios patrones arquitectónicos, no uno solo.
5. El CRUD estándar está centralizado en `useFormGeneral`.
6. `Publicacion` es una excepción que usa `FormData`.
7. `sp_Con_Catalogos` es compartido por muchas rutas.
8. `sp_Auth` atiende Login, Menu y Reset.
9. El menú efectivo combina permisos de Perfil, Entidad y Usuario.
10. Las denegaciones individuales pueden excluir permisos jerárquicamente.
11. `sp_UsuarioPermiso_ResumenUsuario` no incluye permisos por Entidad.
12. `sp_UsuarioPermiso_Guardar` desactiva el permiso anterior e inserta uno nuevo.
13. `sp_UsuarioPermiso_Eliminar` hace baja lógica, no DELETE físico.
14. `sp_MenuCatalogo_Listar` devuelve cuatro result sets cuyo orden consume el backend.
15. `useCodificar` modifica la clave utilizando el día del año.

## Puntos que Codex no debe asumir

- JWT activo.
- Arquitectura hexagonal por proyectos.
- Que todos los CRUD usan JSON.
- Que Permisos usa el Repository genérico.
- Que el resumen de permisos administrativos equivale al menú efectivo.
- Que un Stored Procedure compartido pertenece a un solo Endpoint.
- Que código aparentemente redundante puede eliminarse sin buscar consumidores.

## Observaciones para mantenimiento futuro

`useFormGeneral` contiene actualmente:

- logs de depuración;
- código comentado;
- una variable `usuarioActual` sin uso visible.

No es necesario corregirlo para poder usar Codex. Si se decide limpiar/refactorizar, conviene hacerlo como tarea separada de cambios funcionales.

## Estado

Con el código y SQL revisados, los AGENTS.md V2 ya pueden utilizarse como base operativa para Codex.

Para futuras mejoras, lo ideal es actualizar la documentación cuando se incorporen módulos con patrones arquitectónicos nuevos.
