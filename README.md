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

Zoom использует собственный протокол передачи данных, основанный на HTTP. Поскольку Zoom не раскрывает деталей его реализации мы будем рассматривать аналогичные протоколы [HLS](https://en.wikipedia.org/wiki/HTTP_Live_Streaming) и [DASH](https://en.wikipedia.org/wiki/Dynamic_Adaptive_Streaming_over_HTTP). Для аудио и видео как правило создаются отдельные соеденения.

Для рассчеты будем считать, что в худшем случае все устройства способны отправить видео в 1080p 30fps и стерео (2.0) аудио.
Для сжатия изображения будет использватся алгоритм [HEVC](https://en.wikipedia.org/wiki/High_Efficiency_Video_Coding), для сжатия аудио - [xHE-AAC](https://www.iis.fraunhofer.de/en/ff/amm/broadcast-streaming/xheaac.html). Выбор обусловлен наиболее высоким соотношением битрейта к размеру файла.
Для видео будем рассчитывать битрейт 5800 кбит/c, для аудио - 160 кбит/c

*Источник: [HTTP Live Streaming Authoring Specification](https://developer.apple.com/documentation/http_live_streaming/http_live_streaming_hls_authoring_specification_for_apple_devices)*

Будем считать, что в конференции в среднем 100 человек. Следовательно, каждый пользователь должен иметь подключение не менее чем 580 кбит/c. Срежняя скорость мобильного интернет в США [110 мбит/c](https://worldpopulationreview.com/country-rankings/internet-speeds-by-country#:~:text=203.81-,110.07,-Spain), будем расчитывать, что наш сервис должен потреблять не более половины т.е. 65 мбит/c. Таким образом, требуется снизить объем трафика для передачи, достигнуть этого можно, путем предачи видео в разрешении 768 x 432 для 98 участников. Для говорящего в данный момент пользователя будем все так же загружать видео в 1080p. Критерием того, что человек говрит, будем считать наличие звук в его аудиоканале более 3 секунд. Изображение самого пользователя будем передавать в full hd только при условии, что он говорит. Стоит отметить, что один пользователь может демонстрировать экран, а другой - говрить, в таком случае необходимо передавать изображение экрана и камеры говорящего человека в качетсве 1080p.
В таком случае будем получать битрейт
$$0.3 * 99 + 5.8 * 2 + 0.16 * 100 = 57.3\text{ мбит/c.}$$

### Технические метрики
