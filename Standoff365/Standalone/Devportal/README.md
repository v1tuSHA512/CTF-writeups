Первым делом нужно просканировать порты, чтобы лишний раз не упустить какой-нибудь веб-сервис.

Сканирую порты тулзой nmap, а именно - "sudo nmap -sS --min-hostgroup 4096 --max-hostgroup 65535 --max-retries 2 --max-rtt-timeout 500ms --min-rtt-timeout 50ms --initial-rtt-timeout 350ms --host-timeout 15m --min-rate 5000 --max-rate 10000 --open --stats-every 1m -R -Pn -p- -sC -sV {Host}"

Результат команды выше показал следующие открытые порты на devportal.standalone.stf:
<img width="1237" height="678" alt="image" src="https://github.com/user-attachments/assets/79792196-e8d7-4f77-be6c-52a58350db5d" />

  1. 25 порт - ничего интересного;
  2. 80 - на этом порту запущен GitLab (к нему я вернусь чуть позже);
  3. 8000 - на этом порту был запущен SimpleHTTPServer 0.6 (Python 3.9.2). Который хостил EnterpriseConnect_v2.0.1.cpython-39.pyc файл (.pyc — это скомпилированный файл исходного кода на языке Python. Содержит низкоуровневый байт-код).
    Изучив файл, я ничего не обнаружил, поэтому продолжил изучать дальше;
  4. 8060 - метрики prometheus, но с 403 статус кодом. Идем дальше...;
  5. 8084 - однострочник, на которым был статичный index.html;
    <img width="2365" height="1192" alt="image" src="https://github.com/user-attachments/assets/de384a2e-f01b-41cf-a87c-801c7fa73a69" />
  
  6. 9094 - порт открыт, однако за ним ничего нет.

Изучив все порты, ничего не остается как вернуться к GitLab и продолжить исследовать его.
<img width="2062" height="817" alt="image" src="https://github.com/user-attachments/assets/aa1ff11f-db09-4364-a49e-28d192649f50" />

Первым делом я сразу же попробовал создать юзера и авторизоваться под ним, однако сайт отдавал ошибку о том, что мой юзер не прошел верификацию от админа.

После этого я увидел, что неавторизованный пользователь может просматривать всякие инструкции к гитлабу, публичные репы (их не было) и другую паблик инфу.

Изучая всю публичную информацию я решил перейти в раздел "What's new", где были описаны фичи текущей версии гита как и сама версия (13.10.2).
Погуглив уязвимости этой версии сразу же наткнулся на такую - CVE-2021-22205 (https://nvd.nist.gov/vuln/detail/CVE-2021-22205) + PoC (https://www.exploit-db.com/exploits/50532).

Послу изучения уязвимости оказалось, что уязвимость не в самом GitLab, а в ExifTool (https://ine.com/blog/exiftool-command-injection-cve-2021-22204-exploitation-and-prevention-strategies), которым пользуется GitLab для парсинга файлов.
Более подробно можно почитать перейдя по ссылкам.

После прочтения материалов приступаю к эксплуатации уязвимости. В эксплоите поменял только название файла и IP с портом.
В PoC'е сказано что можно отправить вредонос на любой endpoint сайта (даже на несуществующий), поэтому делаю все в точности:
<img width="1152" height="1235" alt="image" src="https://github.com/user-attachments/assets/dbb3d00b-6275-404c-88cf-06f0a4bd02be" />

Потратив некоторое время на изучения данных на сервере, дохожу до /home директории, где меня ждал RCE flag:
<img width="1021" height="306" alt="image" src="https://github.com/user-attachments/assets/68f61862-c314-412d-93b5-d82ed4c55a02" />

Так же, перейдя в директорию gitlab нашел там тот же файл что и раньше EnterpriseConnect_v2.0.1.cpython-39.pyc, однако уже с флагом внутри:
<img width="1166" height="318" alt="image" src="https://github.com/user-attachments/assets/353a241b-b2dd-4189-8bea-391054459a2b" />

Машина решена!
<img width="1630" height="595" alt="Снимок экрана 2026-05-16 195905" src="https://github.com/user-attachments/assets/43b5580e-32cc-4f94-bfc5-57a19cabcf54" />
