# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Repository state (read this first)

This repository does **not** contain source files directly. The entire Expo/React
Native app lives inside `my-app.zip` at the repo root, uploaded as a single binary
blob (see `git log`: "Add files via upload"). There is no extracted `my-app/`
directory checked into git.

**`my-app.zip` is corrupted/incomplete.** It reports itself as "the last disk of a
multi-part archive" and only a handful of entries are actually recoverable:
`my-app/package.json`, `my-app/package-lock.json`, `my-app/screens/LoginScreen.js`,
and `my-app/screens/Newsreen.js`. Standard `unzip`, Python's `zipfile`, and a raw
local-file-header scan all agree: the compressed bytes for every other file
(`App.js`, `app.json`, `babel.config.js`, everything under `components/`, `data/`,
`navigators/`, and all image assets) are simply not present in this file, even
though their filenames appear in the zip's central directory. Don't spend time
trying harder to unzip it — the missing volume(s) of the archive were never
uploaded. If you need the real source, ask the user for a complete export (a
fresh zip, or better, the extracted source pushed as normal git-tracked files).

The architecture notes below are reconstructed from what *is* recoverable:
the two real source files, the full (name-only) file listing in the zip's
central directory, `package.json`'s dependency list, and the package names
present under `node_modules/` in the archive. Treat unread files' *existence
and purpose* as reliable, but not their exact contents.

## What the app is

An Expo-managed React Native app (Expo SDK 51) that clones a basic Instagram-like
flow: a login screen and a home feed with stories and posts. Judging by
`assets/` (heart, comment, direct-message, plus icons) and the component names
below, the home feed is meant to look like a social media timeline.

## Commands

Run these from inside the extracted `my-app/` directory (after installing deps):

```
npm install          # installs deps (package-lock.json is present and real)
npm start             # expo start
npm run android        # expo start --android
npm run ios            # expo start --ios
npm run web             # expo start --web
```

No lint or test tooling is configured — `package.json` has no `lint`/`test`
script, no ESLint/Prettier config file, and no test framework in
`devDependencies`. Don't assume Jest or ESLint exist unless you see them added.

## Dependencies (from `package.json`, real/verified)

- `expo` ~51.0.8, `expo-status-bar`
- `react` 18.2.0, `react-native` 0.74.1
- `@react-navigation/native`, `@react-navigation/native-stack`, `@react-navigation/bottom-tabs`, plus `react-native-screens` / `react-native-safe-area-context`
- `formik` + `yup` — used for form state and validation (the login form)
- Only dev dependency: `@babel/core`

## Architecture (reconstructed)

Standard Expo entry point flow:

```
App.js
  -> navigators/StackNavigator.js   (@react-navigation/native-stack)
  -> navigators/TapNavigator.js     (@react-navigation/bottom-tabs — note the
                                      filename typo: "Tap" not "Tab". This is
                                      the actual file name, not a mistake to
                                      silently "fix" when referencing it)
  -> screens/LoginScreen.js
  -> screens/Newsreen.js            (home/feed screen — also a filename typo,
                                      short for "NewsScreen")
```

Confirmed by reading the two recovered screens:

- **`screens/LoginScreen.js`** renders `components/login/Brand.js` (branding/logo)
  and `components/login/LoginForm.js` (the Formik+Yup form) inside a plain
  `View`.
- **`screens/Newsreen.js`** renders `components/login/home/Header.js`,
  `components/login/home/Stories.js`, then maps over mock data imported from
  `data/posts.js` to render a `components/login/home/Post.js` per item, inside
  a `SafeAreaView` + `ScrollView`.

So the component tree under `components/login/` covers both the login flow
(`Brand.js`, `LoginForm.js`) and the home feed (`home/Header.js`,
`home/Stories.js`, `home/Post.js`) despite the shared `login/` directory name
— it isn't scoped strictly to authentication.

`data/posts.js` and `data/Users.js` are static mock data modules, not an API
client or backend integration — there is no networking library
(`axios`/`fetch` wrapper) in `package.json`.

## Code conventions (observed in the recovered files)

- Functional components as arrow-function `const X = () => { ... }`, default-exported at the bottom of the file.
- Styles defined with `StyleSheet.create({...})` in a `const styles` declared after the component, at the bottom of the file.
- Imports: React Native primitives from `'react-native'`, `React` from `'react'`, then relative imports (`../components/...`, `../data/...`).
- No TypeScript — plain `.js` files throughout.
