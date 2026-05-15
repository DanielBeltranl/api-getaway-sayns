
## Api registro

**Post**

Ruta microservicio: http://localhost:8081/api/registro

Post body:     {
                    "email": "string",
                    "password": "string", -> ejemplo contraseña: Segura123!
                    "phone": number,
                    "userType": "string",
                    "refugioName": "string",
                    "adress": "string"
                }

Respuesta: 


                {
                
                    "error": "string",
                    "message": "string",
                    "status": number,
                    "token": "string"
                
                }






