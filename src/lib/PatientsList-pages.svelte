<script lang="ts">
  import { Button, DataTable, Modal, Pagination, RadioButton, RadioButtonGroup, TextInput,
    FluidForm, Grid, Row, Column
   } from "carbon-components-svelte";
  import Add from "carbon-icons-svelte/lib/Add.svelte";
  import Delete from "carbon-icons-svelte/lib/Delete.svelte";
  import Edit from "carbon-icons-svelte/lib/Edit.svelte";
  import Search from "carbon-icons-svelte/lib/Search.svelte";

  import axios from "axios";
  import { FHIR_BASE_URL } from "../config";
  import type { Bundle, BundleEntry, Patient } from "fhir/r4";

  import Header from '$lib/Header.svelte'

  import { createForm } from "felte";
  import * as yup from "yup";
    import { WarningOther } from "carbon-icons-svelte";
  //import { redirect } from "@sveltejs/kit";
  
  // for datatable
  interface TableEntry {
    id: number;
    patientId: string;
    name?: string;
    gender?: string;
    dob?: string;
  }

  let pageSize = 10;
  let page = 1;
  let rows:Array<TableEntry> = [];
  let currentRow: number = -1;
  const Theme = 'White';

  let open = false;

  const schema = yup.object().shape({
    // This is the radio button.
    searchType: yup.string().required(),
    searchValue: yup.string()
      .when('searchType', {
        is: 'phone',
        then: yup.number().max(10).required(),
      })
      .when('searchType', {
        is: 'name',
        then: yup.string().required(),
      }),
  }, [['searchType', 'searchValue']]);

  function* iterable_flatten<T>(this: Iterable<T[]>): Generator<T> {
      for (let item of this) {
          for (let itemitem of item)
              yield itemitem;
      }
  }

  const getPatientsList = async () => {
    const patientsResponse = await axios.get<Bundle<BundleEntry<Patient>>>(`${FHIR_BASE_URL}/Patient`);
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
    const results = patientsResponse.data.entry?.filter((entry) => {
      return(entry.resource?.id !== undefined && entry.resource?.id.toString().length > 0);
    }) as BundleEntry<Patient>[];
    for(let i = 0; i < results.length; i++) {
      let te = {} as TableEntry;
      te.id = i;
      te.patientId = (results[i].resource?.identifier?.toString() ?? '');
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
    window.location.href = 'patient/add';
//no funciona bien    redirect(302, 'patient/add');
  }
  function showEditPatient(row: number) {
    // open dialog with title "Edit Patient"
    window.location.href = 'patient/edit?row='+rows[row].patientId.valueOf;
 //no funciona bien   redirect(302, 'patient/edit');
  }
  function editEntry(row: number) {
    currentRow = row;
    showEditPatient(row);
  }
  function deleteEntry(row: number) {
    currentRow = row;
    open = true;  // open a modal: are you sure?
  }
  function searchPatient() {
    // get params from URL
  }
  const deletePatient = async (row: number) => {
    const patientResponse = await axios.delete<Patient>(`${FHIR_BASE_URL}/Patient/${rows[row].patientId}`);
    return patientResponse.data;
  }
</script>

<Header/>
{#await getPatientsList()}
  <div class="mt-10 flex justify-center">
    <span class="loading loading-spinner loading-lg"></span>
    <p>Fetching patient data.  Please wait.</p>
  </div>
{:then patientList} 
  <!-- {#each patientList.entry as patient}
    <li>{patient.name}</li>
  {/each} -->
  <Modal
    bind:open
    modalHeading="Delete this patient?"
    primaryButtonText="Confirm"
    secondaryButtonText="Cancel"
    on:click:button--secondary={() => (open = false)}
    on:click:button--primary={() => {
      open = false;
      deletePatient(currentRow);
    }}
    on:open
    on:close
    on:submit
  >
    <p>Are you sure you want to delete this patient?</p>
  </Modal>
  <form>
  </form>
  <FluidForm>
    <Grid>
      <Row>
        <Column>
          <TextInput labelText="" placeholder="Search criteria" required name="searchValue" />
        </Column>
        <Column>
          <RadioButtonGroup hideLegend name="searchType" orientation="vertical" selected="name">
            <RadioButton labelText="By Name" value="name" />
            <RadioButton labelText="By Phone" value="phone" />
          </RadioButtonGroup>
        </Column>
        <Column>
          <Button icon={Search} on:click={searchPatient}>Search</Button>
          <Button icon={Add} on:click={showAddPatient}>Add Patient</Button>
        </Column>
      </Row>
    </Grid>
  </FluidForm>
  <DataTable
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
        <span style="flex; flex:auto;"> </span>
        <Button style="margin-left: 2em;" icon={Edit} iconDescription="Edit this entry" on:click={() => editEntry(row.id)}></Button>
        <Button style="margin-left: 1em;" icon={Delete} iconDescription="Delete this entry" on:click={() => deleteEntry(row.id)}></Button>
      {/if}
    </svelte:fragment>
  </DataTable>
  <Pagination
    bind:pageSize
    bind:page
    pageSizes={[10,20,50,100]}
    totalItems={rows.length}
  />
{:catch error}
  <p>Error: {error.message}</p>
{/await}
<style>

</style>