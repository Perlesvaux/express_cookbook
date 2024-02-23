### 0 - Installing **Node** and **npm** through **nvm**:
```bash
curl -o- https://raw.githubusercontent.com/nvm-sh/nvm/v0.39.5/install.sh | bash
nvm install 18
nvm use 18
```
### 1 - Setting up the database:
This example uses **MongoDB Atlas**:

Sign up [here](https://account.mongodb.com/account/register), select *JavaScript* as your preferred programming language. On the *Deploy a cloud database* page, leave this as the default: *M0 Sandbox* (Shared RAM, 512 MB Storage).

Q: *How would you like to authenticate your connection?*
A: Username and Password

Q:*Where would you like to connect from?*
A: My Local Environment

Fill in the *IP Address* with: **0.0.0.0/0** and click on *Add Entry*.

Now, look for the *DEPLOYMENT* section. Click on *Database*. Click on **Connect**:
Connect to your application through **Drivers** (e.g. Node.js, Go, etc.)
In Driver, select **Node.js**, in Version, **5.5 or later**

The **connection string** generated below will be used by your express app to interact with the database (Replace *username* and *password* accordingly.)
```bash
mongodb+srv://username:password@cluster0...
```

### 2 - Let's initialize the project:
Create a folder and some files. i.e.: 
```bash
mkdir roster
cd roster
touch .env package.json index.js
```
Let's edit the `.env` file first. Assign the **connection string** to a variable called `MONGO_URI`. (There should be no spaces around the `=` and no need to surround the string in quotation marks.):
```bash
MONGO_URI=mongodb+srv://username:password@cluster0...
```
Now, let's edit the `package.json` file. Make sure to include these dependencies:

- **express**: Node.js web-app framework
- **body-parser**: Provides middleware for parsing JSON, Text, URL-encoded, and raw data sets over an HTTP request body.
- **mongoose** : js-specific MongoDB Object-Document Mapping (ODM) library
- **dotenv**: loads environment variables from the `.env` file into your application
- **cors**: Provides middleware that can be used to enable *Cross-Origin Resource Sharing* with various options.


i.e.: 
```json
{
  "name": "roster",
  "version": "1.0.0",
  "description": "Minimal web API",
  "main": "index.js",
  "scripts": {
    "start": "node index.js"
  },
  "author": "You",
  "license": "GNU",
  "dependencies": {
    "express":"^4.12.4",
    "body-parser":"^1.15.2",
    "mongoose":"^5.11.15",
    "dotenv":"^16.0.1",
    "cors":"^2.8.5"
  }
}
```
Finally, install dependencies:
```bash
npm i
```
### 3 - Time to code!:
Let's edit the `index.js` file:
```javascript
const express = require('express')
const app = express()
const port = 3000

// Enable CORS (import module and use it globally)
const cors = require('cors')
app.use(cors())

// Connect to Atlas-mongoDB Database
// - Load .env file contents
// - Store connection string in a variable
// - Import mongoose and establish connection
// - Last line is to turn off a deprecation warning.
require('dotenv').config()
const mySecret = process.env['MONGO_URI']
mongoose = require('mongoose')
mongoose.connect(mySecret, {useNewUrlParser:true, useUnifiedTopology:true})
mongoose.set('useFindAndModify', false);


// mongose schema modeled after an individual
// - defines the structure of the document in the MongoDB collection.
const individualSchema = mongoose.Schema({
  name: {type: String, required:true},
  age: {type:Number, required: true}
})

// Instantiation of Model
// - this object has all necessary methods to interact with the database
const Individual = mongoose.model('Individual', individualSchema)

// Configure body-parser
const bodyParser = require('body-parser')
app.use(bodyParser.urlencoded({extended:false}))

// This middleware prints the method, path and ip of each request.
// For illustrative purposes, instead of using it globally, we'll
// add it to every endpoint individually =)
function MWLogger (req, res, next){
  console.log(`${req.method} ${req.path} ${req.ip}`)
  next()
}


// *** CRUD starts here! ***
// RETRIEVE all entries
app.get("/roster", MWLogger, async (req, res)=>{
  try {
    const allItems = await Individual.find({})
    res.json(allItems)

  } catch (error) {
    console.error(error);
    res.status(500).send('Internal Server Error');
  }
})

// RETRIEVE a specific entry
app.get("/roster/:id", MWLogger, async(req, res)=>{
  try {
    const item = await Individual.findById(req.params.id)
    res.json(item)

  } catch (error) {
    console.error(error);
    res.status(500).send('Internal Server Error');
  }
})

// CREATE a new entry in a database through a form
app.post("/roster", MWLogger, async(req, res)=>{
  try {
    const newItem = await Individual.create(req.body)
    res.json(newItem)
    
  } catch (error) {
    console.error(error);
    res.status(500).send('Internal Server Error');
  }
})

// DELETE a specific entry
app.delete("/roster/:id", MWLogger, async(req, res)=>{
  try {
    const deletedEntry = await Individual.findByIdAndRemove(req.params.id)
    res.json(deletedEntry)

  } catch (error) {
    console.error(error);
    res.status(500).send('Internal Server Error');
  }
})

// UPDATE a specific entry
app.put("/roster/:id", MWLogger, async(req, res)=>{
  try {
    const updatedItem = await Individual.findOneAndUpdate(
      {_id: req.params.id},
      req.body,
      {new: true}
    )
    res.json(updatedItem)
    
  } catch (error) {
    console.error(error);
    res.status(500).send('Internal Server Error');
  }
})

// CRUD Summary:
// On the Client side, <input> elements have a 
// 'name' & 'value' properties. 
// These correspond to 'key-values' on the 'req.body' 
// when sending a CREATE or UPDATE request.
// 
// When a new entry is created, it gets its own unique ID.  
// To interact with a specific entry, we define an endpoint 
// with a URL parameter '/:id'. This parameter can now be accessed
// through 'req.params.id' when sending a GET, UPDATE or DELETE request

//App is ready to go!
app.listen(port, () => {
  console.log(`Example app listening on port http://localhost:${port}`)
})

```



