<script>
import {NO_OUTPUT, createStateMachine} from "kingly"
import emitonoff from "emitonoff";
import {commands, events, factory, fsmDef, properties} from "suspense-fsm"
// import {commands, events, factory, fsmDef, properties} from "./svelte-suspense-fsm"


// props
export let commandHandlers;
export let effectHandlers;
export let task;
export let timeout;

let done;
let stillLoading;
let errorOccurred;
let data;
const display="inline";
const noDisplay="none";

// Destructure the machine interface (event/command/properties)
const [TIMER_EXPIRED, DONE, FAILED, START] = events;
const [COMMAND_RENDER, RUN, START_TIMER] = commands ;
const [FALLBACK, MAIN, ERR] = properties;

// Event emitter
const getEventEmitterAdapter = emitonoff => {
  const eventEmitter = emitonoff();
  const DUMMY_NAME_SPACE = "_";
  const subscribers = [];

  return {
      next: x => eventEmitter.emit(DUMMY_NAME_SPACE, x),
      complete: () => subscribers.forEach(f => eventEmitter.off(DUMMY_NAME_SPACE, f)),
      subscribe: ({ next: f, error: _, complete: __ }) => {
        return (subscribers.push(f), eventEmitter.on(DUMMY_NAME_SPACE, f));
      }
    }
};
const eventEmitter = getEventEmitterAdapter(emitonoff);
const next = eventEmitter.next.bind(eventEmitter);
// NOTE: this is a Svelte quirk. The imported `events` is not taken into account otherwise
const intents = {
  done: (data) => next({[DONE]: data}),
  failed: (data) => next({[FAILED]: data}),
};

const defaultCommandHandlers = {
  [START_TIMER] : function (dispatch, params, effectHandlers) {
    const duration = params; // in ms
		const { setTimeout } = effectHandlers;
		setTimeout(() => dispatch({[TIMER_EXPIRED]: void 0}), duration)
  },
  [RUN]: function (dispatch, params, effectHandlers) {
    const run = params;
    run(intents, {task, timeout});
  },
  [COMMAND_RENDER]: function (dispatch, params, effectHandlers){
    const display = params.display;
    data = params.data;
    done = display === MAIN;
    stillLoading = display === FALLBACK;
    errorOccurred = display === ERR;
    debugger
  },
}
const defaultEffectHandlers = {
		setTimeout: (fn, duration) => window.setTimeout(fn, duration)
	};
const initEvent = {[START]: void 0}


// Special handling of render command: compose default and custom processing
const customRender =commandHandlers && commandHandlers[COMMAND_RENDER];
const defaultRender = defaultCommandHandlers[COMMAND_RENDER];
let finalRenderHandler={};
if (customRender){
  finalRenderHandler = {
    [COMMAND_RENDER]: function compose(dispatch, params, effectHandlers){
    defaultRender(dispatch, params, effectHandlers);
    customRender (dispatch, params, effectHandlers);
  }
  }
}

const finalCommandHandlers = Object.assign({}, defaultCommandHandlers, commandHandlers, finalRenderHandler);
const finalEffectHandlers = Object.assign({}, defaultEffectHandlers, effectHandlers);

// Create the machine
const fsm = factory(Object.assign({}, {task, timeout}, {debug:{console}}));

// Subscribing to machine events
eventEmitter.subscribe({
	next: event => {
          // 1. Run the input on the machine to obtain the actions to perform
          const actions = fsm(event);

          // 2. Execute the actions, if any
          if (actions === NO_OUTPUT) {return void 0;}
          else {
            const filteredActions = actions.filter(action => action !== NO_OUTPUT);
            filteredActions.forEach(action => {
              const { command, params } = action;

              const commandHandler = finalCommandHandlers[command];
              if (!commandHandler || typeof commandHandler !== "function") {
                throw new Error(
                  `Machine: Could not find command handler for command ${command}!`
                );
              }

              commandHandler(next, params, finalEffectHandlers);
            });

            return void 0;
          }
        },
        error: error => {
          // We may get there for instance if there was a preprocessor throwing an exception
          console.error(
            `<Machine/>: an error in the event processing chain! The machine will not process
            any additional events. Remember that command handlers ought never throw, but should pass errors as events back to the mediator.`,
            error
          );
        },
        complete: () => {
        }
});

	initEvent && next(initEvent)

</script>

<style>
.incognito {
  display: none;
  border: 0;
  padding: 0;
  margin: 0;
}
.cognito {
  display: inherit;
  border: 0;
  padding: 0;
  margin: 0;
}
</style>

{#if stillLoading }
  <slot name="fallback" dispatch={next} intents={intents} ></slot>
{:else if errorOccurred }
  <slot name="error" dispatch={next} intents={intents} data={data}></slot>
{:else if done }
  <slot dispatch={next} intents={intents} data={data}></slot>
{:else }
<div class="incognito">
  <slot dispatch={next} intents={intents} ></slot>
 </div>
{/if }
