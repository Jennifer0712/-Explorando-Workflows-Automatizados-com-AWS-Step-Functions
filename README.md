# Desafio: Explorando Workflows Automatizados com AWS Step Functions

Este repositório documenta os estudos e a implementação prática do desafio do bootcamp sobre a orquestração de workflows com AWS Step Functions.

## 1. Objetivo

O objetivo foi consolidar o conhecimento em AWS Step Functions, aplicando os conceitos em um cenário prático para entender como coordenar múltiplos serviços da AWS em um fluxo de trabalho automatizado, resiliente e "serverless".

## 2. Cenário Prático: Processo de Reserva de Hotel

Para este desafio, modelei um workflow (uma "State Machine") para o processo de **reserva de um quarto de hotel**. Este cenário é ideal pois envolve múltiplos passos, lógica condicional (if/else) e caminhos claros de sucesso ou falha.

### Diagrama do Workflow (State Machine)

O fluxo desenhado no Workflow Studio demonstra a orquestração de várias funções Lambda e serviços de mensageria (SQS e SNS) para gerenciar o processo de reserva.

![Diagrama da State Machine de Reserva](images/image_24faa6.png)

### Componentes e Fluxo da Arquitetura

O workflow é composto pelos seguintes estados:

* **1. Lambda: `Verificar Disponibilidade`**
    * A primeira etapa. Esta função Lambda é invocada para checar (em um banco de dados, como o DynamoDB) se as datas e o quarto solicitados estão disponíveis.

* **2. Choice: `Quarto Disponivel?`**
    * Este é o primeiro "if" do fluxo. Ele analisa a saída da Lambda anterior.
    * **Caminho (Default):** Se o quarto *não* está disponível, o fluxo é desviado imediatamente para o estado final `SNS: Indisponivel`.
    * **Caminho (Rule #1):** Se o quarto *está* disponível, o fluxo continua para a etapa de pagamento.

* **3. Lambda: `Processar Pagamento`**
    * Esta função é chamada para processar o pagamento do cliente (ex: simulando uma chamada a uma API de pagamentos).

* **4. Choice: `Pagamento Aprovado?`**
    * O segundo "if" do fluxo, que verifica a resposta da Lambda de pagamento.
    * **Caminho (Default):** Se o pagamento é *recusado*, o fluxo é desviado para o estado `SNS: Pagamento Recusado`.
    * **Caminho (Rule #1):** Se o pagamento é *aprovado*, o fluxo segue para o "caminho feliz".

* **5. Lambda: `Confirmar Reserva`**
    * Com o pagamento aprovado, esta função finaliza o processo, escrevendo a reserva confirmada no banco de dados.

* **6. SQS: `Notificar Cliente`**
    * Após a confirmação, uma mensagem é enviada para uma fila SQS. Isso é um ótimo padrão de desacoplamento, permitindo que outros serviços (como envio de e-mail, SMS, etc.) consumam esta fila de forma assíncrona.

* **7. SNS: `Sucesso`**
    * Por fim, uma notificação é publicada em um tópico SNS para informar o sucesso geral da transação.

* **Estados Finais (Falha):**
    * **SNS: `Pagamento Recusado`:** Publica uma notificação em um tópico SNS específico para falhas de pagamento.
    * **SNS: `Indisponivel`:** Publica uma notificação em um tópico SNS informando que o quarto não estava disponível.

## 3. Anotações Chave e Insights

* **Orquestração vs. Coreografia:** Este desafio é um exemplo clássico de **Orquestração**. A Step Function é o "maestro" central que diz a cada serviço (Lambda, SQS) exatamente o que fazer e quando.

* **Desacoplamento com SQS/SNS:** Usar SQS para "notificar o cliente" e SNS para os estados finais (`Sucesso`, `Recusado`, `Indisponivel`) é uma prática excelente. Isso permite que a Step Function termine seu trabalho rapidamente, enquanto outros serviços (que assinam esses tópicos ou consomem essa fila) cuidam das notificações sem prender o workflow principal.

## 4. Conclusão

Este laboratório prático demonstrou como as AWS Step Functions são essenciais para construir workflows complexos e serverless. A capacidade de coordenar visualmente Funções Lambda, lógica condicional (`Choice`), e outros serviços da AWS (SQS, SNS) torna o desenvolvimento de aplicações robustas e fáceis de depurar muito mais simples.
