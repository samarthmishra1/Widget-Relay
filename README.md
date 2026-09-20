# Widget Relay

**Widget Relay** is a real-time communication app that lets one person send a status or short message directly to another person's mobile home-screen widget.

Instead of opening an app, checking a notification, or refreshing a page, the latest message is available directly on the recipient's home screen.

## How It Works

```text
┌──────────────────┐
│      USER A      │
│                  │
│  Sends a status  │
│    or message    │
└────────┬─────────┘
         │
         ▼
┌──────────────────┐
│   WIDGET RELAY   │
│                  │
│  Syncs the update│
│   in real time   │
└────────┬─────────┘
         │
         ▼
┌──────────────────┐
│      USER B      │
│                  │
│  Receives it on  │
│  the home screen │
└──────────────────┘
```

**Send → Sync → Display**

The goal is simple: make small pieces of information immediately visible without requiring the recipient to open an app.

## Features

- **Real-time updates** — Status changes are reflected almost instantly.
- **Home-screen widgets** — View the latest status without opening the app.
- **Cross-device communication** — Send an update from one device and receive it on another.
- **User authentication** — Individual accounts for users.
- **Owner & viewer pairing** — Connect specific users for sharing statuses.
- **Android support** — Native Android home-screen widget.
- **iOS support** — Native WidgetKit implementation.
- **Automatic refresh** — Updates refresh when the app reconnects or becomes active.
- **Offline persistence** — The latest received status remains available if the connection drops.
- **Minimal interface** — Simple UI focused on sending and receiving.
- **Responsive design** — Designed for both desktop and mobile screens.

## Communication Flow

```text
      SENDER                              RECEIVER
┌─────────────────┐                 ┌─────────────────┐
│                 │                 │                 │
│  Write Status   │                 │  Mobile Device  │
│                 │                 │                 │
└────────┬────────┘                 └────────▲────────┘
         │                                   │
         ▼                                   │
┌─────────────────┐                 ┌────────┴────────┐
│                 │    Realtime     │                 │
│  Widget Relay   │ ──────────────► │ Latest Status   │
│                 │                 │                 │
└─────────────────┘                 └────────┬────────┘
                                            │
                                            ▼
                                   ┌─────────────────┐
                                   │                 │
                                   │  Home-Screen    │
                                   │     Widget      │
                                   │                 │
                                   └─────────────────┘
```

## Tech Stack

| Area | Technology |
| --- | --- |
| Frontend | Vite + JavaScript |
| Styling | Tailwind CSS |
| Backend | Supabase |
| Database | PostgreSQL |
| Realtime | Supabase Realtime |
| Mobile | Capacitor |
| Android | Java + RemoteViews |
| iOS | Swift + WidgetKit |

## Platforms

### 🌐 Web

A lightweight dashboard for sending and viewing statuses with a responsive interface for desktop and mobile browsers.

### 🤖 Android

Native Android home-screen widget support allows received statuses to remain visible directly on the user's home screen.

### 🍎 iOS

WidgetKit support provides the home-screen widget experience for iOS users.

## User Flow

```text
┌─────────────┐
│   Sign In   │
└──────┬──────┘
       │
       ▼
┌─────────────┐
│   Connect   │
│    Users    │
└──────┬──────┘
       │
       ▼
┌─────────────┐
│ Send Status │
└──────┬──────┘
       │
       ▼
┌─────────────┐
│  Realtime   │
│    Sync     │
└──────┬──────┘
       │
       ▼
┌─────────────┐
│ Home-Screen │
│   Widget    │
└─────────────┘
```

## Use Cases

Widget Relay can be adapted for several types of lightweight real-time communication:

- Shared status widgets
- Friends and family widgets
- Quick messages
- Couple widgets
- Team availability indicators
- Study or productivity status sharing
- Remote work status updates
- Cross-device communication
- Home-screen information displays
- Realtime mobile experiences

## Why Widget Relay?

Most communication apps require the recipient to actively open an application or interact with a notification.

Widget Relay reduces that interaction to:

```text
Someone sends something
          │
          ▼
   It synchronizes
          │
          ▼
It appears on your home screen
```

The information stays visible without demanding the recipient's attention.

## Current Limitations

Mobile operating systems can restrict background activity when an application has been suspended or terminated.

Because of this, real-time updates may not always reach the home-screen widget immediately while the receiving application is fully suspended. The widget continues displaying the most recently received status until a new update becomes available.

## Project Status

Widget Relay is currently a functional prototype focused on real-time status sharing and native home-screen widget integration.

Potential future improvements include:

- Richer widget customization
- Multiple user connections
- Different widget sizes
- Additional status and message types
- Improved background delivery
- Better personalization
- Expanded platform support

## Contributing

Contributions, bug reports, feature suggestions, and improvements are welcome.

Feel free to fork the repository and contribute to the project.

## License

This project is open source. See the repository license for details.

---

## Widget Relay

### **Send something small, straight to someone's home screen.**

`Send` → `Sync` → `Display`
