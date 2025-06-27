## SSG (Static Site Generation)

SSG challenges the hypothesis that content should be requested from the database for each request from a user. Instead, they suggest to generate content at **build time**, because once your pages have been generated from the CMS, they won't need to be changed until the next content change in your CMS.

So why not **generate all your website pages** as static pages **each time your content changes**? The way it usually works is you'll connect your content (CMS, headless CMS, file storage, etc.) to your CI engine using webhooks for instance, so that each time your content changes, a CI pipeline is triggered, thus rebuilding all your website and invalidating the cache.

Newer SSG such as Gatsby are built around frameworks, which means your static site will benefit from the framework architecture and ecosystem. Once your page is initially loaded, additional DOM interaction is handled by your code using the framework.

Benefits of SSG over SPA:

- **Better SEO** since your website consists of multiple HTML pages. Bot crawling is easier and your content will be indexed nicely.
- **Better performance**: Modern SSG tools include only the necessary JS/CSS code to run the requested page and will load additional code if the user navigates your website.
- **Content decoupling**: SSG provides a way to decouple your content from its representation.
- **Multiple source content** integration such as headless CMS, local markup files, or remote files. Serve from S3 or cache.

Drawbacks are limitations:

- **User customized content**: Static generation of content only works if the content pages are the same for all users. Otherwise, you would need to generate each page, for each user, which would take a lot of space and could lead to security issues, as all static pages are public. For user customized content (dashboard, forum poasts, inbox, etc.), you will need to make API calls to get your formatted content.
- **Learning curve**: Each SSG comes with opinionated ways of querying your content and generating your pages.
- **CI overhead and preview latency**: If you want your website to always be up to date with your content, you should plug webhooks and CI pipeline to make it work. If your pipeline takes 5 minutes to run, that's as long as you will have to wait to see your website updated with your content.

## SSR (Server Side Rendering)

Historically, SSR (or scripting) has been around for a long time. SSR simply means rendering your HTML content from the server. JEE, ASP.Net, PHP Symfony, Ruby on Rails, and all those regular MVC frameworks offer some kind of server-side rendering with templating engines. However, these SSRs have limitations: you cannot handle client interaction with it, you're **limited to the initial content** that will be displayed in the browser. Any following behaviors must be coded in additional JS files.

SSR in the JS world realy means **JavaScript isomorphic rendering.** With the advent of NodeJS, JS can now run both in the server and the client, which makes sharing rendering logic possible.

NextJS, Nuxt, and others offer a a way to share the rendering logic of your components, between the initial load from the server and the following interactions in the client.

With SSR, unlike SSG, your pages are rendered at run time for each request, which means you can customize the initial render of your page based on your user context.

Benefits:

- **SEO**: If you want user customized pages to be indexed on search engine or thumbnail preloaded on social media sharing, you need those pages to be server-side rendered.
- **Context update**: If your content sits in a CMS and you want every update to reflect instantly on your website, SSR is what you need.
- **Performance**: As with SSG, SSR usually implies better performance than SPA, at least for the first time to load the page content.
- **Backend and frontend in same codebase**: Since SSR runs on your seerver, you can also host your API with it, having a single repository for your API and your frontend.

Limitations:

- **A server is needed**: For SSR to work, you will need a server (and scale it). Some magical hosting platforms like Vercel will make use of serverless functions to do the pre-rendering, thus removing the need of a server.
- **Performance**: The initial load of the page will contain the content, but it is computed by a server which can be bloated. Contrary to SSG which serves static files, with SSR, you will need to make sure your initial loads is fast enough.
- **Learning curve**: For the isomorphic JS to work, frameworks enforce conventions on the way you create your components.
- **Not really universal**: Even though on paper, JS can be used both in server and client, there are differences between the 2 environments. For instance, `window` does not exist on the server, so you will need some "pragma" like code to make sure some of your frontend dependencies still work fine.
