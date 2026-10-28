# Project Aria: voice and locked-phone investigation

## What the source establishes

- User-confirmed runtime selection: OpenAI backend, Apple speech, glasses microphone enabled. OpenClaw suspension therefore does not explain this user's failure. The exact OpenAI authentication mode/model remains device-local.
- Apple speech completion and recognition-state transitions previously relied on SwiftUI `onChange`. Those transitions now have retained Combine subscriptions in the session view model, so reply completion can resume listening without a view render. This addresses an app-level background hazard for this configuration; it does not guarantee iOS will keep the microphone/recognizer/session alive.

- Defaults: `AppSettings.aiBackend = .openClaw`, empty gateway URL/token, and `ttsEngine = .appleSystem`. The installed phone's saved settings and Keychain override these. Connection to glasses does not identify the AI backend. Record the selected backend, speech engine, model-ready status and microphone toggle from Settings; never copy keys/tokens into an issue.
- Apple speech now automatically prefers the highest-quality installed en-US voice unless a valid explicit voice is selected. Preview retains its synthesizer while the view is open. Cloud/Kokoro failures can still fall back to Apple (`NeuralSpeech.active` and cloud fallback); a robotic voice does not prove which backend answered.
- `VoiceAgentViewModel.onDisappear` stops speech recognition, including on tab navigation. `OpenClawService.handleAppBackground` transitions usable connections to suspended, then reconnects on foreground. These are explicit app policies, not proof iOS forbids background conversation.
- Info.plist already includes audio, external-accessory and Bluetooth background modes. Adding more flags will not fix these policies. Audio uses playAndRecord with Bluetooth HFP for glasses. HFP's headset audio quality may also affect perceived speech quality.
- Local MLX generation rejects background work. Kokoro also uses MLX; it is not a safe route to promising locked-screen synthesis. Apple Intelligence has foreground guards too.
- Meta DAT is pinned to 0.9.0 in both project definitions. This app uses phone speech recognition for its custom wake phrase. It contains no Meta Voice Invocation integration. Newer DAT capabilities need separate version/permission/device validation; do not promise that Hey Meta will launch Aria or that a custom phrase will wake a suspended/terminated app.

## Feasible next background implementation

Use an explicit, user-started background conversation option, initially for a cloud backend and cloud/Apple speech. Move session ownership and speech callbacks out of view lifecycle hooks; preserve the active microphone only during that opted-in session. Suspend idle connections normally. Handle audio interruptions, route changes, socket recovery and returning to foreground before enabling it by default. Validate on a physical phone; no silent-audio keepalive or blanket GPU-guard removal. Camera streaming behavior must be tested separately on the pinned DAT SDK and Emerald; it is not an unconditional iOS prohibition or guarantee.

This patch improves voice selection/preview, decouples Apple speech and recognition transitions from view rendering, and explains the existing lifecycle behavior. It does not enable an untested always-on mode.

## Device test checklist

1. Record iOS/app build, chosen AI backend, speech engine, microphone toggle, glasses firmware and whether camera capture really works. Keep credentials private. Confirm which commit built the installed IPA; repo defaults cannot reveal runtime settings.
2. With Apple speech selected, compare Automatic to an explicitly selected basic voice. Install an Enhanced/Premium US English voice in iOS Accessibility voice settings, reopen the app and preview again. Explicit selection must stay selected after relaunch. Confirm preview remains audible and stops when leaving its screen.
3. For a natural offline voice, choose Kokoro and download its model over Wi-Fi (roughly 600 MB); verify Ready and test in foreground. For cloud speech, configure the chosen provider privately in app Settings. Network use and provider charges apply. Never embed a key in the public IPA/workflow. Compare the same sentence through the phone speaker and Emerald to distinguish synthesis from Bluetooth quality.
4. Start a foreground conversation and ask two questions, including one involving a glasses photo. Confirm complete replies, subsequent listening and Stop. Live Gemini/OpenAI Realtime have their own audio voices; the text-mode speech picker does not necessarily control them.
5. Repeat separately: switch to another app, lock for 30 seconds, lock for two minutes, then return. Note whether playback, listening, AI replies or camera fail independently. For OpenClaw, background suspension is expected in this version; verify foreground reconnection. For local AI, foreground is required. Switching to Settings stops recognition; returning should resume it.
6. Test phone call interruption, Bluetooth disconnect/reconnect and Wi-Fi loss/recovery. Do not regard compilation as passing these tests. Force-quitting is a separate case and must not be expected to leave an always-listening service running.

## Build and configuration safety

Public CI builds app plus test bundles with placeholder Meta config, signing disabled, then packages an unsigned IPA. `build-for-testing` compiles tests; it does not execute them. Windows cannot run Xcode or device tests. A green public CI build proves compilation/packaging, not configured registration, audio quality or background operation. Preserve the real Meta configuration/signing path for an installed update; public placeholder IPA installation can lose working registration. No provider credentials are needed for this patch.

## Platform references

- Apple background recording: https://developer.apple.com/documentation/avfaudio/avaudiosession/category-swift.struct/record
- Apple Metal background restrictions: https://developer.apple.com/documentation/metal/preparing-your-metal-app-to-run-in-the-background
- Meta iOS toolkit and version-specific examples: https://github.com/facebook/meta-wearables-dat-ios
- Meta locked-phone discussion (SDK/version-dependent evidence, not a guarantee): https://github.com/facebook/meta-wearables-dat-ios/discussions/13
