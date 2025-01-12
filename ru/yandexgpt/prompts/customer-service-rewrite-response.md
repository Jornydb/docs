# Переписывание шаблонного ответа

## Пример 1 {#example-1}

### Параметры запроса {#params}

* **Инструкция**: Ты — оператор клиентской поддержки. Переформулируй ответ так, как будто он составлен оператором поддержки.

* **Текст запроса**: Понял! Вернусь с ответом. Постараюсь как можно скорее. Пожалуйста, расскажите как произошло? Постараюсь помочь.

* **Температура**: `0`

* **Ответ**: Здравствуйте! Спасибо за ожидание.\n\nПожалуйста, уточните, что именно произошло, и я постараюсь вам помочь.

### Структура запроса {#structure}

```json
{
  "modelUri": "gpt://<идентификатор_каталога>/yandexgpt-lite",
  "completionOptions": {
    "stream": false,
    "temperature": 0,
    "maxTokens": "2000"
  },
  "messages": [
    {
      "role": "system",
      "text": "Ты — оператор клиентской поддержки. Переформулируй ответ так, как будто он составлен оператором поддержки."
    },
    {
      "role": "user",
      "text": "Понял! Вернусь с ответом. Постараюсь как можно скорее. Пожалуйста, расскажите как произошло? Постараюсь помочь."
    }
  ]
}
```

Где `<идентификатор_каталога>` — [идентификатор каталога](../../resource-manager/operations/folder/get-id.md) {{ yandex-cloud }}, у которого есть доступ к сервису {{ yagpt-name }}.

{% list tabs %}

- cURL

	```bash
	curl -k -v -X POST
     	-H "Authorization: Bearer <значение_IAM-токена>"
     	-d @prompt.json
     	https://llm.{{ api-host }}/foundationModels/v1/completion
	```
	
	Где:

	* `<значение_IAM-токена>` — IAM-токен, полученный для вашего аккаунта.
	* `prompt.json` — файл в формате JSON, содержащий параметры запроса.

{% endlist %}

### Ответ {#answer}

```json
{
    "result": {
        "alternatives": [
            {
                "message": {
                    "role": "assistant",
                    "text": "Здравствуйте! Спасибо за ожидание.\n\nПожалуйста, уточните, что именно произошло, и я постараюсь вам помочь."
                },
                "status": "ALTERNATIVE_STATUS_FINAL"
            }
        ],
        "usage": {
            "inputTextTokens": "54",
            "completionTokens": "22",
            "totalTokens": "76"
        },
        "modelVersion": ""
    }
}
```

## Пример 2 {#example-2}

### Параметры запроса {#params}

* **Инструкция**: Ты — оператор клиентской поддержки. Переформулируй ответ так, как будто он составлен оператором поддержки.

* **Текст запроса**: Ваш заказ поступил и сейчас обрабатывается. Мы отправим вам уведомление о доставке сразу же, как только отправим ваш заказ.

* **Температура**: `0`

* **Ответ**: Здравствуйте!\n\nСпасибо за заказ. Ваш заказ был успешно оформлен и в данный момент обрабатывается. Как только мы отправим его, мы сразу же свяжемся с вами по электронной почте или телефону, чтобы сообщить о доставке.\n\nЕсли у вас возникнут какие-либо вопросы, пожалуйста, не стесняйтесь обращаться к нам. Мы всегда рады помочь!

### Структура запроса {#structure}

```json
{
  "modelUri": "gpt://<идентификатор_каталога>/yandexgpt-lite",
  "completionOptions": {
    "stream": false,
    "temperature": 0,
    "maxTokens": "2000"
  },
  "messages": [
    {
      "role": "system",
      "text": "Ты - оператор клиентской поддержки. Переформулируй ответ так, как будто он составлен оператором поддержки."
    },
    {
      "role": "user",
      "text": "Ваш заказ поступил и сейчас обрабатывается. Мы отправим вам уведомление о доставке сразу же, как только отправим ваш заказ."
    }
  ]
}
```

Где `<идентификатор_каталога>` — [идентификатор каталога](../../resource-manager/operations/folder/get-id.md) {{ yandex-cloud }}, у которого есть доступ к сервису {{ yagpt-name }}.

{% list tabs %}

- cURL

	```bash
	curl -k -v -X POST
     	-H "Authorization: Bearer <значение_IAM-токена>"
     	-d @prompt.json
     	https://llm.{{ api-host }}/foundationModels/v1/completion
	```
	
	Где:

	* `<значение_IAM-токена>` — IAM-токен, полученный для вашего аккаунта.
	* `prompt.json` — файл в формате JSON, содержащий параметры запроса.

{% endlist %}

### Ответ {#answer}

```json
{
    "result": {
        "alternatives": [
            {
                "message": {
                    "role": "assistant",
                    "text": "Здравствуйте!\n\nСпасибо за заказ. Ваш заказ был успешно оформлен и в данный момент обрабатывается. Как только мы отправим его, мы сразу же свяжемся с вами по электронной почте или телефону, чтобы сообщить о доставке.\n\nЕсли у вас возникнут какие-либо вопросы, пожалуйста, не стесняйтесь обращаться к нам. Мы всегда рады помочь!"
                },
                "status": "ALTERNATIVE_STATUS_FINAL"
            }
        ],
        "usage": {
            "inputTextTokens": "53",
            "completionTokens": "66",
            "totalTokens": "119"
        },
        "modelVersion": ""
    }
}
```
