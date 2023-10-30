<script>
  // @ts-nocheck
  import { fade, blur, fly } from 'svelte/transition'
  import { bounceOut, quintOut, elasticOut } from 'svelte/easing'
  import { spring } from 'svelte/motion'

  let isActive = true

  function blah() {
    return {
      css: (t) => {
        return `
          transform: scale(${t});
        `
      },
      easing: elasticOut,
      duration: 2000,
    }
  }

  function drag(node, { color = 'blue' }) {
    let x
    let y

    const coordinates = spring({
      x: 0,
      y: 0,
    })

    coordinates.subscribe((current) => {
      node.style.transform = `translate3d(${current.x}px, ${current.y}px,0)`
    })

    node.addEventListener('mousedown', mousedown)

    function mousedown(event) {
      x = event.clientX
      y = event.clientY

      window.addEventListener('mouseup', mouseup)
      window.addEventListener('mousemove', mousemove)
    }

    function mouseup() {
      window.removeEventListener('mouseup', mouseup)
      window.removeEventListener('mousemove', mousemove)

      coordinates.update(() => {
        return { x: 0, y: 0 }
      })

      node.dispatchEvent(
        new CustomEvent('dragstop', {
          detail: { x, y, color, node },
        })
      )

      x = 0
      y = 0
    }

    function mousemove(event) {
      const dx = event.clientX - x
      const dy = event.clientY - y

      x = event.clientX
      y = event.clientY

      coordinates.update((current) => {
        return {
          x: current.x + dx,
          y: current.y + dy,
        }
      })
    }
  }
</script>

<main>
  <h1>Animation in Svelte</h1>

  <button on:click={() => (isActive = !isActive)}>
    {#if isActive}
      Hide
    {:else}
      Show
    {/if} Box
  </button>

  {#if isActive}
    <div
      class="box"
      transition:blah
      use:drag={{ color: 'purple' }}
      on:dragstop={(event) => {
        console.log(event.detail)
        event.detail.node.style.backgroundColor = event.detail.color
        if (event.detail.x > 300) {
          isActive = false
          event.detail.node.style.backgroundColor = 'red'
        }
      }}
    />
  {/if}
</main>

<style>
  .box {
    background-color: red;
    display: block;
    width: 100px;
    height: 100px;
    margin: 0 auto;
    margin-top: 0.67rem;
  }
</style>
