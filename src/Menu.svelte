<script>
  import { createEventDispatcher, onMount } from 'svelte';
  import Burger from './Burger.svelte';

  const dispatch = createEventDispatcher();


  let timeInput;
  export let isopen = true;
  export let time;

  function handleKeydown(event) {
    let key = event.key;
    let keyCode = event.keyCode;
    
    //Esc
    if (keyCode == 27) {
      isopen = !isopen; 
    }

  }

  $: { dispatch('message', { open: isopen }); }

</script>


<style>
  aside.menu {
    padding: 2em;
    background: #fafafa;
    position: relative;
    right: -120%;
    transition: right 0.4s linear;
    overflow: hidden;
    height: 70%;
  }	
  aside.show{
    right: 0;
  }
  aside.menu ul {
    font-size: 80%;
  }

</style>

<svelte:window on:keydown={handleKeydown}/>

<svelte:head>
</svelte:head>

<Burger bind:isactive={isopen} />

<aside class="menu" class:show={isopen}>
  <slot>
  </slot>
</aside>
