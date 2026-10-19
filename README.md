# Backend de Turnos y Reservas - Pre-entrega 6

API REST de servicios y reservas migrada de archivos JSON a **MongoDB Atlas con Mongoose**. Se mantiene la arquitectura en capas y los mismos endpoints; solo se reemplazó la capa DAO y se agregaron los models.

## Instalación y ejecución

1. Crear un cluster gratuito en [MongoDB Atlas](https://www.mongodb.com/atlas), un usuario de base de datos y habilitar tu IP en **Network Access**. Copiar la cadena de conexión desde **Connect → Drivers**.
2. Instalar y configurar:

```bash
git clone https://github.com/Teovi13/backend-turnos-pe6.git
cd backend-turnos-pe6
npm install
cp .env.example .env
```

3. Completar el `.env` y ejecutar:

```bash
npm run seed
npm start
```

`npm run seed` carga 4 servicios de ejemplo solo si la colección está vacía. El servidor se conecta a MongoDB y queda en `http://localhost:8080`.

## Variables de entorno

| Variable    | Ejemplo                                                                  |
| ----------- | ------------------------------------------------------------------------ |
| `PORT`      | `8080`                                                                   |
| `NODE_ENV`  | `development`                                                            |
| `MONGO_URI` | `mongodb+srv://usuario:clave@cluster0.xxxxx.mongodb.net/turnos_reservas` |

## Models

| Model     | Archivo                   | Campos                                                                                       |
| --------- | ------------------------- | -------------------------------------------------------------------------------------------- |
| `Service` | `models/service.model.js` | `name`, `description`, `duration`, `price`, `category`, `available`                          |
| `Booking` | `models/booking.model.js` | `clientName`, `clientEmail`, `date`, `time`, `status`, `services: [{ service: ObjectId, quantity }]` |
| `Message` | `models/message.model.js` | `user`, `message`, `createdAt`, `updatedAt`                                                  |

Las reservas guardan solo la referencia (`ObjectId`, con `ref: 'Service'`) y la cantidad de cada servicio, nunca el objeto completo.

## Arquitectura

```
router → controller → service → repository → DAO → model (Mongoose) → MongoDB
```

El DAO devuelve el `_id` de Mongo como `id` (string), así que las capas superiores no dependen de tipos de MongoDB.

## Endpoints

| Método | Ruta                               | Descripción                         | Códigos       |
| ------ | ---------------------------------- | ----------------------------------- | ------------- |
| GET    | `/api/services`                    | Lista servicios (`?category=`, `?available=`) | 200, 400 |
| GET    | `/api/services/:sid`               | Obtiene un servicio                 | 200, 404      |
| POST   | `/api/services`                    | Crea un servicio                    | 201, 400      |
| PUT    | `/api/services/:sid`               | Actualiza un servicio               | 200, 400, 404 |
| DELETE | `/api/services/:sid`               | Elimina un servicio                 | 200, 404      |
| POST   | `/api/bookings`                    | Crea una reserva                    | 201, 400      |
| GET    | `/api/bookings/:bid`               | Obtiene una reserva                 | 200, 404      |
| POST   | `/api/bookings/:bid/services/:sid` | Agrega un servicio a una reserva (incrementa `quantity` si ya estaba) | 200, 400, 404 |

## Ejemplo

```bash
curl http://localhost:8080/api/services

curl -X POST http://localhost:8080/api/bookings \
  -H "Content-Type: application/json" \
  -d '{"clientName":"Ana Pérez","clientEmail":"ana@mail.com","date":"2026-11-05","time":"10:30"}'

curl -X POST http://localhost:8080/api/bookings/<bid>/services/<sid>
```
