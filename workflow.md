Fork maintenance and customization workflow for morphe-patches

Goal
- Always keep this fork synced with upstream at https://github.com/MorpheApp/morphe-patches.git before making changes. First check whether upstream has pushed recently.
- Apply my preferred YouTube and YouTube Music defaults.
- Release from my fork without requiring MorpheApp private secrets.

Repository setup
1. Ensure remotes are configured:
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
1. MORPHE Youtube package name
   - Set MORPHE_YOUTUBE_PACKAGE_NAME to com.google.android.apps.youtube.kids
   
2. External downloader
   - Disable Override download action button. morphe_external_downloader_action_button = false
   - Enable show external download button.
   - Set default package name to com.video.fun.app.

3. Player
   - Enable hide endscreen cards button.
   - Enable hide end screen suggested video button.

4. Video
   - Enable remember video quality changes.
   - Enable remember playback speed changes.
   - Enable advanced video quality menu.
   - Enable remember shorts quality changes.

5. General and navigation
   - Set default app name to custom preset index 2.
   - Set custom name entry 2 to Premium Youtube.
   - Set default header logo to Premium.
   - Set default app icon style to Original.

6. Shorts
   - Enable hide shorts on home feed button by default.

7. Seekbar
   - Enable slide to seek.
   - Enable tap to seek.

8. Feed
   - Disable hide mix playlists button by default.
   - Disable hide 'you may like' section button by default.
   - Disable hide 'notify me' button by default.
   - Disable hide 'show more' button by default.
   - Disable hide latest post button by default.
   - Enable hide YouTube Doodles button by default.

9. Swipe controls
   - Enable swipe to change videos.
   - Enable brightness gesture.
   - Enable volume gesture.
   - Enable auto-brightness gesture.

10. Miscellaneous
   - Disable announcements by default.

11. About
   - Hide the About section inside Morphe/Premium YouTube settings. ("About" preference to the top.)

YouTube Music defaults to enforce
1. YT Music package name
   - Set MORPHE_MUSIC_PACKAGE_NAME to app.revanced.android.apps.youtube.music in Constants.kt and other relevant files.
   - Set targetPackage and morphe_music_package_name to app.revanced.android.apps.youtube.music in OverrideYouTubeMusicActionsPatch.kt and Settings.java respectively.

2. General
   - Set custom branding entry 2 to YT Music Premium.
   - Set default icon style to Original.

Expected files to modify
- BaseSetting.java
- Settings.java
- strings.xml
  - Update UI display text from Morphe to Premium Youtube (settings button labels, import/export text, Android VR note, morphe_settings_title to Extra Settings, morphe_settings_submenu_title to Premium Settings)
- CustomBrandingPatch.java
- BaseCustomBrandingPatch.kt
- Constants.kt
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