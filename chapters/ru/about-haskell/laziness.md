----
title: Лень
prevChapter: /ru/about-haskell/immutability.html
nextChapter: /ru/about-haskell/where.html
----

Язык Haskell - ленивый, поэтому он не делает работу, результат которой никому не нужен.

## Начнём с C++

Допустим, нам нужен список из некоторого числа одинаковых IP-адресов. Да, в реальной жизни нам такое едва ли понадобится, но этот пример хорошо покажет нам суть ленивых вычислений.

Функция, возвращающая список адресов, на C++ может выглядеть так:
```cpp
typedef std::vector<std::string> IPAddresses;

IPAddresses generate_addresses( size_t howMany,
                                const std::string& address ) {
    const IPAddresses addresses( howMany, address );
    return addresses;
}
```
Теперь нам понадобилась функция, получающая заданное количество адресов из этого списка и выводящая их на экран. Например:
```cpp
void take_and_print( size_t howMany,
                     const IPAddresses& addresses ) {
    for( size_t i = 0; i < howMany; ++i ) {
        std::cout << addresses[i] << std::endl;
    }
} 

int main() {
    take_and_print( 2, generate_addresses( 100, "127.0.0.1" ) );
}
```
Функция `take_and_print` получает список, возвращённый функцией `generate_addresses`, а потом печатает первые два адреса из этого списка.

Вывод будет таким:
```bash
127.0.0.1
127.0.0.1
```
И всё бы хорошо, но из 100 созданных строк фактически потребовались лишь первые две.  Оставшиеся 98 строк были созданы абсолютно напрасно. Было затрачено время, была затрачена память - и всё впустую.

Это - следствие строгости вычислений, присущей языку C++. Функция `generate_addresses` прямолинейна и сразу рвётся в бой. Сказали ей создать 100 адресов - получите 100. Скажут создать миллион - пожалуйста, вот вам миллион. Скажут миллиард - ну что ж, потерпите чуток, но будет вам и миллиард.

Тем временем функция `take_and_print` столь же прямолинейна, и ей абсолютно наплевать на усилия трудолюбивой функции `generate_addresses`. Если ей сказали отобразить лишь первые два элемента полученного контейнера, именно это она и сделает. И ей без разницы, сколько там ещё осталось элементов, десять или полмиллиарда.

Результатом строгости вычислений является лишняя работа. Но функции в Haskell, в отличие от своих трудолюбивых коллег из C++, терпеть не могут лишней работы.

## А вот как в Haskell

Откроем файл `Main.hs` и перепишем его:
```haskell
main = print (take 2 (replicate 100 "127.0.0.1"))
```
Функция `replicate` создаёт список из 100 адресов вида `127.0.0.1`, а функция `take` берёт 2 первых адреса из этого списка (о чём свидетельствует число 2, переданное ей в качестве первого аргумента). Функция `print` приводит это хозяйство к строковому виду и выводит на экран. Не обращайте внимания на синтаксические непонятности этого кода. В последующих главах они будут разъяснены в высшей степени подробно.

Весь фокус в том, что функция `replicate` создаёт список вовсе не из 100 адресов, а всего из двух. Почему? Потому что именно столько понадобилось функции `take`.

Функция `replicate` - лентяйка. Несмотря на то, что мы попросили её создать список из 100 строк, она смотрит по сторонам и думает: "Так-с, кому тут нужны мои строки? Ага, функции `take` нужны. И сколько же ей нужно? А-а, всего две. Ну так а чего я, глупая что ли, создавать сто строк, когда требуется всего две?! Вот тебе две строки и будь счастлива!"

Да, трудолюбие - это хорошо, а лень - это плохо, однако в данном случае мне более симпатична функция-лентяйка. Она, как хороший рационализатор, делает не столько, сколько её попросили, а столько, сколько реально нужно. В этом и заключается суть ленивых вычислений в Haskell.

Разумеется, если аппетиты функции `take` возрастут и она попросит первые пятьдесят элементов вместо первых двух, то функция `replicate` создаст список уже из 50 строк. Столько, сколько нужно, и ни капли больше.

Да, но откуда мы можем знать, что функция `replicate`  создаёт лишь столько IP-адресов, сколько потребовалось? А вдруг это не так? Давайте проверим.

Ленивость языка Haskell позволяет нам оперировать бесконечно большими списками. Нет, не просто очень большими, но именно бесконечными. Перепишем наш пример следующим образом:
```haskell
main = print (take 2 (repeat "127.0.0.1"))
```
Функция `repeat` создаст бесконечно большой список IP-адресов, элементами которого будет переданный ей адрес `127.0.0.1`. И вот если бы наша трудолюбивая функция `generate_addresses` из C++ захотела стать похожей на свою ленивую коллегу, ей пришлось бы стать примерно такой:
```cpp
IPAddresses generate_addresses( size_t howMany,
                                const std::string& address ) {
    IPAddresses addresses;
    for(;;) {
        addresses.push_back( address );
    }

    return addresses;
}
```
И всё бы хорошо, но это намертво зависнет. И причиной тому служит уже известное нам трудолюбие функции `generate_addresses`. Сказали ей создать бесконечно большой список - будет создавать до последнего вздоха, ведь цикл `for` в данном случае не имеет выхода.

Однако если мы соберём наш Haskell-проект и запустим его, то не будет никакого зависания, и на экран вновь выведутся уже знакомые нам два адреса. А всё потому, что функция `repeat` столь же ленива и рациональна, как и её коллега `replicate`. Да, мы попросили её создать бесконечно большой список, однако на деле она создаст список вовсе не бесконечно большой, а настолько большой, насколько потребуется. И если в данном случае потребовался список только из двух строк - получите список из двух строк. Конечно, если бы потребовался список из миллиона строк - извольте, будет вам миллион.

Вот суть ленивых вычислений в Haskell: не важно, сколько приказали сделать, ведь в конечном итоге будет сделано ровно столько, сколько реально понадобится.
