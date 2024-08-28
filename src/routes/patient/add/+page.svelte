<script lang="ts">
  import Header from "$lib/Header.svelte";
  import PatientForm from "$lib/PatientForm.svelte";

  import "carbon-components-svelte/css/all.css";

  const returnToHome = () => {
    window.localStorage.href = '/patient';
  }
</script>

<Header/>
<PatientForm mode="Add" patientId="" on:returnHome="{returnToHome}" on:click/>

<style>
</style>
