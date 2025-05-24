# UI_GUIDE.md

## 1. Personas

### Primary Personas

**Priya Chen** (25-35, Financial Analyst)
- **Goal**: Achieve FIRE (Financial Independence, Retire Early) by age 45
- **Jobs-to-be-Done**: Optimize CPF vs ETF allocation, share financial plan with fiancé
- **Pain Points**: Spreadsheets don't model CPF complexity, can't easily collaborate on scenarios
- **Success Metrics**: Creates optimized plan in <60 seconds, shares with fiancé same session, achieves CSAT ≥4.5/5

**Mr Lim Wei** (40-50, Parent of Two)  
- **Goal**: Protect children's education funding while carrying mortgage debt
- **Jobs-to-be-Done**: Stress-test ability to handle dual loans, ensure education remains funded
- **Pain Points**: Anxiety about overlapping commitments, no clear view of safety margins
- **Success Metrics**: Runs 3+ stress scenarios in first session, confidence score increases 40%+

### Secondary Personas

**Nadia Rahman** (30, Freelance Designer)
- **Goal**: Take sabbatical without financial stress, manage volatile income
- **Jobs-to-be-Done**: Evaluate cash-flow robustness, determine optimal timing for break
- **Pain Points**: Traditional tools assume steady salaries, can't model income volatility
- **Success Metrics**: Identifies safe sabbatical window with 90%+ confidence, exports actionable plan

## 2. Core User Journeys

### First-Run Onboarding (Target: ≤60s to First Insight)
1. **Land on homepage** → Animated preview "Run the numbers on life"
2. **Click "Start Planning"** → 3-panel carousel launches
   - Panel 1: Welcome with Lolo mascot introduction
   - Panel 2: Goal selection (FIRE/Property/Education/Custom)
   - Panel 3: Minimal inputs (age, income, current savings)
3. **Skip detailed fields** → "Add details later" prominently displayed
4. **First projection appears** → Chart animates within 60 seconds
5. **Tooltip guides interaction** → "Drag to adjust retirement age"
6. **See instant recalculation** → Numbers update in <200ms
7. **"Explain it" pulses** → Subtle animation draws attention
8. **Read AI explanation** → Plain language summary appears
9. **Decision point** → "Create free account" or "Keep exploring"

### Document Import Flow
1. **User drops CPF PDF** → Dropzone overlay with dashed border
2. **Processing begins** → "Analyzing with on-device AI..." spinner
3. **Extracted data displays** → Values shown with confidence badges
4. **Low confidence prompts** → Llama asks "Is your SA balance $45,230?"
5. **User confirms/corrects** → Inline editing with validation
6. **Timeline updates** → Real data replaces estimates
7. **Personalized insight** → "Based on your CPF, retire 3 years earlier"

### LLM Timeout Recovery
1. **Chat stalls** → After 8s show "Taking longer than usual..."
2. **Offer alternatives** → "Try again" or "Use simple mode" buttons
3. **Neutral placeholder** → Generic response while retrying
4. **Fallback templates** → Pre-written answers for common questions
5. **Context preserved** → Previous messages remain visible
6. **Background retry** → Notification when AI becomes available

### Stress-Test Flow
1. **Click "Stress-Test"** → Modal slides up from bottom
2. **Preset scenarios** → "2008 Crisis", "1997 AFC", "Custom"
3. **Real-time preview** → Chart updates as sliders move
4. **Adjust parameters** → Interest +3%, Market -40%, Inflation +2%
5. **See impact overlay** → Red shading on affected years
6. **Compare outcomes** → Split view of base vs stressed scenarios
7. **Get recommendations** → "Add 6 months emergency fund to survive"

## 3. Screen & Route Map

| Screen | Purpose | Path | Auth | Key Elements |
|--------|---------|------|------|--------------|
| **Home** | Product overview, value proposition | `/` | No | Animated hero, testimonials, CTA |
| **Scenario Canvas** | Main planning interface | `/plan` | Optional | Timeline, charts, input panel |
| **Results** | Detailed projections and analysis | `/plan/results` | Optional | Multi-chart dashboard, KPIs |
| **Explain Modal** | AI-generated insights | Modal overlay | Optional | Narrative, confidence badge, trace links |
| **Chat Interface** | Natural language planning | `/plan/chat` | Yes (Free+) | Messages, voice input (Plus), suggestions |
| **Stress Test** | Crisis scenario modeling | `/plan/stress` | Yes (Free+) | Presets, sliders, comparison view |
| **Dashboard** | Saved scenarios overview | `/dashboard` | Yes | Grid cards, last modified, quick actions |
| **Settings** | Account, privacy, data management | `/settings` | Yes | Profile, privacy toggle, data export |
| **Pricing** | Tier comparison and upgrade | `/pricing` | No | Feature matrix, FAQs, testimonials |
| **Share View** | Read-only scenario access | `/share/:id` | No | View-only canvas, comments, fork option |
| **Export Center** | Download options | `/export/:id` | Yes | Format selection, preview, email option |

## 4. Component Inventory

### Layout Components
```typescript
// AppShell.tsx - Main application wrapper
interface AppShellProps {
  children: ReactNode;
  showNav?: boolean;
  showFooter?: boolean;
  tier?: 'free' | 'pro' | 'plus';
  privacyMode?: 'local' | 'cloud';
}

// NavigationBar.tsx - Top navigation with tier indicator
interface NavigationBarProps {
  user?: User;
  tier?: TierType;
  onMenuClick: () => void;
  transparentBg?: boolean;
  showShield?: boolean; // Privacy mode indicator
}

// SidePanel.tsx - Collapsible input panel
interface SidePanelProps {
  isOpen: boolean;
  onClose: () => void;
  children: ReactNode;
  position?: 'left' | 'right';
  width?: 'narrow' | 'standard' | 'wide';
}

// BottomSheet.tsx - Mobile-optimized drawer
interface BottomSheetProps {
  isOpen: boolean;
  onClose: () => void;
  snapPoints?: number[]; // [0.2, 0.5, 0.9] = 20%, 50%, 90% height
  initialSnap?: number;
  children: ReactNode;
}
```

### Canvas Components
```typescript
// ScenarioCanvas.tsx - Main planning container
interface ScenarioCanvasProps {
  scenario: Scenario;
  onUpdate: (updates: ScenarioUpdate) => void;
  isLoading?: boolean;
  tier: TierType;
  showMonteCarlo?: boolean; // Pro+ feature
}

// Timeline.tsx - D3.js horizontal timeline
interface TimelineProps {
  events: TimelineEvent[];
  onEventDrag: (eventId: string, newAge: number) => void;
  onEventEdit: (eventId: string) => void;
  onEventDelete: (eventId: string) => void;
  onEventAdd: (age: number) => void;
  startAge: number;
  endAge: number;
  currentAge: number;
  zoom: number; // 0.5 to 2.0
  onZoom: (delta: number) => void;
  colorMode?: 'standard' | 'deuteranopia';
}

// EventCard.tsx - Draggable timeline event
interface EventCardProps {
  event: TimelineEvent;
  isSelected?: boolean;
  isDragging?: boolean;
  onEdit: () => void;
  onDelete: () => void;
  onLongPress?: () => void; // Mobile context menu
  color?: string;
  stackIndex?: number; // For overlapping events
}

// YearMarker.tsx - Timeline year indicators
interface YearMarkerProps {
  year: number;
  age: number;
  isCurrentYear?: boolean;
  isMilestone?: boolean; // Retirement, etc.
  label?: string;
}
```

### Visualization Components
```typescript
// ProjectionChart.tsx - Main cash flow visualization
interface ProjectionChartProps {
  data: YearlyProjection[];
  showConfidenceBands?: boolean;
  confidenceData?: ConfidenceBands;
  highlightYear?: number;
  onHover: (year: number | null) => void;
  height?: number;
  animate?: boolean;
  colorScheme?: 'standard' | 'deuteranopia';
  onTraceClick?: (dataPoint: DataPoint) => void;
}

// NetWorthChart.tsx - Wealth accumulation line
interface NetWorthChartProps {
  data: NetWorthData[];
  milestones?: Milestone[];
  targetAmount?: number;
  showProbability?: boolean;
  confidenceBands?: ConfidenceBands;
}

// MetricCard.tsx - Key performance indicators
interface MetricCardProps {
  label: string;
  value: string | number;
  change?: number;
  icon?: IconType;
  trend?: 'up' | 'down' | 'neutral';
  onClick?: () => void;
  showTrace?: boolean; // Show calculation trace icon
  confidence?: number; // For AI-generated metrics
}

// StressGauge.tsx - Scenario robustness indicator
interface StressGaugeProps {
  score: number; // 0-100
  label?: string;
  thresholds?: { low: number; medium: number };
  animate?: boolean;
}
```

### Input Components
```typescript
// MoneyInput.tsx - Currency-formatted input
interface MoneyInputProps {
  value: number;
  onChange: (value: number) => void;
  currency?: 'SGD' | 'USD' | 'MYR' | 'IDR';
  min?: number;
  max?: number;
  step?: number;
  placeholder?: string;
  showCalculator?: boolean;
  error?: string;
}

// AgeSlider.tsx - Visual age selection
interface AgeSliderProps {
  value: number;
  onChange: (age: number) => void;
  min?: number;
  max?: number;
  markers?: { age: number; label: string }[];
  currentAge?: number;
  retirementAge?: number;
}

// EventTypeSelector.tsx - Icon-based category picker
interface EventTypeSelectorProps {
  value: EventType;
  onChange: (type: EventType) => void;
  options?: EventType[];
  layout?: 'grid' | 'list';
}

// PercentageInput.tsx - Percentage with validation
interface PercentageInputProps {
  value: number;
  onChange: (value: number) => void;
  min?: number;
  max?: number;
  step?: number;
  label?: string;
  helperText?: string;
}
```

### AI/Chat Components
```typescript
// ExplainButton.tsx - Trigger AI explanation
interface ExplainButtonProps {
  scenarioId: string;
  context?: 'chart' | 'event' | 'metric' | 'general';
  onExplain: (explanation: AIExplanation) => void;
  tier: TierType;
  pulseAnimation?: boolean;
}

// NarrativePanel.tsx - AI explanation display
interface NarrativePanelProps {
  explanation: AIExplanation;
  showConfidence?: boolean;
  showSources?: boolean;
  showTrace?: boolean;
  onTraceClick: (calculationId: string) => void;
  onQuestionChipClick?: (question: string) => void;
}

// ChatInterface.tsx - Conversational planning
interface ChatInterfaceProps {
  scenarioId: string;
  isOpen: boolean;
  onClose: () => void;
  position?: 'bottom-sheet' | 'side-dock';
  tier: TierType;
  enableVoice?: boolean; // Plus tier only
  onMicrophoneClick?: () => void;
}

// TypingIndicator.tsx - AI processing feedback
interface TypingIndicatorProps {
  isTyping: boolean;
  message?: string;
  estimatedTime?: number;
}

// ConfidenceBadge.tsx - AI confidence display
interface ConfidenceBadgeProps {
  score: number; // 0-100
  size?: 'sm' | 'md' | 'lg';
  showLabel?: boolean;
  onClick?: () => void;
}
```

### Export Components
```typescript
// ExportModal.tsx - Download options
interface ExportModalProps {
  isOpen: boolean;
  onClose: () => void;
  scenarioId: string;
  availableFormats: ExportFormat[];
  onExport: (format: ExportFormat) => void;
}

// ShareModal.tsx - Sharing configuration
interface ShareModalProps {
  isOpen: boolean;
  onClose: () => void;
  scenarioId: string;
  shareUrl?: string;
  permissions?: SharePermissions;
  onShare: (options: ShareOptions) => void;
}

// ImportDropzone.tsx - File upload area
interface ImportDropzoneProps {
  onDrop: (files: File[]) => void;
  accept?: string[];
  maxSize?: number;
  isProcessing?: boolean;
  confidence?: number; // Parsing confidence
}
```

### Empty States & Feedback
```typescript
// EmptyState.tsx - Lolo mascot states
interface EmptyStateProps {
  variant: 'no-data' | 'no-scenarios' | 'offline' | 'error';
  title?: string;
  description?: string;
  illustration?: string; // Lolo variant
  action?: {
    label: string;
    onClick: () => void;
  };
}

// ErrorBoundary.tsx - Graceful error handling
interface ErrorBoundaryProps {
  children: ReactNode;
  fallback?: ComponentType<{ error: Error }>;
  onError?: (error: Error, errorInfo: ErrorInfo) => void;
}

// LoadingSpinner.tsx - Consistent loading
interface LoadingSpinnerProps {
  size?: 'sm' | 'md' | 'lg';
  label?: string;
  fullScreen?: boolean;
  progress?: number; // For determinate loading
}

// ToastNotification.tsx - Feedback messages
interface ToastNotificationProps {
  message: string;
  variant?: 'success' | 'error' | 'info' | 'warning';
  duration?: number; // Auto-dismiss after ms
  action?: {
    label: string;
    onClick: () => void;
  };
  onClose?: () => void;
}
```

### Privacy Components
```typescript
// PrivacyToggle.tsx - Local/Cloud mode switch
interface PrivacyToggleProps {
  mode: 'local' | 'cloud';
  onChange: (mode: 'local' | 'cloud') => void;
  showShieldIcon?: boolean;
  disabled?: boolean;
  tooltip?: string;
}

// DataDeleteButton.tsx - GDPR compliance
interface DataDeleteButtonProps {
  onDelete: () => Promise<void>;
  requireConfirmation?: boolean;
  showProgress?: boolean;
  includeLocal?: boolean;
  includeCloud?: boolean;
}

// OfflineBanner.tsx - Offline mode indicator
interface OfflineBannerProps {
  isOffline: boolean;
  mode: 'local' | 'cloud';
  onDismiss?: () => void;
  position?: 'top' | 'bottom';
}
```

## 5. Interaction Rules

### Drag & Drop Specifications
- **Initiation Thresholds**
  - Desktop: Click-hold 200ms or immediate drag >5px
  - Mobile: Long press 500ms with haptic feedback
  - Touch: 44x44px minimum hit area
  
- **Visual Feedback**
  - Dragging: Scale 1.05x, shadow depth 28px, opacity 0.9
  - Ghost image: Follows cursor with 2px offset
  - Drop zones: Mint highlight (#2DF2AE) with 2px border
  - Invalid zones: Red border (#FF4545) with shake animation
  
- **Constraints**
  - Cannot drag to past (before current age)
  - Cannot drag beyond age 100
  - Snap to year boundaries (±6 months)
  - Auto-scroll at edges (20px threshold)
  
- **Collision Handling**
  - Stack vertically with 4px gap
  - Maximum 3 events per year
  - Overflow indicator if more
  
- **Completion**
  - Smooth animation: 120ms ease-out
  - Recalculation: Triggers immediately
  - Undo toast: Shows for 3s with action

### Tooltip Behavior
- **Desktop Hover**
  - Delay: 200ms show, 0ms hide
  - Position: Smart positioning (avoid edges)
  - Content: Value + context + action hint
  
- **Mobile Touch**
  - Tap to show, tap elsewhere to hide
  - Position: Above finger to avoid occlusion
  - Dismiss: After 5s or interaction
  
- **Content Types**
  - Data points: "$125,000 at age 45"
  - Formulas: "Click to see calculation"
  - Abbreviations: "CPF (Central Provident Fund)"
  - Actions: "Drag to adjust"

### Keyboard Shortcuts
```
// Global shortcuts
Cmd/Ctrl + N        New event
Cmd/Ctrl + S        Save scenario
Cmd/Ctrl + K        Quick search
Cmd/Ctrl + /        Focus chat input
Cmd/Ctrl + E        Toggle explain panel
Cmd/Ctrl + P        Print/Export
Cmd/Ctrl + Z        Undo
Cmd/Ctrl + Shift + Z  Redo (or Cmd/Ctrl + Y)

// Timeline navigation
←/→                 Navigate events horizontally
↑/↓                 Navigate stacked events
Enter               Edit selected event
Delete/Backspace    Delete selected event
Space               Toggle event selection
Tab/Shift+Tab       Cycle through events
+/-                 Zoom in/out
0                   Reset zoom

// Accessibility
Tab                 Navigate focusable elements
Shift + Tab         Navigate backwards
Space/Enter         Activate buttons
Escape              Close modals/panels
F6                  Jump between panels
?                   Show keyboard shortcuts
```

### Mobile Gestures
- **Timeline Navigation**
  - Pinch: Zoom 0.5x to 2.0x
  - Two-finger drag: Pan horizontally
  - Swipe momentum: Kinetic scrolling
  
- **Bottom Sheet**
  - Swipe up: Open to snap points
  - Swipe down: Close or minimize
  - Drag handle: Fine position control
  
- **Other Gestures**
  - Long press: Context menu
  - Double tap: Quick zoom to event
  - Pull down: Refresh (with spinner)
  - Edge swipe: Back navigation

### Animation Specifications
```css
/* Framer Motion config */
const transitions = {
  fast: { duration: 0.12, ease: [0.4, 0, 0.2, 1] },
  normal: { duration: 0.2, ease: [0.4, 0, 0.2, 1] },
  slow: { duration: 0.3, ease: [0.4, 0, 0.2, 1] }
};

/* Reduced motion */
const prefersReducedMotion = window.matchMedia(
  "(prefers-reduced-motion: reduce)"
).matches;

const safeTransition = prefersReducedMotion 
  ? { duration: 0.01 } 
  : transitions.normal;
```

## 6. Visual Design Tokens

### Color Palette
```css
/* Primary Brand */
--color-mint: #2DF2AE;           /* Electric Mint - Primary CTA */
--color-mint-dark: #1FD494;      /* Hover state */
--color-mint-light: #5FF5C3;     /* Backgrounds */
--color-mint-alpha-20: rgba(45, 242, 174, 0.2); /* Overlays */

/* Dark Theme Base */
--color-navy: #0F101A;           /* Main background */
--color-navy-light: #1A1B28;     /* Card background */
--color-navy-lighter: #252636;   /* Borders, dividers */
--color-navy-alpha-90: rgba(15, 16, 26, 0.9); /* Modal backdrop */

/* Text Hierarchy */
--color-text-primary: #FFFFFF;   /* Main text */
--color-text-secondary: #A0A0B8; /* Secondary text */
--color-text-muted: #6B6B85;     /* Disabled, hints */
--color-text-inverse: #0F101A;   /* On light backgrounds */

/* Semantic Colors */
--color-success: #2DF2AE;        /* Positive outcomes */
--color-warning: #FFB800;        /* Caution states */
--color-error: #FF4545;          /* Errors, negative */
--color-info: #00B4D8;           /* Information */

/* Data Visualization */
--color-income: #2DF2AE;         /* Income streams */
--color-expense: #FF6B6B;        /* Expenses */
--color-savings: #4ECDC4;        /* Savings rate */
--color-investment: #95E1D3;     /* Investment returns */
--color-cpf: #FFA502;            /* CPF contributions */
--color-tax: #C44569;            /* Tax obligations */
--color-housing: #786FA6;        /* Property costs */
--color-confidence-band: rgba(45, 242, 174, 0.2); /* Monte Carlo bands */

/* Deuteranopia-Safe Palette */
--color-deut-1: #0173B2;         /* Blue */
--color-deut-2: #DE8F05;         /* Orange */
--color-deut-3: #029E73;         /* Green */
--color-deut-4: #CC78BC;         /* Purple */
--color-deut-5: #CA9161;         /* Brown */
--color-deut-6: #FBAFE4;         /* Pink */

/* High Contrast Mode */
@media (prefers-contrast: high) {
  --color-mint: #5FF5C3;
  --color-text-secondary: #E0E0E8;
  --color-navy-light: #000000;
  --card-border-width: 2px;
}
```

### Typography System
```css
/* Font Families */
--font-display: 'Sora', -apple-system, BlinkMacSystemFont, sans-serif;
--font-body: 'Inter', -apple-system, BlinkMacSystemFont, sans-serif;
--font-mono: 'JetBrains Mono', 'SF Mono', Consolas, monospace;

/* Type Scale */
--text-xs: 0.75rem;      /* 12px - Captions, labels */
--text-sm: 0.875rem;     /* 14px - Body small */
--text-base: 1rem;       /* 16px - Body default */
--text-lg: 1.125rem;     /* 18px - Body large */
--text-xl: 1.25rem;      /* 20px - Heading small */
--text-2xl: 1.5rem;      /* 24px - Heading medium */
--text-3xl: 2rem;        /* 32px - Heading large */
--text-4xl: 2.5rem;      /* 40px - Display */

/* Font Weights */
--font-regular: 400;     /* Body text */
--font-medium: 500;      /* Emphasis */
--font-semibold: 600;    /* Headings */
--font-bold: 700;        /* Strong emphasis */

/* Line Heights */
--leading-tight: 1.2;    /* Headings */
--leading-snug: 1.375;   /* Subheadings */
--leading-normal: 1.5;   /* Body text */
--leading-relaxed: 1.75; /* Long form text */

/* Letter Spacing */
--tracking-tight: -0.02em;  /* Headlines */
--tracking-normal: 0;       /* Body */
--tracking-wide: 0.02em;    /* Uppercase labels */
```

### Spacing System
```css
/* Base unit: 4px */
--spacing-0: 0;
--spacing-xs: 0.25rem;    /* 4px */
--spacing-sm: 0.5rem;     /* 8px */
--spacing-md: 1rem;       /* 16px */
--spacing-lg: 1.5rem;     /* 24px */
--spacing-xl: 2rem;       /* 32px */
--spacing-2xl: 3rem;      /* 48px */
--spacing-3xl: 4rem;      /* 64px */

/* Component spacing */
--stack-xs: var(--spacing-xs);
--stack-sm: var(--spacing-sm);
--stack-md: var(--spacing-md);
--stack-lg: var(--spacing-lg);
--inline-xs: var(--spacing-xs);
--inline-sm: var(--spacing-sm);
--inline-md: var(--spacing-md);
```

### Component Styling
```css
/* Cards */
--card-radius: 12px;
--card-shadow: 0 8px 28px rgba(0, 0, 0, 0.3);
--card-shadow-hover: 0 12px 36px rgba(0, 0, 0, 0.4);
--card-padding: var(--spacing-lg);
--card-bg: var(--color-navy-light);
--card-border: 1px solid var(--color-navy-lighter);

/* Buttons */
--button-radius: 8px;
--button-height-sm: 32px;
--button-height-md: 40px;
--button-height-lg: 48px;
--button-padding-sm: 0 12px;
--button-padding-md: 0 16px;
--button-padding-lg: 0 24px;
--button-font-weight: var(--font-medium);
--button-transition: all 120ms cubic-bezier(0.4, 0, 0.2, 1);

/* Inputs */
--input-radius: 8px;
--input-height: 40px;
--input-padding: 0 12px;
--input-font-size: var(--text-base);
--input-border: 1px solid var(--color-navy-lighter);
--input-border-focus: 1px solid var(--color-mint);
--input-shadow-focus: 0 0 0 2px rgba(45, 242, 174, 0.2);
--input-bg: var(--color-navy);
--input-bg-disabled: var(--color-navy-light);

/* Modals */
--modal-radius: 16px;
--modal-shadow: 0 20px 60px rgba(0, 0, 0, 0.5);
--modal-max-width: 600px;
--modal-padding: var(--spacing-xl);
--modal-backdrop: var(--color-navy-alpha-90);

/* Animations */
--animation-fast: 120ms;
--animation-normal: 200ms;
--animation-slow: 300ms;
--easing-default: cubic-bezier(0.4, 0, 0.2, 1);
--easing-ease-in: cubic-bezier(0.4, 0, 1, 1);
--easing-ease-out: cubic-bezier(0, 0, 0.2, 1);
```

## 7. Accessibility Targets

### WCAG 2.2 AA Compliance

**Success Criteria Implementation**

| Criterion | Implementation | Testing Method |
|-----------|----------------|----------------|
| **1.4.3 Contrast (Minimum)** | 4.5:1 normal text, 3:1 large text | Automated + manual audit |
| **1.4.11 Non-text Contrast** | 3:1 UI components and graphics | Design system enforcement |
| **2.1.1 Keyboard** | All functionality keyboard accessible | Manual testing |
| **2.1.2 No Keyboard Trap** | Focus can always escape | Automated testing |
| **2.4.3 Focus Order** | Logical tab order | Screen reader testing |
| **2.4.7 Focus Visible** | 2px mint outline on all elements | Visual regression tests |
| **3.3.1 Error Identification** | Clear error messages with field association | Form validation testing |
| **4.1.2 Name, Role, Value** | Proper ARIA labels and roles | aXe automated testing |

**Component-Specific Accessibility**

```html
<!-- Timeline Accessibility -->
<div 
  role="application"
  aria-label="Financial timeline from age 25 to 90"
  aria-describedby="timeline-help"
>
  <div id="timeline-help" class="sr-only">
    Use arrow keys to navigate events. Press Enter to edit, Delete to remove.
  </div>
  <div
    role="slider"
    aria-label="Retirement age"
    aria-valuemin="50"
    aria-valuemax="70"
    aria-valuenow="65"
    aria-valuetext="Retire at age 65"
    tabindex="0"
  >
    <!-- Event cards -->
  </div>
</div>

<!-- Chart Accessibility -->
<figure role="img" aria-labelledby="chart-title" aria-describedby="chart-desc">
  <figcaption id="chart-title">Projected Cash Flow</figcaption>
  <canvas id="projection-chart"></canvas>
  <p id="chart-desc" class="sr-only">
    Chart showing income of $120,000 annually until age 65, 
    expenses of $80,000 annually, resulting in $40,000 annual savings.
    Total net worth reaches $2.5 million by retirement.
  </p>
  <details>
    <summary>View as data table</summary>
    <table>
      <caption>Cash flow projection data</caption>
      <thead>
        <tr>
          <th>Age</th>
          <th>Income</th>
          <th>Expenses</th>
          <th>Savings</th>
          <th>Net Worth</th>
        </tr>
      </thead>
      <tbody>
        <!-- Data rows -->
      </tbody>
    </table>
  </details>
</figure>

<!-- Form Error Handling -->
<div class="form-field">
  <label for="income" id="income-label">
    Annual Income
    <span class="required" aria-label="required">*</span>
  </label>
  <input
    type="number"
    id="income"
    name="income"
    aria-labelledby="income-label"
    aria-describedby="income-help income-error"
    aria-invalid="true"
    aria-required="true"
    value="abc"
  />
  <span id="income-help" class="help-text">
    Enter your gross annual income
  </span>
  <span id="income-error" role="alert" class="error-message">
    <svg class="icon-error" aria-hidden="true">...</svg>
    Please enter a valid number between $0 and $10,000,000
  </span>
</div>
```

**Screen Reader Announcements**

```typescript
// Live region for dynamic updates
const announce = (message: string, priority: 'polite' | 'assertive' = 'polite') => {
  const liveRegion = document.getElementById('live-region') || createLiveRegion();
  liveRegion.setAttribute('aria-live', priority);
  liveRegion.textContent = message;
  
  // Clear after announcement
  setTimeout(() => {
    liveRegion.textContent = '';
  }, 1000);
};

// Usage examples
announce('Calculation complete. You can retire at age 58.');
announce('Error: Invalid income amount', 'assertive');
announce('Scenario saved successfully');
```

**Keyboard Navigation Patterns**

```typescript
// Roving tabindex for event cards
const EventList = () => {
  const [focusedIndex, setFocusedIndex] = useState(0);
  
  const handleKeyDown = (e: KeyboardEvent, index: number) => {
    switch (e.key) {
      case 'ArrowRight':
      case 'ArrowDown':
        e.preventDefault();
        setFocusedIndex((index + 1) % events.length);
        break;
      case 'ArrowLeft':
      case 'ArrowUp':
        e.preventDefault();
        setFocusedIndex((index - 1 + events.length) % events.length);
        break;
      case 'Home':
        e.preventDefault();
        setFocusedIndex(0);
        break;
      case 'End':
        e.preventDefault();
        setFocusedIndex(events.length - 1);
        break;
    }
  };
  
  return (
    <div role="list" aria-label="Timeline events">
      {events.map((event, index) => (
        <div
          key={event.id}
          role="listitem"
          tabIndex={index === focusedIndex ? 0 : -1}
          onKeyDown={(e) => handleKeyDown(e, index)}
          ref={index === focusedIndex ? focusRef : null}
        >
          {/* Event content */}
        </div>
      ))}
    </div>
  );
};
```

**Focus Management**

```typescript
// Modal focus trap
const Modal = ({ isOpen, onClose, children }) => {
  const modalRef = useRef<HTMLDivElement>(null);
  const previousFocus = useRef<HTMLElement | null>(null);
  
  useEffect(() => {
    if (isOpen) {
      previousFocus.current = document.activeElement as HTMLElement;
      modalRef.current?.focus();
    } else {
      previousFocus.current?.focus();
    }
  }, [isOpen]);
  
  return (
    <FocusTrap active={isOpen}>
      <div
        ref={modalRef}
        role="dialog"
        aria-modal="true"
        tabIndex={-1}
        onKeyDown={(e) => {
          if (e.key === 'Escape') onClose();
        }}
      >
        {children}
      </div>
    </FocusTrap>
  );
};
```

### High Contrast & Color Modes

```css
/* High contrast mode adjustments */
@media (prefers-contrast: high) {
  :root {
    --color-mint: #5FF5C3;
    --color-text-secondary: #E0E0E8;
    --color-navy-light: #000000;
    --card-border: 2px solid #FFFFFF;
    --button-border: 2px solid currentColor;
  }
  
  .chart-line {
    stroke-width: 3px;
  }
  
  .focus-visible {
    outline: 3px solid var(--color-mint);
    outline-offset: 2px;
  }
}

/* Forced colors mode (Windows High Contrast) */
@media (forced-colors: active) {
  .button {
    border: 1px solid ButtonText;
  }
  
  .card {
    border: 1px solid CanvasText;
  }
  
  .selected {
    outline: 2px solid Highlight;
  }
}
```

### Content Accessibility

**Reading Level**
- Target: Grade 10 reading level (Flesch-Kincaid)
- Implementation: Content style guide, automated testing
- Glossary: Hover definitions for financial terms

**Plain Language Examples**
- ❌ "Utilize comprehensive portfolio optimization"
- ✅ "Choose the best mix of investments"
- ❌ "Implement dollar-cost averaging methodology"  
- ✅ "Invest the same amount regularly"

**Error Messages**
- ❌ "Invalid input format"
- ✅ "Please enter a number, like 50000"
- ❌ "Authentication failed"
- ✅ "Wrong password. Try again or reset it"

## 8. Performance Goals

### Bundle Size Budget

| Resource | Budget | Strategy |
|----------|--------|----------|
| **Initial JS** | <250KB gzipped | Code splitting, tree shaking |
| **Initial CSS** | <50KB gzipped | PurgeCSS, critical CSS |
| **Fonts** | <100KB | Variable fonts, subset |
| **Images** | <200KB initial | Lazy loading, WebP |
| **Total Initial** | <500KB | Progressive enhancement |

### Core Web Vitals Targets

| Metric | Target | Mobile 3G | Mobile 4G | Desktop |
|--------|--------|-----------|-----------|---------|
| **FCP** | <1s | ✅ Target | ✅ Target | ✅ Target |
| **LCP** | <2.5s | ✅ Target | ✅ Target | ✅ Target |
| **FID** | <100ms | ✅ Target | ✅ Target | ✅ Target |
| **CLS** | <0.1 | ✅ Target | ✅ Target | ✅ Target |
| **TTI** | <3.5s | ✅ Target | ✅ Target | ✅ Target |

### Runtime Performance

| Operation | Target | Measurement | Optimization |
|-----------|--------|-------------|--------------|
| **Timeline drag** | 60fps | DevTools FPS meter | RAF, will-change |
| **Chart update** | <100ms | performance.measure() | Memoization |
| **Calculation** | <200ms | End-to-end | Web Worker |
| **Route change** | <300ms | Navigation Timing | Prefetch |
| **Search/filter** | <50ms | Input to render | Debounce, virtual list |
| **AI (local)** | <3s | First token | Streaming |
| **AI (Mixtral)** | <4s | First token | Edge caching |
| **AI (GPT-4o)** | <8s | First token | Fallback ready |

### Optimization Implementation

```typescript
// Code splitting strategy
const routes = [
  {
    path: '/',
    component: lazy(() => import('./pages/Home'))
  },
  {
    path: '/plan',
    component: lazy(() => 
      import(/* webpackChunkName: "planner" */ './pages/Planner')
    )
  },
  {
    path: '/dashboard',
    component: lazy(() => 
      import(/* webpackChunkName: "dashboard" */ './pages/Dashboard')
    )
  }
];

// Chart optimization
const ProjectionChart = memo(({ data, ...props }) => {
  // Expensive calculations
  const processedData = useMemo(() => 
    processDataForChart(data), [data]
  );
  
  // Skip re-renders
  return <Chart data={processedData} {...props} />;
}, (prevProps, nextProps) => {
  // Custom comparison
  return prevProps.data === nextProps.data &&
         prevProps.width === nextProps.width;
});

// Web Worker for calculations
const calculationWorker = new Worker(
  new URL('./workers/calculation.worker.ts', import.meta.url)
);

const runCalculation = async (scenario: Scenario) => {
  return new Promise((resolve) => {
    calculationWorker.postMessage({ type: 'CALCULATE', scenario });
    calculationWorker.onmessage = (e) => {
      if (e.data.type === 'RESULT') resolve(e.data.result);
    };
  });
};

// Virtual scrolling for lists
const VirtualScenarioList = ({ scenarios }) => {
  const rowVirtualizer = useVirtual({
    size: scenarios.length,
    parentRef,
    estimateSize: useCallback(() => 120, []),
    overscan: 5
  });
  
  return (
    <div ref={parentRef} style={{ height: '600px', overflow: 'auto' }}>
      <div style={{ height: `${rowVirtualizer.totalSize}px` }}>
        {rowVirtualizer.virtualItems.map(virtualRow => (
          <div
            key={virtualRow.index}
            style={{
              position: 'absolute',
              top: 0,
              left: 0,
              width: '100%',
              height: `${virtualRow.size}px`,
              transform: `translateY(${virtualRow.start}px)`
            }}
          >
            <ScenarioCard scenario={scenarios[virtualRow.index]} />
          </div>
        ))}
      </div>
    </div>
  );
};
```

### Progressive Web App

```javascript
// Service Worker registration
if ('serviceWorker' in navigator) {
  window.addEventListener('load', () => {
    navigator.serviceWorker.register('/sw.js').then(
      registration => console.log('SW registered'),
      err => console.log('SW registration failed')
    );
  });
}

// Offline-first caching strategy
self.addEventListener('install', event => {
  event.waitUntil(
    caches.open('planolo-v1').then(cache => {
      return cache.addAll([
        '/',
        '/plan',
        '/static/css/main.css',
        '/static/js/main.js',
        '/models/llama-3-8b.wasm',
        '/offline.html'
      ]);
    })
  );
});

// Network-first for API, cache-first for assets
self.addEventListener('fetch', event => {
  if (event.request.url.includes('/api/')) {
    // Network first, fallback to cache
    event.respondWith(
      fetch(event.request)
        .then(response => {
          const clone = response.clone();
          caches.open('api-cache').then(cache => {
            cache.put(event.request, clone);
          });
          return response;
        })
        .catch(() => caches.match(event.request))
    );
  } else {
    // Cache first, fallback to network
    event.respondWith(
      caches.match(event.request).then(response => {
        return response || fetch(event.request);
      })
    );
  }
});
```

### Web App Manifest

```json
{
  "name": "Planolo - Run the numbers on life",
  "short_name": "Planolo",
  "description": "Interactive financial planning for Singapore and Southeast Asia",
  "start_url": "/",
  "display": "standalone",
  "theme_color": "#0F101A",
  "background_color": "#0F101A",
  "icons": [
    {
      "src": "/icons/icon-192.png",
      "sizes": "192x192",
      "type": "image/png",
      "purpose": "any maskable"
    },
    {
      "src": "/icons/icon-512.png",
      "sizes": "512x512",
      "type": "image/png"
    }
  ],
  "categories": ["finance", "productivity"],
  "screenshots": [
    {
      "src": "/screenshots/planner.png",
      "sizes": "1280x720",
      "type": "image/png"
    }
  ],
  "shortcuts": [
    {
      "name": "New Scenario",
      "url": "/plan?new=true",
      "icons": [{ "src": "/icons/new.png", "sizes": "96x96" }]
    }
  ]
}
```

## 9. Localization Implementation

### Supported Locales

| Locale | Language | Currency | Date Format | Number Format | Status |
|--------|----------|----------|-------------|---------------|--------|
| **en-SG** | English (Singapore) | SGD | DD/MM/YYYY | 1,234.56 | ✅ Launch |
| **zh-CN** | 简体中文 | SGD | YYYY年MM月DD日 | 1,234.56 | 🔄 Month 3 |
| **ms-MY** | Bahasa Malaysia | MYR | DD/MM/YYYY | 1,234.56 | 🔄 Month 6 |
| **id-ID** | Bahasa Indonesia | IDR | DD/MM/YYYY | 1.234,56 | 🔄 Month 6 |

### i18n Architecture

```typescript
// Locale detection and setup
import i18n from 'i18next';
import { initReactI18next } from 'react-i18next';
import LanguageDetector from 'i18next-browser-languagedetector';

i18n
  .use(LanguageDetector)
  .use(initReactI18next)
  .init({
    resources: {
      'en-SG': { translation: enSG },
      'zh-CN': { translation: zhCN },
      'ms-MY': { translation: msMY },
      'id-ID': { translation: idID }
    },
    fallbackLng: 'en-SG',
    interpolation: {
      escapeValue: false,
      format: (value, format, lng) => {
        if (format === 'currency') {
          return new Intl.NumberFormat(lng, {
            style: 'currency',
            currency: getCurrencyForLocale(lng)
          }).format(value);
        }
        if (format === 'date') {
          return new Intl.DateTimeFormat(lng).format(value);
        }
        return value;
      }
    }
  });

// Currency mapping
const currencyMap = {
  'en-SG': 'SGD',
  'zh-CN': 'SGD', // Still SGD for SG Chinese speakers
  'ms-MY': 'MYR',
  'id-ID': 'IDR'
};

// Number formatting hook
const useNumberFormat = () => {
  const { i18n } = useTranslation();
  
  return useCallback((value: number, options?: Intl.NumberFormatOptions) => {
    return new Intl.NumberFormat(i18n.language, options).format(value);
  }, [i18n.language]);
};

// Date formatting hook  
const useDateFormat = () => {
  const { i18n } = useTranslation();
  
  return useCallback((date: Date, options?: Intl.DateTimeFormatOptions) => {
    return new Intl.DateTimeFormat(i18n.language, options).format(date);
  }, [i18n.language]);
};
```

### Translation Structure

```typescript
// en-SG.json
{
  "common": {
    "appName": "Planolo",
    "tagline": "Run the numbers on life",
    "save": "Save",
    "cancel": "Cancel",
    "delete": "Delete",
    "edit": "Edit",
    "close": "Close"
  },
  "planner": {
    "retireAt": "Retire at {{age}}",
    "monthlyIncome": "Monthly Income",
    "projectedWealth": "Projected Wealth",
    "successRate": "{{rate}}% Success Rate",
    "explainButton": "Explain it",
    "stressTest": "Stress Test"
  },
  "errors": {
    "invalidIncome": "Please enter a valid income amount",
    "networkError": "Connection issue. Please try again.",
    "calculationError": "Unable to calculate. Check your inputs."
  },
  "accessibility": {
    "skipToContent": "Skip to main content",
    "openMenu": "Open navigation menu",
    "closeModal": "Close dialog"
  }
}

// zh-CN.json
{
  "common": {
    "appName": "Planolo",
    "tagline": "为人生算一笔账",
    "save": "保存",
    "cancel": "取消",
    "delete": "删除",
    "edit": "编辑",
    "close": "关闭"
  },
  "planner": {
    "retireAt": "{{age}}岁退休",
    "monthlyIncome": "月收入",
    "projectedWealth": "预计财富",
    "successRate": "{{rate}}% 成功率",
    "explainButton": "解释一下",
    "stressTest": "压力测试"
  }
}
```

### Layout Considerations

```css
/* RTL support preparation (future) */
[dir="rtl"] {
  --direction-factor: -1;
}

[dir="ltr"] {
  --direction-factor: 1;
}

/* Flexible spacing that works with text expansion */
.button {
  min-width: 120px; /* Accommodate longer translations */
  padding: var(--button-padding-md);
  white-space: nowrap;
  overflow: hidden;
  text-overflow: ellipsis;
}

/* Chinese-specific adjustments */
:lang(zh) {
  --font-body: 'Inter', 'PingFang SC', 'Microsoft YaHei', sans-serif;
  --line-height-base: 1.7; /* More space for characters */
}

/* Responsive text that handles expansion */
.card-title {
  min-height: 2em; /* Reserve space for wrapping */
  line-height: var(--leading-snug);
}
```

## 10. Analytics Implementation

### Event Tracking Plan

| Event | Properties | When Triggered | Success Target |
|-------|------------|----------------|----------------|
| **scenario_created** | `{goal_type, input_method, time_to_create}` | First save | 70% of users |
| **simulation_run** | `{type, duration_ms, event_count}` | Calculate clicked | 70% conversion |
| **explain_clicked** | `{context, tier, scenario_complexity}` | Explain button | 60% usage |
| **scenario_shared** | `{method, has_message, recipient_count}` | Share completed | 30% share |
| **chat_prompt_sent** | `{word_count, question_type, response_time}` | Message sent | 40% engagement |
| **stress_test_completed** | `{preset, severity, passed}` | Test finished | 50% of Pro |
| **export_downloaded** | `{format, file_size, pages}` | Download done | 20% export |
| **subscription_started** | `{plan, source, discount}` | Payment success | 5% convert |

### Implementation

```typescript
// Analytics service
class Analytics {
  private queue: AnalyticsEvent[] = [];
  private userId: string | null = null;
  
  identify(userId: string, traits?: Record<string, any>) {
    this.userId = userId;
    if (window.gtag) {
      window.gtag('config', GA_MEASUREMENT_ID, {
        user_id: userId,
        user_properties: traits
      });
    }
  }
  
  track(event: string, properties?: Record<string, any>) {
    const enrichedProps = {
      ...properties,
      timestamp: Date.now(),
      session_id: this.getSessionId(),
      user_id: this.userId,
      tier: this.getUserTier(),
      locale: i18n.language,
      device_type: this.getDeviceType(),
      connection_type: this.getConnectionType()
    };
    
    // Send to GA4
    if (window.gtag) {
      window.gtag('event', event, enrichedProps);
    }
    
    // Queue for batch sending
    this.queue.push({ event, properties: enrichedProps });
    this.flushIfNeeded();
  }
  
  // Timing helper
  timing(category: string, variable: string, value: number) {
    this.track('timing_complete', {
      timing_category: category,
      timing_variable: variable,
      timing_value: value
    });
  }
}

// Usage in components
const ScenarioBuilder = () => {
  const analytics = useAnalytics();
  const startTime = useRef(Date.now());
  
  const handleSave = async (scenario: Scenario) => {
    const duration = Date.now() - startTime.current;
    
    await saveScenario(scenario);
    
    analytics.track('scenario_created', {
      goal_type: scenario.goal,
      input_method: scenario.importedData ? 'import' : 'manual',
      time_to_create: duration,
      event_count: scenario.events.length,
      has_monte_carlo: scenario.monteCarloEnabled
    });
  };
  
  return (
    // Component JSX
  );
};
```

### Privacy-Conscious Analytics

```typescript
// Respect user privacy settings
const useAnalytics = () => {
  const { privacyMode } = usePrivacySettings();
  
  const track = useCallback((event: string, properties?: any) => {
    if (privacyMode === 'local') {
      // Log locally only
      console.log('[Analytics]', event, properties);
      return;
    }
    
    // Remove PII before sending
    const sanitized = sanitizeProperties(properties);
    analytics.track(event, sanitized);
  }, [privacyMode]);
  
  return { track };
};

// Sanitize sensitive data
const sanitizeProperties = (props: any) => {
  const sanitized = { ...props };
  
  // Remove exact monetary values
  if (sanitized.income) {
    sanitized.income_range = getIncomeRange(sanitized.income);
    delete sanitized.income;
  }
  
  // Hash email if present
  if (sanitized.email) {
    sanitized.email_hash = hashEmail(sanitized.email);
    delete sanitized.email;
  }
  
  return sanitized;
};
```

## 11. Error Handling & Recovery

### Error States UI

```typescript
// Network error with retry
const NetworkError: React.FC<{ onRetry: () => void }> = ({ onRetry }) => (
  <EmptyState
    variant="error"
    illustration="/lolo-disconnect.svg"
    title="Connection hiccup"
    description="Check your internet and we'll sync when you're back"
    action={{
      label: "Try Again",
      onClick: onRetry
    }}
  />
);

// Calculation error with details
const CalculationError: React.FC<{ error: CalculationError }> = ({ error }) => (
  <Alert variant="error" icon={<AlertCircle />}>
    <AlertTitle>Numbers didn't add up</AlertTitle>
    <AlertDescription>
      We've rolled back your changes and logged the issue.
      {error.code && <code className="error-code">Error: {error.code}</code>}
    </AlertDescription>
    <AlertActions>
      <Button variant="outline" onClick={() => window.location.reload()}>
        Refresh Page
      </Button>
      <Button variant="ghost" onClick={() => copyErrorDetails(error)}>
        Copy Details
      </Button>
    </AlertActions>
  </Alert>
);

// LLM timeout with fallback
const AITimeoutError: React.FC = () => {
  const [showSimple, setShowSimple] = useState(false);
  
  return (
    <div className="ai-timeout">
      <Spinner size="sm" />
      <p>Taking longer than usual...</p>
      <div className="timeout-actions">
        <Button variant="outline" onClick={() => retryAI()}>
          Try Again
        </Button>
        <Button variant="ghost" onClick={() => setShowSimple(true)}>
          Use Simple Mode
        </Button>
      </div>
      {showSimple && <SimpleExplanation />}
    </div>
  );
};
```

### Empty States with Lolo

```typescript
const emptyStates = {
  'no-scenarios': {
    title: "No scenarios yet—let's plan your first move!",
    description: "Start with a goal and see where life could take you.",
    illustration: "/lolo-welcome.svg",
    action: { label: "Create First Scenario", onClick: () => navigate('/plan') }
  },
  'no-data': {
    title: "Add some life events",
    description: "Drag income, expenses, or milestones onto the timeline.",
    illustration: "/lolo-thinking.svg", 
    action: { label: "Add Event", onClick: () => openEventModal() }
  },
  'offline': {
    title: "You're offline",
    description: "But you can still plan! Using local AI mode.",
    illustration: "/lolo-offline.svg",
    action: null,
    showShield: true
  },
  'error': {
    title: "Something went wrong",
    description: "We've logged this issue—please try again or contact support.",
    illustration: "/lolo-error.svg",
    action: { label: "Retry", onClick: () => window.location.reload() }
  }
};
```

### Progressive Degradation

```typescript
// Feature detection and fallbacks
const features = {
  webgl: !!window.WebGLRenderingContext,
  worker: !!window.Worker,
  wasm: !!window.WebAssembly,
  storage: !!window.localStorage,
  offline: !!window.navigator.onLine
};

// Graceful fallbacks
const Chart = features.webgl ? WebGLChart : CanvasChart;
const Calculator = features.worker ? WorkerCalculator : MainThreadCalculator;
const AI = features.wasm ? LocalLlama : RemoteAPI;
```

## 12. Privacy & Security UI

### Privacy Mode Toggle

```typescript
const PrivacySettings = () => {
  const [mode, setMode] = useState<'local' | 'cloud'>('cloud');
  const [showDetails, setShowDetails] = useState(false);
  
  return (
    <Card className="privacy-card">
      <CardHeader>
        <div className="privacy-header">
          <Shield className={mode === 'local' ? 'shield-active' : ''} />
          <h3>Privacy Mode</h3>
        </div>
      </CardHeader>
      <CardContent>
        <RadioGroup value={mode} onValueChange={setMode}>
          <div className="privacy-option">
            <RadioGroupItem value="cloud" id="cloud" />
            <label htmlFor="cloud" className="privacy-label">
              <Cloud className="icon" />
              <div>
                <p className="option-title">Cloud Sync (Encrypted)</p>
                <p className="option-desc">Access anywhere, automatic backups</p>
              </div>
            </label>
          </div>
          <div className="privacy-option">
            <RadioGroupItem value="local" id="local" />
            <label htmlFor="local" className="privacy-label">
              <Shield className="icon" />
              <div>
                <p className="option-title">Local Only</p>
                <p className="option-desc">Everything stays on your device</p>
              </div>
            </label>
          </div>
        </RadioGroup>
        
        <Button
          variant="link"
          onClick={() => setShowDetails(!showDetails)}
          className="details-toggle"
        >
          {showDetails ? 'Hide' : 'Show'} details
          <ChevronDown className={showDetails ? 'rotate-180' : ''} />
        </Button>
        
        {showDetails && (
          <div className="privacy-details">
            <h4>What this means:</h4>
            <ul>
              {mode === 'cloud' ? (
                <>
                  <li>✓ Data encrypted with AES-256</li>
                  <li>✓ Automatic backups every hour</li>
                  <li>✓ Access from any device</li>
                  <li>✓ Share scenarios with others</li>
                </>
              ) : (
                <>
                  <li>✓ No data leaves your device</li>
                  <li>✓ AI runs locally (slower)</li>
                  <li>✓ Manual export required</li>
                  <li>✓ Maximum privacy</li>
                </>
              )}
            </ul>
          </div>
        )}
      </CardContent>
    </Card>
  );
};
```

### Data Management UI

```typescript
const DataManagement = () => {
  const [deleteProgress, setDeleteProgress] = useState(0);
  const [isDeleting, setIsDeleting] = useState(false);
  
  const handleDeleteAll = async () => {
    if (!confirm('Delete all data? This cannot be undone.')) return;
    
    setIsDeleting(true);
    
    // Simulate progress
    for (let i = 0; i <= 100; i += 10) {
      setDeleteProgress(i);
      await sleep(200);
    }
    
    await deleteAllUserData();
    toast.success('All data deleted successfully');
    navigate('/');
  };
  
  return (
    <Card>
      <CardHeader>
        <h3>Your Data</h3>
        <Database className="icon" />
      </CardHeader>
      <CardContent>
        <div className="data-stats">
          <Stat icon={<FileText />} label="Scenarios" value="12" />
          <Stat icon={<HardDrive />} label="Storage" value="2.4 MB" />
          <Stat icon={<Clock />} label="Last Backup" value="2 hours ago" />
        </div>
        
        <div className="data-actions">
          <Button variant="outline" onClick={exportAllData}>
            <Download className="mr-2" />
            Export All Data
          </Button>
          <Button 
            variant="danger" 
            onClick={handleDeleteAll}
            disabled={isDeleting}
          >
            <Trash className="mr-2" />
            Delete Everything
          </Button>
        </div>
        
        {isDeleting && (
          <div className="delete-progress">
            <Progress value={deleteProgress} />
            <p className="progress-label">
              Deleting your data... {deleteProgress}%
            </p>
          </div>
        )}
      </CardContent>
    </Card>
  );
};
```

## 13. Offline & PWA Behavior

### Offline Detection & UI

```typescript
const OfflineIndicator = () => {
  const isOnline = useOnlineStatus();
  const [dismissed, setDismissed] = useState(false);
  
  if (isOnline || dismissed) return null;
  
  return (
    <div className="offline-banner">
      <div className="offline-content">
        <WifiOff className="offline-icon" />
        <span>Offline mode active—using local AI</span>
      </div>
      <Button
        variant="ghost"
        size="sm"
        onClick={() => setDismissed(true)}
        aria-label="Dismiss offline notification"
      >
        <X className="w-4 h-4" />
      </Button>
    </div>
  );
};

// Sync indicator
const SyncStatus = () => {
  const { pendingChanges, lastSync } = useSyncStatus();
  
  return (
    <div className="sync-status">
      {pendingChanges > 0 ? (
        <>
          <RefreshCw className="animate-spin" />
          <span>{pendingChanges} changes pending</span>
        </>
      ) : (
        <>
          <Check className="text-success" />
          <span>Synced {formatRelativeTime(lastSync)}</span>
        </>
      )}
    </div>
  );
};
```

### Conflict Resolution UI

```typescript
const ConflictResolver = ({ conflicts }: { conflicts: Conflict[] }) => {
  const [resolutions, setResolutions] = useState<Record<string, 'local' | 'remote'>>({});
  
  return (
    <Modal title="Sync Conflicts" size="lg">
      <div className="conflict-list">
        {conflicts.map(conflict => (
          <div key={conflict.id} className="conflict-item">
            <h4>{conflict.scenarioName}</h4>
            <div className="conflict-versions">
              <div className="version local">
                <h5>Your Version</h5>
                <p>Modified: {formatDate(conflict.local.modified)}</p>
                <p>{conflict.local.summary}</p>
                <Button
                  variant={resolutions[conflict.id] === 'local' ? 'primary' : 'outline'}
                  onClick={() => setResolution(conflict.id, 'local')}
                >
                  Keep This
                </Button>
              </div>
              <div className="version remote">
                <h5>Cloud Version</h5>
                <p>Modified: {formatDate(conflict.remote.modified)}</p>
                <p>{conflict.remote.summary}</p>
                <Button
                  variant={resolutions[conflict.id] === 'remote' ? 'primary' : 'outline'}
                  onClick={() => setResolution(conflict.id, 'remote')}
                >
                  Keep This
                </Button>
              </div>
            </div>
          </div>
        ))}
      </div>
      <div className="conflict-actions">
        <Button variant="outline" onClick={keepAllLocal}>
          Keep All Local
        </Button>
        <Button variant="outline" onClick={keepAllRemote}>
          Keep All Cloud
        </Button>
        <Button 
          variant="primary" 
          onClick={() => resolveConflicts(resolutions)}
          disabled={Object.keys(resolutions).length < conflicts.length}
        >
          Apply Resolutions
        </Button>
      </div>
    </Modal>
  );
};
```

## 14. Voice Interface (Plus Tier)

### Voice Input Component

```typescript
const VoiceInput = () => {
  const [isListening, setIsListening] = useState(false);
  const [transcript, setTranscript] = useState('');
  const [confidence, setConfidence] = useState(0);
  const recognition = useRef<SpeechRecognition | null>(null);
  
  useEffect(() => {
    if ('SpeechRecognition' in window || 'webkitSpeechRecognition' in window) {
      const SpeechRecognition = window.SpeechRecognition || window.webkitSpeechRecognition;
      recognition.current = new SpeechRecognition();
      recognition.current.continuous = true;
      recognition.current.interimResults = true;
      recognition.current.lang = getVoiceLanguage(i18n.language);
      
      recognition.current.onresult = (event) => {
        const last = event.results.length - 1;
        const result = event.results[last];
        setTranscript(result[0].transcript);
        setConfidence(result[0].confidence);
      };
    }
  }, []);
  
  const toggleListening = () => {
    if (isListening) {
      recognition.current?.stop();
      setIsListening(false);
    } else {
      recognition.current?.start();
      setIsListening(true);
    }
  };
  
  return (
    <div className="voice-input">
      <button
        className={`voice-button ${isListening ? 'listening' : ''}`}
        onClick={toggleListening}
        aria-label={isListening ? 'Stop voice input' : 'Start voice input'}
      >
        {isListening ? (
          <>
            <Mic className="voice-icon animate-pulse" />
            <span className="voice-indicator">
              <span className="voice-wave" />
              <span className="voice-wave" />
              <span className="voice-wave" />
            </span>
          </>
        ) : (
          <Mic className="voice-icon" />
        )}
      </button>
      
      {transcript && (
        <div className="voice-transcript">
          <p className="transcript-text">{transcript}</p>
          <div className="transcript-meta">
            <span className="confidence">
              {Math.round(confidence * 100)}% confident
            </span>
            <Button size="sm" variant="ghost" onClick={clearTranscript}>
              Clear
            </Button>
          </div>
        </div>
      )}
    </div>
  );
};
```

## 15. Stress Test UI

### Stress Test Modal

```typescript
const StressTestModal = ({ scenario, onApply }) => {
  const [preset, setPreset] = useState<'custom' | '2008' | '1997'>('custom');
  const [parameters, setParameters] = useState({
    marketDrop: 0,
    interestRise: 0,
    inflationRise: 0,
    incomeDrop: 0
  });
  
  const presets = {
    '2008': {
      name: '2008 Financial Crisis',
      marketDrop: -40,
      interestRise: 0,
      inflationRise: 2,
      incomeDrop: -10
    },
    '1997': {
      name: '1997 Asian Financial Crisis',
      marketDrop: -30,
      interestRise: 3,
      inflationRise: 4,
      incomeDrop: -20
    }
  };
  
  return (
    <Modal
      title="Stress Test Your Plan"
      size="lg"
      className="stress-test-modal"
    >
      <div className="stress-presets">
        <RadioGroup value={preset} onValueChange={setPreset}>
          <RadioGroupItem value="custom" id="custom">
            <label htmlFor="custom">Custom Scenario</label>
          </RadioGroupItem>
          {Object.entries(presets).map(([key, preset]) => (
            <RadioGroupItem key={key} value={key} id={key}>
              <label htmlFor={key}>
                <span className="preset-name">{preset.name}</span>
                <span className="preset-params">
                  Market {preset.marketDrop}%, Rates +{preset.interestRise}%
                </span>
              </label>
            </RadioGroupItem>
          ))}
        </RadioGroup>
      </div>
      
      <div className="stress-parameters">
        <div className="parameter">
          <label>Market Drop</label>
          <Slider
            value={[parameters.marketDrop]}
            onValueChange={([value]) => updateParam('marketDrop', value)}
            min={-50}
            max={0}
            step={5}
            disabled={preset !== 'custom'}
          />
          <span className="value">{parameters.marketDrop}%</span>
        </div>
        
        <div className="parameter">
          <label>Interest Rate Rise</label>
          <Slider
            value={[parameters.interestRise]}
            onValueChange={([value]) => updateParam('interestRise', value)}
            min={0}
            max={5}
            step={0.5}
            disabled={preset !== 'custom'}
          />
          <span className="value">+{parameters.interestRise}%</span>
        </div>
        
        <div className="parameter">
          <label>Inflation Rise</label>
          <Slider
            value={[parameters.inflationRise]}
            onValueChange={([value]) => updateParam('inflationRise', value)}
            min={0}
            max={10}
            step={0.5}
            disabled={preset !== 'custom'}
          />
          <span className="value">+{parameters.inflationRise}%</span>
        </div>
      </div>
      
      <div className="stress-preview">
        <h4>Impact Preview</h4>
        <StressPreviewChart
          original={scenario}
          stressed={applyStress(scenario, parameters)}
        />
      </div>
      
      <div className="stress-actions">
        <Button variant="outline" onClick={close}>
          Cancel
        </Button>
        <Button variant="primary" onClick={() => onApply(parameters)}>
          Apply Stress Test
        </Button>
      </div>
    </Modal>
  );
};
```

**UI_GUIDE.md and ARCHITECTURE.md updated successfully.**