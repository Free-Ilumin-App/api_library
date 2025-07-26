As credênciais necessárias são: URL, ApiKey e Instance Name, use como variável se não encontrar uma credêncial do usuário.
A mensagem para enviar ao usuário (text) e o número (number), podem estar na automação ou fixa, depende do usuário.
IMPORTANT: Caso o usuário não tenha fornecido alguma informação relacionado,coloque como variável ou peça ao usuário, exemplo, a instance name pode ser uma variável, assim como texto a enviar e o número (se forem fixo).

```curl
curl --location 'https://{{domain}}/message/sendText/{{instance}}' \
--data '{
    "number": "{{number a enviar}}",
    "text": "{{texto a enviar}}"
}'
```
