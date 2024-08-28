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
    document.getElementById('delete_modal')!.showModal();
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
<!-- Open the modal using ID.showModal() method -->
<button class="btn" on:click="{() => (document.getElementById('my_modal_2')).showModal()}">open modal</button>
<dialog id="my_modal_2" class="modal">
  <div class="modal-box">
    <h3 class="text-lg font-bold">Hello!</h3>
    <p class="py-4">Press ESC key or click outside to close</p>
  </div>
  <form method="dialog" class="modal-backdrop">
    <button>close</button>
  </form>
</dialog>

{#await getPatientsList()}
  <div class="mt-10 flex justify-center">
    <span class="loading loading-spinner loading-lg"></span>
    <p>Fetching patient data.  Please wait.</p>
  </div>
{:then patientList}
  <dialog id="delete_modal" class="modal">
    <form method="dialog" class="modal-backdrop">
      <div class="modal-box">
        <h3 class="text-lg font-bold">Delete Patient</h3>
        <p class="py-4">Are you sure you want to delete this patient?</p>
      </div>
      <button type="submit" class="btn btn-error" on:click="{() => deletePatient(currentRow)}">Delete</button>
      <button type="reset" class="btn">Cancel</button>
    </form>
  </dialog>
  <div class="grid grid-cols-6 gap-4 m-3">
    <div class="grid-cols-subgrid col-span-2">
      <label class="input input-bordered gap-2 flex justify-start">
        <input type="text" class="grow" placeholder="Search" />
        <svg
          xmlns="http://www.w3.org/2000/svg"
          viewBox="0 0 16 16"
          fill="currentColor"
          class="h-4 w-4 opacity-70">
          <path
            fill-rule="evenodd"
            d="M9.965 11.026a5 5 0 1 1 1.06-1.06l2.755 2.754a.75.75 0 1 1-1.06 1.06l-2.755-2.754ZM10.5 7a3.5 3.5 0 1 1-7 0 3.5 3.5 0 0 1 7 0Z"
            clip-rule="evenodd" />
        </svg>
      </label>
    </div>
    <div class="ml-5 join">
      <input class="join-item btn" type="radio" name="searchType" checked value="name" aria-label="By name"/>
      <input class="join-item btn" type="radio" name="searchType" value="phone" aria-label="By phone"/>
    </div>
    <div class="grid-cols-subgrid col-span-2">
      <button class="btn btn-primary" style="vertical-align: baseline" on:click={searchPatient}>
        <svg xmlns="http://www.w3.org/2000/svg" fill="none" viewBox="0 0 24 24" stroke-width="1.5" stroke="currentColor" class="size-6">
          <path stroke-linecap="round" stroke-linejoin="round" d="m21 21-5.197-5.197m0 0A7.5 7.5 0 1 0 5.196 5.196a7.5 7.5 0 0 0 10.607 10.607Z" />
        </svg>
        Search
      </button>
    </div>
    <div class="flex justify-end">
      <button class="btn" on:click={showAddPatient}>
        <svg xmlns="http://www.w3.org/2000/svg" fill="none" viewBox="0 0 24 24" stroke-width="1.5" stroke="currentColor" class="size-6">
          <path stroke-linecap="round" stroke-linejoin="round" d="M12 4.5v15m7.5-7.5h-15" />
        </svg>
        Add Patient
      </button>
    </div>
  </div>
  <div class="overflow-x-auto">
    <table class="table table-zebra">
      <!-- head -->
      <thead>
        <tr>
          <th>Name</th>
          <th>Gender</th>
          <th>Date of Birth</th>
          <th></th>
        </tr>
      </thead>
      <tbody>
      {#each patientList.entry as patient}
        <!-- table row -->
        <tr>
          <td>
            <div class="flex items-center gap-3">
              <div class="avatar">
                <div class="mask mask-squircle h-12 w-12">
                  <img
                    src="https://img.daisyui.com/images/profile/demo/2@94.webp"
                    alt="Avatar Tailwind CSS Component" />
                </div>
              </div>
              <div>
                <div class="font-bold">Hart Hagerty</div>
                <div class="text-sm opacity-50">United States</div>
              </div>
            </div>
          </td>
          <td>
            Zemlak, Daniel and Leannon
            <br />
            <span class="badge badge-ghost badge-sm">Desktop Support Technician</span>
          </td>
          <td>Purple</td>
          <th>
            <button class="btn btn-ghost btn-xs">details</button>
          </th>
        </tr>
      {/each}
      </tbody>
    </table>
  </div>
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