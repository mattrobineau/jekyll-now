---
layout: post
title: Deploy React Apps To Subfolders
---

Trying to deploy a react app to a domain's subfolder requires a bit of documentation which may be a little difficult to understand. The good news is, I've just had to do this and after a bunch of google-fu, I was successful in deploying an app to a subfolder.

By default, when you `yarn build` your app, react will assume that you are deploying to the domain root (`/`). This means that when CSS and JS are transformed, the urls will be relative to the root, eg `/static/css/*`, `/static/js/*`.

But what happens if you have nginx configured like this:

{% highlight conf %}
server {
        # SSL Config

        listen 443 ssl;
        listen [::]:443 ssl;

        server_name matthieurobineau.com www.matthieurobineau.com;

        root /var/www/matthieurobineau.com;
        index index.html;

        location / {
                try_files $uri $uri/ =404;
        }

        location /example {
                index index.html;
                root /var/www;
                try_files $uri /index.html;
        }

        # [..snip..]
}

{% endhighlight %}

I created a directory in `/var/www/example` which contains my react app. When I run the app, I navigate to `matthieurobineau.com/example`, but all my JS/CSS/URLs point to a url similar to `matthieurobineau.com/static/css/`. My css in this example is actually in `matthieurobineau.com/example/static/css/`.

Fixing the css and js during transformation is simple. Add the line `"homepage":"https://matthieurobineau.com/example/"` to your `package.json`.

{% highlight conf %}
{
  "name": "react-redux-typescript",
  "version": "0.1.0",
  "private": true,
  "dependencies": {
    "react": "^16.4.1",
    "react-dom": "^16.4.1",
    "react-scripts-ts": "2.16.0"
  },
  "scripts": {
    "start": "react-scripts-ts start",
    "build": "react-scripts-ts build",
    "test": "react-scripts-ts test --env=jsdom",
    "eject": "react-scripts-ts eject"
  },
  "devDependencies": {
    "@types/jest": "^23.1.1",
    "@types/node": "^10.3.4",
    "@types/react": "^16.4.1",
    "@types/react-dom": "^16.0.6",
    "typescript": "^2.9.2"
  },
  "homepage": "https://www.matthieurobineau.com/wows/"
}
{% endhighlight %}

Now when you build, `yarn build` will output this

```
The project was built assuming it is hosted at /example/.
You can control this with the homepage field in your package.json.
```

The paths will now use the correct url (`/example/static/css/`). Notice that the build will drop the hostname `https://matthieurobineau.com` from the generated url. This will not however update any links which you create yourself. You need to prepend the URLs you use in your js or in my case, tsx files.

_Before_
{% highlight html %}
<img src={props.imageUrl} alt={props.name} />
{% endhighlight %}

_After_
{% highlight html %}
<img src={`${process.env.PUBLIC_URL}` + props.imageUrl} alt={props.name} />
{% endhighlight %}

```${process.env.PUBLIC_URL}``` will add the `homepage` value as a string which will be prepended to `props.imageUrl`. If `props.imageUrl` is `/images/logo.png`, this will output `/example/images/logo.png`.

It might be possible to drop the domain name in the `homepage` field of `package.json`, but I have not tried it and the react [documentation](https://github.com/facebook/create-react-app/blob/master/packages/react-scripts/template/README.md#building-for-relative-paths) uses the full URI.