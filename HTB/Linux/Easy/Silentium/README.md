<img width="1209" height="576" alt="image" src="https://github.com/user-attachments/assets/543873c6-6b0b-4919-8df0-f4f1d3f982fb" />

Первым делым сканирую хост на открытые порты и... ничего особенного: открыты дефолтные 80 и 22 порты
<img width="1559" height="578" alt="image" src="https://github.com/user-attachments/assets/090deeaf-7cd2-4aa8-b3c2-1eb1b6d9547b" />

Перебрав все возможные директории и файлы ничего не нахожу :( Однако решаю просканировать на наличие vhost через FFUF:
<img width="1509" height="724" alt="image" src="https://github.com/user-attachments/assets/2cac65c2-90ba-47f5-ac17-6cdb98350c27" />

FFUF нашел что на машине есть vhost staging

При переходе на staging.silentium.htb меня встречает форма авторизации
<img width="2331" height="1083" alt="image" src="https://github.com/user-attachments/assets/19c97057-aa30-4dc2-adb6-97131bdb5ec4" />

Иходя из названия заголовка веб-страницы понимаю что это Flowise AI.
Потыкав на разные кнопки формы понимаю что формы регистрации нет, а только возможность авторизироваться/сбросить пароль.
Т.к больше мне ничего не доступно я стал изучать документацию к API Flowise, чтобы попытаться продвинуться дальше.
Мое внимание привлек endpoint /api/v1/version, который раскрывает версию Flowise (в моем случае версия - 3.0.5)
<img width="977" height="271" alt="image" src="https://github.com/user-attachments/assets/bcd7a018-4ec2-4106-aa53-4d1bb77feef6" />

Погуглив информацию про это версию наткнулся на 2 критичные уязвимости (CVE-2025-58434, CVE-2025-59528), которые я затрону по порядку.

CVE-2025-58434 - ATO через раскрытие tempToken при сбросе пароля через forgot-password endpoint. Более детально можно почитать здесь -> https://github.com/FlowiseAI/Flowise/security/advisories/GHSA-wgpv-6j63-x5ph 
Поскольку эксплуатация уязвимости требует знание почты жертвы, возвращаюсь на silentium.htb, где перечислен персонал компании. Тут на основе интуиции решаю выбрать в качестве жертвы сотрудника с именем Ben, что оказывается верным.

Теперь отправив запрос на /api/v1/account/forgot-password с валидным email, сервер в своем ответе раскроет tempToken, через который можно будет сменить пароль для Ben на мой
<img width="1896" height="837" alt="image" src="https://github.com/user-attachments/assets/6c11df10-64a6-43e4-93e6-45de8a6d836f" />

Получив токен отправляю запрос на /api/v1/account/reset-password указав в качетсве email - ben@silentium.htb, токен - tempToken из шага выше, пароль - пароль, который задам сам
<img width="1964" height="822" alt="image" src="https://github.com/user-attachments/assets/b02128b4-3853-4297-9def-bcb91fafff6f" />

В результате смены пароля успешно авторизируюсь как Ben(admin)
<img width="2356" height="1012" alt="image" src="https://github.com/user-attachments/assets/5559cb31-4c76-4c60-ae23-d9b8d84dd5f7" />

Вторая уязвимость дает RCE (CVE-2025-59528) из-за того что параметр mcpServerConfig при передаче на endpoint /api/v1/node-load-method/customMCP никак не фильтруется. Если злоумышленик внедрит туда произвольный JS-код, то сервер его выполнит, так еще и "Since this runs with full Node.js runtime privileges, it can access dangerous modules such as child_process and fs" 

Отправив соответствующий вредоносный запрос на /api/v1/node-load-method/customMCP (https://github.com/FlowiseAI/Flowise/security/advisories/GHSA-3gcm-f6qx-ff7p) для получение reverse shell
<img width="2320" height="934" alt="image" src="https://github.com/user-attachments/assets/63e3f9b3-a7c9-4d8b-a81a-94667ccecdf3" />

Тут меня встречает docker. Изначально думал что нужено будет совершить docker escape, поэтому потратил время на поиск различных точек реализации данной уязвимости.
Потратив на это какое-то время понимаю, что возможности сбежать из докера нет, поэтому решаю прочитать файл .ash_history у root пользователя.
В файле находилась история команд, где было видно вызов команды env. Решаю повторить и получаю следуюзий вывод:
<img width="789" height="628" alt="image" src="https://github.com/user-attachments/assets/cff8bc20-1d06-4dc0-82a8-ce167b1b1b53" />

Заметив, что flowise user - это Ben решаю подключиться по ssh к машине с паролем flowise password, однако пароль оказывается неверным, пробую пароль smpt_password... Успех!
<img width="523" height="263" alt="image" src="https://github.com/user-attachments/assets/5b0a5ea8-ffdd-484e-bbc5-7186faf97b37" />

Получаю user флаг
<img width="351" height="61" alt="image" src="https://github.com/user-attachments/assets/176367d6-9e0f-43aa-ad99-c05f1c651626" />

Попробовав найти скрипты/утилиты, которые можно запустить от sudo, прочитав crontab, изучив права sudo Ben'а (их нет) решаю запустить linpeas и посмотреть что выдаст он.
Оказывается на хосте есть уязвимость copy_fail:
<img width="671" height="106" alt="image" src="https://github.com/user-attachments/assets/0e7f652d-7121-4e83-a832-ed4fae0c80be" />

Используя exploit для copy_fail получаю root
<img width="533" height="62" alt="image" src="https://github.com/user-attachments/assets/2da1dc98-fcdc-4edd-9f48-6d588941774b" />

Получаю доступ к root флагу
<img width="771" height="292" alt="image" src="https://github.com/user-attachments/assets/9cdfbe3a-a321-4591-b820-e966f1bda9f4" />

**P.S:** почитав writeup других людей после решения оказалось я повысил привелегии не той уязвимостю (https://dev.to/lewisawe/ctf-writeup-silentium-htb-536i). Предпологалось, что я буду эксплуатировать уязвимость CVE-2025-8110 в локальном Gogs.
Похоже из-за того что copy_fail вышел недавно, админы HTB не успели устранить эту уязвимость на серверах 
