# HolaMundo
pruebas en GitHub

# instruciones de uso 
-primeras lineas de prueba para ver cambios

1-   GET http://localhost:1337/events?populate=professors
2-   GET http://localhost:1337/events?filters[date][$eq]=2025-03-27
3-   GET http://localhost:1337/events?populate=dynamic_content
4-    GET http://localhost:1337/classes?populate=professors
5-       POST http://localhost:1337/professors{
  "first_name": "Juan",
  "last_name": "Pérez",
  "email": "juan.perez@example.com",
  "professorDetails": {
    "specialization": "Matemáticas",
    "experience": "10 años en la enseñanza"
  }
}
6-      . Crear la API para obtener los eventos de un profesor
Esta API recibe el nombre de un profesor y devuelve todos los eventos asignados a ese profesor. Para esto, haremos uso de la relación entre eventos y profesores en Strapi.

Paso 1: Crear el controlador y la ruta
Accede a tu proyecto de Strapi y ve a la carpeta ./src/api/ en tu proyecto.

Crea un controlador personalizado en el directorio de events (o en el modelo donde tienes los eventos), si aún no tienes uno.

Si no tienes un controlador personalizado, crea uno con el siguiente comando:


npx strapi generate:controller events
En el archivo ./src/api/events/controllers/events.js, crea una nueva función para manejar la consulta de eventos por nombre de profesor:


// src/api/events/controllers/events.js

module.exports = {
  async findEventsByProfessor(ctx) {
    try {
      const { professorName } = ctx.params;

      // Buscar los eventos donde el profesor esté asignado
      const events = await strapi.db.query('api::event.event').findMany({
        where: {
          professors: {
            name: professorName
          }
        },
        populate: ['professors'], // Poblar la relación de los profesores
      });

      // Si no se encuentran eventos
      if (events.length === 0) {
        return ctx.badRequest('No se encontraron eventos para este profesor.');
      }

      // Devolver los eventos encontrados
      return events;
    } catch (error) {
      // Manejo de errores
      return ctx.internalServerError('Hubo un error al buscar los eventos.', error);
    }
  },
};
Luego, necesitas definir la ruta para esta función en el archivo routes.json del controlador de eventos. Crea o edita el archivo routes.json:

json

// src/api/events/routes/events.js

module.exports = [
  {
    method: 'GET',
    path: '/events/professor/:professorName',
    handler: 'events.findEventsByProfessor',
    config: {
      policies: [],
      middlewares: [],
    },
  },
];
2. Crear la API para asignar un profesor a una clase
Ahora, creamos una nueva API para asignar a un profesor a una clase, recibiendo como parámetros el nombre de la clase y el ID del profesor.

Paso 1: Crear el controlador para asignar un profesor a una clase
Dentro del controlador de clases (puedes crear uno si aún no existe), abre o crea el archivo ./src/api/classes/controllers/classes.js.


// src/api/classes/controllers/classes.js

module.exports = {
  async assignProfessorToClass(ctx) {
    try {
      const { className, professorId } = ctx.request.body;

      // Buscar la clase por nombre
      const classData = await strapi.db.query('api::class.class').findOne({
        where: { name: className },
      });

      // Verificar que la clase exista
      if (!classData) {
        return ctx.notFound('Clase no encontrada.');
      }

      // Buscar el profesor por ID
      const professorData = await strapi.db.query('plugin::users-permissions.user').findOne({
        where: { id: professorId },
      });

      // Verificar que el profesor exista
      if (!professorData) {
        return ctx.notFound('Profesor no encontrado.');
      }

      // Asignar el profesor a la clase
      await strapi.db.query('api::class.class').update({
        where: { id: classData.id },
        data: {
          professors: [...classData.professors, professorData], // Agregar profesor a la clase
        },
      });

      // Responder con un mensaje de éxito
      return ctx.send({ message: 'Profesor asignado correctamente a la clase.' });
    } catch (error) {
      // Manejo de errores
      return ctx.internalServerError('Hubo un error al asignar el profesor.', error);
    }
  },
};
Paso 2: Definir la ruta para esta API
A continuación, define la ruta para esta función en el archivo de rutas de clases:

json

// src/api/classes/routes/classes.js

module.exports = [
  {
    method: 'POST',
    path: '/classes/assign-professor',
    handler: 'classes.assignProfessorToClass',
    config: {
      policies: [],
      middlewares: [],
    },
  },
];
