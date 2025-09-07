# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

UIGen is an AI-powered React component generator with live preview, built with Next.js 15, React 19, TypeScript, and Tailwind CSS v4. The application uses a virtual file system for component generation and provides real-time preview capabilities.

## Development Commands

### Setup & Installation
```bash
npm run setup                    # Install deps, generate Prisma client, run migrations
npm install                      # Install dependencies only
npx prisma generate             # Generate Prisma client
npx prisma migrate dev          # Run database migrations
npm run db:reset                # Reset database (destructive)
```

### Development & Testing
```bash
npm run dev                     # Start dev server with Turbopack
npm run dev:daemon             # Start dev server in background, logs to logs.txt
npm run build                   # Build for production
npm run start                   # Start production server
npm run lint                    # Run ESLint
npm run test                    # Run Vitest tests
```

### Running Single Tests
```bash
npm run test src/components/chat/ChatInterface.test.tsx
npm run test -- --run          # Run tests once without watch mode
npm run test -- --reporter=verbose  # Verbose test output
```

## Code Architecture

### Core Architecture Pattern
The application follows a **Virtual File System + Real-time Collaboration** pattern:

1. **Virtual File System (`VirtualFileSystem`)** - In-memory file management for generated components
2. **Context Architecture** - React contexts for state management across the app
3. **AI Tool Integration** - Claude AI integration with custom tools for file manipulation
4. **Resizable Panel System** - Three-panel layout (Chat | Preview/Code Editor)

### Key Architectural Components

#### Virtual File System (`src/lib/file-system.ts`)
- **Purpose**: Manages generated React components in memory without writing to disk
- **Key Methods**: `createFile()`, `updateFile()`, `deleteFile()`, `serialize()`, `deserialize()`
- **Integration**: Used by AI tools and chat system for component generation/editing

#### Context System
- **FileSystemProvider** (`src/lib/contexts/file-system-context.tsx`) - Manages virtual file state
- **ChatProvider** (`src/lib/contexts/chat-context.tsx`) - Handles AI chat interactions and tool calls
- **Pattern**: Provider components wrap the main application and provide state through React contexts

#### AI Tool System (`src/lib/tools/`)
- **str-replace** - String replacement operations in virtual files
- **file-manager** - File/directory operations (rename, delete, move)
- **Integration**: Tools are passed to Claude AI for autonomous file system manipulation

#### Database Layer (Prisma + SQLite)
- **Models**: `User`, `Project` 
- **Purpose**: Persist chat messages and virtual file system state for authenticated users
- **Anonymous Support**: App works without authentication, using local state only

### Layout Architecture

The main UI follows a **resizable three-panel layout**:

```
[Chat Interface] | [Preview/Code Editor with File Tree]
     35%         |              65%
```

- **Left Panel**: Chat interface for AI interaction
- **Right Panel**: Tabbed interface switching between:
  - **Preview Mode**: Live React component preview
  - **Code Mode**: File tree + Monaco code editor

### Authentication System
- **Optional Authentication**: App works with or without user accounts
- **Anonymous Mode**: Uses `anon-work-tracker` for temporary session persistence
- **JWT-based**: Uses Jose library for token management
- **Database**: User accounts stored in SQLite via Prisma

### Component Generation Flow
1. User enters component request in chat
2. `ChatProvider` sends request to `/api/chat` endpoint
3. AI uses `str_replace_editor` and `file_manager` tools to create/modify files
4. `VirtualFileSystem` maintains file state in memory
5. Changes trigger re-render in Preview/Code panels
6. For authenticated users, state persists to database

## Key Dependencies & Versions

### Core Stack
- **Next.js 15** with App Router - Main framework
- **React 19** - Component library  
- **TypeScript 5** - Type safety
- **Tailwind CSS v4** - Styling system
- **Prisma 6** - Database ORM with SQLite

### AI Integration
- **@ai-sdk/anthropic 1.2.12** - Claude AI integration
- **ai 4.3.16** - Vercel AI SDK for streaming responses

### UI Components
- **@radix-ui/** - Headless UI primitives for dialogs, tabs, etc.
- **@monaco-editor/react** - Code editor component
- **react-resizable-panels** - Resizable layout system

### Testing
- **Vitest** - Test runner with JSdom environment
- **@testing-library/react** - React component testing utilities

## Environment Configuration

### Required Environment Variables
```bash
# Optional - app works without this
ANTHROPIC_API_KEY=your-api-key-here
```

### Default Behavior
- **Without API Key**: Uses mock provider, returns static code instead of AI-generated components
- **With API Key**: Full AI-powered component generation via Claude

## File System Conventions

### Virtual File System Rules
- **Root Directory**: All files exist under `/` in virtual file system
- **Entry Point**: Every project must have `/App.jsx` as the main component
- **Import Alias**: Use `@/` for importing non-library files (e.g., `@/components/Calculator`)
- **No HTML Files**: React components only, no static HTML
- **Styling**: Use Tailwind CSS classes, not hardcoded styles

### Real File Organization
- **`src/app/`** - Next.js App Router pages and API routes
- **`src/components/`** - Reusable UI components organized by domain
- **`src/lib/`** - Shared utilities, contexts, and business logic
- **`src/actions/`** - Server actions for database operations
- **`prisma/`** - Database schema and migrations

## Testing Strategy

### Test Location Pattern
Tests are co-located with components in `__tests__/` directories:
```
src/components/chat/
├── ChatInterface.tsx
└── __tests__/
    └── ChatInterface.test.tsx
```

### Key Test Areas
- **Virtual File System**: Core functionality testing
- **React Components**: UI component behavior and rendering
- **Context Providers**: State management and data flow
- **JSX Transformer**: Code transformation utilities