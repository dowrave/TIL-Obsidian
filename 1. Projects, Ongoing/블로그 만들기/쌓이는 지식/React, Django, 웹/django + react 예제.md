- [[django + react 설치]]에 이어서 작업함
- 간단한 TodoList App을 작성한다.

## 백엔드 작업
- `myapp`일 경우
```python
# settings.py
INSTALLED_APPS = [
	  'myapp',
	  ...
]
```

### models.py
```python
from django.db import models

class Todo(models.Model):
	title = models.CharField(max_length = 200)
	completed = models.BooleanField(default = False)

    def __str__(self):
        return self.title
```

### 마이그레이션 및 관리자 계정 생성
```python
python manage.py makemigrations
python manage.py migrate
python manage.py createsuperuser
```

### Django API 작성
- `myapp/views.py`
```python
from rest_framework import generics
from .models import Todo
from .serializers import TodoSerializer

class TodoListCreateView(generics.ListCreateAPIView):
	queryset = Todo.objects.all()
	serializers_class = TodoSerializer

class TodoDetailView(generics.RetrieveUpdateDestroyAPIView):
    queryset = Todo.objects.all()
    serializer_class = TodoSerializer
```

- `myapp/serializers.py`
```python
from rest_framework import serializers
from .models import Todo

class TodoSerializer(serializers.ModelSerializer):
    class Meta:
        model = Todo
        fields = ['id', 'title', 'completed']
```

- `myapp/urls.py`
```python
from django.urls import path
from .views import TodoListCreateView

urlpatterns = [
    path('api/todos/', TodoListCreateView.as_view(), name='todo-list-create'),
	path('api/todos/<int:pk>/', TodoDetailView.as_view(), name='todo-detail'),

]
```

- `myproject/urls.py`
```python
from django.contrib import admin
from django.urls import path, include

urlpatterns = [
    path('admin/', admin.site.urls),
    path('', include('myapp.urls')),
]
```

## 프론트엔드 작업
### app.tsx 작성
- 조회, 추가, 삭제, 토글 기능을 넣는다.
```tsx
import axios from 'axios';
import React, { useState, useEffect } from 'react';
import { produce } from 'immer';

// django의 models.py 양식을 따른다. id는 django에서도 자동으로 지정됨.
type Todo = {
  id: number;
  title: string;
  completed: boolean;
}

const App: React.FC = () => {
  const [todos, setTodos]= useState<Todo[]>([])
  const [newTodo, setNewTodo] = useState<string>('');

  // useEffect : 마운트될 때만 동작(빈 배열 전달)
  useEffect(() => {
    fetchTodos();  
  }, []);

  // 조회
  const fetchTodos = async () => {
    try {
    const response = await axios.get<Todo[]>('http://localhost:8000/api/todos/');
    setTodos(response.data);
    console.log(response.data);
    } catch (error) {
      console.error(error)
    }
  };


  const addTodo = () => {
    axios.post('http://localhost:8000/api/todos/', { title: newTodo })
      .then(response => {
        setTodos([...todos, response.data]);
        setNewTodo('');
      })
      .catch(error => {
        console.error(error);
      })
  }

  const toggleTodo = (id: number, completed: boolean) => {
    axios.patch<Todo>(`http://localhost:8000/api/todos/${id}/`, { completed: !completed})
      .then(() => {
        fetchTodos();
      })
      .catch(error => {
        console.error(error);
      });
  } ;

    const deleteTodo = (id: number) => {
      axios.delete(`http://localhost:8000/api/todos/${id}/`)
        .then(() => {
          fetchTodos();
          // setTodos((prevTodos) => prevTodos.filter(todo => todo.id !== id));

        })
        .catch(error => {
          console.error(error);
        })
    };


  return (
    <div>
      <h1>Todo List</h1>
      <ul>
        {todos.map((todo: Todo) => (
          <li key={todo.id}>
            <span style={{ textDecoration: todo.completed ? 'line-through' : 'none'}}>
              {todo.title}
            </span>
            <button onClick = {() => toggleTodo(todo.id, todo.completed)}>Finished</button>
            <button onClick={() => deleteTodo(todo.id)}>Delete</button>
          </li>
        ))}
      </ul>
      <div><input type="text" value={newTodo} onChange={(e) => setNewTodo(e.target.value)}/>
      <button onClick={addTodo}>Add Todo</button></div>
    </div>
  )
}

export default App;

```
> 여기서 `Todo` 타입을 정의할 때 `models.py`에는 없는 `id`가 정의되는데, django에서는 따로 지정하지 않더라도 `id`라는 필드가 자동으로 정의된다.

## 마무리
- 2개의 커맨드쉘을 띄우고 하나는 `npm run dev`, 다른 하나는 `python manage.py runserver`를 띄운다.
- 프론트엔드에서의 상호작용이 백엔드에 잘 반영되는지 확인하면 된다.

## 궁금한 것

### serializer가 뭐임?
- `Django REST framework`에서 사용되는 시리얼라이저를 정의하는 파일이다.
- `시리얼라이저`란, Django 모델 객체나 기타 복잡한 **데이터 타입을 JSON 형태로 변환**하거나 반대로 **JSON 데이터를 파이썬 객체로 변환**하는 역할을 한다. 주로 API에서 직렬화와 역직렬화를 한다.
- 이는 DB 모델의 인스턴스를 JSON 형식으로 변환해서 클라이언트에 제공하거나, 클라이언트에서 보낸 JSON 데이터를 파이썬 객체로 변환해 DB에 저장할 때 유용하다.
- 일반적으로 `serializers.ModelSerializer`를 상속받아 모델과 관련된 필드 및 동작을 정의한다. 시리얼라이저 클래스를 사용하면 모델의 데이터를 쉽게 JSON 형식으로 변환할 수 있다.
- 위 예제에서는 
```python
# myapp/serializers.py
from rest_framework import serializers
from .models import Todo

class TodoSerializer(serializers.ModelSerializer):
    class Meta:
        model = Todo
        fields = ['id', 'title', 'completed']
```
> 같은 방식을 사용했는데, 이는 `Todo` 모델에 대한 시리얼라이저이며, `id, title, completed` 필드를 JSON으로 변환할 때 포함을 시킨다는 의미이다.


### React.FC는 뭐임?
- React 함수 컴포넌트 타입을 정의하는 TS에서 사용되는 유형 중 하나이다. `FC = Functional Component함수 컴포넌트`임.
- `React.FC`를 사용하면 컴포넌트에 대한 기본 프로퍼티와 상태 타입이 사전에 정의된 것으로 간주된다. `children, props` 속성이 자동으로 포함되어 있다.