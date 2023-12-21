# Navigating the Complexities of Transactional Email Development in 2024: A Historical Perspective and Some Solutions

Email development has historically been a complex field, marked by the challenges of dealing with inconsistent HTML rendering across various email clients. This reached a boiling point when Outlook transitioned from the IE6 engine to Microsoft Word's rendering engine around 2009, amplifying the difficulties in creating consistent email layouts.

 As you can guess, the community went crazy. Multiple blog posts and campaigns were launched to try to convince Microsoft to take a step back. Unfortunately, the opposite happened. Microsoft confirmed their choice and published a blog article [The Power of Word in Outlook](http://web.archive.org/web/20090627004005/http://blogs.msdn.com/outlook/archive/2009/06/24/the-power-of-word-in-outlook.aspx). It can be be summarized in one simple quote:

> "We've made the decision to continue to use Word for creating email messages because we believe it's the best email authoring experience around…"

And just like that, *the world changed*; floats were gone, layout with tables it was, and emails never fully recovered, even to this day.

## The Arduous Journey of Email Development

Crafting emails that can render consistently across various clients (outlook, Gmail, mobile clients, etc.) means resorting to unconventional HTML and CSS practices compared to modern practices; in a nutshell, you need to use tables for layout and inline style everything.

Eight years ago, I created [Inker](http://inker.position-absolute.com/); with it, I tried making the ultimate toolbox for building and sending emails. It had it all: strong templating with the Ink framework as a foundation, a bunch of CLI commands to test and generate emails and a service to send them. *It had everything but traction, and no one ever adopted it*.

> It seems transactional emails is this thing every company needs to do, but almost nobody knows how or wants to do it, and since it's so dirty, let's do it dirty. However, I wouldn't say this was the only reason why Inker failed, but that's for another time.

## Modern Solutions: A Search for the Email Holy Grail

And so, it's 2024; I wanted to have a look at current solutions; things must have changed since 2016, right? And it did! But not as much as I wanted.

### React-Email: Bringing React to Email Development

[React-Email](https://react.email/) and its fork, [JSX-Email](https://jsx.email/), tap into the power of React, allowing developers to build email templates using familiar JSX syntax and React components. The advantages should be apparent: you get several pre-backed components, the JSX syntax, and you can quickly build strong conditional logic based on email variables. This happens a lot in transactional email; think of a receipt where the email needs to support many conditions to show product pricing or show sections based on some conditions.

React-Email produces the cleanest email templates I ever witnessed (example below), especially if you compare the tag soup it needs to generate for the email client.

```jsx
<Container style={container}>
  <Heading style={h1}>Confirm your email address</Heading>
  <Text style={heroText}>
    Your confirmation code is below - enter it in your open browser window
    and we'll help you get signed in.
  </Text>
  <Section style={codeBox}>
    <Text style={confirmationCodeText}>{validationCode}</Text>
  </Section>
  <Text style={text}>
    If you didn't request this email, there's nothing to worry about - you
    can safely ignore it.
  </Text>
  <Section>
    <Row style={footerLogos}>
      <Column>
        <Row>
          <Column>
            <Link href="/">
              <Img
                src={baseUrl + '/static/slack-twitter.png'}
                width="32"
                height="32"
                alt="Slack"
                style={socialMediaIcon}
              />
            </Link>
          </Column>
          <Column>
            <Link href="/">
              <Img
                src={baseUrl + '/static/slack-facebook.png'}
                width="32"
                height="32"
                alt="Slack"
                style={socialMediaIcon}
              />
            </Link>
          </Column>
        </Row>
      </Column>
    </Row>
  </Section>
...
```

#### What's the catch?

If you control the entire email codebase, as in, your customer cannot build/modify their own transactional email that are then stored in a DB; this solution makes a lot of sense. 

This is as long as you thoroughly validate the variables injected into the email JSX template. This is still JavaScript, executed on your server. It's your javascript, but email variables (customer name, for example) could come from a variety of sources and could be an attack vector on your server.

##### What about if I want to enable an online email editor to allow email templates to be modified?

You have a platform that allows your customers to modify transactional email templates sent to their own customers? Now, things become much more complicated.

Essentially, there is no way to execute a customer JSX template in a fully secure way on your server; sure, you could use an [Isolate](https://github.com/laverdet/isolated-vm) or [JsxParser](https://www.npmjs.com/package/react-jsx-parser), but it's only as safe as these projects are, and do you want to take that risk for your transactional emails stack? I'm guessing you don't.

```js
// Example of how to render email with jsxParser
ReacEmail.render(<JsxParser jsx={jsxString} components={reactEmailComponents} bindings={{...variablesObject, ...emailStyles}} />)
```

You could also pre-render the JSX email to an HTML soup, but then what about the email variables? You must add a layer on the last-mile delivery to inject those variables into the email before it's sent. Yuck.

There is only one solution I can think of: serverless functions. But this feels too hard for what this provides, I want simple and elegant.

#### Should you use React-Email and JSX-Email

If your company's approach to transactional emails involves using fixed, unmodifiable templates that are designed to be populated with variables at the time of sending, then adopting React-Email or JSX-Email is a strong choice. These tools are particularly well-suited for scenarios where the email templates are not subject to customization by the end user, and only need dynamic variable insertion. The developer experience provided by React-Email and JSX-Email is outstanding, offering streamlined and efficient template creation. 

### MJML: Streamlining Responsive Email Design

[MJML](https://mjml.io/) focuses on easing the creation of responsive emails with its own markup language, ensuring compatibility across different email clients.

#### Advantages of MJML:

- **Simplified Email Markup**: MJML abstracts the complexities of email HTML, offering a simpler markup language specifically tailored for email design.
- **Responsive by Default**: Designed to produce emails that automatically adapt to different screen sizes, ensuring a consistent user experience across devices.
- **Cross-Client Compatibility**: Addresses the varying rendering engines of email clients, significantly reducing the need for client-specific hacks or adjustments.
- **Community and Tooling**: Comes with a supportive community and a range of tools, including an online editor and plugins, which enhance the development workflow.

```jsx
<mjml>
  <mj-body>
    <mj-section>
      <mj-column>
        <mj-text>
          Hello World!
        </mj-text>
      </mj-column>
    </mj-section>
  </mj-body>
</mjml>
```

#### Passing Dynamic variables to MJML

Despite it's strengths, MJML require supplementation with templating engines like Handlebars.js, Mustache.js, or EJS to handle dynamic content, adding a layer of complexity to the process.

It's alright, and if you're not a react shop, this is your best way to build great email templates; it's not the holy grail, but it's alright.

## Sending Transaction Emails

Almost all web frameworks include a library to send email, and most email service providers (Amazon SES, etc) also provide their own SDK, but two problems emerge:
* You still need to compile the email with the dynamic variable, which needs to be done before using those libraries
* Few of those libraries provide an optional queue/retry mechanism if it fails.
For me, it's almost inconceivable that my customers won't receive their emails. Think of a receipt; when those service email providers you use to send emails are down, your customers won't receive their receipt.

You need an optional queue and retry mechanism to manage this issue, with a bunch of integration with Redis, Kafka, Rabbit, etc. 

Furthermore, although using an SDK is all well and good in your monolith, at some point, most companies will have multiple services, and those services will also need to send emails. Those services might be in different languages, but your last template generation will probably be in 1 specific language.

It would be awesome to have a library that can operate either as an SDK integrated inside a service or can be started as its service, exposing API and events integration to send transactional emails.

> The need for a more holistic solution is evident to me — one that not only aids in creating email templates but also manages the intricacies of sending them, including robust queuing mechanisms.


## Conclusion

React-Email and MJML represent significant strides in transactional email development, yet the journey toward an all-encompassing solution continues. As the landscape evolves, the quest for a fully integrated transactional email service remains out of my grasp, for now.
