# SearchServer
Поисковая система, учитывающая минус-слова, стоп-слова. Реализована однопоточная и многопоточная версии поиска документов.  Ранжирование происходит по TF-IDF.

В данный момент в **main** показан пример использования данной программы. (Подробнее ниже)

## Описание

Особенности данной программы: 

* Можно задать количество выводимых топ-документов. Определяется глобальной константой **MAX_RESULT_DOCUMENT_COUNT**
* Обработка стоп-слов (документы, содержащие стоп-слова не будут учитываться при запросе).
* Обработка минус-слов: документы, включащие такие слова, будут исключены из результата
  **Примечание:**   
  *Если нет плюс-слов, то вывод отсутсвует.*  
  *Одно и то же слово, помеченное плюсом и минусом, в итоге будет считься минус-словом.*
* Учёт рейтинга документов осуществляется (опционально) с помощью функции-предиката.
* Учёт статуса документа осуществляется (опционально) с помощью функции-предиката.
* Класс **Paginator** позволяет поисковой системе разбивать результаты на страницы.
* Присутствует механизм исключений.
* Присутствует функция нахождения и удаления дубликатов-документов, у которых наборы встречающихся слов совпадают; стоп-слова игнорируются.  
 >Удаляются при этом документы с бóльшим id.  
* Реализована многопоточная версия поиска документа в дополнении к однопоточной.  
 >Состояние гонки исключено. 
* Ранжирование по TF-IDF:
  Полезность слов оценивают понятием *inverse document frequency* или *IDF*. Эта величина — свойство слова, а не документа.   Чем в большем количестве документов есть слово, тем ниже IDF.  
Второй способ улучшить ранжирование — выше располагать документы, где искомое слово встречается более одного раза. Здесь нужно рассчитать *term frequency* или *TF*. Для конкретного слова и конкретного документа это доля, которую данное слово занимает среди всех.    
При одинаковой релевантности с точностью до 10^-6, сортировка документа происходит по убыванию рейтинга.  
Расчёт релевантности:  
вычисляется IDF каждого слова в запросе,  
вычисляется TF каждого слова запроса в документе,  
IDF каждого слова запроса умножается на TF этого слова в этом документе,  
все произведения IDF и TF в документе суммируются.  
 
 ## Инструкция по использованию
 Измените **main** в соответствии с Вашими входными данными.  
 1. Через конструктор **SearchServer** получает стоп-слова;  
 2. С помощью метода **AddDocument** добавляются: id, текст документа, его статус (опционально), его рейтинг (опционально);  
 3. Методом **FindTopDocuments** этого класса производится поиск документа по запросу;  
На вход подаются: политика параллельного выполнения(опционально), запрос, статус документа (опционально), функция предикат (опционально);  
4. Вывод документа через функцию вывода. 
 ```C++
 #include "process_queries.h"
#include "search_server.h"

#include <execution>
#include <iostream>
#include <string>
#include <vector>

using namespace std;

void PrintDocument(const Document& document) {
    cout << "{ "s
         << "document_id = "s << document.id << ", "s
         << "relevance = "s << document.relevance << ", "s
         << "rating = "s << document.rating << " }"s << endl;
}

int main() {
    SearchServer search_server("and with"s);

    int id = 0;
    for (
        const string& text : {
            "white cat and yellow hat"s,
            "curly cat curly tail"s,
            "nasty dog with big eyes"s,
            "nasty pigeon john"s,
        }
    ) {
        search_server.AddDocument(++id, text, DocumentStatus::ACTUAL, {1, 2});
    }


    cout << "ACTUAL by default:"s << endl;
    // последовательная версия
    for (const Document& document : search_server.FindTopDocuments("curly nasty cat"s)) {
        PrintDocument(document);
    }
    cout << "BANNED:"s << endl;
    // последовательная версия
    for (const Document& document : search_server.FindTopDocuments(execution::seq, "curly nasty cat"s, DocumentStatus::BANNED)) {
        PrintDocument(document);
    }

    cout << "Even ids:"s << endl;
    // параллельная версия
    for (const Document& document : search_server.FindTopDocuments(execution::par, "curly nasty cat"s, [](int document_id, DocumentStatus status, int rating) { return document_id % 2 == 0; })) {
        PrintDocument(document);
    }

    return 0;
}
 ```
 Вывод: 
 ```
ACTUAL by default:
{ document_id = 2, relevance = 0.866434, rating = 1 }
{ document_id = 4, relevance = 0.231049, rating = 1 }
{ document_id = 1, relevance = 0.173287, rating = 1 }
{ document_id = 3, relevance = 0.173287, rating = 1 }
BANNED:
Even ids:
{ document_id = 2, relevance = 0.866434, rating = 1 }
{ document_id = 4, relevance = 0.231049, rating = 1 }
```

## Системные требования
* C++17(C++1z)  

## Планы по доработке
Таковые на данный момент не имеются.

![ezgif com-gif-maker](https://user-images.githubusercontent.com/82444909/168443545-5615a86f-d794-44a6-86c1-8f21b80faa39.gif)
