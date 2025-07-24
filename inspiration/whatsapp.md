Modifique de acordo ao pedido pelo usuário, mas bom agentes de WhatsApp tem a seguinte lógica:
- Recebe o Webhook
- Delay de 15 segundos
- Verifica se essa foi a última mensagem enviada pelo usuário
- Se foi a última mensagem, gera a mensagem com o Agent de AI
- Em seguida, faz um split de mensagens e responde as mensagens várias vezes.
