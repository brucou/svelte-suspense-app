<script>
  import {commands, events, fsmDef, properties} from "suspense-fsm"
  import {createStateMachine} from "kingly"
  import Machine from "svelte-machine"
  import { LazyLoadContainer, LazyLoad } from 'svelte-lazyload';
  import Album from "./Album.svelte"
  import Header from "./Header.svelte"
  import axios from "axios";
  import "./App.css";

	const iTunesUrl = `https://itunes.apple.com/in/rss/topalbums/limit=100/json`;
	// Define the rendering props
	let albums;
	axios.get(iTunesUrl)
          .then(res => {
		     albums = res.data.feed.entry
	    })

</script>

<div class="app">
  <Header />
  <div class="albums">
    {#if albums }
      <LazyLoadContainer>
        {#each albums as album, i}
          <LazyLoad id="{i}">
            <Album {album} />
          </LazyLoad >
        {/each}
      </LazyLoadContainer>
    {/if }
</div>
    </div>
