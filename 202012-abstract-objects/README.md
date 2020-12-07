## Абстрактні об'єкти

(Оригінал цієї статті знаходиться [тут](https://www.yegor256.com/2020/12/01/abstract-objects.html).)

1 грудня 2020 р. Москва, Росія.

Автор: [Єгор Бугаєнко](https://www.yegor256.com).

Як ви створюєте об'єкти у своїй об'єктно-орієнтованій мові? Можемо взяти звичні C++, Java або C#. Спочатку ви визначаєте клас, а тоді створюєте його примірник. Перший крок відомий як [абстракція](https://en.wikipedia.org/wiki/Abstraction_%28computer_science%29), другий — як [інстанціація](https://en.wikipedia.org/wiki/Instance_%28computer_science%29#Object_oriented_programming). Подібна пара дій є і в функціональному програмуванні: визначення функції є абстракцією, а виклик її з певними аргументами — застосуванням, [аплікацією](https://en.wikipedia.org/wiki/Apply). Тоді [таке запитання](https://www.yegor256.com/2016/09/20/oop-without-classes.html): чому ООП потребує класів *і* об'єктів, а ФП обходиться самими тільки функціями?

![The Irishman](/the-irishman.jpg)
(Image :copyright: The Irishman (2019) by Martin Scorsese)

Таким є *абстрактний* об'єкт в [EO](https://www.eolang.org/):

```
[id db] > book
  db.query > title
    "SELECT title FROM book WHERE id=?"
    id
```

Ім'я об'єкту — `book`.

Атоми `stdout`, `sprintf`, `mul` і більшість інших не мають жодних атрибутів, крім одного: 𝜑. Кожен об'єкт і атом має цей особливий атрибут, відомий також як "тіло" об'єкту. Якщо зачепити `stdout.𝜑`, консоль виведе рядок.

*(Будь ласка, [підсвічуйте синтаксис](https://help.disqus.com/commenting/what-html-tags-are-allowed-within-comments) у своїх коментарях, щоб їх було легше читати.)*
