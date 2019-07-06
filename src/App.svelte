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
           .then(res => {albums = res.data.feed.entry})
           .then(() => done(void 0))
    }

</script>

<div class="app">
    <Header />
    <div class="albums">
        <Suspense task={fetchAlbums} timeout=10>
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
