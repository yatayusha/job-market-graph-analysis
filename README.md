[![Review Assignment Due Date](https://classroom.github.com/assets/deadline-readme-button-22041afd0340ce965d47ae6ef1cefeee28c7c493a6346c4f15d667ab976d596c.svg)](https://classroom.github.com/a/DJGv7DPp)
# Визуализация графа вакансий
Эта работа посвящена основам анализа данных с помощью графа на примере данных с [hh.ru](http://hh.ru). 

## Сбор данных

Задача анализа данных будет следующая: проанализировать актуальные вакансии, выявив среди них кластеры похожих вакансий по навыкам, которые требуются от кандидатов. Нам нужна будет следующая информация:

- название вакансии
- требования к кандидату

Также при сборе данных нужно извлечь обязанности в рамках вакансии (достаточно 200-300 вакансий).

Для сбора данных по вакансиям нам понадобится раздел [Поиск вакансий](https://api.hh.ru/openapi/redoc#tag/Poisk-vakansij/operation/get-vacancies). Этот метод в общем случае требует токен:

![image](https://github.com/user-attachments/assets/2c86db03-9b2d-43f5-847b-36970744f67b)


Без токена по запросу мы получим информацию из поиска на сайте, где интересующая нас информация о требованиях размещена в сниппетах. Оттуда и будем ее извлекать. Ниже пример вакансии, полученной по запросу `python`:

```jsx
{'accept_incomplete_resumes': False,
  'accept_temporary': False,
  'address': {'building': '14',
              'city': 'Москва',
              'description': None,
              'id': '695097',
              'lat': 55.77835,
              'lng': 37.623375,
              'metro': None,
              'metro_stations': [],
              'raw': 'Москва, Олимпийский проспект, 14',
              'street': 'Олимпийский проспект'},
  'adv_context': None,
  'adv_response_url': None,
  'alternate_url': 'https://hh.ru/vacancy/119897279',
  'apply_alternate_url': 'https://hh.ru/applicant/vacancy_response?vacancyId=119897279',
  'archived': False,
  'area': {'id': '1', 'name': 'Москва', 'url': 'https://api.hh.ru/areas/1'},
  'branding': {'tariff': None, 'type': 'MAKEUP'},
  'contacts': None,
  'created_at': '2025-04-24T10:27:09+0300',
  'department': None,
  'employer': {'accredited_it_employer': False,
               'alternate_url': 'https://hh.ru/employer/1947',
               'employer_rating': {'reviews_count': 409, 'total_rating': '3.7'},
               'id': '1947',
               'logo_urls': {'240': 'https://img.hhcdn.ru/employer-logo/382088.jpeg',
                             '90': 'https://img.hhcdn.ru/employer-logo/309231.jpeg',
                             'original': 'https://img.hhcdn.ru/employer-logo-original/232625.jpg'},
               'name': 'КРЕДИТ ЕВРОПА БАНК',
               'trusted': True,
               'url': 'https://api.hh.ru/employers/1947',
               'vacancies_url': 'https://api.hh.ru/vacancies?employer_id=1947'},
  'employment': {'id': 'full', 'name': 'Полная занятость'},
  'employment_form': {'id': 'FULL', 'name': 'Полная'},
  'experience': {'id': 'noExperience', 'name': 'Нет опыта'},
  'fly_in_fly_out_duration': [],
  'has_test': False,
  'id': '119897279',
  'insider_interview': None,
  'internship': False,
  'is_adv_vacancy': False,
  'name': 'Аналитик SQL',
  'night_shifts': False,
  'premium': False,
  'professional_roles': [{'id': '156', 'name': 'BI-аналитик, аналитик данных'}],
  'published_at': '2025-04-24T10:27:09+0300',
  'relations': [],
  'response_letter_required': False,
  'response_url': None,
  'salary': None,
  'salary_range': None,
  'schedule': {'id': 'fullDay', 'name': 'Полный день'},
  'show_contacts': True,
  'show_logo_in_search': True,
  'snippet': {'requirement': 'Уверенное владение MS Office (Excel, Word, Power '
                             'Point). Умение проведения ad-hoc исследований с '
                             'помощью <highlighttext>python</highlighttext>, '
                             'либо знание любого другого...',
              'responsibility': 'Поддержка и разработка аналитической '
                                'отчетности. Создание презентаций для '
                                'руководства банка (в том числе на английском '
                                'языке). Создание аналитических витрин в...'},
  'sort_point_distance': None,
  'type': {'id': 'open', 'name': 'Открытая'},
  'url': 'https://api.hh.ru/vacancies/119897279?host=hh.ru',
  'work_format': [{'id': 'HYBRID', 'name': 'Гибрид'}],
  'work_schedule_by_days': [{'id': 'FIVE_ON_TWO_OFF', 'name': '5/2'}],
  'working_days': [],
  'working_hours': [{'id': 'HOURS_8', 'name': '8\xa0часов'}],
  'working_time_intervals': [],
  'working_time_modes': []}
```

Первая задача – реализовать функцию `get_vacancies`, принимающую на вход ключевое слово для поиска вакансии и количество страниц, с которых надо собрать вакансии, а возвращающая список кортежей с айди, названием, требованиями и обязанностями, как в примере ниже:

```jsx
('120013095',
 'Программист (junior)',
 'Знание SQL, PL/SQL и средств оптимизации запросов. Знание PHP, Html, Css, '
 'JavaScript, <highlighttext>Python</highlighttext>. Умение разбираться в '
 'чужом коде и...',
 'Сопровождение и развитие информационных систем. Разработка нового '
 'функционала и сервисов. Создание и оптимизация запросов SQL (хранимых '
 'процедур, триггеров, функций, представлений). ')
```

Шаблон функции:

```python
def get_vacancies(pages, tag):
    """
    Функция принимает на вход количество страниц, с которых надо собрать информацию (pages)
    и ключевое слово, по которому должен осуществляться поиск (tag).
    Вернуть необходимо список уникальных по айди вакнсий.
    """
    url = 'https://api.hh.ru/vacancies'
    headers = {"User-Agent": "example@yandex.ru"} 
    # put your code here
    pass
```

Для выполнения запросов к API мы будем использовать библиотеку [requests](http://docs.python-requests.org/en/master/) (для выполнения запросов в асинхронном режиме можно использовать библиотеку [httpx](https://github.com/encode/httpx)):

```python
(cs102) $ python -m pip install requests
```

Выполняя запросы мы не можем быть уверены, что не возникнет ошибок. Возможны различные ситуации, например:

- есть неполадки в сети;
- удаленный сервер по какой-то причине не может обработать запрос;
- мы слишком долго ждем ответ от сервера.

В таких случаях необходимо попробовать повторить запрос. При этом повторные запросы желательно посылать не через константные промежутки времени, а по алгоритму экспоненциальной задержки.

Далее задача - реализовать класс сессии, который позволит выполнять GET и POST-запросы к указанному адресу, а при необходимости повторять запрос указанное число раз по алгоритму экспоненциальной задержки:

```python
class Session(requests.Session):
    """
    Сессия.

    :param base_url: Базовый адрес, на который будут выполняться запросы.
    :param timeout: Максимальное время ожидания ответа от сервера.
    :param max_retries: Максимальное число повторных запросов.
    :param backoff_factor: Коэффициент экспоненциального нарастания задержки.
    """

    def __init__(
        self,
        base_url: str,
        timeout: float = 5.0,
        max_retries: int = 3,
        backoff_factor: float = 0.3,
    ) -> None:
        pass

    def get(self, url, **kwargs: tp.Any) -> requests.Response:
        pass

    def post(self, url, data=None, json=None, **kwargs: tp.Any) -> requests.Response:
        pass
```

## Построение графа вакансий

Для этого необходимо выполнить следующие задачи:

### Извлечение ключевых слов из текстов

Так как мы будем отталкиваться от графа навыков, то и извлекать ключевые слова будем из поля “требования” (requirements). К задаче извлечения ключевых слов можно подходить по-разному: от частотных методов до нейросетевых. Чтобы не усложнять задачу и не слишком погружаться в многообразие подходов, воспользуемся одним из самых базовых – извлечению ключевых слов по метрике TF-IDF. С нуля ее реализовывать не будем. Воспользуемся встроенным классом в `sklearn`. 

`TfidfVectorizer` формирует для текста вектор, где в качестве значений содержится **TF-IDF** каждого слова.

**TF (term frequency)** – относительная частотность слова в документе:

$$
TF(t,d) = \frac{n_{t}}{\sum_k n_{k}}
$$

**IDF (inverse document frequency)** – обратная частота документов, в которых есть это слово:

$$
IDF(t, D) = log \frac{|D|}{|{d : t \in d}|}
$$

Перемножаем их:

$$
TFIDF(t, d, D) = TF(t,d) \times IDF(i, D)
$$

Идея в том, что если слово часто встречается в одном документе, но в целом по корпусу встречается в небольшом количестве документов, у него высокий TF-IDF, и, соответственно, слова с наивысшим TF-IDF можно считать ключевыми для документа.

```python
from sklearn.feature_extraction.text import TfidfVectorizer

# инициализация векторайзера для символов
tfidf_vectorizer = TfidfVectorizer(ngram_range=(1, 4))

# обучаем его и сразу применяем тексту
tfidf_vectorized_text = tfidf_vectorizer.fit_transform([" ".join(tokens)])

# первые 5 объектов вектора
pd.DataFrame(tfidf_vectorized_text.T.todense(), index=tfidf_vectorizer.get_feature_names_out(), columns=["TF-IDF"]).head()
```

Чтобы успешно и аккуратно найти ключевые слова по TF-IDF, выполним несколько этапов предобработки текста:

1. Оставить в тексте только символы кириллицы, латиницы и дефисы в середине слова.
2. Выполнить нормализацию текста (лемматизацию). В этой работе воспользуйтесь библиотекой `spacy`  и  словарем `ru_core_news_sm`.
3. Оставить в текстах только те слова, которые являются существительными или не для них невозможно определить часть речи, то есть слово написано не на русском языке. Для этого следует ориентироваться на частеречные теги (`token.pos_`) `NOUN` и `X` в `spacy`.

Ключевыми словами будем считать 5 слов текста с наивысшим TF-IDF. 

Далее реализуем две функции: `preprocess_text` и `get_keywords`.

```python
import spacy
import re
from sklearn.feature_extraction.text import TfidfVectorizer
import pandas as pd

def preprocess_text(text: str) -> str:
    """
    Принимает на вход текст с требуемыми навыками.
    Выполняет очистку, лемматизацию и фильтрацию по части речи.
    Возвращает отфильтрованные леммы через пробел без запятых сплошным текстом.
    """
    nlp = spacy.load("ru_core_news_sm")
    # put your code here
    pass

def get_keywords(df):
    """
    Принимает на вход датафрейм с вакансиями и полем обработанных навыков.
    Возвращает этот же датафрейм с новым столбцом с ключевыми словами.
    """
    vectorizer = TfidfVectorizer()
    # put your code here
    return df
```

### Построение графа

Следующей задачей является реализация функции `create_network()`, которая позволяет построить граф вакансий по переданному ей датафрейму. Граф должен быть представлен в виде списка ребер:

```python
def create_network(df):
    """
    Принимает на вход датафрейм с вакансиями и ключевыми словами.
    Возвращает список кортежей из пар вакансий.
    """
    pass
```

![image](https://github.com/user-attachments/assets/5db831af-b88d-49d8-b292-cbd77983ed96)

Очевидно, что этот граф надо некоторым образом декомпозировать, чтобы появилась возможность его проанализировать или хотя бы прочитать. Для этого часто решают задачу поиска кластеров или сообществ на графе (community detection). Воспользуемся для кластеризации одним из самых популярных алгоритмов – Louvain.

```python
>>> net = create_network(df)
>>> plot_communities(net)
```

![image](https://github.com/user-attachments/assets/dde6f0d2-7a3d-4dd3-a502-502921639eb5)


Часть функций для извлечения сообществ и визуализации графа уже готовы. Вы можете воспользоваться ими или модифицировать под свою реализацию. Добавим функцию для визуализации в plotly одного сообщества по переданному ей графу и списку вершин из сообщества.

```python
def plot_one_community(graph, community):
    """
    Строит график в plotly для одного сообщества.
    """
    pass
```

## Мини-приложение для аналитики вакансий

Неинтересно оставлять всю проделанную работу на уровне консольного приложения. Поэтому добавим для него веб-интерфейс. Так как большинство графиков реализуется с помощью `plotly` , удобно будет для веб-интерфейса воспользоваться лоу-код фреймворком [Plotly Dash](https://dash.plotly.com), который может как существовать самостоятельно, так и быть интегрированным в [`flask`](https://flask.palletsprojects.com/en/stable/). 


### Создадим небольшое приложение из трех страниц:

1. стартовая
2. краткое описание исходных данных
3. аналитика сообществ

На стартовой находятся кнопки для переходов к двум разделам:

![image](https://github.com/user-attachments/assets/7a97c2f4-c417-4e3b-be8a-fb62597151af)


В файле `index.html` хранится шаблон для `flask` :

```python
<!DOCTYPE html>
<html>
<head>
    <title>Главная</title>
    <style>
        body {
            font-family: 'Segoe UI', sans-serif;
            text-align: center;
            padding: 50px;
            background-color: #f7f7f7;
        }
        h1 {
            font-size: 2.5em;
            margin-bottom: 30px;
        }
        p {
            font-size: 1.1em;
            color: #555;
        }
        .btn {
            display: inline-block;
            margin: 10px;
            padding: 12px 24px;
            font-size: 1.1em;
            color: white;
            background-color: #007BFF;
            border: none;
            border-radius: 8px;
            text-decoration: none;
            transition: background 0.3s ease;
        }
        .btn:hover {
            background-color: #0056b3;
        }
    </style>
</head>
<body>
    <p>В этом мини-приложении собраны и визуализированы данные о вакансиях с hh.ru</p>
    <a class="btn" href="/dashboard/">📊 Исходные данные</a>
    <a class="btn" href="/network/">📊 Аналитика данных</a>
</body>
</html>
```

В файле [`app.py`](http://app.py) расположен код страниц:

```python
from flask import Flask, render_template
import dash
from dash import html, dcc, dash_table
import pandas as pd
import plotly.express as px
import networkx as nx
import matplotlib.colors as mcolors
from itertools import combinations, chain
import plotly.graph_objects as go

# загрузка данных: эту часть необходимо изменить и дополнить в соответствии с вашими данными
kdf = pd.read_csv("titles_keywords.csv")

df = kdf["title"].value_counts().reset_index()[:10]
fig = px.bar(df, x="title", y="count", title="Самые частые вакансии")

# создание стартовой страницы
server = Flask(__name__)

@server.route('/')
def index():
    return render_template("index.html")

# страница со статистикой по исходным данным
dash_dashboard_app = dash.Dash(
    __name__,
    server=server,
    url_base_pathname='/dashboard/',
    suppress_callback_exceptions=True
)

dash_dashboard_app.layout = html.Div(style={
    'fontFamily': 'Segoe UI',
    'textAlign': 'center',
    'padding': '10px',
    'backgroundColor': '#f0f8ff'
}, children=[
    html.H2("📊 Исходные данные"),
    html.A("← Назад", href='/', style={
        'color': '#28a745',
        'textDecoration': 'none',
        'fontSize': '1.1em'
    }),
    dcc.Graph(figure=fig, style={'marginBottom': '10px', 'marginTop': '10px'}),
    dash_table.DataTable(
        data=kdf.to_dict('records'),
        columns=[{"name": i, "id": i} for i in kdf.columns],
        style_cell={'textAlign': 'center', 'padding': '1px'},
        style_header={
            'backgroundColor': '#28a745',
            'color': 'white',
            'fontWeight': 'bold'
        },
        style_table={'width': '100%', 'margin': '0 auto'}
    ),
    html.Br(),
])

# страница с визуализацией графов
dash_dashboard_app = dash.Dash(
    __name__,
    server=server,
    url_base_pathname='/network/',
    suppress_callback_exceptions=True
)

# эту часть необходимо дописать
dash_dashboard_app.layout = html.Div()

# запуск приложения
if __name__ == '__main__':
    server.run(debug=False)

```

Страница с исходными данными имеет следующий вид:

![image](https://github.com/user-attachments/assets/ca2ca39d-707e-4262-8e20-f409bbebbfc8)
