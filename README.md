docker logs -f kafka

✅ 1️⃣ Ver se o Kafka está de pé
docker ps

Você deve ver o container kafka rodando.

Logs:

docker logs -f kafka

Procure algo como: KafkaServer started.

✅ 2️⃣ Listar tópicos
docker exec -it kafka kafka-topics --list --bootstrap-server kafka:9092

Se não aparecer nada, ok (ainda não criou tópicos).

✅ 3️⃣ Criar um tópico para teste
docker exec -it kafka kafka-topics --create --topic transactions.created --bootstrap-server kafka:9092 --partitions 1 --replication-factor 1

Conferir:

docker exec -it kafka kafka-topics --describe --topic transactions.created --bootstrap-server kafka:9092


  ✅ 4️⃣ Produzir mensagens no terminal (Producer)

Abra um terminal e rode:

docker exec -it kafka kafka-console-producer --topic transactions.created --bootstrap-server kafka:9092

docker exec -it kafka kafka-console-producer --topic payments.processed --bootstrap-server kafka:9092
payments.processed
{"eventId": "5555-6666","eventType": "PAYMENT_PROCESSED","eventVersion": 1,"occurredAt": "2026-02-15T14:24:10Z","source": "payment-service","correlationId": "order-123","payload": {"transactionCode": "49f98798-82c2-4ac2-a0c5-5dd9aac05972","orderId": "order-123","status": "APPROVED","amount": 199.90,"currency": "BRL","paymentMethod": "PIX","provider": "MERCADO_PAGO","authorizationCode": "MP-983746", "reasonCode": "", "message": ""}}

{"eventId": "5555-6666","eventType": "PAYMENT_PROCESSED","eventVersion": 1,"occurredAt": "2026-02-15T14:24:10Z","source": "payment-service","correlationId": "order-123","payload": {"transactionCode": "49f98798-82c2-4ac2-a0c5-5dd9aac05972","orderId": "order-123","status": "DECLINED","amount": 199.90,"currency": "BRL","paymentMethod": "PIX","provider": "MERCADO_PAGO","authorizationCode": "", "reasonCode": "INSUFFICIENT_FUNDS", "message": "Saldo insuficiente"}}

{"eventId": "5555-6666","eventType": "PAYMENT_PROCESSED","eventVersion": 1,"occurredAt": "2026-02-15T14:24:10Z","source": "payment-service","correlationId": "order-123","payload": {"transactionCode": "bc82783b-84c1-4de8-808e-0938c1059404","orderId": "order-123","status": "ERROR","amount": 199.90,"currency": "BRL","paymentMethod": "PIX","provider": "MERCADO_PAGO", "authorizationCode": "", "reasonCode": "", "message": "Timeout ao comunicar com o PSP"}}

Agora digite qualquer coisa e aperte Enter:

{"eventType":"TRANSACTION_CREATED","transactionId":"tx-1","amount":100.50}

Cada linha = uma mensagem no Kafka.

✅ 5️⃣ Consumir mensagens no terminal (Consumer)

Abra outro terminal:

docker exec -it kafka kafka-console-consumer --topic transaction.created --from-beginning --bootstrap-server kafka:9092
--
{"eventId":"607e7634-a1cf-48f2-b36b-fc8b038606ce","eventType":"TRANSACTION_CREATED","occurredAt":[2026,2,23,20,22,45,1109800],"transactionData":{"id":"7074d4bf-d45f-4929-b202-1a1003afc641","productId":null,"productQuantity":1,"clientId":"76b00215-5c8c-4b2e-ae11-3afd7a5ac609","sellerId":"16ba1b15-b7cf-48fa-bbd1-ed1e98e29916","code":"0b73b0e3-d5bc-49b3-9c11-ae06e4ed190e","price":39.66,"transactionStatus":"RESERVED","transactionTypeId":"5109efb8-d05c-45b1-863e-727a861767c5"},"clientData":{"id":"76b00215-5c8c-4b2e-ae11-3afd7a5ac609","code":"QKUS2","name":"Rogério Fontes Toma 8556","email":"rogerio.fontes.tomaz8556@gmail.com","description":"sdas","mediaId":"1c14484f-aee1-4b7e-b7c6-2b79fccc827c","mobile":"(34) 97452-6758","clientTypeId":"6ef13d89-6051-445e-a5c3-4f36d1435ae1"},"productItems":[{"productId":"58d5db30-4638-4159-a312-cc01b8658ec0"}]}
===
docker exec -it kafka kafka-console-consumer --topic orders.ready-for-production.v1 --from-beginning --bootstrap-server kafka:9092
--
{"transactionId":"073cc193-3972-47f6-836d-7193dae4971b","eventId":"a326bfbd-81c9-4dc7-a760-a47bc880522d","eventType":"ORDER_READY_FOR_PRODUCTION","version":1,"occurredAt":[2026,2,21,12,36,58,544212700],"orderId":"order-123","clientId":"9f6cea3f-ec0f-47b4-8508-7816b49a8f5d","productId":"d74baf3d-ed75-4823-b410-d5491969ebeb","productQuantity":1,"items":null,"shippingEvent":{"address":"Rua Exemplo, 123"}}
===
docker exec -it kafka kafka-console-consumer --topic transaction.declined.v1 --from-beginning --bootstrap-server kafka:9092
--
{"transactionId":"073cc193-3972-47f6-836d-7193dae4971b","eventId":"a326bfbd-81c9-4dc7-a760-a47bc880522d","eventType":"ORDER_READY_FOR_PRODUCTION","version":1,"occurredAt":[2026,2,21,12,36,58,544212700],"orderId":"order-123","clientId":"9f6cea3f-ec0f-47b4-8508-7816b49a8f5d","productId":"d74baf3d-ed75-4823-b410-d5491969ebeb","productQuantity":1,"items":null,"shippingEvent":{"address":"Rua Exemplo, 123"}}

Você deve ver as mensagens que digitou no producer. 🎉

✅ 6️⃣ Testar com chave (importante pra ordenação)

docker exec -it kafka kafka-console-producer.sh \
Producer com key:

docker exec -it kafka kafka-console-producer \
  --topic transactions.created \
  --bootstrap-server kafka:9092 \
  --property "parse.key=true" \
  --property "key.separator=:"

Digite:

tx-123:{"eventType":"TRANSACTION_CREATED","transactionId":"tx-123","amount":199.90}
tx-123:{"eventType":"TRANSACTION_CREATED","transactionId":"tx-123","amount":199.90}

Consumer com chave:

docker exec -it kafka kafka-console-consumer \
  --topic transactions.created \
  --bootstrap-server kafka:9092 \
  --property print.key=true \
  --property key.separator=" | "

Saída:

tx-123 | {"eventType":"TRANSACTION_CREATED","transactionId":"tx-123","amount":199.90}
✅ 7️⃣ Limpar / resetar consumo (grupo de consumer)

docker exec -it kafka kafka-console-consumer.sh \

Se você quiser simular grupo de consumidores:

docker exec -it kafka kafka-console-consumer \
  --topic transactions.created \
  --bootstrap-server kafka:9092 \
  --group test-group

Depois resetar offsets:

docker exec -it kafka kafka-consumer-groups \
  --bootstrap-server kafka:9092 \
  --group test-group \
  --reset-offsets --to-earliest --execute --topic transactions.created
⚠️ Erros comuns
❌ Connection to node -1 could not be established

→ Kafka não subiu direito ou porta errada

❌ localhost:9092 refused

→ Container não expôs porta ou está usando IP errado

❌ Kafka subiu mas producer não conecta

→ KAFKA_CFG_ADVERTISED_LISTENERS diferente de localhost

No seu compose eu configurei certo pra uso local 👍# infra-local

netstat -ano | findstr :8761
taskkill /PID 25812 /F; taskkill /PID 7932 /F

===
Ainda no PowerShell (admin):

wsl --shutdown

Depois:

net stop com.docker.service
net start com.docker.service

Ou simplesmente:

Feche Docker

Abra novamente

Teste:

docker info


docker compose -f docker-compose.infra.yml up -d
docker compose -f docker-compose.apps.yml up -d
docker compose -f docker-compose.apps.yml down# infra-local-v2
