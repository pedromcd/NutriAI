[English](README.md) | **Português**

# NutriAI

**Plataforma de planejamento nutricional assistida por IA, construída com React, TypeScript, Firebase e Google Gemini.**

O NutriAI é uma aplicação web que combina dados estruturados do usuário com IA generativa para criar planos alimentares, receitas, listas de compras e orientações nutricionais personalizados. O projeto explora como integrar um LLM em uma aplicação real, mantendo estado do usuário, autenticação, persistência, testes e conteúdo baseado em evidências separados em camadas de software bem definidas.

> Este projeto é educacional e não substitui orientação médica ou nutricional profissional.

## Por que este projeto

O principal objetivo de engenharia não é apenas chamar a API de um LLM, mas integrar a IA a um fluxo de produto completo:

1. coletar informações e preferências estruturadas do usuário;
2. persistir os dados da aplicação com segurança;
3. montar prompts e requisições de IA a partir do contexto do usuário;
4. retornar saídas nutricionais úteis e estruturadas;
5. manter a lógica de IA isolada do código de interface e de persistência;
6. testar a camada de serviços de IA e o comportamento relacionado da aplicação.

## Principais funcionalidades

- Autenticação de usuários e gestão de perfil
- Geração de planos alimentares personalizados
- Geração de receitas
- Suporte a listas de compras
- Substituições de alimentos e sugestões nutricionais
- Chat com IA
- Planos salvos e receitas favoritas
- Controle diário de consumo de água
- Persistência com Firebase

## Camada de IA

A funcionalidade de IA fica separada em módulos de serviço dedicados, incluindo:

- `services/geminiService.ts` — integração com o Gemini e lógica de prompts/aplicação
- `services/aiService.ts` — abstração da aplicação para recursos de IA
- `services/nutritionEvidence.ts` — evidências/contexto nutricional usados pela aplicação
- testes automatizados da camada de serviços de IA com Vitest

Essa separação facilita testar o comportamento relacionado à IA independentemente da interface e evoluir a lógica de modelo/prompts sem acoplá-la aos componentes React.

## Arquitetura

```text
Interface do usuário
    ↓
Aplicação React / TypeScript
    ↓
Hooks e serviços da aplicação
    ├── Serviços de IA do Gemini
    ├── Evidências/contexto nutricional
    └── Serviços do Firebase
            ↓
    Autenticação + Firestore
```

O repositório é organizado em componentes de interface modulares, hooks, serviços, tipos compartilhados e estado da aplicação.

```text
NutriAI/
├── components/             # Componentes e telas da interface
├── hooks/                  # Lógica reutilizável de aplicação/estado
├── services/
│   ├── aiService.ts
│   ├── geminiService.ts
│   ├── nutritionEvidence.ts
│   ├── firebase.ts
│   └── *.test.ts           # Testes dos serviços de IA
├── App.tsx
├── MainApp.tsx
├── types.ts
├── firestore.rules
├── vite.config.ts
└── vitest.config.ts
```

## Stack

**Frontend**
- React 19
- TypeScript
- Vite
- Tailwind CSS

**IA**
- Google Gemini via `@google/genai`

**Dados e autenticação**
- Firebase Authentication
- Cloud Firestore
- Regras de segurança do Firestore

**Qualidade e deploy**
- Vitest
- ESLint
- Prettier
- Configuração Cloudflare / Wrangler

## Considerações de engenharia

### Contexto estruturado da aplicação
As requisições de IA são geradas a partir de informações explícitas do usuário, como objetivos, restrições alimentares e preferências, em vez de depender apenas de mensagens livres no chat.

### Separação de responsabilidades
A integração com IA, o acesso ao Firebase, os componentes de interface e os hooks reutilizáveis ficam em módulos separados.

### Testabilidade
O repositório inclui testes automatizados do comportamento dos serviços de IA e scripts para lint, verificação de tipos e testes.

### Design de produto orientado à segurança
A aplicação trata o conteúdo gerado como orientação, não como diagnóstico ou tratamento profissional, e mantém as evidências/contexto nutricional separados do código de geração do modelo.

## Configuração local

Clone o repositório e instale as dependências:

```bash
git clone https://github.com/pedromcd/NutriAI.git
cd NutriAI
npm install
```

Crie um arquivo `.env` com base no `.env.example` e configure as variáveis necessárias do Gemini e do Firebase.

Depois execute:

```bash
npm run dev
```

## Scripts disponíveis

```bash
npm run dev        # servidor de desenvolvimento
npm run build      # build de produção
npm run preview    # preview local
npm run lint       # verificações do ESLint
npm run typecheck  # verificações do TypeScript
npm run test       # testes automatizados
npm run format     # Prettier
npm run deploy     # build e deploy com Wrangler
```

## Possíveis extensões técnicas / de pesquisa

O projeto pode ir além do desenvolvimento de produto e abordar temas de avaliação de IA, como:

- comparar estratégias de prompt quanto à consistência e ao respeito às restrições;
- avaliar a confiabilidade das saídas estruturadas;
- embasar as respostas geradas em evidências nutricionais curadas;
- medir taxas de alucinação e de afirmações sem embasamento;
- comparar modelos quanto à qualidade da personalização;
- adicionar retrieval-augmented generation (RAG) sobre fontes nutricionais confiáveis.

Essas direções são especialmente relevantes para meus interesses mais amplos em sistemas de LLM confiáveis, recuperação de informação e IA aplicada.

## Autor

**Pedro Marques Correa Domingues**  
Graduando em Ciência da Computação, Brasil  
[Portfólio](https://pedromcd.github.io) · [GitHub](https://github.com/pedromcd)
