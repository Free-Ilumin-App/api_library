Modifique de acordo ao pedido pelo usuário, mas bom agentes de WhatsApp tem a seguinte lógica:
- Recebe o Webhook
- Delay de 15 segundos
- Verifica se essa foi a última mensagem enviada pelo usuário
- Se foi a última mensagem, gera a mensagem com o Agent de AI
- Em seguida, faz um split de mensagens (chunks sem cortar palavra) para enviar mensagens pequenas.
- Envia todas as mensagens pequenas do chunks.

# Ideia por trás do Agente
O agente de WhatsApp por estar dentro do WhatsApp precisa seguir a lógica que a ferramenta possui.

No WhatsApp a maioria dos usuários envia mensagens separadas, exemplo:
1. "Oi, tudo bem?"
2. "O que vai fazer?"
3. "Queria sair hoje".

E por enviar 3 mensagens em seguidas, não podemos responder as 3 seguidas separadamente. Por isso, normalmente salvamos essa mensagem, colocamos um delay de 15 segundos e verificamos se é a última mensagem enviada. Se não foi, descarta a mensagem (pois apenas a última que vim do webhook, que é quando o usuário parar de responder que será gerada pela AI). Se foi a última, faz uma mesclagem de todas as mensagens que ele enviou nesse período, e envia todas de 1 vez só para a AI processar todas de uma única vez. 

Obtém a resposta da AI e responde.

Se a AI responder a mensagem muito grande, também fazemos um chunks para enviar várias mensagens para ser mais natural, já que no WhatsApp é um ambiente de mensagens curtas enviadas várias vezes em um curto período de tempo.

# Porque dessa lógica
Vou te explicar o porque dessa lógica, mas se achar necessário utilizar outra, faça.

- Essa lógica funciona porque: toda pequena mensagem que o usuário enviar vai vim um Webhook (ou seja, seriam 3 webhooks iniciados no exemplo que dei).
- Então, ao aguardar 15 segundos de delay, conseguimos verificar se foi a última mensagem que ele enviou ou não (pois só precisamos responder depois que ele parar de enviar mensagens).
- Se não foi a última, simplesmente descarta, não precisa fazer nada.
- Se foi a última, mescla todas as mensagens que ele fez nesse período e envia para a AI gerar a resposta.
- E também em seguida já exclui as mensagens da memória, pois assim, não será enviada mensagens repetidas para a AI.
- E por úlimo, fazer o chunks da resposta da AI para responder o usuário no WhatsApp com split de mensagens (ser natural).

# Lógica passo a passo
1. 
