# 30 Days of React — 2026 Edition

> **A modern, hooks-first curriculum using React 19, Vite, and npm**

This guide is a complete rewrite of the original 30 Days of React series, updated for 2026. It assumes no prior React knowledge but does expect basic JavaScript familiarity (Day 1 covers the JS you need).

---

## What's New Compared to the 2020 Guide

| Topic | 2020 Guide | 2026 Guide |
|---|---|---|
| React version | 16.13 | 19.x |
| Build tool | Create React App | **Vite** |
| Package manager | Yarn | **npm** |
| Component model | Class + Functional | **Functional only (hooks-first)** |
| Forms | Basic controlled inputs | React Hook Form + Zod + React 19 Actions |
| Data fetching | Fetch / Axios + useEffect | **TanStack Query v5** |
| Routing | React Router v5 | **React Router v7** |
| Styling | Inline / basic CSS | **Tailwind CSS v4 / CSS Modules** |
| State management | useState / setState | useState + **Zustand / Jotai** |
| Testing | Not covered | **Vitest + React Testing Library** |
| New React features | — | Concurrent, Suspense, `use()`, Actions, RSC intro |

---

## Curriculum Overview

| Day | Topic | Phase |
|---|---|---|
| [01](./01_Day_JavaScript_Refresher/01_javascript_refresher.md) | JavaScript / ES2024+ Refresher | Foundations |
| [02](./02_Day_Introduction_to_React/02_introduction_to_react.md) | Introduction to React 19 | Foundations |
| [03](./03_Day_Tooling_Vite_npm/03_tooling_vite_npm.md) | Tooling: Vite + npm | Foundations |
| [04](./04_Day_JSX_Deep_Dive/04_jsx_deep_dive.md) | JSX Deep Dive | Foundations |
| [05](./05_Day_Functional_Components/05_functional_components.md) | Functional Components | Core Concepts |
| [06](./06_Day_Props/06_props.md) | Props | Core Concepts |
| [07](./07_Day_useState/07_use_state.md) | useState | Core Concepts |
| [08](./08_Day_Lists_and_Keys/08_lists_and_keys.md) | Rendering Lists & Keys | Core Concepts |
| [09](./09_Day_Conditional_Rendering/09_conditional_rendering.md) | Conditional Rendering | Core Concepts |
| [10](./10_Day_useEffect/10_use_effect.md) | useEffect | Core Concepts |
| [11](./11_Day_Events/11_events.md) | Events | Core Concepts |
| [12](./12_Day_Forms_Controlled_Inputs/12_forms_controlled_inputs.md) | Forms: Controlled Inputs | Forms & Interactivity |
| [13](./13_Day_React19_Actions/13_react19_actions.md) | Forms: React 19 Actions | Forms & Interactivity |
| [14](./14_Day_Validation_React_Hook_Form/14_validation_react_hook_form.md) | Validation & React Hook Form | Forms & Interactivity |
| [15](./15_Day_Project_Folder_Structure/15_project_folder_structure.md) | Project Folder Structure | Project Architecture |
| [16](./16_Day_Styling_in_React/16_styling_in_react.md) | Styling in React | Project Architecture |
| [17](./17_Day_React_Router_v7/17_react_router_v7.md) | React Router v7 | Routing & Data |
| [18](./18_Day_Data_Fetching/18_data_fetching.md) | Data Fetching Patterns | Routing & Data |
| [19](./19_Day_TanStack_Query/19_tanstack_query.md) | TanStack Query (React Query v5) | Routing & Data |
| [20](./20_Day_useReducer/20_use_reducer.md) | useReducer & Complex State | Advanced Hooks |
| [21](./21_Day_Context_API/21_context_api.md) | Context API | Advanced Hooks |
| [22](./22_Day_Custom_Hooks/22_custom_hooks.md) | Custom Hooks | Advanced Hooks |
| [23](./23_Day_useRef_DOM_Access/23_use_ref.md) | useRef & DOM Access | Advanced Hooks |
| [24](./24_Day_Performance_Optimization/24_performance_optimization.md) | Performance Optimization | Advanced Hooks |
| [25](./25_Day_Suspense_Concurrent/25_suspense_concurrent.md) | Suspense & Concurrent Features | React 19 |
| [26](./26_Day_use_Hook_Server_Components/26_use_hook_server_components.md) | `use()` Hook & Server Components | React 19 |
| [27](./27_Day_Global_State_Management/27_global_state_management.md) | Global State Management | State & Testing |
| [28](./28_Day_Testing/28_testing.md) | Testing React Apps | State & Testing |
| [29](./29_Day_Capstone_Project/29_capstone_project.md) | Capstone Project | Projects |
| [30](./30_Day_Whats_Next/30_whats_next.md) | What's Next & The Ecosystem | Wrap-up |

---

## How to Use This Guide

### Prerequisites

- Basic knowledge of HTML and CSS
- Familiarity with JavaScript fundamentals (Day 1 will cover everything you need for React)
- Node.js 20+ and npm 10+ installed

### Running the Boilerplate

Each day references the shared boilerplate as a starting point. To set it up once:

```bash
cd boilerplate
npm install
npm run dev
```

Then open `http://localhost:5173` in your browser.

For each lesson, you will edit `src/App.jsx` (and create additional files) following the examples in the lesson markdown.

### Recommended Daily Workflow

1. Read the lesson `.md` file
2. Copy the boilerplate to a working directory: `cp -r boilerplate my-day-XX`
3. Run `npm install && npm run dev` inside your copy
4. Type out the code examples (don't copy-paste — muscle memory matters)
5. Complete the exercises at the end of the lesson
6. Compare your solution with the reference answers

---

## 🧡🧡🧡 HAPPY CODING 🧡🧡🧡

[Start Day 1 >>](./01_Day_JavaScript_Refresher/01_javascript_refresher.md)
