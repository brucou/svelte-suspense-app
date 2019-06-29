<script>
  // import { LazyLoadContainer, LazyLoad } from 'svelte-lazyload';
  import  LazyLoad from './LazyLoad.svelte';
  import  LazyLoadContainer from './LazyLoadContainer.svelte';
  import Suspense from "./Suspense.svelte"
  import Album from "./Album.svelte"
  import Header from "./Header.svelte"
  import axios from "axios";
  import "./App.css";

  const iTunesUrl = `https://itunes.apple.com/in/rss/topalbums/limit=100/json`;

	// Define the rendering props
	let albums=[];

	function fetchAlbums(dispatch, {commands, events, properties, settings}){
	  const [TIMER_EXPIRED, SUCCEEDED, FAILED, START] = events;
      axios.get(iTunesUrl)
           .then(res => {albums = res.data.feed.entry})
           .then(() => dispatch({[events[SUCCEEDED]]: void 0}))
    }

</script>

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
