# Architektura projektu InflorAI

## Spis treści
1. [Przegląd architektury](#przegląd-architektury)
2. [Warstwy aplikacji](#warstwy-aplikacji)
3. [Struktura projektu](#struktura-projektu)
4. [Komponenty UI - Atomic Design](#komponenty-ui---atomic-design)
5. [Feature-based organizacja](#feature-based-organizacja)
6. [Logika biznesowa](#logika-biznesowa)
7. [Warstwa danych](#warstwa-danych)
8. [Flow użytkownika](#flow-użytkownika)
9. [Typy i DTOs](#typy-i-dtos)
10. [Kluczowe decyzje architektoniczne](#kluczowe-decyzje-architektoniczne)

---

## Przegląd architektury

InflorAI wykorzystuje **trzywarstwową architekturę** z separacją odpowiedzialności:

```
┌─────────────────────────────────────────────────────────────┐
│                    WARSTWA PREZENTACJI                       │
│  (Astro Pages + React Components + Atomic Design)           │
├─────────────────────────────────────────────────────────────┤
│              WARSTWA LOGIKI BIZNESOWEJ                       │
│     (Services + Helpers + Hooks + Constants)                │
├─────────────────────────────────────────────────────────────┤
│                   WARSTWA DANYCH                             │
│    (Supabase + API Endpoints + Types + Integrations)        │
└─────────────────────────────────────────────────────────────┘
```

### Principia architektoniczne:
- **Separation of Concerns** - każda warstwa ma jasno określoną odpowiedzialność
- **Feature-based Split** - kod organizowany wokół funkcjonalności biznesowych
- **Atomic Design** - komponenty UI budowane od najmniejszych elementów
- **Type Safety** - TypeScript na każdym poziomie
- **Testability** - struktura ułatwiająca testy jednostkowe i integracyjne
- **Scalability** - łatwe dodawanie nowych feature'ów

---

## Warstwy aplikacji

### 1. Warstwa prezentacji (Presentation Layer)

**Odpowiedzialność:**
- Renderowanie UI
- Obsługa interakcji użytkownika
- Wyświetlanie danych

**Technologie:**
- Astro 5 (strony, layouts, SSR)
- React 19 (komponenty interaktywne)
- Tailwind 4 (stylowanie)
- Shadcn/ui (biblioteka komponentów)

**Struktura:**
```
src/
├── pages/           # Astro pages (routing)
├── layouts/         # Astro layouts
├── components/      # React/Astro components (Atomic Design + Features)
```

### 2. Warstwa logiki biznesowej (Business Logic Layer)

**Odpowiedzialność:**
- Logika przeliczeń cenowych
- Parsowanie i ekstrakcja danych
- Walidacja
- Transformacja danych
- Komunikacja z zewnętrznymi API

**Technologie:**
- TypeScript 5
- Pure functions
- Service pattern

**Struktura:**
```
src/lib/
├── services/        # Główna logika biznesowa
├── helpers/         # Funkcje pomocnicze (pure functions)
├── hooks/           # React hooks (bridge między UI a services)
├── constants/       # Stałe i konfiguracja
└── integrations/    # Klienty zewnętrznych API
```

### 3. Warstwa danych (Data Layer)

**Odpowiedzialność:**
- Dostęp do bazy danych
- Definicje typów danych
- API endpoints
- Zarządzanie sesjami

**Technologie:**
- Supabase (PostgreSQL + Auth + SDK)
- Astro API routes

**Struktura:**
```
src/
├── db/              # Supabase client i typy
├── pages/api/       # API endpoints
├── types.ts         # Shared types, DTOs, entities
└── middleware/      # Astro middleware
```

---

## Struktura projektu

### Kompletna struktura katalogów:

```
src/
├── assets/                        # Statyczne assety wewnętrzne
│
├── components/                    # Komponenty React/Astro
│   ├── ui/                        # Shadcn/ui components (atoms)
│   │   ├── button.tsx
│   │   ├── input.tsx
│   │   ├── card.tsx
│   │   ├── table.tsx
│   │   ├── dialog.tsx
│   │   ├── tabs.tsx
│   │   └── ...
│   │
│   ├── atoms/                     # Custom atomic components
│   │   ├── CurrencyBadge.tsx     # Badge z symbolem waluty
│   │   ├── StatusIndicator.tsx   # Wskaźnik statusu (error/warning/success)
│   │   ├── ValidationMessage.tsx # Komunikat walidacji
│   │   ├── LoadingSpinner.tsx    # Spinner ładowania
│   │   └── ErrorBoundary.tsx     # Error boundary
│   │
│   ├── molecules/                 # Kombinacje atoms
│   │   ├── FormField.tsx         # Label + Input + Error
│   │   ├── CurrencyInput.tsx     # Input z wyborem waluty
│   │   ├── DateSelector.tsx      # Data picker z validacją
│   │   ├── TableCell.tsx         # Komórka tabeli z edycją
│   │   ├── ActionButton.tsx      # Button z ikoną i stanem
│   │   └── SearchBar.tsx         # Search input z ikoną
│   │
│   ├── organisms/                 # Złożone komponenty
│   │   ├── EditableTable.tsx     # Tabela z edycją in-place
│   │   ├── PricingConfigForm.tsx # Formularz konfiguracji
│   │   ├── TextInputArea.tsx     # Textarea z formatowaniem
│   │   ├── ValidationPopup.tsx   # Popup z listą błędów
│   │   ├── DataGrid.tsx          # Grid z sortowaniem/filtrowaniem
│   │   └── NavigationBar.tsx     # Top navigation
│   │
│   └── features/                  # Feature-specific components
│       ├── auth/
│       │   ├── LoginForm.tsx
│       │   ├── LogoutButton.tsx
│       │   └── AuthGuard.tsx
│       │
│       ├── input/
│       │   ├── RawTextInput.tsx
│       │   ├── DateSelector.tsx
│       │   └── InputPreview.tsx
│       │
│       ├── pricing/
│       │   ├── CurrencyRateEditor.tsx
│       │   ├── MarginEditor.tsx
│       │   ├── TransportCostEditor.tsx
│       │   ├── VATEditor.tsx
│       │   └── PricingPreview.tsx
│       │
│       ├── extraction/
│       │   ├── ExtractionResults.tsx
│       │   ├── ExtractionProgress.tsx
│       │   └── ExtractionStats.tsx
│       │
│       ├── editing/
│       │   ├── DataTable.tsx
│       │   ├── EditableTextField.tsx
│       │   ├── RowActions.tsx
│       │   ├── BulkActions.tsx
│       │   └── ValidationIndicators.tsx
│       │
│       ├── generation/
│       │   ├── PriceListPreview.tsx
│       │   ├── ExportActions.tsx
│       │   ├── GenerationWarning.tsx
│       │   └── CopyToClipboard.tsx
│       │
│       └── history/
│           ├── HistorySidebar.tsx
│           ├── HistoryPanel.tsx
│           ├── HistoryItem.tsx
│           ├── HistoryFilters.tsx
│           └── HistoryRestore.tsx
│
├── db/                            # Database layer
│   ├── supabase.client.ts        # Supabase client initialization
│   └── database.types.ts         # Generated types from Supabase
│
├── layouts/                       # Astro layouts
│   ├── BaseLayout.astro          # Base HTML structure
│   ├── AuthLayout.astro          # Layout dla auth pages
│   └── AppLayout.astro           # Layout dla głównej aplikacji
│
├── lib/                          # Business logic layer
│   ├── services/                 # Services (business logic)
│   │   ├── auth/
│   │   │   └── auth.service.ts
│   │   │
│   │   ├── extraction/
│   │   │   ├── ai-extraction.service.ts
│   │   │   ├── text-parser.service.ts
│   │   │   └── field-extractor.service.ts
│   │   │
│   │   ├── pricing/
│   │   │   ├── currency-converter.service.ts
│   │   │   ├── price-calculator.service.ts
│   │   │   ├── margin-calculator.service.ts
│   │   │   ├── transport-calculator.service.ts
│   │   │   └── vat-calculator.service.ts
│   │   │
│   │   ├── validation/
│   │   │   ├── data-validator.service.ts
│   │   │   ├── completeness-checker.service.ts
│   │   │   └── format-validator.service.ts
│   │   │
│   │   ├── generation/
│   │   │   ├── pricelist-generator.service.ts
│   │   │   └── text-formatter.service.ts
│   │   │
│   │   ├── history/
│   │   │   └── history.service.ts
│   │   │
│   │   └── metrics/
│   │       └── delta-logger.service.ts
│   │
│   ├── helpers/                  # Helper functions (pure)
│   │   ├── format.helpers.ts    # Formatowanie liczb, dat, walut
│   │   ├── parse.helpers.ts     # Parsowanie tekstów
│   │   ├── validation.helpers.ts # Walidacje
│   │   ├── currency.helpers.ts   # Operacje na walutach
│   │   └── date.helpers.ts       # Operacje na datach
│   │
│   ├── hooks/                    # React hooks
│   │   ├── useAuth.ts
│   │   ├── usePricing.ts
│   │   ├── useExtraction.ts
│   │   ├── useEditing.ts
│   │   ├── useHistory.ts
│   │   ├── useValidation.ts
│   │   └── usePriceListGeneration.ts
│   │
│   ├── constants/                # Constants & configuration
│   │   ├── pricing.constants.ts  # Domyślne kursy, marże, VAT
│   │   ├── validation.constants.ts # Reguły walidacji
│   │   ├── currencies.constants.ts # Definicje walut
│   │   └── app.constants.ts      # Ogólne stałe
│   │
│   └── integrations/             # External API clients
│       └── openrouter/
│           ├── openrouter.client.ts
│           ├── prompts.ts
│           └── types.ts
│
├── middleware/                   # Astro middleware
│   └── index.ts                  # Auth + Supabase injection
│
├── pages/                        # Astro pages (routing)
│   ├── index.astro              # Landing/redirect
│   ├── login.astro              # Strona logowania
│   │
│   ├── app/                     # Główna aplikacja
│   │   └── index.astro          # Multi-step workflow
│   │
│   └── api/                     # API endpoints
│       ├── auth/
│       │   ├── login.ts
│       │   └── logout.ts
│       │
│       ├── extraction/
│       │   └── parse.ts         # AI extraction endpoint
│       │
│       ├── pricelist/
│       │   ├── calculate.ts     # Przeliczanie cen
│       │   ├── generate.ts      # Generowanie cennika
│       │   └── export.ts        # Eksport pliku
│       │
│       └── history/
│           ├── list.ts          # Lista historii
│           ├── get.ts           # Pojedynczy wpis
│           ├── save.ts          # Zapis nowego
│           └── delete.ts        # Usunięcie wpisu
│
├── styles/                       # Global styles
│   └── globals.css              # Tailwind + custom CSS
│
├── types.ts                     # Shared types, DTOs, Entities
├── env.d.ts                     # TypeScript env definitions
└── middleware.ts                # Middleware entry point

public/                          # Statyczne pliki publiczne
├── favicon.ico
└── images/

supabase/                        # Supabase configuration
├── config.toml
└── migrations/                  # Database migrations
    └── ...

.ai/                             # AI/Documentation files
├── prd.md
├── stack.md
└── architecture.md (ten plik)
```

---

## Komponenty UI - Atomic Design

### Poziomy komponentów:

#### **Atoms** (src/components/ui + src/components/atoms)
Najmniejsze, niepodzielne komponenty:
- Button, Input, Label, Badge, Icon
- CurrencyBadge, StatusIndicator, LoadingSpinner

**Zasady:**
- Brak logiki biznesowej
- Tylko props i stylowanie
- Maksymalna reużywalność

**Przykład:**
```tsx
// components/atoms/CurrencyBadge.tsx
interface CurrencyBadgeProps {
  currency: 'EUR' | 'USD' | 'PLN';
  variant?: 'default' | 'outlined';
}

export function CurrencyBadge({ currency, variant = 'default' }: CurrencyBadgeProps) {
  return <Badge variant={variant}>{currency}</Badge>;
}
```

#### **Molecules** (src/components/molecules)
Kombinacje atoms tworzące funkcjonalne jednostki:
- FormField (Label + Input + Error)
- CurrencyInput (Input + CurrencySelect)
- DateSelector (DatePicker + Validation)

**Zasady:**
- Minimalna logika (głównie UI state)
- Kompozycja atoms
- Konkretny cel użycia

**Przykład:**
```tsx
// components/molecules/CurrencyInput.tsx
interface CurrencyInputProps {
  value: number;
  currency: Currency;
  onValueChange: (value: number) => void;
  onCurrencyChange: (currency: Currency) => void;
}

export function CurrencyInput({...props}: CurrencyInputProps) {
  return (
    <div className="flex gap-2">
      <Input type="number" value={props.value} onChange={...} />
      <Select value={props.currency} onValueChange={...}>
        <SelectItem value="EUR">EUR</SelectItem>
        <SelectItem value="USD">USD</SelectItem>
        <SelectItem value="PLN">PLN</SelectItem>
      </Select>
    </div>
  );
}
```

#### **Organisms** (src/components/organisms)
Złożone, samodzielne sekcje UI:
- EditableTable (tabela z edycją, sortowaniem, filtrowaniem)
- PricingConfigForm (kompletny formularz konfiguracji)
- ValidationPopup (popup z listą błędów i akcjami)

**Zasady:**
- Mogą zawierać local state
- Kompozycja molecules i atoms
- Samodzielne funkcjonalności

**Przykład:**
```tsx
// components/organisms/EditableTable.tsx
interface EditableTableProps {
  data: TableRow[];
  onRowChange: (index: number, row: TableRow) => void;
  onRowDelete: (index: number) => void;
  validationErrors: ValidationError[];
}

export function EditableTable({...props}: EditableTableProps) {
  // Logika tabeli, sortowanie, edycja
  return <Table>...</Table>;
}
```

#### **Features** (src/components/features)
Komponenty specyficzne dla domeny biznesowej:
- LoginForm, HistorySidebar, DataTable
- Połączenie UI z business logic przez hooks

**Zasady:**
- Używają hooks do łączenia z services
- Kompozycja organisms, molecules, atoms
- Domain-specific logic

**Przykład:**
```tsx
// components/features/pricing/CurrencyRateEditor.tsx
export function CurrencyRateEditor() {
  const { rates, updateRate } = usePricing();
  
  return (
    <Card>
      <CardHeader>
        <CardTitle>Kursy walut</CardTitle>
      </CardHeader>
      <CardContent>
        <CurrencyInput 
          currency="EUR" 
          value={rates.EUR}
          onValueChange={(v) => updateRate('EUR', v)}
        />
        {/* ... */}
      </CardContent>
    </Card>
  );
}
```

---

## Feature-based organizacja

### Podział na domeny funkcjonalne (features):

```
features/
├── auth/              # Autentykacja
├── input/             # Wejście danych
├── pricing/           # Konfiguracja cenowa
├── extraction/        # Ekstrakcja AI
├── editing/           # Edycja danych
├── generation/        # Generowanie cennika
└── history/           # Historia operacji
```

### Każdy feature zawiera:
- **Components** - w `components/features/{feature}/`
- **Services** - w `lib/services/{feature}/`
- **Hooks** - w `lib/hooks/use{Feature}.ts`
- **Types** - w `types.ts` (sekcja dla feature)

### Przykład: Feature "Pricing"

```
components/features/pricing/
├── CurrencyRateEditor.tsx
├── MarginEditor.tsx
├── TransportCostEditor.tsx
├── VATEditor.tsx
└── PricingPreview.tsx

lib/services/pricing/
├── currency-converter.service.ts
├── price-calculator.service.ts
├── margin-calculator.service.ts
├── transport-calculator.service.ts
└── vat-calculator.service.ts

lib/hooks/
└── usePricing.ts

types.ts (fragment):
// Pricing Feature Types
export interface PricingConfig { ... }
export interface CurrencyRate { ... }
export interface MarginConfig { ... }
```

---

## Logika biznesowa

### Services (src/lib/services)

Services zawierają główną logikę biznesową. Są to pure functions lub klasy z metodami.

**Zasady:**
- Brak zależności od UI
- Testowalne w izolacji
- Single Responsibility Principle
- Explicit dependencies (dependency injection)

#### Przykłady services:

##### 1. Price Calculator Service
```typescript
// lib/services/pricing/price-calculator.service.ts

import type { PricingConfig, ProductData, CalculatedPrice } from '@/types';
import { currencyConverter } from './currency-converter.service';
import { marginCalculator } from './margin-calculator.service';
import { transportCalculator } from './transport-calculator.service';
import { vatCalculator } from './vat-calculator.service';

export const priceCalculator = {
  calculateFinalPrice(
    product: ProductData,
    config: PricingConfig
  ): CalculatedPrice {
    // 1. Konwersja ceny bazowej na PLN
    const basePricePLN = currencyConverter.convert(
      product.basePrice,
      product.currency,
      config.rates
    );

    // 2. Obliczenie marży netto
    const marginNet = marginCalculator.calculate(
      product.length,
      config.margins
    );

    // 3. Obliczenie transportu netto
    const transportNet = transportCalculator.calculate(
      product.length,
      config.transportCosts
    );

    // 4. Suma netto
    const totalNet = basePricePLN + marginNet + transportNet;

    // 5. VAT
    const vatAmount = vatCalculator.calculate(totalNet, config.vatRate);

    // 6. Cena końcowa brutto
    const finalPrice = totalNet + vatAmount;

    return {
      basePricePLN,
      marginNet,
      transportNet,
      totalNet,
      vatAmount,
      finalPrice,
    };
  },
};
```

##### 2. AI Extraction Service
```typescript
// lib/services/extraction/ai-extraction.service.ts

import type { RawInputDTO, ExtractedDataDTO } from '@/types';
import { openRouterClient } from '@/lib/integrations/openrouter/openrouter.client';
import { EXTRACTION_PROMPT } from '@/lib/integrations/openrouter/prompts';

export const aiExtractionService = {
  async extractData(rawInput: RawInputDTO): Promise<ExtractedDataDTO> {
    const response = await openRouterClient.complete({
      prompt: EXTRACTION_PROMPT,
      userInput: rawInput.text,
      examples: FEW_SHOT_EXAMPLES,
    });

    // Parsowanie odpowiedzi AI
    const extracted = this.parseAIResponse(response);

    return extracted;
  },

  parseAIResponse(response: string): ExtractedDataDTO {
    // Logika parsowania JSON z odpowiedzi AI
    // Obsługa błędów i walidacja struktury
    // ...
  },
};
```

##### 3. Validation Service
```typescript
// lib/services/validation/data-validator.service.ts

import type { ExtractedRow, ValidationError } from '@/types';

export const dataValidator = {
  validateRow(row: ExtractedRow): ValidationError[] {
    const errors: ValidationError[] = [];

    if (!row.name || row.name.trim() === '') {
      errors.push({
        field: 'name',
        message: 'Brak nazwy produktu',
        severity: 'error',
      });
    }

    if (!row.quantity || row.quantity <= 0) {
      errors.push({
        field: 'quantity',
        message: 'Nieprawidłowa ilość',
        severity: 'error',
      });
    }

    if (!row.basePrice || row.basePrice <= 0) {
      errors.push({
        field: 'basePrice',
        message: 'Brak lub nieprawidłowa cena',
        severity: 'error',
      });
    }

    if (!row.length) {
      errors.push({
        field: 'length',
        message: 'Brak długości - użyto domyślnej marży',
        severity: 'warning',
      });
    }

    return errors;
  },

  validateAllRows(rows: ExtractedRow[]): Map<number, ValidationError[]> {
    const validationResults = new Map<number, ValidationError[]>();

    rows.forEach((row, index) => {
      const errors = this.validateRow(row);
      if (errors.length > 0) {
        validationResults.set(index, errors);
      }
    });

    return validationResults;
  },

  hasBlockingErrors(errors: ValidationError[]): boolean {
    return errors.some((e) => e.severity === 'error');
  },
};
```

### Helpers (src/lib/helpers)

Helpers to pure functions bez side effects. Używane przez services i components.

**Przykłady:**

```typescript
// lib/helpers/format.helpers.ts

export const formatHelpers = {
  formatCurrency(amount: number, currency: string = 'PLN'): string {
    return new Intl.NumberFormat('pl-PL', {
      style: 'currency',
      currency,
    }).format(amount);
  },

  formatDate(date: Date): string {
    return new Intl.DateTimeFormat('pl-PL').format(date);
  },

  parseDecimal(value: string): number {
    // Obsługa przecinka i kropki dziesiętnej
    const normalized = value.replace(',', '.');
    return parseFloat(normalized);
  },
};
```

```typescript
// lib/helpers/currency.helpers.ts

export const currencyHelpers = {
  normalizeCurrencySymbol(symbol: string): 'EUR' | 'USD' | 'PLN' | null {
    const map: Record<string, 'EUR' | 'USD' | 'PLN'> = {
      '€': 'EUR',
      'EUR': 'EUR',
      '$': 'USD',
      'USD': 'USD',
      'zł': 'PLN',
      'PLN': 'PLN',
    };

    return map[symbol] || null;
  },

  getCurrencySymbol(currency: 'EUR' | 'USD' | 'PLN'): string {
    const symbols = {
      EUR: '€',
      USD: '$',
      PLN: 'zł',
    };
    return symbols[currency];
  },
};
```

### Hooks (src/lib/hooks)

Hooks łączą UI z services. Zarządzają stanem i side effects.

**Przykład:**

```typescript
// lib/hooks/usePricing.ts

import { useState, useCallback } from 'react';
import type { PricingConfig, ProductData, CalculatedPrice } from '@/types';
import { priceCalculator } from '@/lib/services/pricing/price-calculator.service';
import { DEFAULT_PRICING_CONFIG } from '@/lib/constants/pricing.constants';

export function usePricing() {
  const [config, setConfig] = useState<PricingConfig>(DEFAULT_PRICING_CONFIG);

  const updateRate = useCallback((currency: 'EUR' | 'USD', rate: number) => {
    setConfig((prev) => ({
      ...prev,
      rates: {
        ...prev.rates,
        [currency]: rate,
      },
    }));
  }, []);

  const updateMargin = useCallback((lengthThreshold: number, margin: number) => {
    setConfig((prev) => ({
      ...prev,
      margins: {
        ...prev.margins,
        [lengthThreshold]: margin,
      },
    }));
  }, []);

  const calculatePrice = useCallback(
    (product: ProductData): CalculatedPrice => {
      return priceCalculator.calculateFinalPrice(product, config);
    },
    [config]
  );

  return {
    config,
    updateRate,
    updateMargin,
    calculatePrice,
  };
}
```

```typescript
// lib/hooks/useExtraction.ts

import { useState, useCallback } from 'react';
import type { RawInputDTO, ExtractedDataDTO } from '@/types';
import { aiExtractionService } from '@/lib/services/extraction/ai-extraction.service';

export function useExtraction() {
  const [isExtracting, setIsExtracting] = useState(false);
  const [extractedData, setExtractedData] = useState<ExtractedDataDTO | null>(null);
  const [error, setError] = useState<string | null>(null);

  const extractData = useCallback(async (rawInput: RawInputDTO) => {
    setIsExtracting(true);
    setError(null);

    try {
      const result = await aiExtractionService.extractData(rawInput);
      setExtractedData(result);
      return result;
    } catch (err) {
      const errorMessage = err instanceof Error ? err.message : 'Błąd ekstrakcji';
      setError(errorMessage);
      throw err;
    } finally {
      setIsExtracting(false);
    }
  }, []);

  const clearExtraction = useCallback(() => {
    setExtractedData(null);
    setError(null);
  }, []);

  return {
    isExtracting,
    extractedData,
    error,
    extractData,
    clearExtraction,
  };
}
```

---

## Warstwa danych

### Supabase Client (src/db/supabase.client.ts)

```typescript
import { createClient } from '@supabase/supabase-js';
import type { Database } from './database.types';

const supabaseUrl = import.meta.env.SUPABASE_URL;
const supabaseAnonKey = import.meta.env.SUPABASE_KEY;

export const supabaseClient = createClient<Database>(
  supabaseUrl,
  supabaseAnonKey
);
```

### Database Types (src/db/database.types.ts)

Typy generowane z Supabase CLI:

```bash
npx supabase gen types typescript --project-id YOUR_PROJECT_ID > src/db/database.types.ts
```

### API Endpoints (src/pages/api)

API endpoints obsługują komunikację frontend-backend.

**Przykład: Parse Endpoint**

```typescript
// src/pages/api/extraction/parse.ts

import type { APIRoute } from 'astro';
import { aiExtractionService } from '@/lib/services/extraction/ai-extraction.service';

export const POST: APIRoute = async ({ request, locals }) => {
  // Sprawdzenie autentykacji
  const { data: { user } } = await locals.supabase.auth.getUser();
  
  if (!user) {
    return new Response(JSON.stringify({ error: 'Unauthorized' }), {
      status: 401,
    });
  }

  try {
    const body = await request.json();
    const { rawText, defaultDate } = body;

    // Wywołanie service
    const extracted = await aiExtractionService.extractData({
      text: rawText,
      defaultDate,
    });

    return new Response(JSON.stringify(extracted), {
      status: 200,
      headers: { 'Content-Type': 'application/json' },
    });
  } catch (error) {
    console.error('Extraction error:', error);
    return new Response(
      JSON.stringify({ error: 'Extraction failed' }),
      { status: 500 }
    );
  }
};
```

**Przykład: History Save Endpoint**

```typescript
// src/pages/api/history/save.ts

import type { APIRoute } from 'astro';
import type { HistoryEntry } from '@/types';

export const POST: APIRoute = async ({ request, locals }) => {
  const { data: { user } } = await locals.supabase.auth.getUser();
  
  if (!user) {
    return new Response(JSON.stringify({ error: 'Unauthorized' }), {
      status: 401,
    });
  }

  try {
    const historyEntry: HistoryEntry = await request.json();

    const { data, error } = await locals.supabase
      .from('history')
      .insert({
        user_id: user.id,
        raw_input: historyEntry.rawInput,
        extracted_data: historyEntry.extractedData,
        edited_data: historyEntry.editedData,
        final_pricelist: historyEntry.finalPricelist,
        delta: historyEntry.delta,
        created_at: new Date().toISOString(),
      })
      .select()
      .single();

    if (error) throw error;

    return new Response(JSON.stringify(data), {
      status: 201,
      headers: { 'Content-Type': 'application/json' },
    });
  } catch (error) {
    console.error('History save error:', error);
    return new Response(
      JSON.stringify({ error: 'Failed to save history' }),
      { status: 500 }
    );
  }
};
```

### Middleware (src/middleware/index.ts)

```typescript
import { defineMiddleware } from 'astro:middleware';
import { supabaseClient } from '../db/supabase.client';

export const onRequest = defineMiddleware(async (context, next) => {
  // Dodanie Supabase client do context.locals
  context.locals.supabase = supabaseClient;

  // Sprawdzenie sesji dla chronionych ścieżek
  if (context.url.pathname.startsWith('/app')) {
    const { data: { user } } = await supabaseClient.auth.getUser();
    
    if (!user) {
      return context.redirect('/login');
    }
  }

  return next();
});
```

---

## Flow użytkownika

### User Journey - Multi-step Workflow

```
┌─────────────┐
│   Landing   │
│  /index     │
└──────┬──────┘
       │
       ↓
┌─────────────┐
│    Login    │
│   /login    │
└──────┬──────┘
       │
       ↓
┌─────────────────────────────────────────────────────┐
│              Główna aplikacja (/app)                 │
│                                                      │
│  ┌────────────────────────────────────────────┐   │
│  │ Step 1: Input + Pricing Configuration      │   │
│  │ - Wklejenie surowego tekstu                │   │
│  │ - Ustawienie daty domyślnej (jeśli brak)   │   │
│  │ - Edycja kursów walut                      │   │
│  │ - Edycja marż i transportu                 │   │
│  │ - Edycja VAT                               │   │
│  └─────────────────┬──────────────────────────┘   │
│                    ↓                                │
│  ┌────────────────────────────────────────────┐   │
│  │ Step 2: Extraction + Editing               │   │
│  │ - Automatyczna ekstrakcja przez AI         │   │
│  │ - Wyświetlenie tabeli z danymi            │   │
│  │ - Oznaczenie błędów walidacji              │   │
│  │ - Edycja w tabeli                          │   │
│  │ - Edycja w polu tekstowym                  │   │
│  └─────────────────┬──────────────────────────┘   │
│                    ↓                                │
│  ┌────────────────────────────────────────────┐   │
│  │ Step 3: Generation + Export                │   │
│  │ - Popup z ostrzeżeniami (jeśli błędy)     │   │
│  │ - Generowanie cennika                      │   │
│  │ - Podgląd cennika                          │   │
│  │ - Kopiowanie do schowka                    │   │
│  │ - Eksport do pliku                         │   │
│  │ - Zapis do historii                        │   │
│  └────────────────────────────────────────────┘   │
│                                                      │
│  ┌────────────────────────────────────────────┐   │
│  │ Historia (Sidebar - zawsze dostępny)       │   │
│  │ - Lista poprzednich operacji               │   │
│  │ - Filtrowanie i wyszukiwanie               │   │
│  │ - Podgląd szczegółów                       │   │
│  │ - Przywracanie danych                      │   │
│  └────────────────────────────────────────────┘   │
└─────────────────────────────────────────────────────┘
```

### Główne komponenty strony /app/index.astro

```typescript
// Pseudokod struktury głównej strony

<AppLayout>
  <!-- Sidebar z historią -->
  <HistorySidebar />

  <!-- Główny content -->
  <main>
    <!-- Tabs dla kroków -->
    <Tabs value={currentStep}>
      <!-- Step 1: Input + Config -->
      <TabsContent value="input">
        <div className="grid grid-cols-2 gap-4">
          <RawTextInput />
          <PricingConfigForm />
        </div>
      </TabsContent>

      <!-- Step 2: Extraction + Editing -->
      <TabsContent value="editing">
        <ExtractionProgress />
        <DataTable />
        <EditableTextField />
      </TabsContent>

      <!-- Step 3: Generation -->
      <TabsContent value="generation">
        <GenerationWarning />
        <PriceListPreview />
        <ExportActions />
      </TabsContent>
    </Tabs>
  </main>
</AppLayout>
```

---

## Typy i DTOs

### Struktura types.ts

```typescript
// src/types.ts

// ============================================================================
// ENTITIES (Database Models)
// ============================================================================

export interface User {
  id: string;
  email: string;
  created_at: string;
}

export interface HistoryRecord {
  id: string;
  user_id: string;
  raw_input: string;
  extracted_data: ExtractedDataDTO;
  edited_data: string;
  final_pricelist: string;
  delta: DeltaLog[];
  created_at: string;
}

// ============================================================================
// DTOs (Data Transfer Objects)
// ============================================================================

// Input Feature
export interface RawInputDTO {
  text: string;
  defaultDate?: string;
}

// Pricing Feature
export interface PricingConfigDTO {
  rates: CurrencyRates;
  margins: MarginConfig;
  transportCosts: TransportConfig;
  vatRate: number;
}

export interface CurrencyRates {
  EUR: number;
  USD: number;
}

export interface MarginConfig {
  below60cm: number;  // Marża netto dla długości < 60cm
  above60cm: number;  // Marża netto dla długości >= 60cm
}

export interface TransportConfig {
  below60cm: number;  // Transport netto dla długości < 60cm
  above60cm: number;  // Transport netto dla długości >= 60cm
}

// Extraction Feature
export interface ExtractedDataDTO {
  rows: ExtractedRow[];
  metadata: {
    dateExtracted?: string;
    headerDetected: boolean;
  };
}

export interface ExtractedRow {
  group?: string;
  name: string;
  length?: number;         // cm
  quantity?: number;       // szt
  basePrice?: number;      // cena jednostkowa w walucie oryginalnej
  currency?: 'EUR' | 'USD' | 'PLN';
  availabilityDate?: string;
  notes?: string;          // Kolumna UWAGI
}

// Calculation Feature
export interface CalculatedPrice {
  basePricePLN: number;    // Cena bazowa w PLN
  marginNet: number;       // Marża netto
  transportNet: number;    // Transport netto
  totalNet: number;        // Suma netto
  vatAmount: number;       // Kwota VAT
  finalPrice: number;      // Cena końcowa brutto
}

export interface PriceListRow extends ExtractedRow {
  calculated: CalculatedPrice;
}

// Validation Feature
export interface ValidationError {
  field: string;
  message: string;
  severity: 'error' | 'warning';
}

export interface ValidationResult {
  isValid: boolean;
  errors: Map<number, ValidationError[]>;  // index wiersza -> błędy
}

// Generation Feature
export interface GeneratedPriceListDTO {
  header: string[];
  rows: string[][];
  textFormat: string;      // Format tekstowy z tabulatorami
}

// History Feature
export interface HistoryEntryDTO {
  id: string;
  rawInput: string;
  extractedData: ExtractedDataDTO;
  editedData: string;
  finalPricelist: string;
  delta: DeltaLog[];
  createdAt: string;
}

// Metrics Feature
export interface DeltaLog {
  rowIndex: number;
  field: string;
  autoValue: any;
  finalValue: any;
  timestamp: string;
}

// ============================================================================
// VIEW MODELS (for UI)
// ============================================================================

export interface TableRowViewModel extends PriceListRow {
  index: number;
  hasErrors: boolean;
  errors: ValidationError[];
  isEdited: boolean;
}

export interface HistoryItemViewModel {
  id: string;
  preview: string;         // Pierwsze 100 znaków raw input
  itemCount: number;       // Liczba pozycji
  createdAt: string;
  formattedDate: string;   // Sformatowana data dla UI
}

// ============================================================================
// CONSTANTS & ENUMS
// ============================================================================

export type Currency = 'EUR' | 'USD' | 'PLN';

export enum ProcessStep {
  INPUT = 'input',
  EDITING = 'editing',
  GENERATION = 'generation',
}

export enum ValidationSeverity {
  ERROR = 'error',
  WARNING = 'warning',
}
```

---

## Kluczowe decyzje architektoniczne

### 1. Separation of Concerns

**Decyzja:** Ścisła separacja UI, logiki biznesowej i danych.

**Uzasadnienie:**
- Ułatwia testy (można testować services niezależnie)
- Zmniejsza coupling
- Umożliwia wymianę implementacji (np. zmiana AI providera)

**Implementacja:**
- UI components nie wywołują bezpośrednio API
- Services nie zależą od React/Astro
- Hooks jako bridge między UI a services

### 2. Atomic Design dla UI

**Decyzja:** Hierarchiczna organizacja komponentów UI.

**Uzasadnienie:**
- Maksymalna reużywalność
- Łatwe utrzymanie spójności designu
- Ułatwione tworzenie nowych features

**Implementacja:**
- Atoms: podstawowe komponenty (Button, Input)
- Molecules: kombinacje atoms
- Organisms: złożone sekcje
- Features: domain-specific komponenty

### 3. Feature-based Split

**Decyzja:** Kod organizowany wokół funkcjonalności biznesowych.

**Uzasadnienie:**
- Łatwiejsze znajdowanie kodu
- Lepsze skalowanie zespołu (developerzy mogą pracować nad różnymi features)
- Ułatwione dodawanie nowych funkcji

**Implementacja:**
- Każdy feature ma komponenty, services, hooks
- Jasne granice między features
- Możliwość wydzielenia feature do oddzielnego pakietu

### 4. Type Safety na wszystkich poziomach

**Decyzja:** TypeScript wszędzie + generowane typy z Supabase.

**Uzasadnienie:**
- Catch błędów w compile time
- Lepsze IDE support
- Self-documenting code

**Implementacja:**
- Typy generowane z Supabase dla database entities
- Explicit DTOs dla API communication
- View Models dla UI

### 5. Service Pattern dla Business Logic

**Decyzja:** Logika biznesowa w services jako pure functions lub obiekty.

**Uzasadnienie:**
- Testowalne w izolacji
- Reużywalne
- Niezależne od frameworka

**Implementacja:**
- Services eksportują obiekty z metodami
- Dependency injection przez parametry
- Brak side effects poza kontrolowanymi (API calls)

### 6. Hooks jako Bridge

**Decyzja:** React hooks łączą UI z services.

**Uzasadnienie:**
- Separation of concerns
- Reużywalne logic
- Łatwe do testowania (można mockować services)

**Implementacja:**
- Hooks zarządzają stanem
- Hooks wywołują services
- Components używają hooks, nie services bezpośrednio

### 7. Multi-step Workflow w jednej stronie

**Decyzja:** Główna aplikacja jako single page z tabs/steps.

**Uzasadnienie:**
- Lepsza UX (brak przeładowań)
- Stan aplikacji w pamięci
- Łatwe przejścia między krokami

**Implementacja:**
- Tabs component z krokami
- Shared state przez Context lub hooks
- Historia w sidebar zawsze widoczna

### 8. Edytowalne pole tekstowe jako fallback

**Decyzja:** Poza tabelą edycyjną, dodanie pola tekstowego z pełną zawartością.

**Uzasadnienie:**
- Maksymalna elastyczność dla użytkownika
- Szybkie korekty bez klikania w komórki
- Backup na wypadek problemów z tabelą

**Implementacja:**
- Textarea z zawartością TSV (tab-separated values)
- Synchronizacja dwukierunkowa z tabelą
- Walidacja przy każdej zmianie

### 9. Logowanie delt dla Quality Metrics

**Decyzja:** Zapisywanie różnic między auto-extraction a finalną wersją.

**Uzasadnienie:**
- Mierzenie skuteczności AI
- Input do ulepszeń
- Możliwość analizy najczęstszych błędów

**Implementacja:**
- Delta logger service
- Zapis do bazy przy generowaniu cennika
- Dashboard metryk (future feature)

### 10. Supabase jako Backend

**Decyzja:** Supabase dla auth, database, BaaS.

**Uzasadnienie:**
- Szybki start (mniej boilerplate)
- Built-in auth
- PostgreSQL (relational + JSON support)
- SDK dla TypeScript

**Implementacja:**
- Supabase client w middleware
- Row Level Security dla danych użytkownika
- Generated types dla type safety

---

## Podsumowanie

Architektura InflorAI została zaprojektowana z myślą o:

✅ **Modularności** - każdy feature jest niezależny
✅ **Testowalności** - services i helpers jako pure functions
✅ **Skalowalności** - łatwe dodawanie nowych funkcji
✅ **Maintainability** - jasna struktura i separacja concerns
✅ **Type Safety** - TypeScript na każdym poziomie
✅ **Developer Experience** - intuicyjna organizacja kodu

### Następne kroki implementacji:

1. **Setup Supabase** - inicjalizacja bazy danych i auth
2. **Migracje DB** - stworzenie tabel (users, history)
3. **Atomic Components** - implementacja atoms i molecules
4. **Core Services** - price calculator, AI extraction
5. **Feature Components** - główne features (input, editing, generation)
6. **Integration** - połączenie wszystkich części
7. **Testing** - testy jednostkowe i integracyjne
8. **Deployment** - CI/CD + hosting

---

**Dokument stworzony:** 2025-10-20
**Wersja:** 1.0
**Status:** Draft do review

