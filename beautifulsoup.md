# Создание объекта soup
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
# Виды объектов
Beautiful Soup преобразует сложный HTML-документ в сложное дерево объектов Python. Но вам придется иметь дело в основном с четырьмя типами объектов: Tag, NavigableString, BeautifulSoup и Comment.
## Tag
Объект Tag соответствует тегу XML или HTML в исходном документе:
```python
soup = BeautifulSoup('<b class="boldest">Extremely bold</b>')
tag = soup.b
type(tag)
# <class 'bs4.element.Tag'>
```
Теги имеют множество атрибутов и методов, и я расскажу о большинстве из них в разделе "Навигация по дереву" и "Поиск по дереву". На данный момент наиболее важными характеристиками тега являются его имя (name) и атрибуты.
### Name
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
### Attributes
Тег может иметь любое количество атрибутов. Тег ```<b id="boldest">``` имеет атрибут “id”, значение которого равно “boldest”. Вы можете получить доступ к атрибутам тега, рассматривая его как словарь
```python
tag['id']
# u'boldest'
```
Вы можете получить доступ к этому словарю напрямую как ```.attrs```:
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
### Многозначные атрибуты
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
Вы можете отключить это, передав ```multi_valued_attributes=None``` в качестве аргумента ключевого слова в конструктор BeautifulSoup:
```python
no_list_soup = BeautifulSoup('<p class="body strikeout"></p>', 'html', multi_valued_attributes=None)
no_list_soup.p['class']
# u'body strikeout'
```
Вы можете использовать ```get_attribute_list``` чтобы получить значение, которое всегда является списком, независимо от того, является ли оно многозначным атрибутом или нет:
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
## NavigableString (Навигационная строка)
Строка соответствует фрагменту текста внутри тега. Beautiful Soup использует класс NavigableString для хранения этих фрагментов текста:
```python
tag.string
# u'Extremely bold'
type(tag.string)
# <class 'bs4.element.NavigableString'>
```
NavigableString точно так же, как строка Unicode в Python, за исключением того, что она также поддерживает некоторые функции, описанные в разделе Навигация по дереву и поиск по дереву. Вы можете преобразовать NavigableString в строку Unicode с помощью ```unicode()```:
```python
unicode_string = unicode(tag.string)
unicode_string
# u'Extremely bold'
type(unicode_string)
# <type 'unicode'>
```
Вы не можете редактировать строку на месте, но вы можете заменить одну строку другой, используя ```replace_with()```:
```python
tag.string.replace_with("No longer bold")
tag
# <blockquote>No longer bold</blockquote>
```
NavigableString поддерживает большинство функций, описанных в разделе Навигация по дереву и поиск по дереву, но не все из них. В частности, поскольку строка не может содержать ничего (подобно тому, как тег может содержать строку или другой тег), строки не поддерживают атрибуты ```.contents``` или `.string`, а также метод `find()`.

Если вы хотите использовать NavigableString за пределами Beautiful Soup, вам следует вызвать unicode() для него, чтобы превратить его в обычную строку Unicode Python. Если вы этого не сделаете, ваша строка будет содержать ссылку на все дерево синтаксического анализа Beautiful Soup, даже когда вы закончите использовать Beautiful Soup. Это большая трата памяти.
## BeautifulSoup
Объект BeautifulSoup представляет проанализированный документ в целом. Для большинства целей вы можете рассматривать его как объект тега. Это означает, что он поддерживает большинство методов, описанных в разделе Навигация по дереву и поиск по дереву.

Вы также можете передать объект BeautifulSoup в один из методов, определенных при изменении дерева, точно так же, как вы передали бы тег. Это позволяет вам делать такие вещи, как объединение двух проанализированных документов:

```python
doc = BeautifulSoup("<document><content/>INSERT FOOTER HERE</document", "xml")
footer = BeautifulSoup("<footer>Here's the footer</footer>", "xml")
doc.find(text="INSERT FOOTER HERE").replace_with(footer)
# u'INSERT FOOTER HERE'
print(doc)
# <?xml version="1.0" encoding="utf-8"?>
# <document><content/><footer>Here's the footer</footer></document>
```
Поскольку объект BeautifulSoup не соответствует фактическому тегу HTML или XML, у него нет имени и атрибутов. Но иногда полезно посмотреть на его `.name`, поэтому ему было присвоено специальное `.name` “[document]”:

```python
soup.name
# u'[document]'
```
## Комментарии и другие специальные строки
`Tag`, `NavigableString`, and `BeautifulSoup` охватывают почти все, что вы увидите в HTML или XML-файле, но есть несколько оставшихся фрагментов. Единственное, о чем вам, вероятно, когда-либо придется беспокоиться, - это комментарий:

```python
markup = "<b><!--Hey, buddy. Want to buy a used parser?--></b>"
soup = BeautifulSoup(markup)
comment = soup.b.string
type(comment)
# <class 'bs4.element.Comment'>
```
Объект `Comment` - это просто особый тип `NavigableString`:

```python
comment
# u'Hey, buddy. Want to buy a used parser'
```
Но когда он появляется как часть HTML-документа, комментарий отображается со специальным форматированием:

```python
print(soup.b.prettify())
# <b>
#  <!--Hey, buddy. Want to buy a used parser?-->
# </b>
```
Beautiful Soup определяет классы для всего остального, что может отображаться в XML-документе: `CData`, `ProcessingInstruction`, `Declaration` и `Doctype`. Так же, как и `Comment`, эти классы являются подклассами `NavigableString`, которые добавляют что-то дополнительное к строке. Вот пример, который заменяет комментарий блоком `CDATA`:

```python
from bs4 import CData
cdata = CData("A CDATA block")
comment.replace_with(cdata)

print(soup.b.prettify())
# <b>
#  <![CDATA[A CDATA block]]>
# </b>
```
# Навигация по дереву
Вот снова HTML-документ “Три сестры”:

```python
html_doc = """
<html><head><title>The Dormouse's story</title></head>
<body>
<p class="title"><b>The Dormouse's story</b></p>

<p class="story">Once upon a time there were three little sisters; and their names were
<a href="http://example.com/elsie" class="sister" id="link1">Elsie</a>,
<a href="http://example.com/lacie" class="sister" id="link2">Lacie</a> and
<a href="http://example.com/tillie" class="sister" id="link3">Tillie</a>;
and they lived at the bottom of a well.</p>

<p class="story">...</p>
"""

from bs4 import BeautifulSoup
soup = BeautifulSoup(html_doc, 'html.parser')
```
Я использую это в качестве примера, чтобы показать вам, как переходить от одной части документа к другой.
## Going down (спускаясь в глубь)
Теги могут содержать строки и другие теги. Эти элементы являются дочерними для тега. Beautiful Soup предоставляет множество различных атрибутов для навигации и перебора дочерних элементов тега.

Обратите внимание, что строки Beautiful Soup не поддерживают ни один из этих атрибутов, поскольку строка не может иметь дочерних элементов.
### Навигация с использованием имен тегов
Самый простой способ навигации по дереву синтаксического анализа - это произнести название нужного вам тега. Если вам нужен тег `<head>`, просто скажите `soup.head`:
```python
soup.head
# <head><title>The Dormouse's story</title></head>

soup.title
# <title>The Dormouse's story</title>
```
Вы можете использовать этот трюк снова и снова, чтобы увеличить масштаб определенной части дерева синтаксического анализа. Этот код получает первый тег `<b>` под тегом `<body>`:

```python
soup.body.b
# <b>The Dormouse's story</b>
```
Использование имени тега в качестве атрибута даст вам только первый тег с таким именем:

```python
soup.a
# <a class="sister" href="http://example.com/elsie" id="link1">Elsie</a>
```
Если вам нужно получить все теги `<a>` или что-то более сложное, чем первый тег с определенным именем, вам нужно будет использовать один из методов, описанных в разделе Поиск по дереву, например `find_all()`:

```python
soup.find_all('a')
# [<a class="sister" href="http://example.com/elsie" id="link1">Elsie</a>,
#  <a class="sister" href="http://example.com/lacie" id="link2">Lacie</a>,
#  <a class="sister" href="http://example.com/tillie" id="link3">Tillie</a>]
```
### `.contents` и `.children`
Дочерние элементы тега доступны в списке с именем `.contents`:

```python
head_tag = soup.head
head_tag
# <head><title>The Dormouse's story</title></head>

head_tag.contents
[<title>The Dormouse's story</title>]

title_tag = head_tag.contents[0]
title_tag
# <title>The Dormouse's story</title>
title_tag.contents
# [u'The Dormouse's story']
```
У самого объекта BeautifulSoup есть дочерние элементы. В этом случае тег `<html>` является дочерним для объекта BeautifulSoup.:

```python
len(soup.contents)
# 1
soup.contents[0].name
# u'html'
```
Строка не имеет `.contents`, потому что она не может содержать ничего:

```python
text = title_tag.contents[0]
text.contents
# AttributeError: 'NavigableString' object has no attribute 'contents'
```
Вместо того, чтобы получать их в виде списка, вы можете перебирать дочерние элементы тега, используя генератор `.children`:

```python
for child in title_tag.children:
    print(child)
# The Dormouse's story
```
### .descendants
Атрибуты .contents и .children учитывают только прямых дочерних элементов тега. Например, у тега `<head>` есть единственный прямой дочерний элемент – тег `<title>`:

```python
head_tag.contents
# [<title>The Dormouse's story</title>]
```
Но у самого тега `<title>` есть дочерний элемент: строка “The Dormouse’s story”. В некотором смысле эта строка также является дочерней по отношению к тегу `<head>`. Атрибут `.descendants` позволяет вам рекурсивно перебирать все дочерние элементы тега: его прямые дочерние элементы, дочерние элементы его прямых дочерних элементов и так далее:

```python
for child in head_tag.descendants:
    print(child)
# <title>The Dormouse's story</title>
# The Dormouse's story
```
У тега `<head>` есть только один дочерний элемент, но у него есть два потомка: тег `<title>` и дочерний элемент тега `<title>`. Объект BeautifulSoup имеет только одного прямого дочернего элемента (тег `<html>`), но у него есть множество потомков:

```python
len(list(soup.children))
# 1
len(list(soup.descendants))
# 25
```
### .string
Если у тега есть только один дочерний элемент, и этот дочерний элемент является `NavigableString`, дочерний элемент становится доступным как `.string`:
```python
title_tag.string
# u'The Dormouse's story'
```
Если единственным дочерним элементом тега является другой тег, и этот тег содержит `.string`, то считается, что родительский тег имеет ту же `.string`, что и его дочерний элемент:

```python
head_tag.contents
# [<title>The Dormouse's story</title>]

head_tag.string
# u'The Dormouse's story'
```
Если тег содержит более одного элемента, то неясно, на что должен ссылаться `.string`, поэтому `.string` определяется как `None`:

```python
print(soup.html.string)
# None
```
### .strings and stripped_strings
Если внутри тега содержится более одного элемента, вы все равно можете просмотреть только строки. Используйте генератор `.strings`:

```python
for string in soup.strings:
    print(repr(string))
# u"The Dormouse's story"
# u'\n\n'
# u"The Dormouse's story"
# u'\n\n'
# u'Once upon a time there were three little sisters; and their names were\n'
# u'Elsie'
# u',\n'
# u'Lacie'
# u' and\n'
# u'Tillie'
# u';\nand they lived at the bottom of a well.'
# u'\n\n'
# u'...'
# u'\n'
```
Эти строки, как правило, содержат много лишних пробелов, которые вы можете удалить, используя вместо этого генератор `.stripped_strings`:

```python
for string in soup.stripped_strings:
    print(repr(string))
# u"The Dormouse's story"
# u"The Dormouse's story"
# u'Once upon a time there were three little sisters; and their names were'
# u'Elsie'
# u','
# u'Lacie'
# u'and'
# u'Tillie'
# u';\nand they lived at the bottom of a well.'
# u'...'
```
Здесь строки, полностью состоящие из пробелов, игнорируются, а пробелы в начале и конце строк удаляются.

## Going up (Поднимаясь вверх)
Продолжая аналогию с “генеалогическим древом”, у каждого тега и каждой строки есть родитель: тег, который его содержит.
### .parent
Вы можете получить доступ к родительскому элементу с помощью атрибута `.parent`. В примере документа “three sisters” тег `<head>` является родительским для тега `<title>`:

```python
title_tag = soup.title
title_tag
# <title>The Dormouse's story</title>
title_tag.parent
# <head><title>The Dormouse's story</title></head>
```
У самой строки заголовка есть родительский элемент: тег `<title>`, который ее содержит:

```python
title_tag.string.parent
# <title>The Dormouse's story</title>
```
Родительским элементом тега верхнего уровня, такого как `<html>`, является сам объект `BeautifulSoup`:


```python
html_tag = soup.html
type(html_tag.parent)
# <class 'bs4.BeautifulSoup'>
```
И `.parent` объекта `BeautifulSoup` определен как None:

```python
print(soup.parent)
# None
```
### .parents
Вы можете выполнить итерацию по всем родительским элементам элемента с помощью `.parents`. В этом примере `.parents` используется для перемещения от тега `<a>`, скрытого глубоко внутри документа, к самому верху документа:

```python
link = soup.a
link
# <a class="sister" href="http://example.com/elsie" id="link1">Elsie</a>
for parent in link.parents:
    if parent is None:
        print(parent)
    else:
        print(parent.name)
# p
# body
# html
# [document]
# None
```
## Going sideways (Двигаясь в бок)
Рассмотрим простой документ, подобный этому:

```python
sibling_soup = BeautifulSoup("<a><b>text1</b><c>text2</c></b></a>")
print(sibling_soup.prettify())
# <html>
#  <body>
#   <a>
#    <b>
#     text1
#    </b>
#    <c>
#     text2
#    </c>
#   </a>
#  </body>
# </html>
```
Тег `<b>` и тег `<c>` находятся на одном уровне: оба они являются прямыми дочерними элементами одного и того же тега. Мы называем их братьями и сестрами. Когда документ напечатан красиво (pretty-printed), братья и сестры отображаются на одном и том же уровне отступов. Вы также можете использовать эту связь в коде, который вы пишете.
### `.next_sibling` и `.previous_sibling`
Вы можете использовать `.next_sibling` и `.previous_sibling` для навигации между элементами страницы, которые находятся на одном уровне дерева синтаксического анализа:

```python
sibling_soup.b.next_sibling
# <c>text2</c>

sibling_soup.c.previous_sibling
# <b>text1</b>
```
В теге `<b>` есть `.next_sibling`, но нет `.previous_sibling`, потому что перед тегом `<b>` на том же уровне дерева ничего нет. По той же причине в теге `<c>` есть `.previous_sibling`, но нет `.next_sibling`:


```python
print(sibling_soup.b.previous_sibling)
# None
print(sibling_soup.c.next_sibling)
# None
```
Строки “text1” и “text2” не являются родственными, поскольку у них нет одного и того же родителя:

```python
sibling_soup.b.string
# u'text1'

print(sibling_soup.b.string.next_sibling)
# None
```
В реальных документах `.next_sibling` или `.previous_sibling` тега обычно будет строкой, содержащей пробелы. Возвращаясь к документу “три сестры”:

```python
<a href="http://example.com/elsie" class="sister" id="link1">Elsie</a>
<a href="http://example.com/lacie" class="sister" id="link2">Lacie</a>
<a href="http://example.com/tillie" class="sister" id="link3">Tillie</a>
```
Вы могли бы подумать, что `.next_sibling ` первого тега `<a>` будет вторым тегом `<a>`. Но на самом деле это строка: запятая и новая строка, которые отделяют первый тег `<a>` от второго:

```python
link = soup.a
link
# <a class="sister" href="http://example.com/elsie" id="link1">Elsie</a>

link.next_sibling
# u',\n'
```
Второй тег `<a>` на самом деле является `.next_sibling` запятой:

```python
link.next_sibling.next_sibling
# <a class="sister" href="http://example.com/lacie" id="link2">Lacie</a>
```
### `.next_siblings` и `.previous_siblings`
Вы можете перебирать родственные элементы тега с помощью `.next_siblings` или `.previous_siblings`:

```python
for sibling in soup.a.next_siblings:
    print(repr(sibling))
# u',\n'
# <a class="sister" href="http://example.com/lacie" id="link2">Lacie</a>
# u' and\n'
# <a class="sister" href="http://example.com/tillie" id="link3">Tillie</a>
# u'; and they lived at the bottom of a well.'
# None

for sibling in soup.find(id="link3").previous_siblings:
    print(repr(sibling))
# ' and\n'
# <a class="sister" href="http://example.com/lacie" id="link2">Lacie</a>
# u',\n'
# <a class="sister" href="http://example.com/elsie" id="link1">Elsie</a>
# u'Once upon a time there were three little sisters; and their names were\n'
# None
```
## Going back and forth (Ходить взад и вперед)
Взгляните на начало документа “три сестры”:

```python
<html><head><title>The Dormouse's story</title></head>
<p class="title"><b>The Dormouse's story</b></p>
```
Синтаксический анализатор HTML принимает эту строку символов и преобразует ее в серию событий: “открыть тег `<html>`”, “открыть тег `<head>`”, “открыть тег `<title>`”, “добавить строку”, “закрыть тег `<title>`”, “откройте тег `<p>`”, и так далее. Beautiful Soup предлагает инструменты для восстановления первоначального синтаксического анализа документа.

### `.next_element` и `.previous_element`
Атрибут `.next_element` строки или тега указывает на то, что было проанализировано сразу после этого. Это может быть то же самое, что и `.next_sibling`, но обычно оно кардинально отличается.

Вот последний тег `<a>` в документе “три сестры”. Его `.next_sibling` - это строка: завершение предложения, которое было прервано началом тега `<a>.`:

```python
last_a_tag = soup.find("a", id="link3")
last_a_tag
# <a class="sister" href="http://example.com/tillie" id="link3">Tillie</a>

last_a_tag.next_sibling
# '; and they lived at the bottom of a well.'
```
Но .next_element этого тега `<a>`, то, что было проанализировано сразу после тега `<a>`, не является остальной частью этого предложения: это слово “Tillie”.:

```python
last_a_tag.next_element
# u'Tillie'
```
Это потому, что в исходной разметке слово “Tillie” стояло перед этой точкой с запятой. Синтаксический анализатор обнаружил тег `<a>`, затем слово “Tillie”, затем закрывающий тег `</a>`, затем точку с запятой и остальную часть предложения. Точка с запятой находится на том же уровне, что и тег `<a>`, но слово “Tillie” встретилось первым.

Атрибут `.previous_element` является полной противоположностью `.next_element`. Он указывает на любой элемент, который был проанализирован непосредственно перед этим:

```python
last_a_tag.previous_element
# u' and\n'
last_a_tag.previous_element.next_element
# <a class="sister" href="http://example.com/tillie" id="link3">Tillie</a>
```
### `.next_elements` и `.previous_elements`
Ты уже должен был понять эту идею. Вы можете использовать эти итераторы для перемещения вперед или назад по документу по мере его анализа:

```python
for element in last_a_tag.next_elements:
    print(repr(element))
# u'Tillie'
# u';\nand they lived at the bottom of a well.'
# u'\n\n'
# <p class="story">...</p>
# u'...'
# u'\n'
# None
```
# Searching the tree (Поиск по дереву)
Beautiful Soup определяет множество методов поиска в дереве синтаксического анализа, но все они очень похожи. Я собираюсь потратить много времени на объяснение двух наиболее популярных методов: `find()` и `find_all()`. Другие методы используют почти точно такие же аргументы, поэтому я просто кратко опишу их.

Еще раз, я буду использовать документ “три сестры” в качестве примера:

```python
html_doc = """
<html><head><title>The Dormouse's story</title></head>
<body>
<p class="title"><b>The Dormouse's story</b></p>

<p class="story">Once upon a time there were three little sisters; and their names were
<a href="http://example.com/elsie" class="sister" id="link1">Elsie</a>,
<a href="http://example.com/lacie" class="sister" id="link2">Lacie</a> and
<a href="http://example.com/tillie" class="sister" id="link3">Tillie</a>;
and they lived at the bottom of a well.</p>

<p class="story">...</p>
"""

from bs4 import BeautifulSoup
soup = BeautifulSoup(html_doc, 'html.parser')
```
Передавая фильтр в аргумент типа find_all(), вы можете увеличить масштаб интересующих вас частей документа.

## Виды фильтров
Прежде чем подробно рассказать о `find_all()` и подобных методах, я хочу показать примеры различных фильтров, которые вы можете передать в эти методы. Эти фильтры отображаются снова и снова во всем поисковом API. Вы можете использовать их для фильтрации на основе имени тега, его атрибутов, текста строки или некоторой их комбинации.

### A string (строка)
Самый простой фильтр - это строка. Передайте строку методу поиска, и Beautiful Soup выполнит сопоставление именно с этой строкой. Этот код находит все теги `<b>` в документе:

```python
soup.find_all('b')
# [<b>The Dormouse's story</b>]
```
Если вы передадите строку в байтах, Beautiful Soup предположит, что строка закодирована как UTF-8. Вы можете избежать этого, передав вместо этого строку в Юникоде.

### A regular expression (Регулярные выражения)
Если вы передадите объект регулярного выражения, Beautiful Soup выполнит фильтрацию по этому регулярному выражению, используя свой метод `search()`. Этот код находит все теги, имена которых начинаются с буквы “b”; в данном случае тег `<body>` и тег `<b>`:

```python
import re
for tag in soup.find_all(re.compile("^b")):
    print(tag.name)
# body
# b
```
Этот код находит все теги, названия которых содержат букву ‘t’:

```python
for tag in soup.find_all(re.compile("t")):
    print(tag.name)
# html
# title
```
### A list (Список)
Если вы передадите список, Beautiful Soup разрешит сопоставление строки с любым элементом в этом списке. Этот код находит все теги `<a>` и все теги `<b>`:

```python
soup.find_all(["a", "b"])
# [<b>The Dormouse's story</b>,
#  <a class="sister" href="http://example.com/elsie" id="link1">Elsie</a>,
#  <a class="sister" href="http://example.com/lacie" id="link2">Lacie</a>,
#  <a class="sister" href="http://example.com/tillie" id="link3">Tillie</a>]
```
### True
Значение `True` соответствует всему, что только возможно. Этот код находит все теги в документе, но ни одну из текстовых строк:

```python
for tag in soup.find_all(True):
    print(tag.name)
# html
# head
# title
# body
# p
# b
# p
# a
# a
# a
# p
```
### A function (функция)
Если ни одно из других совпадений вам не подходит, определите функцию, которая принимает элемент в качестве единственного аргумента. Функция должна возвращать значение `True`, если аргумент совпадает, и значение `False` в противном случае.

Вот функция, которая возвращает значение `True`, если тег определяет атрибут “class”, но не определяет атрибут “id”:

```python
def has_class_but_no_id(tag):
    return tag.has_attr('class') and not tag.has_attr('id')
```
Передайте эту функцию в `find_all()`, и вы получите все теги `<p>`:

```python
soup.find_all(has_class_but_no_id)
# [<p class="title"><b>The Dormouse's story</b></p>,
#  <p class="story">Once upon a time there were...</p>,
#  <p class="story">...</p>]
```
Эта функция распознает только теги `<p>`. Он не распознает теги `<a>`, потому что эти теги определяют как “класс”, так и “идентификатор”. Он не распознает такие теги, как `<html>` и `<title>`, потому что эти теги не определяют “класс”.

Если вы передаете функцию для фильтрации по определенному атрибуту, такому как `href`, аргументом, передаваемым в функцию, будет значение атрибута, а не весь тег. Вот функция, которая находит все теги `a`, атрибут `href` которых не соответствует регулярному выражению:

```python
def not_lacie(href):
    return href and not re.compile("lacie").search(href)
soup.find_all(href=not_lacie)
# [<a class="sister" href="http://example.com/elsie" id="link1">Elsie</a>,
#  <a class="sister" href="http://example.com/tillie" id="link3">Tillie</a>]
```
Функция может быть настолько сложной, насколько вам это нужно. Вот функция, которая возвращает значение `True`, если тег окружен строковыми объектами:

```python
from bs4 import NavigableString
def surrounded_by_strings(tag):
    return (isinstance(tag.next_element, NavigableString)
            and isinstance(tag.previous_element, NavigableString))

for tag in soup.find_all(surrounded_by_strings):
    print tag.name
# p
# a
# a
# a
# p
```
Теперь мы готовы подробно рассмотреть методы поиска.

## find_all()
Signature: `find_all`(name, attrs, recursive, string, limit, **kwargs)

Метод `find_all()` просматривает потомков тега и извлекает всех потомков, которые соответствуют вашим фильтрам. Я привел несколько примеров в разделе Виды фильтров, но вот еще несколько:

```python
soup.find_all("title")
# [<title>The Dormouse's story</title>]

soup.find_all("p", "title")
# [<p class="title"><b>The Dormouse's story</b></p>]

soup.find_all("a")
# [<a class="sister" href="http://example.com/elsie" id="link1">Elsie</a>,
#  <a class="sister" href="http://example.com/lacie" id="link2">Lacie</a>,
#  <a class="sister" href="http://example.com/tillie" id="link3">Tillie</a>]

soup.find_all(id="link2")
# [<a class="sister" href="http://example.com/lacie" id="link2">Lacie</a>]

import re
soup.find(string=re.compile("sisters"))
# u'Once upon a time there were three little sisters; and their names were\n'
```
Некоторые из них должны выглядеть знакомыми, но другие являются новыми. Что значит передать значение для `string` или `id`? Почему `find_all("p", "title")` находит тег `<p>` с классом CSS “title”? Давайте посмотрим на аргументы функции `find_all()`.

### The name argument

Передайте значение для `name`, и вы скажете Beautiful Soup рассматривать только теги с определенными именами. Текстовые строки будут проигнорированы, как и теги, имена которых не совпадают.

Это самое простое использование:

```python
soup.find_all("title")
# [<title>The Dormouse's story</title>]
```
Вспомните из видов фильтров, что значение для присвоения имени может быть строкой, регулярным выражением, списком, функцией или значением True.

### The keyword arguments

Любой аргумент, который не распознан, будет преобразован в фильтр по одному из атрибутов тега. Если вы передадите значение для аргумента с именем `id`, Beautiful Soup выполнит фильтрацию по атрибуту ‘id’ каждого тега:

```python
soup.find_all(id='link2')
# [<a class="sister" href="http://example.com/lacie" id="link2">Lacie</a>]
```
Если вы передадите значение для `href`, Beautiful Soup выполнит фильтрацию по атрибуту ‘href’ каждого тега:

```python
soup.find_all(href=re.compile("elsie"))
# [<a class="sister" href="http://example.com/elsie" id="link1">Elsie</a>]
```
Вы можете отфильтровать атрибут на основе строки, регулярного выражения, списка, функции или значения True.

Этот код находит все теги, атрибут `id` которых имеет значение, независимо от того, каково это значение:

```python
soup.find_all(id=True)
# [<a class="sister" href="http://example.com/elsie" id="link1">Elsie</a>,
#  <a class="sister" href="http://example.com/lacie" id="link2">Lacie</a>,
#  <a class="sister" href="http://example.com/tillie" id="link3">Tillie</a>]
```
Вы можете отфильтровать сразу несколько атрибутов, передав более одного аргумента ключевого слова:

```python
soup.find_all(href=re.compile("elsie"), id='link1')
# [<a class="sister" href="http://example.com/elsie" id="link1">three</a>]
```
Некоторые атрибуты, такие как атрибуты data-* в HTML 5, имеют имена, которые нельзя использовать в качестве имен аргументов ключевых слов:

```python
data_soup = BeautifulSoup('<div data-foo="value">foo!</div>')
data_soup.find_all(data-foo="value")
# SyntaxError: keyword can't be an expression
```
Вы можете использовать эти атрибуты при поиске, поместив их в словарь и передав словарь в `find_all()` в качестве аргумента `attrs`:

```python
data_soup.find_all(attrs={"data-foo": "value"})
# [<div data-foo="value">foo!</div>]
```
Вы не можете использовать аргумент ключевого слова для поиска элемента HTML ‘name’, потому что Beautiful Soup использует аргумент `name` для указания имени самого тега. Вместо этого вы можете присвоить значение ‘name’ в аргументе `attrs`:

```python
name_soup = BeautifulSoup('<input name="email"/>')
name_soup.find_all(name="email")
# []
name_soup.find_all(attrs={"name": "email"})
# [<input name="email"/>]
```
### Searching by CSS class

Очень полезно искать тег, который имеет определенный класс CSS, но имя атрибута CSS, “class”, является зарезервированным словом в Python. Использование `class` в качестве аргумента ключевого слова приведет к синтаксической ошибке. Начиная с версии Beautiful Soup 4.1.2, вы можете выполнять поиск по классу CSS, используя ключевое слово argument `class_`:

```python
soup.find_all("a", class_="sister")
# [<a class="sister" href="http://example.com/elsie" id="link1">Elsie</a>,
#  <a class="sister" href="http://example.com/lacie" id="link2">Lacie</a>,
#  <a class="sister" href="http://example.com/tillie" id="link3">Tillie</a>]
```
Как и в случае с любым аргументом ключевого слова, вы можете передать `class_` строку, регулярное выражение, функцию или значение `True`:

```python
soup.find_all(class_=re.compile("itl"))
# [<p class="title"><b>The Dormouse's story</b></p>]

def has_six_characters(css_class):
    return css_class is not None and len(css_class) == 6

soup.find_all(class_=has_six_characters)
# [<a class="sister" href="http://example.com/elsie" id="link1">Elsie</a>,
#  <a class="sister" href="http://example.com/lacie" id="link2">Lacie</a>,
#  <a class="sister" href="http://example.com/tillie" id="link3">Tillie</a>]
```
Помните, что один тег может иметь несколько значений для своего атрибута “class”. Когда вы ищете тег, соответствующий определенному классу CSS, вы сопоставляете его с любым из его классов CSS:

```python
css_soup = BeautifulSoup('<p class="body strikeout"></p>')
css_soup.find_all("p", class_="strikeout")
# [<p class="body strikeout"></p>]

css_soup.find_all("p", class_="body")
# [<p class="body strikeout"></p>]
```
Вы также можете выполнить поиск по точному строковому значению атрибута `class`:

```python
css_soup.find_all("p", class_="body strikeout")
# [<p class="body strikeout"></p>]
```
Но поиск вариантов строкового значения не сработает:

```python
css_soup.find_all("p", class_="strikeout body")
# []
```
Если вы хотите выполнить поиск тегов, соответствующих двум или более классам CSS, вам следует использовать селектор CSS:

```python
css_soup.select("p.strikeout.body")
# [<p class="body strikeout"></p>]
```
В старых версиях Beautiful Soup, в которых нет ярлыка `class_`, вы можете использовать трюк `attrs`, упомянутый выше. Создайте словарь, значением которого для “class” является строка (или регулярное выражение, или что-то еще), которую вы хотите найти:

```python
soup.find_all("a", attrs={"class": "sister"})
# [<a class="sister" href="http://example.com/elsie" id="link1">Elsie</a>,
#  <a class="sister" href="http://example.com/lacie" id="link2">Lacie</a>,
#  <a class="sister" href="http://example.com/tillie" id="link3">Tillie</a>]
```
### The `string` argument
С помощью `string` вы можете выполнять поиск по строкам вместо тегов. Как и в случае с аргументами `name` и ключевого слова, вы можете передать строку, регулярное выражение, список, функцию или значение True. Вот несколько примеров:

```python
soup.find_all(string="Elsie")
# [u'Elsie']

soup.find_all(string=["Tillie", "Elsie", "Lacie"])
# [u'Elsie', u'Lacie', u'Tillie']

soup.find_all(string=re.compile("Dormouse"))
[u"The Dormouse's story", u"The Dormouse's story"]

def is_the_only_string_within_a_tag(s):
    """Return True if this string is the only child of its parent tag."""
    return (s == s.parent.string)

soup.find_all(string=is_the_only_string_within_a_tag)
# [u"The Dormouse's story", u"The Dormouse's story", u'Elsie', u'Lacie', u'Tillie', u'...']
```
Хотя `string` предназначен для поиска строк, вы можете объединить его с аргументами, которые находят теги: Beautiful Soup найдет все теги, `.string` которых соответствует вашему значению для `string`. Этот код находит теги `<a>`, строка которых равна “Elsie”.:

```python
oup.find_all("a", string="Elsie")
# [<a href="http://example.com/elsie" class="sister" id="link1">Elsie</a>]
```
Аргумент `string` является новым в Beautiful Soup 4.4.0. В более ранних версиях он назывался `text`:

```python
soup.find_all("a", text="Elsie")
# [<a href="http://example.com/elsie" class="sister" id="link1">Elsie</a>]
```
### The limit argument(Предельный аргумент)
Функция `find_all()` возвращает все теги и строки, соответствующие вашим фильтрам. Это может занять некоторое время, если документ большой. Если вам не нужны все результаты, вы можете ввести число для ограничения. Это работает точно так же, как ключевое слово LIMIT в SQL. Он сообщает Beautiful Soup прекратить сбор результатов после того, как будет найдено определенное число.

В документе “три сестры” есть три ссылки, но этот код находит только первые две:

```python
soup.find_all("a", limit=2)
# [<a class="sister" href="http://example.com/elsie" id="link1">Elsie</a>,
#  <a class="sister" href="http://example.com/lacie" id="link2">Lacie</a>]
```
### The recursive argument (Рекурсивный аргумент)

Если вы вызовете `mytag.find_all()`, Beautiful Soup проверит всех потомков `mytag`: его дочерних элементов, дочерних элементов его дочерних элементов и так далее. Если вы хотите, чтобы Beautiful Soup учитывал только прямых дочерних элементов, вы можете передать значение `recursive=False`. Посмотрите на разницу здесь:
```python
soup.html.find_all("title")
# [<title>The Dormouse's story</title>]

soup.html.find_all("title", recursive=False)
# []
```
Вот эта часть документа

```python
<html>
 <head>
  <title>
   The Dormouse's story
  </title>
 </head>
...
```
Тег `<title>` находится под тегом `<html>`, но не непосредственно под тегом `<html>`: тег `<head>` мешает. Beautiful Soup находит тег `<title>`, когда ему разрешено просматривать всех потомков тега `<html>`, но когда recursive=False ограничивает его непосредственными дочерними элементами тега `<html>`, он ничего не находит.

Beautiful Soup предлагает множество методов поиска по дереву (рассмотренных ниже), и они в основном принимают те же аргументы, что и `find_all()`: `name`, `attrs`, `string`, `limit` и аргументы ключевого слова. Но рекурсивный аргумент отличается: `find_all()` и `find()` - единственные методы, которые его поддерживают. Передача `recursive=False` в такой метод, как `find_parents()`, была бы не очень полезна.

## Вызов тега подобно вызову функции `find_all()`

Поскольку find_all() - самый популярный метод в API поиска Beautiful Soup, вы можете использовать для него ярлык. Если вы относитесь к объекту BeautifulSoup или объекту Tag как к функции, то это то же самое, что вызвать find_all() для этого объекта. Эти две строки кода эквивалентны:

```python
soup.find_all("a")
soup("a")
```
Эти две строки также эквивалентны:

```python
soup.title.find_all(string=True)
soup.title(string=True)
```
## `find()`

Signature: find(name, attrs, recursive, string, **kwargs)

Метод `find_all()` сканирует весь документ в поисках результатов, но иногда вы хотите найти только один результат. Если вы знаете, что в документе есть только один тег `<body>`, сканировать весь документ в поисках дополнительных - пустая трата времени. Вместо того чтобы передавать значение `limit=1` каждый раз, когда вы вызываете `find_all`, вы можете использовать метод `find()`. Эти две строки кода почти эквивалентны:

```python
soup.find_all('title', limit=1)
# [<title>The Dormouse's story</title>]

soup.find('title')
# <title>The Dormouse's story</title>
```
Единственная разница заключается в том, что функция `find_all()` возвращает список, содержащий единственный результат, а функция `find()` просто возвращает результат.

Если функция `find_all()` ничего не может найти, она возвращает пустой список. Если функция `find()` ничего не может найти, она возвращает None:

```python
print(soup.find("nosuchtag"))
# None
```
Помните трюк с `soup.head.title` при навигации по именам тегов? Этот трюк работает при повторном вызове функции `find()`:

```python
soup.head.title
# <title>The Dormouse's story</title>

soup.find("head").find("title")
# <title>The Dormouse's story</title>
```
### `find_parents()` и `find_parent()`
Signature: find_parents(name, attrs, string, limit, **kwargs)

Signature: find_parent(name, attrs, string, **kwargs)

Я потратил много времени на рассмотрение функций `find_all()` и `find()`. API Beautiful Soup определяет десять других методов поиска по дереву, но не бойтесь. Пять из этих методов в основном такие же, как `find_all()`, а остальные пять в основном такие же, как `find()`. Единственные различия заключаются в том, в каких частях дерева они выполняют поиск.

Сначала давайте рассмотрим функции `find_parents()` и `find_parent()`. Помните, что `find_all()` и `find()` продвигаются вниз по дереву, просматривая потомков тега. Эти методы делают обратное: они прокладывают себе путь вверх по дереву, просматривая родительские элементы тега (или строки). Давайте попробуем их, начав со строки, глубоко спрятанной в документе “три дочери”.:

```python
a_string = soup.find(string="Lacie")
a_string
# u'Lacie'

a_string.find_parents("a")
# [<a class="sister" href="http://example.com/lacie" id="link2">Lacie</a>]

a_string.find_parent("p")
# <p class="story">Once upon a time there were three little sisters; and their names were
#  <a class="sister" href="http://example.com/elsie" id="link1">Elsie</a>,
#  <a class="sister" href="http://example.com/lacie" id="link2">Lacie</a> and
#  <a class="sister" href="http://example.com/tillie" id="link3">Tillie</a>;
#  and they lived at the bottom of a well.</p>

a_string.find_parents("p", class="title")
# []
```
Один из трех тегов `<a>` является прямым родительским элементом рассматриваемой строки, поэтому наш поиск находит его. Один из трех тегов `<p>` является косвенным родительским элементом строки, и наш поиск также находит его. Где-то в документе есть тег `<p>` с классом CSS “title”, но он не является одним из родительских элементов этой строки, поэтому мы не можем найти его с помощью `find_parents()`.

Возможно, вы установили связь между `find_parent()` и `find_parents()`, а также атрибутами `.parent` и `.parents`, упомянутыми ранее. Связь очень сильная. Эти методы поиска фактически используют `.parents` для перебора всех родительских элементов и проверки каждого из них на соответствие предоставленному фильтру, чтобы увидеть, соответствует ли он.

## `find_next_siblings()` и `find_next_sibling()`

Signature: find_next_siblings(name, attrs, string, limit, **kwargs)

Signature: find_next_sibling(name, attrs, string, **kwargs)

Эти методы используют `.next_siblings` для перебора остальных дочерних элементов элемента в дереве. Метод `find_next_siblings()` возвращает все совпадающие братья и сестры, а `find_next_sibling()` возвращает только первый из них:

```python
first_link = soup.a
first_link
# <a class="sister" href="http://example.com/elsie" id="link1">Elsie</a>

first_link.find_next_siblings("a")
# [<a class="sister" href="http://example.com/lacie" id="link2">Lacie</a>,
#  <a class="sister" href="http://example.com/tillie" id="link3">Tillie</a>]

first_story_paragraph = soup.find("p", "story")
first_story_paragraph.find_next_sibling("p")
# <p class="story">...</p>
```
## `find_previous_siblings()` и `find_previous_sibling()`
Signature: find_previous_siblings(name, attrs, string, limit, **kwargs)

Signature: find_previous_sibling(name, attrs, string, **kwargs)

Эти методы используют `.previous_siblings` для перебора дочерних элементов элемента, которые предшествуют ему в дереве. Метод `find_previous_siblings()` возвращает все совпадающие братья и сестры, а `find_previous_sibling()` возвращает только первый из них:

```python
last_link = soup.find("a", id="link3")
last_link
# <a class="sister" href="http://example.com/tillie" id="link3">Tillie</a>

last_link.find_previous_siblings("a")
# [<a class="sister" href="http://example.com/lacie" id="link2">Lacie</a>,
#  <a class="sister" href="http://example.com/elsie" id="link1">Elsie</a>]

first_story_paragraph = soup.find("p", "story")
first_story_paragraph.find_previous_sibling("p")
# <p class="title"><b>The Dormouse's story</b></p>
```
## `find_all_next()` и `find_next()`
Signature: find_all_next(name, attrs, string, limit, **kwargs)

Signature: find_next(name, attrs, string, **kwargs)

Эти методы используют `.next_elements` для перебора любых тегов и строк, которые следуют за ним в документе. Метод `find_all_next()` возвращает все совпадения, а `find_next()` возвращает только первое совпадение:

```python
first_link = soup.a
first_link
# <a class="sister" href="http://example.com/elsie" id="link1">Elsie</a>

first_link.find_all_next(string=True)
# [u'Elsie', u',\n', u'Lacie', u' and\n', u'Tillie',
#  u';\nand they lived at the bottom of a well.', u'\n\n', u'...', u'\n']

first_link.find_next("p")
# <p class="story">...</p>
```
В первом примере появилась строка “Elsie”, хотя она содержалась в теге `<a>`, с которого мы начали. Во втором примере появился последний тег `<p>` в документе, хотя он находится не в той же части дерева, что и тег `<a>`, с которого мы начали. Для этих методов все, что имеет значение, - это то, что элемент соответствует фильтру и отображается в документе позже, чем начальный элемент.

## `find_all_previous()` и `find_previous()`
Signature: find_all_previous(name, attrs, string, limit, **kwargs)

Signature: find_previous(name, attrs, string, **kwargs)

Эти методы используют `.previous_elements` для перебора тегов и строк, которые были до него в документе. Метод `find_all_previous()` возвращает все совпадения, а `find_previous() `возвращает только первое совпадение:

```python
first_link = soup.a
first_link
# <a class="sister" href="http://example.com/elsie" id="link1">Elsie</a>

first_link.find_all_previous("p")
# [<p class="story">Once upon a time there were three little sisters; ...</p>,
#  <p class="title"><b>The Dormouse's story</b></p>]

first_link.find_previous("title")
# <title>The Dormouse's story</title>
```
Вызов `find_all_previous("p")` нашел первый абзац в документе (тот, который имеет class=”title”), но он также находит второй абзац, тег `<p>`, который содержит тег `<a>`, с которого мы начали. Это не должно слишком удивлять: мы просматриваем все теги, которые появляются в документе раньше, чем тот, с которого мы начали. Тег `<p>`, содержащий тег `<a>`, должен был отображаться перед тегом `<a>`, который он содержит.

## Селекторы CSS (CSS selectors)
Начиная с версии 4.7.0, Beautiful Soup поддерживает большинство селекторов CSS4 через проект SoupSieve. Если вы установили Beautiful Soup через `pip`, SoupSieve был установлен одновременно, так что вам не нужно делать ничего дополнительного.

В `BeautifulSoup` есть метод .`select()`, который использует SoupSieve для запуска CSS-селектора для проанализированного документа и возврата всех совпадающих элементов. У `Tag` есть аналогичный метод, который запускает CSS-селектор для содержимого одного тега.

(В более ранних версиях `Beautiful Soup` также есть метод `.select()`, но поддерживаются только наиболее часто используемые CSS-селекторы.)

В документации SoupSieve перечислены все поддерживаемые в настоящее время CSS-селекторы, но вот некоторые из основных:

Вы можете найти теги:
```python
soup.select("title")
# [<title>The Dormouse's story</title>]

soup.select("p:nth-of-type(3)")
# [<p class="story">...</p>]
```
Найдите теги под другими тегами:

```python
soup.select("body a")
# [<a class="sister" href="http://example.com/elsie" id="link1">Elsie</a>,
#  <a class="sister" href="http://example.com/lacie"  id="link2">Lacie</a>,
#  <a class="sister" href="http://example.com/tillie" id="link3">Tillie</a>]

soup.select("html head title")
# [<title>The Dormouse's story</title>]
```
Найдите теги непосредственно под другими тегами:

```python
soup.select("head > title")
# [<title>The Dormouse's story</title>]

soup.select("p > a")
# [<a class="sister" href="http://example.com/elsie" id="link1">Elsie</a>,
#  <a class="sister" href="http://example.com/lacie"  id="link2">Lacie</a>,
#  <a class="sister" href="http://example.com/tillie" id="link3">Tillie</a>]

soup.select("p > a:nth-of-type(2)")
# [<a class="sister" href="http://example.com/lacie" id="link2">Lacie</a>]

soup.select("p > #link1")
# [<a class="sister" href="http://example.com/elsie" id="link1">Elsie</a>]

soup.select("body > a")
# []
```
Найдите братьев и сестер тегов:

```python
soup.select("#link1 ~ .sister")
# [<a class="sister" href="http://example.com/lacie" id="link2">Lacie</a>,
#  <a class="sister" href="http://example.com/tillie"  id="link3">Tillie</a>]

soup.select("#link1 + .sister")
# [<a class="sister" href="http://example.com/lacie" id="link2">Lacie</a>]
```
Поиск тегов по классу CSS:

```python
soup.select(".sister")
# [<a class="sister" href="http://example.com/elsie" id="link1">Elsie</a>,
#  <a class="sister" href="http://example.com/lacie" id="link2">Lacie</a>,
#  <a class="sister" href="http://example.com/tillie" id="link3">Tillie</a>]

soup.select("[class~=sister]")
# [<a class="sister" href="http://example.com/elsie" id="link1">Elsie</a>,
#  <a class="sister" href="http://example.com/lacie" id="link2">Lacie</a>,
#  <a class="sister" href="http://example.com/tillie" id="link3">Tillie</a>]
```
Поиск тегов по идентификатору:

```python
soup.select("#link1")
# [<a class="sister" href="http://example.com/elsie" id="link1">Elsie</a>]

soup.select("a#link2")
# [<a class="sister" href="http://example.com/lacie" id="link2">Lacie</a>]
```
Найдите теги, соответствующие любому селектору из списка селекторов:

```python
soup.select("#link1,#link2")
# [<a class="sister" href="http://example.com/elsie" id="link1">Elsie</a>,
#  <a class="sister" href="http://example.com/lacie" id="link2">Lacie</a>]
```
Проверка на наличие атрибута:

```python
soup.select('a[href]')
# [<a class="sister" href="http://example.com/elsie" id="link1">Elsie</a>,
#  <a class="sister" href="http://example.com/lacie" id="link2">Lacie</a>,
#  <a class="sister" href="http://example.com/tillie" id="link3">Tillie</a>]
```
Поиск тегов по значению атрибута:

```python
soup.select('a[href="http://example.com/elsie"]')
# [<a class="sister" href="http://example.com/elsie" id="link1">Elsie</a>]

soup.select('a[href^="http://example.com/"]')
# [<a class="sister" href="http://example.com/elsie" id="link1">Elsie</a>,
#  <a class="sister" href="http://example.com/lacie" id="link2">Lacie</a>,
#  <a class="sister" href="http://example.com/tillie" id="link3">Tillie</a>]

soup.select('a[href$="tillie"]')
# [<a class="sister" href="http://example.com/tillie" id="link3">Tillie</a>]

soup.select('a[href*=".com/el"]')
# [<a class="sister" href="http://example.com/elsie" id="link1">Elsie</a>]
```
Существует также метод `select_one()`, который находит только первый тег, соответствующий селектору:

```python
soup.select_one(".sister")
# <a class="sister" href="http://example.com/elsie" id="link1">Elsie</a>
```
Если вы проанализировали XML, который определяет пространства имен, вы можете использовать их в селекторах CSS.:

```python
from bs4 import BeautifulSoup
xml = """<tag xmlns:ns1="http://namespace1/" xmlns:ns2="http://namespace2/">
 <ns1:child>I'm in namespace 1</ns1:child>
 <ns2:child>I'm in namespace 2</ns2:child>
</tag> """
soup = BeautifulSoup(xml, "xml")

soup.select("child")
# [<ns1:child>I'm in namespace 1</ns1:child>, <ns2:child>I'm in namespace 2</ns2:child>]

soup.select("ns1|child", namespaces=namespaces)
# [<ns1:child>I'm in namespace 1</ns1:child>]
```
При обработке CSS-селектора, использующего пространства имен, Beautiful Soup использует сокращения пространства имен, найденные при разборе документа. Вы можете переопределить это, введя свой собственный словарь сокращений:

```python
namespaces = dict(first="http://namespace1/", second="http://namespace2/")
soup.select("second|child", namespaces=namespaces)
# [<ns1:child>I'm in namespace 2</ns1:child>]
```
Весь этот материал с CSS-селектором удобен для людей, которые уже знают синтаксис CSS-селектора. Вы можете сделать все это с помощью прекрасного API Soup. И если CSS-селекторы - это все, что вам нужно, вам следует проанализировать документ с помощью lxml: это намного быстрее. Но это позволяет вам комбинировать CSS-селекторы с прекрасным Soup API.

# Изменение дерева (Modifying the tree)

Основная сила Beautiful Soup заключается в поиске по дереву синтаксического анализа, но вы также можете модифицировать дерево и записать свои изменения в виде нового HTML- или XML-документа.

## Изменение имен тегов и атрибутов
Я рассказывал об этом ранее, в разделе Атрибуты, но это стоит повторить. Вы можете переименовать тег, изменить значения его атрибутов, добавить новые атрибуты и удалить атрибуты:

```python
soup = BeautifulSoup('<b class="boldest">Extremely bold</b>')
tag = soup.b

tag.name = "blockquote"
tag['class'] = 'verybold'
tag['id'] = 1
tag
# <blockquote class="verybold" id="1">Extremely bold</blockquote>

del tag['class']
del tag['id']
tag
# <blockquote>Extremely bold</blockquote>
```
## Изменение `.string`

Если вы присваиваете атрибуту `.string` тега значение новой строки, содержимое тега заменяется этой строкой:

```python
markup = '<a href="http://example.com/">I linked to <i>example.com</i></a>'
soup = BeautifulSoup(markup)

tag = soup.a
tag.string = "New link text."
tag
# <a href="http://example.com/">New link text.</a>
```
Будьте осторожны: если тег содержал другие теги, они и все их содержимое будут уничтожены.
## `append()`

Вы можете добавить к содержимому тега с помощью `Tag.append()`. Это работает так же, как вызов `.append()` в списке Python:

```python
soup = BeautifulSoup("<a>Foo</a>")
soup.a.append("Bar")

soup
# <html><head></head><body><a>FooBar</a></body></html>
soup.a.contents
# [u'Foo', u'Bar']
```
## `extend()`
Начиная с Beautiful Soup 4.7.0, `Tag` также поддерживает метод с именем `.extend()`, который работает точно так же, как вызов `.extend()` в списке Python:
```python
soup = BeautifulSoup("<a>Soup</a>")
soup.a.extend(["'s", " ", "on"])

soup
# <html><head></head><body><a>Soup's on</a></body></html>
soup.a.contents
# [u'Soup', u''s', u' ', u'on']
```
## `NavigableString()` и .`new_tag()`

Если вам нужно добавить строку в документ, нет проблем – вы можете передать строку Python в функцию `append()` или вызвать конструктор `NavigableString`:

```python
soup = BeautifulSoup("<b></b>")
tag = soup.b
tag.append("Hello")
new_string = NavigableString(" there")
tag.append(new_string)
tag
# <b>Hello there.</b>
tag.contents
# [u'Hello', u' there']
```
Если вы хотите создать комментарий или какой-либо другой подкласс `NavigableString`, просто вызовите конструктор:

```python
from bs4 import Comment
new_comment = Comment("Nice to see you.")
tag.append(new_comment)
tag
# <b>Hello there<!--Nice to see you.--></b>
tag.contents
# [u'Hello', u' there', u'Nice to see you.']
```
(Это новая функция в Beautiful Soup 4.4.0.)

Что, если вам нужно создать совершенно новый тег? Лучшим решением является вызов заводского метода `BeautifulSoup.new_tag()`:

```python
soup = BeautifulSoup("<b></b>")
original_tag = soup.b

new_tag = soup.new_tag("a", href="http://www.example.com")
original_tag.append(new_tag)
original_tag
# <b><a href="http://www.example.com"></a></b>

new_tag.string = "Link text."
original_tag
# <b><a href="http://www.example.com">Link text.</a></b>
```
Требуется только первый аргумент, имя тега.

## `insert()`

Tag.insert() похож на Tag.append(), за исключением того, что новый элемент не обязательно находится в конце родительского .contents. Он будет вставлен в любую указанную вами числовую позицию. Это работает так же, как .insert() в списке Python:

```python
markup = '<a href="http://example.com/">I linked to <i>example.com</i></a>'
soup = BeautifulSoup(markup)
tag = soup.a

tag.insert(1, "but did not endorse ")
tag
# <a href="http://example.com/">I linked to but did not endorse <i>example.com</i></a>
tag.contents
# [u'I linked to ', u'but did not endorse', <i>example.com</i>]
```
## `insert_before()` и `insert_after()`

Метод `insert_before()` вставляет теги или строки непосредственно перед чем-либо другим в дереве синтаксического анализа:

```python
soup = BeautifulSoup("<b>stop</b>")
tag = soup.new_tag("i")
tag.string = "Don't"
soup.b.string.insert_before(tag)
soup.b
# <b><i>Don't</i>stop</b>
```
Метод `insert_after()` вставляет теги или строки непосредственно после чего-либо другого в дереве синтаксического анализа:

```python
div = soup.new_tag('div')
div.string = 'ever'
soup.b.i.insert_after(" you ", div)
soup.b
# <b><i>Don't</i> you <div>ever</div> stop</b>
soup.b.contents
# [<i>Don't</i>, u' you', <div>ever</div>, u'stop']
```
## `clear()`

`Tag.clear()` удаляет содержимое тега:

```python
markup = '<a href="http://example.com/">I linked to <i>example.com</i></a>'
soup = BeautifulSoup(markup)
tag = soup.a

tag.clear()
tag
# <a href="http://example.com/"></a>
```
## `extract()`

`PageElement.extract()` удаляет тег или строку из дерева. Он возвращает тег или строку, которые были извлечены:

```python
markup = '<a href="http://example.com/">I linked to <i>example.com</i></a>'
soup = BeautifulSoup(markup)
a_tag = soup.a

i_tag = soup.i.extract()

a_tag
# <a href="http://example.com/">I linked to</a>

i_tag
# <i>example.com</i>

print(i_tag.parent)
None
```
На данный момент у вас фактически есть два дерева синтаксического анализа: одно с корнем в объекте `BeautifulSoup`, который вы использовали для синтаксического анализа документа, и одно с корнем в извлеченном теге. Вы можете перейти к вызову `extract` для дочернего элемента, который вы извлекли:

```python
my_string = i_tag.string.extract()
my_string
# u'example.com'

print(my_string.parent)
# None
i_tag
# <i></i>
```
## `decompose()`

`Tag.decompose()` удаляет тег из дерева, затем полностью уничтожает его и его содержимое:

```python
markup = '<a href="http://example.com/">I linked to <i>example.com</i></a>'
soup = BeautifulSoup(markup)
a_tag = soup.a

soup.i.decompose()

a_tag
# <a href="http://example.com/">I linked to</a>
```
## `replace_with()`
`PageElement.replace_with() `удаляет тег или строку из дерева и заменяет их тегом или строкой по вашему выбору:

```python
markup = '<a href="http://example.com/">I linked to <i>example.com</i></a>'
soup = BeautifulSoup(markup)
a_tag = soup.a

new_tag = soup.new_tag("b")
new_tag.string = "example.net"
a_tag.i.replace_with(new_tag)

a_tag
# <a href="http://example.com/">I linked to <b>example.net</b></a>
```
`replace_with()` возвращает тег или строку, которые были заменены, чтобы вы могли просмотреть их или добавить обратно в другую часть дерева.

## `wrap()`

`PageElement.wrap()` обертывает элемент в указанном вами теге. Он возвращает новую оболочку:

```python
soup = BeautifulSoup("<p>I wish I was bold.</p>")
soup.p.string.wrap(soup.new_tag("b"))
# <b>I wish I was bold.</b>

soup.p.wrap(soup.new_tag("div")
# <div><p><b>I wish I was bold.</b></p></div>
```
тот метод является новым в Beautiful Soup 4.0.5.

## `unwrap()`

Функция `Tag.unwrap()` является противоположностью функции `wrap()`. Он заменяет тег на то, что находится внутри этого тега. Это хорошо для удаления разметки:

```python
markup = '<a href="http://example.com/">I linked to <i>example.com</i></a>'
soup = BeautifulSoup(markup)
a_tag = soup.a

a_tag.i.unwrap()
a_tag
# <a href="http://example.com/">I linked to example.com</a>
```
Как и` replace_with()`, `unwrap()` возвращает тег, который был заменен.

## `smooth()`
После вызова множества методов, которые изменяют дерево синтаксического анализа, вы можете в конечном итоге получить два или более объектов `NavigableString` рядом друг с другом. У Beautiful Soup с этим нет никаких проблем, но поскольку это не может произойти в только что проанализированном документе, вы можете не ожидать поведения, подобного следующему:
```python
soup = BeautifulSoup("<p>A one</p>")
soup.p.append(", a two")

soup.p.contents
# [u'A one', u', a two']

print(soup.p.encode())
# <p>A one, a two</p>

print(soup.p.prettify())
# <p>
#  A one
#  , a two
# </p>
```
Вы можете вызвать `Tag.smooth()`, чтобы очистить дерево синтаксического анализа, объединив соседние строки:

```python
soup.smooth()

soup.p.contents
# [u'A one, a two']

print(soup.p.prettify())
# <p>
#  A one, a two
# </p>
```
Метод smooth() является новым в Beautiful Soup 4.8.0.

```python

```


```python

```


```python

```


```python

```


```python

```


```python

```


```python

```


```python

```


```python

```


```python

```


```python

```


```python

```