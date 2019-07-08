<script>
  // import { LazyLoadContainer, LazyLoad } from 'svelte-lazyload';
  import  LazyLoad from './LazyLoad.svelte';
  import  LazyLoadContainer from './LazyLoadContainer.svelte';
  import Suspense from "svelte-suspense"
  // import Suspense from "./Suspense.svelte"
  import Album from "./Album.svelte"
  import Header from "./Header.svelte"
  import axios from "axios";
  import "./App.css";

  const iTunesUrl = `https://itunes.apple.com/in/rss/topalbums/limit=100/json`;

	// Define the rendering props
	let albums=[];

	function fetchAlbums(intents, {settings}){
	  const {done, failed} = intents;
      axios.get(iTunesUrl)
           .then(res => res.data.feed.entry)
           .then(done)
           .catch(failed)
    }

</script>

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
