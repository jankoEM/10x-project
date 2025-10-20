# Architektura InflorAI - Dokument Techniczny

## 1. Przegląd Architektury

### 1.1 Typ Aplikacji
**InflorAI = Single-Purpose Web Application**

Aplikacja webowa typu workflow tool z prostą autentykacją, umożliwiająca przetwarzanie tekstu przez AI, weryfikację danych w edytowalnym polu tekstowym oraz generowanie finalnego cennika.

### 1.2 Architektura Wysokiego Poziomu

```
┌─────────────────────────────────────────────────────────────────┐
│                        Frontend Layer                            │
│                 (Astro 5 + React 19 Islands)                     │
│                                                                   │
│  ┌─────────────┐  ┌──────────────┐  ┌──────────────┐           │
│  │   Static    │  │   React      │  │   React      │           │
│  │   Pages     │  │   Islands    │  │   Islands    │           │
│  │  (Astro)    │  │ (Interactive)│  │ (Interactive)│           │
│  └─────────────┘  └──────────────┘  └──────────────┘           │
└─────────────────────────────────────────────────────────────────┘
                              │
                              ↓
┌─────────────────────────────────────────────────────────────────┐
│                      Integration Layer                           │
│                                                                   │
│  ┌──────────────────┐              ┌─────────────────┐          │
│  │  Supabase Client │              │ Openrouter API  │          │
│  │   (Auth + DB)    │              │  (AI Provider)  │          │
│  └──────────────────┘              └─────────────────┘          │
└─────────────────────────────────────────────────────────────────┘
                              │
                              ↓
┌─────────────────────────────────────────────────────────────────┐
│                       Backend Services                           │
│                                                                   │
│  ┌──────────────────┐              ┌─────────────────┐          │
│  │    Supabase      │              │   Openrouter    │          │
│  │    PostgreSQL    │              │   (Multi-LLM)   │          │
│  │   + RLS + Auth   │              │                 │          │
│  └──────────────────┘              └─────────────────┘          │
└─────────────────────────────────────────────────────────────────┘
```

### 1.3 Kluczowe Decyzje Architektoniczne

#### A. **Hybrid Rendering (Astro + React Islands)**
- **Astro pages** dla statycznych części (marketing, landing, login)
- **React Islands** dla interaktywnych modułów (input, editor, configuration)
- Minimalizacja JavaScript - tylko tam gdzie potrzebna interaktywność

**Uzasadnienie:**
- Lepsza wydajność dla stron statycznych (landing, login)
- Możliwość SSR/SSG dla lepszego SEO (przyszłość: blog, docs)
- Optymalna ilość JS (tylko w islands)

#### B. **State Management Strategy**
- **Zustand** dla globalnego stanu aplikacji (config, processing state)
- **React Query** (TanStack Query) dla cache'owania API calls i synchronizacji z Supabase
- **Local state** (useState) dla izolowanych komponentów
- **Context API** dla współdzielenia stanu w ramach pojedynczego feature

**Uzasadnienie:**
- Zustand: lekki, prosty, bez boilerplate
- React Query: automatyczny cache, refetch, optimistic updates
- Unikamy prop drilling bez complexity Redux

#### C. **Layered Architecture**
```
┌──────────────────────────────────────┐
│   Presentation Layer (Components)    │  ← React Islands + Astro
├──────────────────────────────────────┤
│   Application Layer (Hooks + Logic)  │  ← Custom hooks, orchestration
├──────────────────────────────────────┤
│   Domain Layer (Services)            │  ← Business logic (pure functions)
├──────────────────────────────────────┤
│   Infrastructure Layer (API/DB)      │  ← Supabase, Openrouter clients
└──────────────────────────────────────┘
```

---

## 2. Struktura Projektu

### 2.1 File System Architecture

```
10x-inflorai/
├── .env.example                    # Environment variables template
├── .env.local                      # Local env (gitignored)
├── astro.config.mjs                # Astro configuration
├── tailwind.config.cjs             # Tailwind configuration
├── tsconfig.json                   # TypeScript configuration
├── package.json
├── README.md
│
├── public/                         # Static assets
│   ├── favicon.svg
│   └── images/
│
├── src/
│   ├── env.d.ts                   # Astro environment types
│   │
│   ├── features/                  # Feature-based modules (Feature-First)
│   │   ├── auth/
│   │   │   ├── components/       # Feature-specific React components
│   │   │   │   ├── LoginForm.tsx
│   │   │   │   └── LogoutButton.tsx
│   │   │   ├── hooks/            # Feature-specific hooks
│   │   │   │   ├── useAuth.ts
│   │   │   │   └── useSession.ts
│   │   │   ├── services/         # Business logic
│   │   │   │   └── authService.ts
│   │   │   ├── api/              # Data access
│   │   │   │   └── authApi.ts
│   │   │   ├── types/            # Feature types
│   │   │   │   └── index.ts
│   │   │   └── store/            # Feature state (Zustand)
│   │   │       └── authStore.ts
│   │   │
│   │   ├── input-processing/
│   │   │   ├── components/
│   │   │   │   ├── RawInputArea.tsx
│   │   │   │   └── DatePicker.tsx
│   │   │   ├── hooks/
│   │   │   │   └── useInputProcessing.ts
│   │   │   ├── services/
│   │   │   │   ├── dateExtractor.ts
│   │   │   │   └── inputValidator.ts
│   │   │   ├── types/
│   │   │   │   └── index.ts
│   │   │   └── utils/
│   │   │       └── textHelpers.ts
│   │   │
│   │   ├── configuration/
│   │   │   ├── components/
│   │   │   │   ├── ConfigPanel.tsx
│   │   │   │   ├── CurrencyRatesForm.tsx
│   │   │   │   ├── MarginForm.tsx
│   │   │   │   └── VATForm.tsx
│   │   │   ├── hooks/
│   │   │   │   ├── useConfiguration.ts
│   │   │   │   └── useConfigPersistence.ts
│   │   │   ├── store/
│   │   │   │   └── configStore.ts
│   │   │   ├── services/
│   │   │   │   └── configService.ts
│   │   │   ├── api/
│   │   │   │   └── configApi.ts
│   │   │   └── types/
│   │   │       └── index.ts
│   │   │
│   │   ├── extraction/
│   │   │   ├── components/
│   │   │   │   ├── ExtractionProgress.tsx
│   │   │   │   └── ExtractionError.tsx
│   │   │   ├── hooks/
│   │   │   │   └── useExtraction.ts
│   │   │   ├── services/
│   │   │   │   ├── aiExtractor.ts
│   │   │   │   ├── promptBuilder.ts
│   │   │   │   └── responseParser.ts
│   │   │   ├── api/
│   │   │   │   └── openrouterApi.ts
│   │   │   ├── prompts/
│   │   │   │   ├── system-prompt.md
│   │   │   │   └── few-shot-examples.ts
│   │   │   └── types/
│   │   │       └── index.ts
│   │   │
│   │   ├── validation/
│   │   │   ├── components/
│   │   │   │   ├── ValidationSummary.tsx
│   │   │   │   └── ValidationBadge.tsx
│   │   │   ├── hooks/
│   │   │   │   └── useValidation.ts
│   │   │   ├── services/
│   │   │   │   ├── validator.ts
│   │   │   │   └── issueDetector.ts
│   │   │   ├── types/
│   │   │   │   └── index.ts
│   │   │   └── utils/
│   │   │       └── validationRules.ts
│   │   │
│   │   ├── pricing/
│   │   │   ├── components/
│   │   │   │   └── PriceBreakdown.tsx
│   │   │   ├── hooks/
│   │   │   │   └── usePricing.ts
│   │   │   ├── services/
│   │   │   │   ├── priceCalculator.ts
│   │   │   │   ├── currencyConverter.ts
│   │   │   │   ├── marginCalculator.ts
│   │   │   │   ├── transportCalculator.ts
│   │   │   │   └── vatCalculator.ts
│   │   │   ├── types/
│   │   │   │   └── index.ts
│   │   │   └── __tests__/
│   │   │       └── priceCalculator.test.ts
│   │   │
│   │   ├── editor/
│   │   │   ├── components/
│   │   │   │   ├── TextEditor.tsx
│   │   │   │   ├── EditorToolbar.tsx
│   │   │   │   └── RowHighlighter.tsx
│   │   │   ├── hooks/
│   │   │   │   ├── useEditor.ts
│   │   │   │   └── useEditorValidation.ts
│   │   │   ├── services/
│   │   │   │   ├── textParser.ts
│   │   │   │   └── textFormatter.ts
│   │   │   ├── store/
│   │   │   │   └── editorStore.ts
│   │   │   ├── types/
│   │   │   │   └── index.ts
│   │   │   └── utils/
│   │   │       └── tabHelpers.ts
│   │   │
│   │   ├── output-generation/
│   │   │   ├── components/
│   │   │   │   ├── OutputPreview.tsx
│   │   │   │   ├── GenerateButton.tsx
│   │   │   │   └── WarningPopup.tsx
│   │   │   ├── hooks/
│   │   │   │   └── useOutputGeneration.ts
│   │   │   ├── services/
│   │   │   │   ├── outputFormatter.ts
│   │   │   │   └── clipboardService.ts
│   │   │   └── types/
│   │   │       └── index.ts
│   │   │
│   │   ├── history/
│   │   │   ├── components/
│   │   │   │   ├── HistoryPanel.tsx
│   │   │   │   ├── HistoryList.tsx
│   │   │   │   ├── HistoryItem.tsx
│   │   │   │   └── RestoreButton.tsx
│   │   │   ├── hooks/
│   │   │   │   ├── useHistory.ts
│   │   │   │   └── useHistoryRestore.ts
│   │   │   ├── services/
│   │   │   │   └── historyService.ts
│   │   │   ├── api/
│   │   │   │   └── historyApi.ts
│   │   │   └── types/
│   │   │       └── index.ts
│   │   │
│   │   └── observability/
│   │       ├── hooks/
│   │       │   └── useDeltaTracking.ts
│   │       ├── services/
│   │       │   ├── deltaCalculator.ts
│   │       │   └── metricsCollector.ts
│   │       ├── api/
│   │       │   └── metricsApi.ts
│   │       └── types/
│   │           └── index.ts
│   │
│   ├── components/                # Atomic Design - shared UI components
│   │   ├── atoms/
│   │   │   ├── Button.tsx
│   │   │   ├── Input.tsx
│   │   │   ├── Label.tsx
│   │   │   ├── Badge.tsx
│   │   │   ├── Spinner.tsx
│   │   │   └── Alert.tsx
│   │   │
│   │   ├── molecules/
│   │   │   ├── FormField.tsx
│   │   │   ├── PriceInput.tsx
│   │   │   ├── CurrencySelect.tsx
│   │   │   ├── StatusBadge.tsx
│   │   │   ├── SearchInput.tsx
│   │   │   └── DateInput.tsx
│   │   │
│   │   ├── organisms/
│   │   │   ├── Header.tsx
│   │   │   ├── Sidebar.tsx
│   │   │   ├── ProcessingWorkflow.tsx
│   │   │   └── ErrorBoundary.tsx
│   │   │
│   │   └── templates/
│   │       ├── AppTemplate.tsx
│   │       ├── AuthTemplate.tsx
│   │       └── DashboardTemplate.tsx
│   │
│   ├── layouts/                   # Astro layouts
│   │   ├── BaseLayout.astro
│   │   ├── AuthLayout.astro
│   │   └── AppLayout.astro
│   │
│   ├── pages/                     # Astro pages (routing)
│   │   ├── index.astro           # Landing page
│   │   ├── login.astro           # Login page
│   │   ├── dashboard.astro       # Main application
│   │   └── 404.astro             # Not found
│   │
│   ├── lib/                       # Shared libraries & integrations
│   │   ├── supabase/
│   │   │   ├── client.ts         # Supabase browser client
│   │   │   ├── server.ts         # Supabase server client (SSR)
│   │   │   ├── database.types.ts # Generated types
│   │   │   └── middleware.ts     # Auth middleware
│   │   │
│   │   ├── openrouter/
│   │   │   ├── client.ts         # Openrouter API client
│   │   │   ├── models.ts         # Model configurations
│   │   │   └── errorHandler.ts   # API error handling
│   │   │
│   │   ├── types/                # Global TypeScript types
│   │   │   ├── database.ts       # Database types
│   │   │   ├── api.ts            # API request/response types
│   │   │   └── domain.ts         # Domain models
│   │   │
│   │   └── constants/            # Global constants
│   │       ├── pricing.ts        # Default pricing values
│   │       ├── currencies.ts     # Currency definitions
│   │       └── config.ts         # App configuration
│   │
│   ├── hooks/                     # Shared React hooks
│   │   ├── useToast.ts
│   │   ├── useLocalStorage.ts
│   │   ├── useDebounce.ts
│   │   └── useClickOutside.ts
│   │
│   ├── utils/                     # Global utilities
│   │   ├── formatters.ts         # Number, date, currency formatters
│   │   ├── validators.ts         # Input validators
│   │   ├── calculations.ts       # Math helpers
│   │   ├── parsers.ts            # String parsers
│   │   └── cn.ts                 # Tailwind class merger
│   │
│   └── styles/
│       └── globals.css           # Global styles + Tailwind
│
└── __tests__/                     # Test files
    ├── unit/
    ├── integration/
    └── e2e/
```

### 2.2 Feature Module Pattern

Każdy feature module jest self-contained i składa się z warstw:

```
feature/
├── components/        # Presentation (React components)
├── hooks/            # Application Logic (custom hooks)
├── services/         # Domain Logic (pure functions)
├── api/              # Infrastructure (API calls)
├── store/            # State Management (Zustand)
├── types/            # TypeScript definitions
└── utils/            # Feature-specific utilities
```

**Przepływ danych w module:**
```
Component → Hook → Service → API → External Service
    ↑                                      ↓
    └──────────── State Update ────────────┘
```

---

## 3. Architektura Warstw

### 3.1 Presentation Layer (Components)

#### A. **Astro Pages** (Static/SSR)
```astro
---
// src/pages/dashboard.astro
import AppLayout from '@/layouts/AppLayout.astro'
import ProcessingWorkflow from '@/components/organisms/ProcessingWorkflow'
import { supabase } from '@/lib/supabase/server'

// SSR: Check auth
const session = await supabase.auth.getSession()
if (!session) return Astro.redirect('/login')
---

<AppLayout title="InflorAI Dashboard">
  <!-- Static header -->
  <header class="...">
    <h1>InflorAI - Automated Pricing</h1>
  </header>

  <!-- Interactive island -->
  <ProcessingWorkflow client:load />
</AppLayout>
```

#### B. **React Islands** (Interactive)
```tsx
// src/components/organisms/ProcessingWorkflow.tsx
export default function ProcessingWorkflow() {
  const { config } = useConfiguration()
  const { extract, isLoading } = useExtraction()
  const { rows } = useEditor()
  
  return (
    <div className="workflow">
      <RawInputArea />
      <ConfigPanel />
      <TextEditor rows={rows} />
      <OutputPreview />
    </div>
  )
}
```

**Island Loading Strategies:**
```tsx
// Load immediately (critical UI)
<ConfigPanel client:load />

// Load when visible (below fold)
<HistoryPanel client:visible />

// Load on interaction (modals, dropdowns)
<WarningPopup client:idle />

// No hydration (static only)
<Header client:only="react" />
```

#### C. **Atomic Design Hierarchy**

**Atoms** (podstawowe elementy):
```tsx
// src/components/atoms/Button.tsx
import { cn } from '@/utils/cn'

interface ButtonProps extends React.ButtonHTMLAttributes<HTMLButtonElement> {
  variant?: 'primary' | 'secondary' | 'danger'
  size?: 'sm' | 'md' | 'lg'
}

export function Button({ variant = 'primary', size = 'md', className, ...props }: ButtonProps) {
  return (
    <button
      className={cn(
        'rounded font-medium transition-colors',
        {
          'bg-blue-600 text-white hover:bg-blue-700': variant === 'primary',
          'bg-gray-200 text-gray-800 hover:bg-gray-300': variant === 'secondary',
          'bg-red-600 text-white hover:bg-red-700': variant === 'danger',
        },
        {
          'px-3 py-1 text-sm': size === 'sm',
          'px-4 py-2 text-base': size === 'md',
          'px-6 py-3 text-lg': size === 'lg',
        },
        className
      )}
      {...props}
    />
  )
}
```

**Molecules** (kompozycje atomów):
```tsx
// src/components/molecules/FormField.tsx
interface FormFieldProps {
  label: string
  error?: string
  required?: boolean
  children: React.ReactNode
}

export function FormField({ label, error, required, children }: FormFieldProps) {
  return (
    <div className="form-field">
      <Label>
        {label} {required && <span className="text-red-500">*</span>}
      </Label>
      {children}
      {error && <Alert variant="error">{error}</Alert>}
    </div>
  )
}
```

**Organisms** (złożone komponenty):
```tsx
// src/features/configuration/components/ConfigPanel.tsx
export function ConfigPanel() {
  const { config, updateConfig } = useConfiguration()
  
  return (
    <div className="config-panel">
      <h2>Global Parameters</h2>
      
      <section>
        <h3>Currency Rates</h3>
        <FormField label="EUR to PLN" required>
          <Input 
            type="number" 
            value={config.eurRate} 
            onChange={(e) => updateConfig({ eurRate: Number(e.target.value) })}
          />
        </FormField>
        <FormField label="USD to PLN" required>
          <Input 
            type="number" 
            value={config.usdRate} 
            onChange={(e) => updateConfig({ usdRate: Number(e.target.value) })}
          />
        </FormField>
      </section>
      
      {/* More sections... */}
    </div>
  )
}
```

### 3.2 Application Layer (Hooks)

Custom hooks orkiestrują logikę aplikacji:

```tsx
// src/features/extraction/hooks/useExtraction.ts
import { useMutation } from '@tanstack/react-query'
import { aiExtractor } from '../services/aiExtractor'
import { useConfigStore } from '@/features/configuration/store/configStore'
import { useEditorStore } from '@/features/editor/store/editorStore'

export function useExtraction() {
  const config = useConfigStore((state) => state.config)
  const setRows = useEditorStore((state) => state.setRows)
  
  const mutation = useMutation({
    mutationFn: async (rawInput: string) => {
      // Call AI extraction service
      const extracted = await aiExtractor.extract(rawInput, config)
      return extracted
    },
    onSuccess: (data) => {
      // Update editor with extracted data
      setRows(data)
    },
    onError: (error) => {
      console.error('Extraction failed:', error)
      // Handle error...
    }
  })
  
  return {
    extract: mutation.mutate,
    isLoading: mutation.isPending,
    error: mutation.error,
    data: mutation.data
  }
}
```

### 3.3 Domain Layer (Services)

Pure functions implementing business logic:

```typescript
// src/features/pricing/services/priceCalculator.ts
import type { ExtractedRow, GlobalConfig, ComputedRow } from '../types'
import { currencyConverter } from './currencyConverter'
import { marginCalculator } from './marginCalculator'
import { transportCalculator } from './transportCalculator'
import { vatCalculator } from './vatCalculator'

export class PriceCalculator {
  /**
   * Calculate all price components for a single row
   */
  static calculateRow(row: ExtractedRow, config: GlobalConfig): ComputedRow {
    // 1. Convert base price to PLN
    const basePricePln = currencyConverter.toPLN(
      row.unitPriceValue,
      row.unitPriceCurrency,
      config
    )
    
    // 2. Calculate margin based on length
    const marginPln = marginCalculator.calculate(row.lengthCm, config)
    
    // 3. Calculate transport based on length
    const transportPln = transportCalculator.calculate(row.lengthCm, config)
    
    // 4. Calculate net subtotal
    const subtotalNetPln = basePricePln + marginPln + transportPln
    
    // 5. Calculate VAT
    const vatPln = vatCalculator.calculate(subtotalNetPln, config.vatPercent)
    
    // 6. Calculate gross total
    const totalGrossPln = subtotalNetPln + vatPln
    
    return {
      ...row,
      basePricePln,
      marginPln,
      transportPln,
      subtotalNetPln,
      vatPln,
      totalGrossPln
    }
  }
  
  /**
   * Recalculate all rows (used when config changes)
   */
  static recalculateAll(rows: ExtractedRow[], config: GlobalConfig): ComputedRow[] {
    return rows.map(row => this.calculateRow(row, config))
  }
}
```

**Unit Test Example:**
```typescript
// src/features/pricing/__tests__/priceCalculator.test.ts
import { describe, it, expect } from 'vitest'
import { PriceCalculator } from '../services/priceCalculator'

describe('PriceCalculator', () => {
  const config = {
    eurRate: 4.30,
    usdRate: 3.80,
    vatPercent: 8,
    shortMarginPln: 0.40,
    shortTransportPln: 1.32,
    longMarginPln: 0.60,
    longTransportPln: 1.42,
    lengthThreshold: 60
  }
  
  it('should calculate EUR price with short length correctly', () => {
    const row = {
      id: '1',
      name: 'Rosa Grand Prix',
      lengthCm: 50,
      quantity: 10,
      unitPriceValue: 2.50,
      unitPriceCurrency: 'EUR' as const
    }
    
    const result = PriceCalculator.calculateRow(row, config)
    
    // EUR 2.50 * 4.30 = 10.75 PLN
    expect(result.basePricePln).toBe(10.75)
    
    // Short length: margin 0.40 + transport 1.32
    expect(result.marginPln).toBe(0.40)
    expect(result.transportPln).toBe(1.32)
    
    // Subtotal: 10.75 + 0.40 + 1.32 = 12.47
    expect(result.subtotalNetPln).toBe(12.47)
    
    // VAT 8%: 12.47 * 0.08 = 0.9976
    expect(result.vatPln).toBeCloseTo(1.00, 2)
    
    // Total: 12.47 + 1.00 = 13.47
    expect(result.totalGrossPln).toBeCloseTo(13.47, 2)
  })
})
```

### 3.4 Infrastructure Layer (API/DB)

#### A. **Supabase Integration**

```typescript
// src/lib/supabase/client.ts
import { createClient } from '@supabase/supabase-js'
import type { Database } from './database.types'

const supabaseUrl = import.meta.env.PUBLIC_SUPABASE_URL
const supabaseAnonKey = import.meta.env.PUBLIC_SUPABASE_ANON_KEY

export const supabase = createClient<Database>(supabaseUrl, supabaseAnonKey)
```

```typescript
// src/features/history/api/historyApi.ts
import { supabase } from '@/lib/supabase/client'
import type { ProcessingHistory, ProcessingHistoryInsert } from '../types'

export class HistoryApi {
  /**
   * Save processing history to database
   */
  static async save(data: ProcessingHistoryInsert): Promise<ProcessingHistory> {
    const { data: history, error } = await supabase
      .from('processing_history')
      .insert(data)
      .select()
      .single()
    
    if (error) throw error
    return history
  }
  
  /**
   * Get all history for current user
   */
  static async getAll(): Promise<ProcessingHistory[]> {
    const { data, error } = await supabase
      .from('processing_history')
      .select('*')
      .order('created_at', { ascending: false })
    
    if (error) throw error
    return data
  }
  
  /**
   * Get single history record
   */
  static async getById(id: string): Promise<ProcessingHistory> {
    const { data, error } = await supabase
      .from('processing_history')
      .select('*')
      .eq('id', id)
      .single()
    
    if (error) throw error
    return data
  }
}
```

#### B. **Openrouter Integration**

```typescript
// src/lib/openrouter/client.ts
import type { ChatCompletionRequest, ChatCompletionResponse } from './types'

const OPENROUTER_API_URL = 'https://openrouter.ai/api/v1/chat/completions'
const API_KEY = import.meta.env.OPENROUTER_API_KEY

export class OpenrouterClient {
  /**
   * Send chat completion request
   */
  static async chatCompletion(request: ChatCompletionRequest): Promise<ChatCompletionResponse> {
    const response = await fetch(OPENROUTER_API_URL, {
      method: 'POST',
      headers: {
        'Authorization': `Bearer ${API_KEY}`,
        'Content-Type': 'application/json',
        'HTTP-Referer': window.location.origin,
        'X-Title': 'InflorAI'
      },
      body: JSON.stringify(request)
    })
    
    if (!response.ok) {
      throw new Error(`Openrouter API error: ${response.statusText}`)
    }
    
    return response.json()
  }
}
```

```typescript
// src/features/extraction/services/aiExtractor.ts
import { OpenrouterClient } from '@/lib/openrouter/client'
import { promptBuilder } from './promptBuilder'
import { responseParser } from './responseParser'
import type { ExtractedRow, GlobalConfig } from '../types'

export class AiExtractor {
  /**
   * Extract structured data from raw text using AI
   */
  static async extract(rawInput: string, config: GlobalConfig): Promise<ExtractedRow[]> {
    // Build prompt with few-shot examples
    const prompt = promptBuilder.build(rawInput, config)
    
    // Call Openrouter API
    const response = await OpenrouterClient.chatCompletion({
      model: 'openai/gpt-4o-mini',
      messages: [
        {
          role: 'system',
          content: prompt.system
        },
        {
          role: 'user',
          content: prompt.user
        }
      ],
      response_format: { type: 'json_object' }
    })
    
    // Parse and validate response
    const extracted = responseParser.parse(response.choices[0].message.content)
    
    return extracted
  }
}
```

---

## 4. State Management

### 4.1 Global State (Zustand)

```typescript
// src/features/configuration/store/configStore.ts
import { create } from 'zustand'
import { persist } from 'zustand/middleware'
import type { GlobalConfig } from '../types'

interface ConfigState {
  config: GlobalConfig
  updateConfig: (partial: Partial<GlobalConfig>) => void
  resetConfig: () => void
}

const DEFAULT_CONFIG: GlobalConfig = {
  eurRate: 4.30,
  usdRate: 3.80,
  vatPercent: 8,
  shortMarginPln: 0.40,
  shortTransportPln: 1.32,
  longMarginPln: 0.60,
  longTransportPln: 1.42,
  lengthThreshold: 60
}

export const useConfigStore = create<ConfigState>()(
  persist(
    (set) => ({
      config: DEFAULT_CONFIG,
      
      updateConfig: (partial) => 
        set((state) => ({ 
          config: { ...state.config, ...partial } 
        })),
      
      resetConfig: () => 
        set({ config: DEFAULT_CONFIG })
    }),
    {
      name: 'inflorai-config', // localStorage key
    }
  )
)
```

```typescript
// src/features/editor/store/editorStore.ts
import { create } from 'zustand'
import type { ExtractedRow } from '../types'

interface EditorState {
  rows: ExtractedRow[]
  setRows: (rows: ExtractedRow[]) => void
  updateRow: (id: string, updates: Partial<ExtractedRow>) => void
  deleteRow: (id: string) => void
  addRow: (row: ExtractedRow) => void
}

export const useEditorStore = create<EditorState>((set) => ({
  rows: [],
  
  setRows: (rows) => set({ rows }),
  
  updateRow: (id, updates) => 
    set((state) => ({
      rows: state.rows.map(row => 
        row.id === id ? { ...row, ...updates } : row
      )
    })),
  
  deleteRow: (id) => 
    set((state) => ({
      rows: state.rows.filter(row => row.id !== id)
    })),
  
  addRow: (row) => 
    set((state) => ({
      rows: [...state.rows, row]
    }))
}))
```

### 4.2 Server State (React Query)

```typescript
// src/features/history/hooks/useHistory.ts
import { useQuery, useMutation, useQueryClient } from '@tanstack/react-query'
import { HistoryApi } from '../api/historyApi'
import type { ProcessingHistoryInsert } from '../types'

export function useHistory() {
  const queryClient = useQueryClient()
  
  // Fetch all history
  const query = useQuery({
    queryKey: ['history'],
    queryFn: HistoryApi.getAll,
    staleTime: 1000 * 60 * 5, // 5 minutes
  })
  
  // Save new history
  const saveMutation = useMutation({
    mutationFn: (data: ProcessingHistoryInsert) => HistoryApi.save(data),
    onSuccess: () => {
      // Invalidate and refetch
      queryClient.invalidateQueries({ queryKey: ['history'] })
    }
  })
  
  return {
    history: query.data ?? [],
    isLoading: query.isLoading,
    error: query.error,
    save: saveMutation.mutate,
    isSaving: saveMutation.isPending
  }
}
```

### 4.3 State Architecture Diagram

```
┌─────────────────────────────────────────────────────────────┐
│                      React Components                        │
└─────────────────────────────────────────────────────────────┘
                              │
                 ┌────────────┴────────────┐
                 │                         │
                 ↓                         ↓
┌────────────────────────────┐  ┌────────────────────────┐
│  Zustand (Client State)    │  │ React Query (Server)   │
│  - Configuration           │  │ - History (Supabase)   │
│  - Editor State            │  │ - User Data            │
│  - UI State                │  │ - API Cache            │
└────────────────────────────┘  └────────────────────────┘
                 │                         │
                 ↓                         ↓
┌────────────────────────────┐  ┌────────────────────────┐
│    localStorage            │  │   Supabase DB          │
│    (persistence)           │  │   (PostgreSQL)         │
└────────────────────────────┘  └────────────────────────┘
```

---

## 5. Data Flow

### 5.1 Main Processing Flow

```
┌──────────────┐
│   User       │
│ Pastes Text  │
└──────┬───────┘
       │
       ↓
┌──────────────────────────────────────────────┐
│ 1. Input Processing Feature                 │
│    - Validate text not empty                 │
│    - Extract date if present                 │
│    - Store in input state                    │
└──────┬───────────────────────────────────────┘
       │
       ↓
┌──────────────────────────────────────────────┐
│ 2. Configuration Feature                     │
│    - Load config from Zustand store          │
│    - Allow user to edit rates/margins        │
└──────┬───────────────────────────────────────┘
       │
       ↓ (User clicks "Process")
┌──────────────────────────────────────────────┐
│ 3. Extraction Feature                        │
│    - Build few-shot prompt                   │
│    - Call Openrouter API (GPT-4o-mini)       │
│    - Parse JSON response                     │
│    - Validate structure                      │
│    - Add unique IDs to rows                  │
└──────┬───────────────────────────────────────┘
       │
       ↓
┌──────────────────────────────────────────────┐
│ 4. Validation Feature                        │
│    - Check for missing required fields       │
│    - Flag uncertain extractions              │
│    - Generate issue list                     │
└──────┬───────────────────────────────────────┘
       │
       ↓
┌──────────────────────────────────────────────┐
│ 5. Pricing Feature                           │
│    - Calculate prices for all rows           │
│    - Convert currencies                      │
│    - Apply margins & transport               │
│    - Calculate VAT                           │
└──────┬───────────────────────────────────────┘
       │
       ↓
┌──────────────────────────────────────────────┐
│ 6. Editor Feature                            │
│    - Display as editable text field          │
│    - Highlight problematic rows              │
│    - Allow manual corrections                │
│    - Recalculate on edit                     │
└──────┬───────────────────────────────────────┘
       │
       ↓ (User clicks "Generate")
┌──────────────────────────────────────────────┐
│ 7. Output Generation Feature                 │
│    - Check for remaining issues              │
│    - Show warning popup if issues exist      │
│    - Format as tab-separated text            │
│    - Enable copy/download                    │
└──────┬───────────────────────────────────────┘
       │
       ↓
┌──────────────────────────────────────────────┐
│ 8. History Feature                           │
│    - Save to Supabase                        │
│    - Store: raw input, extracted data,       │
│      edited text, final output               │
└──────┬───────────────────────────────────────┘
       │
       ↓
┌──────────────────────────────────────────────┐
│ 9. Observability Feature                     │
│    - Calculate delta (auto vs final)         │
│    - Log to Supabase                         │
│    - Track metrics                           │
└──────────────────────────────────────────────┘
```

### 5.2 Edit Flow (Real-time Recalculation)

```
User edits cell in text editor
       │
       ↓
onChange event fired
       │
       ↓
Parse text → array of rows
       │
       ↓
Update editorStore.rows
       │
       ↓
useMemo triggers recalculation
       │
       ↓
PriceCalculator.recalculateAll()
       │
       ↓
UI re-renders with new prices
```

### 5.3 History Restore Flow

```
User clicks "Restore" in HistoryPanel
       │
       ↓
Fetch history record from Supabase
       │
       ↓
Load saved data:
  - Raw input → inputStore
  - Edited text → editorStore
  - Config → configStore
       │
       ↓
Trigger recalculation
       │
       ↓
UI updates with restored state
```

---

## 6. Database Schema (Supabase)

### 6.1 Tables

```sql
-- Users table (Supabase Auth automatic)
-- supabase.auth.users

-- User configurations (optional, for future)
CREATE TABLE user_configurations (
  id UUID PRIMARY KEY DEFAULT uuid_generate_v4(),
  user_id UUID REFERENCES auth.users(id) ON DELETE CASCADE,
  eur_rate NUMERIC(10, 2) DEFAULT 4.30,
  usd_rate NUMERIC(10, 2) DEFAULT 3.80,
  vat_percent NUMERIC(5, 2) DEFAULT 8.00,
  short_margin_pln NUMERIC(10, 2) DEFAULT 0.40,
  short_transport_pln NUMERIC(10, 2) DEFAULT 1.32,
  long_margin_pln NUMERIC(10, 2) DEFAULT 0.60,
  long_transport_pln NUMERIC(10, 2) DEFAULT 1.42,
  length_threshold_cm INTEGER DEFAULT 60,
  created_at TIMESTAMPTZ DEFAULT NOW(),
  updated_at TIMESTAMPTZ DEFAULT NOW(),
  
  UNIQUE(user_id)
);

-- Processing history
CREATE TABLE processing_history (
  id UUID PRIMARY KEY DEFAULT uuid_generate_v4(),
  user_id UUID REFERENCES auth.users(id) ON DELETE CASCADE,
  
  -- Input stage
  raw_input TEXT NOT NULL,
  default_date DATE,
  
  -- Extraction stage
  extracted_data JSONB NOT NULL,  -- Array of ExtractedRow
  
  -- Editing stage
  edited_text TEXT NOT NULL,       -- Tab-separated text after user edits
  
  -- Output stage
  final_output TEXT NOT NULL,      -- Final generated pricelist
  
  -- Metadata
  config_snapshot JSONB NOT NULL,  -- GlobalConfig at time of processing
  delta JSONB,                     -- Changes between auto and final
  
  created_at TIMESTAMPTZ DEFAULT NOW(),
  
  -- Indexes for performance
  INDEX idx_user_created (user_id, created_at DESC)
);

-- Row Level Security
ALTER TABLE user_configurations ENABLE ROW LEVEL SECURITY;
ALTER TABLE processing_history ENABLE ROW LEVEL SECURITY;

-- RLS Policies
CREATE POLICY "Users can read own config"
  ON user_configurations FOR SELECT
  USING (auth.uid() = user_id);

CREATE POLICY "Users can update own config"
  ON user_configurations FOR UPDATE
  USING (auth.uid() = user_id);

CREATE POLICY "Users can read own history"
  ON processing_history FOR SELECT
  USING (auth.uid() = user_id);

CREATE POLICY "Users can insert own history"
  ON processing_history FOR INSERT
  WITH CHECK (auth.uid() = user_id);
```

### 6.2 TypeScript Types (Generated)

```bash
# Generate types from Supabase schema
npx supabase gen types typescript --project-id YOUR_PROJECT_ID > src/lib/supabase/database.types.ts
```

```typescript
// src/lib/supabase/database.types.ts (generated)
export interface Database {
  public: {
    Tables: {
      user_configurations: {
        Row: {
          id: string
          user_id: string
          eur_rate: number
          usd_rate: number
          // ... etc
        }
        Insert: {
          id?: string
          user_id: string
          eur_rate?: number
          // ... etc
        }
        Update: {
          eur_rate?: number
          // ... etc
        }
      }
      processing_history: {
        Row: {
          id: string
          user_id: string
          raw_input: string
          extracted_data: Json
          // ... etc
        }
        // ... Insert, Update
      }
    }
  }
}
```

---

## 7. Routing & Navigation

### 7.1 File-based Routing (Astro)

```
src/pages/
├── index.astro              → /             (landing page)
├── login.astro              → /login        (authentication)
├── dashboard.astro          → /dashboard    (main app)
└── 404.astro                → /404          (not found)
```

### 7.2 Protected Routes

```astro
---
// src/pages/dashboard.astro
import { supabase } from '@/lib/supabase/server'

const { data: { session } } = await supabase.auth.getSession()

if (!session) {
  return Astro.redirect('/login')
}
---
```

### 7.3 Client-side Navigation

```tsx
// src/features/auth/components/LogoutButton.tsx
import { supabase } from '@/lib/supabase/client'

export function LogoutButton() {
  const handleLogout = async () => {
    await supabase.auth.signOut()
    window.location.href = '/login'
  }
  
  return <Button onClick={handleLogout}>Logout</Button>
}
```

---

## 8. Authentication Flow

### 8.1 Login Flow

```
1. User navigates to /login
2. LoginForm component renders
3. User enters email + password
4. Call supabase.auth.signInWithPassword()
5. On success: redirect to /dashboard
6. On error: show error message
```

```tsx
// src/features/auth/components/LoginForm.tsx
import { useState } from 'react'
import { supabase } from '@/lib/supabase/client'
import { Button, Input, Alert } from '@/components/atoms'

export function LoginForm() {
  const [email, setEmail] = useState('')
  const [password, setPassword] = useState('')
  const [error, setError] = useState<string | null>(null)
  const [loading, setLoading] = useState(false)
  
  const handleSubmit = async (e: React.FormEvent) => {
    e.preventDefault()
    setLoading(true)
    setError(null)
    
    const { error } = await supabase.auth.signInWithPassword({
      email,
      password
    })
    
    if (error) {
      setError(error.message)
      setLoading(false)
    } else {
      window.location.href = '/dashboard'
    }
  }
  
  return (
    <form onSubmit={handleSubmit}>
      <Input
        type="email"
        placeholder="Email"
        value={email}
        onChange={(e) => setEmail(e.target.value)}
        required
      />
      <Input
        type="password"
        placeholder="Password"
        value={password}
        onChange={(e) => setPassword(e.target.value)}
        required
      />
      {error && <Alert variant="error">{error}</Alert>}
      <Button type="submit" disabled={loading}>
        {loading ? 'Loading...' : 'Login'}
      </Button>
    </form>
  )
}
```

### 8.2 Session Management

```tsx
// src/features/auth/hooks/useSession.ts
import { useEffect, useState } from 'react'
import { supabase } from '@/lib/supabase/client'
import type { Session } from '@supabase/supabase-js'

export function useSession() {
  const [session, setSession] = useState<Session | null>(null)
  const [loading, setLoading] = useState(true)
  
  useEffect(() => {
    // Get initial session
    supabase.auth.getSession().then(({ data: { session } }) => {
      setSession(session)
      setLoading(false)
    })
    
    // Listen for auth changes
    const { data: { subscription } } = supabase.auth.onAuthStateChange(
      (_event, session) => {
        setSession(session)
      }
    )
    
    return () => subscription.unsubscribe()
  }, [])
  
  return { session, loading }
}
```

---

## 9. Error Handling Strategy

### 9.1 Error Types

```typescript
// src/lib/types/errors.ts
export class AppError extends Error {
  constructor(
    message: string,
    public code: string,
    public statusCode?: number
  ) {
    super(message)
    this.name = 'AppError'
  }
}

export class APIError extends AppError {
  constructor(message: string, statusCode: number) {
    super(message, 'API_ERROR', statusCode)
    this.name = 'APIError'
  }
}

export class ValidationError extends AppError {
  constructor(message: string, public field?: string) {
    super(message, 'VALIDATION_ERROR', 400)
    this.name = 'ValidationError'
  }
}

export class ExtractionError extends AppError {
  constructor(message: string) {
    super(message, 'EXTRACTION_ERROR', 500)
    this.name = 'ExtractionError'
  }
}
```

### 9.2 Error Boundaries

```tsx
// src/components/organisms/ErrorBoundary.tsx
import React from 'react'
import { Alert, Button } from '@/components/atoms'

interface Props {
  children: React.ReactNode
  fallback?: React.ReactNode
}

interface State {
  hasError: boolean
  error: Error | null
}

export class ErrorBoundary extends React.Component<Props, State> {
  constructor(props: Props) {
    super(props)
    this.state = { hasError: false, error: null }
  }
  
  static getDerivedStateFromError(error: Error) {
    return { hasError: true, error }
  }
  
  componentDidCatch(error: Error, errorInfo: React.ErrorInfo) {
    console.error('ErrorBoundary caught:', error, errorInfo)
  }
  
  render() {
    if (this.state.hasError) {
      if (this.props.fallback) {
        return this.props.fallback
      }
      
      return (
        <div className="error-boundary">
          <Alert variant="error">
            <h2>Something went wrong</h2>
            <p>{this.state.error?.message}</p>
            <Button onClick={() => window.location.reload()}>
              Reload Page
            </Button>
          </Alert>
        </div>
      )
    }
    
    return this.props.children
  }
}
```

### 9.3 API Error Handling

```typescript
// src/lib/openrouter/errorHandler.ts
import { APIError } from '@/lib/types/errors'

export class OpenrouterErrorHandler {
  static handle(error: unknown): never {
    if (error instanceof Response) {
      switch (error.status) {
        case 401:
          throw new APIError('Invalid API key', 401)
        case 429:
          throw new APIError('Rate limit exceeded', 429)
        case 500:
          throw new APIError('Openrouter server error', 500)
        default:
          throw new APIError(`API error: ${error.statusText}`, error.status)
      }
    }
    
    if (error instanceof Error) {
      throw new APIError(error.message, 500)
    }
    
    throw new APIError('Unknown error', 500)
  }
}
```

---

## 10. Performance Optimizations

### 10.1 Code Splitting

```tsx
// Lazy load heavy components
import { lazy, Suspense } from 'react'

const HistoryPanel = lazy(() => import('@/features/history/components/HistoryPanel'))

export function Dashboard() {
  return (
    <Suspense fallback={<Spinner />}>
      <HistoryPanel />
    </Suspense>
  )
}
```

### 10.2 Memoization

```tsx
// Expensive calculations
import { useMemo } from 'react'
import { PriceCalculator } from '@/features/pricing/services/priceCalculator'

export function useComputedRows() {
  const rows = useEditorStore(state => state.rows)
  const config = useConfigStore(state => state.config)
  
  const computedRows = useMemo(
    () => PriceCalculator.recalculateAll(rows, config),
    [rows, config]
  )
  
  return computedRows
}
```

### 10.3 Debouncing

```tsx
// src/hooks/useDebounce.ts
import { useEffect, useState } from 'react'

export function useDebounce<T>(value: T, delay: number): T {
  const [debouncedValue, setDebouncedValue] = useState(value)
  
  useEffect(() => {
    const handler = setTimeout(() => {
      setDebouncedValue(value)
    }, delay)
    
    return () => clearTimeout(handler)
  }, [value, delay])
  
  return debouncedValue
}
```

### 10.4 Virtual Scrolling (Future)

For large datasets (100+ rows):

```tsx
import { useVirtualizer } from '@tanstack/react-virtual'

// Will implement if performance issues with large data
```

---

## 11. Testing Strategy

### 11.1 Test Pyramid

```
         ┌────────────┐
        /    E2E       \      ← Few (critical user flows)
       /──────────────\
      /  Integration    \     ← Some (feature integration)
     /──────────────────\
    /      Unit          \    ← Many (business logic)
   /──────────────────────\
```

### 11.2 Unit Tests (Vitest)

**Focus:** Pure functions (services, utilities)

```typescript
// src/features/pricing/services/__tests__/currencyConverter.test.ts
import { describe, it, expect } from 'vitest'
import { currencyConverter } from '../currencyConverter'

describe('CurrencyConverter', () => {
  const config = {
    eurRate: 4.30,
    usdRate: 3.80
  }
  
  it('should convert EUR to PLN', () => {
    const result = currencyConverter.toPLN(100, 'EUR', config)
    expect(result).toBe(430)
  })
  
  it('should convert USD to PLN', () => {
    const result = currencyConverter.toPLN(100, 'USD', config)
    expect(result).toBe(380)
  })
  
  it('should return same value for PLN', () => {
    const result = currencyConverter.toPLN(100, 'PLN', config)
    expect(result).toBe(100)
  })
})
```

### 11.3 Integration Tests (Vitest + Testing Library)

**Focus:** Feature workflows, API integration

```typescript
// src/features/extraction/__tests__/extraction.integration.test.ts
import { describe, it, expect, vi } from 'vitest'
import { renderHook, waitFor } from '@testing-library/react'
import { useExtraction } from '../hooks/useExtraction'
import { OpenrouterClient } from '@/lib/openrouter/client'

vi.mock('@/lib/openrouter/client')

describe('Extraction Integration', () => {
  it('should extract data and update editor store', async () => {
    // Mock API response
    vi.mocked(OpenrouterClient.chatCompletion).mockResolvedValue({
      choices: [{
        message: {
          content: JSON.stringify({
            items: [
              {
                name: 'Rosa Grand Prix',
                lengthCm: 60,
                quantity: 10,
                basePrice: 2.50,
                currency: 'EUR'
              }
            ]
          })
        }
      }]
    })
    
    const { result } = renderHook(() => useExtraction())
    
    result.current.extract('Rosa Grand Prix 60cm 10 szt 2.50€')
    
    await waitFor(() => {
      expect(result.current.isLoading).toBe(false)
      expect(result.current.data).toHaveLength(1)
      expect(result.current.data[0].name).toBe('Rosa Grand Prix')
    })
  })
})
```

### 11.4 E2E Tests (Playwright - Future)

**Focus:** Critical user journeys

```typescript
// __tests__/e2e/processing-flow.spec.ts
import { test, expect } from '@playwright/test'

test('complete processing flow', async ({ page }) => {
  // Login
  await page.goto('/login')
  await page.fill('[name="email"]', 'test@example.com')
  await page.fill('[name="password"]', 'password')
  await page.click('button[type="submit"]')
  
  // Navigate to dashboard
  await expect(page).toHaveURL('/dashboard')
  
  // Paste raw input
  await page.fill('textarea[name="raw-input"]', 'Rosa Grand Prix 60cm 10 szt 2.50€')
  
  // Process
  await page.click('button:has-text("Process")')
  
  // Wait for extraction
  await expect(page.locator('.editor')).toBeVisible()
  
  // Verify results
  const editorContent = await page.textContent('.editor')
  expect(editorContent).toContain('Rosa Grand Prix')
  
  // Generate output
  await page.click('button:has-text("Generate")')
  
  // Verify output
  await expect(page.locator('.output')).toBeVisible()
})
```

---

## 12. Deployment Architecture

### 12.1 Development Environment

```
Local Development:
┌──────────────────────────────────────┐
│  Developer Machine                   │
│                                      │
│  ┌────────────┐    ┌──────────────┐ │
│  │ Astro Dev  │    │   Supabase   │ │
│  │  Server    │───▶│    Local     │ │
│  │ :4321      │    │  :54321      │ │
│  └────────────┘    └──────────────┘ │
│                                      │
│  Environment:                        │
│  - .env.local                        │
│  - VITE_SUPABASE_URL (local)        │
│  - OPENROUTER_API_KEY               │
└──────────────────────────────────────┘
```

### 12.2 Production Environment

```
Production (DigitalOcean + Docker):
┌──────────────────────────────────────────────────┐
│  DigitalOcean Droplet                            │
│                                                  │
│  ┌────────────────────────────────────────────┐ │
│  │  Docker Container                          │ │
│  │                                            │ │
│  │  ┌──────────────┐     ┌────────────────┐  │ │
│  │  │  Nginx       │────▶│  Astro Static  │  │ │
│  │  │  (Reverse    │     │  + Node Server │  │ │
│  │  │   Proxy)     │     │                │  │ │
│  │  └──────────────┘     └────────────────┘  │ │
│  └────────────────────────────────────────────┘ │
│                                                  │
│           │                        │             │
│           ↓                        ↓             │
│  ┌─────────────────┐    ┌──────────────────┐   │
│  │  Supabase       │    │   Openrouter.ai  │   │
│  │  (Cloud)        │    │   (Cloud)        │   │
│  └─────────────────┘    └──────────────────┘   │
└──────────────────────────────────────────────────┘
```

### 12.3 Docker Configuration

```dockerfile
# Dockerfile
FROM node:20-alpine AS builder

WORKDIR /app

# Install dependencies
COPY package*.json ./
RUN npm ci

# Copy source
COPY . .

# Build
ENV NODE_ENV=production
RUN npm run build

# Production image
FROM node:20-alpine AS runner

WORKDIR /app

# Copy built assets
COPY --from=builder /app/dist ./dist
COPY --from=builder /app/node_modules ./node_modules
COPY --from=builder /app/package.json ./

ENV HOST=0.0.0.0
ENV PORT=4321

EXPOSE 4321

CMD ["node", "./dist/server/entry.mjs"]
```

```yaml
# docker-compose.yml
version: '3.8'

services:
  app:
    build: .
    ports:
      - "4321:4321"
    environment:
      - NODE_ENV=production
      - PUBLIC_SUPABASE_URL=${PUBLIC_SUPABASE_URL}
      - PUBLIC_SUPABASE_ANON_KEY=${PUBLIC_SUPABASE_ANON_KEY}
      - OPENROUTER_API_KEY=${OPENROUTER_API_KEY}
    restart: unless-stopped
  
  nginx:
    image: nginx:alpine
    ports:
      - "80:80"
      - "443:443"
    volumes:
      - ./nginx.conf:/etc/nginx/nginx.conf:ro
      - ./ssl:/etc/nginx/ssl:ro
    depends_on:
      - app
    restart: unless-stopped
```

### 12.4 CI/CD Pipeline (GitHub Actions)

```yaml
# .github/workflows/deploy.yml
name: Deploy to Production

on:
  push:
    branches: [main]

jobs:
  build-and-deploy:
    runs-on: ubuntu-latest
    
    steps:
      - uses: actions/checkout@v3
      
      - name: Setup Node.js
        uses: actions/setup-node@v3
        with:
          node-version: '20'
          
      - name: Install dependencies
        run: npm ci
        
      - name: Run tests
        run: npm run test
        
      - name: Build
        run: npm run build
        env:
          PUBLIC_SUPABASE_URL: ${{ secrets.SUPABASE_URL }}
          PUBLIC_SUPABASE_ANON_KEY: ${{ secrets.SUPABASE_ANON_KEY }}
          
      - name: Build Docker image
        run: docker build -t inflorai:latest .
        
      - name: Push to DigitalOcean Registry
        run: |
          echo "${{ secrets.DO_API_TOKEN }}" | docker login registry.digitalocean.com -u ${{ secrets.DO_API_TOKEN }} --password-stdin
          docker tag inflorai:latest registry.digitalocean.com/inflorai/app:latest
          docker push registry.digitalocean.com/inflorai/app:latest
          
      - name: Deploy to Droplet
        uses: appleboy/ssh-action@master
        with:
          host: ${{ secrets.DROPLET_IP }}
          username: ${{ secrets.DROPLET_USER }}
          key: ${{ secrets.SSH_PRIVATE_KEY }}
          script: |
            cd /opt/inflorai
            docker-compose pull
            docker-compose up -d
            docker system prune -f
```

---

## 13. Security Considerations

### 13.1 Environment Variables

```bash
# .env.example
# Supabase (Public - safe to expose)
PUBLIC_SUPABASE_URL=https://your-project.supabase.co
PUBLIC_SUPABASE_ANON_KEY=your-anon-key

# Openrouter (Private - server-side only)
OPENROUTER_API_KEY=your-api-key

# App
NODE_ENV=production
```

**Important:**
- ✅ Supabase anon key CAN be public (protected by RLS)
- ❌ Openrouter API key MUST stay server-side
- ❌ NEVER commit .env files to git

### 13.2 Row Level Security (RLS)

All Supabase tables MUST have RLS enabled:

```sql
ALTER TABLE processing_history ENABLE ROW LEVEL SECURITY;

CREATE POLICY "Users can only see their own data"
  ON processing_history
  FOR SELECT
  USING (auth.uid() = user_id);
```

### 13.3 Input Sanitization

```typescript
// src/utils/validators.ts
import DOMPurify from 'isomorphic-dompurify'

export class InputValidator {
  /**
   * Sanitize user input to prevent XSS
   */
  static sanitize(input: string): string {
    return DOMPurify.sanitize(input, {
      ALLOWED_TAGS: [],  // Strip all HTML
      KEEP_CONTENT: true
    })
  }
  
  /**
   * Validate max length
   */
  static validateLength(input: string, max: number): boolean {
    return input.length <= max
  }
}
```

### 13.4 API Rate Limiting

```typescript
// src/lib/openrouter/rateLimit.ts
export class RateLimiter {
  private calls: number[] = []
  
  constructor(
    private maxCalls: number,
    private windowMs: number
  ) {}
  
  canMakeRequest(): boolean {
    const now = Date.now()
    
    // Remove old calls outside window
    this.calls = this.calls.filter(time => now - time < this.windowMs)
    
    return this.calls.length < this.maxCalls
  }
  
  recordRequest(): void {
    this.calls.push(Date.now())
  }
}

// Usage: max 10 calls per minute
const limiter = new RateLimiter(10, 60000)
```

---

## 14. Monitoring & Observability

### 14.1 Logging Strategy

```typescript
// src/lib/logger.ts
type LogLevel = 'debug' | 'info' | 'warn' | 'error'

class Logger {
  log(level: LogLevel, message: string, meta?: Record<string, unknown>) {
    const timestamp = new Date().toISOString()
    
    const logEntry = {
      timestamp,
      level,
      message,
      ...meta
    }
    
    // Development: console
    if (import.meta.env.DEV) {
      console[level === 'debug' ? 'log' : level](logEntry)
    }
    
    // Production: send to monitoring service (future)
    if (import.meta.env.PROD) {
      // TODO: Send to Sentry, Datadog, etc.
    }
  }
  
  debug(message: string, meta?: Record<string, unknown>) {
    this.log('debug', message, meta)
  }
  
  info(message: string, meta?: Record<string, unknown>) {
    this.log('info', message, meta)
  }
  
  warn(message: string, meta?: Record<string, unknown>) {
    this.log('warn', message, meta)
  }
  
  error(message: string, error?: Error, meta?: Record<string, unknown>) {
    this.log('error', message, { ...meta, error: error?.message, stack: error?.stack })
  }
}

export const logger = new Logger()
```

### 14.2 Delta Tracking

```typescript
// src/features/observability/services/deltaCalculator.ts
import type { ExtractedRow } from '@/features/extraction/types'

export class DeltaCalculator {
  /**
   * Calculate differences between automatic extraction and final edited version
   */
  static calculate(
    autoRows: ExtractedRow[],
    finalRows: ExtractedRow[]
  ): RowDelta[] {
    const deltas: RowDelta[] = []
    
    for (let i = 0; i < autoRows.length; i++) {
      const auto = autoRows[i]
      const final = finalRows[i]
      
      const changes: Record<string, { from: unknown; to: unknown }> = {}
      
      // Compare each field
      for (const key in auto) {
        if (auto[key] !== final[key]) {
          changes[key] = {
            from: auto[key],
            to: final[key]
          }
        }
      }
      
      if (Object.keys(changes).length > 0) {
        deltas.push({
          rowId: auto.id,
          changes
        })
      }
    }
    
    return deltas
  }
}
```

### 14.3 Metrics Collection

```typescript
// src/features/observability/services/metricsCollector.ts
export class MetricsCollector {
  /**
   * Collect processing metrics
   */
  static async collectProcessingMetrics(data: {
    userId: string
    processingId: string
    totalRows: number
    autoCorrectRows: number  // Rows with no changes
    editedRows: number       // Rows that were manually edited
    deletedRows: number      // Rows that were removed
    processingTimeMs: number
    extractionTimeMs: number
  }) {
    // Calculate accuracy
    const accuracy = (data.autoCorrectRows / data.totalRows) * 100
    
    // Log metrics
    logger.info('Processing completed', {
      ...data,
      accuracy: `${accuracy.toFixed(2)}%`
    })
    
    // Store in database for analysis
    await MetricsApi.save({
      user_id: data.userId,
      processing_id: data.processingId,
      total_rows: data.totalRows,
      auto_correct_rows: data.autoCorrectRows,
      edited_rows: data.editedRows,
      deleted_rows: data.deletedRows,
      accuracy_percent: accuracy,
      processing_time_ms: data.processingTimeMs,
      extraction_time_ms: data.extractionTimeMs,
      created_at: new Date().toISOString()
    })
  }
}
```

---

## 15. Future Enhancements

### 15.1 Roadmap Items (Post-MVP)

#### Phase 2 (1-2 miesiące):
- **Advanced Table Editor**: Edytowalna tabela zamiast pola tekstowego z tabulatorami
- **Add/Duplicate/Delete Rows**: Zarządzanie wierszami
- **Column Sorting & Filtering**: Sortowanie i filtrowanie w tabeli
- **Export Formats**: CSV, Excel oprócz tekstu

#### Phase 3 (2-3 miesiące):
- **Custom AI Models**: Wybór modelu (GPT-4, Claude, Gemini)
- **Fine-tuning**: Dostrajanie modelu na historycznych danych użytkownika
- **Batch Processing**: Przetwarzanie wielu ofert jednocześnie
- **Email Integration**: Import ofert z emaila

#### Phase 4 (3-6 miesięcy):
- **Multi-user**: Zespoły, współdzielenie cenników
- **API**: REST API dla integracji
- **Mobile App**: React Native lub PWA
- **Advanced Analytics**: Dashboard z metrykami, trendy

### 15.2 Scalability Considerations

Obecna architektura jest przygotowana na wzrost:

- **Horizontal Scaling**: Docker containers na wielu serwerach
- **Database**: PostgreSQL skaluje się dobrze, możliwy sharding w przyszłości
- **Caching**: Redis dla cache'owania wyników AI
- **CDN**: Cloudflare dla statycznych assets
- **Load Balancer**: Nginx dla load balancing

---

## 16. Summary

### 16.1 Key Architectural Decisions

| Decision | Rationale |
|----------|-----------|
| **Astro + React Islands** | Optimal performance, minimal JS, good SEO potential |
| **Zustand** | Simple global state, no boilerplate |
| **React Query** | Excellent server state management, caching |
| **Supabase** | Complete backend solution (auth + db + RLS) |
| **Openrouter** | Access to multiple AI models, cost control |
| **Feature-First Structure** | Scalability, maintainability, clear boundaries |
| **Atomic Design** | Reusable components, consistent UI |
| **Pure Functions** | Testability, predictability |

### 16.2 Non-Functional Requirements

- **Performance**: < 2s initial load, < 500ms for calculations
- **Availability**: 99.5% uptime target
- **Security**: RLS, input sanitization, env secrets
- **Maintainability**: Clear architecture, tests, documentation
- **Scalability**: Prepared for 100+ users initially

### 16.3 Success Metrics

**Technical:**
- Lighthouse score > 90
- Test coverage > 70%
- Build time < 1 minute
- Bundle size < 500 KB

**Business (from PRD):**
- 90% automatic extraction accuracy
- 50% time reduction vs manual process
- < 10% rows requiring manual correction

---

## Appendix A: Quick Start Guide

```bash
# Clone repository
git clone https://github.com/your-org/inflorai.git
cd inflorai

# Install dependencies
npm install

# Setup environment
cp .env.example .env.local
# Edit .env.local with your keys

# Start Supabase locally (optional)
npx supabase start

# Run development server
npm run dev

# Open browser
open http://localhost:4321
```

## Appendix B: Useful Commands

```bash
# Development
npm run dev              # Start dev server
npm run build            # Build for production
npm run preview          # Preview production build

# Testing
npm run test             # Run unit tests
npm run test:watch       # Watch mode
npm run test:coverage    # Coverage report

# Database
npx supabase migration new <name>  # Create migration
npx supabase db push              # Apply migrations
npx supabase gen types typescript # Generate types

# Docker
docker-compose up        # Start all services
docker-compose down      # Stop all services
docker-compose logs -f   # View logs
```

---

**Document Version:** 1.0  
**Last Updated:** 2025-01-20  
**Author:** Architecture Team  
**Status:** Draft for Review

