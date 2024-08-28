<script lang="ts">
	import { fade } from 'svelte/transition';

  import axios from "axios";
  import { Button, Modal, FormGroup, RadioButtonGroup, RadioButton, TextInput, ToastNotification } from "carbon-components-svelte";
  import { DatePicker, DatePickerInput } from "carbon-components-svelte";
  
  import Add from "carbon-icons-svelte/lib/Add.svelte";
  import Save from "carbon-icons-svelte/lib/Save.svelte";
  import Exit from "carbon-icons-svelte/lib/Exit.svelte";

  import type { Bundle, BundleEntry, Patient } from "fhir/r4";
  import { FHIR_BASE_URL } from "../config";

  import { createForm, getValue } from "felte";
  import * as yup from "yup";

  import TableEntry from "$lib/PatientsList.svelte";
  import { onMount } from 'svelte';

  import { createEventDispatcher } from 'svelte';

  export let mode: string = 'Add'; // 'Add' or 'Edit'
  export let open: boolean = false;
  export let entry: TableEntry;

  let dispatch = createEventDispatcher();

  let showToast = false;
  let toastTimeout = 3000;
  let toastKind = 'success';
  let toastMessage = '';

  // validation schema
  let formValues = {
    name: '',
    gender: 'male',
    dob: '',
  };
  let errors:any = {}

  const loadData = () => {
    try {
      // id exists if this is an update operation
      if (entry.id >= 0) {
        formValues.name = entry.name;
        formValues.gender = entry.gender;
        formValues.dob = entry.dob;
      }
    } catch(ex) { }
  }

  let maxDate: String = new Date(new Date().getDate() + 1).toISOString().split("T")[0];
  const schema = yup.object().shape({
    name: yup.string().required(),
    // dob: yup.date()
    //   .nullable()
    //   .default<Date>(new Date('2000-01-01'))
    //   .required()
    //   .min(new Date('1800-01-01'), 'Date cannot be this early')
    //   .max(maxDate, 'Birth date cannot be in the future')
      // .test('format', 'Date must be in format YYYY-MM-DD',
      //   date => {
      //     if (date.toISOString().length > 0)
      //       return (date.toISOString().substring(0,10).match('/^([0-9]{4})\-([0-9]{2})\-([0-9]{2})$/') != null);
      //           })
  });

//  const errors = schema.validate(formValues);
  // const { form, errors } = createForm({
  //     validate: async (values) => {
  //         try {
  //             await schema.validate(values, { abortEarly: false });
  //         } catch(err: any) {
  //             const errors = err.inner.reduce((res: any, value: any) => ({
  //               [value.path]: value.message,
  //               ...res,
  //             }), {});
  //             return errors;
  //         }
  //     }
  // });

  const validateWithYup = () => {
    try {
      schema.validate(formValues, { abortEarly: false });
    } catch(err: any) {
      errors = err.inner.reduce((res: any, value: any) => ({
        [value.path]: value.message,
        ...res,
      }), {});
      return errors;
    }
  }

  const validateForm = () => {
    errors = {}
    if (formValues.name.length == 0) {
      errors.name = "name is required";
    }
    if (formValues.dob.length == 0) {
      errors.dob = "birth date is required";
    }
  }

  const addPatient = async (name: string, gender: string, dob: string) => {
    let payload = {
      resourceType: 'Patient',
      name: [{
        given: [`${name.split(/\s/)[0]}`],
        family: `${name.split(/\s/)[1]}`
      }],
      gender: `${gender}`,
      birthDate: `${dob}`
    };
    const patientResponse = await axios.post<Patient>(`${FHIR_BASE_URL}/Patient/`, payload).then(
      function (response) {
        toastKind = 'success';
        toastMessage = 'Add successful.'
        showToast = true;
        open = false;
        // // send msg to parent to refresh data
        // dispatch('refreshData', 1);
        return response.data;
      }
    ).catch((error) =>{
alert('error:'+JSON.stringify(error));
    });
  }

  const updatePatient = async (name: string, gender: string, dob: string) => {
    let payload = {
      resourceType: 'Patient',
      name: [{
        given: [`${name.split(/\s/)[0]}`],
        family: `${name.split(/\s/)[1]}`
      }],
      gender: `${gender}`,
      birthDate: `${dob}`
    };
//x can we do a patch instead of a put so other fields are not obliterated?
//x otherwise, see 22:17 on Create/Update Form in tutorial
    const patientResponse = await axios.put<Patient>(`${FHIR_BASE_URL}/Patient/${entry.patientId}`, payload).then(
      function (response) {
        toastKind = 'success';
        toastMessage = 'Update successful.'
        showToast = true;
        open = false;
        // // send msg to parent to refresh data
        // dispatch('refreshData', 1);
        return response.data;
      }
    ).catch((error) =>{
alert('error:'+JSON.stringify(error));
    });
  }
</script>

{#if showToast}
  <div transition:fade>
    <ToastNotification
      timeout={toastTimeout}
      kind={toastKind}
      title="{toastKind === 'success' ? 'Success' : 'Error'}"
      subtitle="{toastMessage}"
      caption={new Date().toLocaleString()}
      on:close
    />
  </div>
{/if}

<Modal
  bind:open
  modalHeading=""
  primaryButtonText={mode}
  secondaryButtonText="Cancel"
  selectorPrimaryFocus="#focus-me"
  on:click:button--primary={() => {
    validateForm();
    if (Object.keys(errors).length === 0) {
      if (mode === 'Add') {
        addPatient(formValues.name, formValues.gender, formValues.dob);
      } else {
        updatePatient(formValues.name, formValues.gender, formValues.dob);
      }
      // send msg to parent to refresh data
      dispatch('refreshData', 1);
    }
  }}
  on:click:button--secondary={() => (open = false)}
  on:open="{() => loadData()}"
  on:close
  on:submit
>
  <div>
    <h1 class="mb-10">{mode} Patient</h1>
    <TextInput id="focus-me" labelText="Name" placeholder="John Q Public" name="name" bind:value="{formValues.name}"/>
    {#if errors.name}
      <span class="text-red-500 text-sm">{errors.name}</span>
    {/if}

    <DatePicker class="mt-5" datePickerType="single" dateFormat="Y-m-d" bind:value="{formValues.dob}">
      <DatePickerInput id="dateInput"
        labelText="Date of birth"
        placeholder="YYYY-MM-DD"
      />
    </DatePicker>
    {#if errors.dob}
      <span class="text-red-500 text-sm">{errors.dob}</span>
    {/if}

    <FormGroup class="mt-5" legendText="Legal Gender (Sex)">
      <RadioButtonGroup name="radio-button-group" bind:selected="{formValues.gender}">
        <RadioButton value="male" labelText="Male" />
        <RadioButton value="female" labelText="Female" />
        <RadioButton value="other" labelText="Other" />
      </RadioButtonGroup>
    </FormGroup>
  </div>
</Modal>
