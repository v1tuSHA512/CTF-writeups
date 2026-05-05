Первое что вижу при переходе на сайт:
<img width="1297" height="725" alt="image" src="https://github.com/user-attachments/assets/4af8fed8-070d-40fb-8fc6-93e3b3e273be" />

В глаза сразу бросается параметр page. Если в него попробовать вставить произвольную строку, то сервер отвечает ошибкой:
<img width="1486" height="660" alt="image" src="https://github.com/user-attachments/assets/d459a122-2bf2-4251-9ebd-1f35976a9183" />

Изучив ответ сервера видно что к имени файла добавляется расширение .inc.php. Его можно обойти благодаря атаки NULL byte poisoning написав в конце запроса %00 что позволяет отсечь конец строки (.inc.php):
<img width="1503" height="616" alt="image" src="https://github.com/user-attachments/assets/b3d10d8e-5aa5-4460-8c7d-2f3d7ee9bc65" />

Тут получаю уже другую ошибку, которая говорит о том, что используется политика ограничения доступа open_basedir, которая запрещает листинг за пределами текущей директории.
Это можно обойти использовав один из популярных php wrapper - php://filter/convert.base64-encode:
<img width="1501" height="606" alt="image" src="https://github.com/user-attachments/assets/f1492936-cbb0-4068-bda3-3cf5df8276cd" />

Тут уже получаю данные файла, где вижу как называется файл с флагом (.passwd). 
(Скрин с декодингом bs64 пропущен)

Читаю флаг:
<img width="1498" height="744" alt="image" src="https://github.com/user-attachments/assets/f64add01-6f9e-48c9-82f8-b7a86297c9f3" />
