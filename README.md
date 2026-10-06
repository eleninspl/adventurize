# Adventurize

<img src="adventurize/lib/assets/logo.png" alt="Adventurize logo" width="120" align="right">

A Flutter mobile app that mixes travel discovery with a small social network. Users take photos that are pinned to a map at the place they were taken ("memories"), complete photo challenges to earn points, add friends by scanning each other's QR codes, and compete with them on a leaderboard. All data is stored on the device in SQLite.

Adventurize was a team project for **Human-Computer Interaction** (Αλληλεπίδραση Ανθρώπου-Υπολογιστή), a 7th-semester course at the School of Electrical and Computer Engineering, National Technical University of Athens (ECE NTUA), academic year 2024–25. It was built by a team of three: George Isopoulos ([@gtiso](https://github.com/gtiso)), Dimitris Tzellos ([@dimtze03](https://github.com/dimtze03)) and me, Eleni Nasopoulou. This repository is a fork of the team repository, [gtiso/adventurize](https://github.com/gtiso/adventurize).

## Features

| Screen | What it does |
|--------|--------------|
| Login / Register | Sign in with e-mail and password, or create an account with name, e-mail, birth date and password. |
| Main map | Google Map that moves to the user's location. Shows the user's memories and their friends' memories as markers with the poster's avatar. Tapping a marker opens the memory. A badge shows the user's level (one level per 100 points). |
| Camera | Full-screen camera with a flash toggle. A photo can be posted as a free memory or as the answer to a challenge. |
| Post memory | Add a location title and a description, then post. The photo is copied into app storage and saved with the current GPS position. Posting gives haptic feedback and a confetti animation. |
| Challenges | List of photo challenges (for example "Timeless Wonders!" at the Acropolis), each worth a number of points. Starting a challenge opens the camera. Posting the photo marks the challenge as done and adds its points to the user's score. A completed challenge links to the memories page instead. |
| Memories | Map and list of the user's own memories, newest first. |
| Profile | Avatar, name, score, and a QR code that encodes the user's e-mail. From here the user can edit the profile, open settings, or scan a friend's QR code. |
| Add friend | QR scanner. When it recognises another user's code, it shows a card with an "Add Friend" button. Friendship is mutual. |
| Leaderboard | The user and their friends, sorted by points. Tapping a user opens a larger card. |
| Settings | Shows whether camera, microphone and location permissions are granted. Turning one on requests it. Turning one off explains that this has to be done in the system settings and offers to open them. Also has a sign-out button. |

Screens slide in from the side of the main map that matches their button: profile from the top, challenges from the left, leaderboard from the bottom, memories from the right. On the challenges and profile screens, a swipe in the opposite direction goes back to the map.

## Course context

The course project was built in phases, and this repository holds the final implementation. The team's original README listed two differences from the earlier prototype:

- The prototype assumed a shared cloud database. The app stores everything locally, so users and friendships only exist on one device. A real deployment would need a backend such as Firebase.
- Likes between users were not implemented.

## Tech stack

| Area | Technology |
|------|------------|
| Framework | Flutter 3.27, Dart 3.6 |
| Maps and location | `google_maps_flutter` 2.10, `geolocator` 13 |
| Camera and QR | `camera` 0.11, `mobile_scanner` 6, `qr_flutter` 4 |
| Storage | SQLite via `sqflite` 2.4 (`sqflite_common_ffi` on Linux and Windows) |
| Permissions | `permission_handler` 10 |
| Effects | `confetti` 0.8, Flutter `HapticFeedback` |

Versions are the ones locked in [`adventurize/pubspec.lock`](adventurize/pubspec.lock).

## Project structure

The Flutter project is in the [`adventurize/`](adventurize) folder.

```
adventurize/lib
├── main.dart            Entry point: seeds the demo data and opens the login page
├── pages/               One file per screen (login, main map, camera, challenges, ...)
├── components/          Reusable widgets: user, memory and challenge cards, QR code, map background
├── database/            SQLite schema, DatabaseHelper (all queries) and demo data
├── models/              User, Memory and Challenge, with toMap/fromMap for SQLite
├── services/            Location lookup (permission request and current position)
├── utils/               Navigation helpers and slide/fade page transitions
└── assets/              Logo, avatars, challenge photos and the SansitaOne font
```

The database has five tables: `users`, `challenges`, `memories`, `friends` (one row per direction of a friendship) and `userchallenges` (which challenges each user has completed). On every start the app inserts its demo data: eight users, five challenges and nine memories around the world, with the first user already friends with three others. Users, challenges and memories that already exist are skipped.

## Getting started

### Prerequisites

- Flutter 3.27 or later (Dart 3.6 or later)
- Android SDK and an Android device or emulator with Google Play services. The team built the app for Android only; see [Known limitations](#known-limitations).
- A Google Maps SDK for Android API key. The team's key was removed from the code, so without your own key the maps load blank.

### Run

```bash
git clone https://github.com/eleninspl/adventurize.git
cd adventurize/adventurize
flutter pub get
```

Put your API key in [`android/app/src/main/AndroidManifest.xml`](adventurize/android/app/src/main/AndroidManifest.xml):

```xml
<meta-data android:name="com.google.android.geo.API_KEY" android:value="YOUR_KEY"/>
```

Then start the app on a connected device or emulator:

```bash
flutter run
```

Log in with the demo account `john.doe@example.com` / `password123`, or register a new account. The app asks for location permission on the main map and for camera permission when you first open the camera.

## Known limitations

The app is kept as the team submitted it. While writing this README, I reviewed the code again and found the issues below. They do not stop the main flows, but they are worth knowing if you build on this code:

- **Local data only.** Every device has its own database, so you can only add as friends the accounts that exist on the same device.
- **Passwords** are stored and compared in plain text. The login query prints the matching user row, including the password, to the debug log (`database/db_helper.dart:67`), and so does saving the profile (`pages/edit_profile_page.dart:69`).
- **Some memories are silently dropped.** `insMemory` skips a memory whose title and description already exist (`database/db_helper.dart:172`). This keeps the demo data from being inserted twice, but it also applies to user posts. A second photo with the same title and description, for example two photos posted without a title or description, is not saved, even though the app says "Memory posted successfully!".
- **Challenge points are added even when posting fails**, because the points update runs after the error handler (`pages/post_memory_page.dart:112`).
- **Duplicate rows.** Every visit to the challenges screen inserts another set of `userchallenges` rows (`pages/challenges_page.dart:37`). The demo friendships are inserted again on every start, and scanning the same friend twice adds the friendship twice, so the friends' memories query can return the same memory more than once. Nothing stops users from adding themselves.
- **Navigation always pushes a new page**, including going back to the map and signing out. The back stack keeps growing, and after signing out the system back button returns to the app.
- **Registering with an e-mail that is already taken** fails with a database error and shows no message. Registration also switches the global SQLite factory to the FFI implementation (`pages/register_page.dart:77`), which is only needed on desktop.
- **The leaderboard** slides in from the bottom, but the gesture that closes it is a swipe to the right, not down (`pages/leaderboard_page.dart:107`).
- **Platforms.** Only Android is set up. The iOS project has an empty Maps key and no camera or location usage descriptions in `Info.plist`, so iOS would not grant those permissions. `google_maps_flutter` has no desktop implementation, the database path is not set on macOS (`database/db_helper.dart:37`), and the web build cannot use `dart:io`.

## For students taking the course

This repository is here to show what a finished course project can look like: which Flutter packages cover maps, camera, QR codes and local storage, and how the screens fit together. Design and build your own app.

The [Flutter documentation](https://docs.flutter.dev/) and each package's page on [pub.dev](https://pub.dev/) are the best references for everything used here.
