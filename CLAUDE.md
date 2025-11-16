# CLAUDE.md - IPTVnator Development Guide for AI Assistants

This document provides essential information about the IPTVnator codebase structure, development workflows, and conventions for AI assistants working on this project.

## Project Overview

**IPTVnator** is a cross-platform IPTV player application built with Angular and Tauri (formerly Electron). It supports M3U/M3U8 playlists, Xtream Codes, Stalker Portal, EPG integration, and multiple video players.

- **Version**: 1.0.0-beta.7
- **Repository**: https://github.com/4gray/iptvnator
- **License**: MIT
- **Node.js**: >=20.8.1
- **Package Manager**: npm

## Technology Stack

### Frontend
- **Framework**: Angular 20.3.1
- **Language**: TypeScript 5.9.2
- **State Management**: NgRx 20.0.0 (store, effects, entity, signals, router-store, component-store)
- **UI Framework**: Angular Material 20.2.4
- **Styling**: SCSS with Material 3 theme (`src/m3-theme.scss`)
- **Video Players**: ArtPlayer 5.3.0 (primary), Video.js 7.20.3, HLS.js 1.6.13
- **Reactive Programming**: RxJS 7.8.2 + @rx-angular (cdk & template)
- **Internationalization**: @ngx-translate/core (16 languages supported)
- **Local Storage**: @ngx-pwa/local-storage + ngx-indexed-db

### Desktop Backend
- **Framework**: Tauri 2.3.1 (Rust-based)
- **Database**: SQLite via tauri-plugin-sql
- **Key Plugins**: dialog, fs, http, updater, window-state

### Build & Development
- **Bundler**: Angular CLI with esbuild (@angular/build)
- **Target**: ES2022, Chrome 107

## Directory Structure

```
/
├── src/                          # Angular application
│   ├── app/
│   │   ├── home/                 # Welcome screen & playlist imports
│   │   │   ├── file-upload/
│   │   │   ├── url-upload/
│   │   │   ├── xtream-code-import/
│   │   │   └── stalker-portal-import/
│   │   ├── player/               # Core video player functionality
│   │   │   └── components/
│   │   │       ├── video-player/
│   │   │       ├── art-player/   # ArtPlayer integration
│   │   │       ├── vjs-player/   # Video.js player
│   │   │       ├── channel-list-container/
│   │   │       └── epg-list/     # EPG/TV guide
│   │   ├── settings/             # Application settings
│   │   ├── services/             # Core application services
│   │   ├── state/                # NgRx state management
│   │   └── shared/               # Shared components, pipes, services
│   ├── assets/
│   │   └── i18n/                 # Translation files (16 languages)
│   └── environments/             # Environment configurations
├── src-tauri/                    # Tauri (Rust) backend
│   ├── src/
│   └── tauri.conf.json
├── e2e/                          # Playwright end-to-end tests
├── shared/                       # Shared TypeScript interfaces
├── docker/                       # Docker configuration
└── .github/workflows/            # CI/CD pipelines

```

## Key Architectural Patterns

### State Management (NgRx)

Location: `/src/app/state/`

**Key Files:**
- `actions.ts` - Action creators (~40 actions)
- `reducers.ts` - Pure reducer functions
- `selectors.ts` - Memoized selectors
- `effects.ts` - Side effects handling
- `state.ts` - State interface definitions
- `playlists.state.ts` - Entity adapter for playlists

**State Slices:**
- Playlist metadata (using @ngrx/entity adapter)
- Active channel/playlist
- EPG programs
- Favorites
- Selected filters
- Loading states

**Best Practices:**
- Use typed actions with `createAction` and `props`
- Feature store pattern with `createFeatureSelector`
- Effects for side effects (API calls, storage operations)
- Entity adapter for normalized playlist data
- Router state integration via `@ngrx/router-store`

### Platform Abstraction (Factory Pattern)

The application supports multiple platforms (Web PWA, Tauri Desktop) through a factory pattern:

```typescript
// src/app/services/data.service.ts
export function DataFactory() {
    if (isTauri()) {
        return new TauriService();
    }
    return new PwaService();
}
```

**Key Services:**
- `data.service.ts` - Abstract service interface
- `tauri.service.ts` - Tauri-specific implementation (SQLite)
- `pwa.service.ts` - Web-specific implementation (IndexedDB)
- `playlists.service.ts` - Playlist CRUD operations
- `epg.service.ts` - EPG data handling
- `player.service.ts` - Video player management

### Component Architecture

**Mix of Module-based and Standalone:**
- App module uses traditional NgModule
- Many feature components use standalone pattern
- Lazy-loaded routes with `loadComponent()`

**Patterns:**
- Smart/Presentational component separation
- OnPush change detection strategy (implied via Material/RxAngular)
- Reactive programming with Observables and signals
- Component imports array for standalone components

## Development Workflows

### Local Development

```bash
# Install dependencies
npm install

# Start Tauri app (desktop) - opens separate window
npm run tauri dev

# Start Angular only (web/PWA) - http://localhost:4200
npm run serve

# Build for different targets
npm run build:dev        # Development build
npm run build:prod       # Production (Tauri)
npm run build:web        # Web/PWA deployment
```

### Testing

**Unit Tests (Jest):**
```bash
npm test                 # Run all tests
npm run test:watch       # Watch mode
```

**E2E Tests (Playwright):**
```bash
npm run e2e              # Interactive UI mode
npm run e2e:ci           # CI mode with retries
```

**Linting:**
```bash
npm run lint             # ESLint for TS/HTML
```

### Build Configurations

**Environment Files:**
- `environment.ts` - Default (local)
- `environment.dev.ts` - Development
- `environment.prod.ts` - Production (Tauri)
- `environment.web.ts` - Web/PWA

**Environment Variables:**
```typescript
{
    production: boolean,
    environment: 'LOCAL' | 'PRODUCTION' | 'WEB',
    version: string,
    BACKEND_URL: string
}
```

## Naming Conventions

**File Naming:**
- Components: `*.component.ts`
- Services: `*.service.ts`
- Tests: `*.spec.ts`
- Stubs: `*.stub.ts`
- Interfaces: `*.interface.ts`
- Enums: `*.enum.ts`
- Type separator: `.` (e.g., `auth.guard.ts`, `http.interceptor.ts`)

**Angular Schematics Configuration:**
- Component prefix: `app`
- Style: SCSS
- Type separator: `.` (for guards, interceptors, pipes, resolvers)

## Git Workflow & Commit Conventions

### Commit Messages

This project uses **Conventional Commits** enforced by commitlint:

```bash
# Format: <type>(<scope>): <subject>

# Types:
feat:     # New feature
fix:      # Bug fix
docs:     # Documentation changes
style:    # Code style changes (formatting, no logic change)
refactor: # Code refactoring
perf:     # Performance improvements
test:     # Adding or updating tests
build:    # Build system changes
ci:       # CI/CD changes
chore:    # Other changes (dependencies, tooling)

# Examples:
feat(player): integrate Artplayer and remove DPlayer
fix(web-player): remove unused dplayer
chore(deps): bump multiple dev/prod dependency versions
```

### Pre-commit Hooks (Husky)

The following checks run automatically before each commit:
1. Commitlint (validates commit message format)
2. Jest tests (`npm test`)
3. ESLint (`npm run lint`)

### Versioning

- Uses **semantic-release** for automated versioning
- Changelog generated via `conventional-changelog`
- Version updates trigger new releases

## Testing Patterns

### Unit Tests (Jest)

**Configuration:**
- Setup: `/src/setup-jest.ts`
- Config: `tsconfig.spec.json`
- Pattern: `**/*.spec.ts`

**Common Patterns:**
```typescript
import { TestBed } from '@angular/core/testing';
import { provideMockStore, MockStore } from '@ngrx/store/testing';
import { MockComponent, MockModule, MockProviders } from 'ng-mocks';

beforeEach(waitForAsync(() => {
  TestBed.configureTestingModule({
    imports: [MockModule(SomeModule)],
    declarations: [MockComponent(SomeComponent)],
    providers: [
      provideMockStore(),
      MockProviders(SomeService)
    ]
  });
}));
```

### E2E Tests (Playwright)

**Location:** `/e2e/`
**Pattern:** `*.e2e.ts`

**Configuration:**
- Runs against dev server (port 4200)
- Desktop Chrome device emulation
- Screenshot on failure
- Test timeout: 45s

## Data Persistence

### Web (PWA)
- **Technology**: IndexedDB via `ngx-indexed-db`
- **Schema**: `/src/app/indexed-db.config.ts`

### Desktop (Tauri)
- **Technology**: SQLite via tauri-plugin-sql
- **Location**: Application data directory
- **Migrations**: Handled in Rust backend

## CI/CD Pipelines

**GitHub Actions Workflows:**
- `.github/workflows/tauri-release.yml` - Multi-platform Tauri releases (macOS, Windows, Linux)
- `.github/workflows/docker.yml` - Docker builds
- `.github/workflows/codeql-analysis.yml` - Security scanning

**Release Process:**
1. Commits to `master` trigger Tauri release workflow
2. Builds for macOS (Intel + ARM), Windows, Linux
3. Creates draft release with binaries
4. Semantic-release handles versioning and changelog

## Key Integration Points

1. **Multi-platform abstraction** via factory pattern (DataService)
2. **Multiple video player support** (ArtPlayer, Video.js, HTML5)
3. **Multiple playlist formats** (M3U, Xtream Codes, Stalker Portal)
4. **Cross-platform data storage** (IndexedDB for web, SQLite for desktop)
5. **Auto-update mechanism** for playlists
6. **EPG integration** with XMLTV format
7. **External player support** (MPV, VLC via Tauri commands)

## Important Notes for AI Assistants

### When Making Changes

1. **State Management**: Always use NgRx actions for state changes. Never mutate state directly.
2. **Platform Compatibility**: Consider both web (PWA) and desktop (Tauri) platforms. Use the factory pattern for platform-specific code.
3. **Testing**: Add unit tests for new features. Update existing tests when modifying functionality.
4. **Type Safety**: Ensure TypeScript strict type checking passes (though `strict: false` in tsconfig, maintain type safety).
5. **Internationalization**: Add translation keys to `/src/assets/i18n/*.json` for user-facing strings.
6. **Video Players**: The app now uses ArtPlayer as primary player (Video.js and HTML5 as fallbacks).

### Common Tasks

**Adding a New Feature:**
1. Create feature branch
2. Add NgRx actions/reducers/selectors if state management needed
3. Implement component/service
4. Add unit tests
5. Update translations
6. Test in both web and Tauri modes
7. Commit with conventional commit message
8. Ensure pre-commit hooks pass

**Fixing a Bug:**
1. Reproduce the issue
2. Write a failing test (if applicable)
3. Fix the bug
4. Verify test passes
5. Test manually in both platforms
6. Commit with `fix:` prefix

**Updating Dependencies:**
1. Update `package.json`
2. Run `npm install`
3. Test thoroughly (unit tests, e2e tests, manual testing)
4. Update `CHANGELOG.md` if needed
5. Commit with `chore(deps):` prefix

### Code Quality Standards

- **Linting**: ESLint with Angular/TypeScript plugins + @ngrx/eslint-plugin
- **Formatting**: Prettier 3.6.2
- **Style**: Follow Angular style guide
- **Components**: Use OnPush change detection where possible
- **RxJS**: Clean up subscriptions (use `takeUntil`, async pipe, or signals)
- **Accessibility**: Follow WCAG 2.1 guidelines with Angular Material

## Resources

- [Angular Documentation](https://angular.dev)
- [NgRx Documentation](https://ngrx.io)
- [Tauri Documentation](https://tauri.app)
- [Project Repository](https://github.com/4gray/iptvnator)
- [Issue Tracker](https://github.com/4gray/iptvnator/issues)
- [Telegram Channel](https://t.me/iptvnator)

## Support & Community

- GitHub Issues: https://github.com/4gray/iptvnator/issues
- Telegram: https://t.me/iptvnator
- Bluesky: https://bsky.app/profile/iptvnator.bsky.social

---

**Last Updated**: 2025-11-16
**Version**: Based on IPTVnator v1.0.0-beta.7
