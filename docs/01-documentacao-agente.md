# Documentação do Agente

## Caso de Uso

### Problema

> Qual problema financeiro seu agente resolve?

Muitas pessoas físicas perdem o controle dos seus gastos por falta de visibilidade e organização financeira no dia a dia. Sem um acompanhamento claro de onde o dinheiro está indo, é comum gastar mais do que o planejado, esquecer despesas recorrentes e não conseguir poupar. O problema não é apenas falta de disciplina — é falta de uma ferramenta acessível, conversacional e proativa que ajude a entender e gerenciar as finanças pessoais sem fricção.

### Solução

> Como o agente resolve esse problema de forma proativa?

O agente atua como um assistente financeiro pessoal conversacional, permitindo que o usuário registre gastos por linguagem natural (ex: "gastei R$ 45 no mercado hoje"), categorize despesas automaticamente, consulte resumos por período e receba alertas quando uma categoria estiver próxima do limite definido. De forma proativa, o agente identifica padrões de consumo, sugere ajustes no orçamento e antecipa possíveis estouros de categoria antes que aconteçam.

### Público-Alvo

> Quem vai usar esse agente?

Pessoas físicas (adultos entre 20–45 anos) que querem ter mais controle financeiro sem precisar usar planilhas complexas ou aplicativos com curva de aprendizado alta. O perfil típico é alguém que já tentou controlar gastos, mas desistiu por falta de praticidade. O agente precisa ser acessível, sem jargões financeiros, e funcionar como uma conversa natural.

---

## Persona e Tom de Voz

### Nome do Agente

**Clara** — nome simples, feminino, que remete a clareza e transparência. Fácil de lembrar e acolhedor.

### Personalidade

> Como o agente se comporta?

Clara é **consultiva e educativa, mas sem ser pedante**. Ela não julga hábitos de consumo — ela orienta. Tem uma postura proativa: não espera ser perguntada para avisar que o orçamento de lazer está quase no limite. É direta nas respostas, mas calorosa no tom. Pensa no longo prazo do usuário, não apenas na transação atual.

Traços principais:
- Empática: reconhece que finanças pessoais podem gerar ansiedade
- Organizada: sempre estrutura as informações de forma clara
- Proativa: antecipa problemas e sugere ações antes de serem pedidas
- Honesta: quando não tem dados suficientes para responder, diz claramente

### Tom de Comunicação

> Formal, informal, técnico, acessível?

**Informal e acessível**, com leve toque profissional. Usa linguagem do cotidiano brasileiro, evita termos técnicos financeiros sem explicação, e adapta a complexidade da resposta ao contexto da pergunta.

### Exemplos de Linguagem

- **Saudação:** "Oi! Sou a Clara, sua assistente de finanças. Como posso te ajudar hoje?"
- **Registro de gasto:** "Anotei! R$ 45,00 em Mercado no dia de hoje. Quer que eu categorize como Alimentação?"
- **Alerta proativo:** "Atenção: você já usou 87% do seu orçamento de Lazer este mês. Faltam R$ 38,00 para o limite."
- **Resumo mensal:** "Aqui está seu resumo de abril: você gastou R$ 2.340,00 no total. As maiores categorias foram Alimentação (34%) e Transporte (21%)."
- **Confirmação:** "Entendido! Deixa eu verificar isso nos seus registros."
- **Erro / Limitação:** "Não tenho esse dado no momento. Você pode me informar o valor e eu registro agora?"
- **Sem dados suficientes:** "Ainda não tenho histórico suficiente para te dar uma análise confiável, mas em mais algumas semanas consigo identificar padrões!"

---

## Arquitetura

### Diagrama

```mermaid
flowchart TD
    A[Usuário] -->|Mensagem em linguagem natural| B[Interface\nStreamlit]
    B --> C[LLM\nClaude API]
    C --> D[Base de Conhecimento\nJSON / SQLite local]
    D --> C
    C --> E[Módulo de Validação\nChecagem de contexto e limites]
    E --> F[Resposta\nFormato texto + resumo]
    F --> B
```

### Componentes

| Componente | Descrição |
|------------|-----------|
| Interface | Chatbot em Streamlit — permite entrada de texto e exibe respostas formatadas com tabelas e gráficos simples |
| LLM | Claude 3 Haiku ou Sonnet via Anthropic API — responsável por interpretar a mensagem, extrair intenção e gerar resposta |
| Base de Conhecimento | Arquivo JSON ou banco SQLite local contendo transações registradas, categorias, orçamentos e histórico do usuário |
| Módulo de Extração | Função Python que extrai entidades da mensagem (valor, categoria, data) antes de persistir |
| Validação | Checagem de consistência: valor numérico válido, categoria existente, data coerente; aviso quando orçamento for ultrapassado |
| Prompt System | System prompt com instruções de comportamento, formato de resposta esperado e contexto financeiro do usuário |

---

## Segurança e Anti-Alucinação

### Estratégias Adotadas

- [x] O agente só responde com base nos dados financeiros fornecidos e registrados pelo próprio usuário — não inventa transações ou saldos
- [x] Respostas numéricas sempre incluem a fonte ("com base nos seus registros de abril...")
- [x] Quando não tem dados suficientes, a Clara admite explicitamente e orienta o usuário a fornecer a informação
- [x] Não faz recomendações de investimento — o escopo é exclusivamente controle de gastos
- [x] Dados sensíveis (valores, categorias pessoais) são processados localmente; nenhuma informação financeira é enviada a terceiros além da API da Anthropic para geração de resposta
- [x] O sistema prompt instrui o modelo a nunca "adivinhar" valores ou categorias sem confirmação do usuário

### Limitações Declaradas

> O que o agente NÃO faz?

- **Não acessa contas bancárias ou cartões** — todos os dados são inseridos manualmente pelo usuário
- **Não faz recomendações de investimento** — o agente não sugere onde aplicar dinheiro, fundos, ações ou criptomoedas
- **Não prevê o futuro com precisão** — projeções são estimativas baseadas no histórico registrado, não garantias
- **Não se integra a APIs bancárias externas** (Open Finance) nesta versão — isso é uma evolução futura planejada
- **Não tem memória entre sessões por padrão** — o contexto é carregado do arquivo de dados local a cada sessão
- **Não substitui um planejador financeiro profissional** — para decisões financeiras complexas (imóveis, aposentadoria, dívidas de alto valor), o usuário deve consultar um especialista
