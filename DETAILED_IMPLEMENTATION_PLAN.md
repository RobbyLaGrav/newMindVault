I'll analyze the current MindVault codebase and create a comprehensive implementation plan for these major updates. Let me start by exploring the project structure and understanding the existing architecture.
null
null
null
null
null
null
null
null
null
null
null
null
Now let me explore the src directory structure more deeply:
null
null
null
null
null
null
Let me examine the package.json and key application files:
null
null
null
null
null
null
null
null
null
null
null
null
Now let me examine the Zustand store to understand the data model:
null
null
null
null
null
null
null
null
null
Let me check a few more pages to understand the structure better:
null
null
null
null
null
null
null
null
Let me check the CSS file to understand the theming system:
null
null
null
null
null
null
Let me check other pages to understand the complete data model:
null
null
null
null
null
null
null
null
null
Now I have a comprehensive understanding of the codebase. Let me create a detailed implementation plan document.
null
null
null
null
null
I'll explore the codebase thoroughly to understand the current architecture and create a comprehensive implementation plan.
null
null
null
null
Let me fix that and explore the structure properly.
null
null
null
null
null
null
null
null
null
null
null
null
null
null
null
null
null
Perfect. Now I have all the information I need to create a comprehensive implementation plan. Let me write it out:

---

# COMPREHENSIVE IMPLEMENTATION PLAN FOR MINDVAULT MAJOR UPDATES

## Executive Summary

MindVault is a Next.js-based personal command center with 8+ content types (notes, ideas, business ideas, tasks, habits, vault notes, goals, inbox). Current implementation uses client-side storage via Zustand with localStorage persistence. The following plan outlines a phased approach to add authentication, backend persistence, calendar functionality, UI improvements, and customization features while maintaining backward compatibility and managing performance at scale.

---

## CURRENT STATE ANALYSIS

### Technology Stack
- **Frontend**: Next.js 16.1.6, React 19.2.3, TypeScript 5, Zustand 5.0.11, Tailwind 4, Framer Motion 12.35
- **Storage**: Zustand + localStorage (browser-only)
- **State Management**: Client-side Zustand store with persist middleware
- **Styling**: CSS variables system with 5 accent colors (green, blue, coral, purple, amber)
- **Architecture**: App Router (Next.js 13+), fully client-side components

### Data Model (8 Content Types)
1. **Note**: title, content, tags, pinned, starred, archived, timestamps
2. **Idea**: title, description, heat (1-5), status (Raw/Developing/Shelved/Launched), category, color, archived
3. **BusinessIdea**: title, problem, audience, revenue, nextSteps, resources, viability, stage
4. **Task**: title, completed, priority (Chill/Important/URGENT), dueDate, recurring, snoozedUntil
5. **Habit**: name, frequency (daily/weekly), completedDates[], color
6. **VaultNote**: title, content (password-protected section)
7. **Goal**: title, description, type (90day/yearly/life), progress, milestones, imageUrl
8. **InboxItem**: type (text/voice/photo), content, processed

### Current UI Structure
- **Navigation**: Fixed left sidebar with 9 sections (Inbox, Brain Dump, Spark, Empire, Fire List, Rituals, Goals, Vault, Stats)
- **Theming**: 5-color accent system via CSS variables and data-attributes
- **Components**: Sidebar, CommandPalette (Cmd+K search), QuickCapture (+ button), Shell wrapper
- **Responsive**: Basic grid layouts, Framer Motion animations

### Existing Patterns
- Zustand store actions for CRUD operations
- Modal dialogs for forms (AnimatePresence + Framer Motion)
- Inline editing with expand/collapse
- Custom CSS variable system for theming
- Card-based layout design

---

## IMPLEMENTATION STRATEGY OVERVIEW

This plan uses a 4-phase approach, prioritizing user authentication and backend architecture first, then building calendar and UI features on top of a solid foundation.

**Phase 1 (MVP)**: Authentication & Backend Infrastructure
**Phase 2**: Calendar System & Dashboard Redesign
**Phase 3**: UI/UX Improvements & Data Migration
**Phase 4**: Advanced Customization & Optimization

---

## PHASE 1: AUTHENTICATION & BACKEND INFRASTRUCTURE (Weeks 1-3)

### 1.1 Backend Setup & Database Design

#### Database Schema (PostgreSQL recommended for relational integrity)

```sql
-- Users table
CREATE TABLE users (
  id UUID PRIMARY KEY,
  email VARCHAR(255) UNIQUE NOT NULL,
  password_hash VARCHAR(255) NOT NULL,
  created_at TIMESTAMP DEFAULT NOW(),
  updated_at TIMESTAMP DEFAULT NOW(),
  email_verified BOOLEAN DEFAULT FALSE,
  email_verified_at TIMESTAMP
);

-- User settings table
CREATE TABLE user_settings (
  id UUID PRIMARY KEY,
  user_id UUID UNIQUE REFERENCES users(id) ON DELETE CASCADE,
  accent_color VARCHAR(20) DEFAULT 'green',
  theme_mode VARCHAR(20) DEFAULT 'dark',
  font_size INT DEFAULT 16,
  dashboard_layout TEXT,
  created_at TIMESTAMP DEFAULT NOW(),
  updated_at TIMESTAMP DEFAULT NOW()
);

-- Data tables with user_id foreign key
CREATE TABLE notes (
  id UUID PRIMARY KEY,
  user_id UUID REFERENCES users(id) ON DELETE CASCADE,
  title VARCHAR(255) NOT NULL,
  content TEXT,
  tags TEXT[], -- array or JSON
  pinned BOOLEAN DEFAULT FALSE,
  starred BOOLEAN DEFAULT FALSE,
  archived BOOLEAN DEFAULT FALSE,
  created_at TIMESTAMP DEFAULT NOW(),
  updated_at TIMESTAMP DEFAULT NOW(),
  INDEX (user_id, archived, pinned)
);

-- Similar structure for: ideas, business_ideas, tasks, habits, vault_notes, goals, inbox_items
-- Each with: id UUID PK, user_id UUID FK, all existing fields, created_at, updated_at
```

#### Key Design Decisions
1. **Database Choice**: PostgreSQL with Prisma ORM for type safety
2. **Auth Library**: NextAuth.js v5 (App Router compatible) or Auth0/Clerk for managed solution
3. **Password Hashing**: bcryptjs with salt rounds 10+
4. **Email Verification**: JWT tokens sent via email, 24-hour expiration
5. **Session Management**: JWT + HTTP-only cookies
6. **Data Isolation**: All queries filtered by user_id at API layer

#### Migration Path for Existing Data
- New users get fresh data
- Existing localStorage data can be migrated on first login via modal prompt
- Option to "import legacy data" → POST to `/api/migrate` endpoint
- Preserves all data including timestamps and relationships

### 1.2 API Routes Implementation

Create `/src/app/api/` directory structure:

```
/api/
  /auth/
    /signup       (POST) - Register new user
    /login        (POST) - Login with email/password
    /verify-email (POST) - Verify email token
    /logout       (POST) - Clear session
    /me           (GET)  - Get current user
    /refresh      (POST) - Refresh JWT token
  /notes/
    /route        (GET, POST)
    /[id]/route   (GET, PUT, DELETE)
  /ideas/
    /route        (GET, POST)
    /[id]/route   (GET, PUT, DELETE)
  /tasks/
    /route        (GET, POST)
    /[id]/route   (GET, PUT, DELETE)
  /goals/
    /route        (GET, POST)
    /[id]/route   (GET, PUT, DELETE)
  /habits/
    /route        (GET, POST)
    /[id]/route   (GET, PUT, DELETE)
  /[business-ideas, vault-notes, inbox]/...
  /sync          (POST) - Batch sync for offline changes
```

#### API Response Standard
```typescript
{
  success: boolean;
  data?: T;
  error?: { code: string; message: string };
  timestamp: ISO string;
}
```

### 1.3 Authentication Flow Implementation

#### New Files to Create

1. **lib/auth.ts** - Authentication utilities
   - Password hashing (bcryptjs)
   - JWT token generation/validation
   - Session verification middleware

2. **lib/db.ts** - Database connection
   - Prisma client initialization
   - Connection pooling config

3. **middleware.ts** - Route protection (app router level)
   - Check for valid JWT in cookies
   - Redirect unauthenticated users to /login
   - Attach user context to request

4. **app/auth/** - New authentication pages
   - `/login` - Login form
   - `/signup` - Registration form with email verification
   - `/verify-email` - Email confirmation page
   - `/forgot-password` - Password reset flow

5. **components/AuthProvider.tsx** - Context wrapper
   - Manages current user state
   - Provides logout functionality
   - Handles session refresh

6. **hooks/useAuth.ts** - Custom hook
   - `useAuth()` → { user, isLoading, logout }
   - Checks `/api/auth/me` on mount

### 1.4 Zustand Store Refactoring

#### Current Structure Issue
- All data stored in client memory
- No server persistence
- Single monolithic store (100+ lines)

#### New Hybrid Approach

**Split into two stores**:

1. **useAuthStore** - Authentication state
   ```typescript
   - user: User | null
   - isAuthenticated: boolean
   - isLoading: boolean
   - login(email, password)
   - logout()
   - setUser(user)
   ```

2. **useAppStore** - Existing data (enhanced)
   ```typescript
   - All existing content types
   - Actions now POST to server
   - Local cache for optimistic updates
   - isSyncing boolean flag
   - pendingChanges queue for offline support
   ```

#### Server-Client Sync Pattern
```typescript
// Before: addNote triggered local state only
// After: 
addNote: async (title, content) => {
  const optimisticId = uuid();
  // 1. Optimistic update
  set(s => ({ notes: [{ id: optimisticId, ... }, ...s.notes] }))
  
  // 2. Post to server
  const response = await fetch('/api/notes', {
    method: 'POST',
    body: JSON.stringify({ title, content })
  })
  
  // 3. Reconcile with server ID
  if (response.ok) {
    set(s => ({
      notes: s.notes.map(n => 
        n.id === optimisticId ? response.data : n
      )
    }))
  }
}
```

### 1.5 Testing Strategy - Phase 1

- **Unit Tests**: Auth utilities, API endpoints (Jest)
- **Integration Tests**: Login → Create Note → Sync (Playwright)
- **Backwards Compatibility**: Test localStorage import flow
- **Security**: SQL injection, CSRF, XSS vectors

### 1.6 Phase 1 Deliverables

- [ ] PostgreSQL database with schema
- [ ] Prisma ORM setup
- [ ] NextAuth.js configuration
- [ ] `/api/auth/*` endpoints (signup, login, verify-email, logout, me)
- [ ] `/api/[resource]/*` endpoints for all 8 content types
- [ ] Authentication pages (/login, /signup, /verify-email)
- [ ] AuthProvider + useAuth hook
- [ ] Zustand store refactoring
- [ ] Server-client sync logic
- [ ] API middleware for user_id injection
- [ ] Email service integration (SendGrid/Mailgun)

---

## PHASE 2: CALENDAR SYSTEM & DASHBOARD REDESIGN (Weeks 4-6)

### 2.1 Calendar Component Architecture

#### New Component Structure
```
/src/components/Calendar/
  /Calendar.tsx (main container, month view)
  /CalendarDay.tsx (individual day cell)
  /CalendarHeader.tsx (month/year navigation)
  /CalendarLegend.tsx (color coding explanation)
  /TasksForDate.tsx (side panel showing tasks for selected date)
  /useCalendarLogic.ts (date calculations, filtering)
```

#### Calendar Logic (useCalendarLogic.ts)

Key functions:
- `getDaysInMonth(year, month)` - Returns array of Date objects
- `filterTasksByDate(tasks, date)` - Get tasks for specific date
- `getColorForPriority(priority)` - Map priority to color
- `groupTasksByDueDate(tasks)` - Index tasks by date for quick lookup
- `getTaskCount(date)` - Show badge with task count per day

#### Data Model Extensions

Add to Task type:
```typescript
interface Task {
  // ... existing fields
  dueDate: string | null; // YYYY-MM-DD format (already exists)
  // New fields:
  dueTime?: string; // HH:MM format for time-based tasks
  isAllDay?: boolean;
  recurring?: 'daily' | 'weekly' | 'monthly' | null; // Enhanced from current
  recurrenceEnd?: string; // When recurring stops
}
```

### 2.2 Calendar Page Implementation

Create `/src/app/calendar/page.tsx`:

```typescript
"use client";

export default function CalendarPage() {
  // State
  const [selectedDate, setSelectedDate] = useState<Date | null>(null);
  const [currentMonth, setCurrentMonth] = useState(new Date());
  
  // Data
  const tasks = useStore(s => s.tasks);
  const goals = useStore(s => s.goals); // Could show goal-related tasks
  
  // Logic
  const tasksByDate = useMemo(() => groupTasksByDueDate(tasks), [tasks]);
  const tasksForSelectedDate = selectedDate 
    ? tasksByDate[selectedDate.toISOString().split('T')[0]] || []
    : [];
  
  return (
    <div className="grid grid-cols-1 lg:grid-cols-3 gap-6">
      <div className="lg:col-span-2">
        <Calendar 
          month={currentMonth}
          tasksByDate={tasksByDate}
          selectedDate={selectedDate}
          onSelectDate={setSelectedDate}
        />
      </div>
      <div>
        <TasksForDate 
          date={selectedDate}
          tasks={tasksForSelectedDate}
        />
      </div>
    </div>
  );
}
```

#### Calendar Visual Design

- **Month Grid**: 7-column layout (Sun-Sat)
- **Day Cells**: 
  - Background color based on day of week
  - Small colored dots for tasks (1 dot per priority level)
  - Hover shows mini preview
  - Click to select and see task panel
- **Color Coding**:
  - Red dot = URGENT tasks
  - Yellow dot = Important tasks
  - Green dot = Chill tasks
  - Multiple dots if multiple tasks
- **Controls**:
  - Previous/Next month arrows
  - "Today" button
  - View toggle (month/week/day - future)

### 2.3 Dashboard Redesign (Home Page Overhaul)

Current inbox/page.tsx shows:
- Due Today (3 tasks)
- Random Ideas (3 ideas)
- Goals (3 goals)

Redesign to comprehensive dashboard:

```typescript
// New sections:

1. **Daily Overview** (top)
   - Time-based greeting
   - Current date/time
   - Weather (optional API integration)
   - Quick stats: Tasks today, Completion %, Streak

2. **Urgent Section** (high visibility)
   - All URGENT tasks due today/overdue
   - Red background, sticky position on scroll

3. **Upcoming** (next 7 days)
   - Timeline view or table
   - Grouped by date
   - Show priority, due time if set

4. **Quick Stats Dashboard** (4-card grid)
   - Tasks Completed This Week: X/Y
   - Habit Streaks: Show top 3 habits with streaks
   - Goals Progress: Avg % and count by type
   - Notes Created: This month count

5. **Today's Focus** (spotlight)
   - Top 3 most important tasks
   - Milestones due today for goals
   - Habits to complete today

6. **This Week's Goals** (compact)
   - Show 90-day goals with progress
   - Click to expand

7. **Inbox** (if items exist)
   - Slightly less prominent than before

8. **Quick Actions** (floating)
   - + New Task, + New Note, + New Goal buttons
   - Or keep in command palette
```

#### Layout Structure
```
Dashboard
├── Header (Time, Date, Stats)
├── Urgent Banner (if any URGENT tasks)
├── Main Grid (2-3 columns on desktop, 1 on mobile)
│   ├── Upcoming (7 days)
│   ├── Quick Stats (2x2 grid)
│   ├── Today's Focus
│   ├── Goals Progress
│   └── Inbox (if unprocessed items)
└── Footer (Last sync time)
```

### 2.4 Calendar Integration with Existing Pages

**Fire List page changes**:
- Add small calendar mini-view on right sidebar
- Click calendar date to filter tasks
- Show calendar icon next to dueDate field

**Goals page changes**:
- Show upcoming goal-related milestones on calendar
- Calendar view option for milestone timeline

**New Navigation Item**:
- Add Calendar (📅) to sidebar navigation between Fire List and Rituals

### 2.5 Testing - Phase 2

- **Component Tests**: Calendar math, date calculations (Jest)
- **Integration Tests**: Click date → see tasks, create task → appears on calendar
- **Accessibility**: Keyboard navigation, ARIA labels for calendar
- **Mobile**: Responsive calendar, touch interactions

### 2.6 Phase 2 Deliverables

- [ ] Calendar component with month view
- [ ] Calendar page with task filtering
- [ ] Calendar legend/color coding system
- [ ] Dashboard redesign (new home page)
- [ ] Upcoming tasks widget
- [ ] Quick stats cards
- [ ] Today's focus section
- [ ] Calendar mini-views in existing pages
- [ ] Task date picker improvements
- [ ] Recurring task support in calendar
- [ ] Mobile-responsive calendar

---

## PHASE 3: UI/UX IMPROVEMENTS & DATA MIGRATION (Weeks 7-9)

### 3.1 Comprehensive UI/UX Overhaul

#### 3.1.1 Clear Labeling & Descriptions

Add to each page:

```typescript
// Before: Just heading
<h1>Fire List</h1>

// After: Heading + description + help text
<div className="mb-6">
  <h1 className="text-3xl font-bold">🔥 Fire List</h1>
  <p className="text-sm mt-1 text-muted">
    Manage your tasks by priority. URGENT items appear at top.
  </p>
  <details className="text-xs mt-2">
    <summary>How to use</summary>
    <p className="mt-2">• Add tasks with priority levels...</p>
  </details>
</div>
```

Pages needing descriptions:
1. Inbox - "Capture raw thoughts, then route them to appropriate sections"
2. Brain Dump - "Personal knowledge base. Notes are searchable via Cmd+K"
3. Spark - "Track ideas in various stages. Promote to Empire when ready"
4. Empire - "Business ideas with structure: problem, audience, revenue model"
5. Fire List - "Time-boxed tasks with three priority levels"
6. Rituals - "Daily/weekly habits with streak tracking"
7. Goals - "90-day, yearly, and life goals with milestone tracking"
8. Vault - "Password-protected private notes"
9. Stats - "Analytics on productivity and content creation"
10. Calendar - "Visual overview of upcoming deadlines and events"

#### 3.1.2 Visual Hierarchy Improvements

**Typography**:
- Standardize heading sizes across pages
- Increase contrast between primary/secondary text
- Better font scaling for readability
- Add text utility classes to globals.css

**Spacing**:
- Consistent padding for cards (standardize to 20px)
- Consistent gaps between grid items (16px)
- Section margins standardized (mb-8 for sections)

**Colors**:
- Text hierarchy: primary → secondary → muted
- Hover states for all interactive elements
- Disabled states (opacity, color change)
- Consistent focus states (outline, ring)

#### 3.1.3 Navigation & Wayfinding Improvements

**Sidebar Enhancements**:
- Current: 9 items in single list
- Improved: Group related sections
  ```
  WORK
  ├── Fire List (🔥)
  ├── Goals (🎯)
  ├── Calendar (📅)
  
  CAPTURE & ORGANIZE
  ├── Inbox (📥)
  ├── Brain Dump (🧠)
  
  IDEAS & VISION
  ├── Spark (⚡)
  ├── Empire (🏛️)
  
  HABITS & WELLNESS
  ├── Rituals (🔁)
  
  ARCHIVE
  ├── Vault (🔒)
  ├── Stats (📊)
  ```

- Add breadcrumb navigation for nested views
- Show current section subtitle in main area header
- Add section icons to page titles for consistency

**Command Palette Enhancements** (Cmd+K):
- Show keyboard shortcuts
- Quick nav to sections ("Go to Fire List")
- Quick actions ("Create Task", "Log Ritual")
- Show recent items
- Search across all content types

#### 3.1.4 Component-Level UX Improvements

**Buttons**:
- Consistent sizing (sm, md, lg)
- Clear primary/secondary/tertiary states
- Loading states with spinner
- Disabled state styling
- Tooltip support for ambiguous buttons

**Forms**:
- Floating labels option
- Clear validation error messages
- Success feedback (toast notifications)
- Field descriptions/hints
- Smart autofocus on modal open

**Cards**:
- Consistent hover effects
- Click targets >= 44px tall (mobile)
- Reduce motion option for animations
- Skeleton loaders during data fetch

**Modals/Dialogs**:
- Consistent sizing (max-w-lg, max-w-2xl presets)
- Close button (X) in top right
- Keyboard shortcuts (Escape to close, Enter to submit)
- Scroll-safe on mobile

### 3.2 Light Mode Implementation

#### CSS Variables Addition
```css
:root[data-theme="light"] {
  --bg-primary: #ffffff;
  --bg-secondary: #f8f8f8;
  --bg-card: #f2f2f2;
  --bg-card-hover: #e8e8e8;
  --border: #e0e0e0;
  --text-primary: #0a0a0a;
  --text-secondary: #666666;
  --text-muted: #999999;
  --glass: rgba(0, 0, 0, 0.02);
}

:root[data-theme="dark"] {
  /* existing dark mode vars */
}
```

#### User Settings Integration
```typescript
interface UserSettings {
  theme: 'light' | 'dark' | 'system'; // system = auto detect
  accentColor: AccentColor;
  fontSize: 'small' | 'normal' | 'large'; // 14px, 16px, 18px
  reducedMotion: boolean;
  compactMode: boolean; // smaller spacing/cards
}
```

#### Theme Switcher Component
- Add to sidebar settings section
- Show sun/moon icons
- Smooth transition between themes
- Persist to localStorage + database
- Respect system preference if set to 'system'

### 3.3 Font Size Customization

**Implementation**:
```css
:root {
  --font-size: 16px;
  --scale: 1; /* scale factor */
}

:root[data-font-size="small"] {
  --scale: 0.875; /* 14px base */
}

:root[data-font-size="large"] {
  --scale: 1.125; /* 18px base */
}

body {
  font-size: calc(var(--font-size) * var(--scale));
}

h1 { font-size: calc(2.5rem * var(--scale)); }
h2 { font-size: calc(2rem * var(--scale)); }
```

**UI Control**: 
- Settings modal with three buttons: A (small) | A (normal) | A (large)
- Live preview as you adjust

### 3.4 Enhanced Color Customization System

Current: 5 preset accent colors
New: **Custom Color Picker**

**Implementation**:
```typescript
interface UserTheme {
  // Existing
  accentColor: AccentColor | 'custom';
  
  // New for custom
  customColors?: {
    accent: string; // hex color
    accentDim?: string;
    accentGlow?: string;
  };
}
```

**UI Component**:
```typescript
// New /settings page with tabs
<Tabs>
  <Tab name="Appearance">
    <ColorSection>
      <p>Accent Color</p>
      {/* Preset buttons */}
      <button>Green</button>
      <button>Blue</button>
      ...
      <button>Custom</button>
      
      {/* If custom selected */}
      {customMode && (
        <ColorPicker 
          value={customAccent}
          onChange={setCustomAccent}
        />
      )}
    </ColorSection>
  </Tab>
  
  <Tab name="Typography">
    {/* Font size controls */}
  </Tab>
  
  <Tab name="Dashboard">
    {/* Layout customization (see 3.5) */}
  </Tab>
  
  <Tab name="Data & Privacy">
    {/* Export, import, delete account */}
  </Tab>
</Tabs>
```

### 3.5 Dashboard Layout Customization

**Concept**: Allow users to customize what widgets appear on dashboard and their order.

**Implementation**:
```typescript
interface DashboardLayout {
  widgets: {
    id: 'quick-stats' | 'upcoming' | 'goals' | 'habits' | 'inbox' | 'focus';
    enabled: boolean;
    order: number;
  }[];
  columns: 1 | 2 | 3;
  cardSize: 'compact' | 'normal' | 'large';
}
```

**Drag-and-drop Edit Mode**:
- Settings → Dashboard tab
- Toggle each widget on/off
- Drag to reorder (if enabled)
- Adjust card sizes
- Preview live

### 3.6 Data Migration & Import/Export

#### Export Functionality
Users can export all data as JSON:

```typescript
// Endpoint: /api/export
GET /api/export?format=json
Response: {
  user: {...},
  notes: [...],
  ideas: [...],
  tasks: [...],
  // all content types
  exportedAt: ISO string,
  version: "1.0"
}
```

#### Import from Old Data
- Modal on first login: "Migrating data from browser?"
- Parse localStorage, validate format
- Show preview of what will import
- POST to `/api/import` (atomic transaction)
- Mark as imported to prevent duplication

#### Backup Scheduling (Future)
- Optional daily/weekly backup to cloud
- Manual backup button in settings
- Email backup option

### 3.7 Testing - Phase 3

- **Visual Regression**: Screenshots of all pages in light/dark mode
- **Accessibility**: WCAG 2.1 AA compliance check
- **Mobile**: All improvements tested on multiple device sizes
- **Color Contrast**: Ensure text passes WCAG standards
- **Migration**: Test import/export with sample data

### 3.8 Phase 3 Deliverables

- [ ] Clear descriptions for all 10 pages
- [ ] Sidebar navigation grouping
- [ ] Breadcrumb navigation
- [ ] Enhanced command palette
- [ ] Light mode support
- [ ] Theme toggle in sidebar
- [ ] Font size customization (small/normal/large)
- [ ] Color picker for custom accent colors
- [ ] Settings page with tabs
- [ ] Dashboard layout customization UI
- [ ] Drag-and-drop widget ordering
- [ ] Export to JSON functionality
- [ ] Import from old localStorage data
- [ ] Improved form validation & error messages
- [ ] Toast notification system
- [ ] Skeleton loaders
- [ ] Accessibility audit & fixes
- [ ] Mobile responsiveness audit

---

## PHASE 4: ADVANCED CUSTOMIZATION & OPTIMIZATION (Weeks 10-12)

### 4.1 Advanced Theme Customization

#### Custom Color Schemes (Not Just Accent)

Expand from single accent to full scheme:

```typescript
interface CustomColorScheme {
  id: string;
  name: string; // "Ocean Blue", "Sunset", etc.
  
  // Color palette
  colors: {
    primary: string;      // accent-like
    secondary: string;    // for secondary elements
    success: string;      // for completed items
    warning: string;      // for important/overdue
    danger: string;       // for urgent/delete
    
    // Background palette
    bgPrimary: string;
    bgSecondary: string;
    bgCard: string;
    border: string;
    
    // Text palette
    textPrimary: string;
    textSecondary: string;
    textMuted: string;
  };
  
  // Style options
  borderRadius: 'sharp' | 'soft' | 'smooth'; // 0px, 8px, 12px
  spacing: 'compact' | 'normal' | 'spacious';
  fontFamily: 'system' | 'serif' | 'mono';
}
```

**Community Themes**: Publish a GitHub repo for user-created themes. Schema validation + voting system.

### 4.2 Widget/Card Sizing Options

```typescript
interface CardSize {
  size: 'compact' | 'normal' | 'large';
  padding: string;      // 12px, 16px, 20px
  fontSize: string;     // scales text
  imageHeight: string;  // for goal images
}
```

Individual cards can have per-widget size settings different from global.

### 4.3 Performance Optimizations

#### Code Splitting
- Route-based: Each page lazy-loads its components
- Component-based: Calendar component code-split (loaded only on /calendar)

#### Database Query Optimization
- Add database indexes on user_id, created_at, archived
- Implement pagination: `/api/notes?page=1&limit=50`
- Add cursor-based pagination for infinite scroll

#### Caching Strategy
```
Browser Cache (next/cache):
- Static assets: 1 year
- API responses: 5 minutes for list views
- User data: Cache-on-demand + SWR (stale-while-revalidate)

Server-Side Cache:
- Redis cache for expensive queries
- Cache user settings object (rarely changes)
- Invalidate on create/update
```

#### Bundle Size Reduction
- Tree-shake unused Framer Motion animations
- Replace heavy date library with lightweight alternative
- Analyze with `next/bundle-analyzer`

### 4.4 Offline Support & Sync

#### Service Worker Implementation
- Cache critical app shell on first load
- Sync pending changes when online
- Show "offline mode" indicator
- Queue operations in IndexedDB

```typescript
// Offline data queue
interface PendingChange {
  id: string;
  action: 'create' | 'update' | 'delete';
  resource: 'notes' | 'tasks' | 'goals' | ...;
  data: object;
  timestamp: number;
}

// IndexedDB store: pending_changes
// When online, POST to /api/sync with batched changes
```

### 4.5 Advanced Analytics (Stats Page Expansion)

**Current Stats Page**: Likely empty or minimal

**Enhancements**:
- **Productivity Metrics**:
  - Tasks completed per week/month
  - Average completion rate
  - Time to completion (dueDate vs completedAt)
  - Priority breakdown (% URGENT completed)

- **Content Creation**:
  - Notes created over time (line chart)
  - Ideas per category (pie chart)
  - Most used tags (tag cloud)

- **Goals Progress**:
  - Goal completion rates by type
  - Milestone velocity
  - Goal age vs progress correlation

- **Habits**:
  - Calendar heatmap of habit streaks
  - Best habits (highest completion %)
  - Habit impact on task completion

- **Time Analysis**:
  - Most productive time of day
  - Day of week breakdown
  - Seasonal trends

**Libraries**: recharts or chart.js for visualizations

### 4.6 Collaboration Features (Future Roadmap)

Setup groundwork for:
- Shared workspaces
- Invite other users to collaborate
- Shared goal tracking
- Comments on notes/ideas
- Version history

Database schema additions:
```sql
CREATE TABLE workspaces (
  id UUID PRIMARY KEY,
  name VARCHAR(255),
  owner_id UUID REFERENCES users(id),
  created_at TIMESTAMP
);

CREATE TABLE workspace_members (
  id UUID PRIMARY KEY,
  workspace_id UUID REFERENCES workspaces(id),
  user_id UUID REFERENCES users(id),
  role VARCHAR(50), -- 'owner', 'editor', 'viewer'
  joined_at TIMESTAMP
);

-- Add workspace_id to all data tables
```

### 4.7 Mobile App Considerations

**PWA Features** (already partial in Next.js):
- Add Web App Manifest improvements
- Install prompt on mobile
- Splash screen
- Bottom tab navigation for mobile
- Touch-optimized buttons and spacing

**Native App Path** (Future):
- React Native version
- Or Capacitor/Tauri desktop apps

### 4.8 Testing - Phase 4

- **Performance**: Lighthouse audits (95+ scores)
- **Offline**: Simulate offline, verify queued changes sync on reconnect
- **Load Testing**: Test with 10,000+ items per user
- **Security**: Penetration testing on authentication system
- **Analytics**: Verify data collection isn't invasive

### 4.9 Phase 4 Deliverables

- [ ] Custom color scheme builder
- [ ] Community themes repository setup
- [ ] Card sizing customization per widget
- [ ] Database indexes and query optimization
- [ ] API pagination implementation
- [ ] Redis caching layer
- [ ] Service Worker for offline support
- [ ] IndexedDB for pending changes queue
- [ ] Offline mode indicator
- [ ] Stats page redesign with charts
- [ ] Habit heatmap visualization
- [ ] Export analytics as PDF
- [ ] Database schema for collaboration (not UI yet)
- [ ] Performance optimizations
- [ ] Bundle size audit and reduction
- [ ] PWA improvements
- [ ] Mobile-first responsive audit

---

## ARCHITECTURE DECISIONS & RATIONALE

### Why 4 Phases?

1. **Phase 1 (Auth/Backend)** is foundational. Nothing else scales without it.
2. **Phase 2 (Calendar/Dashboard)** is high-impact, visible feature that users want immediately.
3. **Phase 3 (UI/UX)** polish and customization keeps users engaged.
4. **Phase 4 (Advanced)** is nice-to-have optimization and future growth.

### Backend Technology Choices

| Component | Choice | Rationale |
|-----------|--------|-----------|
| Database | PostgreSQL | Relational, ACID, proven at scale, array types for tags |
| ORM | Prisma | Type-safe, excellent with Next.js, auto-generates types |
| Auth | NextAuth.js v5 | Built for App Router, handles OAuth providers easily |
| Email | SendGrid/Mailgun | Reliable, good deliverability, pay-as-you-go |
| Caching | Redis | Simple key-value, fast, TTL support |
| Storage | S3 (images) | For goal images, future file attachments |

### Why Hybrid Client/Server?

- **Server**: Authentication, data persistence, authorization, complex queries
- **Client**: Optimistic updates, offline queue, snappy UX, animations
- **Sync**: Debounced POST to server, websockets for real-time (future)

### Data Model Extensions

All existing fields preserved. Added:
- `user_id` foreign key (linking all data to users)
- Enhanced `Task.recurring` with end date
- `Task.dueTime` for time-specific tasks
- `UserSettings` table for customization
- `workspace_id` for future collaboration

### Security Considerations

1. **Authentication**:
   - HTTP-only cookies for JWT storage
   - CSRF tokens for state-changing operations
   - Rate limiting on auth endpoints (5 attempts/15 mins)

2. **Authorization**:
   - Middleware verifies user_id matches request
   - API doesn't expose other users' data
   - Vault notes encrypted at rest (future)

3. **Data Validation**:
   - Zod schemas for all API inputs
   - Type checking at compile time (TypeScript)
   - HTML escaping on output

4. **GDPR Compliance**:
   - Export endpoint for data portability
   - Delete account with cascade deletion
   - Clear privacy policy
   - Audit logs for admin

---

## FILE STRUCTURE CHANGES

### Current Structure
```
/src
  /app
    /page.tsx (inbox)
    /braindump/page.tsx
    /spark/page.tsx
    /empire/page.tsx
    /firelist/page.tsx
    /rituals/page.tsx
    /goals/page.tsx
    /vault/page.tsx
    /stats/page.tsx
    /layout.tsx
  /components
    /Sidebar.tsx
    /Shell.tsx
    /CommandPalette.tsx
    /QuickCapture.tsx
  /store
    /useStore.ts
```

### New Structure (Post-Phase 4)
```
/src
  /app
    /page.tsx (dashboard - redesigned)
    /(auth)/
      /login/page.tsx
      /signup/page.tsx
      /verify-email/page.tsx
    /(app)/
      layout.tsx (protected layout)
      /braindump/page.tsx
      /spark/page.tsx
      /empire/page.tsx
      /firelist/page.tsx
      /rituals/page.tsx
      /goals/page.tsx
      /calendar/page.tsx (new)
      /vault/page.tsx
      /stats/page.tsx
      /settings/
        page.tsx (new - theme, notifications, export)
    /api/
      /auth/
        /signup/route.ts
        /login/route.ts
        /logout/route.ts
        /verify-email/route.ts
        /me/route.ts
      /notes/
        route.ts
        /[id]/route.ts
      /ideas/route.ts & /[id]/route.ts
      /tasks/route.ts & /[id]/route.ts
      /goals/route.ts & /[id]/route.ts
      /habits/route.ts & /[id]/route.ts
      /vault-notes/route.ts & /[id]/route.ts
      /business-ideas/route.ts & /[id]/route.ts
      /inbox/route.ts & /[id]/route.ts
      /user-settings/route.ts
      /export/route.ts
      /import/route.ts
      /sync/route.ts (batch sync)
      
  /components
    /(auth)/
      /LoginForm.tsx
      /SignupForm.tsx
      /VerifyEmailForm.tsx
    /(app)/
      /Sidebar.tsx
      /Shell.tsx
      /CommandPalette.tsx
      /QuickCapture.tsx
      /Settings/
        /SettingsModal.tsx
        /ThemePanel.tsx
        /LayoutPanel.tsx
        /DataPanel.tsx
      /Dashboard/
        /DashboardStats.tsx
        /QuickActions.tsx
        /UpcomingTasks.tsx
        /TodaysFocus.tsx
      /Calendar/
        /Calendar.tsx
        /CalendarDay.tsx
        /CalendarHeader.tsx
        /TasksForDate.tsx
        /useCalendarLogic.ts
        
  /store
    /useAuthStore.ts (new)
    /useAppStore.ts (refactored from useStore.ts)
    /useSettingsStore.ts (new)
    
  /lib
    /auth.ts (password hashing, JWT)
    /db.ts (Prisma client)
    /api-utils.ts (response formatting, error handling)
    /validators.ts (Zod schemas)
    /colors.ts (color utilities, hex conversion)
    
  /hooks
    /useAuth.ts
    /useApi.ts (fetch wrapper)
    /useLocalStorage.ts (refactored)
    /useTheme.ts (theme switching)
    /useOffline.ts (offline detection)
    
  /utils
    /calendar.ts (date calculations)
    /time.ts (time formatting)
    /storage.ts (localStorage utils)
    
  /middleware.ts (route protection)
  /app/globals.css (enhanced with light mode)
  /app/layout.tsx (root layout, AuthProvider)
```

---

## MIGRATION & ROLLOUT STRATEGY

### Phase 1 Rollout
1. Deploy backend infrastructure (database, API endpoints)
2. Deploy authentication pages (but not required yet)
3. Existing users continue with client-only mode
4. Gradually migrate users on next login

### Backward Compatibility
- localStorage data remains accessible
- Old Zustand state coexists with new API
- Migration modal on first login (optional)
- Ability to continue offline indefinitely

### Testing Before Production
- Staging environment mirrors production
- Run Playwright tests for all critical flows
- Load test with 100+ concurrent users
- Security audit before launch
- Gradual rollout: 10% of users, then 50%, then 100%

### Monitoring & Rollback
- Sentry for error tracking
- Datadog for performance monitoring
- Rollback plan: revert to client-only mode if critical issues
- 24/7 incident response for first 2 weeks

---

## ESTIMATED TIMELINE & RESOURCE NEEDS

### Development Timeline
- **Phase 1**: 3 weeks (1 backend dev, 1 full-stack dev)
- **Phase 2**: 3 weeks (1 frontend dev)
- **Phase 3**: 3 weeks (2 frontend devs, 1 designer)
- **Phase 4**: 3 weeks (1 full-stack dev)
- **Buffer**: 2 weeks (bug fixes, edge cases)
- **Total**: 14-16 weeks (3.5 months)

### Team Composition
- 1 Backend Developer (SQL, API design)
- 2 Full-Stack Developers (React + Node)
- 1 Frontend/UI Developer (styling, animations)
- 1 QA Engineer (testing, edge cases)
- 1 Designer (UI mockups, accessibility review) - part-time

### Infrastructure Costs (Monthly)
- PostgreSQL Database: $50-200 (managed service like Railway, Heroku, or AWS RDS)
- Redis: $10-20 (if needed)
- S3 Storage: $1-10 (for images)
- Email Service: $10-30 (SendGrid)
- Hosting (Vercel): $20-100 (depending on traffic)
- **Total**: ~$100-360/month

---

## RISK MITIGATION

| Risk | Likelihood | Impact | Mitigation |
|------|-----------|--------|-----------|
| Data loss during migration | Low | Critical | Backup database daily, test restore process |
| Performance degradation | Medium | High | Load testing before launch, optimization in Phase 4 |
| Auth bypass vulnerability | Low | Critical | Security audit, rate limiting, CORS properly configured |
| User confusion with changes | High | Medium | In-app tutorial, onboarding flow, clear documentation |
| Concurrent edit conflicts | Low | Medium | Last-write-wins or operational transformation (future) |
| Mobile experience suffers | Medium | Medium | Responsive testing throughout, mobile-first approach |

---

## SUCCESS METRICS

### Phase 1 (Auth/Backend)
- [ ] 100% API uptime
- [ ] Sub-200ms API response times (p95)
- [ ] Zero authentication vulnerabilities in security audit
- [ ] 100% user data successfully migrated from localStorage

### Phase 2 (Calendar)
- [ ] Calendar loads in <1s
- [ ] 95% of users interact with calendar on first week
- [ ] Task click → date assignment works for 100% of tasks
- [ ] Mobile calendar responsive on all device sizes

### Phase 3 (UI/UX)
- [ ] Lighthouse score >95 (all metrics)
- [ ] WCAG 2.1 AA compliance on all pages
- [ ] Light mode adoption by 40% of users within 1 month
- [ ] Settings page visited by 70% of users
- [ ] NPS score improves by 10+ points

### Phase 4 (Advanced)
- [ ] P95 response time <300ms with 10,000 items per user
- [ ] Offline mode used by 15% of users
- [ ] Custom themes created by 5% of users
- [ ] 99.9% uptime SLA maintained

---

## FUTURE ENHANCEMENTS (Post-Phase 4 Roadmap)

1. **Collaboration** - Shared workspaces, real-time sync
2. **AI Integration** - Task suggestions, smart summaries, voice transcription
3. **Mobile Apps** - Native iOS/Android via Capacitor
4. **Integrations** - Slack, Discord, Notion, Zapier webhooks
5. **Time Tracking** - Pomodoro timers, time spent per task
6. **Notifications** - Push notifications for due dates, reminders
7. **Templates** - Task templates, recurring goal templates
8. **Advanced Search** - Full-text search, filters, saved searches
9. **Version History** - Track changes to notes, undo/redo
10. **Blockchain** - Proof of completion, achievement badges (NFT future)

---

## CRITICAL FILES FOR IMPLEMENTATION

### Critical Files for Implementation

**1. /src/store/useAuthStore.ts** - New authentication state management
   - Reason: Core to all authorization and user context

**2. /src/middleware.ts** - Route protection and user injection
   - Reason: Security layer that guards all protected routes and injects user context into requests

**3. /src/app/api/auth/\[slug\]/route.ts** - Auth API endpoints (signup, login, verify, logout)
   - Reason: Foundation for all authentication flows; depends on useAuthStore

**4. /src/app/api/\[resource\]/route.ts** - Generic CRUD API endpoints (notes, tasks, goals, etc.)
   - Reason: All data operations depend on these endpoints; core of backend persistence

**5. /src/app/calendar/page.tsx** - Calendar page component
   - Reason: Major new feature and dashboard overhaul depends on calendar infrastructure

**6. /src/components/Calendar/useCalendarLogic.ts** - Calendar date calculations and filtering logic
   - Reason: Calendar component depends on this; contains complex date math that affects accuracy

**7. /src/app/globals.css** - Global styles with CSS variables
   - Reason: Theme system foundation; all customization features (light mode, colors, fonts) depend on CSS variables here

**8. /src/components/Settings/SettingsModal.tsx** - Settings UI for customization
   - Reason: Controls theme, font size, layout customization; user-facing customization hub

**9. /src/lib/db.ts** - Database connection and Prisma setup
   - Reason: All API routes depend on database access; must be configured correctly for data persistence

**10. /src/store/useAppStore.ts** - Refactored main app store (refactored from existing useStore.ts)
   - Reason: Bridges client state and server persistence; all page components depend on this for data access

---

This comprehensive plan provides a clear roadmap for transforming MindVault from a client-only app into a full-featured, multi-user platform with authentication, persistent storage, calendar functionality, and advanced customization options. The phased approach allows for incremental delivery of value while managing technical debt and ensuring quality at each stage.
null
null
null
null
null
null
null
null
null
null
null
null
null
null
null
null
null
null
null
null
null
null
null
null
null
null
null
null
null
null
null
null
null
null
null
null
null
null
null
null
null
null
null
null
null
null
null
null
null
null
null
null
null
null
null
null
Perfect! I have a comprehensive understanding of the codebase. Let me create a detailed implementation plan.

---

# Comprehensive MindVault Implementation Plan

## Executive Summary

MindVault is a sophisticated personal productivity app with 8+ content types built on Next.js 16, React 19, TypeScript, Zustand, Tailwind, and Framer Motion. The app currently uses localStorage for persistence. The implementation plan addresses all 5 major feature areas while maintaining backward compatibility and performance.

---

## Phase Breakdown & Architecture

### PHASE 0: Foundation (Weeks 1-2)

This is the prerequisite phase to enable all subsequent features.

#### 0.1 Backend Setup & Database Schema

**Objectives:** Establish secure multi-user infrastructure

**Database Design (PostgreSQL recommended for production, SQLite for dev):**

```
users
├── id (UUID, PK)
├── email (unique)
├── passwordHash (bcrypt)
├── createdAt
├── lastLoginAt
└── isEmailVerified

user_sessions
├── id (UUID, PK)
├── userId (FK)
├── token (unique, hashed)
├── expiresAt
└── createdAt

user_settings
├── id (UUID, PK)
├── userId (FK, unique)
├── accentColor (enum)
├── theme (light/dark)
├── fontSize (small/normal/large)
├── customColors (JSON)
├── dashboardLayout (JSON)
└── updatedAt

content_items (polymorphic table for all content types)
├── id (UUID, PK)
├── userId (FK)
├── type (enum: note, idea, businessIdea, task, habit, vaultNote, goal, inboxItem)
├── data (JSONB - stores type-specific fields)
├── createdAt
├── updatedAt
└── indexes on (userId, type, createdAt)

sync_log
├── id (UUID, PK)
├── userId (FK)
├── itemId (FK)
├── action (create/update/delete)
├── timestamp
└── clientVersion (for conflict resolution)
```

**Key Architectural Decision:** Use JSONB polymorphic table with type discriminator rather than separate tables. This enables:
- Single indexed query for "all user content"
- Simpler sync logic
- Easier cross-content search
- Flexible schema evolution

**Tasks:**
- Set up Prisma ORM with schema as above
- Create database migrations for versioning
- Implement connection pooling for prod scalability
- Add database seed script for testing

#### 0.2 Authentication API Routes

**New API Routes Structure:**
```
/api/auth/
├── signup.ts (POST)
├── login.ts (POST)
├── logout.ts (POST)
├── verify-email.ts (POST)
├── refresh-token.ts (POST)
└── reset-password.ts (POST)

/api/user/
├── profile.ts (GET, PATCH)
├── settings.ts (GET, PATCH)
└── data-export.ts (GET)

/api/sync/
├── push.ts (POST - upload local changes)
└── pull.ts (GET - fetch server state)
```

**Authentication Method:** JWT tokens with refresh token rotation
- Access token: 15 min expiry
- Refresh token: 7 days expiry, stored in HttpOnly cookie
- Implement token rotation on each refresh
- CSRF protection for state-changing operations

**Email Service Integration:**
- Use Resend, SendGrid, or similar
- Email templates for: verification, password reset, welcome
- Queue system (Bull, Agenda) for reliable delivery

**Tasks:**
- Implement sign up endpoint with email verification flow
- Implement login with secure password hashing (bcrypt)
- Create JWT middleware for route protection
- Set up email verification email template
- Implement password reset flow

#### 0.3 State Management Architecture for Authentication

**Extend Zustand store with auth slice:**

```typescript
interface AuthState {
  user: User | null;
  isAuthenticated: boolean;
  isLoading: boolean;
  token: string | null;
  
  login: (email, password) => Promise<void>;
  signup: (email, password) => Promise<void>;
  logout: () => void;
  refreshToken: () => Promise<void>;
  checkAuth: () => Promise<void>;
}
```

**Tasks:**
- Create auth store with login/signup/logout actions
- Implement auto-login on app boot (check JWT validity)
- Add loading/error states for UX feedback
- Create useAuth hook for easier component access
- Implement token persistence with auto-refresh

#### 0.4 Data Migration Infrastructure

**Strategy for existing localStorage users:**

```typescript
// Migration on first login/signup
async function migrateLocalStorageData(userId: string) {
  const localData = localStorage.getItem("mindvault-storage");
  if (!localData) return;
  
  const parsed = JSON.parse(localData);
  const migrated = transformLegacyData(parsed);
  
  // Batch upload to server
  await api.sync.push({
    items: migrated,
    clientVersion: "legacy"
  });
  
  // Clear local storage only after successful sync
  localStorage.removeItem("mindvault-storage-migrated");
}
```

**Tasks:**
- Create transformation function from old Zustand format to new polymorphic schema
- Implement one-time migration on first authenticated session
- Add migration status tracking (prevent double-migration)
- Create rollback procedure for failed migrations

#### 0.5 Middleware & Protections

**Create authentication middleware:**
- API route protection with JWT verification
- User ID injection into request context
- Rate limiting (prevent brute force attempts)
- CORS configuration
- Input validation (zod schemas for all API inputs)

**Tasks:**
- Implement auth middleware
- Add rate limiting (express-rate-limit)
- Set up request validation middleware
- Configure CORS properly
- Add request logging

---

### PHASE 1: MVP User Experience (Weeks 3-4)

#### 1.1 Auth UI Components

**New Components:**
- `/src/components/Auth/LoginForm.tsx` - Email/password login
- `/src/components/Auth/SignupForm.tsx` - Registration with validation
- `/src/components/Auth/EmailVerification.tsx` - Verification flow
- `/src/components/Auth/PasswordReset.tsx` - Password recovery

**Login/Signup Pages:**
- `/src/app/login/page.tsx`
- `/src/app/signup/page.tsx`
- `/src/app/auth/verify-email/page.tsx`
- `/src/app/auth/reset-password/page.tsx`

**Requirements:**
- Field validation (email format, password strength)
- Loading states during submission
- Error messaging
- Password visibility toggle
- Remember email checkbox
- Redirect to home after successful auth

**Tasks:**
- Create login page with form validation
- Create signup page with password strength indicator
- Create email verification flow
- Create password reset flow
- Add form submission error handling

#### 1.2 Dashboard Layout Overhaul

**Redesign `/src/app/page.tsx` (Home):**

Replace the current 3-card inbox layout with:

```
┌─────────────────────────────────────────────┐
│  DASHBOARD                 [⚙️ Settings]    │
├─────────────────────────────────────────────┤
│  Daily Summary                              │
│  ┌──────────────────┬──────────────────┐   │
│  │ Quick Stats      │ Upcoming Tasks   │   │
│  │ • X tasks due    │ • Task 1 - Today │   │
│  │ • X ideas        │ • Task 2 - Tomorrow
│  │ • X goals        │ • Goal progress  │   │
│  └──────────────────┴──────────────────┘   │
├─────────────────────────────────────────────┤
│  This Week                                  │
│  ┌──────────────────────────────────────┐  │
│  │ Calendar view (7 days)               │  │
│  │ Mon 10  Tue 11  Wed 12 Thu 13 Fri 14│  │
│  │  3t     2t      1i     4t    1g    │  │
│  └──────────────────────────────────────┘  │
├─────────────────────────────────────────────┤
│  Focus Areas                                │
│  ┌──────────┬──────────┬──────────┐        │
│  │ Habits   │ Ideas    │ Goals    │        │
│  │ 5 total  │ 12 active│ 3 90day  │        │
│  │ 4 today  │ 2 launchd│ 45% avg  │        │
│  └──────────┴──────────┴──────────┘        │
└─────────────────────────────────────────────┘
```

**Home Page Structure:**

```typescript
// src/app/page.tsx structure
export default function Home() {
  return (
    <div className="space-y-8">
      <header className="flex justify-between items-center">
        <h1 className="text-3xl font-bold">Dashboard</h1>
        <SettingsButton />
      </header>
      
      <QuickStatsSection />
      <UpcomingTasksSection />
      <WeeklyCalendarSection />
      <FocusAreasSection />
      <RecentActivitySection />
    </div>
  );
}
```

**New Components:**
- `/src/components/Dashboard/QuickStats.tsx` - Summary cards
- `/src/components/Dashboard/UpcomingTasks.tsx` - Task preview
- `/src/components/Dashboard/WeeklyCalendar.tsx` - 7-day overview
- `/src/components/Dashboard/FocusAreas.tsx` - Category summaries
- `/src/components/Dashboard/RecentActivity.tsx` - Activity log

**Tasks:**
- Create dashboard layout component
- Implement quick stats widget
- Create upcoming tasks widget
- Build weekly calendar mini-view
- Add focus areas summary cards

#### 1.3 User Settings & Profile

**New Page `/src/app/settings/page.tsx`:**

```
Account
├── Profile Info (name, email, avatar)
├── Email Change
├── Password Change
└── Deactivate Account

Preferences
├── Notifications
├── Language/Locale
└── Data Settings

Data Management
├── Export All Data (JSON/CSV)
├── Import Data
└── Delete Account (with confirmation)
```

**Components:**
- `/src/components/Settings/AccountSection.tsx`
- `/src/components/Settings/PreferencesSection.tsx`
- `/src/components/Settings/DataManagementSection.tsx`
- `/src/components/Settings/DangerZone.tsx`

**Tasks:**
- Create settings page structure
- Implement profile edit form
- Add password change flow
- Create data export feature
- Implement account deletion

#### 1.4 Sync Engine - Initial Implementation

**Sync Strategy: Optimistic updates with server reconciliation**

```typescript
// src/lib/sync.ts
interface SyncMessage {
  clientId: string;
  version: number;
  items: {
    id: string;
    type: ContentType;
    action: 'create' | 'update' | 'delete';
    data: any;
    timestamp: number;
  }[];
}

export class SyncManager {
  async push(messages: SyncMessage) {
    // Optimistic update
    updateLocalState(messages);
    
    try {
      // Send to server
      const result = await api.sync.push(messages);
      
      // Handle conflicts if any
      if (result.conflicts.length > 0) {
        resolveConflicts(result.conflicts);
      }
    } catch (error) {
      // Rollback on failure
      revertLocalState(messages);
    }
  }
  
  async pull() {
    const lastSync = getLastSyncTimestamp();
    const updates = await api.sync.pull({ since: lastSync });
    mergeServerUpdates(updates);
  }
}
```

**Conflict Resolution:**
- Last-write-wins for most content
- Merge strategies for specific types (habits: union dates, goals: max progress)
- User notification of conflicts

**Tasks:**
- Implement sync manager class
- Create API endpoints for push/pull
- Add offline queue for failed syncs
- Implement conflict detection
- Create conflict resolution strategies

---

### PHASE 2: Calendar System (Weeks 5-6)

#### 2.1 Interactive Calendar Component

**New Component: `/src/components/Calendar/MonthCalendar.tsx`**

Features:
- Month view with task counts per day
- Click day to see detailed task list
- Color-coded by priority (red=urgent, yellow=important, green=done)
- Task badges with count indicators
- Navigation prev/next month
- Quick task completion from calendar

```typescript
interface CalendarDay {
  date: string;
  tasks: Task[];
  ideas: Idea[];
  habits: Habit[];
  goalProgress: number;
}

<div className="grid grid-cols-7 gap-1">
  {daysInMonth.map(day => (
    <CalendarDayCell
      key={day}
      date={day}
      items={itemsByDate[day]}
      onDayClick={showDayDetail}
    />
  ))}
</div>
```

**New Page: `/src/app/calendar/page.tsx`**

Layout:
```
Month selector: [< Feb 2025 >]
┌────────────────────────────────────┐
│ Sun Mon Tue Wed Thu Fri Sat        │
│  1   2   3   4   5   6   7        │
│  •1  •0  •3  •2  •5  •1  •0      │ (dots = task count)
│     •◆      •◆◆                   │ (◆ = idea)
│  ... (rest of month)               │
└────────────────────────────────────┘
┌────────────────────────────────────┐
│ Today: Feb 13, 2025                │ (Detail panel)
│ Tasks (3)                          │
│ ☐ Task 1 (🔴 URGENT)              │
│ ☑ Task 2 (✓)                      │
│ ☐ Task 3 (🟡 Important)           │
│                                    │
│ Ideas (1)                          │
│ ⚡ New feature idea               │
└────────────────────────────────────┘
```

**Tasks:**
- Create calendar month view component
- Implement date selection logic
- Add task/idea/goal counts per day
- Create day detail panel
- Add month navigation

#### 2.2 Task-Calendar Integration

**Extend Task model with calendar features:**

```typescript
interface Task {
  // existing
  dueDate: string | null;
  
  // new
  repeatPattern?: {
    type: 'daily' | 'weekly' | 'biweekly' | 'monthly';
    endDate?: string;
    daysOfWeek?: number[]; // 0=sun, 1=mon, etc
  };
  snoozedUntil?: string;
  timeOfDay?: string; // "morning", "afternoon", "evening"
}
```

**Modifications to `/src/app/firelist/page.tsx`:**
- Link to calendar on date selection
- Show calendar preview in task item
- Inline calendar popup on due date click

**Tasks:**
- Add repeat pattern UI to task form
- Create recurring task expansion logic
- Show future instances of recurring tasks
- Add snooze functionality

#### 2.3 Calendar Visualization Enhancements

**Heat map view for habit tracking:**

```typescript
// src/components/Calendar/HabitHeatmap.tsx
// Shows 52-week grid, similar to GitHub contributions
// Colors: empty (dark), done 1-5 times (gradient)
```

**Weekly task distribution:**

```typescript
// src/components/Calendar/WeeklyDistribution.tsx
// Bar chart: which days are busiest
// Helps with planning and load balancing
```

**Tasks:**
- Create habit heatmap visualization
- Build weekly distribution chart
- Add task/idea calendar events
- Implement event creation from calendar

---

### PHASE 3: Advanced UI/UX & Customization (Weeks 7-9)

#### 3.1 Complete Theme System

**Replace 5-color system with unlimited customization:**

```typescript
// src/types/theme.ts
interface Theme {
  id: string;
  name: string;
  mode: 'dark' | 'light' | 'auto';
  
  colors: {
    primary: string;     // main accent
    secondary: string;   // supporting
    success: string;     // completed/positive
    warning: string;     // warnings
    error: string;       // errors
    info: string;        // information
  };
  
  surfaces: {
    background: string;
    backgroundSecondary: string;
    card: string;
    cardHover: string;
    border: string;
  };
  
  text: {
    primary: string;
    secondary: string;
    muted: string;
  };
  
  typography: {
    fontFamily: string;
    fontSize: 'small' | 'normal' | 'large' | 'xl';
  };
  
  spacing: {
    scale: 0.8 | 1 | 1.2 | 1.5; // multiplier
  };
}
```

**Theme System Implementation:**

```typescript
// src/hooks/useTheme.ts
export function useTheme() {
  const [theme, setTheme] = useLocalStorage<Theme>('mindvault-theme');
  
  useEffect(() => {
    applyThemeToDOM(theme);
  }, [theme]);
  
  return { theme, setTheme };
}
```

**New Components:**
- `/src/components/Theme/ThemeEditor.tsx` - Visual theme builder
- `/src/components/Theme/ColorPicker.tsx` - Advanced color selection
- `/src/components/Theme/PresetThemes.tsx` - Pre-built themes
- `/src/components/Theme/ThemePreview.tsx` - Live preview

**Default Themes:**
1. Default Dark (current)
2. High Contrast Dark
3. Solarized Dark
4. Dracula
5. Nord
6. Catppuccin
7. Light Mode
8. High Contrast Light

**Tasks:**
- Design complete theme schema
- Implement theme provider context
- Create theme editor UI
- Build theme persistence layer
- Add preset themes library
- Implement light mode support with auto-detection

#### 3.2 Light Mode Implementation

**CSS Structure:**
```css
:root[data-theme="light"] {
  --bg-primary: #ffffff;
  --bg-secondary: #f9f9f9;
  --bg-card: #f0f0f0;
  --border: #e0e0e0;
  --text-primary: #1a1a1a;
  --text-secondary: #666666;
  --text-muted: #999999;
}

:root[data-theme="auto"] {
  @media (prefers-color-scheme: light) {
    /* light mode */
  }
  @media (prefers-color-scheme: dark) {
    /* dark mode */
  }
}
```

**Tasks:**
- Create light mode color palette
- Update all components for light mode compatibility
- Test contrast ratios (WCAG AA minimum)
- Add system theme detection
- Implement theme preference sync to server

#### 3.3 Dashboard Customization

**Dashboard Layout Builder:**

```typescript
// src/types/dashboard.ts
interface DashboardLayout {
  id: string;
  name: string;
  widgets: DashboardWidget[];
  columns: 1 | 2 | 3 | 4;
}

interface DashboardWidget {
  id: string;
  type: 'quickstats' | 'calendar' | 'tasks' | 'ideas' | 'habits' | 'goals' | 'notes';
  size: 'small' | 'medium' | 'large';
  position: number;
  config?: Record<string, any>;
}
```

**Layout Editor Components:**
- `/src/components/Dashboard/LayoutEditor.tsx` - Drag-drop builder
- `/src/components/Dashboard/WidgetLibrary.tsx` - Available widgets
- `/src/components/Dashboard/WidgetContainer.tsx` - Resizable widget

**Tasks:**
- Create dashboard layout data model
- Implement drag-drop layout builder
- Build widget library with preview
- Add layout persistence to DB
- Create preset layouts (personal, executive, developer)

#### 3.4 Typography & Spacing Controls

**Font Size System:**

```typescript
// In theme config
typography: {
  fontFamily: 'system-ui | serif | monospace | custom-url';
  baseSize: 14 | 16 | 18 | 20; // px
  lineHeight: 1.4 | 1.5 | 1.6;
  fontWeight: 'normal' | 'semibold' | 'bold';
}
```

**Spacing Scale:**
```css
/* Based on theme.spacing.scale multiplier */
$scale: 1; /* or 0.8, 1.2, 1.5 */
$base: 4px;
$xs: $base * $scale;      /* 4px */
$sm: $base * 2 * $scale;  /* 8px */
$md: $base * 3 * $scale;  /* 12px */
$lg: $base * 4 * $scale;  /* 16px */
```

**Tasks:**
- Create typography settings in theme
- Update all components for responsive scaling
- Add font import system
- Implement spacing multiplier
- Add accessibility validation (line height, text size)

#### 3.5 Mobile UI Optimization

**Responsive Navigation:**

Current: Fixed 220px sidebar
New:
- Desktop: Collapsible sidebar (220px / 60px)
- Tablet: Bottom navigation bar
- Mobile: Hidden drawer accessible via hamburger

**Components:**
- `/src/components/Navigation/MobileNav.tsx` - Bottom nav for mobile
- `/src/components/Navigation/DrawerNav.tsx` - Slide-out drawer
- `/src/components/Layout/AdaptiveLayout.tsx` - Responsive wrapper

**Mobile-First Updates:**
- Stack layouts vertically on small screens
- Adjust card sizes for smaller viewports
- Optimize form inputs for touch
- Add gesture support (swipe between sections)

**Tasks:**
- Create adaptive navigation component
- Optimize all pages for mobile (< 768px)
- Test on iOS and Android devices
- Implement touch-friendly interactions
- Add viewport meta tags optimization

---

### PHASE 4: Data Persistence & Performance (Weeks 10-11)

#### 4.1 Advanced Sync Implementation

**Implement periodic background sync:**

```typescript
// src/lib/backgroundSync.ts
export function initBackgroundSync() {
  // Sync every 30 seconds when app is in foreground
  const syncInterval = setInterval(async () => {
    if (document.hidden) return; // Don't sync when tab is hidden
    
    try {
      await syncManager.push();
      await syncManager.pull();
    } catch (error) {
      console.error('Sync failed:', error);
      // Queue will retry on next sync
    }
  }, 30000);
  
  // Sync on visibility change
  document.addEventListener('visibilitychange', () => {
    if (!document.hidden) {
      syncManager.pull(); // Catch up when returning to tab
    }
  });
}
```

**Implement conflict resolution UI:**

```typescript
// src/components/Sync/ConflictResolver.tsx
// Shows conflicts and offers merge options
// Last-write-wins button or manual merge
```

**Service Worker for offline support:**

```typescript
// public/sw.js
// Cache API responses
// Background sync API for queued updates
// Notification API for sync status
```

**Tasks:**
- Implement periodic sync service
- Add sync status indicator
- Create conflict resolution UI
- Implement service worker for offline mode
- Add retry logic with exponential backoff

#### 4.2 Data Optimization

**Lazy Loading:**
- Load content types on-demand
- Paginate lists (20 items per page)
- Implement virtual scrolling for large lists

```typescript
// src/components/Virtualized/VirtualList.tsx
// Render only visible items, huge performance boost
```

**Index Strategy:**
Database indexes on high-query fields:
```sql
CREATE INDEX idx_user_items ON content_items(userId, type, createdAt DESC);
CREATE INDEX idx_due_date ON content_items((data->>'dueDate')) WHERE type='task';
CREATE INDEX idx_search ON content_items USING GIN(
  to_tsvector('english', COALESCE(data->>'title', ''))
);
```

**Query Optimization:**
- Use GraphQL or field selection to reduce payload
- Implement query result caching
- Add database query monitoring

**Tasks:**
- Implement virtual list scrolling
- Add pagination to all list views
- Create database indexes
- Implement query result caching
- Add performance monitoring

#### 4.3 Storage Management

**Implement cleanup strategies:**

```typescript
// src/lib/storage.ts
export class StorageManager {
  // Archive old completed tasks (30+ days)
  async archiveOldData() {
    const thirtyDaysAgo = subDays(today(), 30);
    await db.contentItems.updateMany(
      {
        userId,
        type: 'task',
        'data.completed': true,
        updatedAt: { $lt: thirtyDaysAgo }
      },
      { $set: { archived: true } }
    );
  }
  
  // Delete truly old archived data (1+ year)
  async cleanupArchivedData() {
    const oneYearAgo = subYears(today(), 1);
    await db.contentItems.deleteMany({
      userId,
      archived: true,
      updatedAt: { $lt: oneYearAgo }
    });
  }
}
```

**Tasks:**
- Create data archival system
- Implement storage quota per user
- Add cleanup job scheduler
- Create data export before cleanup
- Implement data retention policies

#### 4.4 Search & Indexing

**Full-text search implementation:**

```typescript
// Database: Use PostgreSQL full-text search
SELECT * FROM content_items 
WHERE userId = $1 
AND to_tsvector('english', COALESCE(data->>'title', '') || ' ' || COALESCE(data->>'content', ''))
@@ plainto_tsquery('english', $2)
ORDER BY ts_rank(...) DESC;
```

**Global Search Enhancement:**

Update CommandPalette to use full-text search:

```typescript
// src/components/CommandPalette.tsx (updated)
const results = await api.search.full({
  query,
  limit: 10,
  types: ['all']
});
```

**Tasks:**
- Implement full-text search API
- Add search ranking/relevance
- Create search filters (type, date range, priority)
- Update command palette with new search
- Add search analytics

---

### PHASE 5: Polish & Advanced Features (Weeks 12-13)

#### 5.1 Notifications & Alerts

**Notification Types:**
- Task due soon (2 hours before)
- Daily digest email
- Goal milestone reached
- Habit streak milestone
- New collaboration request
- Account security alerts

**Implementation:**
```typescript
// src/lib/notifications.ts
interface NotificationRule {
  userId: string;
  type: 'task-due' | 'daily-digest' | 'milestone' | 'security';
  channel: 'push' | 'email' | 'in-app';
  enabled: boolean;
  config: Record<string, any>;
}

export async function triggerNotifications() {
  const rules = await db.notificationRules.find({ enabled: true });
  for (const rule of rules) {
    // Check condition and send
  }
}
```

**Components:**
- `/src/components/Notifications/NotificationCenter.tsx` - Bell icon with list
- `/src/components/Notifications/NotificationSettings.tsx` - Preferences UI
- `/src/components/Notifications/InAppNotification.tsx` - Toast component

**Tasks:**
- Create notification schema and database
- Implement in-app notification system
- Set up email notifications
- Create push notification system
- Build notification preferences UI

#### 5.2 Analytics & Insights

**New Page: `/src/app/analytics/page.tsx`**

Features:
- Productivity trends (tasks completed over time)
- Most active times/days
- Content creation patterns
- Goal progress over time
- Habit streak analytics

**Charts:**
- Line chart: Tasks completed per week
- Bar chart: Ideas by category
- Pie chart: Time distribution
- Heatmap: Most productive days/times

**Tasks:**
- Create analytics dashboard
- Implement data aggregation queries
- Build chart components (using recharts)
- Add date range filtering
- Create trend analysis

#### 5.3 Collaboration Features (Optional MVP+)

**Share & Collaboration:**

```typescript
interface SharedContent {
  id: string;
  contentId: string;
  ownerId: string;
  sharedWith: string[]; // email addresses
  permissions: 'view' | 'comment' | 'edit';
  sharedAt: string;
  expiresAt?: string;
}
```

**Features:**
- Share individual items with permission levels
- Shared collections/projects
- Comments on shared items
- Public links with expiry

**Tasks:**
- Create sharing database schema
- Implement permission checking middleware
- Build share UI component
- Add shared item notifications
- Create shared dashboard page

#### 5.4 Integrations

**Third-party integrations:**

```typescript
interface Integration {
  id: string;
  userId: string;
  type: 'calendar' | 'email' | 'slack' | 'notion' | 'github';
  credentials: encrypted;
  enabled: boolean;
  lastSyncedAt?: string;
}
```

**Priority Integrations:**
1. **Calendar Sync** (Google Calendar, Apple Calendar)
   - Export tasks as events
   - Import calendar events as tasks

2. **Email Integration**
   - Forward emails to inbox
   - Create tasks from email subjects

3. **Slack Integration**
   - Daily digest via Slack
   - Quick task capture

**Tasks:**
- Create integration schema
- Implement OAuth flows
- Build integration management UI
- Create sync jobs for each integration
- Add error handling and retry logic

#### 5.5 Advanced Reporting

**Report Generator:**

```typescript
// src/lib/reports.ts
export async function generateMonthlyReport(userId: string) {
  const stats = {
    tasksCompleted: 42,
    tasksCreated: 58,
    completionRate: '72%',
    averageResponseTime: '2.1 days',
    topProductiveDay: 'Tuesday',
    topIdeas: [...],
    topGoals: [...],
  };
  
  return generatePDF(stats); // or email
}
```

**Report Types:**
- Weekly summary
- Monthly review
- Annual summary
- Goal progress report
- Productivity metrics

**Tasks:**
- Create report generator service
- Build PDF export functionality
- Implement email delivery
- Create report scheduling
- Add custom report builder

---

## File Structure Changes

### New Directories:
```
src/
├── app/
│   ├── login/
│   ├── signup/
│   ├── auth/
│   │   ├── verify-email/
│   │   └── reset-password/
│   ├── settings/
│   ├── calendar/
│   ├── analytics/
│   └── api/
│       ├── auth/
│       ├── user/
│       ├── sync/
│       ├── content/
│       └── search/
│
├── components/
│   ├── Auth/
│   │   ├── LoginForm.tsx
│   │   ├── SignupForm.tsx
│   │   └── ProtectedRoute.tsx
│   ├── Dashboard/
│   │   ├── QuickStats.tsx
│   │   ├── UpcomingTasks.tsx
│   │   ├── WeeklyCalendar.tsx
│   │   ├── FocusAreas.tsx
│   │   ├── LayoutEditor.tsx
│   │   └── WidgetLibrary.tsx
│   ├── Calendar/
│   │   ├── MonthCalendar.tsx
│   │   ├── DayDetail.tsx
│   │   ├── HabitHeatmap.tsx
│   │   └── WeeklyDistribution.tsx
│   ├── Theme/
│   │   ├── ThemeEditor.tsx
│   │   ├── ColorPicker.tsx
│   │   └── PresetThemes.tsx
│   ├── Settings/
│   │   ├── AccountSection.tsx
│   │   ├── PreferencesSection.tsx
│   │   ├── DataManagementSection.tsx
│   │   └── ThemeSettings.tsx
│   ├── Navigation/
│   │   ├── MobileNav.tsx
│   │   ├── DrawerNav.tsx
│   │   └── AdaptiveLayout.tsx
│   ├── Sync/
│   │   ├── SyncStatus.tsx
│   │   ├── ConflictResolver.tsx
│   │   └── OfflineIndicator.tsx
│   ├── Notifications/
│   │   ├── NotificationCenter.tsx
│   │   ├── NotificationBell.tsx
│   │   └── NotificationSettings.tsx
│   └── Virtualized/
│       └── VirtualList.tsx
│
├── lib/
│   ├── api/
│   │   ├── client.ts
│   │   ├── auth.ts
│   │   └── sync.ts
│   ├── db/
│   │   ├── client.ts
│   │   └── schema.prisma
│   ├── auth/
│   │   ├── jwt.ts
│   │   ├── password.ts
│   │   └── middleware.ts
│   ├── sync/
│   │   ├── manager.ts
│   │   ├── queue.ts
│   │   └── conflictResolver.ts
│   ├── notifications/
│   │   ├── manager.ts
│   │   └── email.ts
│   ├── storage.ts
│   ├── theme.ts
│   └── migration.ts
│
├── hooks/
│   ├── useAuth.ts
│   ├── useTheme.ts
│   ├── useSync.ts
│   ├── useLocalStorage.ts
│   └── useBreakpoint.ts
│
├── types/
│   ├── auth.ts
│   ├── content.ts
│   ├── sync.ts
│   ├── theme.ts
│   └── notification.ts
│
└── store/
    ├── useStore.ts (refactored)
    ├── useAuthStore.ts (new)
    └── useThemeStore.ts (new)
```

---

## Database Schema (Prisma)

```prisma
// prisma/schema.prisma

model User {
  id        String   @id @default(cuid())
  email     String   @unique
  name      String?
  password  String
  
  createdAt DateTime @default(now())
  updatedAt DateTime @updatedAt
  lastLoginAt DateTime?
  emailVerified DateTime?
  
  // Relations
  sessions  Session[]
  settings  UserSettings?
  contents  ContentItem[]
  notifications NotificationRule[]
  integrations Integration[]
  
  @@index([email])
}

model Session {
  id        String   @id @default(cuid())
  userId    String
  user      User     @relation(fields: [userId], references: [id], onDelete: Cascade)
  token     String   @unique
  expiresAt DateTime
  createdAt DateTime @default(now())
  
  @@index([userId])
  @@index([token])
}

model UserSettings {
  id        String   @id @default(cuid())
  userId    String   @unique
  user      User     @relation(fields: [userId], references: [id], onDelete: Cascade)
  
  // Theme
  accentColor String @default("green")
  themeMode String @default("dark")
  customTheme Json?
  
  // Layout
  dashboardLayout Json?
  sidebarCollapsed Boolean @default(false)
  
  // Typography
  fontSize String @default("normal")
  fontFamily String @default("system-ui")
  
  // Preferences
  notifications Json? // notification rules
  language String @default("en")
  
  createdAt DateTime @default(now())
  updatedAt DateTime @updatedAt
  
  @@index([userId])
}

model ContentItem {
  id        String   @id @default(cuid())
  userId    String
  user      User     @relation(fields: [userId], references: [id], onDelete: Cascade)
  
  type      String   // 'note', 'idea', 'task', etc
  data      Json     // polymorphic data storage
  
  archived  Boolean  @default(false)
  
  createdAt DateTime @default(now())
  updatedAt DateTime @updatedAt
  
  @@index([userId])
  @@index([userId, type])
  @@index([userId, createdAt])
  @@index([userId, archived])
}

model NotificationRule {
  id        String   @id @default(cuid())
  userId    String
  user      User     @relation(fields: [userId], references: [id], onDelete: Cascade)
  
  type      String   // 'task-due', 'daily-digest', etc
  channel   String   // 'push', 'email', 'in-app'
  enabled   Boolean  @default(true)
  config    Json?
  
  createdAt DateTime @default(now())
  updatedAt DateTime @updatedAt
  
  @@index([userId])
}

model Integration {
  id        String   @id @default(cuid())
  userId    String
  user      User     @relation(fields: [userId], references: [id], onDelete: Cascade)
  
  type      String   // 'calendar', 'email', 'slack', etc
  credentials String  // encrypted
  enabled   Boolean  @default(true)
  
  lastSyncedAt DateTime?
  
  createdAt DateTime @default(now())
  updatedAt DateTime @updatedAt
  
  @@index([userId, type])
}

model SyncLog {
  id        String   @id @default(cuid())
  userId    String
  itemId    String
  
  action    String   // 'create', 'update', 'delete'
  timestamp DateTime @default(now())
  clientVersion String?
  
  @@index([userId])
  @@index([userId, timestamp])
}
```

---

## Database Schema Considerations

### Backward Compatibility Migration
- Keep localStorage data structure for 3 months
- Provide export/import for users who want manual control
- Auto-migrate on first login with confirmation
- Allow rollback within 30 days

### Multi-tenant Performance
- User ID in every query WHERE clause
- Row-level security at database level (PostgreSQL)
- Connection pooling for concurrent requests
- Caching layer (Redis) for frequently accessed settings

### Data Isolation
- Never join across users
- Implement RLS (Row Level Security) on all tables
- Audit log all data access
- Encrypt sensitive fields (email in password reset, API keys)

---

## Testing Strategy

### Unit Tests
- Auth service (login, signup, token refresh)
- Store reducers/actions
- Utility functions (date formatting, conflict resolution)
- Sync manager (queue, retry logic)

### Integration Tests
- API endpoints with database
- Authentication flow end-to-end
- Sync push/pull with conflict scenarios
- Migration from localStorage

### E2E Tests (Playwright/Cypress)
- Complete user signup flow
- Authentication and protected routes
- Creating/editing/deleting content
- Sync between tabs
- Offline functionality
- Theme switching

### Performance Tests
- Load time with 1000+ items
- Sync performance with large payloads
- Calendar rendering with events
- Search latency
- Mobile performance on 3G

### Accessibility Tests
- WCAG 2.1 Level AA compliance
- Screen reader testing
- Keyboard navigation
- Color contrast validation
- Form validation messaging

---

## Timeline & Milestones

| Phase | Duration | Key Deliverables |
|-------|----------|------------------|
| Phase 0 | 2 weeks | Database, APIs, Auth system |
| Phase 1 | 2 weeks | Login UI, Dashboard overhaul, Basic sync |
| Phase 2 | 2 weeks | Calendar system, Date integration |
| Phase 3 | 3 weeks | Theme customization, Light mode, Mobile UI |
| Phase 4 | 2 weeks | Advanced sync, Optimizations, Search |
| Phase 5 | 2 weeks | Notifications, Analytics, Polish |
| **Total** | **13 weeks** | **Full MVP with all major features** |

---

## Rollout Strategy

### Staged Rollout
1. **Beta (Week 13-14)**: Invite 50-100 power users
   - Gather feedback on new features
   - Monitor performance and sync reliability
   - Fix critical issues

2. **Early Access (Week 15-16)**: 1,000 users
   - Monitor real-world usage patterns
   - Fine-tune performance
   - Document any edge cases

3. **General Availability (Week 17+)**: Public release
   - Monitor uptake and feedback
   - Support existing users during migration
   - Keep localStorage sync available for 3 months

### Fallback Mechanisms
- Kill switch for sync (revert to localStorage mode)
- Database rollback scripts for failed migrations
- Support for reverting theme to defaults
- Data recovery from backups for accidental deletes

---

## Dependencies to Add

```json
{
  "dependencies": {
    "next-auth": "^5.x",
    "@prisma/client": "^5.x",
    "bcrypt": "^5.x",
    "jsonwebtoken": "^9.x",
    "zod": "^3.x",
    "recharts": "^2.x",
    "react-big-calendar": "^1.x",
    "react-window": "^1.8.x",
    "zustand": "^5.x (already)",
    "framer-motion": "^12.x (already)",
    "tailwindcss": "^4.x (already)",
    "resend": "^3.x",
    "bull": "^4.x"
  },
  "devDependencies": {
    "@playwright/test": "^1.x",
    "vitest": "^1.x",
    "@testing-library/react": "^14.x"
  }
}
```

---

## Critical Success Factors

1. **Data Integrity**: Robust sync with conflict resolution
2. **Performance**: Sub-100ms API responses, <2s page loads
3. **User Trust**: Secure auth, transparent data handling
4. **Backward Compatibility**: Seamless migration for existing users
5. **Mobile Experience**: Works smoothly on iOS/Android
6. **Accessibility**: WCAG AA compliant, screen reader friendly

---

## Risk Mitigation

| Risk | Impact | Mitigation |
|------|--------|-----------|
| Sync conflicts | Data loss | Comprehensive conflict resolution + user UI |
| Migration failures | Lost data | Backup before migration, rollback capability |
| Performance degradation | User churn | Load testing, caching, optimization |
| Security vulnerabilities | Data breach | Regular security audits, OWASP compliance |
| Mobile responsive issues | Poor mobile UX | Mobile-first approach, device testing |
| Database scalability | System slowdown | Proper indexing, query optimization, horizontal scaling |

---

### Critical Files for Implementation

**Phase 0 (Backend):**
- `/src/app/api/auth/` - Core auth endpoints
- `/src/lib/db/schema.prisma` - Database schema definition
- `/src/lib/auth/jwt.ts` - JWT token management
- `/src/lib/sync/manager.ts` - Core sync logic

**Phase 1 (User Experience):**
- `/src/components/Auth/LoginForm.tsx` - Authentication entry point
- `/src/app/page.tsx` - Redesigned dashboard home
- `/src/store/useAuthStore.ts` - Authentication state management
- `/src/lib/sync/queue.ts` - Offline queue management

**Phase 2 (Calendar):**
- `/src/components/Calendar/MonthCalendar.tsx` - Core calendar component
- `/src/app/calendar/page.tsx` - Calendar page
- `/src/lib/api/sync.ts` - Sync API client
- `/src/types/content.ts` - Type definitions for all content

**Phase 3 (UI/UX):**
- `/src/lib/theme.ts` - Theme system implementation
- `/src/components/Theme/ThemeEditor.tsx` - Theme customization UI
- `/src/app/globals.css` - Updated for light mode and theme variables
- `/src/components/Navigation/AdaptiveLayout.tsx` - Responsive layout

**Phase 4 (Performance):**
- `/src/lib/storage.ts` - Data archival and cleanup
- `/src/components/Virtualized/VirtualList.tsx` - Virtual scrolling
- `/src/lib/api/client.ts` - API client with caching
- `/src/types/sync.ts` - Sync protocol definitions

**Phase 5 (Polish):**
- `/src/lib/notifications/manager.ts` - Notification system
- `/src/app/analytics/page.tsx` - Analytics dashboard
- `/src/lib/notifications/email.ts` - Email service
- `/src/components/Notifications/NotificationCenter.tsx` - Notification UI
