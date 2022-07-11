# Project: Week 5: To-do list application
## Introduction
As of now, you have completed Project Week 4 and now have a backend implementation where your todo lists are saved to a file. The next step is to create backend services that will return todo lists we have saved to file. If you run the solution from week 4 you'll notice that everytime you create new todo lists, the file they are stored in is updated, but your aren't able to see the currently existing todo lists stored.  By adding these new backend services this week we'll be able to see the stored todo list entries and allow users to query for specific todo lists. In this week's assignmment you'll create two backend services 1) return all todo lists, 2) return all todo lists with a specified name.


## Requirements
Feature requirements (Week5 task is complete when you):
+ Create a backend service to handle a GET request to return all todo lists
+ Create a backend service to handle a GET request to return todo lists that match the name sent as a parameter to this request
Optional
+ From the front-end call the back-end service to get all todo lists currently stored when a user opens the Home page
+ Create UI components (a textbox and a button) in the front-end to facilitate searching 

Implementation requirements:
+ Continue using the front-end and back-end frameworks from week 4.

## Instructions

### GET Service - Return all Services

#### Implementation

1. Open to-do-list/backend/server.js
2. Go to the GET listener "app.get("/get/items", getItems)"
3. At the comment "//begin here" copy/paste/type the following code to read in the todo lists stored in the database.json file:
```
    var data = fs.readFileSync('database.json');
```
4. Return a response to whoever called the data we just read in, we will return the data from the file but parsed as JSON data:
```
    response.json(JSON.parse(data));
```
#### Testing
We will test this service using the curl utility.  The curl utility is quite useful because it can send requests to services, simulating consuming applications that would utilize the backend service.

1. Stop the backend service if it's currently running (Ctrl-C the terminal/command window where you did the last "npm start" for the backend)
2. Start the backend service, go to the "to-do-list/backend" directory and install required packages and start the backend:
```
    npm install
    npm start
```

3. Open another terminal or command window.  Type this curl command to send a request to this service:
```
    curl http://localhost:8080/get/items
```

### GET Service - Search for and Return a ToDo List

#### Implementation
1. Open to-do-list/backend/server.js
2. Go to the GET listener "app.get("/get/searchitem",searchItems)"
3. On the line after the comment "//begin here" copy/paste/type the following code, this will retrieve a parameter passed to this service, this parameter will be the name of the Todo List we will search for:
```
    var searchField = request.query.taskname;
```
4. Continue editing this function by adding the following to read in the database 
```
    var json = JSON.parse (fs.readFileSync('database.json'));
```
5. Add the following to take the data from the database and apply a filter, this will seperate out only the Todo lists that match our search parameter given to the backend service and stored in "searchField":
```
    var returnData = json.filter(jsondata => jsondata.Task === searchField);
```
6. Whether we have data to return (i.e. todo lists that matches the name we're looking for) or not (i.e. there were no todo lists with the name), we return a response to whoever called this service with the following:
```
    response.json(returnData);
```

#### Testing


1. Stop the backend service if it's currently running (Ctrl-C the terminal/command window where you did the last "npm start" for the backend)

2. Start the backend service, go to the "to-do-list/backend" directory and install required packages and start the backend (you can skip the npm install if you ran that already above):
```
    npm install
    npm start
```

3. Open another terminal or command window.  
Contruct a curl command to search for a task: 
``` 
    curl http://localhost:8080/get/searchitem?taskname=<nametosearchfor>
```

So for example, if you want to search for todo lists with a name of "hello", your command would be:
```
    curl http://localhost:8080/get/searchitem?taskname=hello
```

If you want to search for a task name with a space in it, for example "hello world" you will need to use the html code for a space (%20) in your curl command, like this: 
```
    curl http://localhost:8080/get/searchitem?taskname=hello%20world
```

### Optional - Use the UI to Call the Backend Service to Return All Todo Lists

#### Implementation
1. Open the front end component to-do-list/src/component/TodoData.js, on the line after the comment "//begin here" copy/paste/type the following code:
```
        const [todos, setTodos] = useState([]);
        
        useEffect( () => { 
            async function fetchData() {
                try {
                    const res = await Axios.get('http://localhost:8080/get/items'); 
                    setTodos(JSON.stringify(res.data));
                    console.log(JSON.stringify(res.data));
                } catch (err) {
                    console.log(err);
                }
            }
            fetchData();
        }, []);
        return <div>{todos}</div>
```
2. Note the above code does a number of things, it makes use of the "useEffect" hook in react, and the await keyword, this combination is essentially telling react to wait for a call to a backend service to complete, then proceeds with the rest of the render.  Remember that nodeJS is asynchronous platform, so statements can get executed before data is prepared and ready to return. In the case of our Axios.get above, if we didn't have the await in front of it then the rest of the code will proceed and attempt to render before our response is returned from the backend service.  To solve this we said that this function is asynchronous and hence we will receive a response from the backend service before proceeding.

3. An alternative is you could do something like we've seen in other services, store the response in state and update it as the data is returned, an example of this will be used in the search functionality next.

#### Testing

1. Go to the "to-do-list" directory and run the front-end and run:
```
    npm install
    npm start
```

2. Open a terminal or command window and go to the to-do-list/backend directory and run the backend:
```
    npm start
```

3. Go to a browser and open the front-end, if not open already, http://localhost:3000, this should bring up the home page. Go to the top navigation bar and click on the "TodoPage"

4. Notice on page load that the top of the page is populated with all of your tasks, saved from the backend service.

### Optional - Use the UI to search for a ToDo list

#### Implementation

1. Open the front end component to-do-list/src/component/SearchTodo.js, on the line after the comment "//begin here" copy/paste/type the following code:
```
            e.preventDefault();  
            // HTTP Client to send a GET request
            Axios({
            method: "GET",
            url: "http://localhost:8080/get/searchitem",
            headers: {
                "Content-Type": "application/json" 
            },
            params: {
                taskname: this.state.content
            }
            }).then(res => {
            this.setState({
                tmpdata: JSON.stringify(res.data),
                });
        
            });
```
2. Some things to note, there are some UI components defined in this file, the main things they will do is submit a form which will trigger the call to the searchitem backend service, as part of that submit we will take the name of the Todo to search for from the "this.state.content" parameter, the a user would type in the UI text box.
3. Note also we have state associated with this component "tmpdata", this state will be set to the data returned from the backend service via the "this.setState({tmpdata: JSON.stringify(res.data),});" code we just put in the HandleSubmit method.
4. We will use this state in the render function, underneath the search UI components you'll see "<div>{this.state.tmpdata}</div>", this is empty initially, because we haven't searched for anything, but once you supply a search parameter and click the "Search" button, we will set the state in the HandleSubmit, which will then update the state in our div to hold the return data from the backend service for the search.


#### Testing
1. Go to the "to-do-list" directory and run the front-end and run:
```
    npm install
    npm start
```

2. Open a terminal or command window and go to the to-do-list/backend directory and run the backend:
```
    npm start
```

3. Go to a browser and open the front-end, if not open already, http://localhost:3000, this should bring up the home page. Go to the top navigation bar and click on the "TodoPage"

4. Notice the input text box and button that will search for a Todo list in the backend. Type a task name that you know exists or doesn't exist and click the button. (Note if you left the frontend and backend services running after completing the lab steps above make sure you refresh the page so the changes you made load correctly in the browser)

5. Observe the returned value in the div section below the search UI, it will be updated in real-time after we submit the form, returning with the data obtained from the backend.

## Pre-session Material
What is a REST API
https://www.redhat.com/en/topics/api/what-is-a-rest-api

Rest APIS
https://www.ibm.com/cloud/learn/rest-apis

Microservices Architecture
https://www.ibm.com/cloud/architecture/architectures/microservices

Modernizing Applications
https://www.ibm.com/cloud/architecture/architectures/application-modernization

