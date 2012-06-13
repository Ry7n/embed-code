Meebo Bar embed code
====================

The Meebo Bar loads asynchronously by creating a same-domain iframe and injecting the javascript loading code into that iframe.
See the commented code for details, and watch the Velocity 2010 presentation for an overview of the Meebo Bar performance in general (http://www.youtube.com/watch?v=b7SUFLFu3HI)

Why is this forked?
===================

When Google acquired Meebo, the team took down the blog post describing this project. It contains some useful hints about developing 3rd party API services and working with iFrames. Below is the full text of the blog post as captured from [archive.org](http://web.archive.org/web/20110623052414/http://blog.meebo.com/?p=2956).

> I’m happy to announce that we’ve open sourced the Meebo Bar embed code! It is the result of more than two years of iteration and careful consideration for the best way to embed a third party webapp on your website. I’ll get into the technical details below, but first let’s take a quick trip down memory lane and see how the embed code got to where it is today :)
> 
> The first commit to the Meebo Bar codebase was made by Paul S. on the 4th of June, 2008. Historic weather reports say that it was a beautiful Wednesday in Mountain View CA, with the sun shining and temperatures in the low 70s. Eleven days later, on June 25th, Martin H. made his first commit to the Meebo Bar – he would go on to be the key engineer for the first 7 versions of the bar. As the Meebo Bar gained traction, more and more of our developers started working on the bar codebase.
> 
> Today, roughly two and a half years later, the Meebo Bar sits on thousands of websites and reaches roughly a third of the US population every month. There are many reasons for the bar’s success (the greatest reason is the Meebo team, which you can be a part of too!), but today I want to highlight the embed code.
> 
> We knew when we set out to create the Meebo Bar that we could not let it block the hosting page from loading or rendering. Even if the Meebo servers were to respond slowly, we had to ensure that the rest of the page would not be adversely affected! In addition, we did not want our Javascript to leave a footprint on the hosting page (called “namespace pollution” in the javascript community), and we wanted to avoid having our code break if the Javascript environment of the hosting page changed (e.g. if the webpage made changes to the Javascript prototype objects).
> 
> The solution that addressed all of these issues was to create an iframe and then load all of the Meebo code into it. It sounds simple, but there are evil edge cases and arcane tricks that you need to discover in order to make a truly non-blocking third party webapp embed code. We are happy to share our discoveries, and hope you will find it useful!
> 
> The first couple of iterations of the bar embed code required the hosting site to host an html file on their domain. Our embed code created an iframe that loaded that html file, and once the html file had loaded completely it in turn would load all of the bar’s Javascript and CSS. Since the html file was hosted on the site’s domain, the state of Meebo’s servers would not affect the iframe’s loading time. In addition, the Javascript that was loaded into the iframe would have access to the hosting page to draw the Meebo Bar on it and to integrate with the contents of the page, since the iframe was not “cross-domain”.
> 
> The major shortcoming of this first approach was that the hosting site had to put up a custom html file on their servers. Not only was it a technical and error-prone hassle that would slow down launching the bar on new sites, but some sites did not have the capability to put up a custom html file. Obviously, something had to be done!
> 
> The solution was to still create an iframe, but rather than set its src attribute and load an external html file, Martin came up with a great hack that allowed for us to inject Javascript into the iframe right from the embed code. The resulting context of the iframe is the same domain as the hosting page, so the javascript that executes inside of the iframe still has access to the DOM of the hosting page (i.e. it’s not “cross-domain”).
> 
> With a working solution in hand the solution can often seem obvious. However, I have a genuine appreciation for the two and a half years of discovery that lead us to the current version of the embed code. I’ll let you read the commented source of the embed code by yourself below, and simply leave off with a list of features of the embed code that I find particularly awesome:
> 
> > [Line 46](blob/master/embed-code.js#L46): Create the Meebo function, which is the page’s asynchronous API for Meebo functionality. It can be used right away, even before the Meebo Bar code has finished loading. When the Meebo bar code has finished loading, it grabs all the argument objects already pushed onto the Meebo._ array and executes those API calls – after that the code reassigns Meebo._.push to a function that immediately executes the API call. It works brilliantly!
> > 
> > [Line 69](blob/master/embed-code.js#L69): We want to know how the embed code behaves in the real world, so we track the time it takes to execute the embed code and then load the Meebo bar code. This way we can ensure that the bar doesn’t slow down as we add new features and keep innovating.
> > 
> > [Line 83](blob/master/embed-code.js#L83): Every byte counts! Since the embed code will be included inline on every webpage of your site, we do everything we can to make it tiny. By creating named variables for all the common DOM operations, the closure compiler can compress them all into single-letter variables.
> > 
> > [Line 148](blob/master/embed-code.js#L148): In IE, if the hosting page has set document.domain then the created iframe without the src attribute set is considered “cross-domain” and reaching into its contentWindow will throw an exception. Luckily, we can try/catch this case and in the case of an exception set the iframe src to a javascript: URL. In IE, this will have the javascript in the URL execute in the context of the iframe – that javascript then sets the document.domain of the iframe contentWindow, after which we the iframe is no longer “cross-domain”, and can access the iframe’s contentWindow again.
> > 
> > [Line 159](blob/master/embed-code.js#L159): This function creates the HTML that will be injected into the iframe. The generated HTML does two things: 1) creates an onload function for the contentWindow of the iframe, and 2) inside of the onload function, creates a script tag that points to a meebo server and appends that script tag to the head of the contentWindow of the iframe.
> > 
> > [Line 188](blob/master/embed-code.js#L188): A webpage’s onload event will fire only after the onload of all its children iframes fire. When we’ve injected the HTML that will load the Meebo Bar JS into the iframe, we call document.close on the iframe contentWindow – this is what causes the iframe’s onload event to fire. Now, with the iframe’s onload event fired, we’re free to load the Meebo Bar JS into the iframe without blocking the onload event or holding up the loading of assets of the parent page.
>
> Cheers!  
> Marcus
> 
> ps. This is the raw source of the Meebo embed code. It’s not straight javascript, as we have a pre-processor that we run over the raw embed code to strip out comments and macros in order to produce the compressed “production” versions of the code. This raw version with its detailed comments is perfect for educational purposes!
