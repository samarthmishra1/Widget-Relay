# Widget Relay

A minimal desktop → Supabase → Capacitor → native home-screen widget starter.

- **Dashboard:** Vite + vanilla JavaScript + Tailwind. Password sign-in, text/date composer, live preview, owner selector, connection/error state.
- **Backend:** Supabase Auth + Postgres + Realtime. One status per owner; A writes, explicitly paired B reads. RLS protects reads and writes; SQL explicitly adds the table to `supabase_realtime`.
- **Mobile:** Capacitor + a tiny local bridge. Android uses SharedPreferences/RemoteViews; iOS uses App Group UserDefaults/WidgetKit. No third-party widget plugin required.

## What actually runs in the background

`src/background-listener.js` is an app-lifecycle listener, not an always-running service. It subscribes to `public:widget_status`, handles UPDATE and first-time INSERT, fetches on startup/subscription/reconnection/resume, and forwards received rows to the native bridge. Updates are persisted so a newly added widget shows the most recently received note.

**A JavaScript socket cannot guarantee instant updates after the receiving app is suspended or terminated.** On Android, returning to the launcher may suspend it; on iOS this is expected. Widgets retain the last received value. WidgetKit reload is a request, not an immediate rendering guarantee. This project does not implement push or a persistent native background service. For a hackathon, test with Android split-screen or foreground app first, then verify the cached home-screen widget.

For actual delivery while B stays on the home screen: add a database webhook/Edge Function and authenticated device registration, then FCM on Android and APNs/WidgetKit push on supported iOS. Native receipt must update the same cache and request widget refresh. Keep push credentials server-side. Delivery and widget reload remain subject to OS policy, especially after force-stop. An Android foreground service is another option, but requires a visible notification and appropriate service policy; it has intentionally not been added to this lightweight starter.

## Local setup

Requirements: Node 22.12+ (Node 24 recommended), npm, a Supabase project. Native builds additionally require the SDK/toolchain for Capacitor 8: Android Studio or macOS/Xcode. Use the linked Capacitor environment guide for matching versions.

```sh
npm ci
cp .env.example .env
# Edit .env with your own HTTPS project URL and public anon/publishable key.
npm run dev
```

Open the local URL printed by Vite. The key is a public anon/publishable key; never use a secret/service-role key. Vite embeds these public values at build time. Restart Vite after editing `.env`; rebuild and sync when native configuration changes.

For a new Supabase project, complete steps 1–3. Existing installations should review their deployed policies separately.

1. Run `supabase/schema.sql` in Supabase SQL Editor.
2. In Authentication → Users, create two confirmed email/password users A and B. The app deliberately does not expose signup or password reset UI.
3. Copy their UUIDs. Replace both placeholders in `supabase/pair.sql` and run it as the project administrator. B gets read-only access to A; anonymous users get no access.
4. Sign in as A in the desktop browser. The owner field defaults to A. Send a message (or date). Dates are stored as unambiguous UTC ISO text appended to the message; this is not an alarm/scheduler.
5. Sign in as B on another browser or the native mobile app. Paste A's UUID in “Status owner UUID,” then choose **Watch status**. B cannot edit A's row.
6. Edit A's message. B receives the change while connected. The preview uses `textContent`, never HTML injection.

The SQL is repeatable against its own schema; it is not a migration for an incompatible existing `widget_status` table. `updated_at` is server-controlled `timestamptz`; `user_id` is text as requested and contains A's Auth UUID. No delete grant is exposed. Pair membership is administrator-managed to keep the client small. A UUID is an identifier, not a sharing secret.

## Android: generated project + automatic widget installation

```sh
npm run android:sync
npm run android:open
```

The Android project is included. Use `android:sync` for this checkout. Only use `android:init` when no Android directory exists. `android:init` builds web assets, creates the standard Capacitor Android project, copies the Java/XML widget files, registers the local bridge in MainActivity, adds the manifest receiver, and syncs plugins. Run this once; `cap add` requires the platform folder not to exist. Keep the package ID `com.example.widgetrelay`, or update all native package references before running the installer.

Build/run from Android Studio on a device/emulator. Sign in as B, watch A, then long-press the home screen → Widgets → Widget Relay. Tap the widget to reopen the app and catch up. All widget instances show the same selected owner. Android displays the saved timestamp in ISO format.

After web changes:

```sh
npm run android:sync
```

After editing native boilerplate, run `node scripts/install-android.mjs` and `npx cap sync android`. The installer replaces this starter's MainActivity, so merge manually if you have since customized it.

## iOS: WidgetKit extension setup (macOS required)

```sh
npm run ios:init
npm run ios:open
```

1. Add `native/ios/WidgetBridgePlugin.swift` and `WidgetViewController.swift` to the generated **App target**, with target membership checked.
2. In `Main.storyboard`, select the Capacitor bridge view controller. Set its custom class to `WidgetViewController`, module **App** (or inherit module from target).
3. File → New → Target → **Widget Extension**, name `StatusWidgetExtension`. Disable Include Configuration App Intent and Live Activity if offered. Set the widget extension deployment target to **iOS 17+**; the supplied view uses `containerBackground`.
4. Delete the generated Swift widget/bundle files from this extension target. Add `native/ios/StatusWidget.swift` to **only the extension target**. It contains the extension's single `@main` entry point.
5. In Signing & Capabilities, select your development team for both targets. Add **App Groups** to both, and register/enable the same group: `group.com.example.widgetrelay`. If you choose a unique group instead, replace that string in both Swift files and the example entitlements. Use unique bundle identifiers for app and extension as required by signing.
6. `WidgetRelay.entitlements` shows the needed App Group entitlement. Prefer Xcode's generated entitlements for each target; confirm each target's Code Signing Entitlements points to its own file. `Widget-Info.plist` shows the required extension dictionary; the Xcode template already provides it, so do not replace the full generated plist or add this example as a resource.
7. Confirm the extension is embedded in the App target (Xcode's New Target flow normally adds it). Build/run App, sign in as B, and watch A. Add Widget Relay using the home-screen widget picker.

After web changes run `npm run build` then `npx cap sync ios`. Xcode owns project/signing files. Native source may be committed; local build output and signing credentials are ignored.

## Files

```text
widget-relay/
  index.html
  package.json / package-lock.json
  vite.config.js / capacitor.config.json / .env.example
  src/
    main.js / styles.css / supabase.js
    status.js / background-listener.js / widget-bridge.js
  supabase/
    schema.sql / pair.sql
  scripts/install-android.mjs
  native/android/
    MainActivity.java / WidgetBridgePlugin.java / StatusWidgetProvider.java
    status_widget.xml / status_widget_info.xml / widget_background.xml
  native/ios/
    WidgetBridgePlugin.swift / WidgetViewController.swift / StatusWidget.swift
    WidgetRelay.entitlements / Widget-Info.plist
  tests/status.test.js / listener.test.js
  README.md / VALIDATION.md
```

## Verification checklist with your project/devices

- A's first save emits INSERT; the next emits UPDATE. Both update B's active app.
- B cannot update A's status even by calling Supabase directly. Unpaired C cannot read it. Verify with separate authenticated clients, not SQL Editor's admin role.
- Reconnect/reopen B after missed updates: the latest row replaces cached content.
- Add a widget before and after receiving a status; verify native text. Sign out and confirm cached text is cleared.
- Disable network on B, publish on A, restore network, and confirm catch-up.
- Stop/suspend B: do not expect guaranteed new messages without the separate push architecture described above.

Revocation prevents future authorized reads; it cannot erase content already cached offline. After a refetch finds no readable row, this starter clears the widget. Production should additionally handle explicit account-switching, secure session storage appropriate to the threat model, push registration/revocation, and privacy on a visible lock/home screen.

## Primary references

- Supabase Postgres Changes and publication: https://supabase.com/docs/guides/realtime/postgres-changes
- Supabase RLS: https://supabase.com/docs/guides/database/postgres/row-level-security
- Capacitor custom native iOS registration: https://capacitorjs.com/docs/ios/custom-code
- Capacitor custom Android code: https://capacitorjs.com/docs/android/custom-code
- Capacitor environment: https://capacitorjs.com/docs/getting-started/environment-setup
- Tailwind Vite integration: https://tailwindcss.com/docs/installation/using-vite
- WidgetKit refresh scheduling: https://developer.apple.com/documentation/widgetkit/keeping-a-widget-up-to-date
- WidgetKit push: https://developer.apple.com/documentation/WidgetKit/Updating-widgets-with-widgetkit-push-notifications

## Public repository checks

Run `npm run security:check` before committing. See SECURITY.md for scope and remaining deployment checks. The development server binds to localhost by default. Never commit your configured `.env` or administrator pairing data.
