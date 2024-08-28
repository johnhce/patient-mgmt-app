<script lang="ts">
	import { fade } from 'svelte/transition';

  import { Button, DataTable, Modal, Pagination, RadioButton, RadioButtonGroup, TextInput,
    FluidForm, Grid, Row, Column, ToastNotification
  } from "carbon-components-svelte";
  import Add from "carbon-icons-svelte/lib/Add.svelte";
  import Delete from "carbon-icons-svelte/lib/Delete.svelte";
  import Edit from "carbon-icons-svelte/lib/Edit.svelte";
  import Search from "carbon-icons-svelte/lib/Search.svelte";

  import PatientModal from './PatientModal.svelte';

  import axios from "axios";
  import { FHIR_BASE_URL } from "../config";
  import type { Bundle, BundleEntry, Patient } from "fhir/r4";

  import Header from '$lib/Header.svelte'

  // import { createForm } from "felte";
  // import * as yup from "yup";
    import { createEventDispatcher } from 'svelte';
  //import { redirect } from "@sveltejs/kit";
  
  // for datatable
  interface TableEntry {
    id: number;
    patientId: string;
    name?: string;
    gender?: string;
    dob?: string;
  }

  let searchText: string = '';
  let searchType: string = 'name';

  let pageSize = 10;
  let page = 1;
  let rows:Array<TableEntry> = [];
  let currentRow: number = -1;
  const Theme = 'White';

  let addOpen = false;
  let deleteOpen = false;

  let mode = '';

  let showToast = false;
  let toastTimeout = 3000;
  let toastKind = 'success';
  let toastMessage = '';

  let dispatch = createEventDispatcher();

  // const schema = yup.object().shape({
  //   // This is the radio button.
  //   searchType: yup.string().required(),
  //   searchValue: yup.string()
  //     .when('searchType', {
  //       is: 'phone',
  //       then: yup.number().max(10).required(),
  //     })
  //     .when('searchType', {
  //       is: 'name',
  //       then: yup.string().required(),
  //     }),
  // }, [['searchType', 'searchValue']]);

  // function* iterable_flatten<T>(this: Iterable<T[]>): Generator<T> {
  //     for (let item of this) {
  //         for (let itemitem of item)
  //             yield itemitem;
  //     }
  // }

  const getPatientsList = async () => {
    let headers = {
      'Cache-Control': 'no-cache'
    };
    let url:string = `${FHIR_BASE_URL}/Patient`;
    if (searchText.length > 0) {
      url += '?' + ((searchType == 'name' ? 'name=' : 'telecom=') + searchText);
    }
    const patientsResponse = await axios.get<Bundle<BundleEntry<Patient>>>(url, {}, headers);
      // .catch((error) =>{
      //   if (error.response) {
      //     // The request was made and the server responded with a status code
      //     // that falls out of the range of 2xx
      //     alert(error.response);
      //     console.log(error.response.data);
      //     console.log(error.response.status);
      //     console.log(error.response.headers);
      //   } else if (error.request) {
      //     // The request was made but no response was received
      //     // `error.request` is an instance of XMLHttpRequest in the browser 
      //     // and an instance of http.ClientRequest in node.js
      //     console.log(error.request);
      //   } else {
      //     // Something happened in setting up the request that triggered an Error
      //     alert(error.message);
      //     console.log('Error', error.message);
      //   }
      //   return error;
      // });

    //IterableIterator<[number, BundleEntry<BundleEntry<Patient>>]>
//    Object.entries(patientsResponse.data.entry?.entries).forEach()
//    let entries: (IterableIterator<[number, BundleEntry<BundleEntry<Patient>>]> | undefined) = patientsResponse.data.entry?.entries();
    const results = patientsResponse.data.entry?.filter((entry:any) => {
      return(entry.resource?.id !== undefined && entry.resource?.id.toString().length > 0);
    }) as BundleEntry<Patient>[];
    rows = [];
    for(let i = 0; i < results.length; i++) {
      let te = {} as TableEntry;
      te.id = i;
      te.patientId = (results[i].resource?.id ?? '');
      te.name = (results[i].resource?.name?.[0].given ?? '') + ' ' + (results[i].resource?.name?.[0].family ?? '');
      te.gender = results[i].resource?.gender ?? '';
      let dt = results[i].resource?.birthDate ?? '0000-00-00';
      // if (dt != undefined)
      //   te.date = formatRelative(new Date(dt), new Date());
      // else
      te.dob = dt;
      rows.push(te);
    }
    return patientsResponse.data
  }

  function showAddPatient() {
    // open dialog with title "Add Patient"
    currentRow = -1;
    mode = 'Add';
    addOpen = true;
  }
  function showEditPatient(row: number) {
    currentRow = row;
    mode = 'Update';
    addOpen = true;
  }
  function showDeleteEntry(row: number) {
    currentRow = row;
    deleteOpen = true;  // open a modal: are you sure?
  }

  const deletePatient = async () => {
    const patientResponse = await axios.delete<Patient>(`${FHIR_BASE_URL}/Patient/${rows[currentRow].patientId}`);
    if (true === handleResponse('Delete', patientResponse.data)) {
      deleteOpen = false;
      getPatientsList();    // refresh page
    }
    return patientResponse.data;
  }
  const handleResponse = (op:string, data:Patient) => {
    if (data.error) {
      toastKind = 'error';
      toastMessage = op + ' failed.'
      showToast = true;
      return false;
    } else {
      toastKind = 'success';
      toastMessage = op + ' successful.'
      showToast = true;
      return true;
    }
  }
</script>

<PatientModal bind:open="{addOpen}" mode="{mode}"
    on:refreshData="{async () => {getPatientsList();}}"
    entry="{rows[currentRow]}"/>

<Modal
  bind:open = "{deleteOpen}"
  danger
  modalHeading="Delete this patient?"
  primaryButtonText="Delete"
  secondaryButtonText="Cancel"
  on:click:button--secondary={() => (deleteOpen = false)}
  on:click:button--primary={() => {
    deletePatient();
  }}
  on:open
  on:close
  on:submit
>
  <p>Are you sure you want to delete this patient?</p>
</Modal>

<Header/>
{#await getPatientsList()}
  <div class="mt-10 flex justify-center">
    <div class="loading loading-spinner loading-lg"></div>
  </div>
  <div class="mt-5 flex justify-center">
    <h4>Fetching patient data.  Please wait.</h4>
  </div>
{:then patientList} 
  <FluidForm>
    <Grid>
      <Row>
        <Column>
          <TextInput labelText="" placeholder="Search criteria" required bind:value="{searchText}" />
        </Column>
        <Column>
          <RadioButtonGroup id="_searchType" hideLegend name="searchType" orientation="vertical" bind:selected="{searchType}">
            <RadioButton labelText="By Name" value="name" />
            <RadioButton labelText="By Phone" value="phone" />
          </RadioButtonGroup>
        </Column>
        <Column>
          <Button icon={Search} on:click={getPatientsList}>Search</Button>
          <Button icon={Add} on:click={showAddPatient}>Add Patient</Button>
        </Column>
      </Row>
    </Grid>
  </FluidForm>
  <DataTable
    class="mt-5"
    sortable
    zebra
    title="Patient List"
    description="List of patients."
    headers={[
      { key: "name", value: "Name", width: "100%" , minWidth: "200px"},
      { key: "gender", value: "Gender", width: "10rem" },
      { key: "dob", value: "Date of Birth", width: "20rem" },
    ]}
    {pageSize}
    {page}
    {rows}
  >
    <svelte:fragment slot="cell" let:row let:cell>
      {cell.value}
      {#if cell.key === "dob"}
        <Button kind="secondary" style="margin-left: 2em;" icon={Edit} iconDescription="Edit this entry" on:click={() => showEditPatient(row.id)}></Button>
        <Button kind="danger" style="margin-left: 1em;" icon={Delete} iconDescription="Delete this entry" on:click={() => showDeleteEntry(row.id)}></Button>
      {/if}
    </svelte:fragment>
  </DataTable>
  <Pagination
    bind:pageSize
    bind:page
    pageSizes={[10,20,50,100]}
    totalItems={rows.length}
  />

  {#if showToast}
    <div transition:fade><!-- class="overlay">-->
      <ToastNotification
        timeout={toastTimeout}
        kind="{toastKind}"
        title="{toastKind === 'success' ? 'Success' : 'Error'}"
        subtitle="{toastMessage}"
        on:close
      />
    </div>
  {/if}
{:catch error}
  <p>Error: {error.message}</p>
{/await}
<style>

</style>