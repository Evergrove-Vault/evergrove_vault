# Вступление 

Всем привет. Это стартовая страница для использования данной базы знаний. Прочитайте все инструкции, чтобы лучше понять с чего и как начинать. 

# Зачем этот проект

Основной целью проекта является сбор и структурирование полезной информации по разным около-IT сферам, работе, организации развития и повышения общей экспертизы. 

Мы хотим собрать тут всю полезную информацию из разных сфер деятельности, чтобы на ее основе подготавливать материалы для институтских курсов, делиться полезными знаниями и всесторонне развиваться.

Вы можете присоединиться к проекту в качестве читателя или контрибьютера. Мы будем рады любой помощи.
# Как начать пользоваться 
## Базовая настройка

Чтобы начать пользоваться проектом выполните следующие шаги:

1) Установить git на компьютер с [официального сайта](https://git-scm.com/)
2)  Склонируйте репозиторий в удобную папку на компьютере
```bash
git config --global user.name "ваше имя"
git config --global user.email ваш емейл
git clone https://github.com/Evergrove-Vault/evergrove_vault.git
```
1) Скачайте клиент обсидиана с [официального сайта](https://obsidian.md/download) и установите
2) Выберите **Open folder as vault** и откройте папку в которую вы сохранили репозиторий
3) В настройках (нижний левый угол) включите поддержку **Community plugins** 
4) Во вкладки **Git** в **Community plugins** настройте следующие хот-кеи:
	- Git: Pull
	- Git: Push 
	- Git: Commit all changes
5) Перед началом работы не забывайте обновлять репозиторий с помощью команды **Pull**

>[!tip] 
>Для того чтобы использовать панель команд нажмите ctl + p или на значок терминала слева сверху

>[!tip]
> Основной набор плагинов уже находится в файлах репозитория. Если захотите расширить его, то воспользуйтесь вкладкой Community plugins в настройках
> 

## Как пользоваться Obsidian 

Вот список полезных материалов, которые помогут вам на начальном этапе адоптироваться к приложению:

- [Базовый Markdown (MD) синтаксис](https://skillbox.ru/media/code/yazyk-razmetki-markdown-shpargalka-po-sintaksisu-s-primerami/) Учтите, что синтаксис ссылок в obsidian отличается 
- [Wiki по obsidian](https://help.obsidian.md/Editing+and+formatting/Basic+formatting+syntax)
- [Ru чат Obsidian](t.me/obsidian_z)
- В скором времени добавлю примеры использования плагинов. #todo
## Как организуется Zettelkasten

- [Основные принципы zettelkasten](https://habr.com/ru/articles/508672/)
- [Что такое MOC заметки](https://forum.obsidian.md/t/what-is-a-moc/58423)
- [Краткое введение в evergreen notes](https://www.youtube.com/playlist?list=PLNpm6do7fQnhgDS6vArb-4F-zZygnXppE)

В дальнейшем хочется расширить данную тему. #todo
## Хочу внести изменения 

Чтобы можно было отслеживать изменения в документах и не закопаться в версиях мы будем использовать механизм Pull Request (PR) и [систему ветвления](https://habr.com/ru/articles/106912/). Убедительная просьба с ней ознакомиться. 
Чтобы добавить изменения:
6) Перейдите на dev ветку 
```bash
git checkout dev 
```
7) Создайте свою ветку от dev куда внесете изменения
```bash
git checkout -b branch_name 
```
8) После чего внесите необходимые изменения и запуште их в репозиторий
```bash
git add *
git commit -m 'description'
git push
```
9) [Создайте PR](https://docs.github.com/en/pull-requests/collaborating-with-pull-requests/proposing-changes-to-your-work-with-pull-requests/creating-a-pull-request) 
10) Участвуйте в обсуждении PR 
