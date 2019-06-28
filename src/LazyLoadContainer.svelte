<script>
 import { key } from './util.js';
 import { setContext } from 'svelte';
 export let threshold = 0.01;
 let cb = {};
 const config = {
     root: null,
     rootMargin: '0px',
     threshold: threshold
 };
 const observer = new IntersectionObserver(entries => {
     entries.forEach(entry => {
         if (entry.isIntersecting) {
             /* console.log('loading', entry.target.id); */
             cb[entry.target.id]();
             observer.unobserve(entry.target);
         }
     });
 }, config);
 setContext(key, {
	 registerObserver: (el, fct) => {
         cb[el.id] = fct;
         observer.observe(el);
     }
 });
</script>

<slot></slot>
