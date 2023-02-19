# Проектирование высоконагруженной системы видеосвязи
Zoom — сервис для организации видеоконференций. Она предоставляет сервис видеотелефонии, который позволяет подключать одновременно до 100 устройств бесплатно, с 40-минутным ограничением для бесплатных аккаунтов. Пользователи имеют возможность повысить уровень обслуживания, используя один из тарифных планов, с максимальным числом подключений до 500 человек одновременно, без ограничений по времени.
# 1. Тема и целевая аудитория
Аналоги:
- Google Meet
- Microsoft Teams
- Skype

### MVP
- Проведение видоконференции, создание приглашения на нее
- Чат и доска для рисования, связанные с каждой конференцией
- Запись и сохранение звонков как локально, так и облачно
### Целевая аудитория
|  Дата 	|  Кол-во участников звонков в день, млн 	|  
|---	|---	|
|  Март 2019 	|  10 	|
|  Март 2020 	|   200	|
|  Апрель 2020 	|   300	|
|  Декабрь 2020 	|   350	|

*Источник: Zoom, [Recode](https://www.vox.com/recode/21726260/zoom-microsoft-teams-video-conferencing-post-pandemic-coronavirus)*

![Статистика использования по странам](/img/zoom-usage-by-country.jpg)

*Источник: [Wallstreetzen](https://www.wallstreetzen.com/stocks/us/nasdaq/zm/statistics#:~:text=Zoom%20Traffic%20by%20Country), [SimilarWeb](https://www.similarweb.com/website/zoom.us/#overview)*

![annualized-zoom-minutes-over-time](/img/annualized-zoom-minutes-over-time.jpg "annualized-zoom-minutes-over-time.jpg")

![number-of-zoom-meetings-over-time](/img/number-of-zoom-meetings-over-time.jpg "number-of-zoom-meetings-over-time")

*Источник: [Zippia](https://www.zippia.com/advice/zoom-meeting-statistics/#:~:text=Zoom%E2%80%99s-,biggest,-competitors%20are%20Microsoft)*

# 2. Расчет нагрузки
### Продуктовые метрики
- MAU 12.92 млн в феврале 2020 г.

*Источник: [CNBC](https://www.cnbc.com/2020/02/26/zoom-has-added-more-users-so-far-this-year-than-in-2019-bernstein.html#:~:text=on%20Zoom%20stock.-,Zoom%20had%2012.92,-million%20monthly%20active)*

вычеслим количество конференций в день

$$\frac{350}{12.92} = 27.09 \text{ млн}$$



### Технические метрики
