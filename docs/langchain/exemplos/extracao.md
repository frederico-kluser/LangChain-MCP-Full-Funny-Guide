# Exemplo: Extração de Informações

Este exemplo demonstra como extrair informações estruturadas de textos não estruturados usando LangChain.

## Código (JavaScript)

```javascript
import { ChatOpenAI } from "langchain/chat_models/openai";
import { StructuredOutputParser } from "langchain/output_parsers";
import { PromptTemplate } from "langchain/prompts";
import { z } from "zod";

// Definir o esquema de dados para extração
const personSchema = z.object({
  nome: z.string().describe("O nome completo da pessoa"),
  idade: z.number().describe("A idade da pessoa em anos"),
  profissao: z.string().describe("A profissão ou ocupação atual da pessoa"),
  habilidades: z.array(z.string()).describe("Lista de habilidades ou competências da pessoa"),
  contato: z.object({
    email: z.string().optional().describe("O endereço de email da pessoa, se mencionado"),
    telefone: z.string().optional().describe("O número de telefone da pessoa, se mencionado"),
  }).describe("Informações de contato disponíveis"),
});

// Criar o parser de saída estruturada
const parser = StructuredOutputParser.fromZodSchema(personSchema);

// Definir o modelo LLM
const modelo = new ChatOpenAI({
  temperature: 0, // Zero para respostas determinísticas
  modelName: "gpt-3.5-turbo",
  openAIApiKey: process.env.OPENAI_API_KEY,
});

// Criar o template de prompt
const promptTemplate = new PromptTemplate({
  template: `
  Extraia as informações estruturadas do texto a seguir.
  
  {format_instructions}
  
  Texto: {texto}
  `,
  inputVariables: ["texto"],
  partialVariables: {
    format_instructions: parser.getFormatInstructions(),
  },
});

// Função para extrair dados
async function extrairInformacoes(texto) {
  try {
    // Formatar o prompt
    const prompt = await promptTemplate.format({ texto });
    
    // Executar o modelo
    const resultado = await modelo.invoke(prompt);
    
    // Fazer o parse da resposta
    const dadosExtraidos = await parser.parse(resultado.content);
    
    return dadosExtraidos;
  } catch (erro) {
    console.error("Erro na extração:", erro);
    return null;
  }
}

// Função de demonstração
async function demonstrar() {
  const textoExemplo = `
  João Silva é um engenheiro de software de 34 anos que trabalha na empresa TechSolutions. 
  Ele tem experiência com JavaScript, Python, React e Cloud Computing. Além disso, 
  João é fluente em inglês e espanhol. Você pode entrar em contato com ele pelo 
  email joao.silva@email.com ou pelo telefone (11) 98765-4321.
  `;
  
  console.log("Texto original:");
  console.log(textoExemplo);
  console.log("\nExtraindo informações...");
  
  const informacoes = await extrairInformacoes(textoExemplo);
  
  console.log("\nInformações extraídas:");
  console.log(JSON.stringify(informacoes, null, 2));
}

demonstrar();
```

## Explicado passo a passo

1. **Definição do esquema**:
   - Usamos a biblioteca Zod para definir um esquema de dados tipado
   - Define campos obrigatórios e opcionais com descrições

2. **Configuração do parser e modelo**:
   - `StructuredOutputParser` garante que a saída do LLM seja analisada corretamente
   - Modelo configurado com temperatura zero para respostas consistentes

3. **Template de prompt**:
   - Inclui instruções para o LLM
   - Incorpora as instruções de formatação geradas automaticamente

4. **Função de extração**:
   - Formata o prompt com base no texto de entrada
   - Envia para o modelo
   - Processa e valida a resposta

5. **Exemplo de uso**:
   - Demonstra como extrair dados de um texto de exemplo

## Resultado esperado

```json
{
  "nome": "João Silva",
  "idade": 34,
  "profissao": "engenheiro de software",
  "habilidades": [
    "JavaScript",
    "Python",
    "React",
    "Cloud Computing",
    "inglês",
    "espanhol"
  ],
  "contato": {
    "email": "joao.silva@email.com",
    "telefone": "(11) 98765-4321"
  }
}
```

## Casos de uso para extração estruturada

1. **Processamento de currículos**
   - Extrair informações de candidatos automaticamente
   - Padronizar dados para sistemas de RH

2. **Análise de documentos legais**
   - Extrair cláusulas, partes envolvidas, datas
   - Identificar obrigações e condições

3. **Processamento de relatórios médicos**
   - Extrair diagnósticos, medicamentos, procedimentos
   - Estruturar informações para prontuários eletrônicos

4. **Análise de feedback de clientes**
   - Extrair sentimentos, tópicos, produtos mencionados
   - Identificar problemas recorrentes e sugestões

## Customizações possíveis

1. **Esquemas mais complexos**:
   ```javascript
   const empresaSchema = z.object({
     nome: z.string(),
     setor: z.string(),
     fundacao: z.number(),
     funcionarios: z.array(
       z.object({
         cargo: z.string(),
         departamento: z.string(),
       })
     ),
     localizacoes: z.array(z.string()),
   });
   ```

2. **Usar diferentes formatos de saída**:
   ```javascript
   import { OutputFixingParser, JsonOutputParser } from "langchain/output_parsers";
   
   // Parser para JSON direto
   const jsonParser = new JsonOutputParser();
   
   // Parser com correção automática
   const robustParser = OutputFixingParser.fromParser(parser);
   ```

3. **Combinar com divisão de documento**:
   ```javascript
   import { RecursiveCharacterTextSplitter } from "langchain/text_splitter";
   
   // Para documentos longos, dividir e processar em chunks
   const splitter = new RecursiveCharacterTextSplitter({
     chunkSize: 2000,
     chunkOverlap: 200,
   });
   
   async function processarDocumentoLongo(documento) {
     const chunks = await splitter.createDocuments([documento]);
     const resultados = [];
     
     for (const chunk of chunks) {
       const info = await extrairInformacoes(chunk.pageContent);
       resultados.push(info);
     }
     
     return resultados;
   }
   ```

4. **Transformações de dados pós-extração**:
   ```javascript
   function normalizarDados(dados) {
     // Converte nomes para lowercase
     dados.nome = dados.nome.toLowerCase();
     
     // Normaliza telefone
     if (dados.contato.telefone) {
       dados.contato.telefone = dados.contato.telefone.replace(/\D/g, '');
     }
     
     // Remove duplicações em habilidades
     dados.habilidades = [...new Set(dados.habilidades)];
     
     return dados;
   }
   ```