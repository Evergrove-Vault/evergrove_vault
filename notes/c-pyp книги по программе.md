---
course-id: c-pyp
link: https://drive.google.com/drive/folders/1cFiW56Q0jIwHmIrsw_M6N76xycy_WzRE?usp=sharing
---

# Что и зачем?
В рамках курса подразумевается большой объем разноплановых тем. Так как все и сразу усвоить практически нереально, да и мозг хранит только важные аспекты информации, то наличие первоисточников, в которых собрана более подробная информация, крайне полезно. Тут будут краткие заметки по тематикам книг и тому зачем их стоит прочитать в рамках программы. Более подробную информацию смотрите в самих заметках по каждой книге, к которым я буду прикладывать свои мысли по итогам прочтения.

Для удобства навигации тут будут ссылки на все MOC, которые связаны с книгой (их так же можно посмотреть при переходе в заметку о книге)

```dataviewjs
// получаем course-id из текущей страницы
let current = dv.current();
let courseId = current["course-id"];

if (!courseId) {
    dv.paragraph("❌ У текущей страницы нет course-id");
} else {
    // ищем книги с этим course-id
    let books = dv.pages()
        .where(p => p.file.tags && p.file.tags.includes("#book"))
        .where(p => p["course-id"] == courseId)
        .sort(p => p.file.name);  // сортировка по имени файла

    let table = [];

    for (let book of books) {
        // убираем префикс book_
        let displayName = book.file.name.replace(/^book_/, "");

        // имя страницы курса: "<course-id> книги по программе"
        let programPage = `${courseId} книги по программе`;

        // ссылка на книгу
        let bookLink = `[[${programPage}#${displayName}|${displayName}]]`;

        // собираем ссылки MOC (inlinks + outlinks)
        let mocPages = dv.pages("#MOC")
            .where(moc => 
                moc.file.inlinks.some(l => l.path == book.file.path) ||
                moc.file.outlinks.some(l => l.path == book.file.path)
            );

        // создаем кликабельные ссылки и сортируем по имени
        let mocLinks = mocPages.length > 0
            ? mocPages
                .sort(m => m.file.name)
                .map(m => `[[${m.file.path}|${m.file.name}]]`)
                .join(", ")
            : "❌ Нет MOC";

        table.push([bookLink, mocLinks]);
    }

    // вывод
    if (table.length > 0) {
        dv.table(["📖 Книга", "📑 MOC связанные"], table);
    } else {
        dv.paragraph(`❌ Нет книг с #book для course-id: ${courseId}`);
    }
}
```


# изучаем_spark_молниеносный_анализ_данных
[[book_изучаем_spark_молниеносный_анализ_данных|ссылка]]

![[book_изучаем_spark_молниеносный_анализ_данных#Краткое описание основных тем]]
---

**Рекомендованные главы для соответствующих модулей курса:**

# паттерны_разработки_на_python
[[book_паттерны_разработки_на_python|ссылка]]

![[book_паттерны_разработки_на_python#Краткое описание основных тем]]

---

**Рекомендованные главы для соответствующих модулей курса:**

# современные_паттерны_обработки_больших_данных_spark
[[book_современные_паттерны_обработки_больших_данных_spark|ссылка]]

![[book_современные_паттерны_обработки_больших_данных_spark#Краткое описание основных тем]]

---

**Рекомендованные главы для соответствующих модулей курса:**
# clear_architecture
[[book_clear_architecture|ссылка]]

![[book_clear_architecture#Краткое описание основных тем]]

---

**Рекомендованные главы для соответствующих модулей курса:**
# continuous_api_management
[[book_continuous_api_management|ссылка]]

![[book_continuous_api_management#Краткое описание основных тем]]

---

**Рекомендованные главы для соответствующих модулей курса:**
# fluent_python
[[book_fluent_python|ссылка]]

![[book_fluent_python#Краткое описание основных тем]]

---

**Рекомендованные главы для соответствующих модулей курса:**
# linux_command_line
[[book_linux_command_line|ссылка]]

![[book_linux_command_line#Краткое описание основных тем]]

---

**Рекомендованные главы для соответствующих модулей курса:**
# natural_language_processing_with_spark_NLP_learning
[[book_natural_language_processing_with_spark_NLP_learning|ссылка]]

![[book_natural_language_processing_with_spark_NLP_learning#Краткое описание основных тем]]

---

**Рекомендованные главы для соответствующих модулей курса:**
# nginx_cookbook
[[book_nginx_cookbook|ссылка]]

![[book_nginx_cookbook#Краткое описание основных тем]]

---

**Рекомендованные главы для соответствующих модулей курса:**
# python_OOP
[[book_python_OOP|ссылка]]

![[book_python_OOP#Краткое описание основных тем]]

---

**Рекомендованные главы для соответствующих модулей курса:**
# stream_processing_with_apache_spark
[[book_stream_processing_with_apache_spark|ссылка]]

![[book_stream_processing_with_apache_spark#Краткое описание основных тем]]

---

**Рекомендованные главы для соответствующих модулей курса:**
# UML_distilled
[[book_UML_distilled|ссылка]]

![[book_UML_distilled#Краткое описание основных тем]]

---

**Рекомендованные главы для соответствующих модулей курса:**