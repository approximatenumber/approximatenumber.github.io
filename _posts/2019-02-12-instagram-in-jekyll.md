---
layout: post
title: How To Add Instagram Feed To Your Jekyll Site
tags: instagram jekyll
comments: True
excerpt_separator: <!--more-->
---

*Small tutorial for you who want to show your nice pics from instagram on your small Jekyll site :)*

1. Register your app in [Instagram Dev](https://www.instagram.com/developer).

You will get `Client ID` and `Client Secret` there. Don`t forget to add Redirect URI (you can use just http://example.com as a redirect URI, because we need it only to get the special code).

2. Get the app CODE , directing your browser to `https://api.instagram.com/oauth/authorize/?client_id=CLIENT-ID&redirect_uri=REDIRECT-URI&response_type=code`. Paste your `CLIENT_ID` and `REDIRECT-URI` from previous step.

3. Get code for your app with (paste `CLIENT_ID`, `CLIENT_SECRET`, `AUTHORIZATION_REDIRECT_URI` and `CODE`)

```bash
curl -F 'client_id=CLIENT_ID' \
     -F 'client_secret=CLIENT_SECRET' \
     -F 'grant_type=authorization_code' \
     -F 'redirect_uri=AUTHORIZATION_REDIRECT_URI' \
     -F 'code=CODE' \
     https://api.instagram.com/oauth/access_token
```

Yep, you will get `access_token` in a`curl` answer.

4. [Download](https://github.com/stevenschobert/instafeed.js) `instafeed.js` and move it to the place in your jekyll repo with other javascript-files (example: `assets/js`).

5. [Get](https://codeofaninja.com/tools/find-instagram-user-id) your instagram user id.

6. Add all the options to your `_config.yaml`:

```yaml
instagram_user_id: USER_ID
instagram_client_id: CLIENT_ID
instagram_access_token: ACCESS_TOKEN
```

5. Add `instagram.html` to your `_include` directory:

```html
<script type="text/javascript" src="/assets/js/instafeed.js"></script>
<script type="text/javascript">
    var feed = new Instafeed({
        get: 'user',
        userId: '{{site.instagram_user_id}}',
        clientId: '{{site.instagram_client_id}}',
        accessToken: '{{site.instagram_access_token}}'
    });
    feed.run();
</script>
<div id="instafeed">
```

6. And finally add this include file on the page you want to show an instagram feed (example: `_pages/about.html`):

```html
{% include instagram.html %}
```

That`s all.

You can find more information about instafeed-js plugin [here](http://instafeedjs.com/).
