# Welcome to Advanced GoLang Exercise :rocket:

<br/>

***Note:*** Some of the questions on the left would require you write go programs and/or run go commands. 
To do this, follow the below steps:

1. Create a file with `.go` extension. For example: `main.go`
2. Write your program
3. Open the terminal (see instructions below)

   ***Mac users:***<br/>
   Ctrl + ` 

   ***Windows/Linux users:***<br/>
   Ctrl + Shift + ` 

4. To run your program, type `go run <name_of_the_file>` in the terminal. For example - `go run main.go`.

  You would be able to see the output/errors(if any) in the console.

  All the best! :muscle:


This project has 2 files, main.go and model.go that have incomplete code, which students need to complete.


Complete the main.go and model.go files such that go test gives no error.


Note: You can use go mod tidy to download all dependencies in your code, that are specified in the go.mod file.



main.go

```go
package main

import (
    "github.com/gorilla/mux"
    "log"
    "net/http"
)

type Task struct {
    ID          int    `json:"id"`
    Name        string `json:"name"`
    Description string `json:"description"`
    DueDate     string `json:"due_date"`
}

var tasks []Task
var currentID int

type App struct {
    Router *mux.Router
}

func (app *App) handleRoutes() {
    app.Router.HandleFunc("/tasks", app.getTasks).Methods("GET")
    app.Router.HandleFunc("/task/{id}", app.readTask).Methods("GET")
    app.Router.HandleFunc("/task", app.createTask).Methods("POST")
    app.Router.HandleFunc("/task/{id}", app.updateTask).Methods("PUT")
    app.Router.HandleFunc("/task/{id}", app.deleteTask).Methods("DELETE")
}

func (app *App) Initialise(initialTasks []Task, id int) {
    tasks = initialTasks
    currentID = id
    app.Router = mux.NewRouter().StrictSlash(true)
    app.handleRoutes()
}
func main() {
    app := App{}
    tasks, id := CreateInitialTasks()
    app.Initialise(tasks, id)
    app.Run("localhost:10000")
}

func (app *App) Run(address string) {
    log.Fatal(http.ListenAndServe(address, app.Router))
}

func sendResponse(w http.ResponseWriter, statusCode int, payload interface{}) {
    // your code goes here
}

func sendError(w http.ResponseWriter, statusCode int, err string) {
    // your code goes here
}

func (app *App) getTasks(writer http.ResponseWriter, request *http.Request) {
    // your code goes here
}

func (app *App) createTask(writer http.ResponseWriter, r *http.Request) {
    // your code goes here
}

func (app *App) readTask(writer http.ResponseWriter, request *http.Request) {
    // your code goes here
}

func (app *App) updateTask(writer http.ResponseWriter, request *http.Request) {
    // your code goes here
}

func (app *App) deleteTask(writer http.ResponseWriter, request *http.Request) {
    // your code goes here
}
```

model.go

```go

package main

func CreateInitialTasks() ([]Task, int) {
    tasks := []Task{
        {ID: 1, Name: "Create project proposal", Description: "Write a proposal for the new project", DueDate: "2022-02-01"},
        {ID: 2, Name: "Design website layout", Description: "Create a layout for the company website", DueDate: "2022-03-01"},
        {ID: 3, Name: "Implement payment system", Description: "Integrate a payment system into the website", DueDate: "2022-04-01"},
        {ID: 4, Name: "Conduct user testing", Description: "Gather feedback from user testing to improve the website", DueDate: "2022-05-01"},
        {ID: 5, Name: "Launch website", Description: "Make the website live for the public", DueDate: "2022-06-01"},
        {ID: 6, Name: "Evaluate website performance", Description: "Collect data and analyze website performance", DueDate: "2022-07-01"},
    }
    return tasks, 6
}

func getTasks() ([]Task, error) {
    // your code goes here
}

func (t *Task) createTask() error {
    currentID = currentID + 1
    t.ID = currentID
    tasks = append(tasks, *t)
    return nil
}

func (t *Task) getTask() error {
    // your code goes here
}

func (t *Task) updateTask() error {
    // your code goes here
}

func (t *Task) deleteTask() error {
    // your code goes here
}

```