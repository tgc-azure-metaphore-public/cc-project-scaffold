# Archetype E: Mobile App

Native iOS / Android app, distributed through TestFlight and app stores. This scaffold is deliberately conservative since mobile adds platform-specific overhead that varies by app type.

## Stack (locked defaults)

- **Framework**: Expo (React Native)
- **Language**: TypeScript
- **Navigation**: expo-router (file-based routing)
- **Styling**: NativeWind (Tailwind for React Native) or plain StyleSheet
- **State**: Zustand (lightweight) or React context for small apps
- **Backend** (if needed): separate archetype B or D project, called from the app via fetch
- **Deploy**: EAS Build + EAS Submit

## Files to generate

Rather than hand-scaffolding (Expo's CLI does a better job), generate a starter that the user initializes via Expo CLI, then customize:

```
<project-name>/
├── CLAUDE.md
├── README.md
├── .gitignore
├── EXPO-SETUP.md                  ← one-time init instructions
├── prompts/
│   └── PROMPT-01-INIT-PROJECT.md
└── (optional) kill-criteria.md
```

Then the user runs the Expo CLI from inside the project folder to generate the actual app scaffold.

## EXPO-SETUP.md body

```markdown
# Expo Setup (one-time init)

Run these commands from inside the project folder to generate the Expo scaffold:

```bash
npx create-expo-app@latest . --template blank-typescript
npm install nativewind tailwindcss
npm install -D @types/react
```

Then:

```bash
npx expo install expo-router react-native-safe-area-context react-native-screens
```

After init, paste PROMPT-01-INIT-PROJECT.md into Claude Code to continue.
```

The reason we don't hand-scaffold Expo: the Expo CLI generates a LOT of files (metro config, babel config, app.json, etc.) that we'd have to keep in sync with Expo versions. Delegating to the CLI is the right move.

## CLAUDE.md body for Archetype E

```markdown
# <Project Name>

Mobile app — <one-line description>.

## Tech stack

- Expo (React Native)
- TypeScript
- expo-router for navigation
- NativeWind for styling
- EAS Build for binaries, EAS Submit for store distribution

## Folder structure (after Expo init)

```
<project-name>/
├── app/                  ← expo-router pages
├── components/
├── assets/
├── app.json
├── package.json
└── ...
```

## Deploy

Target: **EAS Build** (Expo Application Services)

Setup:
1. `npm install -g eas-cli`
2. `eas login`
3. `eas build:configure`
4. iOS: `eas build --platform ios`
5. Android: `eas build --platform android`
6. TestFlight distribution: `eas submit --platform ios`

## First commands

1. Run the setup in EXPO-SETUP.md (one-time)
2. `npx expo start` — launches dev server
3. Scan QR code with Expo Go app on phone to preview

## Workspace inheritance

Inherits `@../CONTEXT.md` and `@../CLAUDE.md` from workspace level.
```

## .env handling

Expo handles env vars differently than web. Do not generate a `.env.template` in the scaffold — let Expo's standard pattern apply (variables prefixed with `EXPO_PUBLIC_` are exposed to the client; server-only secrets require a separate backend).

If the app needs a backend, scaffold that as a separate archetype B or D project in a sibling folder.

## Deploy hint

```
1. `npm install -g eas-cli`
2. `eas login` (first time)
3. From project root: `eas build:configure`
4. `eas build --platform ios` (first build takes 15-30 min)
5. `eas submit --platform ios` to push to TestFlight
```

## When to not scaffold archetype E

If the user wants "a mobile-responsive web app," that's archetype B (Next.js with responsive design), NOT E. Mobile-web and native-mobile are different things. Confirm by asking:

> "Do you need it in the App Store / Play Store, or is a phone-friendly website enough?"

- "Phone-friendly website" → archetype B with responsive Tailwind
- "App Store / Play Store" → archetype E
