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
   - git fetch upstream --tags
   - git fetch origin --tags
   - git checkout main
   - git merge --ff-only origin/main   (pick up the last `chore: Release` commit made by CI)
   - git merge upstream/main           (a plain merge is required; --ff-only fails because the fork carries its own commits)
   - git push origin main
3. Push upstream's new stable tags to the fork BEFORE pushing main:
   - git tag --merged main | grep -v -- "-dev" | sort -V   (compare against `git ls-remote --tags origin`)
   - git push origin vX.Y.Z ...
   - Why: semantic-release derives the next version from the newest stable tag present on origin.
     Without upstream's tags the fork would re-release an already-used upstream version number and
     the release notes would replay every upstream commit since the fork's last tag.
   - Never push the `-dev` prerelease tags; the fork only releases from `main`.

Recurring merge conflicts (and how they are resolved)
- `.github/workflows/release.yml` — keep the fork's `github.repository == 'MorpheApp/morphe-patches'` +
  `env.<SECRET> != ''` guards, take upstream's action version bumps.
- `CHANGELOG.md` — take upstream's side, but keep the fork's own most recent `## [x.y.z]` section
  (upstream does not have it) and place it in date order below upstream's new sections. Then prepend
  a fresh `# Unreleased` section describing the fork changes; semantic-release prepends the generated
  notes above it, and a follow-up `chore: Fold fork notes ... [skip ci]` commit merges the two.
- `README.md`, `patches-list.json`, `patches-bundle.json` — take upstream's side; all three are
  regenerated during the release.
- `gradle.properties` — take upstream's `version`; `gradle-semantic-release-plugin` bumps it anyway.
- Source/string conflicts — keep upstream's identifiers and structure, re-apply only the fork's
  default values and display text. Example: upstream renamed
  `morphe_external_downloader_flyout_button` to `morphe_external_downloader_flyout_menu`; the fork
  keeps the new name and only re-applies `morphe_external_downloader = TRUE` and the
  `com.video.fun.app` package default.
- `.releaserc` vs `.releaserc.js` — upstream ships `.releaserc` (JSON); the fork **deleted** it and
  ships `.releaserc.js` instead so the backmerge plugin can be added conditionally. A plain merge
  keeps the deletion silently as long as upstream leaves `.releaserc` untouched. If upstream ever
  edits `.releaserc`, git raises a modify/delete conflict: keep the deletion (`git rm .releaserc`)
  and port the upstream change into `.releaserc.js` by hand. cosmiconfig prefers `.releaserc` over
  `.releaserc.js`, so the two files must never exist at the same time.

Local build verification
- `./gradlew build -PnoProguard` needs a token with the `read:packages` scope to resolve
  `app.morphe.patches` from `https://maven.pkg.github.com/MorpheApp/registry`:
  `export GITHUB_ACTOR=atanuroy22; export GITHUB_TOKEN=<PAT with read:packages>`
- The default `gh auth token` does not carry `read:packages`, so a local build fails at
  `settings.gradle.kts` plugin resolution. CI builds fine with its own `GITHUB_TOKEN`;
  when no scoped PAT is available, rely on the release workflow run for build verification.

Global patch behavior
- Disable environment check warning at the top of `check(Activity context)` in
  `extensions/shared/library/src/main/java/app/morphe/extension/shared/checks/CheckEnvironmentPatch.java`
  (note the path: `extensions/shared/`, not `extensions/shared-youtube/`):
  // Environment checks disabled.
  Check.disableForever();
  if (context != null) {
      return;
  }
- As of upstream v1.41.0 the nag screen is also permanently disabled upstream
  ("FIXME: Nag screen is permanently turned off"), so the fork edit is now belt-and-braces.
  Keep it anyway - it costs nothing and survives if upstream re-enables the dialog.

YouTube defaults to enforce
1. MORPHE Youtube package name
   - Set MORPHE_YOUTUBE_PACKAGE_NAME to com.google.android.apps.youtube.kids in morphe-patches\patches\src\main\kotlin\app\morphe\patches\youtube\misc\gms\Constants.kt
   
2. External downloader
   - Disable Override download action button. morphe_external_downloader_action_button = false
   - Enable show external download button.(morphe_external_downloader = true)
   - Set default package name to com.video.fun.app.

3. Player
   - Enable hide endscreen cards button.(morphe_hide_end_screen_cards = true)
   - Enable hide end screen suggested video button.(morphe_hide_end_screen_suggested_video = true)

4. Video
   - Enable remember video quality changes.(morphe_remember_video_quality_last_selected = true)
   - Enable remember playback speed changes.(morphe_remember_playback_speed_last_selected = true)
   - Enable advanced video quality menu.(morphe_advanced_video_quality_menu = true)
   - Enable remember shorts quality changes.(morphe_remember_shorts_quality_last_selected = true)

5. General and navigation
   - Set default app name to custom preset index 2.
   - Set custom name entry 2 to Premium Youtube.
   - Set default header logo to Premium.(morphe_header_logo = HeaderLogo.PREMIUM)
   - Set default app icon style to Original.

6. Shorts
   - Enable hide shorts on home feed button by default.(morphe_hide_shorts_home = true)

7. Seekbar
   - Enable slide to seek.(morphe_slide_to_seek = true)
   - Enable tap to seek.(morphe_tap_to_seek = true)

8. Feed
   - Disable hide mix playlists button by default.
   - Disable hide 'you may like' section button by default.(morphe_hide_you_may_like_section = false)
   - Disable hide 'notify me' button by default.(morphe_hide_notify_me_button = false)
   - Disable hide 'show more' button by default.(morphe_hide_show_more_button = false)
   - (Obsolete) `morphe_hide_latest_posts` no longer exists upstream - the setting was removed,
     so there is nothing to enforce. Kept here only so the item is not re-added by mistake.
   - Enable hide YouTube Doodles button by default.(morphe_hide_youtube_doodles = true)

9. Swipe controls
   - Enable swipe to change videos.(morphe_swipe_change_video = true)
   - Enable brightness gesture.(morphe_swipe_left_zone = SwipeZoneAction.BRIGHTNESS)
   - Enable volume gesture.(morphe_swipe_right_zone = SwipeZoneAction.VOLUME)
   - Enable auto-brightness gesture.(morphe_swipe_lowest_value_enable_auto_brightness = true)
   - Note: upstream v1.40.0 replaced the `morphe_swipe_brightness` / `morphe_swipe_volume` /
     `morphe_swipe_speed` booleans with the three zone enums `morphe_swipe_left_zone` /
     `morphe_swipe_right_zone` / `morphe_swipe_top_zone` (`SwipeZoneAction` = OFF | VOLUME |
     BRIGHTNESS | SPEED). Leave the top zone OFF.

10. Miscellaneous
   - Disable announcements by default.(morphe_announcements = false)

11. Settings entry name (upstream v1.41.0)
   - Upstream added `morphe_settings_name` (SharedSettings.SETTINGS_NAME, default "DEFAULT") plus the
     `morphe_settings_name_*` strings, letting the user rename the settings entry at runtime.
   - Leave the default at "DEFAULT": it falls back to `morphe_settings_title`, which the fork already
     overrides to `Extra Settings`. No fork change is needed for this feature.

12. About
   - Hide the About section inside Premium Settings/Extra settings. ("About" button located above "Ads" button.i want to hide the button because it is not useful for me.)
   - Implemented by deleting the `preferences += NonInteractivePreference("morphe_settings_screen_00_about", ...)`
     block (and its now-unused `NonInteractivePreference` import) from
     `patches/src/main/kotlin/app/morphe/patches/youtube/misc/settings/SettingsPatch.kt`.
     Leave the `morphe_settings_screen_00_about*` drawables in the `copyResources` list; they are
     harmless and removing them causes needless conflicts on every upstream merge.

YouTube Music defaults to enforce
1. YT Music package name
   - Set targetPackage and morphe_custom_music_package_name to app.revanced.android.apps.youtube.music in morphe-patches\patches\src\main\kotlin\app\morphe\patches\music\misc\gms\Constants.kt  and morphe-patches\extensions\youtube\src\main\java\app\morphe\extension\youtube\settings\Settings.java respectively.
   - Note: upstream v1.40.0 renamed `morphe_music_package_name` (MORPHE_MUSIC_PACKAGE_NAME) to
     `morphe_custom_music_package_name` (CUSTOM_MUSIC_PACKAGE_NAME) and changed its default to "".
     The old key survives only as DEPRECATED_MORPHE_MUSIC_PACKAGE_NAME for migration.

2. General
   - Set custom branding entry 2 to YT Music Premium.
   - Set default icon style to Original.

Crowdin-pull-push workflow
1. Disable pull and push request completely in yml file.(crowdin_pull.yml, crowdin_push.yml)

Expected files to modify
- SharedYouTubeSettings.java
- Settings.java
- strings.xml
  - Update UI display text from Morphe to Premium Youtube (settings button labels, import/export text, Android VR note, morphe_settings_title to Extra Settings, morphe_settings_submenu_title to Premium Settings)
- CustomBrandingPatch.java (extensions/shared-youtube/.../patches/CustomBrandingPatch.java -
  getDefaultAppNameIndex() = 2 and getDefaultIconStyle() = BrandingTheme.ORIGINAL; this one file
  covers both YouTube and YouTube Music, so BaseCustomBrandingPatch.kt needs no fork edit any more)
- Constants.kt
- CheckEnvironmentPatch.java (extensions/shared/library/...)
- SettingsPatch.kt (YouTube - remove the About preference)
- .releaserc.js

Release process from fork
1. Commit using semantic-release trigger format (example):
   build(Needs bump): set preferred defaults
2. Push to main on fork.
3. GitHub Actions release workflow should:
   - Build patches.
   - Publish release assets using GITHUB_TOKEN.
   - Fetch tags before semantic-release so the release notes and version detection see the full tag history.
   - Skip website deploy and FCM steps when related secrets/vars are missing.
   - Skip the `@cleyrop-org/semantic-release-backmerge` plugin (configured dynamically in `.releaserc.js`) to avoid git merge conflicts with the outdated local `dev` branch.
4. Wait and fix until a successful release workflow run completes.

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
- Document and update this workflow.md file when necessary to keep the fork maintenance procedures up to date. 

**NOW COMPLETE THE RELEASE PROCESS AND DONT STOP UNTIL A SUCCESSFUL RELEASE IS COMPLETED ,WAIT FOR NEW STABLE RELEASE IF GOT ANY PROBLEM FIX IT AND WAIT FOR FINAL RELASE AND PATCHES-BUNDLE.JSON IS UPDATED WITH THE NEW RELEASE**

**Do not delete this workflow.md file. If deleted on marge or any other case then restore this file after marge-Because this file is very important to me and it makes the workflow process easy and automate**