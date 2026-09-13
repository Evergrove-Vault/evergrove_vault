---
week: <% moment().format("GGGG-[W]WW") %>
week-start: <% moment().startOf("isoWeek").format("YYYY-MM-DD") %>
week-end: <% moment().endOf("isoWeek").format("YYYY-MM-DD") %>
created: <% tp.date.now("YYYY-MM-DD HH:mm") %>
week-mark:
type: weekly
tags:
  - weekly-review
---
<%* await tp.file.rename(moment().format("GGGG-[W]WW")); -%>

# Обзор недели <% moment().format("GGGG-[W]WW") %>

## Дни недели

```dataviewjs
(() => {
const current = dv.current();
const startDate = dv.date(current["week-start"]);
const endDate = dv.date(current["week-end"]);

if (!startDate || !endDate) {
    dv.paragraph("⚠️ Не заполнены поля `week-start` и `week-end`.");
    return;
}

const dailySource = '"day_notes/daily" or "day_notes/archive/daily"';

const pages = dv.pages(dailySource)
    .where(p => {
        const date = p.file.day ?? dv.date(p.file.name);
        return date && date >= startDate && date <= endDate;
    })
    .sort(p => p.file.name, "asc");

if (pages.length > 0) {
    dv.list(pages.map(p => p.file.link));
} else {
    dv.paragraph("_За эту неделю дневные заметки не найдены._");
}
})();
```

## Сон

```dataviewjs
(() => {
const current = dv.current();
const startDate = dv.date(current["week-start"]);
const endDate = dv.date(current["week-end"]);

if (!startDate || !endDate) {
    dv.paragraph("⚠️ Не заполнены поля `week-start` и `week-end`.");
    return;
}

const pages = Array.from(
    dv.pages('"day_notes/daily" or "day_notes/archive/daily"')
        .where(p => {
            const date = p.file.day ?? dv.date(p.file.name);
            return date && date >= startDate && date <= endDate;
        })
        .sort(p => p.file.name, "asc")
);

function parseDuration(value) {
    if (value === null || value === undefined || value === "") return 0;

    const match = String(value).trim().match(/^(\d+):(\d{1,2})$/);
    if (!match) return 0;

    return Number(match[1]) * 60 + Number(match[2]);
}

function formatDuration(minutes) {
    const hours = Math.floor(minutes / 60);
    const mins = minutes % 60;
    return `${hours}:${String(mins).padStart(2, "0")}`;
}

const tableData = [];
let totalMinutes = 0;

for (const page of pages) {
    const date = page.file.day ?? dv.date(page.file.name);
    const durationMinutes = parseDuration(page["sleep-duration"]);
    const sleepStart = page["sleep-start"] ?? "—";
    const sleepEnd = page["sleep-end"] ?? "—";

    totalMinutes += durationMinutes;

    tableData.push([
        date.setLocale("ru").toFormat("cccc, dd.LL.yyyy"),
        `${sleepStart} – ${sleepEnd}`,
        durationMinutes > 0 ? formatDuration(durationMinutes) : "—"
    ]);
}

if (tableData.length === 0) {
    tableData.push(["Нет данных", "—", "—"]);
}

tableData.push([
    "Итого за неделю",
    "",
    formatDuration(totalMinutes)
]);

dv.table(
    ["День", "Время сна", "Общее время сна"],
    tableData
);
})();
```

## Энергия

```dataviewjs
(() => {
const current = dv.current();
const startDate = dv.date(current["week-start"]);
const endDate = dv.date(current["week-end"]);

if (!startDate || !endDate) {
    dv.paragraph("⚠️ Не заполнены поля `week-start` и `week-end`.");
    return;
}

const pages = Array.from(
    dv.pages('"day_notes/daily" or "day_notes/archive/daily"')
        .where(p => {
            const date = p.file.day ?? dv.date(p.file.name);
            return date && date >= startDate && date <= endDate;
        })
);

const pagesByDate = new Map(
    pages.map(p => [(p.file.day ?? dv.date(p.file.name)).toISODate(), p])
);

const dates = [];
const labels = [];

for (let offset = 0; offset < 7; offset++) {
    const date = startDate.plus({ days: offset });
    dates.push(date.toISODate());
    labels.push(date.setLocale("ru").toFormat("ccc, dd.LL"));
}

function metricValues(propertyName) {
    return dates.map(date => {
        const value = pagesByDate.get(date)?.[propertyName];
        if (value === null || value === undefined || value === "") return null;

        const number = Number(value);
        return Number.isFinite(number) ? number : null;
    });
}

const metrics = [
    {
        name: "Энергия (утро)",
        data: metricValues("morning-energy"),
        color: "orange"
    },
    {
        name: "Энергия (день)",
        data: metricValues("day-energy"),
        color: "purple"
    },
    {
        name: "Энергия (вечер)",
        data: metricValues("evening-energy"),
        color: "red"
    }
];

const traces = metrics.map(metric => ({
    x: labels,
    y: metric.data,
    type: "scatter",
    mode: "lines+markers",
    name: metric.name,
    connectgaps: false,
    line: {
        color: metric.color,
        width: 2
    },
    marker: {
        color: metric.color,
        size: 6,
        opacity: 0.8
    }
}));

const div = this.container.createEl("div");

const layout = {
    title: {
        text: "Динамика энергии за неделю",
        x: 0.5
    },
    xaxis: {
        title: "День"
    },
    yaxis: {
        title: "Оценка",
        range: [0, 10],
        dtick: 1
    },
    legend: {
        orientation: "h",
        x: 0,
        y: 1.12
    },
    margin: {
        t: 100,
        b: 80
    },
    hovermode: "x unified"
};

Plotly.newPlot(div, traces, layout, {
    responsive: true,
    displaylogo: false
});
})();
```

### Почему график такой?

## 1️⃣ Основные выводы за неделю

### Ключевые достижения и события



### Новые инсайты, которые важно не потерять



### То, что получилось сделать особенно хорошо



## 2️⃣ Сравнение запланированного и сделанного

### Основной фокус



### Список задач

- Приоритеты:

### Причины отклонений от плана



## 3️⃣ Личные инсайты и рост

### Новые навыки, знания или привычки



### Чему научился, что попробовал впервые



### Ошибки, которые стали полезным уроком



## 4️⃣ Проблемы и блоки

### Что тормозило выполнение планов



### Внутренние и внешние факторы



### Возможные решения на следующую неделю



## 5️⃣ Мотивация и самооценка

Оценка недели по шкале от 1 до 10 указывается в свойстве `week-mark`.

### Что порадовало



### Что нужно улучшить в подходе или планировании



## 6️⃣ Интересные находки

### Статьи, книги, видео



### Новые инструменты или техники работы



### Идеи для проектов и экспериментов



## 7️⃣ Рефлексия

### Общие размышления о неделе


## Над чем работал — что создал

```dataviewjs
(() => {
const current = dv.current();
const rawStartDate = dv.date(current["week-start"]);
const rawEndDate = dv.date(current["week-end"]);

if (!rawStartDate || !rawEndDate) {
    dv.paragraph("⚠️ Не заполнены поля `week-start` и `week-end`.");
    return;
}

const startDate = rawStartDate.startOf("day");
const endDate = rawEndDate.startOf("day");
const endExclusive = endDate.plus({ days: 1 }).startOf("day");

const pages = dv.pages()
    .where(p =>
        p.file.cday >= startDate &&
        p.file.cday < endExclusive &&
        p.file.path !== current.file.path &&
        !p.file.path.startsWith("day_notes/daily/") &&
        !p.file.path.startsWith("day_notes/archive/daily/")
    )
    .sort(p => p.file.cday, "asc");

const grouped = new Map();

for (const page of pages) {
    const day = page.file.cday.toFormat("yyyy-MM-dd");
    if (!grouped.has(day)) grouped.set(day, []);
    grouped.get(day).push(page);
}

for (let offset = 0; offset < 7; offset++) {
    const day = startDate.plus({ days: offset });
    const dayKey = day.toFormat("yyyy-MM-dd");
    const heading = day
        .setLocale("ru")
        .toFormat("cccc, dd.LL.yyyy");

    dv.header(3, heading.charAt(0).toUpperCase() + heading.slice(1));

    if (grouped.has(dayKey)) {
        dv.list(grouped.get(dayKey).map(p => p.file.link));
    } else {
        dv.paragraph("_Нет заметок._");
    }
}
})();
```


