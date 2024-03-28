# AppSecCloudCamp
Тестовое задание для отбора на стажировку Cloud.ru

## 1. Ответы на вопросы для разогрева.
#### 1. Расскажите, с какими задачами в направлении безопасной разработки вы сталкивались?
Ещё не представилось возможным принять участие в полном SDLC-цикле, но есть практический опыт безопасной разработки веб-приложения. Задание заключалось в разработке одного веб-приложения в двух версиях: уязвимой и защищенной, где первая содержала следующие уязвимости: XSS, IDOR, SQLI, OS command injection, Path Traversal, Brute Force.
#### 2. Если вам приходилось проводить security code review или моделирование угроз, расскажите, как это было?
Security code review не приходилось проводить. Моделирование угроз проводилось в контексте практического задания по одной из дисциплин в Вузе. Задача заключалась в определении негативных последствий от реализации УБИ, возможных объектов воздействия УБИ, источников УБИ, проведение оценки возможностей нарушителей по реализации УБИ, оценки способов реализации УБИ, оценки сценариев реализации УБИ в системах и сетят. Выполнение задания основовалось на нахождении нужной информации в методическом документе ФСТЭК с последующей привязкой этой информации к организации и ее инфраструктуре. 
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
В данном коде присутствует SQL injection уязвимость. Она представлена в виде строки ```query := fmt.Sprintf("SELECT * FROM products WHERE name LIKE '%%%s%%'", searchQuery)``` (строка 38). Содержимое *searchQuery* параметра является контролируемым пользователем и происходит его конкатенация в строку с запросом без какой-либо валидации содержимого, что не есть хорошо, так как успешная атака приведет к несанкционированному доступу к конфиденциальным данным.  
Способы исправления уязвимости:
- Prepared statement  
Для исправления уязвимости следует использовать prepared statement, который позволяет отделить запрос от параметра (параметризация). Сначала происходит сохранение и предкомпиляция запроса без фактического значения параметра, и введенные пользователем данные никак не могут повлиять на логику запроса.  
- Whitelisting и blacklisting  
Помимо этого, можно использовать белые и черные списки, которые будут отвечать за валидацию введенных пользователем данных. В данной ситуации происходит поиск всех продуктов, которые содержат в себе введенную пользователем подстроку, т.е. обращение к данным, которые часто обновляются, поэтому стоит использовать черный список. Белый список можно применять в ситуации, когда, например, пользовательские данные помещаются в поле с именем таблицы в SQL запросе.  
######
По моему мнению prepared statement лучший способ из этих двух, так как он прост в реализации, не зависит от человеческого фактора (нет возможности забыть внести что-либо) и не зависит от релиза  (не нужно следить за новыми выпусками реляционных систем баз данных, чтобы внести нововведения в список).
