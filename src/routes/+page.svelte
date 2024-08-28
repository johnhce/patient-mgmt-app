<script lang="ts">
    import Header from '$lib/Header.svelte'

    const homePage = () => {
      window.location.href = 'patient';
    }
  </script>
  
  <main>
    <Header/>
    <div style="text-align: center;">
      <button class="btn btn-primary justify-center" on:click="{homePage}">
        <svg xmlns="http://www.w3.org/2000/svg" fill="none" viewBox="0 0 24 24" stroke-width="1.5" stroke="currentColor" class="size-6">
          <path stroke-linecap="round" stroke-linejoin="round" d="m12.75 15 3-3m0 0-3-3m3 3h-7.5M21 12a9 9 0 1 1-18 0 9 9 0 0 1 18 0Z" />
        </svg>
        Log In
      </button>
    </div>
  </main>
