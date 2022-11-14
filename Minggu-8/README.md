# MINGGU Ke-8 

## React JS

### Context
Context itu seperti variable global yang bisa diakses dimana saja tanpa harus memparsing props ke setiap komponen.<br/>
Konteks dirancang untuk berbagi data yang dapat dianggap "global" untuk pohon komponen React, seperti pengguna yang diautentikasi saat ini, tema, atau bahasa pilihan. Misalnya, dalam kode di bawah ini kita secara manual memasukkan prop "tema" untuk menata komponen Tombol:

    class App extends React.Component {
    render() {
        return <Toolbar theme="dark" />;
    }
    }

    function Toolbar(props) {
    // The Toolbar component must take an extra "theme" prop
    // and pass it to the ThemedButton. This can become painful
    // if every single button in the app needs to know the theme
    // because it would have to be passed through all components.
    return (
        <div>
        <ThemedButton theme={props.theme} />
        </div>
    );
    }

    class ThemedButton extends React.Component {
    render() {
        return <Button theme={this.props.theme} />;
    }
    }

Dengan menggunakan konteks, kita dapat menghindari melewatkan props melalui elemen perantara:

    // Context lets us pass a value deep into the component tree
    // without explicitly threading it through every component.
    // Create a context for the current theme (with "light" as the default).
    const ThemeContext = React.createContext('light');

    class App extends React.Component {
    render() {
        // Use a Provider to pass the current theme to the tree below.
        // Any component can read it, no matter how deep it is.
        // In this example, we're passing "dark" as the current value.
        return (
        <ThemeContext.Provider value="dark">
            <Toolbar />
        </ThemeContext.Provider>
        );
    }
    }

    // A component in the middle doesn't have to
    // pass the theme down explicitly anymore.
    function Toolbar() {
    return (
        <div>
        <ThemedButton />
        </div>
    );
    }

    class ThemedButton extends React.Component {
    // Assign a contextType to read the current theme context.
    // React will find the closest theme Provider above and use its value.
    // In this example, the current theme is "dark".
    static contextType = ThemeContext;
    render() {
        return <Button theme={this.context} />;
    }
    }

- Membuat Context<br/>
  Di dalam context folder ini, kita akan membuat beberapa file di dalamnya. Untuk langkah ketiga ini, saya hanya akan membuat satu file yang dinamai TodoContext.js. Ini hanya beberapa baris kode, sangat mudah sekali.

        import { createContext } from "react"

        export default createContext()
- Membuat Types<br/>
  Types ini berfungsi untuk mendefinisikan apa saja yang akan dilakukan oleh aplikasi kita.<br/>
  
        /**
        * @description   Types is just for constant that for dispatching our action in order to reducing a Misspelling
        */

        export const GET_TODOS = "GET_TODOS"
        export const SET_TODO_TITLE = "SET_TODO_TITLE"
        export const CLEAR_TODO_TITLE = "CLEAR_TODO_TITLE"
        export const CREATE_TODO = "CREATE_TODO"
        export const DELETE_TODO = "DELETE_TODO"

- Membuat Reducer<br/>
  Reducer di ReactJS mempunyai arti untuk menentukan state-state yang akan berubah di dalam aplikasi kita. Untuk kasus ini, jika kamu ingin mengubah state-state, maka kamu harus mengkontrol semuanya di sini.

        // Dont Forget import the types
        import {
        SET_TODO_TITLE,
        GET_TODOS,
        CREATE_TODO,
        DELETE_TODO,
        CLEAR_TODO_TITLE
        } from "./TodoTypes"

        export default (state, { type, payload }) => {
        switch (type) {
            // Get all todos
            case GET_TODOS:
            return {
                ...state,
                loading: false,
                todos: payload
            }
            // Set Todo title for form
            case SET_TODO_TITLE:
            return {
                ...state,
                title: payload
            }
            // Create a new todo
            case CREATE_TODO:
            return {
                ...state,
                todos: [payload, ...state.todos]
            }
            // Clear todo title after create
            case CLEAR_TODO_TITLE:
            return {
                ...state,
                title: ""
            }
            // Delete a todo
            case DELETE_TODO:
            return {
                ...state,
                todos: state.todos.filter((todo) => todo.id !== payload)
            }
            default:
            return state
        }
        }

- Membuat State<br/>
   Terdapat file yang berisikan state-state, fungsi-fungsi, dan beberapa nilai yang akan digunakan oleh si komponen nantinya. Di sinilah kita membuat macam halnya seperti variable global dengan context, berikut kodenya.

        import React, { useReducer } from "react"

        // Bring the context
        import TodoContext from "./TodoContext"

        // Bring the reducer
        import TodoReducer from "./TodoReducer"

        // Bring the types
        import {
        SET_TODO_TITLE,
        GET_TODOS,
        CREATE_TODO,
        DELETE_TODO,
        CLEAR_TODO_TITLE
        } from "./TodoTypes"

        const TodoState = ({ children }) => {
        // Define our state
        const initialState = {
            todos: [],
            title: "",
            loading: true
        }

        // Dispatch the reducer
        // This come from useReducer from ReactJS
        const [state, dispatch] = useReducer(TodoReducer, initialState)

        // Set the title for the new todo
        // This will change whenever user type in the form later
        const setTodoTitle = (payload) => {
            dispatch({ type: SET_TODO_TITLE, payload })
        }

        // Get todos
        const getTodos = async () => {
            try {
            const todos = await fetch(
        "https://jsonplaceholder.typicode.com/todos?_limit=5"
            )
            const toJSON = await todos.json()

            dispatch({ type: GET_TODOS, payload: toJSON })
            } catch (err) {
            console.error(err.message)
            }
        }

        // Create todo
        const createTodo = async (title) => {
            const newTodo = {
            title,
            completed: false
            }

            try {
            const todo = await fetch("https://jsonplaceholder.typicode.com/todos", {
                method: "POST",
                headers: {
                "Content-Type": "application/json"
                },
                body: JSON.stringify(newTodo)
            })
            const toJSON = await todo.json()

            dispatch({ type: CLEAR_TODO_TITLE })
            dispatch({ type: CREATE_TODO, payload: toJSON })
            } catch (err) {
            console.error(err.message)
            }
        }

        // Delete Todo
        const deleteTodo = async (id) => {
            try {
            await fetch(`https://jsonplaceholder.typicode.com/todos/${id}`, {
                method: "DELETE"
            })

            dispatch({ type: DELETE_TODO, payload: id })
            } catch (err) {
            console.error(err.message)
            }
        }

        // Destruct the states
        const { todos, title, loading } = state

        // Here's where we gonna use this state and funcitons to dealing with the context
        // The context will wrapping our entire application with this component and accept children in it
        // Anything state or function, must be passed in to value props in this provider in order to use it
        // NOTE: PLEASE NOTICE, IF YOU DIDN'T PASS THE STATE OR THE FUNCTION in THIS VALUE PROPS, YOU WILL GET AN ERROR
        return (
            <TodoContext.Provider
            value={{
                todos,
                title,
                loading,
                getTodos,
                setTodoTitle,
                createTodo,
                deleteTodo
            }}
            >
            {children}
            </TodoContext.Provider>
        )
        }

        export default TodoState

  State ini digunakan untuk mendeklarasikan state-state yang akan kita gunakan, fungsi-fungsi yang beritenraksi dengan server, dan beberapa logic untuk aplikasi ini.<br/>
  Ada beberapa hal yang digunakan saat menggunakan context, yaitu:
  1. State.
  2. Reducer.
  3. Type.
  4. CreateContext().


### Test
React Testing Library adalah seperangkat helpers yang memungkinkan Anda mengetes komponen pada React tanpa bergantung pada detail implementasinya. Pendekatan ini membuat refactoring menjadi mudah dan juga mendorong Anda untuk menerapkan best practices untuk aksesbilitas. <br/>

Ada beberapa cara untuk mengeteskomponen pada React. Secara umum, terbagi menjadi dua kategori:
- Rendering component trees di dalam environment tes yang sudah disederhanakan dan ditegaskan pada keluarannya.
- Menjalankan aplikasi lengkap di dalam environment peramban asli (juga dikenal sebagai tes “end-to-end”).

Unit testing bertujuan untuk mendapatkan hasil yang sesuai ekspektasi agar suatu software dapat berjalan dengan baik.<br/>

Unit testing memiliki tiga teknik yang disebut black box testing, white box testing dan gray box testing.
1. Black Box Testing<br/>
   Black box testing adalah sebuah pengujian yang tidak perlu melihat dan memahami suatu software lebih dalam, pengujiannya melalui user interface, input dan output

2. White Box Testing<br/>
   White box testing bersifat transparan jadi kita bisa melihat suatu sistem dari awal sampai akhir untuk dilakukan testing. Untuk testingnya dilakukan untuk menguji struktur internal, desain, fungsi dan detail implementasi dari sebuah aplikasi

3. Grey Box Testing<br/>
   Grey box testing ini merupakan sebuah perpaduan antara black box testing dan white box testing. Pengujiannya ini digunakan untuk eksekusi test, resiko dan metode penilaian

- Kelebihan Unit Testing
  1. Membantu menulis kode lebih baik
  1. Membantu menemukan sebuah bug sebelumnya
  1. Membantu mendeteksi bug regression (bug yang menyebabkan fitur menjadi berhenti)
  1. Membuat kode lebih mudah di refactor
  1. Membuat penulisan kode lebih efisien
- Kekurangan Unit Testing
  1. Membutuhkan waktu untuk melakukan tes
  1. Sangat sulit untuk melakukan test terhadap legacy code
  1. Test membutuhkan waktu yang banyak untuk melakukan maintenance
  1. Sedikit sulit untuk melakukan test kode berbasi GUI
  1. Unit Testing tidak menangkap semua error

1. Set Up React Testing
    -  Installasi<br/>
        1. Set Up dengan Create React App
       
                npm install --save-dev react-test-renderer
        2. Setup tanpa Buat Aplikasi React

                npm install --save-dev jest babel-jest @babel/preset-env @babel/preset-react react-test-renderer
       
            Package.json akan terlihat seperti ini (di mana ```<current-version>```nomor versi terbaru sebenarnya untuk paket tersebut). Silakan tambahkan skrip dan entri konfigurasi

                {
                    "dependencies": {
                        "react": "<current-version>",
                        "react-dom": "<current-version>"
                    },
                    "devDependencies": {
                        "@babel/preset-env":                    "<current-version>",
                        "@babel/preset-react":                  "<current-version>",
                        "babel-jest":                   "<current-version>",
                        "jest":     "<current-version>",
                        "react-test-renderer":                  "<current-version>"
                    },
                    "scripts": {
                        "test": "jest"
                    }
                }
            Dan pada file babel.config.js seperti berikut:

                module.exports = {
                    presets: [
                        '@babel/preset-env',
                        ['@babel/preset-react',                     {runtime: 'automatic'}],
                    ],
                };