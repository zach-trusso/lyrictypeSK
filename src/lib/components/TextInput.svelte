<script>
    export let placeholder = '';
    export let id = '';
    export let type = 'text';
    export let value = '';
  
    // You can add an event dispatcher if you need to handle events like on:keypress externally
    import { createEventDispatcher } from 'svelte';
    const dispatch = createEventDispatcher();
  </script>
  
  <input
    type={type}
    id={id}
    placeholder={placeholder}
    value={value}
    on:input={$event => dispatch('input', $event.target.value)}
    on:keypress={$event => dispatch('keypress', $event)}
    autocomplete="off"
    spellcheck="false"
  />