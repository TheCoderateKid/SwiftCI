[![Jenkins Build Status](https://img.shields.io/jenkins/build?job=YourJenkinsJobName&style=flat-square&logo=jenkins)](https://jenkins.example.com/job/YourJenkinsJobName/)  
[![Swift 5.9+](https://img.shields.io/badge/swift-5.9%2B-orange?style=flat-square&logo=swift)](https://swift.org)  
[![Platforms](https://img.shields.io/badge/platforms-iOS%20%7C%20macOS%20%7C%20tvOS%20%7C%20watchOS-blue?style=flat-square)]()  
[![License: MIT](https://img.shields.io/badge/license-MIT-green?style=flat-square)](LICENSE)  
[![GitHub Release](https://img.shields.io/github/v/release/thecoderatekid/SwiftCI?style=flat-square)](https://github.com/thecoderatekid/SwiftCI/releases)  

# SwiftCI

A **dynamic, cross-platform** Jenkins Declarative Pipeline for Swift projects.  
Automatically detects Swift Package Manager packages, Xcode workspaces, or Xcode projects and runs formatting, linting, building, and testing across **iOS**, **macOS**, **tvOS**, **watchOS**, and **Swift Package** targets.

**Maintainer:** [TheCoderatekid](https://github.com/thecoderatekid)  

---

## 🚀 Why SwiftCI?

Continuous Integration is crucial to maintain code quality, catch regressions early, and enforce consistent style across your team. SwiftCI gives you:

- **Automated Tooling**  
  - On-the-fly installation of **SwiftFormat** & **SwiftLint**  
  - Ensures every build node has the right linters without manual setup

- **Dynamic Project Detection**  
  - **SwiftPM**: builds & tests when your repo has `Package.swift`  
  - **Xcode**: discovers any `*.xcworkspace` or `*.xcodeproj` and all shared schemes

- **Multi-Platform Coverage**  
  - Builds & tests **iOS**, **tvOS**, **watchOS**, and **macOS** targets  
  - Uses simulator destinations for UI & unit tests

- **Enforced Quality Gates**  
  - Style check (lint & format) fails fast before expensive builds  
  - Archiving of build/test logs and JUnit-style reports for easy debugging

---

## 📋 Table of Contents

- [Usage](#usage)  
  - [Prerequisites](#prerequisites)  
  - [Installation](#installation)  
  - [Configuration](#configuration)  
- [Pipeline Stages](#pipeline-stages)  
- [Customizing](#customizing)  
  - [Custom Lint Configuration](#custom-lint-configuration)  
- [Contributing](#contributing)  
- [License](#license)  

---

## 📦 Usage

### Prerequisites

- A **macOS** Jenkins agent (labeled `macos`) with:
  - Xcode & Command Line Tools  
  - Homebrew (will auto-install if missing)  
- Jenkins with the [Pipeline](https://plugins.jenkins.io/workflow-aggregator/) plugin  

### Installation

1. **Create a new repo** named `SwiftCI` (or copy this `Jenkinsfile` into your existing project).
2. Add the `Jenkinsfile` at the repository root.
3. Push to your Git server and configure a Jenkins job pointing at this repo.

### Configuration

Edit the top of your `Jenkinsfile` to adjust:

```groovy
environment {
  SWIFT_VERSION  = '5.9'
  IOS_DEST       = 'platform=iOS Simulator,name=iPhone 14,OS=17.4'
  TVOS_DEST      = 'platform=tvOS Simulator,name=Apple TV 4K (2nd generation),OS=17.4'
  WATCHOS_DEST   = 'platform=watchOS Simulator,name=Apple Watch Series 9 - 45mm,OS=10.4'
  MACOS_SDK      = 'macosx'
}
```

- Change simulator names or OS versions to match your installed simulators.
- Ensure your macOS node has at least one iOS/tvOS/watchOS simulator available.

---

## 🔄 Pipeline Stages

1. **Checkout**  
   Prints the repo URL, branch, and commit, then checks out the code.

2. **Detect Project Type**  
   Sets flags for presence of `Package.swift`, `*.xcworkspace`, or `*.xcodeproj`.

3. **Ensure Tools**  
   Installs Xcode CLI Tools, Homebrew, SwiftFormat, and SwiftLint if they’re missing.

4. **Format & Lint**  
   - `swiftformat . --lint --disable trailingCommas --swiftversion 5.9`  
   - `swiftlint lint --strict`  

5. **SwiftPM Build & Test**  
   Runs `swift build` and `swift test` if `Package.swift` exists.

6. **Xcode Build & Test**  
   - Cleans and builds each shared scheme in any discovered workspace/project  
   - Runs `xcodebuild test` on simulator destinations for iOS, tvOS, watchOS  
   - Skips tests for pure macOS schemes (no simulator)

7. **Post**  
   Archives `*_build.log` and `*_test.log` files, and publishes JUnit reports.

---

## ⚙️ Customizing

- **Additional Schemes**: The pipeline picks up _all_ shared schemes under `xcshareddata/xcschemes`.  
- **Additional Platforms**: Add new `DEST` environment variables and adjust the SDK-selection logic based on scheme naming.  

### Custom Lint Configuration

SwiftCI comes bundled with two fully-commented template files to help you lock in your team’s code style. Drop them in your repo root and **uncomment** or **tweak** the settings you want.  

#### `.swiftformat`

A comment-rich config for [SwiftFormat](https://github.com/nicklockwood/SwiftFormat#options).  
Copy this file to your repo as **`.swiftformat`**, then:

- **Uncomment** any `--option` lines to enable or override defaults.  
- **Disable rules** with `--disable <rule1>,<rule2>` (e.g. `--disable trailingCommas`).  
- **Set version** via `--swiftversion 5.9` to align formatting with your Swift version.

```text
# SwiftFormat configuration (uncomment to enable)
#--maxwidth 100
#--indent 4
#--wraparguments beforefirst
#--wrapelements beforefirst
#--semicolons remove
#--commas insert
#--disable trailingCommas, redundantSelf
--swiftversion 5.9
```

#### `.swiftlint.yml`

A sample [SwiftLint](https://github.com/realm/SwiftLint#rules) YAML config.  
Copy to **`.swiftlint.yml`** at your repo root, then:

- Add rule identifiers under `disabled_rules:` to turn them off.  
- Add rule identifiers under `opt_in_rules:` to enable opt-in checks (e.g. `sorted_imports`).  
- Tweak thresholds (`line_length`, `file_length`, etc.).  
- Define `included:` / `excluded:` paths.  
- Use `custom_rules:` to add your own regex-based checks.

```yaml
disabled_rules:
  # - line_length
  # - force_unwrapping

opt_in_rules:
  # - sorted_imports

included:
  # - Sources

excluded:
  - Carthage
  - Pods
  - .build

line_length:
  warning: 120
  error: 150

reporter: xcode

custom_rules:
  no_print_debug:
    regex: 'print\('
    message: "Remove debug `print(...)` calls."
    severity: warning
```

Drop and customize these two files to enforce consistent formatting and linting across your entire team—no changes to the `Jenkinsfile` required.

---

## 🤝 Contributing

1. Fork this repo  
2. Create a feature branch (`git checkout -b feature/awesome`)  
3. Commit your changes (`git commit -m 'Add awesome feature'`)  
4. Push to the branch (`git push origin feature/awesome`)  
5. Open a Pull Request  

---

## 📄 License

This project is licensed under the **MIT License**. See [LICENSE](LICENSE) for details.  
