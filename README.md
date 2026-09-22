![](https://img.shields.io/badge/AWS-232F3E?style=for-the-badge&logo=amazon-aws&logoColor=white)
![](https://img.shields.io/badge/JSON-000000?style=for-the-badge&logo=json&logoColor=white)

## 1. Introdução e Contexto
Este documento apresenta o relatório final de pesquisa e estudo baseado no curso de criação de assistentes virtuais da plataforma Digital Innovation One (DIO). O objetivo principal do estudo foi compreender a arquitetura e o funcionamento da orquestração de serviços computacionais *serverless* na nuvem da AWS, combinados com a integração de inteligência artificial generativa.

## 2. Fundamentação Tecnológica
O ecossistema estudado baseou-se na integração nativa de serviços da AWS para eliminar a necessidade de infraestrutura dedicada e simplificar o código da aplicação. Os três pilares do projeto foram:

### 2.1. AWS Step Functions
Atuando como o núcleo lógico do sistema, o Step Functions é um serviço de orquestração visual. O estudo focou em compreender como ele coordena componentes distribuídos e microserviços por meio de *State Machines* (Máquinas de Estado). Isso permite gerenciar o fluxo de execução, lidar com falhas inerentes à rede e realizar tentativas automatizadas (*retries*) sem escrever lógica complexa no código base.

### 2.2. Amazon States Language (ASL)
O comportamento do Step Functions é inteiramente ditado por configurações estruturadas em formato JSON, utilizando a Amazon States Language (ASL). A análise do ASL revelou a importância de mapear os estados corretamente:
* **Task States:** Onde o processamento real ocorre, interagindo com outras APIs e serviços da AWS.
* **Pass States:** Componentes de roteamento e transformação de dados, que injetam ou filtram informações no JSON de entrada/saída sem custo computacional direto.
* **Choice States:** Nós de decisão lógica que permitem o controle de fluxo condicional (estruturas semelhantes ao If/Else na programação tradicional).

### 2.3. Amazon Bedrock e Invocação de Modelos
A camada de inteligência do assistente foi estruturada em torno do Amazon Bedrock. O aspecto mais relevante analisado foi a capacidade do Step Functions de utilizar a API `InvokeModel` diretamente no ASL. Essa abordagem direta evita a criação de funções AWS Lambda intermediárias apenas para estabelecer chamadas HTTP, mantendo a arquitetura mais limpa e focada no tráfego dos *prompts* e no retorno gerado pelos Modelos Fundacionais (LLMs).

### 2.4. Flexibilidade Lógica e Adaptação de Escopo
Uma das maiores vantagens observadas nessa arquitetura é a completa separação entre a camada de orquestração (Step Functions) e a camada de inteligência (Bedrock). Essa abstração garante uma imensa flexibilidade lógica: alterando apenas o *prompt* injetado via JSON ou o modelo fundacional escolhido, o assistente pode ser instantaneamente adaptado para **qualquer assunto desejado** (suporte técnico, análise de dados, atendimento ao cliente, etc.) sem a necessidade de reescrever a máquina de estados.

## 3. Estruturação do Fluxo de Trabalho (Exemplo em ASL JSON)
Para validar o entendimento teórico da Amazon States Language, foi analisada a estrutura padrão necessária para o acionamento do assistente. O bloco JSON abaixo exemplifica a declaração de uma *Task* que recebe a entrada do usuário e aciona o Bedrock:

```json
{
  "Comment": "Definição de máquina de estado para assistente de IA",
  "StartAt": "InvocarModeloBedrock",
  "States": {
    "InvocarModeloBedrock": {
      "Type": "Task",
      "Resource": "arn:aws:states:::bedrock:invokeModel",
      "Parameters": {
        "ModelId": "anthropic.claude-v2",
        "Body": {
          "prompt.$": "$.entrada_usuario_formatada",
          "max_tokens_to_sample": 300
        }
      },
      "ResultPath": "$.resultado_processamento",
      "End": true
    }
  }
}
```

## 4. Desafios Técnicos e Metodologia Adotada
Durante a etapa de execução laboratorial, foram encontrados bloqueios relacionados à configuração do ambiente de nuvem. O principal desafio concentrou-se no gerenciamento do IAM (Identity and Access Management) da AWS. A complexidade de provisionar políticas estritas de permissão (*Roles*) para que o Step Functions pudesse se comunicar de forma segura e autônoma com o Bedrock dificultou a implementação prática integral.

Devido a essa barreira de infraestrutura, a metodologia de estudo foi adaptada de uma implementação prática (*hands-on*) para uma pesquisa arquitetural profunda. A decisão técnica foi priorizar o entendimento da modelagem de dados no ASL JSON e a compreensão teórica do ciclo de vida de uma aplicação *serverless*.

## 5. Conclusão e Conhecimentos Adquiridos
Embora o ambiente prático não tenha sido finalizado, o saldo do estudo foi altamente técnico. O contato com a documentação do Step Functions e do Bedrock proporcionou uma visão clara sobre engenharia de software voltada para nuvem. O aprendizado confirmou que dominar linguagens de configuração estruturada, como o JSON no padrão ASL, é tão essencial quanto a codificação tradicional para projetar fluxos de trabalho escaláveis e resilientes. Mais importante, provou que orquestradores de nuvem aliados à IA geram sistemas modulares, capazes de abordar qualquer nicho ou domínio de conhecimento com mínimas alterações arquiteturais.
