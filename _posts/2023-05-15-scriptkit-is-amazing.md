---
tags:
  - javascript
  - scriptkit
---

# Script Kit is amazing

I'd been meaning to try out [Script Kit](https://www.scriptkit.com/) after I'd seen it mentioned by the likes of Kent C. Dodds AND Matt Pocock on Twitter.

I had been looking at Android and WearOS to allow me to quickly start and stop my [productive.io](https://productive.io/) timers without having to use the web interface (which is fine, but far from optimal!). But Java development 🤢. Then I realised this is exactly the kind of thing Script Kit is for! Only on my desktop PC still, not on my watch or phone, alas.

## My Script Kit scripts

Script Kit can make web requests using fetch, has a nice `env` API, can let you choose from lists and show desktop notifications.

### Env settings

Need an environment variable? Just call the `env` function with the name of the key and a prompt for the user to enter it if it isn't there: `const API_KEY = env('API_KEY', 'Please enter your Productive API key')`.

TBH it would have been nice to give some more help text here: I'd written down how to find this value - but I can't see a way to add that extra text. The API is beautifully simple though.

### Shortcuts

Because Script Kit monitors your scripts for changes, you can add an OS-wide shortcut key by just adding a comment to the top of your script like:

```
// Shortcut: ctrl+alt+shift+,
```

### Notifications

I couldn't find mention of notifications in the docs, but I asked on Twitter and was [told it existed](https://twitter.com/scriptkitapp/status/1658456135417102336?s=20):

```
notify({title: 'Timer started'}, { body: `Timer started for ${project.name}`})
```

### Productive OpenAPI

One issue I ran into was that [productive.io](https://productive.io/) don't have a javascript client or even an OpenAPI spec document to allow me to build my own! So I had to muddle through writing fetches manually 😅 with no type safety.

## Productivity++

So far the most useful scripts have been to stop the current timer and start the last active one with a keyboard shortcut - so useful when I want to go for a walk and lunch, and when I get back! 🚶 Here's a video of it in action (without desktop notifications!)

https://github.com/IanIsFluent/blog/assets/34126328/dd3cfd54-7842-4c7e-934e-c6c4174ac379

The next thing I'd like to be able to do is restart a timer on any recent project - and stop the current one if there is one. The problem is that you only want to restart the timer if its the same day - so I need to do a bit of work to create new time entries based on the selected project if it was last run yesterday or before.

I've put my scripts in an open [GitHub repo here](https://github.com/IanIsFluent?tab=repositories).
