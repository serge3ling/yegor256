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

Джошуа Блох наводить три головні переваги використання статичних фабричних методів замість конструкторів (взагалі-то чотири, але четверта [більше не стосується](https://docs.oracle.com/javase/7/docs/technotes/guides/language/type-inference-generic-instance-creation.html) мови Java):
- Вони мають імена.
- Вони мають кеш.
- Вони мають підтип.

Всі ці три переваги є дуже логічними ... якщо дизайн неправильний. Вони підходять для виправдання кривих шляхів. Перегляньмо їх одну за одною.

> **Народження об'єкта конструктором є найбільш "священною" подією в будь-якому об'єктно-орієнтованому програмному забезпеченні, не нехтуйте цією красою!**

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

### Їх можна кешувати

Нехай в мене використовується томатний колір в різних місцях мого додатку:

```Java
Color tomato = new Color(255, 99, 71);
// ... десь пізніше
Color red = new Color(255, 99, 71);
```

Будуть створені два об'єкти; очевидно, це неефективно, бо вони ідентичні. Ліпше зберегти перший десь у пам'яті і повернути його на другий виклик. Статичні методи дають можливість це вирішити:

``` Java
Color tomato = Color.makeFromPalette(255, 99, 71);
// ... десь пізніше
Color red = Color.makeFromPalette(255, 99, 71);
```

Тоді ми десь в класі `Color` зберігаємо приватну статичну `Map` зі всіма вже інстанційованими об'єктами:

``` Java
class Color {
  private static final Map<Integer, Color> CACHE =
    new HashMap<>();
  private final int hex;
  static Color makeFromPalette(int red, int green, int blue) {
    final int hex = red << 16 + green << 8 + blue;
    return Color.CACHE.computeIfAbsent(
      hex, h -> new Color(h)
    );
  }
  private Color(int h) {
    return new Color(h);
  }
}
```

Для швидкодії це — дуже ефективно. З малим об'єктом, як наш `Color`, проблема не дуже очевидна, але коли об'єкти більші, їх інстанціація і збирання сміття забере багато часу.

Це правда.

І все-таки є об'єктно-орієнтований спосіб розв'язати цю проблему. Просто вводимо новий клас `Palette`, який стає сховищем кольорів:

``` Java
class Palette {
  private final Map<Integer, Color> colors =
    new HashMap<>();
  Color take(int red, int green, int blue) {
    final int hex = red << 16 + green << 8 + blue;
    return this.computeIfAbsent(
      hex, h -> new Color(h)
    );
  }
}
```

Тепер ми один раз створюємо один примірник `Palette` і просимо його повертати нам колір кожного разу, коли нам треба:

``` Java
Color tomato = palette.take(255, 99, 71);
// Пізніше отримаємо той же примірник:
Color red = palette.take(255, 99, 71);
```

Бачиш, Джошуа, без статичних методів, без статичних атрибутів.

### Вони мають підтип

Нехай наш клас `Color` має метод `lighter()`, який повинен повернути перший з доступних світліших кольорів:

``` Java
class Color {
  protected final int hex;
  Color(int h) {
    this.hex = h;
  }
  public Color lighter() {
    return new Color(hex + 0x111);
  }
}
```

Але деколи бажано вибрати наступний світліший колір з множини стандарту [Pantone](https://en.wikipedia.org/wiki/Pantone):

``` Java
class PantoneColor extends Color {
  private final PantoneName pantone;
  PantoneColor(String name) {
    this(new PantoneName(name));
  }
  PantoneColor(PantoneName name) {
    this.pantone = name;
  }
  @Override
  public Color lighter() {
    return new PantoneColor(this.pantone.up());
  }
}
```

Тоді ми створюємо статичний фабричний метод, який буде вирішувати, яка імплементація нам потрібна:

``` Java
class Color {
  private final String code;
  static Color make(int h) {
    if (h == 0xBF1932) {
      return new PantoneColor("19-1664 TPX");
    }
    return new RGBColor(h);
  }
}
```

Якщо потрібен [справжній червоний](https://www.pantone.com/color-finder/19-1664-TPX) колір, повертаємо примірник `PantoneColor`. У всіх інших випадках це буде просто звичайний `RGBColor`. Рішення приймає статичний фабричний метод. Викликаємо його так:

``` Java
Color color = Color.make(0xBF1932);
```

Це "розгалуження" неможливо буде зробити конструктором, бо він може повернути тільки примірник класу, в якому оголошений. А статичний метод має повну свободу, щоб повертати будь-який підтип класу `Color`.

Це правда.

І все-таки в об'єктно-орієнтованому світі ми можемо і мусимо робити це все по-іншому. Спочатку `Color` робимо інтерфейсом:

``` Java
interface Color {
  Color lighter();
}
```

Далі переносимо прийняття рішення в окремий клас `Colors`, як і в попередньому прикладі:

``` Java
class Colors {
  Color make(int h) {
    if (h == 0xBF1932) {
      return new PantoneColor("19-1664-TPX");
    }
    return new RGBColor(h);
  }
}
```

І використовуємо примірник класу `Colors` замість статичного фабричного методу всередині класу `Color`:

``` Java
colors.make(0xBF1932);
```

Але і це ще не зовсім об'єктно-орієнтоване мислення, бо ми забрали прийняття рішення з об'єкту, якому воно належить. Або статичним фабричним методом `make()` або новим класом `Colors` — неважливо як — ми розриваємо наші об'єкти на дві частини. Перша — сам об'єкт, друга — алгоритм прийняття рішення, який знаходиться десь зовні.

([Кілька думок щодо конструкторів в ООП (вебінар #7); 7 жовтня 2015 р.](https://www.youtube.com/watch?v=9yjtsCK6Wdk))

Набагато кращим об'єктно-орієнтованим дизайном буде помістити логіку в об'єкт класу `PantoneColor`, а він буде декоратором оригінального `RGBColor`:

``` Java
class PantoneColor {
  private final Color origin;
  PantoneColor(Color color) {
    this.origin = color;
  }
  @Override
  public Color lighter() {
    final Color next;
    if (this.origin.hex() == 0xBF1932) {
      next = new RGBColor(0xD12631);
    } else {
      next = this.origin.lighter();
    }
    return new PantoneColor(next);
  }
}
```

Тоді створюємо примірник `RGBColor` і декоруємо його примірником `PantoneColor`:

``` Java
Color red = new PantoneColor(
  new RGBColor(0xBF1932)
);
```

Ми просимо `red` повернути світліший колір, і він повертає світліший з палітри Pantone, а не просто світліший за мірками RGB:

``` Java
Color lighter = red.lighter(); // 0xD12631
```

Звичайно, це простенький приклад, і його треба [ще покращувати](https://www.yegor256.com/2016/12/20/can-objects-be-friends.html), якщо ми справді хочемо застосовувати його до всіх кольорів Pantone, але ви вже, мабуть, зловили думку. Логіці місце *всередині* класу, не зовні де-небудь, не в статичних фабричних методах або навіть в іншому допоміжному класі. Я говорю про логіку, яка належить до цього конкретного класу, звичайно. Якщо це якось стосується керування примірниками класу, то можна впровадити контейнери і сховища, як у вищенаведеному прикладі.

Підсумуємо: я б настійно радив *ніколи* не використовувати статичні методи, особливо якщо вони покликані замінити конструктори об'єктів. Народження об'єкта конструктором є [найбільш](https://www.yegor256.com/2014/10/03/di-containers-are-evil.html) "священною" подією в будь-якому об'єктно-орієнтованому програмному забезпеченні, не нехтуйте цією красою.
