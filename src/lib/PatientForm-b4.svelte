<script lang="ts">
  import axios from "axios";
  import { Button, Form, FormGroup, RadioButtonGroup, RadioButton, TextInput } from "carbon-components-svelte";
  import { DatePicker, DatePickerInput } from "carbon-components-svelte";
  import Add from "carbon-icons-svelte/lib/Add.svelte";
  import Save from "carbon-icons-svelte/lib/Save.svelte";
  import Exit from "carbon-icons-svelte/lib/Exit.svelte";

  import type { Bundle, BundleEntry, Patient } from "fhir/r4";
  import { FHIR_BASE_URL } from "../config";

  import { createForm } from "felte";
  import * as yup from "yup";

  import { createEventDispatcher } from 'svelte';

  export let mode: string = 'Add'; // 'Add' or 'Edit'
  export let patientId: string;

	let dispatch = createEventDispatcher();

  // validation schema
  let formValues = {
    name: '',
    genderIdentity: '', // wrong email format
    dob: '',
  };

  let maxDate: String = new Date(new Date().getDate() + 1).toISOString().split("T")[0];
  const schema = yup.object().shape({
    name: yup.string().required(),
    genderIdentity: yup.string(),
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
  const { form, errors } = createForm({
      validate: async (values) => {
          try {
              await schema.validate(values, { abortEarly: false });
          } catch(err: any) {
              const errors = err.inner.reduce((res: any, value: any) => ({
                [value.path]: value.message,
                ...res,
              }), {});
              return errors;
          }
      }
  });

  const addPatient = async () => {
    const patientResponse = await axios.post<Patient>(`${FHIR_BASE_URL}/Patient/${patientId}`, {
      //x body goes here
    });
alert(JSON.stringify(patientResponse.data));
		dispatch('returnHome', 1);
    return patientResponse.data
  }

  const updatePatient = async () => {
    const patientResponse = await axios.put<Patient>(`${FHIR_BASE_URL}/Patient/${patientId}`, {
      //x body goes here
    });
alert(JSON.stringify(patientResponse.data));
    dispatch('returnHome', 1);
    return patientResponse.data
  }

  const returnToHome = () => {
    window.localStorage.href = '/patient';
  }
</script>

<div class="flex flex-col items-center justify-center w-screen h-screen bg-gray-200 text-gray-700">
  <h1>{mode} Patient</h1>
  <!-- use:form actions cannot be applied to components -->
  <Form action="#" method="post" on:submit="{(ev) => ev.preventDefault()}">
    <TextInput labelText="Name" placeholder="John Q Public" name="name" bind:value="{formValues.name}"/>
    {#if errors.name}
      <span class="text-red-500 text-sm">{errors.name}</span>
    {/if}

    <p style="margin-top: 20px"/>
    <FormGroup legendText="Legal Gender (Sex)">
      <RadioButtonGroup name="radio-button-group" selected="male">
        <RadioButton value="male" labelText="Male" />
        <RadioButton value="female" labelText="Female" />
        <RadioButton value="other" labelText="Other" />
      </RadioButtonGroup>
    </FormGroup>

    <DatePicker datePickerType="single" on:change dateFormat="Y-m-d">
      <DatePickerInput
        labelText="Date of birth"
        placeholder="YYYY-MM-DD"
        bind:value="{formValues.dob}"
      />
    </DatePicker>
    {#if errors.dob}
      <span class="text-red-500 text-sm">{errors.dob}</span>
    {/if}

    {#if mode === 'Add'}
      <Button icon={Add} on:click={addPatient}>Add Patient</Button>
    {:else}
      <Button icon={Save} on:click={updatePatient}>Update Patient</Button>
    {/if}
    <Button icon="{Exit}" on:click={returnToHome}>Cancel</Button>
  </Form>
</div>