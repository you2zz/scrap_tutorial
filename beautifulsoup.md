## Создание объекта soup
Чтобы проанализировать документ, передайте его в конструктор BeautifulSoup. Вы можете передать строку или открытый дескриптор файла:
```python
from bs4 import BeautifulSoup

with open("index.html") as fp:
    soup = BeautifulSoup(fp)

soup = BeautifulSoup("<html>data</html>")
```
Сначала документ преобразуется в Unicode, а HTML-объекты преобразуются в символы Unicode:
```python
BeautifulSoup("Sacr&eacute; bleu!")
<html><head></head><body>Sacré bleu!</body></html>
```
Затем Beautiful Soup анализирует документ, используя наилучший доступный синтаксический анализатор. Он будет использовать синтаксический анализатор HTML, если вы специально не укажете ему использовать синтаксический анализатор XML. (Смотрите раздел Синтаксический анализ XML.)
## Виды объектов
Beautiful Soup преобразует сложный HTML-документ в сложное дерево объектов Python. Но вам придется иметь дело в основном с четырьмя типами объектов: Tag, NavigableString, BeautifulSoup и Comment.
### Tag
Объект Tag соответствует тегу XML или HTML в исходном документе:
```python
soup = BeautifulSoup('<b class="boldest">Extremely bold</b>')
tag = soup.b
type(tag)
# <class 'bs4.element.Tag'>
```
Теги имеют множество атрибутов и методов, и я расскажу о большинстве из них в разделе "Навигация по дереву" и "Поиск по дереву". На данный момент наиболее важными характеристиками тега являются его имя (name) и атрибуты.
#### Name
У каждого тега есть имя, доступное как .name:
```python
tag.name
# u'b'
```
Если вы измените название тега, это изменение будет отражено в любой HTML-разметке, сгенерированной Beautiful Soup:
```python
tag.name = "blockquote"
tag
# <blockquote class="boldest">Extremely bold</blockquote>
```
#### Attributes
Тег может иметь любое количество атрибутов. Тег ```<b id="boldest">``` имеет атрибут “id”, значение которого равно “boldest”. Вы можете получить доступ к атрибутам тега, рассматривая его как словарь
```python
tag['id']
# u'boldest'
```
Вы можете получить доступ к этому словарю напрямую как .attrs:
```python
tag.attrs
# {u'id': 'boldest'}
```
Вы можете добавлять, удалять и изменять атрибуты тега. Опять же, это делается путем обработки тега как словаря:
```python
tag['id'] = 'verybold'
tag['another-attribute'] = 1
tag
# <b another-attribute="1" id="verybold"></b>

del tag['id']
del tag['another-attribute']
tag
# <b></b>

tag['id']
# KeyError: 'id'
print(tag.get('id'))
# None
```
#### Многозначные атрибуты
HTML 4 определяет несколько атрибутов, которые могут иметь несколько значений. HTML 5 удаляет пару из них, но определяет еще несколько. Наиболее распространенным многозначным атрибутом является class (то есть тег может иметь более одного CSS-класса). Другие включают rel, rev, accept-charset, заголовки и ключ доступа. Beautiful Soup представляет значение (значения) многозначного атрибута в виде списка:
```python
css_soup = BeautifulSoup('<p class="body"></p>')
css_soup.p['class']
# ["body"]

css_soup = BeautifulSoup('<p class="body strikeout"></p>')
css_soup.p['class']
# ["body", "strikeout"]
```
Если атрибут выглядит так, как будто он имеет более одного значения, но это не многозначный атрибут, как определено в любой версии стандарта HTML, Beautiful Soup оставит атрибут в покое:
```python
id_soup = BeautifulSoup('<p id="my id"></p>')
id_soup.p['id']
# 'my id'
```
Когда вы превращаете тег обратно в строку, значения нескольких атрибутов объединяются:
```python
rel_soup = BeautifulSoup('<p>Back to the <a rel="index">homepage</a></p>')
rel_soup.a['rel']
# ['index']
rel_soup.a['rel'] = ['index', 'contents']
print(rel_soup.p)
# <p>Back to the <a rel="index contents">homepage</a></p>
```
Вы можете отключить это, передав multi_valued_attributes=None в качестве аргумента ключевого слова в конструктор BeautifulSoup:
```python
no_list_soup = BeautifulSoup('<p class="body strikeout"></p>', 'html', multi_valued_attributes=None)
no_list_soup.p['class']
# u'body strikeout'
```
Вы можете использовать `get_attribute_list чтобы получить значение, которое всегда является списком, независимо от того, является ли оно многозначным атрибутом или нет:
```python
id_soup.p.get_attribute_list('id')
# ["my id"]
```
Если вы анализируете документ в формате XML, многозначные атрибуты отсутствуют:
```python
xml_soup = BeautifulSoup('<p class="body strikeout"></p>', 'xml')
xml_soup.p['class']
# u'body strikeout'
```
Опять же, вы можете настроить это, используя аргумент multi_valued_attributes:
```python
class_is_multi= { '*' : 'class'}
xml_soup = BeautifulSoup('<p class="body strikeout"></p>', 'xml', multi_valued_attributes=class_is_multi)
xml_soup.p['class']
# [u'body', u'strikeout']
```
Вероятно, вам не нужно будет этого делать, но если вы это сделаете, используйте значения по умолчанию в качестве руководства. Они реализуют правила, описанные в спецификации HTML:
```python
from bs4.builder import builder_registry
builder_registry.lookup('html').DEFAULT_CDATA_LIST_ATTRIBUTES
```
### NavigableString (Навигационная строка)
Строка соответствует фрагменту текста внутри тега. Beautiful Soup использует класс NavigableString для хранения этих фрагментов текста:
```python
tag.string
# u'Extremely bold'
type(tag.string)
# <class 'bs4.element.NavigableString'>
```
NavigableString точно так же, как строка Unicode в Python, за исключением того, что она также поддерживает некоторые функции, описанные в разделе Навигация по дереву и поиск по дереву. Вы можете преобразовать NavigableString в строку Unicode с помощью unicode():
```python
unicode_string = unicode(tag.string)
unicode_string
# u'Extremely bold'
type(unicode_string)
# <type 'unicode'>
```





```python

```