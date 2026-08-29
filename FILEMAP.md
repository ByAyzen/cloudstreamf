# CloudStream (Fork PR) — Comprehensive File Map & AI Agent Evaluation Guide

> **Target Audience:** Autonomous AI Agents, Code Reviewers, and Developers  
> **Repository Root:** `cloudstreamforkPR/`  
> **Architecture Summary:** Kotlin Multiplatform (KMP) Core Library (`:library`) + Android Client with Phone/TV Leanback Support (`:app`) + Documentation Generator (`:docs`).

---

## 1. Architectural Overview & AI Agent Navigation Rules

CloudStream is a modular, provider-driven media streaming and aggregation application for Android (supporting mobile, tablet, and Android TV / Google TV).

### Key Architectural Layers:
1. **Core Library (`:library`)**: Pure Kotlin / KMP multiplatform module containing data models (`MainAPI`, `Episode`, `SearchResponse`), video extractor parsers (50+ streaming host extractors), cryptographic unpackers, metadata resolvers (TMDB/Trakt), and networking abstractions.
2. **App Core & Platform Services (`:app`)**: Android-specific lifecycle (`CloudStreamApp`, `MainActivity`), background download engines (`DownloadManager`), media player orchestration (`CS3IPlayer`, ExoPlayer customization), subtitles pipeline, plugin classloader (`PluginManager`), and Android TV Leanback integration.
3. **UI / Presentation Layer (`app/src/main/java/.../ui`)**: MVVM / MVI architecture handling Home, Search, Result/Details, Fullscreen Video Player, Downloads, User Library, and Settings.
4. **Plugin & Extension System (`plugins/`)**: Dynamic runtime loading of external scrapers/plugins packaged as `.cs3` files via DexClassLoader.

### AI Agent Evaluation Guidelines:
- **Provider / Extractor Issues:** Always inspect `library/src/commonMain/kotlin/com/lagradost/cloudstream3/extractors/` or `helper/`.
- **Playback / Player Issues:** Check `app/.../ui/player/`, `CS3IPlayer.kt`, `LinkGenerator.kt`, or `source_priority/`.
- **Download Engine Bugs:** Check `app/.../utils/downloader/` and `app/.../services/VideoDownloadService.kt`.
- **Sync / Account Integrations:** Check `app/.../syncproviders/` and `syncproviders/providers/`.
- **Plugin Loading & Repositories:** Check `app/.../plugins/PluginManager.kt` and `RepositoryManager.kt`.
- **Compose / Multiplatform Migration:** Consult `COMPOSE.md` and keep all new data logic in `:library` (KMP compatible).

---

## 2. Root Project & Build Configuration

| File Name | Location | Purpose & AI Evaluation Scope |
| :--- | :--- | :--- |
| `settings.gradle.kts` | `/settings.gradle.kts` | Declares repository management and includes subprojects (`:app`, `:library`, `:docs`). |
| `build.gradle.kts` | `/build.gradle.kts` | Root Gradle build configuration, plugin definitions (Android, Kotlin Multiplatform, Dokka, Serialization). |
| `gradle.properties` | `/gradle.properties` | JVM arguments, AndroidX flags, Kotlin daemon memory allocation, and build features. |
| `gradle/libs.versions.toml` | `/gradle/libs.versions.toml` | Version Catalog managing all project dependencies, Kotlin/Android Gradle Plugin versions, ExoPlayer, OkHttp, etc. |
| `jitpack.yml` | `/jitpack.yml` | CI configuration for JitPack builds (JDK version, build command for `:library`). |
| `discoverium.yml` | `/discoverium.yml` | Discoverium configuration for provider/repo indexing and discovery. |
| `AI-POLICY.md` | `/AI-POLICY.md` | Policy and quality requirements for AI-generated code contributions and PRs. |
| `COMPOSE.md` | `/COMPOSE.md` | Architecture roadmap detailing the migration from MVVM to MVI/Jetpack Compose and KMP compatibility. |
| `README.md` | `/README.md` | Main repository overview, feature list, installation guide, and contribution guidelines. |
| `LICENSE` | `/LICENSE` | GPL-3.0 open source license terms. |
| `.gitignore` | `/.gitignore` | Root git ignore rules (build files, IDE configs, local properties). |

### CI/CD & GitHub Configuration (`.github/`)

| File Name | Location | Purpose |
| :--- | :--- | :--- |
| `prerelease.yml` | `.github/workflows/prerelease.yml` | GitHub Actions workflow for automatic pre-release APK builds on master push. |
| `pull_request.yml` | `.github/workflows/pull_request.yml` | Workflow triggered on PRs to run linting, static analysis, and test suites. |
| `build_to_archive.yml` | `.github/workflows/build_to_archive.yml` | Automated build archiving workflow for artifact preservation. |
| `instrumented-tests.yml` | `.github/workflows/instrumented-tests.yml` | Android emulator instrumented UI test execution workflow. |
| `generate_dokka.yml` | `.github/workflows/generate_dokka.yml` | Generates API documentation via Dokka and publishes to GitHub Pages. |
| `update_locales.yml` | `.github/workflows/update_locales.yml` | Automated workflow to synchronize and validate translation strings. |
| `locales.py` | `.github/locales.py` | Python utility script for parsing, sorting, and verifying language localization files. |
| `application-bug.yml` | `.github/ISSUE_TEMPLATE/application-bug.yml` | Structured GitHub issue template for reporting app bugs. |
| `feature-request.yml` | `.github/ISSUE_TEMPLATE/feature-request.yml` | Structured GitHub issue template for proposing new features. |
| `config.yml` | `.github/ISSUE_TEMPLATE/config.yml` | GitHub Issue forms global configuration. |

---

## 3. Module: `:docs` (Documentation Generator)

| File Name | Location | Purpose |
| :--- | :--- | :--- |
| `build.gradle.kts` | `docs/build.gradle.kts` | Configures Dokka multi-module documentation aggregation for the `:library` API. |

---

## 4. Module: `:library` (Kotlin Multiplatform Core & Scraper API)

This module contains the cross-platform interfaces, data contracts, video link extractors, and decoding helpers.

### 4.1 Core Provider API & Models (`library/src/commonMain/kotlin/com/lagradost/cloudstream3/`)

| File Name | Location | Purpose |
| :--- | :--- | :--- |
| `MainAPI.kt` | `library/.../cloudstream3/MainAPI.kt` | **Core Architecture Contract**: Abstract base class `MainAPI` that all media providers implement (search, load, loadLinks, mainPage). Also contains foundational models: `SearchResponse`, `TvType`, `Episode`, `DubStatus`, `LoadResponse`, `MovieLoadResponse`, `TvSeriesLoadResponse`, `AnimeLoadResponse`, `TorrentLoadResponse`, `LiveStreamLoadResponse`, `ExtractorLink`, `SubtitleFile`. |
| `MainActivity.kt` | `library/.../cloudstream3/MainActivity.kt` | Cross-platform compatibility definitions and activity level abstractions for library usage. |
| `ParCollections.kt` | `library/.../cloudstream3/ParCollections.kt` | Asynchronous parallel collection utilities (`amap`, `apmap`, `amapIndexed`) for concurrent network fetching. |

### 4.2 Logging & Context Abstractions (`library/.../com/lagradost/api/`)

| File Name | Location | Purpose |
| :--- | :--- | :--- |
| `Context.kt` | `library/.../api/Context.kt` | Abstract multiplatform Context interface for KMP operations. |
| `ContextHelper.kt` | `library/.../api/ContextHelper.kt` | Expect/actual declarations for platform context access across JVM, Android, and Web. |
| `Log.kt` | `library/.../api/Log.kt` | Multiplatform logging wrapper (`Log.d`, `Log.i`, `Log.e`, `Log.w`) compatible with Android Logcat and standard console. |

### 4.3 Video Stream Extractors (`library/.../cloudstream3/extractors/`)

All classes in this directory implement `ExtractorApi` to resolve raw video URLs (m3u8, mp4) from third-party streaming hosts.

| File Name | Location | Purpose & Extracted Host |
| :--- | :--- | :--- |
| `DoodExtractor.kt` | `library/.../extractors/DoodExtractor.kt` | Extracts video streams from DoodStream (`dood.to`, `doodstream.com`, `dood.wf`). |
| `StreamTape.kt` | `library/.../extractors/StreamTape.kt` | Resolves direct video streams from StreamTape (`streamtape.com`, `streamtape.net`). |
| `StreamWishExtractor.kt` | `library/.../extractors/StreamWishExtractor.kt` | Resolves unpacked streams from StreamWish (`streamwish.to`, `wishembed`). |
| `Vidmoly.kt` | `library/.../extractors/Vidmoly.kt` | Extracts HLS/m3u8 sources from Vidmoly (`vidmoly.me`, `vidmoly.to`). |
| `Voe.kt` | `library/.../extractors/Voe.kt` | Handles Base64/JSON encoded payloads on VOE (`voe.sx`, `tubesquid`). |
| `MixDrop.kt` | `library/.../extractors/MixDrop.kt` | Unpacks obfuscated JavaScript (`p,a,c,k,e,d`) on MixDrop (`mixdrop.co`, `mixdrop.to`). |
| `Mp4Upload.kt` | `library/.../extractors/Mp4Upload.kt` | Extracts direct MP4 streams from Mp4Upload. |
| `Supervideo.kt` | `library/.../extractors/Supervideo.kt` | Resolves Supervideo streaming links. |
| `VidhideExtractor.kt` | `library/.../extractors/VidhideExtractor.kt` | Resolves streams from Vidhide and associated domains. |
| `VidHidePro.kt` | `library/.../extractors/VidHidePro.kt` | Variant extractor for VidHidePro instances. |
| `VidMoxyExtractor.kt` | `library/.../extractors/VidMoxyExtractor.kt` | Extracts streams from VidMoxy. |
| `Vidoza.kt` | `library/.../extractors/Vidoza.kt` | Extracts direct media files from Vidoza (`vidoza.net`). |
| `YoutubeExtractor.kt` | `library/.../extractors/YoutubeExtractor.kt` | Multiplatform YouTube video and adaptive audio stream extractor. |
| `OkRu.kt` | `library/.../extractors/OkRu.kt` | Resolves Odnoklassniki (OK.ru) embedded video links and metadata. |
| `Filemoon.kt` | `library/.../extractors/Filemoon.kt` | Handles Filemoon stream unpacking. |
| `Fastream.kt` | `library/.../extractors/Fastream.kt` | Fastream video link resolver. |
| `Luluvdo.kt` | `library/.../extractors/Luluvdo.kt` | LuluStream / Luluvdo extractor. |
| `UpstreamExtractor.kt` | `library/.../extractors/UpstreamExtractor.kt` | Upstream (`upstream.to`) JS unpacker and stream parser. |
| `Uqload.kt` | `library/.../extractors/Uqload.kt` | Resolves Uqload direct video links. |
| `Userload.kt` | `library/.../extractors/Userload.kt` | Userload stream extractor. |
| `Userscloud.kt` | `library/.../extractors/Userscloud.kt` | Userscloud direct download link resolver. |
| `Videa.kt` | `library/.../extractors/Videa.kt` | Hungarian Videa portal extractor. |
| `VideoSeyredExtractor.kt` | `library/.../extractors/VideoSeyredExtractor.kt` | Turkish VideoSeyred stream resolver. |
| `VkExtractor.kt` | `library/.../extractors/VkExtractor.kt` | VKontakte (VK) embedded player stream extractor. |
| `XStreamCdn.kt` | `library/.../extractors/XStreamCdn.kt` | XStreamCdn / FPlayer extractor. |
| `YourUpload.kt` | `library/.../extractors/YourUpload.kt` | YourUpload stream extractor. |
| `Zplayer.kt` | `library/.../extractors/Zplayer.kt` | Zplayer resolver. |
| `BloggerExtractor.kt` | `library/.../extractors/BloggerExtractor.kt` | Extracts Google Blogspot / Blogger embedded video sources. |
| `CineGrab.kt` | `library/.../extractors/CineGrab.kt` | CineGrab stream resolver. |
| `DailyMotion.kt` | `library/.../extractors/DailyMotion.kt` | Dailymotion HLS stream extractor. |
| `DropLoad.kt` | `library/.../extractors/DropLoad.kt` | DropLoad host extractor. |
| `Embeturbo.kt` | `library/.../extractors/Embeturbo.kt` | Embeturbo stream resolver. |
| `Evload.kt` | `library/.../extractors/Evload.kt` | Evload video extractor. |
| `FileLions.kt` | `library/.../extractors/FileLions.kt` | FileLions stream resolver. |
| `Filesim.kt` | `library/.../extractors/Filesim.kt` | Filesim stream extractor. |
| `Gofile.kt` | `library/.../extractors/Gofile.kt` | Gofile.io API direct link fetcher. |
| `GdrivePlayer.kt` | `library/.../extractors/GdrivePlayer.kt` | Google Drive player proxy extractor. |
| `Guardia.kt` | `library/.../extractors/Guardia.kt` | Guardia video host resolver. |
| `Krakenfiles.kt` | `library/.../extractors/Krakenfiles.kt` | Krakenfiles direct media URL downloader/resolver. |
| `Maxstream.kt` | `library/.../extractors/Maxstream.kt` | Maxstream link extractor. |
| `Mediafire.kt` | `library/.../extractors/Mediafire.kt` | Mediafire direct download file resolver. |
| `Playm4u.kt` | `library/.../extractors/Playm4u.kt` | Playm4u extractor. |
| `Popplayer.kt` | `library/.../extractors/Popplayer.kt` | Popplayer resolver. |
| `RapidCloud.kt` | `library/.../extractors/RapidCloud.kt` | RapidCloud / Megacloud decryptor & stream resolver. |
| `Rentry.kt` | `library/.../extractors/Rentry.kt` | Rentry markdown pastebin parser for dynamic link scraping. |
| `Sendvid.kt` | `library/.../extractors/Sendvid.kt` | Sendvid direct stream resolver. |
| `SlMaxed.kt` | `library/.../extractors/SlMaxed.kt` | SlMaxed video resolver. |
| `SolidFiles.kt` | `library/.../extractors/SolidFiles.kt` | SolidFiles download URL extractor. |
| `StreamEmbed.kt` | `library/.../extractors/StreamEmbed.kt` | Generic StreamEmbed parser. |
| `Streamhub.kt` | `library/.../extractors/Streamhub.kt` | Streamhub stream extractor. |
| `Streamlare.kt` | `library/.../extractors/Streamlare.kt` | Streamlare API resolver. |
| `StreamSB.kt` | `library/.../extractors/StreamSB.kt` | StreamSB / WatchSB / SBEmbed decryptor. |
| `StreamSilk.kt` | `library/.../extractors/StreamSilk.kt` | StreamSilk host extractor. |
| `TauVideoExtractor.kt` | `library/.../extractors/TauVideoExtractor.kt` | TauVideo link parser. |
| `TRsTXExtractor.kt` | `library/.../extractors/TRsTXExtractor.kt` | TRsTX stream resolver. |
| `Up4Stream.kt` | `library/.../extractors/Up4Stream.kt` | Up4Stream host extractor. |
| `Vicloud.kt` | `library/.../extractors/Vicloud.kt` | Vicloud video extractor. |
| `Vidara.kt` | `library/.../extractors/Vidara.kt` | Vidara stream parser. |
| `Vidsonic.kt` | `library/.../extractors/Vidsonic.kt` | Vidsonic link extractor. |
| `VidStack.kt` | `library/.../extractors/VidStack.kt` | VidStack player stream resolver. |
| `Vinovo.kt` | `library/.../extractors/Vinovo.kt` | Vinovo host resolver. |
| `Vtbe.kt` | `library/.../extractors/Vtbe.kt` | Vtbe video extractor. |
| `Wibufile.kt` | `library/.../extractors/Wibufile.kt` | Wibufile host extractor. |

### 4.4 Decryption & Unpacking Helpers (`library/.../extractors/helper/`)

| File Name | Location | Purpose |
| :--- | :--- | :--- |
| `AesHelper.kt` | `library/.../extractors/helper/AesHelper.kt` | AES-CBC/AES-GCM encryption & decryption routines with PKCS7 padding for obfuscated streams. |
| `CryptoJSHelper.kt` | `library/.../extractors/helper/CryptoJSHelper.kt` | Re-implementation of CryptoJS OpenSSL-compatible password decryption (`CryptoJS.AES.decrypt`). |
| `AsianEmbedHelper.kt` | `library/.../extractors/helper/AsianEmbedHelper.kt` | Decryption keys and cipher helpers for AsianEmbed / DramaCool embeds. |
| `GogoHelper.kt` | `library/.../extractors/helper/GogoHelper.kt` | Crypto keys, IVs, and encryption handlers for GogoAnime / Vidstreaming servers. |
| `JWPlayerHelper.kt` | `library/.../extractors/helper/JWPlayerHelper.kt` | Parses JWPlayer JS configurations and embedded JSON source models. |
| `NineAnimeHelper.kt` | `library/.../extractors/helper/NineAnimeHelper.kt` | Vrf token generators and decoders for 9anime / AniWave scrapers. |
| `VstreamhubHelper.kt` | `library/.../extractors/helper/VstreamhubHelper.kt` | Vstreamhub cipher and request payload helper. |
| `WcoHelper.kt` | `library/.../extractors/helper/WcoHelper.kt` | WCO / WatchCartoonOnline cipher decryption functions. |

### 4.5 Meta Providers (`library/.../cloudstream3/metaproviders/`)

| File Name | Location | Purpose |
| :--- | :--- | :--- |
| `TmdbProvider.kt` | `library/.../metaproviders/TmdbProvider.kt` | The Movie Database (TMDB) API integration for metadata, posters, backdrops, and trending items. |
| `TraktProvider.kt` | `library/.../metaproviders/TraktProvider.kt` | Trakt.tv metadata and list syncing provider. |
| `MyDramaList.kt` | `library/.../metaproviders/MyDramaList.kt` | Asian drama metadata scraper and search indexer. |
| `CrossTmdbProvider.kt` | `library/.../metaproviders/CrossTmdbProvider.kt` | Intersects TMDB metadata with provider load responses to fill missing metadata. |
| `SyncRedirector.kt` | `library/.../metaproviders/SyncRedirector.kt` | Directs requests between sync metadata services and media providers. |

### 4.6 Plugin Base & Sync Interfaces (`library/.../plugins/`, `syncproviders/`, `network/`)

| File Name | Location | Purpose |
| :--- | :--- | :--- |
| `BasePlugin.kt` | `library/.../plugins/BasePlugin.kt` | Base plugin class defining `load()` lifecycle and plugin metadata. |
| `CloudstreamPlugin.kt` | `library/.../plugins/CloudstreamPlugin.kt` | Annotation `@CloudstreamPlugin` used to mark entrypoint plugin definitions. |
| `SyncAPI.kt` | `library/.../syncproviders/SyncAPI.kt` | Interfaces for sync services (search, score, status, watch progress). |
| `WebViewResolver.kt` | `library/.../network/WebViewResolver.kt` | Headless / platform webview request interceptor for Cloudflare clearance and JS rendering. |
| `ArchComponentExt.kt` | `library/.../mvvm/ArchComponentExt.kt` | Multiplatform reactive state holders and coroutine dispatchers. |

### 4.7 Multiplatform Utilities & Serializers (`library/.../utils/`)

| File Name | Location | Purpose |
| :--- | :--- | :--- |
| `ExtractorApi.kt` | `library/.../utils/ExtractorApi.kt` | Base definitions for `ExtractorApi`, `ExtractorLink`, `Qualities`, and `ExtractorSubtitleStream`. |
| `M3u8Helper.kt` | `library/.../utils/M3u8Helper.kt` | HLS `.m3u8` master and media playlist parser (extracts stream qualities, bandwidths, audio tracks). |
| `HlsPlaylistParser.kt` | `library/.../utils/HlsPlaylistParser.kt` | Strict spec HLS playlist tag and URI parser. |
| `JsInterpreter.kt` | `library/.../utils/JsInterpreter.kt` | Lightweight JavaScript interpreter / math evaluator for JS-based URL obfuscation. |
| `JsUnpacker.kt` | `library/.../utils/JsUnpacker.kt` | Unpacker for Dean Edwards `eval(function(p,a,c,k,e,d)...)` obfuscated scripts. |
| `JsHunter.kt` | `library/.../utils/JsHunter.kt` | Unpacker for `JJencode` and `AAencode` JavaScript obfuscations. |
| `StringUtils.kt` | `library/.../utils/StringUtils.kt` | String cleanup, sanitization, regex helpers, slugify, and query parsing. |
| `SubtitleHelper.kt` | `library/.../utils/SubtitleHelper.kt` | Subtitle format converter (SRT, VTT, ASS, SSA) and language code mapper (ISO 639-1 / 639-2). |
| `SubtitleHelperPlatform.kt` | `library/.../utils/SubtitleHelperPlatform.kt` | Platform-specific subtitle charset decoding expect/actual declarations. |
| `Levenshtein.kt` | `library/.../utils/Levenshtein.kt` | Levenshtein distance algorithm for fuzzy title matching and similarity scoring. |
| `UnshortenUrl.kt` | `library/.../utils/UnshortenUrl.kt` | Recursively resolves redirected short URLs (bit.ly, tinyurl, etc.). |
| `AtomicList.kt` | `library/.../utils/AtomicList.kt` | Thread-safe synchronized list for concurrent scraping operations. |
| `Coroutines.kt` | `library/.../utils/Coroutines.kt` | Multiplatform coroutine scope, async timeout, and supervisor job wrappers. |
| `AppUtils.kt` | `library/.../utils/AppUtils.kt` | General data conversion, JSON serialization helpers, and URI encoders. |
| `AppDebug.kt` | `library/.../utils/AppDebug.kt` | Debugging inspection and log formatting helpers. |
| `FloatAsIntSerializer.kt` | `library/.../utils/serializers/FloatAsIntSerializer.kt` | Custom Json serializer handling numeric values formatted as Float or Int interchangeably. |
| `FloatAsLongSerializer.kt` | `library/.../utils/serializers/FloatAsLongSerializer.kt` | Serializer converting JSON Float numbers to Kotlin Long safely. |
| `JsonTransformSerializer.kt` | `library/.../utils/serializers/JsonTransformSerializer.kt` | Base JSON content transformer serializer. |
| `NonEmptySerializer.kt` | `library/.../utils/serializers/NonEmptySerializer.kt` | Filters out empty strings / collections during JSON deserialization. |
| `NullableStringSerializer.kt` | `library/.../utils/serializers/NullableStringSerializer.kt` | Safely decodes messy or non-standard null/string JSON values. |
| `WriteOnlySerializer.kt` | `library/.../utils/serializers/WriteOnlySerializer.kt` | Serializer that writes data on export but ignores incoming parse values. |

---

## 5. Module: `:app` (Android Application & UI Architecture)

### 5.1 Application Entry, Core, and Base Activities (`app/src/main/java/com/lagradost/cloudstream3/`)

| File Name | Location | Purpose |
| :--- | :--- | :--- |
| `CloudStreamApp.kt` | `app/.../CloudStreamApp.kt` | **Main Android Application Class**: Initializes ACRA crash reporting, KeyStore, DataStore, PluginManager, OkHttp clients, Notification channels, and CastContext. |
| `MainActivity.kt` | `app/.../MainActivity.kt` | **Primary Host Activity**: Single activity hosting `NavHostFragment`, back navigation handling, Leanback TV focus handling, deep links, and navigation bar setups. |
| `CommonActivity.kt` | `app/.../CommonActivity.kt` | Base Activity implementing runtime theme switching (Amoled, Dark, Light), locale updates, and PiP lifecycle callbacks. |
| `AcraApplication.kt` | `app/.../AcraApplication.kt` | Application sub-class initializing ACRA error dialogs and crash logcat capturing. |
| `DownloaderTestImpl.kt` | `app/.../DownloaderTestImpl.kt` | Test harness implementation for verifying downloader integrity and write speeds. |
| `AndroidManifest.xml` | `app/src/main/AndroidManifest.xml` | Declares Android permissions (Internet, Storage, Foreground Services, PiP, TV Leanback banner), Activities, Services, and Receivers. |
| `build.gradle.kts` | `app/build.gradle.kts` | App module build script (version codes, flavors `prerelease`/`stable`, dependencies, shrinkers). |
| `proguard-rules.pro` | `app/proguard-rules.pro` | Proguard / R8 keep rules for reflection, Jackson, Kotlin Serialization, ACRA, and ExoPlayer. |
| `lint.xml` | `app/lint.xml` | Android Lint check suppressions and rules. |

### 5.2 Network, Security, & Actions (`app/.../network/`, `actions/`, `mvvm/`)

| File Name | Location | Purpose |
| :--- | :--- | :--- |
| `CloudflareKiller.kt` | `app/.../network/CloudflareKiller.kt` | Intercepts Cloudflare 403/503 challenges using Android WebView cookie sync and User-Agent spoofing. |
| `DdosGuardKiller.kt` | `app/.../network/DdosGuardKiller.kt` | Automated bypass for DDoS-GUARD protected streaming endpoints. |
| `DohProviders.kt` | `app/.../network/DohProviders.kt` | DNS-over-HTTPS configuration (Cloudflare, Google, AdGuard, Quad9, DNS.SB) to bypass ISP DNS blocks. |
| `RequestsHelper.kt` | `app/.../network/RequestsHelper.kt` | OkHttp client builder with custom timeouts, cache, cookie jars, and SSL trust managers. |
| `AlwaysAskAction.kt` | `app/.../actions/AlwaysAskAction.kt` | Quick action handler prompting the user to select player/downloader action on video click. |
| `OpenInAppAction.kt` | `app/.../actions/OpenInAppAction.kt` | Quick action triggering internal CloudStream player. |
| `VideoClickAction.kt` | `app/.../actions/VideoClickAction.kt` | Action handler mapping video click settings to internal player, external player (VLC/MPV), or download. |
| `Lifecycle.kt` | `app/.../mvvm/Lifecycle.kt` | Android LifecycleOwner coroutine flow bindings and safe observation helpers. |

### 5.3 Plugins & Dynamic Extension Loader (`app/.../plugins/`)

| File Name | Location | Purpose |
| :--- | :--- | :--- |
| `PluginManager.kt` | `app/.../plugins/PluginManager.kt` | **Core Plugin Loader**: Loads `.cs3` plugins at runtime using `DexClassLoader`, registers loaded `MainAPI` providers into `APIHolder`, handles reload and unload. |
| `Plugin.kt` | `app/.../plugins/Plugin.kt` | Data model representing an installed or downloadable plugin (metadata, author, version, target API). |
| `RepositoryManager.kt` | `app/.../plugins/RepositoryManager.kt` | Manages remote plugin repository URLs (`plugins.json` manifests), download, update verification, and removal. |
| `VotingApi.kt` | `app/.../plugins/VotingApi.kt` | API client for upvoting / downvoting community plugins and repositories. |

### 5.4 Background Services & Broadcast Receivers (`app/.../services/`, `receivers/`)

| File Name | Location | Purpose |
| :--- | :--- | :--- |
| `VideoDownloadService.kt` | `app/.../services/VideoDownloadService.kt` | Foreground service running chunked parallel video downloads with persistent progress notification. |
| `DownloadQueueService.kt` | `app/.../services/DownloadQueueService.kt` | Service orchestrating background download queue transitions and auto-resumes. |
| `PackageInstallerService.kt` | `app/.../services/PackageInstallerService.kt` | Background APK installer service handling downloaded in-app updates. |
| `BackupWorkManager.kt` | `app/.../services/BackupWorkManager.kt` | Periodic WorkManager worker creating automatic local JSON backups. |
| `SubscriptionWorkManager.kt` | `app/.../services/SubscriptionWorkManager.kt` | WorkManager worker checking for new episodes of bookmarked/subscribed shows. |
| `VideoDownloadRestartReceiver.kt` | `app/.../receivers/VideoDownloadRestartReceiver.kt` | Broadcast receiver listening for system boot/connectivity changes to resume interrupted downloads. |

### 5.5 Synchronization & Subtitle Providers (`app/.../syncproviders/`, `subtitles/`)

| File Name | Location | Purpose |
| :--- | :--- | :--- |
| `AccountManager.kt` | `app/.../syncproviders/AccountManager.kt` | Central manager for authenticated user accounts (MAL, AniList, Trakt, OpenSubtitles, etc.). |
| `AuthAPI.kt` & `AuthRepo.kt` | `app/.../syncproviders/AuthAPI.kt` | OAuth2 authentication lifecycle, token refresh, and login webviews. |
| `SyncAPI.kt` & `SyncRepo.kt` | `app/.../syncproviders/SyncAPI.kt` | Sync state repository syncing watch progress, scores, and status across accounts. |
| `BackupAPI.kt` | `app/.../syncproviders/BackupAPI.kt` | Cloud backup API contract. |
| `SubtitleAPI.kt` & `SubtitleRepo.kt`| `app/.../syncproviders/SubtitleAPI.kt` | Subtitle search repository aggregating external subtitle providers. |
| `AniListApi.kt` | `app/.../syncproviders/providers/AniListApi.kt` | GraphQL API integration with AniList for tracking anime progress and scores. |
| `MALApi.kt` | `app/.../syncproviders/providers/MALApi.kt` | MyAnimeList OAuth2 API synchronization integration. |
| `KitsuApi.kt` | `app/.../syncproviders/providers/KitsuApi.kt` | Kitsu.io tracking and sync provider. |
| `SimklApi.kt` | `app/.../syncproviders/providers/SimklApi.kt` | Simkl sync provider for movies, anime, and TV shows. |
| `LocalList.kt` | `app/.../syncproviders/providers/LocalList.kt` | Local watch history list provider (offline fallback). |
| `OpenSubtitlesApi.kt` | `app/.../syncproviders/providers/OpenSubtitlesApi.kt` | REST API client for OpenSubtitles.com / OpenSubtitles.org. |
| `Subdl.kt` | `app/.../syncproviders/providers/Subdl.kt` | Subdl API client for external subtitle downloads. |
| `SubSource.kt` | `app/.../syncproviders/providers/SubSource.kt` | SubSource subtitle search and download provider. |
| `Addic7ed.kt` | `app/.../syncproviders/providers/Addic7ed.kt` | Addic7ed TV show subtitle scraper. |
| `AbstractSubProvider.kt` | `app/.../subtitles/AbstractSubProvider.kt` | Base abstraction for subtitle scraping and downloading. |
| `AbstractSubtitleEntities.kt` | `app/.../subtitles/AbstractSubtitleEntities.kt` | Data entities for subtitle search requests, results, and parsed tracks. |

---

## 6. UI Layer (`app/src/main/java/com/lagradost/cloudstream3/ui/`)

### 6.1 Home & Discovery (`ui/home/`)

| File Name | Location | Purpose |
| :--- | :--- | :--- |
| `HomeFragment.kt` | `app/.../ui/home/HomeFragment.kt` | Main landing page displaying trending carousels, resume watching row, and categorized rows. |
| `HomeViewModel.kt` | `app/.../ui/home/HomeViewModel.kt` | ViewModel fetching home page items from selected providers (`MainAPI.getMainPage`). |
| `HomeParentItemAdapter.kt` | `app/.../ui/home/HomeParentItemAdapter.kt` | Vertical RecyclerView adapter holding horizontal media rows. |
| `HomeChildItemAdapter.kt` | `app/.../ui/home/HomeChildItemAdapter.kt` | Horizontal RecyclerView adapter rendering individual media poster cards. |
| `HomeScrollAdapter.kt` | `app/.../ui/home/HomeScrollAdapter.kt` | ViewPager adapter for top featured media hero banner/carousel. |
| `HomeScrollTransformer.kt` | `app/.../ui/home/HomeScrollTransformer.kt` | Visual page transformer for hero banner scrolling animations. |
| `HomeParentItemAdapterPreview.kt` | `app/.../ui/home/HomeParentItemAdapterPreview.kt` | Fast preview layout adapter for Leanback TV home focus. |

### 6.2 Search & Suggestions (`ui/search/`, `ui/quicksearch/`)

| File Name | Location | Purpose |
| :--- | :--- | :--- |
| `SearchFragment.kt` | `app/.../ui/search/SearchFragment.kt` | Global search screen supporting multi-provider simultaneous queries, filters, and history. |
| `SearchViewModel.kt` | `app/.../ui/search/SearchViewModel.kt` | Dispatches search coroutines across active providers, collects stream results. |
| `SearchResultBuilder.kt` | `app/.../ui/search/SearchResultBuilder.kt` | Aggregates and ranks search results from multiple plugins. |
| `SearchAdaptor.kt` | `app/.../ui/search/SearchAdaptor.kt` | Grid/list adapter rendering search results with provider badges. |
| `SearchHistoryAdaptor.kt` | `app/.../ui/search/SearchHistoryAdaptor.kt` | Adapter for recent search terms and query history. |
| `SearchSuggestionApi.kt` | `app/.../ui/search/SearchSuggestionApi.kt` | Queries search autocomplete suggestions (e.g. Google/IMDb/TMDB suggestions). |
| `SearchSuggestionAdapter.kt`| `app/.../ui/search/SearchSuggestionAdapter.kt`| Dropdown adapter for auto-complete search suggestions. |
| `SyncSearchViewModel.kt` | `app/.../ui/search/SyncSearchViewModel.kt` | Searches sync services (MAL, AniList, Trakt) to link metadata with local search. |
| `QuickSearchFragment.kt` | `app/.../ui/quicksearch/QuickSearchFragment.kt` | Fast modal search overlay for rapid lookups without leaving current screen. |

### 6.3 Media Details & Episode Selection (`ui/result/`)

| File Name | Location | Purpose |
| :--- | :--- | :--- |
| `ResultFragment.kt` | `app/.../ui/result/ResultFragment.kt` | Base details screen orchestrator (routes to Phone or TV layout dynamically). |
| `ResultFragmentPhone.kt` | `app/.../ui/result/ResultFragmentPhone.kt` | Mobile/tablet optimized media details view with tabs (Episodes, Cast, Trailers, Sync). |
| `ResultFragmentTv.kt` | `app/.../ui/result/ResultFragmentTv.kt` | Leanback D-pad optimized TV details screen with horizontal episode carousels. |
| `ResultViewModel2.kt` | `app/.../ui/result/ResultViewModel2.kt` | **Media Details ViewModel**: Fetches full metadata (`load()`), episodes, dub/sub status, and recommendations. |
| `EpisodeAdapter.kt` | `app/.../ui/result/EpisodeAdapter.kt` | RecyclerView adapter rendering episode list with thumbnails, descriptions, and watch progress. |
| `SelectAdaptor.kt` | `app/.../ui/result/SelectAdaptor.kt` | Season and episode selector dropdown adapter. |
| `ActorAdaptor.kt` | `app/.../ui/result/ActorAdaptor.kt` | Cast and crew list adapter with TMDB headshots. |
| `ImageAdapter.kt` | `app/.../ui/result/ImageAdapter.kt` | Gallery adapter for backdrops, screenshots, and posters. |
| `ResultTrailerPlayer.kt` | `app/.../ui/result/ResultTrailerPlayer.kt` | In-line trailer video player inside details view. |
| `SyncViewModel.kt` | `app/.../ui/result/SyncViewModel.kt` | Handles user rating, watch status (Watching, Completed, Plan to Watch) synchronization. |
| `LinearListLayout.kt` | `app/.../ui/result/LinearListLayout.kt` | Custom ViewGroup layout for horizontal item alignment in details page. |

### 6.4 Fullscreen Video Player Engine (`ui/player/`)

| File Name | Location | Purpose |
| :--- | :--- | :--- |
| `FullScreenPlayer.kt` | `app/.../ui/player/FullScreenPlayer.kt` | **Primary Player Activity/Fragment**: Fullscreen video player controller, HUD visibility, gesture controls, PiP, aspect ratio, audio track & subtitle dialogs. |
| `CS3IPlayer.kt` | `app/.../ui/player/CS3IPlayer.kt` | **ExoPlayer Wrapper**: Manages ExoPlayer instance, track selectors, buffering events, DRM sessions, renderers, and media sources. |
| `IPlayer.kt` | `app/.../ui/player/IPlayer.kt` | Player interface abstracting playback methods (play, pause, seek, setSpeed). |
| `GeneratorPlayer.kt` | `app/.../ui/player/GeneratorPlayer.kt` | Player controller backed by link generators (`LinkGenerator`). |
| `LinkGenerator.kt` | `app/.../ui/player/LinkGenerator.kt` | Interface for generating video streams on demand. |
| `ExtractorLinkGenerator.kt` | `app/.../ui/player/ExtractorLinkGenerator.kt` | Invokes `loadLinks()` on providers and executes extractors concurrently to supply video URLs. |
| `RepoLinkGenerator.kt` | `app/.../ui/player/RepoLinkGenerator.kt` | Resolves links from repository/plugin providers. |
| `DownloadFileGenerator.kt` | `app/.../ui/player/DownloadFileGenerator.kt` | Resolves local downloaded files for offline playback. |
| `PreviewGenerator.kt` | `app/.../ui/player/PreviewGenerator.kt` | Generates fast video previews / thumbnails for the player scrub bar. |
| `PlayerGeneratorViewModel.kt`| `app/.../ui/player/PlayerGeneratorViewModel.kt`| ViewModel managing link resolution states, extractor progress, and auto-fallback to next sources. |
| `PlayerView.kt` | `app/.../ui/player/PlayerView.kt` | Custom ExoPlayer `StyledPlayerView` subclass with custom key event handling. |
| `PlayerGestureHelper.kt` | `app/.../ui/player/PlayerGestureHelper.kt` | Touch gestures for volume, brightness, and horizontal double-tap seek. |
| `PlayerPipHelper.kt` | `app/.../ui/player/PlayerPipHelper.kt` | Android Picture-in-Picture (PiP) parameters, aspect ratio, and action buttons. |
| `PlayerSubtitleHelper.kt` | `app/.../ui/player/PlayerSubtitleHelper.kt` | Subtitle synchronization, delay offset adjustments (-10s to +10s), and styling. |
| `CustomSubtitleDecoderFactory.kt`| `app/.../ui/player/CustomSubtitleDecoderFactory.kt`| ExoPlayer subtitle decoder factory supporting custom SRT/SSA/VTT/ASS rendering. |
| `CustomSubripParser.kt` | `app/.../ui/player/CustomSubripParser.kt` | High-tolerance SubRip (`.srt`) parser handling malformed timestamps and HTML styling tags. |
| `OfflinePlaybackHelper.kt`| `app/.../ui/player/OfflinePlaybackHelper.kt`| Checks local disk cache and decrypts local media storage for playback. |
| `Torrent.kt` | `app/.../ui/player/Torrent.kt` | Torrent stream engine integration (torrent streaming over local proxy). |
| `UpdatedDefaultExtractorsFactory.kt`| `app/.../ui/player/UpdatedDefaultExtractorsFactory.kt`| ExoPlayer media container extractors (MKV, MP4, TS, FLV) factory. |
| `UpdatedMatroskaExtractor.kt`| `app/.../ui/player/UpdatedMatroskaExtractor.kt`| Enhanced Matroska/WebM extractor with improved subtitle and audio track detection. |
| `SSLTrustManager.kt` | `app/.../ui/player/SSLTrustManager.kt` | Custom SSL socket factory ignoring self-signed/expired certificates on media CDNs. |
| `SubtitleOffsetItemAdapter.kt`| `app/.../ui/player/SubtitleOffsetItemAdapter.kt`| Adapter for manual subtitle timing calibration. |

#### Source Prioritization (`ui/player/source_priority/`)
| File Name | Location | Purpose |
| :--- | :--- | :--- |
| `SourcePriorityDialog.kt` | `app/.../ui/player/source_priority/SourcePriorityDialog.kt` | Dialog allowing users to reorder provider priority and quality preference. |
| `QualityProfileDialog.kt` | `app/.../ui/player/source_priority/QualityProfileDialog.kt` | Configuration dialog for selecting preferred resolution (4K, 1080p, 720p, etc.). |
| `SourceProfileSettingsDialog.kt`| `app/.../ui/player/source_priority/SourceProfileSettingsDialog.kt`| Advanced source selection rules configuration. |
| `QualityDataHelper.kt` | `app/.../ui/player/source_priority/QualityDataHelper.kt` | Sorts and filters video streams by resolution, codec, and latency. |
| `PriorityAdapter.kt` & `ProfilesAdapter.kt`| `app/.../ui/player/source_priority/PriorityAdapter.kt`| Adapters for drag-and-drop source priority reordering. |

#### Live TV & Video Skip (`ui/player/live/`, `utils/videoskip/`)
| File Name | Location | Purpose |
| :--- | :--- | :--- |
| `LiveManager.kt` & `LiveHelper.kt`| `app/.../ui/player/live/LiveManager.kt`| Live TV stream manager (IPTV M3U8 live stream scrubbing, channel switching). |
| `LivePreviewTimeBar.kt` | `app/.../ui/player/live/LivePreviewTimeBar.kt` | Custom progress bar for live streams with DVR seek window. |
| `SkipAPI.kt` | `app/.../utils/videoskip/SkipAPI.kt` | Base interface for video timestamp skipping services. |
| `AniSkip.kt` | `app/.../utils/videoskip/AniSkip.kt` | Client for AniSkip API (auto-skips anime openings and endings). |
| `AnimeSkip.kt` | `app/.../utils/videoskip/AnimeSkip.kt` | Integration with AnimeSkip community timestamp database. |
| `IntroDbSkip.kt` & `TheIntroDBSkip.kt`| `app/.../utils/videoskip/IntroDbSkip.kt`| TheIntroDB integration for TV show intro/recap/outro skipping. |

### 6.5 Downloads Subsystem (`ui/download/`, `utils/downloader/`)

| File Name | Location | Purpose |
| :--- | :--- | :--- |
| `DownloadFragment.kt` | `app/.../ui/download/DownloadFragment.kt` | Screen listing all downloaded shows, movies, and current storage usage. |
| `DownloadChildFragment.kt` | `app/.../ui/download/DownloadChildFragment.kt` | Detail screen showing downloaded episodes for a specific series. |
| `DownloadViewModel.kt` | `app/.../ui/download/DownloadViewModel.kt` | ViewModel managing download state, deletions, and storage calculation. |
| `DownloadQueueFragment.kt` | `app/.../ui/download/queue/DownloadQueueFragment.kt` | Active download queue screen (pause, resume, cancel, priority reordering). |
| `DownloadQueueViewModel.kt`| `app/.../ui/download/queue/DownloadQueueViewModel.kt`| ViewModel managing the active queue and parallel download workers. |
| `DownloadAdapter.kt` & `DownloadChildAdapter.kt`| `app/.../ui/download/DownloadAdapter.kt`| Adapters rendering downloaded series and episode cards. |
| `DownloadHeaderAdapter.kt`| `app/.../ui/download/DownloadHeaderAdapter.kt`| Adapter for storage usage header (used vs free device storage). |
| `DownloadButtonSetup.kt` | `app/.../ui/download/DownloadButtonSetup.kt` | Helper binding download button states (Not Downloaded, Queued, Downloading, Done). |
| `DownloadedPlayerActivity.kt`| `app/.../ui/player/DownloadedPlayerActivity.kt`| Dedicated offline player launcher for saved media. |
| `DownloadManager.kt` | `app/.../utils/downloader/DownloadManager.kt` | **Core Downloader Engine (1800+ lines)**: Multi-threaded segmented downloading, resume support, storage permissions (SAF), HLS/m3u8 stream re-muxing to MP4, and progress callbacks. |
| `DownloadQueueManager.kt` | `app/.../utils/downloader/DownloadQueueManager.kt` | FIFO queue scheduler limiting concurrent active downloads. |
| `DownloadObjects.kt` | `app/.../utils/downloader/DownloadObjects.kt` | Data classes: `DownloadItem`, `DownloadEpisodeMetadata`, `DownloadStatus`. |
| `DownloadFileManagement.kt`| `app/.../utils/downloader/DownloadFileManagement.kt`| Storage Access Framework (SAF) file operations and Scoped Storage file management. |
| `DownloadUtils.kt` | `app/.../utils/downloader/DownloadUtils.kt` | Helper methods for calculating download speed, ETA, and formatting bytes. |

### 6.6 User Library / Bookmarks (`ui/library/`)

| File Name | Location | Purpose |
| :--- | :--- | :--- |
| `LibraryFragment.kt` | `app/.../ui/library/LibraryFragment.kt` | User library screen with custom tabs (Watching, Plan to Watch, Completed, On Hold, Dropped). |
| `LibraryViewModel.kt` | `app/.../ui/library/LibraryViewModel.kt` | ViewModel querying local bookmarked items and syncing with remote tracking services. |
| `PageAdapter.kt` & `ViewpagerAdapter.kt`| `app/.../ui/library/PageAdapter.kt`| Adapters for library category viewpager pages. |
| `LoadingPosterAdapter.kt` | `app/.../ui/library/LoadingPosterAdapter.kt` | Skeleton loading placeholder adapter for library cards. |

### 6.7 Settings & Extensions Manager (`ui/settings/`)

| File Name | Location | Purpose |
| :--- | :--- | :--- |
| `SettingsFragment.kt` | `app/.../ui/settings/SettingsFragment.kt` | Main settings hub categorizing preferences. |
| `SettingsGeneral.kt` | `app/.../ui/settings/SettingsGeneral.kt` | General preferences (App language, DNS-over-HTTPS, Incognito mode, Auto-update checks). |
| `SettingsPlayer.kt` | `app/.../ui/settings/SettingsPlayer.kt` | Player preferences (Hardware acceleration, default resize mode, buffer size, gestures, external player). |
| `SettingsProviders.kt` | `app/.../ui/settings/SettingsProviders.kt` | Provider management (Enable/disable plugins, preferred media languages, scraper timeout). |
| `SettingsUI.kt` | `app/.../ui/settings/SettingsUI.kt` | UI customization (Theme colors, Amoled black, Layout mode: Phone/TV/Auto, poster layout size). |
| `SettingsAccount.kt` | `app/.../ui/settings/SettingsAccount.kt` | Account management and login for tracking/subtitle sync services. |
| `SettingsUpdates.kt` | `app/.../ui/settings/SettingsUpdates.kt` | Check for app updates (Stable vs Prerelease channel) and Changelog viewer. |
| `ExtensionsFragment.kt` | `app/.../ui/settings/extensions/ExtensionsFragment.kt` | Extension manager tab (installed plugins vs repository browser). |
| `PluginsFragment.kt` & `PluginDetailsFragment.kt`| `app/.../ui/settings/extensions/PluginsFragment.kt`| Plugin details view, permissions list, version info, install/uninstall actions. |
| `PluginsViewModel.kt` & `ExtensionsViewModel.kt`| `app/.../ui/settings/extensions/PluginsViewModel.kt`| ViewModels handling repository scraping, downloading `.cs3` files, and dynamic loading. |
| `PluginAdapter.kt` & `RepoAdapter.kt`| `app/.../ui/settings/extensions/PluginAdapter.kt`| Adapters for repository lists and individual plugin items. |
| `TestFragment.kt` & `TestViewModel.kt`| `app/.../ui/settings/testing/TestFragment.kt`| **Provider Test Harness UI**: Allows testing providers for search, load, and link extraction integrity inside the app. |
| `TestResultAdapter.kt` & `TestView.kt`| `app/.../ui/settings/testing/TestResultAdapter.kt`| UI views rendering provider test pass/fail results. |
| `DirectoryPicker.kt` | `app/.../ui/settings/utils/DirectoryPicker.kt` | Storage Access Framework (SAF) folder picker for custom download directories. |
| `Globals.kt` & `LogcatAdapter.kt`| `app/.../ui/settings/Globals.kt`| In-app Logcat viewer for debugging and crash investigation. |

### 6.8 Onboarding Setup Wizard (`ui/setup/`)

| File Name | Location | Purpose |
| :--- | :--- | :--- |
| `SetupFragmentLayout.kt` | `app/.../ui/setup/SetupFragmentLayout.kt` | Setup wizard step: Select UI layout (Phone vs Leanback Android TV). |
| `SetupFragmentLanguage.kt` | `app/.../ui/setup/SetupFragmentLanguage.kt` | Setup wizard step: Select app interface language. |
| `SetupFragmentProviderLanguage.kt`| `app/.../ui/setup/SetupFragmentProviderLanguage.kt`| Setup wizard step: Filter content provider languages. |
| `SetupFragmentMedia.kt` | `app/.../ui/setup/SetupFragmentMedia.kt` | Setup wizard step: Choose media types (Movies/TV, Anime, Asian Drama, Live TV). |
| `SetupFragmentExtensions.kt` | `app/.../ui/setup/SetupFragmentExtensions.kt` | Setup wizard step: Setup initial plugin repositories. |

### 6.9 Subtitles Configuration (`ui/subtitles/`)

| File Name | Location | Purpose |
| :--- | :--- | :--- |
| `SubtitlesFragment.kt` | `app/.../ui/subtitles/SubtitlesFragment.kt` | Subtitle preferences (Font size, text color, background color, outline/shadow, font typeface). |
| `ChromecastSubtitlesFragment.kt`| `app/.../ui/subtitles/ChromecastSubtitlesFragment.kt`| Custom subtitle style configuration for Google Cast / Chromecast receiver. |

---

## 7. App Utilities & System Integration (`app/src/main/java/com/lagradost/cloudstream3/utils/`)

| File Name | Location | Purpose |
| :--- | :--- | :--- |
| `AppContextUtils.kt` | `app/.../utils/AppContextUtils.kt` | Global application context helpers, clipboard copy, intent dispatchers, and resource lookups. |
| `UIHelper.kt` | `app/.../utils/UIHelper.kt` | Display metrics, DP/PX conversion, navigation bar height, window insets, and keyboard hiding. |
| `DataStore.kt` & `DataStoreHelper.kt`| `app/.../utils/DataStore.kt`| Persistent key-value storage engine using Jackson/JSON serialization for preferences, history, and cache. |
| `BackupUtils.kt` | `app/.../utils/BackupUtils.kt` | Full backup export and restore engine (creates JSON archive of history, bookmarks, accounts, and settings). |
| `InAppUpdater.kt` | `app/.../utils/InAppUpdater.kt` | Fetches releases from GitHub API, parses changelog, downloads update APK, and initiates package installer. |
| `PackageInstaller.kt` | `app/.../utils/PackageInstaller.kt` | Android PackageInstaller API integration for seamless APK installation on Android 12+. |
| `CastHelper.kt` & `CastOptionsProvider.kt`| `app/.../utils/CastHelper.kt`| Google Cast SDK integration: Casting video streams, media metadata, and remote subtitles to Chromecast. |
| `BiometricAuthenticator.kt`| `app/.../utils/BiometricAuthenticator.kt`| Fingerprint / Biometric app lock prompt for privacy protection. |
| `TvChannelUtils.kt` | `app/.../utils/TvChannelUtils.kt` | Publishes resume watching and recommendations to the Android TV Leanback Home Screen channels. |
| `TestingUtils.kt` | `app/.../utils/TestingUtils.kt` | Automated verification utilities for testing provider endpoints against real network responses. |
| `SingleSelectionHelper.kt` | `app/.../utils/SingleSelectionHelper.kt` | Dialog utility for single-choice and multi-choice list popups. |
| `ImageModuleCoil.kt` & `ImageUtil.kt`| `app/.../utils/ImageModuleCoil.kt`| Coil image loading pipeline, disk caching, custom SSL image loaders, and headers. |
| `PowerManagerAPI.kt` | `app/.../utils/PowerManagerAPI.kt` | Battery optimization exemption helper for unthrottled background downloads. |
| `SnackbarHelper.kt` | `app/.../utils/SnackbarHelper.kt` | Standardized Snackbar notifications with action buttons. |
| `TextUtil.kt` | `app/.../utils/TextUtil.kt` | Text formatting, capitalization, time duration strings. |
| `SubtitleUtils.kt` | `app/.../utils/SubtitleUtils.kt` | Subtitle file decoding and encoding helpers. |
| `SyncUtil.kt` | `app/.../utils/SyncUtil.kt` | Sync state helper mapping provider titles to tracking database IDs. |
| `FillerEpisodeCheck.kt` | `app/.../utils/FillerEpisodeCheck.kt` | Queries anime filler databases (AnimeFillerList) to mark filler/canon episodes. |
| `GitInfo.kt` | `app/.../utils/GitInfo.kt` | Injected Git commit hash and build timestamp information. |
| `Event.kt` & `ConsistentLiveData.kt`| `app/.../utils/Event.kt`| Reactive LiveData wrappers preventing duplicate event consumption (SingleLiveEvent pattern). |

### Custom Widgets (`app/.../widget/`)
| File Name | Location | Purpose |
| :--- | :--- | :--- |
| `CenterZoomLayoutManager.kt` | `app/.../widget/CenterZoomLayoutManager.kt` | RecyclerView LayoutManager that zooms/magnifies the centered card item. |
| `FlowLayout.kt` | `app/.../widget/FlowLayout.kt` | Flow layout ViewGroup wrapping tags, genres, and badge chips to multiple lines. |
| `LinearRecycleViewLayoutManager.kt`| `app/.../widget/LinearRecycleViewLayoutManager.kt`| Optimized smooth-scrolling LinearLayoutManager for D-pad navigation on Android TV. |

---

## 8. Test Suites & Quality Assurance

| Test File | Location | Test Scope |
| :--- | :--- | :--- |
| `ExampleInstrumentedTest.kt` | `app/src/androidTest/.../ExampleInstrumentedTest.kt` | Basic Android instrumentation test checking application context and package name. |
| `SerializationClassTester.kt` | `app/src/androidTest/.../SerializationClassTester.kt` | Validates Jackson & Kotlinx serialization/deserialization for all project models. |
| `UriSerializerTest.kt` | `app/src/androidTest/.../UriSerializerTest.kt` | Tests custom Android URI JSON serialization. |
| `SubtitleSelectionTest.kt` | `app/src/test/.../SubtitleSelectionTest.kt` | Unit tests for automatic subtitle language selection and matching algorithms. |
| `EpisodeDateTest.kt` | `library/src/commonTest/.../EpisodeDateTest.kt` | Tests episode air date parsing across various date string formats. |
| `SplitUrlParametersTest.kt` | `library/src/commonTest/.../SplitUrlParametersTest.kt` | Validates query parameter splitting and URL parameter extraction. |
| `ArchComponentExtTest.kt` | `library/src/commonTest/.../ArchComponentExtTest.kt` | Tests reactive observable state and flow components. |
| `JsInterpreterTest.kt` | `library/src/commonTest/.../JsInterpreterTest.kt` | Validates JavaScript evaluator against obfuscated arithmetic expressions. |
| `StringUtilsTest.kt` | `library/src/commonTest/.../StringUtilsTest.kt` | Unit tests for regex extraction, slugification, and string cleanup. |
| `SerializersTest.kt` | `library/src/commonTest/.../SerializersTest.kt` | Tests multiplatform JSON serializers (FloatAsInt, NonEmpty, WriteOnly). |

---

## 9. AI Agent Quick Action & Task Matrix

When evaluating or generating PRs in this repository, reference this matrix to immediately locate target files:

| Task | Primary Target Files | Secondary Target Files |
| :--- | :--- | :--- |
| **Add a new video extractor** | `library/.../extractors/<Host>Extractor.kt` | `library/.../extractors/helper/`, `library/.../utils/ExtractorApi.kt` |
| **Fix broken video playback / stream parsing** | `library/.../extractors/`, `library/.../utils/M3u8Helper.kt` | `app/.../ui/player/ExtractorLinkGenerator.kt`, `app/.../ui/player/CS3IPlayer.kt` |
| **Fix download failure or storage access** | `app/.../utils/downloader/DownloadManager.kt` | `app/.../utils/downloader/DownloadFileManagement.kt`, `app/.../services/VideoDownloadService.kt` |
| **Improve Android TV (Leanback) UX** | `app/.../ui/result/ResultFragmentTv.kt`, `app/.../MainActivity.kt` | `app/.../widget/LinearRecycleViewLayoutManager.kt`, `app/src/main/res/layout-television/` |
| **Add or fix subtitle source** | `app/.../syncproviders/providers/<Sub>.kt` | `library/.../utils/SubtitleHelper.kt`, `app/.../ui/player/PlayerSubtitleHelper.kt` |
| **Update / Fix Tracking Sync (AniList/MAL/Trakt)**| `app/.../syncproviders/providers/` | `app/.../syncproviders/AccountManager.kt`, `app/.../syncproviders/SyncRepo.kt` |
| **Fix Plugin / Extension dynamic loading** | `app/.../plugins/PluginManager.kt` | `app/.../plugins/RepositoryManager.kt`, `library/.../plugins/BasePlugin.kt` |
| **Add Auto-Skip Video Timestamps** | `app/.../utils/videoskip/` | `app/.../ui/player/FullScreenPlayer.kt` |
| **Migrate UI to Jetpack Compose / MVI** | `COMPOSE.md`, `app/.../ui/` | `library/.../mvvm/ArchComponentExt.kt` |
