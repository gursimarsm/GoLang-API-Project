# main.go

```go

package main

import (
    "encoding/json"
    "github.com/gorilla/mux"
    "log"
    "net/http"
    "strconv"
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
    response, _ := json.Marshal(payload)
    w.Header().Set("Content-type", "application/json")
    w.WriteHeader(statusCode)
    w.Write(response)
}

func sendError(w http.ResponseWriter, statusCode int, err string) {
    errorMessage := map[string]string{"error": err}
    sendResponse(w, statusCode, errorMessage)
}

func (app *App) getTasks(writer http.ResponseWriter, request *http.Request) {
    tasks, err := getTasks()
    if err != nil {
        sendError(writer, http.StatusInternalServerError, err.Error())
        return
    }
    sendResponse(writer, http.StatusOK, tasks)
}

func (app *App) createTask(writer http.ResponseWriter, r *http.Request) {
    var p Task

    err := json.NewDecoder(r.Body).Decode(&p)
    if err != nil {
        sendError(writer, http.StatusBadRequest, "Invalid request payload")
        return
    }
    err = p.createTask()
    if err != nil {
        sendError(writer, http.StatusInternalServerError, err.Error())
        return
    }
    sendResponse(writer, http.StatusCreated, p)
}

func (app *App) readTask(writer http.ResponseWriter, request *http.Request) {
    vars := mux.Vars(request)
    key, err := strconv.Atoi(vars["id"])
    if err != nil {
        sendError(writer, http.StatusBadRequest, "invalid task ID")
        return
    }

    t := Task{ID: key}
    err = t.getTask()
    if err != nil {
        sendError(writer, http.StatusNotFound, err.Error())
        return
    }
    sendResponse(writer, http.StatusOK, t)
}

func (app *App) updateTask(writer http.ResponseWriter, request *http.Request) {
    vars := mux.Vars(request)
    key, err := strconv.Atoi(vars["id"])
    if err != nil {
        sendError(writer, http.StatusBadRequest, "invalid task ID")
        return
    }
    var t Task
    err = json.NewDecoder(request.Body).Decode(&t)
    if err != nil {
        sendError(writer, http.StatusBadRequest, "Invalid request payload")
        return
    }
    t.ID = key
    err = t.updateTask()
    if err != nil {
        sendError(writer, http.StatusInternalServerError, err.Error())
        return
    }
    sendResponse(writer, http.StatusOK, t)
}

func (app *App) deleteTask(writer http.ResponseWriter, request *http.Request) {
    vars := mux.Vars(request)
    key, err := strconv.Atoi(vars["id"])
    if err != nil {
        sendError(writer, http.StatusBadRequest, "invalid task ID")
        return
    }
    t := Task{ID: key}
    err = t.deleteTask()
    if err != nil {
        sendError(writer, http.StatusNotFound, err.Error())
        return
    }
    sendResponse(writer, http.StatusOK, map[string]string{"result": "successful deletion"})
}

```

# model.go

```go

package main

import "errors"

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
    return tasks, nil
}

func (t *Task) createTask() error {
    currentID = currentID + 1
    t.ID = currentID
    tasks = append(tasks, *t)
    return nil
}

func (t *Task) getTask() error {
    id := t.ID
    for _, task := range tasks {
        if task.ID == id {
            t.DueDate = task.DueDate
            t.Name = task.Name
            t.Description = task.Description
            return nil
        }
    }
    return errors.New("task not found")
}

func (t *Task) updateTask() error {
    id := t.ID
    for index, task := range tasks {
        if task.ID == id {
            task.DueDate = t.DueDate
            task.Name = t.Name
            task.Description = t.Description
            tasks[index] = task
            return nil
        }
    }
    return errors.New("task not found")
}

func (t *Task) deleteTask() error {
    id := t.ID
    indexToBeDeleted := -1
    for index, task := range tasks {
        if task.ID == id {
            indexToBeDeleted = index
        }
    }
    if indexToBeDeleted == -1 {
        return errors.New("task not found")
    }
    tasks = append(tasks[:indexToBeDeleted], tasks[indexToBeDeleted+1:]...)
    return nil
}

```