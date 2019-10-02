<script>
  export let isactive=true;
</script>

<style>
  .navbar-burger {
    color: #4a4a4a;
    cursor: pointer;
    display: block;
    height: 3.25rem;
    position: relative;
    width: 3.25rem;
    margin-left: auto;
  }

  .navbar-burger span {
    background-color: currentColor;
    display: block;
    height: 1px;
    left: calc(50% - 8px);
    position: absolute;
    transform-origin: center;
    transition-duration: 86ms;
    transition-property: background-color, opacity, transform;
    transition-timing-function: ease-out;
    width: 16px;
  }

  .navbar-burger span:nth-child(1) {
    top: calc(50% - 6px);
  }

  .navbar-burger span:nth-child(2) {
    top: calc(50% - 1px);
  }

  .navbar-burger span:nth-child(3) {
    top: calc(50% + 4px);
  }

  .navbar-burger:hover {
    background-color: rgba(0, 0, 0, 0.05);
  }

  .navbar-burger.is-active span:nth-child(1) {
    transform: translateY(5px) rotate(45deg);
  }

  .navbar-burger.is-active span:nth-child(2) {
    opacity: 0;
  }

  .navbar-burger.is-active span:nth-child(3) {
    transform: translateY(-5px) rotate(-45deg);
  }

</style>

<a role="button" class="navbar-burger" class:is-active={isactive} on:click="{() => isactive = !isactive}" aria-label="menu" aria-expanded="false">
  <span aria-hidden="true"></span>
  <span aria-hidden="true"></span>
  <span aria-hidden="true"></span>
</a>
