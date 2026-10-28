# Project Aria — private Windows/Sideloadly build

This is a **developer-only** configuration procedure for connecting OpenVision to
a Meta Wearables project. It does not replace signing the app on an actual iPhone.

## Why this is necessary

The ordinary `.github/workflows/ci.yml` copies example `Config.xcconfig`
values. Those values let the app compile and launch but cannot register glasses
with Meta. The configured build has to embed real Meta identifiers in its
`Info.plist` through the existing Xcode settings.

**Important:** This repository is currently a public fork. DO NOT build a
configured IPA here. A GitHub Actions secret protects a value in the workflow
interface, but its value will be copied into the app bundle and can be extracted
from the IPA. Artifacts from a public repository should not contain the client
token.

## Safe build sequence

1. Make a **standalone private copy** of this source repository (a private
   repository import/mirror, not a private branch on the public fork).
   Verify its visibility reads **Private** before adding the client token or
   running the configured workflow.
2. In the private repository's **Settings → Secrets and variables → Actions**,
   create these repository **secrets**:
   - `META_APP_ID`: Meta Wearables Developer Center **Project Aria** App ID.
   - `META_CLIENT_TOKEN`: full token, including `AR|<app-id>|<token>`.
3. Create these repository **variables**:
   - `APPLE_TEAM_ID`: the 10-character team ID for the Apple signing identity
     used to install this app. The ID must match the actual installed signature.
   - `IOS_BUNDLE_ID`: the final iOS bundle identifier used by the installed app.
     Verify it is not being changed by Sideloadly's signing process.
   - `APP_LINK_URL_SCHEME`: URL callback scheme, typically `openvision`.
4. In **Meta Wearables Developer Center → Project Aria → Configuration → iOS**,
   enter the **same Apple Team ID** and **same iOS Bundle ID** as the signed app.
   Configure the app callback details according to Meta's current iOS
   integration instructions. The Meta project App ID and client token must
   belong to the same project.
5. In the **private** repository's **Actions** tab, manually run
   **Project Aria - Private iPhone IPA**. Download its
   `Project-Aria-private-unsigned-iphoneos` artifact (one-day retention).
6. Use **Sideloadly on Windows** to sign and install the unsigned IPA on the
   iPhone. Check the actual bundle identifier and signing team because
   re-signing can modify them. If they change, reconcile the values with Meta,
   rebuild as necessary, and reinstall.
7. On the iPhone, keep Meta AI installed, the glasses paired, and Meta AI
   Developer Mode enabled; then try **OpenVision → Settings → Glasses →
   Register with Meta AI**. Physical device testing is required.

## Security and limitations

- Do not commit `Config.xcconfig`, a client token, an OpenAI key, or any
  Apple login credentials.
- Do not attach screenshots displaying the full client token to public issues.
- The workflow explicitly **fails when the repository is public**. Regular
  public CI continues to build with nonfunctional example credentials.
- A token embedded in an IPA is not a secret from anyone who can access that
  IPA. Limit access to the private repo, artifacts, and the resulting binary.
- GitHub Actions does **not** sign the app with Apple credentials.
  Sideloadly's local signing step still determines the installed code signature.
- The supplied workflow does not automatically configure Meta's dashboard,
  create a private repository, or configure GitHub secrets. Those require
  authenticated UI/API access.
- Successful compilation is not proof of glasses registration. Test the
  registration handshake on the physical iPhone.
