# 查看 Telegram 机器人聊天记录
免责声明：这是用于您自己电脑的版本，因为所有密钥都公开存储在文件中。如果您将其部署到托管服务上，任何访问它的人都可以读取您的密钥。

## 特点
1) 所有内容都在一个 HTML 页面中
2) 本地运行
3) 不仅可以查看，还可以代表机器人向用户发送消息
4) 深色/浅色主题切换
5) 声音通知 开/关
6) 通过哈希函数生成彩色"头像"

## 使用方法：
1) 下载
2) 输入您的密钥
3) 在浏览器中运行

## 使用前准备：
1) 您需要一个 Supabase 表 chat_logs（在此处免费连接服务 https://supabase.com/）
重要！将消息存储到数据库的机器人和您连接到 HTML 的机器人必须是同一个。

通过 SQL Editor 输入并运行：
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
2) 在 Supabase 的表设置中 -> Edit table -> 启用 Enable Realtime
（这是为了订阅更新，一旦消息到达数据库，应用程序就会收到通知并更新聊天）

3) 在 admin.html 中修改：
```js
const BOT_TOKEN = 'XXXXXXXXX'; /// 在 Telegram 的 [https://t.me/BotFather](https://t.me/BotFather) 中获取
const SUPABASE_URL = 'https://database-name.supabase.co'; //在 supabase 数据库的项目设置中获取 - Data API 部分
const SUPABASE_ANON_KEY = 'XXXXXXXXXXX'; //同一位置 - API KEYS 部分
```



4) 配置消息如何到达此数据库的逻辑

如果您使用 n8n，可以这样做，使用节点保存来自用户的消息和来自机器人的回复：

这是在"Telegram Trigger"节点之后（您可以将其复制粘贴到工作流中 - 它将作为节点插入）：
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

这是在从机器人向用户发送消息后的最后：
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
