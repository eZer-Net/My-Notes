
TCP — протокол, ориентированный на установление соединения. В его задачи входит практически все. Он устанавливает соединения и обеспечивает надежность сети, выполняя повторную передачу данных, а также осуществляет управление потоком данных и контроль перегрузки — и все это в интересах приложений, которые его используют. Большинству интернет-приложений требуется надежная, последовательная передача, и TCP это рабочая лошадка интернета)

## Основы TCP

Протокол управления передачей (Transmission Control Protocol, TCP) был разработан специально для обеспечения надежного сквозного байтового потока по ненадежной интерсети. Интерсеть отличается от отдельной сети тем, что ее участки могут сильно различаться по топологии, пропускной способности, значениям времени задержки, размерам пакетов и другим параметрам.

TCP был описан в RFC 793 в сентябре 1981 года. Со временем он был во многом усовершенствован, были исправлены различные ошибки и неточности. 
- Уточнения и исправления описаны в RFC 1122
- Расширения для высокой производительности — в RFC 1323
- Выборочные подтверждения — в RFC 2018
- Контроль перегрузки — в RFC 2581
- Использование полей заголовка для QoS — в RFC 2873
- Усовершенствованные таймеры повторной передачи — в RFC 2988
- Явные уведомления о перегрузке — в RFC 3168. 
Поскольку это далеко не полный список, для удобной работы со всеми этими RFC был создан специальный указатель (конечно же, в виде еще одного RFC документа) — **RFC 4614.**

Уровень IP не гарантирует правильной доставки дейтаграмм и не накладывает ограничений на скорость их отправки. Именно TCP приходится выбирать правильную скорость отправки (согласно целям эффективного использования пропускной способности и предотвращения перегрузок), следить за истекшими интервалами ожидания и в случае необходимости заниматься повторной передачей дейтаграмм, не достигших адресата. Иногда дейтаграммы доставляются в неправильном порядке. Восстанавливать из них сообщения также обязан TCP. Таким образом, протокол TCP призван обеспечить хорошую производительность и надежность, о которой мечтают многие приложения и которая не предоставляется протоколом IP.

## Модель службы TCP

В основе службы TCP лежат [[Транспортный уровень#Транспортные службы Сокеты Беркли Сокеты Беркли|сокеты (sockets)]], создаваемые как отправителем,
так и получателем.  У каждого сокета есть номер (адрес), состоящий из IP-адреса хоста и 16-битного номера, локального по отношению к хосту и называемого портом. Порт в TCP — это TSAP-адрес. Для обращения к службе TCP между сокетами двух компьютеров должно быть явно установлено соединение. 

Один сокет может использоваться одновременно для нескольких соединений. Другими словами, два и более соединения могут оканчиваться одним сокетом. Соединения различаются по идентификаторам сокетов на обоих концах: (socket1, socket2). Номера виртуальных каналов или другие идентификаторы не используются.

Номера портов со значениями ниже 1024 зарезервированы стандартными службами и доступны только привилегированным пользователям (например, root в UNIX-системах). Они называются **известными портами (well-known ports)**. К примеру, любой процесс, желающий удаленно загрузить почту с хоста, может связаться с портом 143 хоста-адресата и обратиться, таким образом, к его IMAP-демону.
![[Pasted image 20250105130906.png]]

Порты с номерами от 1024 до 49151 можно зарегистрировать через IANA для непривилегированных пользователей, однако приложения могут выбирать свои собственные порты (что они обычно и делают)

Обычно используется один демон, называемый в UNIX inetd (Internet daemon). Он связывается с несколькими портами и ожидает первое входящее соединение. Когда оно возникает, inetd создает новый процесс и вызывает подходящий демон для обработки запроса. Таким образом, постоянно активен только inetd, остальные вызываются, только когда для них есть работа. Inetd узнает, какие порты нужно использовать из конфигурационного файла.

==Все TCP-соединения являются полнодуплексными и двухточечными.== «Полнодуплексное» означает, что трафик может следовать одновременно в обе стороны, а «двухточечное» — что у него есть две конечные точки. Широковещание и многоадресная рассылка протоколом TCP не поддерживаются.

TCP-соединение представляет собой байтовый поток, а не поток сообщений. ==Границы между сообщениями не сохраняются==. Например, если отправляющий процесс записывает в TCP-поток четыре 512-байтные порции данных, эти данные могут быть доставлены получающему процессу в виде:
- Четырех 512-байтных порций
- Двух 1024-байтных порций
- Одной 2048-байтной порции 
- Или как-то еще. 
Способа, с помощью которого получатель мог бы определить, как записывались данные, не существует.

==Получив данные от приложения, протокол TCP может отправить их сразу или поместить в буфер== (чтобы собрать больше данных и отправить их за один раз) по своему усмотрению. Когда приложению необходимо, чтобы данные были переданы немедленно. **Для ускорения передачи данных в TCP существует флаг PUSH (толкнуть),** который включается в пакеты. Но приложения не могут сами устанавливать PUSH при отправке данных. Вместо этого в различных операционных системах используются специальные параметры, позволяющие ускорить передачу данных (например, TCP_NONDELAY в Windows и Linux).

TCP так же имеет старую функцию все еще входит в состав его протокола, но используется редко. Речь пойдет о **срочных данных (urgent data)**. Функция urgent в позволяет передавать данные с высоким приоритетом, которые должны обрабатываться немедленно. **При отправке таких данных приложение устанавливает флаг URGENT, что заставляет TCP-подсистему немедленно передать эти данные, минуя накопление**, а получающее приложение получает сигнал и должно самостоятельно определить начало срочных данных.

## Протокол TCP

==Ключевым свойством TCP, определяющим всю структуру протокола, является то, что в TCP-соединении у каждого байта есть свой 32-разрядный порядковый номер.== Отдельные 32-разрядные порядковые номера используются для указания позиции раздвижного окна в одном направлении и для подтверждений в обратном. 

Отправляющая и принимающая TCP-подсистемы обмениваются данными в виде сегментов. **Сегмент TCP состоит из фиксированного 20-байтного заголовка (плюс необязательная часть)**, за которым могут следовать байты данных. **Размер сегментов** определяется программным обеспечением TCP. Оно может объединять в один сегмент данные, полученные в результате нескольких операций записи, или, наоборот, распределять результат одной записи между несколькими сегментами. Их размер **ограничен двумя пределами**. 
- Во-первых, каждый сегмент, включая TCP-заголовок, должен помещаться в 65 515-байтное поле пользовательских данных IP-пакета
- Во-вторых, в каждом канале есть максимальный размер передаваемого блока (Maximum Transfer Unit, MTU).

**Современные реализации TCP выполняют обнаружение MTU маршрута (path MTU discovery)**. При этом используется метод из [[Межсетевое взаимодействие#MTU (Path Maximum Transmission Unit — максимальный размер пакета для выбранного пути)|RFC 1191]]. Этот метод вычисляет минимальное значение MTU по всем каналам пути, используя сообщения об ошибках ICMP. На основе этого значения TCP выбирает размер сегмента, позволяющий избежать фрагментации.

==Основной протокол, используемый TCP-подсистемами, — это протокол раздвижного окна с динамическим размером окна==. 
- При передаче сегмента отправитель включает таймер. 
- Когда сегмент приходит по назначению, принимающая TCP-подсистема высылает обратно сегмент (с данными, если они есть, или без) с номером подтверждения (он равен порядковому номеру следующего ожидаемого сегмента) и новым размером окна. 
- Если время ожидания подтверждения истекает, отправитель передает сегмент еще раз.

## Заголовок TCP-сегмента

![[Pasted image 20250105140247.png]]

- **Поля Source port и Destination port** 
	 Указывают локальные конечные точки соединения. TCP-порт вместе с IP-адресом хоста образуют уникальный 48-битный идентификатор конечной точки. **Такой идентификатор соединения называется кортежем из пяти компонентов (5 tuple)**, так как он включает пять информационных составляющих: протокол (TCP), IP-адрес отправителя, порт отправителя, IP- адрес получателя и порт получателя.

- **Поля Sequence number и Acknowledgement number (Номер подтверждения)**
	Выполняют функцию по которой они и названы
	Поле Acknowledgement number относится к следующему по порядку ожидаемому байту, а не к последнему полученному. Это накопительное подтверждение (cumulative acknowledgement), так как один номер объединяет в себе информацию обо всех полученных данных.
	Оба поля 32-разрядные, поскольку в TCP-потоке нумеруется каждый байт данных. (каждое поле содержит 32 бита или 4 байта)

- **Поле TCP header length (Длина TCP-заголовка)**
	Сообщает, сколько 32-разрядных слов содержится в TCP-заголовке. Эта информация необходима, так как поле Options, а вместе с ним и весь заголовок имеет переменную длину

-  **Идет неиспользуемое 4-битное поле**. Тот факт, что эти биты не используются уже 30 лет (изначально поле было 6-битным и из них были задействованы только 2 бита), свидетельствует о том, насколько хорошо продуман дизайн TCP. Иначе протоколы использовали бы эти биты, чтобы справиться с его недостатками.

-  **Восемь 1-битных флагов.** 
- CWR и ECE
	Сообщают о перегрузках сети в случае, если используется явное уведомление о перегрузке. RFC 3168. 
	ECN-Echo (ECN-эхо), предлагая ему снизить скорость отправки. 
	CWR с сигналом Congestion Window Reduced (Окно перегрузки уменьшено)
- Бит URG
	Устанавливается в 1 в случае использования поля Urgent pointer (Указатель срочности), где указано байтовое смещение от текущего порядкового номера до срочных данных.
- Бит ACK
	Установлен в 1, значит, поле Acknowledgement number действует. Если ACK установлен в 0 то игнорируется
- Бит PSH
	Является, по сути, PUSH-флагом, с помощью которого отправитель вежливо просит получателя доставить данные приложению сразу, а не хранить их в буфере, пока тот не наполнится
- Бит RST
	 Используется для внезапного сброса состояния соединения, которое из-за сбоя хоста или по другой причине попало в тупиковую ситуацию.
- Бит SYN
	Применяется для установки соединения. У запроса соединения бит SYN = 1, а бит ACK = 0, то есть поле подтверждения не задействовано. Но в ответе на этот запрос подтверждение есть, поэтому значения этих битов таковы: SYN = 1,ACK = 1. Таким образом, бит SYN используется для обозначения как сегмента CONNECTION REQUEST, так и CONNECTION ACCEPTED, а бит ACK — чтобы различать их.
- Бит FIN
	Используется для разрыва соединения. Он сообщает, что у отправителя больше нет данных для передачи. Однако, даже закрыв соединение, процесс может продолжать получать данные в течение неопределенного времени. У сегментов с битами FIN и SYN есть порядковые номера, что гарантирует правильный порядок их выполнения.

- **Поле Window size (Размер окна)**
	Сообщает, сколько байтов может быть отправлено после подтвержденного байта. Нулевое значение Window size означает, что все байты до Acknowledgement number – 1 включительно пришли, но получатель их еще не обработал, и поэтому остальные байты он пока принять не может. Позже получатель может разрешить дальнейшую передачу, отправив сегмент с таким же значением Acknowledgement number и ненулевым значением Window size. **Получатель может сказать: «Я получил байты вплоть до k-го, пока что достаточно, спасибо». Такое разделение (а если точнее, окно переменного размера) придает протоколу дополни- тельную гибкость.**

- **Поле Checksum**
	Служит для повышения надежности. Как и в UDP, оно содержит контрольную сумму заголовка, данных и псевдозаголовка. Но в отличие от UDP псевдозаголовок содержит номер протокола TCP (6), а контрольная сумма является обязательной.

-  **Поле Urgent pointer (Указатель срочности)**
	Даёт сигнал что данный пакет надо в первую очередь обработать

- **Поле Options**
	Предоставляет дополнительные возможности, не покрываемые стандартным заголовком. Существует множество параметров, и некоторые из них широко используются.
	- Maximum Segment Size (MSS):
		При установлении соединения с помощью трехстороннего рукопожатия (SYN) хост может объявить максимальный размер сегмента (помогает избежать фрагментации пакетов).
	- Window Scale
		Используется для увеличения размера окна TCP, что позволяет улучшить производительность при высоких задержках и больших пропускных способностях.
	- Selective Acknowledgment (SACK)
		Позволяет получателю сообщать отправителю о том, какие сегменты были успешно получены, а какие — потеряны.
	- Timestamp
		Используется для измерения времени передачи сегментов и может помочь в управлении задержками и улучшении производительности.
	- Padding
		Иногда длина поля Options может быть не кратна 32 битам. В таких случаях используется опция Padding для дополнения поля до нужной длины. Это обеспечивает правильное выравнивание заголовка TCP


## Установка TCP-соединения

В TCP соединения устанавливаются с помощью [[Элементы транспортных протоколов#Решение «тройное рукопожатие»|«тройного рукопожатия»]]

Проблема реализации схемы «тройного рукопожатия» состоит в том, что слушающий процесс должен помнить свой порядковый номер до тех пор, пока он не отправит собственный SYN-сегмент. Это значит, что злонамеренный отправитель может **блокировать ресурсы хоста, отправляя на него поток SYN-сегментов и не разрывая соединение. Такие атаки называются лавинной адресацией SYN- сегментов (SYN flood).**

Для защиты от них можно использовать метод под названием SYN cookies. Вместо того чтобы запоминать порядковый номер, хост генерирует криптографическое значение номера, записывает его в исходящий сегмент и забывает. Если «тройное рукопожатие» завершается, этот номер (увеличенный на единицу) вернется на хост.

Одна из тонкостей этого метода состоит в том, что он не работает с дополнительными параметрами TCP. Поэтому SYN cookies можно использовать только в случае лавинной адресации SYN-сегментов. **Более подробно см. RFC 4987**
## Разрыв TCP-соединения

TCP-соединения полнодуплексные, но их разъединение можно рассматривать как пару симплексных соединений. Каждое направление разрывается независимо.

Для разрыва соединения любая сторона отправляет TCP-сегмент с установленным битом FIN, указывая, что больше нет данных для передачи. После подтверждения (ACK) это направление закрывается. Данные могут продолжать передаваться в противоположную сторону.

**Разрыв соединения требует четырех TCP-[[Транспортные службы#Сегмент|сегментов]]**: по одному с битом FIN и ACK в каждом направлении. Первый ACK и второй FIN могут быть в одном сегменте, что уменьшает количество до трех.

Оба конца могут одновременно отправить FIN-сегменты, получая подтверждения, после чего соединение закрывается. Для предотвращения проблем «двух армий» используются таймеры: если ответ на FIN не приходит в течение двух максимальных интервалов времени жизни пакета, отправитель разрывает соединение. В конце концов, другая сторона также отсоединится.

## Модель управления TCP-соединением

Этапы, необходимые для установления и разрыва соединения, могут быть представлены в виде модели конечного автомата
![[Pasted image 20250105152223.png]]

1. Каждое соединение начинается в состоянии CLOSED (закрыто).

2. Оно может покинуть это состояние, предпринимая либо активную (CONNECT), либо пассивную (LISTEN) попытку открыть соединение.

3. Если другая сторона осуществляет противоположное действие, соединение устанавливается и переходит в состояние ESTABLISHED.

4. Инициатором разрыва соединения может выступить любая сторона.

5. По завершении этого процесса соединение возвращается в состояние CLOSED.


![[Pasted image 20250105152820.png]]
Конечный автомат TCP-соединения. Жирная сплошная линия показывает нормальный путь клиента. Жирным пунктиром показан нормальный путь сервера. Тонкими линиями обозначены необычные события. Для каждого перехода через косую черту указано, какое событие его вызывает и к выполнению какого действия он приводит

## Раздвижное окно TCP

Управление окном в TCP решает проблемы подтверждения доставки сегментов и выделения буферов получателя. 

**Пример того как работает раздвижное окно в TCP и с какими проблемами он сталкивается**
Если у получателя есть 4096-байтный буфер и он принимает 2048-байтный сегмент, то подтверждает его получение, оставляя 2048 байт свободного пространства. Получатель сообщает отправителю размер окна (2048) и номер следующего ожидаемого байта.

После этого отправитель передает еще 2048 байт, которые также подтверждаются, но размер окна становится равным нулю. Отправитель должен остановить передачу до освобождения места в буфере и увеличения размера окна.

При нулевом размере окна отправитель может отправлять только срочные данные или 1-байтный сегмент для запроса информации о размере окна. Этот пакет называется пробным сегментом (window probe) и предотвращает тупиковые ситуации при потере объявления о размере окна.

Отправители не обязаны немедленно передавать данные, как только они поступают от приложения. Например, если TCP-подсистема получает первые 2 Кбайт данных и знает, что размер окна равен 4 Кбайт, она может сохранить данные в буфере до получения еще 2 Кбайт для передачи сегмента в 4 Кбайта. Это может улучшить производительность и снизить нагрузку на сеть.

Однако отправитель, передающий множество маленьких пакетов, работает неэффективно. Для повышения эффективности используется алгоритм Нейгла: если данные поступают маленькими порциями, отправляется первый фрагмент, а остальные помещаются в буфер до получения подтверждения. После этого можно переслать все накопленные данные одним TCP-сегментом.

Алгоритм Нейгла широко применяется в различных реализациях TCP, но его лучше отключать в ситуациях, требующих быстрого потока мелких пакетов, например, в интерактивных играх по интернету.

## Управление таймерами в TCP

В TCP используется множество таймеров (по крайней мере, в теории). Наиболее важным из них является ==таймер повторной передачи (Retransmission TimeOut, RTO)==, который запускается при отправке сегмента. Главный вопрос насколько долгим должен быть интервал времени ожидания.
- На канальном уровне всё просто там всё измеряется в мкс и проще вычислить
- На транспортном уровне всё гораздо сложнее 
![[Pasted image 20250105154234.png]]

==Решение состоит в использовании динамического алгоритма, который постоянно меняет период ожидания, основываясь на измерениях производительности сети.== 

Для каждого соединения в TCP предусмотрена переменная ==SRTT (Smoothed Round-Trip Time== — усредненное время в пути туда-обратно), в которой хранится текущее наилучшее ожидаемое время получения подтверждения для данного соединения. 
	При отправке сегмента запускается таймер, который измеряет время получения подтверждения и повторяет передачу, если оно не приходит в срок. Если подтверждение успевает вернуться до истечения периода ожидания, TCP-подсистема подсчитывает время, которое понадобилось для его получения (R). Затем значение переменной SRTT обновляется по формуле.

Но модели очередей случайного (то есть пуассоновского) трафика показывают, что **когда нагрузка приближается к пропускной способности**, задержка растет и становится крайне изменчивой. В результате **может сработать таймер повторной передачи, после чего будет отправлена копия пакета**, хотя оригинальный пакет все еще будет находиться в сети. Как правило, такие ситуации возникают именно при высокой нагрузке — и это не самое лучшее время для отправки в сеть лишних пакетов.

Чтобы решить эту проблему, Джейкобсон предложил сделать **интервал времени ожидания чувствительным к отклонению RTT и к усредненному RTT. Для этого потребовалась еще одна сглаженная переменная — RTTVAR (RoundTrip Time Variation)** — изменение времени в пути туда-обратно, которая вычисляется так же через формулу.
Более подробные сведения о том, как вычислять этот интервал ожидания, а также начальные значения переменных, можно найти в RFC 2988.

При сборе данных (R) для вычисления RTT возникает вопрос, что делать при повторной передаче сегмента. Когда для него приходит подтверждение, неясно, к какой передаче оно относится, первой или последней. Неверная догадка может серьезно нарушить работу RTO. **Предложение Карна было очень простым: не обновлять оценки для сегментов, переданных повторно.** Кроме того, при каждой повторной передаче время ожидания можно удваивать до тех пор, пока сегменты не пройдут с первой попытки. Это исправление получило название ==алгоритма Карна (Karn’s algorithm) (Карн и Партридж; Karn and Partridge, 1987) и применяется в большинстве реализаций TCP.==

##### В протоколе TCP используется не только таймер повторной передачи, но и таймер настойчивости (persistence timer). 

1. Состояние нулевого окна: Когда получатель TCP сообщает отправителю, что размер его окна равен нулю, это означает, что у получателя нет свободного места для приема данных. Отправитель должен приостановить передачу данных до тех пор, пока не получит подтверждение о том, что размер окна увеличился.

2. Таймер настойчивости: **Если получатель не может сразу отправить обновление о размере окна (например, если он еще не освободил место), отправитель начинает отсчитывать время с момента получения уведомления о нулевом размере окна**. Этот период времени называется таймером настойчивости.

3. Запрос состояния: Когда таймер настойчивости истекает, отправитель отправляет специальный пакет, чтобы узнать текущее состояние окна у получателя. Это пакет не содержит данных, а только служит для запроса информации о размере окна.

4. Ответ получателя: Получатель отвечает на этот запрос, указывая текущий размер окна. Если размер окна все еще равен нулю, отправитель снова запускает таймер настойчивости и ждет следующего ответа.

5. Цикл повторений: Этот процесс повторяется до тех пор, пока размер окна не станет больше нуля. Как только получатель сообщает об увеличении размера окна, отправитель может возобновить передачу данных.

Таким образом, протокол настойчивости помогает избежать ситуации, когда обе стороны TCP "зависают" в ожидании действий друг друга. Он обеспечивает регулярные проверки состояния и позволяет отправителю узнать, когда он может продолжить передачу данных.

##### В некоторых реализациях протокола используется третий таймер, называемый таймером проверки активности (keepalive timer). 
Он срабатывает, если соединение простаивает в течение долгого времени, заставляя одну сторону проверить, активна ли другая сторона. Если проверяющая сторона не получает ответа, соединение разрывается. Это свойство протокола довольно противоречиво, поскольку привносит дополнительные накладные расходы и может разорвать вполне жизнеспособное соединение из-за кратковременной потери связи.


##### в каждом TCP-соединении используется таймер, запускаемый в состоянии TIME WAIT при закрытии соединения. Он отсчитывает двойное время жизни пакета, чтобы гарантировать, что после закрытия соединения созданные им пакеты исчезли.


## Контроль перегрузки в TCP

Когда в сеть (в том числе в интернет) поступает больше данных, чем она способна обработать, возникают перегрузки. **Если сетевой уровень узнает, что на маршрутизаторах скопились длинные очереди, он пытается справиться с этой ситуацией (пусть даже простым удалением пакетов)**. Транспортный уровень получает обратную связь от сетевого уровня, что позволяет ему следить за перегрузкой и при необходимости снижать скорость отправки.

###### 1. **Окно перегрузки (Congestion Window)**

• Описание: Окно перегрузки определяет максимальное количество байтов, которые отправитель может передавать без получения подтверждения от получателя. Размер окна зависит от состояния сети и определяется с использованием правила AIMD (Additive Increase Multiplicative Decrease).

• Проблемы:
  - Перегрузка сети может привести к потере пакетов.
  - Неправильный расчет размера окна может вызвать ненужные задержки и блокировки.

• Решения:
  - Использование AIMD позволяет постепенно увеличивать размер окна при отсутствии потерь и резко уменьшать его при обнаружении потерь, что помогает адаптироваться к состоянию сети.

###### 2. **Метод AIMD (Additive Increase Multiplicative Decrease)**

• Описание: ==Метод AIMD увеличивает размер окна перегрузки на фиксированное значение (обычно 1 сегмент) при успешной передаче и уменьшает его в два раза при потере пакета.==

• Проблемы:
  - Может привести к нестабильной работе в условиях изменяющейся нагрузки.
  - Не учитывает особенности различных сетевых путей.

• Решения:
  - Регулировка окна на основе реального состояния сети позволяет более эффективно использовать доступную пропускную способность.

###### 3. **Тайм-ауты и отслеживание потерь**

• Описание: TCP отслеживает тайм-ауты для повторной передачи пакетов и использует порядковые номера для определения потерянных пакетов.

• Проблемы:
  - Неправильная настройка тайм-аутов может привести к преждевременной или запоздалой повторной передаче.
  - Необходимость в быстрой реакции на потерю пакетов.

• Решения:
  - Оптимизация времени ожидания повторной передачи для более точного определения состояния сети.

###### 4. **Медленный старт (Slow Start)**

• Описание: Алгоритм медленного старта начинает с небольшого окна перегрузки и экспоненциально увеличивает его размер, пока не произойдет потеря пакета или не будет достигнуто определенное значение.

• Проблемы:
  - Может быть недостаточно эффективным в условиях высокой пропускной способности сети, если окно слишком маленькое.
  - Резкий рост может вызвать перегрузку на узких местах сети.

• Решения:
  - Экспоненциальный рост позволяет быстро достигнуть оптимального размера окна, но с осторожностью, чтобы не перегружать сеть.
![[Pasted image 20250105165626.png]]

###### 5. **Скорость прихода подтверждений (Ack Clock)**

• Описание: Этот **параметр позволяет TCP контролировать скорость передачи данных на основе получения подтверждений от получателя.**

• Проблемы:
  - Задержка в получении подтверждений может замедлить передачу данных.
  - Очереди на маршрутизаторах могут увеличиваться, если скорость передачи не согласована со скоростью обработки.

• Решения:
  - Выравнивание трафика на основе скорости получения подтверждений помогает избежать ненужных задержек и перегрузок на маршрутизаторах.

###### 6. **TCP Tahoe**

• Описание: TCP Tahoe — это один из первых алгоритмов управления перегрузкой в протоколе TCP, который был разработан для улучшения надежности передачи данных в сетях. Он использует метод AIMD (Additive Increase Multiplicative Decrease) для управления размером окна перегрузки и включает в себя три ключевых механизма: медленный старт, контроль перегрузки и быстрый повторный запрос.

• Механизмы:
  - Медленный старт: При инициализации соединения размер окна начинается с небольшого значения и удваивается с каждым успешным ACK, что позволяет быстро наращивать пропускную способность до тех пор, пока не произойдет потеря пакетов.
  - Контроль перегрузки: Когда обнаруживается потеря пакета (например, по истечении времени ожидания или по получению дубликата ACK), размер окна сбрасывается до одного сегмента, и начинается новый цикл медленного старта.
  - Быстрый повторный запрос: Если отправитель получает три дубликата ACK, он сразу же повторно передает потерянный пакет, что помогает сократить время простоя.

• Проблемы:
  - TCP Tahoe неэффективен в условиях высоких потерь пакетов, так как он сбрасывает размер окна до одного сегмента при каждой потере, что может привести к значительным задержкам в передаче данных.
  - Алгоритм не использует механизм быстрого восстановления, что делает его менее адаптивным к изменяющимся условиям сети.
![[Pasted image 20250105165446.png]]

###### 7. **TCP Reno**

• Описание: TCP Reno — это один из алгоритмов управления перегрузкой, который улучшает стандартный TCP, добавляя механизмы быстрого восстановления и быстрого повторного запроса. Он использует метод AIMD для управления размером окна перегрузки и включает в себя два ключевых механизма: "быстрый повторный запрос" (Fast Retransmit) и "быстрое восстановление" (Fast Recovery).
  
• Проблемы:
  - Неэффективен при множественных потерях пакетов, так как он может не обнаружить все потери сразу.
  - Быстрое восстановление может привести к избыточной передаче данных, если сеть перегружена.
  
• Решения:

  - Механизм быстрого повторного запроса позволяет отправителю сразу же повторно передавать потерянный пакет, когда получает три дубликата ACK, что сокращает время простоя.
  - Быстрое восстановление позволяет отправителю продолжать передачу данных, не возвращаясь в состояние медленного старта, что улучшает использование пропускной способности после потерь.
![[Pasted image 20250105165208.png]]


## CUBIC TCP

Чтобы справиться с растущим значением произведения пропускной способности на задержку, была разработана версия протокола TCP под названием CUBIC TCP.

Суть CUBIC TCP сводится к тому, что окно перегрузки растет в зависимости от времени с момента прибытия последнего дубликата подтверждения (а не просто на основании поступления подтверждений).

Корректировка окна перегрузки в зависимости от времени также производится несколько иным образом. В отличие от описанного ранее стандартного контроля перегрузки по правилу AIMD, окно растет как кубическая функция; при этом после начального роста оно выходит на «плато», после которого следует период еще более быстрого увеличения.
![[Pasted image 20250105164758.png]]

Одно из главных отличий CUBIC от других версий TCP — окно меняется как функция времени, прошедшего после последней перегрузки. Сначала оно быстро увеличивается, затем выходит на «плато» с размером, достигнутым отправителем перед последней перегрузкой, после чего снова растет для обеспечения максимально возможной скорости, пока не возникнет новая перегрузка.

----