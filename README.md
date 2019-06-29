## Motivation
This is a proof of concept for offering functionality similar to that of React Suspense. It is 
inspired from [previous work](https://reactgo.com/tutorial-reactlazy-react-suspense/) by [Sai 
gowtham](https://twitter.com/saigowthamr) that I reimplemented here with Svelte in lieu of React.

## What it is 
![suspense demo](assets/suspense%20demo.gif)

# Markup and semantics
The main screen which will hold the loaded image is handled with a suspense functionality:

````html
let albums=[];

function fetchAlbums(dispatch, {commands, events, properties, settings}){
  const [TIMER_EXPIRED, SUCCEEDED, FAILED, START] = events;
  axios.get(iTunesUrl)
       .then(res => {albums = res.data.feed.entry})
       .then(() => dispatch({[events[SUCCEEDED]]: void 0}))
}
    
<div class="app">
    <Header />
    <div class="albums">
        <Suspense settings={{run: fetchAlbums, duration: 10}}>
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

The `Album` component also benefits from the suspense functionality:

```javascript
<ul class="album">
    <li class="album-item">
        <Suspense let:dispatch={dispatch} let:events="{events}" settings="{{duration: 0}}">
            <div slot="fallback" class="album-img">
                <img alt="" src="https://media.giphy.com/media/y1ZBcOGOOtlpC/200.gif" />
            </div>
            <a href={link} target="blank" class="link">
                <img class="album-img"
                     on:load="{() => {dispatch({[events[DONE]]: void 0})}}"
                     src={image}
                     alt={'itunes' + Math.random()} />
            </a>
        </Suspense>
    </li>
    <li class="title album-item">
        <a href={link} target="blank" class="link">
            {title.slice(0, 20)}..</a></li>
    <li class="price album-item">Price:{price}</li>
    <li class="date album-item">Released:{formatDate(date, "MMM Do YY")}</li>
</ul>

```

## API
The `Suspense` component admits the following props:
- `settings.duration`: the number of milliseconds to wait before displaying the fallback slot 
content
- `settings.run`: a function to run on starting the `Suspense` component. That function will 
receive the `dispatch` event emitter, and other relevant parameters (See example).

It also pass the `dispatch` event emitter, and the `events` array of admitted events to its 
children slots. The event emitter serves to communicate with the `Suspense` component, to singal 
completion, or error. The `events` array holds the events handled by the `Suspense` component 
(`const events = [TIMER_EXPIRED, SUCCEEDED, FAILED, START];`). Events being passed with an array,
 you can rename the events at will. Here we use `DONE` instead of `SUCCEEDED` for the completion 
 event.

For instance, in the previous example, the `Album` component lazily loads an image. When the 
image is loaded, completion is signaled and the wrapping `<a>` DOM element is displayed.

In the previous example too, in the main `App` component, the list of albums is fetched, using 
the `run` parameter of the `Suspense` component. When that is completed, the completion signal is
 sent to the `Suspense` component and the album list is displayed. 

## Issues
I ran into some issues when developing the component, some of which are linked to Svelte, others 
to the DOM.

Concerning Svelte:
- Svelte being template-based, there is no functional composition available. The next best thing 
is slot composition which is a very flexible mechanism, though the syntax of it can be confusing.
 In particular the necessity to pass parameters and handle variable scoping (which comes for free 
 with functions) requires to understand well the syntax proposed by Svelte.
- [Component do not admit slots (yet)!!](https://github.com/sveltejs/svelte/issues/2080). That is
 a strong limitation for this particular example, and leads to a component whose functionality is
  not very robust (see below). 
- I found what seems to be a bug in that imported variables are not 

DOM-wise:
- because of Svelte current limitations, we had to hide our default slot component by wrapping it
 in an extra `div` and use `display:none` on that div. However, that extra `div` can conflict 
 with the surrounding css, which it does in our examples (the surrounding `div` for an album is a
 flex container, which handles direct children -- it would thus pass to handle a unique `div` 
 children instead of a list of `Album` children).
- we opted for a work-around, which is to display conditionally (with template `#if` logic) the 
default slot component. However this lead to some obscure errors such as `li0 is not defined`. 
I assumed this is connected to Svelte's internal and did not pursue the matter further. For this 
demo, we ended up attached the `Suspense` component to the image loading, instea of the `ul` list
 element, to avoid that error. This is however less user-friendly -- which is the goal of 
 Suspense in the first place.

On the bright side, the targetted API which uses slots looks good:
 ````html
 <Suspense let:dispatch="{dispatch}" let:events="{events}"  {settings} />
   <Fallback slot="fallback" />
   ... #Suuspended content
 </Suspense >
````
 
For inline suspended content, it implies minimal to no modification of the 'suspended' component.
 In the `Album` case, we only added an `onload` handler. In the `App` case, we added the `run` 
 function to fetch data, separately from the view displaying the data, effectively decoupling the
  view and the model within the template.

However a usage which reuses existing components like:
 ````html
 <Suspense let:dispatch="{dispatch}" let:events="{events}"  {settings} />
   <Fallback slot="fallback" />
   <Suspended {dispatch} {events} {settings}/>
 </Suspense >
````

involves wrapping those components in a `Suspended` component to put `dispatch`, `events`, and 
`settings` in scope.

I am new to Svelte so I did get something wrong, I welcome your friendly and helpful comments 
(open an issue). Always happy to hear feedback.

## Using state machines
I implemented `Suspense` with the [Kingly](https://github.com/brucou/kingly) state machine library. 
This is not strictly necessarily and the implementation can be replaced by a simple function with no 
dependency (that is a fundamental objective of Kingly). That function may actually be the result 
of compiling the Kingly machine, just like Svelte compiles its template. A state machine is but a
 template for behaviour.
 
While probably too much for a simple behaviour like Suspense, the technique can however be 
generalized easily to more complex scenarios -- only by modifying the underlying machine. Such a 
complex scenario could be handling the synchronization/scheduling of competing content, just like
 React Concurrent does (with probably less flexibility -- it remains to be understood properly). 
 This would be possible without any change in architecture, or impact in the Svelte compiler. All
  this is very preliminary but those are interesting tracks to follow.

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
