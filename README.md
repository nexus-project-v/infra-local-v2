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

{"eventId": "5555-6666","eventType": "PAYMENT_PROCESSED","eventVersion": 1,"occurredAt": "2026-02-15T14:24:10Z","source": "payment-service","correlationId": "order-123","payload": {"transactionCode": "bc82783b-84c1-4de8-808e-0938c1059404","orderId": "order-123","status": "APPROVED","amount": 199.90,"currency": "BRL","paymentMethod": "PIX","provider": "MERCADO_PAGO","authorizationCode": "MP-983746", "reasonCode": "", "message": ""}}

{"eventId": "5555-6666","eventType": "PAYMENT_PROCESSED","eventVersion": 1,"occurredAt": "2026-02-15T14:24:10Z","source": "payment-service","correlationId": "order-123","payload": {"transactionCode": "bc82783b-84c1-4de8-808e-0938c1059404","orderId": "order-123","status": "DECLINED","amount": 199.90,"currency": "BRL","paymentMethod": "PIX","provider": "MERCADO_PAGO","authorizationCode": "", "reasonCode": "INSUFFICIENT_FUNDS", "message": "Saldo insuficiente"}}

{"eventId": "5555-6666","eventType": "PAYMENT_PROCESSED","eventVersion": 1,"occurredAt": "2026-02-15T14:24:10Z","source": "payment-service","correlationId": "order-123","payload": {"transactionCode": "bc82783b-84c1-4de8-808e-0938c1059404","orderId": "order-123","status": "ERROR","amount": 199.90,"currency": "BRL","paymentMethod": "PIX","provider": "MERCADO_PAGO", "authorizationCode": "", "reasonCode": "", "message": "Timeout ao comunicar com o PSP"}}

Agora digite qualquer coisa e aperte Enter:

{"eventType":"TRANSACTION_CREATED","transactionId":"tx-1","amount":100.50}

Cada linha = uma mensagem no Kafka.

✅ 5️⃣ Consumir mensagens no terminal (Consumer)

Abra outro terminal:

docker exec -it kafka kafka-console-consumer --topic transactions.created --from-beginning --bootstrap-server kafka:9092


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
