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
6-      
