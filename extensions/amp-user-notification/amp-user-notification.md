<!--
Copyright 2015 The AMP HTML Authors. All Rights Reserved.

Licensed under the Apache License, Version 2.0 (the "License");
you may not use this file except in compliance with the License.
You may obtain a copy of the License at

      http://www.apache.org/licenses/LICENSE-2.0

Unless required by applicable law or agreed to in writing, software
distributed under the License is distributed on an "AS-IS" BASIS,
WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.
See the License for the specific language governing permissions and
limitations under the License.
-->

# <a name="amp-user-notification"></a> `amp-user-notification`

<table>
  <tr>
    <td width="40%"><strong>Description</strong></td>
    <td>Displays a dismissable notification to the user. </td>
  </tr>
  <tr>
    <td width="40%"><strong>Availability</strong></td>
    <td>Stable</td>
  </tr>
  <tr>
    <td width="40%"><strong>Required Script</strong></td>
    <td>
      <div>
        <code>&lt;script async custom-element="amp-user-notification" src="https://cdn.ampproject.org/v0/amp-user-notification-0.1.js">&lt;/script></code>
      </div>
      <div>
        <code>&lt;script async custom-element="amp-analytics" src="https://cdn.ampproject.org/v0/amp-analytics-0.1.js">&lt;/script></code>
      </div>
      <small>Notice that both "amp-user-notification" and "amp-analytics" scripts are required.</small>
    </td>
  </tr>
  <tr>
    <td class="col-fourty"><strong><a href="https://www.ampproject.org/docs/guides/responsive/control_layout.html">Supported Layouts</a></strong></td>
    <td>nodisplay</td>
  </tr>
  <tr>
    <td width="40%"><strong>Examples</strong></td>
    <td><a href="https://ampbyexample.com/components/amp-user-notification/">amp-user-notification_with_local_storage.html</a><br /><a href="https://ampbyexample.com/advanced/amp-user-notification_with_server_endpoint/">amp-user-notification_with_server_endpoint.html</a><br /><a href="https://github.com/ampproject/amphtml/blob/master/examples/user-notification.amp.html">user-notification.amp.html</a></td>
  </tr>
</table>

## Usage

An `id` is required
as multiple `amp-user-notification` elements are allowed and the
`id` is used to differentiate them.

By supplying two URLs that
get called before the notification is shown and after it is dismissed,
it is possible to control per user as to whether the notification should
be shown (using the `ampUserId` value).
For example, it could only be shown to users in certain geo locations or
prevent showing it again to the user when they've dismissed it before.
If these URLs are not specified, dismissal state will be queried
and/or stored locally to determine whether to show the notification to
the user.

To close `amp-user-notification`, add a `on` attribute to a button with the
following value scheme `on="event:idOfUserNotificationElement.dismiss"`
(see example below). This user action also triggers the `GET` to the
`data-dismiss-href` URL. Be very mindful of the browser caching the `GET` response
and see details below in the `data-show-if-href` section. (We recommend
adding a unique value to the `GET` url like a timestamp as a query string field)

When multiple `amp-user-notification` elements are on a page, only one is shown
at a single time (Once one is dismissed the next one is shown).
The order of the notifications being shown is currently not deterministic. (TODO:
follow up task #1229 to fix this).

Example:

```html
<amp-user-notification
    layout=nodisplay
    id="amp-user-notification1"
    data-show-if-href="https://foo.com/api/show-api?timestamp=TIMESTAMP"
    data-dismiss-href="https://foo.com/api/dismissed">
    This site uses cookies to personalize content.
    <a href="">Learn more.</a>
   <button on="tap:amp-user-notification1.dismiss">I accept</button>
</amp-user-notification>
```

---

## Attributes

**data-show-if-href** (Optional)

When specified, AMP will make a [CORS](https://developer.mozilla.org/en-US/docs/Web/HTTP/Access_control_CORS)
GET request with credentials to this URL to determine whether the notification should be shown.
We will append the `elementId` and `ampUserId` query string fields to the href provided
on the `data-show-if-href` attribute. (see [#1228](https://github.com/ampproject/amphtml/issues/1228) on why this is a GET instead of a POST)

For best practice to not let the browser cache the GET response values you should add
a [`TIMESTAMP` url replacement](https://github.com/ampproject/amphtml/blob/master/spec/amp-var-substitutions.md) value to the `data-show-if-href` attribute value.
You can add it as a query string field. (ex.
`data-show-if-href="https://foo.com/api/show-api?timestamp=TIMESTAMP"`)

 - `CORS GET request` query string fields
    - `elementId`
    - `ampUserId`

  Example:
  ```none
  https://foo.com/api/show-api?timestamp=1234567890&elementId=notification1&ampUserId=cid-value
  ```

 - `CORS GET response` json fields
    The response must contain a single JSON object with a field
    "showNotification" of type boolean. If this field is `true` the
    notification will be shown, otherwise it will not be shown.

    - `showNotification`

    Example:
    ```json
    { "showNotification": true }
    ```

If not specified, AMP will only check if the notification with the specified ID has been "dismissed"
by the user locally. If not, the notification will be shown.

**data-dismiss-href** (Optional)

When specified, AMP will make a [CORS](https://developer.mozilla.org/en-US/docs/Web/HTTP/Access_control_CORS)
POST request to this URL transmitting the `elementId` and
`ampUserId` only when the user has explicitly agreed.

Use the `ampUserId` field to store that the user has seen the notification before
if you want to avoid showing it in the future. (It is the same value that
will be passed in future requests to data-show-if-href)

  - `POST request` json fields

    - `elementId`
    - `ampUserId`

    Example:
    ```json
    { "elementId": "id-of-amp-user-notification", "ampUserId": "ampUserIdString" }
    ```
  - `POST response` should be a 200 HTTP code and no data is expected back.

If not specified, AMP will not send a request upon dismissal and will only store "dismissed"
flag for the specified ID locally.

**data-persist-dismissal** (Optional)
__Default: true__
If set  to `false` AMP will not remember the user's dismissal of the notification. The notification
will always show if the `data-show-if-href` result is show notification. If no `data-show-if-href` is provided
the notification will always show.

Example 1:
```html
<amp-user-notification
      layout=nodisplay
      id="amp-user-notification5"
      data-persist-dismissal="false"
      data-show-if-href="https://example.com/api/shouldShow?timestamp=TIMESTAMP"
      data-dismiss-href="https://example.com/api/echo/post">
This notification should ALWAYS show - if shouldShow endpoint response was true.
<a href="#learn-more">Learn more.</a>
<button on="tap:amp-user-notification5.dismiss">Dismiss</button>
</amp-user-notification>
```

Example 2:
```html
<amp-user-notification
      layout=nodisplay
      id="amp-user-notification6"
      data-persist-dismissal="false">
This notification should ALWAYS show on every page visit.
<a href="#learn-more">Learn more.</a>
<button on="tap:amp-user-notification6.dismiss">Dismiss</button>
</amp-user-notification>
```


---

## JSON Fields

- `elementId` (string) - The HTML id used on `amp-user-notification` element.
- `ampUserId` (string) - This id is passed to both the `data-show-if-href` GET request
    (as a query string field) and the `data-dismiss-href` POST request (as a json field).
    The id will be the same for this user going forward, but no other requests
    in AMP send the same id.
    You can use the id on your side to lookup/store whether the user has
    dismissed the notification before.
- `showNotification` (boolean) - Boolean value indicating whether or not the notification should be shown.
    If `false` the promise associated to the element is resolved right away.

---

## Behavior

A notification is shown when:

1. There's no record locally that the user has dismissed the notification with the
specified ID.
2. When specified, `data-show-if-href` endpoint returns `{ "showNotification": true }`.

When notification is dismissed:

1. AMP stores the "dismiss" record locally for the specified ID. This will prevent the
notification from being shown again.
2. When specified, `data-dismiss-href` is invoked and can be used to make the "dismiss"
record remotely.

---

## Styling

The `amp-user-notification` component should always have `layout=nodisplay`
and will be `position: fixed` after layout (default is bottom: 0, which can be overridden).
If a page has more than 1 `amp-user-notification` element then the notifications
are queued up and only shown when the previous notification has been dismissed.

The `amp-active` (visibility: visible) class is added when the notification is displayed and
and removed when the notification has been dismissed.
`amp-hidden` (visibility: hidden) is added when the notification has been dismissed.

You can for example hook into these classes for a "fade in" transition.

ex. (w/o vendor prefixes)

```css
  @keyframes fadeIn {
    from { opacity: 0; }
    to { opacity: 1; }
  }

  amp-user-notification.amp-active {
    opacity: 0;
    animation: fadeIn ease-in 1s 1 forwards;
  }
```

## Delaying Client ID generation until the notification is acknowledged

Optionally one can delay generation of Client IDs used for analytics and similar purposes until an `amp-user-notification` is confirmed by the user. See these docs for how to implement this:

- [CLIENT_ID URL substitution.](../../spec/amp-var-substitutions.md#CLIENT_ID)
- [`amp-ad`](../../builtins/amp-ad.md)
- [`amp-analytics`](../amp-analytics/amp-analytics.md)

## Validation

See [amp-user-notification rules](https://github.com/ampproject/amphtml/blob/master/extensions/amp-user-notification/0.1/validator-amp-user-notification.protoascii) in the AMP validator specification.
