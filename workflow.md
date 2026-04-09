Fork maintenance and customization workflow for morphe-patches

Goal
- First fetch upstream and update my code i noticed now they pushed right now.
- Apply my preferred YouTube and YouTube Music defaults.
- Release from my fork without requiring MorpheApp private secrets.

Repository setup
1. Ensure taking pull from remotes:
   - origin: my fork
   - upstream: https://github.com/MorpheApp/morphe-patches.git
2. Sync main:
   - git fetch upstream
   - git checkout main
   - git merge --ff-only upstream/main
   - git push origin main

Global patch behavior
- Disable environment check warning in CheckEnvironmentPatch.java near:
  if (!Check.shouldRun() && !DEBUG_ALWAYS_SHOW_CHECK_FAILED_DIALOG) { ... }
- Use:
  // Environment checks disabled.
  Check.disableForever();
  return;

YouTube defaults to enforce
1. External downloader
   - Enable external downloader.
   - Enable external downloader action button.
   - Set default package name to com.video.fun.app.

2. Player
   - Enable hide endscreen cards.
   - Enable hide end screen suggested video.

3. Video
   - Enable remember video quality changes.
   - Enable remember playback speed changes.
   - Enable advanced video quality menu.
   - Enable remember shorts quality changes.

4. General and navigation
   - Set default app name to custom preset index 2.
   - Set custom name entry 2 to Premium Youtube.
   - Set default header logo to Premium.
   - Set default app icon style to Original.
   - Disable hide shorts button by default.
   - Disable hide mix playlists by default.

5. Swipe controls
   - Enable swipe to change videos.
   - Enable brightness gesture.
   - Enable volume gesture.
   - Enable auto-brightness gesture.

6. Miscellaneous
   - Disable announcements by default.

YouTube Music defaults to enforce
1. General
   - Set custom branding entry 2 to YT Music.
   - Set default icon style to Original.

Expected files to modify
- BaseSetting.java
- Settings.java
- strings.xml
- CustomBrandingPatch.java
- BaseCustomBrandingPatch.kt
- CheckEnvironmentPatch.java

Release process from fork
1. Commit using semantic-release trigger format (example):
   build(Needs bump): set preferred defaults
2. Push to main on fork.
3. GitHub Actions release workflow should:
   - Build patches.
   - Publish release assets using GITHUB_TOKEN.
   - Skip website deploy and FCM steps when related secrets/vars are missing.

Manager usage notes
1. Use this source URL in Morphe Manager:
   https://raw.githubusercontent.com/atanuroy22/morphe-patches/refs/heads/main/patches-bundle.json
2. Ensure the JSON includes:
   - version
   - download_url
   - signature_download_url (or empty if not used by your flow)
   - created_at
   - description
3. Keep that JSON updated for each release.

Important
- Do not run destructive git commands.
- Keep changes scoped to defaults only.
- Update CHANGELOG.md whenever behavior changes.