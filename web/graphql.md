### Содержание
- [Содержание](#содержание)
- [Введение](#введение)
- [Основы](#основы)
- [Разработка схемы Movie](#разработка-схемы-movie)
- [Создание запросов](#создание-запросов)
- [Использование библиотеки Graphene](#использование-библиотеки-graphene)
- [Создание модели](#создание-модели)
- [Мутации](#мутации)
- [Создание схемы](#создание-схемы)
- [Тестирование нашего API](#тестирование-нашего-api)
- [Общение через POST](#общение-через-post)
- [Заключение](#заключение)

### Введение
Веб-API — это двигатели, на которых основано большинство наших приложений. В течение многих лет REST был доминирующей архитектурой для API, но со временем у него стало проявляться множество недостатков. И на его замену было разработано GraphQL.

С помощью REST API вы обычно создаете несколько URL для каждого доступного объекта данных. Допустим, мы создаем REST API для приложения справочника по фильмам — у нас будут URL-адреса для самих фильмов, актеров, наград, режиссеров, продюсеров … (вообщем множество URL). Плюс к этому для получения связанных данных может потребоваться множество разных запросов по нескольким URL. Как видите все быстро становиться громоздким. А еще к этому представьте, что вам нужно что бы это приложение работало на мобильном телефоне с медленным интернет-соединением. Создается огромное количество проблем.

GraphQL — это не API-архитектура, подобная REST, это язык, который позволяет нам гораздо проще обмениваться связанными данными. Мы будем использовать его для разработки API для фильмов. Позже мы создадим API для фильмов на Django, с библиотекой Graphene.

Что такое GraphQL
Изначально созданный Facebook, но в настоящее время разрабатываемый в рамках GraphQL Foundation, GraphQL — это язык запросов и среда выполнения, которая позволяет нам получать и манипулировать данные.

В начале нам нужно определить данные, которые мы хотим получать для API. Затем мы создадим схему для API — т.е. набор разрешенных запросов для извлечения и изменения данных.

### Основы
Основным представителем Python является небольшая библиотека под названием Graphene. Но прежде нужно поговорить о некоторых фундаментальных основах GraphQL:

Модель – это объект, который определен с помощью GraphQL.
Схема определяет модели и их атрибуты.
Каждый атрибут модели имеет свой собственный преобразователь. Это функция, которая отвечает за получение данных для этого конкретного атрибута модели.
Запрос – это то, что использует пользователь, дабы получить или отправить данные в GraphQL.
Мутации – особые запросы, которые позволяют изменять данные конкретной модели или набора моделей.
GraphiQL – пользовательский интерфейс, используемый для взаимодействия с сервером GraphQL.


### Разработка схемы Movie
Создание типов данных
Типы описывают виды данных, которые доступны через API. Мы можем использовать базовые типы данных, и мы также можем определять наши собственные пользовательские типы.

Рассмотрим следующие типы для актеров и фильмов:

```cython
type Actor {  
  id: ID!
  name: String!
}
type Movie {  
  id: ID!
  title: String!
  actors: [Actor]
  year: Int!
}
```

Тип ID говорит нам, что поле является уникальным идентификатором для этого типа данных.

Примечание. Восклицательный знак означает, что поле является обязательным.

Вы также могли бы заметить, что в Movie мы используем базовые типа, таких как String и Int, а также наш собственный тип Actor.

Если мы хотим, чтобы поле содержало список типов, мы заключаем его в квадратные скобки — [Actor].

### Создание запросов
Запрос указывает, какие данные могут быть получены и что требуется для этого:

```cython
type Query {  
  actor(id: ID!): Actor
  movie(id: ID!): Movie
  actors: [Actor]
  movies: [Movie]
}
```

Этот тип запроса позволяет нам получать данные об актере и фильме, предоставляя их через ID, или мы можем получить их список без фильтрации.

Создание мутаций (Mutations)
Мутация описывает, какие операции можно выполнить для изменения данных на сервере.

Мутации основаны на двух вещах:

Inputs — специальные типы используются только в качестве аргументов в мутации, когда мы хотим передать весь объект вместо отдельных полей.
Payloads — обычные типы, но по соглашению мы используем их в качестве выходных данных для мутации, поэтому мы можем легко расширять их по мере развития API.
Первое, что мы делаем, это создадим типы Input:

```cython
input ActorInput {  
  id: ID
  name: String!
}
input MovieInput {  
  id: ID
  title: String
  actors: [ActorInput]
  year: Int
}

И затем мы создаем типы Payload:

type ActorPayload {  
  ok: Boolean
  actor: Actor
}
type MoviePayload {  
  ok: Boolean
  movie: Movie
}
```

Обратите внимание на поле ok, для типов Payload означает включение метаданных, таких как состояние или поле ошибки.

Тип Mutation объединяет все это:

```cython
type Mutation {  
  createActor(input: ActorInput) : ActorPayload
  createMovie(input: MovieInput) : MoviePayload
  updateActor(id: ID!, input: ActorInput) : ActorPayload
  updateMovie(id: ID!, input: MovieInput) : MoviePayload
}
```

Для мутатора createActor необходим объект ActorInput, для которого требуется имя актера.

Мутатор updateActor требует ID обновляемой записи.

То же самое следует для мутаторов createMovie и updateMovie.

Примечание. Хотя ActorPayload и MoviePayload не являются необходимыми для успешной мутации, рекомендуется, чтобы API предоставляли обратную связь при обработке действия.

Определение схемы
Наконец, мы сопоставляем созданные нами запросы и мутации с окончательной схемой:

```cython
schema {  
  query: Query
  mutation: Mutation
}
```

### Использование библиотеки Graphene
GraphQL не зависит от платформы, можно создать сервер GraphQL с различными языками программирования (Java, PHP, Go), фреймворками (Node.js, Symfony, Rails) или платформами, такими как Apollo.

С Graphene нам не нужно использовать синтаксис GraphQL для создания схемы, мы используем только Python! Эта библиотека с открытым исходным кодом также была интегрирована с Django, чтобы мы могли создавать схемы, ссылаясь на модели нашего приложения.

Настройка приложения
Виртуальные среды
Лучше всего сразу начать использовать виртуальные среды для проектов Django. Мы будем использовать pipenv c Python 3.6.

Используя терминал, войдите в свое рабочее пространство и создайте следующую папку:

```cython
$ mkdir django_graphql_movies
$ cd django_graphql_movies/
Теперь создайте виртуальную среду:

$ pipenv shell --python 3.6
Установка и настройка Django и Graphene
Находясь в нашей виртуальной среде, мы используем pipenv для установки Django и библиотеки Graphene:

(django_graphql_movies) bash-3.2$ pipenv install django
(django_graphql_movies) bash-3.2$ pipenv install graphene_django
Затем мы создаем наш проект Django:

(django_graphql_movies) bash-3.2$ django-admin.py startproject django_graphql_movies .
Далее создадим приложение для наших фильмов:

$ (django_graphql_movies) bash-3.2$ django-admin.py startapp movies
Прежде чем мы начнем работать над нашим приложением, нам надо запустить миграцию базы данных:

(django_graphql_movies) bash-3.2$ python manage.py migrate
```

### Создание модели
Модели Django описывают макет базы данных нашего проекта. Каждая модель представляет собой класс Python, который обычно отображается в таблицу базы данных. Свойства класса сопоставляются со столбцами базы данных.

Внесите следующие измения в файл movies/models.py:

```cython
from django.db import models
class Actor(models.Model):
    name = models.CharField(max_length=100)
    def __str__(self):
        return self.name
    class Meta:
        ordering = ('name',)
class Movie(models.Model):
    title = models.CharField(max_length=100)
    actors = models.ManyToManyField(Actor)
    year = models.IntegerField()
    def __str__(self):
        return self.title
    class Meta:
        ordering = ('title',)
```

Как и в схеме GraphQL, модель Actor имеет имя, тогда как модель Movie имеет название (title), отношение «многие ко многим» с актерами (actors) и год (year). ID будет автоматически генерировать для нас Django.

Теперь мы можем зарегистрировать наше приложение фильмов Movie в проекте. Перейдите в django_graphql_movies/settings.py и измените INSTALLED_APPS следующим образом:

```cython
INSTALLED_APPS = [  
    'django.contrib.admin',
    'django.contrib.auth',
    'django.contrib.contenttypes',
    'django.contrib.sessions',
    'django.contrib.messages',
    'django.contrib.staticfiles',
    'movies',
]
```

Обязательно запустите миграцию базы данных, чтобы синхронизировать ее с изменениями нашего кода:

```cython
(django_graphql_movies) bash-3.2$ python manage.py makemigrations
(django_graphql_movies) bash-3.2$ python manage.py migrate
```
Загрузка данных теста
После того, как мы создадим наше API, мы хотим иметь возможность выполнить запросы, чтобы проверить, работает ли оно. Давайте сейчас загрузим некоторые данные в нашу базу данных. Для этого сохраните следующий JSON как movies.json в корневом каталоге вашего проекта:

```cython
[
  {
    "model": "movies.actor",
    "pk": 1,
    "fields": {
      "name": "Michael B. Jordan"
    }
  },
  {
    "model": "movies.actor",
    "pk": 2,
    "fields": {
      "name": "Sylvester Stallone"
    }
  },
  {
    "model": "movies.movie",
    "pk": 1,
    "fields": {
      "title": "Creed",
      "actors": [1, 2],
      "year": "2015"
    }
  }
]
```

И выполните следующую команду для загрузки тестовых данных:

(django_graphql_movies) bash-3.2$ python manage.py loaddata movies.json
Вы должны увидеть следующий вывод в терминале:

Installed 3 object(s) from 1 fixture(s)  
Создание нашей схемы с Graphene
Создание запросов
В нашей папке приложения для фильмов (movies) создайте новый файл schema.py . Далее с помощью следующего кода определим наши типы GraphQL:

```cython
import graphene
from graphene_django.types import DjangoObjectType, ObjectType
from movies.models import Actor, Movie
# Create a GraphQL type for the actor model
class ActorType(DjangoObjectType):
    class Meta:
        model = Actor
# Create a GraphQL type for the movie model
class MovieType(DjangoObjectType):
    class Meta:
        model = Movie
```

С помощью Graphene для создания типа GraphQL мы просто указываем, какая модели Django будет использоваться в качестве свойства, которые мы хотим видеть в API.

В том же файле добавьте следующий код для создания типа Query:

```cython
# Create a Query type
class Query(ObjectType):
    actor = graphene.Field(ActorType, id=graphene.Int())
    movie = graphene.Field(MovieType, id=graphene.Int())
    actors = graphene.List(ActorType)
    movies = graphene.List(MovieType)
    def resolve_actor(self, info, **kwargs):
        id = kwargs.get('id')
        if id is not None:
            return Actor.objects.get(pk=id)
        return None
    def resolve_movie(self, info, **kwargs):
        id = kwargs.get('id')
        if id is not None:
            return Movie.objects.get(pk=id)
        return None
    def resolve_actors(self, info, **kwargs):
        return Actor.objects.all()
    def resolve_movies(self, info, **kwargs):
        return Movie.objects.all()
```

Каждое свойство класса Query соответствует запросу GraphQL:

Свойства actor и movie возвращают одно значение ActorType и MovieType соответственно, и оба требуют целочисленного ID.
Свойства actors и movies возвращают список их соответствующих типов.
Четыре метода, которые мы создали в классе Query, называются резольверами (Resolvers). Resolvers связывает запросы в схеме с реальными действиями, выполняемыми базой данных. Как это принято в Django, мы взаимодействуем с нашей базой данных через модели.

Рассмотрим функцию resolve_actor. Мы получаем ID из параметров запроса и возвращаем актера из нашей базы данных с этим ID в качестве его первичного ключа. Функция resolve_actors просто получает всех актеров в базе данных и возвращает их в виде списка.

### Мутации
Когда мы проектировали схему, мы сначала создали специальные типы ввода для наших мутаций. Давайте сделаем то же самое с Graphene, добавьте это код в schema.py:

```cython
# Create Input Object Types
class ActorInput(graphene.InputObjectType):
    id = graphene.ID()
    name = graphene.String()
class MovieInput(graphene.InputObjectType):
    id = graphene.ID()
    title = graphene.String()
    actors = graphene.List(ActorInput)
    year = graphene.Int()
```

Это простые классы, которые определяют, какие поля можно использовать для изменения данных в API.

Создание мутаций требует немного больше работы, чем создание запросов. Давайте добавим мутации для актеров:

```cython
# Create mutations for actors
class CreateActor(graphene.Mutation):
    class Arguments:
        input = ActorInput(required=True)
    ok = graphene.Boolean()
    actor = graphene.Field(ActorType)
    @staticmethod
    def mutate(root, info, input=None):
        ok = True
        actor_instance = Actor(name=input.name)
        actor_instance.save()
        return CreateActor(ok=ok, actor=actor_instance)
class UpdateActor(graphene.Mutation):
    class Arguments:
        id = graphene.Int(required=True)
        input = ActorInput(required=True)
    ok = graphene.Boolean()
    actor = graphene.Field(ActorType)
    @staticmethod
    def mutate(root, info, id, input=None):
        ok = False
        actor_instance = Actor.objects.get(pk=id)
        if actor_instance:
            ok = True
            actor_instance.name = input.name
            actor_instance.save()
            return UpdateActor(ok=ok, actor=actor_instance)
        return UpdateActor(ok=ok, actor=None)
```

Вспомните сигнатуру для мутации createActor, когда мы проектировали нашу схему:

createActor(input: ActorInput) : ActorPayload  
Имя нашего класса соответствует имени запроса GraphQL.
Внутренние свойства класса Arguments соответствуют входным аргументам для мутатора.
Свойства ok и actor составляют ActorPayload.
При написании метода мутации важно знать, что вы сохраняете данные в модели Django:

Мы берем имя из входного объекта (input.name) и создаем новый объект Actor.
Мы вызываем функцию сохранения, чтобы наши изменения сохранились в базе данных, и возвращаем Payload пользователю.
Класс UpdateActor имеет аналогичную структуру с дополнительной логикой для извлечения обновляемого актера и изменения его свойств перед сохранением.

Теперь давайте добавим мутацию для фильмов:
```cython
# Create mutations for movies
class CreateMovie(graphene.Mutation):
    class Arguments:
        input = MovieInput(required=True)
    ok = graphene.Boolean()
    movie = graphene.Field(MovieType)
    @staticmethod
    def mutate(root, info, input=None):
        ok = True
        actors = []
        for actor_input in input.actors:
            actor = Actor.objects.get(pk=actor_input.id)
            if actor is None:
                return CreateMovie(ok=False, movie=None)
            actors.append(actor)
        movie_instance = Movie(
            title=input.title,
            year=input.year
        )
        movie_instance.save()
        movie_instance.actors.set(actors)
        return CreateMovie(ok=ok, movie=movie_instance)
class UpdateMovie(graphene.Mutation):
    class Arguments:
        id = graphene.Int(required=True)
        input = MovieInput(required=True)
    ok = graphene.Boolean()
    movie = graphene.Field(MovieType)
    @staticmethod
    def mutate(root, info, id, input=None):
        ok = False
        movie_instance = Movie.objects.get(pk=id)
        if movie_instance:
            ok = True
            actors = []
            for actor_input in input.actors:
                actor = Actor.objects.get(pk=actor_input.id)
                if actor is None:
                    return CreateMovie(ok=False, movie=None)
                actors.append(actor)
            movie_instance = Movie(
                title=input.title,
                year=input.year
            )
            movie_instance.save()
            movie_instance.actors.set(actors)
            return UpdateMovie(ok=ok, movie=movie_instance)
        return UpdateMovie(ok=ok, movie=None)
```

Поскольку фильмы ссылаются на актеров, поэтому мы должны извлечь данные об актерах из базы данных перед сохранением. Цикл for сначала проверяет, что актеры, предоставленные пользователем, действительно находятся в базе данных, если нет, он возвращает ошибку без сохранения каких-либо данных.

При работе со связями «многие ко многим» в Django мы можем сохранять связанные данные только после сохранения нашего объекта.

Вот почему мы сохраняем наш фильм с movie_instance.save() перед тем, как установить для него актеров с помощью movie_instance.actors.set(actors).

Чтобы завершить наши мутации, создадим тип мутации (класс Mutation):
```cython
class Mutation(graphene.ObjectType):
    create_actor = CreateActor.Field()
    update_actor = UpdateActor.Field()
    create_movie = CreateMovie.Field()
    update_movie = UpdateMovie.Field()
```

### Создание схемы
Как и раньше, когда мы проектировали нашу схему, мы сопоставляем запросы и мутации с API нашего приложения. Теперь добавьте эту строку в конец файла schema.py:

```cython
schema = graphene.Schema(query=Query, mutation=Mutation)
```
Регистрация схемы в проекте
Чтобы наше API заработало, нам нужно сделать схему доступной для всего проекта.

Создайте новый файл schema.py в каталоге django_graphql_movies/ и добавьте в него следующее:
```cython
import graphene
import movies.schema
class Query(movies.schema.Query, graphene.ObjectType):
    # This class will inherit from multiple Queries
    # as we begin to add more apps to our project
    pass
class Mutation(movies.schema.Mutation, graphene.ObjectType):
    # This class will inherit from multiple Queries
    # as we begin to add more apps to our project
    pass
schema = graphene.Schema(query=Query, mutation=Mutation)
```

Теперь мы можем зарегистрировать graphene и сказать ему использовать нашу схему.

Откройте django_graphql_movies/settings.py и добавьте ‘graphene_django‘ в качестве первого элемента в INSTALLED_APPS.

В том же файле добавьте следующий код:

```cython
GRAPHENE = {  
    'SCHEMA': 'django_graphql_movies.schema.schema'
}
```

API-интерфейсы GraphQL доступны через одну конечную точку (через один URL) graphql. Поэтмоу нужно зарегистрировать этот маршрут.

Откройте django_graphql_movies/urls.py и измените содержимое файла на:

```cython
from django.contrib import admin
from django.urls import path
from graphene_django.views import GraphQLView
urlpatterns = [
    path('admin/', admin.site.urls),
    path('graphql/', GraphQLView.as_view(graphiql=True)),
]
```

### Тестирование нашего API
Чтобы протестировать API, давайте запустим проект и перейдем к конечной точке GraphQL. Введите в терминале следующую команду:

(django_graphql_movies) bash-3.2$ python manage.py runserver
Как только сервер запустится, перейдите по ссылке http://127.0.0.1:8000/graphql/. Вы должны увидеть GraphiQL — встроенной IDE для выполнения запросов!

Написание запросов
Для нашего первого запроса, давайте получим всех актеров из базы данных. В верхней левой панели введите следующее:

```cython
query getActors {  
  actors {
    id
    name
  }
}
```

Формат запросов в GraphQL. Мы начинаем с ключевого слова query, за которым следует необязательное имя запроса. Рекомендуется присваивать запросам имена, поскольку это может помочь при ведении журнала и отладке. GraphQL также позволяет нам указать нужные поля — мы выбрали id и name.

Несмотря на то, что у нас есть только один фильм в наших тестовых данных, давайте попробуем запрос фильма и обнаружим еще одну замечательную особенность GraphQL:

```cython
query getMovie {  
  movie(id: 1) {
    id
    title
    actors {
      id
      name
    }
  }
}
```

Для запроса фильма требуется ID, поэтому мы указываем его в скобках. Интересная часть в поле актеров. В нашей модели Django мы включили свойство актеров в наш класс Movie и указали отношения «многие ко многим» между ними. Это позволяет нам получить все свойства типа Actor, которые связаны с данными фильма.

Этот подобный графику обход данных является основной причиной, почему GraphQL считается мощной и интересной технологией!

Написание мутаций
Мутации следуют тому же стилю, что и запросы. Давайте добавим актера в нашу базу данных:

```cython
mutation createActor {  
  createActor(input: {
    name: "Tom Hanks"
  }) {
    ok
    actor {
      id
      name
    }
  }
}
```

Обратите внимание, как параметр input соответствует input свойствам класса Arguments, которые мы создали ранее.

Также обратите внимание, как возвращаемые значения ok и actor отображаются в свойствах класса мутации CreateActor.

Теперь мы можем добавить фильм, в котором снялся Том Хэнкс:

```cython
mutation createMovie {  
  createMovie(input: {
    title: "Cast Away",
    actors: [
      {
        id: 3
      }
    ]
    year: 1999
  }) {
    ok
    movie{
      id
      title
      actors {
        id
        name
      }
      year
    }
  }
}
```

К сожалению, здесь мы ошиблись. «Cast Away» вышел в 2000 году!

Давайте запустим запрос на обновление, чтобы исправить это:

```cython
mutation updateMovie {  
  updateMovie(id: 2, input: {
    title: "Cast Away",
    actors: [
      {
        id: 3
      }
    ]
    year: 2000
  }) {
    ok
    movie{
      id
      title
      actors {
        id
        name
      }
      year
    }
  }
}
```

Теперь все исправлено и мы проверили как работает редактирование!

### Общение через POST
IDE GraphiQL очень полезна во время разработки, но стандартная практика отключать этот экран в производственной среде, поскольку это может позволить стороннему разработчику слишком глубоко залезть в ваше API.

Чтобы отключить GraphiQL, просто отредактируйте django_graphql_movies/urls.py таким образом, чтобы путь (‘graphql/’, GraphQLView.as_view (graphiql = True)), стал путем (‘graphql/’, GraphQLView.as_view (graphiql = False)).

Приложение, взаимодействующее с API, должно отправлять POST-запросы в /graphql. Прежде чем мы сможем отправлять POST-запросы извне сайта Django, нам нужно опять изменить django_graphql_movies/urls.py:

```cython
from django.contrib import admin  
from django.urls import path  
from graphene_django.views import GraphQLView  
from django.views.decorators.csrf import csrf_exempt  # New library
urlpatterns = [  
    path('admin/', admin.site.urls),
    path('graphql/', csrf_exempt(GraphQLView.as_view(graphiql=False))),
]
```

В Django встроена защита CSRF (Cross-Site Request Forgery) — в которой предусмотрены меры по предотвращению некорректной аутентификации пользователей и от совершения ими потенциально вредоносных действий.

Хотя это полезная защита, она не позволит внешним приложениям взаимодействовать с API. Если вам будет мешать эта защита вам нужно будет рассмотреть другие формы аутентификации.

Давай те проверим работу POST запросов. Для этого в своем терминале введите следующую команду, чтобы получить всех актеров:

```cython
(django_graphql_movies) bash-3.2$ curl \
  -X POST \
  -H "Content-Type: application/json" \
  --data '{ "query": "{ actors { name } }" }' \
  http://127.0.0.1:8000/graphql/
Вы должны получить:

{"data":{"actors":[{"name":"Michael B. Jordan"},{"name":"Sylvester Stallone"},{"name":"Tom Hanks"}]}}
```

### Заключение
В этой стате мы разработали схему API для нашего тестового приложения справочника фильмов, создав нужные типы GraphQL, запросы и мутации, необходимые для получения и изменения данных. Так же с помощью Django и Graphene мы создали приложения с работающим GraphQL API которое способно обрабатывать как запросы GraphQL так и POST запросы.
