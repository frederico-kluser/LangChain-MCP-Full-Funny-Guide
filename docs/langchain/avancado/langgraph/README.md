# LangGraph

LangGraph é uma biblioteca para criar aplicações estatísticas complexas utilizando LLMs. Permite construir grafos de execução que coordenam chamadas a LLMs e outras ferramentas para resolver problemas que exigem multiplas etapas de raciocínio.

## O que é LangGraph?

LangGraph estende o LangChain fornecendo:

- Framework para fluxos com estado (stateful)
- Estruturas baseadas em grafos para coordenar LLMs
- Ciclos de feedback e iteração
- Rastreabilidade e introspectabilidade

## Casos de Uso

- **Agentes com memória persistente** - Manter estado entre interações
- **Workflows multietapas** - Dividir problemas complexos em etapas gerenciáveis
- **Raciocínio step-by-step** - Implementar técnicas como Chain-of-Thought
- **Sistemas multiagente** - Coordenar múltiplos agentes especializados

## Exemplo Básico

```javascript
import { ChatOpenAI } from "langchain/chat_models/openai";
import { LangGraphFunction, StateGraph, END } from "langchain/langgraph";

// Define o modelo
const llm = new ChatOpenAI({ temperature: 0 });

// Define nós do grafo
const plantadorDeProblemFunction = LangGraphFunction.from({
  name: "planejador",
  description: "Planeja a solução para um problema",
  input_schema: { type: "object", properties: { problema: { type: "string" } } },
  output_schema: { type: "object", properties: { passos: { type: "array" } } },
  func: async ({ problema, state }) => {
    const result = await llm.invoke(`Crie um plano para resolver o seguinte problema: ${problema}. Retorne uma lista de passos.`);
    const passos = parsePassos(result.content);
    return { passos };
  },
});

const executorFunction = LangGraphFunction.from({
  name: "executor",
  description: "Executa cada passo do plano",
  input_schema: { type: "object", properties: { passos: { type: "array" } } },
  output_schema: { type: "object", properties: { solucao: { type: "string" } } },
  func: async ({ passos, state }) => {
    let solucao = "";
    for (const passo of passos) {
      const resultado = await llm.invoke(`Execute o seguinte passo: ${passo}. Descreva o resultado.`);
      solucao += `\n- ${passo}: ${resultado.content}`;
    }
    return { solucao };
  },
});

// Cria o grafo
const builder = new StateGraph({
  channels: {
    problema: { value: "" },
    passos: { value: [] },
    solucao: { value: "" },
  },
});

// Adiciona nós
builder.addNode("planejador", plantadorDeProblemFunction);
builder.addNode("executor", executorFunction);

// Define arestas
builder.addEdge("planejador", "executor");
builder.addEdge("executor", END);

// Compila o grafo
const graph = await builder.compile();

// Executa o grafo
const result = await graph.invoke({
  problema: "Como organizar uma biblioteca com 1000 livros?",
});

console.log(result.solucao);
```

## Recursos Avançados

### Ciclos de Feedback

O LangGraph permite criar ciclos onde os agentes podem revisar e melhorar suas próprias respostas:

```javascript
// Adiciona um nó para avaliar a qualidade da solução
const avaliadorFunction = LangGraphFunction.from({
  name: "avaliador",
  func: async ({ solucao, problema, state }) => {
    const avaliacao = await llm.invoke(`Avalie a solução para o problema: ${problema}\n\nSolução: ${solucao}\n\nEsta solução é completa e resolve o problema? Responda com SIM ou NÃO.`);
    return { aprovado: avaliacao.content.includes("SIM") };
  },
});

// Adiciona condição de roteamento
builder.addConditionalEdges(
  "avaliador",
  (state) => state.aprovado ? "fim" : "executor",
  {
    true: END,
    false: "executor"
  }
);
```

### Composição de Grafos

Você pode combinar grafos menores em sistemas mais complexos:

```javascript
// Cria subgrafos que podem ser reutilizados
const planningGraph = createPlanningGraph();
const executionGraph = createExecutionGraph();

// Integra os subgrafos no grafo principal
builder.addGraph("planejamento", planningGraph);
builder.addGraph("execucao", executionGraph);
builder.addEdge("planejamento", "execucao");
```

## Integração com LangSmith

LangGraph se integra com LangSmith para visualização e debugging dos grafos:

```javascript
// Configure LangSmith
process.env.LANGCHAIN_TRACING_V2 = "true";
process.env.LANGCHAIN_API_KEY = "sua-api-key";
process.env.LANGCHAIN_PROJECT = "meu-projeto-langgraph";

// Agora todas as execuções serão rastreadas no LangSmith
```

## Quando Usar LangGraph

Considere usar LangGraph quando:

- Sua aplicação requer raciocínio em múltiplas etapas
- Você precisa manter estado entre interações
- Há necessidade de ciclos de refinamento
- Você está implementando sistemas multiagente
- Deseja visualizar o fluxo de execução de sua aplicação