=== WP Agent Bridge ===
Contributors: wp-agent-platform
Tags: ai, automation, wordpress
Requires at least: 6.4
Tested up to: 7.1
Requires PHP: 8.2
Stable tag: 0.193.36
License: GPLv2 or later
License URI: https://www.gnu.org/licenses/gpl-2.0.html

Native WordPress site management for AI agents through secure MCP connections.

== Native write workflow ==

Install and activate Bridge, connect your AI tool using the existing setup, and
approve the requested capabilities inside WordPress. No additional desktop
software, validator service, browser runtime or secret configuration is required.

Native AI block writes use the existing guarded transactions and check the exact
saved content before commit. Failed transactions compensate compiler-owned theme
files while preserving later human file edits and retaining recovery versions.
Human editing, review copies and exact recovery remain available.

JavaScript serializer validation is not enforced. Native editor Save/reopen and
rendered desktop/mobile fidelity remain explicit acceptance checks; successful
PHP parsing or a content readback does not establish editor compatibility.
The completed site runs with Bridge and authoring helpers disabled.

== Connect your AI tool ==

Open WP Agent > Settings and choose Codex, Claude Code, ChatGPT, or Claude.
Codex and Claude Code use a private site key. ChatGPT and Claude desktop/web
connectors use WordPress sign-in and consent. The separate Basic Application
Password MCP endpoint is retired. Existing site keys and OAuth grants are retained.
Before upgrading a site whose MCP client uses `/mcp/wp-agent-server`, move that
client to the site-key endpoint and verify an identity read. Bridge 0.189.22
supports both routes; 0.189.23 removes the Basic route.

Site-key durability was observed on RYP; desktop/web OAuth requires separate
client and natural-refresh qualification. A locally passing flow does not certify
all apps. HTTPS, single-site WordPress 6.9+, and PHP 8.2+ are required for MCP.

== Native theme migration ==

Version 0.189.0 translates Gutenberg presentation input into Core block attributes
and an ordinary child theme. After migration, activate or deactivate the Bridge
without changing the site's front-end design. The Bridge is an authoring and
migration tool; the theme and Site Editor styles own the published design.

For an existing site, capture a baseline, generate and install a child theme from
the current theme and its Site Editor settings, review `read-native-compilation`
for each page, template part and navigation post, then run guarded
`compile-to-native`. Verify the anonymous site at mobile, tablet and desktop
widths. Only after visual approval, run `strip-presentation-metadata` on each
resource and retain the exported `theme/source/presentations-<page>.json` files.
Reopen the pages in Gutenberg and save once to confirm native editor round trips.
Keep snapshot receipts until the site is accepted. No legacy presentation class
or plugin render filter is needed after the strip.

Deploy the child theme folder before pushing the migrated database. WP Migrate's
Themes option or a normal theme upload can transfer the theme; then push the
database as usual. Include bundled fonts, `theme.json`, `styles/*.json`, the
stylesheet, aurora assets and exported provenance. Preserve direct edits to theme
files and `wp_global_styles`: they are authoritative on later compiler runs.

The builder copies the active Site Editor settings and the database template
parts named in `--ids` at build time. WordPress stores database template-part
overrides and global styles separately for each theme, so switching to the
child theme does not carry later edits across automatically. Finish those edits
before building. If either changes between build and activation, rebuild into
a fresh output directory or re-apply the edit in the child theme. When rerun
against an existing output folder, the builder warns about a template-part
file that differs from its current database override and preserves the file.
The builder starts a new child theme at version 1.0.0 and increments the patch
number when a later run changes generated files or detects direct theme edits.
It keeps a theme-file fingerprint in `style.css`; a no-op run leaves the version
unchanged. Direct edits remain authoritative.

Use `tools/update-native-theme.php` to rebuild, review changed file names,
create a child-theme ZIP and optionally install it into a WordPress installation
available on the same filesystem. Example:

`php tools/update-native-theme.php --wordpress=/path/to/source-wordpress --output=/path/to/theme --name='My Child Theme' --slug=my-child --parent=twentytwentyfive --ids=12,34 --package=/path/to/my-child.zip --destination-wordpress=/path/to/destination-wordpress`

The review JSON reports `themeHash`, `destinationHash` (`absent` on first install)
and `changes`. Inspect the changed files and repeat the command with `--apply`
plus `--expected-theme-hash=<reviewed themeHash>` and
`--expected-destination-hash=<reviewed destinationHash>`. Installation refuses a
changed source or destination. If the installed theme has direct edits since its
last generated release, installation stops even after a fresh review; copy those
edits into the source theme and reconcile the destination before retrying. The
updater saves a ZIP backup of the installed theme beside the package and asks
WordPress to verify the child theme. A no-op run keeps the same version and
package bytes. For a remote host without a shared filesystem,
transfer the reviewed ZIP using WP Migrate's Themes option or a theme upload.

Fluid spacing and type values start at 390px and reach the tablet value at
768px by default; the current presentation schema defines tablet at 768px and
desktop at 1024px. To choose another fluid endpoint for a new child theme, set
`settings.custom.wpAgent.breakpoints.tabletBreakpointPx` in its `theme.json`
before running the builder. Responsive gaps use a theme spacing preset because
Core layout support rejects a raw `clamp()` in `style.spacing.blockGap`.
For pages compiled by 0.189.0 that still retain presentation metadata, rebuild
the child theme and rerun the guarded compilation to replace the raw gap.
If metadata was already stripped, use the exported presentation sources or a
reviewed native block edit; the builder cannot infer removed source values.

== Keep connections separate during database migrations ==

WordPress stores Application Password grants in the `users`/`usermeta` data,
not in WP Agent Bridge settings. A full WP Migrate push or pull replaces those
tables on the destination. The copied password may then authenticate against
ordinary WordPress REST endpoints on both sites; OAuth also checks its issuer and resource audience. Site keys are stored in
the options table and hashed with their site address, so include them in the
connection-preservation plan; copying options is not a credential migration.

For environments with the same established user IDs, save separate WP Migrate
push and pull profiles using Select Tables. Exclude both the source `users` and
`usermeta` tables in each direction. This preserves each destination's users,
permissions, Application Passwords, and OAuth grant metadata. WP Migrate
preserves its saved profiles when replacing the options table. Review the table
selection whenever plugins add tables. If users or roles must migrate, use a
separate reviewed migration and reconnect or rotate credentials afterward;
excluding the user tables means user and role changes will not sync.

After an earlier full-table sync, inspect Application Passwords in each
environment and revoke copied grants that belong to another site. Do not reuse
an Authorization header between environments. Keep a separate Bridge connection
per site and verify `get-identity` after each migration.

The connected SEO bridge also provides guarded `review-seo-fields`,
`apply-seo-fields` and `restore-seo-fields` for the active qualified TSF, Yoast SEO,
Rank Math or AIOSEO provider. Review the installed schema and the anonymous
output after every write. Provider-specific unsupported fields are refused.

== Administrator staging delivery tests ==

read-form-delivery-test returns the saved page-local form configuration and exact
page, configuration and profile guards. With explicit authorization for live
staging values, submit-form-delivery-test uses the existing transport without
publishing or enabling the draft. Verify the Administration Email Address and
mail service before declaring administratorEmailVerified:true. route:site-admin
requires configured administrator routing and tests mail acceptance only; it
does not simulate a provider outage or qualify primary-failure fallback.
Identical test-key retries recover the separate signed receipt without resending;
pending/uncertain receipts require independent inspection, never a new-key retry.
Provider storage, subscriber state and inbox receipt require separate evidence.

The native Provider embed block also supports youtube-dialog with an 11-character
video ID, button label, video title and optional local poster attachment. It uses
native dialog accessibility and lazy youtube-nocookie playback; closing removes
the iframe and restores focus. Arbitrary provider URLs, scripts and HTML remain
unavailable. Installed editor Save/reopen and desktop/mobile appearance still
require acceptance.

== Animated decoration through your AI connection ==

Ask your connected agent to add an aurora background to a Gutenberg hero or panel
using your palette. The typed request compiles into a native group and theme-owned
aurora assets. New pages use create-gutenberg-draft-from-composition. Existing
pages use a guarded review and native compilation before merge. The child theme
owns visitor animation, pause/resume and reduced-motion behavior. Keep snapshot
receipts for guarded restoration. No caller-supplied CSS or code.

== Description ==

Version 0.188.0 previously fixed blocks with decimal presentation values losing styling after
migrating between hosts. One canonical encoder sorts structured keys, fixes JSON
flags and restores PHP settings immediately. Existing legacy -1 and precision-17
classes matched CSS without a recompile. Persisted records identified their
fingerprint scheme; CSS numbers have deterministic decimal formatting.
Integer-only design classes are unchanged. Builder profiles diagnose canonical,
legacy and invalid classes; rehash-review-presentations canonicalizes verified
legacy classes in an isolated review before the normal guarded merge/restore.
Invalid classes are not legitimized. Shared resources retain their own owners.
Version 0.189.0 retires this runtime class path. Undo records created before this
update are outside the compatibility guarantee.
New changes retain normal guarded snapshots and restoration. Older pending
workflows or retained integration bindings may require fresh preparation.

Version 0.187.0 extends partial migration through shared shell export and
destination dependency staging. Recoverable text, nested navigation and supported
presentation survive unrelated missing assets. Sessions retain pending references
and source intent, and stage true navigation/header/footer drafts through existing
native owners. Authorized shell preview expands draft menu references only in
memory. Native-edit protection, source guards and separate activation remain;
staged fragments are not a claim of complete site or browser acceptance.

Version 0.186.0 adds ordinary authenticated MCP discovery, reading, guarded updates
and restoration of registered ACF options-page fields. Shared option-backed CTA
values keep their native site-wide owner; page-local content is not rewritten.
Options-page capabilities, field support and source-read-only policy still apply.
The existing permission-guarded page ACF reader is also discoverable through MCP;
its page/administrator checks and separate page-write workflow are unchanged.

Version 0.185.0 adds draft-only semantic recovery beneath qualified Divi mappings.
Supported content and presentation remain the first choice; unresolved fragments
retain source findings and extraction provenance through destination sessions.
Empty layout shells no longer count as transferred content. Final readiness
remains separate from useful draft progress. Destination operations keep their
existing serial preparation and drift guards. This candidate requires rendered
site acceptance; it does not claim a completed migration or automatic site shell.

Version 0.183.0 exports every compiler-valid Gutenberg block from a Divi source
even when other source paths remain unresolved. A page with no qualified blocks
returns an explicit empty partial export for a reserved destination placeholder.
Blocked containers retain qualified descendants. Partial exports carry source mapping
blockers; rebuild and publication guards remain strict.

Version 0.179.0 adds a two-instance migration workflow: administrator-controlled
source protection, destination migration sessions, explicit resource and URL maps,
resumable native builds, source-change reconciliation and paired validation with
separate final-domain launch review. The agent connects to each site independently.
Session records retain client-reported source provenance and do not deploy a site.

Version 0.178.0 expands literal Divi module mappings and adds explicitly reviewed
native replacements for custom modules, including public dynamic post queries.
PHP dependency inspection identifies source owners and native replacement paths;
signed retained integrations preserve exact source shortcodes and dynamic blocks
through independently active plugins. Source/provider drift guards protect writes
and synced-pattern dependencies. Yoast SEO, Rank Math and AIOSEO extend the closed
source-bound metadata transfer and recovery workflow. Rank Math's byte-qualified
shared MCP adapter supports Application Passwords; foreign OAuth is not adopted.
Arbitrary PHP is not executed or automatically translated. Independent site
acceptance remains separate from the packaged implementation.

Version 0.177.0 adds typed native search, archives, categories and social links,
with inspected stock Divi widget proposals. Global typography supports explicit
fixed/fluid sizing and preserves activated font faces. Reviewed cross-site metadata
transfer uses qualified TSF 5.x fields and remapped JSON-LD, with explicit remaining
owner decisions and recovery. Bridge-uploaded images can be retired and restored
without deleting files; stored and recoverable consumers prevent retirement.
The bundled migration skill requires anonymous public-route and metadata validation.
Arbitrary PHP, custom modules and unsupported SEO providers remain explicit gaps.

Version 0.176.0 extends guarded migration lifecycle to eligible public native
content types using their own capabilities and unchanged canonical resource IDs.
Source identity remains stable across an intentional theme switch. Qualified
exclusive accordions map to native Details groups; independent FAQ intent is never
silently substituted with Divi exclusivity. The 0.175.0 discovery capabilities remain.

Version 0.175.0 generalizes source inventory with revision-bound pagination and
adds guarded Divi 4 source normalization, recurrence discovery, nested conversion
to native synced patterns, native design evidence, and named Global Styles tokens
through MCP. A guarded composition export exposes source dependencies requiring
target remapping. Sharing and design consolidation remain explicit choices with
native recovery. Every captured
responsive viewport must qualify before cutover. Unknown modules, CSS, dynamic
behavior and cross-site transfer remain explicit boundaries, not silent omissions.

Version 0.174.0 adds guarded correction of confirmed business addresses and map
coordinates in migrated schema, with original source evidence retained and
nested recovery snapshots. Other schema and SEO fields remain unchanged.

Version 0.173.0 adds a final canonical-page metadata comparison against retained
source evidence, source-bound structured-data and social-image preservation with
recovery, and transactional checks that cutover leaves unrelated metadata intact.
Staging indexing policy and external search/AI visibility remain separate checks.

Version 0.172.0 corrects source-reference capture with native Divi 5 presets
and preserves qualified source navigation, responsive spacing, row widths and
inherited button styling. Native nested lists survive Save and reconciliation.
Guarded page edits can add unique anchors; attachment redirects remain explicit
validation items, and single-location map links target the actual marker.

Version 0.171.0 adds retained source references, bounded rendered-output and link
comparison, and source-backed desktop/mobile cutover guards. Browser reports and
screenshots retain explicit client provenance; they do not establish automatic
visual or provider-delivery proof. A bundled migration guide exposes the workflow.
Typed viewport sizing preserves qualified source hero geometry without raw CSS.

Version 0.170.0 expands qualified Divi 5 migration with native rich content,
responsive styles, fonts, source dependencies, maps and provider embeds.
Shared page/404 templates have guarded activation and recovery. Native group
presets and selective globals preserve local edits. Synced patterns and reusable
styles now track media dependencies through staged edits and recovery.
Behavioral equivalents require explicit selections; unsupported source intent
remains visible. Original-site cutover and external delivery are separate checks.

Version 0.169.0 corrects native Divi preset, editor runtime and migration inspection compatibility.

Version 0.168.0 expands the migration candidate with site inventory, native
header/footer/menu conversion, selective content overrides and explicitly
accepted preset-style equivalents. Divi global modules and typed preset changes
use reviewed native workflows with recovery. Synced patterns include registered
pattern dependencies, shared editor styles and terminal-review-safe retirement.
Unsupported source behavior is reported; site cutover requires a qualified pilot.

Version 0.167.0 adds native Divi composition through MCP, bounded Divi-to-Gutenberg
page migration, and native synced-pattern creation, linked instances, overrides,
reviewed global editing and recovery. Shared-pattern changes invalidate stale
page reviews. Unsupported Divi content remains an explicit migration blocker;
site theme/shell conversion and installed acceptance are separate qualification.


Version 0.166.0 adds a signed page-write baseline to selected Gutenberg builder profiles: changedSinceAgentWrite is true/false when known and null otherwise. Live content remains authoritative; existing pages use isolated reviews. One review-context read now includes bounded text/name/anchor hints for fresh target discovery. Paths remain version-specific, and the separate full-composition drift guard is unchanged.

Responsive inline breaks now use registered span formats that survive native RichText editing. Existing marked BR elements are upgraded locally when their heading or paragraph is opened in the editor; Save persists the change. Already stripped plain breaks cannot be inferred. Isolated real-Core RichText and breakpoint checks passed; installed editor Save/reload remains pending. Aurora discovery explains screen-blended shimmer and the tradeoff between visible motion and text contrast.

Version 0.165.0 adds page-wide presentation reads, compact composition discovery, structured MCP retry/validation errors, real flow spacing, scoped Global Styles typography/padding, malformed font-bucket rejection, explicit site-admin form delivery and lightweight composition polling. Form inbox and installed-client acceptance remain separate checks.

Version 0.164.1 makes site-build discovery consistent: native owners, source lifecycle, supported MCP edit paths and published-page impact are disclosed for templates, shared parts, menus, fonts and styles. Immutable theme files remain editable through supported database overrides. Font installation and activation, template layout and slot edits, and actual routes versus potential theme-wide effects are explicitly distinguished.

Version 0.162.0 adds ordered custom decorative layers with typed gradient, geometry, blur, blending, timing and motion endpoints, plus base and interaction shadow stacks. Existing Aurora designs retain their default layers and motion; responsive replacements reset unused layer styles. Pause, reduced motion, no-script and static editor behavior remain.

Version 0.161.0 unifies hover, focus, current, expanded and disabled presentation, including responsive overrides and shared component styles. Native controls receive their own state styling; disabled takes priority, focused solid surfaces suppress hover gradients, and keyboard/reduced-motion behavior is preserved.

Version 0.160.0 lets supported existing native blocks adopt, switch or remove reusable component-style bindings through the existing presentation review workflow. Reads disclose compatible styles and local/native overrides; edits preserve native content and attributes and reject stale shared definitions.

Version 0.159.0 adds guarded inline link destination edits inside existing headings and paragraphs. Native text reads disclose exact anchor targets; updates preserve formatting, link behavior, neighboring content and later native edits through the existing review, snapshot and recovery lifecycle.

Version 0.158.0 extends native navigation presentation to submenu parents, including toggle-only parents and keyboard/expanded states. Shared-resource impact traces menus and patterns through supported native templates and page content, with explicit unknown coverage. Existing native behavior and guarded recovery remain.

Version 0.157.0 adds reusable component styles in native Global Styles, responsive interaction states, multi-stop linear/radial gradients, typed shadow geometry, per-layer gradients, blur, opacity, blend and animation timing, targeted native text-node edits and connected native-owner guidance. Includes the stored-presentation compatibility fix below.

Version 0.156.1 preserves safe stored presentation and saved ownership classes when unknown optional properties are present, exposes targeted repair diagnostics, and adds durable typed aurora pauseControl support. Strict new-write validation and reduced-motion behavior remain.

Version 0.156.0 adds whole/inline gradient text, builder-controlled transition timing and easing, real gradient crossfades with keyboard/reduced-motion support, typed native menu-link appearance and current-page styling. Explicit page-list conversion preserves reviewed page identities/order and records the loss of automatic inclusion. Shared-resource discovery reports the supported database-override editing path for theme-provided headers and footers through normal MCP. Existing permission, stale-state, retry, snapshot and restore guards remain.


Version 0.155.0 integrates palette-driven three-layer Gutenberg decoration, accessible motion controls, typed gradient and hover surfaces, 240px display type, and connected-agent guidance with the native site-build and font capabilities.

Version 0.154.0 preserves requested Google variable-font optical sizing through native Font Library imports. Existing theme preset names can be rebound to an installed family so native content follows the corrected font without page rewrites. Imports and activation retain their version, retry and recovery guards.

WP Agent Bridge by Jay Karr. Version 0.152.3 adds guarded native site-title, tagline and static-homepage settings with recovery, and preserves styled native buttons during targeted link edits. These complete the remaining Northstar launch operations through the ordinary MCP connection.

Version 0.144.0 adds native responsive navigation, rich text runs, responsive type sizing, semantic page regions, editable email-form previews and bundled vector icons for composition builds.

Version 0.140.0 adds typed Gutenberg draft composition and responsive presentation, with image upload through the ordinary MCP connection. Draft revisions retain permission, ownership, version and recovery guards. The full Northstar draft and original image uploads have been observed through hosted MCP; visual fidelity remains in progress.

Version 0.139.0 preserves native Gutenberg Image boundary whitespace when replacing an attachment and its alt text. Media, source, review and context guards remain enforced.

Version 0.138.0 allows cancellation after a source edit when both current page versions are reviewed. Only the review moves to Trash; the changed source is preserved and exact retries retain their receipts.

Version 0.137.0 binds merge snapshots to the applied page version. Restoring an older merge cannot overwrite later edits even when supplied a fresh page hash. Merge snapshots from older releases without this evidence require reviewed manual recovery; completed connection retries retain their original receipts.

Version 0.136.0 uses the native WordPress query URL for Application Password setup, which can work when a host’s pretty REST route fails. Both setup screens show the same URL, preserve site subdirectories, and explain how to update an existing connection without replacing its credential. Setup checks do not prove external connectivity.

Version 0.135.0 validates OAuth refresh client binding before rotation and correctly classifies authorization callback audit events. Existing strict replay protection is unchanged. WP Agent Setup includes color status circles to WP Agent Setup: green for ready checks, red for unmet requirements and gray for optional integrations not detected. Text labels remain available alongside the indicators.

Bridge 0.133.0 prepares opt-in theme-pattern materialization inside isolated Gutenberg Header/Footer reviews. The original theme source and existing resource version hashes stay unchanged. Registered theme-pattern dependencies bind review application and restoration. Only registered active/parent-theme patterns are eligible; generic page-pattern eligibility is unchanged. Source/native read-only checks do not establish live application, restoration or browser acceptance.


Bridge 0.130.0 adds the native parentPageID filter to top-level dynamic Page Lists. Bare lists and explicitly stored zero filters keep their native representation; positive parent IDs must select a readable published page on this site. Generated descendants remain dynamic. Nested lists and other attributes remain unsupported. Live filtering and menu lifecycle acceptance are pending.


Bridge 0.129.0 preserves a bare native dynamic Page List while editing adjacent navigation links. Generated page links remain dynamic; the existing Page List cannot be removed or moved by this bounded operation. Parent-page filters and other attributed or nested Page Lists remain read-only. Existing resource guards, review-set validation, snapshots and exact retries are reused.

Bridge 0.128.0 preserves existing native section, text and button preset assignments in bounded Divi Library reuse. Fresh native preset values join resource, review and recovery version guards; preset options are never copied or restored by a page operation. Other extra local styling remains outside this slice.

Bridge 0.120.0 preserves exact native Library sections with bounded desktop, tablet and phone padding/margins during draft creation and review insertion. The existing semantic hierarchy and module limits remain enforced. Presets and other extra styling remain outside this slice.

Bridge 0.119.0 preserves an undated review draft’s existing date when inserting a Divi Library layout. This prevents WordPress date advancement from causing a false read-back conflict; native tree, context, source, permission, rollback and retry checks remain unchanged.

Bridge 0.118.0 explains Codex connection setup in three steps on both WP Agent Setup and the one-time configuration page. A field-by-field table shows the site URL, Authorization header and fields to leave empty, with separate-site, restart, old-entry and lost-configuration guidance. Authentication and editing behavior are unchanged.

Bridge 0.117.0 adds native Application Password connections in WP Agent Setup: create a named connection, copy its private MCP configuration once, rename it, or revoke access. The separate HTTPS MCP endpoint uses WordPress credentials without OAuth renewal and preserves existing permissions and reviewed operations. Actual desktop retention and release qualification remain pending.

Bridge 0.116.0 connects reviewed workflow-draft publication and snapshot restoration to normal MCP/OAuth. Publication requires a completed matching review and current page version; native permissions, locks, verification and exact retry receipts remain in force. Restoration refuses stale page documents and preserves unrelated metadata. Source and disposable Local handler checks are separate from everyday-client acceptance.

Bridge 0.115.0 connects guarded merged-page restoration to normal MCP/OAuth and lets safe Gutenberg reviews expose an empty plain-text target list. Native rich text remains preserved and is never flattened into plain-text edits. Restoration reuses native snapshots, checks current-principal provenance and the locked target, refuses later edits and retains exact retry receipts.

Bridge 0.114.0 adds connection inspection, renaming and site-access removal directly in WP Agent Setup for the current user on single-site WordPress. Native permission and POST nonce checks protect changes; a previous-value guard refuses concurrent credential-list changes. The page does not configure external MCP clients or prove their refresh durability. Multisite management and unsafe native credential values fail closed. It also connects isolated review-title editing to normal MCP/OAuth. It reuses native review snapshots, permissions, source/version guards and exact retry receipts, including retries after merge or cancellation. This is a source candidate; normal-client and native-editor acceptance remain pending.

Bridge 0.113.0 adds private, bounded OAuth refresh lineage diagnostics for issuance, successful rotation, and rejection. Diagnostic storage failure never interrupts authentication; token replay protection is unchanged.

Bridge 0.112.0 adds guarded Divi Library append to pending page reviews through the normal OAuth connection. It reuses the native compiler, snapshots, review merge and discard workflow. Library sources stay unchanged; agents supply disclosed resource and review guards, never native serialization.
New native Gutenberg Header/Footer resources use a plain inner layout group. WordPress supplies the single semantic template-part wrapper, avoiding nested header/footer landmarks. Existing resources retain their authored markup.
Bridge 0.111.0 keeps generated Divi asset cache metadata out of shared-resource fingerprints, so viewing a page does not block coordinated restoration. Post fields and all authored or unknown metadata remain guarded. Coordinated design apply and restore also invalidate generated Divi CSS so shared pages receive current colors and layout defaults. Divi line-height inputs explicitly disclose their native 0.1 increments. Native header/footer creation, review, assignment, navigation and design use the existing OAuth connection.

Bridge 0.103.0 extends the existing native review workflow to ordinary unpublished Gutenberg and Divi pages, including pages another author owns when WordPress permits editing. Merge preserves draft status; conflict checks, cancellation, recovery and active-review cleanup protection remain automatic. No new connection operations are required. Native Gutenberg and Divi draft lifecycles and published-page regressions pass, including exact title/excerpt preservation under WordPress unslashing. Normal OAuth/browser and native editor acceptance of this expansion remain pending.

Bridge 0.102.0 expands registered pattern drafts through WordPress native nested-pattern resolution, with child dependency guards and native social links/search. Native acceptance passes a complete six-section Twenty Twenty-Five business homepage, every expected contact-pattern social link, and its same-site search form. Source preservation, stale guards, retries, rollback, and cleanup pass. Normal OAuth/browser and native editor acceptance remain pending.

Bridge 0.101.0 adds three connection abilities for registered Gutenberg pattern discovery, independent native draft creation with supported styling preserved, and current-version draft cleanup. Native acceptance passes controlled styled patterns and a Twenty Twenty-Five image hero, including exact source preservation, rollback, retries, and Unicode/backslash persistence. Normal OAuth/browser and native editor round-trip acceptance remain pending.

Bridge 0.100.0 adds three connection abilities to discover compatible Divi Library layouts, create independent native drafts with editor links, and discard owned drafts with a current version guard. The Library source stays unchanged. Native acceptance passes on exact Divi 5.12.1; normal OAuth/browser and native editor round-trip acceptance remain pending.

Bridge 0.99.0 lets coordinated shared-site reviews include staged native Header, Footer and Navigation resources. Restore recovers the prior content, assignments and global design, then removes only the guarded adopted resources. Initial staging still uses the normal creation operations. Existing-resource workflows remain available. Normal OAuth/browser acceptance of this combined workflow remains pending.



This development bridge exposes a capability manifest plus these narrowly scoped operations for WordPress posts and pages:

* Connect an MCP-compatible agent from Tools > WP Agent Setup using browser WordPress login and OAuth consent. The agent host must keep and renew the connection automatically. WP Agent Setup manages this site access; client registration and consent still start in the client. Bridge 0.114.0 bundles a pinned OAuth acceptance candidate and the official MCP Adapter. Refresh-token rotation now checks initial marker persistence and uses an atomic marker comparison so failed issuance and concurrent refreshes cannot orphan or overwrite the live session; sequential stale-token replay still revokes that session. This experimental feature requires WordPress 6.9+ and HTTPS; existing Bridge functionality retains its 6.4 minimum. Six existing read-only Bridge Abilities expose site discovery, builder profiles, design tokens, Gutenberg global design, site shell, and guarded page context through MCP. Four separate connection Abilities provide identity, draft-page title read, guarded draft-title update, and private-snapshot restore. Thirteen additional OAuth abilities create and inspect review copies, search replacement media, edit review titles/text/buttons/images, create native sections with one to four columns of headings, paragraphs, lists, and buttons, reorder/duplicate/remove native top-level sections, apply a review to its draft or published source while preserving its status, or discard it. Three additional OAuth abilities read, apply, and restore Gutenberg site-wide colors, typography, and spacing through native Global Styles, using installed theme fonts and preserving unmanaged settings. Three further OAuth abilities read, apply, and restore Divi native Customizer typography, text/heading/link/accent colors, content width, and percentage-based section/row padding. Explicit module settings and presets retain precedence. A bounded credential-free native lifecycle audit records OAuth anchor and refresh-marker changes for connection diagnostics. Native transactions and request receipts protect retries; the existing runner retains other material operations. No native credential helper or developer tooling is required by this connection flow.

* Read a post or page.
* Read and reversibly update only the fixed site title and tagline identities. Callers never supply a WordPress option name; each value is version-guarded, exactly verified, and backed by a dedicated target-bound snapshot.
* Inventory and read bounded native navigation menus without exposing destination URLs or arbitrary menu metadata. Guarded structure updates can relabel, reorder, and reparent every existing item as one complete depth-limited tree while preserving destinations; dedicated target-bound snapshots support exact inverse restore.
* Read a content-free post/page lifecycle projection and bounded, permission-filtered author inventory. Guarded status, author, and UTC date updates require the current lifecycle hash, preserve the complete builder document, and verify exact read-back. Status snapshots are retained only for local failure recovery and cannot be passed to generic restore.
* Read and search site-owned image attachment metadata.
* Upload a bounded local PNG, JPEG, WebP, or GIF through a closed payload with principal-bound replay protection, exact read-back, and a provenance-limited Trash compensation route. Remote URLs, SVG, filesystem paths, and arbitrary attachment deletion are not accepted.
* Update a title only when the caller presents the current version hash.
* Create, publish, and trash workflow-created draft pages with a version guard. Separate closed draft-metadata and featured-media routes update only canonical slugs/bounded plain-text excerpts or one selected readable image attachment; each verifies the workflow document and restores locally on mismatch. Publishing preserves and verifies the builder document against a private snapshot; it is non-reversible because publication can have external effects. Managed SafeEdit reviews are excluded from generic draft Trash, publishing, metadata, and featured-media mutation. Retriable create workflows use a principal-bound, bridge-only opaque request ID so an exact response-loss retry returns the original draft instead of creating another.
* Create a server-owned Gutenberg or Divi review draft for a published page, inspect content-free lifecycle evidence, replace one guarded canonical text leaf, Button label/link pair, or site-owned Image/alt pair, remove one canonical semantic leaf when a same-parent sibling remains, or derive and recompile one canonical semantic duplicate immediately after its source, through dedicated typed routes, and merge only with exact source/review hashes. Review mutations preserve unrelated native attributes and siblings, snapshot and verify the review, and check the source remains unchanged. Reviews expire after seven server-owned days and are moved only to WordPress Trash by a provenance-guarded lifecycle task; merge snapshots the source and trashes the review; an unverified or expired clone is never mergeable.
* Create a fixed-title, bridge-marked disposable SafeEdit source draft through a purpose-bound idempotent route, then dispose only that same-principal marked workflow page with an exact hash. Disposal accepts its draft failure-cleanup state or final published state, but published cleanup denies while any managed review remains active. It accepts no caller title, content, metadata, generic source selection, or untrash path.
* Create a fixed-title, bridge-marked disposable taxonomy post draft through a separate purpose-bound idempotent route, then Trash only that same-principal marked post with an exact hash. It accepts no caller title, content, metadata, taxonomy assignment, generic post selection, or untrash path.
* Compose a workflow-marked blank Gutenberg draft from a closed semantic plan, with an expected hash, snapshot, and exact server `parse_blocks` read-back. It supports compiler-owned native ordered/unordered List blocks from bounded plain-text items and accepts no raw HTML/markup, arbitrary attributes, or generic content input.
* Compose a new Divi 5 document from an allowlisted, semantic plan. The installed compiler currently supports native Text, Button, Image, Blurb, and Call To Action modules with one to four equal-width columns; portable lists use a closed compiler-owned variant of the existing Text module.
* Create a new native Divi draft from an explicitly selected, version- and structure-guarded page reference when that source is exactly reproducible by the frozen Text/Button compiler subset. The bridge extracts semantic values and recompiles them; it never copies source serialized content, Divi metadata, styling, presets, arbitrary attributes, or code.
* Report the installed stable Divi 5 version and whether it is approved for the native compiler. Divi draft creation and composition currently fail closed outside the verified 5.9 and 5.10 lines plus exact 5.12.1.
* Read bounded, content-free Divi and Gutenberg builder profiles. BI-2R normalizes Gutenberg/Divi pages, Divi Library layouts, and Divi Theme Builder template/header/body/footer resources into closed source-kind, ownership, mutability, semantic-compatibility, structure-summary, version-compatibility, scope, and drift classifications. Public projections contain no resource IDs, guards, hashes, native paths, serialization, attributes, settings, conditions, or content. Only an explicitly selected current page proven by an existing semantic compiler retains minimal private selection evidence; Library and Theme Builder resources remain read-only and outside compilation.
* Resolve one exact page separately, then read a version- and structure-guarded semantic edit context containing only bounded canonical heading and paragraph paths and plain text. Gutenberg and the frozen Divi Text subset keep every builder attribute, serialized block, module setting, URL field, style, code, and unsupported component outside the response.
* Bootstrap a short-lived, same-origin in-memory browser session for the already authenticated REST user. This HTTPS-only endpoint returns no token or cookie value; WordPress sets HttpOnly cookies directly on the isolated browser context response.
* Capture a private snapshot before every material write.
* Restore a provenance-bound snapshot only when the caller presents the current version hash, verify the complete restored record and builder metadata, and recover the pre-restore state if verification fails.
* Read bounded typed ACF string, boolean, exact-number, constrained single-choice, ordered Checkbox, canonical nontransparent Color Picker, and canonical Date Picker values, update 1–20 selected top-level fields only on workflow-created page drafts, and restore them through a dedicated target/schema/hash-bound snapshot. ACF writes use `update_field` only, verify the whole eligible group and unchanged page document, and never return field values from material responses.
* Seal 1–20 typed ACF replacements in a private bridge-owned overlay beside one pending SafeEdit review, then atomically merge the native review document and overlay under exact source, review, schema, base-values, and overlay hashes. The bridge captures separate document evidence plus a combined document/ACF rollback snapshot, verifies both writes, and permanently discards proposed values on merge, cancellation, or expiry.

The bridge does not execute PHP, shell commands, SQL, or arbitrary filesystem operations. It does not accept generic `post_content`, raw HTML, arbitrary Gutenberg or Divi block data/attributes, code modules, CSS, JavaScript, shortcode payloads, arbitrary ACF configuration or post-meta writes, theme/plugin PHP edits, remote media URLs, SVG, or generic attachment deletion. Typed server-native Gutenberg composition is verified on disposable Local drafts, and the current agent-controlled Gutenberg/ACF fixture has completed genuine Block Editor save/reopen plus Site Editor reopen. Earlier Classic TinyMCE routing remains a negative compatibility case, not the current fixture state.

The bridge is called by the local `wp-agent` control plane. Material writes should be submitted through its versioned operation runner, which applies local environment policy and writes a redacted audit record before dispatching this bridge.

Bridge 0.74.0 adds Gutenberg Global Styles read, apply, and restore. The selected native resource is merged with compiler-owned design values while preserving unrelated settings. Apply snapshots prior content or absence; restore rejects later edits and verifies the restored resource. Global changes use the existing reviewed runner policy. The catalog has 90 operations; 82 are on the Bridge surface, including 21 read-only and 61 material fixed-denial Abilities. Source verification is separate from live rendered propagation.

Bridge 0.73.0 consolidates the demand-driven structural and reusable-resource milestone plus Site Builder V1's native menu and page-frame completion. SafeEdit may duplicate or remove one canonical top-level section, move one supported canonical leaf between two existing top-level sections, and append one uniquely selected current compiler-verified Divi Library resource to a managed Divi review by semantic extraction and recompilation. Site Builder may create a native menu, populate it with workflow-created pages, edit or restore its structure, delete only its own created menu, and assign one existing native Gutenberg page template with snapshot restore. Every operation remains independently approved, exact-request idempotent, snapshot-backed where applicable, source-preserving, and exact-read-back verified. Public intent uses semantic ordinals, workflow page keys, or exact labels only. Gutenberg reusable placement remains unsupported; no raw Library copying, in-place resource mutation, native paths, generic trees, attributes, settings, serialization, CSS, HTML, or JavaScript surface is added. The Bridge surface is 79 operations and the canonical catalog is 87.

Bridge 0.72.0 adds a closed compiler-owned design foundation for blank-site builds. Typed colors, typography scales, spacing, widths, shape, and elevation compile privately into workflow-owned Gutenberg blocks and Divi 5.9 modules when theme tokens or Divi presets do not yet exist; existing verified resources remain supported. It also models Divi's no-custom-Header/Footer state as an immutable restorable baseline and creates or removes only a marker-owned Theme Builder assignment container during guarded activation and compensation. CSS text, selectors, classes, generic settings, serialization, native IDs, preset creation, Global Styles mutation, and theme/plugin-owned resource mutation remain unavailable. The Bridge surface remains 72 operations and the canonical catalog remains 80.

Bridge 0.70.0 adds one closed atomic SafeEdit operation for replacing a canonical paragraph, heading, button, or image leaf with a different supported semantic kind at the same path. Gutenberg and Divi compile only their existing canonical native leaves; the operation preserves the parent hierarchy, sibling state, element count, and published source, requires material approval, returns exact semantic proof, and supports snapshot rollback. Same-kind edits remain on their existing narrow operations. Arbitrary blocks, modules, attributes, settings, serialization, CSS, HTML, JavaScript, cross-parent moves, and section edits remain unavailable. The Bridge surface is 71 operations and the canonical catalog is 79.

Bridge 0.69.0 freezes the ACT-2 capability declarations used by the source-complete FSE-3B control-plane coordinator. The Bridge operation surface remains 70 operations and the canonical catalog remains 78: FSE-3B composes the existing guarded build, publication, FSE-3A creation/cleanup, and site-shell assignment/restore lifecycles with separate approvals and exact reverse compensation. It adds no Bridge operation, combined create-and-assign approval, generic template mutation, native serialization, Navigation write, settings, CSS, HTML, or JavaScript surface.

Bridge 0.68.0 adds FSE-3A compiler-owned Header/Footer resource creation and guarded Trash cleanup. One closed Core intent compiles through Gutenberg or Divi into a new marker-owned database resource, remains unassigned, re-inventories with exact native and semantic proof, and preserves every prior resource and assignment. Creation and cleanup remain separate material operations from the existing site-wide assignment lifecycle. Navigation selection is explicitly unsupported until a shell-only native adapter exists. No generic template/resource creation, existing-resource mutation, native serialization, Navigation/Global Styles/preset mutation, arbitrary settings, or combined create-and-assign orchestration is added.

Bridge 0.67.0 adds DSR-1 as an opt-in, builder-neutral closed design-role layer. Core carries only versioned semantic roles and fixed responsive intent; exact existing theme tokens and Divi presets are resolved privately from complete current inventories. Gutenberg compiles only its verified native role subset and denies unsupported tablet-collapse, density, border, elevation, and radius roles. Divi 5.9 compiles exact existing presets plus fixed native container, density, column, collapse, and media branches. Both paths re-read runtime/resource evidence, preserve the existing exact design-resource application, verify exact native read-back, and compensate failed disposable drafts. No Global Styles, theme.json, Divi preset, CSS, arbitrary attribute, generic setting, reusable resource, or shell authoring surface is added.

Bridge 0.66.0 completes ACF-X3B for top-level page `post_object` fields in existing-page read, direct draft edit, snapshot/restore, and SafeEdit workflows. Public values use only an optional exact `{kind:"post-title",postType,title}` selector or an ordered unique list of those selectors. Exact permission-filtered resolution binds private post IDs plus identity/version evidence and rechecks the field's post-type allowlist, publish-only target status, cardinality, current readability, and disabled dynamic, taxonomy-filter, and bidirectional behavior before every material write. Native scalar/list storage and order are verified, whole-group and page state remain guarded, and public material responses remain value-free. New-page Post Object builds, post creation, status changes, bidirectional writes, generic relationship mutation, public IDs/guards, nested writes, Page Link, and Relationship remain unavailable.

Bridge 0.65.0 closes the first executable Gutenberg site-shell slice behind the same Core assignment contract and shared lifecycle already used by Divi. It may replace exactly one global Header or Footer `core/template-part` reference inside a published database-owned Site Editor template with one exact published database-owned template part. The write freezes content-free inventory, assignment ownership, resource version/structure, retained assignments, and conditions; requires site-wide approval and a private provenance-bound snapshot; changes only the native `slug` and `theme` reference; re-inventories exact read-back; and supports inverse restore. The existing Divi reference workflow now also accepts the private guards emitted by exact-label, compiler-verified `fork-only` Library selection, derives the canonical semantic subset, and recompiles it into a separately created workflow draft while proving the Library source unchanged. Theme/plugin templates, theme/plugin replacement parts, proposal-only custom records, implicit-theme references, resource creation/content mutation, in-place Library mutation, raw Library copying, Navigation, Global Styles, conditions, markup, attributes, settings, native IDs, and generic Full Site Editing remain unavailable.

Bridge 0.64.0 adds atomic semantic sibling insertion for the portable Heading, Paragraph, Button, and guarded Media Library Image family. One ID-free page-edit intent resolves an exact sibling and private image identity, while the material request accepts only the closed semantic element, `before` or `after`, and current review guards. The bridge compiles the new value through the existing Gutenberg or Divi native compiler, snapshots once, writes once, re-reads the exact native tree and semantic projection, proves the published source unchanged, and restores on mismatch. Markup, attributes, settings, trees, ambiguous media, incompatible Gutenberg Button parents, stale guards, and unsupported kinds deny.

Bridge 0.63.0 adds one value-free same-parent reorder operation for canonical Gutenberg and Divi SafeEdit review leaves. The request identifies an already-disclosed source path, exact destination sibling, `before` or `after`, expected semantic kind, and current review structure/context/version guards. Cross-parent, no-op, stale, unsupported, or noncanonical moves deny. The bridge snapshots first, moves the exact native compiler-owned leaf without accepting content or settings, verifies the complete semantic order and native tree after reload, proves the published source unchanged, and restores on any mismatch. The ID-free page-edit workflow owns planning, approval, immutable resume state, re-read, merge, cancellation, and redaction.

Bridge 0.62.0 completes the first cross-extension design-system and Site Editor boundary hardening. Divi page composition may apply only existing native module presets selected by an exact semantic role, native target, bounded preset ID, and current nontruncated inventory fingerprint; the bridge emits the compiler-owned `modulePreset` stack, verifies exact native read-back, and restores on drift without exposing preset values or accepting settings. The existing read-only `site.shell.read` route now also inventories effective Gutenberg templates and template parts plus database-owned Navigation and Global Styles resources as bounded, content-free ownership evidence. Core can produce a review-only Gutenberg shell proposal, but no Gutenberg shell write, restore, template replacement, navigation mutation, Global Styles mutation, operation, or material Ability is added. The existing guarded Divi Theme Builder assignment runtime is unchanged.

Bridge 0.47.0 completes BI-2R read-only reusable-reference intelligence. The existing builder-profile route classifies Gutenberg and Divi pages, Divi Library layouts, and Theme Builder template/header/body/footer records with one closed normalized model. Public CLI projections expose only safe labels and bounded ownership, mutability, compatibility, structural-summary, scope, and drift enums. Exact IDs and version/structure guards remain private and are retained only for explicitly selected current pages proven reusable through an existing semantic compiler. Library and Theme Builder resources remain read-only, receive no compiler evidence, and expose no serialized content, attributes, settings, conditions, paths, IDs, or hashes. No operation, REST route, Ability, WP-CLI command, builder module, preset, placement, cloning, forking, or mutation surface is added.

Bridge 0.50.0 completes ACF-X1B for top-level page `number` fields. Public contracts use bounded canonical decimal strings with at most six fractional places; JSON numbers, exponent/locale forms, and binary floating arithmetic remain unavailable. The bridge binds exact min/max/step constraints, required-empty behavior, and semantic schema drift while reusing guarded draft builds, direct updates, whole-group hashes, SafeEdit overlays, snapshots, restore, verification, and rollback. Current empty or off-constraint values remain readable and snapshotable for guarded repair or exact rollback; nested numbers remain mutation-inert. No operation, REST route, Ability, WP-CLI command, catalog entry, context, or nested write is added. ACF-X1A booleans, ACF-X0 effective-schema parity, and all prior boundaries remain intact.

Bridge 0.51.0 completes ACF-X1C for top-level single-value `select`, `radio`, and `button_group` fields. Public contracts use only canonical raw choice-key strings or the exact empty sentinel; `0` remains a real key and labels never cross the workflow boundary. The private effective schema binds required/null behavior, return format, ordered choices, and label hashes. Requested values must be current choices and clear requires optional plus allow-null, while current safe empty or orphaned keys remain readable/snapshotable for guarded repair and recovery-only restoration. Multiple, dynamic, custom, optgroup, HTML-label, and nested variants remain unavailable. The existing direct update, combined Gutenberg/Divi build, SafeEdit overlay, snapshot, restore, verification, rollback, redaction, and preservation lifecycle is reused without a new operation, route, Ability, WP-CLI command, context, block/module, presentation control, or nested write.

Bridge 0.52.0 completes ACF-X1D for top-level page `checkbox` fields. Public and sealed values are arrays of zero through 100 unique lexicographically sorted canonical raw keys; `[]` is the sole empty sentinel and `0` remains a real key. The private effective schema binds required state, return format, configured choice order, and label hashes while validated layout/toggle remain UI-only. Requested keys must exist in the guarded inventory and required clears deny; current safe empty or orphaned arrays remain readable/snapshotable for guarded repair and recovery-only restoration. Custom/saved choices, conditional logic, and nested variants remain unavailable. The existing direct update, combined Gutenberg/Divi build, SafeEdit overlay, snapshot, restore, verification, rollback, redaction, and preservation lifecycle is reused without a new operation, route, Ability, WP-CLI command, context, block/module, presentation control, or nested write.

Bridge 0.53.0 completes ACF-X1E for top-level page `color_picker` fields only in nontransparent string mode. Public and sealed values are exactly `""` or lowercase six-digit `#rrggbb`; native three-digit or uppercase stored values normalize privately to canonical lowercase six-digit form. The effective schema binds required state, `return_format=string`, `enable_opacity=false`, `format=hex6`, and `transparency=false`; valid palette, theme.json, wheel, and default settings remain editor-only, value-free, and hash-excluded. Required clears, active conditional logic, alpha/RGBA, array returns, malformed presentation settings, public coercion, and nested mutation deny. The existing direct update, combined Gutenberg/Divi build, SafeEdit overlay, snapshot, restore, verification, rollback, redaction, and preservation lifecycle is reused without a new operation, route, Ability, WP-CLI command, context, block/module, presentation control, or nested write.

Bridge 0.54.0 completes ACF-X1F for top-level page `date_picker` fields. Public, sealed, snapshot, and hash values are exactly `""` or strict Gregorian ISO `YYYY-MM-DD` dates for years 0001–9999; the bridge alone converts exact native `Ymd` storage in both directions. The effective schema binds required state, one of ACF's four official return presets, `format=iso-date`, and `storageFormat=Ymd`; the three official editor display presets and week-start setting are bounded UI-only state. Required clears, impossible or noncanonical dates, custom formats, current-date defaults, nonempty defaults, legacy save-format compatibility, conditional logic, Time/Date Time variants, and nested mutation deny. The existing direct update, combined Gutenberg/Divi build, SafeEdit overlay, snapshot, restore, verification, rollback, redaction, and preservation lifecycle is reused without timezone parsing or a new operation, route, Ability, WP-CLI command, context, block/module, presentation control, or nested write.

Bridge 0.55.0 completes ACF-X1G for top-level page `time_picker` fields. Public, sealed, snapshot, hash, and native values are exactly `""` or strict 24-hour `HH:mm:ss` wall-clock times. The effective schema binds required state, one of ACF's two official return presets, `format=hms24`, `storageFormat=H:i:s`, and `timezone=false`; the two official editor display presets remain bounded UI-only state. Required clears, malformed or ambiguous times, custom formats, nonempty/default-to-current behavior, legacy save formats, conditional logic, Date Time Picker, timezone conversion, and nested mutation deny. The existing direct update, combined Gutenberg/Divi build, SafeEdit overlay, snapshot, restore, verification, rollback, redaction, and preservation lifecycle is reused without parsing/coercion or a new operation, route, Ability, WP-CLI command, context, block/module, presentation control, or nested write.

Bridge 0.56.0 completes ACF-X1H for top-level page `date_time_picker` fields. Public and sealed values are exactly `""` or strict Gregorian `YYYY-MM-DDTHH:mm:ss` floating local date-times; native reads and writes mechanically convert only the separator to and from ACF's exact `Y-m-d H:i:s` storage. The effective schema binds required state, one of four official return presets, `format=iso-local-date-time`, `storageFormat=Y-m-d H:i:s`, second precision, and `timezone=false`; the four official display presets and week start remain bounded UI-only state. Required clears, impossible or malformed tuples, offsets, zones, fractions, timestamps, custom formats, nonempty/current-time defaults, legacy save variants, conditional logic, and nested mutation deny. The existing direct update, combined Gutenberg/Divi build, SafeEdit overlay, snapshot, restore, verification, rollback, redaction, and preservation lifecycle is reused without parsing or a new operation, route, Ability, WP-CLI command, context, block/module, presentation control, or nested write.

Bridge 0.57.0 completes ACF-X2A for top-level page `link` fields. Public and sealed values are exactly `""` or `{url,title,target}` with a safe root-relative or credential-free HTTP(S) URL, an optional bounded one-line title, and semantic target `same` or `new`; native ACF arrays and `_blank` remain bridge-private. Required state and `array`/`url` return format are schema-bound. Legacy numeric IDs and URL strings, unsafe schemes, protocol-relative or credentialed URLs, arbitrary targets, missing/extra members, nonempty defaults, conditional logic, and nested mutation deny. The existing direct update, combined Gutenberg/Divi build, SafeEdit overlay, snapshot, restore, verification, rollback, redaction, and preservation lifecycle is reused without a new operation, route, Ability, WP-CLI command, context, block/module, presentation control, or nested write.

Bridge 0.58.0 completes ACF-X2B for top-level page `image` fields. Public values are exactly `""` or an ID-free `{kind:"media-library-title",title}` selector; the bridge and local product state alone retain the resolved attachment ID, stable media version evidence, and MIME type. Required state plus ACF return format, library scope, allowed MIME types, dimensions, and byte bounds are schema-bound while preview size remains UI-only. Existing safe off-constraint values remain readable and snapshotable for repair and recovery. The bridge suppresses ACF attachment reparenting for the request, verifies the attachment remains unchanged, and never exposes IDs, URLs, hashes, or media metadata. New-page builds support `library=all`; `uploadedTo` is supported only for existing pages where the attachment is already parented and otherwise denies without reparenting. Missing, ambiguous, unreadable, nonimage, unsafe-MIME, stale, malformed, defaulted, conditional, and nested cases fail closed. No new operation, route, Ability, WP-CLI command, builder component, presentation control, context, or nested write was added.

Bridge 0.59.0 completes ACF-X2C for top-level page `file` fields. Public values are exactly `""` or an ID-free `{kind:"media-library-title",title}` selector; private state alone retains the attachment ID, stable WordPress-owned media evidence, and MIME type. The effective schema binds required state, `array`/`url`/`id` return format, `all`/`uploadedTo` library scope, exact B/KB/MB/GB byte bounds, and a closed document allowlist: PDF, RTF, DOC/DOCX, XLS/XLSX, PPT/PPTX, TXT, and CSV. Image, audio, video, archive, executable, SVG, unknown, extension/MIME-mismatched, missing, ambiguous, unreadable, stale, defaulted, conditional, and nested cases fail closed. Existing safe off-constraint values remain repairable and recovery-restorable. Exact-title search is permission-filtered and family-aware, so same-title Images do not create false File ambiguity. ACF reparenting is suppressed and verified; existing-page `uploadedTo` requires an already-parented attachment and new-page `uploadedTo` denies before compile. No upload, operation, route, Ability, WP-CLI command, builder component, presentation control, context, or nested write was added.

Bridge 0.60.0 completes ACF-X2D for top-level page `gallery` fields. Public values are ordered arrays of zero through fifty unique ID-free `{kind:"media-library-title",title}` selectors; private state alone retains the corresponding ordered unique attachment IDs, stable WordPress-owned media evidence, and MIME types. The effective schema binds required and item-count state, `array`/`url`/`id` return format, `all`/`uploadedTo` library scope, safe GIF/JPEG/PNG/WebP allowlists, dimensions, and B/KB/MB/GB byte bounds, while preview size and insert direction remain bounded editor-only settings excluded from the schema hash. Exact-title resolution is permission-filtered, paginated, family-aware, and repeated for every ordered item. Duplicate native or requested selections, missing, ambiguous, unreadable, nonimage, unsafe-MIME, stale, malformed, defaulted, conditional, and nested cases fail closed. Existing safe off-constraint lists remain repairable and recovery-restorable, but duplicate legacy lists are incompatible. ACF reparenting is suppressed for the whole request and every attachment's parent and guarded media state are verified unchanged; existing-page `uploadedTo` requires every attachment already parented and new-page `uploadedTo` denies before compile. Whole-list native order is verified and restored atomically on failure. No upload, operation, route, Ability, WP-CLI command, builder component, presentation control, context, or nested write was added.

Bridge 0.61.0 completes ACF-X3A for top-level page `taxonomy` fields. Public values use only exact canonical ID-free `{kind:"taxonomy-term-slug",slug}` selectors: Select/Radio store `""` or one selector, while Checkbox/Multi Select store ordered arrays of zero through fifty unique selectors. The effective schema binds the registered public page taxonomy, required/allow-null behavior, object/ID return format, appearance-derived multiplicity, fixed registered-taxonomy choice source, and disabled Create Terms, Save Terms, Load Terms, legacy combined load/save, dynamic choice filters, and bidirectional relationships. Discovery uses only the existing shared-runner taxonomy reads; the bridge privately resolves integer term IDs, rechecks taxonomy-wide and per-term assignment permission, and binds stable term/taxonomy/slug/name/description/parent evidence while excluding volatile usage count. Exact native scalar/list storage and order are verified. ACF writes snapshot and prove native page-term relationships unchanged; unexpected relationship drift receives guarded compensating restoration. Semantic term drift, duplicate or malformed values, term creation, post-term synchronization, active ACF taxonomy query/list filters, conditional logic, and nested mutation fail closed. Cancellation and expiry discard a sealed overlay through provenance/hash validation even after taxonomy, term, filter, or permission drift. No operation, route, Ability, WP-CLI command, catalog entry, builder component, context, or nested write was added.

Bridge 0.33.0 adds typed inline images to the portable page compiler. Callers provide only an existing site attachment ID and required bounded alt text. Gutenberg and Divi derive the image URL from the verified image attachment and compile their own native Image block/module; prototype `src`, remote URLs, raw attributes, styling, and arbitrary block settings remain prohibited.

Bridge 0.38.0 extends guarded SafeEdit review-text replacement across canonical Gutenberg headings/paragraphs and the existing frozen Divi Text heading/paragraph subset. Its exact closed request binds the review version, native structure, semantic context, selected path, and current text; the bridge snapshots before mutation, verifies exact native read-back and source immutability, and restores the review on verification failure.

Bridge 0.39.0 adds atomic SafeEdit button replacement for canonical Gutenberg Button blocks and the existing frozen Divi Button module. The exact request binds the current label and safe link plus the review, structure, and semantic-context guards; only the native label/link pair changes, while presentation settings, link target, sibling modules, and the published source remain unchanged.

Bridge 0.40.0 adds atomic SafeEdit image replacement for canonical Gutenberg Image blocks and the existing frozen Divi Image module. Human intent and semantic context select images only by exact Media Library title and alt text. The control plane resolves private attachment guards, while the bridge freshly verifies the site-owned image, derives its URL server-side, changes only the native image identity/alt fields, and preserves every unrelated attribute, setting, sibling, and the published source.

Bridge 0.41.0 adds value-free SafeEdit removal of one canonical Gutenberg heading, paragraph, Button, or Image block, or one existing frozen Divi Heading/Text, Button, or Image module. The request binds the exact path, kind, review version, native structure, and semantic context. Removal is allowed only when a canonical same-parent sibling remains; WordPress snapshots first, verifies the exact expected native tree and advanced structure/context fingerprints, preserves the published source, and restores the review on any mismatch.

Bridge 0.42.0 adds value-free SafeEdit duplication of one canonical Gutenberg heading, paragraph, Button, or Image block, or one existing frozen Divi Heading/Text, Button, or Image module. The bridge derives the selected semantic leaf and recompiles a new compiler-owned native sibling immediately after it without copying source serialization, arbitrary attributes, or presentation settings. Exact tree read-back, unchanged source and prior elements, one new equal semantic sibling, advanced structure/context fingerprints, safe bounds, snapshot rollback, and separately guarded merge remain mandatory.

Bridge 0.43.0 adds a private scalar ACF overlay for pending SafeEdit reviews. Staging validates a closed typed update against the published source without changing source or review content. A dedicated merge binds source/review versions, field-group structure, whole-group base values, and the sealed overlay hash; it applies the native review document and ACF values atomically, returns separate document and combined ACF snapshot evidence, verifies full read-back, and discards proposed values on merge, cancellation, or expiry.

Bridge 0.36.0 adds the guarded semantic edit-context read for one explicitly selected Gutenberg or Divi page. Bridge 0.39.0 extends its bounded semantic projection with canonical Button label/safe-link pairs, and 0.40.0 adds canonical Image title/alt pairs without exposing attachment IDs or URLs. It requires current version and structure fingerprints and exposes no serialized builder state, attributes, settings, styling, code, or unsupported module content.

Bridge 0.35.0 adds bounded ordered and unordered lists to the portable page compiler. Callers provide only one to fifty plain-text items and a closed list style. Gutenberg compiles native List/List Item blocks; Divi compiles escaped canonical list content inside the existing frozen Text module. Caller HTML, nested list markup, attributes, and presentation controls remain prohibited.

Bridge 0.34.0 adds a dedicated guarded navigation operation that appends only editable workflow-created draft pages as native root page links. WordPress derives titles and destinations, preserves every existing item, records exact created-item evidence, and can later remove only that append through its dedicated destructive restore route.

Bridge 0.32.1 adds an independent purpose-bound disposable post source for taxonomy evidence. Its create and Trash routes are separate from SafeEdit, use their own principal-bound idempotency namespace and provenance markers, and never expose generic post creation, taxonomy mutation, or post Trash.

Bridge 0.32.0 adds a purpose-bound disposable SafeEdit source draft plus guarded final/failure Trash route. The bridge mints and verifies source provenance, binds it to the creating principal and exact idempotent creation fingerprint, permits no caller title or content, and refuses published cleanup until any managed review is terminal. The route is not a generic published-page Trash surface and has no untrash or direct terminal-replay API.

Bridge 0.31.1 preserves the closed ACF update contract while splitting malformed shape, guard, and value-list failures into bounded machine codes. It also separates internal WordPress route URL parameters from ACF query guards so guarded reads work correctly on WordPress 7. This allows the control plane to diagnose a rejected request without exposing the remote message, request body, or field values.

Bridge 0.31.0 adds guarded workflow-draft ACF scalar updates plus a dedicated inverse restore. Both require administrator schema access, page edit permission, exact page/schema/whole-group value hashes, supported applicable top-level text/textarea/URL fields, and public ACF APIs. Updates deny no-ops, snapshot the complete eligible group, verify requested and sibling fields plus the unchanged core page document, and compensate locally on mismatch. Restore applies only captured ACF values while preserving the current core page, creates an inverse snapshot, and is denied by generic snapshot restore. Material responses expose no field values.

Bridge 0.30.0 adds a bounded ACF values read for one applicable selected group. It requires both administrator-level schema access and edit permission on the page, uses initialized public ACF APIs only, binds the request to the selected-group structure fingerprint, and returns only top-level text, textarea, and URL values under strict typed limits.

Bridge 0.29.0 extends the fixed read-only builder profile with a bounded ACF schema variant. It uses initialized public ACF schema APIs only and returns normalized field-group inventory, selected field/layout structure and safe location-rule projections, registered field types, and registered ACF Block metadata. It never reads field values, defaults, choices, instructions, conditional logic, render paths/callbacks, assets, settings, Local JSON files or paths, or ACF stores; database and local groups are treated identically only after the same normalized API projection.

Bridge 0.28.0 adds guarded Gutenberg v1 preset bindings to the existing closed native composition request. Only current theme-origin color, font-size, and spacing preset IDs may be bound to fixed section, heading, paragraph, and button roles after an exact inventory fingerprint and native core-block support check. The bridge generates its own canonical attributes, classes, styles, values, CSS, markup, or arbitrary block data. Bridge 0.27.0 adds a read-only, theme-origin design-token inventory for Divi and Gutenberg planning. It reports only sorted, bounded preset slugs and safe names from the active theme's theme.json color, font-size, and spacing-size sets; it never returns token values, CSS variables, styles, user/global-style records, Divi settings, content, templates, patterns, ACF, or write effects. Bridge 0.26.0 adds guarded canonical Gutenberg reference composition for one-shot blank workflow drafts. It accepts only the existing compiler's exact unstyled core paragraph, heading, group, columns, column, buttons, button, and separator trees; it derives plain semantic values, recompiles them, snapshots the target, verifies target read-back and source guards, and never copies source serialization or attributes. The fixed bridge surface is now 48 operations: seventeen read-only adapters and thirty-one discovery-only material Ability definitions.

Bridge 0.25.0 extends the existing read-only builder profile with a bounded server-parsed Gutenberg profile. It reports registered block names and fixed portable-plan support plus permission-filtered non-Divi, non-review pages and content-free structural fingerprints. It returns no attributes, content, HTML, URLs, styles, shortcodes, ACF, pattern, or template data. Bridge 0.24.0 adds an explicit SafeEdit review cancellation endpoint. It requires the exact source/review target and both expected hashes, moves only a pending review to Trash, preserves the published source hash, records a cleanup-complete `cancelled` terminal state, and returns that state only for an exact replay. Bridge 0.23.0 extends explicit rate-limit cleanup to every throttled read family, matching the existing atomic fixed-window enforcement. Bridge 0.22.0 adds request-scoped audit correlation for runner-originated REST work. A canonical runner-issued UUID is echoed only as an `X-WP-Agent-Audit-ID` response header on this bridge namespace; it grants no authority and is never stored in request bodies, snapshots, or bridge state. The runner requires the exact echo before accepting any correlated response. Bridge 0.21.0 adds fixed-ID reads and reversible writes for only `blogname` and `blogdescription`. It accepts no caller-controlled option key, rejects stale/no-op/unsafe values, and uses dedicated self-validating snapshots that generic restore cannot consume. Bridge 0.20.0 adds bounded native navigation inventory/read plus version-guarded relabel, reorder, reparent, and dedicated restore operations. Destination fields affect concurrency checks and are hash-verified across writes without being returned or stored raw in snapshots. Bridge 0.19.0 adds content-free lifecycle reads, permission-filtered author inventory, and hash-guarded status, author, and UTC date updates. Lifecycle fields participate in post hashes, snapshots, restore compatibility, and full-document preservation checks. Bridge 0.18.0 adds permission-filtered post/page taxonomy inventory, bounded term discovery, and exact full-assignment replacement with taxonomy-bound private snapshots and an inverse restore route. It only assigns existing terms, never creates or deletes terms, verifies the complete ordered assignment, and does not expose term snapshots through generic snapshot restore. Bridge 0.17.0 adds bounded local image upload plus guarded Trash compensation. It accepts only strict base64 PNG/JPEG/WebP/GIF bytes up to 5 MiB, validates filename/MIME/signature agreement and optional metadata, persists principal-bound create replay records and immutable attachment markers, and denies Trash for any attachment it did not create for that principal.

Bridge 0.16.0 adds closed, reversible metadata updates for image attachments: title, alt, caption, and description are version-guarded, exact-readback verified, and backed by target-bound media snapshots with a dedicated restore route. Bridge 0.15.0 adds a workflow-draft-only, closed featured-media update for one readable image attachment or explicit clear. It version-guards before snapshot, verifies the requested thumbnail ID and unchanged workflow document/builder state, restores locally on mismatch, and records featured media in new snapshots without breaking old snapshot restore. Bridge 0.14.0 adds a workflow-draft-only, closed slug/excerpt metadata update with collision preflight, exact read-back, local recovery, and slug-aware snapshots. Bridge 0.13.0 adds the guarded, non-reversible workflow-draft publish transition: it snapshots and verifies title, excerpt, content, builder, workflow marker, and Divi state, restoring locally on verification failure while never claiming external publication effects can be undone. Bridge 0.12.0 verifies rollback-snapshot provenance before mutation, binds snapshot payloads to immutable target provenance, restores the pre-restore state after reload/read-back failure, and prevents generic Trash or terminal snapshot restore from bypassing the SafeEdit lifecycle. Bridge 0.11.0 added bounded SafeEdit lifecycle inspection plus seven-day, provenance-guarded review expiry and Trash cleanup. It registers a fixed Ability definition for every bridge operation; seven bounded read adapters are available through WordPress Abilities REST. Sixteen material definitions are discovery-only, expose an empty closed input schema, and return a fixed 403 so they cannot bypass runner policy, approval, idempotency, audit, or recovery. Bridge 0.10.0 added catalog-wide Ability parity. Bridge 0.9.0 added one-shot reference composition for an explicitly selected compiler-canonical Text/Button page. Both the source version and structure fingerprint must match, the target must still be a blank workflow-created Divi draft, and the source must remain unchanged through verification. Unsupported modules, styling, presets, arbitrary attributes, custom/core/freeform blocks, and noncanonical trees fail closed before the target write. Bridge 0.8.0 added the bounded, permission-filtered Divi reference inventory with safe labels and content-free structural fingerprints. Bridge 0.7.0 added the first read-only Divi builder profile, and 0.6.0 added the stable Divi version descriptor and compiler gate; the compiler remains pinned to the verified 5.9 line.

== Changelog ==

= 0.193.36 =
* Receive updates from the preconfigured public GitHub release channel using the bundled Plugin Update Checker, with normal WordPress update and auto-update controls.
* Preserve existing custom signed update connections until an administrator explicitly switches channels.

= 0.193.35 =
* Review shared workflow reports as dashboards with scored categories, assessment coverage, prioritized findings, impact, remediation and verification actions.
* Track proposed before/after changes, completed apply steps, verification and recovery from saved task results.
* Convert earlier reports through the connected agent while preserving their evidence and unknowns; pending report links open before policy adoption.

= 0.193.34 =
* Discover exact artifact schemas and validate audit findings before saving a report. All playbooks can bind findings to the claimed task and receive a ready report link.
* Read report-workflow grant requirements before claiming a task. Scoped audit bookkeeping works for selected published pages without requiring live-content changes.
* Use compact identity, task and knowledge responses, or filtered capability discovery with installed input schemas.
* Inspect bounded anonymous public page evidence from the WordPress host when client fetches are unavailable.

= 0.193.33 =
* Read captured colors, typography, spacing and components in a visual Design System guide. Review or hide observations from future agent reuse.
* Review Site Brief guidance and detected site facts independently. Draft changes with an owner or connected agent and approve them through WordPress.
* Compare new design captures with saved evidence. Connect proposed guidance to a reviewed task and review it before adoption.
* Start SEO/AEO/GEO, performance, security and accessibility audits as saved tasks with your connected AI.

= 0.193.32 =
* Keep Gutenberg section styles editable in the native editor and load generated rules when the theme serves its minified stylesheet.
* Use The SEO Framework’s native sitemap and robots settings for review, apply and restore.
* Highlight private previews and revision requests in the task workspace, keep approval disabled while current owner feedback needs revision, and improve review and setup on narrow screens.

= 0.193.31 =
* Compile a builder-neutral comp layout into editable Gutenberg and Divi drafts, with a region map and native recovery.
* Add responsive grid tracks, fluid padding and gaps, and bounded backdrop blur to Gutenberg styling.
* Save site-owner feedback for an exact draft or proposal preview and carry it into the connected revision task.
* Populate Design System evidence from effective block templates, template parts and saved patterns.

= 0.193.30 =
* Turn a design reference inspected by a connected agent into an editable Gutenberg or Divi draft, with a reviewable map of represented and unresolved regions.
* Keep viewport observations with the exact preview, add Gutenberg image focal position and responsive controls, and isolate private style revisions from the source page.
* Populate provisional site design during setup when native evidence is available. Refresh from current evidence without replacing approved design policy.
* Manage supported post, page, homepage and site-wide metadata through The SEO Framework, with exact review, scoped approval and recovery.

= 0.193.29 =
* Let a signed-in administrator remove declined requests from the list while retaining decision history.
* Replace declined or revoked requests with a fresh, linked request that needs its own approval.
* Preserve managed access limits and exact retries when removal or replacement is interrupted.

= 0.193.28 =
* Show the selected permission request’s current approval status and distinguish other pending requests.
* Keep permission decision notices attached to the correct request when navigating between pages.

= 0.193.27 =
* Reconnect authoring helpers after a native Bridge update while preserving edits made with AI tools off.
* Keep larger native page proposals reviewable and recoverable in the same saved task.

= 0.193.26 =
* Update ChatGPT and Claude connection directions and explain local-site reachability in plain English.

= 0.193.25 =
* Preview proposed Divi page edits in an isolated native draft without changing the original.
* Reconcile shared Divi settings before AI editing resumes after helper handoff.
* Require explicit live-change permission for shared site and Divi design updates.
* Explain shared fonts, colors, spacing and reusable-style permissions in plain English.

= 0.193.24 =
* Keep native builder edit proposals and their exact review evidence in saved tasks.
* Reconcile native theme settings and Site Editor resources after AI helpers are turned off.
* Search and browse editable pages and posts without losing task choices.
* Keep native administration working before the secure agent connection is ready.
* Accept installed Divi version metadata while preserving unknown-attribute and embedded-code permission checks.

= 0.193.23 =
* Explain AI access requests in plain English, with named content, exact file scope, lasting access and live-site effects.
* Complete reviewed Gutenberg post draft publication and restoration with native permissions and exact recovery checks.
* Return permission-aware native editor links for page-builder reads and edit receipts.
* Refresh native theme settings before deployment so stale caches do not stop follow-up changes.

= 0.193.22 =
* Extend the existing guarded review workflow to Gutenberg posts, including title/text/native edits, root-block removal, merge snapshots and undo. Expose reviewable post types and specific unsupported-type errors.

= 0.193.21 =
* Make code workspace permission requests discoverable to connected agents and administrators; distinguish CSS/assets from hosted PHP restrictions.

= 0.193.20 =
* Native compilation preserves declared column margins in saved wrappers and keeps raw custom font sizes for Gutenberg Save/reopen; Core applies fluid typography on the frontend. Existing exact review/apply/restore guards remain in place.

= 0.193.19 =
* Build single-post/custom-post-type and search layouts through the existing native template review flow. Render bound post content, author/date/terms, native search controls, public search results and empty states. Preserve saved-task context, drift checks and exact apply/restore.

= 0.193.18 =
* Build native Query Loop and archive layouts through existing template reviews. Discover posts-index, taxonomy and public post-type contexts; stage inherited archiveQuery cards, headings and empty states; preview public results, images, excerpts and pagination; apply and restore with exact guards. Saved tasks retain archive context and invalidate changed results.

= 0.193.17 =
* Edit effective native template layouts through typed private reviews, exact retained blocks, selected-page preview, guarded database apply and snapshot restore. Disclose masked staged theme files and page-template selection; link template previews to the existing saved task.

= 0.193.16 =
* Preserve native templates, parts, user Global Styles, custom CSS and theme settings across managed theme releases. Continue unchanged shell reviews with the same IDs through rebind-shared-shell-review; retain previous native state for independent recovery.

= 0.193.15 =
* Keep existing native header/footer review dependencies valid across their exact authorized draft-menu publication. A signed publication snapshot proves the unchanged menu transition; later content/metadata edits still invalidate the review.

= 0.193.14 =
* Build staged native templates around actual WordPress page title/content, retaining editable page IDs and shared shell/menu owners. Publish an existing draft menu in place with explicit live-change authority; recover its draft through the existing navigation snapshot workflow.

= 0.193.13 =
* Prepare apply, verify and recovery in one reviewed task. Bind native snapshots, guard hashes and provenance keys from completed earlier same-plan receipts; native schemas, permissions and drift checks remain authoritative.

= 0.193.12 =
* Show prepared task previews as ready, lead with the private preview action, keep agent instructions secondary, and name native header/footer editors clearly. Recheck preview before the first planned write regardless of phase label.

= 0.193.11 =
* Keep one exact code/page/shell/menu preview in the existing durable task, expose its current private link and native editors, refuse stale review and the first planned write regardless of phase label, and preserve native recovery after publication.

= 0.193.10 =
* Bind native shared-shell reviews and their unpublished navigation to the existing private page preview. Review/menu/target guards invalidate stale links; rendering publishes and assigns nothing. Menus appear as always-visible native links.

= 0.193.9 =
* Preview a complete staged native page with same-owner template parts, theme.json, inherited/user styles, and existing site images/fonts through the ordinary MCP candidate workflow. Private previews preserve native template-part wrappers and leave live files unchanged.

= 0.193.8 =
* Preview selected staged native theme markup and CSS through the connected agent before applying a code candidate. Return a private WordPress URL bound to its exact candidate revision and owner state.
* Keep generated PHP unexecuted, block scripts and network assets in component previews, and reuse existing guarded candidate apply/restore. No customer runtime or toolchain setup is required.

= 0.193.5 =
Site Brief and Design System now begin with a readable site resource index and support guarded population of actual native values. Capture includes Divi 4 shortcode styles beyond the former 100-record sample. Palette sources are visible; unused Core defaults and synthesized Divi fallback values are separated from site declarations. Human corrections, acceptance, source protection and history are retained.

= 0.193.4 =
* Resolve signed code-candidate site/user identity and owner during subsequent revision, deployment and restore authorization.
* Integrate Restricted Site Access with exact short-lived managed theme/runtime deployment probes; preserve public restrictions, native settings, no-store responses and existing deployment guards.

= 0.193.3 =

* Separate post and code-owner scopes in combined grants; isolated review edits do not change their published source.
* Require live-change permission for publication, and accept existing mixed-case child-theme directory identities.

= 0.193.2 =
* Integrate the task workspace with destination-scoped migration grants and bounded MCP theme-probe diagnostics.

= 0.193.1 =
* Keep signed SEO/security/accessibility reports inside their original task without accepting policy. Prioritize one next action, preserve task-form input after validation errors, and retain matching performance baseline/finding/follow-up context.

= 0.193.0 =
* Add one WP Agent workspace organized around migrating, optimizing, creating and editing pages, with saved task scope, contextual native reviews and connected-agent handoff.
* Populate the Design System from bounded native Gutenberg/Divi evidence; preserve human corrections and policy through reviewable proposals.
* Add a WCAG 2.2 AA accessibility playbook with native contrast/content findings, remediation planning, reviewed shared design fixes, separate policy proposals and snapshot restoration. Rendered and assistive-technology testing remain separate.

= 0.192.0 =
* Add public PageSpeed/CrUX performance findings, cache/plugin/runtime discovery and reviewed native image/LiteSpeed fixes. Measurements and recommendations never grant fix authority; provider/browser/real-site acceptance remains separate.


= 0.191.0 =
* Bootstrap governed Site Brief and Design System records on activation/setup, with human acceptance, exact revision guards, provenance, drift detection and history.
* Add persistent site knowledge and versioned SEO/AEO/GEO audit and security review playbooks, compact builder context, and outcome-based tasks over existing guarded capabilities.
* Procedures and task plans never grant execution permissions; browser/editor/provider acceptance remains separate.

= 0.190.3 =
* Preserve bounded, secret-free native theme probe diagnostics through the MCP execute-ability error response so clients can distinguish HTTP, marker, transport, and authority failures.

= 0.190.2 =
* Return secret-free theme probe HTTP, marker, transport, and authority diagnostics while retaining the exact anonymous bootstrap refusal and active-theme recovery guard.
* Scope cross-site migration journal grants to signed local destination targets rather than remote source IDs.

= 0.190.1 =
* Add guarded native-output handoff: fingerprint outputs and generated helpers, disable authoring plugins and recovery loaders together, run selected fresh anonymous assertions, and reconnect without regenerating outputs.
* Preserve source/vendor/native permission boundaries, explicit live approval, drift refusal and native administrator recovery. Helper-free assertions remain distinct from editor, mobile and provider acceptance.

= 0.190.0 =
* Add shared native builder document and configuration candidates with explicit adapter identity, provenance, version/dependency guards, snapshots, transactions and restoration. Divi supplies native block/shortcode parsing and installed module definitions; additional connectors use the same lifecycle.
* Ship a Divi skill through the installed builder API. AI inspects native definitions and code to author module-specific content and independent site functionality.
* Add credential-bound capability proposals, explicit signed-in administrator approval, scoped grants, guided independent plugin/child/block theme creation, readiness and revocation. Preserve native permissions and protected-source mode.
* Deploy and activate managed native theme releases through anonymous probes and independent recovery; preserve protected parents and site runtime when Bridge is disabled.
* Complete general native/code CLI discovery while retaining current site-key/OAuth guidance. Qualification uses disposable WordPress; native editor, existing-site rendering and live provider acceptance remain separate.

= 0.189.30 =
* Create an independent runtime within an already-authorized nonexistent scope without requiring existing/vendor-code ownership attestation. Keep existing-code/vendor/source guards and return validation errors to the editable form without prematurely changing scope.


= 0.189.27 =
* Store the destination runtime-loader template as a non-plugin file so the native uploader selects WP Agent Bridge as its activation target. Verify folder-specific discovery and the native upgrader activation link, while keeping generated independent destination plugins discoverable.


= 0.189.26 =
* Replace manual workspace identifiers with scope checkboxes, ownership/protection and capability status; create, activate and authorize independent destination runtime plugins from the administrator page. Protected source sites cannot enable code writes.
* Deploy managed runtime code as immutable releases after an anonymous WordPress loopback probe. Preserve the active version on probe failure; provide fatal-error and nonce/current-state-guarded administrator recovery independently of Bridge. Unmanaged hosted PHP remains protected.
* Share registered-block/schema/runtime-owner admission across general native writing, context reading and local subtree targeting, including unowned Core groups and independent custom blocks. Preserve prohibited/shared-resource descendant boundaries, exact guards, merge/snapshot/restore and outside-subtree bytes.
* Requires PHP 8.2 for the guarded workspace API. Packaged fixture qualification does not establish hosted rendering or native Save/reopen acceptance.


= 0.189.24 =
* Permit guarded migration-journal adoption of valid additive form-label links when every literal label, choice value, delivery setting and other form attribute is unchanged. Preserve exact review receipts and drift guards.

= 0.189.23 =
* Offer client-specific setup for Codex, Claude Code, ChatGPT and Claude; retain OAuth for app connectors and remove the standalone Basic MCP route.

= 0.189.22 =
* Guarded typed local insertion and exact additive migration journal adoption.
* Retain one native image unchanged as an editable YouTube dialog poster, with keyboard openers and verified wrapper adoption.
* Preserve owned-image exact-retention layouts and refuse active CSS class collisions with stripped native blocks.
* Consolidates 0.189.20 form label links and 0.189.21 signed native automatic updates in one bootstrap package.

= 0.189.21 =
* Connect a public signed release feed once and use native WordPress updates. Validate release signatures, package SHA-256 and compatibility before replacement; preserve WordPress automatic-update opt-in.

= 0.189.20 =
* Add optional typed links on unique non-overlapping form-label text, including native fieldset questions. Preserve label wording, field identity, provider values and content accounting; reuse the existing URL guard and cross-site map, protect new-tab openers, and retain links through native editing.

= 0.189.19 =
* Recover literal stock pricing-card copy into native headings, paragraphs and columns while retaining unresolved source findings and refusing dynamic, conflicting responsive or conditional values.
* Extend native YouTube dialogs with typed poster geometry, alternative text and a second opener. Reuse native image validation/CSS, defer playback until activation, retain focus return and preserve the values through native block editing.

= 0.189.18 =
* Extend the existing guarded native section insertion with an optional exact parent target for page-local Gutenberg div groups. Preserve every existing parent and sibling byte, native styles, source content, review guards, snapshots and idempotency. Top-level and Divi requests retain their original behavior; shared resources and semantic site containers remain refused.

= 0.189.17 =
* Count authored native form labels, help, choice labels, rendered email placeholders, submit text and notes in source extraction and destination accounting. Existing journal measurements require explicit guarded reconciliation; update the source Bridge before obtaining a fresh form export.
* Check authored dependency fragments with the same page owner and full prior document paths while retaining exact saved-body and non-overlapping content-occurrence guards.
* Adopt a completed guarded review that adds only a unique new native Core wrapper anchor. Preserve all prior tree, copy, styling, source findings and measured units; refuse existing anchor replacements, ID collisions and unrelated changes.

= 0.189.16 =
* Include video poster attachments in cross-site media remapping and existing draft create/revise lock sets. Missing media mappings remain explicit dependencies; source IDs are never reused on the destination.


= 0.189.15 =
* Add an editable closed YouTube video dialog with lazy privacy-enhanced playback and native accessible close/focus behavior.
* Add administrator-only saved-form staging qualification with page/form/profile guards, explicit transmission, typed consent, and durable separate receipts. Drafts remain publicly disabled; provider/mail transport acceptance requires independent delivery confirmation.


= 0.189.14 =
* Reconcile a styled migration draft against the exact native body produced by the read-only authoring compiler. Preserve the strict byte match, source accounting, and no-content-write guard.

= 0.189.13 =
* Add a revocable, HTTPS-only site-key MCP route for Codex, with one hashed key per administrator and setup-page creation/removal. Keep the existing Application Password and OAuth routes.
* Accept legacy measured migration journal records without a body fingerprint only when the exact prior full native baseline and guarded review-merge receipt still match; retain content-accounting and drift refusals.
* Let a destination administrator attach one existing, field-compatible delivery profile to a disabled draft form through a guarded review. The journal adopts only that exact receipt-backed binding, retains measured units and clears validation evidence; published-page and provider-switch edits remain refused.

= 0.189.12 =
* Allow bounded typed inline edits and link insertion in ordinary native paragraphs/headings without compiler-owned presentation metadata. Preserve their native attributes, wrapper and surrounding page bytes; raw HTML, stale guards and shared-boundary edits remain refused.
* Adopt an exact own guarded review merge into a migration session without recomposing the page or discarding measured content counts. Verify the prior full native baseline and current receipt-bound draft; support count-preserving paragraph link and form display-label changes, advance matching URL aliases, and invalidate prior validation evidence. Unrelated drift and content-removing edits require reconciliation.

= 0.189.11 =
* Edit an existing native form's typed fields, option labels/values and copy through guarded read-review-block/replace-review-block without requiring presentation metadata or recomposing the page. Preserve native styles, editor metadata, surrounding bytes and the existing delivery binding; retain review ownership, version guards, replay protection and snapshots. Merging a draft review keeps its source page a draft.

= 0.189.10 =
* Allow explicitly consented standalone Mailchimp signup without forwarding the signup to a Google Form. Reuse guarded administrator-owned profiles, request replay protection and administrator fallback/quarantine; preserve the existing Google Forms plus optional newsletter workflow.
* Retain supported Divi YouTube URLs as editable native baseline links instead of treating them as missing local media. Embedded playback, consent/modal UI and poster fidelity remain explicit pending work, with no player loaded by the baseline.
* Classify valid isolated JSON-LD code as metadata-transfer work for the existing owner, not a missing visual embed. Draft metadata staging remains separate from the existing published-target workflow.

= 0.189.9 =
* Transfer public PDF attachments through guarded MCP media export and upload, so document links such as resumes need not remain unresolved during migration. Existing image-only uploads keep their original contract.
* Add revision-guarded media prepare/execute and exact-byte existing-attachment binding to migration sessions. Retain public-source restrictions, 5 MiB bounds, MIME/extension verification, file confinement, and durable retry keys.

= 0.189.8 =
* Keep measured migration page content after a guarded metadata-only rebind, including SEO social-image edits, using a separate body/dependency fingerprint. Full native drift and validation guards remain unchanged.
* Preserve native reconciliation requirements after lost-receipt recovery and later human edits. Existing erased measurements require exact saved-body reconciliation; they are never inferred from source reports alone.

= 0.189.7 =
* Enable administrator-configured Google Forms forwarding in the existing native form, with repeated grouped-choice fields, administrator-email fallback/quarantine and explicitly consented optional Mailchimp signup.
* Bind submissions to saved public forms and current destination profiles; keep drafts/previews disabled and track inquiry and newsletter outcomes separately to prevent duplicate retries. Transport acceptance does not verify storage, inbox delivery or subscription.

= 0.189.6 =
* Candidate: extend the existing native form with telephone, radio, grouped checkbox, stable option values and editable help; add guarded destination delivery profiles and disabled MCP previews. New provider transmission remains unavailable pending explicit implementation authorization.
* Supersede the site-specific RYP Waitlist ZIP; preserve the migration media, selection and native-column fixes in 0.189.5.

= 0.189.5 =
* Fix invalid Gutenberg columns by matching Core width, vertical-alignment and stacking serialization; expose guarded same-ID repair through native compilation, with exact snapshot restoration.
* Compose installed, explicitly opted-in integration providers using closed native attributes and destination-owned code/configuration fingerprints.
* Extend an existing migration selection with its recorded missing-media dependencies, preserving prior selections and destination drafts instead of requiring every image up front.
* Export public native original images through authenticated, version-guarded MCP reads for the existing image upload workflow. Private/unpublished/password-protected media remain excluded; no arbitrary files or URL fetches.
* Keep rejected-authentication diagnostics read-only even when stored application-password records lack UUIDs. Authentication rules and credentials are unchanged.

= 0.189.4 =
* Fix migration export crashes on legacy scalar padding/margin, retaining qualified shorthand and independently portable content.
* Recover authored HTML and visible form copy as native content without executing source scripts or claiming form delivery.
* Preserve authenticated root-relative images and viewport-only visibility; unresolved links, media and runtime conditions stay explicit.
* Resolve Divi home_url tokens and the native primary-menu location, retaining useful header/footer and menu content independently of styling gaps.
* Allow canonical empty paragraphs in guarded review context; verify exact historical draft baselines before composition revision, while retaining real drift checks and excluding old undo support.

= 0.189.3 =
* Preserve native compiler classes through Gutenberg copy edits and reordering, and adopt existing theme-backed classes on first recompile.
* Mint and persist distinct classes for new styled blocks; report their missing theme rules separately from existing classes.

= 0.189.2 =
* Add a CLI theme iteration workflow that rebuilds, reviews file changes, creates a reproducible ZIP and installs to a local WordPress destination only after source and destination hash checks, with a backup and WordPress verification.
* Detect direct theme edits through a stylesheet fingerprint and bump the child theme patch version on the next build without replacing those edits.

= 0.189.1 =
* Compile responsive gaps through Core-safe theme spacing presets, with theme CSS fallback when no preset exists; report missing rule classes.
* Make generated aurora JavaScript independent of the installed theme folder name and refresh an untouched older generated script on rebuild.
* Verify TSF text fields after its own normalization, and explain incomplete media usage scans by stage and resource.
* Document fluid breakpoint defaults and the rebuild step after Site Editor template-part or global-style edits; bump the generated child theme patch version when a rebuild changes files.

= 0.189.0 =
* Compile typed Gutenberg presentation into Core attributes and a child theme with guarded snapshots, per-field reports and provenance export. New authoring writes native output; the front-end presentation filter is removed.
* Discover and reuse live theme presets and Site Editor styles; preserve direct edits to owned and unowned theme entries.
* Add guarded TSF, Yoast SEO, Rank Math and AIOSEO per-page SEO field review, write and restore abilities.

= 0.188.0 =
* Fix blocks with decimal presentation values losing styling after migrating between hosts with different PHP precision settings.
* Preserve existing legacy -1 and precision-17 design classes at render time; keep integer-only class hashes unchanged and reject invalid classes.
* Add page-local class diagnostics to read-builder-profile and guarded, idempotent rehash-review-presentations before normal review merge/restore.
* Route structured fingerprints and signatures through one sorted canonical encoder, including cross-site option values and migration journals. Mark new persisted schemes and retain exact legacy retry/signature compatibility; changed nested dependency guards require fresh review rather than automatic rebasing.
* Format numeric CSS consistently across hosts without changing site-wide PHP settings.

== Installation ==

1. Install the supplied Bridge release ZIP through Plugins > Add New > Upload Plugin. Developers packaging from source may run `npm run package:bridge`; ordinary customers do not need a source checkout.
2. Activate it.
3. Open WP Agent > Settings as an Administrator, select the client, and follow its instructions. Codex and Claude Code use a revocable site key; ChatGPT and Claude apps use OAuth. Never use a WordPress login password as an integration secret.
4. Select destination resources and capabilities in WordPress. Preview and review proposed work through the connected agent before applying or publishing.
5. Customer use requires no CLI, VM, container, local build toolchain or cloud execution account. Repository tooling is for developers, not the ordinary customer workflow.

For a macOS Local site, trust Local's certificate in Keychain. If Codex still reports `DEPTH_ZERO_SELF_SIGNED_CERT`, quit Codex, run `launchctl setenv NODE_OPTIONS --use-system-ca` once in Terminal, then reopen Codex. This keeps TLS verification enabled while Node reads the macOS trust store. Do not disable TLS verification.

Bridge 0.46.0 adds optional top-level scalar ACF email values to the existing guarded Gutenberg/Divi draft, combined-build, snapshot/restore, and SafeEdit overlay lifecycle. Email values remain bounded, closed, value-hash guarded, and redacted. Nested fields, ACF Blocks, and new builder surfaces remain unavailable.

Bridge 0.45.0 promotes the existing closed Blurb and CTA semantic components to portable typed blueprints for native Divi or canonical Gutenberg creation. It also aligns Gutenberg, Divi, and local CTA URL validation and rejects markup-shaped, shortcode-shaped, control-bearing, credential-bearing, protocol-relative, or unbounded Blurb/CTA fields before native compilation. No route, operation, Ability, module, presentation control, or existing-page edit surface is added.

Bridge 0.44.0 adds the read-only Tools > WP Agent Setup guide for manual Application Password provisioning. WordPress Core continues to own authentication, credential generation, usage tracking, and revocation; the local runtime owns Keychain reference storage and verification. The page adds no REST route, Ability, WP-CLI command, credential field, or stateful admin action.
