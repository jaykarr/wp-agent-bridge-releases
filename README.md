# WP Agent Bridge releases

Installable releases of WP Agent Bridge, the WordPress plugin for authorized AI site work through a normal MCP connection.

## Install or bootstrap updates

Download the versioned `wp-agent-bridge-X.Y.Z.zip` asset from the [latest release](https://github.com/jaykarr/wp-agent-bridge-releases/releases/latest). In WordPress, use Plugins → Add Plugin → Upload Plugin and replace the installed Bridge if prompted.

Starting with 0.193.36, the plugin checks this repository for updates. Enable automatic updates for WP Agent Bridge on the Plugins page to install future approved versions automatically. Manual update controls remain available. No GitHub account or update-feed setup is required on the WordPress site.

Use the versioned plugin ZIP asset. GitHub's automatically generated Source code archives contain this release repository's documentation and are not installable plugins.

Each release includes a SHA-256 checksum for its plugin ZIP. Releases retain the same bytes under a version; corrections use a new version.

Existing custom signed update feeds remain selected until the site administrator explicitly switches to GitHub Releases in WP Agent settings.

Development is maintained separately. This repository contains approved distribution artifacts, versioned update metadata and documentation.

WP Agent Bridge is licensed under GPL-2.0-or-later. Bundled third-party components retain their included licenses.
