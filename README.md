# Trabajo Práctico 11 - Reemplazo completo de reservas desde la web

## 1. Punto de partida y proyecto
Este proyecto (copia denominada `tp-11-salas-web-completa-mongodb`) es la continuación del TP10. El objetivo principal es incorporar la edición por reemplazo completo desde la interfaz web, permitiendo que un formulario HTML (POST) y la API (PUT) compartan y reutilicen el mismo servicio de persistencia en MongoDB Atlas.

## 2. Entorno y Versiones
- **Node.js**: >= 22.13.0
- **Base de Datos**: MongoDB Atlas, clúster gratuito Free (M0). Colección: `reservas`. Base: `reservas_salas_u11`.
- **Dependencias principales**: Express 5.2.1, Mongoose 9.10.3.

## 3. Preparación, Instalación y Ejecución
1. **Instalación:** Ejecutar `npm install`.
2. **Variables de Entorno:** Copiar `.env.example` como `.env` e inyectar la credencial real (`MONGODB_URI`) apuntando a la base `reservas_salas_u11`.
3. **Comando de Inicio:**
   ```bash
   node --env-file=.env src/index.js
   ```

## 4. Diseño y Arquitectura Compartida
Para evitar duplicar código, tanto la API como la aplicación web utilizan el método `servicioReservas.reemplazar()`. 
- La API responde a las peticiones con objetos `JSON`.
- La Web responde renderizando vistas HTML (ej: redireccionando con `303` al éxito, o devolviendo la vista `editar.ejs` con estado `400` conservando el contexto si hay un error de validación).
- Se implementó un guardado de contexto (`res.locals.formularioEdicion`) para asegurar que los rechazos del modelo de Mongoose deriven en el formulario de edición conservando los valores digitados por el usuario, evitando pérdida de datos.

## 5. Rutas de Edición
**API:**
- `PUT /api/reservas/:id`: Recibe un JSON completo con los 7 campos. Devuelve JSON.

**Web:**
- `GET /reservas/:id/editar`: Renderiza el formulario HTML precargado con los datos actuales de la reserva.
- `POST /reservas/:id/editar`: Recibe los datos enviados por formulario (`x-www-form-urlencoded`). Si es exitoso, redirige al detalle de la reserva usando el código `303 See Other` (`/reservas/:id`). Si falla, devuelve la misma vista con código `400`.

## 6. Instrucciones de Prueba en Postman
- **PUT completo (API):** Enviar PUT a `/api/reservas/:id` con `Content-Type: application/json` y los 7 campos requeridos. Devolverá `200 OK`.
- **PUT incompleto (API):** Enviar la misma petición omitiendo un campo (ej: `estado`). Devolverá `400 Bad Request` sin modificar el recurso.
- **Formulario Inválido (Web):** Enviar POST a `/reservas/:id/editar` utilizando `Body > x-www-form-urlencoded`. Enviar los 7 campos pero alterando uno (ej: `personas: 0`). Devolverá `400 Bad Request` y renderizará el HTML de edición conservando los valores previos.

## 7. Matriz Mínima de Pruebas de Edición
*(Pruebas ejecutadas sobre una base inicial de 2 reservas de ejercicio)*

| Caso | Resultado Esperado y Observado |
| :--- | :--- |
| **Abrir edición (Web)** | Se ejecutó GET a `/reservas/:id/editar`. El formulario renderizó con los 7 campos correctamente precargados desde la BD. |
| **Guardar desde web (Éxito)** | Se cambiaron valores y se envió POST. Respondió con `303 See Other` hacia `/reservas/:id`. Los datos se reflejaron correctamente al consultar por API. |
| **Cancelar (Web)** | Se alteraron campos en pantalla y se clickeó "Cancelar". El GET posterior confirmó que el documento persistió intacto. |
| **Formulario inválido (Web)** | Se envió POST web con dato erróneo (personas: 0). Respondió `400 HTML` renderizando el formulario y conservando los valores digitados. |
| **PUT completo (API)** | Se envió PUT con JSON válido. Respondió `200 JSON`. Al recargar el formulario web, la precarga mostró la información actualizada. |
| **PUT incompleto (API)** | Se envió PUT omitiendo `estado`. Respondió `400 Bad Request`. GET posterior confirmó documento sin cambios. |
| **ID inválido / válido ausente (Web)** | GET a ID mal formado devolvió `400`. GET a ID ausente (borrado) devolvió `404`. En ambos casos mostró la página de "Solicitud no completada". |
| **Reiniciar servidor** | Se detuvo el servidor. Al reiniciar, un GET al documento modificado demostró persistencia física de los cambios en la BD. |
| **CRUD Previo** | Se validó que las funciones de Listado, Consulta, Alta y Eliminación previas continúan intactas. |