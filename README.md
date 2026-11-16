**English** | [Português](README.pt-BR.md)

# 🥗 NutriAI

**An AI-powered nutrition web app that generates personalized meal plans, recipes, shopping lists, and daily tips, grounded in a curated evidence base and served through a Cloudflare Worker proxy with automatic fallback across three LLM providers.**

Built with React 19, TypeScript, Firebase, and Gemini as a **capstone project (TCC)** for the Computer Science program at UNISAGRADO, by a team of three.

> Educational project. It does not replace guidance from a doctor or a registered dietitian.

---

## 🎯 Problem and goal

Generic diet plans ignore the person's body, goal, allergies, and restrictions, while professional follow-up is not always accessible. NutriAI explores how **generative AI can personalize nutrition guidance responsibly**:

- every request is built from a **structured user profile**, not from free-form chat alone;
- every prompt carries a **curated evidence base** (WHO and other public-health guidance) to steer the model toward established recommendations;
- outputs are **structured JSON** that the app can validate, display, save, and export.

The engineering goal was to treat the LLM as **one component of a real product**, with authentication, persistence, error handling, testing, and secure API-key management.

---

## ✨ Features

- **Onboarding and profile:** age, weight, height, sex, goal (*lose weight*, *maintain weight*, *gain muscle*), allergies, and dietary restrictions.
- **Personalized meal plan:** daily meals with descriptions, ingredients in grams, macronutrients (calories, protein, carbs, fat), and a **shopping list**.
- **Meal replacement:** swap a single meal while keeping the rest of the plan.
- **Recipe generator** from a free-text request.
- **Nutrition chat** that knows the user's profile.
- **Daily tip** tailored to the profile.
- **Saved plans and favorite recipes**, with **PDF export** of a plan.
- **Water-intake tracker.**
- **Light and dark themes.**
- **Authentication and persistence** with Firebase Auth and Cloud Firestore, protected by Firestore security rules.

The interface is in Portuguese.

---

## 🏗️ Architecture

```text
┌───────────────────────────────┐
│  React + TypeScript (Vite)    │
│  components / hooks / types   │
└───────┬───────────────┬───────┘
        │               │
        │ Firebase SDK  │ POST { prompt, json }
        ▼               ▼
┌──────────────┐  ┌──────────────────────────────────────────┐
│ Firebase     │  │ Cloudflare Worker proxy (nutriai-proxy)  │
│ Auth +       │  │                                          │
│ Firestore    │  │  1. Gemini 2.5 Flash  ──fails──┐         │
└──────────────┘  │  2. Groq              ◄────────┘──fails─┐│
                  │  3. OpenRouter        ◄─────────────────┘│
                  │                                          │
                  │  strip <think> blocks → extract JSON     │
                  │  API keys stored as Worker secrets       │
                  └──────────────────────────────────────────┘
```

### The AI proxy (`proxy/`)

The meal-plan, recipe, meal-replacement, and daily-tip features call a **Cloudflare Worker** instead of calling the LLM providers from the browser:

- **API keys stay on the server** as Worker secrets and never reach the client bundle.
- **Provider fallback:** the Worker tries **Gemini 2.5 Flash**, then **Groq**, then **OpenRouter**. If one provider is down or rate-limited, the next one answers, and the user never sees the failure. If all three fail, it returns a 502 with each provider's error.
- **JSON mode:** when the app needs structured output, the Worker asks Gemini for `application/json`, removes Markdown fences, and checks that the result parses before returning it. If a provider answers with invalid JSON, the Worker treats it as a failure and moves on to the next provider.
- **Reasoning cleanup:** `<think>…</think>` blocks that some reasoning models emit are removed, so only the final answer reaches the app.
- Models are configured in one place, which makes swapping a free-tier model a one-line change.

### Evidence grounding (`services/nutritionEvidence.ts`)

A small, curated base of **evidence summaries with sources** (for example, WHO guidance on fruit and vegetable intake, free sugars, and fat quality) is written in original wording and **injected into every prompt**. This keeps recommendations aligned with established public-health guidance and makes the grounding auditable.

### Service layer (`services/`)

| Module | Responsibility |
|---|---|
| `geminiService.ts` | Builds the prompts from the profile and the evidence base; meal plan, recipe, meal replacement, daily tip, and chat session; user-friendly error messages |
| `proxyService.ts` | Single point of communication with the Worker |
| `aiService.ts` | Direct provider calls with fallback (used by the parts not yet migrated to the proxy) |
| `nutritionEvidence.ts` | Curated evidence base and prompt context |
| `firebase.ts` | Firebase initialization |

---

## 👨‍💻 My contribution

Team project (3 members). My main contributions were the **AI layer**: the AI services, the integration of the generation features with the profile, and the **Cloudflare Worker proxy** with server-side keys and multi-provider fallback, plus related changes to the onboarding and main app flow.

---

## 🛠️ Tech stack

| Area | Technologies |
|---|---|
| Frontend | React 19, TypeScript, Vite, Tailwind CSS |
| AI | Google Gemini (`@google/genai`), Groq, OpenRouter |
| Backend / infrastructure | Cloudflare Workers (Wrangler) |
| Data and auth | Firebase Authentication, Cloud Firestore, Firestore security rules |
| Other | jsPDF (PDF export) |
| Quality | Vitest, ESLint, Prettier, `tsc --noEmit` |

---

## ▶️ Running locally

### 1. App

```bash
git clone https://github.com/pedromcd/NutriAI.git
cd NutriAI
npm install
cp .env.example .env
```

Fill in `.env` with your Firebase project settings, add the proxy URL, and optionally add provider keys for the features that still call providers directly:

```env
VITE_FIREBASE_API_KEY=...
VITE_FIREBASE_AUTH_DOMAIN=...
VITE_FIREBASE_PROJECT_ID=...
VITE_FIREBASE_STORAGE_BUCKET=...
VITE_FIREBASE_MESSAGING_SENDER_ID=...
VITE_FIREBASE_APP_ID=...

VITE_PROXY_URL=https://nutriai-proxy.<your-subdomain>.workers.dev
VITE_GEMINI_API_KEY=...
```

```bash
npm run dev
```

See `FIREBASE_SETUP.md` for the Firebase configuration.

### 2. AI proxy

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
npm run dev        # development server
npm run build      # production build
npm run lint       # ESLint
npm run typecheck  # TypeScript checks
npm run test       # Vitest
npm run format     # Prettier
npm run deploy     # build and deploy with Wrangler
```

---

## 🧩 Structure

```text
NutriAI/
├── components/          # Screens: Auth, Onboarding, Home, Planner, Recipes, Chatbot, SavedPlans, WaterTracker…
├── hooks/               # useAuth, useFirestore, useTheme, local-storage hooks
├── services/            # AI services, proxy client, evidence base, Firebase (+ tests)
├── proxy/               # Cloudflare Worker: multi-provider LLM proxy (+ tests)
├── types.ts             # UserProfile, MealPlan, Recipe, NutritionInfo…
├── firestore.rules      # Firestore security rules
└── vite.config.ts / vitest.config.ts / wrangler.jsonc
```

---

## 🚧 Limitations and next steps

- **Finish the migration to the proxy:** the chat still calls Gemini from the browser, so its key is in the client bundle. Moving it to the Worker removes all keys from the frontend.
- **Harden the proxy:** restrict CORS to the app's domain and add rate limiting per user.
- **Evaluate the outputs:** check generated plans against the profile (calorie targets, allergies, restrictions) automatically and measure how often a plan violates a constraint.
- **Retrieval-augmented generation (RAG):** retrieve only the evidence relevant to each request from a larger, trusted source base, instead of sending the whole base in every prompt.

---

## 👥 Authors

- Pedro Marques Correa Domingues
- João Matheus Veríssimo Francisco
- Thiago Galli

[Portfolio](https://pedromcd.github.io) · [GitHub](https://github.com/pedromcd)
