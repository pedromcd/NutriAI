[English](README.md) | **Português**

# 🥗 NutriAI

**Aplicação web de nutrição com IA que gera planos alimentares, receitas, listas de compras e dicas diárias personalizados, embasados em uma base de evidências curada e servidos por um proxy em Cloudflare Worker com fallback automático entre três provedores de LLM.**

Construída com React 19, TypeScript, Firebase e Gemini como **Trabalho de Conclusão de Curso (TCC)** de Ciência da Computação na UNISAGRADO, por uma equipe de três pessoas.

> Projeto educacional. Não substitui a orientação de um médico ou nutricionista.

---

## 🎯 Problema e objetivo

Dietas genéricas ignoram o corpo, o objetivo, as alergias e as restrições de cada pessoa, e o acompanhamento profissional nem sempre é acessível. O NutriAI explora como a **IA generativa pode personalizar orientações nutricionais de forma responsável**:

- cada requisição é montada a partir de um **perfil estruturado do usuário**, e não só de mensagens livres no chat;
- cada prompt leva uma **base de evidências curada** (orientações da OMS e de outras fontes de saúde pública) para guiar o modelo em direção a recomendações consolidadas;
- as respostas vêm em **JSON estruturado**, que o app consegue validar, exibir, salvar e exportar.

O objetivo de engenharia foi tratar o LLM como **um componente de um produto real**, com autenticação, persistência, tratamento de erros, testes e gestão segura das chaves de API.

---

## ✨ Funcionalidades

- **Onboarding e perfil:** idade, peso, altura, sexo, objetivo (*perder peso*, *manter o peso*, *ganhar massa muscular*), alergias e restrições alimentares.
- **Plano alimentar personalizado:** refeições do dia com descrição, ingredientes em gramas, macronutrientes (calorias, proteínas, carboidratos e gorduras) e **lista de compras**.
- **Troca de refeição:** substitui uma refeição específica mantendo o restante do plano.
- **Gerador de receitas** a partir de um pedido em texto livre.
- **Chat de nutrição** que conhece o perfil do usuário.
- **Dica diária** de acordo com o perfil.
- **Planos salvos e receitas favoritas**, com **exportação do plano em PDF**.
- **Controle de consumo de água.**
- **Tema claro e escuro.**
- **Autenticação e persistência** com Firebase Auth e Cloud Firestore, protegidas por regras de segurança do Firestore.

---

## 🏗️ Arquitetura

```text
┌───────────────────────────────┐
│  React + TypeScript (Vite)    │
│  components / hooks / types   │
└───────┬───────────────┬───────┘
        │               │
        │ SDK Firebase  │ POST { prompt, json }
        ▼               ▼
┌──────────────┐  ┌──────────────────────────────────────────┐
│ Firebase     │  │ Proxy Cloudflare Worker (nutriai-proxy)  │
│ Auth +       │  │                                          │
│ Firestore    │  │  1. Gemini 2.5 Flash  ──falhou──┐        │
└──────────────┘  │  2. Groq              ◄─────────┘─falhou┐│
                  │  3. OpenRouter        ◄─────────────────┘│
                  │                                          │
                  │  remove blocos <think> → extrai o JSON   │
                  │  chaves de API guardadas como secrets    │
                  └──────────────────────────────────────────┘
```

### O proxy de IA (`proxy/`)

As funções de plano alimentar, receita, troca de refeição e dica diária chamam um **Cloudflare Worker** em vez de chamar os provedores de LLM direto do navegador:

- **As chaves de API ficam no servidor**, como secrets do Worker, e nunca chegam ao bundle do cliente.
- **Fallback entre provedores:** o Worker tenta o **Gemini 2.5 Flash**, depois o **Groq** e depois a **OpenRouter**. Se um provedor estiver fora do ar ou com limite atingido, o próximo responde, e o usuário nem percebe a falha. Se os três falharem, ele retorna 502 com o erro de cada provedor.
- **Modo JSON:** quando o app precisa de saída estruturada, o Worker pede `application/json` ao Gemini, remove as cercas de Markdown e confere se o resultado é um JSON válido antes de devolvê-lo. Se um provedor responder com JSON inválido, o Worker trata como falha e passa para o próximo.
- **Limpeza do raciocínio:** blocos `<think>…</think>` que alguns modelos de raciocínio geram são removidos, para que só a resposta final chegue ao app.
- Os modelos ficam configurados em um único lugar, então trocar um modelo gratuito é uma mudança de uma linha.

### Embasamento em evidências (`services/nutritionEvidence.ts`)

Uma base pequena e curada de **resumos de evidências com fonte** (por exemplo, orientações da OMS sobre consumo de frutas e verduras, açúcares livres e qualidade das gorduras) foi escrita com redação própria e é **inserida em todos os prompts**. Isso mantém as recomendações alinhadas com as orientações de saúde pública e deixa o embasamento auditável.

### Camada de serviços (`services/`)

| Módulo | Responsabilidade |
|---|---|
| `geminiService.ts` | Monta os prompts a partir do perfil e da base de evidências; plano alimentar, receita, troca de refeição, dica diária e sessão de chat; mensagens de erro amigáveis |
| `proxyService.ts` | Ponto único de comunicação com o Worker |
| `aiService.ts` | Chamadas diretas aos provedores, com fallback (usado pelas partes ainda não migradas para o proxy) |
| `nutritionEvidence.ts` | Base de evidências curada e contexto para os prompts |
| `firebase.ts` | Inicialização do Firebase |

---

## 👨‍💻 Minha contribuição

Projeto em equipe (3 integrantes). Minhas principais contribuições foram na **camada de IA**: os serviços de IA, a integração das funções de geração com o perfil e o **proxy em Cloudflare Worker**, com chaves no servidor e fallback entre provedores, além de ajustes no onboarding e no fluxo principal do app.

---

## 🛠️ Tecnologias

| Área | Tecnologias |
|---|---|
| Frontend | React 19, TypeScript, Vite, Tailwind CSS |
| IA | Google Gemini (`@google/genai`), Groq, OpenRouter |
| Backend / infraestrutura | Cloudflare Workers (Wrangler) |
| Dados e autenticação | Firebase Authentication, Cloud Firestore, regras de segurança do Firestore |
| Outros | jsPDF (exportação em PDF) |
| Qualidade | Vitest, ESLint, Prettier, `tsc --noEmit` |

---

## ▶️ Como executar localmente

### 1. App

```bash
git clone https://github.com/pedromcd/NutriAI.git
cd NutriAI
npm install
cp .env.example .env
```

Preencha o `.env` com as configurações do seu projeto Firebase, adicione a URL do proxy e, se quiser, as chaves dos provedores para as funções que ainda chamam os provedores diretamente:

```env
VITE_FIREBASE_API_KEY=...
VITE_FIREBASE_AUTH_DOMAIN=...
VITE_FIREBASE_PROJECT_ID=...
VITE_FIREBASE_STORAGE_BUCKET=...
VITE_FIREBASE_MESSAGING_SENDER_ID=...
VITE_FIREBASE_APP_ID=...

VITE_PROXY_URL=https://nutriai-proxy.<seu-subdominio>.workers.dev
VITE_GEMINI_API_KEY=...
```

```bash
npm run dev
```

Veja o `FIREBASE_SETUP.md` para a configuração do Firebase.

### 2. Proxy de IA

```bash
cd proxy
npm install
npx wrangler secret put GEMINI_API_KEY
npx wrangler secret put GROQ_API_KEY
npx wrangler secret put OPENROUTER_API_KEY
npx wrangler deploy
```

### Scripts

```bash
npm run dev        # servidor de desenvolvimento
npm run build      # build de produção
npm run lint       # ESLint
npm run typecheck  # verificação do TypeScript
npm run test       # Vitest
npm run format     # Prettier
npm run deploy     # build e deploy com Wrangler
```

---

## 🧩 Estrutura

```text
NutriAI/
├── components/          # Telas: Auth, Onboarding, Home, Planner, Recipes, Chatbot, SavedPlans, WaterTracker…
├── hooks/               # useAuth, useFirestore, useTheme, hooks de localStorage
├── services/            # Serviços de IA, cliente do proxy, base de evidências, Firebase (+ testes)
├── proxy/               # Cloudflare Worker: proxy de LLM com vários provedores (+ testes)
├── types.ts             # UserProfile, MealPlan, Recipe, NutritionInfo…
├── firestore.rules      # Regras de segurança do Firestore
└── vite.config.ts / vitest.config.ts / wrangler.jsonc
```

---

## 🚧 Limitações e próximos passos

- **Terminar a migração para o proxy:** o chat ainda chama o Gemini direto do navegador, então a chave dele fica no bundle do cliente. Levá-lo para o Worker remove todas as chaves do frontend.
- **Reforçar o proxy:** restringir o CORS ao domínio do app e adicionar limite de requisições por usuário.
- **Avaliar as respostas:** conferir automaticamente os planos gerados contra o perfil (meta de calorias, alergias, restrições) e medir com que frequência um plano viola alguma restrição.
- **RAG (Retrieval-Augmented Generation):** buscar só as evidências relevantes para cada pedido em uma base maior de fontes confiáveis, em vez de enviar a base inteira em todo prompt.

---

## 👥 Autores

- Pedro Marques Correa Domingues
- João Matheus Veríssimo Francisco
- Thiago Galli

[Portfólio](https://pedromcd.github.io) · [GitHub](https://github.com/pedromcd)
