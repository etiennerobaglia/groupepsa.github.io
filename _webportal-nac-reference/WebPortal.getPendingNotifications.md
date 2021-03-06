---
title: WebPortal.getPendingNotifications()
supported:
  - 2
  - 3
  - 4
type: api
---

### `WebPortal.getPendingNotifications()`

| **Description** | Gets the notifications currently pending. Look at [Tutorial(Advanced)]({{site.baseurl}}/webportal/v1/connectivity/message/#receiving-message) for an integration example.
| **Response** | The Notifications as `Array` (either empty or containing Strings) or `Unrecognised Error` if the application is not fully launched.
| **Parameter**   | *String* The application ID (AppID).

#### EXAMPLE

```javascript
var notificationMessages = WebPortal.getPendingNotifications()
// notificationMessages = []
```
