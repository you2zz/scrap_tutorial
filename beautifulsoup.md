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

```python

```


```python

```