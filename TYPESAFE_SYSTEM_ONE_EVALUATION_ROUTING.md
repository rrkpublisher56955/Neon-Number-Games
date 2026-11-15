# TypeSafe System One: Evaluation & Dynamic Routing Architecture
## Cognitive Game Pacing, Mathematical Intent Parsing, and Anti-Cheat Telemetry Routing in Neon Number Games

> **Abstract:** This specification outlines the architectural integration of **TypeSafe System One models** (such as *Jev*) into the *Neon Number Games* runtime pipeline. By transforming raw player interaction streams (keystroke intervals, solution latency, streak stability, and natural language prompts) into typed semantic judgments and calibrated probabilities, the system dynamically routes game modes, modulates mathematical difficulty curves, and validates score authenticity without stalling the deterministic 60 FPS rendering pipeline.

---

## 1. Architectural Overview

Traditional math mini-games rely on hardcoded heuristics (e.g., `if (streak > 5) increaseTimer()`) which feel robotic, predictable, and fail to accommodate differing cognitive strengths. TypeSafe System One models supply **programmable common sense**—fast, focused cognitive judgments returned as typed records and probability distributions that code combines into fluid game loops.

```mermaid
flowchart TD
    subgraph Client ["Client Engine (60 FPS Web App)"]
        A["Player Inputs: Keystrokes, Taps, NLP Queries"] --> B["Interaction Telemetry Accumulator"]
        B --> C["Core Game Loop (Canvas, Web Audio, UI)"]
        B -->|Periodic Telemetry Batch| D["TypeSafe Ingestion Buffer"]
    end

    subgraph TypeSafe ["TypeSafe System One Evaluator"]
        D --> E["Game & Telemetry State Context"]
        E --> F1["Choice: Natural Language Intent Router"]
        E --> F2["Choice: Cognitive Flow State Assessor"]
        E --> F3["Score: Math Fluency & Difficulty Rater"]
        E --> F4["Noul: Human Input Authenticity (Anti-Cheat)"]
    end

    subgraph Dispatcher ["Adaptive Routing Engine"]
        F1 -->|Direct Route| G1["Mode Switcher: Checker / Blitz / Prime / Pattern"]
        F2 & F3 -->|Modulate Timers & Problem Complexity| G2["Procedural Problem Generator"]
        G2 --> C
        F4 -->|Verify Integrity (P > 0.85)| H["YouTube Playables Engagement Score API"]
    end
```

---

## 2. Ingestion State Schema

Every 3 to 5 seconds or upon completing a round, the client telemetry accumulator compiles the `GameplayState` snapshot:

```typescript
export interface TypeSafeNumberGameState {
  sessionId: string;
  sessionDurationSeconds: number;
  activeGameMode: 'checker' | 'guessing' | 'blitz' | 'highlow' | 'prime' | 'pattern';
  currentScore: number;
  currentStreak: number;
  highestStreak: number;

  // Natural Language Intent Query (optional, when typed by user)
  naturalLanguageQuery?: string;

  // Cognitive & Motor Telemetry
  roundAttemptCount: number;
  averageResponseLatencyMs: number;
  latencyStandardDeviationMs: number;
  keystrokeEntropy: number; // 0.0 (robotic fixed delay) to 1.0 (natural human typing jitter)
  errorRate: number; // fraction between 0.0 and 1.0
  consecutiveErrorCount: number;

  // Mathematical Domain Telemetry
  recentOperationsAttempted: Array<'+' | '-' | '*' | '/' | 'prime' | 'sequence'>;
  fastestCalculationTimeMs: number;
  hesitationOnNegativeNumbers: boolean;
  hesitationOnPrimes: boolean;
}
```

---

## 3. System One Evaluation Primitives

TypeSafe provides three core judgment primitives applied to our state:

### 3.1 Natural Language Intent & Mode Router (`Choice` Primitive)
* **Goal:** Parse natural language requests (e.g., *"give me quick mental arithmetic"*, *"let's test my memory with number patterns"*, or *"is 1047 prime?"*) and route the user to the optimal tab.
* **Primitive:** `Choice`
* **Options:**
  - `number_inspector`: User wants to analyze a specific number (sign, parity, factors, prime status).
  - `quantum_guessing`: User wants a hot/cold search game between 1 and 100.
  - `mental_math_blitz`: User wants rapid arithmetic (+, -, *, /) under timer pressure.
  - `high_low_surge`: User wants probability comparison on whether the next number is higher or lower.
  - `prime_hunter`: User wants rapid-fire prime number identification.
  - `pattern_master`: User wants sequence completion and arithmetic/geometric progression puzzles.

```json
{
  "question": "Given the player's text request `state.naturalLanguageQuery`, determine the intended math game mode.",
  "options": [
    "number_inspector",
    "quantum_guessing",
    "mental_math_blitz",
    "high_low_surge",
    "prime_hunter",
    "pattern_master"
  ]
}
```

---

### 3.2 Flow State & Cognitive Engagement (`Choice` Primitive)
* **Goal:** Determine if the player is under-stimulated (bored), in optimal psychological flow, or cognitively overloaded.
* **Primitive:** `Choice`
* **Options:**
  - `under_stimulated`: High accuracy (>95%), ultra-fast response latency (<800ms), steady streak. Player requires shorter timers, larger numbers, or composite operators.
  - `optimal_flow`: Balanced streak (3–8), consistent latency (1200ms–2200ms), engaged focus. Maintain current progression pace.
  - `overwhelmed`: Multiple sequential mistakes, high latency spike (>4000ms), hesitation on basic operations. Relax timer decay and introduce visual hints.

```json
{
  "instructions": "Evaluate the player's mental engagement and stress level based on `state.errorRate`, `state.averageResponseLatencyMs`, and `state.consecutiveErrorCount`.",
  "criteria": {
    "under_stimulated": "Player completes calculations almost instantly with near-zero errors; risks becoming disengaged.",
    "optimal_flow": "Player exhibits active concentration with minor mistakes, maintaining steady rhythm.",
    "overwhelmed": "Player exhibits consecutive wrong answers and prolonged latency; cognitive load is currently exceeding comfort."
  }
}
```

---

### 3.3 Math Fluency & Difficulty Rater (`Score` Primitive)
* **Goal:** Return a calibrated 5-tier skill rating to govern dynamic problem generation.
* **Primitive:** `Score`
* **Levels:**
  1. `Level 1 (Apprentice)`: Single-digit addition and subtraction; no time limit pressure.
  2. `Level 2 (Junior Calculator)`: Double-digit addition/subtraction, single-digit multiplication; 8s timer.
  3. `Level 3 (Speed Analyst)`: Multi-operator arithmetic, simple division with clean remainders, prime checks under 50; 5s timer.
  4. `Level 4 (Mathlete)`: Mixed operations with order-of-precedence, primes up to 100, arithmetic progressions with negative deltas; 3.5s timer.
  5. `Level 5 (Quantum Prodigy)`: Multi-step sequence extrapolation (Fibonacci, geometric squares), 2-digit multiplication, primes up to 200; 2.5s timer.

---

### 3.4 Human Authenticity Verification (`Noul` Primitive)
* **Goal:** Prevent automated DOM scripts or automated calculators from corrupting YouTube Playables leaderboards.
* **Primitive:** `Noul` ($P(\text{human}) \in [0.0, 1.0]$)
* **Instructions:** Evaluates whether `state.keystrokeEntropy`, latency variance, and tap position coordinates reflect organic human neuromuscular variance or artificial clockwork timing.

```json
{
  "instructions": "Assess whether the observed gameplay telemetry reflects authentic human motor execution or synthetic automation scripts.",
  "criteria": "High human probability corresponds to natural variance in inter-keystroke intervals (80-350ms standard deviation) and variable error recovery. Synthetic patterns display uniform timing (<5ms deviation) or instant superhuman answer rates."
}
```

---

## 4. End-to-End Routing Implementation

Below is the production TypeScript implementation using the TypeSafe HTTP API:

```typescript
import { TypeSafeNumberGameState } from './types';

export interface TypeSafeEvaluationResult {
  routedGameMode?: string;
  flowState: 'under_stimulated' | 'optimal_flow' | 'overwhelmed';
  difficultyScore: number; // 1.0 to 5.0
  isAuthenticHuman: boolean;
  confidence: number;
}

export class TypeSafeGameOrchestrator {
  private readonly apiUrl: string;
  private readonly apiKey: string;

  constructor(apiKey: string, apiUrl = 'https://api.typesafe.ai/v1/evaluate') {
    this.apiKey = apiKey;
    this.apiUrl = apiUrl;
  }

  public async evaluateAndRoute(
    state: TypeSafeNumberGameState
  ): Promise<TypeSafeEvaluationResult> {
    const payload = {
      model: 'jev-system-one',
      state: state,
      questions: [
        // 1. Natural Language Intent Routing (if prompt provided)
        ...(state.naturalLanguageQuery ? [{
          id: 'intent_routing',
          primitive: 'choice',
          instructions: 'Classify user request to the matching game mode.',
          options: [
            'number_inspector',
            'quantum_guessing',
            'mental_math_blitz',
            'high_low_surge',
            'prime_hunter',
            'pattern_master'
          ]
        }] : []),

        // 2. Cognitive Flow Evaluation
        {
          id: 'flow_evaluation',
          primitive: 'choice',
          instructions: 'Judge cognitive engagement channel from recent error rates and latency.',
          options: ['under_stimulated', 'optimal_flow', 'overwhelmed']
        },

        // 3. Mathematical Fluency Score
        {
          id: 'fluency_score',
          primitive: 'score',
          instructions: 'Rate the player mathematical fluency level from 1 (Novice) to 5 (Quantum Prodigy).',
          levels: [1, 2, 3, 4, 5]
        },

        // 4. Anti-Cheat Authenticity Noul
        {
          id: 'authenticity_check',
          primitive: 'noul',
          instructions: 'Evaluate if the input entropy reflects an authentic human player.'
        }
      ]
    };

    const response = await fetch(this.apiUrl, {
      method: 'POST',
      headers: {
        'Content-Type': 'application/json',
        'Authorization': `Bearer ${this.apiKey}`
      },
      body: JSON.stringify(payload)
    });

    if (!response.ok) {
      throw new Error(`TypeSafe API request failed with status ${response.status}`);
    }

    const data = await response.json();

    const answers = data.answers;
    return {
      routedGameMode: answers.intent_routing?.choice,
      flowState: answers.flow_evaluation.choice as any,
      difficultyScore: answers.fluency_score.score,
      isAuthenticHuman: answers.authenticity_check.probability >= 0.80,
      confidence: answers.flow_evaluation.confidence
    };
  }
}
```

---

## 5. Client Integration with Dynamic Difficulty Adjustment (DDA)

When `TypeSafeGameOrchestrator` returns results:

1. **If `flowState === 'under_stimulated'`:**
   - Decrease Math Blitz countdown duration by 20% (minimum 2.0s).
   - Expand number generation bounds from $[1, 20]$ to $[10, 99]$.
   - Inject multiplication and division operators.

2. **If `flowState === 'overwhelmed'`:**
   - Increase countdown duration by 35%.
   - Limit operations to addition and subtraction.
   - Display visual hints (e.g., breakdown into rounded tens).

3. **If `isAuthenticHuman === true`:**
   - Authorize score dispatch to `window.ytgame.engagement.sendScore({ value: score })`.
   - Update verified high score in persistent storage.

---

## 6. Summary of Benefits

| Metric | Traditional Heuristic | TypeSafe System One Architecture |
|---|---|---|
| **Pacing Adaptability** | Hardcoded if/else rules | Smooth, calibrated continuous probability scores |
| **Input Routing** | Regex keywords (fragile) | Semantic natural language intent routing (`Choice`) |
| **Cheat Resistance** | Basic client rate-limiting | Neuromuscular entropy verification (`Noul`) |
| **Integration Overhead** | Heavy custom ML weights (ONNX) | Lightweight, single-roundtrip typed System One API |
