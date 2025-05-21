# Matomo Tracker React Quick Start Guide

Based on the comprehensive Matomo Tracker (React) Developer Guide, this quick start README walks you through essential Matomo methods—showing both simple and advanced usage, plus argument details.

---

**Prerequisites:** You need access to a Matomo server (self-hosted or Matomo Cloud) and a Site ID for your application. Ensure you have the Matomo server’s URL and the numeric site ID before proceeding.

## Installation and Setup

**Install the package:** Add the Matomo Tracker React package to your project via npm or yarn:


```bash
npm install @datapunt/matomo-tracker-react
```




### 1. Creating a Matomo Tracker Instance `createInstance()`

Before tracking anything, create a Matomo tracker instance using `createInstance` and wrap your app with the `MatomoProvider` context. Typically, you’ll do this in your application’s entry point (e.g. in **index.js** for CRA or **\_app.jsx** for Next.js):

```jsx
import { MatomoProvider, createInstance } from '@datapunt/matomo-tracker-react';

const instance = createInstance({
  urlBase: 'https://MATOMO.example.com/',   // Your Matomo server URL (base URL)
  siteId: 3,                                // Your Matomo site ID (required)
  userId: 'USER_12345',                     // Optional: unique user ID to identify this user:contentReference[oaicite:0]{index=0}
  trackerUrl: 'https://MATOMO.example.com/matomo.php', // Optional: endpoint for tracking requests (defaults to `${urlBase}matomo.php`)
  srcUrl: 'https://MATOMO.example.com/matomo.js',      // Optional: URL of Matomo JS (defaults to `${urlBase}matomo.js`)
  disabled: false,                         // Optional: if true, completely disables tracking (default false):contentReference[oaicite:1]{index=1}
  heartBeat: { active: true, seconds: 10 },// Optional: enable heartbeat ping every 10s to track visit duration (default active, 15s):contentReference[oaicite:2]{index=2}:contentReference[oaicite:3]{index=3}
  linkTracking: false,                     // Optional: enable automatic link click tracking (default true; we set false for SPA—explained later):contentReference[oaicite:4]{index=4}
  configurations: {                        // Optional: additional Matomo configuration settings:contentReference[oaicite:5]{index=5}
    disableCookies: true,    // e.g. do not use cookies for tracking (visitor will not be tracked via cookies):contentReference[oaicite:6]{index=6}
    setSecureCookie: true,   // e.g. if using HTTPS, set Matomo cookies as secure
    setRequestMethod: 'POST' // e.g. send tracking data via POST requests instead of GET:contentReference[oaicite:7]{index=7}
    // ...any other Matomo tracker config function and value
  }
});
 
ReactDOM.render(
  <MatomoProvider value={instance}>
    <App /> {/* Your root component */}
  </MatomoProvider>,
  document.getElementById('root')
);
```

**Configuration options explained:**

* **urlBase (string)** – **Required.** The base URL of your Matomo server (protocol and domain, ending with a slash). The tracker will load `matomo.js` from here and send data to `matomo.php` by default. For example, if your tracking code provided by Matomo shows `var u="https://yourdomain.matomo.cloud/";` then use that as `urlBase`.
* **siteId (number)** – **Required.** The ID of the website/app in Matomo for which data will be recorded. This must match the site you configured in Matomo.
* **userId (string)** – *Optional.* If you have a logged-in user or a unique identifier for a user, you can set it here to tie analytics to that user. It corresponds to Matomo’s *User ID* feature. If provided at initialization, the tracker will call Matomo’s `setUserId` internally. You can also set or change the user ID later (explained in the **User Identification** section).
* **trackerUrl (string)** – *Optional.* The full URL to the Matomo HTTP tracking endpoint. Defaults to `urlBase + "matomo.php"`. Some Matomo setups use a custom path (like `piwik.php` or `tracking.php`); if so, specify it here.
* **srcUrl (string)** – *Optional.* The URL to the Matomo JavaScript file. Defaults to `urlBase + "matomo.js"`. If your Matomo JS is hosted on a different CDN or path, you can override this.
* **disabled (boolean)** – *Optional.* Default false. If set to true, **all tracking calls become no-ops** – essentially pausing or disabling analytics. Use this if you want to programmatically disable tracking (for example, if a user opts out of analytics).
* **heartBeat (object)** – *Optional.* Configures Matomo’s heartbeat (ping) to more accurately measure time on page. By default, heartbeat is **enabled** with a 15-second interval. You can set `heartBeat.active` to false to disable it, or adjust `heartBeat.seconds` to change the interval. When active, Matomo will send periodic “heartbeat” pings as long as the page is open and in focus, which helps Matomo record how long a user actually stays on the page. This improves accuracy of “Time on page” and “Visit duration” metrics.
* **linkTracking (boolean)** – *Optional.* Default true. If true, the Matomo script will automatically track clicks on **outbound links and downloadable files** as events (Matomo calls this “enableLinkTracking”) once the script loads. In single-page applications (SPAs), however, Matomo’s default link tracking can be flaky, so you might disable it (set to false) and use the React package’s own method `enableLinkTracking()` for better control (discussed in **Link Tracking** below).
* **configurations (object)** – *Optional.* An object of additional low-level Matomo tracker configurations to be set. This is a flexible way to call any Matomo tracker API setup functions. Each key should correspond to a Matomo tracker function name, and the value is the argument to pass (or `true/false` for toggles). In the example above, we include:

  * `disableCookies: true` – Instructs Matomo **not to use any cookies** for this tracker. This is useful for privacy compliance (Matomo will still track visits, but each visit will be treated as new since no persistent cookie ID).
  * `setSecureCookie: true` – Instructs Matomo to set the tracking cookie with the Secure flag (only if cookies are used and the site is HTTPS).
  * `setRequestMethod: 'POST'` – Instructs Matomo to send tracking requests via HTTP POST instead of GET. Using POST can overcome URL length limitations and may improve privacy (query params aren’t exposed).
  * *Other Matomo config options:* You can include any other supported Matomo initialization call here. For example, `setDocumentTitle`, `setDomains` (to set cross-domain tracking domains), `enableCrossDomainLinking`, `requireConsent`, `requireCookieConsent`, etc., as needed. (See **Consent Management** for using consent-related configurations.)

When you call `createInstance(...)`, it internally creates a new `MatomoTracker` from the core JS library with these options. This will inject the Matomo script (`matomo.js`) into the page and apply all configurations. Ensure this code runs **in the browser** (client-side) – avoid calling it during server-side rendering. In a Next.js app, you might guard this with a `typeof window !== 'undefined'` check or initialize inside a `useEffect` to prevent SSR issues (since `document` is not available on the server).



---

## 2. `trackPageView(options)`
Page views are a fundamental metric – Matomo records a page view whenever a user loads or navigates to a new page in your app. With @datapunt/matomo-tracker-react, you manually trigger page view tracking using the trackPageView function from the hook.


**Simple Usage**

```js
const { trackPageView } = useMatomo();
trackPageView();
```

**Advanced Usage**

```js
trackPageView({
  documentTitle: 'Dashboard',
  href: '/dashboard',
  customDimensions: [
    { id: 1, value: 'Member' }
  ]
});
```

**Arguments**

* `documentTitle` **(string, optional)** — Override page title. Default: `document.title`.
* `href` **(string, optional)** — Override URL. Default: `window.location.href`.
* `customDimensions` **(array, optional)** — List of `{ id: number, value: string }`.

---

## 3. `trackEvent({ category, action, name, value, customDimensions })`

Events in Matomo are user interactions or occurrences on the page that you want to track, beyond simple page loads. Common examples include tracking clicks on particular buttons, video plays, form submissions, or any action users take. Matomo events have a Category, Action, an optional Name (label), and an optional numeric Value.


**Simple Usage**

```js
trackEvent({ category: 'UI', action: 'Click' });
```

**Advanced Usage**
```js
import React from 'react'
import { useMatomo } from '@datapunt/matomo-tracker-react'

const MyPage = () => {
  const { trackEvent } = useMatomo()

  const handleOnClick = () => {
    // Track click on button
    trackEvent({
      category: 'sample-page',
      action: 'click-event',
      name: 'test', // optional
      value: 123, // optional, numerical value
      documentTitle: 'Page title', // optional
      href: 'https://LINK.TO.PAGE', // optional
      customDimensions: [
        {
          id: 1,
          value: 'loggedIn',
        },
      ], // optional
    })
  }

  return (
    <button type="button" onClick={handleOnClick}>
      Click me
    </button>
  )
}


```

```js
trackEvent({
  category: 'Video',
  action: 'Play',
  name: 'Intro.mp4',
  value: 120,
  customDimensions: [ { id: 3, value: 'Admin' } ]
});
```

**Arguments**

* `category` **(string, required)** — Top-level grouping.
* `action` **(string, required)** — Action taken.
* `name` **(string, optional)** — Label for the event.
* `value` **(number, optional)** — Numerical value.
* `customDimensions` **(array, optional)** — `{ id: number, value: string }` list.

---

## 4. `trackEvents(options)`
Refer it in offical doc on index: Automatically Binding Event Listeners (Alternate Method)


**Simple Usage**

```js
const { trackEvents } = useMatomo();
trackEvents();
```

**Advanced Usage**

```js
trackEvents({
  delay: 200,
  attributesPrefix: 'data-matomo-'
});
```

**Arguments**

* `delay` **(number, optional)** — ms before auto-binding. Default: library default.
* `attributesPrefix` **(string, optional)** — Data-attribute prefix. Default: `'data-matomo-'`.

---

## 5. `trackSiteSearch({ keyword, category, count })`
If your application has a search feature (e.g., users can search within your site/app), Matomo can track what users search for, how many results were returned, etc. Site Search analytics can reveal popular search terms and content gaps.


**Simple Usage**

```js
trackSiteSearch({ keyword: 'shoes' });
```

**Advanced Usage**

```js
trackSiteSearch({
  keyword: 'shoes',
  category: 'Footwear',
  count: 42
});
```

**Arguments**

* `keyword` **(string, required)** — Search term.
* `category` **(string, optional)** — Search category.
* `count` **(number, optional)** — Number of results.

---

## 6. `enableLinkTracking(options)`

**Simple Usage**

```js
const { enableLinkTracking } = useMatomo();
enableLinkTracking();
```

**Advanced Usage**

```js
enableLinkTracking({
  linkClasses: ['external', 'download']
});
```

**Arguments**

* `linkClasses` **(string\[], optional)** — Only track links with these CSS classes.

---

## 7. `trackLink({ href, linkType, customDimensions })`

**Simple Usage**

```js
trackLink({ href: '/file.pdf', linkType: 'download' });
```

**Advanced Usage**

```js
trackLink({
  href: '/file.pdf',
  linkType: 'download',
  customDimensions: [ { id: 2, value: 'ManualDownload' } ]
});
```

**Arguments**

* `href` **(string, required)** — URL to track.
* `linkType` **(string, required)** — `'download'`, `'link'`, or `'exit'`.
* `customDimensions` **(array, optional)** — `{ id: number, value: string }` list.

---

## 8. `pushInstruction(instructionName, ...args)`
Some advanced functions (like content tracking, media analytics, form analytics – which are plugins in Matomo) would require either including those plugin scripts or using pushInstruction for specific calls. For example, Matomo Content Tracking can track impressions of elements with certain attributes – to use it, you might manually include Matomo’s content tracking JS and then call those methods via pushInstruction. Out of the box, the core library covers the main tracking needs.

In short, use for any Matomo JS tracker API call.
Note: Refer all available instructions in [Matomo JS tracker API](https://developer.matomo.org/api-reference/tracking-javascript)

**Simple Usage**

```js
pushInstruction('trackGoal', 5);
```

**Advanced Usage**

```js
pushInstruction('setCustomUrl', '/new/page');
pushInstruction('setDocumentTitle', 'New Title');
```

**Arguments**

* `instructionName` **(string, required)** — Matomo API method (e.g., `'trackGoal'`).
* `...args` **(any\[])** — Parameters for that API call.

---

## 9. User Identification & Custom Dimensions

### `setUserId(userId)` / `resetUserId()`

*(via `pushInstruction`)*

**Usage**

```js
pushInstruction('setUserId', 'USER_12345');
pushInstruction('resetUserId');
```

**Arguments**

* `userId` **(string, required for setUserId)** — Unique user identifier.

### `setCustomDimension(index, value)`

*(via `pushInstruction`)*

**Usage**

```js
pushInstruction('setCustomDimension', 1, 'Member');
```

**Arguments**

* `index` **(number, required)** — Dimension slot in Matomo.
* `value` **(string, required)** — Dimension value.

---




















# Official Readme
# Matomo Tracker

> **Warning**
> This repository is no longer maintained. I no longer use Matomo, nor do I have enough time between projects to take care of these libraries.

Monorepo for using Matomo tracking in frontend projects

## Content

This monorepo hosts two Matomo projects for tracking (user) analytics in frontend projects, either with (vanilla) JavaScript or React.

- [@jonkoops/matomo-tracker](https://github.com/jonkoops/matomo-tracker/tree/main/packages/js)
- [@jonkoops/matomo-tracker-react](https://github.com/jonkoops/matomo-tracker/tree/main/packages/react)

## References

- [Matomo JavaScript Tracking Guide](https://developer.matomo.org/guides/tracking-javascript-guide)
