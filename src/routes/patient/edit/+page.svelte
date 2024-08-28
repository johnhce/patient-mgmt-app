<script lang="ts">
  import PatientForm from "$lib/PatientForm.svelte";
  import "carbon-components-svelte/css/all.css";

  import { page } from '$app/stores'

  let patientId = $page.url.searchParams.get('patientId') ?? ''

  const returnToHome = () => {
    window.localStorage.href = '/patient';
  }
</script>

<PatientForm mode="Edit" patientId="{patientId}" on:returnHome="{returnToHome}" />

<style>
</style>
