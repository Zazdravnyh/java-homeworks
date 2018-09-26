# Домашнее задание к лекции 1.4	
## Типы данных в Java: объекты

## Задача № 1
И мы продолжаем улучшать наш task manager для нашей любмой комнады!
Источником данных для приложения может быть не только какой-то безопасный способ ввода, например такой как - скопировать текст и вставить (часто чтобы не допустить ошибку люди копируют номера кредитных карт из чатов с друзьями), а ввод текста руками, а иногда разбор аудиотекста или картинки.
В нашей задаче нам нужно научится распозновать введный текст таким образом чтобы мы могли его использовать в своей программе, не воспринимать как ошибку (этот способ похож на тот что используют в голосовых помощниках как Алиса и Google Assistant, там тоже есть шаблоны введеных сообщений, которые программно разбираются).
А теперь сама задача: "Менеджер Иван заводит задачу программисту и вводит следующий текст: Добавить при открытии главного экрана приложения сообщение: Здравствуйте, %username%! Задача начинается в 15:00 и заканчивается в 19:30. Нам нужно найти в тексте который ввел Иван время после слов "начинается в" и "заканчивается в", найти разницу между началом и завершением, перевести её в секуднды и вывести на экран, только вот если результат получится отрицательным нужно выбросить подходящий Exception.     

### Процесс реализации

Давайте логически разделим нашу задачу (декомпозируем) на получение данных и их обработку. Получение мы оставим в общем потоке в методе main, а для их обработки создадим отедльный метод (пока еще можно называть функцие, но напомню в java все функции принадлежат какому-то классу, а у класса есть только поля и методы) назовем его - parseDate.

Этап получения данных:
1. Объявним в методе main ссылку на новый объект Scanner scanner = new Scanner(System.in) (где System.in - это стандартный поток ввода в нашем случае console)
2. Для того чтобы прочитать значения нужно вызвать у объекта scanner.next(), сохраним его в переменную которую назовем input получим следующее:
```
    public static void main(String[] args) throws Exception {
        Scanner scanner = new Scanner(System.in);
        String input = scanner.next();
        //TODO
    }
```

Этап обработки данных:
1. Добавим новый метод в наш класс который как договорились раньше назовем `parseTime`. Метод будет принимать на вход что мы хотим найти и где (что-то вроде шаблона и самой строки где будем искать), а возвращать найденное время
сигнатура будет получится следующая: LocalTime parseTime(String pattern, String input)
2. Самый простой способ поиска строки в подстроке это метод объекта String substring и похожие, они довольно просты. Мы воспользуемся поиском через Regex и Matcher.
```
    static LocalTime parseTime(String pattern, String input) throws Exception {

        //Это общая часть regex для входных шаблонов (patterns) в которой мы ищем цифры
        String regex = pattern + " \\d+:\\d+";
        //Объект проверяет подходит ли входная строка под искомый шаблон
        Matcher inputStringMatcher = Pattern.compile(regex).matcher(input);
        //Проверим что мы нашли совпадение
        if (inputStringMatcher.find()) {
            //Сохраним найденный шаблон в переменную
            String patternWithTime = inputStringMatcher.group();
            //Выделим только время
            Matcher timeMatcher = Pattern.compile("\\d+:\\d+").matcher(patternWithTime);
            //Найдем совпадение
            if (timeMatcher.find()) {
                //Метод split разделит строку 10:20 и вернет массив строк уже раздельно 
                // где 0й-элемент = 10, а 1й-элемент = 20 
                String[] timeArray = timeMatcher.group().split(":");
                //Переведем строку хранящую часы в числовое представление integer и сохраним в переменную
                int hours = Integer.parseInt(timeArray[0]);
                //Переведем строку хранящую минуты в числовое представление integer и сохраним в переменную
                int minutes = Integer.parseInt(timeArray[1]);
                //Создадим общий объект LocalTime - который не привязан к часовому поясу. 
                return LocalTime.of(hours, minutes);
            }
        }
        //В случае если мы не смогли разобрать входную строку вернем ошибку и прервем выполнение программы
        throw new Exception("Error input string" + input);
    }
``` 
3. Нужно вызвать метод `parseTime` передать в него строку шаблон (в поле pattern) например: "начинается в" и соханить в локальную перменную которую можно назвать `startTime`
4. Точно как в предыдущем шаге нужно вызвать parseTime и передать в поле pattern - "заканчивается в" и сохранить в другую локальную переменную `endTime`.
5. Почти закончили. Нужно вычесть из `endTime` пермененную `startTime` сделать это можно следующим образом:
`endTime.minusSeconds(startTime.toSecondOfDay()).toSecondOfDay()` и сохранить в перменную `taskTime' типа `int` потому что результат вычитания будут секунды
6. Выведем полученный результат в консоль, `System.out.println(""+taskTime);`
7. Done! 

## Задача № 2

Продолжаем искать проблемы в пользовательском вводе данных, лишние пробелы давно не дают покоя программистам, они касаются всех и разработчиков для Android, Backend и даже программистов занимающихся BigData/ML. Нам нужно научится удалять в любом тексте лишние пробелы (там где их больше чем один, за исключением начала и конца текста, где их вообще не дожно быть)
Наша задача: написать функцию (метод) котрая будет принимать на вход строку введеную пользователем и возвращать обработанную строку без лишних пробелов. Результат выполнения нужно вывести в консоль.

### Процесс реализации
Для того чтобы удалить лишние пробелы в языке Java у строк есть метод `trim` котрый удаляет пробелы в начале и в конце строки, но что же нам делать если мы встретим лишние пробелы внутри, есть как всегда несколько способов, например сделать разделить весть текст на слова удалить пробелы и соединиться обратно, но можно сделать более "элегантно" используся метод объекта строк replaceAll который найдет и перезапишет все пробелы на один.
1. Создаем объект Scanner scanner = new Scanner(System.in) в методе main и сохраняем введеные даннче (метод next) в перменную input
2. Составляем regex для поиска всех пробелов `s` добавляем к нему `+` (что значит неограниченное множество), получаем `s+` (можно тут ознакомиться https://tproger.ru/articles/regexp-for-beginners/)
3. Выполняем у строки `input.trim()` так мы удалим не нужные пробелы в начале строки и в конце, сохраняем в перменную inputTrimmed
4. Теперь выполняем `inputTrimmed.replaceAll` первым параметром передаем наш regex, а вторым говорим на что заменить = `" "` (пробел) сохраняем полученную строку в перменную result
5. Выводим значение перменной result на экран!
6. Готово.
7. Дополниельно можно вынести такую обработку строк в отдельный `static` метод и назвать extraTrim. 

## Задача № 3

Поговорим о математике, чем старше мы становились тем сложней формулы нам приходилось и приходится до сих пор использоваться, но как часто вбивая на калькуляторе в очередной раз формулу вы забывали закрыть или открыть скобку, я подозреваю что никогда. Но тем не менее наша следующая задача порой встречается на собеседованиях и только по этому на стоит её решить. Звучит она так: На вход методу приходит текст, нужно разобрать это текст и проверть все ли открытые скобки в нем закрыты, нет ли такого что какая-нибудь из закрытых или открытых скобок находится не на своем месте (например 5*)(3+5())), если это не так нужно вернуть false, иначе true.

### Процесс реализации

Задача на первый взгляд не тривиальна, давайте разбираться. С начала нужно продумать алгоритм прежде чем приступать к реализации, что я вам советую сделать это попробвать решить задачу на бумаге. Решить можно следующим образом: считываем каждый символ и если это открывающая скобка увеличиваем спецально отведнную для этого перменную назовем ее счетчик, если закрывающая уменьшаем. Формула будет верная если после обхода всей строки наш счетчик будет равен нулю.

1. Считваем строку из консоли и сохарняем в переменную input
2. Создадим метод `isFormulaValid` (методы возвращающие тип boolean принято начинать не get, а с is, как бы задавая вопрос), возвразать он будет true или false, а входным парамтером будет строка (формулу котрую ввели)
3. В методе создадим перменную (счетчик) типа int назовем ее `counter`, он будет отвечать за подсчет открытых и закрытых скобок, так же явно укажем что она равна нулю. (`int counter = 0`)
4. Нам нужно перебрать всю строку, нам известно как взять длину строки input.length - метод вернет нам количество символов, на основе этих данных пишем цикл посимвольного обхода строки:
```
  for (int i = 0; i < input.length; i++) {
    //TODO
  }
```
5. Чтобы взять символ строки, нужно обратить к строке, вызвать метод `charAt` котрый принимает на вход `index` символа (напомню что индексы в строках и массивах начинаются с 0)
6. Вмето `//TODO` добавляем 
```
  char c = input.charAt(i);
  if (c == '(') {
      //если нашли открывающу скобку увеличиваем счетик
      counter++;
  } else {
      //если нашли закрывающую скобку уменьшаем счетик
      counter--;
  }
``` 
7. Результат выполнения нашего метода `isFormulaValid` будет сравнение counter c нулем. Можно записать как `return counter == 0`
8. Осталось проверить и вызвать наш метод в методе main
9. Результат выполенения можно вывести через `System.out.println(checkFormula(input));`

---

## Инструкция по выполнению домашнего задания:

## Инструкция по выполнению домашнего задания:

1. Зарегистрируйтесь на сайте [Repl.IT](https://repl.it/).
2. Перейдите в раздел **my repls**.
3. Нажмите кнопку **Start coding now!**, если приступаете впервые, или **New Repl**, если у вас уже есть работы.
4. В списке языков выберите `Java`.
5. Код пишите в левой части окна.
6. Посмотреть результат выполнения файла можно, нажав на кнопку **Run**. Результат появится в правой части окна.
7. После окончания работы нажмите кнопку **Share** и скопируйте ссылку из поля _Share link_.
8. В личном кабинете на сайте [netology.ru](http://netology.ru/) в поле комментария к домашней работе вставьте скопированную ссылку и отправьте работу на проверку.

_Никаких файлов прикреплять не нужно._