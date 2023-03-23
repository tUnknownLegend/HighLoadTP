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

*Источник: [Zippia](https://www.zippia.com/advice/zoom-meeting-statistics/#:~:text=Zoom%E2%80%99s-,biggest,-competitors%20are%20Microsoft)*

# 2. Расчет нагрузки
### Продуктовые метрики
- MAU возьмем 400 млн.
- DAU возьмем 270 млн.
*Источник: [Statista](https://www.statista.com/statistics/1033742/worldwide-microsoft-teams-daily-and-monthly-users/#:~:text=The%20number%20of%20daily%20active,to%20270%20million%20in%202022.)*
- Средний размер хранилища пользователя возмем 512 Гб
- Среднее количесвто действий пользователя по типам

| Действие 	                            | Кол-во в день 	                                                                                                                              |  
|---------------------------------------|----------------------------------------------------------------------------------------------------------------------------------------------|
| Отправленных сообщений в чат 	        | 350 млн	                                                                                                                                     |
| Генерации приглашений на коференцию 	 | 270  млн	                                                                                                                                    |
| Минут звонка и записи                 | 500 млн                                                                                                                                      |
| Средняя продолжительность звонка      | [54 мин](https://blog.zoom.us/how-you-zoomed-over-the-past-year-2021/#:~:text=meeting%20length%20was-,54%20minutes,-The%20average%20meeting) |
| Среднее число участников | [10 чел](https://blog.zoom.us/how-you-zoomed-over-the-past-year-2021/#:~:text=meeting%20size%20was-,10%20participants,-Additionally%2C%20these%20were) |

**Вычисления**
- Для сообщений в чате будем считать, что каждый участник конференции отправит по сообщению, т.е. будем иметь 350 млн сообщений в день.
- Для генерации приглашений в чат будем считать, что в худшем случае каждый пользователь создает по одной ссылке в день.
- Для отправки сообщений в чат будем считать, что в худшем случае для в конференции каждый пользователь пишет по одному сообщению
- Имеем 45 млрд минут в квартал. Тогда всего порядка
  $$\frac{45 * 10^9}{3 * 30} \approx 500 \text{ млн минут в день}$$
- Вычислим число коференций в день.
  $$\frac{500 * 10^6}{54} \approx 9.26 \text{ млн в день.}$$

### Технические метрики

Zoom использует собственный протокол передачи данных, основанный на HTTP. Поскольку Zoom не раскрывает деталей его реализации мы будем рассматривать аналогичные протоколы [HLS](https://en.wikipedia.org/wiki/HTTP_Live_Streaming) и [DASH](https://en.wikipedia.org/wiki/Dynamic_Adaptive_Streaming_over_HTTP). Для аудио и видео как правило создаются отдельные соеденения.

Для рассчеты будем считать, что в худшем случае все устройства способны отправить видео в 1080p 30fps и стерео (2.0) аудио.
Для сжатия изображения будет использватся алгоритм [HEVC](https://en.wikipedia.org/wiki/High_Efficiency_Video_Coding), для сжатия аудио - [xHE-AAC](https://www.iis.fraunhofer.de/en/ff/amm/broadcast-streaming/xheaac.html). Выбор обусловлен наиболее высоким соотношением битрейта к размеру файла.
Для видео будем рассчитывать битрейт 4500 кбит/c, для аудио - 160 кбит/c

*Источник: [HTTP Live Streaming Authoring Specification](https://developer.apple.com/documentation/http_live_streaming/http_live_streaming_hls_authoring_specification_for_apple_devices)*

Как ранее было указано, в конференции в среднем 10 человек. Согласно [спецификации](https://support.zoom.us/hc/en-us/articles/201362023-%D0%A1%D0%B8%D1%81%D1%82%D0%B5%D0%BC%D0%BD%D1%8B%D0%B5-%D1%82%D1%80%D0%B5%D0%B1%D0%BE%D0%B2%D0%B0%D0%BD%D0%B8%D1%8F-Zoom-%D0%B4%D0%BB%D1%8F-%D0%9E%D0%A1-Windows-macOS-%D0%B8-Linux#:~:text=1.8Mbps%20(up/down)-,For%201080p%20HD%20video%3A%203.8Mbps/3.0Mbps%20(up/down),-For%20gallery%20view) . 
для проведения звонка должно быть не более 3.8/3.0 мбит/с (загрузка/отправка).
Таким образом, требуется снизить объем трафика для передачи, достигнуть этого можно, 
не передавая видео других участников.
Для говорящего в данный момент пользователя будем все так же загружать видео в 1080p.
Критерием того, что человек говрит, будем считать наличие звук в его аудиоканале более 3 секунд.
Изображение самого пользователя будем передавать в full hd только при условии, что он говорит.
Стоит отметить, что один пользователь может демонстрировать экран, а другой - говрить, 
в таком случае необходимо передавать изображение экрана и камеры говорящего человека в качетсве 1080p.
В таком случае будем получать небоходимый объем трафика для работы видеоконфернеции в 10 человек
На загрузку требуется
$$4500 + 160 * 9 = 5.94\text{ мбит/c.}$$

На отправку требуется
$$4500 + 160 = 4.66\text{ мбит/c.}$$

Получили значения, превышающие системные требования. Значит, протокол, используемый Zoom более эффективный. 
Так же играет роль технология Zoom, позволяющая передавать только изменившуюся часть картинки,
таким образом не передавая фон. Пусть разарешение в таком случае составит 1280 x 720.
Тогда будем иметь битрейт 2400 мбит/с. Проведем рассчеты снова.
На загрузку требуется
$$2400 + 160 * 9 = 3.84\text{ мбит/c.}$$

На отправку требуется
$$2400 + 160 = 2.56\text{ мбит/c.}$$

Отметим, что как правило не все пользователи включают микрофон, поэтому при уведичении числа участников
увеличение трафика будет незанчительное или его не будет вовсе.

[Выберем](https://bitmovin.com/mpeg-dash-hls-segment-length/#:~:text=average%20throughput%20decreases.-,CONCLUSIONS,-Based%20on%20the) длинну чанка с медифайлами 2 секунды. Будем разибвать на наш трафик: одну половину загружаем за первую секунду - вторую за последнюю секунду. Плавность видео и аудио дополнительно будет обеспечивать буффер в 2 секунды.

Вычеслим пиковое сетевое потребление, возьмем запас в 2 раза.
На загрузку
$$\frac{270 * 10^6}{10} * 3.84 * 2 = 207360 \text{ Гбит}$$

На отправку требуется
$$\frac{270 * 10^6}{10} * 2.56 * 2 = 138240 \text{ Гбит}$$

Вычеслим RPS.

- Для видео и аудио файлов, а так же генерации приглашений будем получать
$$\frac{270 * 10^6}{24 * 60} \approx 187.5 \text{ тыс.}$$
- Для отправки сообщений будем получать
$$\frac{350 * 10^6}{24 * 60} \approx 243.1 \text{ тыс.}$$

Сведем RPS в единую таблицу.
|  Действие 	|  Запросов в секунду 	|  Размер чанка, мбит |
|---	|---	|---	|
|  Загрузка видео 	|  187.5 тыс. 	| 41.3 |
|  Загрузка аудио 	|  187.5 тыс.	| 16 | 
|  Отправка сообщения 	|  243.1 тыс.	| - |
|  Генерация приглашений 	|  187.5 тыс.	| - |

Вычеслим средний битрейт записи трансляции для сохранения.
$$2.4 + 0.16 = 2.56\text{ мбит/c.}$$
Тогда средний размер записи будет
$$2.56 * \left(\frac{500 * 10^6}{270 * 10^6} * 60\right) = 284\text{ мбит}$$

Итого будем иметь
$$2.56 * (500 * 10^6 * 60) = 76800\text{ тбит/день записей.}$$

# 3. Логическая схема

![image](https://user-images.githubusercontent.com/57019979/226996946-332aadd7-4ae4-44ef-86b0-d187f05ebe8a.png)

*Примерная схема, может быть изменена после проектировангия физической схемы*.
