# 🌧️ DrenaAi – Documento de Projeto

## 1. 📌 Contexto e Problema

Hoje a coleta de lixo atende os chamados por ordem de chegada, sem considerar se há chuva a caminho nem se a rua costuma alagar. O resultado é que um lixo parado em dia seco pode ter a mesma prioridade de um bueiro entupido horas antes de um temporal.

- **🗑️ Problema:** o descarte irregular de lixo nas calçadas obstrui o escoamento da água em dias chuvosos, gerando alagamentos e mau cheiro que prejudicam a mobilidade e a saúde pública.
- **👥 Beneficiados:** moradores de bairros sujeitos a transbordamentos e pedestres que dependem do transporte público e da circulação a pé.
- **📍 Local:** São Paulo (as bases SP156 e CGE-SP apontam para a cidade). Bairro piloto: **[PENDENTE]**.
  
### 1.1. 🌍 ODS conectados

O enunciado exige explicar como o projeto contribui para as metas de cada ODS.
- **ODS 11 (Cidades e Comunidades Sustentáveis):** reduz o impacto do lixo urbano e a exposição de moradores a alagamentos.
- **ODS 13 (Ação Contra a Mudança Global do Clima):** aumenta a resiliência da cidade a chuvas intensas.
- **[PENDENTE]** escolher e citar as metas exatas (por exemplo 11.5, 11.6 e 13.1, a confirmar no site da ONU).

## 2. 🎯 Necessidade e Validação Obrigatória

A solução precisa transformar cada ocorrência de lixo em uma posição na fila de coleta, calculada por um **Índice de Prioridade de Coleta (IPC)** que combina três variáveis:

- 🗑️ Severidade do lixo reportado e validado.
- 🌧️ Previsão de precipitação (chuva em mm nas próximas 12 h a 24 h).
- 🌊 Histórico de vulnerabilidade topográfica e risco de alagamento da via.

### 2.1. 📈 Indicador de impacto

Redução no tempo de resposta a incidentes de remoção de resíduos e redução de ocorrências de enchentes por acúmulo de matéria orgânica. Linha de base e meta numérica: **[PENDENTE]**.

### 2.2. ✅ Validação obrigatória: cenário de referência

O projeto só está pronto se este cenário funcionar do início ao fim. Ele também é o roteiro comum de todos os vídeos e do pitch.
> Um morador vê um lixo que o sensor não detectou, envia a foto pelo app, a IA avalia a foto e o app mostra o alerta com a severidade. O sistema cruza a severidade com a chuva prevista e o risco de alagamento da rua, calcula o IPC e reordena a fila de coleta. O morador acompanha o status da ocorrência no app.

Exemplo numérico para a demo (ilustrativo, ajustar depois):

|                                 | Ocorrência A (bueiro cheio, via que alaga) | Ocorrência B (lixo leve, via alta) |
| ------------------------------- | :----------------------------------------: | :--------------------------------: |
| **Severidade**                  | Alta (0,75)                                | Média (0,50)                       |
| **Chuva prevista 12 h**         | 35mm (0,70)                               | 0mm (0,00)                        |
| **Vulnerabilidade da via**      | 0,90                                       | 0,20                               |
| **IPC (pesos 0,4 / 0,3 / 0,3)** | 0,78                                       | 0,26                               |

A ocorrência A pode ter chegado depois da B e ainda assim vai para o topo da fila. É isso que a demo precisa mostrar.

## 3. 💡 Solução Proposta – DrenaAi

Aplicativo móvel para o cidadão, backend em C# com as regras de negócio, IA para avaliar a foto e prever o risco de alagamento, sensores ESP32 nos bueiros e um servidor Linux que hospeda e monitora tudo. O sensor detecta automaticamente os bueiros que cobre. O app é usado pelos moradores que veem lixo que o sistema não detectou.

### 3.1. 🔄 Fluxo de alto nível

```
👤 Morador (app: foto + tipo + GPS)        📡 ESP32 (bueiro: ultrassônico + gás)
                  ↓                                      ↓
                  └──────────────────┬───────────────────┘
                                     ↓
                    ⚙️ Backend C# (regras de negócio)
                                     ↓
             🔒 Filtro de dados pessoais (Cybersecurity)
                                     ↓
              🧠 IA da foto: severidade (modelo local)
                                     ↓
       🌧️ Previsão de chuva + 🌊 risco de alagamento da via (ML)
                                     ↓
               📊 IPC calculado → fila de coleta reordenada
                                     ↓
           🔔 Webhook de priorização + 📱 status no app
                                     ↓
              🐧 Servidor Linux hospeda e monitora tudo
```

Ocorrências que vêm do sensor já chegam com a severidade medida e não passam pela IA da foto. Daí em diante seguem o mesmo caminho das do app.

### 3.2. 📱 Passo a passo da denúncia pelo app

1. O morador faz login (ou se cadastra no primeiro uso), abre o app, tira a foto do lixo e escolhe o tipo de ocorrência. A região é preenchida automaticamente.
2. O app envia a denúncia ao backend C#. A descrição passa pelo filtro de dados pessoais antes de ser gravada.
3. O backend envia a foto à IA de visão (modelo de visão local, rodando no servidor Linux), que devolve a severidade.
4. O app mostra ao morador o alerta com a severidade avaliada.
5. O backend consulta a previsão de chuva (próximas 12 h a 24 h; na demo, valor simulado no mesmo formato do dado real) e o risco de alagamento da via (modelo de ML treinado com SP156 + INMET).
6. O backend calcula o IPC, reordena a fila de coleta e dispara o webhook de priorização.
7. O morador acompanha o status da ocorrência no app.

### 3.3. 📡 Passo a passo do sensor

O ESP32 (simulado no Wokwi, com um bueiro monitorado) lê o sensor ultrassônico (obstrução do bueiro) e o sensor de gás (matéria orgânica acumulada). Quando passa de um limite, aciona LEDs e buzzer localmente e envia a telemetria ao backend. O backend cria uma ocorrência com origem SENSOR.

### 3.4. 🗣️ Como explicar em 1 minuto
A coleta de lixo atende os chamados por ordem de chegada, mas um bueiro entupido antes de uma chuva forte é muito mais urgente que um lixo parado em dia seco. Nossa solução recebe relatos com foto pelo app e leituras de sensores nos bueiros, avalia a gravidade com IA, cruza com a previsão de chuva e o histórico de alagamento da rua e reordena a fila para atender primeiro o que mais ameaça o bairro. O resultado esperado é menor tempo de resposta e menos alagamentos causados por lixo.

## 4. 🛠️ Stack Tecnológica

| Camada | Tecnologia escolhida |
| --- | --- |
| 📱 Mobile | React Native com TypeScript (Expo e Expo Router), SQLite local no aparelho |
| ⚙️ Backend e serviços | C# / .NET Web API, injeção de dependência nativa, Entity Framework Core, LINQ, async/await, REST documentado com Swagger ou Scalar |
| 🗄️ Banco do backend | Oracle DB, com migrações do EF Core |
| 🧪 Testes do backend | xUnit, com foco no cálculo do IPC |
| 🧠 IA: risco de alagamento | Python, dados reais SP156 + INMET (ou CGE-SP), modelo **[PENDENTE]** entre XGBoost, Random Forest, MLP, LSTM e KMeans + PCA |
| 🧠 IA: severidade da foto | Modelo de visão local, modelo **[PENDENTE]**, chamado pelo backend |
| 📡 Physical Computing | ESP32 no Wokwi, sensor ultrassônico, sensor de gás, LEDs e buzzer |
| 🔒 Segurança | Motor de parsing de dados pessoais (PII), Taint Analysis, LLM que traduz falhas técnicas em comunicados executivos, login com token (JWT, proposta) e senha guardada só como hash |
| 🐧 Infraestrutura | Linux em VM, Shell Script, systemd, monitoramento de CPU, memória e disco, logs de auditoria |
| ✅ Qualidade | Azure Boards (features e casos de teste) |
| 📦 Versionamento | Git + GitHub (monorepo proposto, ver seção 6) |

## 5. ✨ Funcionalidades (por camada)

### 5.1. 📱 Mobile (Bibas)

O app abre no login e depois mostra três abas (Expo Router), com uma tela de detalhe e uma de perfil.

| Tela | O que mostra ou faz | Dados e endpoints |
| --- | --- | --- |
| 🔐 Login e cadastro | e-mail e senha; cadastro no primeiro uso | POST /api/auth/login e /api/auth/cadastro |
| 👤 Perfil | detalhes da conta e opção de sair | dados do usuário logado |
| 🏘️ Aba Bairro | lista das ocorrências do bairro com status, mais a chuva prevista | GET /api/ocorrencias?bairro= e GET /api/previsao?bairro= |
| 📸 Aba Denunciar | formulário com foto, tipo e GPS | POST /api/ocorrencias |
| 🗺️ Aba Mapa (marcada com * no desenho, extra) | pontos das ocorrências por IPC | GET /api/fila, com latitude, longitude e ipc |
| 🔎 Detalhe | foto, tipo, IPC e linha do tempo de status | GET /api/ocorrencias/{id} 

- 💾 O app salva dados localmente em SQLite (as denúncias do próprio usuário e a última lista do bairro), o que atende ao requisito de banco de dados da disciplina.
- 🧓 Acessibilidade: letras legíveis, bom contraste, linguagem simples e telas pequenas.

### 5.2. ⚙️ Backend em C# e serviços (Vini Boy)

Uma única API em C#/.NET atende as disciplinas de C# e de Arquitetura Orientada a Serviços. O README de cada entrega aponta as partes que cada disciplina avalia.

- 📝 Cadastro da denúncia e da ocorrência
- 📡 Ingestão da telemetria do ESP32
- 🌧️ Consulta aos dados de previsão (ou mock do modelo de IA)
- 🔔 Webhook de priorização da fila de atendimento
- 🔐 Login e cadastro de usuários
- 📊 Cálculo do IPC, com testes em xUnit
- 🗄️ Persistência com Entity Framework Core e migrações, documentação automática dos endpoints

#### 📋 Entidade Ocorrência

| Campo | Tipo | Observação |
| --- | --- | --- |
| id | identificador | gerado pelo backend |
| origem | APP ou SENSOR | quem criou a ocorrência |
| tipo | texto | tipo escolhido no app, ou OBSTRUCAO_BUEIRO quando vem do sensor |
| descricao | texto | já sanitizada pelo filtro de dados pessoais |
| fotoUrl | texto | vazio quando a origem é SENSOR |
| latitude, longitude | decimais | o app preenche automaticamente; o sensor usa a posição cadastrada do bueiro |
| bairro | texto | derivado da localização |
| severidade | número de 0 a 1 | proposta: faixas Baixa, Média, Alta e Crítica; **[PENDENTE]** confirmar escala |
| status | RECEBIDA, VALIDADA, NA_FILA, EM_ATENDIMENTO ou CONCLUIDA | acompanhado pelo morador no app |
| ipc | número de 0 a 1 | calculado pelo backend |
| criadaEm | data e hora |  |
| usuarioId | identificador | quem registrou; vazio quando a origem é SENSOR |
| historicoStatus | lista | cada status com data e hora; alimenta a linha do tempo da tela de detalhe |

Entidade Usuário: id, nome, e-mail, senhaHash e criadoEm. A senha nunca é guardada em texto puro. Por serem dados pessoais, o cadastro coleta só o mínimo.

#### 🚏 Endpoints previstos (proposta)

| Método e rota | Quem chama | Entrada | Saída |
| --- | --- | --- | --- |
| POST /api/auth/cadastro | App | nome, e-mail, senha | 201 com id do usuário |
| POST /api/auth/login | App | e-mail, senha | token de sessão (JWT) usado nas demais chamadas |
| POST /api/ocorrencias | App | foto (multipart), tipo, descrição, latitude, longitude | 201 com id, severidade e status |
| GET /api/ocorrencias?bairro=&status= | App | filtros opcionais | lista de ocorrências do bairro |
| GET /api/ocorrencias/{id} | App | id | ocorrência com status, IPC e histórico |
| POST /api/telemetria | ESP32 | sensorId, bueiroId, distanciaCm, gasPpm, timestamp | 202 aceito |
| GET /api/previsao?bairro= | Backend | bairro ou coordenadas | chuvaMm12h, chuvaMm24h, riscoAlagamento (0 a 1) |
| POST /api/severidade | Backend, para o serviço de IA | foto | severidade (0 a 1), confiança, justificativa curta |
| GET /api/fila | App e painel | bairro opcional | ocorrências ordenadas por IPC |
| POST /api/fila/priorizar (webhook) | Backend | ocorrência atualizada | fila reordenada e notificação ao sistema de coleta |

Todos os endpoints entram na documentação Swagger. A previsão e a severidade podem começar como mock com o mesmo formato e depois trocar pelo modelo real, sem mudar o contrato. Os endpoints usados pelo app exigem o token do login. O endpoint de telemetria do ESP32 não tem usuário, então precisa de uma chave própria do dispositivo. **[PENDENTE]** Vini e Rafael definem como.

#### 🧮 Fórmula do IPC (proposta)

$$ \mathrm{IPC} = 0{,}4 \cdot S + 0{,}3 \cdot C + 0{,}3 \cdot V $$

- $S$ é a severidade, $C$ é a chuva prevista normalizada e $V$ é a vulnerabilidade da via. Todas ficam entre 0 e 1.
- $C$ é a chuva prevista em mm dividida por um teto (proposta: 50 mm, **[PENDENTE]**).
- $V$ vem do modelo de ML (histórico de alagamento e topografia).
- Pesos 0,4 / 0,3 / 0,3 e o teto de chuva são provisórios. Devem ser definidos pelo grupo e ficar em configuração, não fixos no código.
- O xUnit deve cobrir: pesos somando 1, limites (todos 0 e todos 1), chuva acima do teto, ordenação correta da fila e desempate por antiguidade.

### 5.3. 🧠 Inteligência Artificial (João)

São duas IAs diferentes, e só uma delas é o modelo avaliado na disciplina de ML.

- **🌊 Modelo de risco de alagamento:** prevê se uma região sofrerá consequências com acúmulo de lixo e chuvas recorrentes. Cruza os chamados do SP156 com a precipitação (INMET ou CGE-SP) e o histórico de alagamento. A saída alimenta o V do IPC. Exige dados reais e públicos (dados simulados não são aceitos), pelo menos 10 combinações de 3 hiperparâmetros com validação cruzada só no treino, métricas de treino e teste lado a lado e discussão de overfitting.
- **📷 Severidade da foto:** modelo de visão local, chamado pelo backend. Devolve severidade, confiança e uma justificativa curta. Entra como serviço integrado, não como o modelo avaliado.

### 5.4. 📡 Sensores ESP32 (Bradesco)

Sensoriamento dos bueiros e dos locais de descarte comunitário para evitar acúmulo excessivo, simulado no Wokwi com um bueiro monitorado.

- 📏 Sensor ultrassônico validando obstrução
- 🧪 Sensor de gás para detectar emanação de matéria orgânica acumulada
- 💡 LEDs e buzzer para sinalização de emergência
- 📤 Envio da telemetria ao backend (POST /api/telemetria), com bueiroId para permitir mais bueiros depois
- **[PENDENTE]** limites de distância e de gás que disparam o alerta e como as leituras viram severidade, já que a ocorrência de origem SENSOR chega ao IPC sem passar pela foto

### 5.5. 🔒 Cybersecurity (Rafael)

Blindar os dados dos moradores que fazem denúncias, com adequação à LGPD e prevenção a vazamentos. Nada será reaproveitado de aulas, então o motor de parsing é construído do zero.

- 🔍 Mapeamento de PII: regras de parsing (CPF, saúde, senhas e outros) para identificar e sanitizar dados sensíveis informados acidentalmente ou voluntariamente nos relatos
- 🧬 Taint Analysis: rastreamento do ciclo de vida desses dados para que não vazem em logs públicos de requisição
- 🤖 Riscos com IA: módulo com LLM que interpreta relatórios de vulnerabilidade e traduz as falhas técnicas em comunicados executivos e alertas claros de risco
- 📊 Dashboard executivo com o indicador Risco Cidadão
- 🔐 Cadastro de usuários: coletar o mínimo (nome, e-mail, senha), guardar a senha só como hash e usar usuários fictícios nos testes

### 5.6. 🐧 Operating Systems (João)

Infraestrutura Linux para hospedar e manter o sistema em operação.

- 🌐 Scripts de gerenciamento de rede (rede, DNS, hostname) e de instalação de dependências
- 🚀 Provisionamento automático da aplicação web (a API em C#) e do modelo de visão local na VM
- 📈 Monitoramento de memória, CPU e disco, principalmente pelo envio de fotos, com alertas e auditoria em log
- 🔁 Serviços gerenciados pelo systemd, com reinício automático em caso de falha
- 📝 Logs de auditoria e de execução dos scripts

### 5.7. ✅ Testing, Compliance & QA (Rafael)

- 🗂️ Backlog de features no Azure Boards, com descrição e critérios de aceite (o professor entra como Admin)
- 🧪 Casos de teste funcionais, positivos e negativos, caixa-preta, com objetivo, preparação, inputs, outputs esperados e passos
- 🔗 Mapeamento completo com critérios de aceite, inputs e outputs documentados
- 🎤 Pitch de até 4 minutos, enviado nesta disciplina

## 6. ❓ Dúvidas e Decisões

### 6.1. ✅ Já decididas

| Dúvida | Decisão |
| --- | --- |
| 🖥️ Qual backend? | C#/.NET Web API, uma única API para as disciplinas de C# e de Serviços |
| 📷 Onde roda a IA que avalia a foto? | Modelo de visão local, na VM Linux. Falta escolher o modelo (leve, para caber na memória da VM) |
| 🌧️ Como será a previsão de chuva na demo? | Valor simulado, no mesmo formato do dado real. O treino do modelo de ML continua com dados reais |
| 🔐 O app terá login? | Sim, login e cadastro, com dados mínimos e senha só como hash |
| 💾 Como o app guarda dados? | SQLite local no aparelho, mais a API |
| 📡 Quantos bueiros no ESP32 da demo? | Um bueiro monitorado. O bueiroId continua no contrato para permitir mais depois |
| 🙋 Quem usa o app e quem usa o sensor? | O sensor detecta automaticamente. O app é usado pelos moradores que veem lixo que o sistema não detectou |
| ♻️ Algo de aulas anteriores será reaproveitado? | Não, tudo começa do zero |
| 🏷️ Nome do projeto | DrenaAi, provisório (outras ideias: Pluvia e Escoa) |

### 6.2. ⏳ Ainda pendentes

| # | Decisão | Opções | Responsável | Prazo |
| --- | --- | --- | --- | --- |
| 1 | Qual modelo de visão local | modelo pré-treinado (classificação por regras ou zero-shot) ou ajuste fino. Sem imagens rotuladas de lixo, o pré-treinado tende a ser mais viável | João | 10/11 |
| 2 | Escala da severidade | 0 a 1 com faixas nomeadas (proposta) ou 1 a 5 | Vini e João | 10/11 |
| 3 | Pesos do IPC e teto de chuva | proposta: 0,4 / 0,3 / 0,3 e 50 mm | Vini, João e Rafael | 11/11 |
| 4 | Estrutura do repositório | monorepo com pastas /mobile, /backend, /ml, /esp32, /infra, /seguranca e /docs | todos | 10/11 |
| 5 | Modelo de ML do risco de alagamento | XGBoost, Random Forest, MLP, LSTM ou KMeans + PCA | João | 11/11 |

## 7. 👥 Responsabilidades dos Integrantes

| Integrante | Disciplinas | Papel e responsabilidades |
| --- | --- | --- |
| Bibas 👩‍💻 | Mobile Development & IOT | 📱 App em React Native: login, abas, formulário, detalhe, SQLite local, acessibilidade, README com prints e vídeo de até 3 min |
| Vini Boy 🧑‍💻 | C# Software Development e Arquitetura Orientada a Serviços | ⚙️ API única em C#: entidades, endpoints, IPC com xUnit, EF Core com migrações, Swagger |
| João 🧠 | Inteligência Artificial & ML e Operating Systems | 🧠 Modelo de risco de alagamento e modelo de visão local; 🐧 scripts de Linux, monitoramento e vídeo de até 7 min |
| Rafael 🔒 | Cybersecurity e Testing, Compliance & QA | 🔒 Parsing de PII, Taint Analysis, LLM de comunicados, dashboard Risco Cidadão; ✅ Azure Boards e pitch |
| Pedro 📡 | Physical Computing | 📡 Circuito ESP32 no Wokwi com sensores, LEDs, buzzer e telemetria, vídeo de até 4 min |

## 8. 📦 Entregáveis Esperados

Prazo: 19/11 às 23h55, no Teams. Só um integrante envia cada entrega (sugestão: o responsável da disciplina). A apresentação final do projeto é o cenário de referência da seção 2.2.

| Disciplina | O que entregar | Formato |
| --- | --- | --- |
| 📱 Mobile | Repositório com código, README com instruções e prints; vídeo de até 3 min | link do GitHub |
| ⚙️ C# | Repositório com projeto, Web API, EF Core, README, Swagger ou Scalar e evidências | link do GitHub |
| 🌐 Serviços (SOA) | Repositório com código, README com instruções, descrição e nomes dos integrantes, Swagger e evidências | link do GitHub |
| 🧠 IA & ML | Pasta ml/ (notebook executado, dados/README.md, requirements.txt), RELATORIO_ML.md de 2 a 4 páginas e vídeo de até 5 min | links do GitHub e do YouTube (não listado) |
| 🐧 Operating Systems | Repositório com scripts Bash, configurações, evidências e README; vídeo de até 7 min | link do GitHub |
| 🔒 Cybersecurity | Código do módulo de parsing, evidências da Taint Analysis, vídeo e print do dashboard Risco Cidadão | arquivo ZIP no Teams |
| 📡 Physical Computing | Link do projeto no Wokwi, vídeo de até 4 min e arquivo .txt com o nome dos integrantes | links e arquivo |
| ✅ Testing | Features e Test Cases no Azure Boards com o professor como Admin, link do projeto no PDF e pitch de até 4 min | PDF e link do YouTube |
