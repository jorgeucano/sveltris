<style>
  :global(body) {
    background-color: black;
    color: white;
  }
  main {
    grid-template-columns: minmax(auto, 1024px);
    display: grid;
    justify-content: center;
    align-items: center;
  }
  .Main_content {
    display: flex;
  }
  .Main_Header {
    margin: 0 0 2em 0;
  }
  figure,
  img {
    width: 600px;
    margin: auto;
    display: block;
  }
</style>

<main>
  <div class="Main_Header">
    <figure>
      <img src="https://arepa.s3.amazonaws.com/sveltris.png" alt="Sveltris" />
    </figure>
  </div>
  <div class="Main_content">
    <slot />
  </div>
</main>
