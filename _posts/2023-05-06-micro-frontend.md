---
layout: post
title: "Exploring Micro-Frontends with Module Federation: Insights from Practice"
---

> This is a companion post to my [Confoo 2023 presentation](https://drive.google.com/file/d/1K0OWytOmc3tB21ohnBKv-tOQdn5xnQwY/view?usp=sharing).

## Introduction to Micro-Frontends

The concept of micro-frontends has become increasingly popular as organizations look to scale their teams and applications effectively. Micro-frontends essentially involve splitting a large, monolithic front-end application into smaller, independent applications. This approach offers numerous benefits, including improved scalability, better performance, and enhanced flexibility in technology choices.

## Why Micro-Frontends?

1. **Independent Development**: Teams can work on different parts of the application without impacting each other, leading to faster development cycles.
2. **Technology Agnostic*: Micro-frontends allow for using various technologies and frameworks, making it easier to adopt new tech or upgrade existing tools.
3. **Simplified Deployments**: Each micro-frontend can be deployed independently, enabling quicker updates and reducing deployment risks.

## Webpack Module Federation in Action

Webpack's [Module Federation](https://webpack.js.org/concepts/module-federation/) feature has largely improved how we implement micro-frontends. It allows for sharing code and components at runtime, making it possible to import a component from one micro-frontend to another as if they were local modules.

### Key Features:

- **Runtime Code Sharing**: Code can be shared and updated in real-time across different micro-frontends.
- **No NPM Dependency for Incremental Updates**: Changes can be pushed without relying on package managers like NPM, leading to more efficient update processes.
- **Framework-Agnostic**: It integrates directly into Webpack, offering a flexible approach that's not tied to any specific framework.

## Strategies for Implementing Module Federation and the AppDirect Approach

At AppDirect, we chose a tailored approach that aligns with our organizational structure and product requirements. Our strategy focuses on:

- **Container Architecture**: The [Container application](https://medium.com/nerd-for-tech/micro-front-ends-hands-on-project-63bd3327e162) manages shared logic across all micro front-ends. This architecture not only improves performance by handling routing efficiently (thus eliminating page refreshes) but also ensures better domain isolation. Each Micro front-end contains domain-specific code, facilitating independent development and deployment.

- **Version Tracking with APIs**: To manage the deployments of our micro-frontends, we employ an API-based approach for version tracking. This ensures that our container is always aware of the latest versions of each micro-frontend, aiding in seamless integration and updates.

- **Internal Design System for Consistency**: To maintain design consistency across our micro-frontends, we promote an internal design system. This includes a UI components library essential for stylistic uniformity, speeding up development time, and enhancing collaboration between developers and designers.

- **Handling Multi-React Versions**: Due to the distinct nature of our micro-frontends, we face scenarios where different React versions are used. We address this by ensuring that micro-frontends do not share scope with the container application and load the applications with `React.render()`, which means every micro front-ends exposes it's own React-Dom and are not part of the container React life cycle.

## Deployment Strategies and Considerations

Implementing micro-frontends requires thoughtful considerations around deployments and version tracking. It's crucial to have a strategy that allows the container application to be aware of updates in micro-frontends.

### Considerations:

- **Version Tracking**: Ensuring that the container application is aware of the versions of micro-frontends is key to smooth deployments.
- **Leaking CSS**: CSS can leak from one micro-frontend to another, leading to conflicts and styling issues. This can be addressed by using CSS modules or CSS-in-JS. Be careful which UI library you choose, as some of them are not compatible with the micro front-ends approach.
- **Design Consistency**: Maintaining a consistent look and feel across micro-frontends is essential. This can be achieved through shared design systems and close collaboration between developers and designers but it might never be perfect, this is a trade-off that you need to be aware of.
- **Server-Side Rendering (SSR)**: While possible with micro-frontends, SSR introduces additional complexities, especially regarding security and performance.

### Should You Use Micro-Frontends?

Micro-frontends offer significant advantages for organizations struggling with monolithic front-end architectures or scaling issues. However, they require a careful approach to implementation, particularly in areas like deployment strategy and managing dependencies.

## Conclusion

Micro-frontends, empowered by tools like Webpack Module Federation, present a compelling solution for scaling front-end development. They promote independent development cycles, enable a diverse technology stack, and facilitate easier upgrades and deployments. However, the journey to micro-frontends demands careful planning, especially around deployment strategies and maintaining a unified design language across different front-end segments.

*Looking to implement micro-frontends in your organization? I recommend you look at the [Module Federation examples](https://github.com/module-federation/module-federation-examples), it contains examples of how to implement module federation for every case imaginable.*

