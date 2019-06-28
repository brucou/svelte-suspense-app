<script>
import {commands, events, fsmDef, properties} from "suspense-fsm"
import {createStateMachine} from "kingly"
import moment from 'moment';
import Machine from "svelte-machine"
import Spinner from "./Spinner.svelte"
import './albums.css'

export let album;
$: image=album["im:image"][2].label
$: title=album.title.label
$: link=album.id.label
$: price=album["im:price"].label
$: date=album["im:releaseDate"].label

let imageLoaded = false;
let stillLoading  = false;
let errorOccurred = false;

function formatDate(date, format) {
  return moment(date).format(format);
};

// Destructure the machine interface (event/command/properties)
const [TIMER_EXPIRED, DONE, FAILED, START] = events;
const [COMMAND_RENDER, RUN, START_TIMER] = commands ;
const [FALLBACK, MAIN, ERR] = properties;

// Create the machine
const fsm = createStateMachine(fsmDef, { debug: {console} });

  // Command handlers
  const commandHandlers = {
    [START_TIMER] : function (dispatch, params, effectHandlers) {
      const duration = params; // in ms
		const { setTimeout } = effectHandlers;
		setTimeout(() => dispatch({[TIMER_EXPIRED]: void 0}), duration)
    },
	[RUN]: function (dispatch, params, effectHandlers) {	},
    [COMMAND_RENDER]: function (dispatch, params, effectHandlers){
      const {display, data} = params;
      imageLoaded = display === MAIN;
      stillLoading = display === FALLBACK;
      errorOccurred = display === ERR;
    },
  }
  const effectHandlers = {
		setTimeout: (fn, duration) => window.setTimeout(fn, duration)
	};
  const initEvent = {[START]: void 0}

</script>

<Machine let:dispatch={dispatch} {commandHandlers} {fsm} {initEvent} {effectHandlers}>
    <ul class="album">
      <li class="album-item">
        {#if stillLoading }
          <Spinner />
        {/if }
         <a href={link} target="blank" class="link">
                <img class="album-img" hidden={!imageLoaded}
                on:load="{() => {console.log(`nsnfdlkn`); dispatch({[DONE]: void 0})}}"
                style="min-height: 200px"
                src={image}
                alt={'itunes' + Math.random()} />
         </a>
      </li>
      <li class="title album-item">
          <a href={link} target="blank" class="link">
              {title.slice(0, 20)}..</a></li>
      <li class="price album-item">Price:{price}</li>
      <li class="date album-item">Released:{formatDate(date, "MMM Do YY")}</li>

    </ul>
</Machine>

