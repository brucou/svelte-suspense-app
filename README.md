## Motivation
This is a proof of concept for offering functionality similar to that of React Suspense. It is 
inspired from [previous work](https://reactgo.com/tutorial-reactlazy-react-suspense/) by [Sai 
gowtham](https://twitter.com/saigowthamr) that I reimplemented here with Svelte in lieu of React.

## What it is 
![suspense demo](assets/suspense%20demo.gif)

# Markup and semantics
The main screen which will hold the loaded image is handled with a suspense functionality:

````html
<div class="app">
    <Header />
    <div class="albums">
        <Suspense {settings}>
            <div slot="fallback" class="album-img">
                <img alt="" src="https://media.giphy.com/media/y1ZBcOGOOtlpC/200.gif" />
            </div>
            <LazyLoadContainer>
                {#each albums as album, i}
                <LazyLoad id="{i}">
                    <Album {album} />
                </LazyLoad >
                {/each}
            </LazyLoadContainer>
        </Suspense>
    </div>
</div>

````

The default slot content of the `Suspense` component will be immediately put in the DOM and 
hidden (`display: none`). Two things can happen:

- the default slot content signals completion before `settings.duration` milliseconds. In that 
case, the slot content is displayed as soon as it has indicated completion
- the default slot content does not signal completion before `settings.duration` milliseconds 
expires. In that case, the fallback content (e.g. the `fallback` slot content) will be displayed
. When the default slot content signals completion, it will be displayed.
 
The `Suspense` component behaviour is better summarized with the defining state machine:

![suspense finite state machine](assets/suspense%20machine.png)

## Get started

```bash
npm install
npm run start
```

Navigate to [localhost:5000](http://localhost:5000). You should see your app running.

## Deploying to the web

### With [now](https://zeit.co/now)

Install `now` if you haven't already:

```bash
npm install -g now
```

Then, from within your project folder:

```bash
now
```

As an alternative, use the [Now desktop client](https://zeit.co/download) and simply drag the unzipped project folder to the taskbar icon.
