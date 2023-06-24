## Конструктори чи статичні фабричні методи?

(Оригінал цієї статті знаходиться [тут](https://www.yegor256.com/2017/11/14/static-factory-methods.html).)

14 листопада 2017 р. Одеса, Україна

Думаю, першим це сказав Джошуа Блох у своїй чудовій книжці [Effective Java](http://amzn.to/2zgpiRI): статичним фабричним методам треба віддавати перевагу перед конструкторами при інстанціюванні об'єктів. Я не згоден. Не тільки через моє переконання, що статичні методи є чисте зло, але більше тому, що в цьому частинному випадку вони прикидаються добрими і змушують нас думати, що ми їх маємо любити.

![Extract](/extract.jpg)
(Image :copyright: Extract (2009) by Mike Judge)

Проаналізуймо доводи і погляньмо, чому вони хибні з об'єктно-орієнтованої точки зору.

Ось клас із одним типовим (primary) і двома додатковими конструкторами:

``` Java
class Color {
  private final int hex;
  Color(String rgb) {
    this(Integer.parseInt(rgb, 16));
  }
  Color(int red, int green, int blue) {
    this(red << 16 + green << 8 + blue);
  }
  Color(int h) {
    this.hex = h;
  }
}
```

А це подібний клас із трьома фабричними методами:

``` Java
class Color {
  private final int hex;
  static Color makeFromRGB(String rgb) {
    return new Color(Integer.parseInt(rgb, 16));
  }
  static Color makeFromPalette(int red, int green, int blue) {
    return new Color(red << 16 + green << 8 + blue);
  }
  static Color makeFromHex(int h) {
    return new Color(h);
  }
  private Color(int h) {
    this.hex = h;
  }
}
```

Як вважаєте, який ліпший?

Джошуа Блох наводить три головні переваги використання статичних фабричних методів замість конструкторів (взагалі-то чотири, але четверта [більше не стосується](https://docs.oracle.com/javase/7/docs/technotes/guides/language/type-inference-generic-instance-creation.html) Java):
- Вони мають імена.
- Вони мають кеш.
- Вони мають підтип.

Всі ці три переваги є дуже логічними ... якщо дизайн неправильний. Вони підходять для виправдання кривих шляхів. Перегляньмо їх одну за одною.

> **Народження об'єкта конструктором є "священною" подією в будь-якому об'єктно-орієнтованому програмному забезпеченні, не нехтуйте цією красою!**

### Вони мають імена

Ось як ви створюєте об'єкт [томатного](http://www.rapidtables.com/web/color/red-color.htm) кольору конструктором:

``` Java
Color tomato = new Color(255, 99, 71);
```

А так ви це робите статичним фабричним методом:

``` Java
Color tomato = Color.makeFromPalette(255, 99, 71);
```

Здається, `makeFromPalette()` — це семантично багатше, ніж просто `new Color()`, правда? Хто його знає, що це за три числа, передані конструктору. А от слово "palette" ("палітра") допомагає все уявити негайно.

Це правда.

І все-таки правильно буде застосувати поліморфізм і інкапсуляцію, щоб розбити проблему на кілька семантично багатих класів:

``` Java
interface Color {
}
class HexColor implements Color {
  private final int hex;
  HexColor(int h) {
    this.hex = h;
  }
}
class RGBColor implements Color {
  private final Color origin;
  RGBColor(int red, int green, int blue) {
    this.origin = new HexColor(
      red << 16 + green << 8 + blue
    );
  }
}
```

А тепер використовуємо правильний конструктор правильного класу:

``` Java
Color tomato = new RGBColor(255, 99, 71);
```

Бачиш, Джошуа?

### Вони мають кеш


### Вони мають підтип


