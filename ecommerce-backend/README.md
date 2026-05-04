# E-Commerce Backend System

A secure, scalable REST API built with Node.js, Express, and MongoDB. Supports customer, admin, and vendor roles with JWT authentication.

---

## Tech Stack

- **Runtime:** Node.js
- **Framework:** Express.js
- **Database:** MongoDB + Mongoose
- **Auth:** JWT + bcryptjs
- **Validation:** express-validator
- **Logging:** morgan

---

## Project Structure

```
ecommerce-backend/
├── src/
│   ├── config/
│   │   └── db.js               # MongoDB connection
│   ├── middleware/
│   │   ├── auth.js             # JWT protect + role authorize
│   │   ├── errorHandler.js     # Centralized error handler
│   │   └── validate.js         # Input validation middleware
│   ├── models/
│   │   ├── User.js
│   │   ├── Product.js
│   │   ├── Category.js
│   │   ├── Cart.js
│   │   └── Order.js
│   ├── controllers/
│   │   ├── auth.controller.js
│   │   ├── product.controller.js
│   │   ├── cart.controller.js
│   │   ├── order.controller.js
│   │   ├── user.controller.js
│   │   └── category.controller.js
│   ├── services/
│   │   ├── auth.service.js
│   │   ├── product.service.js
│   │   ├── cart.service.js
│   │   └── order.service.js
│   ├── routes/
│   │   ├── auth.routes.js
│   │   ├── product.routes.js
│   │   ├── cart.routes.js
│   │   ├── order.routes.js
│   │   ├── user.routes.js
│   │   └── category.routes.js
│   └── app.js
├── server.js
├── .env.example
├── ecommerce.postman_collection.json
└── package.json
```

---

## Getting Started

### Prerequisites

- Node.js v18+
- MongoDB (local or [MongoDB Atlas](https://www.mongodb.com/atlas))

### Installation

```bash
# 1. Clone the repo
git clone <your-repo-url>
cd ecommerce-backend

# 2. Install dependencies
npm install

# 3. Set up environment variables
cp .env.example .env
# Edit .env with your values

# 4. Start the server
npm run dev       # development (nodemon)
npm start         # production
```

### Environment Variables

```env
PORT=5000
MONGO_URI=mongodb://localhost:27017/ecommerce
JWT_SECRET=your_jwt_secret_key_here
JWT_EXPIRES_IN=7d
NODE_ENV=development
```

---

## API Reference

Base URL: `http://localhost:5000/api/v1`

### Authentication

| Method | Endpoint        | Access  | Description        |
|--------|-----------------|---------|--------------------|
| POST   | /auth/register  | Public  | Register a user    |
| POST   | /auth/login     | Public  | Login and get token|
| GET    | /auth/me        | Private | Get current user   |

**Register body:**
```json
{
  "name": "John Doe",
  "email": "john@example.com",
  "password": "password123",
  "role": "customer"
}
```
> `role` can be `customer`, `admin`, or `vendor`. Defaults to `customer`.

**Login response:**
```json
{
  "token": "<jwt_token>",
  "user": { "id": "...", "name": "...", "email": "...", "role": "..." }
}
```

---

### Categories

| Method | Endpoint           | Access  | Description         |
|--------|--------------------|---------|---------------------|
| GET    | /categories        | Public  | Get all categories  |
| POST   | /categories        | Admin   | Create category     |
| PUT    | /categories/:id    | Admin   | Update category     |
| DELETE | /categories/:id    | Admin   | Delete category     |

---

### Products

| Method | Endpoint        | Access        | Description          |
|--------|-----------------|---------------|----------------------|
| GET    | /products       | Public        | Get all products     |
| GET    | /products/:id   | Public        | Get single product   |
| POST   | /products       | Admin/Vendor  | Create product       |
| PUT    | /products/:id   | Admin/Vendor  | Update product       |
| DELETE | /products/:id   | Admin         | Delete product       |

**Query parameters for GET /products:**

| Param     | Type   | Description                    |
|-----------|--------|--------------------------------|
| search    | string | Full-text search               |
| category  | string | Filter by category ID          |
| minPrice  | number | Minimum price filter           |
| maxPrice  | number | Maximum price filter           |
| page      | number | Page number (default: 1)       |
| limit     | number | Results per page (default: 10) |

**Example:** `GET /products?search=phone&minPrice=100&maxPrice=500&page=1&limit=10`

---

### Cart

All cart routes require authentication.

| Method | Endpoint               | Description              |
|--------|------------------------|--------------------------|
| GET    | /cart                  | Get current user's cart  |
| POST   | /cart/items            | Add item to cart         |
| PUT    | /cart/items/:productId | Update item quantity     |
| DELETE | /cart/items/:productId | Remove item from cart    |

**Add item body:**
```json
{
  "productId": "<product_id>",
  "quantity": 2
}
```

---

### Orders

| Method | Endpoint              | Access   | Description              |
|--------|-----------------------|----------|--------------------------|
| POST   | /orders               | Customer | Place order from cart    |
| GET    | /orders/my            | Customer | Get my orders            |
| GET    | /orders/:id           | Customer | Get single order         |
| GET    | /orders               | Admin    | Get all orders           |
| PUT    | /orders/:id/status    | Admin    | Update order status      |

**Place order body:**
```json
{
  "shippingAddress": {
    "street": "123 Main Street",
    "city": "Kathmandu",
    "state": "Bagmati",
    "zip": "44600",
    "country": "Nepal"
  }
}
```

**Order statuses:** `pending` → `confirmed` → `shipped` → `delivered` / `cancelled`

---

### Users (Admin only)

| Method | Endpoint      | Description       |
|--------|---------------|-------------------|
| GET    | /users        | Get all users     |
| GET    | /users/:id    | Get single user   |
| PUT    | /users/:id    | Update user       |
| DELETE | /users/:id    | Delete user       |

---

## Authentication

All protected routes require a Bearer token in the `Authorization` header:

```
Authorization: Bearer <your_jwt_token>
```

---

## User Roles

| Role     | Permissions                                              |
|----------|----------------------------------------------------------|
| customer | Browse products, manage own cart, place/view own orders  |
| vendor   | All customer permissions + create/update own products    |
| admin    | Full access to all resources                             |

---

## Postman Collection

Import `ecommerce.postman_collection.json` into Postman to get all pre-built requests.

Collection variables are auto-set via test scripts — just run requests in this order:
1. Register Admin → Login Admin
2. Register Customer → Login Customer
3. Create Category → Create Product
4. Add to Cart → Place Order

---

## Error Responses

All errors follow this format:

```json
{
  "message": "Error description here"
}
```

| Status | Meaning                        |
|--------|--------------------------------|
| 400    | Bad request / validation error |
| 401    | Unauthorized (no/invalid token)|
| 403    | Forbidden (insufficient role)  |
| 404    | Resource not found             |
| 500    | Internal server error          |
