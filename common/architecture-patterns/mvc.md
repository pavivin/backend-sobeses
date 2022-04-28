# Model-View-Controller

* Model - хранит состояние системы
* View - показывает информацию модели пользователю
* Controller - интерпретирует действия пользователя, оповещая модель, о необходимости изменений

https://openclassrooms.com/en/courses/6900866-write-maintainable-python-code/7009312-structure-an-application-with-the-mvc-design-pattern

## Model

Библиотека песен

```python
PRICE_PER_SONG = 1.20

class Song:
    def __init__(self, name, artist, genre, artwork):
        self.artist = artist
        self.name = name
        self.genre = genre
        self.artwork = artwork

class Library:
    def __init__(self):
        self.songs = []

class ServiceInfo:
    def __init__(self, status, engineer_name):
        self.service_date = datetime.now()
        self.status = status
        self.engineer = engineer_name
```

## View

Представление и взаимодействие с пользователем

```python
class Touchscreen:
    def select_song(self):
        pass

    def prompt_for_next_song(self, songs):
        for song in songs:
            # display the songs
            pass
        return "Dark Chest of Wonders"

class Speakers:
    def __init__(self):
        self.volume = 5

    def get_louder(self):
        self.volume += 1

    def get_quieter(self):
        self.volume -= 1

    def play_song(self, song):
        pass

class CoinSlot:
    def __init__(self, float_):
        self.amount = float_

    def request_money(self, amount):
        # wait for money
        # give change
        self.amount += amount
        return True
```

## Controller

Контроллер содержит логику приложения

```python
class Controller:
    def __init__(self):
        self.library = Library()
        self.service_history = []
        self.audio_output = Speakers()
        self.ui = Touchscreen()
        self.bank = CoinBox()

    def play_next_song(self):
        songs_to_suggest = []
        for song in self.library:
            # filter logic
            songs_to_suggest.append(song)
        chosen_song = self.ui.prompt_for_next_song(songs_to_suggest)
        request_money(PRICE_PER_SONG)
        self.audio_output.play_song(chosen_song)
     
    # Lots more functions go here...
```


## Использование:

___Django___

__M__, доступ к данным, обрабатывается слоем работы с базой данных, который описан в этой главе.

__V__, эта часть, которая определяет какие данные получать и как их отображать, обрабатывается представлениями и шаблонами.

__C__, эта часть, которая выбирает представление в зависимости от пользовательского ввода, обрабатывается самой средой разработки, следуя созданной вами схемой URL, и вызывает соответствующую функцию Python для указанного URL. 