По классике начинаю со сканирования открытых портов утилитой Nmap 

sudo nmap -sS --min-hostgroup 4096 --max-hostgroup 65535 --max-retries 2 --max-rtt-timeout 500ms --min-rtt-timeout 50ms --initial-rtt-timeout 350ms --host-timeout 15m --min-rate 5000 --max-rate 10000 --open --stats-every 1m -R -Pn -p- -sC -sV rawmatex.standalone.stf
<img width="1167" height="681" alt="image" src="https://github.com/user-attachments/assets/1cba64ac-59c1-4b58-9830-6d00021e263d" />

Кроме 22 и 80 ничего нет, поэтому сразу перехожу к веб-серверу на 80-ом порту.

Сразу же встретила форма авторизации без возможности авторизоваться. Перебрав директории/файлы я ничего не нашел, поэтому попробовал авторизоваться с кредами admin:admin.
Результата не получил, однако решил попробовать реализовать SQL injection. Оказалось, что приложение уязвимо к этому типу уязвимостей.
<img width="2382" height="1051" alt="image" src="https://github.com/user-attachments/assets/00fb5943-3466-4d6c-9ab8-b9234a43ce03" />

Определив, что это Blined time based инъекция, я запускаю тулзу sqlmap для автоматизации процесса эксплуатации SQLi.
Команда - sqlmap -l rawmatex_burp --batch --level 5 --risk 3 --dbms PostgreSQL -p email --dbs
<img width="1167" height="1131" alt="image" src="https://github.com/user-attachments/assets/90683f83-cbaf-4584-9ca2-65f89b8935c5" />

sqlmap подтвердил наличие уязвимости - это означает, что можно идти дальше и начать перечислять БД, таблицы, колонки.
В ходе перебора я нашел одну и единственную БД - public, которая содержала, следующие таблицы:
  1. customers;
  2. documents;
  3. materials;
  4. orders;
  5. secret;
  6. shipments;
  7. users.

Извлекаю flag из таблицы secret и сдаю его
<img width="672" height="305" alt="image" src="https://github.com/user-attachments/assets/2966e255-0981-43ab-9eb7-71a0d66fbdb3" />

<img width="1503" height="787" alt="image" src="https://github.com/user-attachments/assets/7b04ea37-d75e-45a6-9cfa-c26c1a98caa1" />

После получения флага, я решил изучить содержимое users, где было 2 пользователя: homer и guts + плюс хеши их паролей.

Сразу запускаю hashcat для получения паролей - hashcat -m 3200 {hash} /usr/share/wordlists/rockyou.txt
<img width="1160" height="621" alt="image" src="https://github.com/user-attachments/assets/dafc5aed-bd2c-4244-bd73-1b9aa9711063" />

После N-ого кол-ва времени получаю пароли для пользователей: homer и guts
<img width="759" height="212" alt="image" src="https://github.com/user-attachments/assets/5d9175d6-c26e-4582-b166-b06297ca18d2" />

Использую креды homer прохожу авторизацию.
<img width="2340" height="1255" alt="image" src="https://github.com/user-attachments/assets/3dd93dae-4e75-4e6a-9f0b-614312455e49" />

Нахожу файл - export_contract_CN_2025_03_12.md -> скачиваю -> читаю -> получаю флаг за КС 
<img width="2356" height="1057" alt="image" src="https://github.com/user-attachments/assets/0885eb97-7c53-4f70-9564-bbc78dafd022" />

<img width="1574" height="629" alt="image" src="https://github.com/user-attachments/assets/6be93f97-c645-44e8-a06e-77cf230cc9a3" />
