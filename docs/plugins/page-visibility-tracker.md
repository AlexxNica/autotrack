# `pageVisibilityTracker`

This guide explains what the `pageVisibilityTracker` plugin is and how to integrate it into your `analytics.js` tracking implementation.

## Overview

It's becoming increasingly common for users to visit your site, and then leave it open in a browser tab for hours or days. And with rise in popularity of single page applications, some tabs almost never get closed.

Because of this shift, the traditional model of pageviews and sessions simply does not apply in a growing number of cases.

The `pageVisibilityTracker` plugin changes this paradigm by shifting from pageload being the primary indicator to [Page Visibility](https://developer.mozilla.org/en-US/docs/Web/API/Page_Visibility_API). To understand why this is important, consider this scenario: you've just updated your website to be able to fetch new content in the background and display it to the user when they return to your page (without forcing them to reload). If you were only using the default analytics.js tracking snippet, this change would result is dramatically fewer pageviews, even though the user is consuming the same amount of content.

By taking Page Visibility into consideration, the `pageVisibilityTracker` plugin is able to report consistent numbers regardless of whether the user needs to reload the page.

The `pageVisibilityTracker` plugin also calculates how long a given page was in the visible state for a given session, which is a much better indicator of user engagement than [*Session Duration*](https://support.google.com/analytics/answer/1006253).

The following sample reports show how you can use the `pageVisibilityTracker` plugin to more accurately measure user engagement with your content.

**Top pages by visible time:**

![page-visibility-page](https://cloud.githubusercontent.com/assets/326742/22574482/e635b26a-e963-11e6-95d4-25b7face7621.png)

**Traffic origins (source/medium) resulting in the longest visible sessions:**

![page-visibility-source-medium](https://cloud.githubusercontent.com/assets/326742/22574483/e636607a-e963-11e6-928a-4c49948bf8d8.png)

### How it works

The `pageVisibilityTracker` plugin listens for [`visibilitychange`](https://developer.mozilla.org/en-US/docs/Web/Events/visibilitychange) events on the current page and sends hits to Google Analytics capturing how long the page was in each state. It also programmatically starts new sessions and sends new pageviews when the visibility state changes from hidden to visible (if the previous session has timed out).

### Impact on session and pageview counts

When using the `pageVisibilityTracker` plugin, you'll probably notice an increase in your session and pageview counts. This is not an error, the reality is your current implementation (based just on pageloads) is likely underreporting these metrics.

## Usage

To enable the `pageVisibilityTracker` plugin, run the [`require`](https://developers.google.com/analytics/devguides/collection/analyticsjs/using-plugins) command, specify the plugin name `'pageVisibilityTracker'`, and pass in any configuration options (if any) you wish to set:

```js
ga('require', 'pageVisibilityTracker', options);
```

### Using a custom metric

The easiest way to track the time a page was visible is to create a [custom metric](https://support.google.com/analytics/answer/2709828) called *Page Visible Time* that you set in your plugin configuration options, and then to create [calculated metrics](https://support.google.com/analytics/answer/6121409) called *Avg. Page Visible Time (per Page)* and *Avg. Page Visible Time (per Session)* that you use in your reports.

Which calculated metric you need will depend on which dimensions you're using in your report. For session-level dimensions (e.g. *Referrer* or *Device Category*) you'll want to use the session version, and for page-specific dimensions (e.g. *Page* or *Title*) you'll want to use the page version.

Here are the formulas for both:

```
{{Page Visible Time}} / {{Sessions}}
```

```
{{Page Visible Time}} / {{Unique Pageviews}}
```

The screenshot in the [overview](#overview) shows some examples of what reports with these custom and calculated metrics look like.

## Options

The following table outlines all possible configuration options for the `pageVisibilityTracker` plugin. If any of the options has a default value, the default is explicitly stated:

<table>
  <tr valign="top">
    <th align="left">Name</th>
    <th align="left">Type</th>
    <th align="left">Description</th>
  </tr>
  <tr valign="top">
    <td><code>sessionTimeout</code></td>
    <td><code>number</code></td>
    <td>
      The <a href="https://support.google.com/analytics/answer/2795871">session timeout</a> amount (in minutes) of the Google Analytics property. By default this value is 30 minutes, which is the same default used for new Google Analytics properties. The value set for this plugin should always be the same as the property setting in Google Analytics.<br>
      <strong>Default:</strong> <code>30</code>
    </td>
  </tr>
  <tr valign="top">
    <td><code>timeZone</code></td>
    <td><code>string</code></td>
    <td>
      A time zone to pass to the <a href="https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/DateTimeFormat"><code>Int.DateTimeFormat</code></a> instance. Since sessions in Google Analytics are limited to a single date in the time zone of the view, this setting can be used to more accurately predict session boundaries. (Note: if your property contains views in several different time zones, do not use this setting).
    </td>
  </tr>
  <tr valign="top">
    <td><code>visibleThreshold</code></td>
    <td><code>number</code></td>
    <td>
      The time in milliseconds to wait before sending a Page Visibility event (or a new pageview in the case of a session timeout). This helps prevent unwanted events from being sent in cases where a user is quickly switching tabs or closing tabs they no longer want open.<br>
      <strong>Default:</strong> <code>5000</code>
    </td>
  </tr>
  <tr valign="top">
    <td><code>visibleMetricIndex</code></td>
    <td><code>number</code></td>
    <td>If set, a <a href="https://support.google.com/analytics/answer/2709828">custom metric</a> at the index provided is sent when the page's visibility state changes from visible to hidden. The metric value is the amount of time (in seconds) the page was in the visible state.</td>
  </tr>
  <tr valign="top">
    <td><code>fieldsObj</code></td>
    <td><code>Object</code></td>
    <td>See the <a href="/docs/common-options.md#fieldsobj">common options guide</a> for the <code>fieldsObj</code> description.</td>
  </tr>
  <tr valign="top">
    <td><code>hitFilter</code></td>
    <td><code>Function</code></td>
    <td>See the <a href="/docs/common-options.md#hitfilter">common options guide</a> for the <code>hitFilter</code> description.</td>
  </tr>
</table>

## Default field values

### Page Visibility events

The `pageVisibilityTracker` plugin sets the following default field values on event hits it sends. To customize these values, use one of the [options](#options) described above.

<table>
  <tr valign="top">
    <th align="left">Field</th>
    <th align="left">Value</th>
  </tr>
  <tr valign="top">
    <td><a href="https://developers.google.com/analytics/devguides/collection/analyticsjs/field-reference#hitType"><code>hitType</code></a></td>
    <td><code>'event'</code></td>
  </tr>
  <tr valign="top">
    <td><a href="https://developers.google.com/analytics/devguides/collection/analyticsjs/field-reference#eventCategory"><code>eventCategory</code></a></td>
    <td><code>'Page Visibility'</code></td>
  </tr>
  <tr valign="top">
    <td><a href="https://developers.google.com/analytics/devguides/collection/analyticsjs/field-reference#eventAction"><code>eventAction</code></a></td>
    <td><code>'track'</code></td>
  </tr>
  <tr valign="top">
    <td><a href="https://developers.google.com/analytics/devguides/collection/analyticsjs/field-reference#eventValue"><code>eventValue</code></a></td>
    <td>The elapsed time (in seconds) spent in the visible state.</td>
  </tr>
  <tr valign="top">
    <td><a href="https://developers.google.com/analytics/devguides/collection/analyticsjs/field-reference#nonInteraction"><code>nonInteraction</code></a></td>
    <td><code>true</code></td>
  </tr>
</table>

### New pageviews

If the page's visibility state changes from `hidden` to `visible` and the session has timed out, a new pageview is sent.

<table>
  <tr valign="top">
    <th align="left">Field</th>
    <th align="left">Value</th>
  </tr>
  <tr valign="top">
    <td><a href="https://developers.google.com/analytics/devguides/collection/analyticsjs/field-reference#hitType"><code>hitType</code></a></td>
    <td><code>'pageview'</code></td>
  </tr>
</table>

## Methods

The following table lists all methods for the `pageVisibilityTracker` plugin:

<table>
  <tr valign="top">
    <th align="left">Name</th>
    <th align="left">Description</th>
  </tr>
  <tr valign="top">
    <td><code>remove</code></td>
    <td>Removes the <code>pageVisibilityTracker</code> plugin from the specified tracker, removes all event listeners from the DOM, and restores all modified tasks to their original state prior to the plugin being required.</td>
  </tr>
</table>

For details on how `analytics.js` plugin methods work and how to invoke them, see [calling plugin methods](https://developers.google.com/analytics/devguides/collection/analyticsjs/using-plugins#calling_plugin_methods) in the `analytics.js` documentation.

## Examples

### Setting a session timeout and time zone

If you've set the default session timeout in your Google Analytics property to 4 hours and the timezone of all your views to Pacific Time, you can ensure the `pageVisibilityTracker` plugin knows about these settings with the following configuration options:

```js
ga('require', 'pageVisibilityTracker', {
  sessionTimeout: 4 * 60,
  timeZone: 'America/Los_Angeles',
});
```

### Setting a custom metrics to track visible time

If you want to track the total (or average) time a user spends in the visible state via a [custom metric](https://support.google.com/analytics/answer/2709828) (by default it's tracked as the `eventValue`), you can use the visible metric indexes.

```js
ga('require', 'pageVisibilityTracker', {
  visibleMetricIndex: 1,
});
```

**Note:** this requires [creating custom metrics](https://support.google.com/analytics/answer/2709829) in your Google Analytics property settings.
