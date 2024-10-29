Should играет роль ждущего assert. Это явное ожидание.

Явные ожидания ждут, когда определенное условие выполнится.

В контексте проверок бывают случае, когда мы не хотим, чтобы они были как assert.

Assert - если условие оправдалось, что мы идем дальше, если не оправдалось, то тест завершается с ошибкой.

Но иногда мы просто хотим знать, выполнилось условие или нет и пойти дальше. (чтобы не вылетала ошибка)

В Selenium `wait.until` есть assert. Если условие не выполнится по итогу ожидание, то оно упадет.

А в Selene `wait_until` будет ждать выполнения условия и если оно не выполнится, то он не бросит ошибку, 
а просто вернет False.

`matching` в Selene не ждет, как `wait_until`, он сразу проверяет.

Title  ждать нет необходимости, так как этот тот элемент страницы, который не поменяется.

Т.е. по факту `matching` и `wait_until` в Selene нужны для того, чтобы использовать их с оператором `if` для проверки 
выполнения каких-то условий без вызова ошибки.

If и циклы ы в тестах не используем. Код тестов должен быть максимально тупой. Тесты просто должны вызывать команды.
И никаких ветвлений там не должно быть.

И если так случилось, что необходимо создать какое-то ветвление, то нужно его скрыть, создав функцию или используя 
Page Object. Но в самих тестах их не должно быть.

___

#### Фриз инспектора для заморозки пропадающих компонентов

В консоли браузера можем вызвать метод setTimeout, который вызовет заданную функцию через заданное время. 

```
setTimeout(() => alert('3 seconds passed'), 3000)
```

Аналогично можно вызвать функцию, которая зафризит браузер на заданное время, а именно:
```
setTimeout('debugger', 3000)
```
или
```
setTimeout(function(){debugger;}, 5000)
```

___
#### 
 `browser.config.hold_driver_at_exit = True` - опция, чтобы следующий тест открылся в том же браузере

Весь кол ниже (фикстуры нет вообще):
```python
from selene import browser, have


def test_finds_selene():
    browser.open('https://www.google.com/ncr').should(have.title('Google'))
    browser.element('[name=q]').type('selene').press_enter()
    browser.element('#search').should(have.text('User-oriented Web UI browser tests in'))

    results = browser.all('#rso>div')
    results.should(have.size_greater_than_or_equal(6))


def test_finds_selene_with_refined_query():
    browser.config.hold_driver_at_exit = True
    browser.open('https://www.google.com/ncr').should(have.title('Google'))
    browser.element('[name=q]').type('selene').press_enter()
    results = browser.all('#rso>div')
    results.should(have.size_greater_than_or_equal(6))

    browser.element('[name=q]').type(' yashaka github').press_enter()
    results.first.element('h3').click()
    browser.should(have.title_containing('yashaka/selene'))
```

___

Когда нужен элемент, который можно найти только по характеристикам, указанным во внутренних элементах:

`//*[@class="todo-list"]/li[.//*[text()="b"]]` или `//*[@class="todo-list"]/li[.//*[.="b"]]`
Чтобы остаться на уровне `li`, характеристики мы указываем в квадратных скобках, что от текущего местоположения `.` на 
любую глубину вложенности `//` есть елемент с тестом `'b'`.

___

Суммарный селектор на XPath очень мудреный и по нему очень сложно понять, что сломалось, есливылезет ошибка. Придется 
тратить много времени на восстановление работоспособности теста

`browser.element('//*[@class="todo-list"]/li[.//*[.="b"]]//*[contains(@class,"toggle")]').click()`

Аналогичное на Selene:
`browser.all('.todo-list>li').element_by(have.exact_text('b')).element('.toggle').click()`

____
Selene:
`browser.all('.todo-list>li').by(have.no.css_class('completed')).should(have.exact_texts('a.', 'c.', 'd.'))`
CSS:
`browser.all('.todo-list>li:not(.completed)').should(have.exact_texts('a.', 'c.', 'd.'))`

___

Если рабтаем в основном с Селеном, но необходимо использовать ActionChains, в котором используются элементы Selenium, то
нужно будет найденным элементам Selene добавить `.locale()`. который как раз вернут WebElement Selenium:

```python
from selene import browser
from selenium.webdriver import ActionChains

ActionChains(browser.driver).drag_and_drop(browser.element('#from').locate(), browser.element('to').locate())
```
Или более краткое:
```python
from selene import browser
from selenium.webdriver import ActionChains

ActionChains(browser.driver).drag_and_drop(browser.element('#from')(), browser.element('to')())
```
___

Для полей ввода, то, что мы ввели в html является value.

Для нахождения какой-то value через devtools:

```javascipt
document.querySelector('#userName').value
```
___
Если нужно получить атрибут text, используя Selene, то можно так:
`browser.element('#userName').locate().text`
либо так
`browser.element('#userName').get(query.text)`

Разработчиком это было сделано специально, чтобы не было большого соблазна использовать текст для последующих проверок,
так как в тестах опираться на текст очень опрометчиво.

В большинстве случае хороший тест можно написать без данных атрибутов. Для этого у теста должен быть хороший прекондишн,
благодаря которому не придется через UI считывать необходимое значение, а оно уже есть в базе, либо через API туда 
зафигачено. 
___

Почему assert это зло? Потому что не ждущий. А при тестировании UI мы имеем дело с динамическими приложениями, 
которые грузятся какое-то время.

___

Если нам необходимо, чтобы создавалось сколько угодно браузеров, то нас нужно создать команду, которая бы создавала
браузеры.

```python
new_browser = lambda _: Browser(Config(driver=webdriver.Chrome(service=ChromeService(ChromeDriverManager().install()))))
```

или (одно и то же)

```python
def new_browser():
    return Browser(Config(driver=webdriver.Chrome(service=ChromeService(ChromeDriverManager().install()))))
```