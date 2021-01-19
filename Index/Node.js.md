# Express.js app (Hello world route)

´´´bash

# Initialize the node.js prject
npm init

# Install web framework 
npm install --save express

# Creating a simple endpoint
First create file called andreas_project.js and import the express framework:
const express = require('express');

# Creating the app
const app = express();

const port = 3000;

app.get('/', (req, res) => {
    res.send('Hello World, from express');
});

app.listen(port, () => console.log(`Hello world app listening on port ${port}!`))

# Run the application
node andreas_project.js

´´´
