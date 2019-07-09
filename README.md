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

function fetchAlbums(intents){
  const {done, failed} = intents;
  axios.get(iTunesUrl)
       .then(res => res.data.feed.entry)
       .then(done)
       .catch(failed)
}
 
<div class="app">
    <Header />
    <div class="albums">
        <Suspense task={fetchAlbums} let:data={albums} timeout=10>
            <div slot="fallback" class="album-img">
                <img alt="loading" src="https://media.giphy.com/media/y1ZBcOGOOtlpC/200.gif" />
            </div>
            <div slot="error" class="album-img">
                <h1>ERROR!</h1>
            </div>
            <LazyLoadContainer>
                {#if albums}
                  {#each albums as album, i}
                  <LazyLoad id="{i}">
                      <Album {album} />
                  </LazyLoad >
                  {/each}
                {/if }
            </LazyLoadContainer>
        </Suspense>
    </div>
</div>

````

The default slot content of the `Suspense` component will be immediately put in the DOM and 
hidden (`display: none`). Two things can happen:

- completion is signalled (`done` or `failed`) before `timeout` milliseconds. In that case, the corresponding slot content (default slot or `error` slot) is displayed as soon as the completion signal is processed
- completion is not signalled before `timeout` milliseconds expires. In that case, the fallback content (e.g. the `fallback` slot content) will be displayed. When the default slot content signals completion, it will be displayed. When completion is signalled (`done` or `failed`), the corresponding slot content (default slot or `error` slot) is displayed as soon as the completion signal is processed
 
The `Suspense` component behaviour is better summarized with the defining state machine:

![suspense finite state machine](assets/suspense%20machine.png)

The `Album` component also benefits from the suspense functionality:

```javascript
<ul class="album">
    <li class="album-item">
        <Suspense let:intents={{done, failed}} timeout=0 >
            <div slot="fallback" class="album-img">
                <img alt="" src="https://media.giphy.com/media/y1ZBcOGOOtlpC/200.gif" />
            </div>
            <a href={link} target="blank" class="link">
                <img class="album-img"
                     on:load="{done}"
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
- `timeout`: the number of milliseconds to wait before displaying the fallback slot 
content
- `task`: a function to run on starting the `Suspense` component. That function will 
receive an object parameter `{done, failed}` which can be used to indicate termination or failure
 to the underlying suspense state machine. 

The `Suspense` component also passes on the `intents` parameter to its children slots. The 
aforementioned parameter is an object with properties `{done, failed}` which can be used to 
indicate termination or failure to the underlying suspense state machine. Both properties are 
function which can also pass event data if necessary.  

For instance, in the previous example, the `Album` component lazily loads an image. When the 
image is loaded, completion is signaled and the wrapping `<a>` DOM element is displayed. In this 
example, there is no need to pass data with the `done` or `failed` events. 

In the previous example too, in the main `App` component, the list of albums is fetched, using 
the `task` parameter of the `Suspense` component. When that is completed, the completion signal with the fetched albums as data is
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
 <Suspense task={fetchAlbums} timeout=10>
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
 <Suspense let:intents={{done, failed}} timeout=0>
   <Fallback slot="fallback" />
   <Suspended {intents} />
 </Suspense >
````

involves wrapping those components in a `Suspended` component to put `intents`  in scope.

I am new to Svelte so I did get something wrong, I welcome your friendly and helpful comments 
(open an issue). Always happy to hear feedback.

## Using state machines
I implemented `Suspense` with the [Kingly](https://github.com/brucou/kingly) state machine library. 
This is not strictly necessarily and the implementation can be replaced by a simple function with no 
dependency (that is a fundamental objective of Kingly). That function may actually be the result 
of compiling the Kingly machine, just like Svelte compiles its template. A state machine is but a
 template for behaviour.
 
While using a formalized state machine may feel like too much for a simple behaviour like 
Suspense, the technique can however be generalized easily to more complex scenarios -- only by 
modifying the underlying machine. 

A more complex scenario could be to implement a more sophisticated version of suspense which 
deals with a full range of lazy loading formula as described in this excellent [CSS tricks article](https://css-tricks.com/the-complete-guide-to-lazy-loading-images/). 

Another complex scenario could be handling the synchronization/scheduling of competing content, 
similar to but with less flexibility than React Concurrent. This would be possible without any change in architecture, or impact in the Svelte compiler. All this is very preliminary but those are interesting tracks to follow.

## Get started
```bash
npm install
npm run start
```

Navigate to [localhost:5000](http://localhost:5000). You should see your app running.

