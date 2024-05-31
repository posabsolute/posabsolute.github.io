---
layout: post
title: "Exploring Cloudflare Workers"
---

<style>
     h1 {
        font-weight: normal;
        line-height: 1.5em;
        font-size: 28px;
        margin-bottom: 10px;
    }
    .post-title {
        margin-bottom: -0.5rem;
    }
    blockquote {
        margin-left: 10px;
        margin-right: 10px;
    }
    h2 { font-weight: normal; }
    .w {
        padding: 3em 1em;
    }
</style>

![](https://github.com/posabsolute/posabsolute.github.io/raw/main/_posts/jsdev.png)

The JavaScript landscape has always been a fertile ground for innovation. New frameworks, libraries, and platforms emerge every year, each promising to simplify development or enhance performance in unique ways. This constant churn of ideas and implementations can be overwhelming, but it also represents an opportunity to push the boundaries of what we can achieve as developers. How many languages enable you to build back-end services, web UI and native apps all at the same time? 

> Side note, if you want a fresh look at where full-stack JS frameworks are going, in my opinion, your first stop should be [Solid JS Start](https://start.solidjs.com/) and [Ryan Youtube Channel](https://www.youtube.com/@ryansolid).

This speed at which our ecosystem evolves can also be prone to be comical; the [Programmers are also human Youtube channel](https://www.youtube.com/watch?v=aWfYxg-Ypm4) hits right in the heart of what makes the JS landscape so overwhelming.

It also makes choosing long-term frameworks and libraries difficult. I want to build software that I can enjoy building for years to come, and picking out small libraries tends to result in half your stack not being maintained, and that's just painful. Rant off, I wanted to test out what 2024 had to offer with a small side project and here is how it went.

### Cloudflare has now a Developer Platform?

I was quite surprised to learn that the internet DDOS protection platform had launched products into the serverless ecosystem, the promise is interesting: an entire set of tools set for serverless replicated around the world, meaning no round trip back to the US if you are serving in Australia, it felt like a good place to start.

- **[Cloudflare Pages](https://pages.cloudflare.com/)** to host my static site and front-end app (no Next.js here).
- **[Cloudflare Workers](https://workers.cloudflare.com/)** to serve my API.
- **[Cloudflare D1](https://developers.cloudflare.com/d1/)** as my database.
- **[Cloudflare KV](https://developers.cloudflare.com/workers/learning/how-kv-works/)** as my cache.

For the app, let's go all in in the hype cycle [TypeScript](https://www.typescriptlang.org/), [Hono](https://honojs.dev/), [Drizzle](https://github.com/drizzle-team/drizzle-orm), [Lucia Auth](https://lucia-auth.vercel.app/), and [Mantine](https://mantine.dev/) for the front-end.

### The Cloudflare DX

Cloudflare's CLI tool, [Wrangler](https://developers.cloudflare.com/workers/wrangler/), streamlines the developer experience. it provides easy installation, deployment, local development capabilities, and configuration management.

### Enabling Services with Wrangler

Wrangler makes it easy to kickstart your project. I must say that I was very impressed by their local development tool. Don't get me wrong; it's simple and perhaps a bit shallow, but there is something quite nice about its simplicity. For instance, you can create a new Cloudflare Workers project with a simple command:

```sh
wrangler generate my-worker
```

Need a database or a key-value store? Add a few lines to the Wrangler config and restart your local server. Here’s an example configuration for KV and D1:

```toml
# wrangler.toml

kv_namespaces = [
  { binding = "MY_KV", id = "kv-id" }
]

[[d1_databases]]
binding = "MY_DB"
database_name = "my-database"
database_id = "db-id"
```

With these configurations, your KV store and database are readily available in Hono. There are no secrets or instantiations; they are just functions to be leveraged automatically.

```
// Using D1

const { results } = await env.DB.prepare(
    "SELECT * FROM Customers WHERE CompanyName = ?"
    )
    .bind("Bs Beverages")
    .all();

// Using KV


```


#### Lean and Mean Local Machine

One of the great aspects of Wrangler is it's low resource usage. Running my local development environment with my database and key-value store takes about 100MB of RAM and my CPU is mostly idle. No Docker required, just one command. This setup is perfect for my Macbook Air, saving me about 8GB of RAM and a ton of battery over a docker setup. 

### Deploying on Cloudflare

Deploying on Cloudflare is easy. You can auto-connect GitHub repos and deploy on PR merges. There are templates for [GitHub Actions](https://github.com/marketplace/actions/deploy-to-cloudflare-workers), and I managed to set this up in just one hour.

### The Concerns

#### Cloudflare D1 is SQLite

SQLite is an excellent database, but it feels like an unusual choice for a cloud product. With a 10GB limit, it's not really designed for web-scale applications. However, it can be a good fit for specific use cases. For instance, if each customer has their own database, this model works well. This is model that could work well when you deploy products for a customer, a good example would be how campfire is now being sold, have a look at [Once](https://once.com/) to know more. The $5 paid plan allows for up to 10,000 databases. 

#### The Free Tier Limitations

The 10ms CPU limit can be a bottleneck for  almost any average computations. Due to this limit, creating hashed passwords with most algorithms is challenging. The documentation is sparse, but further exploration reveals that Cloudflare provides native Web Crypto support, alleviating some of these issues.

With that said, there is no way to get around that in many cases, you won't be able to build your app and fully test it without paying 5$ per month for the paid worker version. Which gets us to another problem, for my side project I would really like to set usage limit, I'm not interested in paying hundreds of dollars because someone decided to attack my platform, however there is no way to stop usage limit, only notifications, if they comes in the night while you sleep, too bad.

#### Workers Are Not Very Powerful

When I first ran password verification, the endpoint took 5 seconds to respond, compared to 50ms on my local machine. Workers lack the computational power of most servers you would run yourself, so this is an important consideration when using Workers as a web API, I guess it's the same with any serverless setup, but I was quite surprised at the performance so soon in my exploration.

### Everything else

Cloudflare platform has much more features, Cloud storage, AI APIs, it's kinda the perfect platform dabbling into stuff rapidly, but still, the fact that you cannot set a limit is kinda unfortunate.


#### What about Hono, Drizzle, Lucia

What is there to say? It's just another set of libraries doing what they are advertised to do. Hono is like Express and quite good. Drizzle is just a small layer over SQL, and Lucia does a good job exposing best practices in auth and sessions.

### Conclusion

Overall, I'm pleased with my experiment. It allowed me to explore new technologies, delve into the serverless paradigm, and see what serverless can achieve in 2024. The results are impressive, and I look forward to the future. Happy coding!
