## Об'єкти без методів

(Оригінал цієї статті знаходиться [тут](https://www.yegor256.com/2020/11/24/objects-without-methods.html).)

24 листопада 2020 р. Москва, Росія.

Автор: [Єгор Бугаєнко](https://www.yegor256.com).

Як ви думаєте, що таке об'єкт в ООП? На якій мові ви б не програмували, мабуть, ви погодитеся з Брюсом Екелом (Bruce Eckel), автором ["Thinking in Java"](https://amzn.to/3pRHv1Q), який сказав, що "кожен об'єкт має стан і має дії, які можна попросити його виконати", і з Бенджаміном Евансом, автором ["Java in a Nutshell"](https://amzn.to/35uKVPU), який заявив, що об'єкт — це "набір полів даних для збереження значень і методів, які працюють з цими даними". Але хвилинку... Якби я вам сказав, що об'єкт може бути без "операцій" і все-таки залишатися ідеальним "еквівалентом квантів, з яких побудований Всесвіт", як висловився Дейвід Вест у своїй чудовій книжці ["Object Thinking"](https://amzn.to/3kuXHlL)?

![The Ballad of Buster Scruggs](/the-ballad-of-buster-scruggs.jpg)
(Image :copyright: The Ballad of Buster Scruggs (2018) by Coen brothers)

В [EO](https://www.eolang.org/), нашій експериментальній мові, ми зробили спробу перевизначити ООП і його об'єкти. Є два види *речей* в EO: атоми й об'єкти. Атом є мовним примітивом найнижчого рівня, який неможливо виразити іншими атомами. Наприклад, арифметичне додавання двох об'єктів є атомом (не тікайте від мене, це синтаксис EO, ви звикнете):

```
add 5 y > x
```

У більш традиційній Java-подібній [інфіксній нотації](https://uk.wikipedia.org/wiki/%D0%86%D0%BD%D1%84%D1%96%D0%BA%D1%81%D0%BD%D0%B0_%D0%BD%D0%BE%D1%82%D0%B0%D1%86%D1%96%D1%8F) цей код виглядає так:

```java
x = add(5, y)
```

Тут атом — `add`, а конкретними аргументами є `5` і `y`. Цей вираз створює новий атом, беручи за основу існуючий і вказуючи його аргументи. Ім'я нового створеного атому є `x`. Як тільки ми просимо цей новостворений атом зробити щось, він бере те, що лежить в `y`, додає `5` і починає поводитись, як їх сума. А до того часу він мовчить. EO є декларативною мовою.

Атоми надаються часом виконання EO. Наприклад, `add`, `sub`, `mul` і `div` є для арифметичних операцій; `if` і `for` є для розгалуження й ітерації; `less`, `and`, `eq`, `or` — для логічних операцій, і так далі. Атоми нагадують низькорівневі функції з аргументами. Однак вони не обчислюють результати негайно, а тільки коли треба. Вираз `add(5, file)` не призведе до негайного читання файлу і додавання 5 до нього. Аж коли зі створеним атомом будуть щось робити, відбудеться читання файлу.

Далі. На основі цих атомів програміст може створити свої об'єкти. Наприклад, цей об'єкт представляє коло:

```
[r] > circle
  mul 2 3.14 r > perimeter
  mul 3.14 r r > area
```

Перший рядок створює "абстрактний" об'єкт з ім'ям `circle`. Він абстрактний, бо один з його атрибутів (`r`) є "вільний". Він не визначений в цьому об'єкті, і тому об'єкт не можна використати в такому стані, його треба скомпілювати з визначеним `r`. Наприклад, це є коло `c` з радіусом 30:

```
circle 30 > c
```

Об'єкт `circle` має три атрибути. Перший — це `r`, він вільний. Інші два — `perimeter` і `area`. Вони "зв'язані", бо їхні атоми вже визначені: `mul` в обох випадках. Щоб отримати площу круга, обмеженого колом `c`, робимо так:

```
c.area > a
```

Виглядає, як виклик методу, але це не так. Ми не викликаємо методу, просто беремо об'єкт-площу (`area`) з об'єкту `c`. Він для нас не створюється в ту мить, коли ми робимо `c.area`! Він вже є: сидить і чекає, коли ми його заберемо. Він був створений тоді, коли був збудований об'єкт `c`.

Це і є різниця між методами в Java й атрибутами в EO. В Java кожен метод є процедурою, яка виконується, як тільки викликається. Цей механізм виклику методів (чи надсилання повідомлень, в термінах [ранніх адептів ООП](https://www.yegor256.com/2017/12/12/alan-kay-was-wrong.html)) був успадкований від функцій C, а їх ми успадкували від процедур мови ALGOL, наскільки я знаю. EO робить це по-іншому. Викликів методів нема. Ця мова просто бере атрибути від об'єктів і дає їх іншим об'єктам, поки їм не буде передано контроль, і це зійде на рівень атомів.

> **Ми маємо об'єкти, але не маємо методів. Є тільки атрибути, які представляють інші об'єкти.**

У вищеподаному прикладі об'єкт `a` не є порахованим числом. Це атом `mul`, який інкапсулює `3.14` і `30` (радіус). Результат обчислення ще не відомий. Якщо ми не схочемо нічого робити з `a`, процесор ніколи не виконає обчислення. Але якщо ми, приміром, вирішимо надрукувати число на консоль, тоді обчислення відбудеться:

```
stdout
  sprintf
    "Radius is %d, Area is %d"
    r
    a
```

Тут атом `sprintf` будує рядок, який інкапсулює три атрибути: сам текст, `r` і `a`. До речі, для побудови об'єктів можна вжити вертикальну або горизонтальну нотацію. Той самий код можна записати так:

```
stdout (sprintf "Radius is %d, Area is %d" r a)
```

Атом `stdout` інкапсулює рядок, побудований через `sprintf`, і — сидить тихо. Він нічого не друкує! Аж коли хтось колись "зачіпає" цей об'єкт, витягаючи один з його атрибутів, тоді атом `stdout` видасть рядок на консоль.

Атоми `stdout`, `sprintf`, `mul` і більшість інших не мають жодних атрибутів, крім одного: 𝜑. Кожен об'єкт і атом має цей особливий атрибут, відомий також як "тіло" об'єкту. Якщо зачепити `stdout.𝜑`, консоль виведе рядок.

Таким чином, ми маємо об'єкти, але не маємо методів. Є тільки атрибути, які представляють інші об'єкти.

*(Будь ласка, [підсвічуйте синтаксис](https://help.disqus.com/commenting/what-html-tags-are-allowed-within-comments) у своїх коментарях, щоб їх було легше читати.)*
