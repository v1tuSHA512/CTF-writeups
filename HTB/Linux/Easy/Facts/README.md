Главная страница машины 
<img width="2319" height="1221" alt="image" src="https://github.com/user-attachments/assets/5d4e46dc-600e-4a0b-8f5b-be7e1a37fb32" />

При переборе директорий меня встретил следующий endpoint
<img width="918" height="978" alt="Снимок экрана 2026-05-04 231734" src="https://github.com/user-attachments/assets/fd56edae-dbc0-4c1e-9bba-bfe349501b3c" />

Т.к сайт позволяет зарегистрировать нового пользователя я этим воспользуюсь и создам своего
<img width="2360" height="1049" alt="Снимок экрана 2026-05-04 231849" src="https://github.com/user-attachments/assets/1958f3f7-d915-4fb4-aee8-682135a9c6b4" />

Хоть на скрине этого и не видно, но это панель Camaleon CMS 2.9.0.
После небольшого ресерча я нашел 2 уязвимости в этой CMS:
  1. CVE-2025-2304 (https://nvd.nist.gov/vuln/detail/CVE-2025-2304) - уязвимость mass assignment, которая позволяет включить параметр password[role]=admin и тем самым эскалировать привилегии в CMS;
  2. CVE-2026-1776 (https://nvd.nist.gov/vuln/detail/CVE-2026-1776) - уязвимость заключается в функции download_private_file, когда приложение настроено на использование CamaleonCmsAwsUploader, т.к AWS uploader не проверяет пути к файлам с помощью valid_folder_path. В итоге, злоумышленник может отправить запрос на URL вида http://TARGET/admin/media/download_private_file?file=PAYLOAD и скачать произвольный файл с сервера

Зная все это первым делом необходимо повысить привилегии пользователя, чтобы получить права админа и использовать их для эксплуатации CVE-2026-1776.

Отправляю запрос на обновление пароля моего пользователя + указываю параметр password[role]=admin, чтобы эскалировать привилегии
<img width="1977" height="1105" alt="Снимок экрана 2026-05-04 232134" src="https://github.com/user-attachments/assets/2224b86f-4012-4884-a7aa-43df0a9cf973" />

Получаю права админа
<img width="2356" height="952" alt="Снимок экрана 2026-05-04 232158" src="https://github.com/user-attachments/assets/3c87f91b-967e-4139-9106-67f207e744f9" />

Зная про вторую уязвимость сразу отправляю запрос с указанием пути к файлу /etc/hosts
<img width="2273" height="705" alt="Снимок экрана 2026-05-05 001720" src="https://github.com/user-attachments/assets/86a56d36-b8f1-4f34-ad9b-4cba65abd90f" />

Отлично! Уязвимость есть. Теперь необходимо получить удаленный доступ к серверу.

После тщетных попыток получить удаленный доступ через log poisoning/загрузку обратного шела на сервер, я предположил что на сервере должны быть пользователи с доступом к серверу по ssh (с использованием закрытого ключа)

Если прочитать файл /etc/passwd, то можно обнаружить 3 пользователя на сервере
<img width="1961" height="970" alt="image" src="https://github.com/user-attachments/assets/e8b67d7e-505c-4682-bdb9-b822d774a825" />

После попыток получить закрытый ключ SSH одного из пользователей, я обнаружил что только у пользователя trivia есть закрытый ключ
<img width="2312" height="708" alt="Снимок экрана 2026-05-05 001746" src="https://github.com/user-attachments/assets/b16751a7-82f7-4ac9-b486-5a01d3dbd8aa" />

Оказалось что закрытый ключ защищен кодовой фразой, поэтому в ход придется подключить john the ripper для брутфорса фразы.
Преобразовав ключ в hash с помощью утилиты ssh2john, начинаю брутфорс. Подождав немного john подобрал кодовую фразу
<img width="1099" height="320" alt="Снимок экрана 2026-05-05 002009" src="https://github.com/user-attachments/assets/f015b5e3-f54b-4ada-933b-78dde495da0b" />

Пробую подключиться по SSH к серверу от пользователя trivia...Успех!
Сразу проверяю права sudo
<img width="1354" height="159" alt="Снимок экрана 2026-05-05 004852" src="https://github.com/user-attachments/assets/e0f28a00-569c-4dbe-8414-4d575bef3d12" />

trivia может запускать скрипт /usr/bin/facter от sudo. Попробую прочитать этот скрипт, чтобы понять что он из себя представляет
<img width="703" height="250" alt="Снимок экрана 2026-05-05 004945" src="https://github.com/user-attachments/assets/43b40ef9-56fa-42a2-8ef8-73e146d3d6ef" />

Оказывается это скрипт на ruby, который запускает framework facter через cli.
Т.к тут ограничений нет и утилита запускается с правами sudo, то можно написать свой "факт", который прочитает содержимое /root/root.txt
<img width="748" height="211" alt="Снимок экрана 2026-05-05 005000" src="https://github.com/user-attachments/assets/4256ce60-54b9-4ba2-9580-2a3ecb1aa122" />

После запуска утилиты со своим exploit получаю флаг root'а
<img width="818" height="65" alt="Снимок экрана 2026-05-05 005014" src="https://github.com/user-attachments/assets/dad00cf6-b7ce-4c0e-a04e-26f1e1a82a3d" />

Забыл вставить, но флаг пользователя получил на этапе брута кодовой фразы к ключу ssh
<img width="2342" height="669" alt="Снимок экрана 2026-05-05 001832" src="https://github.com/user-attachments/assets/833062bb-f63c-40e8-b1ba-757b5592dd0f" />
