# Просмотр чатов тг-бота
ДИСКЛЕЙМЕР: это версия для использования на своем ПК, т.к. все ключи хранятся открыто в файле. Если вы разместите его на хостинг, то кто откроет его сможет прочитать ваши ключи.

## Особенности
1) все в одной html странице
2) запускается локально
3) можно не только просматривать но и отправлять сообщения пользователю от имени бота
3) смена темы дарк/лайт
4) звуковые уведомления вкл/откл
5) разноцветные "аватарки" через хэш-функцию

## Использование:
1) скачать
2) ввести свои ключи
3) запустить в браузере

## Перед использованием:
1) Вам нужна Supabase табличка chat_logs (подключить сервис бесплатно тут https://supabase.com/)
ВАЖНО! Бот который складывает сообщение в базу данных и которого вы подключаете к html, должен быть одним и тем же.

через SQL Editor вводим и Run
```sql
CREATE TABLE public.chat_logs (
    id SERIAL PRIMARY KEY NOT NULL,
    session_id TEXT NOT NULL,
    user_id TEXT NULL,
    username TEXT NULL,
    role TEXT NULL,
    content TEXT NULL,
    metadata JSONB NULL,
    created_at TIMESTAMP WITH TIME ZONE NULL DEFAULT NOW(),
    first_name TEXT NULL
);
```
2) В настройках таблички в Supabase -> Edit table -> включить Enable Realtime
(это нужно для того что мы подписываемся на ее обновления, и как только сообщения поступят в базу данных, к нам в приложение придет уведомление и обновится чат)

3) Изменить в admin.html:
```js
const BOT_TOKEN = 'XXXXXXXXX'; /// Получается в телеграм в https://t.me/BotFather
const SUPABASE_URL = 'https://database-name.supabase.co'; //получается в project settings вашей базы в supabase - раздел Data API
const SUPABASE_ANON_KEY = 'XXXXXXXXXXX'; //там же - раздел API KEYS
```



4) Настроить свою логику того как к вам приходят сообщения в эту базу данных 

Если у вас n8n, то можете сделать так, по узлу для сохранения сообщения от юзера и ответов от бота:

Это сразу после узла "Telegram Trigger" (можете сделать ctrl+c ctrl+v себе в воркфлоу - вставится как нода)
```js
{
  "nodes": [
    {
      "parameters": {
        "tableId": "chat_logs",
        "fieldsUi": {
          "fieldValues": [
            {
              "fieldId": "session_id",
              "fieldValue": "={{ 'user_' + $json.message.from.id }}"
            },
            {
              "fieldId": "user_id",
              "fieldValue": "={{ $json.user_id || $json.message.from.id }}"
            },
            {
              "fieldId": "username",
              "fieldValue": "={{ $json.username || $json.message.from.username }}"
            },
            {
              "fieldId": "role",
              "fieldValue": "user"
            },
            {
              "fieldId": "content",
              "fieldValue": "={{ $json.text || $json.message.text }}"
            },
            {
              "fieldId": "metadata",
              "fieldValue": "={{ JSON.stringify($json) }}"
            },
            {
              "fieldId": "first_name",
              "fieldValue": "={{ $json.first_name || $json.message.from.first_name }}"
            }
          ]
        }
      },
      "type": "n8n-nodes-base.supabase",
      "typeVersion": 1,
      "position": [
        144,
        -48
      ],
      "id": "",
      "name": "Save User Message",
      "credentials": {
        "supabaseApi": {
          "id": "",
          "name": ""
        }
      }
    }
  ],
	"connections": {
  },
  "pinData": {},
  "meta": {
    "templateCredsSetupCompleted": true,
    "instanceId": ""
  }
}
```

Это в самом конце после отправки пользователю сообщения от бота: 
```js
{
  "nodes": [
    {
      "parameters": {
        "tableId": "chat_logs",
        "fieldsUi": {
          "fieldValues": [
            {
              "fieldId": "session_id",
              "fieldValue": "={{ 'user_' + $json.message.from.id }}"
            },
            {
              "fieldId": "user_id",
              "fieldValue": "={{ $json.user_id || $json.message.from.id }}"
            },
            {
              "fieldId": "username",
              "fieldValue": "={{ $json.username || $json.message.from.username }}"
            },
            {
              "fieldId": "role",
              "fieldValue": "user"
            },
            {
              "fieldId": "content",
              "fieldValue": "={{ $json.text || $json.message.text }}"
            },
            {
              "fieldId": "metadata",
              "fieldValue": "={{ JSON.stringify($json) }}"
            },
            {
              "fieldId": "first_name",
              "fieldValue": "={{ $json.first_name || $json.message.from.first_name }}"
            }
          ]
        }
      },
      "type": "n8n-nodes-base.supabase",
      "typeVersion": 1,
      "position": [
        144,
        -48
      ],
      "id": "",
      "name": "Save Bot Message",
      "credentials": {
        "supabaseApi": {
          "id": "",
          "name": ""
        }
      }
    }
  ],
  "connections": {
  },
  "pinData": {},
  "meta": {
    "templateCredsSetupCompleted": true,
    "instanceId": ""
  }
}
```