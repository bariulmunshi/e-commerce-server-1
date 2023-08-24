## Table of content
- [Pagination set up](#pagination-set-up)
  - [totalItems determine the total number of items](#totalitems-determine-the-total-number-of-items)
  - [Decide the total number of item per page](#decide-the-total-number-of-item-per-page)
- [What's JWT?](#whats-jwt)
  - [Introduction](#introduction)
  - [How to use JWT?](#how-to-use-jwt)
  - [From Programming hero](#from-programming-hero)
- [CRUD Method-Database Integrate](#crud-method-database-integrate)
  - [step by step New database setup](#step-by-step-new-database-setup)
  - [POST Method](#post-method)
  - [READ Method](#read-method)
  - [DELETE Method](#delete-method)
  - [Update Data](#update-data)
- [express.js-Backend](#expressjs-backend)
  - [Reuse:step by step set-up](#reusestep-by-step-set-up)
  - [DirectUse: express.js set up-Backend](#directuse-expressjs-set-up-backend)
- [Firebase-React Authentication](#firebase-react-authentication)

# step by step project setup

- React Router
- Tailwind
- .env.local: 69-7 (Recap) Create a simple Login page with firebase integration
- eslint file error solve: "react/prop-types","off",
- project build command

  ```sh
   >mkdir coffee-store-server
   >npm init -y
   >npm i express cors mongodb dotenv
   >nodemon index.js
   >In index.js file: require('dotenv').config()
   >npm install sweetalert2
  ```

- Insert bulk data in database
  ```sh
     shortcut system for insert data
   step-1: copy data & insert the collection to database collection
    load data
    Load services data
   step-2: for specific use query & for all use find
  ```
# Pagination set up 
### totalItems: determine the total number of items
  ```sh
   step-1: create api
   >app.get('/totalProducts',async(req,res)=>{
      const result=await productCollection.estimatedDocumentCount();
    res.send({totalProducts: result});
    })
   step-2: create route loader 
   >path: "/",
        element: <Shop></Shop>,
        loader:()=>fetch('http://localhost:5000/totalproducts')
   step-3: create loader in file
   > const {totalProducts}=useLoaderData();
  ```
### Decide the total number of item per page
  ```sh
   const [itemsPerPage,setItemsPerPage]=useState(10);
  ```
### calculate the total number of pages
  ```sh
   const totalPages=Math.ceil(totalProducts/itemsPerPage)
  ```
### For button mapping & Declare State
  ```sh
  >const pageNumbers = [...Array(totalPages).keys()];
  > const options=[5,10,20];
  function handleSelectChange(event){
    setItemsPerPage(parseInt(event.target.value));
    setCurrentPage(1);
  }
  >const [currentPage,setCurrentPage]=useState(0);
  ><div className="pagination">
    <p>current Page: {currentPage} and Items per page:{itemsPerPage}</p>
        {pageNumbers.map((number) => (
          <button key={number}
          className={currentPage===number?'selected':''}
          onClick={()=>setCurrentPage(number)}
          >{number+1}</button>
        ))}
        <select value={itemsPerPage} onChange={handleSelectChange}>
          {
            options.map(option=>(
              <option key={option} value={option}>
                {option}
              </option>
            ))
          }
        </select>
      </div>
  ```
###  Set Page Size State And Send Search Query To Server
    ```sh
     step-1: useEffect(() => {
    async function fetchData() {
      const response = await fetch(
        `http://localhost:5000/products?page=${currentPage}&limit=${itemsPerPage}`
      );
      const data = await response.json();
      setProducts(data);
    }
    fetchData();
   }, [currentPage, itemsPerPage]);
     step-2: 
    ```
### Load Data Based On The Page Number And Size Using Chatgpt
    ```sh
    
    
    ```
# What's JWT?

### Introduction

- How to secure an API using a JWT token?
- JWT mainly is mainly used for authorization(from server) purpose, not authentication(from firebase)
- Industry standard RFC 7519
- Securely transmits information between parties as a JSON object.
- Digitally Signed
- Get Two Token {access(like passport,XSS,Local storage,HTTP only cookie),refresh(like national id card)} from server
- Header(Authorization:Bearer-access token), payload(data), verify Signature

### How to use JWT?

```sh
step-1: >npm install jsonwebtoken
step-2: >const jwt=require('jsonwebtoken');
step-3: create Access token Secret:>node>require('crypto').randomBytes(64).toString('hex')
& set it on env file:ACCESS_TOKEN_SECRET=cdb62f1b191c3430452c53d718c183d8f2c43d2ea733ec1bf9ac9345ddc559ba4bd2b4bc2bf14b7f5c01efd852d2b755d04a440233ce671d4da223cfa90731af
step-4: Create JWT Token on server
>/JWT:API create
 app.post('/jwt',(req,res)=>{
   const user=req.body;
   console.log(user);
   const token=jwt.sign(user,process.env.ACCESS_TOKEN_SECRET,{
     expiresIn:'1h'});
     console.log(token);
     res.send({token});
 })
step-5: GET JWT Token on client side
>signIn(email,password)
 .then(result=>{
   const user=result.user;
   const loggedUser={
     email:user.email
   }
   console.log(loggedUser);
   fetch('https://car-doctor-server-three-weld.vercel.app/jwt',{
     method:'POST',
     headers:{
       'content-type':'application/json'
     },
     body:JSON.stringify(loggedUser)
   })
   .then(res=>res.json())
   .then(data=>{
     console.log('jwt response',data);
     //warning:Local storage isn't the best(second best place) to store access token
     localStorage.setItem('car-access-token',data.token);
     navigate(from,{replace:true})
   })
 })
 step-6: Remove token from local storage: got to Navbar logout
 > localStorage.removeItem('car-access-token');
```

### Send jwt token in the server, verify and decode jwt token

```sh
step-1: Declare method from  client side
>{
       method:'GET',
       headers:{
         authorization:`Bearer ${localStorage.getItem('car-access-token')}`
       }
step-2: Declare function from sever side
>const verifyJWT=(req,res,next)=>{
console.log('hitting verify JWT');
console.log(req.headers.authorization);
const authorization=req.headers.authorization;
if(!authorization){
 return res.status(401).send({error:true,message:'Unauthorized Access'})
}
const token=authorization.split(' ')[1];
console.log('token inside Verify JWT',token)
//verify & decode
jwt.verify(token,process.env.ACCESS_TOKEN_SECRET, (error,decoded)=>{
 if(error){
   return res.status(403).send({error:true,message:'unauthorized access'})
 }
 req.decoded=decoded;
 next();
})
}
```

### From Programming hero

```sh
  /**
* JWT Main purpose: secure your api
* ---------------------------------------
*              CREATE TOKEN
* --------------------------------------
* 1. client: after user login send user basic info to create token
*
* 2. in the server side: install npm i jsonwebtoken
* 3. import jsonwebtoken
* 4. jwt.sign(payload, secret, {expires} )
* 5. return token to the client side
*
* 6. after receiving the token store it either httponly cookies or localstorage (second best solution)
*
* 7. use a general space onAuthStateChange > AuthProvider
* -------------------------------------
*              SEND TOKEN TO SERVER
* ---------------------------------------
* 1. for sensitive api call ( ) send authorization headers
*  { authorization: 'Bearer token'}
*
* -------------------------------------
*              VERIFY TOKEN
* --------------------------------------
*
* 1. Create a function called verifyJWT (middleware)
* 2. this function will have three params: req, res, next
* 3. First check whether the authorization headers exists
* 4. if not send 401
* 5. get the token out of the authorization header
* 6. call jwt.verify(token, secret, (err, decoded))
* 7. if err => send 401
* 8. set decoded to the req object so that we can retrieve it later
* 9. call the next() to go to the next function
*
* -----------------------
* 1. check wether token has the email that matches with the request email
*
*/
```

### Create JWT token for Sign Up and Social Login users

    ```sh
     step-1: create social login file

    ```

# CRUD Method-Database Integrate

## step by step New database setup

- Mongodb Database connection for new database
  ```sh
   Go to mongodb atlas site
   > Database > connect >Drivers >copy & paste it index.js file
   > create file(.env) for keep password
   > In index.js file: require('dotenv').config()
     DB_USER=docUser
     DB_PASS=rfzGdApBdPpLH2kF
   > check:console.log(process.env.DB_PASS)
   > convert the username pass holder in template string &
   keep username & password
   const uri = `mongodb+srv://${process.env.DB_USER}:${process.env.DB_PASS}@cluster0.8uchuib.
   mongodb.net/?retryWrites=true&w=majority`;
   > Database Access: create username password
  ```

### POST Method

- create POST Method api in backend
  ```sh
    > app.post("/coffee", async (req, res) => {
      const newCoffee = req.body;
      console.log("new NewCoffee", newCoffee);
      });
  ```
- Send data from client side to server side using fetch
  ```sh
  > fetch("https://car-doctor-server-three-weld.vercel.app/coffee", {
      method: "POST",
      headers: {
        "content-type": "application/json",
      },
      body: JSON.stringify(newCoffee),
      })
      .then((res) => res.json())
      .then((data) => {
        console.log(data);
      });
  ```
- create database & collection name

  ```sh
   > const database = client.db("usersDB");
     const usersCollection = database.collection("users");

   or
   > const coffeeCollection=client.db('coffeeDB').collection('coffee');
  ```

- In POST method API >send server data to Database
  ```sh
   > const result=await coffeeCollection.insertOne(newCoffee);
      res.send(result)
  ```
- Give a response to a user from client side
  ```sh
   >npm install sweetalert2
   >import Swal from 'sweetalert2'
   >if(data.insertedId){
          Swal.fire({
            title: "Error!",
            text: "Do you want to continue",
            icon: "error",
            confirmButtonText: "Cool",
          });
        }
  ```

### READ Method

- Data READ

  ```sh
          for read all data
   step-1:get/read data from server site
   > app.get('/coffee',async(req,res)=>{
      const cursor=coffeeCollection.find()
      const result=await cursor.toArray()
      res.send(result)
    })
   step-2: load data in client side from server link
   >path: "/",
    element: <App></App>,
    loader: () => fetch("https://car-doctor-server-three-weld.vercel.app/services"),
    step-2.1: In the created file use useLoaderData
    >const coffees=useLoaderData()

          for read some data
   app.get('/bookings',verifyJWT,async(req,res)=>{
       // console.log(req.query.email);
      // console.log(req.headers.authorization);
      const decoded=req.decoded;
      console.log('came back after verify',decoded);
      if(decoded.email !==req.query.email){
        return res.status(403).send({error:1,message:'forbidden access'})
      }
      let query={};
      if(req.query?.email){
        query={email:req.query.email}
      }
      const result=await bookingCollection.find(query).toArray();
      res.send(result);
      })


          for read specific data
   step-1:  /* created api for read specific data */
    app.get('/services/:id',async(req,res)=>{
      const id=req.params._id;
      const query={_id: new ObjectId(id)}
      const options = {
        // Include only the `title` and `imdb` fields in the returned document
        projection: { title: 1, price: 1,service_id:1},
      };
      const result=await serviceCollection.findOne(query,options);
      res.send(result);
    })
    step-2: load data in client side from server link
       > path:'/checkout/:id',
        element:<Checkout></Checkout>,
        loader:({params})=>fetch(`https://car-doctor-server-three-weld.vercel.app/services/${params._id}`)
    step-2.1: In the created file use useLoaderData
    >const checkout=useLoaderData()

    step-3: Read data by sort
    > /* created api for read all data:Service routes */
    app.get("/services", async (req, res) => {
      const query = {};
      const options = {
        // sort matched documents in descending order by rating
        sort: { "price": -1 }
      };
      const cursor = serviceCollection.find(query,options);
      const result = await cursor.toArray();
      res.send(result); 
    });
    step-3.1:set data by price range
    >const [asc,setAsc]=useState(true);
    ><button 
        className="btn btn-primary"
        onClick={()=>setAsc(!asc)}
        >{asc?'Price: Hight to Low':'Price: Low to High'}</button>
    > fetch(`http://localhost:5000/services?sort=${asc?'asc':'desc'}`)
    
    In backend index.js file
    > app.get("/services", async (req, res) => {
      const sort=req.query.sort;
      const query = {};
      const options = {
        // sort matched documents in descending order by rating
        sort: { 
          "price": sort==='asc'?1:-1 
        }
      };
      const cursor = serviceCollection.find(query,options);
      const result = await cursor.toArray();
      res.send(result); 
    });

         MongoDB operator 
    > const query = { price: { $lt: 100 } };      
      const query = { price: { $gt: 100 } };
      const query={price:{$gte:50,$lte:150}};    


  ```
- Indexing and text search in mongodb:Use of useRef
  ```sh
   step-1: Take input group button from daisyUI
   step-2: declare useRef
   > const searchRef = useRef(null); & const [search,setSearch]=useState('');
   > set it in search input: ref={searchRef}
   > set handler in button: onClick={handleSearch} 
   > set function of handler: 
   const handleSearch=()=>{
    console.log(searchRef.current.value);
    setSearch(searchRef.current.value);
    } 
    step-3: send search to backend as query parameter
    example one query parameter already sent as sort like:
    > fetch(`http://localhost:5000/services?sort=${asc ? "asc" : "desc"}`)
    for send multiple query parameter for search 
    >useEffect(() => {
    fetch(`http://localhost:5000/services?search=${search}&sort=${asc ? "asc" : "desc"}`)
      .then((res) => res.json())
      .then((data) => setServices(data));
    }, [asc,search]);
    step-3.1: Now need to receive query search from server side
    > //const sort=req.query.sort; it's for sort query
      const search=req.query.search;
      const query={title:{$regex:search,$options:'i'}}
  ```

### DELETE Method

- Data DELETE

  ```sh
            send user delete data to backend
   step-1: In users file create delete button and handler
   > <button
            onClick={()=>handleDelete(user._id)}
            >X</button>
   > const handleDelete=_id=>{
    console.log('delete',_id)
    fetch(`https://car-doctor-server-three-weld.vercel.app/users/${_id}`,{
      method:'DELETE',

    })
    .then(res=>res.json())
    .then(data=>{
        console.log(data);
        if(data.deletedCount>0){
          alert('Deleted successfully');
        }
    })
   }
   or, if use sweet alert:
   >const handleDelete=_id=>{
    console.log(_id)
    Swal.fire({
      title: "Are you sure?",
      text: "You won't be able to revert this!",
      icon: "warning",
      showCancelButton: true,
      confirmButtonColor: "#3085d6",
      cancelButtonColor: "#d33",
      confirmButtonText: "Yes, delete it!",
    }).then((result) => {
      if (result.isConfirmed) {
        // Swal.fire("Deleted!", "Your file has been deleted.", "success");
        fetch(`https://car-doctor-server-three-weld.vercel.app/coffee/${_id}`, {
          method: "DELETE",
        })
          .then((res) => res.json())
          .then((data) => {
            console.log(data);
            if (data.deletedCount > 0) {
               Swal.fire("Deleted!", "Your coffee has been deleted.", "success");
               const remaining=users.filter(user=>user._id!==_id);
               setUsers(remaining);
            }
          });
      }
    });
  }


    Remove data from server side
   step-2: In server file
   > app.delete('/users/:id', async(req,res)=>{
      const id=req.params._id;
      console.log('please delete from database',id);
      const query={_id: new ObjectId(id)}
      const result=await userCollection.deleteOne(query);
      res.send(result)
      })

   step-3: Delete without Refresh
   > const loadedUsers=useLoaderData();
     const [users,setUsers]=useState(loadedUsers);
   > const remaining=users.filter(user=>user._id!==_id);
     setUsers(remaining);
  ```

### Update Data

- Update data

  ```sh
       put=na thakle add kore deewa
       patch=thakle oita update kora
  step-1: create user data loader api in backend side
   > app.get("/coffee/:id", async (req, res) => {
      const id = req.params._id;
      const query = { _id: new ObjectId(id) };
      const result = await coffeeCollection.findOne(query);
      res.send(result);
      //after write this method check by id in browser
    });

   step-2: create a file and dynamic route
   > {
    path: "/updateCoffee/:id",
    element: <UpdateCoffee></UpdateCoffee>,
    loader: ({ params }) => fetch(`https://car-doctor-server-three-weld.vercel.app/coffee/${params._id}`),
  },

   step-3: create a dynamic link button
    > <Link to={`/updateCoffee/${_id}`}>
      <button className="btn">Edit</button>
      </Link>
   step-4: display coffee in update route
    > const coffee=useLoaderData()

   step-4.1:create the UI like post method(if need copy then copy)

   step-5: set default value for update data entry like create value
    > defaultValue={loadedUser?.name} or, > defaultValue={name}

   step-6: Now change the handle  name, object name, method name,update condition & make dynamic fetch path

    send from client-side> receive server-side> update database>client side>display user
    step-4: client side data send by PUT method like post method
    > fetch(`https://car-doctor-server-three-weld.vercel.app/users/${loadedUser._id}`,{
      method:'PUT',
      headers:{
        'content-type':'application/json'
      },
      body:JSON.stringify(updatedUser)
    })
      .then((res) => res.json())
      .then((data) => {
        console.log(data);
      });

    step-5: Receive backend data by PUT method
    > /* put method update data */
       app.put("/coffee/:id", async (req, res) => {
      const id = req.params._id;
      const updatedCoffee = req.body;
      console.log(updatedCoffee);
      });
    step-6: Database receive :In server file
    > const filter={_id: new ObjectId(id)}
       const options={upsert: true}
       const updatedUser={
        $set: {
          name:user.name,
          email:user.email
        }
       }
       const result=await userCollection.updateOne(filter,updatedUser,options);
       res.send(result);


    step-5&6: Together
    >app.put("/coffee/:id", async (req, res) => {
      const id = req.params._id;
      const updatedCoffee = req.body;
      // console.log(updatedCoffee);
      const filter = { _id: new ObjectId(id) };
      const options = { upsert: true };
      const coffee = {
        $set: {
          // name: user.name,
          // email: user.email,
          name: updatedCoffee.name,
          quantity: updatedCoffee.quantity,
          supplier: updatedCoffee.supplier,
          taste: updatedCoffee.taste,
          category: updatedCoffee.category,
          details: updatedCoffee.details,
          photo: updatedCoffee.photo,
        },
      };
      const result = await coffeeCollection.updateOne(
        filter,
        coffee,
        options
      );
      res.send(result);
     });

  ```

# express.js-Backend

## Reuse:step by step set-up

```sh
   step-1: create file >mkdir file-name, run command on this file >npm init -y & Express install: npm install express
   or npm i express or npm  i express cors mongodb dotenv jsonwebtoken
   step-2: Add in script > "start": "node index.js",
```

```sh
                Go to hello world doc
step-3.1:  create file named same as entry point from package.json file (index.js)
step-3.2: In index.js file import express > const express=require('express') (it used after come ES6 module)
step-3.3: create app using express > const app=express();
step-3.4: create port > const port =process.env.PORT ||5000;  (port)server entry point(a server has multiple entry point)
step-3.5: get data by root path from server
> app.get('/',(req,res)=>{res.send('Hello from my first ever server')}) it send response when get  request is made
step-3.6: get data by custom path from server
> app.get('/data',(req,res)=>{res.send('Hello from my first ever server')})
step-3.7: check connection with app & port(check the console In which server port is running the app)
 > app.listen(port,()=>{
   console.log(`My first server is running on port:${port}`)
   })
 This code starts the server and listens on a specified port, printing a message to the console indicating that the server is running.
step-3.8: check by nodemon for watch live updating
    check version >nodemon -v
    start server watch >nodemon index.js
step-3.9: create file(.env) root of the project
   > In index.js file: require('dotenv').config()

                       middleware setup
               To  Allow access-control-allow-origin
               Need middleware from express>resource
step-4.1: In server folder >npm install cors
step-4.2: In server index.js file import
 > const cors=require('cors')
 > app.use(cors())
 > app.use(express.json()); {
   In a Node.js and Express application, the app.use(express.json()) middleware is used to parse
   incoming JSON data from client requests. When a client sends a request with JSON data in the
   request body, this middleware parses the JSON data and populates the req.body object with the parsed data}
```

- Additional > if I want to run data from json file

  ```sh
    step-1: import file in a variable
    > const phones=require('./phones.json');
    step-2: Get data by server path
    > app.get('/phones',(req,res)=>{
    res.send(phones);
    })

                    Now to get id data from json file
    step-1: create server path for dynamic id  or specific id news
    > app.get('/phones/:id',(req,res)=>{
      const id=parseInt(req.params._id);
      console.log('I need data for id:',id);
      const phone=phones.find(phone=>phone._id===id) ||{};
      res.send(phone);
    })

    or,
    app.get('/news/:id',(req,res)=>{
    const id=req.params._id;
    console.log(id);
    const selectedNews=news.find(n=>n._id===id);
    res.send(selectedNews)
    })

    or,
    pp.get('/categories/:id',(req,res)=>{
    const id=parseInt(req.params._id);
    console.log(id)
    if(id===0){
    res.send(news)
    }
    else{
    const categoryNews=news.filter(n=>parseInt(n.category_id)===id)
    res.send(categoryNews);
    }
    })
  ```

- connect api from created server if there had no route
  ```sh
  step-1: fetch by state
  >const [categories,setCategories]=useState([]);
   useEffect(()=>{
   fetch("https://the-news-dragon-server-bariulmunshi.vercel.app/categories")
  .then(res=>res.json())
  .then(data=>setCategories(data))
  .catch(error=>console.error(error))
   },[])
  step-2: check length
  ```
- connect sever api with client side if there had route

  ```sh
    create component for fetch data use
              for fetch all data
   step-1: create route for component
   > path: "/phones",
    element: <Phones />,
    loader:()=>fetch('https://the-news-dragon-server-bariulmunshi.vercel.app/phones')

   step-2: load data in created component
   > const phones=useLoaderData();
   > <div>
    <h2>all phones here:{phones.length}</h2>
   {
    phones.map(phone=><li key= {phone._id}>
      <Link to={`/phone/${phone._id}`}>{phone.name}</Link></li>)
   }
  </div>

              for fetch individual data
   step-1: create dynamic route for component
   > {
    path:'/phone/:id',
    element:<Phone></Phone>,
    loader:({params})=>fetch(`https://the-news-dragon-server-bariulmunshi.vercel.app/phones/${params._id}`)
   }
   step-2: load individual data by dynamic id
  > const phone =useLoaderData();
  > <div>
  <h2>{phone.name}</h2>
  <img src={phone.image} alt="" />
  </div>
  ```

  step-5: add file >.gitignore write > node_modules,.env & check it by git init

  step-8: create a post api on the server side

  > app.post('/users',(req,res)=>{
  > console.log('Post API hitting')
  > console.log(req.body);
  > const newUser=req.body;
  > newUser.\_id=users.length+1;
  > users.push(newUser);
  > res.send(newUser);
  > })

## DirectUse: express.js set up-Backend

```sh
 step-1: > npm init -y
 step-2: > npm  i express cors mongodb dotenv
 step-3: in package.json file create >"start": "node index.js",
 step-4: create file > index.js
 step-5: In index.js file
 >const express = require('express');
  const cors = require('cors');
  require('dotenv').config()
  const app=express();
  const port=process.env.PORT || 5000;

  /* middleware */
  app.use(cors());
  app.use(express.json());

  /* check root path */
  app.get('/',(req,res)=>{
    res.send('Simple crud running')
  })

  /* check running server port */

  app.listen(port,()=>{
    console.log(`Simple crud is running:${port}`);
  })

 step-6: import code from atlas dbms
 step-7: set password carefully and see the server is pinged or not
 step-8: Now go to for > set up client side
 step-9: send server data to mongodb
 >  const database = client.db("usersDB");
    const usersCollection = database.collection("users");
    const result = await usersCollection.insertOne(user);
    res.send(result);
```

# Firebase-React Authentication

## setup firebase in project

1. create firebase project & create a web app
2. npm install firebase & save firebase config and export app
3. Build >Authentication >Get started >Enable SignIn method
4. Complete the sign up & Login form
5. Add onSubmit handler for collect form data

   ```sh
   onSubmit={handleSignUp}

   const handleSignUp=event=>{
     event.preventDefault()
     const form=event.target
     const email=form.email.value
     const password=form.password.value
     const confirmPassword=form.confirm.value
     console.log(email,password,confirmPassword)
     }
   ```

6. validation form data

   ```sh
     step-1: declare state
     const [error, setError] = useState("");

     step-2:Add condition for validation
     if(password!==confirmPassword){
     setError('Your password did not match')
     return
   }
     else if (!/(?=.*[A-Z])/.test(password)) {
       setError("Please Add at least one uppercase");
       return;
     } else if (!/(?=.*[A-Z].*[A-Z])/.test(password)) {
       setError("Please add at least two numbers");
       return;
     } else if (password.length < 6) {
       setError("Please add at least 6 character in your password");
       return;
     }

     step-3: Display the catch error if exist
     <p className="text-error">{error}</p>
   ```

7. {context Api/redux(redux-toolkit)/database} for share form authentication information in every route

   here for context api:

   step-1: create context Provider file(AuthProvider)

   step-2: createContext with export & set context value with children props

   ```sh
     export const AuthContext=createContext(null);

     const AuthProvider = ({children}) => {
     const user={displayName:'Bariul'}
      const authInfo={
          user
      }
     return (
     <AuthContext.Provider value={authInfo}>
      {children}
     </AuthContext.Provider>
     );};
   ```

8. set the AuthProvider path
   ```sh
      ReactDOM.createRoot(document.getElementById("root")).render(
      <React.StrictMode>
         <AuthProvider>
           <RouterProvider router={router} />
         </AuthProvider>
      </React.StrictMode>
      );
   ```
9. Now createContext as useContext For use Another component
   ```sh
    1. const {user,createUser}=useContext(AuthContext)
    1. const {user,signIn}=useContext(AuthContext)
    2. check:  console.log(user);
    2. check:  console.log(signIn,createUser);
    3. call the function in eventHandler function: createUser(email,password)
    3. call the function in eventHandler function: signIn(email,password)
   ```
10. Now set Auth in AuthProvider file from firebase authentication doc
    ```sh
      import { getAuth } from "firebase/auth";
      const auth = getAuth(app);
      import app from './../../firebase/firebase.config';
    ```
11. set auth user value to useState
    ```sh
    const [user,setUser]=useState(null)
    ```
12. for register

    ```sh
      step-1: In AuthProvider file
      const createUser=(email,password)=>{
        return createUserWithEmailAndPassword(auth,email,password); //its firebase function
      }

       step-2:set createUser in context object

       step-3: In register/signUp file: call the function in eventHandler function:

        {before call follow step-9:}

         createUser(email,password)
        .then(result=>{
          const loggedUser=result.user;
          console.log(loggedUser);
        })
        .catch(error=>{
          console.log(error)
        })
    ```

13. For reset Error

    ```sh
        setError("") //call it before validation
    ```

14. for Login/signIn

    ```sh
      step-1: In AuthProvider file
      const signIn=(email,password)=>{
        return signInWithEmailAndPassword(auth,email,password); //its firebase function
      }

       step-2:set signIn in context object

       step-3: In signIn/Login file: call the function in eventHandler function:

        {before call follow step-9:}

         signIn(email,password)
        .then(result=>{
          const loggedUser=result.user;
          console.log(loggedUser);
          form.reset()
        })
        .catch(error=>{
          console.log(error)
        })
    ```

15. For logOut: we will use logout in header file

    ```sh
      step-1: In AuthProvider file
      const logOut=()=>{
      return signOut(auth)
      }
      step-2:set logOut in context object
      step-3:In header file: call the function in eventHandler button:
      {before call follow step-9:}

      {user && <span>Welcome{user.email} <button onClick={handleLogOut}>Sing Out</button> </span>}

    ```

### Here Just set up firebase sign Up, Login & LogOut using Context API from one file & step-11 still null

16. For set value in step-11 user need call outside api by useEffect

    what's the purpose of onAuthStateChanged in Firebase authentication?
    Answer: It listens for changes in the user authentication state.

    What's the work of unsubscribe? Answer: catch the changes

    ```sh
      /* observer user auth state */
    useEffect(()=>{
     const unsubscribe= onAuthStateChanged(auth,currentUser=>{
        setUser(currentUser)
      })
      /* stop observing while unmounting */
      return ()=>{
        return unsubscribe()
      }
    },[])
    ```

17. Private Route & Navigate after Login

    ```sh
      step-1: create file
      const PrivateRoute = ({children}) => {
      const {user}=useContext(AuthContext);
      if(user){
        return children;
      }
      return <Navigate to='/login' replace={true}></Navigate>;
    };
    step-2: do private which route want to private
     <PrivateRoute>
            <CheckOut></CheckOut>
    </PrivateRoute>
    ```

18. For ignore Re-loading issue

    ```sh
        step-1: In AuthProvider file create a useState
        const [loading,setLoading]=useState(true)

        step-2: In AuthProvider file set setLoading(true) in createUser , signIn & logOut
        > setLoading(true)

        step-3: If state gonna change then call setLoading in useEffect
        > setLoading(false)

        step-4: For use it now call it in context
        loading,

        step-5:use it PrivateRoute
        const {user,loading}=useContext(AuthContext)
        if(loading){
        return <progress className="progress w-56"></progress>;
      }
    ```

19. After Login where I want to go

    ```sh
          useNavigate from react Router dom
      step-1: In login file set useNavigate state
      const navigate=useNavigate()

      step-2: In Login file call it in below of signIn(email,password) & within   .then(result=>{})
      > navigate('/')
    ```

20. After Login Redirect Navigate to the right route

    ```sh
      step-1: In login & PrivateRoute both file set
        > const location=useLocation()
        check location: console.log(location)

      step-2:In PrivateRoute set state={{from: location}} replace in Navigate
      return<Navigate to="/login" state={{from: location}} replace></Navigate>

      step-3: In login file
      const navigate=useNavigate();
      const from=location.state?.from?.pathname || '/'


      step-4: In login file call it in signIn/login function
       navigate(from,{replace:true})
    ```

21. Host your react app to firebase and Show password

    ```sh
      just one time need install in pc
      step-1: npm install -g firebase-tools
      step-2: firebase login

      for each project one time
       * HOSTING
        * --------------------
        * One time per PC
        * 1. npm install -g firebase-tools
        * 2. firebase login
        *
        * For each project one time
        * 1. firebase init
        * 2. proceed
        * 3. hosting: firebase (up and down arrow) use space bar to select
        * 4. existing project
        * 5. select the project careful
        * 6. which project as public directory: dist
        * 7. single page application: yes
        * 8. continuous deployment: no
        *
        * For every time deploy
        * 1. npm run build
        * 2. firebase deploy
    ```

22. show password

    ```sh
      step-1: set state
        const [show,setShow]=useState(false)

      step-2: set onClick button
      <p onClick={()=>setShow(!show)}><small>
            {
              show? <span>Hide Password</span>:<span>Show password</span>
            }
            </small></p>

      step-3: set input type with ternary condition
        type={show? "text": "password"}
    ```

23. Accept Terms and conditions

    ```sh
    step-1: create a component >set Route >In route set a link
    > <p>Go back to <Link to="/register">Register</Link></p>

    step-2: In register file create a checkbox with onClick handler
    > <Form.Group className="mb-3" controlId="formBasicCheckbox">
          <Form.Check
          onClick={handleAccepted}
          type="checkbox"
          name='a'
          label={<>Accept<Link to="/terms">Terms & condition</Link></>} />
        </Form.Group>

    step-3: Declare a state in register file
    > const [accepted,setAccepted]=useState(false);

    step-4: add function & call for checked
    >const handleAccepted=event=>{
    //console.log(event.target.checked)
    setAccepted(event.target.checked);
    }

    step-5: set button disable if not accept term & condition
    > <Button variant="primary" disabled={!accepted} type="submit">
          Register
        </Button>
    ```

24. firebase setup
    ```sh
    step-1: firebase init
    step-2: npm run build
    step-3: firebase deploy
    ```

# VS code set up

- Word wrap
- cursor expand
- Prettier - Code formatter
- formatter: format on save ,prettier formatter
- vs code font family
- Mouse Wheel zoom
- mini map
- Material icon theme
- Path Intelligence
- Markdown Preview Enhanced
- Image preview
- Markdown Preview Enhanced
- Live server
- code runner
- Code Spell Check
- Tailwind CSS intelligence
- Learn with sumit
- Terminal set up
- React Extension Pack
- ES7+ React/Redux/React-Native snippets
- React Native Tools
- Auto Import
- Auto Rename Tag
- ESLint
- npm intelligence
- Postman
- Stylelint
- VSCode React Refactor
