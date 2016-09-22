# Amazon Web Service + Jupyter Notebook (sklearn, xgboost, vowpal wabbit, …)

## Зачем это нужно?
Когда вам надоедает оставлять вычисления на ночь и слушать вентиляторы вашего ноутбука и хочется чего-то по-настоящему мощного… 
Тогда можно переложить труд на сервера AWS. А как это сделать проще всего я напишу ниже. 
Скрипты и материал взяты, исправлены и дополнены из следующих мест:
- [Англоязычная статья с кучей картинок.](https://gist.github.com/iamatypeofwalrus/5183133)
- [Русскоязычный вольный перевод на Хабрахабре.](https://habrahabr.ru/post/280562/)

## Что мы сделаем?
- Выберем необходимую по мощности машину и запустим её. 
- Установим некоторые необходимые библиотеки.
- Запустим всем любимый IPython Notebook.

## Приступим.
### Часть 1. 
1. Регистрируемся на [AWS](https://aws.amazon.com/ru/). Необходимо указать номер платёжной карты (все действия после этого становятся на порядок осторожнее). Выбирайте пароль посложнее и берегите аккаунт, чтобы не повторить [опыта других людей](https://geektimes.ru/post/247794/).
2. Заходим в Services, выбираем EC2. Instances для машин по требованию с фиксированной ценой и Spot Requests для машин с плавающей ценой по спросу на них (нужно балансировать между выгодной стоимостью и вероятностью того, что вас отключат при превышении текущей ценой вашей). Дальше нажимаем на соответствующую синюю кнопку. 
3. Выбираем операционную систему Ubuntu (можно и другое по надобности). На этом этапе можно также выбрать готовые образы от компаний и других людей (в том числе есть и с установленными библиотеками машинного обучения). 
4. Настало время выбрать машину нужной мощности. Подробней о них можно прочитать [здесь](https://aws.amazon.com/ru/ec2/instance-types/), а [здесь](https://aws.amazon.com/ru/ec2/spot/pricing/) текущие цены на спотовые инстансы. Для наших задач стоит использовать группы M4 и C4. 
5. На третьем этапе настроек при выборе спотового инстанса требуется указать цену (ориентировочно раза в полтора-два выше текущей цены). Оплачивать придётся не выбранную цену, а всего лишь текущую (но которая может заметно меняться), не превышающую заданную. В разных регионах цены отличаются друг от друга. На мой взгляд в Орегоне они несколько ниже цен остальных регионов. 
6. Далее выбирается размер основного диска и подключаются дополнительные разделы. Стоит увеличить по надобности, уменьшить не получится. Если данные будут необходимы при дальнейших запусков, то будет нужно убрать галочку с Delete on Termination. Однако тогда на вас будет «висеть» этот раздел, который будет требовать отдельной оплаты. 
7. Задаём имя для удобства (всего лишь для отображения в списке инстансов). 
8. Теперь один из самых важных этапов настройки. Необходимо создать группу безопасности с 4 пунктами: SSH, HTTP, HTTPS и Custom TCP Rule с портом 8888 (по привычке). С точки зрения безопасности стоит разрешить доступ только со своего IP-адреса. В дальнейшем можно использовать уже созданные настройки. 
9. В завершении необходимо создать ключ *.pem для доступа к удалённой машине и скачать себе. Вот как раз его и стоит беречь от всех.
10. Через некоторое время машина запустится. Щёлкнув на неё можно будет увидеть внизу Public DNS. Его необходимо скопировать.

### Часть 2.
11. Запустим скрипт для установки на удалённой машине необходимых пакетов (не забываем подставить название ключа и public DNS удалённой машины):
> ssh -i *.pem ubuntu@public_dns 'bash -s' < remote_setup.sh
12. Если всё прошло успешно, то можно запускать Jupyter Notebook. Для этого сначала подключаемся к машине (заодно прикидываем порты, чтобы открыть локально ноутбук):
  > ssh -i *.pem -L localhost:8888:localhost:8888 ubuntu@public_dns
17. Для того, чтобы при внезапном отключении связи с машиной, у нас не падал ноутбук, будем запускать Jupyter в сеансе screen (если просто, то терминал в терминале):
> screen -S jupyter
18. Запускаем Jupyter:
  > jupyter notebook --no-browser
19. Если не возникло никаких проблем, то вы должны увидеть привычные строки при открытии ноутбука. Теперь можно открывать браузер и проходить по адресу:
  > localhost:8888
20. Чтобы выйти из текущего сеанса screen нажмите cntr+a, а потом d (detach). Чтобы подключиться обратно к созданному ранее сеансу следует ввести:
> screen -r jupyter
21. Для отправки данных с машины или на машину используйте следующую команду:
  > scp -i *.pem ubuntu@public_dns:~/file pwd_to_copy/
Например, отравим тестовый файл для проверки библиотек:
> scp -i *.pem example.ipynb ubuntu@public_dns:~/
Теперь мы должны увидеть его в Jupyter.

### Часть 3.
1. А теперь делаем то, для чего всё и создавали. 
2. И самое важное: не забыть при завершении отключить машину на странице с запущенными инстансами (нажать именно на terminate, а не stop).

  > EC2 -> Instances -> Actions -> Instance State -> Terminate

## Что ещё? 
1. Можно добавить в файл remote_setup.sh дополнительные пакеты для установки, убрать не всегда нужный XGBoost и так далее. 
2. Если вычислительные мощности нужны довольно часто, то можно не настраивать машину в каждый раз, а сохранять её диск для дальнейшего использования (за дополнительную плату).
3. Передавать файлы через scp на машину в далёком регионе может быть слишком долго, поэтому можно воспользоваться wget, если есть прямая ссылка на файлы (например, для kaggle.com понадобится ещё cookies). Свои файлы можно загружать и из Google Drive, однако для этого потребуется консольный клиент (я пользуюсь [этим](https://github.com/prasmussen/gdrive), но для него потребуется компилятор Go). Так скорость загрузка возрастёт на порядок.

UPD: я удалил сложную вторую часть с настройкой пароля для ноутбука (зато не надо было иметь доступа по ssh к машине после запуска jupyter). Если кому и понадобится, то можно посмотреть историю файлов этого репозитория. 
