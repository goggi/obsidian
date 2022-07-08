
```
// ==UserScript==
// @name         crawlyfi.gitpod.com
// @namespace    http://tampermonkey.net/
// @version      0.1
// @description  try to take over the world!
// @author       You
// @match        https://*.gitpod.crawlyfi.com
// @icon         data:image/gif;base64,R0lGODlhAQABAAAAACH5BAEKAAEALAAAAAABAAEAAAICTAEAOw==
// @grant        none
// ==/UserScript==
history.pushState(null, document.title, location.href);
window.addEventListener('popstate', function (event){
    history.pushState(null, document.title, location.href);
});
```

```
n// ==UserScript==
// @name         gitpod.io
// @namespace    http://tampermonkey.net/
// @version      0.1
// @description  try to take over the world!
// @author       You
// @match        https://*.gitpod.io
// @icon         data:image/gif;base64,R0lGODlhAQABAAAAACH5BAEKAAEALAAAAAABAAEAAAICTAEAOw==
// @grant        none
// ==/UserScript==
history.pushState(null, document.title, location.href);
window.addEventListener('popstate', function (event){
    history.pushState(null, document.title, location.href);
});
```

