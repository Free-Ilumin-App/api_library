As credênciais necessárias são: URL, ApiKey e Instance Name, use como variável se não encontrar uma credêncial do usuário.
A mensagem para enviar ao usuário (text) e o número (number), podem estar na automação ou fixa, depende do usuário.

```curl
curl --location 'https://{{domain}}/message/sendText/{{instance}}' \
--data '{
    "number": "{{number a enviar}}",
    "text": "{{texto a enviar}}"
}'
```
