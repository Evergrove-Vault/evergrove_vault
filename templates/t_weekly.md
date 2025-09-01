# Дни недели
```dataviewjs
const weekFile = dv.current().file.name; // например "2025-W34"
const [year, week] = weekFile.split("-W").map(x => Number(x));

// Функция для получения даты понедельника по ISO-неделе
function getDateOfISOWeek(w, y) {
    const simple = new Date(y, 0, 1 + (w - 1) * 7);
    const dow = simple.getDay();
    const ISOweekStart = simple;
    if (dow <= 4)
        ISOweekStart.setDate(simple.getDate() - simple.getDay() + 1);
    else
        ISOweekStart.setDate(simple.getDate() + 8 - simple.getDay());
    return ISOweekStart;
}

const startDate = getDateOfISOWeek(week, year); 
const endDate = new Date(startDate);
endDate.setDate(startDate.getDate() + 7);

// Список заметок за неделю
const pages = dv.pages('"day_notes/daily"')
    .where(p => {
        const d = new Date(p.file.name); // ожидается формат YYYY-MM-DD
        return d >= startDate && d <= endDate;
    })
    .sort(p => p.file.name, 'asc');

// Вывод
dv.list(pages.map(p => dv.fileLink(p.file.path)));
```
# Сон 
```dataviewjs
const weekFile = dv.current().file.name; // например "2025-W34"
const [year, week] = weekFile.split("-W").map(x => Number(x));

// Функция для получения даты понедельника по ISO-неделе
function getDateOfISOWeek(w, y) {
    const simple = new Date(y, 0, 1 + (w - 1) * 7);
    const dow = simple.getDay();
    const ISOweekStart = simple;
    if (dow <= 4)
        ISOweekStart.setDate(simple.getDate() - simple.getDay() + 1);
    else
        ISOweekStart.setDate(simple.getDate() + 8 - simple.getDay());
    return ISOweekStart;
}

const startDate = getDateOfISOWeek(week, year); 
const endDate = new Date(startDate);
endDate.setDate(startDate.getDate() + 7);

// Список заметок за неделю
const pages = dv.pages('"day_notes/daily"')
    .where(p => {
        const d = new Date(p.file.name); // ожидается формат YYYY-MM-DD
        return d >= startDate && d <= endDate;
    })
    .sort(p => p.file.name, 'asc');

// Функция для подсчёта минут из строки вида "5:50"
function parseDuration(str) {
    if (!str) return 0;
    const parts = str.split(":");
    return parseInt(parts[0])*60 + parseInt(parts[1]);
}

// Функция для форматирования минут обратно в "чч:мм"
function formatDuration(mins) {
    const h = Math.floor(mins / 60);
    const m = mins % 60;
    return `${h}:${m.toString().padStart(2, "0")}`;
}

// Собираем данные для таблицы
let tableData = [];
let totalMinutes = 0;

for (let p of pages) {
    const durationMins = parseDuration(p["sleep-duration"]);
    totalMinutes += durationMins;

    tableData.push([
        moment(p.file.name, "YYYY-MM-DD").format("dddd, DD.MM.YYYY"), // день недели + дата
        `${p["sleep-start"]} - ${p["sleep-end"]}`,
        formatDuration(durationMins)
    ]);
}

// Добавляем строку с общей суммой
tableData.push([
    "Итого за неделю",
    "",
    formatDuration(totalMinutes)
]);

dv.table(
    ["День", "Время сна", "Общее время сна"],
    tableData
);
```

```dataviewjs
const weekFile = dv.current().file.name; // например "2025-W34"
const [year, week] = weekFile.split("-W").map(x => Number(x));

// Функция для получения даты понедельника по ISO-неделе
function getDateOfISOWeek(w, y) {
    const simple = new Date(y, 0, 1 + (w - 1) * 7);
    const dow = simple.getDay();
    const ISOweekStart = simple;
    if (dow <= 4)
        ISOweekStart.setDate(simple.getDate() - simple.getDay() + 1);
    else
        ISOweekStart.setDate(simple.getDate() + 8 - simple.getDay());
    return ISOweekStart;
}

const startDate = getDateOfISOWeek(week, year); 
const endDate = new Date(startDate);
endDate.setDate(startDate.getDate() + 7);

// Список заметок за неделю
const pages = dv.pages('"day_notes/daily"')
    .where(p => {
        const d = new Date(p.file.name); // ожидается формат YYYY-MM-DD
        return d >= startDate && d <= endDate;
    })
    .sort(p => p.file.name, 'asc');
    
// --- Данные ---
const dates            = Array.from(pages).map(p => p.file.name);
const morningEnergyArr = Array.from(pages).map(p => Number(p["morning-energy"]) || 0);
const dayEnergyArr     = Array.from(pages).map(p => Number(p["day-energy"]) || 0);
const eveningEnergyArr = Array.from(pages).map(p => Number(p["evening-energy"]) || 0);

// --- Контейнер ---
const container = this.container;
const div = container.createEl("div");

// --- Свойства для графика (только энергии) ---
const metrics = [
    { name: "Энергия (утро)", data: morningEnergyArr, color: "orange" },
    { name: "Энергия (день)", data: dayEnergyArr, color: "purple" },
    { name: "Энергия (вечер)", data: eveningEnergyArr, color: "red" }
];

// --- Создаем trace для каждого свойства ---
let traces = metrics.map(metric => ({
    x: dates,
    y: metric.data,
    type: "scatter",
    mode: "lines+markers",
    name: metric.name,
    line: { color: metric.color, width: 2 },
    marker: { size: 4, opacity: 0.6 }
}));

// --- Настройка layout ---
const layout = {
    title: { text: "Динамика энергии за последнюю неделю", x: 0.5, y: 0.98},
    xaxis: { title: "Дата", rangeslider: { visible: true }, type: "date" },
    yaxis: { title: "Оценка", rangemode: "tozero" },
    legend: { orientation: "h", y: 0.95, x: 0, xanchor: "left" },
    margin: { t: 80, b: 100 }
};

// --- Построение графика ---
Plotly.newPlot(div, traces, layout, { responsive: true });
```

## Почему график такой?

# Над чем работал (что создал)

```dataviewjs
const weekFile = dv.current().file.name; // например "2025-W34"
const [year, week] = weekFile.split("-W").map(x => Number(x));

// Функция получения понедельника ISO-недели
function getDateOfISOWeek(w, y) {
    const simple = new Date(y, 0, 1 + (w - 1) * 7);
    const dow = simple.getDay();
    const ISOweekStart = simple;
    if (dow <= 4)
        ISOweekStart.setDate(simple.getDate() - simple.getDay() + 1);
    else
        ISOweekStart.setDate(simple.getDate() + 8 - simple.getDay());
    return ISOweekStart;
}

const startDate = getDateOfISOWeek(week, year);
const endDate = new Date(startDate);
endDate.setDate(startDate.getDate() + 6);

// Собираем заметки за неделю (кроме дневных)
const pages = dv.pages()
    .where(p => 
        p.file.cday >= startDate && 
        p.file.cday <= endDate &&
        !p.file.path.startsWith("day_notes/daily") // исключаем дневные
    )
    .sort(p => p.file.cday, 'asc');

// Группируем по дате создания вручную
const grouped = {};
for (let page of pages) {
    const day = page.file.cday.toFormat("yyyy-MM-dd");
    if (!grouped[day]) grouped[day] = [];
    grouped[day].push(page);
}

// Вывод всех дней недели (пн–вс)
for (let i = 0; i < 7; i++) {
    const day = new Date(startDate);
    day.setDate(startDate.getDate() + i);
    const dayStr = dv.luxon.DateTime.fromJSDate(day).toFormat("yyyy-MM-dd");

    dv.header(3, day.toLocaleDateString("ru-RU", { weekday: 'long', day: '2-digit', month: '2-digit' }));
    if (grouped[dayStr]) {
        dv.list(grouped[dayStr].map(p => p.file.link));
    } else {
        dv.paragraph("_нет заметок_");
    }
}
```

# 1️⃣ Основные выводы за неделю
## Ключевые достижения и события
## Новые инсайты, которые важно не потерять
## То, что получилось сделать особенно хорошо

# 2️⃣ Сравнение запланированного и сделанного
## Основной фокус
## Список задач:
- приоритеты
## Причины отклонений от плана

# 3️⃣ Личные инсайты и рост
## Новые навыки, знания или привычки
## Чему научился, что попробовал впервые
## Ошибки, которые стали полезным уроком
# 4️⃣ Проблемы и блоки
## Что тормозило выполнение планов
## Внутренние и внешние факторы
## Возможные решения на следующую неделю

# 5️⃣ Мотивация и самооценка
Оценка недели по шкале от 1 до 10:
week-mark::
## Что порадовало
## Что нужно улучшить в подходе или планировании

# 6️⃣ Интересные находки
## Статьи, книги, видео
## Новые инструменты или техники работы
## Идеи для проектов и экспериментов
# 7️⃣ Рефлексия
Общие размышления о недели 


