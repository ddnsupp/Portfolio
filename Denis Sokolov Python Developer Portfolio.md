# Python Developer

## Контакты
- [Linkedin](https://www.linkedin.com/in/ddnsupp/)
- [Headhunter](https://hh.ru/resume/d983a6a9ff0cbfc8200039ed1f594d44484d4e)
- [Telegram](https://t.me/ddnsupp)
- [GitHub](https://github.com/ddnsupp)
## Проекты
- [Агрегатор скраперов новостей ](#news-aggregator)
- [Телеграм бот для авто генерации и продажи ключей VPN по подписке ](#телеграм-бот-для-авто-генерации-и-продажи-ключей-vpn-по-подписке)
- [Телеграм бот для продажи чая (тестовое задание с примером кода в репозитории)]()

## Агрегатор скраперов новостей

Локально запускаемая программа для сбора текстов статей с новостных сайтов и tg каналов с веб интерфейсом на Streamlit отвечающем за выбор режимов (сбор новостей за период времени \ в реальном времени + по ключевому слову \ без него), с возможностью включить \ выключить любой из 30 источников сбора информации. Интерфейс слушает локальный сервер на FastAPI который собирает данные о выбранных источниках, инициализирует инстансы скраперов \ парсеров с заданными параметрами. Сами скраперы (сайты новостей) и парсеры (tg каналы) написаны в парадигме ООП и используют под капотом Playwright (для захода на сайт, обхода блокировок и сбора списка ссылок на статьи, которые соответствуют параметрам поиска (находятся в выдаче)) + Aiohttp + Trafilatura (собирают и очищают тексты статей для сохранения их в txt файлы, xlsx и базу данных (sqlite)). Изначально программа написана на процессах + потоках, но практически переписана в асинхронный вид.
### Технологии
- Python
- Streamlit (Веб интерфейс по требованию заказчика)
- FastAPI (Веб сервер чтобы слушать интерфейс и инициализировать инстансы)
- Playwright (Переход по основной ссылке поиска с выбранными параметрами)
- Aiohttp (Переход по ссылкам статей для сбора и последующей очистки текстов)
- Trafilatura (Переход по ссылкам статей для сбора и последующей очистки текстов)
- Sqlite (Легковесная база для недопущения дублирования скачанных статей)
- Logging (Система логирования настроена в 2 контура - общая по программе и на случай отказа proxy чтобы их можно было поменять. По требованию заказчика часть критических сообщений об ошибках отправляется ему через tg бота)
- Telegram API (легковесный бот находящийся на том же FastAPI, принимающий сообщение лога (если этот параметр выставлен) и отправляет его в tg бота)
- Threading (переписывается в полный asyncio)

## Телеграм бот для авто-генерации и продажи ключей VPN по подписке

Телеграм бот на Python Aiogram с админ панелью на Django, Celery beat для учета подписок пользователей (заказчиком выбрана рекуррентная модель платежей через tinkoff), с базой на PostgreSQL в которой хранятся в том числе и данные о сопряженных серверах на которых развернуты VPN протоколы: openvpn, outline, vless, wireguard. Для развертки впн используются официальные образы (сформирована одна общая консольная команда для ubuntu 22 по их развертке) после введения которой сервер добавляется вручную в базу данных (прописывая какие протоколы установлены). 

На основании информации в БД, бот учитывает сколько сейчас клиентов на каждом сервере впн и какие на нем доступны протоколы и исходя из этого может обратиться через ssh к vpn серверу и создать \ удалить клиент пользователю, если платеж прошел \ не прошел и истекло время подписки. Заказчиком выбраны reply клавиатуры. 

Структура бота предполагает частое изменение и расширение, поэтому написана управляющий скрипт регистрирующий обработчики при старте бота на основании json структуры содержащей тексты, клавиатуры и вложенность. Непосредственно в python коде прописаны только те обработчики которые работают с бизнес логикой (проверка оплаты, создание \ удаление VPN клиентов пользователя). 
### Технологии
- Python
- Aiogram (Бот на Reply клавиатурах, но основная часть текстовых обработчиков вынесена в json структуру, которая обходится в глубину при старте бота и регистрирует обработчики т.к. клиент скорее всего будет часто менять тексты)
- Django (4.2 без кастомизации, модели совпадают с aiogram)
- PostgreSQL (и в Django и в Aiogram используется Django orm)
- Redis (очередь для передачи данных о подписках пользователей)
- Celery beat (учет запланированных событий отключения VPN за неуплату \ выставления следующего рекуррентного платежа в Tinkoff API)
- Linux (Ubuntu 20\22 и на сервере бота-магазина и на сервере VPN протоколов)
- SSH (paramiko - обращение к серверам VPN для создания \ удаления протоколов \ запросов нагрузки сервера \ данных об учете клиентов)
- Docker (контейнеры Aiogram, Django, Postgres, Redis, Celery beat)
- CI/CD Github Actions (контейнеры сразу разворачиваются на сервере, чтобы ускорить разработку)
## Примеры кода из этого проекта

### Пример управляющей структуры для регистрации обработчиков
```
[  
  {  
  "id": "1",  
  "command": [],  
  "text": "Заглушка для основного меню",  
  "keyboard": [],  
  "nested": [  

{  
  "id": "1.2",  
  "command": ["2) Личный кабинет", "Личный кабинет"],  
  "text": "Это текст для экрана личный кабинет",  
  "keyboard": [  
    "Управление подпиской",  
    "Партнерская программа",  
    "Главное меню",  
    "Назад"  
  ],  
  "nested": [  
    {  
      "id": "1.2.1",  
      "command": ["Партнерская программа"],  
      "text": "Это текст для 2) Личный кабинет",  
      "keyboard": [  
        "Создать партнерскую ссылку",  
        "Запросить выплату",  
        "Главное меню"  
      ],  
      "checks": [  
      {  
        "function": "check_subscription_active",  
        "error_text": "Для доступа к этой функции необходима активная подписка.",  
        "error_keyboard": ["Я ознакомился и принимаю условия партнерской программы", "Главное меню"]  
      }  
```

### Функция регистрации обработчиков
```
def register_handlers(items, parent_command=None):  
    for item in items:  
        try:  
            commands = item['command']  
            text = item['text']  
            buttons = item['keyboard']  
            message_id = item['id']  
            nested = item.get('nested', [])  
            checks = item.get('checks', [])  
  
            for command in commands:  
                handler = partial(command_handler, cmd=command, txt=text, btns=buttons, state=message_id, checks=checks)  
                router.message.register(handler, F.text == command)  
  
            if nested:  
                register_handlers(nested)  
  
        except Exception as e:  
            log_message(  
                log_type='error',  
                message=f'Ошибка при регистрации обработчиков: {e}',  
                traceback_info=traceback.format_exc(),  
            )
```

### Функция создания обработчика на основании Json структуры 
```
async def command_handler(msg: types.Message, cmd, txt, btns, state, checks=None):  
    user_id = msg.from_user.id  
    log_message(  
        'info',  
        f'Параметры вызова: User ID={user_id}, Command={cmd}, State={state}',  
        traceback.format_exc()  
    )  
  
    if checks:  
        for check in checks:  
            check_func = globals().get(check['function'])  
            if check_func and not await check_func(user_id):  
                error_kb = await create_keyboard(check['error_keyboard'])  
                await msg.answer(check['error_text'], reply_markup=error_kb)  
                return  
  
    try:  
        await VPNUser.set_user_state(telegram_id=user_id, state=state)  
        keyboard = await create_keyboard(btns)  
        await msg.answer(txt, reply_markup=keyboard)  
    except Exception as e:  
        log_message(  
            'error',  
            f'Ошибка команды {cmd} у пользователя {user_id}: {e}',  
            traceback.format_exc()  
        )  
```

### Пример метода создания VPN ключа Openvpn на удаленном сервере

```
def create_key(self, client_name='adminclient6', *args, **kwargs):  
    client = paramiko.SSHClient()  
    client.set_missing_host_key_policy(paramiko.AutoAddPolicy())  
    try:  
        client.connect(
        self.server_ip, 
        username=self.username, 
        password=self.password
        )  
        
        shell = client.invoke_shell()  
        time.sleep(1)  
   
        create_command = f"""  
        expect -c 'set timeout -1;        
        spawn sudo bash /vpn_server/openvpn-install.sh;   
		expect "Select an option" {{send "1\\r"}};   
		expect "Provide a name for the client" {{send "{client_name}\\r"}};  
        expect eof;'        """        shell.send(create_command)  
        time.sleep(5)  
        local_file_path = rf'/app/data/{client_name}.ovpn'  
        remote_file_path = f"/root/{client_name}.ovpn"  
  
        try:  
            sftp = client.open_sftp()  
            sftp.get(remote_file_path, local_file_path)  
            sftp.close()  
            return {  
                "type": "file",  
                "value": local_file_path,  
                "client_name": client_name  
            }  
        except Exception as e:  
            return {  
                "type": "error",  
                "value": f"Ошибка при скачивании файла конфигурации для '{client_name}': {e}",  
                "client_name": client_name  
            }  
  
    finally:  
        client.close()
```

## Телеграм бот для продажи чая (тестовое задание с примером кода в репозитории)

Телеграм бот на Python Aiogram с админ панелью на Django, с базой на PostgreSQL который был сделан за 3 дня в качестве тестового вступительного задания для работодателя в связи с чем могу показать код в открытом репозитории. 

По тестовому заданию должна была быть реализована пагинация товаров, их добавление и удаление из корзины, тестовая оплата через интегрированную с Botfather платежными системами.

Проект написан быстро под срочную задачу и в последующем не правился т.к. не будет использован для бизнес задач.

[GitHub репозиторий бота](https://github.com/ddnsupp/Test_bot_for_Galchenko_T_V/tree/main/aiogram_app)
### Технологии
- Python
- Aiogram
- Django
- PostgreSQL
- SQLAlchemy + Django ORM
- Тестовое API оплаты
