Hey everyone. Last time we talked a bit about what accessibility is, why it's important, and how you can incorporate it into your process. Now it's time to roll up our sleeves and get something done.
 
I'll be guiding you through my process of making LiveBeats more accessible by using Git's super powers to travel back in time. Let's set our time machines to [ccfccd2](https://github.com/fly-apps/live_beats/tree/ccfccd251c9fc6babce9ce5c05f38b8febbd95c3), or mid-November. Winter is about to ramp up, Thanksgiving is on the horizon, and I'm starting to dig into this little app called LiveBeats...
 
## Low-hanging fruit
 
Sometimes my initial work to make an app accessible feels like surveying a fog-shrouded landscape. Is that supposed to be a button? What's all that clutter over there? Eventually the fog will clear but getting there takes effort. Fortunately, there are usually easy early victories that make a huge difference.
 
Labels are a great way to clear the fog. You've got a few tools to help with this, each of which has its own use cases:
* The faithful `alt` attribute is essential for images that have meaning, whether they're beautiful photos or actionable controls.
* Embedded SVGs are a separate beast entirely. An SVG image doesn't use `alt` at all, but instead has a `<desc/>` child tag. Also, since SVGs are their own unique element, there's a [*lot* you can do with child tags and attributes to make these more accessible](https://www.deque.com/blog/creating-accessible-svgs/).
* For elements that aren't images, and for which there is no easy way to add text, the `aria-label` attribute adds a label that is only visible to assistive technologies.
 
Unfortunately this commit no longer builds--my time travel analogy has its limits--but in [fad3706](https://github.com/fly-apps/live_beats/tree/fad3706), I use `aria-label` to add an accessible label to a button that skips to the previous track in the playlist:
`<button type="button" class="sm:block xl:block mx-auto scale-75" phx-click={js_prev(@own_profile?)} aria-label="Previous">`
 
Ideally you'd just add text as a child of the button, particularly since users with cognitive disabilities may struggle with what a given icon means. If you’re using an image for your button, add an `alt` attribute. Where neither is the case, `aria-label` is the ticket.
 
Labeling controls is only *part* of the story, however. *Hiding* irrelevant images can be just as important. If my screen reader presents a thing, then it should be relevant to me. Decorations and placeholders usually aren’t, and should be hidden either by applying `aria-hidden="true"` to the element, or by adding a blank `alt` attribute to images.
 
You can see an example of hiding icons in [f7db67f](https://github.com/fly-apps/live_beats/tree/f7db67f):
 
`<span class="mt-1"><.icon name={:user_circle} aria-hidden="true"/></span>`
 
Since that icon is only decorative, hiding it removes what appears to be an empty element from being read by my screen reader. It's a small thing, but half a dozen small things add up.
 
## Role with it, but not *too* much
 
We've labeled some things and hidden others. The fog is burning away, and we have a slightly clearer view of the land around us. It's now time to fill in the details.
 
Roles are powerful tools that make one thing appear to be another. Say you have a `<div/>` tag that your designer insists on using as a button. You can make it *seem* like a button like so:
 
`<div role="button">No really, click me!</div>`
 
Unfortunately, the above is the equivalent of someone slapping on a fake mustache and glasses, then pretending to be a master of disguise. Adding `role=”button”` doesn’t change anything about the behavior of this `<div>`. To my screen reader, the `<div>` *looks* like a button. But it doesn't focus when I tab, doesn't click when I press space or enter, and behaves just oddly enough that something doesn't feel quite right.
 
It's better to use a `<button/>` if what you want is a button. But if you can't, or if you're building a widget like a dropdown menu or tree, roles are crucial. Roles are also great for marking up semantic regions on pages. Here are the most important:
* `role="navigation"`: Use this, or a `<nav/>` element, for application menus.
* `role="main"`: Use this, or the `<main/>` element, for the main area of your page. There should only be one `main` element or role per page.
* `role="article"`: Use this, or the `<article/>` element, for self-contained page elements representing posts in a blog or forum.
 
Notice that all of the above roles have semantic HTML equivalents. This isn't universally true--there isn't a semantic HTML equivalent of a tree control for instance--but you should prefer HTML where possible.
 
There's a lot to unpack here, but [5cf58b2](https://github.com/fly-apps/live_beats/tree/5cf58b2) has some interesting highlights. Here we use the `<main>` element to surround the page content that changes when new routes are visited. `role="main"` would have behaved identically. 
 
We also use another technique to associate names with areas of the page:
 
```
    <!-- player -->                                                                                                                
    ...
    <div id="audio-player" phx-hook="AudioPlayer" class="w-full" role="region" aria-label="Player" >                                
      <div phx-update="ignore">                                                                                                    
        <audio></audio>                                                                                                            
      </div>                                                                                                                        
    </div>
```
 
This last snippet does a couple cool things. It starts by adding a new region landmark to the page for easier navigation via hotkey. In [NVDA](https://www.nvaccess.org/about-nvda/), I can jump between this and other interesting regions of the page by pressing _d_. It also associates the label "Player" with that region, such that navigating into or out of the player explicitly announces that focus enters or leaves the player. Since you can't guarantee that a user will enter the player from the top of the page, giving the entire box of controls a meaningful label is a very useful touch. If you find yourself writing documentation with phrases like "In the player, click _Pause_" or "Find this in the messages section of the dashboard," named regions help make those instructions more relevant to screen reader users. Styling may make the player region visually obvious, but the `region` role and "Player" label makes it very apparent when the caret enters the audio player.

Roles are powerful in large part because they're promises. If you promise your user that a thing is a button, then it needs a `tabindex` for keyboard focus, as well as handlers to activate it when enter or space is pressed. If you're curious about what these promises are, the [WAI-ARIA Authoring Practices](https://www.w3.org/TR/wai-aria-practices-1.1/) document is an exhaustive source of all commonly expected keyboard behaviors for a bunch of widget types. Want to know how a list box should behave when arrowing down past the last element? This resource has you covered.
 
That said, it is very possible to overuse roles in your application. Here are two examples of role misuse I often find in the wild.
 
### Menus aren't what you think
 
`role="menu"` is not intended for every list of links in your app that might possibly be a menu if you tilt your head and the sunlight lands just so. These are either application menus like you'd find in a menu bar, or standalone dropdown menus that capture keyboard focus and expand when you click on them. Misusing `role=”menu”`  won't necessarily make a given control unusable, but it’s confusing because it presents a given element as something it often isn't.
 
## This one weird trick makes your entire app inaccessible in 30 seconds!
 
This one gets its own buildup and section because it's *that* bad. If you want your app to be so inaccessible that most screen reader users turn away in the first few seconds, slap `role="application"` on one of the top-level elements. Explaining just why this is takes a bit of effort, so please bear with me for a bit.
 
Broadly speaking, screen reader users browse the web in one of two modes. Caret browsing mode presents pages as if they're documents. We can arrow through their contents line by line or character by character, select and copy text, etc. You can usually experience a limited version of this by pressing F7 in most browsers, though screen readers enable this mode by default. They also add various conveniences for jumping between headings with _h_, buttons with _b_, landmarks with _d_, etc.
 
Focus mode, sometimes called “forms mode” because it enables automatically when form fields are highlighted, passes keys directly through to the application. This is generally what you want when typing into forms, or when using keyboard commands that you don't want filtered out by caret browsing. Incidentally, focus mode is closest to how native applications behave. You don't normally read application screens as if they were documents, or jump between controls by element type.
 
And that, more or less, is what `role="application"` enforces. It enables focus mode by default, meaning your screen reader users can't explore via caret browsing and its associated commands. It also changes the way the site presents itself to screen reader users such that it appears to be a native app. It’s a promise that you've gone through the extraordinary effort to ensure all your controls are focusable, they all have sensible keyboard behaviors, and that your users won't be struggling to read text that changes. You might feel you have to use role="application" just because, well, you're making an application. But this is often not the right choice.
 
If you've ever been annoyed by an Electron app's non-native behavior, multiply that by about 11 and you're in the ballpark of how frustrating `role="application"` can be when it’s not backed up by thorough and consistent handling of every standard interaction. While I've got this podium: Slack, you're one of the biggest perpetrators of this and need to cut it out. Most of my usability issues with Slack spring from its use of `role="application"` everywhere, and the haphazard and nonstandard workarounds they've put in place to patch in what the browser gives them for free. 
 
## Closing thoughts
 
While this post has been a bit light on the real-time aspects of LiveBeats, sorting out this low-hanging fruit is an important step to making any web app accessible. Next time we'll start going live, exploring the challenges and methods to accessibly presenting changes as they roll in over the wire. Meanwhile, if you have questions or topics you'd like covered, drop a line [here](https://community.fly.io/t/accessibility-for-real-time-web-apps/4395) and I'll try to work them in. Thanks for reading!

