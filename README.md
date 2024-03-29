# MVP

Новый инструмент на данном этапе должен решать 2 задачи:

* Формировать список страниц под создание
* Формировать список страниц для дооптимизации

## Описание базовой концепции работы инструмента

Для конкретного сайта мы можем выгрузить большое количество запросов с дополнительной статистикой через интерфес API.

Для MVP модели продукта можно ограничиться только данными Google Search Console API, т.к. данный функционал уже используется в [облаке](https://colab.research.google.com/drive/13pZlwl4o9AENle754icBmVywQzY9kWJl#scrollTo=my6FUIvfzHYq).

## Формирование списка страниц под создание

>Мы предполагаем, что запросы, которые находятся за ТОП **N**, где N - это эвристика, и мы предполагаем, что, если запрос находится за N в основной поисковой системе, то релевантная посадочная страница отсутствует. Идея состоит в том, чтобы автоматически получить такие запросы, подтянуть статистику, сгруппировать их, вычленить нецелевые, выгрузить результат, проанализировав который, SEO специалист сможет принять решение относительно целесообразности создания новых страниц. 

### Этап 1: Задаем входные параметры
На первом этапе мы указываем:

* адрес сайта (предварительно настроив интегарцию с API сервисов)
* тип страниц (хотим ли мы находить коммерческие или информационные страницы)
* указать список минус-слов для последующей фильтрации
* указать диапазон, за который мы хотим получить статистику (оптимально 3 месяца)

На этом этапе мы задаем вводные параметры, с которыми запускается программа.

### Этап 2: Получаем первичные данные

На основе полученных на предыдущем этапе данных программа делает обращение к API и получает список запросов и статистику этих запросов.

1. Запрос
2. URL
3. Клики
4. Показы
5. Позиция
6. CTR

На этом этапе мы получаем большой набор данных, который затем планируем обработать программным методом. 

### Этап 3: Фильтруем данные

Мы делаем допущение, что если сайт по конкретному запросу находится за опеределенной позицией, то релевантная страница отсутствует (возможно не в индексе, но скорее всего просто отсутствует). Мы оставляем в выборке только запросы, в которых позиция > N. 

Число N можно сделать именованым параметром, т.е. мы можем его задавать на старте, а можем не завадать и применять дефолтное значение, например N = 30.

Тут также все зависит от текущего уровня оптимизации проекта. 

Затем после фильтрации сразу чистим список по минус-словам. 

### Этап 4: Подтягиваем коммерциализацию

На этом этапе мы должны получить данные из Тулс по коммерциализации запросов. 

Затем фильтруем по значению, например для коммерческих страниц, коммерциализация > 60, для информационных < 60.

### Этап 5: Группировка

На этом этапе мы уже имеем такую выборку:

1. Запрос
2. URL
3. Клики
4. Показы
5. Позиция
6. CTR
7. Коммерциализация

Этот датасет мы группируем по полю "Запрос", используя алгоритм k-means (как один из вариантов, который уже опробован), либо по URL.

### Этап 6: Обработка кластеров

На этом этапе мы уже имеем сгруппированные запросы. 
Далее к этим данным добавляем 2 поля: количество запросов в группе и сумамрное количество показов запросов данной группы.

Затем мы фильтруем данные и оставляем только группы где количество запросов в группе больше среднего количества запросов во всех группах и суммарное количество показов больше среднего суммарного количества показов во всех группах.

Таким образом мы получаем группы, которые содержат запросы с нужным уровнем коммерциализации, без минус-слов, с высоким количеством показов (что может говорить либо о бОльшем спросе либо о высокой релевантности группы в тематике сайта) и высоким количеством запросов в группе (разбирать 10.000 кластеров с 1 и 2 запросами в группе - не эффективное занятие).

### Этап 7: Выгрузка

На этом этапе просто выгружаем запросы в excel или другой формат. 

Далее специалисту нужно (пока еще) руками пройтись и проанализировать файл. 
Обязательно зафиксировать все ошибки, чтобы можно было доработать алгоритм. Обработать файл (например, посмотреть выдачу, посмотреть, а точно ли нет рел. страницы, перегруппировать вызывающие сомнения кластера уже стандартным кластеризатором по ТОПу), создать новые страницы.

### Варианты для улучшения на будущее:

* **повышение качества работы инструмента: скорость, четкость данных, удобство и простота в использовании**
* **автоматическая кластеризация по ТОПу, если k-means не даст качественных результатов**
* **интеграция данного функционала с API популярных CMS, для автоматического создания страниц на сайта**
* **подлючение разных источников данных, например: Метрика, Вебмастер, GA, другие инструменты и сервисы для получения более качественной статистики**

## Формирование списка страниц для оптимизации

>Мы предполагаем, что если собрать запросы по которым страница сайта находится на позициях ТОП 3-15, а затем наиболее значимые слова из этих запросов добавить на страницу (тем самым увеличив ширину облака), мы сможем повысить ранжирование страницы по данным запросам, тем самым получить дополнительный трафик.

### Этап 1: Задаем входные параметры

На первом этапе мы указываем:

* адрес сайта (предварительно настроив интегарцию с API сервисов)
* указать список минус-слов для последующей фильтрации
* указать диапазон, за который мы хотим получить статистику (оптимально 3 месяца)

На этом этапе мы задаем вводные параметры, с которыми запускается программа.

### Этап 2: Получаем первичные данные

На основе полученных на предыдущем этапе данных программа делает обращение к API и получает список запросов и статистику этих запросов.

1. Запрос
2. URL
3. Клики
4. Показы
5. Позиция
6. CTR

На этом этапе мы получаем большой набор данных, который затем планируем обработать программным методом.

### Этап 3: Фильтруем данные

Мы отбираем запросы по которым сайт ранжируется в диапазоне [n=3, m=15], где n, m имеют заданыне значения, но могут быть изменены, если это необходимо.

Далее убираем запросы с минус-словами.
Убираем стоп-слова.

### Этап 4: Группируем данные по URL

Группировка позволяет получить сгруппированные данные для дальнейшей обработки.

### Этап 5: Обработка кластеров

На этом этапе мы уже имеем сгруппированные запросы.
Далее к этим данным добавляем 2 поля: количество запросов в группе и сумамрное количество показов запросов данной группы.

Затем мы фильтруем данные и оставляем только группы где количество запросов в группе больше среднего количества запросов во всех группах и суммарное количество показов больше среднего суммарного количества показов во всех группах.

### Этап 6: Фильтруем по полям 

Формируем выборку, где количество запросов в группе > чем у 20% групп и суммарное количество показов > чем у 20% всех групп.

### Этап 7: Формируем массив слов

Для каждой группы из всех слов в запросе (стоп-слова уже удалены на этапе 3).

Затем массив лемматизируем.

### Этап 8: Для каждого массива строим частотный словарь

Для каждого массива строим частотный словарь и оставляем 10 наиболее часто встречающихся слов в массиве.

### Этап 9: Получаем результирующую выборку

На выходе получаем следующие данные:

* ЗАПРОС
* URL
* КЛИКИ
* ПОКАЗЫ
* ПОЗИЦИЯ
* CTR
* ГРУППА
* ОБЪЕМ ГРУППЫ
* СУММАРНОЕ КОЛ-ВО ПОКАЗОВ ГРУППЫ
* СПИСОК НАИБОЕЕ ЧАСТОТНЫХ ЛЕММ

### Этап 10: Выгружаем данные

На выходе у нас пояляется файл с группами у которых наибольшее число показов и запросов. А также список наиболее часто встречающихся слов из запросов для каждой группы.
Далее останется проверить леммы и страницы и вписать эти леммы в контент страницы.

--- 

## Как посмотреть блок-схему

В репозитории находится файл **block-diagram.drawio**,
который представляет собой блок-схему описанного тут процесса. 

Данный файл можно открыть, воспользовавшись сайтом <https://app.diagrams.net/>.

Для того, чтобы открыть файл, его необходимо скачать из репозитория.
