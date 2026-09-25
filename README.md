# NutriAI

**AI-assisted nutrition planning platform built with React, TypeScript, Firebase, and Google Gemini.**

NutriAI is a web application that combines structured user data with generative AI to create personalized meal plans, recipes, grocery lists, and nutrition-oriented guidance. The project explores how an LLM can be integrated into a real application while keeping user state, authentication, persistence, testing, and evidence-oriented content separated into clear software layers.

> This project is educational and is not a substitute for professional medical or nutritional advice.

## Why this project

The main engineering goal is not only to call an LLM API, but to integrate AI into a complete product flow:

1. collect structured user information and preferences;
2. persist application data securely;
3. build prompts and AI requests from user context;
4. return useful, structured nutrition-oriented outputs;
5. keep AI logic isolated from UI and persistence code;
6. test the AI service layer and related application behavior.

## Main features

- User authentication and profile management
- Personalized meal-plan generation
- Recipe generation
- Grocery-list support
- Food substitutions and nutrition-oriented suggestions
- AI-powered chat experience
- Saved plans and favorite recipes
- Daily water-intake tracking
- Firebase-backed persistence

## AI layer

The AI functionality is separated into dedicated service modules, including:

- `services/geminiService.ts` — Gemini integration and prompt/application logic
- `services/aiService.ts` — AI-related application abstraction
- `services/nutritionEvidence.ts` — nutrition evidence/context used by the application
- automated tests for the AI service layer using Vitest

This separation makes it easier to test AI-related behavior independently from the interface and to evolve model/prompt logic without tightly coupling it to React components.

## Architecture

```text
User Interface
    ↓
React / TypeScript application
    ↓
Application hooks and services
    ├── Gemini AI services
    ├── Nutrition evidence/context
    └── Firebase services
            ↓
    Authentication + Firestore
```

The repository is organized around modular UI components, hooks, services, shared types, and application-level state.

```text
NutriAI/
├── components/             # UI components and screens
├── hooks/                  # Reusable application/state logic
├── services/
│   ├── aiService.ts
│   ├── geminiService.ts
│   ├── nutritionEvidence.ts
│   ├── firebase.ts
│   └── *.test.ts           # AI service tests
├── App.tsx
├── MainApp.tsx
├── types.ts
├── firestore.rules
├── vite.config.ts
└── vitest.config.ts
```

## Tech stack

**Frontend**
- React 19
- TypeScript
- Vite
- Tailwind CSS

**AI**
- Google Gemini via `@google/genai`

**Data and authentication**
- Firebase Authentication
- Cloud Firestore
- Firestore security rules

**Quality and deployment**
- Vitest
- ESLint
- Prettier
- Cloudflare / Wrangler configuration

## Engineering considerations

### Structured application context
AI requests are generated from explicit user information such as goals, dietary restrictions, and preferences instead of relying only on free-form chat input.

### Separation of concerns
AI integration, Firebase access, UI components, and reusable hooks are kept in separate modules.

### Testability
The repository includes automated tests for AI service behavior and provides scripts for linting, type checking, and testing.

### Safety-oriented product design
The application treats generated content as guidance rather than professional diagnosis or treatment and keeps nutrition evidence/context separate from model-generation code.

## Local setup

Clone the repository and install dependencies:

```bash
git clone https://github.com/pedromcd/NutriAI.git
cd NutriAI
npm install
```

Create a `.env` file based on `.env.example` and configure the required Gemini and Firebase variables.

Then run:

```bash
npm run dev
```

## Available scripts

```bash
npm run dev        # development server
npm run build      # production build
npm run preview    # local preview
npm run lint       # ESLint checks
npm run typecheck  # TypeScript checks
npm run test       # automated tests
npm run format     # Prettier
npm run deploy     # build and deploy with Wrangler
```

## Possible research / technical extensions

The project can be extended beyond product development into AI evaluation topics such as:

- comparing prompt strategies for consistency and constraint adherence;
- evaluating structured-output reliability;
- grounding generated responses in curated nutrition evidence;
- measuring hallucination and unsupported-claim rates;
- comparing models on personalization quality;
- adding retrieval-augmented generation (RAG) over trusted nutrition sources.

These directions are especially relevant to my broader interests in reliable LLM systems, information retrieval, and applied AI.

## Author

**Pedro Marques Correa Domingues**  
B.Sc. Computer Science candidate, Brazil  
[Portfolio](https://pedromcd.github.io) · [GitHub](https://github.com/pedromcd)
