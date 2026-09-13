---
<%*
const bedTimeStart = await tp.system.prompt("Во сколько лег в кровать? (HH:mm)");
const sleepStart = await tp.system.prompt("Во сколько заснул? (HH:mm)");
const sleepEnd = await tp.system.prompt("Во сколько проснулся? (HH:mm)");

const quickFallAsleep = await tp.system.prompt("Заснул быстро?");
const nightAwakenings = await tp.system.prompt("Просыпался ли ночью?");
const deepSleep = await tp.system.prompt("Сон был глубоким?");
const rememberedDreams = await tp.system.prompt("Запомнил сны?");
const noNightmare = await tp.system.prompt("Не было кошмаров?");
const morningMood = await tp.system.prompt("Оцени утреннее настроение (1–10)");
const sleepQuality = await tp.system.prompt("Оцени общее качество сна (1–10)");
const noPhone = await tp.system.prompt("Перед сном не залипал в экран?");
const physicalExercise = await tp.system.prompt("Занимался спортом?");
const lateDinner = await tp.system.prompt("Поздно ужинал?");

/**
 * Преобразует время H:mm или HH:mm в минуты от начала суток.
 */
function parseTime(value) {
    if (!value) return null;

    const match = String(value)
        .trim()
        .match(/^([01]?\d|2[0-3]):([0-5]\d)$/);

    if (!match) return null;

    return Number(match[1]) * 60 + Number(match[2]);
}

/**
 * Рассчитывает продолжительность между двумя моментами.
 * Если конец раньше начала, считается, что конец наступил
 * на следующий календарный день.
 */
function calculateDuration(start, end) {
    const startMinutes = parseTime(start);
    let endMinutes = parseTime(end);

    if (startMinutes === null || endMinutes === null) {
        return "";
    }

    if (endMinutes < startMinutes) {
        endMinutes += 24 * 60;
    }

    const durationMinutes = endMinutes - startMinutes;
    const hours = Math.floor(durationMinutes / 60);
    const minutes = durationMinutes % 60;

    return `${hours}:${String(minutes).padStart(2, "0")}`;
}

/**
 * Безопасно записывает введённое значение в YAML.
 */
function yamlString(value) {
    return JSON.stringify(value ?? "");
}

const sleepDuration = calculateDuration(sleepStart, sleepEnd);

tR += [
    `bed-time-start: ${yamlString(bedTimeStart)}`,
    `sleep-start: ${yamlString(sleepStart)}`,
    `sleep-end: ${yamlString(sleepEnd)}`,
    `sleep-duration: ${yamlString(sleepDuration)}`,
    `quick-fall-asleep: ${yamlString(quickFallAsleep)}`,
    `night-awakenings: ${yamlString(nightAwakenings)}`,
    `deep-sleep: ${yamlString(deepSleep)}`,
    `remembered-dreams: ${yamlString(rememberedDreams)}`,
    `no-nightmare: ${yamlString(noNightmare)}`,
    `morning-mood: ${yamlString(morningMood)}`,
    `sleep-quality: ${yamlString(sleepQuality)}`,
    `no-phone: ${yamlString(noPhone)}`,
    `physical-exercise: ${yamlString(physicalExercise)}`,
    `late-dinner: ${yamlString(lateDinner)}`,
    `morning-energy:`,
    `day-energy:`,
    `evening-energy:`,
    `type: daily`,
    `tags:`,
    `  - daily-review`,
].join("\n");
%>
---
# Day planner

- [ ] 07:10 - 07:20 Событие
--- 
# Планы на день
## Фокус дня
- 
## Основные дела

## Второстепенные задачи

---
# Чему я рад и что получилось

___
# Что пошло не так 

## Что
### Причина

### Последствия

---
# Заметки

---
# Надо подумать о
