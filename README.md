# Viewing Telegram Bot Chats
DISCLAIMER: This is a version for use on your own PC, as all keys are stored openly in the file. If you deploy it on hosting, anyone who accesses it will be able to read your keys.

![image](https://github.com/myslithell/images/blob/main/onehtml-tgbot.png)

**Read this in other languages:** [Русский](README-RU.md) | [简体中文](README-CN.md)

## Features
1) Everything in a single HTML page
2) Runs locally
3) Not only view but also send messages to users on behalf of the bot
4) Dark/light theme switching
5) Sound notifications on/off
6) Colorful "avatars" via hash function

## Usage:
1) Download [admin.html](admin.html)
2) Enter your keys
3) Run in browser

## Before Using:
1) You need a Supabase table chat_logs (connect the service for free here https://supabase.com/)
IMPORTANT! The bot that stores messages in the database and the one you connect to HTML must be the same.

Enter via SQL Editor and Run:
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
2) In the table settings in Supabase -> Edit table -> enable Enable Realtime
(This is needed so we can subscribe to its updates, and as soon as messages arrive in the database, we receive a notification in the application and the chat is updated)

3) Modify in admin.html:
```js
const BOT_TOKEN = 'XXXXXXXXX'; /// Obtained in Telegram at [https://t.me/BotFather](https://t.me/BotFather)
const SUPABASE_URL = 'https://database-name.supabase.co'; //obtained in project settings of your database in supabase - Data API section
const SUPABASE_ANON_KEY = 'XXXXXXXXXXX'; //same place - API KEYS section
```



4) Configure your own logic for how messages arrive in this database


## JUST FOR n8n
If you have n8n, you can do it this way, with a node for saving messages from the user and responses from the bot:

This is right after the "Telegram Trigger" node (you can copy-paste this into your workflow - it will be inserted as a node):
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

This is at the very end after sending a message to the user from the bot:
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

## 📬 Author
Ex-NASA project contributor | AI enthusiast | Gamedev hobbyist | Engineer Generalist

Follow my Telegram channel for:
- 🤖 AI news and tools
- 🧠 Neuroscience insights  
- 🚀 Space exploration
- 💭 Thoughts on future tech

[![Author telegram](https://img.shields.io/badge/Telegram-2CA5E0?style=for-the-badge&logo=telegram&logoColor=white)](https://t.me/+VKz5IExlz08zNTAy)
