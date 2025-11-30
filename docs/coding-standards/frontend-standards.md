# ⚛️ Frontend Package Coding Standards

> **Package**: `@botfleet/frontend`
> **Framework**: Next.js 15 with App Router
> **Last Updated**: November 2025

## 📋 Table of Contents

1. [General Coding Standards](#general-coding-standards)
2. [Package Structure](#package-structure)
3. [Component Patterns](#component-patterns)
4. [Custom Hooks](#custom-hooks)
5. [State Management](#state-management)
6. [Data Fetching](#data-fetching)
7. [SSE Implementation](#sse-implementation)
8. [Styling Guidelines](#styling-guidelines)
9. [Internationalization](#internationalization)
10. [Performance](#performance)
11. [Error Handling](#error-handling)
12. [Testing](#testing)

---

## 1. General Coding Standards

### Code Style and Structure

```typescript
// ✅ DO: Use functional and declarative patterns
export function MonitorCard({ monitor, onStatusChange }: MonitorCardProps) {
  const isOnline = monitor.status === 'up';
  const hasRecentFailures = monitor.failureCount > 0;

  return (
    <Card className={cn('border', isOnline && 'border-green-200')}>
      {/* Component content */}
    </Card>
  );
}

// ✅ DO: Use descriptive variable names with auxiliary verbs
const isLoading = query.isLoading;
const hasError = query.error !== null;
const canEdit = user.permissions.includes('monitor:edit');

// ✅ DO: Structure files with clear organization
// 1. Imports (external first, internal second)
// 2. Types and interfaces
// 3. Main component
// 4. Subcomponents
// 5. Helper functions
// 6. Static content
```

### TypeScript Best Practices

```typescript
// ✅ DO: Use interfaces over types for object shapes
interface MonitorFormProps {
  monitor?: Monitor;
  onSubmit: (data: CreateMonitorDto) => void;
  isLoading?: boolean;
}

// ✅ DO: Use Zod for schema validation and type inference
import { z } from 'zod';

const MonitorSchema = z.object({
  name: z.string().min(1, 'Monitor name is required'),
  url: z.string().url('Please enter a valid URL'),
  interval: z.number().min(60, 'Minimum interval is 60 seconds'),
});

type MonitorFormData = z.infer<typeof MonitorSchema>;

// ✅ DO: Avoid enums; use literal types
type MonitorStatus = 'up' | 'down' | 'paused' | 'maintenance';

// ❌ DON'T: Use enums
enum MonitorStatus { // ❌
  UP = 'up',
  DOWN = 'down'
}
```

### Syntax and Formatting Rules

```typescript
// ✅ DO: Use function keyword for pure functions
function calculateUptime(monitor: Monitor): number {
  if (!monitor.createdAt) return 0;

  const totalTime = Date.now() - new Date(monitor.createdAt).getTime();
  const downTime = monitor.incidents.reduce((acc, incident) =>
    acc + incident.duration, 0
  );

  return ((totalTime - downTime) / totalTime) * 100;
}

// ✅ DO: Use early returns for error conditions
function validateMonitorUrl(url: string): boolean {
  if (!url) return false;
  if (!url.startsWith('http')) return false;
  if (url.length > 2048) return false;

  return isValidUrl(url);
}

// ✅ DO: Write declarative JSX with clear structure
return (
  <Card>
    <CardHeader>
      <CardTitle>{monitor.name}</CardTitle>
      <Badge variant={monitor.status === 'up' ? 'success' : 'destructive'}>
        {monitor.status}
      </Badge>
    </CardHeader>
    <CardContent>
      <MonitorMetrics monitor={monitor} />
      <MonitorActions
        monitor={monitor}
        onEdit={handleEdit}
        onDelete={handleDelete}
      />
    </CardContent>
  </Card>
);

// ✅ DO: Avoid unnecessary curly braces in conditionals
{monitor.status === 'up' && <SuccessIcon />}
{hasError && <ErrorMessage error={error} />}
```

### Import and Export Conventions

```typescript
// ✅ DO: Favor named exports for components and functions
export function MonitorCard(props: MonitorCardProps) { }
export function useMonitors() { }
export const MONITOR_INTERVALS = [60, 300, 600] as const;

// ✅ DO: Structure imports properly
// External libraries first
import { useState, useEffect } from 'react';
import { useQuery } from '@tanstack/react-query';
import { z } from 'zod';

// Internal imports second
import { Button } from '@/components/ui/button';
import { useApiClient } from '@/hooks/use-api-client';
import { Monitor } from '@botfleet/shared';

// ✅ DO: Use lowercase with dashes for directory names
components/
├── auth-wizard/
├── monitor-dashboard/
├── incident-alerts/
└── team-settings/
```

### General Rules

#### ✅ DO:
- Write concise, technical TypeScript code with accurate examples
- Use functional and declarative programming patterns
- Prefer iteration and modularization over code duplication
- Use descriptive variable names with auxiliary verbs (`isLoading`, `hasError`, `canEdit`)
- Favor named exports for components and functions
- Handle errors and edge cases at the beginning of functions
- Use early returns to avoid deep nesting
- Implement proper TypeScript interfaces for all props

#### ❌ DON'T:
- Use classes for React components
- Use enums (prefer literal types or const assertions)
- Use `React.FC` type annotation (deprecated pattern)
- Create deep nesting in conditional logic
- Use `any` types
- Mix default and named exports inconsistently

---

## 2. Package Structure

```
packages/frontend/
├── src/
│   ├── app/                 # Next.js App Router
│   │   ├── (auth)/         # Auth group routes
│   │   ├── (dashboard)/    # Dashboard group routes
│   │   ├── layout.tsx      # Root layout
│   │   └── page.tsx        # Home page
│   ├── components/         # Reusable components
│   │   ├── ui/            # Base UI components
│   │   ├── layout/        # Layout components
│   │   ├── common/        # Shared components
│   │   └── [feature]/     # Feature-specific
│   ├── hooks/             # Custom React hooks
│   ├── lib/              # Utilities and helpers
│   ├── providers/        # React context providers
│   ├── stores/          # Zustand stores
│   └── types/          # TypeScript types
├── public/            # Static assets
└── tests/            # Test files
```

### File Naming Convention
```typescript
// Components: PascalCase.tsx
MonitorCard.tsx
UserAuthForm.tsx

// Hooks: use-[name].ts
use-monitors.ts
use-api.ts

// Utils: kebab-case.ts
date-formatter.ts
api-client.ts

// Pages: lowercase/page.tsx (Next.js convention)
app/monitors/page.tsx
app/teams/[teamId]/page.tsx
```

---

## 2. Component Patterns

### Component Structure
```typescript
/**
 * MonitorCard - Display monitor status and details
 *
 * PATTERN: Functional components with TypeScript
 * - Use function declaration (not arrow function) for components
 * - Export as default for pages, named export for components
 * - Props interface defined above component
 */
'use client'; // Only if using client features

import { Card, CardContent, CardHeader } from '@/components/ui/card';
import { Badge } from '@/components/ui/badge';
import { MonitorDto } from '@botfleet/shared';

interface MonitorCardProps {
  monitor: MonitorDto;
  onEdit?: (monitor: MonitorDto) => void;
  onDelete?: (id: number) => void;
  className?: string;
}

export function MonitorCard({
  monitor,
  onEdit,
  onDelete,
  className,
}: MonitorCardProps) {
  return (
    <Card className={className}>
      <CardHeader>
        <h3 className="text-lg font-semibold">{monitor.name}</h3>
        <Badge variant={monitor.status === 'up' ? 'success' : 'destructive'}>
          {monitor.status}
        </Badge>
      </CardHeader>
      <CardContent>
        <p className="text-sm text-muted-foreground">{monitor.url}</p>
        {/* Component content */}
      </CardContent>
    </Card>
  );
}
```

### Component Rules

#### ✅ DO:
```typescript
// Use 'use client' directive when needed
'use client';

// Destructure props with TypeScript
export function Button({
  children,
  variant = 'default',
  onClick
}: ButtonProps) {
  return <button onClick={onClick}>{children}</button>;
}

// Use proper event handlers with types
const handleClick = (e: React.MouseEvent<HTMLButtonElement>) => {
  e.preventDefault();
  // Handle click
};

// Use composition over configuration
<Card>
  <CardHeader>Title</CardHeader>
  <CardContent>Content</CardContent>
</Card>
```

#### ❌ DON'T:
```typescript
// Don't use React.FC (deprecated pattern)
const Component: React.FC<Props> = ({ children }) => { // ❌
  return <div>{children}</div>;
};

// Don't use inline functions in render
<button onClick={() => doSomething(item.id)}> // ❌
  Click
</button>

// Don't use any types
const handleClick = (e: any) => { // ❌
  // Handle click
};
```

### Server vs Client Components
```typescript
// Server Component (default in app directory)
// app/monitors/page.tsx
import { MonitorList } from '@/components/monitors/monitor-list';

export default async function MonitorsPage() {
  // Can fetch data directly
  const monitors = await fetchMonitors();

  return <MonitorList monitors={monitors} />;
}

// Client Component (interactive)
// components/monitors/monitor-list.tsx
'use client';

import { useState } from 'react';

export function MonitorList({ monitors }: { monitors: Monitor[] }) {
  const [selected, setSelected] = useState<number | null>(null);

  return (
    // Interactive component
  );
}
```

---

## 3. Custom Hooks

### Hook Template
```typescript
/**
 * useMonitors - Hook for fetching and managing monitors
 *
 * PATTERN: Custom hooks encapsulate logic
 * - Start with 'use' prefix
 * - Return object for multiple values
 * - Handle loading and error states
 */
import { useQuery, useMutation, useQueryClient } from '@tanstack/react-query';
import { useApiClient } from './use-api-client';

export function useMonitors(teamId?: number) {
  const api = useApiClient();

  const query = useQuery({
    queryKey: ['monitors', teamId],
    queryFn: () => api.get(`/teams/${teamId}/monitors`),
    enabled: !!teamId,
    staleTime: 30_000, // 30 seconds
  });

  return {
    monitors: query.data?.data || [],
    isLoading: query.isLoading,
    error: query.error,
    refetch: query.refetch,
  };
}

export function useCreateMonitor() {
  const api = useApiClient();
  const queryClient = useQueryClient();

  return useMutation({
    mutationFn: (data: CreateMonitorDto) =>
      api.post('/monitors', data),
    onSuccess: () => {
      queryClient.invalidateQueries({ queryKey: ['monitors'] });
    },
  });
}
```

### Hook Rules

#### ✅ DO:
- Prefix with `use`
- Return consistent shape
- Handle all states (loading, error, success)
- Use React Query for server state
- Abstract complex logic

#### ❌ DON'T:
- Call hooks conditionally
- Use hooks outside components
- Mix concerns in one hook

---

## 4. State Management

### React Query for Server State
```typescript
// lib/query-client.ts
import { QueryClient } from '@tanstack/react-query';

export const queryClient = new QueryClient({
  defaultOptions: {
    queries: {
      staleTime: 60_000, // 1 minute
      gcTime: 300_000, // 5 minutes (formerly cacheTime)
      retry: 1,
      refetchOnWindowFocus: false,
    },
  },
});
```

### Zustand for Client State
```typescript
// stores/auth-store.ts
import { create } from 'zustand';
import { persist } from 'zustand/middleware';

interface AuthState {
  isAuthenticated: boolean;
  user: User | null;
  login: (user: User) => void;
  logout: () => void;
}

export const useAuthStore = create<AuthState>()(
  persist(
    (set) => ({
      isAuthenticated: false,
      user: null,
      login: (user) => set({ isAuthenticated: true, user }),
      logout: () => set({ isAuthenticated: false, user: null }),
    }),
    {
      name: 'auth-storage',
      partialize: (state) => ({ isAuthenticated: state.isAuthenticated }),
    }
  )
);
```

### Context for Component Trees
```typescript
// providers/theme-provider.tsx
'use client';

import { createContext, useContext, useState } from 'react';

interface ThemeContextValue {
  theme: 'light' | 'dark';
  toggleTheme: () => void;
}

const ThemeContext = createContext<ThemeContextValue | undefined>(undefined);

export function ThemeProvider({ children }: { children: React.ReactNode }) {
  const [theme, setTheme] = useState<'light' | 'dark'>('light');

  const toggleTheme = () => {
    setTheme((prev) => (prev === 'light' ? 'dark' : 'light'));
  };

  return (
    <ThemeContext.Provider value={{ theme, toggleTheme }}>
      {children}
    </ThemeContext.Provider>
  );
}

export function useTheme() {
  const context = useContext(ThemeContext);
  if (!context) {
    throw new Error('useTheme must be used within ThemeProvider');
  }
  return context;
}
```

---

## 5. Data Fetching

### Server Components (Recommended)
```typescript
// app/teams/[teamId]/monitors/page.tsx
import { headers } from 'next/headers';

async function getMonitors(teamId: string) {
  const response = await fetch(
    `${process.env.API_URL}/teams/${teamId}/monitors`,
    {
      headers: headers(), // Forward cookies for auth
      next: { revalidate: 60 }, // Cache for 60 seconds
    }
  );

  if (!response.ok) {
    throw new Error('Failed to fetch monitors');
  }

  return response.json();
}

export default async function MonitorsPage({
  params,
}: {
  params: { teamId: string };
}) {
  const monitors = await getMonitors(params.teamId);

  return <MonitorList monitors={monitors} />;
}
```

### Client Components with React Query
```typescript
// hooks/use-api.ts
import { useQuery, useMutation } from '@tanstack/react-query';

export function useApiQuery<T>(
  path: string | string[],
  options?: UseQueryOptions
) {
  const url = Array.isArray(path)
    ? `/api/v2/${path.filter(Boolean).join('/')}`
    : `/api/v2/${path}`;

  return useQuery<T>({
    queryKey: Array.isArray(path) ? path : [path],
    queryFn: async () => {
      const response = await fetch(url);
      if (!response.ok) {
        throw new Error(`API Error: ${response.status}`);
      }
      return response.json();
    },
    ...options,
  });
}
```

---

## 6. SSE Implementation

### SSE Hook
```typescript
// hooks/use-sse.ts
import { useEffect, useRef, useCallback } from 'react';
import { useQueryClient } from '@tanstack/react-query';

export function useSSE(teamId?: number) {
  const queryClient = useQueryClient();
  const eventSourceRef = useRef<EventSource | null>(null);

  const connect = useCallback(() => {
    if (!teamId) return;

    const eventSource = new EventSource(
      `/api/v2/teams/${teamId}/events`
    );

    eventSource.onmessage = (event) => {
      const data = JSON.parse(event.data);

      switch (data.type) {
        case 'monitor:updated':
          queryClient.invalidateQueries({
            queryKey: ['monitors', teamId],
          });
          break;

        case 'incident:created':
          queryClient.invalidateQueries({
            queryKey: ['incidents', teamId],
          });
          break;
      }
    };

    eventSource.onerror = () => {
      console.error('SSE connection lost, reconnecting...');
      // Browser automatically reconnects
    };

    eventSourceRef.current = eventSource;
  }, [teamId, queryClient]);

  useEffect(() => {
    connect();

    return () => {
      eventSourceRef.current?.close();
    };
  }, [connect]);

  return {
    isConnected: eventSourceRef.current?.readyState === EventSource.OPEN,
  };
}
```

### Using SSE in Components
```typescript
// app/(dashboard)/layout.tsx
'use client';

import { useSSE } from '@/hooks/use-sse';
import { useUserContext } from '@/hooks/use-user-context';

export default function DashboardLayout({
  children,
}: {
  children: React.ReactNode;
}) {
  const { currentTeam } = useUserContext();
  const { isConnected } = useSSE(currentTeam?.id);

  return (
    <div>
      {isConnected && (
        <div className="fixed top-4 right-4">
          <Badge variant="success">Live</Badge>
        </div>
      )}
      {children}
    </div>
  );
}
```

---

## 7. Styling Guidelines

### TailwindCSS Classes
```typescript
// ✅ GOOD: Use Tailwind utilities
<div className="flex items-center justify-between p-4 bg-white rounded-lg shadow-sm">
  <h2 className="text-lg font-semibold text-gray-900">Title</h2>
  <Button variant="primary" size="sm">Action</Button>
</div>

// ✅ GOOD: Use cn() for conditional classes
import { cn } from '@/lib/utils';

<div
  className={cn(
    'rounded-lg border p-4',
    isActive && 'border-blue-500 bg-blue-50',
    isDisabled && 'opacity-50 cursor-not-allowed'
  )}
>
```

### Component Variants
```typescript
// components/ui/button.tsx
import { cva, type VariantProps } from 'class-variance-authority';

const buttonVariants = cva(
  'inline-flex items-center justify-center rounded-md text-sm font-medium transition-colors',
  {
    variants: {
      variant: {
        default: 'bg-primary text-primary-foreground hover:bg-primary/90',
        destructive: 'bg-destructive text-destructive-foreground',
        outline: 'border border-input bg-background',
        ghost: 'hover:bg-accent hover:text-accent-foreground',
      },
      size: {
        default: 'h-10 px-4 py-2',
        sm: 'h-9 rounded-md px-3',
        lg: 'h-11 rounded-md px-8',
      },
    },
    defaultVariants: {
      variant: 'default',
      size: 'default',
    },
  }
);

export interface ButtonProps
  extends React.ButtonHTMLAttributes<HTMLButtonElement>,
    VariantProps<typeof buttonVariants> {}

export function Button({ className, variant, size, ...props }: ButtonProps) {
  return (
    <button
      className={cn(buttonVariants({ variant, size, className }))}
      {...props}
    />
  );
}
```

### Styling Rules

#### ✅ DO:
- Use Tailwind utilities
- Use CSS variables for theming
- Use cn() helper for conditional classes
- Keep styles co-located with components

#### ❌ DON'T:
- Use inline styles
- Create global CSS files
- Use !important
- Mix CSS-in-JS with Tailwind

---

## 8. Internationalization

### i18n Setup
```typescript
// i18n/config.ts
export const locales = ['en', 'es', 'fr'] as const;
export type Locale = (typeof locales)[number];

export const defaultLocale: Locale = 'en';

export const messages = {
  en: () => import('@/messages/en.json').then((m) => m.default),
  es: () => import('@/messages/es.json').then((m) => m.default),
  fr: () => import('@/messages/fr.json').then((m) => m.default),
};
```

### Using Translations
```typescript
// components/monitor-card.tsx
'use client';

import { useTranslations } from '@/hooks/use-translations';

export function MonitorCard({ monitor }: { monitor: Monitor }) {
  const t = useTranslations();

  return (
    <Card>
      <CardHeader>
        <h3>{monitor.name}</h3>
        <Badge>
          {t(`monitor.status.${monitor.status}`)}
        </Badge>
      </CardHeader>
      <CardContent>
        <p>{t('monitor.lastChecked', { time: monitor.lastCheck })}</p>
      </CardContent>
    </Card>
  );
}
```

### Message Files
```json
// messages/en.json
{
  "monitor": {
    "status": {
      "up": "Online",
      "down": "Offline",
      "paused": "Paused"
    },
    "lastChecked": "Last checked: {time}",
    "actions": {
      "edit": "Edit Monitor",
      "delete": "Delete Monitor",
      "pause": "Pause Monitoring"
    }
  }
}
```

---

## 9. Performance

### Code Splitting
```typescript
// Lazy load heavy components
import dynamic from 'next/dynamic';

const MonitorChart = dynamic(
  () => import('@/components/charts/monitor-chart'),
  {
    loading: () => <Skeleton className="h-64" />,
    ssr: false, // Disable SSR for client-only components
  }
);
```

### Image Optimization
```typescript
// Use Next.js Image component
import Image from 'next/image';

export function Logo() {
  return (
    <Image
      src="/logo.png"
      alt="Botfleet"
      width={120}
      height={40}
      priority // Load immediately for LCP
    />
  );
}
```

### Memoization and useEffect Minimization

```typescript
// ✅ DO: Minimize useEffect usage, prefer derived state
import { useMemo } from 'react';

export function MonitorList({ monitors }: { monitors: Monitor[] }) {
  // Prefer derived state over useEffect + useState
  const groupedMonitors = useMemo(
    () => groupMonitorsByStatus(monitors),
    [monitors]
  );

  const onlineCount = useMemo(
    () => monitors.filter(m => m.status === 'up').length,
    [monitors]
  );

  // ✅ DO: Use early returns for conditional rendering
  if (monitors.length === 0) {
    return <EmptyState />;
  }

  return (
    <div>
      <p>{onlineCount} of {monitors.length} monitors online</p>
      {/* Render grouped monitors */}
    </div>
  );
}

// ✅ DO: Memoize components that re-render frequently
import { memo } from 'react';

export const MonitorRow = memo(function MonitorRow({
  monitor,
}: {
  monitor: Monitor;
}) {
  return <tr>{/* Row content */}</tr>;
});

// ❌ DON'T: Use useEffect for derived state
function BadExample({ monitors }: { monitors: Monitor[] }) {
  const [onlineCount, setOnlineCount] = useState(0);

  useEffect(() => {
    setOnlineCount(monitors.filter(m => m.status === 'up').length);
  }, [monitors]); // ❌ This should be derived state

  return <div>{onlineCount}</div>;
}
```

### Avoiding Inline Objects and Functions

```typescript
// ❌ DON'T: Create inline objects/arrays in render
function BadComponent({ items }: { items: Item[] }) {
  return (
    <div>
      {items.map(item => (
        <button
          key={item.id}
          style={{ padding: '8px', margin: '4px' }} // ❌ Inline object
          onClick={() => handleClick(item)} // ❌ Inline function
        >
          {item.name}
        </button>
      ))}
    </div>
  );
}

// ✅ DO: Define objects and handlers outside render or use useMemo/useCallback
import { useCallback, useMemo } from 'react';

const BUTTON_STYLE = { padding: '8px', margin: '4px' }; // ✅ Static object

function GoodComponent({ items }: { items: Item[] }) {
  const handleClick = useCallback((item: Item) => {
    // Handle click logic
  }, []);

  const buttonStyle = useMemo(() => ({
    padding: '8px',
    margin: '4px',
  }), []); // ✅ Memoized if dynamic

  return (
    <div>
      {items.map(item => (
        <button
          key={item.id}
          style={BUTTON_STYLE} // ✅ Static reference
          onClick={() => handleClick(item)} // ✅ Stable callback
        >
          {item.name}
        </button>
      ))}
    </div>
  );
}
```

### Performance Rules

#### ✅ DO:
- Use React.memo for expensive components that re-render frequently
- Use useMemo/useCallback appropriately (not everywhere)
- Minimize useEffect usage; prefer derived state and memoization
- Use early returns for conditional rendering to avoid unnecessary computation
- Lazy load heavy components with `dynamic()` imports
- Optimize images with Next.js Image component
- Use virtual scrolling for long lists (100+ items)
- Define static objects and constants outside components
- Prefer derived state over useEffect + useState patterns
- Use code splitting at route level for better initial load

#### ❌ DON'T:
- Over-optimize prematurely (measure first)
- Memoize everything (adds overhead)
- Create inline objects/arrays/functions in render
- Use useEffect for calculations that can be derived
- Import heavy libraries in components (lazy load instead)
- Forget to handle loading states for dynamic imports

---

## 11. Error Handling

### Toast Notifications & API Error Handling

#### Toast System Architecture
Botfleet uses a custom toast notification system for user feedback, built on top of shadcn/ui components.

**Core Components:**
- **Toast Hook**: `useToast()` - Provides toast functions and state management
- **Toast Components**: `@/components/ui/toast` - UI components for display
- **Error Handler**: `@/lib/error-handler` - Centralized error parsing and toast integration

**Available Toast Functions:**
```tsx
import { useToast } from '@/hooks/use-toast';

const {
  toast,        // Base toast function
  toastSuccess, // Green success toasts
  toastError,   // Red error toasts
  toastWarning, // Yellow warning toasts
  toastInfo     // Blue info toasts
} = useToast();
```

#### API Error Structure
All API errors follow this structure from the backend:
```json
{
  "code": "INTERNAL_ERROR",
  "message": "An unexpected error occurred",
  "correlationId": "638f711c-0390-4e9b-8dfb-f0092ebcb475"
}
```

#### Error Handler Utility Usage

**✅ Recommended Approach - React Query Hooks:**
```tsx
import { useToast } from '@/hooks/use-toast';
import { createErrorHandlers } from '@/lib/error-handler';

export function useIncidents() {
  const { toastError, toastWarning } = useToast();
  const { handleQueryError, handleMutationError } = createErrorHandlers(toastError, toastWarning);

  return useApiQuery<IncidentResponse>(queryKey, {
    enabled: !!teamId,
    staleTime: 30_000,
    onError: (error) => handleQueryError(error, 'incidents'),
  });
}

export function useCreateIncident() {
  const { toastError, toastWarning } = useToast();
  const { handleMutationError } = createErrorHandlers(toastError, toastWarning);

  return useApiMutation('/incidents', 'POST', {
    onError: (error) => handleMutationError(error, 'create incident'),
  });
}
```

**✅ Direct Toast Usage - Components:**
```tsx
import { useToast } from '@/hooks/use-toast';

export function IncidentPage() {
  const { toastError, toastSuccess } = useToast();

  const handleAcknowledge = async () => {
    try {
      await acknowledgeIncident(incidentId);
      toastSuccess({
        title: 'Success',
        description: 'Incident acknowledged successfully',
      });
    } catch (error) {
      toastError({
        title: 'Error',
        description: 'Failed to acknowledge incident',
      });
    }
  };

  return <div>...</div>;
}
```

#### Error Classification

**Server Errors (5xx)** - Show as `toastError`:
- System errors, database issues, API failures
- Include correlation ID for debugging
- Red toast with "System Error" title

**Client Errors (4xx)** - Show as `toastWarning`:
- Validation errors, missing data, unauthorized access
- Yellow toast with "Invalid Request" title
- User can potentially fix these

#### Toast Variants & Usage

```tsx
// Success notifications (green)
toastSuccess({
  title: 'Success',
  description: 'Monitor created successfully',
});

// Error notifications (red)
toastError({
  title: 'System Error',
  description: 'Failed to load incidents: Database connection failed',
});

// Warning notifications (yellow)
toastWarning({
  title: 'Invalid Request',
  description: 'Monitor name is required',
});

// Info notifications (blue)
toastInfo({
  title: 'Information',
  description: 'Data sync will occur in 5 minutes',
});
```

#### Toast Best Practices

**✅ DO:**
- Use `createErrorHandlers()` for React Query hooks
- Provide contextual error messages (`'incidents'`, `'create monitor'`)
- Use appropriate toast variants based on error type
- Include correlation IDs for server errors
- Keep toast descriptions concise and actionable

**❌ DON'T:**
- Use generic error messages like "Something went wrong"
- Show raw API error responses to users
- Create multiple toasts for the same error
- Use toast for debugging information (use console.log)
- Forget to handle loading states alongside errors

### Error Boundaries and Component-Level Error Handling

```typescript
// components/error-boundary.tsx
'use client';

import { Component, ErrorInfo, ReactNode } from 'react';
import { Button } from '@/components/ui/button';
import { AlertTriangle } from 'lucide-react';

interface Props {
  children: ReactNode;
  fallback?: ReactNode;
}

interface State {
  hasError: boolean;
  error?: Error;
}

export class ErrorBoundary extends Component<Props, State> {
  public state: State = {
    hasError: false
  };

  public static getDerivedStateFromError(error: Error): State {
    return { hasError: true, error };
  }

  public componentDidCatch(error: Error, errorInfo: ErrorInfo) {
    console.error('ErrorBoundary caught an error:', error, errorInfo);

    // Log to external service
    // logErrorToService(error, errorInfo);
  }

  public render() {
    if (this.state.hasError) {
      return this.props.fallback || (
        <div className="flex flex-col items-center justify-center p-8 text-center">
          <AlertTriangle className="h-12 w-12 text-red-500 mb-4" />
          <h2 className="text-xl font-semibold mb-2">Something went wrong</h2>
          <p className="text-muted-foreground mb-4">
            We're sorry, but something unexpected happened.
          </p>
          <Button onClick={() => this.setState({ hasError: false })}>
            Try again
          </Button>
        </div>
      );
    }

    return this.props.children;
  }
}
```

### Hook-Level Error Handling

```typescript
// hooks/use-api-error.ts
import { useCallback } from 'react';
import { toast } from '@/hooks/use-toast';

interface ApiError {
  message: string;
  status?: number;
  code?: string;
}

export function useApiError() {
  const handleError = useCallback((error: unknown, context?: string) => {
    // Handle early return for no error
    if (!error) return;

    // Guard clauses for different error types
    if (error instanceof Response) {
      handleResponseError(error, context);
      return;
    }

    if (error instanceof Error) {
      handleGenericError(error, context);
      return;
    }

    // Fallback for unknown error types
    handleUnknownError(error, context);
  }, []);

  const handleResponseError = useCallback(async (response: Response, context?: string) => {
    try {
      const errorData = await response.json();
      const message = errorData.message || `HTTP ${response.status}`;

      toast({
        variant: 'destructive',
        title: context ? `${context} Error` : 'Request Failed',
        description: message,
      });
    } catch {
      toast({
        variant: 'destructive',
        title: 'Network Error',
        description: 'Unable to connect to the server',
      });
    }
  }, []);

  const handleGenericError = useCallback((error: Error, context?: string) => {
    console.error(`Error in ${context}:`, error);

    toast({
      variant: 'destructive',
      title: context ? `${context} Error` : 'Error',
      description: error.message || 'An unexpected error occurred',
    });
  }, []);

  const handleUnknownError = useCallback((error: unknown, context?: string) => {
    console.error(`Unknown error in ${context}:`, error);

    toast({
      variant: 'destructive',
      title: 'Unexpected Error',
      description: 'Something went wrong. Please try again.',
    });
  }, []);

  return { handleError };
}
```

### Query Error Handling with React Query

```typescript
// hooks/use-monitors.ts
import { useQuery } from '@tanstack/react-query';
import { useApiError } from './use-api-error';

export function useMonitors(teamId?: number) {
  const { handleError } = useApiError();

  return useQuery({
    queryKey: ['monitors', teamId],
    queryFn: async () => {
      // Early return for missing teamId
      if (!teamId) {
        throw new Error('Team ID is required');
      }

      const response = await fetch(`/api/v2/teams/${teamId}/monitors`);

      // Handle HTTP errors early
      if (!response.ok) {
        throw response; // Will be handled by useApiError
      }

      return response.json();
    },
    enabled: !!teamId,
    onError: (error) => {
      handleError(error, 'Monitors');
    },
    retry: (failureCount, error) => {
      // Don't retry on authentication errors
      if (error instanceof Response && error.status === 401) {
        return false;
      }

      // Retry up to 2 times for other errors
      return failureCount < 2;
    },
  });
}
```

### Form Validation with Zod

```typescript
// components/forms/monitor-form.tsx
'use client';

import { zodResolver } from '@hookform/resolvers/zod';
import { useForm } from 'react-hook-form';
import { z } from 'zod';

const MonitorSchema = z.object({
  name: z.string()
    .min(1, 'Monitor name is required')
    .max(100, 'Monitor name must be less than 100 characters'),
  url: z.string()
    .url('Please enter a valid URL')
    .refine((url) => url.startsWith('http'), {
      message: 'URL must start with http or https'
    }),
  interval: z.number()
    .min(60, 'Minimum interval is 60 seconds')
    .max(86400, 'Maximum interval is 24 hours'),
});

type MonitorFormData = z.infer<typeof MonitorSchema>;

export function MonitorForm({ monitor, onSubmit }: MonitorFormProps) {
  const form = useForm<MonitorFormData>({
    resolver: zodResolver(MonitorSchema),
    defaultValues: {
      name: monitor?.name || '',
      url: monitor?.url || '',
      interval: monitor?.interval || 300,
    },
  });

  const handleSubmit = async (data: MonitorFormData) => {
    try {
      await onSubmit(data);
    } catch (error) {
      // Handle submission errors
      if (error instanceof z.ZodError) {
        error.errors.forEach((err) => {
          form.setError(err.path[0] as keyof MonitorFormData, {
            message: err.message,
          });
        });
        return;
      }

      // Handle other errors
      form.setError('root', {
        message: error instanceof Error ? error.message : 'Failed to save monitor',
      });
    }
  };

  return (
    <Form {...form}>
      <form onSubmit={form.handleSubmit(handleSubmit)}>
        {/* Form fields */}
        {form.formState.errors.root && (
          <div className="text-sm text-red-600">
            {form.formState.errors.root.message}
          </div>
        )}
      </form>
    </Form>
  );
}
```

### Error Handling Rules

#### ✅ DO:
- Handle errors and edge cases at the beginning of functions
- Use early returns for error conditions to avoid deep nesting
- Implement guard clauses for preconditions and invalid states
- Use custom error types or factories for consistent error handling
- Log errors appropriately with context
- Provide user-friendly error messages
- Validate data with Zod schemas
- Handle loading and error states in UI components
- Use React Query's built-in error handling
- Implement proper error boundaries for fallback UI

#### ❌ DON'T:
- Ignore errors or fail silently
- Use generic catch-all error handlers without context
- Display technical error messages to users
- Create deeply nested error handling logic
- Forget to handle async errors properly
- Use `any` types for error objects

### Custom Error Types

```typescript
// lib/errors.ts
export class ApiError extends Error {
  constructor(
    message: string,
    public status: number,
    public code?: string
  ) {
    super(message);
    this.name = 'ApiError';
  }
}

export class ValidationError extends Error {
  constructor(
    message: string,
    public field: string
  ) {
    super(message);
    this.name = 'ValidationError';
  }
}

// Usage
function createMonitor(data: CreateMonitorDto) {
  // Early validation
  if (!data.name?.trim()) {
    throw new ValidationError('Monitor name is required', 'name');
  }

  if (!isValidUrl(data.url)) {
    throw new ValidationError('Invalid URL format', 'url');
  }

  // Continue with creation...
}
```

---

## 12. Testing

### Component Tests
```typescript
// tests/components/monitor-card.test.tsx
import { render, screen } from '@testing-library/react';
import userEvent from '@testing-library/user-event';
import { MonitorCard } from '@/components/monitors/monitor-card';

describe('MonitorCard', () => {
  const mockMonitor = {
    id: 1,
    name: 'Test Monitor',
    url: 'https://example.com',
    status: 'up',
  };

  it('should display monitor information', () => {
    render(<MonitorCard monitor={mockMonitor} />);

    expect(screen.getByText('Test Monitor')).toBeInTheDocument();
    expect(screen.getByText('https://example.com')).toBeInTheDocument();
    expect(screen.getByText('Online')).toBeInTheDocument();
  });

  it('should call onEdit when edit button clicked', async () => {
    const onEdit = jest.fn();
    const user = userEvent.setup();

    render(<MonitorCard monitor={mockMonitor} onEdit={onEdit} />);

    await user.click(screen.getByRole('button', { name: /edit/i }));

    expect(onEdit).toHaveBeenCalledWith(mockMonitor);
  });
});
```

### Hook Tests
```typescript
// tests/hooks/use-monitors.test.ts
import { renderHook, waitFor } from '@testing-library/react';
import { QueryClient, QueryClientProvider } from '@tanstack/react-query';
import { useMonitors } from '@/hooks/use-monitors';

describe('useMonitors', () => {
  const createWrapper = () => {
    const queryClient = new QueryClient({
      defaultOptions: { queries: { retry: false } },
    });

    return ({ children }: { children: React.ReactNode }) => (
      <QueryClientProvider client={queryClient}>
        {children}
      </QueryClientProvider>
    );
  };

  it('should fetch monitors successfully', async () => {
    const { result } = renderHook(() => useMonitors(1), {
      wrapper: createWrapper(),
    });

    expect(result.current.isLoading).toBe(true);

    await waitFor(() => {
      expect(result.current.isLoading).toBe(false);
    });

    expect(result.current.monitors).toHaveLength(2);
  });
});
```

### E2E Tests
```typescript
// tests/e2e/monitors.spec.ts
import { test, expect } from '@playwright/test';

test.describe('Monitor Management', () => {
  test('should create a new monitor', async ({ page }) => {
    await page.goto('/monitors');

    // Click create button
    await page.getByRole('button', { name: 'Create Monitor' }).click();

    // Fill form
    await page.getByLabel('Name').fill('Production API');
    await page.getByLabel('URL').fill('https://api.example.com');
    await page.getByLabel('Interval').selectOption('300');

    // Submit
    await page.getByRole('button', { name: 'Create' }).click();

    // Verify creation
    await expect(page.getByText('Production API')).toBeVisible();
    await expect(page.getByText('Monitor created successfully')).toBeVisible();
  });
});
```

---

## 📚 Additional Resources

- [Next.js Documentation](https://nextjs.org/docs)
- [React Query Documentation](https://tanstack.com/query)
- [TailwindCSS Documentation](https://tailwindcss.com/docs)
- [React Testing Library](https://testing-library.com/docs/react-testing-library/intro)

---

**Remember**: Prefer Server Components by default. Only use Client Components when you need interactivity!