# TDD

Test-Driven Development (TDD) - методология в разработке програмного обеспечения, которая фокусируется на итеративном цикле разработки. Особенностью методологии является написание тестов до написания самого кода. 
TDD использует повторение коротких циклов разработки, совмещает в себе разработку и тестирование. Этот процесс не только помогает обеспечить корректность кода, но также косвенно развивает дизайн и архитектуру проекта.

TDD обычно следует циклу "Red-Green-Refactor"


1. Добавьте тест
2. (<span style="color:red">Red</span>) Запустите все тесты, чтобы убедиться что новые падают.
3. (<span style="color:green">Green</span>) Напишите ровно столько кода, чтобы прошёл ровно один тест
4. Запустите все тесты
5. (<span style="color:yellow">Refactor</span>) Улучшите исходный код, сохраняя при этом тесты "зелёными"
6. Повторите

![image](assets/red-green-refactor.png)

Этот процесс кажется медленным и часто он может быть краткосрочным, но он улучшает качество проекта в долгосрочной перспективе. Наличие достаточного тестового покрытия служит гарантией того, что вы случайно не измените функциональность. Намного лучше обнаружить ошибку локально из своего набора тестов, чем это сделает клиент.

В целом, наборы тестов могут выразить ожидания от вашего проекта, так чтобы стейкхолдеры могли лучше понять проект.


Преимущества

TDD поощряет написание тестируемого, слабо-связанного кода, который обычно более модульный. Поскольку хорошо структурированный, модульный код легче писать, отлаживать, понимать, управлять и переиспользовать, TDD помогает:

* Уменьшить затраты
* Сделать рефакторинг и переписывание проще и быстрее
* Оптимизировать процесс внедрения проекта
* Предотвратить баги и связанность
* Улучшить общее взаимодействие в команде
* Повысить уверенность в том, что код работает как и ожидалось
* Улучшение шаблонов кода
* Убрать страх поменять что-нибудь 

TDD также поощряет постоянное размышление и совершенствование. Это часто показывает области и абстракции в вашем коде, которые необходимо переосмыслить, что помогает стимулировать и улучшать общий дизайн.

Наконец, благодаря наличию обширного набора тестов, охватывающего практически все возможные пути, разработчики могут получать быструю обратную связь в режиме реального времени во время разработки. Это снижает общий стресс, повышает эффективность и повышает производительность.

## Итеративный процесс
Как и при написании кода в целом, при написании тестов старайтесь не слишком переживать по поводу написания идеального теста с первого раза. Напишите быстрый тест с информацией, которая у вас есть в данный момент, и переработайте его позже, когда узнаете больше.

## Подходы
Существует два основных подхода к TDD - Inside Out и Outside In.

## Inside Out
При подходе Inside Out основное внимание уделяется результатам (или состоянию). Тестирование начинается с самого маленького уровня устройства, и архитектура возникает органично. Этот подход, как правило, легче освоить новичкам, он пытается свести к минимуму моки и помогает предотвратить чрезмерную разработку. Проектирование происходит на стадии рефакторинга, что, к сожалению, может привести к большим рефакторингам.

## Outside In
Подход Outside In фокусируется на поведении пользователя. Тестирование начинается на самом внешнем уровне, и детали появляются по мере того, как вы продвигаетесь вперед. Этот подход в значительной степени опирается на моки и стабы  внешних зависимостей. Как правило, его сложнее освоить, но он помогает обеспечить соответствие кода общим потребностям бизнеса. Дизайн происходит на красной стадии.

### Какой подход лучше?
Часто проще использовать подход Outside In при работе со сложными приложениями, которые имеют большое количество быстро меняющихся внешних зависимостей (например, микросервисов). Небольшие монолитные приложения часто лучше подходят для подхода Inside Out.

Внешний подход также, как правило, лучше работает с интерфейсными приложениями, поскольку код настолько близок к конечному пользователю.

## Почему важно видеть, что тест падает?
Это может показаться странным, но увидеть провал теста так же важно, как и увидеть прохождение теста в TDD.

Проще говоря, неудачный тест:
1. Подтверждает, что новый тест является значимым и уникальным, помогая гарантировать, что реализованный код не только полезен, но и необходим.
2. Обеспечивает конечную цель, то, к чему вы должны стремиться. Это фокусирует ваше мышление таким образом, чтобы вы написали ровно столько кода, чтобы достичь этой цели.

[Пример (Flask, Pytest)](service.md)

[Источник](https://testdriven.io/test-driven-development)