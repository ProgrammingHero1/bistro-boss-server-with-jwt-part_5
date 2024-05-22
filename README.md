# Bistro Boss Server with JWT

Client Repo Link - [bistro-boss-client-with-jwt-part_5](https://github.com/ProgrammingHero1/bistro-boss-client-with-jwt-part_5)

## Overview
BistroBoss is a comprehensive restaurant management system designed to streamline operations, enhance customer experience, and improve overall efficiency. The system leverages modern web technologies to provide a robust, scalable, and high-performance solution for managing complex data structures and interactions within a restaurant environment.

## Added Middle Wares and APIS
**JWT/POST**-creating Token
```js
  app.post('/jwt', async (req, res) => {
      const user = req.body;
      const token = jwt.sign(user, process.env.ACCESS_TOKEN_SECRET, { expiresIn: '1h' });
      res.send({ token });
    })
```
###  **Verify TOken middleware** 
```js
  const verifyToken = (req, res, next) => {
      console.log('inside verify token', req.headers.authorization);
      if (!req.headers.authorization) {
        return res.status(401).send({ message: 'unauthorized access' });
      }
      const token = req.headers.authorization.split(' ')[1];
      jwt.verify(token, process.env.ACCESS_TOKEN_SECRET, (err, decoded) => {
        if (err) {
          return res.status(401).send({ message: 'unauthorized access' })
        }
        req.decoded = decoded;
        next();
      })
    }

    // use verify admin after verifyToken
    const verifyAdmin = async (req, res, next) => {
      const email = req.decoded.email;
      const query = { email: email };
      const user = await userCollection.findOne(query);
      const isAdmin = user?.role === 'admin';
      if (!isAdmin) {
        return res.status(403).send({ message: 'forbidden access' });
      }
      next();
    }

```

## API End Points
### Users

- **GET /users** - Retrieve all users.

```js
app.get("/users", verifyToken, verifyAdmin, async (req, res) => {
  const result = await userCollection.find().toArray();
  res.send(result);
});
```

- **GET /users/admin/:email** - Check if a user is an admin.

```js
app.get('/users/admin/:email', verifyToken, async (req, res) => {
    const email = req.params.email;

    if (email !== req.decoded.email) {
        return res.status(403).send({ message: 'forbidden access' });
    }

    const query = { email: email };
    const user = await userCollection.findOne(query);
    let admin = false;
    if (user)
```

- **POST /users** - Insert a new user.

```js
app.post("/users", async (req, res) => {
  const user = req.body;
  const query = { email: user.email };
  const existingUser = await userCollection.findOne(query);
  if (existingUser) {
    return res.send({ message: "user already exists", insertedId: null });
  }
  const result = await userCollection.insertOne(user);
  res.send(result);
});
```

- **PATCH /users/admin/:id** - Promote a user to admin.

```js
app.patch("/users/admin/:id", verifyToken, verifyAdmin, async (req, res) => {
  const id = req.params.id;
  const filter = { _id: new ObjectId(id) };
  const updatedDoc = {
    $set: {
      role: "admin",
    },
  };
  const result = await userCollection.updateOne(filter, updatedDoc);
  res.send(result);
});
```

- **DELETE /users/:id** - Delete a user.

```js
app.delete("/users/:id", verifyToken, verifyAdmin, async (req, res) => {
  const id = req.params.id;
  const query = { _id: new ObjectId(id) };
  const result = await userCollection.deleteOne(query);
  res.send(result);
});
```

### Cart 
- **GET /cart:** Retrieve all cart items.
```js
app.get('/carts', async (req, res) => {
      const email = req.query.email;
      const query = { email: email };
      const result = await cartCollection.find(query).toArray();
      res.send(result);
    });
```
- **POST /cart:** insert into  all cart collection.
 ```js
  app.post('/carts', async (req, res) => {
      const cartItem = req.body;
      const result = await cartCollection.insertOne(cartItem);
      res.send(result);
    });
 ```
- **DELETE /cart:** Delete a  specific cart item from   cart collection.
 ```js
  app.delete('/carts/:id', async (req, res) => {
      const id = req.params.id;
      const query = { _id: new ObjectId(id) }
      const result = await cartCollection.deleteOne(query);
      res.send(result);
    })
 ```
### Menu
- **GET /menu:** Retrieve all menu items.
  ```js
  app.get('/menu', async (req, res) => {
      const result = await menuCollection.find().toArray();
      res.send(result);
  });
