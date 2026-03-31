# Waulter Consent Management (Cookie and Other Consents Management) — Google Tag Manager Template

A Google Tag Manager (GTM) Community Template for **Waulter automatic GDPR AI assistent** that loads the [Waulter CMP](https://waulter.eu) consent banner and integrates with **Google Consent Mode v2**. The template handles both default consent state and consent updates entirely within GTM's sandboxed JavaScript environment — no additional custom HTML tags are needed for consent management.

## What This Template Does

- **Loads the Waulter CMP SDK** (`sdk.js`) onto the page via a single GTM tag.
- **Sets default consent state** — all Google consent signals start as `denied` on every page load (Consent Initialization trigger).
- **Listens for user consent decisions** — when the user interacts with the Waulter banner, the template receives the `Waulter:Decision` dataLayer event and calls `updateConsentState` with the appropriate granted/denied values.
- **Maps Waulter purposes to Google consent types** — each Waulter purpose (e.g., `PU046`, `PU061`) is mapped to the corresponding Google consent signal (`analytics_storage`, `ad_storage`, etc.).
- **Works with Material Icons** — the Waulter banner UI can use Material Icons for its interface elements.

## Requirements

- A **Waulter CMP account** — sign up at [waulter.eu](https://waulter.eu)
- A configured **Scenario ID** (e.g., `SC00045`) from the Waulter selfcare portal
- A **GTM Web container** (not Server container)
- GTM **Consent Overview** enabled in your container (Admin > Container Settings)

## Installation

### Step 1: Add the Template

**From the Community Template Gallery (recommended):**

1. In GTM, go to **Templates** > **Tag Templates** > **Search Gallery**
2. Search for **"Waulter consent management (cookie and other consents management)"**
3. Click **Add to workspace**

**Manual import:**

1. Download `template.tpl` from this repository
2. In GTM, go to **Templates** > **Tag Templates** > **New**
3. Click the three-dot menu (top right) > **Import**
4. Select the downloaded `template.tpl` file
5. Click **Save**

### Step 2: Create the Tag

1. Go to **Tags** > **New**
2. Choose **Waulter consent management (cookie and other consents management)** as the tag type
3. Fill in the required field:
   - **Waulter ID (CCID or Scenario ID)** — your Scenario ID (e.g., `SC00045`) or CCID
4. Optionally configure:
   - SDK URL (defaults to `https://cdn.waulter.cz/sdk.js`)
   - Advanced consent management settings (wait for update, debug)
   - Custom fields for scenario targeting
5. Click **Save**

### Step 3: Set the Trigger

1. In the tag configuration, click **Triggering**
2. Select **Consent Initialization - All Pages**
   - This built-in trigger fires before all other triggers, ensuring default consent is set before any tags attempt to fire
3. Click **Save**

### Step 4: Configure Consent-Dependent Tags

Your other tags (Google Analytics, Google Ads, Facebook Pixel, etc.) should use consent-based firing. Two approaches:

**Approach A — Built-in Consent Checks (recommended):**

In each tag's advanced settings, set **Consent Settings** to require the appropriate consent types (e.g., `analytics_storage` for GA4, `ad_storage` for Google Ads). GTM will automatically block/unblock them based on consent state.

**Approach B — Custom Triggers using dataLayer events:**

Import the [Waulter GTM scaffold](waulter_gtm_scaffold.json) — this companion file contains pre-built Waulter consent variables and triggers (79 variables + 17 triggers + 2 tags = 98 items, covering 50 purposes across 9 categories). Import it into your GTM container to get ready-made consent gating for non-Google tags.

| Trigger | Fires When |
|---------|------------|
| `Waulter - All Consent Granted` | User accepts all purposes |
| `Waulter - Analytics & Optimization Granted` | User grants analytics purposes |
| `Waulter - Marketing Granted` | User grants marketing/ad purposes |
| `Waulter - Advanced Tracking Granted` | User grants advanced tracking purposes |
| `Waulter - AB Testing Granted` | User grants A/B testing purpose |

### Step 5: Preview and Publish

1. Click **Preview** to open GTM Preview mode
2. Verify:
   - The Waulter banner appears on the page
   - Default consent state shows all signals as `denied`
   - After user consent, `updateConsentState` fires with correct values
   - Consent-dependent tags fire/block correctly
3. Once verified, click **Submit** to publish the container

## Template Field Reference

| Field | Parameter Name | Type | Required | Default | Description |
|-------|---------------|------|----------|---------|-------------|
| Waulter ID (CCID or Scenario ID) | `waulterId` | Text | Yes | — | Scenario ID (e.g., `SC00045`) or CCID |
| SDK URL | `sdkUrl` | Text | No | `https://cdn.waulter.cz/sdk.js` | SDK script URL. Change only for dev/staging environments |
| Wait for Update | `waitForUpdate` | Number | No | `500` | Milliseconds to wait for consent update before firing tags |
| Debug Mode | `debug` | Checkbox | No | Unchecked | Enable SDK debug logging to browser console |
| Custom Field 1–10 | `customField1`–`customField10` | Text | No | `{{Waulter - Custom Field N}}` | Custom context values for scenario rule evaluation. Defaults to scaffold DLV for auto-lookup (see below). |

> **Note:** Consent duration settings (`defaultAllowDuration`, `defaultMixedDuration`, `defaultRejectDuration`) are reserved for a future release. Consent durations are currently managed server-side in your Waulter configuration.

### Custom Fields Auto-Lookup

When the [companion scaffold](waulter_gtm_scaffold.json) is imported, custom fields work **automatically** without additional GTM configuration. Each template field defaults to a scaffold Data Layer Variable (`{{Waulter - Custom Field 1}}` through `{{Waulter - Custom Field 10}}`), which reads values set by the scaffold's Custom Fields Loader tag.

**How it works:** Your website sets a value in localStorage or a cookie using the standard key naming convention (`waulter_cf1` through `waulter_cf10`). The Loader tag reads the value, pushes it to the dataLayer, the DLV picks it up, and the template passes it to `WaulterConfig` — all automatically.

```javascript
// On your website — set a custom field value
localStorage.setItem('waulter_cf1', 'premium');
// That's it — the value flows automatically to scenario rules
```

**Key naming convention:**

| Field | localStorage / Cookie Key | Scaffold DLV |
|-------|--------------------------|--------------|
| Custom Field 1 | `waulter_cf1` | `{{Waulter - Custom Field 1}}` |
| Custom Field 2 | `waulter_cf2` | `{{Waulter - Custom Field 2}}` |
| ... | ... | ... |
| Custom Field 10 | `waulter_cf10` | `{{Waulter - Custom Field 10}}` |

**Priority:** If you replace a field's default DLV reference with a static value or different GTM variable, that override takes highest priority. Otherwise: localStorage > cookie.

## Google Consent Mode v2 Mapping

The template maps Waulter consent decisions to Google consent signals:

| Google Consent Signal | Granted When | Description |
|----------------------|--------------|-------------|
| `ad_storage` | Marketing purposes accepted (e.g., `PU047`, `PU072`) | Enables storage for advertising |
| `ad_user_data` | Marketing purposes accepted | Allows sending user data to Google for advertising |
| `ad_personalization` | Marketing purposes accepted | Enables ad personalisation |
| `analytics_storage` | Analytics purposes accepted (e.g., `PU046`, `PU061`) | Enables storage for analytics |
| `functionality_storage` | Full consent (`allow` decision) | Enables storage for site functionality |
| `personalization_storage` | Personalisation purposes accepted | Enables storage for personalisation |
| `security_storage` | Always `granted` | Essential security features (fraud prevention, authentication) |

### Default Consent State

On every page load (before user interaction), the template sets:

```
ad_storage:              denied
ad_user_data:            denied
ad_personalization:      denied
analytics_storage:       denied
functionality_storage:   denied
personalization_storage: denied
security_storage:        granted
```

The `wait_for_update` parameter (default 500ms) tells Google tags to wait briefly for a consent update before using the default values, improving accuracy for returning visitors whose stored consent loads asynchronously.

## Waulter Purpose IDs

These are the standard purpose codes used by Waulter configurations. Your specific CCID may use a subset.

| Purpose ID | Category | Description |
|-----------|----------|-------------|
| `PU046` | Analytics | Basic web analytics and traffic measurement |
| `PU047` | Marketing | Advertising and ad network tracking |
| `PU050` | Testing | A/B testing and experimentation |
| `PU061` | Analytics | Extended analytics and audience measurement |
| `PU072` | Marketing | Remarketing and audience building |
| `PU073` | Marketing | Conversion tracking |
| `PU074` | Marketing | Ad personalisation and user profiling |

The mapping between purposes and Google consent signals is configured server-side in each CCID's `googleCategory` field. The template reads these mappings automatically — you do not need to configure them manually.

## Updating the Template

**Gallery updates:** When a new version is published to the GTM Community Template Gallery, GTM will show an update notification in your workspace. Review the change notes and click **Update** to apply.

**Manual updates:** Download the latest `template.tpl` from this repository and re-import it via **Templates** > select the template > three-dot menu > **Import**.

## Localisation Note

The template and all Gallery-facing names use English. If your GTM container uses Czech or Slovak naming conventions, here is a mapping of common trigger and variable names:

| Scaffold Name (English) | Legacy Czech Equivalent |
|------------------------|------------------------|
| `Waulter - All Consent Granted` | `Consent_Decision_Allow` |
| `Waulter - [PC005] Analytics & Optimization Category Allowed` | `Waulter - web analysis allowed?` |
| `Waulter - [PC007] Marketing Category Allowed` | `Waulter - Ads network allowed?` |
| `Waulter - AB Testing (PU050)` | `Waulter - AB testing allowed?` |
| `Waulter - Decision` | `Waulter_Decision` |
| `Waulter - Purposes` | `Waulter_Purposes` |

## Troubleshooting

**Banner does not appear:**
- Verify the Scenario ID is correct and active in the Waulter selfcare portal
- Check that your domain is whitelisted in the B2BWebSiteConfig
- Open browser DevTools console and look for errors from `cdn.waulter.cz`
- Ensure the tag is firing on the **Consent Initialization - All Pages** trigger

**Consent state not updating after user decision:**
- In GTM Preview mode, check that the `Waulter:Decision` event appears in the event timeline
- Verify the `decision` and `purposes` values in the dataLayer
- Ensure no other tag or script is also calling `gtag('consent', 'default', ...)` after the template (this would overwrite the update)

**Tags still firing without consent:**
- Confirm that consent-dependent tags have the correct **Consent Settings** in their advanced configuration
- Check that the Waulter tag fires on **Consent Initialization**, not **All Pages** (the latter fires too late)

**Banner icons appear broken:**
- The banner UI uses Material Icons. Add the stylesheet to your site manually: `<link rel="stylesheet" href="https://fonts.googleapis.com/icon?family=Material+Icons">`
- If your site has a strict Content Security Policy, you may also need to allow `fonts.googleapis.com` and `fonts.gstatic.com`

**Debug mode:**
- Enable the **Debug Mode** checkbox in the tag configuration to see detailed SDK logs in the browser console
- Add `?waulter_debug=true` to your page URL as an alternative

## Support and Resources

| Resource | URL |
|----------|-----|
| Waulter Homepage | [waulter.eu](https://waulter.eu) |
| Documentation | [docs.waulter.eu](https://docs.waulter.eu) |
| Report Issues | [GitHub Issues](../../issues) |
| Google Consent Mode Docs | [developers.google.com/tag-platform/tag-manager/templates/consent-apis](https://developers.google.com/tag-platform/tag-manager/templates/consent-apis) |
| GTM Template Gallery | [tagmanager.google.com/gallery](https://tagmanager.google.com/gallery/) |

## License

This template is licensed under the [Apache License 2.0](LICENSE).

Copyright 2026 Four musketeers s.r.o.
