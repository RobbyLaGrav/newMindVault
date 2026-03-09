# MindVault Major Updates - Implementation Plan

## Overview
Complete redesign and expansion of MindVault from a single-user, localStorage-based app to a multi-user, cloud-backed productivity platform with enhanced UI/UX, calendar integration, and customizable themes.

---

## Phase 1: Foundation (Authentication & Backend Setup)

### 1.1 Database & Backend Infrastructure
**Objective:** Move from localStorage to cloud-backed persistence

**Tasks:**
- [ ] Select backend solution (options: Supabase, Firebase, or custom Node.js/PostgreSQL)
- [ ] Set up authentication system
  - Email/password signup and login
  - Email verification flow
  - Password reset functionality
  - Session management
- [ ] Design database schema for:
  - Users (id, email, password_hash, preferences, created_at, updated_at)
  - User profiles (display_name, avatar, timezone, preferences)
  - Notes, Ideas, Business Ideas, Tasks, Habits, Goals, Vault items (all with user_id foreign key)
  - User settings (theme, accent color, ui_preferences)
- [ ] Create API routes for CRUD operations
- [ ] Implement data migration tool (localStorage → Cloud for existing users)
- [ ] Set up authentication middleware

**New Files:**
- `src/lib/db/` - Database utilities and queries
- `src/lib/auth/` - Authentication helpers
- `src/app/api/auth/` - Auth endpoints (signup, login, verify-email, etc.)
- `src/app/api/[resource]/` - Resource endpoints for all data types
- `src/types/auth.ts` - Auth type definitions
- `src/types/database.ts` - Database schema types

**Modified Files:**
- `src/store/useStore.ts` - Add API integration, remove localStorage-only logic
- `src/app/layout.tsx` - Add auth provider/session context
- `package.json` - Add backend dependencies

---

### 1.2 Authentication UI
**Objective:** Create signup, login, and email verification flows

**Tasks:**
- [ ] Create `/auth` page with routing
  - `/auth/signup` - Email signup form
  - `/auth/login` - Email login form
  - `/auth/verify-email` - Email verification confirmation
  - `/auth/forgot-password` - Password reset flow
  - `/auth/reset-password` - Reset password form
- [ ] Design authentication forms with validation
- [ ] Implement email verification flow
- [ ] Add account settings page (`/account` or `/settings`)
  - View/edit profile
  - Change password
  - Logout
  - Delete account option
- [ ] Add session/auth context provider

**New Components:**
- `src/components/auth/SignupForm.tsx`
- `src/components/auth/LoginForm.tsx`
- `src/components/auth/EmailVerification.tsx`
- `src/components/auth/PasswordReset.tsx`
- `src/components/account/AccountSettings.tsx`
- `src/components/account/ProfileEditor.tsx`

**Modified Components:**
- `src/components/Shell.tsx` - Add user profile menu, logout
- `src/components/Sidebar.tsx` - Update with account link

---

## Phase 2: Calendar System & Home Page Redesign

### 2.1 Calendar System
**Objective:** Interactive calendar showing tasks with dates

**Tasks:**
- [ ] Create calendar component using library (options: react-big-calendar, react-calendar, or custom)
- [ ] Create `/calendar` page
- [ ] Implement calendar features:
  - Monthly/weekly view toggle
  - Click date to view tasks for that day
  - Color-coded tasks by priority (URGENT=red, Important=orange, Chill=green)
  - Visual indicators for tasks with due dates
  - Highlight today's date
  - Navigate between months
  - Show task count on dates with tasks
- [ ] Create date-based task filtering logic
- [ ] Add task quick-add modal from calendar date
- [ ] Sync with Fire List (tasks with due dates)
- [ ] Show recurring tasks on calendar

**New Files/Components:**
- `src/app/calendar/page.tsx`
- `src/components/calendar/Calendar.tsx`
- `src/components/calendar/CalendarDay.tsx`
- `src/components/calendar/TasksForDate.tsx`
- `src/components/calendar/DayView.tsx`
- `src/lib/calendar.ts` - Calendar utilities

**Dependencies to Add:**
- `react-big-calendar` or similar (TBD based on needs)

---

### 2.2 Home Page Redesign
**Objective:** Clear overview of upcoming items and important information

**New Home Page Features:**
- [ ] **Top Section - Quick Stats**
  - Upcoming tasks (next 7 days)
  - Overdue tasks count
  - Tasks due today
  - Habits completed today

- [ ] **Upcoming Due Dates Widget**
  - List of next 5-10 items due soon
  - Color-coded by type (task=blue, goal=purple, ritual=green)
  - Click to view/edit item
  - Snooze functionality

- [ ] **Today's Focus Section**
  - Today's habits to complete
  - Today's important tasks
  - Completed items today

- [ ] **Quick Action Cards**
  - Recent notes
  - Active ideas (heat-rated)
  - In-progress business ideas
  - Current goals with progress

- [ ] **Mini Calendar Widget**
  - Small calendar showing upcoming events
  - Click to navigate to calendar view

**New/Modified Files:**
- `src/app/page.tsx` - Complete redesign of inbox/home
- `src/components/home/UpcomingDueDates.tsx`
- `src/components/home/TodaysFocus.tsx`
- `src/components/home/QuickStats.tsx`
- `src/components/home/MiniCalendar.tsx`
- `src/components/home/RecentItems.tsx`

---

## Phase 3: UI/UX Improvements & Clarity

### 3.1 Navigation & Section Clarity
**Objective:** Make it clear what each tab is for

**Tasks:**
- [ ] Redesign sidebar with:
  - Section headers (e.g., "📝 Knowledge", "💡 Ideas", "📊 Business", "⚙️ Productivity")
  - Clear descriptions/subtitles under each nav item
  - Icons remain but with better visual grouping
- [ ] Add help/onboarding tooltips on first visit
- [ ] Create page header with:
  - Section title
  - Description of what this section is for
  - Quick help button (?)

**Revised Navigation Structure:**
```
📝 KNOWLEDGE
├── 📥 Inbox - Process and capture new items
├── 🧠 Brain Dump - Organize your notes and ideas
├── 🔒 Vault - Private, secured notes

💡 IDEAS & BUSINESS
├── ⚡ Spark - Capture raw ideas and validate them
├── 🏛️ Empire - Develop and track business ideas
├── 📊 Stats - Analytics and overview

⚙️ PRODUCTIVITY
├── 🔥 Fire List - Tasks and todo items
├── 🔁 Rituals - Daily habits and routines
├── 🎯 Goals - Long-term goals and milestones
├── 📅 Calendar - View tasks on calendar

⚙️ SETTINGS
├── ⚙️ Settings - Customize your experience
├── 👤 Account - Profile and account settings
```

**New/Modified Files:**
- `src/components/Sidebar.tsx` - Redesign with sections
- `src/components/PageHeader.tsx` - New component for page titles/descriptions
- `src/app/[section]/layout.tsx` - Add consistent headers

---

### 3.2 Visual Hierarchy & Polish
**Objective:** Clearer, more organized UI

**Tasks:**
- [ ] Implement consistent card designs with clear spacing
- [ ] Add better typography hierarchy
- [ ] Improve color contrast and readability
- [ ] Add empty states for all sections (illustrations + helpful text)
- [ ] Improve modals with better titles and descriptions
- [ ] Add breadcrumbs or context for nested views
- [ ] Implement loading states and skeletons
- [ ] Better error messages and states
- [ ] Mobile-first responsive improvements

**Files to Update:**
- `src/app/globals.css` - Enhanced CSS custom properties
- All page components - Consistent layout patterns
- All modal components - Better structure

---

## Phase 4: Customizable UI/Theme System

### 4.1 Theme Customization
**Objective:** Allow users to fully customize the appearance

**Tasks:**
- [ ] Extend theme system beyond accent colors:
  - Background color options (true dark, dark gray, light, etc.)
  - Text color intensity (high contrast, normal, low contrast)
  - Font family options (sans-serif, serif, monospace)
  - Font size multiplier (80%, 90%, 100%, 110%, 120%)
  - Spacing multiplier (compact, normal, spacious)
- [ ] Create Settings page with theme controls
  - Color picker or preset selection
  - Dark/light mode toggle
  - Font size slider
  - Spacing options
  - Preview of changes in real-time
- [ ] Save preferences to user profile/database
- [ ] Implement CSS variable system for all theme options

**New Components:**
- `src/components/settings/ThemeCustomizer.tsx`
- `src/components/settings/FontSettings.tsx`
- `src/components/settings/ColorPicker.tsx`
- `src/components/settings/PreviewPanel.tsx`

**New Files:**
- `src/lib/themes.ts` - Theme definitions and utilities
- `src/app/settings/page.tsx`
- `src/styles/themes.css` - Dynamic CSS variables

**Modified Files:**
- `src/store/useStore.ts` - Add user preferences state
- `src/app/layout.tsx` - Apply theme on load
- `src/app/globals.css` - Extensive CSS variable system

---

### 4.2 Dashboard Customization
**Objective:** Let users customize home page layout

**Tasks:**
- [ ] Create dashboard widget system:
  - Draggable widgets (use react-beautiful-dnd or similar)
  - Show/hide widgets
  - Resize widgets
  - Reorder widgets
- [ ] Allow users to create custom views
- [ ] Save dashboard layout to profile
- [ ] Multiple saved layouts/dashboards

**New Components:**
- `src/components/dashboard/DraggableWidget.tsx`
- `src/components/dashboard/WidgetGallery.tsx`
- `src/components/dashboard/DashboardEditor.tsx`

**Dependencies:**
- `react-beautiful-dnd` or `dnd-kit`

---

## Phase 5: Feature Enhancements & Polish

### 5.1 Task Improvements
**Objective:** Better task management with calendar integration

**Tasks:**
- [ ] Add recurring task improvements
  - Skip occurrence
  - Modify future occurrences
  - End recurring task
- [ ] Add task templates
- [ ] Add task dependencies
- [ ] Better date/time picker for due dates
- [ ] Time zone support

---

### 5.2 Data & Performance
**Objective:** Ensure app works smoothly with backend

**Tasks:**
- [ ] Implement pagination for large lists
- [ ] Add data syncing status indicator
- [ ] Offline support (sync when online)
- [ ] Implement data search index
- [ ] Add data export functionality (JSON, CSV)
- [ ] Improve Command Palette with server-side search

---

### 5.3 Mobile Experience
**Objective:** First-class mobile support

**Tasks:**
- [ ] Optimize for mobile (touch targets, responsive)
- [ ] Mobile-specific navigation (bottom nav or hamburger menu)
- [ ] Mobile optimized calendar view
- [ ] Mobile optimized modals
- [ ] Add PWA capabilities (offline, installable)

---

## Technical Architecture

### Tech Stack Changes
**Frontend (no changes):**
- Next.js 16
- React 19
- TypeScript
- Tailwind CSS 4
- Framer Motion

**New Backend:**
- Authentication: NextAuth.js v5 or Supabase Auth
- Database: Supabase, Firebase, or PostgreSQL + Prisma
- Email: SendGrid or similar
- File Storage: For user avatars and media

**New Dependencies:**
```json
{
  "next-auth": "^5.x",
  "prisma": "^5.x",
  "@prisma/client": "^5.x",
  "zustand": "^5.x",
  "react-big-calendar": "^1.x",
  "react-hot-toast": "^2.x",
  "zod": "^3.x"
}
```

### API Routes Structure
```
/api/auth/
  - signin
  - signup
  - signout
  - verify-email
  - reset-password

/api/notes/
  - GET, POST (list/create)
  - [id] GET, PUT, DELETE

/api/tasks/
  - GET, POST (list/create)
  - [id] GET, PUT, DELETE
  - [id]/complete
  - [id]/snooze

/api/ideas/
  - GET, POST
  - [id] GET, PUT, DELETE

/api/empire/
  - GET, POST
  - [id] GET, PUT, DELETE

/api/goals/
  - GET, POST
  - [id] GET, PUT, DELETE

/api/rituals/
  - GET, POST
  - [id] GET, PUT, DELETE
  - [id]/track

/api/user/
  - /profile GET, PUT
  - /preferences GET, PUT
  - /settings GET, PUT
```

### Database Schema (High-level)
```sql
-- Users
Users {
  id UUID PRIMARY KEY
  email VARCHAR UNIQUE
  password_hash VARCHAR
  display_name VARCHAR
  avatar_url VARCHAR
  timezone VARCHAR
  email_verified BOOLEAN
  created_at TIMESTAMP
  updated_at TIMESTAMP
}

-- User Preferences
UserPreferences {
  id UUID PRIMARY KEY
  user_id UUID FK
  accent_color VARCHAR
  dark_mode BOOLEAN
  font_size FLOAT
  theme_settings JSON
  dashboard_layout JSON
}

-- Notes (with user_id)
Notes {
  id UUID PRIMARY KEY
  user_id UUID FK
  title VARCHAR
  content TEXT
  tags VARCHAR[]
  pinned BOOLEAN
  archived BOOLEAN
  created_at TIMESTAMP
  updated_at TIMESTAMP
}

-- Tasks (with user_id)
Tasks {
  id UUID PRIMARY KEY
  user_id UUID FK
  title VARCHAR
  content TEXT
  priority VARCHAR
  due_date TIMESTAMP
  completed BOOLEAN
  completed_at TIMESTAMP
  recurring VARCHAR (daily, weekly, monthly)
  snoozed_until TIMESTAMP
  created_at TIMESTAMP
  updated_at TIMESTAMP
}

-- Similar structure for: Ideas, BusinessIdeas, Goals, Rituals, VaultNotes
-- All with user_id foreign key
```

---

## Implementation Phases Timeline

| Phase | Focus | Estimated Size |
|-------|-------|-----------------|
| Phase 1 | Backend, Auth, Data Migration | Large |
| Phase 2 | Calendar, Home Redesign | Medium |
| Phase 3 | UI Polish, Clarity | Medium |
| Phase 4 | Theme Customization | Medium |
| Phase 5 | Polish, Mobile, Performance | Variable |

---

## Breaking Changes & Considerations

1. **Data Migration:** Existing localStorage data must be migrated to cloud
2. **Authentication:** All users must create accounts (migration path needed)
3. **API Rate Limiting:** Add rate limiting to prevent abuse
4. **Data Privacy:** Clear privacy policy and data handling documentation
5. **Backup Strategy:** User data backup and recovery features
6. **Performance:** Larger datasets require pagination and optimization

---

## Success Metrics

- [ ] All users successfully migrated with data intact
- [ ] Email verification working reliably
- [ ] Calendar displays tasks correctly
- [ ] Home page loads in <1s
- [ ] Mobile responsive and usable
- [ ] Theme customization saves correctly
- [ ] No data loss during migration

---

## Recommendations Before Starting

1. **Choose Backend:** Decide between Supabase, Firebase, or custom stack
2. **Email Service:** Set up email service for verification flows
3. **Data Export:** Create tool to export user data before changes
4. **Testing:** Create test accounts and data for each phase
5. **Design System:** Finalize design tokens and component library
6. **Security Audit:** Review auth flow and data handling
7. **Database Backup:** Automated backups before Phase 1 goes live
