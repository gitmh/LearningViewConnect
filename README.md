
# LearningViewConnect
This repository describes the API between LearningView and embedded interactive content providers.

## Description

LearningView (LV) is a Learning Management System that allows the connection to other services providing interactive content via **iframes**. LearningView attaches parameters to the iframe-URL based on the currently logged in user. The interactive content can send data back to LV by using iframe messaging via *postMessage*. The interactive content can also use the attached user ID to load a previous state of the user. The system is client-side and intended to be simple for providers and LearningView and does not require the configuration of keys and validation mechanism. Therefor it should not be used for critical data as it can easily manipulated for cheating (e.g. by executing specific JavaScript in Browser console). 

In addition to the handling of interactive content providers can also provide an API endpoint for a catalog of content. The catalog will be accessible for teachers within LearningView. In order to include the providers catalog, adjustments in LearningView are required and need to be addressed by the LearningView-Team. A custom service logo and API endpoints are required. 

## Creation of iframe 

A teacher will select or paste the URL of a given interactive content into the task creation form in LearningView. This URL requires to be accessible within an **iframe**. As a content provider make sure to adjust Content-Security-Policy (CSP) if in place, to allow embedding inside iframes.

-   Remove `X-Frame-Options` restrictions
-   Remove `Content-Security-Policy: frame-ancestors` restrictions 
-   Test the iframe on different browsers (Chrome enforces CSP more strictly)

The **iframe** will be created within LearningView with the provided URL like this:
```html
<iframe src="https://...?user=RbzZ5qxbNxmYkX3X"></iframe>
```
An additional URL parameter will be attached with `user=RbzZ5qxbNxmYkX3X` with a unique user ID from LearningView, based on the currently logged in user. This ID can be used to identify the user and reload a last state or similar in the interactive content.  The content provider can access this ID for example via JavaScript or on with server side. Example for JavaScript on client:
```javascript
const urlParams =  new  URLSearchParams(window.location.search);
const userid = urlParams.get('user');
```

A teacher in LearningView can access the students solution by accessing the URL with a slightly different parameter `student=RbzZ5qxbNxmYkX3X`. The interactive content can display a unchangeable version of the student solution or even provide feedback options for the teacher in this case.
```html
<iframe src="https://...?student=RbzZ5qxbNxmYkX3X"></iframe>
```
## Communication with LearningView 

The **iframe** can communicate back to LearningView in various ways. LearningView will use this data to display a done state for the task. The data send may be in a custom format or a xAPI score.

```javascript
window.parent.postMessage('*', {result:{score:{scaled:0.75, raw:3, max:4}}}
```
See [xAPI Documentation](https://xapi.com/statements-101/) for an example of a xAPI **Result** Statement. LearningView will only require either the `result.score.scaled` or `result.score.raw` and `result.score.max` values. If `result.score.scaled` is provided this value is used. Otherwise `data.result.score.raw / data.result.score.max` is calculated to get a percentage value. This solved value will be displayed as 75% on the LearningView task. If the value is 1 (100%) the task is marked as completed in LearningView with a checkmark instead of 100%.

The interactive content can request LearningView to download a file by providing an URL. This is required because the LearningView app will not allow file downloads from inside iframes directly. To invoke a download the iframe can call:
```javascript
window.parent.postMessage('*', 'DOWNLOADFILE|https://myurltodownload.com'}
```

There might be custom messages send via postMessage defined by the provider specific to his content. 

