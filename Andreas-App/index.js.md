# Express.js app (Hello world route)



# Initialize the node.js project
npm init

# Install web framework 
npm install --save express

# Creating a simple endpoint
First create file called índex.js and import the express framework:
const express = require('express');

# Creating the app
const express = require('express')
const app = express()
const port = 3000

app.get('/', (req, res) => {
  res.send('Hello World!')
})

app.listen(port, () => {
  console.log(`Example app listening at http://localhost:${port}`)
})

# Run the application
node index.js

Relaod is done by restart. Nodemon

Stop with control c

express.js 

npm

pm2

nodemon

npm mocha

supertest

body parser

curl http://localhost:3000
