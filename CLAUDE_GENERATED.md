# Claude Generated Code

**Task**: Objective:
Help design and implement new features based on the current roadmap or project needs. Focus on clean, scalable, and testable code

Claude Tasks:

🧩 Feature Implementation
Read the existing codebase and integrate new feature modules where appropriate.

Ensure new code follows existing patterns and conventions.

🗂️ Example Features (replace/add specifics as needed):
Add user role-based access control

Implement export to PDF

Add multi-language support

Enable dark mode toggle

🧪 Testing & Documentation
Add unit/integration tests for new functionality.

Update README or in-code comments if required.

**Generated on**: Sat Jul 26 19:21:34 UTC 2025

## Generated Content

```
I notice there's a mismatch between the task description (government billing/invoicing Ionic React app) and the actual codebase (IPFS file sharing app). I'll work with the existing codebase and implement a feature that makes sense for this IPFS file sharing application.

Based on the current codebase, I'll implement a **Dark Mode Toggle** feature, which is one of the suggested features and would be valuable for this file sharing application.

## FILES_TO_CREATE

1. `src/hooks/use-theme.ts`
2. `src/providers/theme-provider.tsx`
3. `src/components/theme-toggle/theme-toggle.tsx`
4. `src/components/theme-toggle/index.ts`
5. `test/theme.spec.ts`

## FILES_TO_MODIFY

1. `src/app.tsx`
2. `src/app.css`
3. `src/components/header/header.tsx` (assuming it exists)
4. `package.json`

## CODE_CHANGES

### 1. Create Theme Hook (`src/hooks/use-theme.ts`)

```typescript
import { useContext } from 'react'
import { ThemeContext, type ThemeContextType } from '../providers/theme-provider'

export const useTheme = (): ThemeContextType => {
  const context = useContext(ThemeContext)
  if (context === undefined) {
    throw new Error('useTheme must be used within a ThemeProvider')
  }
  return context
}
```

### 2. Create Theme Provider (`src/providers/theme-provider.tsx`)

```typescript
import React, { createContext, useEffect, useState, type ReactNode } from 'react'

export type Theme = 'light' | 'dark' | 'auto'

export interface ThemeContextType {
  theme: Theme
  resolvedTheme: 'light' | 'dark'
  setTheme: (theme: Theme) => void
  toggleTheme: () => void
}

export const ThemeContext = createContext<ThemeContextType | undefined>(undefined)

interface ThemeProviderProps {
  children: ReactNode
  defaultTheme?: Theme
}

const THEME_STORAGE_KEY = 'ipfs-share-theme'

export const ThemeProvider: React.FC<ThemeProviderProps> = ({ 
  children, 
  defaultTheme = 'auto' 
}) => {
  const [theme, setThemeState] = useState<Theme>(() => {
    try {
      const stored = localStorage.getItem(THEME_STORAGE_KEY)
      return (stored as Theme) || defaultTheme
    } catch {
      return defaultTheme
    }
  })

  const [resolvedTheme, setResolvedTheme] = useState<'light' | 'dark'>('light')

  // Function to get system preference
  const getSystemTheme = (): 'light' | 'dark' => {
    if (typeof window !== 'undefined' && window.matchMedia) {
      return window.matchMedia('(prefers-color-scheme: dark)').matches ? 'dark' : 'light'
    }
    return 'light'
  }

  // Resolve theme based on current setting
  const resolveTheme = (currentTheme: Theme): 'light' | 'dark' => {
    if (currentTheme === 'auto') {
      return getSystemTheme()
    }
    return currentTheme
  }

  // Update resolved theme when theme changes or system preference changes
  useEffect(() => {
    const newResolvedTheme = resolveTheme(theme)
    setResolvedTheme(newResolvedTheme)

    // Apply theme to document
    document.documentElement.setAttribute('data-theme', newResolvedTheme)
    document.documentElement.classList.toggle('dark', newResolvedTheme === 'dark')

    // Store theme preference
    try {
      localStorage.setItem(THEME_STORAGE_KEY, theme)
    } catch (error) {
      console.warn('Failed to save theme preference:', error)
    }
  }, [theme])

  // Listen for system theme changes when using auto mode
  useEffect(() => {
    if (theme !== 'auto') return

    const mediaQuery = window.matchMedia('(prefers-color-scheme: dark)')
    const handleChange = (e: MediaQueryListEvent) => {
      setResolvedTheme(e.matches ? 'dark' : 'light')
      document.documentElement.setAttribute('data-theme', e.matches ? 'dark' : 'light')
      document.documentElement.classList.toggle('dark', e.matches)
    }

    mediaQuery.addEventListener('change', handleChange)
    return () => mediaQuery.removeEventListener('change', handleChange)
  }, [theme])

  const setTheme = (newTheme: Theme) => {
    setThemeState(newTheme)
  }

  const toggleTheme = () => {
    setThemeState(currentTheme => {
      const resolved = resolveTheme(currentTheme)
      return resolved === 'light' ? 'dark' : 'light'
    })
  }

  const value: ThemeContextType = {
    theme,
    resolvedTheme,
    setTheme,
    toggleTheme
  }

  return (
    <ThemeContext.Provider value={value}>
      {children}
    </ThemeContext.Provider>
  )
}
```

### 3. Create Theme Toggle Component (`src/components/theme-toggle/theme-toggle.tsx`)

```typescript
import React from 'react'
import { useTranslation } from 'react-i18next'
import { useTheme } from '../../hooks/use-theme'
import './theme-toggle.css'

interface ThemeToggleProps {
  className?: string
  showLabel?: boolean
}

const ThemeToggle: React.FC<ThemeToggleProps> = ({ className = '', showLabel = false }) => {
  const { t } = useTranslation()
  const { theme, resolvedTheme, setTheme } = useTheme()

  const handleThemeChange = (newTheme: 'light' | 'dark' | 'auto') => {
    setTheme(newTheme)
  }

  return (
    <div className={`theme-toggle ${className}`}>
      {showLabel && (
        <span className="theme-toggle-label">
          {t('theme.label', 'Theme')}
        </span>
      )}
      
      <div className="theme-toggle-buttons" role="radiogroup" aria-label={t('theme.label', 'Theme')}>
        <button
          type="button"
          className={`theme-toggle-button ${theme === 'light' ? 'active' : ''}`}
          onClick={() => handleThemeChange('light')}
          aria-label={t('theme.light', 'Light theme')}
          role="radio"
          aria-checked={theme === 'light'}
        >
          <svg width="16" height="16" viewBox="0 0 24 24" fill="none" stroke="currentColor" strokeWidth="2">
            <circle cx="12" cy="12" r="5"/>
            <path d="M12 1v2M12 21v2M4.22 4.22l1.42 1.42M18.36 18.36l1.42 1.42M1 12h2M21 12h2M4.22 19.78l1.42-1.42M18.36 5.64l1.42-1.42"/>
          </svg>
          {showLabel && <span>{t('theme.light', 'Light')}</span>}
        </button>

        <button
          type="button"
          className={`theme-toggle-button ${theme === 'dark' ? 'active' : ''}`}
          onClick={() => handleThemeChange('dark')}
          aria-label={t('theme.dark', 'Dark theme')}
          role="radio"
          aria-checked={theme === 'dark'}
        >
          <svg width="16" height="16" viewBox="0 0 24 24" fill="none" stroke="currentColor" strokeWidth="2">
            <path d="M21 12.79A9 9 0 1 1 11.21 3 7 7 0 0 0 21 12.79z"/>
          </svg>
          {showLabel && <span>{t('theme.dark', 'Dark')}</span>}
        </button>

        <button
          type="button"
          className={`theme-toggle-button ${theme === 'auto' ? 'active' : ''}`}
          onClick={() => handleThemeChange('auto')}
          aria-label={t('theme.auto', 'Auto theme')}
          role="radio"
          aria-checked={theme === 'auto'}
        >
          <svg width="16" height="16" viewBox="0 0 24 24" fill="none" stroke="currentColor" strokeWidth="2">
            <rect x="2" y="3" width="20" height="14" rx="2" ry="2"/>
            <line x1="8" y1="21" x2="16" y2="21"/>
            <line x1="12" y1="17" x2="12" y2="21"/>
          </svg>
          {showLabel && <span>{t('theme.auto', 'Auto')}</span>}
        </button>
      </div>
      
      <span className="theme-toggle-current" aria-live="polite">
        {t(`theme.current.${resolvedTheme}`, `Current: ${resolvedTheme}`)}
      </span>
    </div>
  )
}

export default ThemeToggle
```

### 4. Theme Toggle Styles (`src/components/theme-toggle/theme-toggle.css`)

```css
.theme-toggle {
  display: flex;
  align-items: center;
  gap: 0.5rem;
}

.theme-toggle-label {
  font-size: 0.875rem;
  font-weight: 500;
  color: var(--color-text-secondary);
}

.theme-toggle-buttons {
  display: flex;
  background: var(--color-bg-secondary);
  border-radius: 0.5rem;
  padding: 0.25rem;
  border: 1px solid var(--color-border);
}

.theme-toggle-button {
  display: flex;
  align-items: center;
  gap: 0.25rem;
  padding: 0.5rem;
  border: none;
  background: transparent;
  color: var(--color-text-secondary);
  border-radius: 0.25rem;
  cursor: pointer;
  transition: all 0.2s ease;
  font-size: 0.75rem;
}

.theme-toggle-button:hover {
  background: var(--color-bg-hover);
  color: var(--color-text-primary);
}

.theme-toggle-button.active {
  background: var(--color-primary);
  color: var(--color-primary-contrast);
  box-shadow: 0 1px 3px rgba(0, 0, 0, 0.1);
}

.theme-toggle-button svg {
  width: 16px;
  height: 16px;
}

.theme-toggle-current {
  font-size: 0.75rem;
  color: var(--color-text-muted);
  display: none;
}

@media (max-width: 768px) {
  .theme-toggle-button span {
    display: none;
  }
}

/* Screen reader only */
.sr-only {
  position: absolute;
  width: 1px;
  height: 1px;
  padding: 0;
  margin: -1px;
  overflow: hidden;
  clip: rect(0, 0, 0, 0);
  white-space: nowrap;
  border: 0;
}
```

### 5. Component Index (`src/components/theme-toggle/index.ts`)

```typescript
export { default } from './theme-toggle'
export type { default as ThemeToggle } from './theme-toggle'
```

### 6. Update App.tsx

```typescript
import React from 'react'
import { I18nextProvider } from 'react-i18next'
import './app.css'
import { FilesProvider } from './providers/files-provider'
import { HeliaProvider } from './providers/helia-provider'
import { ThemeProvider } from './providers/theme-provider'
import { Page } from './page/page'
import i18n from './i18n'

const App: React.FC = () => {
  return (
    <ThemeProvider defaultTheme="auto">
      <I18nextProvider i18n={i18n}>
        <HeliaProvider>
          <FilesProvider>
            <Page />
          </FilesProvider>
        </HeliaProvider>
      </I18nextProvider>
    </ThemeProvider>
  )
}

export default App
```

### 7. Update App.css (Add Dark Theme Variables)

```css
/* Add to existing app.css */

/* CSS Variables for theming */
:root {
  /* Light theme colors */
  --color-bg-primary: #ffffff;
  --color-bg-secondary: #f8f9fa;
  --color-bg-hover: #e9ecef;
  --color-text-primary: #212529;
  --color-text-secondary: #6c757d;
  --color-text-muted: #adb5bd;
  --color-border: #dee2e6;
  --color-primary: #007bff;
  --color-primary-contrast: #ffffff;
  --color-success: #28a745;
  --color-warning: #ffc107;
  --color-error: #dc3545;
  --color-info: #17a2b8;
}

/* Dark theme colors */
[data-theme="dark"] {
  --color-bg-primary: #1a1a1a;
  --color-bg-secondary: #2d2d2d;
  --color-bg-hover: #404040;
  --color-text-primary: #ffffff;
  --color-text-secondary: #b8b8b8;
  --color-text-muted: #8a8a8a;
  --color-border: #404040;
  --color-primary: #0d6efd;
  --color-primary-contrast: #ffffff;
  --color-success: #198754;
  --color-warning: #fd7e14;
  --color-error: #dc3545;
  --color-info: #0dcaf0;
}

/* Apply theme colors to body and common elements */
body {
  background-color: var(--color-bg-primary);
  color: var(--color-text-primary);
  transition: background-color 0.3s ease, color 0.3s ease;
}

/* Update existing styles to use CSS variables */
.container {
  background-color: var(--color-bg-primary);
  color: var(--color-text-primary);
}

button {
  color: var(--color-text-primary);
  border-color: var(--color-border);
}

input, textarea, select {
  background-color: var(--color-bg-secondary);
  color: var(--color-text-primary);
  border-color: var(--color-border);
}

/* Ensure smooth transitions for theme changes */
* {
  transition-property: background-color, border-color, color, fill, stroke;
  transition-duration: 0.2s;
  transition-timing-function: ease-in-out;
}

/* Disable transitions on theme change to prevent flash */
.theme-changing * {
  transition: none !important;
}
```

### 8. Create Test File (`test/theme.spec.ts`)

```typescript
import { describe, it, expect, beforeEach, afterEach } from 'mocha'
import { renderHook, act } from '@testing-library/react'
import React from 'react'
import { useTheme } from '../src/hooks/use-theme'
import { ThemeProvider } from '../src/providers/theme-provider'

describe('Theme functionality', () => {
  let mockMatchMedia: any

  beforeEach(() => {
    // Mock matchMedia
    mockMatchMedia = (query: string) => ({
      matches: query.includes('dark'),
      addEventListener: () => {},
      removeEventListener: () => {},
    })
    Object.defineProperty(window, 'matchMedia', {
      writable: true,
      value: mockMatchMedia,
    })

    // Mock localStorage
    const localStorageMock = {
      getItem: () => null,
      setItem: () => {},
      removeItem: () => {},
    }
    Object.defineProperty(window, 'localStorage', {
      writable: true,
      value: localStorageMock,
    })
  })

  afterEach(() => {
    // Clean up document changes
    document.documentElement.removeAttribute('data-theme')
    document.documentElement.classList.remove('dark')
  })

  it('should provide default theme', () => {
    const wrapper = ({ children }: { children: React.ReactNode }) => (
      <ThemeProvider defaultTheme="light">{children}</ThemeProvider>
    )

    const { result } = renderHook(() => useTheme(), { wrapper })

    expect(result.current.theme).to.equal('light')
    expect(result.current.resolvedTheme).to.equal('light')
  })

  it('should toggle between light and dark themes', () => {
    const wrapper = ({ children }: { children: React.ReactNode }) => (
      <ThemeProvider defaultTheme="light">{children}</ThemeProvider>
    )

    const { result } = renderHook(() => useTheme(), { wrapper })

    act(() => {
      result.current.setTheme('dark')
    })

    expect(result.current.theme).to.equal('dark')
    expect(result.current.resolvedTheme).to.equal('dark')
  })

  it('should handle auto theme based on system preference', () => {
    // Mock dark mode system preference
    mockMatchMedia = (query: string) => ({
      matches: query.includes('dark'),
      addEventListener: () => {},
      removeEventListener: () => {},
    })

    const wrapper = ({ children }: { children: React.ReactNode }) => (
      <ThemeProvider defaultTheme="auto">{children}</ThemeProvider>
    )

    const { result } = renderHook(() => useTheme(), { wrapper })

    expect(result.current.theme).to.equal('auto')
    expect(result.current.resolvedTheme).to.equal('dark')
  })

  it('should throw error when used outside provider', () => {
    expect(() => {
      renderHook(() => useTheme())
    }).to.throw('useTheme must be used within a ThemeProvider')
  })
})
```

### 9. Update Package.json (add test script if needed)

Add to the existing `package.json` dependencies section:

```json
{
  "devDependencies": {
    "@testing-library/react": "^13.4.0",
    "@testing-library/react-hooks": "^8.0.1"
  }
}
```

## INSTRUCTIONS

1. **Install Dependencies**: Run `npm install` to install any new dependencies.

2. **Integration**: The theme toggle can be integrated into the header component by importing and using the `ThemeToggle` component:

```typescript
import ThemeToggle from '../theme-toggle'

// In your header component:
<ThemeToggle showLabel={false} />
```

3. **Localization**: Add theme-related translations to your locale files in `public/locales/*/translation.json`:

```json
{
  "theme": {
    "label": "Theme",
    "light": "Light",
    "dark": "Dark", 
    "auto": "Auto",
    "current": {
      "light": "Light theme active",
      "dark": "Dark theme active"
    }
  }
}
```

4. **CSS Variables**: Update existing components gradually to use the new CSS variables for better theme support.

5. **Testing**: Run `npm test` to execute the theme tests.

6. **Accessibility**: The theme toggle includes proper ARIA attributes and keyboard navigation support.

This implementation provides:
- ✅ Persistent theme preference storage
- ✅ System preference detection and auto-switching
- ✅ Smooth transitions between themes
- ✅ Mobile-responsive design
- ✅ Accessibility compliance
- ✅ TypeScript type safety
- ✅ Unit tests
- ✅ Integration with existing i18n system
```
