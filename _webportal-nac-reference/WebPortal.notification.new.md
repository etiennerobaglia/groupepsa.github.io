---
layout: api-reference
section: webportal
subsection: v1
categorie: API Reference
title: References
name: WebPortal.notification.new
domain: WebPortal
supported: #todo
type: api
---


### `WebPortal.notification.new`


| **Description** | This API trigger a WebPortal Notification. Check out the [dedicated tutorial]({{site.baseurl}}/webportal/v1/interactivity/notification/#article).
| **Type**| `postMessage` command. See [implementation tutorial]({{site.baseurl}}/webportal/v1/application/events/#article) for post message commands. |

| **Parameter** | **Type** | **Description** | **Required**|
|-----|-----|-----|-----|
|**id**|*String*|Id of the notification|True|
|**body**|*String*|Text of the notification|True|
|**appId**|*String*|Id of the application. Not generated by app|True|
|**title**|*String*|Title of the notification (default is app name)|False|
|**icon**|*String*|App icon in notification center|False|
|**data**|*String*|Extra custom data, these data will be received in the [WebPortal.notification.evt.click]({{site.baseurl}}/webportal/v1/api-reference/webportal-notification-evt-click#article)|False|

>**Caution:** For accessibility reasons, popup should not be created more often than once every 20 seconds.


```js
window.parent.postMessage({
  type: "WebPortal.notification.new",
  notification: {
    id: "<generated_id>",
    title: "Notification Name",
    body: "This is a notification with information about something. \n Thank you for your attention.",
    appId: "my.app.name",
    icon: "<app_icon>",
    data : "<extra_custom_data>"
  }
}, '*');
```

### See Also

- Method **Create a NC Notification**: [WebPortal.notification.new]({{site.baseurl}}/webportal/v1/api-reference/webportal-notification-new.html#article)
- Event **Creation Success**: [WebPortal.notification.evt.created]({{site.baseurl}}/webportal/v1/api-reference/webportal-notification-evt-created.html#article)
- Event **Creation Error**: [WebPortal.notification.evt.error]({{site.baseurl}}/webportal/v1/api-reference/webportal-notification-evt-error.html#article)
- Event **Prompt Overflow**: [WebPortal.OverflowPopup]({{site.baseurl}}/webportal/v1/api-reference/webportal-overflowpopup#article)
- Event **NC Notification Opened**: [WebPortal.notification.show]({{site.baseurl}}/webportal/v1/api-reference/webportal-notification-show.html#article)
- Event **NC Notifcation: Open App button** clicked: [WebPortal.notification.evt.click]({{site.baseurl}}/webportal/v1/api-reference/webportal-notification-evt-click#article)
- Event **NC Notification: Delete button** clicked: [WebPortal.notification.event.close]({{site.baseurl}}/webportal/v1/api-reference/webportal-notification-evt-close.html#article)