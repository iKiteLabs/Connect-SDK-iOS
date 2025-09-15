# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Overview

Connect SDK is an iOS framework that enables mobile apps to connect with multiple TV platforms (LG WebOS, Roku, Chromecast, Apple TV, etc.) through various protocols (DLNA, DIAL, AirPlay, Google Cast, etc.). The SDK abstracts the complexity of different TV protocols into a unified API.

## Build Commands

### Building the framework
```bash
# Build for simulator
xcodebuild -scheme ConnectSDK -configuration Debug -sdk iphonesimulator build

# Build for device
xcodebuild -scheme ConnectSDK -configuration Release -sdk iphoneos build

# Build framework target
xcodebuild -scheme Framework -configuration Release build
```

### Running tests
```bash
# Unit tests (fast, no network)
xcodebuild -scheme ConnectSDKTests -sdk iphonesimulator test

# Integration tests (fast, no network)
xcodebuild -scheme ConnectSDKIntegrationTests -sdk iphonesimulator test

# Acceptance tests (slow, requires real devices on network)
xcodebuild -scheme ConnectSDKAcceptanceTests -sdk iphonesimulator test
```

### Installing dependencies
```bash
# Initialize submodules (required for first-time setup)
git submodule update --init

# For CocoaPods users
pod install
```

## Architecture

### Core Components

**Discovery System** (`core/Discovery/`)
- `DiscoveryManager`: Central manager for device discovery across all protocols
- `SSDPDiscoveryProvider`: Discovers DLNA, DIAL, and Roku devices via SSDP
- `ZeroConfDiscoveryProvider`: Discovers AirPlay and Chromecast devices via mDNS

**Service Layer** (`core/Services/`)
- `DeviceService`: Base class for all TV platform services
- Platform-specific services:
  - `WebOSTVService`: LG WebOS TVs (uses SSAP protocol)
  - `DLNAService`: DLNA-compatible devices
  - `DIALService`: DIAL protocol (YouTube, Netflix launching)
  - `AirPlayService`: Apple TV support
  - `RokuService`: Roku devices (ECG protocol)
  - `NetcastTVService`: Legacy LG TVs

**Device Management** (`core/Device/`)
- `ConnectableDevice`: Represents a discovered TV, manages its services
- `DevicePicker`: UI component for device selection

**Capability System** (`core/Services/Capabilities/`)
- Protocol interfaces defining TV capabilities:
  - `MediaPlayer`: Video/audio playback
  - `Launcher`: App launching
  - `TVControl`: Channel control
  - `VolumeControl`: Volume adjustment
  - `MediaControl`: Play/pause/seek
  - `KeyControl`: Remote control keys
  - `MouseControl`: Cursor control
  - `TextInputControl`: Keyboard input
  - `ToastControl`: Toast notifications
  - `WebAppLauncher`: Web app support

### Module System

External modules in `modules/`:
- `google-cast/`: Google Cast SDK integration
- `firetv/`: Amazon Fire TV support (requires Amazon Fling SDK)

### Protocol Stack

The SDK intelligently selects protocols based on device and capability:
- **Media Playback**: DLNA, Google Cast, AirPlay, or proprietary protocols
- **App Launching**: DIAL for cross-platform apps, native protocols for platform-specific
- **System Control**: UDAP (LG), ECG (Roku), or device-specific protocols
- **Discovery**: SSDP for DLNA/DIAL devices, mDNS for AirPlay/Chromecast

### Key Design Patterns

1. **Service Abstraction**: Each TV platform has a service class implementing capability protocols
2. **Automatic Protocol Selection**: SDK chooses optimal protocol per feature
3. **Unified Discovery**: Single device list from multiple discovery protocols
4. **Command Pattern**: `ServiceCommand` for async operations with callbacks

## Testing Approach

- **Unit Tests**: Test individual components with mocks (OCMock, OHHTTPStubs)
- **Integration Tests**: Test SDK behavior without network (Specta, Expecta)
- **Acceptance Tests**: End-to-end tests with real devices

## Important Configuration

- Requires ARC (Automatic Reference Counting)
- Link with: `libz.dylib`, `libicucore.dylib`
- Other Linker Flags: `-ObjC`
- Minimum iOS: 11.0
- Logging: Enable by uncommenting `#define CONNECT_SDK_ENABLE_LOG` in prefix header

## Required SDKs

### Google Cast SDK (Required for Chromecast support)
```bash
# Download Google Cast SDK 2.7.1
curl -L -o cast.zip https://redirector.gvt1.com/edgedl/chromecast/sdk/ios/GoogleCastSDK-2.7.1-Release-ios-default.zip
unzip cast.zip 'GoogleCastSDK-2.7.1-Release/GoogleCast.framework/*'
mv GoogleCastSDK-2.7.1-Release/GoogleCast.framework modules/google-cast/
rm -rf cast.zip GoogleCastSDK-2.7.1-Release/
```

### Amazon Fling SDK (Optional for Fire TV support)
- No longer publicly available as of 2024
- Fire TV support can be disabled by commenting out FireTVService in ConnectSDKDefaultPlatforms.h
- To fully exclude FireTV from build, remove FireTV*.m files from Xcode project targets