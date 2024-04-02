# AppSecCloudCamp
Тестовое задание для отбора на стажировку Cloud.ru

## 1. Ответы на вопросы для разогрева.
#### 1. Расскажите, с какими задачами в направлении безопасной разработки вы сталкивались?
Ещё не представилось возможным принять участие в полном SDLC-цикле, но есть практический опыт безопасной разработки веб-приложения. Задание заключалось в разработке одного веб-приложения в двух версиях: уязвимой и защищенной, где первая содержала следующие уязвимости: XSS, IDOR, SQLI, OS command injection, Path Traversal, Brute Force.
#### 2. Если вам приходилось проводить security code review или моделирование угроз, расскажите, как это было?
Security code review не приходилось проводить. Моделирование угроз проводилось в контексте практического задания по одной из дисциплин в Вузе. Задача заключалась в определении негативных последствий от реализации УБИ, возможных объектов воздействия УБИ, источников УБИ, проведение оценки возможностей нарушителей по реализации УБИ, оценки способов реализации УБИ, оценки сценариев реализации УБИ в системах и сетят. Выполнение задания свелось к  нахождению нужной информации в методическом документе ФСТЭК с последующей привязкой этой информации к организации и ее инфраструктуре. 
#### 3. Если у вас был опыт поиска уязвимостей, расскажите, как это было?
Активно решаю лабораторные на PortSwigger Academy.  
Есть опыт участия в CTF соревнованиях:
- Kettle CTF 2021г. – 9 место из 122.
- Tinkoff CTF 2023г. – 477 место из 1233.
- SCBCTF 2024г. – 6 место из 71 (индивидуальное соревнование).
#### 4. Почему вы хотите участвовать в стажировке?
С недавнего времени открыл для себя направление application security, с которым хочу связать свою профессиональную деятельность. В декабре 2023 г. была попытка попасть на стажировку как application security engineer, но она не увенчалась успехом. Сейчас активно прокачиваю свои хард скиллы, чтобы пройти отбор на новые волны стажировок. Особенно в данной сфере мне нравится следующее: логическое мышление и креативность, которые идут рука об руку; ответственность, так как буду вносить вклад в защиту пользователей и организации от киберпреступлений; постоянное развитие, работа в команде и возможность влиять на то, как происходит разработка и использование приложения.  
Верю, что Cloud.ru может предоставить мне знания и практику, которой так не хватает в учебном заведении. А полученный от Вас ценный опыт, я смогу применить в выбранном мной направлении информационной безопасности.  
## 2. Security code review
### Часть 1. Security code review: GO
В данном коде присутствует **SQL injection** уязвимость. Она представлена в виде строки ```query := fmt.Sprintf("SELECT * FROM products WHERE name LIKE '%%%s%%'", searchQuery)``` (строка 38). Содержимое *searchQuery* параметра является контролируемым пользователем и происходит его конкатенация в строку с запросом без какой-либо валидации содержимого, что не есть хорошо, так как успешная атака приведет к несанкционированному доступу к конфиденциальным данным.  
Способы исправления уязвимости:
- **Prepared statement**  
Для исправления уязвимости следует использовать prepared statement, который позволяет отделить запрос от параметра (параметризация). Происходит сохранение и предкомпиляция запроса без фактического значения параметра, и поэтому введенные пользователем данные никак не могут повлиять на логику запроса.  
- **Валидация входящих значений**  
Можно использовать белые и черные списки, которые будут отвечать за валидацию введенных пользователем данных. В данной ситуации происходит поиск всех продуктов, которые содержат в себе введенную пользователем подстроку, т.е. обращение к данным, которые часто обновляются, поэтому стоит использовать черный список. Белый список можно применять в ситуации, когда, например, пользовательские данные помещаются в поле с именем таблицы в SQL запросе.  
######
По моему мнению prepared statement лучший способ из этих двух, так как он прост в реализации, не зависит от человеческого фактора (нет возможности забыть внести что-либо) и не зависит от релиза  (не нужно следить за новыми выпусками реляционных систем баз данных, чтобы внести нововведения в список).
### Часть 2. Security code review: Python
#### Пример №2.1
В данном коде присутствует **Reflected XSS** уязвимость. Она представлена в виде строки ```output = Template('Hello ' + name + '! Your age is ' + age + '.').render()``` (строка 10). Содержимое *name* и *age* параметров являются котролируемыми пользователем и происходит их конкатенация в строку, содержимое которой рендерится на странице пользователя, без какой-либо валидации содержимого, что может привести к критическим последствиям, таким как: компрометации пользователей и их данных.  
Способы исправления уязвимости:
- **Типизация входящих значений**  
Строковые данные переменной *age* можно привести к целочисленному типу.
- **Валидация входящих значений**  
Для переменной *name* можно сделать проверку, что символы строки - это только буквы алфавита. Соответственно, значение переменной *age* проверить на содержание только цифр.  
- **Санитизация выходящих значений**  
В данной ситуации строка добавляется в html код. Это говорит о том, что необходимо использовать html кодирование, которое предотвратит выполнение полезной нагрузки на стороне клиента.  
Также для данного способа можно использовать библиотеку DOMPurify, которая позволяет очищать HTML от XSS атак.
######
По моему мнению валидации и типизации входящих значений достаточно для защиты от XSS в данном примере, так как проблему валидации решает тривиальное правило. Кодирование выходящих значение необязательно, так как строки содержащие символы отличные от цифр и букв будут отбрасываться, а цифробуквенные символы всё равно никак не кодируются.
######
Помимо cross-site scripting уязвимости, в данном коде присутствует **Information Disclosure** уязвимость. Она представлена в виде строки ```app.run(debug=True)``` (строка 14). Эта строка говорит о том, что приложение запущенов режиме отладки, который предоставляет подробную информацию об ошибках, что может привести к раскрытию важных данных о коде приложения.  
Эта уязвимость исправляется путем отключения отладки.  
#### Пример №2.2
В данном коде присутствует **OS command injection** уязвимость. Она представлена в виде следующих строк (строки 9-10):  
```cmd = 'nslookup ' + hostname```  
```output = subprocess.check_output(cmd, shell=True, text=True)```  
Содержимое *hostname* параметра является контролируемым пользователем и происходит его конкатенация к строке, содержимое которой выполняется как команда с аргументом, без какой-либо валидации, что может привести от кражи конфиденциальной информации до RCE.  
Способы исправления уязвимости:
- **Отказ от вызова команд операционной системы**  
Считается хорошей практикой избегание вызовов команд операционной системы, а заменять их на вызовы функций из встроенных библиотек с похожим функционалом или использование безопасных API. Например, библиотека socket, функция getaddrinfo(), которая согласно RFC 3493 является безопасной.  
- **Валидация входящих значений**  
Согласно найденной информации домен может содержать только алфавитные символы (A-Z), числовые символы (0-9), знак минус (-) и  точку (.). Соответственно сделать проверку whitelist на содержание в введенной строке вышеупомянутых символов.  
- **Принцип минимальных привилегий**  
Приложению следует быть запущенным с минимальными привилегиями, которые необходимы для успешного выполнения его рабочей цели.
######
По моему мнению, если есть возможность заменить вызов команды операционной системы, то следует выбрать именно этот способ исправления уязвимости, так как эта уязвимость может привести к критической. Тем более в данном случае это довольно простой вариант исправления, который не займет много времени. В дополнении к этому способу все равно стоит выдавать минимальные привилегии на стороне сервера, так как, если будет найдена дыра в способе выше, то это значительно сократит риск ущерба от потенциальных внутренних угроз.
######
Помимо OS command injection уязвимости, в данном коде присутствует **Information Disclosure** уязвимость. Она представлена в виде строки ```app.run(debug=True)``` (строка 13). В разборе примера №2.1 упомянулось о последствиях эксплутации данной уязвимости.  
Эта уязвимость исправляется путем отключения отладки.  
## 3. Моделирование угроз
Для категоризации потенциальных проблем была использована модель STRIDE.
### Spoofing
|потенциальные проблемы|последствия|способы исправления|
|--|--|--|
|перехват идентификатора сессии|выполнение злонамернных действий от имени другого пользователя|1) обнаружение кражи на основании внезапного изменения IP или user-agent или использование одного refresh токена <br>2) предотвращение, заключающееся в удаление refresh токена|
|перехват кредов|выполнение злонамернных действий от имени другого пользователя|1)использование HTTPS вместо HTTP <br>2) использование 2FA <br>3) предупреждение пользователей сервиса о возможности мошенничества|
|неограниченный перебор паролей|выполнение злонамернных действий от имени другого пользователя|установка лимита на введение невалидных пар логин/пароль|
######
Вопрос разработчику: На основании чего будет определяться, что пользователь заблокирован на ввод кредов?  
Если ip, то это плохая идея, так как у кого-нибудь может использоваться протокол PAT для преобразования сетевых адресов, и другие пользователи, находящиеся в одной подсети с ним, не смогут вводить пароль, хотя они не были заблокированы.
### Tampering
|потенциальные проблемы|последствия|способы исправления|
|--|--|--|
|изменение конфигураций клиентов сервиса путем манипуляция над запросом, отправленным на сервер|модификация данных|использование ЭЦП|
|внутренний злоумышленник|модификация данных|доступ к базе данных только небольшого количества доверенных лиц|
|внедрение sql инъекций|модификация данных|валидация входящий запросов|
|перезапись критически важных файлов путем загрузки файла с таким же именем|модификация данных|1) переименование файлов или хранение в разных директориях для избежания коллизий <br>2) хранение загруженных файлов на отдельном сервере <br>3) проверка имени файла на path traversal уязвимость|
######
Вопрос разработчику: Настроен ли процесс резервирования баз данных и как часто происходит их синхронизация?
### Repudiation
|потенциальные проблемы|последствия|способы исправления|
|--|--|--|
|отправка информации нелегитимным пользователем от имени легитимного|невозможность доказательства выполнения этой операции нелегитимным пользователем|добавление или улучшение процессов логирования|
######
Вопрос разработчику: Каким образом происходит отслеживание исходного запроса между различными сервисами?  
Если timestamp, то это не лучший вариант, так как может быть отличие во временных зонах и будет непросто отследить весь путь запроса. Вместо этого стоит использовать trace id, который будет связан с одним запросом на протяжении всего его времени жизни.
### Information Disclosure
|потенциальные проблемы|последствия|способы исправления|
|--|--|--|
|креды тестового пользователя, админа или API доступны в открытом виде|утечка информации|использование практик SAST для нахождения не удаленных секретов|
######
Вопрос разработчику: Используется ли шифрование для хранения данных? Как ограничивается доступ к конфиденциальной информации?
### Denial of Service
|потенциальные проблемы|последствия|способы исправления|
|--|--|--|
|заполнение всего дискогово пространства путем загрузки файла|отказ в обслуживании|установка ограничений на размер файла|
|вывод из строя сервера путем DDoS атаки|отказ в обслуживании|установка обратного прокси перед backend уровнем для балансировки нагрузки, установки WAF и скрытия настоящего IP сервера|
######
Вопрос разработчику: Какие методы защиты от DoS-атак используются (ограничение скорости запросов, фильтрация IP, использование каптчи)?
### Elevation of Privilege
|потенциальные проблемы|последствия|способы исправления|
|--|--|--|
|эксплуатация широко известной уязвимости из Интеренета|повышение привилегий|регулярное обновление системы|
|подделка содержимого JWT токена повысив права до администратора|повышение привилегий|использование современных библиотек для работы с JWT, проверка надежности подписи|
######
Вопрос к разработчику: Как происходит отслеживание и применение патчей безопасности? Как в сервисе организована система управления доступом на основе ролей?
