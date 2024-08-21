<!-- START doctoc generated TOC please keep comment here to allow auto update -->
<!-- DON'T EDIT THIS SECTION, INSTEAD RE-RUN doctoc TO UPDATE -->
**Table of Contents**  *generated with [DocToc](https://github.com/thlorenz/doctoc)*

  - [DJANGO Y REACT](#django-y-react)
- [Creación del entorno virtual](#creaci%C3%B3n-del-entorno-virtual)
- [Django](#django)
  - [Configurando Django](#configurando-django)
  - [Django REST Framework](#django-rest-framework)
  - [Middleware CORS - django-cors-headers](#middleware-cors---django-cors-headers)
  - [Utilizando el ORM](#utilizando-el-orm)
  - [Creación de API](#creaci%C3%B3n-de-api)
  - [Añadir módulo docs automático](#a%C3%B1adir-m%C3%B3dulo-docs-autom%C3%A1tico)
- [React](#react)
  - [Comunicación entre servidores](#comunicaci%C3%B3n-entre-servidores)
  - [Crear](#crear)
  - [Editar y Eliminar](#editar-y-eliminar)
  - [React-hot-toast (mensajes)](#react-hot-toast-mensajes)
  - [Tailwind](#tailwind)

<!-- END doctoc generated TOC please keep comment here to allow auto update -->

## DJANGO Y REACT

# Creación del entorno virtual

```bash

pip install virtualenv

```

comando + nombre de carpeta

```bash

python -m venv venv

```

Activamos el entorno virtual para luego ejecutar los comandos de django.

# Django

Para inicializar un proyecto en django lo primero que tenemos que hacer es ejecutar el siguiente comando, el punto es para que no cree una carpeta.

```bash
django-admin startproject django_crud_api .
```

Luego para ejecutarlo, en la termina vamos hacia la carpeta que tiene el manage.py

```bash
python manage.py runserver
```

## Configurando Django

Creando una app

```bash
python manage.py startapp nombreapp
```

Agregando la app a django

Para ello vamos a ir a las settings.py en django_crud_api y agregamos en INSTALLED_APPS el parametro de como se llama la app, en nuestro caso task.

```python
INSTALLED_APPS = [
    'django.contrib.admin',
    'django.contrib.auth',
    'django.contrib.contenttypes',
    'django.contrib.sessions',
    'django.contrib.messages',
    'django.contrib.staticfiles',
    'task'
]
```

Luego realizaremos una migración.

If your database doesn't exist yet, migrate creates all the necessary tables to match your model definitions.

Otherwise if the database already exists, migrate updates the existing table definitions to match the model definitions -- i.e. maybe you added a field to one of your models, so migrate would add that column to the database table.

```python
python manage.py migrate
```

## Django REST Framework

Sitio alternativo pero con documentación directa a REST

<a>https://www.django-rest-framework.org</a>

Instalamos el modulo

```bash
pip install djangorestframework
```

## Middleware CORS - django-cors-headers

Para conectar los dos servidores de desarrollo necesitamos habilitar el CORS.

```bash
pip install django-cors-headers
```

Para configurarlo vamos a incluir en installed aps -> rest_framework y corsheaders **añadimos**.

```python
INSTALLED_APPS = [
    'django.contrib.admin',
    'django.contrib.auth',
    'django.contrib.contenttypes',
    'django.contrib.sessions',
    'django.contrib.messages',
    'django.contrib.staticfiles',
    'corsheaders',
    'rest_framework',
    'task'
]
```

Luego en MIDDLEWARE debemos añadir el cors pero no en cualquier lado sino que deberia estar antes del django.middleware.common.CommonMiddleware

```python
MIDDLEWARE = [
    'django.middleware.security.SecurityMiddleware',
    'django.contrib.sessions.middleware.SessionMiddleware',
    "corsheaders.middleware.CorsMiddleware",
    "django.middleware.common.CommonMiddleware",
    'django.middleware.csrf.CsrfViewMiddleware',
    'django.contrib.auth.middleware.AuthenticationMiddleware',
    'django.contrib.messages.middleware.MessageMiddleware',
    'django.middleware.clickjacking.XFrameOptionsMiddleware',
]
```

Luego configuremos cual servidor puede conectarse a django. Agregaremos al final

```python
CORS_ALLOWED_ORIGINS = [
    'poner la url aqui'
]
```

## Utilizando el ORM

Crearemos unos modelos para que el orm lo utilice. Para ello vamos a task y luego a models.py

```python
from django.db import models

# Create your models here.

class Task(models.Model):
    title = models.CharField(max_length=200)
    description = models.TextField(blank=True)
    done = models.BooleanField(default=False)
```

Para crear la tabla

Para crear todas las tablas

```bash
python manage.py makemigrations
```

Para crear una en especifico

```bash
python manage.py makemigrations task
```

Al ejecutar esto se creara una carpeta llamada migrations donde habrá un archivo que se encontrara el codigo que ejecutara python para crear nuestra tabla. Aún asi aun no se ejecutó.

Para ello ejecutaremos el siguiente comando

```bash
python manage.py migrate nombredeapp
```

Perfecto.

Ahora iremos al panel de administrador de django localhost:8000/admin

Pero antes tendremos que crear una cuenta.

```bash
python manage.py createsuperuser
```

Ahora ya podemos entrar pero no vamos a ver nuestras tablas por lo que tenemos que agregarlas en admin.py

```python
from django.contrib import admin

# Register your models here.

from .models import Task
admin.site.register(Task)
```

Y listo ya podemos utilzar el crud que viene por defecto en django.

Aún asi al registrar una tarea al leerla nos aparece Task Object. Debemos configurar eso.

Vamos a model y utilizamos def **str**(self):

```python
def __str__(self):
        return self.title
```

## Creación de API

Primero le diremos al modelo de tarea que datos serán seleccionados para que se puedan enviar desde el backend y puedan ser convertidos en json.

Para ello crearemos un archivo serializer.py

```python
from rest_framework import serializers

class TaskSerializer(serializers.ModelSerializer):
    class Meta:
        fields = ('id','title','description', 'done')
```

De esta forma le decimos que datos queremos que los convierta en JSON.

Y si son muuuchos campos

```python
from rest_framework import serializers

class TaskSerializer(serializers.ModelSerializer):
    class Meta:
        fields = '__all__'
```

Le agreraremos un "nombre" al modelo.

```python
from rest_framework import serializers
from .models import Task

class TaskSerializer(serializers.ModelSerializer):
    class Meta:
        model = Task
        fields = ('id','title','description', 'done')
```

Proseguiremos a crear vistas. Las vistas son funciones que le responden algo al cliente, en nuestro caso un CRUD. Al ser una tarea repetitiva, django nos provee una funcionalidad de crear todo el crud de forma automatica.

serializer_class basicamente tiene la clase que ya habia creado, para darle informacion que campos seran consultados importamos el modelo de tarea.

```python
from rest_framework import viewsets
from .serializer import TaskSerializer
from .models import Task

# Create your views here.}

class TaskView(viewsets.ModelViewSet):
    serializer_class = TaskSerializer
    queryset = Task.objects.all()
```

Ahora seguiremos con las URL. Crearemos un archivo urls.py

El código de abajo genera las rutas GET, POST, PUT y DELETE

```python
from django.urls import path, include
from rest_framework import routers
from task import views

router = routers.DefaultRouter()
router.register(r'tasks', views.TaskView, 'tasks')

urlpatterns = [
    path('api/v1', include(router.urls))
]
```

Ahora debemos hacer que la aplicación principal (django) conozca tambien las url.

vamos a su urls.py

```python
from django.contrib import admin
from django.urls import path, include

urlpatterns = [
    path('admin/', admin.site.urls),
    path('tasks/', include('task.urls'))
]
```

Accedemos <a>http://127.0.0.1:8000/tasks/api/v1/tasks/</a>

y si queremos por tarea

<a>http://127.0.0.1:8000/tasks/api/v1/tasks/1/</a>

Podemos hacer el CRUD también ahi con su interfaz.

## Añadir módulo docs automático

Este módulo documenta.

```python
pip install coreapi
```

Vamos a configurarlo. En INSTALLED_APPS, encima de tasks, insertamos el modulo coreapi.

```python
INSTALLED_APPS = [
    'django.contrib.admin',
    'django.contrib.auth',
    'django.contrib.contenttypes',
    'django.contrib.sessions',
    'django.contrib.messages',
    'django.contrib.staticfiles',
    'corsheaders',
    'rest_framework',
    'coreapi',
    'task'
]
```

En tasks urls añadimos las siguientes lineas. /docs con su importación.

```python
from django.urls import path, include
from rest_framework.documentation import include_docs_urls
from rest_framework import routers
from task import views

router = routers.DefaultRouter()
router.register(r'tasks', views.TaskView, 'tasks')

urlpatterns = [
    path('api/v1/', include(router.urls)),
    path('docs/', include_docs_urls(title= 'Task API'))
]
```

Por ultimo añadiremos esto en settings.py

```python
REST_FRAMEWORK = {
    ...: ...,
    "DEFAULT_SCHEMA_CLASS": "rest_framework.schemas.coreapi.AutoSchema",
}
```

Al ejecutar a mi me salio un error, me faltaba un paquete, setuptools

```bash
pip install setuptools
```

# React

Utilizaremos una herramienta moderna como vitesjs para poner React en marcha.

Creamos una nueva terminal.

```bash
npm create vite
```

Seguimos los pasos. Y cuando termine

Done. Now run:

cd client

npm install

npm run dev

## Comunicación entre servidores

Instalaremos los siguientes modulos.

react-router-dom (para tener multiples paginas en el front)
react-hot-toast (obtener mensajitos si eliminamos algo por ejemplo)
axios (para hacer peticiones algo como fetch pero mas sencillo)
react-hook-form (validar inputs del frontend)

```bash
npm i react-router-dom react-hot-toast axios react-hook-form
```

Limpiaremos un poco el proyecto.

src>App.jsx

Eliminamos casi todo el contenido excepto la función App() y su export.

```js
function App() {
  return <div>Hello world!</div>
}

export default App
```

Eliminamos contenido de index.css

Creamos las carpetas **pages**, otra **components**, **api**, todo dentro de src.

Pages para nuestras páginas.
Components para almacenar fracciones de interfaz.
Api para ddefinir que funciones pediran datos al backend.

Dentro de pages creamos dos archivos TaskPage.jsx para listar todas las tareas y TaskFormPage para crear un formulario de tareas.

En TaskPage crearemos un componente basico.

```jsx
export function TasksPage() {
  return <div>TaskPage</div>
}
```

Y en TaskFormPage lo mismo.

```js
export function TaskFormPage() {
  return <div>TaskFormPage</div>
}
```

Ahora llamaremos a esas páginas, para ello vamos a App.jsx

path es el nombre de la ruta.
element es el componente que renderizaremos.

```js
import { BrowserRouter, Routes, Route } from 'react-router-dom'
import { TasksPage } from './pages/TaskPage'
import { TaskFormPage } from './pages/TaskFormPage'

function App() {
  return (
    <BrowserRouter>
      <Routes>
        <Route path='/tasks' element={<TasksPage />} />
        <Route path='/tasks-create' element={<TaskFormPage />} />
      </Routes>
    </BrowserRouter>
  )
}

export default App
```

Y lo podemos comprobar accediendo a la url.

Ahora setearemos la página de inicio. Podríamos hacer lo mismo pero también podemos decirle que redireccione con el componente Navigate.

```js
import { BrowserRouter, Routes, Route, Navigate } from 'react-router-dom'
import { TasksPage } from './pages/TaskPage'
import { TaskFormPage } from './pages/TaskFormPage'

function App() {
  return (
    <BrowserRouter>
      <Routes>
        <Route path='/' element={<Navigate to='/tasks' />} />
        <Route path='/tasks' element={<TasksPage />} />
        <Route path='/tasks-create' element={<TaskFormPage />} />
      </Routes>
    </BrowserRouter>
  )
}

export default App
```

Ahora crearemos una navegación bien sencilla dentro de components.

```js
import { Link } from 'react-router-dom'

export function Navigation() {
  return (
    <div>
      <Link to='/tasks'>
        <h1>Task App</h1>
      </Link>
      <Link to='/tasks-create'>Create task</Link>
    </div>
  )
}
```

Añadimos la navegación en App.jsx

```js
import { BrowserRouter, Routes, Route, Navigate } from 'react-router-dom'
import { TasksPage } from './pages/TaskPage'
import { TaskFormPage } from './pages/TaskFormPage'
import { Navigation } from './components/Navigation'

function App() {
  return (
    <BrowserRouter>
      <Navigation />

      <Routes>
        <Route path='/' element={<Navigate to='/tasks' />} />
        <Route path='/tasks' element={<TasksPage />} />
        <Route path='/tasks-create' element={<TaskFormPage />} />
      </Routes>
    </BrowserRouter>
  )
}

export default App
```

Crearemos un nuevo componente llamado **TaskList.jsx** que nos listara cada tarea.

```js
export function TaskList() {
  return <div>TaskList</div>
}
```

Ahora queremos que cada vez que sea llamado ese componente se ejecute un comportamiento. Para ello importamos useEffect y que cuando se cargue realizar un simple console.log()

```js
import { useEffect } from 'react'

export function TaskList() {
  useEffect(() => {
    console.log('Página cargada')
  }, [])

  return <div>TaskList</div>
}
```

Ahora debemos llamar al componente, lo llamaremos en TaskPage.jsx

```js
import { TaskList } from '../components/TaskList'

export function TasksPage() {
  return <TaskList />
}
```

Ahora vamos a la consola y vemos que **2** mensajes aparecieron y no 1. Ello se debe a que en el modo desarrollo existe un Componente en React llamado React.StrictMode que es el culpable. Se encuentra en main.jsx.

Crearemos un archivo llamado tasks.api.js en api. Utilizaremos axios.

```js
import axios from 'axios'

export const getAllTasks = () => {
  return axios.get('http://localhost:8000/tasks/api/v1/tasks/')
}
```

Vamos a importar la funcion en TaskList para que haga la petición.

```js
import { useEffect } from 'react'
import { getAllTasks } from '../api/tasks.api'

export function TaskList() {
  useEffect(() => {
    async function loadTasks() {
      const res = await getAllTasks()
      console.log(res)
    }

    loadTasks()
  }, [])

  return <div>TaskList</div>
}
```

Nos saldrá error pues el backend no habilitó la entrada de esta petición.+

Vamos a django y en settings.py

```python
CORS_ALLOWED_ORIGINS = ['http://localhost:5173']
```

Para guardar los datos en React no se utilizan variables sino que se usa la función useState.

```js
import { useEffect, useState } from 'react'
import { getAllTasks } from '../api/tasks.api'

export function TaskList() {
  const [tasks, setTasks] = useState()

  useEffect(() => {
    async function loadTasks() {
      const res = await getAllTasks()
      setTasks(res.data)
    }

    loadTasks()
  }, [])

  return <div>TaskList</div>
}
```

Ahora vamos a listarla en html, ver que cambie en useState y le agregué []

```js
import { useEffect, useState } from 'react'
import { getAllTasks } from '../api/tasks.api'

export function TaskList() {
  const [tasks, setTasks] = useState([])

  useEffect(() => {
    async function loadTasks() {
      const res = await getAllTasks()
      setTasks(res.data)
    }

    loadTasks()
  }, [])

  return (
    <div>
      {tasks.map((tasks) => (
        <div key={tasks.id}>
          <h1>{tasks.title}</h1>
          <p>{tasks.description}</p>
        </div>
      ))}
    </div>
  )
}
```

Vamos a ordenar, ahora crearemos un componente para renderizar las tareas.

```js
export function TaskCard({ task }) {
  return (
    <div key={tasks.id}>
      <h1>{tasks.title}</h1>
      <p>{tasks.description}</p>
    </div>
  )
}
```

Y en listcard

```js
import { useEffect, useState } from 'react'
import { getAllTasks } from '../api/tasks.api'
import { TaskCard } from './TaskCard'

export function TaskList() {
  const [tasks, setTasks] = useState([])

  useEffect(() => {
    async function loadTasks() {
      const res = await getAllTasks()
      setTasks(res.data)
    }

    loadTasks()
  }, [])

  return (
    <div>
      {tasks.map((task) => (
        <TaskCard key={task.id} task={task} />
      ))}
    </div>
  )
}
```

Taskcard

```js
export function TaskCard({ task }) {
  return (
    <div>
      <h1>{task.title}</h1>
      <p>{task.description}</p>
      <hr />
    </div>
  )
}
```

## Crear

Iremos a TaskFormPage y haremos un formulario.

```js
export function TaskFormPage() {
  return (
    <div>
      <form action=''>
        <input type='text' placeholder='Title' />
        <textarea rows='3' placeholder='Description'></textarea>
        <button>Save</button>
      </form>
    </div>
  )
}
```

Utilizaremos la librería react-hook-form para validar los datos.

```js
import { useForm } from 'react-hook-form'

export function TaskFormPage() {
  const { register } = useForm()

  return (
    <div>
      <form action=''>
        <input
          type='text'
          placeholder='Title'
          {...register('title', { required: true })}
        />
        <textarea
          rows='3'
          placeholder='Description'
          {...register('description', { required: true })}
        ></textarea>
        <button>Save</button>
      </form>
    </div>
  )
}
```

Manejaremos el botón de save con errores.

```js
import { useForm } from 'react-hook-form'

export function TaskFormPage() {
  const {
    register,
    handleSubmit,
    formState: { errors },
  } = useForm()

  const onSubmit = handleSubmit((data) => {
    console.log(data)
  })

  return (
    <div>
      <form onSubmit={onSubmit}>
        <input
          type='text'
          placeholder='Title'
          {...register('title', { required: true })}
        />
        {errors.title && <span>this field is required</span>}
        <textarea
          rows='3'
          placeholder='Description'
          {...register('description', { required: true })}
        ></textarea>
        {errors.description && <span>this field is required</span>}
        <button>Save</button>
      </form>
    </div>
  )
}
```

Crearemos el servicio de createTask, para ello configuraremos antes axios para que tenga una urlbase y asi no tener que repetir.

```js
import axios from 'axios'

const taskApi = axios.create({
  baseURL: 'http://localhost:8000/tasks/api/v1/tasks/',
})

export const getAllTasks = () => taskApi.get('/')
export const createTask = (task) => taskApi.post('/', task)
```

Ahora vamos al handleSubmit para llamar al servicio.

```js
const onSubmit = handleSubmit(async (data) => {
  const res = await createTask(data)
  console.log(res)
})
```

Por ultimo haremos que se redireccione al inicio para ello utilizaremos react-router-dom, con useNavigate

```js
import { useForm } from 'react-hook-form'
import { createTask } from '../api/tasks.api'
import { useNavigate } from 'react-router-dom'

export function TaskFormPage() {
  const {
    register,
    handleSubmit,
    formState: { errors },
  } = useForm()
  const navigate = useNavigate()

  const onSubmit = handleSubmit(async (data) => {
    await createTask(data)
    navigate('/')
  })

  return (
    <div>
      <form onSubmit={onSubmit}>
        <input
          type='text'
          placeholder='Title'
          {...register('title', { required: true })}
        />
        {errors.title && <span>this field is required</span>}
        <textarea
          rows='3'
          placeholder='Description'
          {...register('description', { required: true })}
        ></textarea>
        {errors.description && <span>this field is required</span>}
        <button>Save</button>
      </form>
    </div>
  )
}
```

## Editar y Eliminar

Modificaremos el TaskCard para dejarlo listo para cuadno pongamos una ruta.

```js
import { useNavigate } from 'react-router-dom'
export function TaskCard({ task }) {
  const navigate = useNavigate()
  return (
    <div onClick={() => {}}>
      <h1>{task.title}</h1>
      <p>{task.description}</p>
      <hr />
    </div>
  )
}
```

Vamos a crear la ruta. Recordemos que se encuentra en App.jsx

```js
<Routes>
  <Route path='/' element={<Navigate to='/tasks' />} />
  <Route path='/tasks' element={<TasksPage />} />
  <Route path='/tasks-create' element={<TaskFormPage />} />
  <Route path='/tasks/:id' element={<TaskFormPage />} />
</Routes>
```

Vamos a hacer la tarjeta navegable.

```js
import { useNavigate } from 'react-router-dom'
export function TaskCard({ task }) {
  const navigate = useNavigate()
  return (
    <div
      onClick={() => {
        navigate(`/tasks/${task.id}`)
      }}
    >
      <h1>{task.title}</h1>
      <p>{task.description}</p>
      <hr />
    </div>
  )
}
```

Y ahora agregaremos el botón de eliminar con un if comparando el parametro de la url para eso utilizaremos un hook llamado useParams

```js
export function TaskFormPage() {
  const { register, handleSubmit, formState: {errors} } = useForm()
	const navigate = useNavigate()
	const params = useParams()

  const onSubmit = handleSubmit(async data => {
    await createTask(data)
		navigate('/')
  })

  return ( ........
```

Agregamos el if, con console log podemos ver que trae params. ejemplo { id: '5'}, si está vacío no trae nada, solo {}

```js
{
  params.id && <button>Delete</button>
}
```

Creamos el servicio. No nos olvidemos que debe terminar con /

```js
export const deleteTask = (id) => taskApi.delete(`/${id}/`)
```

Manejamos el click.

```js
import { useForm } from 'react-hook-form'
import { createTask, deleteTask } from '../api/tasks.api'
import { useNavigate, useParams } from 'react-router-dom'

export function TaskFormPage() {
  const {
    register,
    handleSubmit,
    formState: { errors },
  } = useForm()
  const navigate = useNavigate()
  const params = useParams()

  const onSubmit = handleSubmit(async (data) => {
    await createTask(data)
    navigate('/')
  })

  return (
    <div>
      <form onSubmit={onSubmit}>
        <input
          type='text'
          placeholder='Title'
          {...register('title', { required: true })}
        />
        {errors.title && <span>this field is required</span>}
        <textarea
          rows='3'
          placeholder='Description'
          {...register('description', { required: true })}
        ></textarea>
        {errors.description && <span>this field is required</span>}
        <button>Save</button>
      </form>

      {params.id && (
        <button
          onClick={async () => {
            const accepted = window.confirm('are you sure')
            if (accepted) {
              await deleteTask(params.id)
              navigate('/')
            }
          }}
        >
          Delete
        </button>
      )}
    </div>
  )
}
```

Ahora proseguiremos con editar. Pondremos un if al igual que antes para ver si se está actualizando o creando.

```js
const onSubmit = handleSubmit(async (data) => {
  if (params.id) {
    console.log('actualizando')
  } else {
    await createTask(data)
  }
  navigate('/')
})
```

Crearemos el servicio.

```js
import axios from 'axios'

const taskApi = axios.create({
  baseURL: 'http://localhost:8000/tasks/api/v1/tasks/',
})

export const getAllTasks = () => taskApi.get('/')
export const createTask = (task) => taskApi.post('/', task)
export const deleteTask = (id) => taskApi.delete(`/${id}/`)
export const updateTask = (id, task) => taskApi.put(`/${id}/`, task)
```

Y lo ponenmos dentro del if

```js
export function TaskFormPage() {
  const {
    register,
    handleSubmit,
    formState: { errors },
  } = useForm()
  const navigate = useNavigate()
  const params = useParams()

  const onSubmit = handleSubmit(async (data) => {
    if (params.id) {
      console.log("actualizando")
    } else {
      await createTask(data)
    }
    navigate("/")
  })
```

Haremos como que obtenemos los datos...

```js
import { useEffect } from 'react'
import { useForm } from 'react-hook-form'
import { createTask, deleteTask } from '../api/tasks.api'
import { useNavigate, useParams } from 'react-router-dom'

export function TaskFormPage() {
  const {
    register,
    handleSubmit,
    formState: { errors },
  } = useForm()
  const navigate = useNavigate()
  const params = useParams()

  const onSubmit = handleSubmit(async (data) => {
    if (params.id) {
      console.log('actualizando')
    } else {
      await createTask(data)
    }
    navigate('/')
  })

  useEffect(() => {
    if (params.id) {
      console.log('obteniendo datos')
    }
  })

  return (
    <div>
      <form onSubmit={onSubmit}>
        <input
          type='text'
          placeholder='Title'
          {...register('title', { required: true })}
        />
        {errors.title && <span>this field is required</span>}
        <textarea
          rows='3'
          placeholder='Description'
          {...register('description', { required: true })}
        ></textarea>
        {errors.description && <span>this field is required</span>}
        <button>Save</button>
      </form>

      {params.id && (
        <button
          onClick={async () => {
            const accepted = window.confirm('are you sure')
            if (accepted) {
              await deleteTask(params.id)
              navigate('/')
            }
          }}
        >
          Delete
        </button>
      )}
    </div>
  )
}
```

Crearemos el servicio de getOne.

```js
export const getTask = (id) => taskApi.get(`/${id}/`)
```

Haremos una función asincrona dentro del useEffect.

```js
useEffect(() => {
  async function loadTask() {
    if (params.id) {
      await getTask(params.id)
    }
  }

  loadTask()
})
```

Rellenamos los inputs. desde useForm utilizamos setValue('')

```js
	useEffect(()=>{
		async function loadTask() {
			if (params.id){
				const res = await getTask(params.id)
				setValue('title', res.data.title)
				setValue('description', res.data.description)
			}
		}
```

Quedando todo TaskFormPage así:

```js
import { useEffect } from 'react'
import { useForm } from 'react-hook-form'
import { createTask, deleteTask, getTask, updateTask } from '../api/tasks.api'
import { useNavigate, useParams } from 'react-router-dom'

export function TaskFormPage() {
  const {
    register,
    handleSubmit,
    formState: { errors },
    setValue,
  } = useForm()
  const navigate = useNavigate()
  const params = useParams()

  const onSubmit = handleSubmit(async (data) => {
    if (params.id) {
      await updateTask(params.id, data)
    } else {
      await createTask(data)
    }
    navigate('/')
  })

  useEffect(() => {
    async function loadTask() {
      if (params.id) {
        const res = await getTask(params.id)
        setValue('title', res.data.title)
        setValue('description', res.data.description)
      }
    }

    loadTask()
  })

  return (
    <div>
      <form onSubmit={onSubmit}>
        <input
          type='text'
          placeholder='Title'
          {...register('title', { required: true })}
        />
        {errors.title && <span>this field is required</span>}
        <textarea
          rows='3'
          placeholder='Description'
          {...register('description', { required: true })}
        ></textarea>
        {errors.description && <span>this field is required</span>}
        <button>Save</button>
      </form>

      {params.id && (
        <button
          onClick={async () => {
            const accepted = window.confirm('are you sure')
            if (accepted) {
              await deleteTask(params.id)
              navigate('/')
            }
          }}
        >
          Delete
        </button>
      )}
    </div>
  )
}
```

## React-hot-toast (mensajes)

Importamos en app.jsx react-hot-toast y lo colocamos luego de las rutas. El componente siempre estará disponible solo hay que visualizarlo.

```js
import { BrowserRouter, Routes, Route, Navigate } from 'react-router-dom'
import { TasksPage } from './pages/TaskPage'
import { TaskFormPage } from './pages/TaskFormPage'
import { Navigation } from './components/Navigation'
import { Toaster } from 'react-hot-toast'

function App() {
  return (
    <BrowserRouter>
      <Navigation />

      <Routes>
        <Route path='/' element={<Navigate to='/tasks' />} />
        <Route path='/tasks' element={<TasksPage />} />
        <Route path='/tasks-create' element={<TaskFormPage />} />
        <Route path='/tasks/:id' element={<TaskFormPage />} />
      </Routes>
      <Toaster />
    </BrowserRouter>
  )
}

export default App
```

Para ello importaremos toast en el lugar que queremos.

```js
const onSubmit = handleSubmit(async (data) => {
  if (params.id) {
    await updateTask(params.id, data)
  } else {
    await createTask(data)
    toast.success('Task created')
  }
  navigate('/')
})
```

Si queremos lo podemos personalizar.

```js
toast.success('Task created', {
  position: 'bottom-right',
  style: {
    background: '#101010',
    color:'#fff'
  }
})
```
Aaí con el resto
```js
  const onSubmit = handleSubmit(async (data) => {
    if (params.id) {
      await updateTask(params.id, data)
      toast.success('Task updated', {
        position: 'bottom-right',
        style: {
          background: '#101010',
          color: '#fff',
        },
      })
    } else {
      await createTask(data)
      toast.success('Task created', {
        position: 'bottom-right',
        style: {
          background: '#101010',
          color: '#fff',
        },
      })
    }
    navigate('/')
  })
```
Delete

```js
<button
          onClick={async () => {
            const accepted = window.confirm('are you sure')
            if (accepted) {
              await deleteTask(params.id)
              toast.success('Task deleted', {
                position: 'bottom-right',
                style: {
                  background: '#101010',
                  color: '#fff',
                },
              })
              navigate('/')
            }
          }}
        >
          Delete
        </button>
```
Quedando así todoo.

```js
import { useEffect } from 'react'
import { useForm } from 'react-hook-form'
import { createTask, deleteTask, getTask, updateTask } from '../api/tasks.api'
import { useNavigate, useParams } from 'react-router-dom'
import { toast } from 'react-hot-toast'

export function TaskFormPage() {
  const {
    register,
    handleSubmit,
    formState: { errors },
    setValue,
  } = useForm()
  const navigate = useNavigate()
  const params = useParams()

  const onSubmit = handleSubmit(async (data) => {
    if (params.id) {
      await updateTask(params.id, data)
      toast.success('Task updated', {
        position: 'bottom-right',
        style: {
          background: '#101010',
          color: '#fff',
        },
      })
    } else {
      await createTask(data)
      toast.success('Task created', {
        position: 'bottom-right',
        style: {
          background: '#101010',
          color: '#fff',
        },
      })
    }
    navigate('/')
  })

  useEffect(() => {
    async function loadTask() {
      if (params.id) {
        const res = await getTask(params.id)
        setValue('title', res.data.title)
        setValue('description', res.data.description)
      }
    }

    loadTask()
  })

  return (
    <div>
      <form onSubmit={onSubmit}>
        <input
          type='text'
          placeholder='Title'
          {...register('title', { required: true })}
        />
        {errors.title && <span>this field is required</span>}
        <textarea
          rows='3'
          placeholder='Description'
          {...register('description', { required: true })}
        ></textarea>
        {errors.description && <span>this field is required</span>}
        <button>Save</button>
      </form>

      {params.id && (
        <button
          onClick={async () => {
            const accepted = window.confirm('are you sure')
            if (accepted) {
              await deleteTask(params.id)
              toast.success('Task deleted', {
                position: 'bottom-right',
                style: {
                  background: '#101010',
                  color: '#fff',
                },
              })
              navigate('/')
            }
          }}
        >
          Delete
        </button>
      )}
    </div>
  )
}
```

## Tailwind

Instalamos tailwind, obviamente dentro de client.

```bash
npm install -D tailwindcss postcss autoprefixer
npx tailwindcss init -p
```

Editaremos tailwind config para que se vea asi

```js
/** @type {import('tailwindcss').Config} */
export default {
  content: ['./index.html', './src/**/*.{js,ts,jsx,tsx}'],
  theme: {
    extend: {},
  },
  plugins: [],
}

```

En index.css importamos base, components y utilities.

```css
@tailwind base;
@tailwind components;
@tailwind utilities;
```

 Y empezamos a personalizar, recordar que en React es className.